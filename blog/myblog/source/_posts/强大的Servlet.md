---
title: 强大的Servlet
date: 2022-01-27 09:45:49
tags: Springboot
---
#### If I have seen further, it is by standing on the shoulders of giants
#### 如果我比别人看得更远，那是因为我站在巨人的肩膀上

### 如今回头看下Servlet不仅如此强大，还具有很强烈的参考意义，能在现如今流行的大部分框架中找到它的影子。下面文章不止与探索Servlet，可能在其中穿插其他的关联知识点，旨在能从此次的学习中获取更多的知识点~参考资料总结，转化为自己的理解输出,在文中我尽量以截图+复制全限定类名的方式记录，以便感兴趣的再次查找。

## Springboot与Servlet
在springboot中内嵌了Tomcat容器，而Tomcat又是Servlet的容器，Springboot就与Servlet产生了紧密的联系。
在分析各个类时，注意下每个类所在的包是如何在tomcat与boot之间跨越的~
## 生命周期
1、初始化
2、处理请求
3、销毁
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/servlet.png)
### 应用上下文
应用上下文即可看做：一次请求到达，到响应结束的过程中间的catlog，即阅读中结合上下文语境，是一个广义定义。
为什么说到上下文呢？来看下ServletContext的实现，第一个经典实现既是ApplicationContext我们不止在一次源码和应用中见到它，
```java
/**
 * Standard implementation of <code>ServletContext</code> that represents
 * a web application's execution environment.  An instance of this class is
 * associated with each instance of <code>StandardContext</code>.
 * 代表web应用程序的执行环境。这个类的一个实例是
 *与StandardContext的每个实例关联。
 * @author Craig R. McClanahan
 * @author Remy Maucherat
 */
public class ApplicationContext implements ServletContext {

    protected static final boolean STRICT_SERVLET_COMPLIANCE;///翻译为是否严格遵守

    protected static final boolean GET_RESOURCE_REQUIRE_SLASH;//我的蹩脚英语翻译为获取资源是否需要斜线。

    static {
        STRICT_SERVLET_COMPLIANCE = Globals.STRICT_SERVLET_COMPLIANCE;

        String requireSlash = System.getProperty("org.apache.catalina.core.ApplicationContext.GET_RESOURCE_REQUIRE_SLASH");
        if (requireSlash == null) {
            GET_RESOURCE_REQUIRE_SLASH = STRICT_SERVLET_COMPLIANCE;
        } else {
            GET_RESOURCE_REQUIRE_SLASH = Boolean.parseBoolean(requireSlash);
        }
    }

```
注：特别重要上述配置为tomcat中第一个开关配置，决定多个属性的值。来自于下面的Globals.STRICT_SERVLET_COMPLIANCE;默认为false
验证：
```java
     public static final boolean STRICT_SERVLET_COMPLIANCE =Boolean.parseBoolean(System.getProperty("org.apache.catalina.STRICT_SERVLET_COMPLIANCE", "false"));
 ```
和官网截图
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/compliance.png)
问题:会因为tomcat的版本配置不同改变此值，在8.5.57当中会改变为true，当cantroller中配置多个映射路径会出现访问不到的问题
此处参考博文：https://blog.csdn.net/xing930408/article/details/111225064
Tomcat文档：https://tomcat.apache.org/tomcat-8.5-doc/config/systemprops.html
而GET_RESOURCE_REQUIRE_SLASH直接赋值为STRICT_SERVLET_COMPLIANCE
 

### Servlet与HttpServlet
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/servlet%E5%85%B3%E7%B3%BB%E5%9B%BE.png)  
类图标明很是明显，在这个图中展示了servlet，tomcat，Springboot的关系，完美解释了那句Springboot是内嵌了tomcat的嵌入式引擎，嵌入式容器的说法~
```
而HttpServlet即是大部分请求的处理对象，嵌入式引擎----嵌入式容器----  webfilter ---weblistener
javax.servlet.ServletContext#addServlet(java.lang.String, java.lang.Class<? extends javax.servlet.Servlet>)返回一个ServletRegistration对象，可用于进一步
配置已注册的servlet
javax.servlet.ServletRegistration
```

```java
public interface ServletRegistration extends Registration {

    /**
     * TODO
     * @param urlPatterns The URL patterns that this Servlet should be mapped to
     * @return TODO
     * @throws IllegalArgumentException if urlPattern is null or empty
     * @throws IllegalStateException if the associated ServletContext has
     *                                  already been initialised
     */URL必须映射
    public Set<String> addMapping(String... urlPatterns);

    public Collection<String> getMappings();

    public String getRunAsRole();

    public static interface Dynamic
    extends ServletRegistration, Registration.Dynamic {
        public void setLoadOnStartup(int loadOnStartup);
        public Set<String> setServletSecurity(ServletSecurityElement constraint);
        public void setMultipartConfig(MultipartConfigElement multipartConfig);
        public void setRunAsRole(String roleName);
    }
}

```
思考：为什么Applacationcontext会有那么多重载方法？
```java
 @Override
    public ServletRegistration.Dynamic addServlet(String servletName, String className) {
        return addServlet(servletName, className, null, null);
    }


    @Override
    public ServletRegistration.Dynamic addServlet(String servletName, Servlet servlet) {
        return addServlet(servletName, null, servlet, null);
    }


    @Override
    public ServletRegistration.Dynamic addServlet(String servletName,
            Class<? extends Servlet> servletClass) {
        return addServlet(servletName, servletClass.getName(), null, null);
    }

```
--用在不同场景下解决同一类问题
而在HttpServlet中的关键方法service可看到平时请求接口的所有方法
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/servlet-service.png)
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/service-post.png)

```java

    private static final String METHOD_DELETE = "DELETE";
    private static final String METHOD_HEAD = "HEAD";
    private static final String METHOD_GET = "GET";
    private static final String METHOD_OPTIONS = "OPTIONS";
    private static final String METHOD_POST = "POST";
    private static final String METHOD_PUT = "PUT";
    private static final String METHOD_TRACE = "TRACE";

 protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        String method = req.getMethod();
        //GET
        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                if (ifModifiedSince < lastModified) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
            
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
            
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
            
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
            //There's no need to override this method. 没有必要~
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }


    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {

    String protocol = req.getProtocol();
    String msg = lStrings.getString("http.method_post_not_supported");
    if (protocol.endsWith("1.1")) {
        resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, msg);
    } else {
        resp.sendError(HttpServletResponse.SC_BAD_REQUEST, msg);
    }
}
```
注：在javaHttpServlet中，与Tomcat中的dopost方法如出一辙

