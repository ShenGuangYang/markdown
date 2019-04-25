# 系统学习 Java IO (十三)----字符读写 Reader/Writer 及其常用子类

## Reader
Reader 类是 Java IO API 中所有 Reader 子类的基类。 Reader 类似于 InputStream ，除了它是基于字符而不是基于字节的。 换句话说， Reader 用于读取文本，而 InputStream 用于读取原始字节。

## Writer
Writer 类是 Java IO API 中所有 Writer 子类的基类。 Writer 就像一个 OutputStream ，除了它是基于字符而不是基于字节的。 换句话说，Writer 用于写入文本，而 OutputStream 用于写入原始字节。
Writer 通常连接到某些数据目标，如文件，字符数组，网络套接字等。

## Unicode中的字符
许多应用程序使用 UTF（UTF-8或UTF-16）来存储文本数据。 可能需要一个或多个字节来表示 UTF-8 中的单个字符。 在 UTF-16 中，每个字符需要 2 个字节来表示。 因此，在读取或写入文本数据时，数据中的单个字节可能与 UTF 中的一个字符不对应。 如果只是通过 InputStream 一次读取或写入 UTF-8 数据的一个字节，并尝试将每个字节转换为字符，可能不会得到正确的文本。

Reader 类能够将字节解码为字符。 只需要告诉Reader 要解码的字符集。 这是在实例化 Reader 时执行的（当实例化其中一个子类时）。 通常会直接使用 Reader 子类而不是 Reader。Writer 同理。

## 读取

部分方法如下:
| 方法 	| 描述 	|
|:------------------------------------------------:	|:--------------------------------:	|
| void mark(int readAheadLimit) 	| 标记流中的当前位置。 	|
| int read() 	| 读取单个字符。 	|
| int read(char[] cbuf) 	| 将字符读入数组。 	|
| abstract int read(char[] cbuf, int off, int len) 	| 将字符读入数组的某一部分。 	|
| int read(CharBuffer target) 	| 试图将字符读入指定的字符缓冲区。 	|
| boolean ready() 	| 判断是否准备读取此流。 	|

具体的使用需要参考对应的子类。

和 InputStream 类似，如果 read() 方法返回 -1 ，则 Reader 中没有更多数据要读取，并且可以关闭它。-1 作为 int 值，而不是 -1 作为 byte 或 char 值。

## 将字节流包装成字符流 InputStreamReader/OutputStreamWrite

### InputStreamReade
InputStreamReade 类用于包装 InputStream ，从而将基于字节的输入流转换为基于字符的 Reader 。 换句话说，InputStreamReader 将 InputStream 的字节解释为文本而不是数字数据，是字节流通向字符流的桥梁。
为了达到最高效率，可要考虑在 BufferedReader 内包装 InputStreamReader。例如：
```java
BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
```
通常用于从文件（或网络连接）中读取字符，其中字节表示文本。 例如，一个文本文件，其中字符编码为 UTF-8 。 可以使用InputStreamReader 来包装 FileInputStream 以读取此类文件。
一个示例如下：

```java
InputStream inputStream = new FileInputStream("D:\\test\\1.txt");
Reader inputStreamReader = new InputStreamReader(inputStream);

int data = inputStreamReader.read();
while (data != -1) {
    char c = (char) data;
    System.out.print(c);
    data = inputStreamReader.read();
}
inputStreamReader.close();
```
首先创建一个 FileInputStream ，然后将其包装在 InputStreamReader 中。 其次，该示例通过 InputStreamReader 从文件中读取所有字符.
注意：为清楚起见，此处已跳过正确的异常处理。 要了解有关正确异常处理的更多信息，请转至目录的 Java IO 异常处理。

#### 指定字符集
底层 InputStream 中的字符将使用某些字符编码进行编码。 此字符编码称为字符集，Charset。 两种常用字符集是 ASCII 和 UTF8（在某些情况下为UTF-16）。

默认字符集可能因为环境不同而不同，所以建议告诉 InputStreamReader 实例 InputStream 中的字符用什么字符集进行编码。 这可以在 InputStreamReader 构造函数中指定，可以只提供字符集的名字字符串，在底层会调用 Charset.forName("UTF-8") 进行转换的。 以下是设置 InputStreamReader 使用的字符集的示例：
```java
InputStream inputStream = new FileInputStream("D:\\test\\1.txt");
Reader inputStreamReader = new InputStreamReader(inputStream, "UTF-8");
```
#### 关闭 InputStreamReader
同样建议使用 try with resources 来关闭流。同样，只要关闭最外层的包装流，里面的流会被系统自动关闭。

