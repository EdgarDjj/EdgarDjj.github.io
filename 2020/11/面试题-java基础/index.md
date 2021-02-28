# 面试题-Java基础


# 一、Java基础

1. JDK和JRE有什么区别？
   JDK是Java Development Kit，包含了Java的运行环境还有运行java程序的指令等例如javac，javadoc等，是可以进行运行和开发的。提供了开发和运行环境。
   而JRE是Java Runtime Environment，包含了Java的运行环境，仅仅只能运行Java的程序而无法进行开发等需求
   
2. == 和 equals 的区别
   都是用来对数据进行判断的，但在基本数据类型和引用数据类型上有所不同。
   ==：
   在判断引用类型时，判断的是两边的引用类型的内存地址是否为同一个内存地址。而在判断基本数据类型时，判断的是值是否相同（注意自动拆箱和自动装箱时，使用缓存池对象的情况）
   equals：
   本质上与==相同，但对进行了重写，用于判断两个引用对象是否相等。有两种使用方式：一种是类覆盖率equals方法，一种是没有覆盖equals方法。
   若没有类覆盖equals方法那么它与==的作用没有区别，但若进行类的重写，一般就判断是两个对象的内容是否相等。

3. 两个对象的hashcode（）相同，则equals（）也一定为true吗？
   不对，两个对象的hashcode相等，equals也不一定相等。因为在散列表中，hashcode（）相等即两个键值对的hash值相等，并不能得出键值对相等。

* hashcode（）的默认行为是对堆上的对象产生独特值，如果没有重写hashcode（）则该class的两个对象无论如何也不相等（即使这两个对象指向相同的数据），因此不仅进行重写，equals相等，那hashcode也不相等。
  hashcode（）作用是获取hash码，hash码的作用是确定该对象在hash表中的索引位置。hashcode（）定义在JDK的Object.java中，因此每一个Java类都包含该方法。散列表中指的是：Java集合中本质的是散列表的类，如HashMap、Hashtable、HashSet。即hashCode（）在散列表中有用，其它情况无用。

4. final 在 Java中的作用
   final是一个Java中的关键字，可以修饰在方法、类、字段上
   方法：该方法不可被子类进行重写
   类：该类不可被继承
   字段：该字段被初始化后就不可被修改

5. Java中的Math.round(-1.5)等于多少？
   等于-1，round（）返回一个最接近参数值的整数，并舍入往无穷大方向

6. String属于基本数据类型吗？
   不属于，String是一个对象。基本的数据类型有byte、short、int、long、char、float、double、boolean

7. Java中的操作字符串有哪些类？它们之间有什么区别？
   有String、StringBuffer、StringBuilder
   String的底层在JDK9以下是final char[] object，而JDK9开始是final char[] byte，而由于用了final进行修饰，因此在使用String操作字符串的时候，例如String a = a+ “1”会创建新的一个String常量，并将引用指针指向新的常量地址。
   而StringBuffer和StringBuilder之间的区别在于线程安全的问题，StringBuffer的每个方法都经过了synchronized的修饰，是线程安全的，除此之外没有其余的差别。StringBuffer和StringBuilder都继承AbstractStringBuilder抽象类，而AbstractStringBuilder抽象类继承了Appendable的接口。在对字符串进行操作的时候都是在当前字符串上进行操作，而不是像String一样在堆内存中重新创建一个。因此如果需要进行大量的字符串操作的时候，应避免使用String类型。

8. String str = “i” 与 String str = new String(“i”)一样么？
   不一样，内存分配方式不一样。String str = “i”会将“i”分配到常量池中，而String str = new String（“i”）会将“i”分配到堆内存中。

9. 如何将字符串反转
   使用StringBuilder和StirngBuffer的reverse（）方法，它们该方法存在于AbstractStringBuidler类中。
   将字符串转换成char数组，利用双指针进行数组反转。

10、String类的常用方法有哪些？
indexOf()：返回指定字符的索引。
charAt()：返回指定索引处的字符。
replace()：字符串替换。
trim()：去除字符串两端空白。
split()：分割字符串，返回一个分割后的字符串数组。
getBytes()：返回字符串的 byte 类型数组。
length()：返回字符串长度。
toLowerCase()：将字符串转成小写字母。
toUpperCase()：将字符串转成大写字符。
substring()：截取字符串。
equals()：字符串比较。

11. 抽象类必须要有抽象方法吗？
    抽象类不一定要有抽象方法。

12. 普通类和抽象类之间有什么区别？

* 普通类不包含抽象方法，抽象类包含抽象方法
* 抽象类无法进行实例化操作

13. 抽象类能使用final修饰吗？
    不能，抽象类本身的用途就是用来被继承的，如果使用会被编译器报错。

14. 接口和抽象类有哪些区别？

* 接口用implements进行实现，而抽象类用extends进行实现。
* 一个类可以有多个接口，但只能单继承
* 抽象类可以有构造函数，而接口不可以
* 接口中的方法默认使用public修饰，而抽象类中的方法可以使用任意访问修饰符
* 抽象类可以有main方法

15. Java 中的IO流分几种？
    按功能分：输入和输出
    按类型分：字节流和字符流
    字节流和字符流的区别，字节流是以8位bit为字节单位输入输出数据，字符流按字（看计算机一个字可以为多个字节）为单位输入输出。

16. BIO、NIO、AIO有什么区别？
    BIO
    阻塞型IO

NIO
非阻塞型IO

AIO
驱动型IO

17. Files的常用方法都有哪些？
    Files.exists()：检测文件路径是否存在
    Files.createFiles()：创建文件夹
    Files.delete()：删除一个文件或者目录
    Files.copy()：复制文件
    Files.move()：移动文件
    Files.size()：查看文件个数
    Files.read()：读取文件
    Files.write()：写入文件


