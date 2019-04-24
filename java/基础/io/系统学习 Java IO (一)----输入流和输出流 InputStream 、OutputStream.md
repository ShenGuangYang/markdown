# 系统学习 Java IO (一)----输入流和输出流 InputStream/OutputStream

## InputStream

是Java IO API中所有输入流的父类。
表示有序的字节流，换句话说，可以将 InputStream 中的数据作为有序的字节序列读取。
这在从文件读取数据或通过网络接收时非常有用。
InputStream 通常连接到某些数据源，如文件，网络连接，管道等
看如下代码片段：

```java
public static void main(String[] args) throws Exception {
    InputStream inputStream = new FileInputStream("D:\\out.txt");
    //do something with data...
    int data = inputStream.read();
    while (data != -1) {
        System.out.print((char) data);
        data = inputStream.read();
    }
    inputStream.close();
}
```

> 注意：为了代码清晰，这里并没有考虑处理异常的情况，IO 异常处理有专门的介绍。

### read()
此方法返回的是 int 值，其中包含**读取的字节的字节值**,可以将返回的 int 强制转换为 char 输出。
如果 read() 方法返回 -1 ，则表示已到达流的末尾，这意味着在 InputStream 中不再有要读取的数据。
也就是说，-1 作为 int 值，而不是 -1 作为 char 或 short，这里有区别！
InputStream 类还包含两个 read() 方法，这些方法可以将 InputStream 源中的数据读入字节数组。
这些方法是：
+ int read(byte[]);
+ int read(byte[], int offset, int length);

一次读取一个字节数比一次读取一个字节要快得多，所以在可以的时候，使用这些读取方法而不是 read() 方法。

read（byte []）方法将尝试将尽可能多的字节读入作为参数给出的字节数组，因为数组具有空间。
该方法返回一个 int ，其值是**实际读取了多少字节**，这点和 read() 方法不一样。
如果可以从 InputStream 读取的字节少于字节数组的空间，则字节数组的其余部分将包含与读取开始之前相同的数据。例如：

```java
InputStream input = new ByteArrayInputStream("123456789".getBytes());
byte[] bytes = new byte[4]; // 每次只读取 4 个字节
int data = input.read(bytes);
while (data != -1) {
      System.out.print(new String(bytes));
      data = input.read(bytes);
}
```
将输出 123456789678 ，而不是预期的 123456789 ！
因为第一次读取进 bytes 是 1234 ，第二次将是 5678 ，现在只剩下 9 一个数字了，注意此时 bytes 的值是 5678 ，然后再读取剩下 1个 9，不能装满 bytes 了，只能覆盖 bytes的第一个字节，最后返回的bytes 是 9678。
所以记住检查返回的 int 以查看实际读入字节数组的字节数。

int read(byte[], int offset, int length);方法和 read（byte []）方法差不多，只是增加了偏移量和指定长度。
和 read() 一样，都是返回 -1 表示数据读取结束。
使用实例如下：
```java
InputStream inputstream = new FileInputStream("D://out.txt");
byte[] data = new byte[1024];
int bytesRead = inputstream.read(data);
while(bytesRead != -1) {
  doSomethingWithData(data, bytesRead);
  bytesRead = inputstream.read(data);
}
inputstream.close();
```
首先，此示例创建一个字节数组。
然后它创建一个名为 bytesRead 的 int 变量来保存每次读取 byte [] 调用时读取的字节数，
并立即分配 bytesRead 从第一次读取 byte [] 调用返回的值。

### mark() and reset()
InputStream 类有两个名为 mark() 和 reset() 的方法，InputStream 的子类可能支持也可能不支持：
1. 该子类覆盖 markSupported() 并返回true，则支持 mark( )和 reset() 方法。
2. 该子类覆盖 markSupported() 并返回 false ，则不支持 mark() 和 reset() 。
3. 该子类不重写 markSupported() 方法 ，则是父类的默认实现 `public boolean markSupported() { return false; }` 也是不支持 mark( )和 reset() 方法

