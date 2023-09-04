## SSM整合

### 概览

- SSM整合
- 表现层数据封装
- 异常处理器
- 项目异常处理方案
- 案例：SMM整合标准开发

### SSM整合流程

1. 创建工程，并导入坐标

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>org.example</groupId>
       <artifactId>springmvc_08_ssm</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>war</packaging>
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <maven.compiler.source>1.8</maven.compiler.source>
           <maven.compiler.target>1.8</maven.compiler.target>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.18.12</version>
           </dependency>
   
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.2.10.RELEASE</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.2.10.RELEASE</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-test</artifactId>
               <version>5.2.10.RELEASE</version>
           </dependency>
           <!--    spring整合mybatis需要三个坐标：
                   mybatis
                   mybatis-spring
                   mysql-connect-java
           -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.6</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>1.3.2</version>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.47</version>
           </dependency>
           <!--    该项目中选择使用druid数据源-->
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>druid</artifactId>
               <version>1.2.11</version>
           </dependency>
           <!--    测试用，不解释-->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
           <!--    做springmvc开发时，web容器要用到的-->
           <dependency>
               <groupId>javax.servlet</groupId>
               <artifactId>servlet-api</artifactId>
               <version>2.5</version>
               <scope>provided</scope>
           </dependency>
           <!--    实现json文件与各数据类型之间的互相转换-->
           <dependency>
               <groupId>com.fasterxml.jackson.core</groupId>
               <artifactId>jackson-databind</artifactId>
               <version>2.9.0</version>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.apache.tomcat.maven</groupId>
                   <artifactId>tomcat7-maven-plugin</artifactId>
                   <version>2.1</version>
                   <configuration>
                       <port>8080</port>
                       <path>/</path>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

2. SSM整合

   - Spring

     - SpringConfig

       ```java
       @Configuration
       @ComponentScan("com.hcx")
       @PropertySource("classpath:jdbc.properties")
       @Import({JdbcConfig.class,MyBatisConfig.class})
       public class SpringConfig{
       }
       ```

   - MyBatis

     - MybatisConfig

       ```java
       import org.mybatis.spring.SqlSessionFactoryBean;
       import org.mybatis.spring.mapper.MapperScannerConfigurer;
       import org.springframework.context.annotation.Bean;
       
       import javax.sql.DataSource;
       
       public class MyBatisConfig {
           @Bean
           public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
               SqlSessionFactoryBean factoryBean =  new SqlSessionFactoryBean();
               factoryBean.setDataSource(dataSource);
               factoryBean.setTypeAliasesPackage("com.hcx.domain");
               return factoryBean;
           }
       
           @Bean
           public MapperScannerConfigurer mapperScannerConfigurer(){
               MapperScannerConfigurer mapperScannerConfigurer=new MapperScannerConfigurer();
               mapperScannerConfigurer.setBasePackage("com.hcx.dao");
               return mapperScannerConfigurer;
           }
       }
       ```

     - JdbcConfig

       ```java
       public class JdbcConfig{
       @Value("${jdbc.driver}")
       private String driver;
       @Value("${jdbc.url}")
       private String url;
       @Value("${jdbc.username}")
       private String userName;
       @Value("${jdbc.password}")
       private String password;
       @Bean
       public DataSource dataSource(){
       DruidDataSource dataSource = new DruidDataSource();
         dataSource.setDriverClassName(driver);
               dataSource.setUrl(url);
               dataSource.setUsername(username);
               dataSource.setPassword(password);
               return dataSource;
       }
       
           //开启事务管理
           @Bean
           public PlatformTransactionManager transactionManager(DataSource dataSource){
               DataSourceTransactionManager ds = new DataSourceTransactionManager();
               ds.setDataSource(dataSource);
               return ds;
           }
       }
       ```

     - jdbc.properties

       ```properties
       jdbc.driver=com.mysql.jdbc.Driver
       jdbc.url=jdbc:mysql://localhost:3306/ssm_db
       jdbc.username=root
       jdbc.password=******
       ```

   - SpringMVC

     - ServletConfig

       ```java
       package com.hcx.config;
       
       import org.springframework.web.filter.CharacterEncodingFilter;
       import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
       
       import javax.servlet.Filter;
       
       public class ServletConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
           protected Class<?>[] getRootConfigClasses() {
               return new Class[]{SpringConfig.class};
           }
       
           protected Class<?>[] getServletConfigClasses() {
               return new Class[]{SpringMvcConfig.class};
           }
       
           protected String[] getServletMappings() {
               return new String[]{"/"};
           }
       
           //乱码处理——post表单提交时处理中文乱码
           @Override
           protected Filter[] getServletFilters() {
               CharacterEncodingFilter filter = new CharacterEncodingFilter();
               filter.setEncoding("UTF-8");
               return new Filter[]{filter};
           }
       }
       ```

     - SpringMvcConfig

       ```java
       import org.springframework.context.annotation.ComponentScan;
       import org.springframework.context.annotation.Configuration;
       import org.springframework.web.servlet.config.annotation.EnableWebMvc;
       
       @Configuration
       @ComponentScan("com.hcx.controller")
       @EnableWebMvc
       public class SpringMvcConfig {
       }
       ```

