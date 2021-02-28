# JVM内存区域


# JVM内存区域

由于Java不需要像C/C++一样为每个new操作去对应一个delete/free操作，不会出现内存泄漏和内存溢出问题，但正因如此，在出现此问题的时候，将会更加麻烦，因此我们需要了解虚拟机是如何使用内存的。

## 运行时的数据区域

JDK8之前：

线程共享：

* Heap
* Method Area
  * Runtime Constant pool

线程私有：

* VM Stack
* Native Method Stack
* Program Counter Register

JDK8：

线程共享：

* Heap
* 元空间Metaspace
  * 直接内存 Direct Memory

线程私有：

* VM Stack
* Native Method Stack
* Program Counter Register

### Program Counter Register 程序计数器

程序计数器是一块小的内存空间，可以认为是当前线程所执行字节码的行号指示器。

**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。**

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

#### 作用：

1. 字节码解释器通过程序计数器来保证指令的顺序读取，从而实现代码的流程控制。
2. 在多线程情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道上次线程所运行的位置。

程序计数器是唯一一个不会出现OutOfMemoryError的内存区域，生命周期随着线程的创建而创建，随着线程的结束而死亡。

### 虚拟机栈

**于程序计数器一样，Java虚拟机栈线程私有，生命周期与线程相同，描述的是Java方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java内存可粗糙分为，堆内存Heap和栈内存Stack，即栈就是指虚拟机栈，或是虚拟机中局部变量表的部分。**（实际上，Java虚拟机栈是由一个一个栈帧组成，而每个栈帧中拥有：局部变量表、操作数表、动态链接、方法出口信息）

局部变量表主要存放了编译期可知的各种数据类型（boolean、byte、char等等）、对象引用（reference类型，是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）

**Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。**

- **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** 若 Java 虚拟机堆中没有空闲内存，并且垃圾回收器也无法提供更多内存的话。就会抛出 OutOfMemoryError 错误。

Java 虚拟机栈也是线程私有的，每个线程都有各自的 Java 虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

#### 那么方法和函数如何调用

Java栈可用相当于数据结构中的栈，Java栈中保存的内容主要是栈帧，每一次函数调用都会有一儿栈帧被压入Java栈，每一个函数结束，会有一个栈帧被弹出。

两种返回方式：

* return
* 抛出异常

### 本地方法栈 Native Method Stack

与虚拟机栈相类似，区别是： **虚拟机栈为虚拟机执行Java方法（即字节码）服务，而本地方法栈则为虚拟机使用的到Native方法服务器**

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种错误。

### 堆 Heap

JVM所管理的内存中最大的一块，Java堆是所有线程共享的一块内存区域，在JVM启动时创建。

**唯一目的**就是存放对象实例，几乎所有对象实例和数组都在这分配内存。

**Java中”几乎“所有对象都在堆中分配，但是，随着JIT编译期的发展和逃逸分析的技术成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从jdk 1.7开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。**

在 JDK 7 版本及JDK 7 版本之前，堆内存被通常被分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

大部分情况，对象都会首先在Eden区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入s0或者s1，并且对象的年龄还会增加1（Eden区->Survivor区后对象的初始年龄变为1）当它的年龄增加到了一定程度（默认为15岁）就会被升到老年代中。对象晋升到老年代的年龄阈值，可以通过 `-XX：MaxTenuringThreshold`来设置。

> 修正：“Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```java
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
>   size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
>   size_t total = 0;
>   uint age = 1;
>   while (age < table_size) {
>     total += sizes[age];//sizes数组是每个年龄段对象大小
>     if (total > desired_survivor_size) break;
>     age++;
>   }
>   uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> ```

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当JVM花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发`java.lang.OutOfMemoryError: Java heap space` 错误。(和本机物理内存无关，和你配置的内存大小有关！)
3. ......

### 方法区

方法区也与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态常量、即使编译器编译后的代码等数据。

