---
title: HashMap 与 HashTable区别
date: 2020-09-21 18:49:59
tags: [HashMap]
---

HashMap 是最常用的Map，它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的的访问速度。由于HahsMap和HashTable都采用了hash法进行索引，因此二者具有很多相似之处，他们主要有如下的区别：

- HashMap是HashTable的轻量级实现（非线程安全的实现），他们都完成了Map接口，主要区别在于HashMap允许空（null）键值（key） （最多允许一条记录的键为null，不允许多条记录的值为null），而HashTable不允许。
- HashMap把HashTable的contains方法去掉了，改成了containsValue和containsKey，因为contains方法容易引起误解。HashTable继承自Dictionary类，而HashMap是Java 1.2引进的Map 接口的一个实现。
- HashTable是线程安全的，而HashMap不支持线程的同步，所以它不是线程安全的。在多个线程访问HashTable时，不需要开发人员对它进行同步，而对于HashMap，开发人员必须提供额外的同步机制。所以，就效率而言，HashMap可能高于HashTable。
- HashTable使用Enumeration，HashMap使用Iterator。
- HashTable和HashMap采用的hash/rehash算法几乎都一样，所以性能不会有很大的差异。
- 在HashTable中，hash数组默认大小为11，增加方式是 old * 2 + 1。在HashMap中，hash数组的默认大小是16，而且一定2的指数倍扩容。
- hash值得使用不同，HashTable直接使用对象得hashCode。

Java还提供了一种WeakHashMap，与HashMap类似，不同之处在于WeakHashMap中key采用得是“弱引用”的方式，只要WeakHashMap中的key不再被外部引用，它就可以被GC回收。

HashMap中key采用的是“强引用的方式”，当HashMap中的key没有被外部引用时，只有在这个key从HashMap中删除后，才可以被GC回收。

**Q：** 

**在HashTable上下文中，同步指的是什么？**

同步意味着在一个时间点只能有一个线程可以修改hash表，任何线程在执行HashTable的更新操作前都需要获得对象锁，其他线程等待锁的释放。

**如何实现HashMap的同步？**

HashMap可以通过`Map m = Collections.synchronizedMap(new HashMap())`来达到同步效果。具体而言，该方法返回一个同步的Map，该Map封装了底层的HashMap的所有方法，使得底层的HashMap即使是在多线程的环境也是安全的。