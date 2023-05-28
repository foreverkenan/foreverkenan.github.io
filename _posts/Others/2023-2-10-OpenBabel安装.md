---
layout: post
title:  "OpenBabel下载编译修改"
comments: true
categories: Others
---

1. 下载
[Open Babel 官网](http://openbabel.org/wiki/Main_Page)自己找下载按钮
或[Open Babel Github](https://github.com/openbabel/openbabel/releases/tag/openbabel-3-1-1)下载**source code**
或直接**速通**[->**点我下载** Source code(zip)<-](https://github.com/openbabel/openbabel/archive/refs/tags/openbabel-3-1-1.zip)
git clone等方法均可
2. 解压
3. 创建**build**文件夹，如图

4. 此时可能需要安装并配置Boost、Eigen库的环境变量，但我暂时没有配置
5. 在build文件夹中打开CMD，输入
`cmake ..`
回车等待

其中有一些报错，是 4. 提到的无法找到的库

6. 打开**openbabel.sln**
7. 对**ALL_BUILD**右键，**重新生成**

老三样 include、lib、dll
openbabel311是我自己改了名字，同时也改了位置
1. include有两个地方，要注意，一个在build外，一个在build内

2. lib

3. dll




plugin.h里要改个东西 开头加上，否则strcasecmp报错
```cpp
#if defined(_MSC_VER)
#define strcasecmp _stricmp
#endif
```
把std:: 删掉
```cpp
struct OBERROR CharPtrLess : public std::binary_function<const char*,const char*, bool>
```
前面加一个东西
```cpp
template <class _Arg1, class _Arg2, class _Result>
struct binary_function { // base class for binary functions
    using first_argument_type  = _Arg1;
    using second_argument_type = _Arg2;
    using result_type          = _Result;
};
```