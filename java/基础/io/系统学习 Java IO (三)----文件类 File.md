# 系统学习 Java IO (三)----文件类 File

Java IO API 中的 File 类可以访问基础文件系统。 使用 File 类，可以：
+ 检查文件或目录是否存在。
+ 如果目录不存在，创建一个目录。
+ 读取文件的长度。
+ 重命名或移动文件。
+ 删除文件。
+ 检查路径是文件还是目录。
+ 读取目录中的文件列表。

> 注意：“File”仅允许访问文件和文件系统元数据。 如果需要读取或写入文件内容，则应使用 FileInputStream，FileOutputStream 或 RandomAccessFile 执行此操作。

File 类一共有 4 个字段，可以提供平台无关的一些字符：

| 字段 	| 描述 	|
|:------------------------------------------:	|:------------------------------------------------------------:	|
| static String pathSeparator 	| 与系统有关的路径分隔符，为了方便，它被表示为一个字符串。 	|
| static char pathSeparatorChar 	| 同上值的字符表示,UNIX系统为 '/' ,Windows 系统为 '\\'。 	|
| static String separator 	| 与系统有关的默认名称分隔符，Unix系统是 ':' Windows系统是 ';' 	|
| public static final char pathSeparatorChar 	| 同上值的字符表示 	|

构造器和一些方法示例如下：

```java
public class FileExample {
    public static void main(String[] args) throws IOException, URISyntaxException {
        // File 一共有 4 种构造器，如下：
        File parentDir = new File("D://test"); // 通过目录路径字符串创建
        File file1 = new File("D://test//file1.txt"); // 通过文件路径字符串创建
        File file2 = new File(parentDir, "childDir"); // 通过指定父目录方式创建
        File uri = new File(new URI("file:/D:/test/uri")); // 通过 URI 对象创建

        /*
        File 对象既可用来创建目录，也可以创建文件，不会受到路径名影响，
        比如可以创建名为 file1.txt 的目录,创建名为 childDir 的文件
         */
        parentDir.mkdirs(); // 创建 test 目录
        file1.mkdirs(); // 创建 file1.txt 目录
        file2.createNewFile(); // 创建 childDir 文件
        file2.mkdirs(); // file2 已创建了 childDir 文件，同一目录下再创建 childDir 目录会失败

        uri.mkdirs();
        System.out.println(uri.exists());

        System.out.println(file1.getName()); // file1.txt
        System.out.println(file1.exists()); // true
        System.out.println(file1.isFile()); // false
        System.out.println(file1.isAbsolute()); // true
        System.out.println(file1.getAbsolutePath()); // 返回绝对路径，D:\test\file1.txt
        System.out.println(file1.getAbsoluteFile()); // 等同于 new File(this.getAbsolutePath())
        System.out.println(file1.toURI()); // file:/D:/test/file1.txt/

        // 列出系统的根目录，C:\ D:\
        File[] files = File.listRoots();
        for (int i = 0; i < files.length; i++) {
            System.out.println(files[i]);
        }

        /* 返回 parentDir 目录下所有的文件
            D:\test\childDir
            D:\test\file1.txt
            D:\test\ uri
         */
        File[] fs = parentDir.listFiles();
        for (int i = 0; i < fs.length; i++) {
            System.out.println(fs[i]);
        }

        // 立即删除此抽象路径名表示的文件或目录。如果是目录，则该目录必须为空才能删除。
        file1.delete();
        // 将删除的命令缓存，JVM 终止时才真正删除
        parentDir.deleteOnExit();
    }
}
```
注意：`boolean mkdirs()` 方法创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。注意，此操作失败时也可能已经成功地创建了一部分必需的父目录。如果目录已存在，返回 false 。