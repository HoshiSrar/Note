# Netty

对于传统的NIO有存在一些问题，而在Netty框架中，这些问题都被巧妙的解决了。

Netty 是由 JBOSS 提供的一个开源的 java 网络编程框架，主要是对 java 的 nio 包进行了再次封装。Netty 比java 原生的 nio 包提供了更加强大、稳定的功能和易于使用的 api。 netty 的作者是 Trustin Lee，这是一个韩国人，他还开发了另外一个著名的网络编程框架，mina。二者在很多方面都十分相似，它们的线程模型也是基本一致 。不过netty社区的活跃程度要 mina 高得多。

Netty 实际上应用场景非常多，比如 Minecraft 游戏服务器，

Java 版本的 Minecraft 服务器就是使用 Netty 框架作为网络通信的基础，正是得益于 Netty 框架的高性能，我们才能愉快地和其他的小伙伴一起在服务器里面炸服。

当然除了游戏服务器之外，我们微服务之间的远程调用也可以使用 Netty 来完成，比如 Dubbo 的 RPC 框架，包括最新的 SpringWebFlux 框架，也抛弃了内嵌 Tomcat 而使用 Netty 作为通信框架。

首先导包：

~~~XML
<dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.95.Final</version>
</dependency>
~~~

## 缓冲区ByteBuf介绍

Netty 并没有使用 NIO 中提供的 ByteBuffer 来进行数据装载，而是自行定义了一个 ByteBuf 类（与Java 的 ByteBuff 特意区分）。

那么这个类相比NIO中的ByteBuffer有什么不同之处呢？

- 写操作完成后无需进行`flip()`翻转。
- 具有比ByteBuffer更快的响应速度。
- 动态扩容。

首先我们来看看它的内部结构：

~~~java
public abstract class AbstractByteBuf extends ByteBuf {
    ...
    int readerIndex;   //index被分为了读和写，是两个指针在同时工作
    int writerIndex;
    private int markedReaderIndex;    //mark操作也分两种
    private int markedWriterIndex;
    private int maxCapacity;    //最大容量，没错，这玩意能动态扩容
~~~

读操作和写操作分别由两个指针在进行维护，每写入一次，`writerIndex`向后移动一位，每读取一次，也是`readerIndex`向后移动一位，当然`readerIndex`不能大于`writerIndex`，这样就不会像NIO中的ByteBuffer那样还需要进行翻转了。

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231209202337.png)

其中`readerIndex`和`writerIndex`之间的部分就是是可读的内容，而`writerIndex`之后到`capacity`都是可写的部分。

### 如何使用

我们来实际使用一下看看：

~~~java
public static void main(String[] args) {
    //创建一个初始容量为10的ByteBuf缓冲区，这里的Unpooled是用于快速生成ByteBuf的工具类
    //至于为啥叫Unpooled是池化的意思，ByteBuf有池化和非池化两种，区别在于对内存的复用，我们之后再讨论
    ByteBuf buf = Unpooled.buffer(10);
    System.out.println("初始状态："+Arrays.toString(buf.array()));
    //写入一个Int数据
    buf.writeInt(-888888888);   
    System.out.println("写入Int后："+Arrays.toString(buf.array()));
    //无需翻转，直接读取一个short数据出来
    buf.readShort();   
    System.out.println("读取Short后："+Arrays.toString(buf.array()));
    //丢弃操作，会将当前的可读部分内容丢到最前面，并且读写指针向前移动丢弃的距离
    buf.discardReadBytes();   
    System.out.println("丢弃之后："+Arrays.toString(buf.array()));
    //清空操作，清空之后读写指针都归零
    buf.clear();    
    System.out.println("清空之后："+Arrays.toString(buf.array()));
}
~~~

通过结合断点调试，我们可以观察读写指针的移动情况，更加清楚的认识一下ByteBuf的底层操作。

我们再来看看划分操作是不是和之前一样的：

~~~java
public static void main(String[] args) {
  	//我们也可以将一个byte[]直接包装进缓冲区（和NIO是一样的）不过写指针的值一开始就跑到最后去了，但是这玩意是不是只读的
    ByteBuf buf = Unpooled.wrappedBuffer("abcdefg".getBytes());
  	//除了包装，也可以复制数据，copiedBuffer()会完完整整将数据拷贝到一个新的缓冲区中
    buf.readByte();   //读取一个字节
    ByteBuf slice = buf.slice();   //现在读指针位于1，然后进行划分

    System.out.println(slice.arrayOffset());   //得到划分出来的ByteBuf的偏移地址
    System.out.println(Arrays.toString(slice.array()));
}
~~~

