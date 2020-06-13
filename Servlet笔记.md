# Servlet笔记

## 1. Tomcat

Tomcat的功能：

> * Web服务器
> * JSP/Servlet容器 

浏览器请求服务器所做的工作：

浏览器 -> 本地host/DNS->目的服务器->JSP/Servlet->数据库

这是早期的JavaWeb架构，所有的请求交由JSP/Servlet来处理。



## 2. HTTP协议

**HTTP协议：**

> 超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW文件都必须遵守这个标准。它是TCP/IP协议的一个**应用层协议**

**简单来说，HTTP协议就是客户端和服务器交互的一种通讯格式。**

一般客户端发起一个HTTP请求到服务端，服务端进行请求处理，处理完成后，封装一个HTTP响应给客户端(浏览器)。

### HTTP请求

一个HTTP请求包含三个部分：

* 1. 请求行 (描述客户端的**请求方式**、**请求的资源名称**，以及使用的**HTTP协议版本号**)
* 2. 请求头 (描述客户端请求哪台主机，以及**客户端的一些环境信息**等)
* 3. 一个空行
* 4. 请求实体

HTTP请求行：一个HTTP请求行包含请求方法，请求资源的路径，HTTP协议版本	

> GET /java.html HTTP/1.1

HTT请求方法包含：**POST,GET,HEAD,OPTIONS,DELETE,TRACE,PUT** 

> - GET： 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器
> - POST：用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。
> - PUT： 传输文件，报文主体中包含文件内容，保存到对应URI位置。
> - HEAD： 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
> - DELETE：删除文件，与PUT方法相反，删除对应URI位置的文件。
> - OPTIONS：查询相应URI支持的HTTP方法。

HTTP请求头：

