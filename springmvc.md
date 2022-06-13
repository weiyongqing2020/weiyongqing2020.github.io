# SpringMVC概述
## SpringMVC的特点
- 更简洁的web层开发
- 与spring框架无缝衔接
- 注解支持restful风格
- 支持简单的异常处理
- 支持web层简单的单元测试
## SpringMVC的工作流程
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/5.png)
## SpringMVC的各个组件
- 前端控制器(DispatcherServlet)SpringMVC的核心，拦截所有请求，解耦其他模块。
- 处理器映射器(HandlerMapping)
- 根据请求的url找到对应的handler及 Interceptor 拦截器，将它们封装在HandlerExecutionChain中返回前端控制器。 SpringMVC提供了多重映射方式，比如配置文件方式，实现接口方式，注解方式等，在开发中常用注解方式。
- 处理器（Handler）即自己开发的controller
- 处理器适配器(HandlerAdapter)通过HandlerAdapter执行处理器
- 视图解析器(ViewResolver)将逻辑视图解析为物理视图，生成View视图对象

# SpringMVC的配置与使用
依赖：
```
 <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
```
添加了 spring-webmvc 依赖之后，其他的 spring-web、spring-aop、spring-context 等等就全部都加入进来了。  
web-xml里的配置：
```
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

init-param里指定了springmvc配置文件的位置，如果放在webapp/WEB-INF/文件夹下，且文件名为DispatcherServlet 的名字+ -servlet，则可以省略init-param这个配置。


其他的配置：  

| 参数 | 描述 |
| -------- | -------- |
|contextClass| 实现WebApplicationContext接口的类，当前的servlet用它来创建上下文。如果这个参数没有指定， 默认使用XmlWebApplicationContext。| 
| contextConfigLocation|  传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。| 
| namespace|  WebApplicationContext命名空间。默认值是[server-name]-servlet。| 

springmvc的配置文件：
```
<context:component-scan base-package="org.javaboy.helloworld" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--这个是处理器映射器，这种方式，请求地址其实就是一个 Bean 的名字，然后根据这个 bean 的名字查找对应的处理器-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" id="handlerMapping">
    <property name="beanName" value="/hello"/>
</bean>
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" id="handlerAdapter"/>
<!--视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
    <property name="prefix" value="/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```
在实际的开发中，springmvc和spring是分开配置的，也就是说有两个容器。spring的容器是全局的，
spring配置文件的加载，在web.xml中配置：
```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/6.png)

可以在springmvc容器中配置所有的bean，但一般不这样做。

不能在spring容器中配置所有的bean。

在开发中。我们一般使用RequestMappingHandlerAdapter 和RequestMappingHandlerAdapter来做处理器映射器和处理器适配器。 
 ```
 <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" id="handlerMapping"/>
  <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" id="handlerAdapter"/>
```
上面两个配置可以简写为：
```
 <mvc:annotation-driven/>