可以看到，划分也是根据当前读取的位置来进行的。

我们继续来看看它的另一个特性，动态扩容，比如我们申请一个容量为10的缓冲区：

~~~java
public static void main(String[] args) {
    ByteBuf buf = Unpooled.buffer(10);    //容量只有10字节
    System.out.println(buf.capacity());
  	//直接写一个字符串
    buf.writeCharSequence("牛逼！", StandardCharsets.UTF_8);   //很明显这么多字已经超过10字节了
    System.out.println(buf.capacity());
}
~~~

在写入一个超出当前容量的数据时，会进行动态扩容，扩容会从64开始，之后每次触发扩容都会x2，当然如果我们不希望它扩容，可以指定最大容量：

~~~java
public static void main(String[] args) {
    //在生成时指定maxCapacity也为10
    ByteBuf buf = Unpooled.buffer(10, 10);
    System.out.println(buf.capacity());
    //此时不再扩容，将会抛出异常
    buf.writeCharSequence("牛逼！", StandardCharsets.UTF_8);
    System.out.println(buf.capacity());
}
~~~

### 三种缓冲区

一共有三种缓冲区的实现模式：堆缓冲区模式、直接缓冲区模式、复合缓冲区模式。

* 堆、直接缓冲区模式：堆缓冲区（数组实现）和直接缓冲区（堆外内存实现）不用多说，前面我们在NIO中已经了解过了，创建一个直接缓冲区也很简单，但由于底层不是数组实现的所以获取数组时会报错。

~~~java
public static void main(String[] args) {
    ByteBuf buf = Unpooled.directBuffer(10);
    System.out.println(Arrays.toString(buf.array()));//buf.array()无法获取，报错
}
~~~



* 复合缓冲区模式：复合模式可以任意地拼凑组合其他缓冲区（只在逻辑上为一个缓冲区，实际仍然是多个），比如我们可以：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231211191819.png)

这样，如果我们想要对两个缓冲区组合的内容进行操作，我们就不用再单独创建一个新的缓冲区了，而是直接将其进行拼接操作，相当于是作为多个缓冲区组合的视图。在使用上：

~~~java
//创建一个复合缓冲区
CompositeByteBuf buf = Unpooled.compositeBuffer();
//abc、def将会拼凑到一起
buf.addComponent(Unpooled.copiedBuffer("abc".getBytes()));
buf.addComponent(Unpooled.copiedBuffer("def".getBytes()));

for (int i = 0; i < buf.capacity(); i++) {
    System.out.println((char) buf.getByte(i));
}
~~~

我们也可以正常操作组合后的缓冲区 buf 缓冲区。

------

## Netty工作模型

前面我们了解了Netty为我们提供的更高级的缓冲区类，我们接着来看看Netty是如何工作的，在 Java 的 NIO 中使用了Reactor模式，而 Netty 正是以主从Reactor多线程模型为基础，构建出了一套高效的工作模型。

大致工作模型图如下：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231211192320.png)

与主从 Reactor 多线程模型非常类似：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231211192350.png)

所有的客户端需要连接到主Reactor完成Accept操作后，其他的操作由从Reactor去完成，这里也是差不多的思想，但是它进行了一些改进，我们来看一下它的设计：

- Netty 使用抽象出的两组线程池BossGroup和WorkerGroup，BossGroup专门负责接受客户端的连接, WorkerGroup专门负读写操作，就像前面的主从Reactor一样。
- 无论是BossGroup还是WorkerGroup，都是使用EventLoop（事件循环，很多系统都采用了事件循环机制，比如前端框架Node.js，事件循环顾名思义，就是一个循环，不断地进行事件通知）来进行事件监听的，整个Netty也是使用事件驱动来运作的，比如当客户端已经准备好读写、连接建立时，都会进行事件通知，说白了就像我们之前写NIO多路复用那样，只不过这里换成EventLoop了而已，它已经帮助我们封装好了一些常用操作，而且我们可以自己添加一些额外的任务，如果有多个EventLoop，会存放在EventLoopGroup中，EventLoopGroup就是BossGroup和WorkerGroup的具体实现。
- 在BossGroup之后，会正常将SocketChannel绑定到WorkerGroup中的其中一个EventLoop上，进行后续的读写操作监听。

前面我们大致了解了一下Netty的工作模型，接着我们来尝试创建一个Netty服务器：

