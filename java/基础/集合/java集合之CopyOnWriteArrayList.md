# CopyOnWriteArrayList

# 简介

CopyOnWriteArrayList 是 ArrayList 的线程安全版本，内部也是通过数组实现，每次对数组的修改都完全拷贝一份新的数组来修改，修改完了再替换掉老数组，这样保证了只阻塞写操作，不阻塞读操作，实现了读写分离。



# 继承体系

![CopyOnWriteArrayList](../../../img/collection/CopyOnWriteArrayList.png)

- CopyOnWriteArrayList 实现了List, RandomAccess, Cloneable,Serializable等接口。
- CopyOnWriteArrayList 实现了List，提供了基础的添加、删除、遍历等操作。
- CopyOnWriteArrayList 实现了RandomAccess，提供了随机访问的能力。
- CopyOnWriteArrayList 实现了Cloneable，可以被克隆。
- CopyOnWriteArrayList 实现了Serializable，可以被序列化。

# 源码解析

## 属性

```java
// 用于写操作是加锁
final transient ReentrantLock lock = new ReentrantLock();
// 真正存储元素的地方，只能通过getArray()/setArray() 访问
private transient volatile Object[] array;
```

1. lock：用于写操作加锁，使用transient修饰表示不自动序列化
2. array：真正存储元素的地方，使用transient修饰表示不自动序列化，使用volatile修饰表示一个线程对这个字段的修改，其它线程可见



## `CopyOnWriteArrayList()` 构造方法

创建空数组

```java
public CopyOnWriteArrayList() {
    // 所有操作对array的操作都是通过setArray()/getArray() 进行
    setArray(new Object[0]);
}
final void setArray(Object[] a) {
    array = a;
}
```



## `CopyOnWriteArrayList(Collection c)` 构造方法

如果c是CopyOnWriteArrayList类型，直接把它的数组赋值给当前list的数组，注意这里是浅拷贝，两个集合共用同一个数组。

如果c不是CopyOnWriteArrayList类型，则进行拷贝把c的元素全部拷贝到当前list的数组中。

```java
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class)
        // 如果c也是CopyOnWriteArrayList类型
        // 那么直接把它的数组拿过来使用
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else {
        // 否则调用其toArray()方法将集合元素转化为数组
        elements = c.toArray();
		// 这里c.toArray()返回的不一定是Object[]类型
        if (elements.getClass() != Object[].class)
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    setArray(elements);
}
```



## `CopyOnWriteArrayList(E[] toCopyIn)` 构造方法

把toCopyIn的元素拷贝给当前list的数组。

```java
public CopyOnWriteArrayList(E[] toCopyIn) {
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```

## `add(E e)` 方法

添加一个元素到末尾。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 将旧数组元素拷贝到新数组中
        // 将旧数组元素拷贝到新数组中
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 将新添加的元素放在最后一位
        newElements[len] = e;
        // 更新旧数组元素
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

>代码逻辑：
>
>1. 加锁
>2. 获取旧数组
>3. 新增一个数组，长度为旧数组长度+1，并把旧数组元素拷贝到新数组
>4. 把新添加的元素放到数组的最后一位
>5. 更新旧数组
>6. 释放锁



## `add(int index, E element)` 方法

```java
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 检查越界
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+ ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            // 如果插入的位置是最后一位
            // 那么拷贝一个n+1的数组, 其前n个元素与旧数组一致
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            // 如果插入的位置不是最后一位
            // 那么新建一个n+1的数组
            newElements = new Object[len + 1];
            // 拷贝旧数组前index的元素到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index及其之后的元素往后挪一位拷贝到新数组中
            // 这样正好index位置是空出来的
            System.arraycopy(elements, index, newElements, index + 1, numMoved);
        }
        newElements[index] = element;
        // 更新旧数组元素
        setArray(newElements);
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

> 代码逻辑：
>
> 1. 加锁
> 2. 检查索引是否合法
> 3. 如果索引等于数组长度，那就拷贝一个len+1的数组
> 4. 如果索引不等于数组长度，那就新建一个len+1的数组，并按索引位置分成两部分，索引之前（不包含）的部分拷贝到新数组索引之前（不包含）的部分，索引之后（包含）的位置拷贝到新数组索引之后（不包含）的位置，索引所在位置留空；
> 5. 把索引位置赋值为待添加的元素；
> 6. 把新数组赋值给当前对象的array属性，覆盖原数组；
> 7. 释放锁





