# 系统学习 Java IO (六)----管道流 PipedInputStream/PipedOutputStream

PipedInputStream 类使得可以作为字节流读取管道的内容。 管道是同一 JVM 内的线程之间的通信通道。
使用两个已连接的管道流时，要为每个流操作创建一个线程，read() 和 write() 都是阻塞方法，如果一个线程同时读写就会造成死锁

看一个例子：
```java
public class Pipe {
    public static void main(String[] args) throws IOException {
        final PipedOutputStream output = new PipedOutputStream();
        final PipedInputStream input = new PipedInputStream(output);

        // 写线程，创建匿名 Runnable 对象
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    output.write("Hello Pipe".getBytes());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
        // 读线程,用一下 Lambda 表达式创建匿名 Runnable 对象
        Thread thread2 = new Thread(() -> {
            try {
                int data = input.read();
                while (data != -1) {
                    System.out.print((char) data);
                    data = input.read();
                }
                System.out.println();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        );
        thread1.start();
        thread2.start();
    }
}

```
这里通过利用构造方法来直接指定管道输入流的管道输出流。
`PipedInputStream input = new PipedInputStream(output);`

也可以使用 pipe1.connect(pipe2) 来连接两个管道流，例如：
```java
PipedInputStream pis = new PipedInputStream(); 
pis.connect(pos);
```
除了管道之外，还有许多其他方法可以在同一个 JVM 中进行通信。
事实上,线程更经常交换完整的对象而不是原始的字节数据。
但是如果需要在线程之间交换原始字节数据，Java IO 的管道是能做到的。


