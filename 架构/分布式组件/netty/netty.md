# netty 

## netty 仿写im初体验 

### 简单im代码demo 

**服务端代码**

```java
public class NettyServer {
    public static void main(String[] args) {
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        NioEventLoopGroup boss = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();

        serverBootstrap
                .group(boss, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringDecoder());
                        ch.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
                                System.out.println(msg);
                            }
                        });
                    }
                }).bind(8000);
    }
}
```



**客户端代码**

```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        Bootstrap bootstrap = new Bootstrap();
        NioEventLoopGroup group = new NioEventLoopGroup();

        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel ch) {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                });

        Channel channel = bootstrap.connect("127.0.0.1", 8000).channel();

        while (true) {
            channel.writeAndFlush(new Date() + ": hello world!");
            Thread.sleep(1000);
        }
    }
}
```



### api 初步使用介绍



### 服务端



```java
public class NettyServer {
    public static void main(String[] args) {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                .group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    protected void initChannel(NioSocketChannel ch) {
                    }
                });

        serverBootstrap.bind(8000);
    }
}
```

- 首先看到，我们创建了两个`NioEventLoopGroup`，这两个对象可以看做是传统IO编程模型的两大线程组，`bossGroup`表示监听端口，accept 新连接的线程组，`workerGroup`表示处理每一条连接的数据读写的线程组。用生活中的例子来讲就是，一个工厂要运作，必然要有一个老板负责从外面接活，然后有很多员工，负责具体干活，老板就是`bossGroup`，员工们就是`workerGroup`，`bossGroup`接收完连接，扔给`workerGroup`去处理。
- 接下来 我们创建了一个引导类 `ServerBootstrap`，这个类将引导我们进行服务端的启动工作，直接new出来开搞。
- 我们通过`.group(bossGroup, workerGroup)`给引导类配置两大线程组，这个引导类的线程模型也就定型了。
- 然后，我们指定我们服务端的 IO 模型为`NIO`，我们通过`.channel(NioServerSocketChannel.class)`来指定 IO 模型，当然，这里也有其他的选择，如果你想指定 IO 模型为 BIO，那么这里配置上`OioServerSocketChannel.class`类型即可，当然通常我们也不会这么做，因为Netty的优势就在于NIO。
- 接着，我们调用`childHandler()`方法，给这个引导类创建一个`ChannelInitializer`，这里主要就是定义后续每条连接的数据读写，业务处理逻辑，不理解没关系，在后面我们会详细分析。`ChannelInitializer`这个类中，我们注意到有一个泛型参数`NioSocketChannel`，这个类呢，就是 Netty 对 NIO 类型的连接的抽象，而我们前面`NioServerSocketChannel`也是对 NIO 类型的连接的抽象，`NioServerSocketChannel`和`NioSocketChannel`的概念可以和 BIO 编程模型中的`ServerSocket`以及`Socket`两个概念对应上

我们的最小化参数配置到这里就完成了，我们总结一下就是，要启动一个Netty服务端，必须要指定三类属性，分别是线程模型、IO 模型、连接读写处理逻辑，有了这三者，之后在调用`bind(8000)`，我们就可以在本地绑定一个 8000 端口启动起来，以上这段代码读者可以直接拷贝到你的 IDE 中运行。

#### 自动绑定递增端口

在上面代码中我们绑定了 8000 端口，接下来我们实现一个稍微复杂一点的逻辑，我们指定一个起始端口号，比如 1000，然后呢，我们从1000号端口往上找一个端口，直到这个端口能够绑定成功，比如 1000 端口不可用，我们就尝试绑定 1001，然后 1002，依次类推。

`serverBootstrap.bind(8000);`这个方法呢，它是一个异步的方法，调用之后是立即返回的，他的返回值是一个`ChannelFuture`，我们可以给这个`ChannelFuture`添加一个监听器`GenericFutureListener`，然后我们在`GenericFutureListener`的`operationComplete`方法里面，我们可以监听端口是否绑定成功，接下来是监测端口是否绑定成功的代码片段

```java
serverBootstrap.bind(8000).addListener(new GenericFutureListener<Future<? super Void>>() {
    public void operationComplete(Future<? super Void> future) {
        if (future.isSuccess()) {
            System.out.println("端口绑定成功!");
        } else {
            System.err.println("端口绑定失败!");
        }
    }
});
```

我们接下来从 1000 端口号，开始往上找端口号，直到端口绑定成功，我们要做的就是在 `if (future.isSuccess())`的else逻辑里面重新绑定一个递增的端口号，接下来，我们把这段绑定逻辑抽取出一个`bind`方法

```java
private static void bind(final ServerBootstrap serverBootstrap, final int port) {
    serverBootstrap.bind(port).addListener(new GenericFutureListener<Future<? super Void>>() {
        public void operationComplete(Future<? super Void> future) {
            if (future.isSuccess()) {
                System.out.println("端口[" + port + "]绑定成功!");
            } else {
                System.err.println("端口[" + port + "]绑定失败!");
                bind(serverBootstrap, port + 1);
            }
        }
    });
}
```

