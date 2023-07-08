---
layout: post
title:  "HDK主线 1 最大矩形 Maximal_Rectangle_ALL"
comments: true
categories: HDKMain
---

.[Maximal_Rectangle_ALL](https://www.bilibili.com/video/BV1WY4y1Y7DG)

这是我第一个HDK节点。主要意义在于搞清楚流程，以及尝试一些雨落柏拉图的divide节点中没有用到的东西。

---

开头的几篇，均使用主要代码写在OP_ERROR中的方式，与雨落柏拉图的divide节点相同。直接把代码写在
```c
OP_ERROR xxx::cookMySop(OP_Context& context)
{
	// -> here <-
}
```
同时，如果在开头几篇我给出的代码中，你发现了 detail ，如
```c
	detail->getBBox(&bbox);

	GA_RWHandleV3 gencolor(detail->addFloatTuple(GA_ATTRIB_POINT, "Cd", 3));
```
这说明我反复修改这里的示例代码，改完没有再次检查。你可以直接把 detail 改成 gdp ，即可正常使用
```c
	gdp->getBBox(&bbox);

	GA_RWHandleV3 gencolor(gdp->addFloatTuple(GA_ATTRIB_POINT, "Cd", 3));
```
关于 detail 会在后面讲

---

# <font size=5 color = YellowGreen>含变量的赋值</font>

我们知道使用 { } 进行常量赋值
```c
UT_Vector3 pos = {1, 2, 3};
```
但如果其中带有变量，则需要使用括号的方式
```c
int z = 3;
UT_Vector3 pos(1, 2, z);
```
这种方法类似于VEX使用 set( )
```c
int z = 3;//vex
vector pos = set(1, 2, z);//vex
```
但也可以这样
```c
int a = 5;
UT_Vector3 pos = { 3,4,float(a) };

int t = 8;
UT_Vector3 pos2;
pos2 = { 6,7,float(t) };
```

# <font size=5 color = YellowGreen>printf调试与cout调试</font>
能打印是非常重要的，可以帮助我们找到问题
```c
int a = 3;
float b = 5.6;
fpreal32 c = 9.4;
UT_Vector3 d = { 6,7,8 };
UT_Vector3 e(a, b, c);

printf("a:%d b:%f c:%f\n", a, b, c);
std::cout << d << " " << e;
```
打印结果会与VEX中使用 printf 一样，显示在这个白框，如下图
![[Pasted image 20221031121037.png]]
这里的 fpreal 实质就是 float 或者 double，通过查找可得
```c
using fpreal32 = float;
using fpreal64 = double;
```
我建议用 cout ，因为比较方便，不用想 printf 的 %

# <font size=5 color = YellowGreen>求BoundingBox</font>
与VEX中 getbbox 和Hscript中 bbox 效果一样
核心代码为
```c
UT_BoundingBox bbox;
gdp->getBBox(&bbox);
```
获取也很简单，不过在我测试下， centerX() 与 xcenter() 的结果在数值是一样的，我不知道有什么区别
```c
float cx = bbox.centerX();
float xc = bbox.xcenter();
float yi = bbox.ymin();
float ya = bbox.ymax();
```

# <font size=5 color = YellowGreen>给点加颜色Cd属性</font>
核心代码为
```c
GA_RWHandleV3 gencolor(gdp->addFloatTuple(GA_ATTRIB_POINT, "Cd", 3));
gencolor.set(ptoff,color);
```

如果加 int、float 的属性，就分别改成 GA_RWHandleI、GA_RWHandleF，中间的函数改 Int、不变，都需要把最后的 3 改成 1

创建一个点，并赋予红色的颜色，此处注释掉的 appendPoint() 已经弃用了，替代他的是 appendPointOffset() ，你非要用的话好像也没啥问题
关于 GA_Offset ，会在后面讲，你可以暂时当他是个类似 int 的东西，先这么用着吧
```c
GA_Offset ptoff = gdp->appendPointOffset();
//GA_Offset ptoff2 = gdp->appendPoint();
UT_Vector3 color = { 1,0,0 };

GA_RWHandleV3 gencolor(gdp->addFloatTuple(GA_ATTRIB_POINT, "Cd", 3));
gencolor.set(ptoff, color);
```

# <font size=5 color = YellowGreen>求intersect</font>
与VEX中 intersect 函数一样
但这个函数已经停用，但又没说用啥instead，'GU_Detail::intersectRay': Deprecated since version 16.0

从 { 0,1,0 } 朝下(即y负方向)发射射线，距离为2，因此你可以直接创建一个 grid ，把 grid位置设为 { 0,0.5,0 }，所求的交点会储存在 pos 中，可以试试在后面跟一个 cout ，打印 pos 的值是否为 { 0,0.5,0 }
```c
UT_Vector3 opos = { 0,1,0 };
UT_Vector3 dir = { 0,-1,0 };
float itdistance = 2;
UT_Vector3 pos = { 0,1,0 };
gdp->intersectRay(opos, dir, 2, 2, &itdistance, &pos);
```

我猜测新函数应该是使用 GU_RayIntersect
用法变复杂了，射线相交面序号为 primidx ，相交面偏移量为 primoff ，相交于面上的 uv 坐标为 UVpos  ，在世界空间中坐标为 pos
如果不使用 if 判断，无交点时会直接崩Houdini，所以写HDK一定要小心，一定要多考虑各种奇奇怪怪的情况，不然就是崩整个工程
情景同上，不多说，你可以修改数值和情景，挨个打印看看结果
```c
UT_Vector3 org = { 0,1,0 };
UT_Vector3 dir = { 0,-2,0 };
GU_RayIntersect ri;
GU_RayInfo hitinfo;
hitinfo.init();
ri.init(gdp);
int rayret = ri.sendRay(org, dir, hitinfo);
if (rayret != 0)
{
	GA_Offset primoff = hitinfo.myPrim.offset();
	GA_Index primidx = gdp->primitiveIndex(primoff);
	UT_Vector2 UVpos(hitinfo.myU, hitinfo.myV);
	const GEO_Primitive* hitprim = hitinfo.myPrim;
	UT_Vector4 pos;
	hitprim->evaluateInteriorPoint(pos, hitinfo.myU, hitinfo.myV, hitinfo.myW);
}
```

---
---
这个节点主要思路在[Houdini PCG 平面分割内接矩形思路](https://www.bilibili.com/video/BV1UQ4y1m7z5)里已经讲了，这里不多说。