> - Accept: text/html,image/*    【浏览器告诉服务器，它支持的数据类型】
> - Accept-Charset: ISO-8859-1	【浏览器告诉服务器，它支持哪种**字符集**】
> - Accept-Encoding: gzip,compress 【浏览器告诉服务器，它支持的**压缩格式**】
> - Accept-Language: en-us,zh-cn 【浏览器告诉服务器，它的语言环境】
> - Host: [www.it315.org:80](www.it315.org:80)【浏览器告诉服务器，它的想访问哪台主机】
> - If-Modified-Since: Tue, 11 Jul 2000 18:23:51 GMT【浏览器告诉服务器，缓存数据的时间】
> - Referer: http://www.it315.org/index.jsp【浏览器告诉服务器，客户机是从那个页面来的---**反盗链**】
> - 8.User-Agent: Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.0)【浏览器告诉服务器，浏览器的内核是什么】
> - Cookie【浏览器告诉服务器，**带来的Cookie是什么**】
> - Connection: close/Keep-Alive  【浏览器告诉服务器，请求完后是断开链接还是保持链接】 
> - Date: Tue, 11 Jul 2000 18:23:51 GMT【浏览器告诉服务器，请求的时间】 

### HTTP响应

一个完整的HTTP响应应该包含四个部分:

* 1. 一个状态行【用于描述**服务器对请求的处理结果。**】
* 2. 响应头【用于描述**服务器的基本信息**，以及**数据的描述**，**服务器通过这些数据的描述信息，可以通知客户端如何处理等一会儿它回送的数据**】
* 3. 一个空行
* 4. 响应实体【**服务器向客户端回送的数据**】

状态行：

> * 100~199 表示成功接受请求，要求客户端继续提交下一次请求才能完成整个处理过程
> * 200~299 表示成功接收请求并已完成整个处理过程。
> * 300~399 表示请求的资源已经移动到一个新的地址(跳转)
> * 400~499 客户端请求出错
> * 500~599 服务器出现错误

响应头：

> - Location: http://www.it315.org/index.jsp 【服务器告诉浏览器**要跳转到哪个页面**】
> - Server:apache tomcat【服务器告诉浏览器，服务器的型号是什么】
> - Content-Encoding: gzip 【服务器告诉浏览器**数据压缩的格式**】
> - Content-Length: 80 【服务器告诉浏览器回送数据的长度】
> - Content-Language: zh-cn 【服务器告诉浏览器，服务器的语言环境】
> - Content-Type: text/html; charset=GB2312 【服务器告诉浏览器，**回送数据的类型**】
> - Last-Modified: Tue, 11 Jul 2000 18:23:51 GMT【服务器告诉浏览器该资源上次更新时间】
> - Refresh: 1;url=http://www.it315.org【服务器告诉浏览器要**定时刷新**】
> - Content-Disposition: attachment; filename=aaa.zip【服务器告诉浏览器**以下载方式打开数据**】
> - Transfer-Encoding: chunked  【服务器告诉浏览器数据以分块方式回送】
> - Set-Cookie:SS=Q0=5Lb_nQ; path=/search【服务器告诉浏览器要**保存Cookie**】
> - Expires: -1【服务器告诉浏览器**不要设置缓存**】
> - Cache-Control: no-cache  【服务器告诉浏览器**不要设置缓存**】
> - Pragma: no-cache   【服务器告诉浏览器**不要设置缓存**】
> - Connection: close/Keep-Alive   【服务器告诉浏览器连接方式】
> - Date: Tue, 11 Jul 2000 18:23:51 GMT【服务器告诉浏览器回送数据的时间】

**HTTP是无状态协议**：

> 无状态协议对于事务处理没有记忆能力**。**缺少状态意味着如果后续处理需要前面的信息 。也就是说，当客户端一次HTTP请求完成以后，客户端再发送一次HTTP请求，HTTP并不知道当前客户端是一个”老用户“。

**HTTP通过Cookie来解决这个问题。**

**一次HTTP连接：**

>建立TCP连接->客户端发送请求行->客户端发送请求头->服务器
>
>服务器发送状态行->服务器发送响应头->服务器发送响应体->TCP连接断开



## 3. Servlet

**什么是Servlet？**

> Servlet就是实现了Servlet接口的Java类。Servlet是由服务器调用，运行在服务器端。
>

### Servlet接口

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    void destroy();
    
    ServletConfig getServletConfig();
    
    String getServletInfo();
}
```

### Servlet的生命周期

1. 加载Servlet：当Tomcat第一次访问Servlet的时候，Tomcat会负责创建Servlet的实例

2. 初始化：当Servlet被实例化后，Tomcat会调用init()方法初始化这个对象

3. 处理服务：当浏览器访问Servlet的时候，Servlet 会调用service()方法处理请求

4. 销毁：当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，让该实例释放掉所占的资源。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁

5. 卸载：当Servlet调用完destroy()方法后，等待垃圾回收。如果有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作。

通过上面我们可以学习到：

* **只要访问Servlet，service()就会被调用。init()只有第一次访问Servlet的时候才会被调用。destroy()只有在Tomcat关闭的时候才会被调用**
* **Servlet是单例加载，浏览器会对Servlet进行多次请求 一般情况下服务器只创建一个Servlet对象 Servlet对象一旦创建 就会驻留在内存中。浏览器的每次请求访问都会创建一个新的HttpServletRequest和HttpServletResponse 然后将这两个对象传给Servlet的service方法**
* **当多个用户访问同一Servlet资源时 服务器会为每个用户创建一个线程 这可能出现线程安全问题 需要用syn进行同步锁**

###  HttpServlet

我们可以看到Servlet接口包含五个方法，如果我们每个Servlet都实现这个接口的话，势必会非常繁琐，所以Java帮我们做了一层封装，我们只需要继承**HttpServlet**父类即可。

HttpServlet类继承了Servlet接口，同时帮我们重写了接口方法，并且在service方法处理中，根据请求类型，创建了不同的响应方法，我们只需要根据自己的需要，重写父类里的响应方法即可。

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String returnMsg = "hello man we see again, i am get";
        OutputStream outputStream = resp.getOutputStream();
        outputStream.write(returnMsg.getBytes());
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello man we see again, i am post");
    }
}
```

### Servlet Mapping

一个Servlet的创建不光要继承HttpServlet(Servlet接口)，还要给出自己的Servlet映射。

试想这样一种情况：

你写了一个Servlet来处理用户下载QQ，这时客户端发送HTTP请求，HTTP请求会包含一个请求资源路径(URL链接)，服务器想去解析这个URI链接，以此来找到这是对应Web服务中的哪一个Servlet，找到后，再调用你这个Servlet处理用户的请求，但如果你没有写Servlet映射，Web服务器就找不到你的Servlt，自然也无法处理用户的请求。

所以Servlet映射的作用就是将URL路径与你的Servlet对应起来，类似于一张对应表，这样用户一请求，就能通过这个表找到是哪个Servlet。

```xml
<servlet>
    <servlet-name>myServlet</servlet-name>
    <servlet-class>Servlet.MyServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>myServlet</servlet-name>
    <url-pattern>/myServlet</url-pattern>
</servlet-mapping>
```

### ServletConfig

一些通用的Servlet初始化配置，比如你可以将JDBC的一些参数放到ServletConfig中，这样在每个Servlet中可以直接拿来用，而且因为在web.xml中配置，修改也简单，不用动代码。

```xml
 <context-param>
        <param-name>url</param-name>
        <param-value>jdbc:mysql://localhost:3306/smbms?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</param-value>
    </context-param>
    <context-param>
        <param-name>username</param-name>
        <param-value>root</param-value>
    </context-param>
    <context-param>
        <param-name>password</param-name>
        <param-value>root</param-value>
    </context-param>
```

```java
public class MyServlet extends HttpServlet {
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletConfig servletConfig = this.getServletConfig();
        String url = servletConfig.getInitParameter("url");
        String userName = servletConfig.getInitParameter("username");
        String password = servletConfig.getInitParameter("password");
    }
}    


```

### ServletContext（重点）

**ServletContext**代表着当前Web站点。

这样说可能难以理解，我们可以换种说法：ServletContext是你Web项目的**全局变量**。每个Servlet都可以访问ServletContext，当然访问的也是同一个ServletContext，这样我们的Servlet就可以通过ServletContext实现交互。

```java
public class ServletContext1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        servletContext.setAttribute("name","coderZoe");
    }

}
```

```java
public class ServletContext2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String name = servletContext.getAttribute("name");
        System.out.println(name);
    }

}
```

### HttpServletResponse

HttpServletResponse代表HTTP响应，通常我们可以将返回信息通过输出流要返给客户端。

```java
public class Servlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletOutputStream servletOutputStream = response.getOutputStream();
        response.setHeader("Content-type","text/html;charset = UTF-8");
        servletOutputStream.write(new String("Hello 客户端").getBytes(StandardCharsets.UTF_8));
    }
}
```

```java
public class Servlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       response.setContentType("text/html;charset=UTF-8");
        PrintWriter printWriter = response.getWriter();
        printWriter.write("Hello 客户端");
    }
}
```

**注：**

>* getWriter()和getOutputStream()两个方法不能同时调用。如果同时调用就会出现异常
>* Servlet程序向ServletOutputStream或PrintWriter对象中写入的数据将被Servlet引擎从response里面获取，Servlet引擎将这些数据当作响应消息的正文，然后再与响应状态行和各响应头组合后输出到客户端。
>* Servlet的serice()方法结束后【也就是doPost()或者doGet()结束后】，Servlet引擎将检查getWriter或getOutputStream方法返回的输出流对象是否已经调用过close方法，如果没有，Servlet引擎将调用close方法关闭该输出流对象.

通过输出流完成文件下载

```java
public class DownloadServlet extends HttpServlet {
   @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String path = this.getServletContext().getRealPath("/WEB-INF/resource/picture/西电.jpeg");
        String fileName = path.substring(path.lastIndexOf("\\") + 1);
        //告诉浏览器这是一个下载请求
        response.setHeader("Content-Disposition", "attachment; filename="+ URLEncoder.encode(fileName,"UTF-8"));
        try(FileInputStream fileInputStream = new FileInputStream(path);
            ServletOutputStream servletOutputStream = response.getOutputStream();) {
            int len = -1;
            byte[] bytes = new byte[1024];
            while ((len = fileInputStream.read(bytes))!=-1){
                servletOutputStream.write(bytes,0,len);
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

实现页面自动刷新

```java
public class Refresh extends HttpServlet {
    /**
     * 以规定的时间让页面自动刷新
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().write("3秒后页面将跳转");

        //设置3秒后跳转
        response.setHeader("Refresh","3;url = '/JavaWebLearn/index.jsp'");

        //设置取消缓存
        response.setDateHeader("Expires",-1);
        response.setHeader("Cache-Control","no-cache");
        response.setHeader("Pragma","no-cache");
    }
}

```

压缩

```java
public class Compression extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //对数据进行压缩 返给浏览器 浏览器再进行解码展示

        //gzip函数是一种压缩方式 他通过接受源数据的字节流，将源数据字节流写到一个字节输出流内 可以看出gzip是属于IO装饰流
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        GZIPOutputStream gzipOutputStream = new GZIPOutputStream(byteArrayOutputStream);
        String path = this.getServletContext().getRealPath("/WEB-INF/resource/picture/西电.jpeg");
        FileInputStream inputStream = new FileInputStream(path);

        int len = -1;
        byte[] bytes = new byte[1024];
        while ((len = inputStream.read(bytes))!=-1){
            gzipOutputStream.write(bytes,0,len);
        }
        //取出gzip写入后压缩的字节 传给浏览器
        byte[] result = byteArrayOutputStream.toByteArray();
        System.out.println(result.length);

        //告诉浏览器压缩格式 用于浏览器解析
        response.setHeader("Content-Encoding","gzip");
        response.getOutputStream().write(result);

        //关闭资源
        inputStream.close();
        gzipOutputStream.close();
    }
}
```

**重定向跳转**

> 什么是重定向跳转呢？点击一个超链接，**通知浏览器跳转到另外的一个页面**就叫重定向跳转。**是通知浏览器去跳转，这很重要。**页面之间的跳转有两种方式：**重定向和转发**

```java
public class Redirect extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //当请求这个Servlet时，重定向到index页面
        response.sendRedirect("/JavaWebLearn/index.jsp");
    }
}

```

### HttpServletRequest

HttpServletRequest对象代表客户端的请求，我们可以通过这个对象获得浏览器信息和请求时发过来的数据

1. 浏览器信息

   >* getRequestURL方法返回客户端发出请求时的完整URL。
   >* getRequestURI方法返回请求行中的资源名部分。
   >* getQueryString 方法返回请求行中的参数部分。
   >* getPathInfo方法返回请求URL中的额外路径信息。额外路径信息是请求URL中的位于Servlet的路径之后和查询参数之前的内容，它以“/”开头。
   >* getRemoteAddr方法返回发出请求的客户机的IP地址
   >* getRemoteHost方法返回发出请求的客户机的完整主机名
   >* getRemotePort方法返回客户机所使用的网络端口号
   >* getLocalAddr方法返回WEB服务器的IP地址。
   >* getLocalName方法返回WEB服务器的主机名

2. 请求头

   > * getHeader方法
   > * getHeaders方法
   > * getHeaderNames方法

3. 请求参数

   > * getParameter方法
   > * getParameterValues（String name）方法
   > * getParameterNames方法
   > * getParameterMap方法

防盗链

> 简单来说防盗链就是你如果想访问xxx资源网页，你就必须从yyy网页跳转过来，通过别的跳转过来(或直接URL请求)不行，我会重定向，让你回到yyy网页

```java
public class Referer extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //防盗链
        //通过request的header信息判断访问当前网页的来源 如果不是由导航窗口进来 那就直接跳转回导航窗口 让用户再点一遍
        //得到网站的来源并进行判断
        String referer = request.getHeader("Referer");
        if(referer==null||!referer.contains("http://localhost:8080/JavaWebLearn/index.jsp")){
            response.sendRedirect("/JavaWebLearn/index.jsp");
            return;
        }
        response.setContentType("text/html; charset = UTF-8");
        response.getWriter().write("这是我的资源页 你只能通过首页跳转过来");
    }
}

```

表单数据提交

HTML(JSP)页面写一个表单，表单请求指向我们的Servlet，在Servlet通过HttpServletRequest类获得用户表单传递的参数。

```jsp
  <form action="/JavaWebLearn/form" method="post">
    <table>
      <tr>
        <td>用户名</td>
        <td><input type="text" name = "userName"></td>
      </tr>
      <tr>
        <td>密码</td>
        <td><input type="password" name = "password"></td>
      </tr>
      <tr>
        <td>性别</td>
        <td><input type="radio" name = "sex" value="男">男</td>
        <td><input type="radio" name="sex" value="女">女</td>
      </tr>
      <tr>
        <td>爱好</td>
        <td><input type="checkbox" name="hobbies" value="打球">打球</td>
        <td><input type="checkbox" name="hobbies" value="电子游戏">电子游戏</td>
        <td><input type="checkbox" name="hobbies" value="唱歌">唱歌</td>
      </tr>
      <input type="hidden" name="test" value="my name is yhs">

      <tr>
        <td>你来自哪</td>
        <td><input type="text" name="from"></td>
      </tr>
      <tr>
        <td><input type="submit" value="提交">提交</td>
        <td><input type="reset" value="重置">重置</td>
      </tr>
    </table>
  </form>
```

```java
@WebServlet(name = "FormRequest",urlPatterns = "/form")
public class FormRequest extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding(StandardCharsets.UTF_8.toString());
        //获得form表单提交的信息
        String userName = request.getParameter("userName");
        System.out.println("userName"+userName);
        String password = request.getParameter("password");
        System.out.println("password"+password);
        String sex = request.getParameter("sex");
        System.out.println("sex"+sex);
        String[] hobbies = request.getParameterValues("hobbies");
        System.out.println("hobbies"+ Arrays.toString(hobbies));
        String hiddenInfo = request.getParameter("test");
        System.out.println("hiddenInfo"+hiddenInfo);
        String from = request.getParameter("from");
        System.out.println("from"+from);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
}
```

转发

在HttpServletResponse中说过浏览器的页面跳转有两种方式：重定向和转发

```java
@WebServlet(name = "Forward",urlPatterns = "/forward")
public class Forward extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //转发到index页面
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/index.jsp");
        requestDispatcher.forward(request,response);
    }
}
```

**注  重定向和转发的相同点与区别(重点)：**

1. 相同点

   >* response.sendRedirect()方法可以实现重定向 功能是页面跳转request.getRequestDispatcher.forward(request,response)实现转发，做到的功能也是页面跳转

2. 区别

   > 1. 发生位置不同 浏览器地址栏也不同
   >    * 重定向是由浏览器进行重新定向跳转的 url地址栏也会相应的变成重定向目的地的地址 相当于重定向的时候再次发生了一次http请求
   >    * 转发是由服务器实现的跳转 浏览器只进行了一次请求 服务器得到请求 然后转发给另一个servlet  另一个servlet处理完再返给浏览器
   >    * 转发的整个过程是只有一个request和response 所以浏览器的地址栏没变化
   >    * 重定向：浏览器请求->servlet1->返回浏览器->浏览器再次请求->servlet2->返回浏览器
   >    * 转发：浏览器请求->servlet1->servlet2->返回浏览器
   >
   >    所有我们可以在转发源的HttpServletRequest设置一些属性，而转发目的的HttpServletRequest可以获得这个属性
   >
   >    ```java
   >     request.setAttribute("username","yhs");
   >     RequestDispatcher requestDispatcher = request.getRequestDispatcher("/form");
   >     requestDispatcher.forward(request,response);
   >    ```
   >
   >    ```java
   >    String userName = request.getAttribute("username")
   >    ```
   >
   > 2. 请求的资源和写法不同
   >    * 重定向对于返回的url是完应用名+资源名 /JavaWebLearn/index.jsp
   >    * 而转发的资源地址是直接就是资源名 /form
   >    * 所以转发只能访问当前web的资源
   >    * 但重定向可以访问任意资源
   >    
   > 3. 传递数据的类型不同
   >    * 转发的request对象可以传递各种类型的数据，包括对象
   >    * 重定向只能传递字符
   >    
   > 4. 跳转的时间不同
   >    * 转发时：执行到跳转语句时就会立刻跳转
   >    * 重定向：整个页面执行完之后才执行跳转
   
3. 应用场景

   >**转发**: 访问 Servlet 处理业务逻辑，然后 forward 到 jsp 显示处理结果，浏览器里 URL 不变
   >
   >**重定向**: 提交表单，处理成功后 redirect 到另一个 jsp，防止表单重复提交，浏览器里 URL 变了



### Cookie

**会话：**

> 用户打开浏览器，访问一个网站，只要不关闭浏览器，无论用户点击多少超链接，访问多少资源，直到用户关闭浏览器，这个过程我们称为一次会话。

前面已经说过，HTTP请求是无状态请求，即每次HTTP请求都是新的。但在实际生活中，我们发现登录一次后，并不需要每次请求都再次登录。

**Cookie**

> W3C提出了Cookie：给每个用户颁发一个通行证，那么下次无论谁访问的时候，都需要携带通行证，这样服务器就可以通过HTTP带来的通行证来确定用户信息，这个通行证就叫Cookie。

客户端第一次请求：

Client->HttpRequest->Server->Response(Cookie)->Client

客户端第二次请求:

Client->HttpRequest(Cookie)->Server->Response->Client

浏览器访问服务器，**如果服务器需要记录该用户的状态，就使用response向浏览器发送一个Cookie，浏览器会把Cookie保存起来。当浏览器再次访问服务器的时候，浏览器会把请求的网址连同Cookie一同交给服务器**。

```java
package Servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

/**
 * @author yhs
 * @date 2020/4/29 14:47
 * @description cookie
 */
