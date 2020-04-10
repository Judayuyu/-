![image-20200406223009022](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200406223009022.png)

![image-20200406225852774](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200406225852774.png)

### HandlerMapping

1. HandlerMapping定义了请求到处理器（Controller）之间的映射

2.将Url交给HandlerMapping可以得到一个处理器链HandlerExecutionChain

![image-20200406230152284](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200406230152284.png)

3.根据上面获取的HandlerExecutionChain得到一个adapter，用来处理表单数据校验或者类型转换等事情

![image-20200406230803290](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200406230803290.png)

4.根据ha调用目标方法

![image-20200406231711637](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200406231711637.png)

5.调用完返回ModelAndView对象，交给视图解析器ViewResolver转成真正的视图