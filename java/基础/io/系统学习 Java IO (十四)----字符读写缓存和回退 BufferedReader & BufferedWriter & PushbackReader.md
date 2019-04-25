# 系统学习 Java IO (十四)----字符读写缓存和回退 BufferedReader/BufferedWriter & PushbackReader

## BufferedReader
BufferedReader 类构造器接收一个 Reader 对象，为 Reader 实例提供缓冲。 缓冲可以加快 IO 的速度。 BufferedReader 不是一次从网络或磁盘读取一个字符，而是一次读取一个更大的块。 这通常要快得多，特别是对于磁盘访问和更大的数据量。

类似于 BufferedInputStream ，主要区别在于 BufferedReader 读取字符（文本），而 BufferedInputStream 读取原始字节。

除了向Reader实例添加缓冲外，BufferedReader 的行为与 Reader 非常相似。 BufferedReader 有一个额外的方法，即 readLine() 方法。 如果您需要一次读取一行输入，则此方法很方便。
`String line = bufferedReader.readLine();`

## BufferedWriter

BufferedWriter 类构造器接收一个 Writer 对象，为 Writer 实例提供缓冲。 缓冲可以加快 IO 的速度。 BufferedWriter 不是一次写一个字符到网络或磁盘，而是一次写一个更大的块。 这通常要快得多，特别是对于磁盘访问和更大的数据量。

可以包装 FileReader 的 BufferedReader 。 BufferedReader 将从 FileReader 读取一个字符块（通常为 char 数组）。 因此，从 read() 返回的每个字符都从此内部数组返回。 当数组被读完时，BufferedReader 将一个新的数据块读入数组等。

可以设置 BufferedReader/BufferedWriter 在内部使用的缓冲区大小。默认大小是 8192 的字符数组。

一个简单的使用实例如下：

```java
File file = new File("D:\\test\\1.txt");
BufferedWriter writer = new BufferedWriter(new FileWriter(file));
writer.write("string from BufferedWriter");
writer.close();

int bufferSize = 8 * 1024; // 可选的缓冲字符数组大小
BufferedReader reader = new BufferedReader(new FileReader(file), bufferSize);
int data = reader.read();
while (data != -1) {
    System.out.print((char) data); // string from BufferedWriter
    data = reader.read();
}
reader.close();
```

## PushbackReader
PushbackReader 类旨在从 Reader 解析数据时使用，它可以包装一个 Reader 对象。 PushbackReader 允许将读取的字符推回到 Reader 中下次调用 read() 时，将再次读取这些字符。通俗来讲，PushbackReader 提供了一种可能，让我们能读取流的部分内容而不破坏流。

PushbackReader 的工作方式与 PushbackInputStream 非常相似，只是 PushbackReader 适用于字符，而 PushbackInputStream 适用于字节。所以请参考前面的文章，不再赘述了。下面提供一个简单的例子：
```java
PushbackReader pushbackReader = new PushbackReader(new FileReader("c:\\data\\input.txt"));
int data = pushbackReader.read();
pushbackReader.unread(data);
```

### 设置 PushbackReader 的后推限制
有一个构造函数 `public PushbackReader(Reader in, int size)` 可以设置 PushbackReader 的后推限制，如果不设置这个值，那默认为 1 ，这个值很重要，表示了最多能往回推多少个字符，如果读取了 10 个字符，但是后退限制为 1 的话，那总共只能推回 1 个字符，剩下的 9 个字符没办法推回去，流就被破坏了。
一个使用示例如下：
```java
public class PushbackReaderExample {
    public static void main(String[] args) throws IOException {
        int limit = 2; // 可选，最多只能推回 2 个字符，默认值是 1 
        File file = new File("D:\\test\\1.txt"); // 文件内容是 123456789
        PushbackReader reader = new PushbackReader(new FileReader(file), limit);
        char[] bytes = new char[9]; // 读取 9 个字符；
        reader.read(bytes);
        System.out.println(bytes); // 123456789

        reader.unread(97); // 推回操作都是将内容复制到推回缓冲区的前面
        reader.unread(97); // 97 是字符 'a' 的 int 值，推回 2 个 'a'
        // reader.unread(97); // 会失败并抛出异常，因为最多只能推回2个字符
        reader.read(bytes);
        System.out.println(bytes); // aa3456789
        reader.close();
    }
}
```
### close()
当完成从 PushbackReader 读取字符后，记得关闭它。 关闭 PushbackReade还将关闭 PushbackReader 正在读取的 Reader 实例。

## FilterReader/FilterWriter
FilterReader 是用于实现自己的过滤阅读器的基类。 基本上它只是覆盖了 Reader 中的所有方法。
与 FilterInputStream 一样，我认为这个类没有明智的目的。 我无法看到这个类实际上添加或更改了 Reader 中的任何行为，只是它在构造函数中需要一个 Reader 。 如果想选择扩展此类，则可以直接扩展 Reader 类，并避免层次结构中的额外类。FilterWriter 同样。

