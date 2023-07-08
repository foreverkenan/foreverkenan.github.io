区分开vex和hdk
vex因为wrangle，10个面 for(1->10) removeprim，执行完才会全删除
hdk，执行第一次deleteprim的时候，面就少了一个，如果还用 for(1->10)，
那么后几次循环都是空指针操作，崩houdini




```c
std::cout << "origin" << detail->getNumPrimitives()

        << "a" << std::endl;

    GA_Primitive* prim;

    for (int i = 0; i < detail->getNumPrimitives(); i++)

    {

        GEO_PrimPoly* poly = (GEO_PrimPoly*)detail->appendPrimitive(GA_PRIMPOLY);

        GA_Offset ptoff = detail->appendPointOffset();

        poly->appendVertex(ptoff);

        std::cout << detail->getNumPrimitives() << "   ";

        if (detail->getNumPrimitives() > 30)

            break;

    }
```

如果用GA_FOR循环面，则是安全的，因为有个maco什么什么，但和vex也有不同