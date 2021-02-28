# Java反射机制


# Java反射机制

## Class类

RTTI（Run-Time Type Identification）运行时类型识别，其作用是运行时识别一个对象的类型和类的信息。

- 运编译器已知道所有类型（在没有反射机制创建和使用类对象时，一般都是编译期已确定其类型）
- 反射机制，允许我们在运行时发现和使用类型的信息

在Java中用来表示运行时类型信息对应的类就是Class类，Class类也是一个实际存在的类。

> 实际上在Java中每个类都有一个Class对象，每当我们编写并且编译一个新创建的类就会产生一个对应Class对象并且这个Class对象会被保存在同名.class文件里(编译后的字节码文件保存的就是Class对象)，那为什么需要这样一个Class对象呢？是这样的，当我们new一个新对象或者引用静态成员变量时，Java虚拟机(JVM)中的类加载器子系统会将对应Class对象加载到JVM中，然后JVM再根据这个类型信息相关的Class对象创建我们需要实例对象或者提供静态变量的引用值。

### Class对象

- Class类也是类的一种，与class关键字是不一样的。
- 手动编写的类被编译后会产生一个Class对象，其表示的是创建的类的类型信息，而且这个Class对象保存在同名.class的文件中(字节码文件)，比如创建一个Shapes类，编译Shapes类后就会创建其包含Shapes类相关类型信息的Class对象，并保存在Shapes.class字节码文件中。
- 每个通过关键字class标识的类，在内存中有且只有一个与之对应的Class对象来描述其类型信息，无论创建多少个实例对象，其依据的都是用一个Class对象。
- Class类只存私有构造函数，因此对应Class对象只能有JVM创建和加载
- Class类的对象作用是运行时提供或获得某个对象的类型信息，这点对于反射技术很重要(关于反射稍后分析)。

### Class对象的加载

Class对象是由JVM加载的，所有类都是在对其第一次使用动态加载到JVM中的，当程序创建第一个对类的静态成员引用时，就会加载这个被使用的类（实际上加载的就是这个类的字节码文件）

注意：使用new创建类的新实例的对象也会被当做类的成员引用（构造函数也是类的静态方法），由此看来Java程序在他们运行之前也并非一次性全部加载到内存，各个部分是按需加载，所以在使用该类的时，Class Loader会先根据类名查找.class文件，在进行检查完全没有问题后会被动态加载到内存中，此时Class对象也就被加载如内存了。

### Class.forName()方法

会返回一个类的Class对象。

## 反射

### 反射API

* Field 类：提供有关类的属性信息，以及对它的动态访问权限。它是一个封装反射类的属性的类。
* Constructor 类：提供有关类的构造方法的信息，以及对它的动态访问权限。它是一个封装反射类的构造方法的类。
* Method 类：提供关于类的方法的信息，包括抽象方法。它是用来封装反射类方法的一个类。
* Class 类：表示正在运行的 Java 应用程序中的类的实例。
* Object 类：Object 是所有 Java 类的父类。所有对象都默认实现了 Object 类的方法。

### 获取Class对象的三种方式

获取 Class 对象有三种方式：

```java
package Advance.reflection;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/12
 **/
public class Main {
    public static void main(String[] args) {
        try {
            // 通过字符串加载
            Class test = Class.forName("Advance.reflection.A");
            // 通过class属性
            Class test1 = A.class;
            // 通过类的getClass（）方法
            A a = new A();
            Class test2 = a.getClass();
            System.out.println(test + "\n" + test1 + "\n" + test2);
            System.out.println("test == test ? " + (test == test1));
            System.out.println("test 1 == test 2 ? " + (test1 == test2));
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

- 第一种方法是通过类的全路径字符串获取 Class 对象，这也是我们平时最常用的反射获取 Class 对象的方法；
- 第二种方法有限制条件：需要导入类的包；
- 第三种方法已经有了 Student 对象，不再需要反射。

通过这三种方式获取到的 Class 对象是同一个，也就是说 Java 运行时，每一个类只会生成一个 Class 对象。

运行程序，输出如下：

```text
class Advance.reflection.A
class Advance.reflection.A
class Advance.reflection.A
test == test ? true
test 1 == test 2 ? true
```


