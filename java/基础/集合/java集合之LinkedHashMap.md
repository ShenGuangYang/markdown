# LinkedHashMap

# 简介

LinkedHashMap内部维护了一个双向链表，能保证元素按插入的顺序访问，也能以访问顺序访问，可以用来实现LRU（最近最少使用）缓存策略。

LinkedHashMap可以看成是 LinkedList + HashMap。

# 继承体系

![LinkedHashMap](../../../img/collection/LinkedHashMap.png)



LinkedHashMap继承HashMap，拥有HashMap的所有特性，并且额外增加了按一定顺序访问的特性。





# 存储结构

- [ ] 画图
- [x] 画图1



我们知道HashMap使用（数组 + 单链表 + 红黑树）的存储结构，那LinkedHashMap是怎么存储的呢？

通过上面的继承体系，我们知道它继承了HashMap，所以它的内部也有这三种结构，但是它还额外添加了一种“双向链表”的结构存储所有元素的顺序。

添加删除元素的时候需要同时维护在HashMap中的存储，也要维护在LinkedList中的存储，所以性能上来说会比HashMap稍慢。



# 源码解析

## 属性

```java
// 双向链表头节点 
transient LinkedHashMap.Entry<K,V> head;
// 双向链表尾节点 
transient LinkedHashMap.Entry<K,V> tail;
// 是否按访问顺序排序
final boolean accessOrder;
```

1. head：双向链表的头节点，旧数据存在头节点。
2. tail：双向链表的尾节点，新数据存在尾节点。
3. accessOrder：是否需要按访问顺序排序，如果为false则按插入顺序存储元素，如果是true则按访问顺序存储元素。



## 内部类



```java
// 位于LinkedHashMap中
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
// 位于HashMap中
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

存储节点，继承自HashMap的Node类，next用于单链表存储于桶中，before和after用于双向链表存储所有元素。

## 构造方法

```

```