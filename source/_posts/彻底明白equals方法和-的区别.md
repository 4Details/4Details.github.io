---
title: 彻底明白equals方法和==的区别
date: 2020-09-30 13:26:56
tags: []
category: [Java基础]
---

被Java基础中的`equal()`和`==`困扰很久，决心弄一篇博客，让自己彻底记住！

一直固有的观点就是：`==`是比较内存地址，`equals()`是比较值，有些类（如`String`）重写了`equals()`方法

但是这种观点不够完善：

- `==` 比较基本数据类型的时候，比较的是值，引用数据类型则比较的是地址（**`new`的对象，`==`的结果永远是false**）

- `equals()`属于`Object`类的方法，如果没有重写过`euqals()`方法，那么等同于`==`，但是`String`类里面的`equals()`方法被重写过了，所以比较的是值。

下面通过几个例子来强化

**Demo 01**

```java
/**
 * equals和等等的区别
 *
 */
public class EqualsDemo {

    static class User {
        private String name;

        public User(String name) {
            this.name = name;
        }
    }
    public static void main(String[] args) {
        String s1 = new String("abc");
        String s2 = new String("abc");

        System.out.println(s1 == s2);
        System.out.println(s1.equals(s2));
        Set<String> set1 = new HashSet<>();
        set1.add(s1);
        set1.add(s2);
        System.out.println(set1.size());

        System.out.println("==============");

        String s3 = "cbd";
        String s4 = "cbd";

        System.out.println(s3 == s4);
        System.out.println(s3.equals(s4));
        Set<String> set3 = new HashSet<>();
        set3.add(s3);
        set3.add(s3);
        System.out.println(set3.size());

        System.out.println("==============");

        Person user1 = new User("abc");
        Person user2 = new User("abc");
        System.out.println(user1 == user2);
        System.out.println(user1.equals(user2));
        Set<Person> set2 = new HashSet<>();
        set2.add(user1);
        set2.add(user2);
        System.out.println(set2.size());
    }
}

```

打印出来的结果：

```bash
false（==：如果是new出来的对象，比较的时候永远是false）
true：（字符串中的equals被重写过，比较的是值）
1：（HashSet底层是HashMap，HashMap内部是调用equals 和 HashCode，但是String内部的HashCode和equals也被复写）
==============
true（我们通过这种方式创建的会放在一个字符串常量池中，相同的字符串，会指向常量池中同一个对象，因此他们的地址是一样的）
true（字符串中的equals被重写过，比较的是值）
1
==============
false（==：如果是new出来的对象，比较的时候永远是false）
false（Person中的equals没有被重写，相当于等等）
2

```

**Demo 02**

```java
String str1 = "abc";
String str2 = new String("abc");
String str3 = "abc";
String str4 =  "xxx";
String str5 = "abc" + "xxx";
String str6 = str3 + str4;

System.out.println("str1 == str2：" + (str1 == str2));
System.out.println("str1 == str3：" + (str1 == str3));
System.out.println("str1.equals(str2)：" + (str1.equals(str2)));
System.out.println("str1 == str5：" + (str1 == str5));
System.out.println("str1 == str6：" + (str1 == str6));
System.out.println("str5 == str6：" + (str5 == str6));
System.out.println("str5.equals(str6)：" + (str5.equals(str6)));
System.out.println("str1 == str6.intern()：" + (str1 == str6.intern()));
System.out.println("str1 == str2.intern()：" + (str1 == str2.intern()));

```

打印结果

```bash
str1 == str2：false
str1 == str3：true
str1.equals(str2)：true
str1 == str5：false
str1 == str6：false
str5 == str6：false
str5.equals(str6)：true
str1 == str6.intern()：false
str1 == str2.intern()：true
```



**注**

关于`intern()`方法

`intern()`方法就是从常量池中获取对象，返回字符串对象的规范表现形式；

字符串池最初为空，由类字符串私下维护，调用`intern()`方法时，如果池中已包含由`equals(Object)`方法确定的与此`String`对象相等的字符串，则返回池中的字符串，否则就将该字符串添加到池中，并返回对此字符串对象的引用。

因此，对于任意两个字符串`s`和`t`，`s.intern() == t.intern()`当且仅当`s.equals(t)`为`true`时候，所以文字字符串和字符串值常量表达式都会被插入。