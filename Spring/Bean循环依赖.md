#Spring默认支持单例下非构造方法的循环依赖

![image-20200131172159517](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200131172159517.png)

## SingletonObejects 

![image-20200131172944819](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200131172944819.png)

![image-20200209164014588](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209164014588.png) 

![image-20200209151352736](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209151352736.png)

# SingletonsCurrentlyInCreation

该对象表示正在被创建的bean

![image-20200209161630424](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209161630424.png)

![image-20200209161708274](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209161708274.png)

注入属性

![image-20200209161934976](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209161934976.png)

# earlySingletonObjects



![image-20200209163754595](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209163754595.png)



![image-20200209170739453](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200209170739453.png)

# 循环依赖中注入时不是X不是代理对象，也不是引用，而是一个工厂，所以在Y对象调用X的方法时，切面生效，说明其X是代理对象

* getBean时，先获取到工厂类，而不是直接获取到X的实例 先从三级缓存拿

* 这里用三级缓存是为了提高效率。例如X从三级缓存中拿，拿不到再从二级缓存拿，并将其放入三级缓存，

  从二级缓存移除（方便gc）

![image-20200211145108103](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200211145108103.png)

* 实现BeanPostProcessor接口的getEarlyBeanReference接口就可以在下面的代码中执行

![image-20200211145628118](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200211145628118.png) 