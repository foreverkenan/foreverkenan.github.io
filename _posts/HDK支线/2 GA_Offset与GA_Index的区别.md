```
C:\Program Files\Side Effects Software\Houdini 19.5.368\toolkit\slides\HDK12_Intro.pdf
```

在pdf的26-28页解释了offset与index的区别

index其实就是Houdini里我们常见的序号，@ptnum、@primnum等
offset是不会改变的，删除点后他的offset会仍然空着，而index会由后面的点补位
在往常操作中，我们可以发现点序号、面序号肯定连续，不可能中间断开

当需要互转的时候
```c
gdp->pointOffset(ptidx)
gdp->pointIndex(ptoff)

gdp->primitiveOffset(primidx)
gdp->primitiveIndex(primoff)
```