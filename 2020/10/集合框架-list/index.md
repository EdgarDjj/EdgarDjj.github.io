# 集合框架


# Collection-List

## 概述

![img](集合框架/Java-Collections.jpeg)

除了Map结尾的类以外，其他的都继承了Collection接口。

* **List**(对付顺序)： List接口存储一组不唯一（可以有多个元素引用相同的对象），有序的对象
* **Set**(注重独一无二的性质): 不允许重复的集合。不会有多个元素引用相同的对象。
* **Map**(用Key搜索): 使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象。

> Map接口没有继承自Collection接口，因为Map表示的是关联式容器而不是集合。但Java为我们提供了从Map转换到Collection的方法，可以方便的将Map切换到集合视图。 上图中提供了Queue接口，却没有Stack，这是因为Stack的功能已被JDK 1.6引入的Deque取代。

### 泛型（Generics）

JDK1.5引入泛型，所有集合接口和实现都大量地使用，使得Java容器可以接受任何类型的对象。即是编译提供的一个**语法糖**。

>泛型本身并不需要Java虚拟机的支持，只需要在编译阶段做一下简单的字符串替换即可。实质上Java的单继承机制才是保证这一特性的根本，因为所有的对象都是Object的子类，容器里只要能够存放Object对象就行了。 事实上，所有容器的内部存放的都是Object对象，泛型机制只是简化了编程，由编译器自动帮我们完成了强制类型转换而已。而在此之前，类型转换需要显式完成。  

### 内存管理

>跟C++复杂的内存管理机制不同，Java GC自动包揽了一切，Java程序并不需要处理令人头疼的内存问题，因此JCF并不像C++ STL那样需要专门的空间适配器（alloctor）。 另外，由于Java里对象都在堆上，且对象只能通过引用（reference，跟C++中的引用不是同一个概念，可以理解成经过包装后的指针）访问，容器里放的其实是对象的引用而不是对象本身，也就不存在C++容器的复制拷贝问题。

### Collections 和 Collection 的区别？

Collection是一个集合接口。它提供了对集合对象进行基本操作的通用接口方法。而Collections是一个包装类，包含有关集合操作的静态多态方法，仅用于服务Java的Collection框架。

### Collection

Collection集合下的所有集合都共有以下基本方法：

* add()
* addAll(Collection)
* remove()
* removeAll(Collection)
* removeIf()
* size()
* contains(Object)
* clear()
* isEmpty()
* toArray()
* retainAll(Collection)
  * 从此集合中删除指定集合中不包含的所有元素

### List、Set、Map

List：存储的元素有序，可重复

Set：存储的元素无序，不可重复

Map：使用键值对（key-value）存储，key是无序不可重复的，value是无序可重复的

#### List

ArrayList：Object[]数组

Vector：Object[]数组

LinkedList：双向链表（JDK1.6为循环链表，JDK1.7取消了）

#### Set

HashSet（无序，唯一）：基于HashMap实现的，底层采用HashMap来保存元素

LinkedHashSet：是HashSet的子类

TreeSet（有序，唯一）：红黑树（自平衡的排序二叉树）

#### Map

HashMap：JDK8之前HashMap由数组+链表组成的，数组是HashMap的主题，链表则是主要为了解决哈希冲突而存在的（“拉链法”）

ListedHashMap：LinkedHashMap继承自HashMap，在其结构的基础上，增加了一条双向链表，是上面结构可以保持键值对的插入顺序。

TreeMap：红黑树（自平衡的排序二叉树）

### ArrayList、LinkedList和Vector的区别

![list](集合框架/list.png)

ArrayList是一个可改变大小的数组。当更多元素加入到ArrayList中时，其大小将会动态地增大，但每一次扩大都是进行数组的copyOf()，因此若要进行大量增删操作时不适合使用，但若进行查找操作时效率比LinkedList快的多。

LinkedList是一个双链表，在添加和删除元素时具有比ArrayList更好的性能。但在get与set方面远弱于ArrayList。

Vector 和ArrayList类似,但属于强同步类。如果你的程序本身是线程安全的(thread-safe,没有在多个线程之间共享同一个集合/对象),那么使用ArrayList是更好的选择。

而 LinkedList 还实现了 Queue 接口,该接口比List提供了更多的方法,包括 offer(),peek(),poll()等。

注意: 默认情况下ArrayList的初始容量非常小,所以如果可以预估数据量的话,分配一个较大的初始值属于最佳实践,这样可以减少调整大小的开销。

> 同步处理：Vector同步，ArrayList非同步 Vector缺省情况下增长原来一倍的数组长度，ArrayList是0.5倍. ArrayList: int newCapacity = oldCapacity + (oldCapacity >> 1); ArrayList自动扩大容量为原来的1.5倍（实现的时候，方法会传入一个期望的最小容量，若扩容后容量仍然小于最小容量，那么容量就为传入的最小容量。扩容的时候使用的Arrays.copyOf方法最终调用native方法进行新数组创建和数据拷贝）
>
> Vector: int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
>
> Vector指定了`initialCapacity，capacityIncrement`来初始化的时候，每次增长`capacityIncrement`

