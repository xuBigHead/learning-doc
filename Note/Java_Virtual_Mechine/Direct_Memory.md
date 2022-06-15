# 传输类型直接内存
## 介绍
直接内存（Direct Memory）不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存区域。直接内存是在Java堆外的、直接向系统申请的内存区间。来源于NIO（New IO/ Non-Blocking IO），通过存在堆中的DirectByteBuffer操作Native内存



| 类型 | 传输类型      | 基于    | 备注                   |
| ---- | ------------- | ------- | ---------------------- |
| IO   | byte[]/char[] | Stream  |                        |
| NIO  | Buffer        | Channel | New IO/Non-Blocking IO |



通常，访问直接内存的速度会优于Java堆，即读写性能高。因此出于性能考虑，读写频繁的场合可能会考虑使用直接内存。Java的NIO库允许Java程序使用直接内存，用于数据缓冲区。

直接内存大小可以通过MaxDirectMemorySize设置，如果不指定，默认与堆的最大值-Xmx参数值一致。

使用下列代码，直接分配本地内存空间
```java
int BUFFER = 1024*1024*1024; // 1GB
ByteBuffer byteBuffer = ByteBuffer.allocateDirect(BUFFER);
```


## 读取文件

### 使用IO读取文件
读写文件，需要与磁盘交互，需要由用户态切换到内核态。在内核态时，需要内存如下图的操作。使用IO，这里需要两份内存存储重复数据，效率低。

![飞书20220609-113246](../../Image/2022/06/220609-1.png)



### 使用NIO读取文件
使用NIO时，如图所示，操作系统划出的直接缓存区可以被java代码直接访问，只有一份。NIO适合对大文件的读写操作。

![飞书20220609-113319](../../Image/2022/06/220609-2.png)

### 代码示例
```java
public class DirectMemoryBufferTest {
    private static final int _100MB = 1024 * 1024 * 100;

    @Test
    public void compareCopyFile() {
        String src = "D:\\Books\\重学Java设计模式.pdf";
        for (int i = 0; i < 1; i++) {
            String dest = src + i;
            calculateSpendTime(this::copyByIO, src,  dest + "io" + i + ".pdf");
            calculateSpendTime(this::copyByDirectBuffer, src, dest + "nio" + i + ".pdf");
        }
    }

    private void calculateSpendTime(BiConsumer<String, String> consumer, String src, String dest) {
        Stopwatch started = Stopwatch.createStarted();
        consumer.accept(src, dest);
        started.stop();
        System.err.println(started.elapsed(TimeUnit.MILLISECONDS));
    }

    private void copyByDirectBuffer(String src, String dest) {
        try (FileChannel inChannel = new FileInputStream(src).getChannel();
             FileChannel outChannel = new FileOutputStream(dest).getChannel()) {
            ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_100MB);
            while (inChannel.read(byteBuffer) != -1) {
                byteBuffer.flip();  // 修改为读数据模式
                outChannel.write(byteBuffer);
                byteBuffer.clear(); // 清空
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void copyByIO(String src, String dest) {
        try (FileInputStream fis = new FileInputStream(src);
             FileOutputStream fos = new FileOutputStream(dest)) {
            byte[] buffer = new byte[_100MB];
            while (true) {
                int len = fis.read(buffer);
                if (len == -1) {
                    break;
                }
                fos.write(buffer, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 存在的问题
- 可能导致OutOfMemoryError异常；
- 由于直接内存在Java堆外，因此它的大小不会直接受限于-Xmx指定的最大堆大小，但是系统内存是有限的，Java堆和直接内存的总和依然受限于操作系统能给出的最大内存；
- 分配回收成本较高，不受JVM内存回收管理。
  
### 代码示例
```java
public class DirectMemoryBufferTest {
    private static final int BUFFER = 1024 * 1024 * 40; // 20MB
    @Test
    public void directMemoryOutOfMemory() {
        List<ByteBuffer> byteBuffers = new ArrayList<>();
        int count = 0;
        try {
            while (true) {
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect(BUFFER);
                byteBuffers.add(byteBuffer);
                count++;
            }
        } finally {
            System.out.println(count);
        }
    }
}
```



