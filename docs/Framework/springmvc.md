## SpringMVC

> SpringMVC技术与Servlet技术功能相同，均属于web层开发技术，但SpringMVC技术更简洁，能用更少的代码进行开发

#### SpringMVC概述

- SpringMVC是一种基于Java实现MVC模型的轻量级Web框架
- 功能
  - 用于进行表现层功能开发
- 优点
  - 使用简单，开发便捷（相比于Servlet）
  - 灵活性强

#### SpringMVC入门案例

1. 使用SpringMVC技术需要县导入SpringMVC坐标与Servlet坐标

   ```xml
    <!--1.导入坐标springmvc和servlet的-->
       <dependency>
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>3.1.0</version>
         <scope>provided</scope>
       </dependency>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>5.2.10.RELEASE</version>
       </dependency>
   ```

2. 创建SpringMVC控制器类（等同于Servlet功能）

   ```java
   @Controller
   public class UserController{
    @RequestMapping("/save")
    @ResponseBody
    public String save(){
    	System.out.println("user save ...");
    	return "{'module':'springmvc'}";
    }
   }
   ```

3. 初始化SpringMVC环境（同Spring环境），设定SpringMVC加载对应的bean

   ```java
   @Configuration
   @ComponentScan("com.iteima.controller")
   public class SpringMvcConfig{
   }
   ```

4. 初始化Servlet容器，加载SpringMVC环境，并设置SpringMVC技术处理请求

   ```java
   import org.springframework.web.context.WebApplicationContext;
   import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
   import org.springframework.web.servlet.support.AbstractDispatcherServletInitializer;
   
   //4.定义一个servlet容器启动的配置类，在里面加载spring的配置
   public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
       //加载springMVC容器配置
       @Override
       protected WebApplicationContext createServletApplicationContext() {
           AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
           context.register(SpringMvcConfig.class);
           return context;
       }
       //设置哪些请求归属springMVC处理
       @Override
       protected String[] getServletMappings() {
           return new String[]{"/"};
       }
    //加载spring容器配置
       @Override
       protected WebApplicationContext createRootApplicationContext() {
           return null;
       }
   }
   ```

   #### 入门案例用到的注解

   - 名称：@Controller

   - 类型：类注解

   - 位置：SpringMVC控制器定义上方

   - 作用：设置SpringMVC的核心控制器bean

   - 范例：

     ```java
     @Controller
     public class UserController{
     }
     ```

   - 名称：@ResquestMapping

   - 类型：方法注释

   - 位置：SpringMVC控制器方法定义上方

   - 作用：设置当前控制器方法请求访问路径

   - 范例：

     ```java
     @ResquestMapping("/save")
     public void save(){
        System.out.println("user save ...");
     }
     ```

   - 相关属性

     - value(默认)：请求访问路径

   - 名称：@ResponseBody

   - 类型：方法注释

   - 位置：SpringMVC控制器方法定义上方

   - 作用：设置当前控制器方法响应内容为当前返回值，无需解析

   - 范例：

     ```java
     @RequestMapping("/save")
     @ResponseBody
     public String save(){
     System.out.println("user save ...");
     return "{'info':"springmvc"}";
     }
     ```

   - SpringMVC入门程序开发总结（1+N）

     - 一次性工作
       - 创建工程，设置服务器，加载工程
       - 导入坐标
       - 创建web容器启动类，加载SpringMVC配置，并设置SpringMVC请求拦截路径
       - SpringMVC核心配置类（设置配置类，扫描controller包，加载Controller控制器bean）
     - 多次工作
       - 定义处理请求的控制器类
       - 定义处理请求的控制器方法，并配置映射路径（@RequestMapping）与返回json数据（@ResponseBody）

#### Servlet容器配置类

- AbstractDispatcherServletInitializer类是SpringMVC提供的快速初始化web3.0容器的抽象类