3. 功能模块

   - 表与实体类

     ```java
     import lombok.AllArgsConstructor;
     import lombok.Data;
     import lombok.NoArgsConstructor;
     import lombok.ToString;
     
     @Data
     @NoArgsConstructor
     @AllArgsConstructor
     @ToString
     public class Book {
         private Integer id;
         private String type;
         private String name;
         private String description;
     
     }
     ```

   - dao（接口+自动代理）

     ```java
     import com.hcx.domain.Book;
     import org.apache.ibatis.annotations.Delete;
     import org.apache.ibatis.annotations.Insert;
     import org.apache.ibatis.annotations.Select;
     import org.apache.ibatis.annotations.Update;
     import org.springframework.stereotype.Repository;
     
     import java.util.List;
     
     public interface BookDao {
         /*写法一：@Insert("insert into tbl_book values(null,#{type},#{name},#{description})")*/
         @Insert("insert into tbl_book (type,name,description) values(#{type},#{name},#{description})")
         public void save(Book book);
     
         @Update("update tbl_book set type= #{type},name = #{name},description = #{description} where id = #{id}")
         public void update(Book book);
     
         @Delete("delete from tbl_book where id = #{id}")
         public void delete(Integer id);
     
         @Select("select * from tbl_book where id = #{id}")
         public Book getById(Integer id);
     
         @Select("select * from tbl_book")
         public List<Book> getAll();
     }
     ```

   - service（接口+实现类）

     - 接口

     ```java
     import com.hcx.domain.Book;
     import org.springframework.transaction.annotation.Transactional;
     
     import java.util.List;
     
     @Transactional
     public interface BookService {
         /**
          * 保存
          * @param book
          * @return
          */
         public Boolean save(Book book);
     
         /**
          * 修改
          * @param book
          * @return
          */
         public Boolean update(Book book);
     
         /**
          * 根据id删除
          * @param id
          * @return
          */
         public Boolean delete(Integer id);
     
         /**
          * 根据id查询
          * @param id
          * @return
          */
         public Book getById(Integer id);
     
         /**
          * 查询全部
          * @return
          */
         public List<Book> getAll();
     }
     ```

     实现类

     ```java
     import com.hcx.dao.BookDao;
     import com.hcx.domain.Book;
     import com.hcx.service.BookService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Service;
     
     import java.util.List;
     @Service
     public class BookServiceImpl implements BookService {
         @Autowired
         private BookDao bookDao;
         public Boolean save(Book book) {
             bookDao.save(book);
             return true;
         }
     
         public Boolean update(Book book) {
             bookDao.update(book);
             return true;
         }
     
         public Boolean delete(Integer id) {
             bookDao.delete(id);
             return true;
         }
     
         public Book getById(Integer id) {
             return bookDao.getById(id);
         }
     
         public List<Book> getAll() {
     
             return bookDao.getAll();
         }
     }
     ```

     - 业务层接口测试（整合JUnit）

       ```java
       import com.hcx.config.SpringConfig;
       import com.hcx.domain.Book;
       import org.junit.Test;
       import org.junit.runner.RunWith;
       import org.springframework.beans.factory.annotation.Autowired;
       import org.springframework.test.context.ContextConfiguration;
       import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
       
       import java.util.List;
       
       @RunWith(SpringJUnit4ClassRunner.class)
       @ContextConfiguration(classes = SpringConfig.class)
       public class BookServiceTest {
           @Autowired
           private BookService bookService;
           @Test
           public void testGetById(){
               Book book = bookService.getById(1);
               System.out.println(book);
           }
       
           @Test
           public void testGetAll(){
               List<Book> list=bookService.getAll();
               System.out.println(list);
           }
       }
       ```

   - controller

     ```java
     import com.hcx.domain.Book;
     import com.hcx.service.BookService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.*;
     
     import java.util.List;
     
     @RestController
     @RequestMapping("/books")
     public class BookController {
         @Autowired
         private BookService bookService;
     
     
         @PostMapping
         public Result save(@RequestBody Book book) {
             boolean flag = bookService.save(book);
             return new Result(flag ? Code.SAVE_OK : Code.SAVE_ERR, flag);
         }
     
         @PutMapping
         public Result update(@RequestBody Book book) {
             boolean flag = bookService.update(book);
             return new Result(flag ? Code.UPDATE_OK : Code.UPDATE_ERR, flag);
         }
     
         @DeleteMapping("/{id}")
         public Result delete(@PathVariable Integer id) {
             boolean flag = bookService.delete(id);
             return new Result(flag ? Code.DELETE_OK : Code.DELETE_ERR, flag);
         }
     
         @GetMapping("/{id}")
         public Result getById(@PathVariable Integer id) {
             Book book=bookService.getById(id);
             Integer code = book != null ? Code.GET_OK:Code.GET_ERR;
             String msg = book != null ? "数据查询成功" : "数据查询失败请重试";
             return new Result(code,book,msg);
         }
     
         @GetMapping
         public Result getAll() {
             List<Book> bookList = bookService.getAll();
             Integer code = bookList != null ? Code.GET_ALL_OK:Code.GET_ALL_ERR;
             String msg = bookList != null ? "全部数据查询成功":"数据查询失败请重试";
             return new Result(code,bookList,msg);
         }
     }
     ```

     数据传输协议所要用到的bean

     Result.java

     ```java
     import lombok.AllArgsConstructor;
     import lombok.Data;
     import lombok.NoArgsConstructor;
     import lombok.ToString;
     
     @Data
     @AllArgsConstructor
     @NoArgsConstructor
     @ToString
     public class Result {
         private Integer code;
         private Object data;
         private String msg;
     
         public Result(Integer code, Object data) {
             this.code = code;
             this.data = data;
         }
     }
     ```

     Code.java

     ```java
     public class Code {
         public static final Integer SAVE_OK = 20011;
         public static final Integer DELETE_OK = 20021;
         public static final Integer UPDATE_OK = 20031;
         public static final Integer GET_OK = 20041;
         public static final Integer GET_ALL_OK = 20051;
         public static final Integer SAVE_ERR = 20010;
         public static final Integer DELETE_ERR = 20020;
         public static final Integer UPDATE_ERR = 20030;
         public static final Integer GET_ERR = 20040;
         public static final Integer GET_ALL_ERR = 20050;
     }
     ```

     - 表现层接口测试（Apifox）

