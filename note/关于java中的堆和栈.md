### Java中的堆和栈的区别

当一个人开始学习Java或者其他编程语言的时候，会接触到堆和栈，由于一开始没有明确清晰的说明解释，很多人会产生很多疑问，什么是堆，什么是栈，堆和栈有什么区别？更糟糕的是，Java中存在栈这样一个后进先出（Last In First Out）的顺序的数据结构，这就是java.util.Stack。这种情况下，不免让很多人更加费解前面的问题。事实上，堆和栈都是内存中的一部分，有着不同的作用，而且一个程序需要在这片区域上分配内存。众所周知，所有的Java程序都运行在JVM虚拟机内部，我们这里介绍的自然是JVM（虚拟）内存中的堆和栈。

之前做应用OOM分析的时候提取堆内存中的引用，主要是分析引用来定位无法释放的对象，而没有了解堆中存放的含义。

#### 区别
java中堆和栈的区别自然是面试中的常见问题，下面几点就是其具体的区别

#### 各司其职
最主要的区别就是栈内存用来存储局部变量和方法调用。
而堆内存用来存储Java中的对象。无论是成员变量，局部变量，还是类变量，它们指向的对象都存储在堆内存中。

#### 独有还是共享
栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存。
而堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问。

#### 异常错误
如果栈内存没有可用的空间存储方法调用和局部变量，JVM会抛出java.lang.StackOverFlowError。
而如果是堆内存没有可用的空间存储生成的对象，JVM会抛出java.lang.OutOfMemoryError。

#### 空间大小
栈的内存要远远小于堆内存，如果你使用递归的话，那么你的栈很快就会充满。如果递归没有及时跳出，很可能发生StackOverFlowError问题。
你可以通过-Xss选项设置栈内存的大小。-Xms选项可以设置堆的开始时的大小，-Xmx选项可以设置堆的最大值。

这就是Java中堆和栈的区别。理解好这个问题的话，可以对你解决开发中的问题，分析堆内存和栈内存使用，甚至性能调优都有帮助。

#### 查看默认值(Updated)
查看堆的默认值，使用下面的代码，其中InitialHeapSize为最开始的堆的大小，MaxHeapSize为堆的最大值。
```java
 $ java -XX:+PrintFlagsFinal -version | grep HeapSize
    uintx ErgoHeapSizeLimit                         = 0                                   {product}
    uintx HeapSizePerGCThread                       = 87241520                            {product}
    uintx InitialHeapSize                          := 134217728                           {product}
    uintx LargePageHeapSizeThreshold                = 134217728                           {product}
    uintx MaxHeapSize                              := 2147483648                          {product}
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
查看栈的默认值,其中ThreadStackSize为栈内存的大小。

 $ java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
     intx CompilerThreadStackSize                   = 0                                   {pd product}
     intx ThreadStackSize                           = 1024                                {pd product}
     intx VMThreadStackSize                         = 1024                                {pd product}
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
```