- AbstractDispatcherServletInitializer提供三个接口方法供用户实现

  - `createServletAppilcationContext（）`方法，创建servlet容器时，加载SpringMVC对应的bean并放入WebApplicationContext对象中，而WebApplicationContext的作用范围为ServletContext范围，即整个web容器范围

    ```java
    protected WebApplicationContext createServletApplicationContext(){
    AnnotattionConfigWebApplicationContext context = new AnnotattionConfigWebAppklicationtContext();
    context.register(SpringMvcConfig.class);
    return context;
    }
    ```

  - `getServletMappings()`方法，设定SpringMVC对应的请求映射路径，设置为/表示拦截所有请求，任意请求都将转入道SpringMVC进行处理

    ```java
    protected String[] getServletMapping(){
    	return new String[]("/");
    }
    ```

  - `createRootApplicationContext()`方法，如果创建Servlet容器时需要加载飞SpringMVC对应的bean，使用当前方法进行，使用方式用createServletAppilcationContext（）

    ```java
    protected WebApplicationContext createRootApplicationContext(){
    return null;
    }
    ```

    ### 入门案例工作流程分析

    - 启动服务器初始化过程
      1. 服务器启动，执行ServletContainersInitConfig类，初始化web容器
      2. 执行createServletApplicationContext方法，创建了WebApplicationContext对象
      3. 加载SpringMvcConfig
      4. 执行@ComponentScan加载对应的bean
      5. 加载UserController，每过@RequestMapping的名称对应一个具体的方法
      6. 执行getServletMappings方法，定义所有请求都通过SpringMVC
    - 单次请求过程
      1. 发射请求localhost/save
      2. web容器发现所有请求都经过SpringMVC，将请求交割SpringMVC处理
      3. 解析请求路径/save
      4. 由/save匹配执行对应的方法saev()
      5. 执行save()
      6. 检测道由@ResponseBody之际将save()方法的返回值作为响应体返回给请求方

#### Controller加载控制与业务bean加载控制

- SpringMVC相关bean（表现层bean）
- Spring控制的bean
  - 业务bean（Service）
  - 功能bean（DatatSource等）
- SpringMVC相关bean加载控制
  - SpringMVC加载的bean对应的包均在com.hcx.controller包内
- Spring相关bean加载控制
  - 方式一：Spring加载的bean设定扫描范围为com.hcx,排除controller包内的bean
  - 方式二：Spring加在的bean设定扫描范围为精准范围，例如service包，dao包等

#### 注解复习

- 名称：@ComponentScan

- 类型：类注解

- 范例：

  ```java
  @Configuration
  @ConmponentScan(value="com.itheima",
    excludeFilters = @ComponentScna.Filter(
    	type = FilterType.ANNOTATION,
    	classer = Controller.class
    	)
  )
  public class SpringConfig{}
  ```

- 属性

  - excludeFilters: 排除扫描路径中加载的bean，需要指定类别（type）与具体项（classer）
  - includeFilters: 加载指定的bean，需要指定类别（type）与具体项（class）

- bean的加载格式

  ```java
  //4.定义一个servlet容器启动的配置类，在里面加载spring的配置
  public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
      //加载springMVC容器配置
      @Override
      protected WebApplicationContext createServletApplicationContext() {
          AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
          context.register(SpringMvcConfig.class);
          return context;
      }
      //设置哪些请求归属springMVC处理
      @Override
      protected String[] getServletMappings() {
          return new String[]{"/"};
      }
  //加载spring容器配置
      @Override
      protected WebApplicationContext createRootApplicationContext() {
           AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
          context.register(SpringConfig.class);
          return context;
      }
  }
  ```

- 简化开发

  ```java
  public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer{
  
      @Override
      protected Class<?>[] getRootConfigClasses() {
          return new Class[]{SpringMvcConfig.class};
      }
  
      @Override
      protected Class<?>[] getServletConfigClasses() {
          return new Class[0];
          /* return new Class[]{SpringConfig.class}*/
      }
  
      @Override
      protected String[] getServletMappings() {
          return new String[]{"/"};
      }
  }
  ```

#### 请求与响应

- 名称：@RequestMapping

- 类型：方法注解 类注解

- 位置：SpringMVC控制器方法定义上方

