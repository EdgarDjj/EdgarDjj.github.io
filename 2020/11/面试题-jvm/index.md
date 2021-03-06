# 面试题-JVM


# JVM

194. 说一下JVM的主要组成部分？及其作用？
* 类加载器（ClassLoader）
* 运行时数据区（Runtime Data Area）
* 执行引擎（Execution Engine）
* 本地库接口（Native Interface）
类加载器将.java文件转化成字节码.class文件，运行时数据区将字节码加载到内存中，由于.class文件只是JVM的一套指令集，无法被CPU执行，因此需要执行引擎去将其翻译成CPU可执行的二进制文件，而这个过程需要调用其它语言的本地接口（Native Interface）实现。

195. 说一下JVM运行时数据区
java应用程序在运行的时候，JVM会将内存区域进行划分，每个区域有着各自的用途和自己的创建时间。
分为了
* 方法区
* 虚拟机栈
* 堆
* 程序计数器
* 本地方法栈
有的区域随着虚拟机的进程启动而存在，有的区域依赖用户进程的启动和结束而创建和销毁。
方法区：
JDK8之前为永久代的存放区，方法区这个称呼是对虚拟机规范而言，而永久代则是对JVM（Hotspot）而言方法区的一种实现，在JDK8之后为元空间。用于存放已被虚拟机加载的类信息、常量、静态常量，即使编译器编译后的代码等数据。永久带到元空间的升级，原本永久代有个JVM本身设置的固定内存大小上线，无法调整，而现在元空间没有了这些限制，可以动态的进行调整，由系统的实际空间进行控制，这样加载的类就更多了。
程序计数器：
是一个块小的内存空间，JVM线程独有的，用于标记字节码指令的行号，方便在线程切换的时候可以找到原来工作状态的位置，因此每个线程内都独有一个程序计数器，同时也保证程序的顺序读取，用于代码的流程控制。
JVM栈：
是JVM线程独有的一块区域，生命周期与线程相同，描述的是Java方法执行的内存模型，每次方法调用的数据都是通过栈传递的。
本地方法栈：
虚拟机栈为虚拟机执行Java方法（即字节码）服务，而本地方法栈则为虚拟机使用的到Native方法服务器。
堆：
Java中几乎所有对象都在对堆中分配，但是，随着JIT编译期的发展和逃逸分析的技术成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变的不那么绝对，从jdk 1.7开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

196. 说一下堆栈的区别？
几乎所有对象都在堆中分配，而栈一般为JVM栈，或是JVM栈中的局部变量表的部分，一般存储着一些局部量。
栈内存存储的是局部变量而堆内存存储的是实体
栈内存的更新速度要快于堆内存，因为局部变量的生命周期很短
栈内存存放的变量生命周期一旦结束就会被释放，而堆内存存放的实体由gc机制不定时的回收。

197. 队列和栈是什么？有什么区别？
队列和栈都是用来存储数据的。
队列的结构特点是FIFO，先进先出，但也有特例Deque接口，运行从两端检索元素。
但栈为先进后出。

198. 什么是双亲委派模型？
对于任何一个类，都需要由它的类加载器和这个类本身一同确立在JVM中的唯一性，每一个类加载器，都有一个独立的类空间。类加载器就是根据全限定名称将class文件加载到JVM内存，然后再转换为class对象。
类加载器分类：
* 启动器类加载器（BootStrap ClassLoader）是虚拟机自身的一部分，用来加载Java_HOME/lib目录中的，或者是被-Xbootclasspath参数所指定的路径中并且被虚拟机识别的类库
* 拓展类加载器（Extension ClassLoader）
* 应用程序类加载器（Application ClassLoader）负责加载用户类路径classpath上的指定类库，我们可以直接使用这个类加载器。（默认）
双亲委派模型就是，一个类加载器收到了类加载的请求，它首先不会自己去加载这个类，而是把这个请求委派给父类加载器去完成，每一层的类加载器都是如此，这样所有的加载请求都会被传送到顶层的启动类加载器中，只有当父加载无法完成加载请求（它的搜索范围中没找到所需的类）时，子加载器才会尝试去加载类。

199. 说一下类加载的过程
加载：根据查找路径找到全限定类名的class文件导入
检查：检查加载的class文件的正确性
准备：正式为类中的变量（静态变量static）分配内存空间，并设置类变量初始值
解析：JVM将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个表示，而在直接引用直接指向内存地址
初始化：对静态变量和静态代码块执行初始化工作