### SSM整合-表现层与前端数据传输协议定义

数据返回格式：

```json
{
	"code":Integer,
	"data":Obj,
	"msg":String
}
```

完成方式：

在Controller里面添加Result类和Code类，并让Controller里面的方法都返回Result类型的结果

### SSM整合-异常处理

**异常处理器**

- 程序开发过程中不可避免的会出现异常
- 出现异常现象的常见位在与常见诱因如下：
  - 框架内部抛出异常：因使用不规范导致
  - 数据层抛出的异常：因为外部服务器故障导致（例如：服务器访问超时）
  - 业务层抛出的异常：因业务逻辑书写错误导致（例如：遍历业务书写操作，导致索引异常等）
  - 表现层抛出的异常：因数据收集，校验规则导致（例如：不匹配的数据类型间导致异常）
  - 工具类抛出异常：因工具类书写不严谨不狗健壮导致（例如：必要释放的连接长使劲啊未释放等）

1. 各个层级均出现异常，异常处理代码书写在哪一层

   ——所有的异常均抛出到表现层处理

2. 表现层处理异常，每个方法中单独书写，代码书写量巨大且意义不强，如何解决——AOP思想

- 定义异常处理器

  - 集中的，统一的处理项目中出现的异常

  ```java
  @ResControllerAdvice
  public class ProjectExceptionAdvice{
    @ExceptionHandler(Exception.class)
    public Result doExpetion(Exception ex){
    	System.out.println("异常被捕获");
    	return new Result(001,null,"异常被捕获");
    }
  }
  ```