- 作用：设置当前控制器方法请求访问路径，如果设置在类上统一设置当前方法请求访问路径前缀

- 范例：

  ```java
  @Controller
  @RequestMapping("/user")
  public class UserController{
    @RequestMapping("/save")
    @ResponseBody
    public String save(){
    	System.out.println("user save ...");
    	return "{'module':'user save'}";
    }
  }
  ```

- 属性

  - value(默认)：请求访问路径，或访问路径前缀

##### 中文乱码处理

###### **Post请求中文乱码处理**

- 为web容器添加过滤器并指定字符集，Spring-web包中提供了专用的字符过滤器

  ```java
  //乱码处理
      @Override
      protected Filter[] getServletFilters() {
          CharacterEncodingFilter filter = new CharacterEncodingFilter();
          filter.setEncoding("UTF-8");
          return new Filter[]{filter};
      }
  ```

##### 请求参数

- 参数种类

  - 普通参数
  - POJO类型参数
  - 嵌套POJO类型参数
  - 数组类型参数
  - 集合类型参数

- 普通参数：url 地址穿参，地址参数名与形参变量名相同，定义形参即可接收参数

  ```java
  @RequestMapping("/commoParam")
  @ResponseBody
  public String commonParam(String name,int age){
    Stystem.out.println("普通参数传递 name ==> " + name );
    System.out.println("普通参数 age ==> " + age);
    return "{'module':'common param'}";
  }
  ```

##### 使用到的注解

- 名称：@RequestParam

- 类型：形参注解

- 位置：SpringMVC控制器方法形参定义前面

- 作用：绑定请求参数与处理器方法形参间的关系

- 范例：

  ```java
  @RequestMapping("/commonParamDifferentName")
  @ResponseBody
  public String commonParamDifferentName(@RequestParam("name") String userName , int age){
   	System.out.println("普通参数 userName ==>" + userName);
   	System.out.println("普通参数 userName ==>" + age);
   	return "{'module':'common param different name'}";
  }
  ```

- 参数：

  - required: 是否为必传参数
  - defaultValue：参数默认值

###### POJO参数/嵌套的POJO参数

- POJO参数/嵌套的POJO参数：请求数名与形参对象属性名相同，定义POJO类型形参即可接收参数

  ```java
  //嵌套pojo参数
      @RequestMapping("/saveObj")
      @ResponseBody
      public String saveObj(User user){
          System.out.println("对象接收数据"+user);
          return "{'module':'pojo contain pojo param'}";
      }
  ```

##### Json数据传参

**请求参数**

- json数组
- json对象（POJO）
- json数组（POJO）

###### 步骤一：导入JSON的pom坐标

```xml
<!--json数据处理-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.4</version>
    </dependency>
```

###### 步骤二：设置发送json数据（请求body中添加json数据）

###### 步骤三：在SpringMvcConfig.class里面加入@EnableWebMvc，开启自动转换json数据的支持

```java
//3创建springmvc的配置文件，加载controller对应的bean
@Configuration
//@ComponentScan("com.hcx.controller")
@ComponentScan(value = "com.hcx"/*, excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)*/)
@EnableWebMvc
public class SpringMvcConfig {
}
```

###### 第四步：在Controller（控制层）编写代码，设置json数据

```java
//集合参数：JSON格式
    @RequestMapping("/listParamJson")
    @ResponseBody
    public String listParamJson(@RequestBody List<String> list){
        System.out.println("集合参数传递 list ==>" + list);
        return "{'module':'listJson param'}";
    }
    //POJO参数：JSON格式
    @RequestMapping("/pojoParamJson")
    @ResponseBody
    public String pojoParamJson(@RequestBody User user){
        System.out.println("pojo(json)参数传递 user ==>"+user);
        return "{'module':'pojoJson param'}";
    }
    //对象集合类型：JSON格式
    @RequestMapping("/listUserParamJson")
    @ResponseBody
    public String listUserParamJson(@RequestBody List<User> list){
        System.out.println("List<User>参数传递 ==>" + list);
        return "{'module':'List<User>Json param'}";
    }
```

##### 注解复习

