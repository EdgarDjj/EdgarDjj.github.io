# Java并发容器和框架


# Java并发容器和框架

## ConcurrentHashMap 的实现原理与使用

ConcurrentHashMap是线程安全且高效的HashMap。

在并发编程中使用HashMap可能操作程序死循环，而使用线程安全的HashTable效率又十分低下，因此ConcurrentHashMap诞生。

1. HashMap在并发情况下，执行put操作会引起死循环，是因为多线程会导致HashMap的Empty链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永不为空，就会产生死循环，无法获取节点。
2. HashTable由于每个方法都是用了synchronized进行同步，因此在线程竞争激烈的情况下，会导致其它线程也访问HashTable的同步方法时会被阻塞或轮询状态，从而也无法自己使用get或put操作，造成效率低下。
3. ConcurrentHashMap的分段锁技术

HashTable在容器竞争激烈的并发情况下表现的效率低，是因为多个线程竞争一把锁，假如容器内有多把锁，每一个把锁中有一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而有效提升并发效率，这就是**分段锁**。、

### ConcurrentHashMap的结构

**JDK7：**

ConcurrentHashMap是由segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock）在ConcurrentHashMap扮演锁的角色。HashEntry则用于存储键值对数据。

一个ConcurrentHashMap里包含一个Segment[]数组，Segment的结构与HashMap结构类似，是一种链表加数组的散列表，一个Segment里包含一个HashEntry[]数组，每个HashEntry是一个种链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，每个想对HashEntry[]数组进行修改的时候，都要获取当前HashEntry所在Segment的锁。

**JDK8：**

ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全，数据结构和HashMap1.8的结构类似，数组+链表/红黑树。Java8在链表超过一定阈值（8）的时候，将链表（寻址时间复杂度O（n））转换为红黑树（O（log（n））

synchronized只锁定当前链表或红黑树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升了许多。

### ConcurrentHashMap如何保证线程安全



关于node节点的状态，有以下四种:

```java
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        //计算当前key的hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        //如果这次没有put成功，会重试
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //ConcurrentHashMap的初次put会做init操作
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            //如果对应的node为空，以cas的方式put元素，如果成功，直接退出循环
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //如果tab[i]的hash值为MOVED，表明该链表正在进行transfer操作,当前线程先帮助进行扩容操作,然后put
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                //锁住node节点，对节点下的链表/红黑树进行同步操作
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {  //大于0说明当前node是一个可用的链表节点
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                //如果当前key已经存在，判断是否新值替换旧值(根据传入的onlyIfAbsen决定),然后退出
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                //如果下一个节点为空，直接put进去，然后退出
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                                //继续遍历
                            }
                        }
                        //如果当前节点是一个树状结构，向树中插入一个元素
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //binCount是用来统计链表长度的
                if (binCount != 0) {
                    //如果长度达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        //统计长度，检查时候需要扩容
        addCount(1L, binCount);
        return null;
    }
```

## 关于Hash算法和哈希冲突

**Hash算法**:就是根据设定的Hash函数H(key)和处理冲突方法，将一组关键字映射到一个有限的地址区间上的算法。所以Hash算法也被称为散列算法、杂凑算法。

**Hash表**:通过Hash算法后得到的有限地址区间上的集合。数据存放的位置和key之前存在一定的关系(`H(key)=stored_value_hash(数据存放位置)`),可以实现快速查询。与之相对的，如果数据存放位置和key之间不存在任何关联关系的集合，称之为`非Hash表`。

**Hash冲突**:由于用于计算的数据是无限的`H(key),key属于(-∞,+∞)`,而映射到区间是有限的，所以肯定会存在两个key:key1,key2，H(key1)=H(key2)，这就是hash冲突。一般的解决Hash冲突方法有:开放定址法、再哈希法、链地址法（拉链法）、建立公共溢出区。

#### 开放地址法

开放定址法也称为`再散列法`，基本思想就是，如果`p=H(key)`出现冲突时，则以`p`为基础，再次hash，`p1=H(p)`,如果p1再次出现冲突，则以p1为基础，以此类推，直到找到一个不冲突的哈希地址`pi`。 因此开放定址法所需要的hash表的长度要大于等于所需要存放的元素，而且因为存在再次hash，所以`只能在删除的节点上做标记，而不能真正删除节点。`

缺点:容易产生堆积问题;不适合大规模的数据存储;插入时会发生多次冲突的情况;删除时要考虑与要删除元素互相冲突的另一个元素，比较复杂。

#### 再哈希法(双重散列，多重散列)

提供多个不同的hash函数，当`R1=H1(key1)`发生冲突时，再计算`R2=H2(key1)`，直到没有冲突为止。 这样做虽然不易产生堆集，但增加了计算的时间。

#### 链地址法(拉链法)

链地址法:将哈希值相同的元素构成一个同义词的单链表,并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。HashMap采用的就是链地址法来解决hash冲突。(链表长度大于等于8时转为红黑树)

#### 建立公共溢出区

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

#### HashMap中的处理冲突

下面是HashMap的put方法:

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) //如果hash数组为空，初始化一下
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)  //计算落在hash桶的位置，如果当前桶为空，直接新增节点
        tab[i] = newNode(hash, key, value, null);
    else {  //当前桶存在元素
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k)))) 
            //如果key已经存在，替换元素
            e = p;
        else if (p instanceof TreeNode) //如果当前是树结构了(不是链表了),向树上添加元素
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {  //当前结构依然时链表，遍历链表，直到末尾或者找到key相同的元素替换
            for (int binCount = 0; ; ++binCount) {
                //到达末尾，新增元素，如果链表长度达到8，转为红黑树
                if ((e = p.next) == null) { 
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //遍历链表的过程中，发现了有key相同的元素，直接替换，然后break
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { //如果是已经存在的元素，判断是否替换(onlyIfAbsent)
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //如果容量超过阈值，扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```


