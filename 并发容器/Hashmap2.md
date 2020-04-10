 

# 解决hashMap线程不安全的方法

```java
 private static ConcurrentHashMap<String,String> map=new  ConcurrentHashMap<>();

 private static Map<String,String> map=Collections.synchronizedMap(new HashMap<>());

```

# hashMap1.7 源码

#### 0.存在的问题

1. 并发中扩容死锁
2. String作为key冲突容易让人利用，引发DOS
3. 链表过长查询效率降低

#### 1.初始化

此时table还是空的table。并没有进行初始化

```java
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    init();
}
```



#### 2.put方法

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
final int hash(Object k) {
    int h = hashSeed;
    //为了防止String类型的key冲突过于严重，被人利用
    //这里用了hashSeed随机数加入hash，并用其他的hash算法
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

//扩容时将旧的元素移到新的table上
 void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

#### 3. 解决String的hash碰撞

```java
final int hash(Object k) {
    int h = hashSeed;
    //为了防止String类型的key冲突过于严重，被人利用
    //这里用了hashSeed随机数加入hash，并用其他的hash算法
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

# hashMap1.8中的改进

#### 0.改进的内容

![image-20191202181501316](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191202181501316.png)

#### 1.put以及hash

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    //尾插法，后来的节点在链表尾部
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

//将hashCode高16位与低16位异或，把高位也加入运算，减少hashCode低位相同情况下的hash冲突
  static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

#### 2. 扩容

```java
 /*  hash=1011//或者是1000      01
   length=10
       newlength=1000
       index=hash&(length-1)//1
       
       newindex=hash&(newlength-1)//11
       
       对于newindex来说，要么跟index一样(当hash值高一位为0时)，要么是比原来的index前面多了一个1(多一个1即代表着多了length)
       
       原来的index=1   length=2
       扩容后
       newindex=3         newlength=4   
       或者
       newindex=1         newlength=4
       
       index=1的叫lohead
       index=3的叫hihead
       
                    01&10==0    11&10==1      
       可以通过e.hash & oldlength来判断高一位的hash值是0还是1
       由此可以得到  (e.hash & oldlength==0)就是lohead  
       否则就是hihead
      */ 
       final Node<K,V>[] resize() {
       Node<K,V>[] oldTab = table;
       int oldCap = (oldTab == null) ? 0 : oldTab.length;
       int oldThr = threshold;
       int newCap, newThr = 0;
       if (oldCap > 0) {
           if (oldCap >= MAXIMUM_CAPACITY) {
               threshold = Integer.MAX_VALUE;
               return oldTab;
           }
           else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
               newThr = oldThr << 1; // double threshold
       }
       else if (oldThr > 0) // initial capacity was placed in threshold
           newCap = oldThr;
       else {               // zero initial threshold signifies using defaults
           newCap = DEFAULT_INITIAL_CAPACITY;
           newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
       }
       if (newThr == 0) {
           float ft = (float)newCap * loadFactor;
           newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                     (int)ft : Integer.MAX_VALUE);
       }
       threshold = newThr;
       @SuppressWarnings({"rawtypes","unchecked"})
       Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
       table = newTab;
       if (oldTab != null) {
           for (int j = 0; j < oldCap; ++j) {
               Node<K,V> e;
               if ((e = oldTab[j]) != null) {
                   oldTab[j] = null;
                   if (e.next == null)
                       newTab[e.hash & (newCap - 1)] = e;
                   else if (e instanceof TreeNode)
                       ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                   else { // preserve order
                       Node<K,V> loHead = null, loTail = null;
                       Node<K,V> hiHead = null, hiTail = null;
                       Node<K,V> next;
                       do {
                           next = e.next;
                        
                           if ((e.hash & oldCap) == 0) {
                               if (loTail == null)
                                   loHead = e;
                               else
                                   loTail.next = e;
                               loTail = e;
                           }
                           else {
                               if (hiTail == null)
                                   hiHead = e;
                               else
                                   hiTail.next = e;
                               hiTail = e;
                           }
                       } while ((e = next) != null);
                       if (loTail != null) {
                           loTail.next = null;
                           newTab[j] = loHead;
                       }
                       if (hiTail != null) {
                           hiTail.next = null;
                           newTab[j + oldCap] = hiHead;
                       }
                   }
               }
           }
       }
       return newTab;
   }
