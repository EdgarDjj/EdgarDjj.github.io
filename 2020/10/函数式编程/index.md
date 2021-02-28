# 函数式编程

# 函数式编程

> 函数式编程语言操纵代码片段就像操作数据一样容易。 虽然 Java 不是函数式语言，但 Java 8 Lambda 表达式和方法引用 (Method References) 允许你以函数式编程。

**函数编程（FP）**：通过合并现有代码来生成新功能而不是从头开始编写所有内容，我们可以更快地获得更可靠的代码。至少在某些情况下，这套理论似乎很有用。在这一过程中，函数式语言已经产生了优雅的语法，这些语法对于非函数式语言也适用。

* OO（Object Oriented，面向对象）是抽象数据
* FP（Functional Programming，函数式编程）是抽象方法

纯粹的函数式语言在安全性方面更进一步。它强加了额外的约束，即所有数据必须是不可变的：设置一次，永不改变。将值传递给函数，该函数然后生成新值但从不修改自身外部的任何东西（包括其参数或该函数范围之外的元素）。当强制执行此操作时，你知道任何错误都不是由所谓的副作用引起的，因为该函数仅创建并返回结果，而不是其他任何错误。

## Lambda表达式

- Java 8 的 Lambda 表达式，其参数和函数体被箭头 `->` 分隔开。箭头右侧是从 Lambda 返回的表达式。它与单独定义类和采用匿名内部类是等价的，但代码少得多。
- Java 8 的**方法引用**，它以 `::` 为特征。 `::` 的左边是类或对象的名称， `::` 的右边是方法的名称，但是没有参数列表。

Lambda 表达式是使用**最小可能**语法编写的函数定义：

1. Lambda 表达式产生函数，而不是类。 在 JVM（Java Virtual Machine，Java 虚拟机）上，一切都是一个类，因此在幕后执行各种操作使 Lambda 看起来像函数 —— 但作为程序员，你可以高兴地假装它们“只是函数”。
2. Lambda 语法尽可能少，这正是为了使 Lambda 易于编写和使用。

任何 Lambda 表达式的基本语法是：

1. 参数。
2. 接着 `->`，可视为“产出”。
3. `->` 之后的内容都是方法体。
   - **[1]** 当只用一个参数，可以不需要括号 `()`。 然而，这是一个特例。
   - **[2]** 正常情况使用括号 `()` 包裹参数。 为了保持一致性，也可以使用括号 `()` 包裹单个参数，虽然这种情况并不常见。
   - **[3]** 如果没有参数，则必须使用括号 `()` 表示空参数列表。
   - **[4]** 对于多个参数，将参数列表放在括号 `()` 中。

到目前为止，所有 Lambda 表达式方法体都是单行。 该表达式的结果自动成为 Lambda 表达式的返回值，在此处使用 **return** 关键字是非法的。 这是 Lambda 表达式简化相应语法的另一种方式。

**[5]** 如果在 Lambda 表达式中确实需要多行，则必须将这些行放在花括号中。 在这种情况下，就需要使用 **return**。

> Lambda表达式并不能取代所有的匿名内部类，只能用来取代**函数接口（Functional Interface）**的简写

例子1：无参函数的简写

如果需要新建一个线程，一种常见的写法是这样：

```java
// JDK7 匿名内部类写法
new Thread(new Runnable(){// 接口名
	@Override
	public void run(){// 方法名
		System.out.println("Thread run()");
	}
}).start();
```

上述代码给`Tread`类传递了一个匿名的`Runnable`对象，重载`Runnable`接口的`run()`方法来实现相应逻辑。这是JDK7以及之前的常见写法。匿名内部类省去了为类起名字的烦恼，但还是不够简化，在Java 8中可以简化为如下形式：

```java
// JDK8 Lambda表达式写法
new Thread(
		() -> System.out.println("Thread run()")// 省略接口名和方法名
).start();
```

上述代码跟匿名内部类的作用是一样的，但比匿名内部类更进一步。这里连**接口名和函数名都一同省掉**了，写起来更加神清气爽。如果函数体有多行，可以用大括号括起来，就像这样：

