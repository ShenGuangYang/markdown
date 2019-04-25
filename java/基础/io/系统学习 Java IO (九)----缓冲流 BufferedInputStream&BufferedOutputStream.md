# 系统学习 Java IO (九)----缓冲流 BufferedInputStream/BufferedOutputStream

## BufferedInputStream

BufferedInputStream 类为输入流提供缓冲。 缓冲可以加快IO的速度。 BufferedInputStream 不是一次从网络或磁盘读取一个字节，而是一次将更大的块读入内部缓冲区。 当从 BufferedInputStream 读取一个字节时，您正在从其内部缓冲区中读取它。 当缓冲区被完全读取时，BufferedInputStream 将另一个更大的数据块读入缓冲区。 这通常比从 InputStream 一次读取单个字节快得多，特别是对于磁盘访问和更大的数据量。

### 构造器
- BufferedInputStream(InputStream in) ： 创建一个 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。
- BufferedInputStream(InputStream in, int size) ： 创建具有指定缓冲区大小的 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。

例子：
```java
InputStream input1 = new BufferedInputStream(new FileInputStream("D:\\test.txt"));

int bufferSize = 8 * 1024;
InputStream input2 = new BufferedInputStream(new FileInputStream("D:\\test.txt"), bufferSize);
```
最好使用 1024 字节倍数的缓冲区大小，最适合硬盘中的大多数内置缓冲等。
除了为输入流添加缓冲外，BufferedInputStream 的行为与 InputStream 完全相同,也支持 mark() 和 reset(); 具体请参考 InputStream ，不赘述了；

### BufferedInputStream 的最佳缓冲区大小
应该使用不同的缓冲区大小进行一些实验，以找出哪些缓冲区大小似乎可以在你的具体硬件上提供最佳性能。 最佳缓冲区大小可能取决于是否将 BufferedInputStream 与磁盘或网络 InputStream 一起使用。

对于磁盘和网络流，最佳缓冲区大小也可能取决于计算机中的具体硬件。 如果硬盘一次至少读取 4KB，那么使用少于 4KB 的缓冲区是愚蠢的。 然后最好使用 4KB 倍数的缓冲区大小。 例如，使用 6KB 也是愚蠢的。

即使你的磁盘读取例如块 一次 4KB ，使用大于此的缓冲区仍然是个好主意。 磁盘擅长顺序读取数据 - 这意味着它擅长读取位于彼此之后的多个块。 因此，使用带有 BufferedInputStream 的 16KB 缓冲区或 64KB 缓冲区（甚至更大）仍然可以提供比仅使用 4KB 缓冲区更好的性能。

## BufferedOutputStream

BufferedOutputStream 类为输出流提供缓冲。 缓冲可以加快 IO 的速度。 您不是一次向网络或磁盘写入一个字节，而是一次写入一个更大的块。 这通常要快得多，特别是对于磁盘访问和更大的数据量。

### 构造器
参考 BufferedInputStream

和 BufferedInputStream 差不多，除了为输入流添加缓冲外，BufferedOutputStream 的行为与 OutputStream 完全相同。 唯一的区别是，如果您需要绝对确定到目前为止写入的数据是从缓冲区刷出并进入网络或磁盘，则可能需要调用 `flush()`方法。

### BufferedOutputStream的最佳缓冲区大小

参考 BufferedInputStream ；
