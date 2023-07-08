[Shape_Regularization](https://www.bilibili.com/video/BV16U4y1i7bm)

这个节点也就是一个简单调库节点，仍然CGAL，和上一篇没啥区别，所以我会加一些新知识进来。趁着这个节点，我们尝试使用 PRM_Template 和 THEDSFILE (the Ds File) 两种方式来做参数面板。关于 THEDSFILE ，会有专门的支线来从头讲，不然直接搞这么复杂的可能会看懵。我建议可以跳过 THEDSFILE 部分，等学过基础再来重看。支线还没开始写，等以后吧

---

# <font size=5 color = YellowGreen>参数面板制作下拉菜单（ordered menu）</font>
下拉菜单在Houdini中的原文为 ordered menu ，百度翻译出来是 有序菜单，

头文件
```c
// .h
private:
	int PARM_SETMODE(fpreal t) { return evalInt("setmode", 0, t); }
```
cpp文件
```c
// .cpp
PRM_Name modeOrdName("setmode", "SetMode");
PRM_Name modeOrdChoices[] =
{
    PRM_Name("mode1", "3.1 Closed Contours"),
    PRM_Name("mode2", "3.2 Open Contours"),
    PRM_Name(0)
};
PRM_ChoiceList modeOrdinalMenu(PRM_CHOICELIST_SINGLE, modeOrdChoices);

PRM_Template Shape_Regularization::myTemplateList[] =
{
    PRM_Template(PRM_ORD_J, 1, &modeOrdName, 0, &modeOrdinalMenu),
    PRM_Template(),
};
```
OP_ERROR
```c
// .cpp OP_ERROR
fpreal now = context.getTime();
int setmode = PARM_SETMODE(now);
if (setmode == 0)
{
	// ...
}
else if (setmode == 1)
{
	// ...
}
```
此三处最终效果如图
![[Pasted image 20221106221832.png]]

`------------------------------THEDSFILE部分------------------------------`
由于这篇比较简单，那我再写一下使用 THEDSFILE 制作这个功能，这部分建议等看过支线再来
在代码层面，请忽略 mode3 this is THEDSFILE ，这是为了更好区分所以我故意加这个mode3
cpp文件
```c
// .cpp
static const char* theDsFile = R"THEDSFILE(
{
    name        parameters
    parm {
        name    "setmode"
        label   "Set Mode"
        type    ordinal
        default { "0" }
        menu {
            "mode1"     "3.1 Closed Contours"
            "mode2"     "3.2 Open Contours"
            "mode3"     "this is THEDSFILE"
        }
    }
}
)THEDSFILE";
```
void
```c
// .cpp void
const CeshiParms::Setmode UserChooseMode = sopparms.getSetmode();
if (UserChooseMode == CeshiParms::Setmode::MODE1)
{
	// ...
}
else if (UserChooseMode == CeshiParms::Setmode::MODE2)
{
	// ...
}
```
此两处最终效果如图
![[Pasted image 20221108144754.png]]
如果你觉得用 if 显得很长，那也可以换成 switch ，其实switch也挺长的，c语言就不演示了
`------------------------------THEDSFILE部分------------------------------`

# <font size=5 color = YellowGreen>控制参数显示/隐藏/禁用</font>
PRM_Template 会比较长

头文件
```c
// .h
protected:
	Shape_Regularization(OP_Network* net, const char* name, OP_Operator* op);
	bool updateParmsFlags() override;
private:
	int PARM_SETMODE(fpreal t) { return evalInt("setmode", 0, t); }
	int PARM_MINLEN(fpreal t) { return evalInt("minlen", 0, t); }
	int PARM_MAXANG(fpreal t) { return evalInt("maxang", 0, t); }
	int PARM_MAXOFF(fpreal t) { return evalInt("maxoff", 0, t); }
	int PARM_MAXOFF2(fpreal t) { return evalInt("maxoff2", 0, t); }
```
cpp文件

```c
// .cpp
PRM_Name modeOrdName("setmode", "SetMode");
PRM_Name modeOrdChoices[] =
{
    PRM_Name("mode1", "3.1 Closed Contours"),
    PRM_Name("mode2", "3.2 Open Contours"),
    PRM_Name(0)
};
PRM_ChoiceList modeOrdinalMenu(PRM_CHOICELIST_SINGLE, modeOrdChoices);

PRM_Name minlenName("minlen", "3.1 Min Len");
PRM_Default minlenDefault(2);
PRM_Range minlenRange(PRM_RANGE_UI, 0, PRM_RANGE_RESTRICTED, 10);

PRM_Name maxangName("maxang", "3.1 Max Ang");
PRM_Default maxangDefault(90);
PRM_Range maxangRange(PRM_RANGE_UI, 0, PRM_RANGE_RESTRICTED, 90);

PRM_Name maxoffName("maxoff", "3.1 Max Off");
PRM_Default maxoffDefault(2);
PRM_Range maxoffRange(PRM_RANGE_UI, 0, PRM_RANGE_RESTRICTED, 10);

PRM_Name maxoff2Name("maxoff2", "3.2 Max Off");
PRM_Default maxoff2Default(2);
PRM_Range maxoff2Range(PRM_RANGE_UI, 0, PRM_RANGE_RESTRICTED, 10);

PRM_Template Shape_Regularization::myTemplateList[] =
{
    PRM_Template(PRM_ORD_J, 1, &modeOrdName, 0, &modeOrdinalMenu),

    PRM_Template(PRM_INT_J, 1, &minlenName, &minlenDefault, 0, &minlenRange),
    PRM_Template(PRM_INT_J, 1, &maxangName, &maxangDefault, 0, &maxangRange),
    PRM_Template(PRM_INT_J, 1, &maxoffName, &maxoffDefault, 0, &maxoffRange),
    PRM_Template(PRM_INT_J, 1, &maxoff2Name, &maxoff2Default, 0, &maxoff2Range),

    PRM_Template(),
};

bool Shape_Regularization::updateParmsFlags()
{
	int state;
	bool changed = SOP_Node::updateParmsFlags();

	fpreal  t = CHgetEvalTime();
	state = (PARM_SETMODE(t)) ? 0 : 1;

	changed |= setVisibleState("minlen", state);
	changed |= setVisibleState("maxang", state);
	changed |= setVisibleState("maxoff", state);

	changed |= setVisibleState("maxoff2", -state + 1);

	return changed;
}
```
OP_ERROR
```c
fpreal now = context.getTime();

int setmode = PARM_SETMODE(now);

int minlen = PARM_MINLEN(now);
int maxang = PARM_MAXANG(now);
int maxoff = PARM_MAXOFF(now);

int maxoff2 = PARM_MAXOFF2(now);

if (setmode == 0)
{
	// ...
}
else if (setmode == 1)
{
	// ...
}
```
此三处最终效果如图
![[Pasted image 20221108152154.png]]
![[Pasted image 20221108152204.png]]

在cpp后半段
setVisibleState 函数不难看出是切换显示/隐藏
那么按理说 setEnableState 是切换显示/禁用，但你会发现你找不到这个函数，HDK不给你讲理，你应该使用 enableParm ，只能说这俩命名毫不相干
只需要修改这cpp部分的后半段代码，其他均不变，其实只改这四个函数就行
```c
// .cpp
bool Shape_Regularization::updateParmsFlags()
{
	int state;
	bool changed = SOP_Node::updateParmsFlags();

	fpreal  t = CHgetEvalTime();
	state = (PARM_SETMODE(t)) ? 0 : 1;
	
    changed |= enableParm("minlen", state);
    changed |= enableParm("maxang", state);
    changed |= enableParm("maxoff", state);

    changed |= enableParm("maxoff2", -state + 1);

	return changed;
}
```
效果如下图
![[Pasted image 20221108183353.png]]
![[Pasted image 20221108183402.png]]

`------------------------------THEDSFILE部分------------------------------`
由于这篇比较简单，那我再写一下使用 THEDSFILE 制作这个功能，这部分建议等看过支线再来
在代码层面，请忽略 mode3 this is THEDSFILE ，这是为了更好区分所以我故意加这个mode3

cpp文件
```c
// .cpp
static const char* theDsFile = R"THEDSFILE(
{
    name        parameters
    parm {
        name    "setmode"
        label   "Set Mode"
        type    ordinal
        default { "0" }
        menu {
            "mode1"     "3.1 Closed Contours"
            "mode2"     "3.2 Open Contours"
            "mode3"     "this is THEDSFILE"
        }
    }
    parm {
        name        "minlen"
        label       "3.1 Min Len"
        type        integer
        default     { "2" }
        range       { 0 10! }
        hidewhen    "{ setmode != mode1 }"
    }
    parm {
        name        "maxang"
        label       "3.1 Max Ang"
        type        integer
        default     { "90" }
        range       { 0 90! }
        hidewhen    "{ setmode != mode1 }"
    }
    parm {
        name        "maxoff"
        label       "3.1 Max Off"
        type        integer
        default     { "2" }
        range       { 0 10! }
        hidewhen    "{ setmode != mode1 }"
    }
    parm {
        name        "maxoff2"
        label       "3.2 Max Off"
        type        integer
        default     { "2" }
        range       { 0 10! }
        hidewhen    "{ setmode != mode2 }"
    }
}
)THEDSFILE";
```
hidewhen 显而易见就是切换显示/隐藏
disablewhen 即为切换显示/禁用
由于重复代码太多，就只截这一小段，其实就只换了个单词，其他均不变
```c
// .cpp
    parm {
        name        "minlen"
        label       "3.1 Min Len"
        type        integer
        default     { "2" }
        range       { 0 10! }
        disablewhen    "{ setmode != mode1 }"
    }
```
效果如下图
![[Pasted image 20221108184854.png]]
看这个this is THEDSFILE ，我重新截的，我没拿上面的图糊弄
![[Pasted image 20221108184458.png]]
![[Pasted image 20221108184506.png]]
`------------------------------THEDSFILE部分------------------------------`

---
---
调库，跟上一篇其实思路差不多，也简单
就展示一部分吧
```c
std::vector<Point_2> contour;
GA_Offset ptoff;
GA_RWHandleV3 Phandle(gdp->findAttribute(GA_ATTRIB_POINT, "P"));
GA_FOR_ALL_PTOFF(gdp, ptoff)
{
	UT_Vector3 Pvalue = Phandle.get(ptoff);
	contour.push_back(Point_2(Pvalue.x(), Pvalue.z()));
}
gdp->clearAndDestroy();
if (setmode == 0)
{
	FT min_length_2 = FT(minlen);
	FT max_angle_2 = FT(maxang);
	FT max_offset_2 = FT(maxoff);

	const bool is_closed = true;

	Contour_directions directions(contour, is_closed, CGAL::parameters::
		minimum_length(min_length_2).maximum_angle(max_angle_2));
	std::vector<Point_2> regularized;
CGAL::Shape_regularization::Contours::regularize_closed_contour(contour,directions, std::back_inserter(regularized),CGAL::parameters::maximum_offset(max_offset_2));
	GEO_PrimPoly* poly = (GEO_PrimPoly*)gdp->appendPrimitive(GA_PRIMPOLY);
	for (int i = 0; i < regularized.size(); i++)
	{
		Point_2 outpt2 = regularized.at(i);
		UT_Vector3 outv3(outpt2.x(), 0, outpt2.y());
		GA_Offset ptoff0 = gdp->appendPointOffset();
		Phandle.set(ptoff0, outv3);
		poly->appendVertex(ptoff0);
	}
	poly->close();
}
else if (setmode == 1)
{
	// ...
}
```
