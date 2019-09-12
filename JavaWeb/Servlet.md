---
typora-copy-images-to: img
---

# JavaWeb-Servlet

## 一. HTTP协议

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

**HTTP是一个基于TCP/IP通信协议来传递数据**（HTML 文件, 图片文件, 查询结果等）。

### 1. HTTP工作原理

**HTTP协议工作于客户端-服务端架构为上。**浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。
Web服务器有：Apache服务器，IIS服务器（Internet Information Services）等。
Web服务器根据接收到的请求后，向客户端发送响应信息。

HTTP默认端口号为80，但是你也可以改为8080或者其他端口。

**HTTP三点注意事项：**

- **HTTP1.0是无连接**：**无连接的含义是限制每次连接只处理一个请求(HTTP1.0版本)。服务器处理完客户的请求，并收到客户的应答后，即断开连接。**采用这种方式可以节省传输时间。**而HTTP1.1在同一个连接中可以传送多个请求和响应，多个请求可以重叠和同时进行，HTTP1.1必须有Host字段。HTTP 1.1默认进行持久连接**
- **HTTP是媒体独立的**：**这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。**客户端以及服务器指定使用适合的MIME-type内容类型。
- **HTTP是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。**

### 2. HTTP 消息结构

**HTTP是基于客户端/服务端（C/S）的架构模型，通过一个可靠的链接来交换信息，是一个无状态的请求/响应协议。**
一个HTTP"客户端"是一个应用程序（Web浏览器或其他任何客户端），通过连接到服务器达到向服务器发送一个或多个HTTP的请求的目的。
**一个HTTP"服务器"同样也是一个应用程序（通常是一个Web服务，如Apache Web服务器或IIS服务器等），通过接收客户端的请求并向客户端发送HTTP响应数据。**
HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。

一旦建立连接后，数据消息就通过类似Internet邮件所使用的格式[RFC5322]和多用途Internet邮件扩展（MIME）[RFC2045]来传送。

### 3. 客户端请求结构

**一个HTTP请求报文由四个部分组成：请求行、请求头部、空行、请求数据。**

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：

- 请求行（request line）

  - **请求行由请求方法字段、URL字段和HTTP协议版本字段3个字段组成，它们用空格分隔。**比如 `GET /data/info.html HTTP/1.1`;方法字段就是HTTP使用的请求方法，比如常见的GET/POST

  - 其中HTTP协议版本有两种：**HTTP1.0/HTTP1.1** 可以这样区别：

    **HTTP1.0对于每个连接都只能传送一个请求和响应，请求就会关闭，HTTP1.0没有Host字段;而HTTP1.1在同一个连接中可以传送多个请求和响应，多个请求可以重叠和同时进行，HTTP1.1必须有Host字段。**