mark() 在 InputStream 内部设置一个标记，默认值在位置 0 处。
可以手动标记到目前为止已读取数据的流中的点，然后，代码可以继续从 InputStream 中读取数据。
如果想要返回到设置标记的流中的点，在 InputStream 上调用 reset() ，然后 InputStream “倒退”并返回标记，
如此，便可再次从该mark点开始返回（读取）数据。很明显这可能会导致一些数据从 InputStream 返回多次。我来举个例子：

```java
public static void testMarkAndReset() throws IOException {
    InputStream input = new ByteArrayInputStream("123456789".getBytes());
    System.out.println("第一次打印：");

    int count = 0;// 计算是第几次读取，将在第二次读取时做标记；
    byte[] bytes = new byte[3]; // 每次只读取 3 个字节
    int data = input.read(bytes);
    while (data != -1) {
        System.out.print(new String(bytes));
        if (++count == 2) { // 在第二轮读取，即读到数字 4 的时候，做标记
            input.mark(16); // 从 mark 点开始再过 readlimit 个字节，mark 将失效
        }
        data = input.read(bytes);
    }

    input.reset();
    System.out.println("\n在经过 mark 和 reset 之后从 mark 位置开始打印：");
    data = input.read(bytes);
    while (data != -1) {
        System.out.print(new String(bytes));
        data = input.read(bytes);
    }
}
```
将会输出：
```
第一次打印：
123456789
在经过 mark 和 reset 之后从 mark 位置开始打印：
789  
```
另外要说明一下 mark(int readlimit) 参数，readlimit 是告诉系统，过了这个 mark 点之后，给本宫记住往后的 readlimit 个字节，因为到时候 reset 之后，要从 mark 点开始读取的；但实际情况和 jdk 文档有出入,很多情况下调用 mark(int readlimit) 方法后，即使读取超过 readlimit 字节的数据，mark 标记仍有效,这又是为什么呢？网上有人解答，但我还是决定亲自探索一番。

我们这个实例引用的实际对象是 `ByteArrayInputStream` 先看一下它的源码：
```java
/* Note: The readAheadLimit for this class has no meaning.*/
public void mark(int readAheadLimit) {
    mark = pos;
}
```

好家伙，它说这个参数对于这个类没有任何作用。
**注意：这段是源码分析可看可不看，跳过不影响阅读**

那我们在看看其他的 InputStream 子类，经验证，FileInputStream 和一些实现类不支持 mark() 方法，我们看看 BufferedInputStream类源码：
我先把一些字段的含义说明一下：

+ `count` 索引1大于缓冲区中最后一个有效字节的索引。 该值始终在0到buf.length的范围内; 元素buf [0]到buf [count-1]包含从底层输入流获得的缓冲输入数据。在 read() 方法中读完数据返回 -1 就是因为if (pos >= count) return -1;
+ `pos` 指缓冲区中的当前位置。 这是要从 buf 数组中读取的下一个字符的索引。
该值始终在 0 到 count 范围内。 如果它小于 count，则 buf [pos] 是要作为输入提供的下一个字节; 如果它等于 count ，则下一个读取或跳过操作将需要从包含的输入流中读取更多字节。（即重新从输入流中取出一段数据缓存）
+ `markpos` 是调用最后一个 mark() 方法时 pos 字段的值。该值始终在-1到pos的范围内。 如果输入流中没有标记位置，则此字段为-1。

BufferedInputStream 是每次读取一定量的数据到 buf 数组中的，设置了 readlimit 肯定是想让数组从 mark 索引开始至少记录到 (mark + readlimit) 索引。