方法区也被称为 **永久代**。

#### 方法区和永久代的关系

> 《Java虚拟机规范》只是规定了有方法区这个概念和它的作用，并没有规定如何去实现它。那么在不同的JVM上方法区的实现肯定不同。
>
> **方法区和永久代的关系类似于Java中接口和类的关系，类实现了接口，而永久代就是HotSpot虚拟机对虚拟机规范中方法区的一种实现方式。**

即永久代是HotSpot的概念，方法区是Java虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其它的虚拟机实现并没有永久代这一说。

#### 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

#### 为什么要将永久代（PermGen）替换为元空间（MetaSpace）

1. 整个永久代有一个JVM本身设置固定大小上限，无法调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

当你元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

2. 元空间里面放的是类的元数据，这样加载多少类的元数据就不由MaxPerSize控制了，而由系统的实际空间来控制，这样能加载的类就更多了。

3. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

### 运行时常量池

运行时常量池时方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息，还有常量池表（用于存放编译器生成的各种字面量和符号引用）

既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemory。

1. **JDK1.7之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时hotspot虚拟机对方法区的实现为永久代**
2. **JDK1.7 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是hotspot中的永久代** 。
3. **JDK1.8 hotspot移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)**

### 直接内存

**直接内存并不是虚拟机运行时数据区对的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致OOM错误**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

## HotSpot 虚拟机对象探秘

### 对象的创建

1. 类加载检查
2. 分配内存
3. 初始化零值
4. 设置对象头
5. 执行init方法

#### Step1 类加载检查

虚拟机遇到一条new指令时，首先去检查这个指令的参数是否能够在常量池中定位到这个类的符号引用，并且检查引用代表的类是否已经被加载过、解析过和初始化过。如果没有，必须先执行相应的类加载过程。

#### Step2 分配内存

**在类加载检查后**，接下来虚拟机将为新生对象**分配对象**。对象所需的内存大小，在类加载完成后便可确定，为对象空间的任务等同于把一块确定大小的内存够从Java堆中划分出来。

分配方式有 **指针碰撞** 和 **空闲列表**，选择哪种方式，由Java堆是否规整决定，而Java堆是否规整由GC是否带有压缩整理功能决定（标记-清除和标记-清理）。

##### 指针碰撞

**适合场景：**堆内存规整（即没有内存碎片）的情况

**原理：**用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可。

**GC收集器：**Serial、ParNew

##### 空闲列表

**适用场景：**堆内存不规整的情况下

**原理：**虚拟机会维护一个列表，该列表会记录哪些内存块是可用的，在分配的时候，找一块足够大的内存来划分给对象实例，最后更新列表。

**GC收集器：**CMS

##### 内存并发问题

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事，作为虚拟机必须保证线程安全，通常来讲，有两种方式：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

#### Step3 初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步保证了对象实例字段在Java代码中可以不赋值就直接使用。

#### Step4 设置对象头

初始化零值后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的hashcode、对象的GC分代年龄等信息。这些信息存放在对象头中。另外，根据虚拟机当前运行状态的不同，如果是否使用偏向锁，对象头会有不同的设置。

#### Step5 执行init方法

上述步骤完成后，从虚拟机来看，一个新的对象已经产生，但从Java程序角度来看，对象创建才刚开始 `init`方法还没有执行，所有字段都还为0。所以一般来说，执行new指令后会接着执行init方，把对象按照程序员的意愿将对象进行初始化。

### 对象的内存布局

在HotSpot虚拟机中，对象在内存中的布局可以分为：**对象头、实例数据、对齐填充**

HotSpot 虚拟机的对象头包括两部分信息，**第一部分**用于**存储对象自身的运行时数据**（hashcode、GC分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据指针，虚拟机通过该指针来确定这个对象是哪个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。**因为HotSpot虚拟机的自动内存管理系统要求对象的起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

### 对象的访问定位

