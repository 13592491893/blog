---
abbrlink: 16094
title: 微服务中远程调用Dubbo与Feign对比
comments: true
toc: true
description: 微服务中远程调用Dubbo与Feign对比
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - dubbo
  - feign
  - 远程调用
date: 2020-3-21 16:00:00
---
# [微服务中远程调用Dubbo与Feign对比](https://www.cnblogs.com/ying-z/p/14781757.html)

## 一、实现远程调用的方式

Http接口（web接口、RestTemplate+Okhttp）、Feign、RPC调用（Dubbo、Socket编程）、Webservice。

 

## 二、什么是Feign？

Feign是Spring Cloud提供的一个声明式的伪Http客户端，它使得调用远程服务就像调用本地服务一样简单，只需要创建一个接口并添加一个注解即可。

Nacos注册中心很好的兼容了Feign，Feign默认集成了Ribbon，所以在Nacos下使用Fegin默认就实现了负载均衡的效果。

 

## 三、什么是Dubbo？

Dubbo是阿里巴巴开源的基于Java的高性能RPC分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。

Spring-cloud-alibaba-dubbo是基于SpringCloudAlibaba技术栈对dubbo技术的一种封装，目的在于实现基于RPC的服务调用。

 

## 四、Feign与Dubbo的对比

Feign与Dubbo功能上有很多类似的地方，因为都是专注于远程调用这个动作。比如注册中心解耦、负载均衡、失败重试熔断、链路监控等。

Dubbo除了注册中心需要进行整合，其它功能都自己实现了，而Feign大部分功能都是依赖全家桶的组件来实现的。Dubbo小而专一，专注于远程调用。而Spring全家桶而言，远程调用只是一个重要的功能而已。

## 五、协议支持方面：

Feign更加优雅简单。Feign是通过REST API实现的远程调用，基于Http传输协议，服务提供者需要对外暴露Http接口供消费者调用，服务粒度是http接口级的。通过短连接的方式进行通信，不适合高并发的访问。Feign追求的是简洁，少侵入（因为就服务端而言，在SpringCloud环境下，不需要做任何额外的操作，而Dubbo的服务端需要配置开放的Dubbo接口)。

Dubbo方式更灵活。Dubbo是通过RPC调用实现的远程调用，支持多传输协议(Dubbo、Rmi、http、redis等等)，可以根据业务场景选择最佳的方式，非常灵活。默认的Dubbo协议：利用Netty，TCP传输，单一、异步、长连接，适合数据量小、高并发和服务提供者远远少于消费者的场景。Dubbo通过TCP长连接的方式进行通信，服务粒度是方法级的。

从协议层选择看，Dubbo是配置化的，更加灵活。Dubbo协议更适合小数据高并发场景。

 

## 六、通信性能方面：

从通信的性能上来分析，SpringCloud的通信采用Openfeign（feign）组件。

Feign基于Http传输协议，底层实现是rest。从OSI 7层模型上来看rest属于应用层。

在高并发场景下性能不够理想，成为性能瓶颈（虽然他是基于Ribbon以及带有熔断机制可以防止雪崩），需要改造。具体需要改造的内容需要时再研究。

Dubbo框架的通信协议采用RPC协议，属于传输层协议，性能上自然比rest高。提升了交互的性能，保持了长连接，高性能。

Dubbo性能更好，比如支持异步调用、Netty性能更好。Dubbo主要是配置而无需改造。

|             | RPC                            | REST            |
| ----------- | ------------------------------ | --------------- |
| 耦合性      | 强耦合                         | 松耦合          |
| 消息协议    | 二进制 thrift/protobuf         | 文本 xml、jason |
| 通信协议    | TCP                            | HTTP            |
| 接口契约IDL | thrift/protobuf                | swagger         |
| 开发调试    | 消息不可读                     | 可读，可调试    |
| 对外开放    | 一般作为内部各个系统的通信框架 | 对接外部系统    |

使用SpringCloud整合Dubbo，正所谓是强强联合。

 

## 七、负载均衡方面：

Feign默认使用Ribbon作为负载均衡的组件。

Dubbo和Ribbon（Feign默认集成Ribbon）都支持负载均衡策略，但是Dubbo支持的更灵活。

Dubbo和Ribbon对比：

Ribbon的负载均衡策略：随机、规则轮询、空闲策略、响应时间策略。

