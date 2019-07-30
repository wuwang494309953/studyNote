### Dubbo看到的一些问题

**1.你对Dubbo源码这么熟悉,那请问你使用的时候,有没有遇到什么经典的坑？**

>eg: 提供者抛出的异常和消费者捕获的异常类型不一致。原因是消费者会将不能序列化的异常包装成RuntimeException抛给客户端。解决办法:要求提供者提供的接口jar包中包含异常类。
>
>参考: <https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247487320&idx=1&sn=3856df1643aa6c2bc10c41ef82fddbb5>

**2.消费端要调到zk上的服务的流程，dubbo都做了那些事**

>1.从zk中获取服务信息；
>
>2.负载均衡；
>
>3.重试；
>
>4.Netty连接；