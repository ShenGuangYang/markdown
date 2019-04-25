# 系统学习 Java IO (十五)----字符读写 Reader/Writer 其他子类

## 跟踪行号的缓冲字符输入流 LineNumberReader

LineNumberReader 类是一个 BufferedReader ，用于跟踪读取字符的行号。行号从 0 开始。每当 LineNumberReader 在包装的 Reader 返回的字符中遇到行终止符时，行号递增。

可以通过调用 getLineNumber() 方法获取当前行号，也可以通过调用 setLineNumber() 方法设置当前行号。

**注意设置的行号不会改变实际的文件指针，仅仅是改变了内部的一个 lineNumber 变量，这样getLineNumber() 方法获取的也是 lineNumber 变量的值，不要妄想通过这个 setLineNumber() 方法随机访问文本行。
**
一个示例如下：
```java
public class LineNumberReaderExample {
    public static void main(String[] args) throws IOException {
        /*文件内容是：
        first line
        second line
        third line
         */
        LineNumberReader reader = new LineNumberReader(new FileReader("D:\\test\\input.txt"));
        System.out.println("line= " + reader.getLineNumber() + ", context= " + reader.readLine());
        reader.setLineNumber(666); // 这个方法只是指示了当前行号，并不会改读取结果
        System.out.println("line= " + reader.getLineNumber() + ", context= " + reader.readLine());
    }
}
```
输出的结果是：
```
line= 0, context= first line
line= 666, context= second line
```
如果要解析可能包含错误的文本文件，则此类很方便。 向用户报告错误时，如果指出错误在第几行，则更容易纠正错误。

完成从 LineNumberReader 读取字符时，记得关闭它，LineNumberReader 还将关闭 LineNumberReader 正在读取的 Reader 实例。调用 close() 方法可以关闭 LineNumberReader 。

## 标记字符输入流 StreamTokenizer

在解析文件或计算机语言时，在进一步处理它们之前将输入分解为标记是正常的。 此过程也称为“词法(lexing)”或“ 符号化(tokenizing)”。
使用 StreamTokenizer 类，可以浏览基础 Reader 中的标记。 通过在循环内调用 StreamTokenizer 的 nextToken() 方法来实现此目的。 每次调用 nextToken() 之后，StreamTokenizer 都有几个字段，可以阅读这些字段以查看读取的令牌类型和值等。这些字段是：

`ttype` token type ,读取的令牌类型（字，数字，行尾）
`sval` string val ,令牌的字符串值，如果令牌是字符串（word）
`nval` number val ,令牌的数字值，如果令牌是数字。

这是一个简单的 StreamTokenizer 示例：
```java
StreamTokenizer streamTokenizer = new StreamTokenizer(new StringReader("I had 1 little cat.."));
while (streamTokenizer.nextToken() != StreamTokenizer.TT_EOF) {
    switch (streamTokenizer.ttype) {
        case StreamTokenizer.TT_WORD:
            System.out.println(streamTokenizer.sval);
            break;
        case StreamTokenizer.TT_NUMBER:
            System.out.println((int) streamTokenizer.nval);
            break;
        case StreamTokenizer.TT_EOL: // end of the line
            System.out.println();
    }
}
```

StreamTokenizer 能够识别标识符，数字，带引号的字符串和各种注释样式。 还可以指定要将哪些字符解释为空格，注释开始，结束等。在开始解析其内容之前，所有这些都在 StreamTokenizer 上配置。 有关更多信息，请参阅 JavaDoc。

当从 StreamTokenizer 读完令牌后，记得关闭它。 关闭 StreamTokenizer 还将关闭 StreamTokenizer 正在读取的 Reader 实例。

## 打印字符流 PrintWriter
PrintWriter 类可以将格式化数据写入基础 Writer 。 例如，将 int，long 和其他基本类型数据写为格式化为文本，而不是作为其字节值。
如果要生成必须混合文本和数字的报表（或类似报表），PrintWriter 非常有用。 除了写入原始字节的方法之外，PrintWriter 类具有与 PrintStream 相同的所有方法。 作为 Writer 子类，PrintWriter 旨在编写文本。
一个示例如下：

```java
public class PrintWriterExample {
    public static void main(String[] args) throws IOException {
        PrintWriter printWriter = new PrintWriter(new FileWriter("D:\\test\\input.txt"));

        printWriter.print(true);
        printWriter.print((int) 123);
        printWriter.print((float) 123.456);
        printWriter.printf("Text + data: %d", 123);

        printWriter.close();
    }
}
```
最后 input.txt 文件内容是“true123123.456Text + data: 123"

### 构造器
PrintWriter 具有多种结构选择器，可以将其连接到 File，OutputStream 或 Writer 。 以这种方式，PrintWriter 与其他 Writer 子类有点不同，后者往往需要将其他 Writer实例作为参数的构造函数（除了少数，如 OutputStreamWriter ）。


|方法|描述|
|-|-|
|PrintWriter(File file, String csn) | 创建具有指定文件和字符集且不带自动刷行新的新 PrintWriter。|
|PrintWriter(OutputStream out, boolean autoFlush) | 通过现有的 OutputStream 创建新的可指定自动刷新的 PrintWriter。|
|PrintWriter(String fileName, String csn) |创建具有指定文件名称和字符集且不带自动行刷新的新 PrintWriter。|
|PrintWriter(Writer out, boolean autoFlush) |创建新的可指定自动刷新的 PrintWriter。|

此外，PrintWriter 有 重载的 append()方法，print() 方法，printf()/format()方法，println()方法和 writer()方法。

### close()

完成将字符写入 PrintWriter 后，记得关闭它。 关闭 PrintWriter 还将关闭 PrintWriter 正在写入的 Writer 实例。

## 字符串读写 StringReader/StringWriter
StringReader 类可以将普通的 String 转换为 Reader 。
StringWriter 类能够以 String 形式将字符写入 Writer 。

写入 StringWriter 的字符可以通过两个方法 toString() 和 getBuffer() 获得，两个方法返回的内容一致。
区别在于方法 toString() 返回 String 类型。方法 getBuffer() 返回 StringBuffer 类型。

一个简单的示例如下：
```java
public class StringRW {
    public static void main(String[] args) throws IOException {
        StringWriter stringWriter = new StringWriter();
        stringWriter.write('H');
        stringWriter.append('e');
        stringWriter.write("llo");
        stringWriter.write("World".toCharArray());

        String data = stringWriter.toString(); // HelloWorld
        StringBuffer dataBuffer = stringWriter.getBuffer();
        System.out.println(data.equals(dataBuffer.toString())); // true
        stringWriter.close();

        StringReader stringReader = new StringReader(data);
        int d = stringReader.read();
        while (d != -1) {
            System.out.print((char) d); // HelloWorld
            d = stringReader.read();
        }
        stringReader.close();
    }
}
```
