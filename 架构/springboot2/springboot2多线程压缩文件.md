# 多线程压缩文件

利用commons-compress压缩文件

## 添加pom

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.19</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.2</version>
</dependency>
```



## 实现代码

```java
import org.apache.commons.compress.archivers.zip.*;
import org.apache.commons.compress.parallel.InputStreamSupplier;

import java.io.File;
import java.io.IOException;
import java.util.concurrent.ExecutionException;

public class ScatterSample {

    private String rootPath;
    ParallelScatterZipCreator scatterZipCreator = new ParallelScatterZipCreator();

    ScatterZipOutputStream dirs = ScatterZipOutputStream.fileBased(
            File.createTempFile("whatever-preffix", ".whatever"));
 
    public ScatterSample(String rootPath) throws IOException {
        this.rootPath = rootPath;
    }

    public void addEntry(final ZipArchiveEntry zipArchiveEntry, final InputStreamSupplier streamSupplier)
            throws IOException {
        if (zipArchiveEntry.isDirectory() && !zipArchiveEntry.isUnixSymlink()) {
            dirs.addArchiveEntry(ZipArchiveEntryRequest.createZipArchiveEntryRequest(zipArchiveEntry, streamSupplier));
        } else {
            scatterZipCreator.addArchiveEntry(zipArchiveEntry, streamSupplier);
        }
    }

    public void writeTo(final ZipArchiveOutputStream zipArchiveOutputStream)
            throws IOException, ExecutionException, InterruptedException {
        dirs.writeTo(zipArchiveOutputStream);
        dirs.close();
        scatterZipCreator.writeTo(zipArchiveOutputStream);
    }

    public String getRootPath() {
        return rootPath;
    }

    public void setRootPath(String rootPath) {
        this.rootPath = rootPath;
    }

}
```



```java
public class CompressUtil {

    /**
     * 生成压缩文件
     * @param rootPath 压缩文件
     * @param result   压缩完文件
     * @throws Exception
     */
    public static void createZipFile(final String rootPath, final File result) {
        try {
            File dstFolder = new File(result.getParent());
            if (!dstFolder.isDirectory()) {
                dstFolder.mkdirs();
            }
            File rootDir = new File(rootPath);
            final ScatterSample scatterSample = new ScatterSample(rootDir.getAbsolutePath());
            compressCurrentDirectory(rootDir, scatterSample);
            final ZipArchiveOutputStream zipArchiveOutputStream = new ZipArchiveOutputStream(result);
            scatterSample.writeTo(zipArchiveOutputStream);
            zipArchiveOutputStream.close();
        }catch (Exception e) {
            e.printStackTrace();
            throw new BizException("压缩文件出错");
        }
    }

    private static void addEntry(String entryName, File currentFile, ScatterSample scatterSample) throws IOException {
        ZipArchiveEntry archiveEntry = new ZipArchiveEntry(entryName);
        archiveEntry.setMethod(ZipEntry.STORED);
        final InputStreamSupplier supp = () -> {
            try {
                // InputStreamSupplier api says:
                // 返回值：输入流。永远不能为Null,但可以是一个空的流
                return currentFile.isDirectory() ? new NullInputStream(0) : new FileInputStream(currentFile);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
            return null;
        };
        scatterSample.addEntry(archiveEntry, supp);
    }

    private static void compressCurrentDirectory(File dir, ScatterSample scatterSample) throws IOException {
        if (dir == null) {
            throw new IOException("源路径不能为空！");
        }
        String relativePath = "";
        if (dir.isFile()) {
            relativePath = dir.getName();
            addEntry(relativePath, dir, scatterSample);
            return;
        }
        // 空文件夹
        if (dir.listFiles().length == 0) {
            relativePath = dir.getAbsolutePath().replace(scatterSample.getRootPath(), "");
            addEntry(relativePath + File.separator, dir, scatterSample);
            return;
        }
        for (File f : dir.listFiles()) {
            if (f.isDirectory()) {
                compressCurrentDirectory(f, scatterSample);
            } else {
                relativePath = f.getParent().replace(scatterSample.getRootPath(), "");
                addEntry(relativePath + File.separator + f.getName(), f, scatterSample);
            }
        }
    }

}
```







