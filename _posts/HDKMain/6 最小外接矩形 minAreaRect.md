[minAreaRect](https://www.bilibili.com/video/BV1ae411T7Xr)

调库，OpenCV，这个节点是为了给后面的凹多边形的最大内接做铺垫
这节点没啥好说的，也没啥新知识

---

# <font size=5 color = YellowGreen>Matrix赋值</font>
```c
UT_Matrix4 sc = { precision,        0,        0,0,
						  0,precision,        0,0,
						  0,        0,precision,0,
						  0,        0,        0,1 };
```

---
---

同时存在两个 GA_FOR 会报错
```c
GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	// ...
}

// ...

GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	// ...
}
```
解决办法是给其中一个 GA_FOR 加个 { }
```c
GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	// ...
}

// ...

{
	GA_FOR_ALL_PTOFF(gdp, ptoff)
	{
		// ...
	}
}
```
原因是 GA_FOR 本质是一个 define，当 GA_Offset	lcl_start, lcl_end; 替换到你写的代码里，就重复定义了
```c
#define GA_FOR_ALL_PTOFF(gdp, ptoff)	\
    GA_Offset	lcl_start, lcl_end;	\
    for (GA_Iterator lcl_it((gdp)->getPointRange()); lcl_it.blockAdvance(lcl_start, lcl_end); )	\
	for (ptoff = lcl_start; ptoff < lcl_end; ++ptoff)
```