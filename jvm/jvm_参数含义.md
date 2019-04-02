# 堆内存

## 原理

JVM堆内存分为2块：Permanent Space 和 Heap Space。
Permanent 即 持久代（Permanent Generation），主要存放的是Java类定义信息，与垃圾收集器要收集的Java对象关系不大。
Heap = { Old + NEW = {Eden, from, to} }，Old 即 年老代（Old Generation），New 即 年轻代（Young Generation）。年老代和年轻代的划分对垃圾收集影响比较大。

## 堆内存参数设置

* -Xmx1024m：设置JVM最大堆内存为1024M。
* -Xms1024m：设置JVM初始堆内存为1024M。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

* -XX:+HeapDumpOnOutOfMemoryError：JVM会在遇到OutOfMemoryError时拍摄一个“堆转储快照”，并将其保存在一个文件中。



参考网站: [jvm参数设置](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDJJFI)