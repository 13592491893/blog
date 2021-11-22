---
abbrlink: 33119
title: RPC与Http的区别
comments: true
toc: true
description: RPC与Http的区别
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - rpc
  - http
  - 远程调用
date: 2021-9-21 16:00:00
---
# RPC与Http的区别

## 一.远程调用方式

无论是微服务还是分布式服务（都是SOA，都是面向服务编程），都面临着服务间的远程调用。那么服务间的远程调用方式有哪些呢？

常见的远程调用方式有以下几种：

- RPC：Remote Produce Call远程过程调用，类似的还有RMI（Remote Methods Invoke 远程方法调用，是JAVA中的概念，是JAVA十三大技术之一）。自定义数据格式，基于原生TCP通信，速度快，效率高。早期的webservice，现在热门的dubbo，都是RPC的典型

  - RPC的框架：webservie(cxf)、dubbo
  - RMI的框架：hessian

- Http：http其实是一种网络传输协议，基于TCP，规定了数据传输的格式。现在客户端浏览器与服务端通信基本都是采用Http协议。也可以用来进行远程服务调用。缺点是消息封装臃肿。

  现在热门的Rest风格，就可以通过http协议来实现。

  - http的实现技术：HttpClient

- **相同点**：底层通讯都是基于socket，都可以实现远程调用，都可以实现服务调用服务

- **不同点：**
  **RPC**：框架有：dubbo、cxf、（RMI远程方法调用）Hessian
  当使用RPC框架实现服务间调用的时候，要求服务提供方和服务消费方 都必须使用统一的RPC框架，要么都dubbo，要么都cxf

  跨操作系统在同一编程语言内使用
  优势：调用快、处理快

  **http**：框架有：httpClient
  当使用http进行服务间调用的时候，无需关注服务提供方使用的编程语言，也无需关注服务消费方使用的编程语言，服务提供方只需要提供restful风格的接口，服务消费方，按照restful的原则，请求服务，即可

  跨系统跨编程语言的远程调用框架
  优势：通用性强

  **总结：对比RPC和http的区别**
  1 RPC要求服务提供方和服务调用方都需要使用相同的技术，要么都hessian，要么都dubbo
  而http无需关注语言的实现，只需要遵循rest规范
  2 RPC的开发要求较多，像Hessian框架还需要服务器提供完整的接口代码(包名.类名.方法名必须完全一致)，否则客户端无法运行
  3 Hessian只支持POST请求
  4 Hessian只支持JAVA语言

