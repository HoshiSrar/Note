# 内网穿透入门到实战

想要搭建一个可供所有用户访问的网站项目，我们必须在一台具有公网IP地址的云服务器上进行部署，而我们家中的电脑由于存在于NAT环境中，无法被外部网络主动访问，因此，搭建网站的首选就是购买云服务器。

![image-20231019161218338](https://s2.loli.net/2023/10/19/omAKyL4EeBvDfRH.png)



但是一些情况下，比如我们的业务比较消耗CPU资源，那么我们就不得不去加强云服务器的配置，而这笔费用往往对我们来说很难去承担。包括用OSS搭建个人网盘系统，其中的流量费用实在是太贵了。那有什么办法可以节省成本呢？

实际上我们发现，我们购买云服务器，主要就是图一个公网IP，而我们需要的一些计算资源、存储空间，实际上自行购买硬件放在家里反而更加经济实惠（云服务器加个硬盘容量都要很多钱）关键是家里的环境外网无法访问，那该怎么办呢？我们可以让家里的机器主动连接到云服务器，再由云服务器向我们家里的机器进行数据包转发，实现内网穿透的效果。

## 内网穿透原理图

![image-20231019155658120](https://s2.loli.net/2023/10/19/qRi6m3JhPvfMGaU.png)



显而易见，内网穿透就是依靠转发的形式实现的，这样就可以让家里的设备实现外网访问了！

内网穿透优点：

- 省钱，计算资源、存储资源可以全部自行部署
- 安全，内网环境下只要限制好端口，黑客很难入侵
- 宽带大，家用宽带一般是10M上行速度起步，办理千兆宽带甚至可以达到100M的上行速度，相比腾讯云的3M小宽带，直接真香，我们可以增加内网穿透服务器的数量并进行负载均衡来做到吃满上行带宽。

内网穿透缺点：

- 不稳定，停电、断网、电费等，都是我们需要考虑的因素
- 网络性能瓶颈，网络速度多快，以及能承受的QPS，全部受限于做内网穿透的云服务器上限

说了这么多，我们现在就来搭建一下吧，这里我们需要用到的是frp（采用Go语言编写，性能相比java更上一层楼）使用起来也非常稳定！

开源地址：https://github.com/fatedier/frp

## 服务端部署frps服务

在GitHub上下载对应的frp程序，解压，然后上传frps到服务器，执行：

```sh
chmod +x frps
./frps
```

这样就可以直接运行了：

![image-20231019165711439](https://s2.loli.net/2023/10/19/BTXfY2yRrNwJl8n.png)



接着我们来编写一下配置文件：

```sh
vim frps.toml
```

内容如下：

```properties
bindPort = 7000    #端口
auth.method = "token"   #验证方式
auth.token = "123456"   #token秘钥
```

然后我们接着将其配置为Linux服务的形式运行：

文档：https://gofrp.org/zh-cn/

```sh
sudo vim /etc/systemd/system/frps.service
```

配置文件如下：

```properties
[Unit]
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
ExecStart = /root/frps -c /root/frps.toml

[Install]
WantedBy = multi-user.target
```

然后我们就可以启动这个服务了：

```sh
systemctl enable frps
systemctl start frps
```

还有别忘了开防火墙端口，包括用于注册的7000端口以及后续给内网其他服务进行穿透的端口（如Nginx的80端口）

## 客户端部署frpc服务

跟服务端一样，直接上传frpc客户端：

```sh
chmod +x frpc
./frpc
```

只不过需要先进行配置才能运行：

![image-20231019175038881](https://s2.loli.net/2023/10/19/MOAoDteFLchb2Hv.png)



接着我们来编写一下配置文件：

```sh
vim frpc.toml
```

内容如下：

```sh
serverAddr = "47.109.97.136"   //之前搭建好的内网穿透服务器地址
serverPort = 7000      //内网穿透服务器注册端口
auth.method = "token"  //使用token验证
auth.token = "123456"  //使用密码

[[proxies]]
name = "nginx"   //配置需要代理的本地服务
type = "tcp"     //类型选择tcp即可（大部分都是）
localIP = "127.0.0.1"   //本地服务的IP地址，因为是直接部署在本地，所以说直接127.0.0.1
localPort = 80    //本地服务端口
remotePort = 80    //远程代理端口
```

配置完成后，就可以启动拉：

```sh
./frpc -c frpc.toml
```

![image-20231019180806628](https://s2.loli.net/2023/10/19/OhqUKHCSd5l4yBj.png)



为了方便，我们可以直接注册为服务：

```sh
sudo vim /etc/systemd/system/frpc.service
```

内容如下：

```sh
[Unit]
Description = frp client
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
ExecStart = /root/frpc -c /root/frpc.toml

[Install]
WantedBy = multi-user.target
```

然后我们就可以启动这个服务了：

```sh
systemctl enable frpc
systemctl start frpc
```

然后就可以试试看访问啦~

## 开启管理面板

服务端开启管理面板，可以更加有效的监控流量情况：

![image-20231019181459596](https://s2.loli.net/2023/10/19/Zob8niGWe6Rfvum.png)



在服务端配置文件中添加以下内容：

```sh
webServer.port = 7500   #管理面板端口
webServer.addr = "0.0.0.0"   #绑定到所有网络上，外网访问
webServer.user = "admin"  #管理员名称
webServer.password = "admin"  #管理员密码
```

OK，完成！
