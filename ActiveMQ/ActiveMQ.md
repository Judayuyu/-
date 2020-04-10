![image-20200407171537731](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200407171537731.png)

![image-20200407171549060](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200407171549060.png)

![image-20200407171600822](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200407171600822.png)

### 简单使用

#### queue 队列模式 点对点

producer发送消息后，mq队列中有一则消息

consumer设置监听器，接受消息

![image-20200408214552115](D:%5C%E7%AC%94%E8%AE%B0%5C%E9%9D%A2%E8%AF%95%E9%A2%98%5Cjava%E9%94%81%5Cassets%5Cimage-20200408214552115.png)

#### topic  一对多  一个发布者 多个订阅者

特点：必须提前监听该topic，否则会错过消息