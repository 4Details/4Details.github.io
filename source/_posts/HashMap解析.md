---
title: HashMap解析
date: 2020-08-22 23:25:32
tags: [HashMap]
category: [算法笔记]
top: 27
---

#  HashMap继承体系

![HashMap继承体系](https://s1.ax1x.com/2020/09/08/wQ8ldI.png)

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

![结构](https://s1.ax1x.com/2020/09/08/wQ1ek6.png)

外层Node长度为16，当链表长度大于达到8且hash中所有元素达到64个，链表会升级为红黑树。

**HashMap底层结构其实就是 数组+链表+红黑树**

Hash碰撞指的是不同的key被映射到相同的位置，hashMap使用链表连接这些碰撞值，当碰撞非常严重时，查询效率由O(1) 退化为 O(n)

jdk1.8 之后引入红黑树（自平衡的二叉查找树） 的目的就是解决链化严重的问题。

hashMap加入红黑树优化不是一开始的选择，所以没有向TreeMap那样，要求实现Comparable接口或者提供相应的比较器。但由于树化过程需要比较两个键对象的大小，在键类没有实现Comparable接口的情况下，hashMap做出以下处理以保证可以比较出两个键的大小：

1. 比较键与键之间hash的大小，如果hash相同，则继续
2. 检测键类是否实现了Comparable接口，如果实现了则调用`compareTo()`方法进行比较
3. 如果仍未比较出大小，就需要进行仲裁，方法为 `tieBreakOrder`

##  核心元素

```java
// 缺省table大小  16，必须是2的次幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
// table最大长度
static final int MAXIMUM_CAPACITY = 1 << 30;
// 负载因子大小，默认 0.75f，计算得到扩容阈值  16 * 0.75 = 12。大于1数组不会扩容，牺牲性能换内存
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 树化阈值 8，链表长度超过8，链表会被升级成树
static final int TREEIFY_THRESHOLD = 8;
// 树降级成链表的阈值
static final int UNTREEIFY_THRESHOLD = 6;
// hash表中元素超过64且某个链上元素超过8，才会树化。
// 树化阈值的第二条件，不满足链表不会转换成红黑树，而是优先扩容数组。
static final int MIN_TREEIFY_CAPACITY = 64;

// Node数组构建的 Hash表
// 什么时候初始化？put第一个元素的时候
transient HashMap.Node<K,V>[] table;
// 当前hash中元素个数
transient int size;
// 当前哈希表中结构修改次数，导致结构变化的修改
transient int modCount;
// 扩容阈值， 当哈希表中的元素超过阈值时，会触发扩容
int threshold;
// 负载因子
// 计算threshold  = capacity * loadFactor
final float loadFactor;

```

##  继承关系

```java
// table 内部数组是节点类型
static class Node<K,V> implements Map.Entry<K,V> {
     final int hash; 
     final K key;
     V value;
     Node<K,V> next; //下一个节点
    //省略...
}
```

hashMap内部数组是节点类型的，hash冲突解决方案是**拉链法**。hash值是经过`hash()`方法处理过的hashCode，也就是数组索引`bucket`，为了使hashCode分布更加随机。

```java
java.util.HashMap<K, V>.Node<K, V>
    java.util.LinkedMap<K, V>.Entry<K, V>
        java.util.HashMap<K, V>.TreeNOde<K, V>
```

TreeNode 是 Node的子类，继承关系如下：

Node是单向链表节点，Entry是双向链表节点，TreeNode是红黑树节点。

##  简单总结

HashMap是一个基于拉链法实现的一个散列表，内部由数组、链表和红黑树实现。

1. 数组的初始大小为16，而且是以2的次幂扩容，一是为了提高性能能使用足够大的数组；二是为了能使用位运算代替取模运算（据说提升了5-8倍）
2. 数组是否需要扩容是根据负载因子来判断的，如果当前数组中元素个数大于`数组长度 * 负载因子`大小时会触发扩容。默认负载因子为0.75，可由构造函数传入。当然也可以设置负载因子大于1，这样数组不会发生扩容，牺牲性能，节省内存。
3. 为了解决hash碰撞，数组中的元素是单向链表型。当链表长度超过一个阈值（默认是8）时，会将链表转化为红黑树以提高性能；而当链表长度缩小到一个阈值（默认是6）时，红黑树又会退化成链表来提高性能。
4. 补充说明，在链表树化之前还会检查数组中元素总数有没有超过一个阈值（默认64），如果没有达到阈值，则会放弃树化，优先选择扩容数组提高性能；如果超过了阈值，则会发生链表树化。

> **第二条件？** 
>
> 当bucket数目较小时，hash散列产生冲突的概率较高，导致链表长度过长，此时应该扩容，而不是立即树化（保证查找性能）。产生碰撞的主要原因是bucket的数目过少，优先扩容可以避免一些不必要的树化过程。同时，当bucket容量较少时，扩容会比较频繁，扩容时需要差分红黑树并重新映射。**所以在bucket容量较小情况下，将长链表树化不是最优选择。**
>
> **以6和8来作为平衡点的原因：**
>
> 红黑树的平均查找长度为O(log n)，长度为8查找长度为3，链表的平均查找长度为4 （n/2），这才有转化为树的必要；链表长度如果是≤6，虽然速度也很快，但是转化为树结构和生成树花费的时间不会太短。
>
> 6和8之间有个插值7，可以防止链表和树之间频繁切换。假设转换阈值只有8，那么链表长度超过8就会树化，小于8就会转化为链表，如果一个HashMap不断地插入、删除元素，链表长度在8左右徘徊，就会频繁发生树化、链表转树，效率会很低。
>
> - 链表：如果元素个数小于8，查询成本高，退化为O(n)，新增成本低；
> - 红黑树：如果元素个数大于8，查询成本低，新增成本高

# 源码分析

`put`、`resize`、`get`、`remove`、`replace`

##  构造方法

```java
// 构造方法一：默认数组初始容量为16，负载因子为0.75f
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

// 构造方法二：指定数组的初始容量
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 构造方法三
public HashMap(int initialCapacity, float loadFactor) {
// 校验：0 < initialCapacity <  MAXIMUM_CAPACITY; 
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                    initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
    // 校验：loadFactor > 0
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                    loadFactor);
    
        this.loadFactor = loadFactor;
    // tableSizeFor() 返回一个大于等于initialCapacity的2的次幂数
        this.threshold = tableSizeFor(initialCapacity);
    }

/**
* tableSizeFor() 返回一个大于等于initialCapacity的2的次幂数
* 任何一个十进制数，转换为二进制，
* 函数返回结果是 从最高位1开始，所有位全部变为 1，最后再 +1 （即进位变为2的次方数）输出
*/
// 用实际例子可以测试，如 10 可以得到 16
static final int tableSizeFor(int cap) {
        int n = cap - 1; // 若跳过，得到结果会 *2 
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

##  数组的索引bucket

HashMap使用hash算法来决定集合中元素的存储位置， 每当系统初始化HashMap时，会创建一个 **capacity** 的数组，这个数组中可以存储元素的位置被称为bucket，每个bucket都有其索引，可以根据索引快速访问存储的元素。


```java
public V put(K key, V value) {
    // 传入的key经过了 hash(key) 方法
    return putVal(hash(key), key, value, false, true);
}


/*
*  作用： 在table长度不大得时候 让key的hash值得高16位也参与路由运算
*  h = 0b 0010 0101 1010 1100 0011 1111 0010 1110
*  0b 0010 0101 1010 1100 0011 1111 0010 1110
*  ^
*  0b 0000 0000 0000 0000 0010 0101 1010 1100
*  => 0010 0101 1010 1100 0001 1010 1000 0010
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

Java中每个对象都有一个`hashCode()`方法，这个就是散列函数，可以返回一个32位整数（为了尽可能避免发生碰撞）

## HashMap.put()  流程

1. map.put(key,value)  map.put("tkey", "tv") 
2. 获取key的hash值    
3. 经过hash值扰动函数，使此hash值更散列
4. 构造Node对象  Hash-> 1122, Key-> "tkey", Value->"tv", Next-> null
5. 路由算法，找出node应存放的数组的位置

> 路由寻址公式   (table.length - 1) & node.hash   // table.length一定是2的次幂
>
> (16 - 1) & 1122 => 0000 0000 1111 & 0100 0110 0010 => 0010 => 2  将此次的值放在index为2的位置

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// onlyIfAbsent：当存入键值对时，如果该key已存在，是否覆盖它的value。false为覆盖，true为不覆盖 参考putIfAbsent()方法。
// evict：用于子类LinkedHashMap。
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    HashMap.Node<K,V>[] tab; // tab：内部数组
    HashMap.Node<K,V> p;   // p：hash对应的索引位中的首节点
    int n, i;  // n：内部数组的长度    i：hash对应的索引位
    
    // 首次put时，内部数组为空，扩充数组。
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 计算数组索引，获取该索引位置的首节点，如果为null，添加一个新的节点
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {   
        HashMap.Node<K,V> e; K k;
        // 如果首节点的key和要存入的key相同，那么直接覆盖value的值。
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果首节点是红黑树的，将键值对插添加到红黑树
        else if (p instanceof HashMap.TreeNode)
            e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 此时首节点为链表，如果链表中存在该键值对，直接覆盖value。
        // 如果不存在，则在末端插入键值对。然后判断链表是否大于等于7，尝试转换成红黑树。
        // 注意此处使用“尝试”，因为在treeifyBin方法中还会判断当前数组容量是否到达64，
        // 否则会放弃次此转换，优先扩充数组容量。
        else {
            // 走到这里，hash碰撞了。检查链表中是否包含key，或将键值对添加到链表末尾
            for (int binCount = 0; ; ++binCount) {
                // p.next == null，到达链表末尾，添加新节点，如果长度足够，转换成树结构。
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 检查链表中是否已经包含key
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }

        // 覆盖value的方法。
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount; // fail-fast机制
    
    // 如果元素个数大于阈值，扩充数组。
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

  **`put()`步骤小结** 

1. 检查数组是否为空，执行`resize()`方法扩容；
2. 通过hash值计算索引，获得该索引位的首节点
3. 如果首节点为null，没有发生碰撞，直接添加节点到该索引位（bucket）
4. 如果首节点不为null，分三种情况：
   - key和首节点的key相同，覆盖old value（保证key的唯一性）；
   - 如果首节点是红黑树节点（TreeNode），将键值对添加到红黑树
   - 如果首节点是链表，将键值对添加到链表，添加之后会判断是否达到树化阈值
5. 判断数组是否达到扩容阈值

## HashMap扩容原理

hash数组太小，链化严重，查询效率退化为O(n)，为了解决哈希冲突导致的上述问题，扩容会缓解，以空间换时间。

```java
final HashMap.Node<K,V>[] resize() {
    HashMap.Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果数组已经是最大长度，不进行扩充。
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 否则数组容量扩充一倍。（2的N次方）
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 如果数组还没创建，但是已经指定了threshold（这种情况是带参构造创建的对象），threshold的值为数组长度
    // 在 "构造函数" 那块内容进行过说明。
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 这种情况是通过无参构造创建的对象
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 可能是上面newThr = oldThr << 1时，最高位被移除了，变为0。
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
  
    // 到了这里，新的数组长度已经被计算出来，创建一个新的数组。
    @SuppressWarnings({"rawtypes","unchecked"})
    HashMap.Node<K,V>[] newTab = (HashMap.Node<K,V>[])new HashMap.Node[newCap];
    table = newTab;
    
    // 下面代码是将原来数组的元素转移到新数组中。问题在于，数组长度发生变化。 
    // 那么通过hash%数组长度计算的索引也将和原来的不同。
    // jdk 1.7中是通过重新计算每个元素的索引，重新存入新的数组，称为rehash操作。
    // 这也是hashMap无序性的原因之一。而现在jdk 1.8对此做了优化，非常的巧妙。
    if (oldTab != null) {
        
        // 遍历原数组
        for (int j = 0; j < oldCap; ++j) {
            // 取出首节点
            HashMap.Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 如果链表只有一个节点，那么直接重新计算索引存入新数组。
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果该节点是红黑树，执行split方法，和链表类似的处理。
                else if (e instanceof HashMap.TreeNode)
                    ((HashMap.TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                
                // 此时节点是链表
                else { // preserve order
                    // loHead，loTail为原链表的节点，索引不变。
                    HashMap.Node<K,V> loHead = null, loTail = null;
                    // hiHeadm, hiTail为新链表节点，原索引 + 原数组长度。
                    HashMap.Node<K,V> hiHead = null, hiTail = null;
                    HashMap.Node<K,V> next;
                    
                   // 遍历链表
                    do {
                        next = e.next;
                        // 新增bit为0的节点，存入原链表。
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 新增bit为1的节点，存入新链表。
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原链表存回原索引位
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 新链表存到：原索引位 + 原数组长度
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

在jdk 1.7中，通过rehash操作，遍历每一个元素、每一个节点，重新计算索引值，存入新的数组中。

jdk1.8 之后对rehash进行了优化，数组以2的次幂数扩容，有以下规律：

元素的位置要么在原位置，要么是在原位置再移动2的次幂的位置。因此，在扩容HashMap时无需重新计算hash值，只需要找出原hash值新增的那个bit位是1还是0即可，是0则索引不变，是1索引变成`原索引+oldCap`，以上操作可以简单通过比特位的变化判断索引位，效率显著提高。

##  HashMap.get()  方法

```java
public V get(Object key) {
    Node<K,V> e;
    // 也会获取节点时也调用了hash()方法
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    // tab：内部数组  
    // first: 索引位首节点 
    // n: 数组长度 
    // k: 索引位首节点的key
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 数组不为null 数组长度大于0 索引位首节点不为null
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 如果索引位首节点的hash==key的hash 或者 key和索引位首节点的k相同
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            // 返回索引位首节点(值对象)
            return first;
        if ((e = first.next) != null) {
            // 如果是红黑树节点 则到红黑树中查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 发送碰撞 key.equals(k)
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**get方法小结**

1. 检查数组是否为null，索引位节点是否为null
2. 如果索引节点的hash==key的hash 或者key和索引节点的k相同则直接返回索引首节点（bucket的第一个节点）
3. 如果是红黑树，则到红黑树中查找
4. 如果有冲突，则通过key.equals(k)查找
5. 以上都没找到则返回null

##  树化、树转链表

```java
static final int TREEIFY_THRESHOLD = 8;

/**
 * 当桶数组容量小于该值时，优先进行扩容，而不是树化
 */
static final int MIN_TREEIFY_CAPACITY = 64;

static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
}

/**
 * 将普通节点链表转换成树形节点链表
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 桶数组容量小于 MIN_TREEIFY_CAPACITY，优先进行扩容而不是树化
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // hd 为头节点（head），tl 为尾节点（tail）
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将普通节点替换成树形节点
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);  // 将普通链表转成由树形节点链表
        if ((tab[index] = hd) != null)
            // 将树形链表转换成红黑树
            hd.treeify(tab);
    }
}

TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

```java
// 红黑树转链表阈值
static final int UNTREEIFY_THRESHOLD = 6;

final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    /* 
     * 红黑树节点仍然保留了 next 引用，故仍可以按链表方式遍历红黑树。
     * 下面的循环是对红黑树节点进行分组，与上面类似
     */
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    if (loHead != null) {
        // 如果 loHead 不为空，且链表长度小于等于 6，则将红黑树转成链表
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            tab[index] = loHead;
            /* 
             * hiHead == null 时，表明扩容后，
             * 所有节点仍在原位置，树结构不变，无需重新树化
             */
            if (hiHead != null) 
                loHead.treeify(tab);
        }
    }
    // 与上面类似
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

