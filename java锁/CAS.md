### CAS原理

调用Unsafe类的方法

```java
 @HotSpotIntrinsicCandidate
	//o是当前对象，offset是地址偏移量  o+offset等于当前要修改变量的地址   delta是要update的值
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }
```



### CAS操作的三大问题

1. ABA问题

   ![1573392050480](D:\笔记\面试题\java锁\assets\1573392050480.png)

   ```java
   public class AtomicDemo {
       private static AtomicStampedReference<Integer> atomicStampedReference =new AtomicStampedReference<>(1,1);
   
       public static void main(String[] args) {
           boolean result = atomicStampedReference.compareAndSet(1, 2, 1, 2);
           System.out.println(result);
           System.out.println(atomicStampedReference.getReference());
       }
   }
   ```

   

2. 循环时间开销大

   ![1573392215732](D:\笔记\面试题\java锁\assets\1573392215732.png)

3. 只能保证一个共享变量的操作

![1573392242773](D:\笔记\面试题\java锁\assets\1573392242773.png)

![image-20191220111727066](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191220111727066.png)