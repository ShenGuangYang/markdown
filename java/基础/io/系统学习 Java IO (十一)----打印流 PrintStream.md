# 系统学习 Java IO (十一)----打印流 PrintStream

PrintStream 类可以将格式化数据写入底层 OutputStream 或者直接写入 File 对象。 PrintStream 类可以格式化基本类型，如int，long等格式化为文本，而不是其字节值。 这就是为什么它被称为 PrintStream ，因为它将原始字节格式化为文本 - 就像它们在打印到屏幕（或打印到纸张）时看起来一样。

## 构造器
| 方法 	| 描述 	|
|:-----------------------------------------------------------------:	|:-------------------------------------------------------------------------------------------------------------------------------------:	|
| PrintStream(File file) 	| 创建具有指定文件且不带自动行刷新的新打印流。 	|
| PrintStream(File file, String csn) 	| 创建具有指定文件名称和 csn 字符集且不带自动行刷新的新打印流。 	|
| PrintStream(String fileName) 	| 创建具有指定文件名称且不带自动行刷新的新打印流。 	|
| PrintStream(String fileName, String csn) 	| 创建指定名称和字符集且不带自动行刷新的打印流 	|
| PrintStream(OutputStream out) 	| 创建新的打印流。out - 将向其打印值和对象的输出流 	|
| PrintStream(OutputStream out, boolean autoFlush) 	| autoFlush - boolean 变量；如果为 true，则每当写入 byte 数组、调用其中一个 println 方法或写入换行符或字节 ( '\n') 时都会刷新输出缓冲区 	|
| PrintStream(OutputStream out, boolean autoFlush, String encoding) 	| 同上，同时指定字符集 	|

## 常见方法

| 方法 	| 描述 	|
|:--------------------------------------------------------:	|:------------------------------------------------------:	|
| void print(Xxx x) 	| 可以打印指定类型的数据 	|
| PrintStream append(char c) 	| 将指定字符添加到此输出流。 	|
| PrintStream append(CharSequence csq, int start, int end) 	| 添加指定字符序列，后两个参数可选 	|
| PrintStream format(String format, Object... args) 	| 使用指定格式字符串和参数将格式化字符串写入此输出流中。 	|
| println(Xxx x) 	| 打印完，然后换行。 	|
| void write(int b) 	| 将指定的字节写入此流。 	|

PrintStream 方便的提供了重载的 print() 方法。
看一个例子：

```java
PrintStream printStream = new PrintStream(new FileOutputStream("D:\\test\\1.txt"));
printStream.print(true);
printStream.print(" print ");
printStream.print((int)123);
printStream.print('&');
printStream.print((float) 123.456);
printStream.close();
```

输出结果为 `true print 123&123.456`

## System.out 和 System.err 就是 PrintStreams
我们很熟悉这两个众所周知的 PrintStream 实例：System.out 和 System.err，所以我们早就在使用 PrintStream 了。

PrintStream 类包含强大的 format() 和 printf() 方法（它们完全相同，但 C 程序员更熟悉名称“printf”）。 这些方法允许使用格式化字符串以非常高级的方式混合文本和数据。如 `System.out.format("Text + data: %d", 123);` 其他高级用法可以查 JavaDoc 。

## close()
用完要关闭，关闭 PrintStream 系统会同时关闭对应的包装流，如上例，不需要单独关闭 FileOutputStream 。建议使用 try-with-resources 。