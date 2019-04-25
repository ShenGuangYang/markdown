# 系统学习 Java IO (十二)----数据流和对象流

## DataInputStream & DataOutputStream

允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型。
要想使用数据输出流和输入流，必须按指定的格式保存数据，才可以将数据输入流将数据读取进来，所以通常使用 DataInputStream 来读取 DataOutputStream 写入的数据。

DataInputStream 类能够从 InputStream 中读取 Java 基本类型（int，float，long等），而不仅仅是原始字节。 将InputStream包装在 DataInputStream 中，就可以从 DataInputStream 中读取 Java 基本类型。 这就是为什么它被称为 DataInputStream - 因为它读取数据（数字）而不仅仅是字节。

如果需要读取的数据包含大于一个字节的Java 基本类型，则 DataInputStream 非常方便。DataInputStream 希望接受有序多字节类型数据。

### 同时使用 DataInputStream 和 DataOutputStream
如前所述，DataInputStream 类通常与 DataOutputStream 一起使用，首先使用 DataOutputStream 写入数据，然后使用 DataInputStream 再次读取数据。 以下是示例Java代码：

```java
public class DataStream {
    public static void main(String[] args) throws IOException {
        String file = "D:\\test\\output.txt";
        DataOutputStream output = new DataOutputStream(new FileOutputStream(file));
        output.write(1); // 默认是 byte
        output.writeInt(123); // 指定写入 int
        output.writeInt(321);
        output.writeLong(789);
        output.writeFloat(123.45f);
        output.close();

        // 一定要按照写入的顺序和类型读取，否则会出错；
        DataInputStream input = new DataInputStream(new FileInputStream(file));
        byte b = (byte) input.read();
        int i1 = input.readInt();
        int i2 = input.readInt();
        Long l = input.readLong();
        Float f = input.readFloat();
        input.close();

        System.out.println("i1 = " + i1);
        System.out.println("i2 = " + i2);
        System.out.println("b = " + b);
        System.out.println("l = " + l);
        System.out.println("f = " + f);
    }
}
```
注意：一定要按照写入的顺序和类型读取，否则会出错；
其实 DataInputStream 类的实现中，读取方法中只有一个 read() 方法是真正干活，其他的 readXXX() 都是调用 read() 完成任务。看如下代码：

```java
public final byte readByte() throws IOException {
    int ch = in.read();
    if (ch < 0)
        throw new EOFException();
    return (byte)(ch);
}

public final char readChar() throws IOException {
    int ch1 = in.read();
    int ch2 = in.read();
    if ((ch1 | ch2) < 0)
        throw new EOFException();
    return (char)((ch1 << 8) + (ch2 << 0));
}

public final int readInt() throws IOException {
    int ch1 = in.read();
    int ch2 = in.read();
    int ch3 = in.read();
    int ch4 = in.read();
    if ((ch1 | ch2 | ch3 | ch4) < 0)
        throw new EOFException();
    return ((ch1 << 24) + (ch2 << 16) + (ch3 << 8) + (ch4 << 0));
}
```

可以看到，XXX 占 多少个字节，就会在其内部调用多少次 read() 方法。

## ObjectInputStream & ObjectOutputStream

和 DataInputStream 包装成 Java 基本类型类似，ObjectInputStream 类能够从 InputStream 中读取Java对象，而不仅仅是原始字节。 当然，读取的字节必须表示有效的序列化 Java 对象。 通常，使用 ObjectInputStream 来读取 ObjectOutputStream 编写（序列化）的对象。
下面是一个例子：
```java
public class ObjectStream {
    public static void main(String[] args) throws Exception {
        // Serializable 是一个标识接口，实现类只要承诺能被序列化就行了
        class People implements Serializable {
            String name;
            int age;
        }
        File file = new File("D:\\test\\object.data");

        ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream(file));
        People someOne = new People();
        someOne.name = "Json";
        someOne.age = 18;
        output.writeObject(someOne);
        output.close();

        ObjectInputStream input = new ObjectInputStream(new FileInputStream(file));
        People people = (People) input.readObject();
        System.out.println("name = " + people.name + ", age = " + people.age);
        input.close();
    }
}
```
### Close()
使用完数据流后记得关闭它。 关闭 DataInputStream 还将关闭 DataInputStream 正在读取的 InputStream 实例。可以使用 try-with-resources 方式自动关闭。ObjectInputStream 同理。