- 请求头部（header）

  - **HTTP客户程序(例如[浏览器](http://www.2cto.com/os/liulanqi/))，向服务器发送请求的时候必须指明请求类型(一般是GET或者 POST)。如有必要，客户程序还可以选择发送其他的请求头。**大多数请求头并不是必需的，但Content-Length除外。对于POST请求来说 Content-Length必须出现。

- 空行

  - 它的作用是通过一个空行，告诉服务器请求头部到此为止。

- 请求数据

  - **若方法字段是GET，则此项为空，没有数据;若方法字段是POST,则通常来说此处放置的就是要提交的数据**比如要使用POST方法提交一个表单，其中有user字段中数据为“admin”, password字段为123456，那么这里的请求数据就是 user=admin&password=123456，使用&来连接各个字段。



![å¨è¿éæå¥å¾çæè¿°](.\img\2019051811435083.png)

![è¿éåå¾çæè¿°](.\img\70)

**常见的请求头: **

```
Accept： 浏览器可接受的MIME类型。

Accept-Charset：浏览器可接受的字符集。

Accept-Encoding：浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间。

Accept-Language：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。

Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中。

Content-Length：表示请求消息正文的长度。

Host： 客户机通过这个头告诉服务器，想访问的主机名。Host头域指定请求资源的Intenet主机和端口号，必须表示请求url的原始服务器或网关的位置。HTTP/1.1请求必须包含主机头域，否则系统会以400状态码返回。

If-Modified-Since：客户机通过这个头告诉服务器，资源的缓存时间。只有当所请求的内容在指定的时间后又经过修改才返回它，否则返回304“Not Modified”应答。

Referer：客户机通过这个头告诉服务器，它是从哪个资源来访问服务器的(防盗链)。包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。

User-Agent：User-Agent头域的内容包含发出请求的用户信息。浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用。

Cookie：客户机通过这个头可以向服务器带数据，这是最重要的请求头信息之一。

Pragma：指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝。

From：请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它。

Connection：处理完这次请求后是否断开连接还是继续保持连接。如果Servlet看到这里的值为“Keep- Alive”，或者看到请求使用的是HTTP 1.1(HTTP 1.1默认进行持久连接)，它就可以利用持久连接的优点，当页面包含多个元素时(例如Applet，图片)，显著地减少下载所需要的时间。要实现这一点，Servlet需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入 ByteArrayOutputStream，然后在正式写出内容之前计算它的大小。

Range：Range头域可以请求实体的一个或者多个子范围。例如，

表示头500个字节：bytes=0-499

表示第二个500字节：bytes=500-999

表示最后500个字节：bytes=-500

表示500字节以后的范围：bytes=500-

第一个和最后一个字节：bytes=0-0,-1

同时指定几个范围：bytes=500-600,601-999

但是服务器可以忽略此请求头，如果无条件GET包含Range请求头，响应会以状态码206(PartialContent)返回而不是以200 (OK)。

UA-Pixels，UA-Color，UA-OS，UA-CPU：由某些版本的IE浏览器所发送的非标准的请求头，表示屏幕大小、颜色深度、操作系统和CPU类型。
```

```
请求行
    * 请求方式
        主要有两种，get和post。之间区别：get拼接到地址栏，不安全，post封装在请求体，可进行大数据的操作。
    * 提交的地址 
    * 协议版本  HTTP/1.1
请求头
    Accept: text/html,image/*    
    Accept-Charset: ISO-8859-1
    Accept-Encoding: gzip
    Accept-Language:zh-cn 
    Host: www.itcast.com:80
    If-Modified-Since: Tue, 11 Jul 2000 18:23:51 GMT
    Referer: http://www.itcast.com/index.jsp
    User-Agent: Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.0)
    Connection: close/Keep-Alive   
    Date: Tue, 11 Jul 2000 18:23:51 GMT
    * 重点的有
        * If-Modified-Since     需要和响应头和304（状态码）和在一起使用，控制本地的缓存。
        * Referer               记住当前网页的来源（作用：统计网站的访问，防止盗链）
        * User-Agent                获取浏览器的版本信息
空行
请求体
    * 封装的是post提交方式的参数列表。
```

### 4. 服务器响应消息

响应体就是响应的消息体，如果是纯数据就是返回纯数据，如果请求的是HTML页面，那么返回的就是HTML代码，如果是JS就是JS代码，如此之类。

HTTP响应也由四个部分组成，分别是：

- 状态行
- 消息报头
- 空行
  - 它的作用是通过一个空行，告诉服务器请求头部到此为止。
- 响应正文。

![å¨è¿éæå¥å¾çæè¿°](.\img\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1NjUxMjc=,size_16,color_FFFFFF,t_70)

**消息报头: **

```
Allow	
服务器支持哪些请求方法（如GET、POST等）。

Content-Encoding	
文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。

Content-Length	
表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。

Content-Type	
表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。

Date	
当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。

Expires	
应该在什么时候认为文档已经过期，从而不再缓存它？

Last-Modified	
文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。

Location	
表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。

Refresh	
表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。
注意这种功能通常是通过设置HTML页面HEAD区的＜META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path"＞实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。
注意Refresh的意义是"N秒之后刷新本页面或访问指定页面"，而不是"每隔N秒刷新本页面或访问指定页面"。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是＜META HTTP-EQUIV="Refresh" ...＞。
注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。

Server	
服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。

Set-Cookie	
设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。

WWW-Authenticate	
客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。
注意Servlet一般不进行这方面的处理，而是让Web服务器的专门机制来控制受密码保护页面的访问（例如.htaccess）。
```

```
* 响应行
    * 协议版本
    * 状态码（重点记住）
        * 200 ：请求成功处理，一切OK      
        * 302 ：请求重定向
        * 304 ：服务器端资源没有改动，通知客户端查找本地缓存 
        * 404 ：客户端访问资源不存在
        * 500 ：服务器内部出错 
 
    * 状态码描述
* 响应头
    Location: http://www.it315.org/index.jsp 
    Server:apache tomcat
    Content-Encoding: gzip 
    Content-Length: 80 
    Content-Language: zh-cn 
    Content-Type: text/html; charset=GB2312 
    Last-Modified: Tue, 11 Jul 2000 18:23:51 GMT
    Refresh: 1;url=http://www.it315.org
    Content-Disposition: attachment; filename=aaa.zip
    Expires: -1
    Cache-Control: no-cache  
    Pragma: no-cache   
    Connection: close/Keep-Alive   
    Date: Tue, 11 Jul 2000 18:23:51 GMT
 
    * 重点的响应头    
        * Location                  和302一起完成重定向
        * Last-Modified 和 If-Modified-Since  和304一起来完成控制缓存的操作。
        * Refresh                   定时页面刷新（页面定时跳转）
        * Content-Disposition       文件下载的时候需要使用     
        * 下面这三个头需要一起使用
            Expires: -1
            Cache-Control: no-cache  
            Pragma: no-cache
            作用：禁用浏览器缓存。
 
* 空行
* 响应体：服务器向客户端返回的数据。
```

**get和post: **

1.提交数据的形式：

GET请求的数据会附在URL之后(就是把数据放置在HTTP协议头中)，会直接展现在地址栏中，以?分割URL和传输数据，参数之间以&相连，如：login.action?name=hyddd&password=idontknow&verify=%E4%BD%A0%E5 %A5%BD。

如果数据是英文字母/数字，原样发送，如果是空格，转换为+，如果是中文/其他字符，则直接把字符串用BASE64加密，

得出如：%E4 %BD%A0%E5%A5%BD，其中%XX中的XX为该符号以16进制表示的ASCII。

而POST方法则会把数据放到请求数据字段中以&分隔各个字段，请求行不包含数据参数，地址栏也不会额外附带参数

2.提交数据的大小

get方法提交数据的大小直接影响到了URL的长度，但HTTP协议规范中其实是没有对URL限制长度的，限制URL长度的是客户端或服务器的支持的不同所影响：比如IE对URL长度的限制是2083字节(2K+35)。对于其他浏览器，如Netscape、FireFox等，理论上没有长度限制，其限制取决于操作系统的支持。

post方式HTTP协议规范中也没有限定，起限制作用的是服务器的处理程序的处理能力。

所以大小的限制还是得受各个web服务器配置的不同而影响。

3.提交数据的安全

POST比GET方式的安全性要高

**通过GET提交数据，用户名和密码将明文出现在URL上**，因为一下几个原因get方式安全性会比post弱：

(1)登录页面有可能被浏览器缓存

(2)其他人查看浏览器的历史纪录，那么别人就可 以拿到你的账号和密码了

(3)当遇上跨站的攻击时，安全性的表现更差了



## 二. Cookie

http是无状态协议.

**Cookie的诞生**

由于HTTP协议是无状态的，而服务器端的业务必须是要有状态的。Cookie诞生的最初目的是为了存储web中的状态信息，以方便服务器端使用。比如判断用户是否是第一次访问网站。目前最新的规范是RFC 6265，它是一个由浏览器服务器共同协作实现的规范。

**Cookie的处理分为：**

服务器向客户端发送cookie

浏览器将cookie保存

之后每次http请求浏览器都会将cookie发送给服务器端

Cookie 是存储在客户端计算机上的文本文件，并保留了各种跟踪信息。

**识别返回用户包括三个步骤：**

- 服务器脚本向浏览器发送一组 Cookie。例如：姓名、年龄或识别号码等。
- 浏览器将这些信息存储在本地计算机上，以备将来使用。
- 当下一次浏览器向 Web 服务器发送任何请求时，浏览器会把这些 Cookie 信息发送到服务器，服务器将使用这些信息来识别用户。

![](.\img\3a3fa6208a502a1137e5cf33b970e0ec.png)

```
1）首先浏览器向服务器发出请求。

2）服务器就会根据需要生成一个Cookie对象，并且把数据保存在该对象内。

3）然后把该Cookie对象放在响应头，一并发送回浏览器。

4）浏览器接收服务器响应后，提出该Cookie保存在浏览器端。

5）当下一次浏览器再次访问那个服务器，就会把这个Cookie放在请求头内一并发给服务器。

6) 服务器从请求头提取出该Cookie，判别里面的数据，然后作出相应的动作。
```

### 1.  创建Cookie

设置Cookie对象的名与值的保存时间,其中`setMaxAge`的参数如果为负数,退出浏览器删除Cookie;为0,立即删除Cookie;为正数,正数为多少则Cookie保存多少秒;

```java
    //1. 现在服务器端创建一个cookie,有两个参数,名与值, 其中值需要解码. 
    Cookie myCookie=new Cookie("color1",URLEncoder.encode(“值”,”utf-8”));  

    //2. 该cookie存在的时间 以秒为单位  
    myCookie.setMaxAge(30000);  
    //如果你不设置存在时间,那么该cookie将不会保存  

    //3. 将该cookie写回到客户端  
    res.addCookie(myCookie);  

    pw.println("已经创建了cookie");  
```

### 2. 获取Cookie

通过请求对象Request的方法`getCookies()`,可以获取客户端中所有的Cookie信息,因为浏览器中可以保存多个Cookie,所以该方法返回的是一个Cookie对象类型的数组;

`Cookie[] cookies = req.getCookies();`

```java
    4.//从客户端得到所有cookie信息  
    Cookie [] allCookies=req.getCookies();  

    int i=0;  
    //如果allCookies不为空...  
    if(allCookies!=null){  

        //从中取出cookie  
        for(i=0;i<allCookies.length;i++){  

            //依次取出  
            Cookie temp=allCookies[i];  

            if(temp.getName().equals("color1")){  

                //得到cookie的值  
                String val=URLEncoder.encode(temp.getValue(),”utf-8”);  

                pw.println ("color1="+val);  
                break;  

            }  
        }  
        if(allCookies.length==i){  

            pw.println("cookie 过期");  
        }  

    }else{            
        pw.println ("不存在color1这个cookie/或是过期了!");  
    } 
```

### 3. 删除Cookie

```java
	Cookie temp = cookies[i];
    //通过Cookie的名找到Cooike数组中对应的Cookie
    if(cookieName.equals("name")){
        //把符合条件的Cookie的保存时间改为0秒,参数为0,即立即删除该Cookie,-1为退出浏览器删除Cookie.
        cookies[i].setMaxAge(0);
        //将修改后的Cookie通过addCookie发送至客户端修改信息.
        resp.addCookie(cookies[i]);
        resp.getWriter().println("Sex删除成功!");
    }
```

### 4. Cookie存储中文

Cookie对象的值参数通过`RULEncoder`的方法`encoder()`来进行编码,编码集为utf-8来存储中文 ;

```java
Cookie cookie = new Cookie("name", RULEncoder.encoder(value, encoding));
```

通过`RULDecoder.decode(cookies[index].getName,”utf-8”)`可以将Cookie保存的名与值改为对应的编码集;

```java
Cookie[] cookies = HttpServletRequest.getCookies();
String value = URLDecoder.decoder(cookies[index].getValue(), encoding);
```

### 5. Cookie存活时间

- 默认情况下(不设置Cookie的声明周期, setMaxAge的值为-1), 当浏览器关闭后, Cookie数据被销毁;**存储在浏览器内存中;**

- 正数: 将Cookie写入到硬盘中. 持久化存储; 并指定Cookie存活时间, 时间到, cookie文件自动失效
- 负数: 当浏览器关闭后, Cookie数据被销毁;**存储在浏览览器内存中;**
- 零: 删除指定cookie的信息

### 6. Cookie共享问题

**1. 假设在一个Tomcat中, 部署了多个Web项目, 那么在这些Web项目中cookie能不能共享**

- 默认情况下, cookie不能共享
- `setPath(Stirng path)`: 设置cookie的获取范围. 默认情况下, 设置当前的虚拟目录; 如果要共享, 则可以路径设置为 `"/"`

**2. 不同的Tomcat服务器间cookie共享问题?**

`setDomain(String path)`: **如果设置一级域名相同, 那么多个服务器之间cookie可以共享**

```java
//这样设置, tieba.baidu.com和news.baidu.com中cookie可以共享
setDomain(".baidu.com");
```

### 7. Cookie的特点和作用

**特点: **

- cookie存储数据在客户端和浏览器
- 浏览器对于单个cookie的大小有限制(4kb, 不同的浏览器限制不同)以及对同一个域名下的总cookie数量也有限制(20个)

**作用: **

- cookie一般用于存储少量的不太敏感的信息
- 在不登录的情况下, 完成服务器对客户端的身份识别



## 三. Session

**原理: **

用户第一次访问服务器，服务器会创建一个session对象给此用户，并将**该session对象的JSESSIONID使用Cookie技术存储到浏览器中**，保证用户的其他请求能够获取到同一个session对象，也保证了不同请求能够获取到共享的数据。

**特点: **

- 存储在服务器端
- 服务器进行创建
- **依赖Cookie技术**
- **默认存储时间是30分钟**

> **session机制是一种服务器端的机制**，服务器使用一种类似于散列表的结构(也可能就是使用散列表)来保存信息。
> **当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否包含了一个session标识－称为session id,如果已经包含一个session id则说明以前已经为此客户创建过session，服务器就按照session id把这个session检索出来使用**(如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的session对象，但用户人为地在请求的URL后面附加上一个JSESSION的参数)。
> **如果客户请求不包含session id，则为此客户创建一个session并且生成一个与此session相关联的session id，这个session id将在本次响应中返回给客户端保存。**

![](E:\软件开发资料\Java\Java资料\JavaWeb\img\14f8ea0cc93f012fdda055ea73d9d3ef.png)

```
1）浏览器发出请求到服务器。

2）服务器会根据需求生成Session对象，并且给这个Session对象一个编号，一个编号对应一个Session对象

3）服务器把需要记录的数据封装到这个Session对象里，然后把这个Session对象保存下来。

4）服务器把这个Session对象的编号放到一个Cookie里，随着响应发送给浏览器

5）浏览器接收到这个cookie就会保存下来

6）当下一次浏览器再次请求该服务器服务，就会发送该Cookie

7）服务器得到这个Cookie，取出它的内容，它的内容就是一个Session的编号！！！

8）凭借这个Session编号找到对应的Session对象，然后利用该Session对象把保存的数据取出来！
```



|                       方法                        | 解释                                                         |
| :-----------------------------------------------: | ------------------------------------------------------------ |
| void setAttribute(String attribute, Object value) | 设置Session属性。value参数可以为任何Java Object。通常为Java Bean。value信息不宜过大 |
|       String getAttribute(String attribute)       | 返回Session属性                                              |
|      void removeAttribute(String attribute)       | 移除Session属性                                              |
|                  String getId()                   | 返回Session的ID。该ID由服务器自动创建，不会重复              |
|              long getCreationTime()               | 返回Session的创建日期。                                      |
|            long getLastAccessedTime()             | 返回Session的最后活跃时间。返回类型为long                    |
|           int getMaxInactiveInterval()            | 返回Session的超时时间。单位为秒。超过该时间没有访问，服务器认为该Session失效 |
|      void setMaxInactiveInterval(int second)      | 设置Session的超时时间。单位为秒                              |

### 1.创建Session对象

`HttpSession session = request.getSession();`

`HttpSession session = request.getSession(true);`

`HttpSession session = request.getSession(false);`

`getSession()/getSession(true)`：**当session存在时返回该session，否则新建一个session并返回该对象**
`getSession(false)`：**当session存在时返回该session，否则不会新建session，返回null**

**如果请求中拥有session的标识符也就是JSESSIONID，则返回其对应的session队形**

**如果请求中没有session的标识符也就是JSESSIONID，则创建新的session对象，并将其JSESSIONID作为从cookie数据存储到浏览器内存中**

**如果session对象是失效了，也会重新创建一个session对象，并将其JSESSIONID存储在浏览器内存中。**

**设置session存储时间:** 

`session.setMaxInactiveInterval(int seconds);` 单位为秒

**web.xml(Tomcat服务器或者web应用程序)文件中设置: **

```xml
<session-config>
     <!-- 单位为分钟 -->
    <session-timeout>30</session-timeout>
</session-config>
```

**注意：**

​	**在指定的时间内session对象没有被使用则销毁，*如果使用了则重新计时。***



### 2. 删除Session

- 程序调用`HttpSession.invalidate();` **使`session`强制失效**
- **距离上一次收到客户端发送的session id时间间隔超过了session的最大有效时间**
- 服务器进程被停止

**注意: **

关闭浏览器只会使存储在客户端浏览器内存中的session cookie失效，不会使服务器端的session对象失效。

### 3. 存储和获取数据

存储：`session.setAttribute(String name,Object value);`

获取：`session.getAttribute(String name)` 返回的数据类型为Object

`setAttribute`会替换任何之前设定的值；如果想要在不提供任何代替的情况下移除某个值，则应使用`removeAttribute`。这个方法会触发所有实现了HttpSessionBindingListener接口的值的valueUnbound
方法。

清除: `session.removeAttribute(String name)`: 移除某个值

**注意：**

*存储的动作和取出的动作发生在不同的请求中，但是存储要先于取出执行。*





























