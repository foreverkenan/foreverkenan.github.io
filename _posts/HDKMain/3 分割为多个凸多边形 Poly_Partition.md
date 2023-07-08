---
layout: post
title:  "HDK主线 分割为多个凸多边形 Poly_Partition"
comments: true
categories: HDKMain
---

[Poly_Partition](https://www.bilibili.com/video/BV1yZ4y1a7v9)

纯纯的调库节点，还是CGAL，内容比较简单，所以很适合教新手过一遍流程

---

# <font size=5 color = YellowGreen>创建点组并给点打组</font>
打组 Group 原版里
```c
std::string grpname = "poly";
GA_PointGroup* ptgrp = gdp->newPointGroup(grpname);
ptgrp->addOffset(ptoff);

ptgrp->addIndex(ptidx);
```
使用Offset和Index添加都可以

对组的操作会在以后讲解

---
---
1. 读取
使用循环，则每个点的坐标均会读到每次循环的 Pvalue 中
```c
GA_Offset ptoff;
GA_RWHandleV3 Phandle(gdp->findAttribute(GA_ATTRIB_POINT, "P"));

GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	UT_Vector3 Pvalue = Phandle.get(ptoff);
}
```
但这样还不够，我们还需要把上面的代码改进一下
我们用的是 CGAL::optimal_convex_partition_2 函数（太长了，简称 ocp2 函数）
看文档与例子可知，需要把坐标存到 Polygon_2 polyon 中，由于 ocp2 函数是二维的，所以得把Houdini的东西 “ 抄成它的格式 ” 
```c
Polygon_2 poly;

GA_Offset ptoff;
GA_RWHandleV3 Phandle(gdp->findAttribute(GA_ATTRIB_POINT, "P"));

GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	UT_Vector3 Pvalue = Phandle.get(ptoff);
	poly.push_back(Point_2(Pvalue.x(), Pvalue.z()));
}
```
现在我们已经成功把Houdini场景中的点数据，存进 poly 中了

2. 使用函数
这个函数的输出在 partition_polys 中
```c
Polygon_list partition_polys;
CGAL::optimal_convex_partition_2(poly.vertices_begin(),poly.vertices_end(),std::back_inserter(partition_polys));
```

3. 输出
`老手请无视此句---如果你是第一次搞这种东西的话，千万不要想当然，写个 for int，它可能不是int类型，也不要想当然 i++ ，有时候他不能这么加，要用 i.addvance() `
```c
for(int i=0;i<=xxx;i++)//maybe error

for (GA_Iterator it(input_geo1->getPrimitiveRange()); !it.atEnd(); it.advance())
for (GA_Iterator it(range); it.blockAdvance(start, end);)
```
回到正文，这里要这么做循环，for 中的是每个 poly
```c
for (std::list<Polygon_2>::iterator listit = partition_polys.begin();listit != partition_polys.end(); listit++)
{
	//each poly
}
```
为了方便Houdini的生成，对每个面还要做一次循环，生成边，里面的 for 的每个边
```c
for (std::list<Polygon_2>::iterator listit = partition_polys.begin();listit != partition_polys.end(); listit++)
{
	for (Traits::Polygon_2::Edge_const_iterator polyedgeit = listit->edges_begin();polyedgeit != listit->edges_end(); polyedgeit++)
	{
		//each edge
	}
}
```

剩下的大部分工作就在内层 for 中进行
从边中读取两个端点
```c
Point_2 p2pt0 = polyedgeit->point(0);
Point_2 p2pt1 = polyedgeit->point(1);
```
这个 Point_2 是CGAL的类型，要转为 UT_Vector3 才能给Houdini使用，跟第一步读取的时候正好相反
```c
UT_Vector3 v3pt0(p2pt0.x(), 0, p2pt0.y());
UT_Vector3 v3pt1(p2pt1.x(), 0, p2pt1.y());
```
其他东西以前都讲过了，不重复了

[CGAL 2D Polygon Partitioning 总览](https://doc.cgal.org/latest/Partition_2/index.html)
[CGAL 2D Polygon Partitioning Example](https://doc.cgal.org/latest/Partition_2/examples.html)