注解使用：

- 名称：@RestControllerAdvice

- 类型：类注解

- 位置：Rest风格开发的控制器增强类定义上方

- 作用：为Rest风格开发的控制器类做增强

- 范例：

  ```java
  @ResControllerAdvice
  public class ProjectExceptionAdvice{
  }
  ```

- 说明：

  - 此注解自带@ResponseBody注解与@Component注解，具备对应的功能

- 名称：@ExceptionHandler

- 类型：方法注解

- 位置：专用于异常处理的控制器上方

- 作用：设定指定异常的处理方案，功能等同于控制器方法，出现异常后终止原始控制器执行，并转入当前方法执行

- 范例：

  ```java
  @RestControllerAdvice
  public class ProjectExceptionAdvice{
    @ExceptionHandler(Exception.class)
    public Result doException(Exception exception){
    	return new Result(001,null,"异常已被捕获")
    }
  }
  ```

- 说明：

  - 此类方法可以根据处理的异常不同，制作多个方法分别处理对应的异常

### SSM整合-项目异常处理方案

- **项目异常分类**
  - 业务异常（BusinessException）
    - 规范的用户行为产生的异常
    - 不规范的用户行为操作产生的异常
  - 系统异常（SystemException）
    - 项目运行过程中可预计且无法避免的异常
  - 其他异常（Exception）
    - 编程人员为预期到的异常
- **项目异常处理方案**
  - 业务异常（BusinessException）
    - 发送对应消息传递给用户，提醒规范操作
  - 系统异常（SystemException）
    - 发送固定消息传递给用户，安抚用户
    - 发送特定消息给运维人员，提醒维护
    - 记录日志
  - 其他异常（Exception）
    - 发送固定消息给用户，安抚用户
    - 发送特定消息给编程人员，提醒维护（纳入预期范围内）
    - 记录日志

#### 项目异常处理

1. 自定义项目系统级异常

   ```java
   public class SystemException extends RuntimeException{
       private  Integer code;
   
       public Integer getCode() {
           return code;
       }
   
       public void setCode(Integer code) {
           this.code = code;
       }
   
       public SystemException(Integer code) {
           this.code = code;
       }
   
       public SystemException(Integer code,String message ) {
           super(message);
           this.code = code;
       }
   
       public SystemException(Integer code,String message, Throwable cause ) {
           super(message, cause);
           this.code = code;
       }
   
       public SystemException(Integer code,Throwable cause ) {
           super(cause);
           this.code = code;
       }
   
       public SystemException(Integer code,String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
           super(message, cause, enableSuppression, writableStackTrace);
           this.code = code;
       }
   }
   ```

