[Quick_RoofNEW](https://www.bilibili.com/video/BV14B4y1i7zK)

这个节点是我的第二个节点，同时也是第一次调用CGAL库，这个节点实现的效果与原版SOP_PolyExtrude一样，也可以通过[HDK中调用PolyExtrude来实现](https://www.sidefx.com/forum/topic/75466/)，但为了学习HDK与CGAL库同时顺带Dijkstra，我还是尝试完成了这个节点，由于一些不明的原因，这个节点目前仍然很容易崩溃，应该是预先的判断不够多，造成使用时的错误指针

这个节点是我的第二个节点，同时也是第一次调用CGAL库，单看最终效果，这个节点与原版 SOP_PolyExtrude 一样（当然，需要一些参数修改），但为了学习HDK与CGAL库同时顺带使用 Dijkstra ，我还是尝试完成了这个节点

---

# <font size=5 color = YellowGreen>获取总点数</font>
与VEX中 npoints 函数效果相同
```c
int numpt = gdp->getNumPoints();
```
总面数、总顶点数，可举一反三

# <font size=5 color = YellowGreen>读取点坐标</font>
如VEX中 point(输入,'P',点序号) 相同
```c
GA_RWHandleV3 Phandle(gdp->findAttribute(GA_ATTRIB_POINT, "P"));
UT_Vector3 Pvalue = Phandle.get(ptoff);
```
读取点/线/面的任意属性，可举一反三

# <font size=5 color = YellowGreen>实现fuse合并点</font>
与SOP_Fuse合并点相同

这个函数已经停用了，'GU_Detail::snapPoints': Deprecated since version 17.5. Use GU_Snap instead.
第一个参数 0 表示了合并后的位置，填 0 便是中间
 The type is 0 for average, 1 for round to lowest point number and 2 for highest point number.
 但我的测试中，这个函数只改了点位置，没有把点数量减少，点只会重合在那里
```c
gdp->snapPoints(0, 0.1, 0, true, 0, true);
```

这个函数也停用了，'GU_Detail::consolidatePoints': Deprecated since version 17.5. Use GU_Snap instead.
使用这个函数，点数量减少了，但好像不能选择将合并后的位置定在两点之间(Average)
```c
gdp->consolidatePoints(0.1, 0, false, false, false);
```

这个也是停用的。'GU_Detail::fastConsolidatePoints': Deprecated since version 17.5. Use GU_Snap instead.
```c
gdp->fastConsolidatePoints(0.1, 0, false, false);
```

GU_Snap 是目前最新的 fuse ，但是我至今没有找到正确写法，会崩Houdini，报错 Segmentation fault或是直接关闭Houdini
由于近期没有时间研究，先放一个目前失败的代码
```c
// have problem
GU_Detail qgdp;
qgdp.copy(*detail);
GU_Snap::PointSnapParms parms;
parms.myDistance = 1;
parms.myAlgorithm = GU_Snap::PointSnapParms::SnapAlgorithm::ALGORITHM_LOWEST_POINT;//may be SOP_FUSE "using" what
GU_Snap::AttributeMergeMethod m = GU_Snap::AttributeMergeMethod::MERGE_ATTRIBUTE_MEAN;
// ... may be some i dont know
GU_Snap::snapPoints(qgdp, detail, parms);
```

# <font size=5 color = YellowGreen>实现neighbor邻居点</font>
与VEX中 neighbor 函数相同
buildRingZeroPoints 会将 gdp 中所有连接性关系全部返回，如果你只需要拿到某个点（程序中的 i ）的所有邻居点（程序中的 neighbors ）
```c
int i = 0;
UT_Array<UT_ValArray<GA_Offset> > ringz;
UT_ValArray<int>* ringv = nullptr;

detail->buildRingZeroPoints(ringz, ringv);

std::vector<int> neighbors;
for (int j = 0; j < ringz[i].size(); j++)
{
	GA_Offset rz = ringz[i][j];
	int rzpt = detail->pointIndex(rz);
	neighbors.push_back(rzpt);
}
```
用不到半边，可以直接写nullptr
```c
UT_Array<UT_ValArray<GA_Offset>> ringz;   //这里应该是GA_Index
detail->buildRingZeroPoints(ringz, nullptr);
std::vector<std::vector<GA_Index>> neighbors(ringz.size());

for (GA_Index i = 0; i < ringz.size(); i++)
{
	for (int j = 0; j < ringz[i].size(); j++)
	{
		GA_Offset rz = ringz[i][j];
		GA_Index rzpt = detail->pointIndex(rz);
		neighbors[i].push_back(rzpt);
	}
}
```

---
---
[HOUDINI HDK 屋顶无声讲解](https://www.bilibili.com/video/BV1Hg411f764)已经讲思路了，这里不多说
Dijkstra封面思路在[CSDN小胖七少爷-任意多边形三维屋顶自动生成算法](https://blog.csdn.net/weixin_43712770/article/details/104986220)，这篇文章被多次转载，所以我不能确定此处一定为原作者

由于这前两个节点已经出过讲解视频，所以这个板块不多写
下篇《3. Houdini HDK 分割为多个凸多边形 Poly_Partition》会在这个板块讲讲思路