## channel

## EventLoop

## Future和Promise

## 解码器和编码器

在 Netty 中，我们的数据发送和接收都是以 ByteBuf 形式传输，但是这样并不方便，我们可以像 Java Web中的 Filter 一样，在开始处理数据之前对数据先进行过滤，并在过滤的途中将数据转换成我们想要的类型，也可以将发出的数据进行转换。想要实现这样的功能就需要用到编码和解码器了。

> 编码和解码器的原理很简单，它们的本质就是 handler。使用时将解码器对象添加到 netty 的 channel 流水线中即可。数据在 channel 中流动处理并发送到下一个 Handler 中。解码器我们可以使用 Netty 提供的 Handler，也可以继承相应接口编写我们自己的Handler。

我们先来看看最简的，与字符串相关的。如果我们要直接在客户端或是服务端处理字符串，可以直接添加一个字符串解码器到我们的管道流水线中：

~~~java
new StringDecoder();//将接收到的 ByteBuf 类型转换成 String 类型发送给下一个 channel。
~~~

```java
@Override
protected void initChannel(SocketChannel channel) {
    channel.pipeline()
            //解码器本质上也算是一种ChannelInboundHandlerAdapter，用于处理入站请求
            .addLast(new StringDecoder())   //当客户端发送来的数据只是简单的字符串转换的ByteBuf时，我们直接使用内置的StringDecoder即可转换
            .addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                    //经过StringDecoder转换后，msg直接就是一个字符串，所以打印就行了
                    System.out.println(msg);
                }
            });
}
```

可以看到，使用起来还是非常方便的，我们只需要将其添加到流水线即可，实际上解码器本质就是一个ChannelInboundHandlerAdapter：

![132](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213003608.png)

我们看到它是继承自MessageToMessageDecoder，用于将传入的Message转换为另一种类型，我们来编写自己的解码器：

```java
/**
 * 自定义的解码器，ByteBuf-> String (UTF-8格式)
 */
public class TestDecoder extends MessageToMessageDecoder<ByteBuf> {
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf buf, List<Object> list) throws Exception {
        System.out.println("数据已收到，正在进行解码...");
        String text = buf.toString(StandardCharsets.UTF_8);  //直接转换为UTF8字符串
        list.add(text);   //解码后需要将解析后的数据丢进List中，如果丢进去多个数据，相当于数据被分成了多个，后面的Handler就需要每个都处理一次
    }
}
```

运行，可以看到：

![image-20230306174333758](https://s2.loli.net/2023/03/06/aM6gy1BAeLUlEui.png)

当然如果我们在List里面丢很多个数据的话，也会处理多次：

```java
public class TestDecoder extends MessageToMessageDecoder<ByteBuf> {
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf buf, List<Object> list) throws Exception {
        System.out.println("数据已收到，正在进行解码...");
        String text = buf.toString(StandardCharsets.UTF_8);  //直接转换为UTF8字符串
        list.add(text);
        list.add(text+"2");
        list.add(text+'3');   //一条消息将会被解码成三条消息
    }
}
```

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213003946.png)

可以看到，后面的Handler会依次对三条数据都进行处理，当然，除了 MessageToMessageDecoder 之外，还有其他类型的解码器，比如 ByteToMessageDecoder 等。

Netty内置了很多的解码器实现来方便我们开发，比如HTTP，SMTP、MQTT等，以及我们常用的Redis、Memcached、JSON等数据包。

当然，有了解码器处理发来的数据，那发出去的数据肯定也是需要被处理的，所以编码器就出现了：

```java
channel.pipeline()
        .addLast(new StringDecoder())
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println("收到客户端的数据："+msg);
                ctx.channel().writeAndFlush("可以，不跟你多BB");  //直接发字符串回去
            }
        })
        .addLast(new StringEncoder());  //使用内置的StringEncoder可以直接将出站的字符串数据编码成ByteBuf
```

StringEncoder 和上面的 StringDecoder 一样，StringEncoder 本质上就是一个 ChannelOutboundHandlerAdapter：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213004010.png)

是不是感觉前面学习的Handler和Pipeline突然就变得有用了，直接一条线把数据处理安排得明明白白啊。

现在我们把客户端也改成使用编码、解码器的样子：

