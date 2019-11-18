# Lock接口

1. jdk1.5后新增
2. 显示获取释放锁
3. 简单使用

```java
public class LockTest {
    public static void main(String[] args) {
        //获取锁的过程不要放在try块中，防止获取锁时抛出异常无故释放锁
        Lock lock = new ReentrantLock();
        lock.lock();
        try {
        } finally {
            lock.unlock();
        }
    }
}
```

4. 相关api

   ![1574047111161](C:\Users\czd\AppData\Roaming\Typora\typora-user-images\1574047111161.png)

### 与synchronized的区别

![1574047083891](C:\Users\czd\AppData\Roaming\Typora\typora-user-images\1574047083891.png)

# 队列同步器

