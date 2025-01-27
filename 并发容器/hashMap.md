#### 1.核心数据结构

![1572762955862](D:\笔记\面试题\java锁\assets\1572762955862.png)

![1572786456394](D:\笔记\面试题\java锁\assets\1572786456394.png)

#### 2.初始容量

##### 2.1 hash冲突解决方法：

* java7  头插入法，后插入的节点作为头节点

![1572787387336](D:\笔记\面试题\java锁\assets\1572787387336.png)

##### 2.2 为什么容量是2的指数此幂 【https://zhuanlan.zhihu.com/p/31610616】

######  	2.2.1 要使hash位运算后的值能在数组的下标之内

###### 	 2.2.2 使hash值均匀散列，还可以避免有些index永远不会出现

  			 假如是与 110 作&运算，实际上只有两个有效位，并且 index 1 永远不会被用到

###### 	 2.2.3 使位运算等于取模运算【hash&（length-1）==hash%length】 ，提高效率，扩容时rehash更快

 		      取模的缺点：对负数取模还是负数，效率慢。

​			   不是2的指数幂时： 111&（7-1）=110 不等于 7%7=0

​			   是2的指数幂时：	111&（8-1）=111 等于 7%8=7

![1572788355495](D:\笔记\面试题\java锁\assets\1572788355495.png)

#### 3.扩容与死锁，扩容是原来的2倍

java 7中会死锁



java 8中扩容时不需要rehash

扩容前

![1572791817885](D:\笔记\面试题\java锁\assets\1572791817885.png)

扩容后

![1572791860649](D:\笔记\面试题\java锁\assets\1572791860649.png)

#### 4. 加载因子0.75

加载因子偏低时，空间利用率低

加载因子高时，hash冲突增加，复杂度增加

取平衡

#### 5.链表转红黑树

![1572794220910](D:\笔记\面试题\java锁\assets\1572794220910.png)

![1572793932077](D:\笔记\面试题\java锁\assets\1572793932077.png)