```java
// JDK8 Lambda表达式代码块写法
new Thread(
        () -> {
            System.out.print("Hello");
            System.out.println(" Hoolee");
        }
).start();
```

例子2：带参函数的简写

如果要给一个字符串列表通过自定义比较器，按照字符串长度进行排序，Java 7的书写形式如下：

```java
// JDK7 匿名内部类写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, new Comparator<String>(){// 接口名
    @Override
    public int compare(String s1, String s2){// 方法名
        if(s1 == null)
            return -1;
        if(s2 == null)
            return 1;
        return s1.length()-s2.length();
    }
});
```

上述代码通过内部类重载了`Comparator`接口的`compare()`方法，实现比较逻辑。采用Lambda表达式可简写如下：

```java
// JDK8 Lambda表达式写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, (s1, s2) ->{// 省略参数表的类型
    if(s1 == null)
        return -1;
    if(s2 == null)
        return 1;
    return s1.length()-s2.length();
});
```

上述代码跟匿名内部类的作用是一样的。除了省略了接口名和方法名，代码中把参数表的类型也省略了。这得益于`javac`的**类型推断**机制，编译器能够根据上下文信息推断出参数的类型，当然也有推断失败的时候，这时就需要手动指明参数类型了。注意，Java是强类型语言，每个变量和对象都必需有明确的类型。

**匿名内部类会产生额外的class文件，而使用Lambda表达式则不会**

### 比较器

> 确定两个对象之间的大小关系及排列顺序称为比较，能实现这个比较功能的类或方法称之为比较器

- 内部比较器是comparable接口
- 外部比较器是comparator接口

一位数组的降序排序：



#### forEach()

```java
// 使用曾强for循环迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(String str : list){
    if(str.length()>3)
        System.out.println(str);
}
```

```java
// 使用forEach()结合Lambda表达式迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach( str -> {
        if(str.length()>3)
            System.out.println(str);
    });
```

#### removeIf()

该方法签名为`boolean removeIf(Predicate<? super E> filter)`，作用是**删除容器中所有满足`filter`指定条件的元素**，其中`Predicate`是一个函数接口，里面只有一个待实现方法`boolean test(T t)`，同样的这个方法的名字根本不重要，因为用的时候不需要书写这个名字。

需求：*假设有一个字符串列表，需要删除其中所有长度大于3的字符串。*

我们知道如果需要在迭代过程冲对容器进行删除操作必须使用迭代器，否则会抛出`ConcurrentModificationException`，所以上述任务传统的写法是：

```java
// 使用迭代器删除列表元素
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Iterator<String> it = list.iterator();
while(it.hasNext()){
    if(it.next().length()>3) // 删除长度大于3的元素
        it.remove();
}
```

现在使用`removeIf()`方法结合匿名内部类，我们可是这样实现：

```java
// 使用removeIf()结合匿名名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(new Predicate<String>(){ // 删除长度大于3的元素
    @Override
    public boolean test(String str){
        return str.length()>3;
    }
});
```

上述代码使用`removeIf()`方法，并使用匿名内部类实现`Precicate`接口。相信你已经想到用Lambda表达式该怎么写了：

```java
// 使用removeIf()结合Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```

使用Lambda表达式不需要记忆`Predicate`接口名，也不需要记忆`test()`方法名，只需要知道此处需要一个返回布尔类型的Lambda表达式就行了

#### replaceAll()

```java
// 使用Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```

#### sort

```java
// List.sort()方法结合Lambda表达式
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.sort((str1, str2) -> str1.length()-str2.length());
```

...

## Streams API()

> 函数接口是指内部只有一个抽象方法的接口

#### forEach()

我们对`forEach()`方法并不陌生，在`Collection`中我们已经见过。方法签名为`void forEach(Consumer<? super E> action)`，作用是对容器中的每个元素执行`action`指定的动作，也就是对元素进行遍历。

```java
// 使用Stream.forEach()迭代
Stream<String> stream = Stream.of("I", "love", "you", "too");
stream.forEach(str -> System.out.println(str));
```

