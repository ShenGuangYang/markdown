# 系统学习 Java IO (十六)----这么多类，应该用哪个?
Java IO目的和功能
Java IO 包含 InputStream，OutputStream，Reader 和 Writer 类的许多子类。 原因是，所有这些子类都在解决各种不同的目的。 所涉及的目的总结如下：
- 网络访问
- 内部缓冲区访问
- 线程间通信（管道）
- 缓冲
- 过滤
- 解析
- 读写文本（Reader/Writer）
- 读写基本类型数据（long，int等）
- 读写对象

## Java IO类概述表
在讨论了 Java IO 类所针对的源，目标，输入，输出和各种 IO 目的之后，这里列出了大多数（不是全部）Java IO 类除以输入，输出，基于字节或基于字符的任何目的，以及任何他们可能正在解决的更具体的目的，如缓冲，解析等。

| 数据类型 	| 基于字节的 Input 	| 基于字节的 Output 	| 基于字符的 Input 	| 基于字符的 Output 	|
|:-------------:	|:------------------------------------:	|:----------------------------------:	|:--------------------------------:	|:--------------------------:	|
| 基础 	| InputStream 	| OutputStream 	| Reader 、 InputStreamReader 	| Writer、OutputStreamWriter 	|
| 数组 	| ByteArrayInputStream 	| ByteArrayOutputStream 	| CharArrayReader 	| CharArrayWriter 	|
| Files 	| FileInputStream、RandomAccessFile 	| FileOutputStream、RandomAccessFile 	| FileReader 	| FileWriter 	|
| 管道 	| PipedInputStream 	| PipedOutputStream 	| PipedReader 	| PipedWriter 	|
| 缓冲 	| BufferedInputStream 	| BufferedOutputStream 	| BufferedReader 	| BufferedWriter 	|
| 过滤 	| FilterInputStream 	| FilterOutputStream 	| FilterReader 	| FilterWriter 	|
| 解析 	| PushbackInputStream、StreamTokenizer 	|  	| PushbackReader、LineNumberReader 	|  	|
| 字符串 	|  	|  	| StringReader 	| StringWriter 	|
| 数据 	| DataInputStream 	| DataOutputStream 	|  	|  	|
| 数据 - 格式化 	|  	| PrintStream 	|  	| PrintWriter 	|
| 对象 	| ObjectInputStream 	| ObjectOutputStream 	|  	|  	|