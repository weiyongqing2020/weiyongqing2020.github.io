# Spring概述
通常说的spring指spring framework，而spring framework是spring家族的一个分支。
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/1.png)
  
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/2.png)
# ioc
控制反转，即把对象的创建，初始化，销毁都交给容器来按完成。 
 
DI(依赖注入)是实现控制反转的一种方式。
## 对象注入到容器的主要方式：
1. xml配置文件注入
```
  <bean class="org.javaboy.Book" id="book"/>
```
id与name基本没有区别，只不过name可以命名多个，创建多个对象，id不可以。
对于一些用build方法来进行构造的外部bean，可以通过工厂方法来将它注入到ioc容器中。
静态工厂：
```
public class OkHttpUtils {
    private static OkHttpClient OkHttpClient;
    public static OkHttpClient getInstance() {
        if (OkHttpClient == null) {
            OkHttpClient = new OkHttpClient.Builder().build();
        }
        return OkHttpClient;
    }
}
<bean class="org.javaboy.OkHttpUtils" factory-method="getInstance" id="okHttpClient"></bean>
```
实例工厂：
```
public class OkHttpUtils {
    private OkHttpClient OkHttpClient;
    public OkHttpClient getInstance() {
        if (OkHttpClient == null) {
            OkHttpClient = new OkHttpClient.Builder().build();
        }
        return OkHttpClient;
    }
}


<bean class="org.javaboy.OkHttpUtils" id="okHttpUtils"/>
<bean class="okhttp3.OkHttpClient" factory-bean="okHttpUtils" factory-method="getInstance" id="okHttpClient"></bean>
```

2. java配置
```
@Configuration
public class JavaConfig {
    @Bean
    SayHello sayHello() {
        return new SayHello();
    }
}
```

@Configuration表示这是一个配置类，作用等同于 applicationContext.xml 文件。
@Bean注解的方法的返回值将注入到spring容器中，bean的名称默认是方法名，也可以在注解中指定。
需要在项目启动时加载配置类：
```
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JavaConfig.class);
        SayHello hello = ctx.getBean(SayHello.class);
        System.out.println(hello.sayHello("javaboy"));
```

3. 自动化扫描
实际开发中主要使用自动化扫描，配置自动化扫描可以通过xml，也可以通过@configuration.
```
@Configuration
@ComponentScan(basePackages = "org.javaboy.javaconfig.service")
public class JavaConfig {
}


<context:component-scan base-package="org.javaboy.javaconfig"/>
```
自动化扫描的注解：
- Repository  在dao层添加
- Service  在service层添加
- Controller  在dao层添加
- Component 通用

bean的名字默认类名首字母小写。

## 属性注入的几种方式：
1. 通过构造方法注入
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.javaboy.Book" id="book">
        <constructor-arg index="0" value="1"/>
        <constructor-arg index="1" value="三国演义"/>
        <constructor-arg index="2" value="30"/>
    </bean>
</beans>


<bean class="org.javaboy.Book" id="book2">
    <constructor-arg name="id" value="2"/>
    <constructor-arg name="name" value="红楼梦"/>
    <constructor-arg name="price" value="40"/>
</bean>
```
2. set方法的注入
```
<bean class="org.javaboy.Book" id="book3">
    <property name="id" value="3"/>
    <property name="name" value="水浒传"/>
    <property name="price" value="30"/>
</bean>
```
3. p名称控件注入  

复杂属性的注入：

- 数组注入：
```
<bean class="org.javaboy.User" id="user">
    <property name="cat" ref="cat"/>
    <property name="favorites">
        <array>
            <value>足球</value>
            <value>篮球</value>
            <value>乒乓球</value>
        </array>
    </property>
</bean>
<bean class="org.javaboy.Cat" id="cat">
    <property name="name" value="小白"/>
    <property name="color" value="白色"/>
</bean>
```
- map注入：
```
property name="map">
    <map>
        <entry key="age" value="99"/>
        <entry key="name" value="javaboy"/>
    </map>
</property>
```
- 也可以从配置文件中获取属性的值：
```
<context:property-placeholder location="classpath:config.properties"/>
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
<!-- 连接数据库的驱动，连接字符串，用户名和登录密码-->
<property name="driverClassName" value="${jdbcDriver}"/>
<property name="url" value="${jdbcUrlZsjyjl}"/>
<property name="username" value="${jdbcUsername}"/>
<property name="password" value="${jdbcPassword}"/>

<!-- 数据池中最大连接数和最小连接数-->
<property name="maxActive" value="10"/>
<property name="minIdle" value="10"/>
</bean>
```
通过注解注入属性时，可以用@value的方式，也可以用类型安全的属性注入：
```
@Component
@Data
@PropertySource("classpath:book.properties")
public class Book {
    @Value("${book.id}")
    private long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;
}