200. 怎么判断对象是否可以回收
两种方式：
引用计数器： 为每个对象创建一个计数器，当该对象被引用的时候+1，被释放的时候-1，当计数器为0的时候可以被进行回收，但出现循环引用的问题，到时无法进行回收。
可达性分析：从GC roots开始扫描，搜索走过的路径称为引用链。当一个对象到GC roots开始向下搜索，搜索走过的路径称为引用链，而对象到GC roots没有任何引用链相连的，将对其进行回收

201. Java中都有哪些引用类型？
强引用：Object obj = new Object（）强引用是最普遍的，一个对象如果具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机会抛出OOME错误是程序终止也不会随意对其进行回收，G1gc垃圾收集器可能会进行标记整理
剩下三个引用都继承自Reference
软引用：SoftReferences 在内存不够的时候会对其进行回收
弱引用：弱引用对象生命周期很短，当GC扫描所在的区域一旦发现了只具有弱引用的对象，不管当前内存空间是否足够都会对其进行回收。
虚引用：Java中最弱的引用，它是如此脆弱以至于我们通过虚引用甚至无法获取到被引用的对象，虚引用存在的唯一作用就是当它指向的对象被回收后，虚引用本身会被加入到引用队列中，用作记录它指向的对象已被回收。

202. 说一下JVM有哪些垃圾回收算法
标记-复制
标记-清理
标记-整理
分代收集理论

203. 说一下JVM有哪些垃圾回收器
Serial：最早的单线程串行垃圾回收器
Serial Old：Serial 的老版本，可作为CMS（Concurrent Mark Swap降低停顿时间）的备选
ParNew：Serial的多线程版本
Parallel 和 ParNew 收集器是多线程的，但Parallel是吞吐量优先的收集器，可以牺牲等待时间换取系统的吞吐量。
Parallel Old：是Parallel老生代版本，Parallel使用的是标记-复制的内存回收算法，Parallel Old使用的是标记-整理的算法
CMS（Concurrent Mark Swap）：一种以获得最短停顿时间为目标的收集器，非常适合B/S系统
G1：一种兼顾吞吐量和停顿时间的GC实现，是JDK9以后的默认GC选项

204. 详细介绍一下CMS垃圾回收器
CMS（Concurrent Mark Swap），是以牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器。对于要求服务器响应速度的应用上，这种垃圾回收器非常适合。在启动JVM的参数上：-XX:+UseConcMarkSweepGC 来指定使用CMS垃圾回收器。
CMS使用的事标记-清理的算法实现的，所以在GC的时候会产生大量的内存碎片，当剩余内存不能满足程序运行的时候，系统会出现Concurrent Mode Failure，临时CMS会采用Serial Old回收器进行标记-整理对内存进行整理，但是停顿的时间就会更长了。

205. 新生代垃圾回收器和老生代回收器都有哪些？有什么区别？
新生代：Serial、ParNew、Parallel Scavenge
老生代：Serial Old、ParNew Old、CMS
整堆回收器：G1
新生代垃圾回收器一般采用的是标记-复制算法，复制算法的优点是效率高，缺点是内存利用率低；老年代回收器一般是标记-整理。

206. 简述垃圾分代回收器是怎么工作的？
分代回收区有两个：老年代、新生代
新生代默认占总空间的1/3，老年代占2/3。
新生代中分eden、survivor区，两者占新生代空间的80%和10%，其中survivor中有from survivior 和 to survivor两个，用于进行标记-复制算法的清理。
执行步骤：
1. 把Eden+from survivor存活的对象放入到to survivor中
2. 清空Eden 和 From survivor分区
3. From Survivor 和 To Survivor 分区交换， From Survivor -> <-To Survivor
每次在From Survivor 到 To Survivor中移动存活对象的时候，年龄就+1，当达到15的时候（默认配置）会被升为老生代。大对象直接进入老生代。
老生代当空间占用达到某个值后就会触发全局垃圾回收，一般使用标记-整理算法。

207. 说一下JVM调优工具？
JDK自带了很多监控工具，都位于JDK的bin目录下
* jvisualvm：全能分析工具
* jconsole：用于对JVM中的内存、线程和类进行监控

208. 常用的JVM调优参数有哪些？
-Xms2g：初始堆大小为2g
-Xmx2g：初始堆的最大大小为2g
-XX：NewRatio=4：设置年轻和老年代的内存比例为1：4
-XX：SurvivorRatio=8：设置新生代Eden：Survivor的比例为8：2
-XX：+UseParNewGC：指定使用ParNew+Serial Old垃圾收集器
-XX：+UserParallelOldGC：指定使用Parallel Scavege+Parallel Old垃圾收集器
-XX：+UseConcMarkSweepGC：指定使用CMS+Serial Old垃圾回收组合
-XX：+PrintGC：开启打印GC信息
-XX：+PrintGC：PrintGCDetails：打印GC详细信息
