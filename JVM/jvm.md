![image-20191129154823888](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191129154823888.png)

![image-20191129161146785](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191129161146785.png)

### 类装载器

1. 启动类加载器，加载Object、String等类的class文件
2. 扩展类加载器 ，加载javax包下的扩展类
3. 应用程序类加载器，自己编写的类

![image-20191129161206416](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191129161206416.png)

### 双亲委派机制

加载class文件要加载类时，先从最顶层的加载器里找，找到就加载进来。

即使自定义了一个java.lang.Object，在加载Object这个类的时候，会先从bootstrap加载器中的rt.jar中去找，

而自定义的这个被屏蔽掉

![image-20191210173330291](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191210173330291.png)

# 内存区域

![image-20191209221522915](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191209221522915.png)

### 栈

1. 对每个线程都在栈中开辟出单独的一块区域
2. 在线程的栈中，为每个方法的执行开辟空间，叫栈帧
3. 在栈帧中，包括局部变量表和操作数栈等。
4. 操作数栈是为了做临时的运算开辟出的一块内存，运算完存放操作结果
5. 方法出口是main方法在调用compute方法时放入的main方法返回地址

![image-20191209224513214](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191209224513214.png)

### 程序计数器

1. 程序计数器存放的是当前执行的指令的地址
2. 在多线程程序中，如果当前线程被挂起，当该线程重新抢占cpu时，cpu从程序计数器指向的指令开始执行

### 堆、栈之间的关系

![image-20191209231926350](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191209231926350.png)

### 堆

* 在堆中分为老年代和年轻代

* 年轻代中有Eden区，Survivor0区，Survivor1区，内存比例如图
* 当new一个对象时，加入Eden区，满了之后触发**minor gc**，存活的对象放入Survivor0区，并且当前的**分代年龄**增加1。
* 当Eden区再次触发**minor gc**时，收集Eden区和Survivor0区的内存，存活的放入Survivor1区，分代年龄+1
* 当Eden区第三次触发**minor gc**时，收集Eden区和Survivor1区的内存，存活的放入Survivor0区，分代年龄+1
* 循环执行上面两个步骤，当分代年龄达到15（可以设置）时，移到老年代
* 老年代内存满时，触发**full gc**

![image-20191210105334886](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191210105334886.png)

![image-20191210102903087](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191210102903087.png)

### STW 

#### stop the world

* 当进行full gc时，会STW，暂停所有线程。保证从gc root查找垃圾对象时不会被中断

#### 调优

减少full gc的次数

![image-20191223101359794](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20191223101359794.png)