真正的调用链(妥妥的责任链模式)是
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/HTTP%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B.jpg)
```
Tomcat与SpringMVC的结合点：ApplicationFilterChain与DispatcherServlet（继承于FrameworkServlet）；
（1）所有配置了路由信息的处理方法最终都是通过反射的方式进行调用的；
（2）在Java8中，反射方法调用最终落脚于NativeMethodAccessorImpl类的native方法：
private static native Object invoke0(Method var0, Object var1, Object[] var2);
在此处与JVM底层交互，实现跨代码衔接执行；
（3）观察到的比较重要的设计模式：职责链模式（ApplicationFilterChain）、委派模式（DelegatingFilterProxy）、
工厂模式、策略模式、代理模式（FilterChainProxy）、外观模式、适配器模式（HandlerAdapter）；
（4）Tomcat与SpringMVC的结合点：ApplicationFilterChain与DispatcherServlet（继承于FrameworkServlet）；
（5）在集成了Tomcat的SpringBoot项目中，先启动的不是Tomcat，而是Spring，Spring的工厂（默认DefaultListableBeanFactory）
读取注解完成各类Bean（WebApplicationContext、securityFilterChainRegistration、dispatcherServletRegistration、各类FilterInitializer与Filter）
的初始化，放入IoC容器，然后做路由Mapping，创建FilterChain，开启JMX等；
（6）Servlet、Filter是单实例多线程的，成员变量线程不安全，方法内局部变量线程安全；SingleThreadModel采用同步/实例池的方式来确保不会有两个线程同时执行servlet的service方法，但已被弃用，需自行确保成员变量线程安全；
————————————————
版权声明：本文为CSDN博主「wanxu12345678910」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/wanxu12345678910/article/details/83352371
```
### HttpServletResponse响应码
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/statusCode.png)

#### 监听器：   实现接口、标记
比如MQ，观察者模式，所有的时间监听都会继承  extend   java.util.EventListener接口，但里面什么都没有
，称之为mark接口，经典实现：ContextLoaderListener、RequestContextListener(重要)
```java
/**
 * A tagging interface that all event listener interfaces must extend.
 * @since 1.1
 */
public interface EventListener {
}

```
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/listener.png)
```java
/**
 * Bootstrap listener to start up and shut down Spring's root {@link WebApplicationContext}.
 * Simply delegates to {@link ContextLoader} as well as to {@link ContextCleanupListener}.
 *
 * <p>As of Spring 3.1, {@code ContextLoaderListener} supports injecting the root web
 * application context via the {@link #ContextLoaderListener(WebApplicationContext)}
 * constructor, allowing for programmatic configuration in Servlet 3.0+ environments.
 * See {@link org.springframework.web.WebApplicationInitializer} for usage examples.
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 17.02.2003
 * @see #setContextInitializers
 * @see org.springframework.web.WebApplicationInitializer
 */
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {


}
```
见名知意既然包含contextLoader必然跟上线文息息相关，在初始化容器时加载配置~
```java

@Override
    public void requestInitialized(ServletRequestEvent requestEvent) {
        if (!(requestEvent.getServletRequest() instanceof HttpServletRequest)) {
            throw new IllegalArgumentException(
                    "Request is not an HttpServletRequest: " + requestEvent.getServletRequest());
        }
        HttpServletRequest request = (HttpServletRequest) requestEvent.getServletRequest();
        ServletRequestAttributes attributes = new ServletRequestAttributes(request);
        request.setAttribute(REQUEST_ATTRIBUTES_ATTRIBUTE, attributes);
        //将请求对象放入ThreadLocal中
        LocaleContextHolder.setLocale(request.getLocale());
        RequestContextHolder.setRequestAttributes(attributes);
    }

```
<font color='red'> 重要：Servlet在同一个线程中，当初始化时放到对象里，当请求销毁时，自动将Threadlocal对象销毁，防止了内存泄漏的问题 </font>
当有请求到达时，会从线程池中取出一个线程来执行任务，执行完毕后再将线程回收至线程池,这样当前请求不可能拿到上一个请求保存在ThreadLocal对象里的值
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/Threadlocal.png)
```java

@Override
    public void requestDestroyed(ServletRequestEvent requestEvent) {
        ServletRequestAttributes attributes = null;
        Object reqAttr = requestEvent.getServletRequest().getAttribute(REQUEST_ATTRIBUTES_ATTRIBUTE);
        if (reqAttr instanceof ServletRequestAttributes) {
            attributes = (ServletRequestAttributes) reqAttr;
        }
        RequestAttributes threadAttributes = RequestContextHolder.getRequestAttributes();
        if (threadAttributes != null) {
            // We're assumably within the original request thread...
            LocaleContextHolder.resetLocaleContext();
            RequestContextHolder.resetRequestAttributes();
            if (attributes == null && threadAttributes instanceof ServletRequestAttributes) {
                attributes = (ServletRequestAttributes) threadAttributes;
            }
        }
        if (attributes != null) {
            attributes.requestCompleted();
        }
    }

```
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/ThreadLocalremove.png)
```java
/**
 * Bootstrap listener to start up and shut down Spring's root {@link WebApplicationContext}.
 * Simply delegates to {@link ContextLoader} as well as to {@link ContextCleanupListener}.
 *
 * <p>As of Spring 3.1, {@code ContextLoaderListener} supports injecting the root web
 * application context via the {@link #ContextLoaderListener(WebApplicationContext)}
 * constructor, allowing for programmatic configuration in Servlet 3.0+ environments.
 * See {@link org.springframework.web.WebApplicationInitializer} for usage examples.
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 17.02.2003
 * @see #setContextInitializers
 * @see org.springframework.web.WebApplicationInitializer
 */
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {

}
```

插播一句：在书写过程中发现某URL响应变慢，在分析SQL时，用到了in查询，执行分析计划用到了索引


## Servlet on Springboot

#### 组件声明注解：
servletContext---ApplicationContext
### 组件扫描：ServletComponentScan
熟悉波~ 是不是应用跟Springboot的@ComponentScan如出一辙
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(ServletComponentScanRegistrar.class)
public @interface ServletComponentScan {

    /**
     * Alias for the {@link #basePackages()} attribute. Allows for more concise annotation
     * declarations e.g.: {@code @ServletComponentScan("org.my.pkg")} instead of
     * {@code @ServletComponentScan(basePackages="org.my.pkg")}.
     * @return the base packages to scan
     */
    @AliasFor("basePackages")
    String[] value() default {};

    /**
     * Base packages to scan for annotated servlet components. {@link #value()} is an
     * alias for (and mutually exclusive with) this attribute.
     * <p>
     * Use {@link #basePackageClasses()} for a type-safe alternative to String-based
     * package names.
     * @return the base packages to scan
     */
    @AliasFor("value")
    String[] basePackages() default {};

    /**
     * Type-safe alternative to {@link #basePackages()} for specifying the packages to
     * scan for annotated servlet components. The package of each class specified will be
     * scanned.
     * @return classes from the base packages to scan
     */
    Class<?>[] basePackageClasses() default {};

}

