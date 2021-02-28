# 单例模式


# Java实现单例模式

## 单例定义

Singleton 模式主要作用保证在Java应用程序中，一个类Class只有一个实例存在。

另外方面，Singleton 也能够被无状态化。提供工具性质的功能， Singleton 模式就为我们提供了这样实现的可能。使用 Singleton 的好处还在于可以节省内存，因为它限制了实例的个数，有 利于 Java 垃圾回收（garbage collection）。

## 案例场景

1. 数据库的连接池不会反复创建
2. spring中一个单例模式bean的生成和使用
3. 在我们平常开发的代码中需要设置全局的一些属性保存

主要解决的是，一个全局使用的类频繁的创建和消费，从而提升整体的代码性能。

## 设计思想

及解决问题的关键就是保证应用中仅存有一个对象。

* 不允许其它应用程序用new对象
  * 因为new就是开辟新的空间，更改数据只是更改所创建对象的数据，如果可以new的话，每一次new都产生一个对象，这样肯定保证不了对象的唯一性。
* 在该类中创建对象
  * 因为不允许其它程序new对象，所以这里的对象需要在本类中new出来
* 对外提供一个可以让其它程序获取该对象的方法
  * 因为对象是在本类中创建的，所以需要提供一个方法让其它类获取该对象

1、私有化该类的构造函数

2、通过new在本类中创建一个本类对象

3、定义一个公有方法

## 如何使用？

一般有以下几种模式：

* 懒汉式
* 饿汉式
* 双重校验锁
* 静态内部类
* 枚举

饿汉与懒汉的区别：

饿汉是在内部已经创建好了实例，只要Singleton被装载就会实例化，因此会产生延迟（来源实例化）

### 饿汉式[可用]：

```java
public class Singleton {
  private Singleton() {
    // 在自己内部定义自己一个实例
    // 注意这里是private只供内部调用
    private static Singleton instance = new Singleton();
    // 这里提供一个外部访问本class 的静态方法
    public static Singleton getInstance() {
      return instance;
    }
  }
}
```

获取实例方法：

```java
Singleton instance = Singleton.getInstance();
```

及它在类创建的时候就已经实例化了对象。

 这种方式跟饿汉式方式采用的机制类似，但又有不同。两者都是采用了类装载的机制来保证初始化实例时只有一个线程。不同的地方在饿汉式方式是只要Singleton类被装载就会实例化，没有Lazy-Loading的作用，而静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonHolder类，从而完成Singleton的实例化。

类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。

优点：避免了线程不安全，延迟加载，效率高。

### 懒汉式：

#### 单例模式的懒汉式[线程安全，效率低]：

```java
public class Singleton {
  private static Singleton instance = null;
  public static synchronized Singleton getInstance() {
    // 这个方法比上面有所改进，不用每次都进行生成对象，只是第一次
    // 使用时生成实例，提高了效率
    if(instance == null) {
      instance = new Singleton();
      return instance;
    }
  }
}
```

这种方式是在调用getInstance方法时才创建对象的，因为比较懒，才被称为懒汉式。

缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。

#### 单例模式懒汉式双重校验锁[推荐用]

```java
package Advance.design_pattern.singleton;

/**
 * Description:
 * static + 锁 双重锁检验
 * 减少了锁带来的资源开销
 * 注意实例前的volatile关键字，由于实例化一个对象需要三步
 * 1、为instance分配内存
 * 2、初始化instance
 * 3、将instance指向内存空间
 * 由于JVM具有指令重排的特性，所以以上的执行步骤有可能变为132，因此有可能对象拿到了一个未初始化的对象实例
 * 在多线程的情况下，可能会有线程在进行判断未初始化成功，进入锁代码块中重新实例化了一次，从而破坏了单例模式
 * 使用volatile修饰变量，防止指令的重排列
 * 1、解决内存可见性问题 2、防止指令重排列
 * @author:edgarding
 * @date:2020/11/4
 **/
public class SingletonLanHan03 {
    private static volatile SingletonLanHan03 instance;

    private SingletonLanHan03() {}

    public static SingletonLanHan03 getInstance() {
        // 判断对象是否实例化过，没有才进入
        if(instance == null) {
            // 没有实例化，对其类对象进行加锁
            synchronized (SingletonLanHan03.class) {
                if (instance == null) {
                    // 进行实例化，但注意这里是非原子性操作
                    instance = new SingletonLanHan03();
                }
            }
        }
        return instance;
    }
}

```

访问方式：

```java
Singleton instance = Singleton.getInstance();
```

优点：线程安全；延迟加载；效率较高。

### 枚举[极其推荐使用]

```java
/**
 * 枚举[极推荐使用]
 *
 * 这里SingletonEnum.instance
 * 这里的instance即为SingletonEnum类型的引用所以得到它就可以调用枚举中的方法了。
 借助JDK1.5中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象
 */

public enum SingletonEnum {

    instance;

    private SingletonEnum() {
    }

    public void whateverMethod() {

    }

    // SingletonEnum.instance.method();

}

```

访问方式：

```java
SingletonEnum.instance.method();
```