- 名称：@EnableWebMvc

- 类型：配置类注解

- 位置：SpringMVC配置定义上方

- 作用：开启SpringMVC多项负责功能

- 范例：

  ```java
  //3.创建springmvc的配置文件，加载controller对应的bean
  @Configuration
  //@ComponentScan("com.hcx.controller")
  @ComponentScan(value = "com.hcx"/*, excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)*/)
  @EnableWebMvc
  public class SpringMvcConfig {
  }
  ```

- 名称：@RequestBody

- 类型：形参注解

- 位置：SpringMVC控制器方法形参定义前面

- 作用：将请求中请求体所包含的数据传递给请求参数，此注解一个处理器方法只能使用一次

- 范例：

  ```java
  @RequestMapping("/listParamForJson")
  @ResponseBody
  public String listParamForJson(@RequestBody List<String> likes){
   	System.out.println("list common(json)参数传递 list ==>"+ likes);
   	return "{'moudle':'list common for json param'}";
  }
  ```

###### **@RequestBody与@RequestParam区别**

- 区别
  - @RequestParam用于接收url地址传参，表单传参【application/x-www-form-urlencoded】
  - @RequestBody用于接收json数据【application/json】
- 应用
  - 后期开发中，发送json格式数据为主，@ResquestBody应用较广
  - 如果发送飞json格式数据，选用@RequestParam接收请求参数

##### 日期类型参数传递

- 日期类型数据基于系统不同格式也不尽相同

  - 2022-08-30
  - 2022/08/30
  - 08/18/2022

- 接收形参是，根据不同的日期格式设置不同的接收方式

  ```java
  @RequestMapping("/dateParam")
  @ResponseBody
  public String dateParam(Date date,
    					@DateTimeFormat(pattern = "yyyy-MM-dd") Date date1,
    					@DateTimeFormat(pattern = "yyyy/MM/dd HH:mm:ss")Date date2){
    					System.out.println("参数传递 date ==>" + date);
    					System.out.println("参数传递 date(yyyy-MM-dd) ==>" + date1);
    					System.out.println("参数传递 date(yyyy/MM/dd HH:mm:ss) ==>" + date2);
    		retrun "{'module':'data param'}";				
    	}				
  ```

##### 日期类型注解

- 名称：@DateTimeFormat

- 类型：形参注解

- 位置：SpringMVC控制器方法形参前面

- 作用：设定日期时间型数据格式

- 范例：

  ```java
  @RequestMapping("/dateParam")
  @ResponseBody
  public String dateParam(@DateTimeFormat(pattern = "yyyy-MM-dd") Date date){
    System.out.println("参数传递 date ==>" + date);
    retrun "{'module':'data param'}";	
  }
  ```

- 属性：pattern = "日期时间格式字符串"

##### 类型转化器

- Converter接口

  ```java
  public interface Converter<S,T>{
    @Nullable
    T convert(S source);
  }
  ```

  - 请求参数年龄数据（String+Integer）
  - 日期格式转换 （String → Date）

- @EnableWebMvc功能之一：根据类型匹配对应类型转换器

##### 响应

- 响应页面

- 响应数据

  - 文本数据
  - JSON数据

- 响应文本数据

  ```java
  @RequestMapping("/toText")
  @ResponseBody
  public String toText(){
    return "response text";
  }
  ```

- 响应json数据（对象转json）

  ```java
  //响应POJO对象数据
      @RequestMapping("/toJsonPOJO")
      @ResponseBody
      public User toJsonPOJO(){
          System.out.println("返回json对象数据");
          User user = new User();
          user.setName("hcx");
          user.setAge(21);
          user.setAddress(new Address("南昌","江西"));
          return user;
      }
  ```

- 响应对象集合转json数组

  ```java
  //响应POJO集合对象
      @RequestMapping("/toJsonList")
      @ResponseBody
      public List<User> toJsonList(){
          System.out.println("返回json对象集合");
          User user= new User();
          user.setName("开源协议");
          user.setAge(21);
          user.setAddress(new Address("南昌","江西"));
  
          User userOne= new User();
          userOne.setName("阿里巴巴开发手册");
          userOne.setAge(15);
          userOne.setAddress(new Address("杭州","浙江"));
  
          List<User> userList = new ArrayList<>();
          userList.add(user);
          userList.add(userOne);
          return userList;
      }
  ```