然后呢，以上代码中最关键的就是在端口绑定失败之后，重新调用自身方法，并且把端口号加一，然后，在我们的主流程里面，我们就可以直接调用

```java
bind(serverBootstrap, 1000)
```

读者可以自定修改代码，运行之后可以看到效果，最终会发现，端口成功绑定了在1024，从 1000 开始到 1023，端口均绑定失败了，这是因为在我的 MAC 系统下，1023 以下的端口号都是被系统保留了，需要 ROOT 权限才能绑定。

以上就是自动绑定递增端口的逻辑，接下来，我们来一起学习一下，服务端启动，我们的引导类`ServerBootstrap`除了指定线程模型，IO 模型，连接读写处理逻辑之外，他还可以干哪些事情？





#### handler() 方法

```java
serverBootstrap.handler(new ChannelInitializer<NioServerSocketChannel>() {
    protected void initChannel(NioServerSocketChannel ch) {
        System.out.println("服务端启动中");
    }
})
```

`handler()`方法呢，可以和我们前面分析的`childHandler()`方法对应起来，`childHandler()`用于指定处理新连接数据的读写处理逻辑，`handler()`用于指定在服务端启动过程中的一些逻辑，通常情况下呢，我们用不着这个方法。

#### attr() 方法

```java
serverBootstrap.attr(AttributeKey.newInstance("serverName"), "nettyServer")
```

`attr()`方法可以给服务端的 channel，也就是`NioServerSocketChannel`指定一些自定义属性，然后我们可以通过`channel.attr()`取出这个属性，比如，上面的代码我们指定我们服务端channel的一个`serverName`属性，属性值为`nettyServer`，其实说白了就是给`NioServerSocketChannel`维护一个map而已，通常情况下，我们也用不上这个方法。

那么，当然，除了可以给服务端 channel `NioServerSocketChannel`指定一些自定义属性之外，我们还可以给每一条连接指定自定义属性

#### childAttr() 方法

```java
serverBootstrap.childAttr(AttributeKey.newInstance("clientKey"), "clientValue")
```

上面的`childAttr`可以给每一条连接指定自定义属性，然后后续我们可以通过`channel.attr()`取出该属性。

#### childOption() 方法

```java
serverBootstrap
        .childOption(ChannelOption.SO_KEEPALIVE, true)
        .childOption(ChannelOption.TCP_NODELAY, true)
```

`childOption()`可以给每条连接设置一些TCP底层相关的属性，比如上面，我们设置了两种TCP属性，其中

- `ChannelOption.SO_KEEPALIVE`表示是否开启TCP底层心跳机制，true为开启
- `ChannelOption.TCP_NODELAY`表示是否开启Nagle算法，true表示关闭，false表示开启，通俗地说，如果要求高实时性，有数据发送时就马上发送，就关闭，如果需要减少发送次数减少网络交互，就开启。

其他的参数这里就不一一讲解，有兴趣的同学可以去这个类里面自行研究。

#### option() 方法

除了给每个连接设置这一系列属性之外，我们还可以给服务端channel设置一些属性，最常见的就是so_backlog，如下设置

```java
serverBootstrap.option(ChannelOption.SO_BACKLOG, 1024)
```

表示系统用于临时存放已完成三次握手的请求的队列的最大长度，如果连接建立频繁，服务器处理创建新连接较慢，可以适当调大这个参数



### 客户端

```java
public class NettyClient {
    public static void main(String[] args) {
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        
        Bootstrap bootstrap = new Bootstrap();
        bootstrap
                // 1.指定线程模型
                .group(workerGroup)
                // 2.指定 IO 类型为 NIO
                .channel(NioSocketChannel.class)
                // 3.IO 处理逻辑
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) {
                    }
                });
        // 4.建立连接
        bootstrap.connect("juejin.im", 80).addListener(future -> {
            if (future.isSuccess()) {
                System.out.println("连接成功!");
            } else {
                System.err.println("连接失败!");
            }

        });
    }
}
```

从上面代码可以看到，客户端启动的引导类是 `Bootstrap`，负责启动客户端以及连接服务端，而上一小节我们在描述服务端的启动的时候，这个辅导类是 `ServerBootstrap`，引导类创建完成之后，下面我们描述一下客户端启动的流程

1. 首先，与服务端的启动一样，我们需要给它指定线程模型，驱动着连接的数据读写
2. 然后，我们指定 IO 模型为 `NioSocketChannel`，表示 IO 模型为 NIO，当然，你可以可以设置 IO 模型为 `OioSocketChannel`，但是通常不会这么做，因为 Netty 的优势在于 NIO
3. 接着，给引导类指定一个 `handler`，这里主要就是定义连接的业务处理逻辑，不理解没关系，在后面我们会详细分析
4. 配置完线程模型、IO 模型、业务处理逻辑之后，调用 `connect` 方法进行连接，可以看到 `connect` 方法有两个参数，第一个参数可以填写 IP 或者域名，第二个参数填写的是端口号，由于 `connect` 方法返回的是一个 `Future`，也就是说这个方是异步的，我们通过 `addListener` 方法可以监听到连接是否成功，进而打印出连接信息

