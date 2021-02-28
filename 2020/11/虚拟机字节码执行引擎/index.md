# 虚拟机字节码执行引擎


# 虚拟机字节码执行引擎

## 简述时栈帧结构

**栈帧**（Stack Frame）是用于支持虚拟机进行方法调用和方法执行后背后的数据结构。

存储了方法的

1. 局部变量表
2. 操作数栈
3. 动态连接
4. 方法返回地址
5. 一些额外的附加信息

### 局部变量表（Local Variables Table）

一组变量值的存储空间，存放方法的局部变量和方法参数。

局部变量的容量以变量槽（Variable Slot）为最小单位，一个变量槽可以存储一个32位以内的数据类型（随着计算机的处理、操作系统变化也可为64位）。Java中仅有long和double为8字节。

### 操作数栈（Operand Stack）

操作栈如同局部变量表一样，操作数栈的最大深度也在编译的时候写入到Code属性的max_stacks数据项中。操作栈中元素的数据类型必须与字节码指令的序列严格匹配，在编译程序代码的时候，编译器必须要严格保证这一点，在类校验阶段的数据流分析还要再次验证这一点。

Java**虚拟机的解释执行引擎**，里面的栈就是操作数栈。

### 动态连接

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用视为了支持方法调用过程中的动态连接（Dynamic Linking）

### 方法返回地址

两种方式退出当前方法

* 执行引擎遇到任意一个方法返回的字节码指令
* 遇到异常

## 方法调用

方法调用不等同于方法中的代码被执行，方法调用的唯一阶段就是确定哪个方法被调用，暂时还未涉及方法内部的运行过程。

#### 解析（resolve）

在类加载的解析阶段，会将其中的一部分符号引用转换为直接引用，这种解析能够成立的前提是：方法在程序真正执行之前就有一个确定的调用版本，并且该方法的调用版本在运行期是不可修改的。这类方法的调用称为 **解析**。

#### 分派（Dispatch）

#### 静态分配

静态分配和重载（Overload）

```java
package test;

/**
 * Description:
 * 方法静态分配演示
 * @author:edgarding
 * @date:2020/11/6
 **/
public class StaticDispatch {
    static class Human {

    }

    static class Woman extends Human {
    }

    static class Man extends Human {
    }

    public void sayHello(Human guy) {
        System.out.println("this is human say");
    }

    public void sayHello(Woman guy) {
        System.out.println("this is woman say");
    }

    public void sayHello(Man guy) {
        System.out.println("this is man say");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch staticDispatch = new StaticDispatch();
        staticDispatch.sayHello(man);
        staticDispatch.sayHello(woman);
    }
}

```

```
输出结果：
this is human say
this is human say
```

上面的“Human”称为变量的”静态类型“（Static Type）或者叫”外观类型“（Apparent Type），而后面的”man“或者”woman“称为变量的实际类型（Actual Type）或运行时类型（Runtime Type）。

静态类型和实际类型在程序中都可能会发生变化，区别是静态类型的变化仅仅是在使用时发生的，变量本身的静态类型不会发生改变，并且最终的静态类型是在编译期可知的；而实际类型只有在运行期才可以确定，编译器在编译时并不知道一个对象的实际类型是什么。

如上述代码，在main（）方法中的两次sayHello（）方法的调用，在方法接收者已经确定是对象”staticDispatch“的前提下，使用哪个版本就完全取决于传入参数的数量和数据类型。

代码中故意定义了两个静态类型相同、实际类型不同的两个参数，但虚拟机（编译器）在重载的时候是通过参数的静态类型作为判断标准，而不是实际类型。

**所有依赖静态类型来决定方法执行版本的分派动作都称为静态分配。静态分配的嘴典型应用表现就是方法重载。静态分配的动作实际上不是由JVM来决定而是编译器决定的，因此将静态分派的动作在一些资料中将其划入”解析“而不是”分派“的原因**。

##### 重载方法的优先级

Javac编译器虽然能确定出重载的版本，但在有些时候重载的版本不是唯一的，只能确定一个”相对合适的版本“。