###### 注解复习

- 名称：@ResponseBody

- 类型：方法注解

- 位置：SpringMVC控制器方法定义上方

- 作用：设置当前控制器方法响应内容为当前返回值，无需解析

- 范例：

  ```java
  @RequestMapping("/save")
  @ResponseBody
  public String save(){
    System.out.print("Save...");
    return "{'info':'springmvc'}";
  }
  ```

- HttpMessageConverter接口

  ```java
  public interface HttpMessageConverter<T> {
      boolean canRead(Class<?> var1, @Nullable MediaType var2);
  
      boolean canWrite(Class<?> var1, @Nullable MediaType var2);
  
      List<MediaType> getSupportedMediaTypes();
  
      T read(Class<? extends T> var1, HttpInputMessage var2) throws IOException, HttpMessageNotReadableException;
  
      void write(T var1, @Nullable MediaType var2, HttpOutputMessage var3) throws IOException, HttpMessageNotWritableException;
  }
  ```

#### REST风格

- REST（Representational State Transfer）,表现形式状态转换

  - 传统风格资源描述形式

    http://localhost/user/getById?id=1

    http://localhost/user/saveUser

  - REST风格描述形式

    http://localhost/user/1

    http://localhost/user

- 优点：

  - 隐藏资源的访问行为，无法通过地址得治对资源是何种操作
  - 简化书写

###### REST风格简介

- 安装REST风格访问资源是使用行为动作区分对资源进行何种操作
  - http://localhost/users 查询全部用户信息 GET （查询）
  - http://loocalhost/users/1 查询指定用户信息 GET（查询）
  - http://localhost/users 添加用户信息 POST（新增/保存）
  - http://localhost/users 修改用户信息 PUT（修改/更新）
  - http://localhost/users/1 删除用户信息 DELETE（删除）

注意事项：

上述行为是约定方式，约定不是规范，可以打破，所以称REST风格，而不是REST规范

描述模块的名称通常使用复数，也就是加s的格式描述，表示此类资源，而非单个资源，例如：users，books,accounts....

###### RESTful入门案例

1. 设定http请求动作（动词）

   ```java
   @RequestMapping(value="/users",method =RequestMethod.POST)
   @ResponseBody
   public String save(@RequestBody User user){
    System.out.println("user save..." + user);
    return "{'module':'user save'}";
   }
   
   @RequestMapping(value="/users",method=RequestMethod.PUT)
   @ResponseBody
   publiic String update(@RequestBody User user){
    System.out.println("user update ..."+ user);
    return "{'module':'user update'}";
   }
   ```

2. 设置请求参数（路径变量）

   ```java
   @RequestMapping(value ="/users/{id}",method =RequestMethod.DELETE)
   @ResponseBody
   public String delete(@PathVariable Integer id){
    System.out.println("user delete ..."+ id);
    return "{'module':'user deleter'}";
   }
   ```

   ###### 注解复习

   - 名称：@PathVariable

   - 类型：形参注解

   - 位置：SpringMVC控制器方法形参定义前面

   - 作用：绑定路径参数与处理器方法形参间的关系，要求路径参数名与形参名一一对应

   - 范例：

     ```java
     @RequestMapping(value = "/users/{id}",method =RequestMethod.DELETE)
     @ResponseBody
     public String save(@PatVariable Integer id){
        System.out.println("user delete ..." + id);
        return "{'module':'user delete'}";
     }
     ```

   - 名称：@RequestMapping

   - 类型：方法注解

   - 位置：SpringMVC控制器定义上方

   - 作用：设置当前控制器方法请求访问路径

   - 范例：

     ```java
     @RequestMapping(value = "/users",method =RequestMethod.POST)
     @ResponseBody
     public String save(@RequestBody User user){
        System.out.println("user save ..." + user);
        return "{'module':'user save'}";
     }
     ```

   - 属性

     - value(默认)：请求访问路径
     - method:http请求动作，标准动作（POST/GET/PUT/DELETE）