```

再加上对静态资源的配置：  
springmvc中，静态资源会被拦截，需要添加配置
````
<mvc:resources mapping="/static/html/**" location="/static/html/"/>
<mvc:resources mapping="/static/js/**" location="/static/js/"/>
<mvc:resources mapping="/static/css/**" location="/static/css/"/>
````
在映射路径的定义中，最后是两个 *，这是一种 Ant 风格的路径匹配符号，一共有三个通配符：

|通配符| 含义|
|----|----|
|**| 匹配多层路径|
|*| 匹配一层路径|
|?| 匹配任意单个字符|

可以简化为：
```
<mvc:resources mapping="/**" location="/"/>
```

springmvc配置的最终形态:
````
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="org.javaboy.helloworld"/>

    <mvc:annotation-driven/>
    
    <mvc:resources mapping="/**" location="/"/>
    
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="viewResolver">
        <property name="prefix" value="/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
````
## RequestMapping注解
@RequestMapping用来标记一个接口
````
@Controller
public class HelloController {
    @RequestMapping({"/hello","/hello2"})
    public ModelAndView hello() {
        return new ModelAndView("hello");
    }
}
````
请求窄化
````
@Controller
@RequestMapping("/user")
public class HelloController {
    @RequestMapping({"/hello","/hello2"})
    public ModelAndView hello() {
        return new ModelAndView("hello");
    }
}
````
请求方法的绑定
````
@Controller
@RequestMapping("/user")
public class HelloController {
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public ModelAndView hello() {
        return new ModelAndView("hello");
    }
}
````
绑定请求的方法后，前端如果以错误的方法访问，会报405
## controller的返回值
- 返回ModelAndView  
在前后端部分的开发中，经常需要返回ModelAndView，即数据模型+视图。
````
@Controller
@RequestMapping("/user")
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView hello() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("username", "javaboy");
        return mv;
    }
}
````
- 返回void  
不是真的返回空，而是用servlet的那一套方案。
通过HttpServletRequest 做服务器端跳转：
````
@RequestMapping("/hello2")
public void hello2(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    req.getRequestDispatcher("/jsp/hello.jsp").forward(req,resp);//服务器端跳转
}
````

通过HttpServletResponse 浏览器重定向：
````
@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.sendRedirect("/hello.jsp");
}
````
手动设置响应头来实现重定向：
````
@RequestMapping("/hello3")
public void hello3(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setStatus(302);
    resp.addHeader("Location", "/jsp/hello.jsp");
}


@RequestMapping("/hello4")
public void hello4(HttpServletRequest req, HttpServletResponse resp) throws IOException {
    resp.setContentType("text/html;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write("hello javaboy!");
    out.flush();
    out.close();
}
````
- 返回字符串  
Model在参数中设置，返回视图（视图逻辑名）
````
@RequestMapping("/hello5")
public String hello5(Model model) {
    model.addAttribute("username", "javaboy");//这是数据模型
    return "hello";//表示去查找一个名为 hello 的视图
}
````
通过返回字符串来做重定向和服务器跳转：
````
@RequestMapping("/hello5")
public String hello5() {
    return "forward:/jsp/hello.jsp";
}


@RequestMapping("/hello5")
public String hello5() {
    return "redirect:/user/hello";
}
````

真的返回字符串可以用@ResponseBody注解。  
## controller的参数  
controller的参数可以是以下几种：  
- HttpServletRequest
- HttpServletResponse
- HttpSession
- Model

对于前端以key-value形式的传参，基本数据类型的参数可以直接解析。
````
@Controller
public class BookController {
    @RequestMapping("/book")
    public String addBook() {
        return "addbook";
    }

    @RequestMapping(value = "/doAdd",method = RequestMethod.POST)
    @ResponseBody
    public void doAdd(String name,String author,Double price,Boolean ispublic) {
        System.out.println(name);
        System.out.println(author);
        System.out.println(price);
        System.out.println(ispublic);
    }
}
````

参数是实体类也可以直接解析
````
@RequestMapping(value = "/doAdd",method = RequestMethod.POST)
@ResponseBody
public void doAdd(Book book) {
    System.out.println(book);
}
````

而对于key与方法的参数名不一样的情况， 可以通过@RequestParam注解， @RequestParam可以指定前端传来的参数名，设置参数是否是必须的，以及给参数设置一个默认值。
````
@RequestMapping(value = "/doAdd",method = RequestMethod.POST)
@ResponseBody
public void doAdd(@RequestParam(value = "name",required = true,defaultValue = "三国演义") String bookname, String author, Double price, Boolean ispublic) {
    System.out.println(bookname);
    System.out.println(author);
    System.out.println(price);
    System.out.println(ispublic);
}
````


# 文件上传
springmvc的文件上传主要有两种方式：
- CommonsMultipartResolver
- StandardServletMultipartResolver

CommonsMultipartResolver对于浏览器的兼容更好，但是要额外导入依赖。
StandardServletMultipartResolver对于浏览器的兼容较差，不依赖第三方工具。
- CommonsMultipartResolver的使用：
````
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
````
````
<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
    <!--默认的编码-->
    <property name="defaultEncoding" value="UTF-8"/>
    <!--上传的总文件大小-->
    <property name="maxUploadSize" value="1048576"/>
    <!--上传的单个文件大小-->
    <property name="maxUploadSizePerFile" value="1048576"/>
    <!--内存中最大的数据量，超过这个数据量，数据就要开始往硬盘中写了-->
    <property name="maxInMemorySize" value="4096"/>
    <!--临时目录，超过 maxInMemorySize 配置的大小后，数据开始往临时目录写，等全部上传完成后，再将数据合并到正式的文件上传目录-->
    <property name="uploadTempDir" value="file:///E:\\tmp"/>
</bean>
````
````
@RequestMapping("/upload")
@ResponseBody
public String upload(MultipartFile file, HttpServletRequest req)  {
    String format = sdf.format(new Date());
    String realPath = req.getServletContext().getRealPath("/img") + format;
    System.out.printf("realPath:"+realPath);
    File folder = new File(realPath);
    if (!folder.exists()) {
        folder.mkdirs();
    }
    String oldName = file.getOriginalFilename();
    String newName = UUID.randomUUID().toString() + oldName.substring(oldName.lastIndexOf("."));

    try {
        file.transferTo(new File(folder, newName));
        String url = req.getScheme() + "://" + req.getServerName() + ":" + req.getServerPort() + "/img" + format + newName;
        return url;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return "failed";
}
````

上述代码获取文件的旧名，加了个UUID作为新名，防止文件名冲突，然后将文件以新名放进指定的文件夹里。
对于多文件上传，key相同的以数组来接收即可，key不同的用多个参数来接收。
````
@RequestMapping("/upload2")
@ResponseBody
public void upload2(MultipartFile[] files, HttpServletRequest req) {
    
}
@RequestMapping("/upload3")
@ResponseBody
public void upload3(MultipartFile file1, MultipartFile file2, HttpServletRequest req) {
    
}
````


# SpringMVC的全局异常处理
将异常的堆栈信息直接展示给用户，用户体验不好，而且不安全。

springmvc以@ControllerAdvice和@ExceptionHandler这两个注解来实现全局的异常处理。
````
@ControllerAdvice//表示这是一个增强版的 Controller，主要用来做全局数据处理
public class MyException {
    @ExceptionHandler(Exception.class)
    public ModelAndView fileuploadException(Exception e) {
        ModelAndView error = new ModelAndView("error");
        error.addObject("error", e.getMessage());
        return error;
    }
}
````

# 服务器端的数据校验
客户端的校验更多的是为了用户体验，前端无法保证数据的完整性，B/S架构中。可以很方便的获取请求的地址，传递非法参数，所以服务器端必须做数据的二次校验。

## 用 Hibernate Validator来做数据校验
````
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.1.0.Final</version>
</dependency>
````
在springmvc的配置文件中配置：
````
<bean class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" id="validatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
</bean>
<mvc:annotation-driven validator="validatorFactoryBean"/>
````
在传递的实体类中添加注解：
````
public class Student {
    @NotNull
    private Integer id;
    @NotNull
    @Size(min = 2,max = 10)
    private String name;
    @Email
    private String email;
    @Max(150)
    private Integer age;  
}
````
controller的方法参数中添加注解@Validated：
````
@Controller
public class StudentController {
    @RequestMapping("/addstudent")
    @ResponseBody
    public void addStudent(@Validated Student student, BindingResult result) {
        if (result != null) {
            //校验未通过，获取所有的异常信息并展示出来
            List<ObjectError> allErrors = result.getAllErrors();
            for (ObjectError allError : allErrors) {
                System.out.println(allError.getObjectName()+":"+allError.getDefaultMessage());
            }
        }
    }
}
````
自定义显示的错误信息：
在类路径下新建MyMessage.properties配置文件。
````
student.id.notnull=id 不能为空
student.name.notnull=name 不能为空
student.name.length=name 最小长度为 2 ，最大长度为 10
student.email.error=email 地址非法
student.age.error=年龄不能超过 150
````
在springmvc的配置文件中加载这个配置：
````
<bean class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" id="validatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
    <property name="validationMessageSource" ref="bundleMessageSource"/>
</bean>
<bean class="org.springframework.context.support.ReloadableResourceBundleMessageSource" id="bundleMessageSource">
    <property name="basenames">
        <list>
            <value>classpath:MyMessage</value>
        </list>
    </property>
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="cacheSeconds" value="300"/>
</bean>
<mvc:annotation-driven validator="validatorFactoryBean"/>
````
最后，在实体类上的注解中，加上校验出错时的信息：
````
public class Student {
    @NotNull(message = "{student.id.notnull}")
    private Integer id;
    @NotNull(message = "{student.name.notnull}")
    @Size(min = 2,max = 10,message = "{student.name.length}")
    private String name;
    @Email(message = "{student.email.error}")
    private String email;
    @Max(value = 150,message = "{student.age.error}")
    private Integer age;
    }
````
## 数据校验的分组：
在不同的业务逻辑下，对数据的校验规则可能不一样，比如student添加时id可以为空，修改时不能为空。
先定义空接口：
````
public interface ValidationGroup1 {
}
public interface ValidationGroup2 {
}
````
然后，在实体类中，指定每一个校验规则所属的组：
````
public class Student {
    @NotNull(message = "{student.id.notnull}",groups = ValidationGroup1.class)
    private Integer id;
    @NotNull(message = "{student.name.notnull}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    @Size(min = 2,max = 10,message = "{student.name.length}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    private String name;
    @Email(message = "{student.email.error}",groups = {ValidationGroup1.class, ValidationGroup2.class})
    private String email;
    @Max(value = 150,message = "{student.age.error}",groups = {ValidationGroup2.class})
    private Integer age;
    }
````
最后，在接收参数的地方，指定校验组：
````
@Controller
public class StudentController {
    @RequestMapping("/addstudent")
    @ResponseBody
    public void addStudent(@Validated(ValidationGroup2.class) Student student, BindingResult result) {
        if (result != null) {
            //校验未通过，获取所有的异常信息并展示出来
            List<ObjectError> allErrors = result.getAllErrors();
            for (ObjectError allError : allErrors) {
                System.out.println(allError.getObjectName()+":"+allError.getDefaultMessage());
            }
        }
    }
}
````
# SpringMVC的拦截器
springmvc自定义拦截器：  
实现接口HandlerInterceptor 
````
@Component
public class MyInterceptor1 implements HandlerInterceptor {
    /**
     * 这个是请求预处理的方法，只有当这个方法返回值为 true 的时候，后面的方法才会执行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1:preHandle");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1:postHandle");

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1:afterCompletion");

    }
}
@Component
public class MyInterceptor2 implements HandlerInterceptor {
    /**
     * 这个是请求预处理的方法，只有当这个方法返回值为 true 的时候，后面的方法才会执行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor2:preHandle");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor2:postHandle");

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor2:afterCompletion");

    }
}


<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <ref bean="myInterceptor1"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <ref bean="myInterceptor2"/>
    </mvc:interceptor>
</mvc:interceptors>
````
如果存在多个拦截器，拦截规则如下：
- preHandle 按拦截器定义顺序调用
- postHandler 按拦截器定义逆序调用
- afterCompletion 按拦截器定义逆序调用
- postHandler 在拦截器链内所有拦截器返成功调用
- afterCompletion 只有 preHandle 返回 true 才调用

# restful简介
URI 是名词，不能包含动词。  
客户端用到的手段，只能是 HTTP 协议。具体来说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：
- GET 用来获取资源
- POST 用来新建资源（也可以用于更新资源）
- PUT 用来更新资源
- DELETE 用来删除资源

综合上面的解释，我们总结一下什么是 RESTful 架构：
- 每一个 URI 代表一种资源
- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过四个 HTTP 动词，对服务器端资源进行操作，实现"表现层状态转化"。

# JSON工具
目前主流的json工具主要有三种：
- jackson
- gson
- fastjson

在 SpringMVC 中，对 jackson 和 gson 都提供了相应的支持，就是如果使用这两个作为 JSON 转换器，只需要添加对应的依赖就可以了，返回的对象和返回的集合、Map 等都会自动转为 JSON，但是，如果使用 fastjson，除了添加相应的依赖之外，还需要自己手动配置 HttpMessageConverter 转换器。其实前两个也是使用 HttpMessageConverter 转换器，但是是 SpringMVC 自动提供的，SpringMVC 没有给 fastjson 提供相应的转换器。

- jackson的使用：  
添加依赖即可，添加依赖后，接口中返回的对象，集合等，都会转换为json字符串。
```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.1</version>
</dependency>
```
如果想要定制返回日期的格式，可以通过加注解的方式：
```
public class Book {
    private Integer id;
    private String name;
    private String author;
    @JsonFormat(pattern = "yyyy-MM-dd",timezone = "Asia/Shanghai")
    private Date publish;
    }