由于`forEach()`是结束方法，上述代码会立即执行，输出所有字符串。

#### Filter()

数原型为`Stream<T> filter(Predicate<? super T> predicate)`，作用是返回一个只包含满足`predicate`条件元素的`Stream`。

```java
// 保留长度等于3的字符串
Stream<String> stream= Stream.of("I", "love", "you", "too");
stream.filter(str -> str.length()==3)
    .forEach(str -> System.out.println(str));
```

上述代码将输出为长度等于3的字符串`you`和`too`。注意，由于`filter()`是个中间操作，如果只调用`filter()`不会有实际计算，因此也不会输出任何信息。





### 递归

整数n的阶乘：

```java
interface IntCall {
    int call(int n);
}

public class RecursionExample {
    static IntCall intCall;
    public static void main(String[] args) {
        intCall = n -> n == 0 ? 1 : n * intCall.call(n-1);
        Stream.of(intCall.call(10)).forEach(System.out::println);
    }
}
```

输出结果：

`3628800`

实现Fibonacci数列：

```java
public class RecursiveFibonacci {
    IntCall fib;

    RecursiveFibonacci() {
        fib = n -> {
            if (n == 0 || n == 1) {
                return n;
            } else {
                return fib.call(n - 1) + fib.call(n - 2);
            }
        };
    }

    public int calFib(int n) {
        return fib.call(n);
    }

    public static void main(String[] args) {
        RecursiveFibonacci rec = new RecursiveFibonacci();
        for (int i = 0; i < 10; i++) {
            System.out.print(rec.calFib(i) + " ");
        }
    }
}
```

输出结果：

`0 1 1 2 3 5 8 13 21 34` 

### 方法引用

Java8方法引用没有历史包袱。方法引用组成：类名或对象名，后面跟：：然后跟方法名称。

### Runnable接口

**Runnable** 接口自 1.0 版以来一直在 Java 中，因此不需要导入。它也符合特殊的单方法接口格式：它的方法 `run()` 不带参数，也没有返回值。因此，我们可以使用 Lambda 表达式和方法引用作为 **Runnable**：

```java
// functional/RunnableMethodReference.java

// 方法引用与 Runnable 接口的结合使用

class Go {
  static void go() {
    System.out.println("Go::go()");
  }
}

public class RunnableMethodReference {
  public static void main(String[] args) {

    new Thread(new Runnable() {
      public void run() {
        System.out.println("Anonymous");
      }
    }).start();

    new Thread(
      () -> System.out.println("lambda")
    ).start();

    new Thread(Go::go).start();
  }
}复制ErrorOK!
```

输出结果：

```java
Anonymous
lambda
Go::go()
```

**Thread** 对象将 **Runnable** 作为其构造函数参数，并具有会调用 `run()` 的方法 `start()`。 **注意**，只有**匿名内部类**才需要具有名为 `run()` 的方法。

### 函数式接口

方法引用和Lambda表达式都必须被赋值，同时复制需要类型信息才能使编译器保证类型的正确性尤其是Lambda 表达式，它引入了新的要求。 代码示例：

```java
x -> x.toString()复制ErrorOK!
```

我们清楚这里返回类型必须是 **String**，但 `x` 是什么类型呢？

Lambda 表达式包含类型推导（编译器会自动推导出类型信息，避免了程序员显式地声明）。编译器必须能够以某种方式推导出 `x` 的类型。

下面是第二个代码示例：

```java
(x, y) -> x + y 复制ErrorOK!
```

现在 `x` 和 `y` 可以是任何支持 `+` 运算符连接的数据类型，可以是两个不同的数值类型或者是 一个 **String** 加任意一种可自动转换为 **String** 的数据类型（这包括了大多数类型）。 但是，当 Lambda 表达式被赋值时，编译器必须确定 `x` 和 `y` 的确切类型以生成正确的代码。

```java
(x, y) -> x + y 复制ErrorOK!
```

