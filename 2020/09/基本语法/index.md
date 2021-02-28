# 基本语法


# 基本语法

## 基本数据类型

### 8大基本类型

| 基本类型 | 大小  | 包装器类型 | 默认值  |
| -------- | ----- | ---------- | ------- |
| boolean  |       | Boolean    | false   |
| char     | 2字节 | Character  | ‘u0000’ |
| byte     | 1字节 | Byte       | 0       |
| short    | 2字节 | Short      | 0       |
| int      | 4字节 | Integer    | 0       |
| long     | 8字节 | Long       | 0L      |
| float    | 4字节 | Float      | 0f      |
| double   | 8字节 | Double     | 0d      |

#### tip

整数计算时，数据溢出，预防例子：

`avg = a + ((b - a) >> 1)`

## 自动拆装箱

### 基本数类型有什么好处

new 一个对象时存储在堆里的，我们通过栈中的引用来使用这些对象，因此对象本身来说比较消耗资源。基本数据类型在栈内存中存储，因此会更加高效。

### 为什么包装类

Java是一种面向对象语言，很多地方需要使用对象而不是基本数据类型。为了让基本数据类型也有对象的特征，就出现包装类型。

### 自动拆箱与自动装箱

从JavaSE5中，Java开始提供了自动拆箱与装箱功能。

- 自动装箱：Integer.valueOf()
- 自动拆箱：xxxValue（）

## String

### 字符串的不可变性

#### 定义一个字符串

`String s = “abcd”`

#### 使用变量赋值变量

`String s2 = s`

s2 保存了相同的引用值，因为他们代表同意对象

### 字符串拼接的几种方式和区别

#### 字符串不变性与字符串拼接

`s = s.concat(“ef”)`

#### 使用+拼接字符串

使用+拼接字符串不能理解为**运算符重载**，其实并不是，Java是不支持运算符重载的。其实只是Java提供的一个**语法糖**。

#### StringBuffer

字符串常量的String类以外，还提供了可以用来定义字符串变量的StringBuffer类，它的对象是可以扩充和修改的。

其与StringBuilder的区别是synchronize线程安全。

#### StringBuilder

和StringBuffer作用是一样的，但不是线程安全的，但效率比StringBuffer高，因此在不考虑线程安全的情况下，建议使用StringBuilder。

#### StringUtils.join

除了JDK中内置的字符串拼接方法，还可以使用一些开源类库apache.commons中提供的StringUtils类，其中join方法可以拼接字符串

```java
String a = “dasd”;

StringUtils.join(a, “dafd”);
```

运行时间：

StringBuilder`<`StringBuffer`<`concat`<`+`<`StringUtils.join

### String总结

常用的字符串拼接方式有五种，分别是使用`+`、使用`concat`、使用`StringBuilder`、使用`StringBuffer`以及使用`StringUtils.join`。

直接使用`StringBuilder`的方式是效率最高的。因为`StringBuilder`天生就是设计来定义可变字符串和字符串的变化操作的

## Java各种关键字

|                      |          |            |          |              |            |           |        |
| -------------------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 访问控制             | private  | protected  | public   |              |            |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|                      | new      | static     | strictfp | synchronized | transient  | volatile  |        |
| 程序控制             | break    | continue   | return   | do           | while      | if        | else   |
|                      | for      | instanceof | switch   | case         | default    |           |        |
| 错误处理             | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关               | import   | package    |          |              |            |           |        |
| 基本类型             | boolean  | byte       | char     | double       | float      | int       | long   |
|                      | short    | null       | true     | false        |            |           |        |
| 变量引用             | super    | this       | void     |              |            |           |        |
| 保留字               |          | const      |          |              |            |           |        |

### transient

> Java语言的关键字，变量修饰符，如果用transient声明一个实例变量，当对象存储时，它的值不需要维持。这里的对象存储是指，Java的serialization提供的一种持久化对象实例的机制。当一个对象被序列化的时候，transient型变量的值不包括在序列化的表示中，然而非transient型的变量是被包括进去的。使用情况是：当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。

简单点说，就是被transient修饰的成员变量，在序列化的时候其值会被忽略，在被反序列化后， transient 变量的值被设为初始值， 如 int 型的是 0，对象型的是 null。

