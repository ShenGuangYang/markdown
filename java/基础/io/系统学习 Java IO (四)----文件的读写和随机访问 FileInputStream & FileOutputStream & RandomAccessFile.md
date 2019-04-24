# 系统学习 Java IO (四)----文件的读写和随机访问 FileInputStream &FileOutputStream & RandomAccessFile

## 文件输入流 FileInputStream

这是一个简单的FileInputStream示例：
```java
InputStream input = new FileInputStream("D:\\input.txt");
int data = input.read();
while(data != -1) {
  //do something with data...
  doSomethingWithData(data);
  data = input.read();
}
input.close();
```

> 注意：为了代码清晰，这里并没有考虑处理异常的情况，IO 异常处理有专门的介绍。

### FileInputStream 构造器
FileInputStream 类有三个不同的构造函数，可用于创建 FileInputStream 实例。

- 构造函数将一个包含文件系统中要读取的文件所在的路径 String 作为参数：`FileInputStream fileInputStream = new FileInputStream( "D:\\1.txt");`
注意路径需要双反斜杠`\\`，因为反斜杠是Java字符串中的转义字符，通过此方式获得单个反斜杠。
在unix上，文件路径可能如下所示： `String path = "/home/czwbig/data/thefile.txt";` 
注意使用正斜杠`/`作为目录分隔符。 这是在 unix 上编写文件路径的方法。 实际上，Java 也会理解在 Windows 上使用`/`作为目录分隔符,例如`new FileInputStream("D:/out.txt")`
- 构造函数将 File 对象作为参数。 File 对象必须指向要读取的文件。 这是一个例子：
```java
String path = "D:\\out.txt";
File   file = new File(path);
FileInputStream fileInputStream = new FileInputStream(file);
```
应该使用哪个构造函数取决于在打开 FileInputStream 之前具有该路径的形式。 如果您已经有一个 String 或 File ，只需按原样使用它。 将 String 转换为 File 或将 File 转换为 String 没有特别的好处。

- public FileInputStream(FileDescriptor fdObj) 通过使用文件描述符 fdObj 创建一个 FileInputStream，该文件描述符表示到文件系统中某个实际文件的现有连接。不常用。

### read(byte[])
作为 InputStream 的子类，FileInputStream 还有两个 read() 方法，可以将数据读入字节数组。 可以在我的有关 InputStream 的文章中阅读，不展开了。

### close()
建议使用 try 自动关闭，参考 IO 异常处理章节。

## 文件输出流 FileOutputStream

这是一个简单的 FileOutputStream 示例：
```java
OutputStream output = new FileOutputStream("D:\\out.txt");
while(moreData) {
  int data = getMoreData();
  output.write(data);
}
output.close();
```

### FileOutputStream 构造器
和 FileInputStream 的 3 个构造器差不多，参考上面即可。
另外多了两个构造方法：
- FileOutputStream(File file, boolean append) ;
- FileOutputStream(String name, boolean append) ;
  参数 `append` ：如果不给出 append 参数，其值默认是 false 的，即默认是覆盖模式。如果为 true，则将字节写入文件末尾处，而不是写入文件开始处，这样就能不覆盖文件，而是追加内容。
所以，在新建 FileOutputStream 对象的时候，要非常小心，只要没有指定 append 为 true ，那么**只要 FileOutputStream 对象创建成功，对应的文件会被立即清空。**
如代码：`OutputStream outputStream = new FileOutputStream("D:\\test\\1.txt"); `，执行后对应的 1.txt 文件(如果存在)会立刻被清空，而不管有没有调用 OutputStream 的 write() 方法。

### write(...) and flush()
参考 OutputStream 。

## 使用 RandomAccessFile 随机访问文件
这里的随机访问是指，指定任何一个位置，都能够访问它；而不是不确定的随机访问某一个位置。
在使用 RandomAccessFile 类之前，必须实例化它。它有两个构造器，如下：
1. RandomAccessFile(File file, String mode)
2. RandomAccessFile(String name, String mode)

实例：
```java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
```
参数：
file、name- 该文件对象
mode - 访问模式，如下表：
| 字段 	| 描述 	|
|:------------------------------------------:	|:------------------------------------------------------------:	|
| static String pathSeparator 	| 与系统有关的路径分隔符，为了方便，它被表示为一个字符串。 	|
| static char pathSeparatorChar 	| 同上值的字符表示,UNIX系统为 '/' ,Windows 系统为 '\\'。 	|
| static String separator 	| 与系统有关的默认名称分隔符，Unix系统是 ':' Windows系统是 ';' 	|
| public static final char pathSeparatorChar 	| 同上值的字符表示 	|

### "rwd" 模式
可用于减少执行的 I/O 操作数量.使用 "rwd" 仅要求更新要写入存储的文件的内容；使用 "rws" 要求更新要写入的文件内容及其元数据，这通常要求至少一个以上的低级别 I/O 操作。

### "rws" 和 "rwd" 模式
如果该文件位于本地存储设备上，那么当返回此类的一个方法的调用时，可以保证由该调用对此文件所做的所有更改均被写入该设备。这对确保在系统崩溃时不会丢失重要信息特别有用。如果该文件不在本地设备上，则无法提供这样的保证。

### 在文件中跳转
要在 RandomAccessFile 中的特定位置读取或写入，必须首先将文件指针放在要读取或写入的位置。 这是使用 seek() 方法完成的。 可以通过调用 getFilePointer() 方法获取文件指针的当前位置。
read() 方法将文件指针递增为指向刚刚读取的字节后文件中的下一个字节！ 这意味着可以继续调用 read() 而无需手动移动文件指针。
看如下例子：
```java
public class RandomAccessFileExample {
    public static void main(String[] args) throws IOException {
        // out.txt 此时的文件内容为 "123456789"
        RandomAccessFile file = new RandomAccessFile("D:\\out.txt", "rw");

        System.out.println("pointer: " + file.getFilePointer()); // 输出 pointer: 0
        System.out.println("char: " + (char) file.read()); // 输出 char: 1
        System.out.println("pointer: " + file.getFilePointer()); // 输出 pointer: 1

        file.seek(4); // 下标从 0 开始的，让其指向第 5 个字节

        System.out.println("pointer: " + file.getFilePointer()); // 输出 pointer: 4
        System.out.println("char: " + (char) file.read()); // 输出 char: 5
        System.out.println("pointer: " + file.getFilePointer()); // 输出 pointer: 5
        file.close();
    }
}
```

### read & write
从 RandomAccessFile 读取是使用其众多 read() 方法之一完成的。

| 方法 	| 描述 	|
|:-------------------------:	|:----------------------------------------------------------------:	|
| read(byte[] b) 	| 将最多 b.length 个数据字节从此文件读入 byte 数组。 	|
| readByte() 	| 从此文件读取一个有符号的八位值。 	|
| readChar() 	| 从此文件读取一个字符。 	|
| readFully(byte[] b) 	| 将 b.length 个字节从此文件读入 byte 数组，并从当前文件指针开始。 	|
| readLine() 	| 从此文件读取文本的下一行。 	|
| skipBytes(int n) 	| 尝试跳过输入的 n 个字节以丢弃跳过的字节。 	|
| setLength(long newLength) 	| 设置此文件的长度。 	|
| writeChars(String s) 	| 按字符序列将一个字符串写入该文件。 	|