现在 `x` 和 `y` 可以是任何支持 `+` 运算符连接的数据类型，可以是两个不同的数值类型或者是 一个 **String** 加任意一种可自动转换为 **String** 的数据类型（这包括了大多数类型）。 但是，当 Lambda 表达式被赋值时，编译器必须确定 `x` 和 `y` 的确切类型以生成正确的代码。

该问题也适用于方法引用。 假设你要传递 `System.out :: println` 到你正在编写的方法 ，你怎么知道传递给方法的参数的类型？

为了解决这个问题，Java 8 引入了 `java.util.function` 包。它包含一组接口，这些接口是 Lambda 表达式和方法引用的目标类型。 每个接口只包含一个抽象方法，称为函数式方法。

在编写接口时，可以使用 `@FunctionalInterface` 注解强制执行此“函数式方法”模式：

```java
// functional/FunctionalAnnotation.java

@FunctionalInterface
interface Functional {
  String goodbye(String arg);
}

interface FunctionalNoAnn {
  String goodbye(String arg);
}

/*
@FunctionalInterface
interface NotFunctional {
  String goodbye(String arg);
  String hello(String arg);
}
产生错误信息:
NotFunctional is not a functional interface
multiple non-overriding abstract methods
found in interface NotFunctional
*/

public class FunctionalAnnotation {
  public String goodbye(String arg) {
    return "Goodbye, " + arg;
  }
  public static void main(String[] args) {
    FunctionalAnnotation fa =
      new FunctionalAnnotation();
    Functional f = fa::goodbye;
    FunctionalNoAnn fna = fa::goodbye;
    // Functional fac = fa; // Incompatible
    Functional fl = a -> "Goodbye, " + a;
    FunctionalNoAnn fnal = a -> "Goodbye, " + a;
  }
}
```

`@FunctionalInterface` 注解是可选的; Java 在 `main()` 中把 **Functional** 和 **FunctionalNoAnn** 都当作函数式接口。 在 `NotFunctional` 的定义中可看到`@FunctionalInterface` 的作用：接口中如果有多个抽象方法则会产生编译期错误。

仔细观察在定义 `f` 和 `fna` 时发生了什么。 `Functional` 和 `FunctionalNoAnn` 定义接口，然而被赋值的只是方法 `goodbye()`。首先，这只是一个方法而不是类；其次，它甚至都不是实现了该接口的类中的方法。这是添加到Java 8中的一点小魔法：如果将方法引用或 Lambda 表达式赋值给函数式接口（类型需要匹配），Java 会适配你的赋值到目标接口。 编译器会在后台把方法引用或 Lambda 表达式包装进实现目标接口的类的实例中。

尽管 `FunctionalAnnotation` 确实适合 `Functional` 模型，但 Java不允许我们像`fac`定义中的那样，将 `FunctionalAnnotation` 直接赋值给 `Functional`，因为 `FunctionalAnnotation` 并没有显式地去实现 `Functional` 接口。唯一的惊喜是，Java 8 允许我们将函数赋值给接口，这样的语法更加简单漂亮。

`java.util.function` 包旨在创建一组完整的目标接口，使得我们一般情况下不需再定义自己的接口。主要因为基本类型的存在，导致预定义的接口数量有少许增加。 如果你了解命名模式，顾名思义就能知道特定接口的作用。

以下是基本命名准则：

1. 如果只处理对象而非基本类型，名称则为 `Function`，`Consumer`，`Predicate` 等。参数类型通过泛型添加。
2. 如果接收的参数是基本类型，则由名称的第一部分表示，如 `LongConsumer`，`DoubleFunction`，`IntPredicate` 等，但返回基本类型的 `Supplier` 接口例外。
3. 如果返回值为基本类型，则用 `To` 表示，如 `ToLongFunction <T>` 和 `IntToLongFunction`。
4. 如果返回值类型与参数类型一致，则是一个运算符：单个参数使用 `UnaryOperator`，两个参数使用 `BinaryOperator`。
5. 如果接收两个参数且返回值为布尔值，则是一个谓词（Predicate）。
6. 如果接收的两个参数类型不同，则名称中有一个 `Bi`。