@WebServlet(name = "Cookie",urlPatterns = "/cookie")
public class Cookie extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");

        //创建cookie 设置编码格式 避免乱码
        Cookie cookie = new Cookie("username", URLEncoder.encode("殷华盛", "UTF-8"));  //key value的形式创建
        cookie.setMaxAge(1000*3600);    //设置cookie的有效期

        //将cookie加入response 并返给浏览器
        response.addCookie(cookie);
        response.getWriter().write("我颁给了浏览器一个cookie");
    }
}
```

**Cookie的一些属性**

* Cookie具有不可跨域性 简单来说就是一个网站的Cookie 你不会提交给另一个网站的服务器
* Cookie的有效期通过setMaxAge(int time)函数来设置
* 当time的值是负的时候 Cookie的值是临时性的 仅在本浏览器内有效 关闭浏览器就失效了 cookie也不会写到硬盘中 其中创建cookie时提供的默认值就是-1
* 当time = 0 时表示删除该Cookie cookie并没有提供删除机制 通过setMaxAge(0)达到删除的目的
* 当time是正数时 浏览器会将Cookie写入硬盘中 只要还在time秒前登录该网站就有效
* Cookie不存在修改的方法 只存在覆盖的方法 当Cookie的名相同通过response返给浏览器存储 原同名Cookie就会被覆盖
* 删除，修改Cookie时，新建的Cookie除了value、maxAge之外的所有属性都要与原Cookie相同，否则浏览器将视为不同的Cookie，不予覆盖，导致删除修改失败。
* Cookie本身可以被该项目下的所有web资源获取，但Cookie的path属性可以设置只能某个资源获取Cookie 其他不可以，Cookie.setPath("/getCookie") 这样只有getCookie路径的资源才能访问Cookie 其他不允许

```java
package Servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLDecoder;