```java
public static void main(String[] args) {
    Bootstrap bootstrap = new Bootstrap();
    bootstrap
            .group(new NioEventLoopGroup())
            .channel(NioSocketChannel.class)
            .handler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel channel) throws Exception {
                    channel.pipeline()
                            .addLast(new StringDecoder())  //解码器安排
                            .addLast(new ChannelInboundHandlerAdapter(){
                                @Override
                                public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                    System.out.println(">> 接收到客户端发送的数据：" + msg);  //直接接收字符串
                                }
                            })
                            .addLast(new StringEncoder());  //编码器安排
                }
            });
    Channel channel = bootstrap.connect("localhost", 8080).channel();
    try(Scanner scanner = new Scanner(System.in)){
        while (true) {
            System.out.println("<< 请输入要发送给服务端的内容：");
            String text = scanner.nextLine();
            if(text.isEmpty()) continue;
            channel.writeAndFlush(text);  //直接发送字符串就行
        }
    }
}
```

这样我们的代码量又蹭蹭的减少了很多：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213004031.png)

当然，除了编码器和解码器之外，还有编解码器。它是两者的结合，里面需要实现两个方法处理出入站。

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213004050.png)

可以看到它是既继承了ChannelInboundHandlerAdapter也实现了ChannelOutboundHandler接口，又能处理出站也能处理入站请求，实际上就是将之前的给组合到一起了，比如我们也可以实现一个缝合在一起的StringCodec类：

```java
//需要指定两个泛型，第一个是入站操作时传入的消息类型，还有一个是出站的消息类型，这里出站操作时传入的是String类型，我们这里把它转成ByteBuf
public class StringCodec extends MessageToMessageCodec<ByteBuf, String> {
    
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf buf, List<Object> list) throws Exception {
        System.out.println("正在处理入站数据...");
        list.add(buf.toString(StandardCharsets.UTF_8));  //和之前一样，直接一行解决
    }
}

@Override
    protected void encode(ChannelHandlerContext channelHandlerContext, String buf, List<Object> list) throws Exception {
        System.out.println("正在处理出站数据...");
        list.add(Unpooled.wrappedBuffer(buf.getBytes()));   //同样的，添加的数量就是出站的消息数量
    }
```

可以看到实际上就是需要我们同时去实现编码和解码方法，继承MessageToMessageCodec类即可。

当然，如果整条流水线上有很多个解码器或是编码器，那么也可以多次进行编码或是解码，比如：

```java
public class StringToStringEncoder extends MessageToMessageEncoder<String> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, String s, List<Object> list) throws Exception {
        System.out.println("我是预处理编码器，就要皮这一下。");
        list.add("[已处理] "+s);
    }
}
```

```java
channel.pipeline()
        //解码器本质上也算是一种ChannelInboundHandlerAdapter，用于处理入站请求
        .addLast(new StringDecoder())
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println("收到客户端的数据："+msg);
                ctx.channel().writeAndFlush("可以，不跟你多BB");  //直接发字符串回去
            }
        })
        .addLast(new StringEncoder())    //最后再转成ByteBuf
        .addLast(new StringToStringEncoder());  //先从我们自定义的开始
```

可以看到，数据在流水线上一层一层处理最后再回到的客户端：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213004105.png)



对于粘包/拆包问题，也可以使用一个解码器解决：

```java
channel.pipeline()
        .addLast(new FixedLengthFrameDecoder(10))  
        //第一种解决方案，使用定长数据包，每个数据包都要是指定长度（这里指定为10），数据超过该长度将会多次发生直至发送完毕（数据有被强行截断的风险），该handler需要放在最前面
  			...
```

```java
channel.pipeline()
        .addLast(new DelimiterBasedFrameDecoder(1024, Unpooled.wrappedBuffer("!".getBytes())))
        //第二种，就是指定一个特定的分隔符，比如我们这里以感叹号为分隔符
  		//在收到分隔符之前的所有数据都会被封装成在一个数据包里的内容，
```

```java
channel.pipeline()
        .addLast(new LengthFieldBasedFrameDecoder(1024, 0, 4))
        //第三种方案，就是在头部添加长度信息，来确定当前发送的数据包具体长度是多少，客户端也需要配合添加一个编码器
        //offset是从哪里开始，length是长度信息占多少字节，这里是从0开始读4个字节表示数据包长度，1024为数据包最大长度
        .addLast(new StringDecoder())
```

```java
channel.pipeline()
        .addLast(new StringDecoder())
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println(">> 接收到客户端发送的数据：" + msg);
            }
        })
        .addLast(new LengthFieldPrepender(4))   //客户端在发送时也需要将长度拼到前面去，指定长度信息的长度即可
        .addLast(new StringEncoder());
```