建立对象就是为了使用对象，我们Java程序通过栈上的reference数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问有两种：

* 使用句柄
* 直接指针

**句柄：** 如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

![](./JVM内存区域/对象的访问定位-使用句柄.png)

**直接指针：**如果使用直接指针访问。那么Java堆对象的布局必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象的地址。

![](./JVM内存区域/对象的访问定位-直接指针.png)

**这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。**

> 对象类型数据：虚拟机加载的类信息加载的类信息
>
> 对象实例数据：被new出来的对象信息

## 重点补充

### String类和常量池

```
String str1 = "abcd";//先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str2 = new String("abcd");//堆中创建一个新的对象
String str3 = new String("abcd");//堆中创建一个新的对象
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

这两种不同的创建方法是有差别的。

- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。

记住一点：**只要使用 new 方法，便需要创建新的对象。**

**String 类型的常量池比较特殊。它的主要使用方法有两种：**

- 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
- 如果不是用双引号声明的 String 对象，可以使用 String 提供的 intern 方法。String.intern() 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7之前（不包含1.7）的处理方式是在常量池中创建与此 String 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用。

```java
String s1 = new String("计算机");
String s2 = s1.intern();
String s3 = "计算机";
System.out.println(s2);//计算机
System.out.println(s1 == s2);//false，因为一个是堆内存中的 String 对象一个是常量池中的 String 对象，
System.out.println(s3 == s2);//true，因为两个都是常量池中的 String 对象
```

**字符串拼接:**

```java
String str1 = "str";
String str2 = "ing";

String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象	  
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

![](./JVM内存区域/字符串拼接-常量池2.png)

尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer。

### String s1 = new String("abc");这句话创建了几个字符串对象？

**将创建 1 或 2 个字符串。如果池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。如果池中没有字符串常量“abc”，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象。**

**验证：**

```java
		String s1 = new String("abc");// 堆内存的地址值
		String s2 = "abc";
		System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
		System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```
false
true
```

### 8种基本类型的包装类和常量池

Java基本类型的包装类大部分都实现了常量池技术，**即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。** 为啥把缓存设置为[-128，127]区间？性能和资源之间的权衡。

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
private static class CharacterCache {         
    private CharacterCache(){}
          
    static final Character cache[] = new Character[127 + 1];          
    static {             
        for (int i = 0; i < cache.length; i++)                 
            cache[i] = new Character((char)i);         
    }   
}
```

**两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。**

```java
		Integer i1 = 33;
		Integer i2 = 33;
		System.out.println(i1 == i2);// 输出 true
		Integer i11 = 333;
		Integer i22 = 333;
		System.out.println(i11 == i22);// 输出 false
		Double i3 = 1.2;
		Double i4 = 1.2;
		System.out.println(i3 == i4);// 输出 false
```

**Integer 缓存源代码：**

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

**应用场景：**

1. Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
2. Integer i1 = new Integer(40);这种情况下会创建新的对象。

```java
  Integer i1 = 40;
  Integer i2 = new Integer(40);
  System.out.println(i1==i2);//输出 false
```

**Integer 比较更丰富的一个例子:**

```java
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);
  
  System.out.println("i1=i2   " + (i1 == i2));
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
  System.out.println("i1=i4   " + (i1 == i4));
  System.out.println("i4=i5   " + (i4 == i5));
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6));   
  System.out.println("40=i5+i6   " + (40 == i5 + i6));     
```

结果：

```java
i1=i2   true
i1=i2+i3   true
i1=i4   false
i4=i5   false
i4=i5+i6   true
40=i5+i6   true
```

解释：

语句 i4 == i5 + i6，因为+这个操作符不适用于 Integer 对象，首先 i5 和 i6 进行自动拆箱操作，进行数值相加，即 i4 == 40。然后 Integer 对象无法与数值进行直接比较，所以 i4 自动拆箱转为 int 值 40，最终这条语句转为 40 == 40 进行数值比较。
