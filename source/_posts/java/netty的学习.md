---
title: netty的学习
tags: [netty]
---
最近写android的tcp通信，用来传输文件，用原生的写太麻烦了，还有开线程互相监听，一堆操作比较繁琐，了解到netty比较好用，封装的简单，就用了这个来写。
用的netty版本 'io.netty:netty-all:4.1.6.Final' ,
先注册一个线程池 EventLoopGroup group =new NioEventLoopGroup();放在全局，
## 这里是服务端，只写了接收信息
```java
// MainApplication 里面的 因为这里是写的android所以放到了MainApplication里面注册的
EventLoopGroup group = MainApplication.getGroup();
ChannelFuture cf = MainApplication.getCf();
//


public static ChannelFuture cf =MainApplication.getCf();
public static EventLoopGroup group = MainApplication.getGroup();
public static void  start(){
    try {
    Bootstrap b = new Bootstrap();
    b.group(group) // 注册线程池
            .channel(NioSocketChannel.class)
            // 设置缓存区大小
    //                    .option(ChannelOption.RCVBUF_ALLOCATOR,
    //                            new AdaptiveRecvByteBufAllocator(64,1024,204800))
            // 使用NioSocketChannel来作为连接用的channel类
            .remoteAddress(new InetSocketAddress(host, port))
            // 绑定连接端口和host信息
            .handler(new ChannelInitializer<SocketChannel>() { // 绑定连接初始化器
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    Log.d(TAG, "initChannel: 正在连接中...");
                    // 设置好可以打印传输的流信息
    //                            ch.pipeline().addLast(new LoggingHandler(LogLevel.INFO));
                    ch.pipeline().addLast(new StringEncoder(Charset.forName("UTF-8")));
                    // 实例化一个处理类，传一些需要用到的参数参数
                    ch.pipeline().addLast(new ClientHandle(filePath, data, promise, host, port));
                    ch.pipeline().addLast(new ByteArrayEncoder());
                    ch.pipeline().addLast(new ChunkedWriteHandler());
                }
            });
    // System.out.println("服务端连接成功..");
    cf = b.connect().sync(); // 异步连接服务器
    cf.channel().closeFuture().sync();// 异步等待关闭连接channel
    }
    finally {
        // 最后执行完可以释放线程池，如果有多次操作就没必要释放了，
    //            group.shutdownGracefully().sync(); // 释放线程池资源
    }
}

```

```java

public class ClientHandle extends SimpleChannelInboundHandler<ByteBuf> {
    int fileSize = 0;
    private String TAG ="LoggingHandler";
    private DataOutputStream out;
    public ClientHandle(String filePath, Map data, String host, int port) throws FileNotFoundException {
        super();
        // 其他一些参数无视掉
        Log.e(TAG, "ClientHandle: "+"进来了ClientHandle");
        this.filePath = filePath;
        // 这里被实例化后就创建文件流。然后在下面写入文件
        out = new DataOutputStream(
                new BufferedOutputStream(
                        new FileOutputStream(filePath)));
    }
    /**
     * 向服务端发送数据
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
//        Log.d(TAG, "客户端与服务端通道-开启：" + ctx.channel().localAddress() +
//                "channelActive");
        Log.d(TAG, "channelActive: ");
        ctx.writeAndFlush(); // 可以发送流信息
    }
    /**
     * channelInactive
     *
     * channel 通道 Inactive 不活跃的
     *
     * 当客户端主动断开服务端的链接后，这个通道就是不活跃的。也就是说客户端与服务端的关闭了通信通道并且不可
     以传输数据
     *
     */
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {

        Log.d(TAG, "客户端与服务端通道-关闭：" + ctx.channel().localAddress() +
                "channelInactive");
    }
    // 这里读取数据，每次流进来就会触发此函数
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        // 这里msg当然需要转化下的，转换格式根据服务端的编码转换
        out.write(msg)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws
            Exception {
        ctx.close();
        cause.printStackTrace();
        Log.d(TAG, "异常退出");
    }
}
```

## 客户端
这里可能有点不是很搭，但是毕竟好配合我们的业务，因为客户端是安装在一个小板子上面的，板子存储能力很弱，所有就选择了node来做服务端来发送消息。
### node tcp发送消息
```js
var net = require('net');
net.createServer(function(socket){

	// socket.setKeepAlive(true, 1000);
  socket.on('data', function(data){
    transferUtils.resolveAndSendFile(data, socket);
  });
  socket.on('close',function () {
	  console.log("client close");
  });
  socket.on('error', function(err){
    console.log("Socket transmit error!", err);
  })
  // 监听15000端口
}).listen(15000, "0.0.0.0");  

```