```
配置声明：@interface  暴露SpringBean @Bean
事件：Event
#### filter：
webFilter
OncePerRequestFilter：只调用一次且是线程安全的
而其子类得ApplicationContextHeaderFilter调用的dofilter方法就是我们上面提到的真正在请求中执行的filter
### 激活Springbootweb
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/oncePerRequestFilter.png)

文无第一，武无第二，没有最好的技术框架或体系，只有最适合当下业务的框架或体系
谈谈你对技术的理解：天上飞的理念，必定有落地的实现

### 组装SpringApplicationBuilder
你看看这名字就知道他以后干啥的，并且它包含了太多太多的东西,
SpringApplication和ApplicationContext实例的生成器,基本包含了所有的SpringbootApplacation特性
```java
    private final SpringApplication application;

    private ConfigurableApplicationContext context;

    private SpringApplicationBuilder parent;
    AtomicBoolean是不是的看看
    private final AtomicBoolean running = new AtomicBoolean(false);

    private final Set<Class<?>> sources = new LinkedHashSet<>();

    private final Map<String, Object> defaultProperties = new LinkedHashMap<>();

    private ConfigurableEnvironment environment;

    private Set<String> additionalProfiles = new LinkedHashSet<>();

    private boolean registerShutdownHookApplied;

    private boolean configuredAsChild = false;

```

```Java
@SpringBootApplication
public class TestProfiles {
 
    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(TestProfiles.class)
                .properties("spring.config.location=classpath:/test-profiles.yml")
                .properties("spring.profiles.active=oracle")
                .run(args);
        // 输出变量
        System.out.println(context.getEnvironment().getProperty("jdbc.driver"));
 
        // 启动第二个Spring容器，指定端口为8848
        ConfigurableApplicationContext context2 = new SpringApplicationBuilder(TestProfiles.class)
                .properties("spring.config.location=classpath:/test-profiles.yml")
                .properties("spring.profiles.active=mysql")
                .properties("server.port=8848")
                .run(args);
        // 输出变量
        System.out.println(context2.getEnvironment().getProperty("jdbc.driver"));
    }
}
```

Springboot自动装配
/META-INF/spring.factories
XXXAotuConfigration
NIO不是异步IO而是非阻塞IO
java9推崇模块化


ClassLoader
```java
public static void main(String[] args) {
    ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
    while (true) {

        System.out.println(contextClassLoader.getClass().getName());
        ClassLoader parent = contextClassLoader.getParent();
        if (parent == null) {
            break;
        }

    }

    ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
    System.out.println(systemClassLoader.getClass().getName());
}

```

----------
update_time =2022年2月14日13:48:19

## 传统的Servlet容器 Apache Tomcat 
这里只记录了部分重要场景
包含核心组件
静态资源处理
类加载
连接器
JDBC数据源

## HttpServletResponse
javax.servlet.http.HttpServletResponse
```java
public static void main(String[] args) {
    ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
    while (true) {

        System.out.println(contextClassLoader.getClass().getName());
        ClassLoader parent = contextClassLoader.getParent();
        if (parent == null) {
            break;
        }

    }

    ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
    System.out.println(systemClassLoader.getClass().getName());
}
```
除了其中的状态码之外，结合最近的测试其中的实现可具体参考
addHeader方法，getHeader方法等等
BootStrap--system---common---webapp

## 静态资源处理类org.apache.catalina.servlets.DefaultServlet
注意下包名

大多数情况下我们关注的更多是server.xml中Tomcat的配置，而在web.xml中除了路径映射等配置外
```java
 <!-- The mapping for the default servlet -->
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- The mappings for the JSP servlet -->
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>
```
关于是否是开发模式
```java
 <!--   development         Is Jasper used in development mode? If true,   -->
  <!--                       the frequency at which JSPs are checked for    -->
  <!--                       modification may be specified via the          -->
  <!--                       modificationTestInterval parameter. [true]     -->
```
由于DefaultServlet是HttpServlet的子类，所以在此不展开讨论
而在server.xml中标签与后台接口是一一绑定的
```java
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```
而在JDBC中的大多数类中也遵循此规则，那么就上面这段分析标签Connector则对应
org.apache.catalina.connector.Connector，验证一下标签中对应protocol,connectionTimeout,redirectPort
其中标签对应部分
```java
 /**
     * Defaults to using HTTP/1.1 NIO implementation.
     */
    public Connector() {
        this("org.apache.coyote.http11.Http11NioProtocol");
    }
```
而在tomcat8.0+中getProtocol对应protocol
redirectPort对应属性默认值
```java
/**
     * The redirect port for non-SSL to SSL redirects.
     */
    protected int redirectPort = 443;
```
关于标签中connector中这个Http11NioProtocol则在tomcat官方文档中可见其中一句话
```
When using HTTP connectors (based on APR or NIO/NIO2), Tomcat supports using sendfile to send large static files. These writes, as soon as the system load increases, will be performed asynchronously in the most efficient way. Instead of sending a large response using blocking writes, it is possible to write content to a static file, and write it using a sendfile code. A caching valve could take advantage of this to cache the response data in a file rather than store it in memory. Sendfile support is available if the request attribute org.apache.tomcat.sendfile.support is set to Boolean.TRUE
```
也可在server.xml中搜索
```
<!-- Define an AJP 1.3 Connector on port 8009 -->
    <!--
    <Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />
    -->
```
server.port在文件中的位置
<!-- {
      "name": "server.port",
      "type": "java.lang.Integer",
      "description": "Server HTTP port.",
      "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties",
      "defaultValue": 8080
    }, -->
## 重点来了 ServerProperties包含了tomcat,Jetty,Undertow,而在Springboot2.2.6中则存在Netty
那么理所当然，在tomcat中的一些配置也存在于此
```java
/**
 * Maximum amount of worker threads.
 */
private int maxThreads = 200;

/**
 * Minimum amount of worker threads.
 */
private int minSpareThreads = 10;
```
## 那么为什么Tomcat被称之为嵌入式容器呢？
在启动时无需自启动容器，在Bootstrap中调用tomcat，另外tomcat中TomcatEmbeddedContext，Embedded即直译为嵌入式
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/embedded.png)
这里记忆有些混乱了，有点找不过来哪里是入口了，但先从TomcatServletWebServerFactoryCustomizer的customize()方法调用找，
```java
private Stream<Wrapper> getLoadOnStartupWrappers(Container[] children) {
        Map<Integer, List<Wrapper>> grouped = new TreeMap<>();
        for (Container child : children) {
            Wrapper wrapper = (Wrapper) child;
            int order = wrapper.getLoadOnStartup();
            if (order >= 0) {
                grouped.computeIfAbsent(order, ArrayList::new);
                grouped.get(order).add(wrapper);
            }
        }
        return grouped.values().stream().flatMap(List::stream);
    }