有关编码器和解码器的内容就先介绍到这里。

------

## 实现HTTP协议通讯

前面我们介绍了Netty为我们提供的编码器和解码器，这里我们就来使用一下支持HTTP协议的编码器和解码器。

> new HttpRequestDecoder()，入站解析器，添加到channel的流水线处理器开头位置，用于讲接收到的http数据解析

```java
channel.pipeline()
        .addLast(new HttpRequestDecoder())   //Http请求解码器
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println("收到客户端的数据："+msg.getClass());  //看看是个啥类型
              	//收到浏览器请求后，我们需要给一个响应回去
                //HTTP版本为1.1，状态码就OK（200）即可
                FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);  
              	//直接向响应内容中写入数据
                response.content().writeCharSequence("Hello World!", StandardCharsets.UTF_8);
                ctx.channel().writeAndFlush(response);   //发送响应
                ctx.channel().close();   //HTTP请求是一次性的，所以记得关闭
            }
        })
        .addLast(new HttpResponseEncoder());   //响应也需要编码后发送哦
```

现在我们用浏览器访问一下我们的服务器吧：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213013016.png)

可以看到浏览器成功接收到服务器响应，然后控制台打印了以下类型：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213013046.png)

可以看到一次请求是一个DefaultHttpRequest+LastHttpContent$1，这里有两组是因为浏览器请求了一个地址之后紧接着请求了我们网站的favicon图标。

这样把数据分开处理肯定是不行的，要是直接整合成一个多好，安排：

~~~java
channel.pipeline()
        .addLast(new HttpRequestDecoder())   
    	.addLast(new HttpObjectAggregator(Integer.MAX_VALUE))  //搞一个聚合器，将内容聚合为一个FullHttpRequest，参数是最大内容长度
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println("收到客户端的数据："+msg.getClass());  
                FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);  
              	//直接向响应内容中写入数据
                response.content().writeCharSequence("Hello World!", StandardCharsets.UTF_8);
                ctx.channel().writeAndFlush(response);   //发送响应
                ctx.channel().close();   
            }
        })
        .addLast(new HttpResponseEncoder());   //响应需要编码后发送
~~~

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213013358.png)

用这个 FullHttpRequest 看一下URI路径：

```java
channel.pipeline()
        .addLast(new HttpRequestDecoder())   //Http请求解码器
        .addLast(new HttpObjectAggregator(Integer.MAX_VALUE))  //搞一个聚合器，将内容聚合为一个FullHttpRequest，参数是最大内容长度
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                FullHttpRequest request = (FullHttpRequest) msg;
                System.out.println("浏览器请求路径："+request.uri());  //直接获取请求相关信息
                FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
                response.content().writeCharSequence("Hello World!", StandardCharsets.UTF_8);
                ctx.channel().writeAndFlush(response);
                ctx.channel().close();
            }
        })
        .addLast(new HttpResponseEncoder());
```

再次访问，我们发现可以正常读取请求路径了：

