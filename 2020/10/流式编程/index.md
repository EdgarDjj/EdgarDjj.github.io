# 流式编程

# 流式编程

> 集合优化了对象的存储，而流和对象的处理有关

流是一系列与特定存储机制无关的元素——实际上，流并没有“存储”之说。

使用流，无需迭代集合中的元素，就可以从管道提取和操作元素。这些管道通常被组合在一起，形成一系列对流进行操作的管道。

在大多数情况下，将对象存储在集合中是为了处理他们，因此你将会发现你将把编程的主要焦点从集合转移到了流上。流的一个核心好处是，它使得程序更加短小并且更易理解。当 Lambda 表达式和方法引用（method references）和流一起使用的时候会让人感觉自成一体。流使得 Java 8 更具吸引力。

例子：

```java
public class StreamExample {
    public static void main(String[] args) {
        new Random()
                .ints(1, 100)
                .limit(7)
                .distinct()
                .sorted()
                .forEach(System.out::println);
    }
}
```

**流可以在不使用赋值或可变数据的情况下对有状态的系统建模**

ints（）方法产生一个流并且ints（）方法多种方式重载——两个参数限定了产生的数值的边界，这将随机生成一个1～100的随机整数流。用中间流操作（intermediate stream operation）distinct（）使流中的整数不重复，然后使用limit（）方法获取前7个元素，用sorted（）方法排序，最后使用forEach（）方法遍历输出，在这传递一个可以在控制台显示每个元素的方法引用：`System.out::println`

注意 `Randoms.java` 中没有声明任何变量。流可以在不使用赋值或可变数据的情况下对有状态的系统建模，这非常有用。

* 内部迭代产生的代码可读性更强，而且能更简单的使用多核处理器。通过放弃对迭代过程的控制，可以把控制权交给并行化机制。
* 流是懒加载的。这代表着它只在绝对必要时才计算。你可以将流看作“延迟列表”。由于计算延迟，流使我们能够表示非常大（甚至无限）的序列，而不需要考虑内存问题。

### 流支持

Java 设计者面临着这样一个难题：现存的大量类库不仅为 Java 所用，同时也被应用在整个 Java 生态圈数百万行的代码中。如何将一个全新的流的概念融入到现有类库中呢？

比如在 **Random** 中添加更多的方法。只要不改变原有的方法，现有代码就不会受到干扰。

一个大的挑战来自于使用接口的库。集合类是其中关键的一部分，因为你想把集合转为流。但是如果你将一个新方法添加到接口，那就破坏了每一个实现接口的类，因为这些类都没有实现你添加的新方法。

Java 8 采用的解决方案是：在接口中添加被 `default`（`默认`）修饰的方法。通过这种方案，设计者们可以将流式（*stream*）方法平滑地嵌入到现有类中。流方法预置的操作几乎已满足了我们平常所有的需求。流操作的类型有三种：创建流，修改流元素（中间操作， Intermediate Operations），消费流元素（终端操作， Terminal Operations）。最后一种类型通常意味着收集流元素（通常是到集合中）。

### 流创建

可以通过Stream.of()将一组元素转化为流

```java
public static void showStreamOf() {
    Stream.of("this is ", " a  ", "word").forEach(System.out::print);
    System.out.println();
    Stream.of(1, 32," ", 32, " ",34, 43).forEach(System.out::print);
}
```

输出结果：

`this is  a  word
132 32 3443`

除此之外，每个集合都可以通过调用stream（）方法来产生一个流

```java
public static void showStream() {
    Map<String, Integer> m = new HashMap<>();
    m.put("a", 1);
    m.put("b", 2);
    m.put("c", 3);
    m.entrySet().stream().map(e -> e.getKey() + ":" + e.getValue()).forEach(System.out::println);
}
```

输出结果

`a:1
b:2
c:3`

为了从 **Map** 集合中产生流数据，我们首先调用 `entrySet()` 产生一个对象流，每个对象都包含一个 `key` 键以及与其相关联的 `value` 值。然后分别调用 `getKey()` 和 `getValue()` 获取值。

### 随机数流

`Random` 类被一组生成流的方法增强了。代码示例：

```java
// streams/RandomGenerators.java
import java.util.*;
import java.util.stream.*;
public class RandomGenerators {
    public static <T> void show(Stream<T> stream) {
        stream
        .limit(4)
        .forEach(System.out::println);
        System.out.println("++++++++");
    }

    public static void main(String[] args) {
        Random rand = new Random(47);
        show(rand.ints().boxed());
        show(rand.longs().boxed());
        show(rand.doubles().boxed());
        // 控制上限和下限：
        show(rand.ints(10, 20).boxed());
        show(rand.longs(50, 100).boxed());
        show(rand.doubles(20, 30).boxed());
        // 控制流大小：
        show(rand.ints(2).boxed());
        show(rand.longs(2).boxed());
        show(rand.doubles(2).boxed());
        // 控制流的大小和界限
        show(rand.ints(3, 3, 9).boxed());
        show(rand.longs(3, 12, 22).boxed());
        show(rand.doubles(3, 11.5, 12.3).boxed());
    }
}复制ErrorOK!
```

输出结果：

```java
-1172028779
1717241110
-2014573909
229403722
++++++++
2955289354441303771
3476817843704654257
-8917117694134521474
4941259272818818752
++++++++
0.2613610344283964
0.0508673570556899
0.8037155449603999
0.7620665811558285
++++++++
16
10
11
12
++++++++
65
99
54
58
++++++++
29.86777681078574
24.83968447804611
20.09247112332014
24.046793846338723
++++++++
1169976606
1947946283
++++++++
2970202997824602425
-2325326920272830366
++++++++
0.7024254510631527
0.6648552384607359
++++++++
6
7
7
++++++++
17
12
20
++++++++
12.27872414236691
11.732085449736195
12.196509449817267
++++++++
```

为了消除冗余代码，我创建了一个泛型方法 `show(Stream<T> stream)` （在讲解泛型之前就使用这个特性，确实有点作弊，但是回报是值得的）。类型参数 `T` 可以是任何类型，所以这个方法对 **Integer**、**Long** 和 **Double** 类型都生效。但是 **Random** 类只能生成基本类型 **int**， **long**， **double** 的流。幸运的是， `boxed()` 流操作将会自动地把基本类型包装成为对应的装箱类型，从而使得 `show()` 能够接受流。

### IntStream.range()

```java
public class RangeOfInt {
    public static void main(String[] args) {
        // 传统方法
        int res = 0;
        for (int i = 1; i <= 100; i++) {
            res += i;
        }
        System.out.println(res);
        // for-in循环
        res = 0;
        for (int i : range(1, 101).toArray()) {
            res += i;
        }
        System.out.println(res);
        // 流
        System.out.println(range(1, 101).sum());
    }
}
```



### 流的建造者模式

在建造者模式（Builder design pattern）中，首先创建一个 `builder` 对象，然后将创建流所需的多个信息传递给它，最后`builder` 对象执行”创建“流的操作。**Stream** 库提供了这样的 `Builder`。
