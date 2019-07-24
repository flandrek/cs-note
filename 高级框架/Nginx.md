### 简介

Nginx 是一个十分轻量级的 HTTP 服务器，是一个高性能的 HTTP 和反向代理服务器，同时也是一个 IMTP / POP3 / STMP 邮件代理服务器。

### 优点

**1、可以高并发连接**

​          官方测试Nginx能够支撑5万并发连接，实际生产环境中可以支撑2~4万并发连接数。

​          原因主要是Nginx使用了最新的epoll（Linux2.6内核）和kqueue（freeBSD）网路I/O模型，而Apache使用的是传统的Select模型，其比较稳定的Prefork模式为多进程模式，需要经常派生子进程，所以消耗的CPU等服务器资源，要比Nginx高很多。

**2、内存消耗少**

​          Nginx+PHP（FastCGI）服务器，在3万并发连接下，开启10个Nginx进程消耗150MB内存，15MB*10=150MB，开启的64个PHP-CGI进程消耗1280内存，20MB*64=1280MB，加上系统自身消耗的内存，总共消耗不到2GB的内存。

​          如果服务器的内存比较小，完全可以只开启25个PHP-CGI进程，这样PHP-CGI消耗的总内存数才500MB。 

**3、成本低廉**

​          购买F5BIG-IP、NetScaler等硬件负载均衡交换机，需要十多万到几十万人民币，而Nginx为开源软件，采用的是2-clause BSD-like协议，可以免费试用，并且可用于商业用途。

​          BSD开源协议是一个给使用者很大自由的协议，协议指出可以自由使用、修改源代码、也可以将修改后的代码作为开源或专用软件再发布。

**4、配置文件非常简单**

​          网络和程序一样通俗易懂，即使，非专用系统管理员也能看懂。

**5、支持Rewrite重写**

​          能够根据域名、URL的不同，将http请求分到不同的后端服务器群组。

**6、内置的健康检查功能**

​          如果NginxProxy后端的某台Web服务器宕机了，不会影响前端的访问。

**7、节省带宽**

​          支持GZIP压缩，可以添加浏览器本地缓存的Header头。

**8、稳定性高**

​          用于反向代理，宕机的概率微乎其微。

**9、支持热部署**

​          Nginx支持热部署，它的自动特别容易，并且，几乎可以7天*24小时不间断的运行，即使，运行数个月也不需要重新启动，还能够在不间断服务的情况下，对软件版本进行升级。



### 缺点

1、nginx 仅支持 http，https，Email 协议，相对于 LVS 适用范围较小；

2、nginx 健康检查只支持通过端口来检测，不支持通过 URL 来检测；

3、nginx 不支持 session 保持，但可以通过 ip_hash 来解决；

4、处理动态页面使不如更稳定的 apache，tomcat 等。



### 基本特性

1、处理静态文件，索引文件以及自动索引，打开文件描述符缓冲；

2、无缓存的反向代理加速，简单的负载均衡和容错；

3、FastCGI，简单的负载均衡和容错；

4、模块化的结构。包括 gzipping，byte ranges，chunked responses，以及 SSI-filter 等 filter。如果由 FastCGI 或其它代理服务器处理单页中存在的多个 SSI，则这项处理可以并行运行，而不需要相互等待；

5、支持 SSL 和 TLSSNI．



### Nginx 高效的原因

#### 原因一：IO多路复用epoll

什么是IO复用？