```java
try(InputStreamReader inputStreamReader =
    new InputStreamReader(input)){
    int data = inputStreamReader.read();
    while(data != -) {
        System.out.print((char) data));
        data = inputStreamReader.read();
    } 
}
```

### OutputStreamWriter
OutputStreamWriter 类用于包装 OutputStream ，从而将基于字节的输出流转换为基于字符的 Writer 。

如果需要将字符写入文件，OutputStreamWriter 非常有用，例如编码为 UTF-8 或 UTF-16。 然后可以将字符（char 值）写入 OutputStreamWriter ，它将正确编码它们并将编码的字节写入底层的 OutputStream 。
一个简单的示例如下：
```java
OutputStream outputStream = new FileOutputStream("D:\\test\\1.txt");
Writer outputStreamWriter = new OutputStreamWriter(outputStream);
outputStreamWriter.write("Hello OutputStreamWriter");
outputStreamWriter.close();
```
注意：为清楚起见，此处已跳过正确的异常处理。 要了解有关正确异常处理的更多信息，请转至目录的 Java IO 异常处理。

同 InputStreamReader，OutputStreamWriter 也可以使用指定的字符集输出，如：

```java
Writer outputStreamWriter = new OutputStreamWriter(outputStream, "UTF-8");
```
#### 关闭 OutputStreamReader
参考关闭 InputStreamReader 或 Java IO 异常处理。


## 读写文件 FileReader/FileWriter

### FileReader
FileReader类使得可以将文件的内容作为字符流读取。 它的工作方式与 FileInputStream 非常相似，只是 FileInputStream 读取字节，而 FileReader 读取字符。 换句话说，FileReader 旨在读取文本。 取决于字符编码方案，一个字符可以对应于一个或多个字节。

FileReader 假定您要使用运行应用程序的计算机的默认字符编码来解码文件中的字节。 这可能并不总是你想要的，但是不能改变它！
如果要指定其他字符解码方案，请不要使用 FileReader 。 而是在 FileInputStream 上使用 InputStreamReader 。 InputStreamReader 允许您指定在读取基础文件中的字节时使用的字符编码方案。

### FileWriter
FileWriter 类可以将字符写入文件。 在这方面它的工作原理与 FileOutputStream 非常相似，只是 FileOutputStream 是基于字节的，而 FileWriter 是基于字符的。 换句话说，FileWriter 用于写文本。 一个字符可以对应于一个或多个字节，这取决于使用的字符编码方案。
创建 FileWriter 时，可以决定是否要覆盖具有相同名称的任何现有文件，或者只是要追加内容到现有文件。 可以通过选择使用的 FileWriter 构造函数来决定。

- FileWriter(File file, boolean append): 根据给定的 File 对象构造一个 FileWriter 对象。 示例如下：
```java
Writer fileWriter = new FileWriter("c:\\data\\output.txt", true);  // 追加模式
Writer fileWriter = new FileWriter("c:\\data\\output.txt", false); // 默认情况，直接覆盖原文件
```
**注意，只要成功 new 了一个 FileWriter 对象，没有指定是追加模式的话，那不管有没有调用 write() 方法，都会清空文件内容。**

下面是一个读和写的例子：

```java
public class FileRW {
    public static void main(String[] args) throws IOException {
        // 默认是覆盖模式
        File file = new File("D:\\test\\1.txt");
        Writer writer1 = new FileWriter(file);
        writer1.write("string from writer1, ");
        writer1.close();

        Writer writer2 = new FileWriter(file, true);
        writer2.write("append content from writer2");
        writer2.close();


        Reader reader = new FileReader(file);
        int data = reader.read();
        while (data != -1) {
            // 将会输出 string from writer1, append content from writer2
            System.out.print((char) data);
            data = reader.read();
        }
        reader.close();
    }
}
```
注意：为清楚起见，此处已跳过正确的异常处理。 要了解有关正确异常处理的更多信息，请转至Java IO异常处理。

其他行为和 InputStreamReader 差不多，就不展开讲了。

## PipedReader/PipedWriter

### 读写管道 PipedReader 和 PipedWriter
PipedReader 类使得可以将管道的内容作为字符流读取。 因此它的工作方式与 PipedInputStream 非常相似，只是PipedInputStream 是基于字节的，而不是基于字符的。 换句话说，PipedReader 旨在读取文本。PipedWriter 同理。