```java
public synchronized void mark(int readlimit) {
    marklimit = readlimit;
    markpos = pos;
}

public synchronized void reset() throws IOException {
    getBufIfOpen(); // Cause exception if closed
    if (markpos < 0)
        throw new IOException("Resetting to invalid mark");
    pos = markpos;
}

private void fill() throws IOException {
    byte[] buffer = getBufIfOpen();
    if (markpos < 0)
        pos = 0;            /* 如果标记点不在缓冲数组里（没标记点），丢掉buffer，取新数据 */
    else if (pos >= buffer.length)  /* 缓冲区中当前位置比buffer数组大，才执行下面代码 */
        if (markpos > 0) {  /* 可以把 markpos 左边的数据丢掉 */
            int sz = pos - markpos; // 需要缓存的字节长度，从 markpos 开始
            System.arraycopy(buffer, markpos, buffer, 0, sz); // 复用内存空间
            pos = sz;
            markpos = 0;
        } else if (buffer.length >= marklimit) { // 如果 buffer 的长度已经大于 marklimit
            markpos = -1;   /* 那 mark 就失效了*/
            pos = 0;        /* 删除buffer内容，取新数据 */
        } else if (buffer.length >= MAX_BUFFER_SIZE) { // 如果buffer过长就抛错
            throw new OutOfMemoryError("Required array size too large");
        } else {            /* buffer 还没 marklimit 大，扩容到 pos 的2倍或者最大值 */
            int nsz = (pos <= MAX_BUFFER_SIZE - pos) ?
                    pos * 2 : MAX_BUFFER_SIZE;
            if (nsz > marklimit)
                nsz = marklimit;
            byte nbuf[] = new byte[nsz];
            System.arraycopy(buffer, 0, nbuf, 0, pos);
            if (!bufUpdater.compareAndSet(this, buffer, nbuf)) {
                throw new IOException("Stream closed");
            }
            buffer = nbuf;
        }
    count = pos;
    int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
    if (n > 0)
        count = n + pos;
}
```
可以得出：设置标记后，
1. 如果缓冲区中当前位置比 buffer 数组小，也就是还没读完 buffer 数组，那 mark 标记不会失效；
2. 下次继续读取，超过 buffer 大小字节后，判断 markpos 是否大于0，如果 markpos 大于0，即还在 buffer 数组内，则把 markpos 左边的数据清除，markpos 指向 0 , 复用内存空间，并设置 buffer 的大小为 (pos - markpos) 的值；
3. 再继续读取，此时 markpos 肯定不在 buffer 数组包含范围了，此时判断 buffer 的长度是否大于等于
marklimit ，如果小于 marklimit ，那说明设置 mark 后读取的数据长度还没达到要求的 marklimit 了，给我继续，保持从 mark 点开始缓存, mark 标记不会失效。然后 buffer 就扩容到`Math.min(2倍 pos 或最大值 ，marklimit)；`
4. 再继续读取，同上，buffer 这么努力扩容，总有大于 marklimit 的时候，这时说明设置 mark 后继续读取的数据长度已经超过要求的 marklimit 了，仁尽义至，标记失效；

## OutputStream
OutputStream 通常始终连接到某个数据目标，如文件，网络连接，管道等。 OutputStream 的目标是将数据写入到外部。

### write(byte)
write(byte) 方法用于将单个字节写入 OutputStream。 OutputStream 的 write() 方法接受一个 int ，其中包含要写入的字节的字节值。 只写入 int 值的第一个字节。 其余的被忽略了。
OutputStream 的子类有重写的 write() 方法。 例如，DataOutputStream 允许使用相应的方法writeBoolean()，writeDouble() 等编写诸如 int，long，float，double，boolean 等 Java 基本类型。

write(byte[] bytes) ， write(byte[] bytes, int offset, int length)
和 InputStream 一样，它们也可以将一个数组或一部分字节写入 OutputStream 。

### flush()
OutputStream 的flush() 方法将写入 OutputStream 的所有数据刷新到底层数据目标。 例如，如果 OutputStream 是 FileOutputStream ，则写入 FileOutputStream 的字节可能尚未完全写入磁盘。 即使您的代码已将其写入 FileOutputStream ，但数据也可能还在某处缓存在内存中。 通过调用 flush() 可以确保将任何缓冲的数据刷新（写入）到磁盘（或网络，或 OutputStream 的目标）。