```
也可以通过全局配置的方式：
```
<mvc:annotation-driven>
    <mvc:message-converters>
        <ref bean="httpMessageConverter"/>
    </mvc:message-converters>
</mvc:annotation-driven>
<bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter" id="httpMessageConverter">
    <property name="objectMapper">
        <bean class="com.fasterxml.jackson.databind.ObjectMapper">
            <property name="dateFormat">
                <bean class="java.text.SimpleDateFormat">
                    <constructor-arg name="pattern" value="yyyy-MM-dd HH:mm:ss"/>
                </bean>
            </property>
            <property name="timeZone" value="Asia/Shanghai"/>
        </bean>
    </property>
</bean>
```
- fastjson的使用：
```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.60</version>
</dependency>


<mvc:annotation-driven>
    <mvc:message-converters>
        <ref bean="httpMessageConverter"/>
    </mvc:message-converters>
</mvc:annotation-driven>
<bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter" id="httpMessageConverter">
    <property name="fastJsonConfig">
        <bean class="com.alibaba.fastjson.support.config.FastJsonConfig">
            <property name="dateFormat" value="yyyy-MM-dd"/>
        </bean>
    </property>
    //用来解决中文乱码的问题
     <property name="supportedMediaTypes">
        <list>
            <value>application/json;charset=utf-8</value>
        </list>
    </property>
</bean>
```
在接收参数时将json字符串转换为对象：  
通过@ResponseBody注解。
```
@RequestMapping("/addbook3")
@ResponseBody
public void addBook3(@RequestBody Book book) {
    System.out.println(book);
}
```