Dubbo的负载均衡策略：Dubbo支持4种算法，随机、权重轮询、最少活跃调用数、一致性Hash策略。而且算法里面引入权重的概念。

Dubbo可以使用路由策略，然后再进行负载均衡。

Dubbo配置的形式不仅支持代码配置，还支持Dubbo控制台灵活动态配置。

Dubbo负载均衡的算法可以精准到某个服务接口的某个方法，而Ribbon的算法是Client级别的。Ribbon需要进行全局配置，个性化配置比较麻烦。

## 八、容错机制方面：

Feign默认使用Hystix作为服务熔断的组件。Hystix提供了服务降级，服务熔断，依赖隔离，监控（Hystrix Dashboard）等功能。Feign是利用熔断机制来实现容错的，与Dubbo处理的方式不一样。

Dubbo支持多种容错策略，FailOver、FailFast、Failsafe、FailBack、Aviailable、Broadcast、Forking策略等，以及Mock。也引入了retry次数，timeout等配置参数。Dubbo自带了失败重试的功能。

## 九、其他方面（以下方便并未进行详细整理仅做参考）：

Dubbo附带了白名单功能、结果缓存、同步和异步调用的功能。

Dubbo支持更多更灵活的并发控制：

客户端配置actives参数，配置单个Cunsumer最大并发请求数，超出则线程阻塞等待，超时报错。

Provider可以配置executes参数来限制最大的并发线程数，超出报错。

Provider可以配置accepts参数来限制最大长连接数来限制最大的连接数。

Provider的通过配置任务线程池的类型和最大线程数来控制并发量，超负载直接丢弃。

路由、流量调度、ABtest方面：

Ribbon需自己实现，应用不灵活。

Ribbon主要通过扩展AbstractLoadBalancerRule负载均衡的方法来实现，在负载均衡的部分还要进行改造升级。

Dubbo更加灵活方便。

Dubbo通过界面化、校本化配置路由规则，可以实现灰度发布、动态流量调度、容量计算等，方案成熟。

另外，Dubbo 还支持多版本调用。

Dubbo支持更完善的监控和管理界面，SC也有Actuator等工具进行监控，但是并不是针对远程调用这一块的

Dubbo支持客户端设置调用结果缓存，支持配置3种策略的结果缓存(LRU、LFU、FIO)，但是要自己实现超时管理。

 

## 十、总结

Dubbo支持更多功能、更灵活、支持高并发的RPC框架。

SpringCloud全家桶里面（Feign、Ribbon、Hystrix），特点是非常方便。Ribbon、Hystrix、Feign在服务治理中，配合Spring Cloud做微服务，使用上有很多优势，社区也比较活跃，看将来更新发展。

业务发展影响着架构的选型，当服务数量不是很大时，使用普通的分布式RPC架构即可，当服务数量增长到一定数据，需要进行服务治理时，就需要考虑使用流式计算架构。Dubbo可以方便的做更精细化的流量调度，服务结构治理的方案成熟，适合生产上使用，虽然Dubbo是尘封后重新开启，但这并不影响其技术价值。

如果项目对性能要求不是很严格，可以选择使用Feign，它使用起来更方便。

如果需要提高性能，避开基于Http方式的性能瓶颈，可以使用Dubbo。

Dubbo Spring Cloud的出现，使得Dubbo既能够完全整合到Spring Cloud的技术栈中，享受Spring Cloud生态中的技术支持和标准化输出，又能够弥补Spring Cloud中服务治理这方面的短板。

## 十一、Example

### 1.openfeign

provider：

![image-20210811175918261](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811175918261.png)

![image-20210811175933040](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811175933040.png)

consumer：

![image-20210811180017625](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811180017625.png)

![image-20210811180108994](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811180108994.png)

![image-20210811180136385](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811180136385.png)

![image-20210811180156161](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811180156161.png)

### 2.dubbo

1. 建一个接口模块，POM文件引入zookeeper，不需要启动类![image-20210811181745545](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811181745545.png)

2. 建服务提供方，service目录与api里的保持一致![image-20210811181915053](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811181915053.png)![image-20210811182038175](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210811182038175.png)

