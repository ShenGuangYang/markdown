# 集群容错

dubbo 提供了6种容错机制，分别如下：
1. failsafe 失败安全，可以认为是把错误跳过（记录日志）
2. failover（默认） 重试其他服务器，retries(2) 
3. failfast 快速失败，失败后立马报错
4. failback 失败后自动恢复
5. forking forks 设置并行数
6. broadcast 广播，任意一台报错，则执行的方法报错

配置方式如下：通过`cluster`方式，配置指导的容错方案
```xml
<dubbo:reference interface="com.shen.IHello" id="helloService"
                     registry="zookeeper"
                     mock="com.shen.Mock"
                     timeout="1000"
                     version="1.0.0" 
                     cluster="failsafe"/>
```
# 服务降级
降级的目的是为了保证核心服务可用。
降级可以有几个层面的分类：自动降级和人工降级。
1. 对一些非核心服务进行人工降级，在大促之前通过降级开关关闭那些推荐内容、评价等对主流程没有影响的功能。
2. 故障降级，比如调用的远程服务挂了，网络故障、或者rpc服务返回异常。那么可以直接降级，降级方案比如设置默认值、采用兜底数据等等
3. 限流降级，在秒杀这种流量比较集中并且流量特别大的情况下，因为突发访问量特别大可能会导致系统支撑不了。这个时候可以采用限流来限制访问量。当达到阀值时，后续的请求被降级，比如进入排队页面，或跳转到错误页。

dubbo的降级方式： Mock
实现步骤
1. 在 client 端创建一个HelloMock 类，实现对应的Ihello接口
```java
public class HelloMock implements IHello {
    @Override
    public String sayHello(String s) {
        return "系统繁忙" + s;
    }
}
```
2. 在 client 端的xml配置文件中，添加如下配置，增加一个`mock`属性指向HelloMock
3. 模拟错误（设置timeout），模拟超时异常，运行测试代码即可访问到HelloMock这个类。当服务端故障解除以后，调用过程将恢复正常。

```xml
<dubbo:reference interface="com.shen.IHello" id="helloService"
                     registry="zookeeper"
                     mock="com.shen.HelloMock"
                     timeout="1000"
                     version="1.0.0" cluster="failsafe"/>s
```


