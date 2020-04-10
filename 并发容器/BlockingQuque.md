# BlockingQueue

### Api

![image-20191211114234827](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191211114234827.png)

![image-20191211114254125](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191211114254125.png)

![image-20191211114259606](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191211114259606.png)

```java
public class BlockingQueueDemo {
    public static void main(String[] args) {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3,true);

        System.out.println(queue.poll());//null
        System.out.println(queue.element());//NoSuchElementException
    }
}
```



#### 实现原理

* put方法
* 先加锁，while判断是否已满，满则阻塞
* 未满则将元素加入队列，总元素数量+1，并通知消费者线程

```java
 public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }

 public void put(E e) throws InterruptedException {
         if (e == null)
            throw new NullPointerException();
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            /** Number of elements in the queue */
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }

 private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal();
    }

```

* take()方法
* 加锁，判断队列是否为空，是则阻塞
* 不是则从队列中takeIndes位置获取一个元素。并将此位置设为null，总元素数量-1
* 对takeIndex位置+1，如果和数组长度相等，则置为0
* 通知生产者线程

```java
   public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
  private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }
```

