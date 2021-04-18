# 基础篇：JVM运行时内存布局

JVM运行时内存布局如下：
![image](https://user-images.githubusercontent.com/13992911/115152364-5ddfe600-a0a3-11eb-9c90-a805dfa14cb0.png)

其中，Java虚拟机栈、程序计数器、Heap、本地方法栈、Metaspace属于JVM运行时的内存；

JAVA堆和MetasSpace元空间属于线程共享的（黄色部分）；虚拟机栈和本地方法栈、程序计数器是线程私有的


## 1、JVM的内存区域布局
* java代码的执行步骤有三点
   * java源码文件->编译器->字节码文件
   * 字节码文件->JVM->机器码
   * 机器码->系统CPU执行
* JVM执行的字节码需要用类加载来载入；字节码文件可以来自本地文件，可以在网络上获取，也可以实时生成。就是说你可以跳过写java代码阶段，直接生成字节码交由JVM执行
* 其中Java虚拟机栈、程序计数器、Heap、本地方法栈、Metaspace属于JVM运行时的内存；
