---
abbrlink: 49848
title: JavaWeb——Servlet
comments: true
toc: true
description: JavaWeb——Servlet
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - web
  - servlet
  - java
date: 2021-5-21 16:00:00
---
# JavaWeb——Servlet

版权声明：本文为CSDN博主「刘扬俊」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_19782019/article/details/80292110

————————————————

## 一、Tomcat工作机制动画演示(点击动图可全屏观看）

![img](https://img-blog.csdnimg.cn/2018120522281643.gif)

## 二、什么是Servlet

Servlet（Server Applet），全称Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。
Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。

## 三、Servlet的工作模式

1. 客户端发送请求至服务器
2. 服务器启动并调用Servlet，Servlet根据客户端请求生成响应内容并将其传给服务器
3. 服务器将响应返回客户端

## 四、Servlet API 概览

Servlet API 包含以下4个Java包:

1. javax.servlet  其中包含定义servlet和servlet容器之间契约的类和接口。
2. javax.servlet.http  其中包含定义HTTP Servlet 和Servlet容器之间的关系。
3. javax.servlet.annotation  其中包含标注servlet，Filter,Listener的标注。它还为被标注元件定义元数据。
4. javax.servlet.descriptor，其中包含提供程序化登录Web应用程序的配置信息的类型。

## 五、Servlet 的主要类型

![img](https://img-blog.csdn.net/20180512170931739?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 六、Servlet 的使用方法

Servlet技术的核心是Servlet，它是所有Servlet类必须直接或者间接实现的一个接口。在编写实现Servlet的Servlet类时，直接实现它。在扩展实现这个这个接口的类时，间接实现它。

> Servlet中@WebServlet属性详解:(同理理解到Filter)
>
> 在Servlet中,设置了@WebServlet注解,当请求该Servlet时,服务器就会自动读取当中的信息,如果注解@WebServlet("/category"),则表示该Servlet默认的请求路径为…/category,这里省略了urlPatterns属性名,完整的写法应该是:@WebServlet(urlPatterns = “/category”),如果在@WebServlet中需要设置多个属性,必须给属性值加上属性名称,中间用逗号隔开,否则会报错.
> 若没有设置@WebServlet的name属性，默认值会是Servlet的类完整名称.
>
> 在servlet3.0以后，web.xml中对Servlet配置，同样可以在@WebServlet注解中配置.
> 下面是@WebServlet的属性列表:
> ![image-20210703173226572](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210703173226572.png)

**servlet的url-pattern匹配规则：**

首先需要明确几容易混淆的规则：

1. servlet容器中的匹配规则既不是简单的通配，也不是正则表达式，而是特定的规则。所以不要用通配符或者正则表达式的匹配规则来看待servlet的url-pattern。
2. Servlet 2.5开始，一个servlet可以使用多个url-pattern规则，<servlet-mapping>标签声明了与该servlet相应的匹配规则，每个<url-pattern>标签代表1个匹配规则；
3. 当servlet容器接收到浏览器发起的一个url请求后，容器会用url减去当前应用的上下文路径，以剩余的字符串作为servlet映射，假如url是http://localhost:8080/appDemo/index.html，其应用上下文是appDemo，容器会将http://localhost:8080/appDemo去掉，用剩下的/index.html部分拿来做servlet的映射匹配
4. url-pattern映射匹配过程是有优先顺序的
5. 而且当有一个servlet匹配成功以后，就不会去理会剩下的servlet了。

需要注意的问题：

1. 路径匹配和扩展名匹配无法同时设置：匹配方法只有三种，要么是路径匹配（以“/”字符开头，并以“/*”结尾），要么是扩展名匹配（以“*.”开头），要么是精确匹配，三种匹配方法不能进行组合，不要想当然使用通配符或正则规则。如`<url-pattern>/user/*.action</url-pattern>`是非法的，另外注意：`<url-pattern>/aa/*/bb</url-pattern>`是精确匹配，合法，这里的*不是通配的含义
2. "/*"和"/"含义并不相同
   “/*”属于路径匹配，并且可以匹配所有request，由于路径匹配的优先级仅次于精确匹配，所以“`/*`”会覆盖所有的扩展名匹配，很多404错误均由此引起，所以这是一种特别恶劣的匹配模式，一般只用于filter的url-pattern
   “/”是servlet中特殊的匹配模式，切该模式有且仅有一个实例，优先级最低，不会覆盖其他任何url-pattern，只是会替换servlet容器的内建default servlet ，该模式同样会匹配所有request。
   配置“/”后，一种可能的现象是myServlet会拦截诸如http://localhost:8080/appDemo/user/addUser.action、http://localhost:8080/appDemo/user/updateUser的格式的请求，但是并不会拦截http://localhost:8080/appDemo/user/users.jsp、http://localhost:8080/appDemo/index.jsp，这是应为servlet容器有内置的“*.jsp”匹配器，而扩展名匹配的优先级高于缺省匹配，所以才会有上述现象。

## 七、Servlet 的工作原理

 Servlet接口定义了Servlet与servlet容器之间的契约。这个契约是：Servlet容器将Servlet类载入内存，并产生Servlet实例和调用它具体的方法。**但是要注意的是，在一个应用程序中，每种Servlet类型只能有一个实例**。
用户请求致使Servlet容器调用Servlet的Service（）方法，并传入一个ServletRequest对象和一个ServletResponse对象。ServletRequest对象和ServletResponse对象都是由Servlet容器（例如TomCat）封装好的，并不需要程序员去实现，程序员可以直接使用这两个对象。
ServletRequest中封装了当前的Http请求，因此，开发人员不必解析和操作原始的Http数据。ServletResponse表示当前用户的Http响应，程序员只需直接操作ServletResponse对象就能把响应轻松的发回给用户。
对于每一个应用程序，Servlet容器还会创建一个ServletContext对象。这个对象中封装了上下文（应用程序）的环境详情。每个应用程序只有一个ServletContext。每个Servlet对象也都有一个封装Servlet配置的ServletConfig对象。

## 八、Servlet 接口中定义的方法

让我们首先来看一看Servlet接口中定义了哪些方法吧。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
 
    ServletConfig getServletConfig();
 
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
 
    String getServletInfo();
 
    void destroy();
}
```

## 九、Servlet 的生命周期

其中，init( ),service( ),destroy( )是Servlet生命周期的方法。代表了Servlet从“出生”到“工作”再到“死亡 ”的过程。Servlet容器（例如TomCat）会根据下面的规则来调用这三个方法：

1.init( ),当Servlet第一次被请求时，Servlet容器就会开始调用这个方法来初始化一个Servlet对象出来，但是这个方法在后续请求中不会在被Servlet容器调用，就像人只能“出生”一次一样。我们可以利用init（ ）方法来执行相应的初始化工作。调用这个方法时，Servlet容器会传入一个ServletConfig对象进来从而对Servlet对象进行初始化。

2.service( )方法，每当请求Servlet时，Servlet容器就会调用这个方法。就像人一样，需要不停的接受老板的指令并且“工作”。第一次请求时，Servlet容器会先调用init( )方法初始化一个Servlet对象出来，然后会调用它的service( )方法进行工作，但在后续的请求中，Servlet容器只会调用service方法了。

3.destory(),当要销毁Servlet时，Servlet容器就会调用这个方法，就如人一样，到时期了就得死亡。在卸载应用程序或者关闭Servlet容器时，就会发生这种情况，一般在这个方法中会写一些清除代码。

首先，我们来编写一个简单的Servlet来验证一下它的生命周期：

```java
public class MyFirstServlrt implements Servlet {
 
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("Servlet正在初始化");
    }
 
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }
 
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        //专门向客服端提供响应的方法
        System.out.println("Servlet正在提供服务");
 
    }
 
    @Override
    public String getServletInfo() {
        return null;
    }
 
    @Override
    public void destroy() {
        System.out.println("Servlet正在销毁");
    }
}      
```

然后在xml中配置正确的映射关系，在浏览器中访问Servlet，第一次访问时，控制台输出了如下信息：

![img](https://img-blog.csdn.net/20180512164825188?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后，我们在浏览器中刷新3遍：

控制台输出的信息变成了下面这样：

![img](https://img-blog.csdn.net/20180512164922863?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

接下来，我们关闭Servlet容器：

![img](https://img-blog.csdn.net/20180512165004507?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

控制台输出了Servlet的销毁信息，这就是一个Servlet的完整生命周期。

## 十、Servlet 的其它两个方法

getServletInfo（ ），这个方法会返回Servlet的一段描述，可以返回一段字符串。

  getServletConfig（ ），这个方法会返回由Servlet容器传给init（ ）方法的ServletConfig对象。

## 十一、ServletRequset接口

Servlet容器对于接受到的每一个Http请求，**都会创建一个ServletRequest对象**(*ServletRequset是一个接口，为什么可以创建一个ServletRequset对象?==>>HttpServletRequest接口的实现类是Web容器负责的，Tomcat服务器有自己的实现。但是程序要还是只需要面向HttpServletRequest接口调用方法即可，不需要关心具体的实现类*)，并把这个对象传递给Servlet的Sevice( )方法。其中，ServletRequest对象内封装了关于这个请求的许多详细信息。

让我们来看一看ServletRequest接口的部分内容：

```java
public interface ServletRequest {
  
    int getContentLength();//返回请求主体的字节数
 
    String getContentType();//返回主体的MIME类型
 
    String getParameter(String var1);//返回请求参数的值
 
}
```

getParameter获取参数并返回给页面：

![](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210702152308787.png)

![image-20210702152337321](C:\Users\45917\AppData\Roaming\Typora\typora-user-images\image-20210702152337321.png)

其中，getParameter是在ServletRequest中最常用的方法，可用于获取查询字符串的值。

## 十二、ServletResponse接口

 javax.servlet.ServletResponse接口表示一个Servlet响应，在调用Servlet的Service( )方法前，Servlet容器会先创建一个ServletResponse对象，并把它作为第二个参数传给Service( )方法。ServletResponse隐藏了向浏览器发送响应的复杂过程。

让我们也来看看ServletResponse内部定义了哪些方法：

```java
public interface ServletResponse {
    String getCharacterEncoding();
 
    String getContentType();
 
    ServletOutputStream getOutputStream() throws IOException;
 
    PrintWriter getWriter() throws IOException;
 
    void setCharacterEncoding(String var1);
 
    void setContentLength(int var1);
 
    void setContentType(String var1);
 
    void setBufferSize(int var1);
 
    int getBufferSize();
 
    void flushBuffer() throws IOException;
 
    void resetBuffer();
 
    boolean isCommitted();
 
    void reset();
 
    void setLocale(Locale var1);
 
    Locale getLocale();
}
```

其中的getWriter方法，它返回了一个可以向客户端发送文本的的Java.io.PrintWriter对象。默认情况下，PrintWriter对象使用ISO-8859-1编码（该编码在输入中文时会发生乱码）。

在向客户端发送响应时，大多数都是使用该对象向客户端发送HTML。

还有一个方法也可以用来向浏览器发送数据，它就是getOutputStream，从名字就可以看出这是一个二进制流对象，因此这个方法是用来发送二进制数据的。

在发送任何HTML之前，应该先调用setContentType（）方法，设置响应的内容类型，并将“text/html”作为一个参数传入，这是在告诉浏览器响应的内容类型为HTML，需要以HTML的方法解释响应内容而不是普通的文本，或者也可以加上“charset=UTF-8”改变响应的编码方式以防止发生中文乱码现象。

## 十三、ServletConfig接口

当Servlet容器初始化Servlet时，Servlet容器会给Servlet的init( )方式传入一个ServletConfig对象。

其中几个方法如下：

![img](https://img-blog.csdn.net/20180512175029254?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 十四、ServletContext对象

ServletContext对象表示Servlet应用程序。每个Web应用程序都只有一个ServletContext对象。在将一个应用程序同时部署到多个容器的分布式环境中，每台Java虚拟机上的Web应用都会有一个ServletContext对象。

通过在ServletConfig中调用getServletContext方法，也可以获得ServletContext对象。

那么为什么要存在一个ServletContext对象呢？存在肯定是有它的道理，因为有了ServletContext对象，就可以共享从应用程序中的所有资料处访问到的信息，并且可以动态注册Web对象。前者将对象保存在ServletContext中的一个内部Map中。保存在ServletContext中的对象被称作属性。

ServletContext中的下列方法负责处理属性：

```java
Object getAttribute(String var1);
 
Enumeration<String> getAttributeNames();
 
void setAttribute(String var1, Object var2);
 
void removeAttribute(String var1);
```

> 热知识：
>
> 1.Session(做验证码时疑问)
> 访问同一个应用程序，一个浏览器对应一个Session，利用此知识可以产生验证码图片和字符串，把字符串存到session里，图片返回前端，登录时传入验证码与session里的做对比
>
> 2.Request、Session、servletContext区别
> **首先从作用范围来说**
>
> 1. Request       保存的键值仅在下一个request对象中可以得到。
> 2. Session        它是一个会话范围，相当于一个局部变量，从Session第一次创建知道关闭，数据都一直 保存，每一个客户都有一个Session，所以它可以被客户一直访问，只要Session没有关闭和超时即浏览器关闭。
> 3. servletContext    它代表了servlet环境的上下文，相当于一个全局变量，即只要某个web应用在启动中，这个对象就一直都有效的存在，所以它的范围是最大的，存储的数据可以被所有用户使用，只要服务器不关闭，数据就会一直都存在。
>
> **它们的优缺点：**
>
> 1. **request：**
>    好处：用完就仍，不会导致资源占用的无限增长。
>    弊处：数据只能被下一个对象获取，所以在写程序时会因为无法共享数据导致每次要用都从数据库中取，多做操作，自然会对性能有一些影响。
>
> 2. **session：**好处：是一个局部变量，可以保存用户的信息并直接取出，不用每次都去数据库抓，少做操作，极大的方便了程序的编写。
>    弊处：每个客户都有一个session，只能自己使用，不同session可能保存大量重复数据； 可能耗费大量服务器内存； 另外session构建在cookie和url重写的基础上，所以用session实现会话跟踪，会用掉一点点服务器带宽和客户端保持联络， 当然session越多，耗费的带宽越多，理论上也会对性能造成影响。 集群的session同步会是个问题。
>
> 3. **servletContext：**
>
>    好处：不用每次都去数据库抓，少做操作。 存储的数据所有客户都可以用。 可减少重复在内存中存储数据造成的开销。

``` java
package com.cdsxt.action;
 
import java.io.IOException;
import java.io.PrintWriter;
 
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
 
public class SessionServlet extends HttpServlet {
 
	
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		this.doPost(request, response);
	}
	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//创建 session的时机
		//首先 服务器调用request.getSession();
		//请求头里没有JsessionId     创建一个新的session 对象
		//请求头里有JsessionId    按照这个JsessionId去找对应的session对象
		//如果session对象没有失效   不创建新的session   
		//如果session对象失效    创建新的session
		
		//什么叫一次会话    笼统回答 浏览器启动并且访问项目  到  关闭浏览器 .事实上，取决于JsessionID对应的session是否失效
		
		HttpSession session=request.getSession();
		session.setAttribute("name", "aaaaaaaa");
		session.setAttribute("value", "bbbbbbbbb");
//		session.setMaxInactiveInterval(10); 
		//请求转发可以拿到
//		request.getRequestDispatcher("sessionServlet2").forward(request, response);
		//重定向也可以拿到
//		response.sendRedirect("sessionServlet2");
		
		//request    同一次请求
		//servletContext(application)  整个web的servlet都享用   从web启动  到web卸载
		//session    同一次会话
		
		
	}
 
}
```



## 十五、GenericServlet抽象类

前面我们编写Servlet一直是通过实现Servlet接口来编写的，但是，使用这种方法，则必须要实现Servlet接口中定义的所有的方法，即使有一些方法中没有任何东西也要去实现，并且还需要自己手动的维护ServletConfig这个对象的引用。因此，这样去实现Servlet是比较麻烦的。

```java
void init(ServletConfig var1) throws ServletException;
```

幸好，GenericServlet抽象类的出现很好的解决了这个问题。本着尽可能使代码简洁的原则，GenericServlet实现了Servlet和ServletConfig接口，下面是GenericServlet抽象类的具体代码：

```java
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final String LSTRING_FILE = "javax.servlet.LocalStrings";
    private static ResourceBundle lStrings = ResourceBundle.getBundle("javax.servlet.LocalStrings");
    private transient ServletConfig config;
 
    public GenericServlet() {
    }
 
    public void destroy() {
    }
 
    public String getInitParameter(String name) {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getInitParameter(name);
        }
    }
 
    public Enumeration<String> getInitParameterNames() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getInitParameterNames();
        }
    }
 
    public ServletConfig getServletConfig() {
        return this.config;
    }
 
    public ServletContext getServletContext() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getServletContext();
        }
    }
 
    public String getServletInfo() {
        return "";
    }
 
    public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }
 
    public void init() throws ServletException {
    }
 
    public void log(String msg) {
        this.getServletContext().log(this.getServletName() + ": " + msg);
    }
 
    public void log(String message, Throwable t) {
        this.getServletContext().log(this.getServletName() + ": " + message, t);
    }
 
    public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
 
    public String getServletName() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized"));
        } else {
            return sc.getServletName();
        }
    }
}
```

其中，GenericServlet抽象类相比于直接实现Servlet接口，有以下几个好处：

1.为Servlet接口中的所有方法提供了默认的实现，则程序员需要什么就直接改什么，不再需要把所有的方法都自己实现了。

2.提供方法，包围ServletConfig对象中的方法。

3.将init( )方法中的ServletConfig参数赋给了一个内部的ServletConfig引用从而来保存ServletConfig对象，不需要程序员自己去维护ServletConfig了。

```java
public void init(ServletConfig config) throws ServletException {
    this.config = config;
    this.init();
}
```

但是，我们发现在GenericServlet抽象类中还存在着另一个没有任何参数的Init()方法：

```java
public void init() throws ServletException {
}
```
设计者的初衷到底是为了什么呢？在第一个带参数的init（）方法中就已经把ServletConfig对象传入并且通过引用保存好了，完成了Servlet的初始化过程，那么为什么后面还要加上一个不带任何参数的init（）方法呢？这不是多此一举吗？
当然不是多此一举了，存在必然有存在它的道理。我们知道，抽象类是无法直接产生实例的，需要另一个类去继承这个抽象类，那么就会发生方法覆盖的问题，如果在类中覆盖了GenericServlet抽象类的init（）方法，那么程序员就必须手动的去维护ServletConfig对象了，还得调用super.init(servletConfig）方法去调用父类GenericServlet的初始化方法来保存ServletConfig对象，这样会给程序员带来很大的麻烦。GenericServlet提供的第二个不带参数的init( )方法，就是为了解决上述问题的。
这个不带参数的init（）方法，是在ServletConfig对象被赋给ServletConfig引用后，由第一个带参数的init(ServletConfig servletconfig)方法调用的，那么这意味着，当程序员如果需要覆盖这个GenericServlet的初始化方法，则只需要覆盖那个不带参数的init( )方法就好了，此时，servletConfig对象仍然有GenericServlet保存着。
说了这么多，通过扩展GenericServlet抽象类，就不需要覆盖没有计划改变的方法。因此，代码将会变得更加的简洁，程序员的工作也会减少很多。
然而，虽然GenricServlet是对Servlet一个很好的加强，但是也不经常用，因为他不像HttpServlet那么高级。HttpServlet才是主角，在现实的应用程序中被广泛使用。那么我们接下来就看看传说中的HttpServlet到底厉害在哪里吧。

## 十六、javax.servlet.http包内容

之所以所HttpServlet要比GenericServlet强大，其实也是有道理的。HttpServlet是由GenericServlet抽象类扩展而来的，HttpServlet抽象类的声明如下所示：

```java
public abstract class HttpServlet extends GenericServlet implements Serializable 
```

HttpServlet之所以运用广泛的另一个原因是现在大部分的应用程序都要与HTTP结合起来使用。这意味着我们可以利用HTTP的特性完成更多更强大的任务。Javax。servlet.http包是Servlet API中的第二个包，其中包含了用于编写Servlet应用程序的类和接口。Javax.servlet.http中的许多类型都覆盖了Javax.servlet中的类型。
![img](https://img-blog.csdn.net/20180513104757248?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 十七、HttpServlet抽象类

HttpServlet抽象类是继承于GenericServlet抽象类而来的。使用HttpServlet抽象类时，还需要借助分别代表Servlet请求和Servlet响应的HttpServletRequest和HttpServletResponse对象。

HttpServletRequest接口扩展于javax.servlet.ServletRequest接口，HttpServletResponse接口扩展于javax.servlet.servletResponse接口。

```java
public interface HttpServletRequest extends ServletRequest
public interface HttpServletResponse extends ServletResponse
```

HttpServlet抽象类覆盖了GenericServlet抽象类中的Service( )方法，并且添加了一个自己独有的Service(HttpServletRequest request，HttpServletResponse方法。

让我们来具体的看一看HttpServlet抽象类是如何实现自己的service方法吧：

首先来看GenericServlet抽象类中是如何定义service方法的：

```java
public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
```

我们看到是一个抽象方法，也就是HttpServlet要自己去实现这个service方法，我们在看看HttpServlet是怎么覆盖这个service方法的：

```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException("non-HTTP request or response");
    }
 
    this.service(request, response);
}
```

我们发现，HttpServlet中的service方法把接收到的ServletRequsest类型的对象转换成了HttpServletRequest类型的对象，把ServletResponse类型的对象转换成了HttpServletResponse类型的对象。之所以能够这样强制的转换，是因为在调用Servlet的Service方法时，Servlet容器总会传入一个HttpServletRequest对象和HttpServletResponse对象，预备使用HTTP。因此，转换类型当然不会出错了。

转换之后，service方法把两个转换后的对象传入了另一个service方法，那么我们再来看看这个方法是如何实现的：

```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String method = req.getMethod();
    long lastModified;
    if (method.equals("GET")) {
        lastModified = this.getLastModified(req);
        if (lastModified == -1L) {
            this.doGet(req, resp);
        } else {
            long ifModifiedSince = req.getDateHeader("If-Modified-Since");
            if (ifModifiedSince < lastModified) {
                this.maybeSetLastModified(resp, lastModified);
                this.doGet(req, resp);
            } else {
                resp.setStatus(304);
            }
        }
    } else if (method.equals("HEAD")) {
        lastModified = this.getLastModified(req);
        this.maybeSetLastModified(resp, lastModified);
        this.doHead(req, resp);
    } else if (method.equals("POST")) {
        this.doPost(req, resp);
    } else if (method.equals("PUT")) {
        this.doPut(req, resp);
    } else if (method.equals("DELETE")) {
        this.doDelete(req, resp);
    } else if (method.equals("OPTIONS")) {
        this.doOptions(req, resp);
    } else if (method.equals("TRACE")) {
        this.doTrace(req, resp);
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[]{method};
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(501, errMsg);
    }
 
}
```

## 十八、HttpServletRequest接口

HttpServletRequest表示Http环境中的Servlet请求。它扩展于javax.servlet.ServletRequest接口，并添加了几个方法。

```java
String getContextPath();//返回请求上下文的请求URI部分
Cookie[] getCookies();//返回一个cookie对象数组
String getHeader(String var1);//返回指定HTTP标题的值
String getMethod();//返回生成这个请求HTTP的方法名称
String getQueryString();//返回请求URL中的查询字符串
HttpSession getSession();//返回与这个请求相关的会话对象
```

## 十九、HttpServletRequest内封装的请求

因为Request代表请求，所以我们可以通过该对象分别获得HTTP请求的请求行，请求头和请求体。

  关于HTTP具体的详细解释，可以参考我的另一篇博文：JavaWeb——HTTP。

![img](https://img-blog.csdn.net/20180513130638615?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二十、通过request获得请求行

假设查询字符串为：username=zhangsan&password=123

获得客户端的请求方式：String getMethod()

获得请求的资源：

String getRequestURI()

StringBuffer getRequestURL()

String getContextPath() ---web应用的名称

String getQueryString() ---- get提交url地址后的参数字符串

## 二十一、通过request获得请求头

long getDateHeader(String name)

String getHeader(String name)

Enumeration getHeaderNames()

Enumeration getHeaders(String name)

int getIntHeader(String name)

referer头的作用：执行该此访问的的来源，做防盗链

## 二十二、通过request获得请求体

请求体中的内容是通过post提交的请求参数，格式是：

username=zhangsan&password=123&hobby=football&hobby=basketball

key ---------------------- value

username                               [zhangsan]

password                               [123]

hobby                                          [football，basketball]                                       

以上面参数为例，通过一下方法获得请求参数：

String getParameter(String name)

String[] getParameterValues(String name)

Enumeration getParameterNames()

Map<String,String[]> getParameterMap()

注意：get请求方式的请求参数 上述的方法一样可以获得

## 二十三、Request乱码问题的解决方法

在前面我们讲过，**在service中使用的编码解码方式默认为：ISO-8859-1编码**，但此编码并不支持中文，因此会出现乱码问题，所以我们需要手动修改编码方式为UTF-8编码，才能解决中文乱码问题，下面是发生乱码的具体细节：

![img](https://img-blog.csdn.net/20180513195355316?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

解决post提交方式的乱码：request.setCharacterEncoding("UTF-8");

 解决get提交的方式的乱码：parameter = newString(parameter.getbytes("iso8859-1"),"utf-8");

## 二十四、HttpServletResponse接口

在Service API中，定义了一个HttpServletResponse接口，它继承自ServletResponse接口，专门用来封装HTTP响应消息。    由于HTTP请求消息分为状态行，响应消息头，响应消息体三部分，因此，在HttpServletResponse接口中定义了向客户端发送响应状态码，响应消息头，响应消息体的方法

## 二十五、HttpServletResponse内封装的响应

![img](https://img-blog.csdn.net/20180513203525684?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二十六、通过Response设置响应

```java
void addCookie(Cookie var1);//给这个响应添加一个cookie
void addHeader(String var1, String var2);//给这个请求添加一个响应头
void sendRedirect(String var1) throws IOException;//发送一条响应码，讲浏览器跳转到指定的位置
void setStatus(int var1);//设置响应行的状态码
addHeader(String name, String value)
addIntHeader(String name, int value)
addDateHeader(String name, long date)
setHeader(String name, String value)
setDateHeader(String name, long date)
setIntHeader(String name, int value)
```

PrintWriter getWriter()

获得字符流，通过字符流的write(String s)方法可以将字符串设置到response   缓冲区中，随后Tomcat会将response缓冲区中的内容组装成Http响应返回给浏览器端。

ServletOutputStream getOutputStream()

获得字节流，通过该字节流的write(byte[] bytes)可以向response缓冲区中写入字节，再由Tomcat服务器将字节内容组成Http响应返回给浏览器。
![img](https://img-blog.csdn.net/2018051320493643?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

注意：虽然response对象的getOutSream（）和getWriter（）方法都可以发送响应消息体，但是他们之间相互排斥，不可以同时使用，否则会发生异常。

![img](https://img-blog.csdn.net/2018051320525170?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二十七、Response的乱码问题

![img](https://img-blog.csdn.net/20180513205522756?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

原因：response缓冲区的默认编码是iso8859-1，此码表中没有中文。所以需要更改response的编码方式：

![img](https://img-blog.csdn.net/20180513205620262?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过更改response的编码方式为UTF-8，任然无法解决乱码问题，因为发送端服务端虽然改变了编码方式为UTF-8，但是接收端浏览器端仍然使用GB2312编码方式解码，还是无法还原正常的中文，因此还需要告知浏览器端使用UTF-8编码去解码。

![img](https://img-blog.csdn.net/20180513210539156?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上面通过调用两个方式分别改变服务端对于Response的编码方式以及浏览器的解码方式为同样的UTF-8编码来解决编码方式不一样发生乱码的问题。

response.setContentType("text/html;charset=UTF-8")这个方法包含了上面的两个方法的调用，因此在实际的开发中，只需要调用一个response.setContentType("text/html;charset=UTF-8")方法即可。
![img](https://img-blog.csdn.net/20180513210927639?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180513204459256?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二十八、Response的工作流程

![img](https://img-blog.csdn.net/20180513204718428?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二十九、Servlet的工作流程

![img](https://img-blog.csdn.net/20180513204808318?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 三十、编写第一个Servlet

首先，我们来写一个简单的用户名，密码的登录界面的html文件：

```html
<form action="/form"method="get">
```

该html文件在最后点击提交按钮时，把表单所有数据通过Get方式发送到/form虚拟路径下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/form" method="get">
    <span>用户名</span><input type="text" name="username"><br>
    <span>密码</span><input type="password" name="password"><br>
    <input type="submit" name="submit">
</form>
 
</body>
</html>
```

访问一下我们刚才写的这个简单的登录界面：

![img](https://img-blog.csdn.net/20180514163550574?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

接下来，我们就开始写一个Servlet用来接收处理表单发送过来的请求，这个Servlet的名称就叫做FormServlet：

``` java
public class FormServlet extends HttpServlet {
    private static final long serialVersionUID = -4186928407001085733L;
 
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
    }
 
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
         //设置响应的编码格式为UTF-8编码，否则发生中文乱码现象
        response.setContentType("text/html;charset=UTF-8");
        //1.获得请求方式
        String method = request.getMethod();
        //2.获得请求的资源相关的内容
        String requestURI = request.getRequestURI();//获得请求URI
        StringBuffer requestURL = request.getRequestURL();
        String webName = request.getContextPath();//获得应用路径（应用名称）
        String querryString = request.getQueryString();//获得查询字符串
 
        response.getWriter().write("<h1>下面是获得的字符串</h1>");
        response.getWriter().write("<h1>method(HTTP方法):<h1>");
        response.getWriter().write("<h1>"+method+"</h1><br>");
        response.getWriter().write("<h1>requestURi(请求URI）:</h1>");
        response.getWriter().write("<h1>" + requestURI + "</h1><br>");
        response.getWriter().write("<h1>webname(应用名称):</h1>");
        response.getWriter().write("<h1>" + webName + "</h1><br>");
        response.getWriter().write("<h1>querrystring(查询字符串):</h1>");
        response.getWriter().write("<h1>" + querryString + "</h1>");
 
 
    }
}
```

该Servlet的作用是，接收form登录表单发送过来的HTTP请求，并解析出请求中封装的一些参数，然后在回写到response响应当中去，最后在浏览器端显示。

  最后一步，我们在XML中配置好这个Servlet的映射关系：

```java
</servlet-mapping>
    <servlet>
        <servlet-name>FormServlet</servlet-name>
        <servlet-class>com.javaee.util.FormServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FormServlet</servlet-name>
        <url-pattern>/form</url-pattern>
    </servlet-mapping>
```

接下来，启动tomcat，在浏览器中输入登录表单的地址：

填入用户名为：root，密码为：123,最后点击提交：

![img](https://img-blog.csdn.net/20180514171147398?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

提交之后，表单数据将会发送到相应的Servlet进行处理，此时，浏览器的地址变成如下所示：

![img](https://img-blog.csdn.net/20180514171352799?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 我们会发现，在地址栏中，多了后面的“?username=root&password=123&提交=提交”字符串，这其实就是我们开始填写的参数，以Get的方法发送过去，所以查询字符串会直接加在链接后面，如果采用的是Post方式则不会出现在链接中，因此，登录表单为了安全性大多采用Post方式提交。

我们来看看Servlet给我们返回了什么东西：

![img](https://img-blog.csdn.net/20180514171653913?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

正如我们在Servlet中写的那样，Servlet把HTTP请求中的部分参数给解析出来了。

因此，可以再翻到上面的Servlet重新去理解一遍Servlet的工作原理，可能会有更清晰的认识。

## 三十一、Servlet的局限性

我们已经看到，Servlet如果需要给客户端返回数据，比如像下面这样的一个HTML文件：

![img](https://img-blog.csdn.net/20180514172209114?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Servlet内部需要这样写输出语句：

```html
PrintWriter writer = response.getWriter();
writer.write("<!DOCTYPE html>\n" +
        "<html>\n" +
        "\t<head>\n" +
        "\t\t<meta charset=\"UTF-8\">\n" +
        "\t\t<title>标题标签</title>\n" +
        "\t</head>\n" +
        "\t<body>\n" +
        "\t\t<!--标题标签-->\n" +
        "\t\t<h1>公司简介</h1><br />\n" +
        "\t\t<h2>公司简介</h2><br />\n" +
        "\t\t<h3>公司简介</h3><br />\n" +
        "\t\t<h4>公司简介</h4><br />\n" +
        "\t\t\n" +
        "\t\t<!--加入一条水平线-->\n" +
        "\t\t<hr />\n" +
        "\t\t\n" +
        "\t\t<h5>公司简介</h5><br />\n" +
        "\t\t<h6>公司简介</h7><br />\n" +
        "\t\t<h100>公司简介</h100>\n" +
        "\t</body>\n" +
        "</html>\n");
```

即一行一行的把HTML语句给用Writer输出，早期简单的网页还能应付得住，但是随着互联网的不断发展，网站的内容和功能越来越强大，一个普通的HTML文件可能就达到好几百行，如果在采用使用Servlet去一行一行的输出HTML代码的话，将会非常的繁琐并且浪费大量的时间，且在当时，出现了PHP这种可以内嵌到HTML文件的动态语言，使得制作动态网页变得异常的简单和轻松，因此大量的程序员转上了PHP语言的道路，JAVA的份额急剧减小，当时JAVA的开发者Sun公司为了解决这个问题，也开发出了自己的动态网页生成技术，使得同样可以在HTML文件里内嵌JAVA代码，这就是现在的JSP技术，关于JSP技术的具体内容，我们将留到下一节进行讲解。

## 三十二、ServletContextListener（Servlet全局监听器）

首先要说明的是，ServletContextListener是一个接口，我们随便写一个类，只要这个类实现了ServletContextListener接口，那么这个类就实现了【监听ServletContext】的功能。那么，这个神奇的接口是如何定义的呢？我们来看一下这个接口的内部情况：

```java
package javax.servlet;

import java.util.EventListener;

public interface ServletContextListener extends EventListener {
    void contextInitialized(ServletContextEvent var1);

    void contextDestroyed(ServletContextEvent var1);
}
```

我们发现，在这个接口中只声明了两个方法，分别是void contextInitialized(ServletContextEvent var1)和void contextDestroyed(ServletContextEvent var1)方法，所以，我们很容易的就能猜测到，ServletContext的生命只有两种，分别是：

1.ServletContext初始化。（应用start时）---------->Servlet容器调用void contextInitialized(ServletContextEvent var1)

2.ServletContext销毁。（应用stop时）---------->Servlet容器调用 void contextDestroyed(ServletContextEvent var1)

因此，我们大概能够猜到ServletContextListener的工作机制了，当应用启动时，ServletContext进行初始化，然后Servlet容器会自动调用正在监听ServletContext的ServletContextListener的void contextInitialized(ServletContextEvent var1)方法，并向其传入一个ServletContextEvent对象。当应用停止时，ServletContext被销毁，此时Servlet容器也会自动地调用正在监听ServletContext的ServletContextListener的void contextDestroyed(ServletContextEvent var1)方法。

为了验证我们的猜测，我们来随便写一个类，并且实现ServletContextListener接口，即实现监听ServletContext的功能：

```java
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContextListener.contextInitialized方法被调用");
    }
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContextListener.contextDestroyed方法被调用");
    }
}
```

然后，在web.xml中注册我们自己写的这个MyListener:

```html
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <listener>
        <listener-class>MyListener</listener-class>
    </listener>
</web-app>
```

接下来，让我们启动一下Tomcat，看一看会发生什么吧！控制台打印信息如下：

![img](https://img-blog.csdnimg.cn/20181222161836740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5,size_16,color_FFFFFF,t_70)

我们发现，当应用启动时，ServletContextListener.contextInitialized（）方法被调用了。这其实是Servlet容器偷偷干的事情。那么，当我们停止Tomcat时，按照猜想，Servlet容器应该也会偷偷调用void contextDestroyed(ServletContextEvent var1)方法，来通知ServletContextListener监听器：ServletContext已经被销毁了。那么，事实是不是和我们猜想的一模一样呢？让我们来停止Tomcat的运行，看一看控制台的情况吧：
我们发现，void contextDestroyed(ServletContextEvent var1)方法确实被Servlet容器调用了。因此，我们的猜想得到了证实。

## 三十三、【进阶】ServletContextListener在Spring中的应用

如果基础好一点的童鞋，或者已经学过Spring框架的同学，建议阅读下面的内容，没有学过Spring也没有关系，可以先学或者学完之后再回头来看一看，Spring容器是如何借用ServletContextListener这个接口来实例化的。

首先让我们再来回顾一下ServletContext的概念，ServletContext翻译成中文叫做“Servlet上下文”或者“Servlet全局”，但是这个翻译我认为翻译的实在是有点牵强，也导致了许多的开发者不明白这个变量到底具体代表了什么。其实ServletContext就是一个“域对象”，它存在于整个应用中，并在在整个应用中有且仅有1份，它表示了当前整个应用的“状态”，你也可以理解为某个时刻的ServletContext代表了这个应用在某个时刻的“一张快照”，这张“快照”里面包含了有关应用的许多信息，应用的所有组件都可以从ServletContext获取当前应用的状态信息。ServletContext随着程序的启动而创建，随着程序的停止而销毁。通俗点说，我们可以往这个ServletContext域对象中“存东西”，然后也可以在别的地方中“取出来”。

我们知道，Spring容器可以通过：

ApplicationContext ctx=new ClassPathXmlApplicationContext("配置文件的路径"）;

显示地实例化一个Spring IOC容器。也可以像下面一样，在web.xml中注册Spring IOC容器：

```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        classpath:applicationContext.xml
    </param-value>
</context-param>
```

其中的监听器类【org.springframework.web.context.ContextLoaderListener】实现了ServletContextListener接口，能够监听ServletContext的生命周期中的“初始化”和“销毁”。注意，这个【org.springframework.web.context.ContextLoaderListener】监听器类当然不是我们自己写的哦，是人家Spring团队写的，我们只要拿来用就行了。当然，别忘记导入相关的Jar包。（spring-web-4.2.4.RELEASE.jar）

那么，Spring团队给我们提供的这个监听器类是如何实现：当ServletContext初始化后，Spring IOC容器也能跟着初始化的呢？怀着好奇心，让我们再来看一看【org.springframework.web.context.ContextLoaderListener】的内部实现情况吧。

```java
package org.springframework.web.context;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }
    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }
--------------------------------------------------------重点关注下面这里哦！-----------------------------------------------------------------------

    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }
--------------------------------------------------------重点关注上面这里哦！-----------------------------------------------------------------------

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
```

我们发现，【org.springframework.web.context.ContextLoaderListener】这个类实现了ServletContextListener接口中的两个方法，其中，当ServletContext初始化后， public void contextInitialized(ServletContextEvent event)方法被调用，接下来执行initWebApplicationContext(event.getServletContext())方法，但是我们发现这个方法并没有在这个类中声明，因此，我们再看一下其父类中是如何声明的：

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
 
    if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
 
        throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
 
    } else {
 
        Log logger = LogFactory.getLog(ContextLoader.class);
 
        servletContext.log("Initializing Spring root WebApplicationContext");
 
        if (logger.isInfoEnabled()) {
 
            logger.info("Root WebApplicationContext: initialization started");
 
        }
 
 
 
        long startTime = System.currentTimeMillis();
 
 
 
        try {
 
            if (this.context == null) {
 
                this.context = this.createWebApplicationContext(servletContext);
 
            }
 
 
 
            if (this.context instanceof ConfigurableWebApplicationContext) {
 
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)this.context;
 
                if (!cwac.isActive()) {
 
                    if (cwac.getParent() == null) {
 
                        ApplicationContext parent = this.loadParentContext(servletContext);
 
                        cwac.setParent(parent);
                    }
                    this.configureAndRefreshWebApplicationContext(cwac, servletContext);
                }
            }       servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
            ClassLoader ccl = Thread.currentThread().getContextClassLoader();
            if (ccl == ContextLoader.class.getClassLoader()) {
               currentContext = this.context;
            } else if (ccl != null) {
                currentContextPerThread.put(ccl, this.context);
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" + WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
            }
            if (logger.isInfoEnabled()) {
                long elapsedTime = System.currentTimeMillis() - startTime;
                logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
            }
            return this.context;
        } catch (RuntimeException var8) {
            logger.error("Context initialization failed", var8);     servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, var8);
            throw var8;
        } catch (Error var9) {
            logger.error("Context initialization failed", var9);
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, var9);
            throw var9;
        }
    }
}
```

分析到这一步，我们发现Spring容器在这个方法中被实例化了。接下来，就让我们整理一下整体的思路：

当Servlet容器启动时，ServletContext对象被初始化，然后Servlet容器调用web.xml中注册的监听器的

public void contextInitialized(ServletContextEvent event)

方法，而在监听器中，调用了this.initWebApplicationContext(event.getServletContext())方法，在这个方法中实例化了Spring IOC容器。即ApplicationContext对象。

因此，当ServletContext创建时我们可以创建applicationContext对象，当ServletContext销毁时，我们可以销毁applicationContext对象。这样applicationContext就和ServletContext“共生死了”。
————————————————
版权声明：本文为CSDN博主「刘扬俊」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_19782019/article/details/80292110

