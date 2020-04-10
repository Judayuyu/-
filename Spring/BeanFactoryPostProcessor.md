## 可以用来优化以下代码

根据请求参数中的type，进行if判断，根据不同的type调用不同的service

![ ](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219155441564.png)

## 执行时间

**Spring扫描完需要加入容器的类，封装成BeanDefinition对象并放入map中，完成后调用BeanFactoryPostProcessor接口，并把map传到此接口上。可以在此将自己定义的类加入容器**

## 具体场景

### 1、设计一个接口，有多个实现

![image-20200219163614191](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219163614191.png)

### 2、设计一个注解，定义在实现的接口上

![image-20200219163644367](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219163644367.png)

### 3、定义自己的类handler，维护一个map，key为type的值，value是实现接口的class名字，当用户发送请求时，获取请求参数中的type，从map中获取对应的class。然后从Spring容器中取出接口实现类的bean。调用对应的方法。

![image-20200219163957033](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219163957033.png)

![image-20200219165222147](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219165222147.png)

### 4、 实现Bean...Processor接口，扫描标有注解的类，手动加入到容器中

![image-20200219164412112](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219164412112.png)

### 5、在service中注入Handler，根据请求参数type从Spring容器中获取相应的bean，调用对应的方法

![image-20200219164553639](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200219164553639.png)