#### ArrayList 和 LinkedList 区别？

* 线程安全：都不同步，即不保证线程安全
* 底层数据结构：
  * ArrayList 底层使用的是 Object 数组 
  * LinkedList 底层使用的是**双向链表**
* 插入与删除：
  * ArrayList：插入指定位置的时候，第i个元素和第i个元素之后的（n-i）元素都要执行向后移一位的操作。只在执行插入末尾的时候复杂度为O（1）
  * LinkedList：插入和删除不受位置影响，而指定位置的时候需要移动到指定位置在插入。
* 访问查询：
  * ArrayList支持随机访问RandomAccess接口的实现
  * LinkedList不支持高效的随机访问，在访问元素为中间的时候效率最低，由于为双向链表，所以在判断访问元素的下标时，会考虑从后往前还是从前往后。
* 空间占用：
  * ArrayList的空间浪费主要在list列表的结尾会预留一定的容量空间，而LinkedList的主要浪费在前后指针上和数据节点。

#### RandomAccess接口

```java
public interface RandomAccess {
}
```

所以，在我看来 `RandomAccess` 接口不过是一个标识罢了。标识什么？ 标识实现这个接口的类具有随机访问功能。

`ArrayList` 实现了 `RandomAccess` 接口， 而 `LinkedList` 没有实现。为什么呢？我觉得还是和底层数据结构有关！`ArrayList` 底层是数组，而 `LinkedList` 底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为快速随机访问。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问。，`ArrayList` 实现了 `RandomAccess` 接口，就表明了他具有快速随机访问功能。 `RandomAccess` 接口只是标识，并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！

### ArrayList的扩容机制

**ArrayList有三种方式来初始化，构造方法源码如下：**

```java
   /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
    

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {//初始容量大于0
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//初始容量等于0
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }


   /**
    *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
    *如果指定的集合为null，throws NullPointerException。 
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

即以无参构造方法创建ArrayList的时候，实际上是赋值一个空数组。但在添加元素的时候才真正分配容量。

#### add方法

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

在执行add方法时，会先执行ensureExplicitCapacity()方法



```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

> **注意** ：JDK11 移除了 `ensureCapacityInternal()` 和 `ensureExplicitCapacity()` 方法

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

> DEFAULT_CAPACITY=10
>
> 当 要 add 进第1个元素时，minCapacity为1，在Math.max()方法比较后，minCapacity 为10。

- 当我们要 add 进第1个元素到 ArrayList 时，elementData.length 为0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为10。此时，`minCapacity - elementData.length > 0 `成立，所以会进入 `grow(minCapacity)` 方法。
- 当add第2个元素时，minCapacity 为2，此时e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0 `不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第3、4···到第10个元素时，依然不会执行grow方法，数组容量都为10。

直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。

#### grow（）方法

```java
/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
  	// oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
  	// newCpaacity相当与oldCapacity的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**int newCapacity = oldCapacity + (oldCapacity >> 1),所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity为偶数就是1.5倍，否则是1.5倍左右）！** 奇偶不同，比如 ：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数.

> ">>"（移位运算符）：>>1 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源 　

**我们再来通过例子探究一下`grow()` 方法 ：**

- 当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 `hugeCapacity` 方法。数组容量为10，add方法中 return true,size增为1。
- 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
- 以此类推······

**这里补充一点比较重要，但是容易被忽视掉的知识点：**

- java 中的 `length `属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
- java 中的 `length()` 方法是针对字符串说的,如果想看这个字符串的长度则用到 `length()` 这个方法.
- java 中的 `size()` 方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!

```Java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

### System.arraycopy()和Arrays.copyOf()方法

#### `System.arraycopy()` 方法

```java
    /**
     * 在此列表中的指定位置插入指定的元素。 
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()方法实现数组自己复制自己
        //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }
```

#### `Arrays.copyOf()`方法

```java
   /**
     以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 返回的数组的运行时类型是指定数组的运行时类型。 
     */
    public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
        return Arrays.copyOf(elementData, size);
    }
```

Arrays.copyOf()主要是为了给原有数组进行扩容

### Iterator迭代器

Iterator对象称为迭代器（设计模式的一种），迭代器可以对集合进行遍历，但每个集合内部的数据结构都是不尽相同的，所以每一个集合的存或取都很可能是不一样的，但如果每一个类中都定义一个hasNext（）和next（）方法，会过于臃肿。

迭代器是将这样的方法抽取出接口，然后在每个类的内部，定义自己迭代方式，这样做就规定了整个集合体系的遍历方式都是 `hasNext()`和`next()`方法，使用者不用管怎么实现的，会用即可。迭代器的定义为：提供一种方法访问一个容器对象中各个元素，而又不需要暴露该对象的内部细节。

1. 使用 `iterator()` 方法要求集合返回一个 **Iterator**。 **Iterator** 将准备好返回序列中的第一个元素。
2. 使用 `next()` 方法获得序列中的下一个元素。
3. 使用 `hasNext()` 方法检查序列中是否还有元素。
4. 使用 `remove()` 方法将迭代器最近返回的那个元素删除。