```
因为要看下Netty，所以还是重新看下server.properties
我将Spring Boot AutoConfigure升级到了2.6.2，内置的Tomcat就升级到9.0了
为了方便查看才升级的，之前的2.1.x就不截图了
server.properties的位置在configuration的下面的json文件
spring-configuration-metadata.json
```
{
      "name": "server",
      "type": "org.springframework.boot.autoconfigure.web.ServerProperties",
      "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
}
```
那既然为了看Netty在这个json文件中同样存在
```
{
      "name": "server.netty",
      "type": "org.springframework.boot.autoconfigure.web.ServerProperties$Netty",
      "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties",
      "sourceMethod": "getNetty()"
}
```
既然在json中存在getNetty等方法，猜测那么对应的ServerProperties也存在对应的方法，
因为存在实例么，tomcat搭配server.xml和web.xml简单看了一下，当然Tomcat还是主要和Servlet的关联关系更为重要，
本身tomcat知识点也最够庞大的，包含类加载器，双拼委派，打破双亲委派、jvm调优等等，可以顺带看下这里的专题
## 当一次请求发起都发生了什么？

用户通过浏览器进行了一个操作，这个操作可以是输入url地址并回车，或者是点击超链接，或者是在搜索框中输入关键字进行搜索，接着浏览器就捕获到了这个事件
由于 HTTP 协议底层具体的数据传输使用的是 TCP/IP 协议，因此浏览器需要向服务端发出 TCP 连接请求
服务器接受浏览器的连接请求，并经过 TCP 三次握手建立连接
浏览器将请求数据打包成一个 HTTP 协议格式的数据包
浏览器将打包好的数据包推入网络，经过网络传输最终到达服务器指定程序
服务端程序拿到数据包后，根据 HTTP 协议格式进行解包，获取到客户端的意图
得知客户端意图后进行处理，比如提供静态文件或者调用服务端程序获得动态结果
服务器将响应结果按照 HTTP 协议格式打包
服务器将响应数据包推入网络，数据包经过网络传输最终达到到浏览器
浏览器拿到数据包后，按照 HTTP 协议的格式解包，然后对数据进行解析
浏览器将解析后的静态数据（如html、图片）展示给用户

Tomcat 作为一个 HTTP 服务器，主要需要完成的功能是接受连接、解析请求数据、处理请求和发送响应这几个步骤。
作者：若兮缘
链接：https://www.jianshu.com/p/7c9401b85704
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
关于tomcat的架构就取自这篇文章，图文都很喜欢~

导入过程Running With JRE 7 Or Later

启动tomcat所需环境
```java
 else
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$CATALINA_LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
      -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
      -classpath "\"$CLASSPATH\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
```
后续不在赘述。重点在Server.properties中版本区别是否包含Netty的这个类,
本来我是想跟着dei一下bug的,实际是我没起来，版本又不兼容，中间穿插了需求，就不dei了skr~

## 一方库、二方库、三方库说明
有些二方库为apache所需类库,当然定义也尽相同，以统一标准为准吧~就像嵌入式这个单词,
如果学习的时候根据服务的命名，猜测其作用，然后再去证实的话，可能我早就认识这个单词了
> 一方库：本工程中的各模块的相互依赖
> 二方库：公司内部的依赖库，一般指公司内部的其他项目发布的jar包
> 三方库：公司之外的开源库， 比如apache、ibm、google等发布的依赖
为什么写这句话就是因为javax是指扩展我的java,因为原生的二方库是不允许被覆盖的。提到的
```java

private Stream<Wrapper> getLoadOnStartupWrappers(Container[] children) {
        Map<Integer, List<Wrapper>> grouped = new TreeMap<>();
        for (Container child : children) {
            Wrapper wrapper = (Wrapper) child;
            int order = wrapper.getLoadOnStartup();
            if (order >= 0) {
                grouped.computeIfAbsent(order, ArrayList::new);
                grouped.get(order).add(wrapper);
            }
        }
        return grouped.values().stream().flatMap(List::stream);
    }

