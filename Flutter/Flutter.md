# Flutter

1. 由来：利用`Chrome 2D`渲染引擎，然后精简Css布局演变而来，直接从优化语言本身着手
2. 在各个原生的平台中，使用自己的`C++`引擎渲染界面，也就是说平台只是给Flutter提供了一块画布
3. `Dart`：支持`JIT和AOT`模型的强类型语言
   程序主要有两种运行方式：静态编译和动态解释
   静态编译的程序在执行前全部被编译为机器码，通常就是`AOT（Ahead of Time）`提前编译；
   动态解释则是边解释边运行，即`JIT（Just in Time）`即是编译
   `AOT`程序的典型代表是`C/C++`开发的应用，`JIT`则有代表`JavaScript和Python`等（脚本语言），Java既可`JIT`也可`AOT`，**区分是否为`AOT`的标准是在代码执行之前是否需要编译，只要需要编译，无论是`java`的字节码还是其他语言编译出来的机器码，都属于`AOT`**。
4. Flex布局方式
5. Flutter业务书写的`Widget`在渲染之前`diff`转化成`Render Object`，类似于`虚拟DOM`，以此确保开发体验和性能。