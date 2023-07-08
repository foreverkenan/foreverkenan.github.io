---
layout: post
title:  "HDK支线 1 两种格式 OP_ERROR与void"
comments: true
categories: HDKBranch
---

打开雨落柏拉图的divide和官方示例SOP_Star
```
C:\Program Files\Side Effects Software\Houdini 19.5.303\toolkit\samples\SOP\SOP_Star
```
对比cpp可以明显发现divide使用的是PRM_Template制作参数面板，
```c
PRM_Template Sop_Dividehdk::myTemplateList[] =
{
	......
    PRM_Template(),

};
```
而SOP_Star使用的是DsFile
```c
static const char *theDsFile = R"THEDSFILE(

{
	name        parameters
    parm {
	    ......
	    ......
	}
}
```

还有，divide中的actual work写在（我简称为OP_ERROR）
```c
OP_ERROR Sop_Dividehdk::cookMySop(OP_Context& context)
{
	......
}
```
而SOP_Star写在（我简称为void）
```c
void SOP_StarVerb::cook(const SOP_NodeVerb::CookParms &cookparms) const
{
	......
}
```

这是两种格式的区别

在CMakeLists.txt中是否注释proto.h
```
houdini_generate_proto_headers( FILES SOP_Star.C )
```

使用DsFile更方便制作大量参数框，且需要切换隐藏与禁用的情况，会更加清晰
如果不需要大量参数，只有1到2个甚至没有参数，使用PRM_Template会更加方便

待验证：由红泥酱（Q 913254040）发现换用void后 节点可compile，至少Houdini中不显示齿轮
![[Pasted image 20221018224453.png]]其中SHP导入节点使用的是void，poly节点使用的是OP_ERROR
等待补充最大矩形HDK的图片 已经改写void模板后 不显示齿轮

void中的gdp需要使用proto.h中的内容获取
```c
    auto &&sopparms = cookparms.parms<SOP_StarParms>();
    GU_Detail *detail = cookparms.gdh().gdpNC();
```
而OP_ERROR的gdp直接可以用

一般DsFile可以直接生成解决方案，如果生成的时候报错显示找不到proto.h，你可以看一下实际文件夹中有没有这个文件，如果有，那就把cpp文件除了THEDSFILE的代码，其他全注释掉，重新生成解决方案，就能生成proto.h，如果有proto.h文件是存在的，但就是显示找不到，可以在解决方案资源管理器，找到你的项目，右键，重新扫描整个解决方案。如果还是不行，大概率是你代码本身有语法错误，比如少了后半个括号这种，但他报成和proto.h有关的东西，你找到语法错误再生成基本就能成功了