```
再比如这里面的grouped.computeIfAbsent(order, ArrayList::new);其中Absent译为缺席，入参是key=order,以及函数方法，在key!=null的情况下赋值为newAraayList并返回去。
and this flatMap VS map,其他人举的例子很明朗，我就不摘抄了,https://www.cnblogs.com/yucy/p/10260014.html
## JDBC中的servlet
>数据库三大范式：
1．第一范式(确保每列保持原子性)
2．第二范式(确保表中的每列都和主键相关)
3．第三范式(确保每列都和主键列直接相关,而不是间接相关)
1、DML:Data Manipulation Language  操作语句
2、DDL：data define Language、
3、存储过程执行后
4、查询中也是有事务的：select查询后结果集关闭后
事务并发可能的影响：
1、脏读（读取未提交数据）
A事务读取B事务尚未提交的数据，此时如果B事务发生错误并执行回滚操作，那么A事务读取到的数据就是脏数据。
就好像原本的数据比较干净、纯粹，此时由于B事务更改了它，这个数据变得不再纯粹。这个时候A事务立即读取了这个脏数据，
但事务B良心发现，又用回滚把数据恢复成原来干净、纯粹的样子，而事务A却什么都不知道，最终结果就是事务A读取了此次的脏数据，称为脏读。
2、不可重复读（前后多次读取，数据内容不一致）
事务A在执行读取操作，由整个事务A比较大，前后读取同一条数据需要经历很长的时间 。而在事务A第一次读取数据，
比如此时读取了小明的年龄为20岁，事务B执行更改操作，将小明的年龄更改为30岁，此时事务A第二次读取到小明的年龄时，
发现其年龄是30岁，和之前的数据不一样了，也就是数据不重复了，系统不可以读取到重复的数据，成为不可重复读
3、幻读（前后多次读取，数据总量不一致）
事务A在执行读取操作，需要两次统计数据的总量，前一次查询数据总量后，此时事务B执行了新增数据的操作并提交后，
这个时候事务A读取的数据总量和之前统计的不一样，就像产生了幻觉一样，平白无故的多了几条数据，成为幻读
幻读产生的根本原因是采用的行级锁，所以只针对脏读和重复读有用
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/drivermanager.png)
Drivermanager-->getconnection--->connection-->createStatement-->ResultSet executeQuery(String sql) throws SQLException;
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/connection.png)
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/drivermanager.png)
重载connection方法可实现在各个数据库中切换，基本不需要太多的代码，JDBC中用到的设计模式？----桥接模式
不知道为啥都在强调jdbc的设计模式，所以引用下《重学设计模式--小博哥》中的案例分析
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F.png)
![场景](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E6%A1%88%E4%BE%8B%E5%9C%BA%E6%99%AF.png)
代码实现登陆：
```java
    public class PayController {
        private Logger logger = LoggerFactory.getLogger(PayController.class);

        public boolean doPay(String uId, String tradeId, BigDecimal amount,
                             int channelType, int modeType) {
            // 微信⽀付
            if (1 == channelType) {
                logger.info("模拟微信渠道⽀付划账开始。uId：{} tradeId：{} amount：
                {
                }
                ", uId, tradeId, amount);
                if (1 == modeType) {
                    logger.info("密码⽀付，⻛控校验环境安全");
                } else if (2 == modeType) {
                    logger.info("⼈脸⽀付，⻛控校验脸部识别");
                } else if (3 == modeType) {
                    logger.info("指纹⽀付，⻛控校验指纹信息");
                    123456789
                    10
                    11
                    12
                    13
                    14
                    上⾯的类提供了⼀个⽀付服务功能，通过提供的必要字段； ⽤户ID 、交易ID 、 ⾦额 、渠道 、模 式 ，来控制⽀付⽅式。
                    以上的 ifelse 应该是最差的⼀种写法，即使写 ifelse 也是可以优化的⽅式去写的。
                    3. 测试验证
                    3.1 编写测试类
                    以上分别测试了两种不同的⽀付类型和⽀付模式；微信⼈脸⽀付、⽀付宝指纹⽀付
                    3.2 测试结果
                }
            }
            // ⽀付宝⽀付
            else if (2 == channelType) {
                logger.info("模拟⽀付宝渠道⽀付划账开始。uId：{} tradeId：{}
                        amount： {
                }
                ", uId, tradeId, amount);
                if (1 == modeType) {
                    logger.info("密码⽀付，⻛控校验环境安全");
                } else if (2 == modeType) {
                    logger.info("⼈脸⽀付，⻛控校验脸部识别");
                } else if (3 == modeType) {
                    logger.info("指纹⽀付，⻛控校验指纹信息");
                }
            }
            return true;
        }
    }

```
其实面对这种情况一般我是看到大多数是应用策略+模板的，桥接真的很少听
```java
public abstract class Pay {
 protected Logger logger = LoggerFactory.getLogger(Pay.class);
 protected IPayMode payMode;
 public Pay(IPayMode payMode) {
 this.payMode = payMode;
 }
 public abstract String transfer(String uId, String tradeId, BigDecimal
amount);
}
```
在这个类中定义了⽀付⽅式的需要实现的划账接⼝： transfer ，以及桥接接⼝； IPayMode ，并
在构造函数中⽤户⽅⾃⾏选择⽀付⽅式。
如果没有接触过此类实现，可以᯿点关注 IPayMode payMode ，这部分是桥接的核⼼
```java
public class WxPay extends Pay {
 public WxPay(IPayMode payMode) {
 super(payMode);
 }
 public String transfer(String uId, String tradeId, BigDecimal amount) {
 logger.info("模拟微信渠道⽀付划账开始。uId：{} tradeId：{} amount：{}",
uId, tradeId, amount);
 boolean security = payMode.security(uId);
 logger.info("模拟微信渠道⽀付⻛控校验。uId：{} tradeId：{} security：
{}", uId, tradeId, security);
 if (!security) {
 logger.info("模拟微信渠道⽀付划账拦截。uId：{} tradeId：{} amount：
{}", uId, tradeId, amount);
 return "0001";
 }
 logger.info("模拟微信渠道⽀付划账成功。uId：{} tradeId：{} amount：{}",
uId, tradeId, amount);
 return "0000";
 }
} 
```
支付宝支付
```Go
public class ZfbPay extends Pay {
 public ZfbPay(IPayMode payMode) {
 super(payMode);
 }
 public String transfer(String uId, String tradeId, BigDecimal amount) {
 logger.info("模拟⽀付宝渠道⽀付划账开始。uId：{} tradeId：{} amount：
{}", uId, tradeId, amount);
 boolean security = payMode.security(uId);
 logger.info("模拟⽀付宝渠道⽀付⻛控校验。uId：{} tradeId：{} security：
{}", uId, tradeId, security);
 if (!security) {
 logger.info("模拟⽀付宝渠道⽀付划账拦截。uId：{} tradeId：{}
amount：{}", uId, tradeId, amount);
 return "0001";
 }
 logger.info("模拟⽀付宝渠道⽀付划账成功。uId：{} tradeId：{} amount：
{}", uId, tradeId, amount);
 return "0000";
 }
}
```
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%9E%8B.png)
桥接模式接口
```Go

public interface IPayMode {
 boolean security(String uId);
}
```
刷脸
```java
public class PayFaceMode implements IPayMode{
 protected Logger logger = LoggerFactory.getLogger(PayCypher.class);
 public boolean security(String uId) {
 logger.info("⼈脸⽀付，⻛控校验脸部识别");
 return true;
 }
}
其他同上
```
测试类编写
```java
@Test
public void test_pay() {
 System.out.println("\r\n模拟测试场景；微信⽀付、⼈脸⽅式。");
 Pay wxPay = new WxPay(new PayFaceMode());
 wxPay.transfer("weixin_1092033111", "100000109893", new
BigDecimal(100));
 System.out.println("\r\n模拟测试场景；⽀付宝⽀付、指纹⽅式。");
 Pay zfbPay = new ZfbPay(new PayFingerprintMode());
 zfbPay.transfer("jlu19dlxo111","100000109894",new BigDecimal(100));
} 
```
## ResultSet executeQuery(String sql) throws SQLException;
```java
<dependency>
  <groupId>commons-dbcp</groupId>
  <artifactId>commons-dbcp</artifactId>
</dependency>
org.springframework.transaction.interceptor.TransactionInterceptor
@Nullable
public Object invoke(MethodInvocation invocation) throws Throwable {
    Class<?> targetClass = invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null;
    Method var10001 = invocation.getMethod();
    invocation.getClass();
    return this.invokeWithinTransaction(var10001, targetClass, invocation::proceed);
}
org.springframework.transaction.TransactionDefinition
```
其实这里看不出来跟servlet的关联性有多么高，如果实在要说其中的关联性，
还不如将jdbc的整合过程与Mybatis进行比较,或者分析jdbc代码分析封装硬编码的过程，
就连其报下的大部分类名都不与之相关,当然你要说再Servlet与jdbc集成开发的时代，他也是有一定时代和代表性的。
```java
    @CallerSensitive 
    public static Connection getConnection(String url,
        java.util.Properties info) throws SQLException {

        return (getConnection(url, info, Reflection.getCallerClass()));
    }
```
### CallerSensitive是什么鬼？
CallerSensitive老规矩，猜测下Caller=调用，Sensitive=敏感的，那么标识在方法上则是当调用方法时的一些控制。
其中特指Reflection.getCallerClass()能够追踪到调用者的第一人。项目中用是用不到。
### 学习方法就是学习大佬的学习方法
这里JDBC就先到此为止，我先不得不先记录下我在javacache中遇到的小问题思考。
## 为什么在ConcurrentHashMap还要加入synchronized
在org.springframework.cache.support.AbstractCacheManager中有一段关于初始化缓存静态配置的代码。
```java
/**
     * Initialize the static configuration of caches.
     * <p>Triggered on startup through {@link #afterPropertiesSet()};
     * can also be called to re-initialize at runtime.
     * @since 4.2.2
     * @see #loadCaches()
     */
public void initializeCaches() {
        Collection<? extends Cache> caches = loadCaches();

        synchronized (this.cacheMap) {
            this.cacheNames = Collections.emptySet();
            this.cacheMap.clear();
            Set<String> cacheNames = new LinkedHashSet<>(caches.size());
            for (Cache cache : caches) {
                String name = cache.getName();
                this.cacheMap.put(name, decorateCache(cache));
                cacheNames.add(name);
            }
            this.cacheNames = Collections.unmodifiableSet(cacheNames);
        }
    }
```
注：Triggered：触发，边学习还是边记单词
原因：本身put和add的线程安全是由ConcurrentHashMap保证的，但是此时获取的值ConcurrentHashMap并不能保证其他线程对共享变量的值操作时还是原来的值。
怎么说呢，这么看来可能失去了map的本来特性，但其实还是不理解，是不理解这个原因准不准确。

谁提出谁解决:concurrentHashMap只能保证一次操作的原子性，一系列操作的时候就需要加锁了，不能保证第N+1个线程进来的时候获取到的状态是未clear的

Collections.emptySet()：如果你想 new 一个空的 List ，而这个 List 以后也不会再添加元素，那么就用 Collections.emptyList() 好了。
new ArrayList() 或者 new LinkedList() 在创建的时候有会有初始大小，多少会占用一内存。
每次使用都new 一个空的list集合，浪费就积少成多，浪费就严重啦，就不好啦。

还有一个
```java
@Override
    @Nullable
    public Cache getCache(String name) {
        Cache cache = this.cacheMap.get(name);
        if (cache != null) {
            return cache;
        }
        else {
            // Fully synchronize now for missing cache creation...
            synchronized (this.cacheMap) {
                cache = this.cacheMap.get(name);
                if (cache == null) {
                    cache = getMissingCache(name);
                    if (cache != null) {
                        cache = decorateCache(cache);
                        this.cacheMap.put(name, cache);
                        updateCacheNames(name);
                    }
                }
                return cache;
            }
        }
    }

其中的getMissingCache方法
@Nullable
    protected Cache getMissingCache(String name) {
        return null;
    }
无论如何都要返回null，那么他还要进行判空意义又何在？

源码注释是这样写的
Return a missing cache with the specified name, or null if such a cache does not exist or could not be created on demand.
Caches may be lazily created at runtime if the native provider supports it. If a lookup by name does not yield any result, an AbstractCacheManager subclass gets a chance to register such a cache at runtime. The returned cache will be automatically added to this cache manager.
返回指定名称的缺失缓存，如果此类缓存不存在或无法按需创建，则返回null。
如果本机提供程序支持，可以在运行时延迟创建缓存,就是其扩展实际是在子类中来复写的，
注意：在spring-data-redis的1.7.2中是没有复写此方法的
在官网中查询https://spring.io/projects/spring-data-redis#support
接入了
 <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
      <version>2.1.9.RELEASE</version>
</dependency>
低版本可是没有的呢，具体变化是在2.X前后的区别
protected RedisCache getMissingCache(String name) {
        return allowInFlightCacheCreation ? createRedisCache(name, defaultCacheConfig) : null;
}

```


## 第二次见到identityHashMap
```
https://mp.weixin.qq.com/s?__biz=Mzg2ODA3NjA1MA==&mid=2247484317&idx=1&sn=1a5d78d0e5d5e09b1d2ca969a5ae7d23&chksm=ceb09ce0f9c715f6688bf4b38a933730f61df7b432b3ac452924b502fba735e048289c3e0d91&token=950928768&lang=zh_CN#rd
```
```
public static void main(String[] args) {
    var var1 = new Integer(1);
    var var2 = new Integer(1);
    System.out.println(var1.equals(var2));

    System.out.println(var1.hashCode());
    System.out.println(var2.hashCode());


    System.out.println(System.identityHashCode(var1));
    System.out.println(System.identityHashCode(var2));
}

控制台输出
true
1
1
1524960486
117009527
```
org.springframework.cache.Cache对于缓存是应用接口，
Hashmap是否是在并发写的情况下，如果是则不是线程安全的
Consistency(一致性)
getinclude
想要看到源文档时，搜索：JSR107规范即可
推荐文章：https://www.jianshu.com/p/f6a1eae
### 接着来看缓存类CacheManager
从名字就能看出是管理缓存的类，CacheManager有两种，一种是Spring的，一种是javax的，就是上面所说的扩展类，但实现确实大体一致，
就接口实现入手，先从最简单的看起，从名字看就是SimpleCacheManager，提供最基本的set方法，load方法。
SimpleCacheManager在spring-context包下，5.1.4版本，rediscachemanager在spring-data-redis包下
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/fg15k701aa.png)
> CachingProvider：创建、配置、获取、管理和控制多个CacheManager
> CacheManager：创建、配置、获取、管理和控制多个唯一命名的Cache。（一个CacheManager仅被一个CachingProvider所拥有）
> Cache：一个类似Map的数据结构。（一个Cache仅被一个CacheManager所拥有）
> Entry：一个存储在Cache中的key-value对
> Expiry：每一个存储在Cache中的条目有一个定义的有效期，过期后不可访问、更新、删除。缓存有效期可以通过ExpiryPolicy设置
> https://cloud.tencent.com/developer/article/1497762
缓存么，除了快之外，还要满足有过期时间，但是除了在redis中并没有提供响应的方法，为什么呢？我觉得既然你启动或者加载就将bean放入cache管理了
就不可能伴随过期，应该会有响应的destroy方法在实例结束运行时清理，要不不可能实例还没运行玩就进行清理吧。
```java
@Override
    protected Collection<RedisCache> loadCaches() {

        List<RedisCache> caches = new LinkedList<>();

        for (Map.Entry<String, RedisCacheConfiguration> entry : initialCacheConfiguration.entrySet()) {
            caches.add(createRedisCache(entry.getKey(), entry.getValue()));
        }

        return caches;
    }
```
读取缓存的配置时间，一级缓存60s，二级缓存30s
在spring-autoconfigure-metadata.properties中的org.springframework.data.redis.cache.RedisCacheConfiguration配置此参数
```java
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration.AutoConfigureAfter=
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
```
启动时
```java
c.y.c.b.redis.config.RedisConfiguration  : [BSF][Redis]已启动,addressList:
2022-02-18 15:04:45.674  INFO 2604 --- [           main] c.y.c.b.e.c.EurekaClientConfiguration    : [BSF][Eureka-Client]已启动!!! eureka.client.serviceUrl.defaultZone=http://10.:8080/eureka/
2022-02-18 15:04:47.846  WARN 2604 --- [           main] c.n.c.sources.URLConfigurationSource     : No URLs will be polled as dynamic configuration sources.
2022-02-18 15:04:47.847  INFO 2604 --- [           main] c.n.c.sources.URLConfigurationSource     : To enable URLs as dynamic configuration sources, define System property archaius.configurationSource.additionalUrls or make config.properties available on classpath.
2022-02-18 15:04:47.943  INFO 2604 --- [           main] c.netflix.config.DynamicPropertyFactory  : DynamicPropertyFactory is initialized with configuration sources: com.netflix.config.ConcurrentCompositeConfiguration@4bf9f44b
2022-02-18 15:04:54.662  INFO 2604 --- [           main] c.y.c.b.s.ShardingJdbcConfiguration      : [BSF][Sharding-jdbc]已启动!!!
2022-02-18 15:04:57.232  INFO 2604 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
2022-02-18 15:04:58.661  INFO 2604 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-2} inited
2022-02-18 15:05:02.531 ERROR 2604 --- [           main] c.b.mybatisplus.MybatisConfiguration 

```

或者在redisconfig中配置
```java
@Configuration
public class RedisCacheConfig {

    @Bean
    public KeyGenerator simpleKeyGenerator() {
        return (o, method, objects) -> {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append(o.getClass().getSimpleName());
            stringBuilder.append(".");
            stringBuilder.append(method.getName());
            stringBuilder.append("[");
            for (Object obj : objects) {
                stringBuilder.append(obj.toString());
            }
            stringBuilder.append("]");

            return stringBuilder.toString();
        };
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        return new RedisCacheManager(
            RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory),
            this.getRedisCacheConfigurationWithTtl(600), // 默认策略，未配置的 key 会使用这个
            this.getRedisCacheConfigurationMap() // 指定 key 策略
        );
    }

    private Map<String, RedisCacheConfiguration> getRedisCacheConfigurationMap() {
        Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();
        redisCacheConfigurationMap.put("UserInfoList", this.getRedisCacheConfigurationWithTtl(3000));
        redisCacheConfigurationMap.put("UserInfoListAnother", this.getRedisCacheConfigurationWithTtl(18000));

        return redisCacheConfigurationMap;
    }

    private RedisCacheConfiguration getRedisCacheConfigurationWithTtl(Integer seconds) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        redisCacheConfiguration = redisCacheConfiguration.serializeValuesWith(
            RedisSerializationContext
                .SerializationPair
                .fromSerializer(jackson2JsonRedisSerializer)
        ).entryTtl(Duration.ofSeconds(seconds));

        return redisCacheConfiguration;
    }
}
过期时间
    @Cacheable(value = "UserInfoList", keyGenerator = "simpleKeyGenerator") // 3000秒
    @Cacheable(value = "UserInfoListAnother", keyGenerator = "simpleKeyGenerator") // 18000秒
    @Cacheable(value = "DefaultKeyTest", keyGenerator = "simpleKeyGenerator") // 600秒，未指定的key，使用默认策略
    
```
在spring2.0前后差异
构造器差异
before
```java
RedisCacheManager cacheManager = new RedisCacheManager(RedisTemplate redisTemplate);
```
after
```java
RedisCacheManager cacheManager = new RedisCacheManager(RedisCacheWriter redisCacheWriter,RedisCacheConfiguration redisCacheConfiguration);
```
创建RedisCacheWriter分为有锁和无锁
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/rediscachewriter.png)
```java
static RedisCacheWriter nonLockingRedisCacheWriter(RedisConnectionFactory connectionFactory) {

        Assert.notNull(connectionFactory, "ConnectionFactory must not be null!");

        return new DefaultRedisCacheWriter(connectionFactory);
    }
    JedisConnectionFactory
    /**
     * Create new {@link RedisCacheWriter} with locking behavior.
     *
     * @param connectionFactory must not be {@literal null}.
     * @return new instance of {@link DefaultRedisCacheWriter}.
     */
    static RedisCacheWriter lockingRedisCacheWriter(RedisConnectionFactory connectionFactory) {

        Assert.notNull(connectionFactory, "ConnectionFactory must not be null!");

        return new DefaultRedisCacheWriter(connectionFactory, Duration.ofMillis(50));
    }
```
即使是同一个缓存CacheManager管理的缓存实例，配置有可能不一样。
指定redis数据序列化
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/connectionfactory.png)
```java
.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(keySerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(valueSerializer()))
```
JAVA序列化方式
序列化方式一实现：Serializable接口
序列化方式二：Externalizable显式序列化
序列化方式三：实现Serializable接口+添加writeObject()和readObject()方法。(显+隐序列化)
对了，想要使用cache记得开启缓存注解，@EnableCaching
转过头来看下CacheOperation，这里面是缓存相关注解的父类，在SpringCacheAnnotationParser中管理了子类相关
```
static {
        CACHE_OPERATION_ANNOTATIONS.add(Cacheable.class);
        CACHE_OPERATION_ANNOTATIONS.add(CacheEvict.class);
        CACHE_OPERATION_ANNOTATIONS.add(CachePut.class);
        CACHE_OPERATION_ANNOTATIONS.add(Caching.class);
    }
```
解析注解的时候他们的入参，实现一模一样，只有返回值不一样，但是一模一样的代码写了三遍,为什么不判断类型动态返回呢~
```java

@Nullable
    private Collection<CacheOperation> parseCacheAnnotations(
            DefaultCacheConfig cachingConfig, AnnotatedElement ae, boolean localOnly) {

        Collection<? extends Annotation> anns = (localOnly ?
                AnnotatedElementUtils.getAllMergedAnnotations(ae, CACHE_OPERATION_ANNOTATIONS) :
                AnnotatedElementUtils.findAllMergedAnnotations(ae, CACHE_OPERATION_ANNOTATIONS));
        if (anns.isEmpty()) {
            return null;
        }

        final Collection<CacheOperation> ops = new ArrayList<>(1);
        anns.stream().filter(ann -> ann instanceof Cacheable).forEach(
                ann -> ops.add(parseCacheableAnnotation(ae, cachingConfig, (Cacheable) ann)));
        anns.stream().filter(ann -> ann instanceof CacheEvict).forEach(
                ann -> ops.add(parseEvictAnnotation(ae, cachingConfig, (CacheEvict) ann)));
        anns.stream().filter(ann -> ann instanceof CachePut).forEach(
                ann -> ops.add(parsePutAnnotation(ae, cachingConfig, (CachePut) ann)));
        anns.stream().filter(ann -> ann instanceof Caching).forEach(
                ann -> parseCachingAnnotation(ae, cachingConfig, (Caching) ann, ops));
        return ops;
    }

    private CacheableOperation parseCacheableAnnotation(
            AnnotatedElement ae, DefaultCacheConfig defaultConfig, Cacheable cacheable) {

        CacheableOperation.Builder builder = new CacheableOperation.Builder();

        builder.setName(ae.toString());
        builder.setCacheNames(cacheable.cacheNames());
        builder.setCondition(cacheable.condition());
        builder.setUnless(cacheable.unless());
        builder.setKey(cacheable.key());
        builder.setKeyGenerator(cacheable.keyGenerator());
        builder.setCacheManager(cacheable.cacheManager());
        builder.setCacheResolver(cacheable.cacheResolver());
        builder.setSync(cacheable.sync());

        defaultConfig.applyDefault(builder);
        CacheableOperation op = builder.build();
        validateCacheOperation(ae, op);

        return op;
    }

    private CacheEvictOperation parseEvictAnnotation(
            AnnotatedElement ae, DefaultCacheConfig defaultConfig, CacheEvict cacheEvict) {

        CacheEvictOperation.Builder builder = new CacheEvictOperation.Builder();

        builder.setName(ae.toString());
        builder.setCacheNames(cacheEvict.cacheNames());
        builder.setCondition(cacheEvict.condition());
        builder.setKey(cacheEvict.key());
        builder.setKeyGenerator(cacheEvict.keyGenerator());
        builder.setCacheManager(cacheEvict.cacheManager());
        builder.setCacheResolver(cacheEvict.cacheResolver());
        builder.setCacheWide(cacheEvict.allEntries());
        builder.setBeforeInvocation(cacheEvict.beforeInvocation());

        defaultConfig.applyDefault(builder);
        CacheEvictOperation op = builder.build();
        validateCacheOperation(ae, op);

        return op;
    }

    private CacheOperation parsePutAnnotation(
            AnnotatedElement ae, DefaultCacheConfig defaultConfig, CachePut cachePut) {

        CachePutOperation.Builder builder = new CachePutOperation.Builder();

        builder.setName(ae.toString());
        builder.setCacheNames(cachePut.cacheNames());
        builder.setCondition(cachePut.condition());
        builder.setUnless(cachePut.unless());
        builder.setKey(cachePut.key());
        builder.setKeyGenerator(cachePut.keyGenerator());
        builder.setCacheManager(cachePut.cacheManager());
        builder.setCacheResolver(cachePut.cacheResolver());

        defaultConfig.applyDefault(builder);
        CachePutOperation op = builder.build();
        validateCacheOperation(ae, op);

        return op;
    }

    private void parseCachingAnnotation(
            AnnotatedElement ae, DefaultCacheConfig defaultConfig, Caching caching, Collection<CacheOperation> ops) {

        Cacheable[] cacheables = caching.cacheable();
        for (Cacheable cacheable : cacheables) {
            ops.add(parseCacheableAnnotation(ae, defaultConfig, cacheable));
        }
        CacheEvict[] cacheEvicts = caching.evict();
        for (CacheEvict cacheEvict : cacheEvicts) {
            ops.add(parseEvictAnnotation(ae, defaultConfig, cacheEvict));
        }
        CachePut[] cachePuts = caching.put();
        for (CachePut cachePut : cachePuts) {
            ops.add(parsePutAnnotation(ae, defaultConfig, cachePut));
        }
    }


```
JSR(JCP  Java Community Process)文档查询地址，例如JSR107规范，servlet规范等等，但是都是英文文档，好处就是跟源码匹配，坏处咩咩咩咱这水平不太行
```
https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/web.html
```
以及各类中文版在线开发文档
```java
https://www.docs4dev.com/docs
```
对了本次的主题虽然是Servlet,但不全是Servlet,都是用servlet带出来的,比如现在
```java
 public DynamicThreadPoolExecutor(int corePoolSize,
                                     int maximumPoolSize,
                                     long keepAliveTime,
                                     TimeUnit unit,
                                     boolean waitForTasksToCompleteOnShutdown,
                                     long awaitTerminationMillis,
                                     @NonNull BlockingQueue<Runnable> workQueue,
                                     @NonNull String threadPoolId,
                                     @NonNull ThreadFactory threadFactory,
                                     @NonNull RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, waitForTasksToCompleteOnShutdown, awaitTerminationMillis, workQueue, threadPoolId, threadFactory, handler);
        this.threadPoolId = threadPoolId;

        RejectedExecutionHandler rejectedProxy = (RejectedExecutionHandler) Proxy
                .newProxyInstance(
                        handler.getClass().getClassLoader(),
                        new Class[]{RejectedExecutionHandler.class},
                        new RejectedProxyInvocationHandler(handler, rejectCount));
        setRejectedExecutionHandler(rejectedProxy);
    }
```
mybatis动态代理
```java
protected T newInstance(MapperProxy<T> mapperProxy) {
    //这里使用JDK动态代理，通过Proxy.newProxyInstance生成动态代理类
    // newProxyInstance的参数：类加载器、接口类、InvocationHandler接口实现类
    // 动态代理可以将所有接口的调用重定向到调用处理器InvocationHandler，调用它的invoke方法
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }
```
问题： 一个接口方法,返回值相同,方法相同,参数为Person,现在有子类PersonMan和PersonWoman,如何对接口进行适配？
```java
public class Person {

    private  String name;


    private  Integer age;

}
public class PersonMan extends  Person{

    private String character;
}

public class PersonWoman  extends Person{

    private String constellation;
}

```
在泛型类型中支持class等，以及省去参数转换的上界通配符<? extends E>：上界通配符，表明参数化类型可能是所指定的类型，或者此类型的子类；

```java

public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```
如下
```

public interface PersonService {

    public void  queryPerson(List<? extends Person> list);
    
}


@Service
public class PersonServiceImpl implements PersonService{


    @Override
    public void queryPerson(List<? extends Person> list) {
        System.out.println(JSONObject.toJSONString(list));
    }
}

List<Person> list = new ArrayList<>();


//List<PersonMan> list = new ArrayList<>();


//List<PersonWoman> list = new ArrayList<>();




personService.queryPerson(list);
```
那么如果参数类型为注解呢？
```java
@Override
    public void queryPerson(List<? extends Person> list,Class<? extends Annotation> t) {
        System.out.println(JSONObject.toJSONString(list));
       
        System.out.println(cast.annotationType());
    }
```
那么为什么上面的parsePutAnnotation不用呢？
NONONO,其实是用了的,只不过方法不同
```
private static final Set<Class<? extends Annotation>> CACHE_OPERATION_ANNOTATIONS = new LinkedHashSet<>(8);

    static {
        CACHE_OPERATION_ANNOTATIONS.add(Cacheable.class);
        CACHE_OPERATION_ANNOTATIONS.add(CacheEvict.class);
        CACHE_OPERATION_ANNOTATIONS.add(CachePut.class);
        CACHE_OPERATION_ANNOTATIONS.add(Caching.class);
    }
```
其实我就是看他的写的emoj,而且在caching这个注解里他包含了4种注解,然后统一进行管理的。

插播：Dbeaver中普通操作都会,安利一个类似idea的快捷键功能，Template--->SQL编辑器中可设置常用SQL,快捷键加Tab唤出.