2. 自定义项目业务级异常

   ```java
   public class BusinessException extends RuntimeException{
       private  Integer code;
   
       public Integer getCode() {
           return code;
       }
   
       public void setCode(Integer code) {
           this.code = code;
       }
   
       public BusinessException(Integer code) {
           this.code = code;
       }
   
       public BusinessException(Integer code, String message ) {
           super(message);
           this.code = code;
       }
   
       public BusinessException(Integer code, String message, Throwable cause ) {
           super(message, cause);
           this.code = code;
       }
   
       public BusinessException(Integer code, Throwable cause ) {
           super(cause);
           this.code = code;
       }
   
       public BusinessException(Integer code, String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
           super(message, cause, enableSuppression, writableStackTrace);
           this.code = code;
       }
   }
   ```

3. 自定义异常编码（持续补充）

   ```java
   public class Code{
   public static final Integer SAVE_OK = 20011;
       public static final Integer DELETE_OK = 20021;
       public static final Integer UPDATE_OK = 20031;
       public static final Integer GET_OK = 20041;
       public static final Integer GET_ALL_OK = 20051;
       public static final Integer SAVE_ERR = 20010;
       public static final Integer DELETE_ERR = 20020;
       public static final Integer UPDATE_ERR = 20030;
       public static final Integer GET_ERR = 20040;
       public static final Integer GET_ALL_ERR = 20050;
   
       public static final Integer SYSTEM_ERR = 20060;
       public static final Integer SYSTEM_TIMEOUT_ERR = 20070;
   
       public static final Integer BUSINESS_ERR = 20080;
       public static final Integer SYSTEM_UNKNOW_ERR = 20090;
   }
   ```

4. 触发定义异常

   ```java
   @Service
   public class BookServiceImpl implements BookService {
       @Autowired
       private BookDao bookDao;
       public Boolean save(Book book) {
           bookDao.save(book);
           return true;
       }
   
       public Boolean update(Book book) {
           bookDao.update(book);
           return true;
       }
   
       public Boolean delete(Integer id) {
           bookDao.delete(id);
           return true;
       }
   
       public Book getById(Integer id) {
           //模拟业务异常
           if(id == 1){
               throw new BusinessException(Code.BUSINESS_ERR,"访问连接过多异常");
           }
           //将可能出现的异常进行包装，转换称自定义异常
           try {
               int i = 1/0;
           }catch (Exception exception){
               throw new SystemException(Code.SYSTEM_TIMEOUT_ERR,"服务器访问超时，请重试",exception);
           }
           return bookDao.getById(id);
       }
   
       public List<Book> getAll() {
           return bookDao.getAll();
       }
   }
   ```

5. 拦截并处理异常

   ```java
   @RestControllerAdvice
   public class ProjectExceptionAdvice {
       @ExceptionHandler(Exception.class)
       public Result doException(Exception exception) {
           //记录日志(错误堆栈)
           //发送消息给运维
           //发送邮件给开发人员，exception对象发送给开发人员
           System.out.println("其他异常被捕获");
           return new Result(Code.SYSTEM_UNKNOW_ERR, null, "系统繁忙请稍后再试");
       }
   
       @ExceptionHandler(SystemException.class)
       public Result doSystemException(SystemException exception) {
           //记录日志(错误堆栈)
           //发送消息给运维
           //发送邮件给开发人员，exception对象发送给开发人员
           System.out.println("系统异常被捕获");
           return new Result(exception.getCode(), null, exception.getMessage());
       }
   
       @ExceptionHandler(BusinessException.class)
       public Result doSystemException(BusinessException exception) {
           System.out.println("业务异常被捕获");
           return new Result(exception.getCode(), null, exception.getMessage());
       }
   }
   ```

6. 异常处理效果对比

**总结：**

- 异常处理器
- 自定义异常
- 异常编码
- 自定义消息

### SSM整合标准开发

...

### SpringMVC拦截器