#  面试相关

1. **是否使用过HashMap？什么是HashMap？为何会使用？**

2. **能否让HashMap同步？**

   `Map map = Collections.synchronizeMap(hashMap)`

3. **HashMap的工作原理？**

   初始化数组16，以2的次幂数扩容；扩容阈值由负载因子判断；拉链法解决hash冲突；冲突节点数大于8且数组中节点总数大于64发生树化，链表节点数小于6，红黑树转化成链表。

4. **HashMap的put()和get()方法的原理？**

   put方法

   - 首次进入，数组为空，需要resize扩容；
   - 计算数组索引，获取该位置首节点，为空则直接插入；
   - 若不为空，分三种情况：
     - 如果首节点的key和要插入节点的key相同（冲突），直接覆盖old value
     - 如果首节点是红黑树节点，则将键值对插入红黑树
     - 如果首节点是链表节点，存在该键值对，则直接覆盖old value，如果不存在该键值对，判断是否应该树化。判断满足什么条件，做出相应的相应（见前文）。

   get方法

   - 检查数组是否为null，索引位是否为null；
   - 如果索引位节点hash与查询节点key的hash相同， 或者key与索引节点的k相同直接返回索引首节点；
   - 如果是红黑树，则到红黑树中查找；
   - 如果有冲突，则通过key.equals(k)查找；
   - 以上均无返回值，则get结果为null。

