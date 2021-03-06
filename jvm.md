# Jvm  
<!-- TOC -->

- [Jvm](#jvm)
    - [内存机制](#内存机制)
    - [类加载机制](#类加载机制)
    - [垃圾回收机制](#垃圾回收机制)
        - [堆区组成](#堆区组成)
        - [对象存活判断](#对象存活判断)
        - [垃圾收集算法](#垃圾收集算法)
        - [分代收集算法](#分代收集算法)
    - [回收机制](#回收机制)
        - [Minor GC](#minor-gc)
        - [Full GC](#full-gc)
    - [垃圾回收器](#垃圾回收器)
        - [Serial垃圾回收器](#serial垃圾回收器)
        - [Serial Old垃圾回收器](#serial-old垃圾回收器)
        - [ParNew垃圾回收器](#parnew垃圾回收器)
        - [ParNew Old垃圾回收器](#parnew-old垃圾回收器)
        - [CMS垃圾回收器](#cms垃圾回收器)
        - [G1垃圾回收器](#g1垃圾回收器)
    - [Jvm工具](#jvm工具)

<!-- /TOC -->
## 内存机制  
> jvm将内存划分为以下几个部分：
+ 程序计数器：记录当前线程正在执行的字节码地址，执行本地方法时为空  
  
+ 虚拟机栈：存储java方法调用时的局部变量，对象引用，操作数，方法参数等数据  
  
+ 本地方法栈：存储执行本地方法调用时产生的帧栈数据  
  
+ 堆区：存储几乎所有的java对象  
  
+ 方法区：存储常量、静态变量、类字节码等  
  
+ 直接内存：jvm堆外分配的内存，java NIO中引入  
  
![jmm](./pic/jmm.png)
## 类加载机制
> **加载**-->**验证**-->**准备**-->**解析**-->**初始化**
+ 加载  
> &#160; &#160; &#160; &#160;根据类的全限定名获取类的二进制字节流，将类的静态数据结构转换为方法区的运行时数据结构，在堆中生成一个代表该类的Class对象，做为访问方法区中类运行时数据结构的入口  
+ 验证
> &#160; &#160; &#160; &#160;对类的二进制字节流进行验证，保证class文件符合虚拟机要求，且不会危害虚拟机自身  
+ 准备  
> &#160; &#160; &#160; &#160;为静态变量在方法区中分配内存空间并将其初始化为默认值，如果静态变量被final修饰此时初始化为定义值，否则静态变量在准备阶段只在方法区申请内存，并初始化为默认值，真正的初始化工作在初始化阶段  
+ 解析  
> &#160; &#160; &#160; &#160;把类中的符号引用转换为直接引用，即是将引用地址转换为内存中的真正地址  
+ 初始化  
> &#160; &#160; &#160; &#160;执行类的构造器<client>方法，对静态变量进行赋值和执行静态代码块  
## 垃圾回收机制
### 堆区组成
![heap](./pic/heap.gif)  
+ 新生代  
> + Eden  
> + From Survivor  
> + To Survivor  
+ 老年代
+ 永久代(方法区)  
### 对象存活判断
+ 可达性分析算法  
> 将对象之间的引用关系以GC Roots为根节点组织成树，以根节点作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收  
+ 引用计数算法  
> 每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法实现简单，但无法解决对象相互循环引用的问题  
### 垃圾收集算法
+ 复制算法  
> &#160; &#160; &#160; &#160;将Eden区与Survivor区存活的对象复制到另一个Survivor中，清除Eden区与该Survivor  
+ 标记清除  
> &#160; &#160; &#160; &#160;标记可被回收的内存，清除内存
+ 标记整理  
> &#160; &#160; &#160; &#160;标记可被回收的内存，清除内存，同时对内存碎片进行整理
### 分代收集算法
> Jvm按照堆区划分的策略，选择不同的垃圾收集算法  
+ 新生代每次垃圾收集时有大批对象死去，只有少量存活，选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集  
+ 老年代中因为对象存活率高,选择使用“标记-清理”或“标记-整理”算法来进行回收
## 回收机制  
### Minor GC
+ 当Eden区满时，触发Minor GC
### Full GC
+ 调用System.gc时，系统建议执行Full GC，但是不必然执行
+ 老年代空间不足  
+ 方法区空间不足  
+ 通过Minor GC后进入老年代的对象大小大于老年代的可用内存  
## 垃圾回收器
### Serial垃圾回收器  
+ 新生代垃圾回收器  
+ 使用单线程工作
+ 采用复制算法
+ 垃圾回收时会Stop the World
### Serial Old垃圾回收器
+ 老年代垃圾回收器  
+ 使用单线程工作
+ 采用标记整理算法
+ 垃圾回收时会Stop the World
### ParNew垃圾回收器  
+ 新生代垃圾回收器  
+ 使用多线程工作
+ 线程默认与CPU核数相同，可指定线程数目  
+ 采用复制算法
+ 垃圾回收时会Stop the World
### ParNew Old垃圾回收器
+ 老年代垃圾回收器  
+ 使用多线程工作
+ 采用标记整理算法
+ 垃圾回收时会Stop the World
### CMS垃圾回收器
> 并发低停顿垃圾回收器，追求业务响应时间，适用于互联网业务领域，整个过程如下：  
+ 初始标记：Stop the World  
+ 并发标记：与用户线程并发进行  
+ 重新标记：Stop the World，修正并发标记阶段  
+ 并发清除：与用户线程并发进行  
> CMS采用标记清除算法，会产生内存碎片
### G1垃圾回收器
+ 将整个堆划分为大小相同的独立区域  
+ 采用标记整理算法，不会产生内存碎片  
+ 可预测的GC停顿时间，追求业务响应时间  
+ 与CMS相比，适用于大内存的垃圾回收
## Jvm工具
+ javap
> &#160; &#160; &#160; &#160;反编译工具，根据Java字节码文件反编译为Java源代码文件  
+ jconsole  
> &#160; &#160; &#160; &#160;图形化界面检测工具，可检测并可视化显示Java程序的性能、资源占用及垃圾回收等  
+ jps
> &#160; &#160; &#160; &#160;查看系统上运行的Java进程 
+ jstat  
> &#160; &#160; &#160; &#160;提供在控制台下监视Java进程中的各种运行状态信息，包括gc统计信息等 
+ jinfo
> &#160; &#160; &#160; &#160;查看与修改JVM运行时的参数  
+ jhat
> &#160; &#160; &#160; &#160;生成heapdump文件，提供用户在浏览器中查看分析堆内存中的对象信息  
+ jstack  
> &#160; &#160; &#160; &#160;查看Java进程中各线程的堆栈跟踪信息
+ jdeps
> &#160; &#160; &#160; &#160;用于分析Java class的依赖关系