3. 建消费方

   ![image-20210811182126150](https://gitee.com/gsshy/picgo/raw/master/img/image-20210811182126150.png)

## 十二、[HttpClient、RestTemplate和Feign相关知识](https://www.cnblogs.com/lushichao/p/12796408.html)

先了解一下HTTP 协议

**史前时期**

　　 HTTP 协议在我们的生活中随处可见，打开手机或者电脑，只要你上网，不论是用 iPhone、Android、Windows 还是 Mac，不论是用浏览器还是 App，不论是看新闻、短视频还是听音乐、玩游戏，后面总会有 HTTP 在默默为你服务。

　　据 NetCraft 公司统计，目前全球至少有 16 亿个网站、2 亿多个独立域名，而这个庞大网络世界的底层运转机制就是 HTTP。

　　那么，在享受如此便捷舒适的网络生活时，你有没有想过，HTTP 协议是怎么来的？它最开始是什么样子的？又是如何一步一步发展到今天，几乎“统治”了整个互联网世界的呢？

　　20 世纪 60 年代，美国国防部高等研究计划署（ARPA）建立了 ARPA 网，它有四个分布在各地的节点，被认为是如今互联网的“始祖”。

　　然后在 70 年代，基于对 ARPA 网的实践和思考，研究人员发明出了著名的 TCP/IP 协议。由于具有良好的分层结构和稳定的性能，TCP/IP 协议迅速战胜其他竞争对手流行起来，并在 80 年代中期进入了 UNIX 系统内核，促使更多的计算机接入了互联网。

**创世纪**

　　1989 年，任职于欧洲核子研究中心（CERN）的蒂姆·伯纳斯 - 李（Tim Berners-Lee）发表了一篇论文，提出了在互联网上构建超链接文档系统的构想。这篇论文中他确立了三项关键技术。

- URI：即统一资源标识符，作为互联网上资源的唯一身份；
- HTML：即超文本标记语言，描述超文本文档；
- HTTP：即超文本传输协议，用来传输超文本。

　　所以在这一年，我们的HTTP诞生了。

**HTTP/0.9**

　　20 世纪 90 年代初期的互联网世界非常简陋，计算机处理能力低，存储容量小，网速很慢，还是一片“信息荒漠”。网络上绝大多数的资源都是纯文本，很多通信协议也都使用纯文本，所以 HTTP 的设计也不可避免地受到了时代的限制。

　　这一时期的 HTTP 被定义为 0.9 版，结构比较简单，为了便于服务器和客户端处理，它也采用了纯文本格式。蒂姆·伯纳斯 - 李最初设想的系统里的文档都是只读的，所以只允许用“GET”动作从服务器上获取 HTML 文档，并且在响应请求之后立即关闭连接，功能非常有限。

　　HTTP/0.9 虽然很简单，但它作为一个“原型”，充分验证了 Web 服务的可行性，而“简单”也正是它的优点，蕴含了进化和扩展的可能性，因为：

　　**“把简单的系统变复杂”，要比“把复杂的系统变简单”容易得多。**

**HTTP/1.0**

　　1993 年，NCSA（美国国家超级计算应用中心）开发出了 Mosaic，是第一个可以图文混排的浏览器，随后又在 1995 年开发出了服务器软件 Apache，简化了 HTTP 服务器的搭建工作。

　　同一时期，计算机多媒体技术也有了新的发展：1992 年发明了 JPEG 图像格式，1995 年发明了 MP3 音乐格式。

　　这些新软件新技术一经推出立刻就吸引了广大网民的热情，更的多的人开始使用互联网，研究 HTTP 并提出改进意见，甚至实验性地往协议里添加各种特性，从用户需求的角度促进了 HTTP 的发展。

　　于是在这些已有实践的基础上，经过一系列的草案，HTTP/1.0 版本在 1996 年正式发布。它在多方面增强了 0.9 版，形式上已经和我们现在的 HTTP 差别不大了，例如：

- 增加了 HEAD、POST 等新方法
- 增加了响应状态码，标记可能的错误原因
- 引入了协议版本号概念
- 引入了 HTTP Header（头部）的概念，让 HTTP 处理请求和响应更加灵活
- 传输的数据不再仅限于文本

**HTTP/1.1**

　　1995 年，网景的 Netscape Navigator 和微软的 Internet Explorer 开始了著名的“浏览器大战”，都希望在互联网上占据主导地位。于是在“浏览器大战”结束之后的 1999 年，HTTP/1.1 发布了 RFC 文档，编号为 2616，正式确立了延续十余年的传奇。

　　HTTP/1.1 主要的变更点有：

- 增加了 PUT、DELETE 等新的方法；
- 增加了缓存管理和控制；
- 明确了连接管理，允许持久连接；
- 允许响应数据分块（chunked），利于传输大文件；
- 强制要求 Host 头，让互联网主机托管成为可能。

**HTTP/2**

　　HTTP/1.1 发布之后，整个互联网世界呈现出了爆发式的增长，度过了十多年的“快乐时光”，更涌现出了 Facebook、Twitter、淘宝、京东等互联网新贵。

　　这期间也出现了一些对 HTTP 不满的意见，主要就是连接慢，无法跟上迅猛发展的互联网，但 HTTP/1.1 标准一直“岿然不动”，无奈之下人们只好发明各式各样的“小花招”来缓解这些问题，比如以前常见的切图、JS 合并等网页优化手段。

　　终于有一天，搜索巨头 Google 忍不住了，首先开发了自己的浏览器 Chrome，然后推出了新的 SPDY 协议，并在 Chrome 里应用于自家的服务器，如同十多年前的网景与微软一样，从实际的用户方来“倒逼”HTTP 协议的变革，这也开启了第二次的“浏览器大战”。

　　历史再次重演，不过这次的胜利者是 Google，Chrome 目前的全球的占有率超过了 60%。Google 借此顺势把 SPDY 推上了标准的宝座，互联网标准化组织以 SPDY 为基础开始制定新版本的 HTTP 协议，最终在 2015 年发布了 HTTP/2，RFC 编号 7540。

```
SPDY（读作“SPeeDY”）是Google开发的基于TCP的会话层协议，用以最小化网络延迟，提升网络速度，优化用户的网络使用体验。
SPDY并不是一种用于替代HTTP的协议，而是对HTTP协议的增强。新协议的功能包括数据流的多路复用、请求优先级以及HTTP报头压缩。
谷歌表示，引入SPDY协议后，在实验室测试中页面加载速度比原先快64%。
```

　　HTTP/2 的制定充分考虑了现今互联网的现状：宽带、移动、不安全，在高度兼容 HTTP/1.1 的同时在性能改善方面做了很大努力，主要的特点有：

- 二进制协议，不再是纯文本
- 可发起多个请求，废弃了 1.1 里的管道
- 使用专用算法压缩头部，减少数据传输量
- 允许服务器主动向客户端推送数据
- 增强了安全性，“事实上”要求加密通信

　　虽然 HTTP/2 到今天已经五岁，也衍生出了 gRPC 等新协议，但由于 HTTP/1.1 实在是太过经典和强势，目前它的普及率还比较低，大多数网站使用的仍然还是 20 年前的 HTTP/1.1。

**HTTP/3**

　　在 HTTP/2 还处于草案之时，Google 又发明了一个新的协议，叫做 QUIC，而且还是相同的“套路”，继续在 Chrome 和自家服务器里试验着“玩”，依托它的庞大用户量和数据量，持续地推动 QUIC 协议成为互联网上的“既成事实”。

　　2018 年，互联网标准化组织 IETF 提议将“HTTP over QUIC”更名为“HTTP/3”并获得批准，HTTP/3 正式进入了标准化制订阶段，也许两三年后就会正式发布，到时候我们很可能会跳过 HTTP/2 直接进入 HTTP/3。

```
QUIC（Quick UDP Internet Connection）是谷歌制定的一种基于UDP的低时延的互联网传输层协议。
在2016年11月国际互联网工程任务组(IETF)召开了第一次QUIC工作组会议，受到了业界的广泛关注。
这也意味着QUIC开始了它的标准化过程，成为新一代传输层协议
```

**了解这么多，那到底HTTP是什么呢？**

　　你可能会不假思索、脱口而出：“HTTP 就是超文本传输协议，也就是 HyperText Transfer Protocol。”

回答的也没错，但是太过简单。更准确的回答应该是“HTTP 是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范”

 

 **关于HttpClient**

**简介**　　

　　官网这样说

```
超文本传输协议（HTTP）可能是当今Internet上使用的最重要的协议。Web服务，支持网络的设备和网络计算的增长继续将HTTP协议的作用扩展到用户驱动的Web浏览器之外，同时增加了需要HTTP支持的应用程序的数量。

尽管java.net软件包提供了用于通过HTTP访问资源的基本功能，但它并未提供许多应用程序所需的全部灵活性或功能。HttpClient试图通过提供高效，最新且功能丰富的程序包来实现此空白，以实现最新HTTP标准和建议的客户端。

HttpClient是为扩展而设计的，同时提供了对基本HTTP协议的强大支持，对于构建HTTP感知的客户端应用程序（例如Web浏览器，Web服务客户端或利用或扩展HTTP协议进行分布式通信的系统）的任何人来说，HttpClient都可能会感兴趣。
```

　　HttpClient 是 Apache Jakarta Common 下的子项目，用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。HttpClient 已经应用在很多的项目中，比如 Apache Jakarta 上很著名的另外两个开源项目 Cactus 和 HTMLUnit 都使用了 HttpClient。

　　HttpClient 相比传统 JDK 自带的`URLConnection`，增加了易用性和灵活性，它不仅是客户端发送 HTTP 请求变得容易，而且也方便了开发人员测试接口（基于 HTTP 协议的），即提高了开发的效率，也方便提高代码的健壮性。因此熟练掌握 HttpClient 是很重要的必修内容，掌握 HttpClient 后，相信对于 HTTP 协议的了解会更加深入。

　　了解更新详情，参考官网http://hc.apache.org/index.html

**特性**

- 基于标准、纯净的 Java 语言。实现了 HTTP 1.0 和 HTTP 1.1
- 以可扩展的面向对象的结构实现了 HTTP 全部的方法（GET, POST, PUT, DELETE, HEAD, OPTIONS, and TRACE）。
- 支持 HTTPS 协议。
- 通过 HTTP 代理建立透明的连接。
- 利用 CONNECT 方法通过 HTTP 代理建立隧道的 HTTPS 连接。
- Basic, Digest, NTLMv1, NTLMv2, NTLM2 Session, SNPNEGO/Kerberos 认证方案。
- 插件式的自定义认证方案。
- 便携可靠的套接字工厂使它更容易的使用第三方解决方案。
- 连接管理器支持多线程应用。支持设置最大连接数，同时支持设置每个主机的最大连接数，发现并关闭过期的连接。
- 自动处理 Set-Cookie 中的 Cookie。
- 插件式的自定义 Cookie 策略。
- Request 的输出流可以避免流中内容直接缓冲到 Socket 服务器。
- Response 的输入流可以有效的从 Socket 服务器直接读取相应内容。
- 在 HTTP 1.0 和 HTTP 1.1 中利用 KeepAlive 保持持久连接。
- 直接获取服务器发送的 response code 和 headers。
- 设置连接超时的能力。
- 实验性的支持 HTTP 1.1 response caching。
- 源代码基于 Apache License 可免费获取。

**使用流程**

1. 创建 `HttpClient` 对象
2. 创建请求方法的实例，并指定请求 URL。如果需要发送 GET 请求，创建 `HttpGet` 对象；如果需要发送 POST 请求，创建 `HttpPost` 对象
3. 如果需要发送请求参数，可调用 `HttpGet`、`HttpPost` 共同的 `setParams(HttpParams params)` 方法来添加请求参数；对于 `HttpPost` 对象而言，也可调用 `setEntity(HttpEntity entity)` 方法来设置请求参数
4. 调用 `HttpClient` 对象的 `execute(HttpUriRequest request)` 发送请求，该方法返回一个 `HttpResponse`
5. 调用 `HttpResponse` 的 `getAllHeaders()`、`getHeaders(String name)` 等方法可获取服务器的响应头
6. 调用 `HttpResponse` 的 `getEntity()` 方法可获取 `HttpEntity` 对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容
7. 释放连接。无论执行方法是否成功，都必须释放连接

**使用用例**

　　pom配置

```
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.12</version>
        </dependency>
```

　　创建Get请求

```
　　
@Test
void testGet() {
    // 创建 HttpClient 客户端
    CloseableHttpClient httpClient = HttpClients.createDefault();
    // 创建 HttpGet 请求
    HttpGet httpGet = new HttpGet("http://192.168.1.250:15005/dsm-ubm/base-role-info/list");
    // 设置长连接
    httpGet.setHeader("Connection", "keep-alive");
    // 设置认证信息
    httpGet.setHeader("Authorization", "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZXN0IiwiY3JlYXRlZCI6MTU4ODA2NjM1NTMxNiwiZXhwIjoxNTg4NjcxMTU1fQ.mfroxGQMf_QbHGViEBhQ0hzHoxdNM0TwpGWT64t3LPUl8Sn_ZSBFFKUAt0aKkywM3Lq8245LSXu6BYOptVwYZg");
    // 设置代理（模拟浏览器版本）
    httpGet.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36");

    CloseableHttpResponse httpResponse = null;
    try {
        // 请求并获得响应结果
        httpResponse = httpClient.execute(httpGet);
        HttpEntity httpEntity = httpResponse.getEntity();
        //打印结果
        System.out.println(EntityUtils.toString(httpEntity));
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (httpResponse != null) {
            try {
                httpResponse.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        if (httpClient != null) {
            try {
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

 　keep-alive说明

```
keep-alive：从HTTP/1.1起，浏览器默认都开启了Keep-Alive，保持连接特性，客户端和服务器都能选择随时关闭连接，则请求头中为connection:close。
简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的TCP连接。
但是Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。
```

　　创建Post请求，content-type=application/json

```
@Test
    void testPost() {
        // 创建 HttpClient 客户端
        CloseableHttpClient httpClient = HttpClients.createDefault();
        // 创建 HttpPost 请求
        HttpPost httpPost = new HttpPost("http://192.168.1.250:15005/dsm-ubm/api/getInAreaPatientList");
        // 设置长连接
        httpPost.setHeader("Connection", "keep-alive");
        // 设置认证信息
        httpPost.setHeader("Authorization", "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZXN0IiwiY3JlYXRlZCI6MTU4ODA2NjM1NTMxNiwiZXhwIjoxNTg4NjcxMTU1fQ.mfroxGQMf_QbHGViEBhQ0hzHoxdNM0TwpGWT64t3LPUl8Sn_ZSBFFKUAt0aKkywM3Lq8245LSXu6BYOptVwYZg");
        // 设置代理（模拟浏览器版本）
        httpPost.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36");

        // 创建 HttpPost 参数
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("FromTime", "");
        jsonObject.put("Ward", "4008");

        CloseableHttpResponse httpResponse = null;
        try {
            StringEntity entity = new StringEntity(jsonObject.toJSONString(), "utf-8");
            entity.setContentType("application/json");
            httpPost.setEntity(entity);
            httpResponse = httpClient.execute(httpPost);
            HttpEntity httpEntity = httpResponse.getEntity();
            //打印结果
            System.out.println(EntityUtils.toString(httpEntity));
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (httpResponse != null) {
                    httpResponse.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                if (httpClient != null) {
                    httpClient.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

 　创建Post请求，content-type=application/x-www-form-urlencoded

```
@Test
    void testPost() {
        // 创建 HttpClient 客户端
        CloseableHttpClient httpClient = HttpClients.createDefault();
        // 创建 HttpPost 请求
        HttpPost httpPost = new HttpPost("http://localhost:8080/hello");
        // 设置长连接
        httpPost.setHeader("Connection", "keep-alive");
        // 设置认证信息
        httpPost.setHeader("Authorization", "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ0ZXN0IiwiY3JlYXRlZCI6MTU4ODA2NjM1NTMxNiwiZXhwIjoxNTg4NjcxMTU1fQ.mfroxGQMf_QbHGViEBhQ0hzHoxdNM0TwpGWT64t3LPUl8Sn_ZSBFFKUAt0aKkywM3Lq8245LSXu6BYOptVwYZg");
        // 设置代理（模拟浏览器版本）
        httpPost.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36");

        // 创建 HttpPost 参数
        List<BasicNameValuePair> params = new ArrayList<>();
        params.add(new BasicNameValuePair("message", "鹧鸪哨"));

        CloseableHttpResponse httpResponse = null;
        try {
            httpPost.setEntity(new UrlEncodedFormEntity(params, "utf-8"));
            httpResponse = httpClient.execute(httpPost);
            HttpEntity httpEntity = httpResponse.getEntity();
            //打印结果
            System.out.println(EntityUtils.toString(httpEntity));
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (httpResponse != null) {
                    httpResponse.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                if (httpClient != null) {
                    httpClient.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

 

**application/json与application/x-www-form-urlencoded区别**

　　application/json和application/x-www-form-urlencoded都是表单数据发送时的编码类型。默认地，表单数据会编码为application/x-www-form-urlencoded。就是说，在发送到服务器之前，所有字符都会进行编码。

　　application/json，随着json规范的越来越流行，并且浏览器支持程度原来越好，许多开发人员将application/json作为请求content-type，告诉服务器请求的主体内容是json格式的字符串，服务器端会对json字符串进行解析，这种方式的好处就是前端人员不需要关心数据结构的复杂度，只要是标准的json格式就能提交成功，需要封装成对象的话，可以加上@RequestBody注解

　　application/x-www-form-urlencoded是Jquery的Ajax请求默认方式，这种方式的好处就是浏览器都支持，在请求发送过程中会对数据进行序列化处理，以键值对形式，数据拼接方式为key=value的方式，后台如果使用对象接收的话，可以自动封装成对象

**@RequestBody和@RequestParam区别**

@RequestParam

1. 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容。（Http协议中，如果不指定Content-Type，则默认传递的参数就是application/x-www-form-urlencoded类型）。
2. 在Content-Type: application/x-www-form-urlencoded的请求中， get 方式中queryString的值，和post方式中 body data的值都会被Servlet接受到并转化到Request.getParameter()参数集中，所以@RequestParam可以获取的到。

@RequestBody

1. 处理HttpEntity传递过来的数据，一般用来处理非Content-Type: application/x-www-form-urlencoded编码格式的数据。
2. GET请求中，因为没有HttpEntity，所以@RequestBody并不适用。
3. POST请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型Content-Type，SpringMVC通过使用HandlerAdapter 配置的HttpMessageConverters来解析HttpEntity中的数据，然后绑定到相应的bean上。

 

**@ResponseBody 和 @RequestBody 区别**

　　@ResponseBody是作用在方法上的，@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，一般在异步获取数据时使用【也就是AJAX】，在使用 @RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody 后返回结果不会被解析为跳转路径，而是直接写入 HTTP response body 中。 比如异步获取 json 数据，加上 @ResponseBody 后，会直接返回 json 数据。

　　@RequestBody 用于读取Request请求的body部分数据，使系统默认的HttpMessageConverter进行解析，然后把相应的数据绑定要返回的对象上，再把HttpMessageConverter返回的对象数据绑定到Controller方法的参数上。

 **#关于RestTemplate**

**介绍**

　　RestTemplate 是从 Spring3.0 开始支持的一个 HTTP 请求工具，它提供了常见的REST请求方案的模版，例如 GET 请求、POST 请求、PUT 请求、DELETE 请求以及一些通用的请求执行方法 exchange 以及 execute。RestTemplate 继承自 InterceptingHttpAccessor 并且实现了 RestOperations 接口，其中 RestOperations 接口定义了基本的 RESTful 操作，这些操作在 RestTemplate 中都得到了实现。是比httpClient更优雅的Restful URL访问。

- RestTemplate是Spring提供的用于访问Rest服务的客户端，
- RestTemplate提供了多种便捷访问远程Http服务的方法,能够大大提高客户端的编写效率。
- 调用RestTemplate的默认构造函数，RestTemplate对象在底层通过使用java.net包下的实现创建HTTP 请求，
- 可以通过使用ClientHttpRequestFactory指定不同的HTTP请求方式。
- ClientHttpRequestFactory接口主要提供了三种实现方式
- 1、SimpleClientHttpRequestFactory方式，此处生成SimpleBufferingClientHttpRequest，使用HttpURLConnection创建底层的Http请求连接
- 2、HttpComponentsClientHttpRequestFactory方式，此处生成HttpComponentsClientHttpRequest，使用http client来实现网络请求
- 3、OkHttp3ClientHttpRequestFactory方式，此处生成OkHttp3ClientHttpRequest，使用okhttp来实现网络请求

**优点**

- 并没有重写底层的HTTP请求技术，而是提供配置，可选用OkHttp/HttpClient等
- 在OkHttp/HttpClient之上，封装了请求操作，可以定义Convertor来实现对象到请求body的转换方法，以及返回body到对象的转换方法。

**配置**

```
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        return new RestTemplate(factory);
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        //单位为ms
        factory.setReadTimeout(5000);
        //单位为ms
        factory.setConnectTimeout(5000);
        return factory;
    }

    @Primary
    @Bean
    public ClientHttpRequestFactory httpComponentsClientHttpRequestFactory() {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        //单位为ms
        factory.setReadTimeout(6000);
        //单位为ms
        factory.setConnectTimeout(6000);
        return factory;
    }

    /*@Bean
    public ClientHttpRequestFactory okHttp3ClientHttpRequestFactory() {
        OkHttp3ClientHttpRequestFactory factory = new OkHttp3ClientHttpRequestFactory();
        //单位为ms
        factory.setReadTimeout(7000);
        //单位为ms
        factory.setConnectTimeout(7000);
        return factory;
    }*/

}
```

**使用**

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestGet {

    @Autowired
    private RestTemplate restTemplate;

    @Test
    public void testGet() {
        String url = "http://localhost:8080/welcome?message={1}";
        //可以使用map来封装请求参数
        Map<String, String> map = new HashMap<>();
        map.put("1", "world");
        String jsonResult = restTemplate.getForObject(url, String.class, map);
        System.out.println("result:" + jsonResult);
    }

}
```

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestPost {

    @Autowired
    private RestTemplate restTemplate;

    @Test
    public void testPost() {
        RequestObj requestObj = RequestObj.builder().id(1)
                .age(20)
                .name("鹧鸪哨").build();
        String url = "http://localhost:8080/test";
        //发起请求
        String jsonResult = restTemplate.postForObject(url, requestObj, String.class);
        System.out.println("result:" + jsonResult);
    }
}
```

 **#关于Feign**

**介绍**

　　Feign 的英文表意为“假装，伪装，变形”， 是一个http请求调用的轻量级框架，可以以Java接口注解的方式调用Http请求，而不用像Java中通过封装HTTP请求报文的方式直接调用。Feign通过处理注解，将请求模板化，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的请求，这种请求相对而言比较直观。
　　Feign被广泛应用在Spring Cloud 的解决方案中，是学习基于Spring Cloud 微服务架构不可或缺的重要组件。

　具体详情参考https://github.com/OpenFeign/feign

pom配置

```
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

```
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
　　　　<!-- feign底层采用的http请求方式 不加则默认使用JDK的HttpURLConnection -->
　　　　<dependency>
    　　　　<groupId>io.github.openfeign</groupId>
    　　　　<artifactId>feign-httpclient</artifactId>
　　　　</dependency>
</dependencies>
```

Feign配置

```
@Configuration
public class FeignConfig {
 
    @Bean
    Logger.Level feignLoggerLevel() {
        //记录请求和响应的标头，正文和元数据
        return Logger.Level.FULL;
    }
 
    /**
     * 如果远程接口由于各种问题没有在响应中设置content-type，
     * 导致FeignClient接收的时候contentType为null，HttpMessageConverterExtractor将其设置为MediaType.APPLICATION_OCTET_STREAM
     * 此时MessageConverter需要增加MediaType.APPLICATION_OCTET_STREAM支持
     */
    @Bean
    public Decoder feignDecoder() {
        MappingJackson2HttpMessageConverter hmc = new MappingJackson2HttpMessageConverter(customObjectMapper());
        List<MediaType> unModifiedMediaTypeList = hmc.getSupportedMediaTypes();
        List<MediaType> mediaTypeList = new ArrayList<>(unModifiedMediaTypeList.size() + 1);
        mediaTypeList.addAll(unModifiedMediaTypeList);
        mediaTypeList.add(MediaType.APPLICATION_OCTET_STREAM);
        hmc.setSupportedMediaTypes(mediaTypeList);
        ObjectFactory<HttpMessageConverters> objectFactory = () -> new HttpMessageConverters(hmc);
        return new ResponseEntityDecoder(new SpringDecoder(objectFactory));
    }
    @Bean
    public ObjectMapper customObjectMapper() {
```

　　　　 //解决LocalDate、LocalDateTime反序列化问题

　　　　 ObjectMapper objectMapper = new ObjectMapper();

　　　　 objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

　　　  objectMapper.registerModule(new JavaTimeModule());

```
return new ObjectMapper();
    }
}
```

启用feign客户端

```
@SpringBootApplication
@EnableFeignClients
public class DemoHttpApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(DemoHttpApplication.class, args);
    }
 
}
```

定义feign客户端

```
@FeignClient(name = "test-service", path = "/", url = "http://localhost:8080")
public interface TestClient {

    @GetMapping("/welcome")
    String welcome(@RequestParam String message);

    @PostMapping("/test")
    String test(@RequestBody RequestObj param);

}
```

测试调用

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class FeignTest {

    @Autowired
    private TestClient testClient;

    @Test
    public void testGet() {
        System.out.println("result:" + testClient.welcome("world"));
    }

    @Test
    public void testPost() {
        RequestObj requestObj = RequestObj.builder().id(1)
                .age(20)
                .name("鹧鸪哨").build();
        System.out.println("result:" + testClient.test(requestObj));
    }

}
```

 