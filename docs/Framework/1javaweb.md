# JDBC

> JDBC API 详解 

- DriverManafer(驱动管理类)作用

  - 注册驱动

  ~~~mysql
  class.forName("com.mysql.cj.jdbc.Driver");
   -- mysql 5之后的驱动包，可以省略注册驱动步骤
  ~~~

  ~~~java
  //查看Driver的代码块
      static {
          try {
              DriverManager.registerDriver(new Driver());
          } catch (SQLException var1) {
              throw new RuntimeException("Can't register driver!");
          }
      }
  ~~~

  - 获取数据库连接

  ~~~java
  // URL username  password
  // 语法：jdbc：mysql：//ip地址：端口号/数据库名称?参数键值对 
  String url=jdbc:mysql://127.0.0.1:3306/ok_db?useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai;
  String username = 'root';
  String password = 'root';
  Connection coon = DriverManager.getConnection(url,username,password);
  ~~~

- Connection

  - 获取执行sql的对象

    1、普通执行对象  statement createStatement()

    2、预编译SQL的执行SQL的对象，防止SQL注入  PreparedStatement   preparedStatement (sql)

    3、执行存储过程的对象   CallableStatement prepareCall(sql)

  - 管理事务

  ~~~java
  //开启事务：setAutoCommit(boolean autoCommit); true 为自动提交，false为手动提交   默认是自动提交
  //提交事务：commit();
  //回滚事务：rollback();
  ~~~

- Statement

  - 执行SQL语句

  ~~~java
  Statement stmt = conn.createStatement();
  ~~~

- ResultSet(结果集对象)

- PreparedStatement   预防sql注入



# 数据库连接池

> 

- 接口标准：DataSource

- 常见数据库连接池

  - DBCP

  - C3P0

  - Druid

Druid连接池的使用步骤

- 引入Druid和mysql的依赖

~~~xml
  <dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid</artifactId>
     <version>1.1.8</version>
  </dependency>

   <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.19</version>
   </dependency>    
~~~

~~~properties
url=jdbc:mysql://127.0.0.1/db_student?serverTimezone=UTC
#这个可以缺省的，会根据url自动识别
driverClassName=com.mysql.cj.jdbc.Driver
username=root
password=abcd

##初始连接数，默认0
initialSize=10
#最大连接数，默认8
maxActive=30
#最小闲置数
minIdle=10
#获取连接的最大等待时间，单位毫秒
maxWait=2000
#缓存PreparedStatement，默认false
poolPreparedStatements=true
#缓存PreparedStatement的最大数量，默认-1（不缓存）。大于0时会自动开启缓存PreparedStatement，所以可以省略上一句设置
maxOpenPreparedStatements=20
~~~







# Servlet

> java提供的一门动态web资源开发技术。Servlet有web服务器创建，Servlet方法由web服务器调用。自定义的Service方法要实现Servlet接口

- 快速入门

创建web项目，导入Servlet依赖坐标

~~~xml
<dependency>
   <groupId>javax.servlet</groupId>
   <artifactId>javax.servlet-api</artifactId>
   <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
~~~

定义一个类，实现Servlet接口，重写接口中的所有方法

~~~java
@WebServlet("/demo")
public class servletDemo implements Servlet {
    private ServletConfig servletConfig;
    public void service() {
    }
    /**
    *初始化方法
    *调用时机：默认情况下，Servlet第一次被访问时
    *调用次数：1次
    *可以在配置@WebServlet(urlPatterns="demo",`loadOnStartup=1`）  
    *说明：负整数，第一次被访问时创建对象，0或正整数，服务器启动的时候创建对象，数字越小优先级越高
    */
    
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        this.servletConfig = servletConfig;
    }
    
    @Override
    public ServletConfig getServletConfig() {
        return servletConfig;
    }
    /**
    *提供服务
    *调用时机：每一次Servlet被访问时
    *调用次数：多次
    */

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    }

    @Override
    public String getServletInfo() {
        return null;
    }
    /**
    *销毁方法
    *调用时机：内存释放或者服务器关闭的时候
    *调用次数：1次
    */
    @Override
    public void destroy() {
    }
}
~~~

配置：在上面的类上使用@WebServlet注解，配置该Servlet的访问路径

访问：启动tomcat，进行访问

- Servlet执行流程



- Servlet生命周期

其生命周期由Servlet容器管理，分为4个阶段

`1、加载和实例化`：默认情况下，当servlet第一次被访问，由容器创建Servlet对象。[可以在配置@WebServlet(urlPatterns="demo",`loadOnStartup=1`）  说明：负整数，第一次被访问时创建对象，0或正整数，服务器启动的时候创建对象，数字越小优先级越高]

`2、初始化`：在初始化之后，容器将调用Servlet的`init()`方法初始化这个对象，完成加载配置、创建连接等初始化的工作，该方法`只调用一次`

`3、请求处理`：`每次`请求Servlet时，容器都会调用`service()`方法对请求进行处理

`4、服务终止`：当需要释放内存或关闭容器时，容器就会调用Servlet的`destroy`方法完成资源的释放。最后程序会被java的垃圾回收器回收

- Servlet体系结构

Servlet <----GenericServlet <-----HttpServlet 

大多时候继承HttpServlet，重写doGet和doPost方法就可以（也可以自定义MyServlet，在里面判断是get/post请求方式）

为什么要调用不同的请求方法？get/post的请求消息不一样，要分别处理

HttpServlet的原理？根据不同的请求方式，调用不同的doXXX方法

~~~java
@WebServlet("/demo2")
public class ServletDemo2 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }
}
~~~