#### 拦截器概念

- 拦截器（Interceptor）是一种动态拦截方法调用的机制，再SpringMVC中动态拦截控制器方法的执行
- 作用：
  - 在指定的方法调用前后执行预先设定的代码
  - 阻止原始方法的执行

#### 拦截器和过滤器的区别

- 归属不同：Filter属于Servlet计算，Interceptor属于SpringMvc技术
- 拦截内容不同：Filter对所有访问进行增强，Interceptor仅针对SpringMVC的访问进行增强

#### 拦截器入门案例

1. 制作拦截器功能类

2. 配置拦截器的执行位置

3. 声明拦截器的bean，并实现HandlerInterCeptor接口（注意：扫描加载bean）

   ```java
   @Component
   public class ProjectInterCeption implements HandlerInterceptor{
    public boolean preHandle(..) throws Exception{
    	System.out.println("preHandle...");
    	return true;
    }
    public void postHandle(..) throws Exception{
    	System.out.println("postHandle...");
    }
    public void afterCompletion(..) throws Exception{
    	Syetm.out.println("afterCompletion...");
    }
   }
   ```

4. 定义配置类，集成WebMvcConfigurationSupport,实现addInterceptor方法（注意：扫描加载配置）

   ```java
   @Configuration
   public class SpringMvcSupport extends WebMvcConfigurationSupport{
    @Override
    public void addInterceptors(InterceptorRegistry registry){
    	...
    }
   }
   ```

5. 添加拦截器并设置拦截访问路径，路径可以通过可变参数设置多个

   ```java
   @Configuration
   public class SpringMvcSupport extends WebMvcConfigurationSupport{
    @Autowired
    private ProjectInterceptor projectInterceptor;
   
    @Override
    public void addInterCeptors(InterceptorRegisetry registry){
    	regisyry.addInterceptor(proejectInterceptor).addPathPatternas("/books");
    }
   }
   ```

6. 使用标准接口WebMvcConfigurer简化开发（注意：侵入式较强）

   ```java
   @Configuration
   @ComponentScan("com.hcx.controller")
   @EnableWebMvc
   public class SpringMvcConfig implements WebMvcConfigurer{
    @Autowired
    private ProjectInterCeptor projectInterceptor;
   
    public void addInterceptors(InterceptorRegistry registry){
    	registry.addInterceptor(projectInterceptor).addPatternas("/books","/books");
    }
   }
   ```

#### 拦截器执行流程

- preHandle
  - return true
    - contrller
    - postHandle
    - afterCompletion
  - return false
    - 结束

#### 拦截器参数

- **前置处理**

  ```java
  @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          System.out.println("preHandle...");
          return true;
      }
  ```

- **具体参数：**

  - handler：被调用的处理器对象，本质上是一个方法对象，对反射计数中的Method对象进行了再包装，可用于获取拦截方法信息
  - request：请求对象
  - response：响应对象
  - modelAndView:获取页面跳转相关数据
  - ex：拿到原始程序执行过程中出现的异常，表现层出现的异常

- **返回值**

  - 返回值未false，被拦截的处理器将不执行

#### 多个拦截器执行顺序

- 当配置多个拦截器是，形成拦截器链
- 拦截器链的运行属性呢参照拦截器添加顺序未准
- 当拦截器中出现对原始处理器的拦截，后面的拦截器均终止运行
- 当拦截器运行中断，仅运行配置在前面的拦截器的afterCompletion操作

##### 总结：

1. 拦截器链配置方式

2. 拦截器链的运行顺序

   - preHandle:与配置顺序相同，必定运行【配置顺序就是运行顺序】
   - postHandle：与配置顺序相反，可能不运行【配置顺序就是运行顺序反过来】
   - afterComletion: 与配置顺序相反，可能不运行【配置顺序就是运行顺序反过来】

   解释：顺序情况下的话，走到哪里出现错误返回false，则后面的postHandle拦截器都不运行，afterComletion会运行