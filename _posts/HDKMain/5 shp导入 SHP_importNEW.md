---
layout: post
title:  "AutoDL+PyCharm训练"
comments: true
categories: HDKMain
---

[SHP_importNEW](https://www.bilibili.com/video/BV1DW4y1y7iu)
[Houdini HDK 读取shapefile矢量数据]
https://blog.csdn.net/weixin_44415965/article/details/128388723?spm=1001.2014.3001.5501

这个节点调库，GDAL

---

# <font size=5 color = YellowGreen>参数面板制作文件读取（file）</font>
输入框+下拉三角+选择文件
![](Pasted%20image%2020221222194550.png)
选择文件，默认*.shp
![](Pasted%20image%2020221222194906.png)

THEDSFILE 主要代码
```c
name        parameters
parm {
			name    "shpfile"
			label   "SHP File"
			type    file
			default { "" }
			parmtag { "autoscope" "0000000000000000" }
			parmtag { "filechooser_pattern" "*.shp" }
			parmtag { "import_token" "shp_file" }
		}

```

# <font size=5 color = YellowGreen>红三角警告提示</font>
![](Pasted%20image%2020221222195443.png)
![](Pasted%20image%2020221222195459.png)

```c
cookparms.sopAddError(SOP_MESSAGE, "ba la ba la");
```

SOP_MESSAGE可以换成SOP_ERR_INVALID_SRC，会显示No source or source has error，等等，有好几十种

此外，还有两种
warning是黄色警告
message是白色的字
```c
cookparms.sopAddWarning( ... )
cookparms.sopAddMessage( ... )
```

---
---

Houdini的斜杠方向是 **/** ，而Windows的路径为 **\\** ，所以需要 replace 一下，由于 UT_StringHolder 和 std::string 我都没找到有这个功能，我就自己for了一遍，如果你们知道的话可以直接用现成的 replace
```c
const UT_StringHolder utsh = sopparms.getShpfile();
std::string stds = utsh.toStdString();
char* charp = stds.data();
for (int i = 0; i < strlen(charp); i++)//replace \ to /
{
	if (charp[i] == '\\') //转义！！
	{
		charp[i] = '/';
	}
}
const char* shpFile = charp;
```

开头要多做判断，不然如果用户开头点错了个文件，直接崩houdini
```c
if (poDS == nullptr)
	return;
```

GEO_PrimPoly 获取面序号和面 offset 的名字有点怪，叫 Map 而不是 Prim / Primitive ，不知道的话还真想不到，除非一个一个找过去...
```c
GEO_PrimPoly* poly = (GEO_PrimPoly*)detail->appendPrimitive(GA_PRIMPOLY);

poly->getMapOffset()
poly->getMapIndex()
```

其他代码都是以前出现过的知识点，或者纯粹GDAL调库内容了