# 系统学习 Java IO (七)----字节数组流 ByteArrayInputStream/ByteArrayOutputStream

## ByteArrayInputStream
如果数据存储在数组中，ByteArrayInputStream 可以很方便读取，它可以返回 InputStream ， 这样，ByteArrayInputStream 可以包装字节数组，并将其转换为流。

### 构造器

`public ByteArrayInputStream(@NotNull byte[] buf)` : 创建一个 ByteArrayInputStream ，以便它使用 buf 作为其缓冲区数组。 pos 的初始值为 0 ，count 的初始值为 buf 的长度。

> 注意：关闭 ByteArrayInputStream 无效。此类中的方法在关闭此流后仍可被调用，而不会产生任何 IOException。

## ByteArrayOutputStream
创建一个新的字节数组输出流。 缓冲区容量最初为32字节，但必要时其大小会增加。

### 构造器
- `OutputStream output = new ByteArrayOutputStream();` 创建一个32字节（默认大小）的缓冲区。
- `ByteArrayOutputStream(int size)` 创建一个新的 byte 数组输出流，它具有指定大小的缓冲区容量（以字节为单位）。

可以将数据写入 ByteArrayOutputStream ，完成后，调用 ByteArrayOutputStream 的方法 toByteArray() 以获取字节数组中的所有写入数据。
`byte[] bytes = output.toByteArray();`

> 同样，关闭 ByteArrayOutputStream 无效。此类中的方法在关闭此流后仍可被调用，而不会产生任何 IOException 。

如果有一个程序需要将其数据输出到 OutputStream ，但我们需要其返回字节数组的情况下，ByteArrayOutputStream 可以很方便完成任务。

很明显 ByteArrayOutputStream 是用来缓存数据的，向它的内部自动增长的缓冲区写入数据，再通过 output.toByteArray()可以从中提取数据。

一个例子：

```java
public class ByteArray {
    public static void main(String[] args) throws IOException {
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        output.write("Hello ByteArray".getBytes());
        // 只需调用方法 toByteArray()，所有写入的数据都以数组形式返回。
        byte[] bytes = output.toByteArray();

        InputStream input = new ByteArrayInputStream(bytes);
        try {
            int data = input.read();
            while (data != -1) {
                System.out.print((char) data);
                data = input.read();
            }
            System.out.println();
        } catch (
                IOException e) {
            e.printStackTrace();
        }
    }
}
```