![img](https:////upload-images.jianshu.io/upload_images/3245377-4550bc6f87b41351.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



多个描述符的 I/O 操作都能在一个线程内并发交替地顺序完成，这就叫 I/O 多路复用，这里的"复用"指的是复用同一个线程。
 具体详细请查看文章：
 [https://segmentfault.com/a/1190000003063859](https://link.jianshu.com?t=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000003063859)
 [https://segmentfault.com/a/1190000004909797](https://link.jianshu.com?t=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000004909797)

#### 原因二：轻量级

**功能模块少：**源代码里只有核心代码，其他代码以插件形式安装；
**代码模块化：** 适合二次改进；

#### 原因三：CPU亲和-affinity

nginx 正是利用到了 cpu 的亲和来提高并发处理能力以及减少不必要的 cpu 损耗。

1.什么是 CPU 亲和：
 是一种把 CPU 核心和 Nginx 工作进程绑定方式，把每个 worker 进程固定在一个 cpu 上执行，减少 cpu 的 cache miss，获得更好的性能。
 2.为什么需要 CPU 亲和：



![img](https:////upload-images.jianshu.io/upload_images/3245377-10c4d492dfe64546.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



nginx 作为接入层的中间件，nginx 通过多个 work 进程进行处理。
 假设我们主机是两个 CPU，每个有四个核心，我们把 CPU 的八个进程分别绑定到不同的 CPU 上（也就是不同的work 分配到不同的核心上）。如果有多个 CPU 利用自带的 CPU 切换，会造成性能损失。利用这种 CPU 的亲和绑定，就能减少切换的损耗。

#### 原因四：sendfile

 nginx 采用 sendfile 机制处理静态文件，因此效率很高。



![img](https:////upload-images.jianshu.io/upload_images/3245377-9fe1d17602b4fdfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



上图是传统的 http 服务，当我们访问一个文件时，会先经过内核空间，再经过用户空间，传给 socket，最后通过response 返回给用户。该过程需要多次与用户空间进行切换，但是静态文件其实不需要与用户空间进行过多的逻辑处理，直接可以通过内核空间传输。



![img](https:////upload-images.jianshu.io/upload_images/3245377-8ab002e729a328e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



sendfile 机制只通过内核空间，将文件传给 socket，最终响应给用户。
因此 nginx 在处理 CDN 和动静分离服务时有很大优势。

附：
 什么是内核空间和用户空间：[https://blog.adolphlwq.xyz/user-and-kernel-space/](https://link.jianshu.com?t=https%3A%2F%2Fblog.adolphlwq.xyz%2Fuser-and-kernel-space%2F)
 什么是 CPU 亲和力：<https://www.cnblogs.com/wenqiang/p/6049978.html>
 什么是 Linux 的 cache miss：[https://www.ibm.com/developerworks/community/blogs/5144904d-5d75-45ed-9d2b-cf1754ee936a/entry/perf_introduction?lang=en](https://link.jianshu.com?t=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcommunity%2Fblogs%2F5144904d-5d75-45ed-9d2b-cf1754ee936a%2Fentry%2Fperf_introduction%3Flang%3Den)
 什么是零拷贝：<https://www.jianshu.com/p/fad3339e3448>





### 代理服务器-Proxy Serve

​    提供代理服务的电脑系统或其它类型的网络终端，代替网络用户去取得网络信息。

#### 为什么使用代理服务器？

1. **提高访问速度** 
       由于目标主机返回的数据会存放在代理服务器的硬盘中，因此下一次客户再访问相同的站点数据时，会直接从代理服务器的硬盘中读取，起到了缓存的作用，尤其对于热门网站能明显提高访问速度。

2. **防火墙作用** 
       由于所有的客户机请求都必须通过代理服务器访问远程站点，因此可以在代理服务器上设限，过滤掉某些不安全信息。同时正向代理中上网者可以隐藏自己的IP,免受攻击。

3. **突破访问限制** 
       互联网上有许多开发的代理服务器，客户机在访问受限时，可通过不受限的代理服务器访问目标站点，通俗说，我们使用的翻墙浏览器就是利用了代理服务器，可以直接访问外网。

#### 正向代理

​     正向代理（forward proxy） ，一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并制定目标（原始服务器），然后代理向原始服务器转发请求并将获得的内容返回给客户端，客户端才能使用正向代理。我们平时说的代理就是指正向代理。 
​    简单一点：A向C借钱，由于一些情况不能直接向C借钱，于是A想了一个办法，他让B去向C借钱，这样B就代替A向C借钱，A就得到了C的钱，C并不知道A的存在，B就充当了A的代理人的角色。 

![è¿éåå¾çæè¿°](图片/20171231131324208.png)



#### 反向代理

​    反向代理（Reverse Proxy），以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求的客户端，此时代理服务器对外表现为一个反向代理服务器。 
​    理解起来有些抽象，可以这么说：A向B借钱，B没有拿自己的钱，而是悄悄地向C借钱，拿到钱之后再交给A,A以为是B的钱，他并不知道C的存在。 

#### 正向代理和反向代理的区别

1. **位置不同** 
   正向代理，架设在客户机和目标主机之间； 反向代理，架设在服务器端；
2. **代理对象不同** 
   正向代理，代理客户端，服务端不知道实际发起请求的客户端； 
   反向代理，代理服务端，客户端不知道实际提供服务的服务端； 

例：正向代理 –HTTP代理为多个人提供翻墙服务；

​	反向代理–百度外卖为多个商户提供平台给某个用户提供外卖服务。

3. **用途不同** 
   正向代理，为在防火墙内的局域网客户端提供访问 Internet 的途径； 
   反向代理，将防火墙后面的服务器提供给 Internet 访问；
4. **安全性不同** 
   正向代理允许客户端通过它访问任意网站并且隐藏客户端自身，因此必须采取安全措施以确保仅为授权的客户端提供服务； 
   反向代理都对外都是透明的，访问者并不知道自己访问的是哪一个代理。
5. **正向代理的应用**
          1. 访问原来无法访问的资源 ；
          2. 用作缓存，加速访问速度 ；
          3. 对客户端访问授权，上网进行认证 ；
          4. 代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息；

6. **反向代理的应用**
       1. 保护内网安全 ；
       2. 负载均衡 ；
       3. 缓存，减少服务器的压力 ；
          

#### 总结

​    正向代理是从客户端的角度出发，服务于特定用户（比如说一个局域网内的客户）以访问非特定的服务；反向代理正好与此相反，从服务端的角度出发，服务于非特定用户（通常是所有用户），已访问特定的服务。 

