# 系统学习 Java IO (十)----回退流 PushbackInputStream

PushbackInputStream 旨在从 InputStream 解析数据时使用。 有时您需要先读取几个字节以查看将要发生的事情，然后才能确定如何解释当前字节， PushbackInputStream 允许这样做。 实际上，它允许将读取的字节推回到流中，这样就像流没有被动过，下次调用 read() 时，将再次重新读取。通俗来讲，就像男人对女人(Stream）说：我只看看，不动手。

## 构造器

- PushbackInputStream(InputStream in)：通过输入流 in 创建 PushbackInputStream 
- PushbackInputStream(InputStream in, int size)：使用指定 size 创建 , size 代表推回缓冲区的大小。

这是一个简单的PushbackInputStream示例：

```java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"));
int data = input.read();
input.unread(data);
```
对 read() 的调用就像从 InputStream 中读取一个字节。 对 unread() 的调用将一个字节推回到PushbackInputStream 中。 下次调用 read() 时，将首先读取推回的字节。 如果将多个字节被推回,则推回的最新字节将首先从 read() 返回，就像在堆栈上一样。

### 字段摘要
流本身不支持回退功能， PushBackInputStream 内部维护了一个 byte 数组来实现推回操作的。

`protected byte[] buf` 推回缓冲区。

`protected int pos ` 推回缓冲区中的位置，将读取该位置的下一个字节。

## 设置 PushbackInputStream 的后推限制
可以在 PushbackInputStream 的构造函数中设置应该能够读取的字节数。 以下是如何通过 PushbackInputStream 构造函数设置回退限制：

```java
int pushbackLimit = 8;
PushbackInputStream input = new PushbackInputStream(
        new FileInputStream("c:\\data\\input.txt"),
        pushbackLimit);
```
此示例设置 8 字节的内部缓冲区。 这意味着您可以一次读取最多 8 个字节，然后推回去。

## 方法摘要

| 方法 	| 说明 	|
|:------------------------------------:	|:-------------------------------------------------------------------------:	|
| int read(byte[] b, int off, int len) 	| 从此输入流将最多 len 个数据字节读入 byte 数组。 	|
| void unread(byte[] b) 	| 推回一个 byte 数组：将其复制到推回缓冲区之前。 	|
| void unread(int b) 	| 推回一个字节：将其复制到推回缓冲区之前。 	|
| void mark(int readlimit) 	| 标记当前位置。和 ByteInputStream 一样，此类的 mark() 方法不执行任何操作。 	|
| void reset() 	| 此类的 reset() 方法不执行任何操作，只抛出 IOException。调用就异常。 	|