```

# 面试题

### (1)HashMap的实现原理?

- **你看过HashMap源码嘛，知道原理嘛?为什么用数组+链表?**

   HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体。 只是在JDK1.8中，链表长度大于8的时候，链表会转成红黑树！ *为什么用数组+链表？* 数组是用来确定桶的位置，利用元素的key的hash值对数组长度取模得到. 链表是用来解决hash冲突问题，当出现hash值一样的情形，就在数组上的对应位置形成一条链表。 

  

- **hash冲突你还知道哪些解决办法？**

  (1)开放定址法

  (2)链地址法

  (3)再哈希法

  (4)公共溢出区域法 

- **我用LinkedList代替数组结构可以么?**

  虽然可以，但是存取效率低。至于ArrayList，扩容是按1.5倍(size+size>>1)扩容的，不适合。


### (2)HashMap在什么条件下扩容?

* **HashMap在什么条件下扩容?**

如果需要往原来新增一个元素，新增后(超过load factor*current capacity)，就要resize。 load factor为0.75，为了最大程度避免哈希冲突 current capacity为当前数组大小。

如果新增的元素index所对应的数组元素为null，则不需要resize

* **那为什么是2的n次方呢？**

为了使&运算等于取模运算  hash%(length)==hash&(length-1)

* **hash算法为什么要高16位去异或低16位**

 减少hash冲突。

```java
//将hashCode高16位与低16位异或，把高位也加入运算，减少hashCode低位相同情况下的hash冲突
  static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

比如有几个不同的key，对应的hashCode低4位都一样。那么在length=16时，

通过 hashCode&（length-1） 得到的index都是一样的。因为高位的hashCode值并没有作用。

1.8中将hashCode高16位与低16位异或，把高位也加入运算，减少hashCode低位相同情况下的hash冲突

### (3)讲讲hashmap的get/put的过程?

- **知道hashmap中put元素的过程是什么样么?**

   对key的hashCode()做hash运算（对hashcode的高位与低位进行异或运算），计算index; 如果没碰撞直接放到bucket里； 如果碰撞了，以链表的形式存在buckets后； 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树(JDK1.8中的改动)； 如果节点已经存在就替换old value(保证key的唯一性) 如果bucket满了(超过load factor*current capacity)，就要resize。 

- **知道hashmap中get元素的过程是什么样么？**

  ```java
  /*
  	直接通过hash值求出索引，获取table中对应的元素first。
  	判断first是否为空，不为空判断hash值是否相等，相等判断key是否相等或者和equals，如果成功则返回
  	失败则判断是树还是链表，如果是链表则遍历first.next一致往下找O(n)
  	如果是树则是O(logn)
  */
  final Node<K,V> getNode(int hash, Object key) {
      Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
      if ((tab = table) != null && (n = tab.length) > 0 &&
          (first = tab[(n - 1) & hash]) != null) {
          if (first.hash == hash && // always check first node
              ((k = first.key) == key || (key != null && key.equals(k))))
              return first;
          if ((e = first.next) != null) {
              if (first instanceof TreeNode)
                  return ((TreeNode<K,V>)first).getTreeNode(hash, key);
              do {
                  if (e.hash == hash &&
                      ((k = e.key) == key || (key != null && key.equals(k))))
                      return e;
              } while ((e = e.next) != null);
          }
      }
      return null;
  }
  ```

  

- **你还知道哪些hash算法？**

   先说一下hash算法干嘛的，把任意长度的二进制映射成固定长度的二进制。把大范围映射到一个小范围的目的往往是为了节省空间，使得数据容易保存。 比较出名的有MurmurHash、MD4、MD5等等 

- **说说String中hashcode的实现?(此题很多大厂问过)**

```java
   /**
     *之所以选择31，是因为它是个奇素数，如果乘数是偶数，并且乘法溢出的话，信息就会丢失，因为与2相乘        *  等价于移位运算。使用素数的好处并不是很明显，但是习惯上都使用素数来计算散列结果。31有个很          * 好的特性，就是用移位和减法来代替乘法，可以得到更好的性能：31*i==(i<<5)-i。现在的VM可以自动完      *  成这种优化。
     * 31是奇质数  31*i=32*i-i=(i<<5)-i 效率高
     * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
     */
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }

```

### (4)为什么hashmap的在链表元素数量超过8时改为红黑树?

**当table的长度小于64时优先扩容，不会转成红黑树。**

