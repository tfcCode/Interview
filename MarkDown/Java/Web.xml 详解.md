# Web.xml 详解

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">  
    <filter>  
        <filter-name>characterEncodingFilter</filter-name>  
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
        <init-param>  
            <param-name>encoding</param-name>  
            <param-value>UTF-8</param-value>  
        </init-param>  
        <init-param>  
            <param-name>forceEncoding</param-name>  
            <param-value>true</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  
        <filter-name>characterEncodingFilter</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
  	<!--启用spring-->
    <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
    <!--2、部署applicationContext的xml文件--> 
    <context-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:spring/applicationContext.xml</param-value>  
    </context-param>  
    <!--3、启用springmvc--> 
    <servlet>  
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
        
        <init-param>  
            <param-name>contextConfigLocation</param-name>  
            <param-value>classpath:spring/dispatcher-servlet.xml</param-value>  
        </init-param>  
        <load-on-startup>1</load-on-startup><!--是启动顺序，让这个Servlet随Servletp容器一起启动。-->  
    </servlet>  
    <servlet-mapping>  
        <servlet-name>DispatcherServlet</servlet-name>  
        <url-pattern>/</url-pattern> <!--会拦截URL中带“/”的请求。-->  
    </servlet-mapping>  

    <welcome-file-list><!--指定欢迎页面-->  
        <welcome-file>login.html</welcome-file>  
    </welcome-file-list>  
    <error-page> <!--当系统出现404错误，跳转到页面nopage.html-->  
        <error-code>404</error-code>  
        <location>/nopage.html</location>  
    </error-page>  
    <error-page> <!--当系统出现java.lang.NullPointerException，跳转到页面error.html-->  
        <exception-type>java.lang.NullPointerException</exception-type>  
        <location>/error.html</location>  
    </error-page>  
    <session-config><!--会话超时配置，单位分钟-->  
        <session-timeout>360</session-timeout>  
    </session-config>  
</web-app>
```



## 1、welcome-file-list

* 指定首页，可以后多个值，从上到下依次扫描，哪个符合用哪个

```xml
<welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>index1.jsp</welcome-file>
    <welcome-file>index2.jsp</welcome-file>
</welcome-file-list>
```



## 2、context-param

* 使用指定内容来初始化上下文，就是指定哪些 Bean 需要装配到容器中

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext.xml</param-value>
</context-param>
```



## 3、listener

* 为web应用程序定义监听器，监听器用来监听各种事件

```xml
<!--自动装载 ApplicationContext 的配置信息，配合<context-param>一起用-->
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```



## 4、filter

* 过滤器
    * 用来进行一些预处理，比如预处理文件的编码

```xml
<!--解决中文乱码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <!--提供参数-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!--过滤哪些请求-->
    <url-pattern>/*</url-pattern> 
</filter-mapping>
```



































