- Servlet urlPattern配置（访问路径）

一个Servlet，可以配置多个访问路径

配置规则：精确匹配、目录匹配  @WebServlet(`urlPatterns = "/demo2/*"`) 、扩展名匹配  @WebServlet(`urlPatterns = "*.do"`)  、任意匹配  @WebServlet(“/”,”*”)

- xml配置方式编写 Servlet

~~~xml
<!-- 在web.xml  -->
<servlet>
	<serlet-name>demo</serlet-name>
	<servlet-class>com.hatu.web.ServletDemo</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>demo</servlet-name>
    <url-pattern>/demo1</url-pattern>
</servlet-mapping>
~~~



# Request 与 Response



**Request**

- 获取请求数据

请求数据分为3部分：请求行

~~~java
String getMethod(); // 获取请求方式
String getContextPath(); // 获取虚拟目录（项目访问路径）
StringBuffer getRequestURL(); // 获取URL（统一资源定位符）
String getRequestURI(); // 获取RUI（统一资源标识符）
String getQueryString(); // 获取get请求方式的请求参数
~~~

 请求头

~~~java
String getHeader(String name); // 根据请求头名称，获取对应的值
~~~

  请求体

~~~java
ServletInputStream getInputStream(); // 获取字节输入流
Bufferedreader getReader(); //获取字符输入流
~~~

- Request通用方式获取请求参数

~~~java
Map<String, String[ ]> getParameterMap();  // 获取所有参数的map集合
String[] getparameterValues(String name); //根据名称获取参数值（数组）
String getParameter(String name); //根据名称获取参数值（单个值） 
~~~

- 解决中文乱码

post方式：在重写的doXXX方法中`request.setCharacterEncoding(utf-8);`

get方式：



- Request的请求转发

特点：浏览器的路径不发生变化、只能转发到服务器的内部资源、一次请求 可以在转发的资源间使用request共享数据

实现方式

~~~java
request.getRequestDispatcher("/req6").forward(request,response);  //把请求转发到req6
~~~

请求转发资源间共享数据：使用request对象

~~~java
void setAttribute(String name, Obiect o);  //存储数据到request中
Object getAttribute(String name); //根据key 获取值
void removeAttribute(String name); //根据key 删除对应的键值对
~~~



**Response**

- 设置响应数据

响应行

~~~java
void setStatus(int sc); //设置响应状态码
~~~

响应头

~~~java
void setHeader(String name,String value); //设置响应头的键值对
~~~

响应体

~~~java
PrintWriter getWriter(); //获取字符输出流
ServletOtputStream getOutputStream(); //获取字节输出流
~~~

- 完成重定向

特点：浏览器路径发生变化、可以重定向到任意位置的资源（服务器内部、外部均可）、两次请求不能共享数据

实现方式

~~~java
resp.setStatus(302); //状态码固定的302
resp.setHeader("location","资源B的路径"); //location是固定的

//简化写法
resp.sendRedirect("资源B的路径");
~~~

- 响应字符数据

~~~java
//设置响应格式
response.setContentType("text/html;charset=utf-8");
//获取字符输出流
PrintWriter writer = response.getWriter();
writer.write("aaa");
~~~

- 响应字节数据

~~~java
//读取文件
FileInputStream fis = new FileInputStream("d://a.jpg");
//获取字节输出流
ServletOutputStream os = response.getOutputStream();
//完成copy
bytr[] buff = new byte[1024];
int len = 0;
while((len = fis.read(buff)) != -1){
    os.write(buff,0,len);
}
fis.close();
~~~

~~~java
//对于上述的copy可以使用工具类进行简化,引入commos-io依赖
  IOUtils.copy(fis,os);
~~~

~~~xml
  <dependency>
     <groupId>commos-io</groupId>
     <artifactId>commos-io</artifactId>
     <version>2.6</version>
  </dependency>
~~~





# Cookie

> 客户端会话技术，将数据保存到客户端，以后每次请求都携带Cookie数据进行访问。Cookie的实现基于HTTP协议。

- Cookie的基本使用

发送cookie

~~~java
Cookie cookie = new Cookie("key","value"); // 创建cookie对象，设置数据
response.addCookie(cookie); //发送cookie到客户端
~~~

获取cookie

~~~java
Cookie[] cookies = request.getCookies(); //获取客户端携带的cookies
//遍历数组，找到自己想要的cookie
for(Cookie cookie:cookies){
    String name = cookie.getName();
    if("username".equals(name)){
        cookie.getValue(); //获取username对应的cookie值
    }
}
~~~

Cookie的存活时间

1、默认情况下，浏览器关闭，内存释放，cookie被销毁