#### 构造器

| 方法 	| 描述 	|
|:------------------------------------------:	|:-----------------------------------------------------------------------------------:	|
| PipedReader() 	| 创建尚未连接的 PipedReader。 	|
| PipedReader(int pipeSize) 	| 创建一个尚未连接的 PipedReader，并对管道缓冲区使用指定的管道大小。 	|
| PipedReader(PipedWriter src) 	| 创建直接连接到传送 PipedWriter src 的 PipedReader。 	|
| PipedReader(PipedWriter src, int pipeSize) 	| 创建一个 PipedReader，使其连接到管道 writer src，并对管道缓冲区使用指定的管道大小。 	|
| PipedWriter() 	| 创建一个尚未连接到传送 reader 的传送 writer。 	|
| PipedWriter(PipedReader snk) 	| 创建传送 writer，使其连接到指定的传送 reader。 	|

读写之前，必须先建立连接
PipedReader 必须连接到 PipedWriter 才可以读 ，PipedWriter 也必须始终连接到 PipedReader 才可以写。就是说读写之前，必须先建立连接，有两种方式可以建立连接。

1. 通过构造器创建，伪代码如 `Piped piped1 = new Piped(piped2);`
2. 调用其中一个的 connect() 方法，伪代码如 `Piped1.connect(Piped2);`

并且通常，PipedReader 和 PipedWriter 由不同的线程使用。 注意只有一个 PipedReader 可以连接到同一个 PipedWriter ，一个示例如下：

```java
PipedWriter writer = new PipedWriter();
PipedReader reader = new PipedReader(writer);

writer.write("string form pipedwriter");
writer.close();

int data = reader.read();
while (data != -1) {
    System.out.print((char) data); // string form pipedwriter
    data = reader.read();
}
reader.close();
```
注意：为清楚起见，这里忽略了正确的 IO 异常处理，并且没有使用不同线程，不同线程操作请参考 PipedInputStream 。

正如在上面的示例中所看到的，PipedReader 需要连接到 PipedWriter 。 当这两个字符流连接时，它们形成一个管道。 要了解有关 Java IO 管道的更多信息，请参考 管道流 PipedInputStream 部分。

## 读写字符数组 CharArrayReader/CharArrayWriter
ByteArrayInputStream/ByteArrayOutputStream 是对字节数组处理，CharArrayReader/CharArrayWriter 则是对字符数组进行处理，其用法是基本一致的，所以这里略微带过。

### CharArrayReader
CharArrayReader 类可以将 char 数组的内容作为字符流读取。
只需将 char 数组包装在 CharArrayReader 中就可以很方便的生成一个 Reader 对象。

### CharArrayWriter
CharArrayWriter 类可以通过 Writer 方法（CharArrayWriter是Writer的子类）编写字符，并将写入的字符转换为 char 数组。
在写入所有字符时，CharArrayWriter 上调用 toCharArray() 能很方便的生成一个字符数组。

#### 两个类的构造函数：

| 方法 	| 描述 	|
|:---------------------------------------------------:	|:--------------------------------------------------:	|
| CharArrayReader(char[] buf) 	| 根据指定的 char 数组创建一个 CharArrayReader 	|
| CharArrayReader(char[] buf, int offset, int length) 	| 根据指定的 char 数组创建一个 CharArrayReader 	|
| CharArrayWriter() 	| 创建一个新的 CharArrayWriter ，默认缓冲区大小为 32 	|
| CharArrayWriter(int initialSize) 	| 创建一个具有指定初始缓冲区大小的新 CharArrayWriter 	|

注意：设置初始大小不会阻止 CharArrayWriter 存储比初始大小更多的字符。 如果写入的字符数超过了初始 char 数组的大小，则会创建一个新的更大的 char 数组，并将所有字符复制到新数组中。

一个使用实例如下：
```java
CharArrayWriter writer = new CharArrayWriter();
writer.append('H');
writer.write("ello ".toCharArray());
writer.write("World");
char[] chars = writer.toCharArray();
writer.close();

CharArrayReader reader = new CharArrayReader(chars);
int data = reader.read();
while (data != -1) {
    System.out.print((char) data); // Hello World
    data = reader.read();
}
reader.close();
```
注意：为清楚起见，此处已跳过正确的异常处理。 要了解有关正确异常处理的更多信息，请转至 Java IO 异常处理。