/**
 * @author yhs
 * @date 2020/4/29 15:07
 * @description
 */
@WebServlet(name = "GetCookie",urlPatterns = "/getCookie")
public class GetCookie extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //刚才颁发给了浏览器一个cookie
        response.setContentType("text/html;charset=UTF-8");
        //这里的目的是获得cookie 看看是不是得到了 其实在浏览器里也可以看 确实是得到了
        Cookie[] cookies = request.getCookies();
        System.out.println(cookies.length);
        for(Cookie cookie:cookies){
            String name = cookie.getName();
            //这里因为在颁布cookie的时候 对value字段进行了编码 所以现在想获得cookie就得进行解码
            String value = URLDecoder.decode(cookie.getValue(),"UTF-8");
            response.getWriter().write(name+value);
        }
    }
}

```

Cookie的应用 ：

1. 显示用户上次登陆时间：

   ```java
   package Servlet;
   
   import javax.servlet.ServletException;
   import javax.servlet.annotation.WebServlet;
   import javax.servlet.http.Cookie;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.io.IOException;
   import java.io.PrintWriter;
   import java.text.SimpleDateFormat;
   import java.util.Date;
   
   /**
    * @author yhs
    * @date 2020/4/29 16:40
    * @description
    */
   @WebServlet(name = "CookiesApplication",urlPatterns = "/cookieApp")
   public class CookiesApplication extends HttpServlet {
       @Override
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
   
       }
   
       @Override
       protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
           String cookieValue = null;
           Cookie[] cookies = request.getCookies();
           response.setContentType("text/html;charset=UTF-8");
           SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd-HH:mm:ss.SSS");
           PrintWriter writer = response.getWriter();
           for(Cookie cookie:cookies){
               if(cookie.getName().equals("time2")){
                   cookieValue = cookie.getValue();
                   writer.write("上次登录时间"+cookieValue);
                   cookie.setValue(simpleDateFormat.format(new Date()));
                   response.addCookie(cookie);
                   break;
               }
           }
           if(cookieValue==null){
               Cookie cookie = new Cookie("time2",simpleDateFormat.format(new Date()));
               cookie.setMaxAge(100000);
               response.addCookie(cookie);
               writer.write("欢迎第一次登陆");
           }
       }
   
   }
   ```
   
   代码解释：
   
   ```txt
   先遍历当前Http请求的Cookie，如果不包含名为time2的Cookie，则认为用户是首次登录，这时创建一个time2 Cookie，value为当前时间。返给用户信息为首次登录。
   如果Cookie中包含time2 Cookie，则取出当前Cookie的value，将上次用户登录的时间返给用户，并且更新当前Cookie为当前时间。
   ```
   
2. 显示用户浏览过的书籍

   ```java
   package Servlet;
   
   import java.util.LinkedHashMap;
   
   /**
    * @author yhs
    * @date 2020/4/29 19:54
    * @description
    */
   public class BookCollector {
       private static LinkedHashMap<Integer,Book> linkedHashMap = new LinkedHashMap<>();
       static {
           linkedHashMap.put(1,new Book(1,"javaWeb"));
           linkedHashMap.put(2,new Book(2,"coreJava"));
           linkedHashMap.put(3,new Book(3,"mysql"));
           linkedHashMap.put(4,new Book(4,"javaScript"));
           linkedHashMap.put(5,new Book(5,"hadoop"));
       }
   
       public static LinkedHashMap<Integer,Book>getAllBook(){
           return BookCollector.linkedHashMap;
       }
       public static String getById(String idString){
           int id = Integer.parseInt(idString);
           return linkedHashMap.get(id).getName();
       }
   
       public static Book getBookById(String idString){
           int id = Integer.parseInt(idString);
           return linkedHashMap.get(id);
       }
   }
   
   class Book{
       private int id;
       private String name;
   
       public Book() {
       }
   
       public Book(int id, String name) {
           this.id = id;
           this.name = name;
       }
   
   
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   }
   
   ```

   ```java
   package Servlet;
   
   import javax.servlet.ServletException;
   import javax.servlet.annotation.WebServlet;
   import javax.servlet.http.Cookie;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.io.IOException;
   import java.io.PrintWriter;
   import java.util.Map;
   import java.util.Set;
   
   /**
    * @author yhs
    * @date 2020/4/29 19:53
    * @description
    */
   @WebServlet(name = "CookieApplication2",urlPatterns = "/book")
   public class CookieApplication2 extends HttpServlet {
       @Override
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       }
       @Override
       protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
           response.setContentType("text/html;charset=UTF-8");
           //显示所有书籍
           PrintWriter writer = response.getWriter();
           writer.write("网上所有书籍"+"<br/>");
           Set<Map.Entry<Integer,Book>> entries = BookCollector.getAllBook().entrySet();
   
           for(Map.Entry<Integer,Book> bookEntry:entries){
               Book book = bookEntry.getValue();
               String url = "/JavaWebLearn/bookServlet?id="+book.getId();
               writer.write(book.getName());
               writer.write("<a href='"+url+"'>购买</a>");
               writer.write("<br/>");
           }
           writer.write("您曾经浏览过："+"<br/>");
           Cookie[] cookies = request.getCookies();
           String bookHistory = null;
           for(Cookie cookie:cookies){
               if(cookie.getName().equals("bookHistory")){
                   bookHistory = cookie.getValue();
               }
           }
           String[] strings = null;
           if(bookHistory!=null){
               strings = bookHistory.split("_");
           }
           for(String string:strings){
               writer.write(BookCollector.getById(string)+"<br/>");
           }
       }
   }
   
   ```

   ```java
   package Servlet;
   
   import javax.servlet.ServletException;
   import javax.servlet.annotation.WebServlet;
   import javax.servlet.http.Cookie;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.io.IOException;
   import java.util.Arrays;
   import java.util.LinkedList;
   import java.util.List;
   
   /**
    * @author yhs
    * @date 2020/4/29 20:20
    * @description
    */
   @WebServlet(name = "BookServlet",urlPatterns = "/bookServlet")
   public class BookServlet extends HttpServlet {
       @Override
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       }
   
       @Override
       protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
           response.setContentType("text/html;charset=UTF-8");
           String id = request.getParameter("id");
           String bookHistory = null;
           Cookie[] cookies = request.getCookies();
           for(Cookie cookie:cookies){
               if(cookie.getName().equals("bookHistory")){
                   bookHistory = cookie.getValue();
               }
           }
           if(bookHistory!=null){
               String[] strings = bookHistory.split("_");
               List<String> list = Arrays.asList(strings);
               LinkedList<String> linkedList = new LinkedList<>(list);
               if(linkedList.contains(id)){
                   linkedList.remove(id);
                   linkedList.addFirst(id);
               }else {
                   if(linkedList.size()>=3){
                       linkedList.removeLast();
                       linkedList.addFirst(id);
                   }else {
                       linkedList.addFirst(id);
                   }
               }
               StringBuffer stringBuffer = new StringBuffer();
               for(String bookid:linkedList){
                   stringBuffer.append(bookid+"_");
               }
               bookHistory = stringBuffer.deleteCharAt(stringBuffer.length()-1).toString();
           }else {
               bookHistory = id;
           }
           Cookie cookie = new Cookie("bookHistory",bookHistory);
           cookie.setMaxAge(1000);
           response.addCookie(cookie);
       }
   }
   
   ```

   代码解释：

   ```txt
   代码块1是一个Book类加一个初始化的Book集合，用于制造原始数据。
   代码块2是核心类，他调用代码块1的初始化数据展示给用户 然后通过查询bookHistory这个Cookie，将用户浏览过的数据返回给页面显示
   代码块3是业务处理类，主要处理Cookie，当用户点击展示的Book时，就会触发一个指向代码块3的Http请求，代码块3通过HTTP参数获得当前用户请求的书籍，判断当前bookHistory Cookie的情况，如果为空，就造一个bookHistory Cookie并将当前HTTP参数的Book id作为value。如果当前bookHistory不为空，判断当前id是否已经存在，如果存在就移动当前id，并置于第一位，如果不存在，判断当前Cookie里记录的用户浏览数是否小于3，小于3就直接将当前请求id置于最前，否则就删除最后一本书，再将当前id置于最前。转为字符串("_"分割)更新Cookie
   ```

### Session

与Cookie类似，Session也是用于记录用户身份，标记浏览器状态的机制。但不同的是，Cookie是给用户颁发通行证，本地浏览器存储Cookie，而Session保存在服务器中。

>Session 是另一种记录浏览器状态的机制。不同的是Cookie保存在浏览器中，Session保存在服务器中。用户使用浏览器访问服务器的时候，服务器把用户的信息以某种的形式记录在服务器，这就是Session

简单来说，Session相当于在服务器中建立了一份“客户明细表”。然后通过检查服务器上的“客户明细表”来确认用户的身份的。

但与Cookie不同，Session除了可以保存字符串外，还可以存储对象。

**Session API**

- long getCreationTime();【获取Session被创建时间】
- **String getId();【获取Session的id】**
- long getLastAccessedTime();【返回Session最后活跃的时间】
- ServletContext getServletContext();【获取ServletContext对象】
- **void setMaxInactiveInterval(int var1);【设置Session超时时间】**
- **int getMaxInactiveInterval();【获取Session超时时间】**
- **Object getAttribute(String var1);【获取Session属性**】
- Enumeration getAttributeNames();【获取Session所有的属性名】
- **void setAttribute(String var1, Object var2);【设置Session属性】**
- **void removeAttribute(String var1);【移除Session属性】**
- **void invalidate();【销毁该Session】**
- boolean isNew();【该Session是否为新的】

**Session的生命周期**

* Session在用户**第一次访问服务器Servlet，jsp等动态资源就会被自动创建，Session对象保存在内存里**，这也就为什么上面的例子可以**直接使用request对象获取得到Session对象**。如果访问HTML,IMAGE等静态资源Session不会被创建。

* Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，无论**是否对Session进行读写，服务器都会认为Session活跃了一次**

* 由于会有越来越多的用户访问服务器，因此Session也会越来越多。**为了防止内存溢出，服务器会把长时间没有活跃的Session从内存中删除，这个时间也就是Session的超时时间**

  Session的超时时间默认是30分钟，有三种方式可以对Session的超时时间进行修改

  >* 在tomcat/conf/web.xml文件中设置 对所有的WEB应用都有效 单位为分钟
  >
  >  ```xml
  >   <session-config>
  >       <session-timeout>60</session-timeout>
  >   </session-config>
  >  ```
  >
  >* 在单个的web.xml文件中设置 对单个web应用有效 如果有冲突，以自己的web应用为准
  >
  >  ```xml
  >   <session-config>
  >       <session-timeout>60</session-timeout>
  >   </session-config>
  >  ```
  >
  >* 通过setMaxInactiveInterval()方法设置
  >
  >  ```java
  >  httpSession.setMaxInactiveInterval(60);  //单位是秒
  >  ```

  **与Cookie不同，session周期指的是不活动时间 当在超时时间内访问session session会重新计时，如果重启tomcat 或reload web应用 或者关机等 session也会消失，而Cookie的生命周期是按累计时间来算的 不管用户是否访问了Session**

**Session的原理**

思考一个问题：HTTP是无状态请求，Session为每个用户建立一个用户明细表，当用户再次请求，服务器可以取出这个请求用户的Session，但**HTTP是无状态的，他怎么知道每次请求对应的是哪个用户呢？**

> **其实Session并没有摆脱Cookie，Session之所以能识别用户，靠得就是Cookie，这个Cookie叫做JESSIONID 它的值是Session的id值**
>
> 这个Cookie是由服务器自动颁发给浏览器的，不用手动创建，这个Cookie的MaxAge是-1，也即关闭浏览器后就失效。

**Session的应用**

```java
@WebServlet(name = "booklist",urlPatterns = "/booklist")
public class Booklist extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        //显示所有书籍
        PrintWriter writer = response.getWriter();
        writer.write("网上所有书籍"+"<br/>");
        Set<Map.Entry<Integer,Book>> entries = BookCollector.getAllBook().entrySet();

        for(Map.Entry<Integer,Book> bookEntry:entries){
            Book book = bookEntry.getValue();
            String url = "/JavaWebLearn/buyBook?id="+book.getId();
            writer.write(book.getName());
            writer.write("<a href='"+url+"'>购买</a>");
            writer.write("<br/>");
        }
    }
}
```

```java
@WebServlet(name = "buyBook",urlPatterns = "/buyBook")
public class BuyBook extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String id = request.getParameter("id");
        Book book = BookCollector.getBookById(id);

        //获得Session对象
        HttpSession session = request.getSession();
        List<Book> bookList = (List<Book>) session.getAttribute("list");

        //判断是否为空 若为空 则新建一个list并保存在session中
        if(bookList==null){
            bookList = new ArrayList<>();
            session.setAttribute("list",bookList);
        }
        bookList.add(book);
        //做跳转
        //encodeURL(String url)
        //encodeRedirectURL(String url)
        //

        String url = "/JavaWebLearn/showBuyBook";
        response.sendRedirect(response.encodeURL(url));
    }
}
```

```java
@WebServlet(name = "showBuyBook",urlPatterns = "/showBuyBook")
public class ShowBuyBook extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter writer = response.getWriter();
        HttpSession session = request.getSession();
        List<Book> bookList = (List<Book>) session.getAttribute("list");
        if(bookList==null||bookList.size()==0){
            writer.write("您暂时未购买任何书籍");
        }else {
            writer.write("您购买过以下商品:"+"<br/>");
            for(Book book:bookList){
                writer.write(book.getName()+"<br/>");
            }
        }
    }
}