- **知道jdk1.8中hashmap改了啥么?**

  - 由**数组+链表**的结构改为**数组+链表+红黑树**。
  - 优化了高位运算的hash算法：h^(h>>>16)
  - 扩容后，元素要么是在原位置，要么是在原位置再移动2次幂的位置，且链表顺序不变。

- **为什么在解决hash冲突的时候，不直接用红黑树?而选择先用链表，再转红黑树?**

   因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。 当元素小于8个当时候，此时做查询操作，链表结构已经能保证查询性能。当元素大于8个的时候，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。 

- **我不用红黑树，用二叉查找树可以么?**

  极端情况下，二叉查找树也可能变成线性结构。比如一直插入更大的数据。而红黑树是平衡二叉树。

- **那为什么阀值是8呢?**

  泊松分布，在链表长度大于8的概率小于千万分之一。

- **当链表转为红黑树后，什么时候退化为链表?**

   为6的时候退转为链表。中间有个差值7可以防止链表和树之间频繁的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。 

### (5)HashMap的并发问题?

- **HashMap在并发编程环境下有什么问题啊?**

  - (1)多线程扩容，引起的死循环问题
  - (2)多线程put的时候可能导致元素丢失
  - (3)put非null元素后get出来的却是null

- **在jdk1.8中还有这些问题么?**

   在jdk1.8中，死循环问题已经解决。其他两个问题还是存在。 

- **你一般怎么解决这些问题的？**

   比如ConcurrentHashmap，Hashtable等线程安全等集合类。 

```java
 private static ConcurrentHashMap<String,String> map=new  ConcurrentHashMap<>();

 private static Map<String,String> map=Collections.synchronizedMap(new HashMap<>());
```

### (6)你一般用什么作为HashMap的key?

- **健可以为Null值么?**

  可以，此时hash值为0，index也为0

- **你一般用什么作为HashMap的key?**

  一般用Integer、String这种不可变类当HashMap当key，而且String最为常用。

  - (1)因为字符串是不可变的，所以在它**创建的时候hashcode就被缓存了**（可变类的hashcode会随着对象值的改变而改变），不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

  - (2)因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的覆写了hashCode()以及equals()方法。

    ```java
    //未重写hashcode
     HashMap<Test, String> map = new HashMap<>();
            Test test = new Test(1);
            map.put(test,"1");
            System.out.println(map.get(test));//输出1
            test.set(2);
            System.out.println(map.get(test));//输出1
    //Test类重写hashcode
      @Override
        public int hashCode() {
            return i;
        }
    
    HashMap<Test, String> map = new HashMap<>();
            Test test = new Test(1);
            map.put(test,"1");
            System.out.println(map.get(test));//1
            test.set(2);
            System.out.println(map.get(test));//null
    ```

    

- **我用可变类当HashMap的key有什么问题?**

  hashcode可能发生改变，导致put进去的值，无法get出，如下所示	

  ```java
  HashMap<List<String>, Object> changeMap = new HashMap<>();
  List<String> list = new ArrayList<>();
  list.add("hello");
  Object objectValue = new Object();
  changeMap.put(list, objectValue);
  System.out.println(changeMap.get(list));
  list.add("hello world");//hashcode发生了改变
  System.out.println(changeMap.get(list));//null  ArrayList重写了hashcode方法
  ```

- **如果让你实现一个自定义的class作为HashMap的key该如何实现？**

  此题考察两个知识点

  - 重写hashcode和equals方法注意什么? 
  - 如何设计一个不变类
  

 **针对问题一，记住下面四个原则即可** 

(1)两个对象相等，hashcode一定相等 

(2)两个对象不等，hashcode不一定不等 

(3)hashcode相等，两个对象不一定相等

 (4)hashcode不等，两个对象一定不等

**针对问题二，记住如何写一个不可变类**

(1)类添加final修饰符，保证类不被继承。 如果类可以被继承会破坏类的不可变性机制，只要继承类覆盖父类的方法并且继承类可以改变成员变量值，那么一旦子类以父类的形式出现时，不能保证当前类是否可变。

(2)保证所有成员变量必须私有，并且加上final修饰 通过这种方式保证成员变量不可改变。但只做到这一步还不够，因为如果是对象成员变量有可能再外部改变其值。所以第4点弥补这个不足。