#### instanceof

instanceof是Java的保留关键字，用于测试左边和右边的实例是否相等，返回一个boolean的数据类型。

#### volatile

与synchronized不同，volatile是一个变量修饰符，只能用来修饰变量。volatile的用法比较简单，只需要在声明一个可能被多线程同时访问的变量时，使用voilatile修饰就可以了。

1 . 保证了不同线程对该变量操作的内存可见性;

2 . 禁止指令重排序

#### synchronized

`synchronized`是Java提供的一个并发控制的关键字。主要有两种用法，分别是同步方法和同步代码块。也就是说，`synchronized`既可以修饰方法也可以修饰代码块。

#### final

表示“这部分是无法修改的”。使用final定义：变量、方法、类。

* final变量：则不能更改final变量的值
* final方法：则无法对该方法进行覆盖重写
* final类：则无法对该类进行进程

#### static

用来修饰成员变量和成员方法。

* 静态变量：不属于类的对象或者实例。因为静态变量与所有的对象实例共享，因此不具有线程安全性。
* 静态方法：静态方法是属于类而不是实例。
* 静态代码块：静态块是一组指令在类装载的时候在内存中由ClassLoader执行。常用语初始化类的静态变量。大多时候还用于在类装载时候创建静态资源。Java不允许在静态块中使用非静态变量。一个类中可以有多个静态块，尽管这似乎没有什么用。静态块只在类装载入内存时，执行一次。

#### const

const是Java预留关键词，用于后期拓展用，用法跟final相似不常用。

## Object类

### Object常见方法

Object是一个特殊的类，它是所有类的父类。

```java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
```

### hashcode（）介绍

hashcode（）的作用就是获取hashcode（散列码），实际上是一个int整数，作用是确定该对象在hash表中的索引位置。hashcode（）定义在Object类中，因此，Java的任意一个对象都拥有一个hashCode（）函数，且hashcode（）方法为native方法，是由c++和c实现的，该方法通常用来将对象的内存地址转换为整数之后返回。

hash表中存储的是键值对（key-value），用来快速查找对象。

### 为什么要有hashcode

例如HashSet，HashSet会先计算对象的hashcode值，然后通过该hashcode判断对象加入的位置，同时对其它已经加入的对象的hashcode进行比较，如果没有相符的hashcode，HashSet会假设对象没有重复。如果发现有相同的hashcode值的对象会调用equals（）方法来检查hashcode相等的对象是否真的相等，如果相等HashSet就不会让其加入操作。

### hashcode（）与equals（）的相关规定

1. 如果两个对象相等，则hashcode一定是相同的
2. 两个对象相等，对两个对象分别调用equals方法返回true
3. 两个对象有着相同的hashcode值，它们也不一定相等
4. **因此，equals方法被覆盖过，则hashCode（）也必须被覆盖**
5. hashCode（）的默认行为对heap上的对象产生独特值。如果没有重写hashCode（），则该class的两个对象无论如何都不会相等（即使两个对象指向相同的数据）

### 为什么两个对象有相同的hashcode值，它们也不一定相等

因为在散列表中，hashcode（）相等，及两个键值对的hash值相等。然而，hash值相等，并不能一定得出键值对相等。

“两个不同的键值对，hash值相等”。这就hash冲突。

此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。

> 举个例子：我们改写下hashCode()的算法为：num%3 ;
>
> 当num=1时，余数为1；
>
> 当num=4时，余数也为1；
>
> 这时他们的hashCode()是相同的，但equals()却是不同的。

### ==与equals

==：判断两个对象的地址是不是相等，即判断两个对象是不是同一个对象。（基本数据类型==比较的是值，引用数据类型==比较的是内存地址）

equals（）：判断的也是两个对象是否相等，一般有两种使用情况：

* 类没有覆盖equals（）方法，则通过equals（）比较该类的两个对象时，等价于“==”比较这两个对象
* 类覆盖率equals（）方法。

```java
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        String aa = "ab"; // 放在常量池中
        String bb = "ab"; // 从常量池中查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，非同一对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

- String 中的 equals()方法是被重写过的，因为 Object 的 equals()方法是比较的对象的内存地址，而 String 的 equals()方法比较的是对象的值。
- 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。
