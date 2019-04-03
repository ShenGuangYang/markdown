# 引言

泛型是Java中一个非常重要的知识点，在Java集合类框架中泛型被广泛应用。本文我们将从零开始来看一下Java泛型的设计，将会涉及到通配符处理，以及让人苦恼的类型擦除。

# 1. 泛型基础

## 1.1 泛型类

我们首先定义一个简单的Box类：
```java
public class Box {

    private String object;

    public String getObject() {
        return object;
    }

    public void setObject(String object) {
        this.object = object;
    }
}
```

这是最常见的做法，这样做的一个坏处是Box里面现在只能装入String类型的元素，今后如果我们需要装入Integer等其他类型的元素，还必须要另外重写一个Box，代码得不到复用，使用泛型可以很好的解决这个问题。

```java
public class Box<T> {

    private T t;

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
}
```
这样我们的Box类便可以得到复用，我们可以将T替换成任何我们想要的类型：
```java
Box box1 = new Box<Integer>();
Box box2 = new Box<String>();
Box box3 = new Box<Double>();
```

## 1.2 泛型方法

看完了泛型类，接下来我们来了解一下泛型方法。声明一个泛型方法很简单，只要在返回类型前面加上一个类似的形式就行了：
```java
public class Util {
    public static <K,V> boolean compare(Pair<K,V> p1, Pair<K,V> p2){
        return p1.getKey().equals(p2.getKey()) &&
                p1.getValue().equals(p2.getValue());
    }
}

public class Pair<K,V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() {
        return key;
    }

    public void setKey(K key) {
        this.key = key;
    }

    public V getValue() {
        return value;
    }

    public void setValue(V value) {
        this.value = value;
    }
}
```
我们可以像下面这样去调用泛型方法：
```java
Pair<Integer, String> p1 = new Pair<>(1, "Apple");
Pair<Integer, String> p2 = new Pair<>(2, "Pear");
boolean same = Util.compair(p1, p2);
```

## 1.3 边界符

现在我们要实现这样一个功能，查找一个泛型数组中大于某个特定元素的个数，我们可以这样实现：
```java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray) {
        if (e > elem) { // 报错 Operator '>' cannot be applied to 'T', 'T'
            ++count;
        }

    }
    return count;
}
```
但是这样很明显是错误的，因为除了short, int, double, long, float, byte, char等原始类型，其他的类并不一定能使用操作符>，所以编译器报错，那怎么解决这个问题呢？答案是使用**边界符**。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
做一个类似于下面这样的声明，这样就等于告诉编译器类型参数T代表的都是实现了Comparable接口的类，这样等于告诉编译器它们都至少实现了compareTo方法。
```java
public static <T extends Comparable<T>> int countGreaterThan(T[] array, T elem) {
    int count = 0;
    for (T e : array) {
        if (e.compareTo(elem) > 0)
            ++count;

    }
    return count;
}
```

## 1.4 通配符