```

代码解释：

```txt
与Cookie的应用相同，用户买书，记录用户买过的书。
代码块1展示了当前的书籍，并对每个书籍加了一个URL，对应一个HTTP请求。
点击购买的HTTP请求会到代码块2，这时代码块2记录用户购买的书籍，放入Session中
代码块3取出Session中的记录，展示给用户当前购买的(Sesion中记录的)书籍。
```

**Session与Cookie的区别**

**从存储方式上比较**

- Cookie只能存储字符串，如果要存储非ASCII字符串还要对其编码。
- Session可以存储任何类型的数据，可以把Session看成是一个容器

**从隐私安全上比较**

- **Cookie存储在浏览器中，对客户端是可见的**。信息容易泄露出去。如果使用Cookie，最好将Cookie加密
- **Session存储在服务器上，对客户端是透明的**。不存在敏感信息泄露问题。

**从有效期上比较**

- Cookie保存在硬盘中，只需要设置maxAge属性为比较大的正整数，即使关闭浏览器，Cookie还是存在的
- **Session的保存在服务器中，设置maxInactiveInterval属性值来确定Session的有效期。并且Session依赖于名为JSESSIONID的Cookie，该Cookie默认的maxAge属性为-1。如果关闭了浏览器，该Session虽然没有从服务器中消亡，但也就失效了。**

**从对服务器的负担比较**

- Session是保存在服务器的，每个用户都会产生一个Session，如果是并发访问的用户非常多，是不能使用Session的，Session会消耗大量的内存。
- Cookie是保存在客户端的。不占用服务器的资源。像baidu、Sina这样的大型网站，一般都是使用Cookie来进行会话跟踪。

**从浏览器的支持上比较**

- 如果浏览器禁用了Cookie，那么Cookie是无用的了！
- 如果浏览器禁用了Cookie，Session可以通过URL地址重写来进行会话跟踪。

**从跨域名上比较**

- Cookie可以设置domain属性来实现跨域名
- Session只在当前的域名内有效，不可夸域名