![image-20230306174557790](https://s2.loli.net/2023/03/06/P3QRD2kHme1Vhga.png)

额外乐子：

我们来试试看搞个静态页面代理玩玩，尝试看看 TomCat 是个啥感觉，拿出我们的陈年老模板：

![image-20230306174609270](https://s2.loli.net/2023/03/06/ITE5izhrG4ZedSO.png)

全部放进Resource文件夹，一会根据浏览器的请求路径，我们就可以返回对应的页面了。

先创建一个解析器，用于解析路径然后将静态页面的内容返回：

```java
public class PageResolver {
		//直接单例模式
    private static final PageResolver INSTANCE = new PageResolver();
    private PageResolver(){}
    public static PageResolver getInstance(){
        return INSTANCE;
    }

  	//请求路径给进来，接着我们需要将页面拿到，然后转换成响应数据包发回去
    public FullHttpResponse resolveResource(String path){
        if(path.startsWith("/"))  {  //判断一下是不是正常的路径请求
            path = path.equals("/") ? "index.html" : path.substring(1);    //如果是直接请求根路径，那就默认返回index页面，否则就该返回什么路径的文件就返回什么
            try(InputStream stream = this.getClass().getClassLoader().getResourceAsStream(path)) {
                if(stream != null) {   //拿到文件输入流之后，才可以返回页面
                    byte[] bytes = new byte[stream.available()];
                    stream.read(bytes);
                    return this.packet(HttpResponseStatus.OK, bytes);  //数据先读出来，然后交给下面的方法打包
                }
            } catch (IOException e){
                e.printStackTrace();
            }
        }
      	//其他情况一律返回404
        return this.packet(HttpResponseStatus.NOT_FOUND, "404 Not Found!".getBytes());
    }

  	//包装成FullHttpResponse，把状态码和数据写进去
    private FullHttpResponse packet(HttpResponseStatus status, byte[] data){
        FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, status);
        response.content().writeBytes(data);
        return response;
    }
}
```

现在我们的静态资源解析就写好了，接着：

```java
channel.pipeline()
        .addLast(new HttpRequestDecoder())   //Http请求解码器
        .addLast(new HttpObjectAggregator(Integer.MAX_VALUE))  //搞一个聚合器，将内容聚合为一个FullHttpRequest，参数是最大内容长度
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                FullHttpRequest request = (FullHttpRequest) msg;
              	//请求进来了直接走解析
                PageResolver resolver = PageResolver.getInstance();
                ctx.channel().writeAndFlush(resolver.resolveResource(request.uri()));
                ctx.channel().close();
            }
        })
        .addLast(new HttpResponseEncoder());
```

现在我们启动服务器来试试看吧：

![image-20230306174624966](https://s2.loli.net/2023/03/06/PpFUKSMXDi5ItjN.png)

可以看到页面可以正常展示了，是不是有Tomcat哪味了。

## 其他内置Handler介绍

Netty也为我们内置了一些其他比较好用的Handler，比如日志处理：

```java
channel.pipeline()
        .addLast(new HttpRequestDecoder())
        .addLast(new HttpObjectAggregator(Integer.MAX_VALUE)) //聚合作用
        .addLast(new LoggingHandler(LogLevel.INFO))   //添加一个日志Handler，在请求到来时会自动打印相关日志
        ...
```

日志级别我们选择INFO，现在我们用浏览器访问一下：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213183751.png)

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213183731.png)

可以看到每次请求的内容和详细信息都会在日志中出现，包括详细的数据包解析过程，请求头信息都是完整地打印在控制台上的。

**我们也可以使用Handler对IP地址进行过滤**，比如我们不希望某些IP地址连接我们的服务器：

~~~java
channel.pipeline()
        .addLast(new HttpRequestDecoder())
        .addLast(new HttpObjectAggregator(Integer.MAX_VALUE))
        .addLast(new RuleBasedIpFilter(new IpFilterRule() {
            @Override
            public boolean matches(InetSocketAddress inetSocketAddress) {
                return !inetSocketAddress.getHostName().equals("127.0.0.1");  
              	//进行匹配，返回false表示匹配失败，true表示匹配成功，执行下面的类型操作（这里表示true时拒绝访问）
              	//如果匹配失败，那么会根据下面的类型决定该干什么，比如我们这里判断是不是本地访问的，如果是那就拒绝
            }
            @Override
            public IpFilterRuleType ruleType() {
                return IpFilterRuleType.REJECT;   //类型，REJECT表示拒绝连接，ACCEPT表示允许连接
            }
        }))
~~~

**我们也可以对那些长期处于空闲的连接进行处理：**

```java
channel.pipeline()
        .addLast(new StringDecoder())
   		//IdleStateHandler能够侦测连接空闲状态
        //第一个参数表示连接多少秒没有读操作时触发事件；第二个是写操作；第三个是读写操作都算，0表示禁用
        //事件需要在ChannelInboundHandlerAdapter中进行监听处理，重写userEventTriggered方法
        .addLast(new IdleStateHandler(10, 10, 0))  
    
        .addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                System.out.println("收到客户端数据："+msg);
                ctx.channel().writeAndFlush("已收到！");
            }
			
            @Override
            public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
                //没想到吧，handler中的这个方法原来是在这个时候用的，evt是事件，我们来判断一下他是不是上面的IdleStateHandler发出的。
                if(evt instanceof IdleStateEvent) {
                    IdleStateEvent event = (IdleStateEvent) evt;
                    if(event.state() == IdleState.WRITER_IDLE) {
                        System.out.println("好久都没写了，看视频的你真的有认真在跟着敲吗");
                    } else if(event.state() == IdleState.READER_IDLE) {
                        System.out.println("已经很久很久没有读事件发生了，好寂寞");
                    }
                }
            }
        })
        .addLast(new StringEncoder());
```

可以看到，当我们超过一段时间不发送数据时，就会这样：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231213213940.png)

通过这种机制，我们就可以直接关掉某些可能被废弃的连接。

## 启动流程简介