2、setMaxAge(int seconds) 设置cookie的存活时间       正数：持久化存储，到时间删除；负数，浏览器关闭，cookie销毁；零：删除对应的cookie

Cookie存储中文     

cookie不能直接存储中文，需先进行URL编码，再存储





# Seeion

> 服务端会话跟踪技术，将数据保存在服务端，javaEE提供HttpSession接口，实现一次会话的多次请求间数据共享功能。Session的实现是基于Cookie的。

~~~java
HttpSession session = request.getSession(); //获取session对象
// Session对象的功能
void setAttribute(String name ,Object o); //存储数据到session中
Object getAttribute(String name); //根据key 获取值
void removeAttribute(String name); //根据key，删除该键值对
~~~

Session的钝化、活化  （保存、读取）

Session的销毁  默认30分钟可以进行配置

~~~xml
<sesion-config>
	<session-timeout>100</session-timeout>
</sesion-config>
~~~

可以调用session对象的`invalidate()`方法主动销毁





# Filter

> 过滤器，是javaweb的三大组件（Servlet、Filter、listener）之一。一般完成一些通用的功能，如权限控制，统一编码处理，敏感字符处理

- 快速入门

定义类，实现Filter接口，重写所有方法

配置Filter拦截资源的路径，在类上定义@WebFilter注解  `@WebFilter("/*")`

在doFilter方法中配置放行

~~~java
import javax.servlet.annotation.WebFilter;
@WebFilter("/*")
public class FilterDemo implements Filter {
 
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("okokok~~")
        //放行
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
~~~

`放行前逻辑 ----》放行-----》访问资源-----》放行后逻辑`



**使用细节**

- Filter拦截路径配置

  ~~~java
  @WebFilter("/*")
  public class FilterDemo
     /index.jsp //拦截具体的资源
     /user/*    //目录拦截
  	*.jsp     //后缀名拦截
  	/*        //拦截所有
  ~~~

- 过滤器链



# Listener

> 监听器，在application，session，request三个对象创建、销毁或者其中添加修改删除属性时自动执行代码的功能组件

有8个监听器，介绍一个`ServletContextListener`  定义一个类，实现ServletContextListener接口

~~~java
@WebListener
public class ContextDemo implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContextListener.super.contextInitialized(sce);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        ServletContextListener.super.contextDestroyed(sce);
    }
}
~~~





# AJAX

> 异步的javascript和xml。
>
> - 与服务器进行数据交换：通过AJAX可以给服务器发送请求，并获取服务器响应的数据
> - 异步交互：在不重新加载整个页面的情况下，更新部分网页技术

快速入门

1、编写AjaxServlet,使用response输出字符串

2、前端代码

~~~html
//1、创建核心对象
var xmlhttp;
if (window.XMLHttpRequest)
{ //  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
    xmlhttp=new XMLHttpRequest();
} else{
    // IE6, IE5 浏览器执行代码
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
//发送请求
xmlhttp.open("GET","ajax_info.txt",true); //method请求类型，url服务器（文件）位置，async   true（异步）false（同步）
xmlhttp.send();
//获取响应
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
}
~~~



# Axios

引入axios文件

~~~js
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
~~~

使用axios发送请求，并获取响应结果

~~~html
//get实例
new Vue({
  el: '#app',
  data () {
    return {
      info: null
    }
  },
  mounted () {
    axios
      .get('https://www.runoob.com/try/ajax/json_demo.json')
      .then(response => (this.info = response))
      .catch(function (error) { // 请求失败处理
        console.log(error);
      });
  }
})


//post实例
new Vue({
  el: '#app',
  data () {
    return {
      info: null
    }
  },
  mounted () {
    axios
      .post('https://www.runoob.com/try/ajax/demo_axios_post.php')
      .then(response => (this.info = response))
      .catch(function (error) { // 请求失败处理
        console.log(error);
      });
  }
})
~~~



# Json

> 一种数据格式，多用于数据载体，在网络请求中进行数据传输，用键值对形式的书写
>
> 数据类型：数字（整数或浮点）、字符串（在双引号中）、逻辑值（true/false）、数组（在方括号中）、对象（在花括号中）、null

~~~json
{
  "homeTown": "Metro City",
  "formed": 2016,
  "active": true,
  "members": [
    {
      "name": "Molecule Man",
      "age": 29,
      "secretIdentity": "Dan Jukes",
      "powers": ["Radiation resistance", "Turning tiny", "Radiation blast"]
    },
    {
      "name": "Madame Uppercut",
      "age": 39,
      "secretIdentity": "Jane Wilson",
      "powers": [
        "Million tonne punch",
        "Damage resistance",
        "Superhuman reflexes"
      ]
    },
    {
      "name": "Eternal Flame",
      "age": 1000000,
      "secretIdentity": "Unknown",
      "powers": [
        "Immortality",
        "Heat Immunity"
      ]
    }
  ]
}
~~~

- json数据与java对象的转换  `Fastjson`

导入坐标

~~~xml
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
  </dependency>
~~~

相互转换

~~~java
String jsonStr = JSON.toJSONString(obj);  //java转json
User user = JSON.parseObject(jsonStr,User.class); //json转java
~~~