### 1.1.认识RPC

  RPC，即 Remote Procedure Call（远程过程调用），是一个计算机通信协议。 该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程。说得通俗一点就是：A计算机提供一个服务，B计算机可以像调用本地服务那样调用A计算机的服务。

  通过上面的概念，我们可以知道，实现RPC主要是做到两点：

  - 实现远程调用其他计算机的服务
    - 要实现远程调用，肯定是通过网络传输数据。A程序提供服务，B程序通过网络将请求参数传递给A，A本地执行后得到结果，再将结果返回给B程序。这里需要关注的有两点：
      - 1）采用何种网络通讯协议？
        - 现在比较流行的RPC框架，都会采用TCP作为底层传输协议
      - 2）数据传输的格式怎样？
        - 两个程序进行通讯，必须约定好数据传输格式。就好比两个人聊天，要用同一种语言，否则无法沟通。所以，我们必须定义好请求和响应的格式。另外，数据在网路中传输需要进行序列化，所以还需要约定统一的序列化的方式。
  - 像调用本地服务一样调用远程服务
    - 如果仅仅是远程调用，还不算是RPC，因为RPC强调的是过程调用，调用的过程对用户而言是应该是透明的，用户不应该关心调用的细节，可以像调用本地服务一样调用远程服务。所以RPC一定要对调用的过程进行封装

  RPC调用流程图：

  ![在这里插入图片描述](https://gitee.com/gsshy/picgo/raw/master/img/17ed3c177e02fb87dd2cada0bd165792.png)

### 1.2.认识Http

  Http协议：超文本传输协议，是一种应用层协议。规定了网络传输的请求格式、响应格式、资源定位和操作的方式等。但是底层采用什么网络传输协议，并没有规定，不过现在都是采用TCP协议作为底层传输协议。说到这里，大家可能觉得，Http与RPC的远程调用非常像，都是按照某种规定好的数据格式进行网络通信，有请求，有响应。没错，在这点来看，两者非常相似，但是还是有一些细微差别。

  - RPC并没有规定数据传输格式，这个格式可以任意指定，不同的RPC协议，数据格式不一定相同。
  - Http中还定义了资源定位的路径，RPC中并不需要
  - 最重要的一点：RPC需要满足像调用本地服务一样调用远程服务，也就是对调用过程在API层面进行封装。Http协议没有这样的要求，因此请求、响应等细节需要我们自己去实现。
    - 优点：RPC方式更加透明，对用户更方便。Http方式更灵活，没有规定API和语言，跨语言、跨平台
    - 缺点：RPC方式需要在API层面进行封装，限制了开发的语言环境。

  例如我们通过浏览器访问网站，就是通过Http协议。只不过浏览器把请求封装，发起请求以及接收响应，解析响应的事情都帮我们做了。如果是不通过浏览器，那么这些事情都需要自己去完成。

  ![在这里插入图片描述](https://gitee.com/gsshy/picgo/raw/master/img/a9c94647b4879d21a99870b5df110f03.png)

### 1.3.如何选择？

  既然两种方式都可以实现远程调用，我们该如何选择呢？

  - 速度来看，RPC要比http更快，虽然底层都是TCP，但是http协议的信息往往比较臃肿
  - 难度来看，RPC实现较为复杂，http相对比较简单
  - 灵活性来看，http更胜一筹，因为它不关心实现细节，跨平台、跨语言。

  因此，两者都有不同的使用场景：

  - 如果对效率要求更高，并且开发过程使用统一的技术栈，那么用RPC还是不错的。
  - 如果需要更加灵活，跨语言、跨平台，显然http更合适

  那么我们该怎么选择呢？

  微服务，更加强调的是独立、自治、灵活。而RPC方式的限制较多，因此微服务框架中，一般都会采用基于Http的Rest风格服务。	

### 1.4.Http客户端工具

  　　既然微服务选择了Http，那么我们就需要考虑自己来实现对请求和响应的处理。不过开源世界已经有很多的http客户端工具，能够帮助我们做这些事情，例如：

   - HttpClient
   - OKHttp
   - URLConnection

  　　接下来，我们就一起了解一款比较流行的客户端工具：HttpClient

``` java
public void testGet() throws IOException {
       　　 HttpGet request = new HttpGet("http://www.baidu.com");
       　　 String response = this.httpClient.execute(request, new BasicResponseHandler());
      　　  System.out.println(response);
   　 }
　　public void testPost() throws IOException {
   　　 HttpPost request = new HttpPost("http://www.oschina.net/");
   　　 request.setHeader("User-Agent",
                      "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36");
    　　String response = this.httpClient.execute(request, new BasicResponseHandler());
   　　 System.out.println(response);
　　}
　　
```

HttpClient请求数据后是json字符串，需要我们自己把Json字符串反序列化为对象，我们会使用JacksonJson工具来实现。

`JacksonJson`是SpringMVC内置的json处理工具，其中有一个`ObjectMapper`类，可以方便的实现对json的处理：

``` java
#### 对象转json
// json处理工具
    private ObjectMapper mapper = new ObjectMapper();
    @Test
    public void testJson() throws JsonProcessingException {
        User user = new User();
        user.setId(8L);
        user.setAge(21);
        user.setName("柳岩");
        user.setUserName("liuyan");
        // 序列化
        String json = mapper.writeValueAsString(user);
        System.out.println("json = " + json);
    }

#### json转对象
​```java
// json处理工具
private ObjectMapper mapper = new ObjectMapper();
@Test
public void testJson() throws IOException {
    User user = new User();
    user.setId(8L);
    user.setAge(21);
    user.setName("柳岩");
    user.setUserName("liuyan");
    // 序列化
    String json = mapper.writeValueAsString(user);
    // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
    User result = mapper.readValue(json, User.class);
    System.out.println("result = " + result);
}
#### json转集合
json转集合比较麻烦，因为你无法同时把集合的class和元素的class同时传递到一个参数。
因此Jackson做了一个类型工厂，用来解决这个问题：
​```java
// json处理工具
private ObjectMapper mapper = new ObjectMapper();
@Test
public void testJson() throws IOException {
    User user = new User();
    user.setId(8L);
    user.setAge(21);
    user.setName("柳岩");
    user.setUserName("liuyan");
    // 序列化,得到对象集合的json字符串
    String json = mapper.writeValueAsString(Arrays.asList(user, user));
    // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
    List<User> users = mapper.readValue(json, mapper.getTypeFactory().constructCollectionType(List.class, User.class));
    for (User u : users) {
        System.out.println("u = " + u);
    }
}
#### json转任意复杂类型
当对象泛型关系复杂时，类型工厂也不好使了。这个时候Jackson提供了TypeReference来接收类型泛型，然后底层通过反射来获取泛型上的具体类型。实现数据转换。
​```java
// json处理工具
private ObjectMapper mapper = new ObjectMapper();
@Test
public void testJson() throws IOException {
    User user = new User();
    user.setId(8L);
    user.setAge(21);
    user.setName("柳岩");
    user.setUserName("liuyan");
    // 序列化,得到对象集合的json字符串
    String json = mapper.writeValueAsString(Arrays.asList(user, user));
    // 反序列化，接收两个参数：json数据，反序列化的目标类字节码
    List<User> users = mapper.readValue(json, new TypeReference<List<User>>(){});
    for (User u : users) {
        System.out.println("u = " + u);
    }
}
 
 
Spring提供了一个RestTemplate模板工具类，对基于Http的客户端进行了封装，并且实现了对象与json的序列化和反序列化，非常方便。RestTemplate并没有限定Http的客户端类型，而是进行了抽象，目前常用的3种都有支持：
- HttpClient
- OkHttp
- JDK原生的URLConnection（默认的）
首先在项目中注册一个`RestTemplate`对象，可以在启动类位置注册：
​```java
@SpringBootApplication
public class HttpDemoApplication {
 public static void main(String[] args) {
  SpringApplication.run(HttpDemoApplication.class, args);
 }
 @Bean
 public RestTemplate restTemplate() {
        // 默认的RestTemplate，底层是走JDK的URLConnection方式。
  return new RestTemplate();
 }
}
​```
 
在测试类中直接`@Autowired`注入：
​```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = HttpDemoApplication.class)
public class HttpDemoApplicationTests {
 @Autowired
 private RestTemplate restTemplate;
 @Test
 public void httpGet() {
  User user = this.restTemplate.getForObject("http://localhost/hello", User.class);
  System.out.println(user);
 }
}
​```
- 通过RestTemplate的getForObject()方法，传递url地址及实体类的字节码，RestTemplate会自动发起请求，接收响应，并且帮我们对响应结果进行反序列化。
```

