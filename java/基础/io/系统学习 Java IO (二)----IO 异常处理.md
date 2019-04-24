# 系统学习 Java IO (二)----IO 异常处理
我们使用流后，需要正确关闭 Streams 和 Readers / Writers 。
这是通过调用 close() 方法完成的,看看下面这段代码：
```java
InputStream input = new FileInputStream("D:\out.txt");

int data = input.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = input.read();
}
input.close();
```

这段代码乍一看似乎没问题。
但是如果从 doSomethingWithData() 方法内部抛出异常会发生什么？
对！`input.close();` 将得不到执行的机会，InputStream 永远不会关闭！
为避免这种情况，可以把关闭流的代码放在 finally 块里面确保一定被执行，将代码重写为：

```java
InputStream input = null;
try {
    input = new FileInputStream("D:\\out.txt");
    int data = input.read();
    while (data != -1) {
        //do something with data...
        doSomethingWithData(data);
        data = input.read();
    }
} catch (IOException e) {
    //do something with e... log
} finally {
    if (input != null) input.close();
}
```
但是如果 close() 方法本身抛出异常会发生什么？ 这样流会被关闭吗？
好吧，要捕获这种情况，你必须在 try-catch 块中包含对 close() 的调用，如下所示：
```java
finally {
    try {
        if (input != null) input.close();
    } catch (IOException e) {
        //do something, or ignore.
    }
}
```
这样确实可以解决问题，就是太啰嗦太难看了。
有一种方法可以解决这个问题。就是把这些重复代码抽出来定义一个方法，这些解决方案称为“异常处理模板”。
创建一个异常处理模板，在使用后正确关闭流。此模板只编写一次，并在整个代码中重复使用，但感觉还是挺麻烦的,就不展开讲了。
还好，从 Java 7 开始，我们可以使用名为 try-with-resources 的构造，

```java
try (FileInputStream input = new FileInputStream("file.txt")) {
    int data = input.read();
    while (data != -1) {
        System.out.print((char) data);
        data = input.read();
    }
}
```
当try块完成时，FileInputStream 将自动关闭，换句话说，只要线程已经执行出try代码块，inputstream 就会被关闭。
因为 FileInputStream 实现了 Java 接口 java.lang.AutoCloseable ，
实现此接口的所有类都可以在 try-with-resources 构造中使用。
```java
public interface AutoCloseable {
    void close() throws Exception;
}
```
FileInputStream 重写了这个 close() 方法，查看源码可以看到其底层是调用 native 方法进行操作的。
try-with-resources构造不仅适用于Java的内置类。
还可以在自己的类中实现 java.lang.AutoCloseable 接口，
并将它们与 try-with-resources 构造一起使用，如：
```java
public class Main {
    public static void main(String[] args) throws Exception {
        try (MyAutoClosable myAutoClosable = new MyAutoClosable()) {
            int a = 3 / 0;
            myAutoClosable.doIt();
        }
    }
}

class MyAutoClosable implements AutoCloseable {
    public void doIt() {
        System.out.println("MyAutoClosable doing it!");
    }
    @Override
    public void close() throws Exception {
        System.out.println("MyAutoClosable closed!");
    }
}
```
try-with-resources 是一种非常强大的方法，可以确保 try-catch 块中使用的资源正确关闭，
无论这些资源是您自己的创建，还是 Java 的内置组件。
