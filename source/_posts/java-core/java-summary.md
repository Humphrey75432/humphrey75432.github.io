---
title: Java面试汇总
date: 2022-02-24 15:57:25
updated: 2022-02-24 15:57:25
tags: Java
categories: 面试技巧
---

# 一、Java基础

## 1. JDK和JRE有什么区别？

+ JDK：Java Development Kit的简称，提供Java开发环境和运行环境；
+ JRE：Java Runtime Environment的简称，Java运行环境；

简单来说：JDK包含JRE，想要开发Java程序需要安装JDK；想要运行Java运行只需要JRE；



## 2. == 和 equals的区别

### 2.1 基本区别

+ 基本类型：比较值是否相同
+ 引用类型：比较引用是否相同

equals本质上也是 ==，只不过String和Integer重写了equals方法，所以变成了值比较；

如果要保证两个对象相等就要重写它的equals方法



### 2.2 equals和hashCode的区别

+ 两个对象的hashCode()相同，equals()不一定相同；
+ 两个对象equals()相同，hashCode()一定相同；



## 3. final在java类中的作用

+ final修饰的类不能被继承；
+ final修饰的方法不能被重写；
+ final修饰的量叫常量，常量必须初始化，初始化后的值不能被修改；



## 4. Java的基础类型

有8种，分别是：byte, boolean, char, short, float, long和double

String不属于Java的基本类型，属于对象；



## 5. Java操作字符串的类

有：String，StringBuilder和StringBuffer。其中StringBuffer是线程安全的，而StringBuilder是非线程安全的。但是StringBuilder的性能高于StringBuffer。所以单线程环境下使用StringBuilder，多线程环境下使用StringBuffer；



## 6. String str = "i"和String str = new String("i")区别

+ String str = "i"会在JVM虚拟机中分配到常量池

+ String str = new String("i")会被分配到堆内存中（并且会产生冗余对象）；



## 7. String类的常用方法

+ indexOf()：返回指定字符串的索引
+ charAt()：返回指定索引处的字符
+ replace()：字符串替换；
+ trim()：去除字符串两端的空白
+ split()：分割字符串，返回一个分割后的字符串数组
+ getBytes()：返回字符串的byte类型数组
+ length()：返回字符串长度
+ toLowerCase()：字符串转小写
+ toUpperCase()：字符串转大写
+ substring()：截取字符串
+ equals()：字符串比较



## 8. 抽象类必须要有抽象方法吗？

不需要，抽象类不一定非要有抽象方法

```java
public abstract class Cat {
    public static void sayHi() {
        System.out.println("hi~");
    }
}
```

+ 普通类不能包括抽象方法，抽象类可以包括抽象方法；
+ 抽象类不能直接实例化，普通类可以直接实例化；
+ 抽象类不能被final修饰（因为final 修饰的类不能被继承）



## 9. 接口和抽象类的区别

实现：抽象子类使用`extends`继承，接口必须使用`implements`实现；

构造函数：抽象类有构造函数，接口没有（当然1.8中接口可以有default方法）

实现数量：类可以实现很多个接口，但是只能继承一个抽象类；

访问修饰符：接口的方法默认使用public修饰，抽象类无此限制；



## 10. Java I/O流

+ 功能分类：输入流、输出流
+ 类型分类：字节流（8位）和字符流（16位）



## 11. BIO、NIO、AIO区别

+ BIO：Block IO同步阻塞式IO，平常使用的传统IO，特点是简单方便使用，并发能力低；
+ NIO：同步非阻塞IO，客户端和服务端通过Channel（通道）通讯，实现了多路复用；
+ AIO：异步非阻塞IO，异步IO的操作基于事件和回调机制；



## 12. File常用方法

+ exists()：检测文件路径是否存在；
+ createFile()：创建文件；
+ createDirectory()：创建目录；
+ delete()：删除一个文件或者目录
+ copy()：复制文件
+ move()：移动文件
+ size()：查看文件个数
+ read()：读取文件
+ write()：写入文件



# 二、容器

## 13. java容器分类

![image-20220224165841246](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220224165841246.png)

## 14. HashMap和Hashtable的区别

+ 存储：HashMap允许key和value为null，而Hashtable不允许；
+ 线程安全：Hashtable是线程安全的，而HashMap是非线程安全的；
+ 推荐使用：Hashtable是保留类不建议使用，单线程环境用HashMap，多线程用ConcurrentHashMap；



## 15. 如何决定使用HashMap还是TreeMap

+ 想要对一个key集合进行有序遍历，使用TreeMap；
+ 只做插入、删除、定位一个元素的操作，使用HashMap；



## 16. HashMap/HashSet的实现原理

开个专题学习



## 17. ArrayList和LinkedList的区别

+ **数据结构：**ArrayList是动态数组，LinkedList是双链表结构；

+ **随机访问：**ArrayList是数组结构，所以随机访问效率高；LinkedList是线性数据存储，所以需要移动指针从前往后遍历；

+ **增加和删除效率：**非首尾增加删除操作，LinkedList效率比ArrayList高；

综述：需要频繁读取集合中的元素，推荐使用ArrayList，插入删除操作比较多就用LinkedList；



## 18. 数组和List的转换

+ 数组转List：`Arrays.asList(array)`；
+ List转数组：List自带的toArray()方法；

```java
List<String> list = new ArrayList<>();
list.add("sdv");
list.add("drgd");
Object[] objects = list.toArray();
System.out.println(Arrays.toString(objects));

String[] array = new String[]{"ssdv", "svsdfb"};
System.out.println(Arrays.asList(array));
```



## 19. ArrayList和Vector的区别

+ 线程安全：Vector使用Synchronized实现同步，是线程安全的；而ArrayList是非线程安全的；
+ 性能：ArrayList性能优于Vector；
+ 扩容：两者都会根据实际需要动态扩容，但是Vector扩容每次增加1倍，ArrayList只会增加50%；



## 20. Queue中的poll()和remove()的区别

+ 相同：返回第一个元素，并在队列中删除返回的对象；
+ 不同：poll()没元素返回null，remove()会直接抛出NoSuchElementException异常；



## 21. 哪些集合类是线程安全的

Vector、Hashtable、Stack是线程安全的，HashMap是非线程安全的，对应的线程安全类是ConcurrentHashMap;



## 22. 迭代器iterator是什么

提供遍历任何Collection的接口，可以从一个Collection中使用迭代器方法获取迭代器实例。

```java
List<String> list = new ArrayList<>();
Iterator<String> it = List.iterator();
while(it.hasNext()) {
    String obj = it.next();
    System.out.println(obj);
}
```

Iterator特点是安全，因为可以确保在当前遍历集合元素被更改时，抛出ConcurrentModificationException异常；



## 23. 怎么确保一个集合不能被更改

```java
List<String> list = new ArrayList<>();
list.add("x");
Collection<String> clist = Collections.unmodifiableCollection(list);
clist.add("y"); // throw error
System.out.println(list.size());
```