(3)不提供改变成员变量的方法，包括setter 避免通过其他接口改变成员变量的值，破坏不可变特性。

(4)通过构造器初始化所有成员，进行**深拷贝(deep copy)** 如果构造器传入的对象直接赋值给成员变量，还是可以通过对传入对象的修改进而导致改变内部变量的值。例如：

```text
public final class ImmutableDemo {  
    private final int[] myArray;  
    public ImmutableDemo(int[] array) {  
        this.myArray = array; // wrong  
    }  
}
```

这种方式不能保证不可变性，myArray和array指向同一块内存地址，用户可以在ImmutableDemo之外通过修改array对象的值来改变myArray内部的值。 为了保证内部的值不被修改，可以采用深度copy来创建一个新内存保存传入的值。正确做法：

```text
public final class MyImmutableDemo {  
    private final int[] myArray;  
    public MyImmutableDemo(int[] array) {  
        this.myArray = array.clone();   
    }   
}
```

(5)在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝 这种做法也是防止对象外泄，防止通过getter获得内部可变成员对象后对成员变量直接操作，导致成员变量发生改变。



### **7.HashMap，LinkedHashMap，TreeMap 有什么区别？**

HashMap 参考其他问题；

LinkedHashMap 保存了记录的插入顺序，在用 Iterator 遍历时，先取到的记录肯定是先插入的；遍历比 HashMap 慢；

![image-20191218104805234](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191218104805234.png)

```java
//在hashMap节点的基础上，加了before、和after两个引用，指向前一个插入的节点和后一个节点
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
//重写get方法，afterNodeAccess()方法将被访问的元素移动至双向链表的尾部
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;	
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
//put之后将节点添加到双向链表的尾部

```



TreeMap 实现 SortMap 接口，能够把它保存的记录根据键排序（默认按键值升序排序，也可以指定排序的比较器）

由于底层是红黑树，那么时间复杂度可以保证为log(n)

**TreeMap key不能为null，为null为抛出NullPointException的**

想要自定义比较，在构造方法中传入Comparator对象，否则使用key的自然排序来进行比较

TreeMap非同步的，想要同步可以使用Collections来进行封装

### **8.HashMap & TreeMap & LinkedHashMap 使用场景？**

一般情况下，使用最多的是 HashMap。

HashMap：在 Map 中插入、删除和定位元素时；

TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；

LinkedHashMap：在需要输出的顺序和输入的顺序相同的情况下。

### **9.HashMap 和 HashTable 有什么区别？**

①、HashMap 是线程不安全的，HashTable 是线程安全的；

②、由于线程安全，所以 HashTable 的效率比不上 HashMap；

③、**HashMap最多只允许一条记录的键为null，允许多条记录的值为null，而 HashTable不允许**；

④、HashMap 默认初始化数组的大小为16，HashTable 为 11，前者扩容时，扩大两倍，后者扩大两倍+1；

⑤、HashMap 需要重新计算 hash 值，而 HashTable 直接使用对象的 hashCode

### **10.Java 中的另一个线程安全的与 HashMap 极其类似的类是什么？同样是线程安全，它与 HashTable 在线程同步上有什么不同？**

ConcurrentHashMap 类（是 Java并发包 java.util.concurrent 中提供的一个线程安全且高效的 HashMap 实现）。

HashTable 是使用 synchronize 关键字加锁的原理（就是对对象加锁）；

而针对 ConcurrentHashMap，在 JDK 1.7 中采用 分段锁的方式；JDK 1.8 中直接采用了CAS（无锁算法）+ synchronized。

### **11.HashMap & ConcurrentHashMap 的区别？**

除了加锁，原理上无太大区别。另外，HashMap 的键值对允许有null，但是ConCurrentHashMap 都不允许。

### **12.为什么 ConcurrentHashMap 比 HashTable 效率要高？**

HashTable 使用一把锁（锁住整个链表结构）处理并发问题，多个线程竞争一把锁，容易阻塞；

ConcurrentHashMap

- JDK 1.7 中使用分段锁（ReentrantLock + Segment + HashEntry），相当于把一个 HashMap 分成多个段，每段分配一把锁，这样支持多线程访问。锁粒度：基于 Segment，包含多个 HashEntry。
- JDK 1.8 中使用 CAS + synchronized + Node + 红黑树。锁粒度：Node（首结点）（实现 Map.Entry<K,V>）。锁粒度降低了。