5. **当两个对象的hashcode相同会发生什么？**

   两个对象的hashcode相同说明两者在数组中的索引bucket相同，会发生哈希冲突。HashMap使用链表存储对象，这个Entry会存储在链表中，存储时会检查链表中是否包含key（key != null && key.equals(k) ， 或者将键值对添加到链表尾部。如果链表长度≥8，链表树化....）。

6. **如果两个键的hashcode相同，如何获取值对象？**

   两个对象的hashcode相同说明在数组中的索引位置相同，找到索引位bucket之后，会调用keys.equals()方法去找到链表中的正确节点（key != null && key.equals(k) )

7. **怎么减少碰撞 ？**

   使用final修饰的对象、不可变的对象作为键，例如Integer、String（不可变，final，而且已经重写了equals和hashCode方法）作为键比较好。

   可以使用自定义的对象作为键，只要遵守了equals和hashCode方法定义规则，并且当对象插入到hashMap中之后将不会再改变。

8. **如果HashMap的大小超过了负载因子的定义容量，怎么办？**

   会调用`resize()`方法进行数组扩容

9. **了解重新调整HashMap大小存在什么问题麽？**

   在多线程的情况下，可能产生条件竞争。

   因为如果两个线程都发现HashMap需要重新调整大小，二者会同时试着调整数组大小。在调整过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表尾部而是放在头部（头插法），这是为了避免尾部遍历（Tail Traversing）。如果竞争条件发生了，那么就会造成死循环。

10. **HashMap是非线程安全的，那么原因是什么？（HashMap的死锁）**

    由于HashMap的容量是有限的，如果HashMap中的数组的容量很小，存储大量元素时，冲突就会很频繁，此时O(1)的查找效率就会退变成O(n)，这是hash表的缺陷。

    为了解决上述问题，为HashMap设计了一个阈值，即负载因子 * 数组长度，超过阈值则会触发自动扩容。

11. **影响HashMap性能的因素？**

    负载因子；

    哈希值，理想情况是均匀分布散列到各个bucket。一般HashMap使用String类型作为key，而String类重写了hashCode函数。