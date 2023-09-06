## Spring Security

FilterSecurityInterceptor：一个方法级的权限过滤器，基本位于过滤链的最底部

ExceptionTranslationFilter：异常过滤器，处理在认证授权过程中抛出的异常

UsernamePasswordAuthenticationFilter：对login的post请求做拦截，验证表单中的用户名，密码

### springscurity的运行逻辑

#### 过滤器是如何加载的

DelegatingFilterProxy(dofilter)

doFilter(initDelegate)

`后面没懂`

#### UserDetailsService接口

- 创建类继承UsernamePasswordAuthenticationFilter，重写attemptAuthentication、successfulAuthentication、unsuccessfulAuthentication三个方法

- 创建类实现UserDetailsService接口，编写查询数据库中的用户名和密码的方法，返回user对象，这个对象是安全框架提供的

#### PasswordEncoder接口

数据加密接口，用于返回User对象里面的密码加密





#### web权限方案

设置登录的用户名和密码

- 配置文件

~~~xml
spring.security.user.name=hatu
spring.security.user.passsword=123
~~~

- 配置类

~~~java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //密码加密
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encode = passwordEncoder.encode("123456");
        //设置用户名，密码，角色
        auth.inMemoryAuthentication().withUser("hatu").password(encode).roles("admin");
    }

    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder();
    }
}
~~~

- 自定义实现类

  ~~~java
  @Configuration
  public class SecurityConfig2 extends WebSecurityConfigurerAdapter {
      @Autowired
      private UserDetailsService userDetailsService;
  
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          auth.userDetailsService(userDetailsService).passwordEncoder(password());
  
      }
  
      @Bean
      PasswordEncoder password() {
          return new BCryptPasswordEncoder();
      }
  }
  
  
  @Service("userDetailsService")
  public class MyUserDetailsService implements UserDetailsService {
      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
          return new User("marry", new BCryptPasswordEncoder().encode("123"), auths);
      }
  }
  ~~~



#### 实现数据库认证完成用户登录

- 引入相关依赖
- 创建数据库、表，并配置
- 创建user对应实体类
- 创建接口，继承BaseMapper<>，加注解@Repository

~~~java
@Repository
public interface UserMapper extends BaseMapper<MyUser> {
}
~~~

- 在MyUserDetailsService中调用接口，数据查询

~~~java
@Service("detailsService")
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //根据用户名进行查询
        QueryWrapper<MyUser> wrapper = new QueryWrapper();
        wrapper.eq("username", username);
        MyUser user = userMapper.selectOne(wrapper);
        if (user == null) {
            //没有数据，认证失败
            throw new UsernameNotFoundException("用户名不存在");
        }
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User(user.getUsername(), new BCryptPasswordEncoder().encode(user.getPassword()), auths);
    }
}
~~~

- 启动类加注解mapperScan(mapper所在的包)

#### 自定义用户登录界面

在配置类中重写方法

~~~java
@Configuration
public class SecurityConfig3 extends WebSecurityConfigurerAdapter {
	//重写的其它方法
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html")  //自定义登录页面
                .loginProcessingUrl("/user/login") //登录页面设置
                .defaultSuccessUrl("/ok/index").permitAll() //登录访问路径
                .and()
                .authorizeRequests()
                .antMatchers("/", "test/hello", "user/login").permitAll()  //设置不需要认证的路径
                .anyRequest().authenticated()
                .and()
                .csrf().disable();  //关闭防护模式
    }
}
~~~



#### 基于角色或权限进行访问控制

**hasAuthority**方法

在配置类设置，当前访问地址有哪些权限

~~~java
 .antMatchers("/ok/index").hasAuthority("admin") //设置访问权限
~~~

在MyUserDetailsService，把返回User对象设置权限

~~~java
 List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
~~~

**hasanyAuthority**方法

在配置类设置，当前访问地址有哪些权限

~~~java
.antMatchers("/ok/index").hasAnyAuthority("admins,manager")  //设置访问权限，可以多个权限
~~~

**hasRole**方法

如果当前用户具备给定的角色就允许访问，否则出现403

在配置类设置，当前访问地址有哪些角色

~~~java
 .antMatchers("/ok/index").hasRole("sale")  //设置访问角色，一个角色
~~~

在MyUserDetailsService，把返回User对象设置角色，加`ROLE_`

~~~java
 List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale,ROLE_emp");
~~~

**hasAnyRole**方法类似上面

#### 设置没有权限时跳转的界面

~~~java
 http.exceptionHandling().accessDeniedPage("/unauth.html"); //配置没有权限时跳转到的界面
~~~

---

### 注解的使用

#### @Secured  用户具有某个角色，可以访问方法

在启动类上添加注解`@EnableGlobalMethodSecurity(securedEnabled = true)`

在Controller的方法上使用@Secured（“ROLE_sale”,”ROLE_emp”）

在MyUserDetailsService，把返回User对象设置角色



#### @PreAuthorize     进入方法前进行权限验证，可以将登录用户的roles、permissins参数传到方法中

开启注解`@EnableGlobalMethodSecurity(securedEnabled = true,preAuthorize= true)`

在Controller的方法上使用@PreAuthorize（“hasAnyAuthority(‘`admins`’)”）



#### @PostAuthorize   在方法执行后进行权限验证，适合带有返回值的权限

开启注解`@EnableGlobalMethodSecurity(securedEnabled = true,preAuthorize= true，prePostAuthorize = true`

在Controller的方法上使用@PostAuthorize （“hasAnyAuthority(‘menu:system’)”）



#### @PostFilter  权限验证之后对数据进行过滤，留下用户名是admin1的数据，表达式中filterObject引用的是方法返回值List中的某一个元素

#### @PreFilter   传入方法的数据进行过滤

### 用户注销

~~~java
 http.logout().logoutUrl("/logout").logoutSuccessUrl("index").permitAll(); //退出路径设置
~~~

### 基于数据库实现记住我

创建数据表  persistent_logins  （可以自动创建）

配置类操作

~~~java
  //注入数据源
    @Autowired
    private DataSource dataSource;
    //配置对象
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        jdbcTokenRepository.setCreateTableOnStartup(true);  //自动创建对应的数据表
        return jdbcTokenRepository;
    }

//配置相关配置，实现自动登录
.rememberMe().tokenRepository(persistentTokenRepository())  //设置免登录
                .tokenValiditySeconds(60) //设置时间，以秒为单位
                .userDetailsService(detailsService)
~~~

登录页面中，进行相关配置 

~~~html
 <input type="checkbox" name="remember-me"/>  自动登录
~~~

### CSRF功能

.csrf().disable();  //关闭防护模式     把这个代码注释掉，开启防护模式，在登录界面添加一段代码





………………………………未完待续