###### **@RequestBody @RequestParam @PathVariable**

- 区别
  - @RequestParam用于接收url地址传参或表单传参
  - @RequestBody用于接收JSON数据
  - @PathVariable用于接收路径参数，使用{参数名称}描述路径参数
- 应用
  - 后期开发中，发送请求参数超过一个时，以json格式为主，@RequestBody应用较广
  - 如果发送非json格式数据，选用@RequestParam接收请求参数
  - 采用RESTful进行开发，当参数数量较少时，例如1个，可以采用@PathVariable接收路径变量，常用于传递id值

###### RESTful快速开发

```java
@RequestMapping(value="/books",method = RequestMethod.POST)
@ResponseBody
public String save(@RequestBody Book book){
	System.out.println("book save ..."+ book);
	return "{'module':'book save'}";
}
@RequestMapping(value="/books",method = RequestMethod.PUT)
@ResponseBody
public String save(@RequestBody Book book){
	System.out.println("book update ..."+ boook);
	return "{'module':'book update'}";
}
```

###### 合并注解事项简化开发

- 名称：@RestController

- 类型：类注解

- 位置：基于SpringMVC的RESTful开发控制器类定义上方

- 作用：设置当前控制器类为RESTful风格，等同于@Controller与@ResponseBody两个注解组合功能

- 范例：

  ```java
  @RestControoler
  public class BookController{
  }
  ```

- 名称：@GetMapping @PostMapping @PutMapping @DeleteMapping

- 类型：方法注解

- 位置：基于SpringMVC的RESTful开发控制器方法定义上方

- 作用：设置当前控制器方法请求访问路径与请求动作，每种对应一个请求动作，例如@GetMapping对应Get请求

- 范例：

  ```java
  @GetMapping("/{id}")
  public String getById(@PathVariable Integer id){
      System.out.pringln("book getById ..." + id);
      return "{'module':'book getById'}";
  }
  ```

- 属性

  - value(默认)：请求访问路径

###### 案例：基于RESTful页面数据交互

1. 制作SpringMVC控制器，并通过PostMan测试接口功能

   ```java
   @RestController
   @RequestMapping("/books")
   public Class BookController{
    @PostMapping
    public String save(@RequestBody Book book){
    	System.out.println("book save ==>" + book);
    	return "{'moudule':'book save success'}";
    }
    @GetMapping
       public List<Book> getAll(){
           List<Book> bookList = new ArrayList<>();
           Book book = new Book();
           book.setType("计算机");
           book.setName("SpringMVC入门教程");
           book.setDescription("小试牛刀");
           bookList.add(book);
           Book book1 = new Book();
           book1.setType("计算机");
           book1.setName("SpringMVC入门教程");
           book1.setDescription("加油");
           bookList.add(book1);
           //模拟数据
           return bookList;
       }
   }
   ```

2. 设置对静态支援的访问通行

   ```java
   @Configuration
   public class SpringMvcSupport extends WebMvcConfigurationSupport {
       @Override
       protected void addResourceHandlers(ResourceHandlerRegistry registry) {
           //当访问/pages/？？？不要走mvc，走/pages目录下的内容访问
           registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
           //放行js
           registry.addResourceHandler("/js/**").addResourceLocations("/js/");
       //放行css
           registry.addResourceHandler("/css/**").addResourceLocations("/css/");
           //放行插件
           registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
       }
   }
   ```

3. 前端页面通过异步提交访问后台控制器

   ```js
   //添加
   saveBook(){
    axios.post("/books",this.formData).then((res)=>{
   
    });
   },
   
   //主页列表查询
   getAll(){
    axios.get("/books").then((res)=>{
    	this.dataList = res.data;
    });
   },
   ```

4. 总结

   - 先做后台功能，开发接口并调通接口
   - 再做页面异步调用，确认功能可以正常访问
   - 最后再完成页面数据的显示
   - 补充：放行静态资源访问