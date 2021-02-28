# 面试题-对象拷贝和异常


# 面试题-对象拷贝和异常

## 对象拷贝

61. 为什么要使用克隆？
想对一个对象进行处理，又想保留原有数据进行接下来操作，Java中克隆针对的是类的实例。

62. 如何实现对象克隆？
两种方式：
1. 实现Cloneable接口并重写Object类中的clone方法
2. 实现Serializable接口，通过对象的序列化和反序列化实现克隆
注意：基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是好过把问题留到运行时。

63. 深拷贝和浅拷贝的区别是什么？
浅拷贝：对基本数据类型进行值传递，对引用数据进行引用传递拷贝。
深拷贝：对基本数据类型进行值传递，对引用类型创建一个新的对象，并将引用变量指向该对象的内存区域。

## 异常

74. throw 和 throws的区别？
throws 是用来声明一个方法可能抛出的所有异常信息，是将异常声明但不处理，而是将异常向上传递，谁能调用给谁处理。而throw 则是抛出一个具体的异常类型，可以自己创建。


75. final、finally、finalize 有什么区别？
* final 可以修饰类（表示该类不可被继承）、变量（声明该类为一个常量）、方法（声明该方法不可被重写）
* finally 一般作用于try-catch代码块中，处理异常的时候，finally的代码块一定会被执行。
* finalize 是一个方法，属于Object类的一个方法，而Object类是所有类的父类，该方法一般由gc收集器进行调用，当我们调用System.gc()方法的时候，由垃圾回收器调用finalize()，回收垃圾

76. try-catch-finally 中哪个部分可以省略?
catch可以

77. try-catch-finally中，如果catch中return了，finally还会执行吗？
会，会在retrun之前执行

78. 常见的异常类有哪些？
NullPointException
IndexOutOfArrayException
ArithmeticException
ClassCastException
IllegalArgumentException
IOException
NoSuchMethod
RuntimeException
NumberFormatException