@Component
@Data
@PropertySource("classpath:book.properties")
@ConfigurationProperties(prefix = "book")
public class Book {
    private long id;
    private String name;
    private String author;
}
```

从容器中取出对象的几种方式  
- @Autowire 
- @Resources
- @Injected  
@Autowire 按照bean的类型去取，@Resources按照bean的名称去取。如果存在同一类型的多个bean，用@Autowire会报错。可以将@Autowire与@Qualifier配合使用，@Qualifier可指定
## 条件注解
springboot实现自动装配的关键
## aware
## bean的生命周期
向spring容器注入bean默认是单例模式，可以通过设置scope手动配置，scope设置为prototype，表示每次从容器中取都是不同的对象。scope也可以设置为request 和 session，在web环境中有效。



# aop
面向切面编程，提取模板化代码，弥补面向对象的不足。
  
|概念|说明|
|----|----|
|切点|要添加代码的地方，称作切点|
|通知（增强）|通知就是向切点动态添加的代码|
|切面|切点+通知|
|连接点|切点的定义|

aop通过动态代理来实现，动态地生成代理类。

java中有两种实现动态代理的方式：
- jdk
- cglib

jdk动态代理需要实现接口，而cglib不需要，cglib即是增强版的aop。

五种通知：
- 前置通知
- 后置通知
- 返回通知
- 异常通知
- 环绕通知

动态代理相当于是对方法包了一层try，catch，finally。

其中环绕通知就相当于是动态代理本身。
```
public interface Calculate {
    int add(int i,int j);
    int sub(int i,int j);
    int mul(int i,int j);
    int div(int i,int j);
    void test(int i, int j);
}


public int add(int i, int j) {
    System.out.printf("方法内部调用");
    return i+j;
}

public int sub(int i, int j) {
    return i-j;
}

public int mul(int i, int j) {
    return i*j;
}

public int div(int i, int j) {
    return i/j;
}


public class CalculateAspect {

    @Pointcut("execution(* com.sudy.aop.CalculateImpl.add(int,int))")
    private void aspectJMethod() {

    }

    @Before("aspectJMethod()")
    public void beforeCalculate(JoinPoint joinPoint) {
        Object[] objects = joinPoint.getArgs();
        System.out.printf("【前置通知】-开始计算前,参数：" + objects[0] + "," + objects[1]);
    }

    @After(value = "aspectJMethod()")
    public void afterCalculate() {
        System.out.printf("【后置通知】");
    }

    @AfterThrowing(value = "aspectJMethod()", throwing = "exception")
    public void expectionDeal(JoinPoint joinPoint, Exception exception) {
        System.out.printf("【异常通知】-异常信息：" + exception.toString());
    }

    @AfterReturning(value = "aspectJMethod()", returning = "returnObj")
    public void afterReturnCalculate(JoinPoint joinPoint, Object returnObj) {
        System.out.printf("【后置返回通知】-计算结果："+returnObj);
    }

    @Around(value = "aspectJMethod()")
    public Object aroundCalculate(ProceedingJoinPoint pjp) throws Throwable {
        System.out.printf("【环绕通知】");
        //try{
        //pjp.getArgs()[0]= (Integer) 10;
        return pjp.proceed(pjp.getArgs());
        //}
        /*catch (Throwable ex){
            System.out.printf("环绕通知内部异常："+ex.getMessage());
            return null;
        }*/


    }
}


<context:component-scan base-package="com.sudy"></context:component-scan>

<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
# JdbcTemplate和与声明式事务
jdbcTemplate是spring用aop思想封装的jdbc操作工具。
```
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
```
```
@Configuration
public class JdbcConfig {
    @Bean
    DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUsername("root");
        dataSource.setPassword("123");
        dataSource.setUrl("jdbc:mysql:///test01");
        return dataSource;
    }
    @Bean
    JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }
}
```
```
public class Main {

    private JdbcTemplate jdbcTemplate;

    @Before
    public void before() {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(JdbcConfig.class);
        jdbcTemplate = ctx.getBean(JdbcTemplate.class);
    }

    @Test
    public void insert() {
        jdbcTemplate.update("insert into user (username,address) values (?,?);", "javaboy", "www.javaboy.org");
    }
    @Test
    public void update() {
        jdbcTemplate.update("update user set username=? where id=?", "javaboy123", 1);

    }
    @Test
    public void delete() {
        jdbcTemplate.update("delete from user where id=?", 2);
    }

    @Test
    public void select() {
        User user = jdbcTemplate.queryForObject("select * from user where id=?", new BeanPropertyRowMapper<User>(User.class), 1);
        System.out.println(user);
    }
}
```
```
@Test
public void select3() {
    User user = jdbcTemplate.queryForObject("select * from user where id=?", new RowMapper<User>() {
        public User mapRow(ResultSet resultSet, int i) throws SQLException {
            int id = resultSet.getInt("id");
            String username = resultSet.getString("username");
            String address = resultSet.getString("address");
            User u = new User();
            u.setId(id);
            u.setName(username);
            u.setAddress(address);
            return u;
        }
    }, 1);
    System.out.println(user);
}
```

spring的事务，利用了aop的思想，简化了事务的配置。  
转账的例子：
```
@Repository
public class UserDao {
    @Autowired
    JdbcTemplate jdbcTemplate;

    public void addMoney(String username, Integer money) {
        jdbcTemplate.update("update account set money=money+? where username=?", money, username);
    }

    public void minMoney(String username, Integer money) {
        jdbcTemplate.update("update account set money=money-? where username=?", money, username);
    }
}
@Service
public class UserService {
    @Autowired
    UserDao userDao;
    @Transactional
    public void updateMoney() {
        userDao.addMoney("zhangsan", 200);
        int i = 1 / 0;
        userDao.minMoney("lisi", 200);
    }
}
<tx:annotation-driven transaction-manager="transactionManager" />
```
