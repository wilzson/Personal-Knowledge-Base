## 文件编程

```java
public static void main(String[] args) throws FileNotFoundException {
        try (FileChannel channel = new FileInputStream("data.txt").getChannel();) {
            // 准备缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(10);
            // 从 channel 读取数据, 向 buffer 写入
            while (true) {
                int len = channel.read(buffer);
                if (len == -1)
                    break;
                // 打印 buffer 内容
                buffer.flip(); // 调用filp切换为读模式
                while (buffer.hasRemaining()) { // 是否还有剩余未读数据
                    byte b = buffer.get();
                    System.out.println((char) b);
                }
                // 切换成写模式
                buffer.clear();
            }
        } catch (Exception e) {

        }

    }
```



class java.nio.HeapByteBuffer  - java 堆内存

class java.nio.DirectByteBuffer  - 直接内存，不会受到 GC 影响。