```java
package JVM;

import java.io.Serializable;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/7
 **/
public class OverLoad {
    public static void say(Character character) {
        System.out.println("Hello character");
    }

    public static void say(char arg) {
        System.out.println("Hello char");
    }

    public static void say(int arg) {
        System.out.println("Hello int arg");
    }

    public static void say(long arg) {
        System.out.println("Hello long arg");
    }

    public static void say(char... arg) {
        System.out.println("Hello char...");
    }

    public static void say(Serializable serializable) {
        System.out.println("Hello serializable");
    }

    public static void main(String[] args) {
        say('a');
    }
}

```

```
输出结果：
Hello ch
```

由于’a’是一个char类型，所以自动会寻找参数为char的重载方法

如果注释掉`say(char ch)`，那么输出就变为 `Hello int`，这时发生了一次自动类型转换，将’a’转换为了97，因此自动寻参找int参数的方法。

如果在将 `say(int arg)`注释，那么会发生两次自动类型转换，97将转换成97L，会输出 `Hello long`。

如果 依次将其注释，其转换顺序为

> char - int - long - float - double

进行匹配，但不会匹配到byte 和 short类型重载，**因为char到byte或short是不安全的**。

如果将基本类型的参数全部注释，后面的参数类型转换结果为

```
Hello Character
Hello serializable
```

出现serializable的原因是java.lang.Serializable是java.lang.Character类的一个实现接口。

**由此可知，重载的转换从子类向父类依次往上搜寻，这也是Java实现方法重载的本质。**

#### 动态分派

动态分配的一个重要体现——“重写（override）

**运行期根据实际类型确定方法执行版本的过程称为动态分派**。

通过字节码中发现`invokevirtual`是确定最终执行的目标方法。

它的执行步骤

1. 找到操作数栈顶的第一个元素所执行的对象的  **实际类型** ，记住C
2. 如果在C中找到与常量中描述符合简单名称都相同你的方法，则进行访问校验，通过则返回方法的直接引用，否则抛出异常。
3. 否则按照继承关系从下往上一次进行搜索
4. 如果始终没有找到合适的方法则抛出 `java.lang.AbstractMethodError` 异常。

**多态性的根源在于虚方法调用 `invokevirtual` 指令，但只对方法有效，对字段无效。**

```java
package JVM;

/**
 * Description:
 * 字段无多态性
 * @author:edgarding
 * @date:2020/11/7
 **/
public class FieldHasNoPolymorphic {
    static class Father {
        public int money = 1;
        public Father() {
            money = 2;
          
        }
        public void showMoney() {
            System.out.println("Father's money:" + money);
        }
    }

    static class Son extends Father{
        public int money = 3;

        public Son() {
            money = 4;
        }

        @Override
        public void showMoney() {
            System.out.println("Son's money:" + money);
        }
    }

    public static void main(String[] args) {
        Father guy = new Son();
        System.out.println("this guy's money:" + guy.money );
    }
}

```

```
输出结果：
Son's money:0
Son's money:4
this guy's money:2
```

在进行 `Father guy = new Son()`时，首先会隐式调用了Father的构造函数，而Father构造函数中对`showMoney()`的调用是一次虚方法调用，实际执行的是`Son::showMoney()`方法，所以输出的是”Son’s money：0“，虽然这时候父类的money=2，但子类中money还没被进行初始化，只有要用到子类的构造函数执行时，才会被进行初始化。因此第一句 `Son's money:0`的由来就是如此。

第二句的 `Son's money:4`进行了子类的构造函数。

第三句，有main（）函数访问的是通过静态类型访问到了父类中money，输出了2。

> 虚函数：在运行的时候，系统根据对象的实际类型决定。即不使用final修饰的就是虚方法。

Java是一门静态多分派、动态单分派的语言。

> 单分派、多分派：
>
> 方法的接受者与方法的参数统称为方法的宗量。根据分派基于多少种宗量分为单分派和多分派。

#### 动态类型语言

关键特征：类型检查的主体过程是在运行期而不是编译期运行的，如python、JavaScript、Ruby等

相对的，在编译期进行检查的就是静态类型，如Java、C等。

**静态类型：**

能在编译期确定变量类型，最显著的好处编译器可以提供更加全面严谨的类型检查，利于稳定和项目容易达到更大的规模。

**动态类型：**

在运行期才确定类型，可以为开发人员提供更大的灵活性，代码更加清晰整洁。