到了这里，一个客户端的启动的 Demo 就完成了。



#### 失败重连

在网络情况差的情况下，客户端第一次连接可能会连接失败，这个时候我们可能会尝试重新连接，重新连接的逻辑写在连接失败的逻辑块里

```java
bootstrap.connect("juejin.im", 80).addListener(future -> {
    if (future.isSuccess()) {
        System.out.println("连接成功!");
    } else {
        System.err.println("连接失败!");
        // 重新连接
    }
});
```

重新连接的时候，依然是调用一样的逻辑，因此，我们把建立连接的逻辑先抽取出来，然后在重连失败的时候，递归调用自身

```java
private static void connect(Bootstrap bootstrap, String host, int port) {
    bootstrap.connect(host, port).addListener(future -> {
        if (future.isSuccess()) {
            System.out.println("连接成功!");
        } else {
            System.err.println("连接失败，开始重连");
            connect(bootstrap, host, port);
        }
    });
}
```

上面这一段便是带有自动重连功能的逻辑，可以看到在连接建立失败的时候，会调用自身进行重连。

但是，通常情况下，连接建立失败不会立即重新连接，而是会通过一个指数退避的方式，比如每隔 1 秒、2 秒、4 秒、8 秒，以 2 的幂次来建立连接，然后到达一定次数之后就放弃连接，接下来我们就来实现一下这段逻辑，我们默认重试 5 次

```java
connect(bootstrap, "juejin.im", 80, MAX_RETRY);

private static void connect(Bootstrap bootstrap, String host, int port, int retry) {
    bootstrap.connect(host, port).addListener(future -> {
        if (future.isSuccess()) {
            System.out.println("连接成功!");
        } else if (retry == 0) {
            System.err.println("重试次数已用完，放弃连接！");
        } else {
            // 第几次重连
            int order = (MAX_RETRY - retry) + 1;
            // 本次重连的间隔
            int delay = 1 << order;
            System.err.println(new Date() + ": 连接失败，第" + order + "次重连……");
            bootstrap.config().group().schedule(() -> connect(bootstrap, host, port, retry - 1), delay, TimeUnit.SECONDS);
        }
    });
}
```

从上面的代码可以看到，通过判断连接是否成功以及剩余重试次数，分别执行不同的逻辑

1. 如果连接成功则打印连接成功的消息
2. 如果连接失败但是重试次数已经用完，放弃连接
3. 如果连接失败但是重试次数仍然没有用完，则计算下一次重连间隔 `delay`，然后定期重连

在上面的代码中，我们看到，我们定时任务是调用 `bootstrap.config().group().schedule()`, 其中 `bootstrap.config()` 这个方法返回的是 `BootstrapConfig`，他是对 `Bootstrap` 配置参数的抽象，然后 `bootstrap.config().group()` 返回的就是我们在一开始的时候配置的线程模型 `workerGroup`，调 `workerGroup` 的 `schedule` 方法即可实现定时任务逻辑。

在 `schedule` 方法块里面，前面四个参数我们原封不动地传递，最后一个重试次数参数减掉一，就是下一次建立连接时候的上下文信息。读者可以自行修改代码，更改到一个连接不上的服务端 Host 或者 Port，查看控制台日志就可以看到5次重连日志。

以上就是实现指数退避的客户端重连逻辑，接下来，我们来一起学习一下，客户端启动，我们的引导类`Bootstrap`除了指定线程模型，IO 模型，连接读写处理逻辑之外，他还可以干哪些事情？



#### attr() 方法

```java
bootstrap.attr(AttributeKey.newInstance("clientName"), "nettyClient")
```

`attr()` 方法可以给客户端 Channel，也就是`NioSocketChannel`绑定自定义属性，然后我们可以通过`channel.attr()`取出这个属性，比如，上面的代码我们指定我们客户端 Channel 的一个`clientName`属性，属性值为`nettyClient`，其实说白了就是给`NioSocketChannel`维护一个 map 而已，后续在这个 `NioSocketChannel` 通过参数传来传去的时候，就可以通过他来取出设置的属性，非常方便。

#### option() 方法

```java
Bootstrap
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
        .option(ChannelOption.SO_KEEPALIVE, true)
        .option(ChannelOption.TCP_NODELAY, true)
```

`option()` 方法可以给连接设置一些 TCP 底层相关的属性，比如上面，我们设置了三种 TCP 属性，其中

- `ChannelOption.CONNECT_TIMEOUT_MILLIS` 表示连接的超时时间，超过这个时间还是建立不上的话则代表连接失败
- `ChannelOption.SO_KEEPALIVE` 表示是否开启 TCP 底层心跳机制，true 为开启
- `ChannelOption.TCP_NODELAY` 表示是否开始 Nagle 算法，true 表示关闭，false 表示开启，通俗地说，如果要求高实时性，有数据发送时就马上发送，就设置为 true 关闭，如果需要减少发送次数减少网络交互，就设置为 false 开启

其他的参数这里就不一一讲解，有兴趣的同学可以去这个类里面自行研究。













