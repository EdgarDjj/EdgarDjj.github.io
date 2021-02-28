# 面试题-容器


# 二、容器

18. Java的容器都有哪些？
    Map：HashMap->LinkHashSet、ConcurrentMap->ConcurrentHashMap、TreeMap
    Set：HashSet->LinkHashSet、TreeSet
    List：ArrayList、LinkedList、Vector->Stack
    Queue：Priority Queue，LinkedList
19. Collection 和 Collections 有什么区别？
    java.util.Collection 是一个集合接口（集合类是一个顶级接口）。他提供了集合对象进行基本操作的通用接口方法。Collection接口在Java类中有很多的具体的实现。Collection的意义是为了给各种集合统一的操作方式，其直接继承接口有List与Set。
    Collections则是集合类的一个工具类/帮助类，其中提供了一系列静态方法，用于对集合中元素进行排序、搜索以及线程安全等各种操作。

20. List、Set、Map之间的区别是什么？
    List：存储一个组不唯一（可以由多个元素引用相同的对象），有序的对象。
    Set：不允许重复的集合。不会有多个元素引用相同的对象。
    Map：使用键值对存储。Map会维护Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，也可以是任何类型。

21. HashMap 和 HashTable 有什么区别？
    HashMap去掉了HashTable的contains方法，但加上了containsValue（）方法和containsKey（）方法
    HashTable是同步的，所有的方法都是经过synchronized修饰的，是线程安全的
    HashMap允许空键值，而HashTable不允许

22. 如何决定使用HashMap还是TreeMap？
    对于在Map中插入、删除和定位元素这类操作，HashMap是最好的选择。
    然而如果需要对一个key集合进行遍历排序，TreeMap是更好地选择。基于你的Collection的大小，也许向HashMap中添加元素会更快，将map换成TreeMap会遍历更快

23. 说一下HashMap的实现原理？
    HashMap是基于hash表的Map接口非同步实现。
    JDK7之前是使用数组+链表实现的，结合在一起就是链表散列。JDK8之后在解决哈希冲突时有了较大的变化，HashMap在链表长度大于8（默认为8）的时候会自动转换为红黑树，提升搜索效率。

24. 说一下HashSet的实现原理？
    HashSet底层就是使用HashMap实现的，HashSet的值存在HashMap的键上，值为PRESENT（就是一个Object对象）

25. ArrayList 和 LinkedList 的区别
    相同点：
    都是非线程安全的，不保证同步。

ArrayList：
底层为Object[]数组，继承了RandomAccess接口，该接口什么都没有但表示ArrayList是一个支持随机访问的集合。在空间占用方面，Object[]有扩容的机制（每次扩大为当前容量的1.5倍）来保证有足够的空间进行插入，因此会空间浪费在这多余的空间上面。在元素添加和去除的上面（add和remove），ArrayList的采用的是System.arraycopy()和Arrays.copyOf()（实际上就是调用System.arraycopy())，来进行扩容，因此在需要大量元素操作的时候，对时间和空间效率的影响较大。

LinkedList：
底层使用的是双向链表（JDK7之前为循环双向链表，JDK7取消了循环），由于是链表，没有大小的限制，也无需进行扩容操作等，但不支持随机访问操作，需要遍历到需要的位置进行操作，但对于末尾和头部的插入时间复杂度不受影响，但对于指定位置ArrayList为O（n-i）而LinkedLIst为O（n），在空间的占用上，主要浪费在头尾节点指针上。

26. 如何实现数组和List之间的转换？
    a. 使用循环，将List中的元素添加到数组中去
    List->数组：调用ArrayList中的toArray（）方法，但需要使用子类进行初始化，不可使用List接口进行初始化。
    数组->List：调用Arrays的asList方法。

27. ArrayList和Vector的区别？
    从线程安全角度：Vector是线程安全的而ArrayList不是，Vector的每个方法都有synchronized进行修饰。
    Vector由于有锁的修饰，因此在效率方面不如ArrayList，在不考虑线程安全的情况下应该避免使用Vector。

28. 数组 和 ArrayList 有何区别？
    数组在进行时候的时候必须指定其大小，且没有ArrayList所自带的方法（add，remove，indexOf等等等），ArrayList的大小是动态的，数组可以容纳基本类型和对象，但ArrayList只能接受对象。

29. 在Queue 中 poll（）和remove（）有什么区别？
    Queue是一个接口，LinkedList继承了它，remove（）在元素为空进行使用会抛出异常，而poll（）则会返回空，这就是二者的区别。

30. 哪些集合类是线程安全的？
    Vector、ConcurrentHashMap、HashTable、Stack（在jdk6的时候推出Deque->Deque<T> stack = new ArrayDeque<>())、enum（static类型的属性会在类加载之后被初始化，当一个Java类第一次被真正使用到的时候静态资源被初始化、Java类的加载和初始化过程都是线程安全的）

31. Iterator 是什么？
    迭代器是一种设计模式，是一个对象，可以轻松遍历并选择序列中的对象，而开发者无需了解该序列的底层数据结构。

32. Iterator如何使用？有什么特点？
    Java中的Collection实现了Iterable接口，里面有个Iterator方法，在我们需要使用的时候，调用该集合的Iterator方法，即可初始化一个迭代器可以对集合进行迭代。
    1、使用iterator（）返回一个iterator
    2、使用next（）获取序列下一个元素
    3、使用hashNext（）判断下一个元素是否存在
    4、使用remove（）将迭代器新返回的元素删除

33. Iterator 和 ListIterator有什么区别？
    Iterator可以用来遍历Set和List集合，但是ListIterator只能用来遍历List
    Iterator对集合只可前向遍历，而ListIterator即可前也可后向
    ListIterator实现了Iterator接口，并拓展了其它功能


