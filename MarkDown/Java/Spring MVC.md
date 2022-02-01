# 一、HelloWorld

1、导包

```xml
<!--导入这个包会将其他的 spring 核心模块的包一同导进来-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>

<!--SpringMVC 的包-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>

<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

2、配置 `web.xml` 文件

* 主要就是配置一个前端控制器：`DispatcherServlet` 

```xml
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <!--contextConfigLocation：SpringMVC 配置文件位置-->
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--服务器已启动就创建该 Servlet，值越小，优先级越高，越优先创建对象-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

3、编写 `SpringMVC` 配置文件

```xml
<!--配置包扫描路径-->
<context:component-scan base-package="com.tfc"/>

<!--视图解析器，用于解析返回的字符串-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/page/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

4、测试

```java
@Controller
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "success";
    }
}
```

## 细节

1）如果在配置前端控制器的时候没有指定配置文件路径，`SpringMVC` 会默认去`WEB-INF` 路径下找一个叫 `xxx-servlet.xml` 的文件，`xxx` 代表前端控制器的名字

2）配置 `/`、`/*` 的区别

`/`：拦截所有请求，但不拦截 `jsp` 页面，`*.jsp` 请求

`/*`：拦截所有请求，包括 `jsp` 页面和请求

解释：例如处理一个 `index.html` 页面

​	处理 `.jsp` 是 `Tomcat` 做的事，项目中的 `web.xml` 都继承于 `Tomcat` 服务器中的 `web.xml`，服务器中有一个 `DefaultServlet` 用来处理静态资源，并且拦截配置为 `url-pattern = /`，但项目中的配置的也是 `/`，这就覆盖了服务器中的配置，所以静态资源也会由我们自己配置的前端控制器进行处理， 而我们没有配置 `/xxx.html` 的 `Controller`，故访问报错

​	那为什么可以处理 `.jsp` 页面呢？因为服务器中有一个 `JspServlet` 用来处理 `jsp`，且 `url-pattern` 的值为 `*.jsp`，我们没有覆盖该配置，故可以访问 `jsp` 

> 静态资源

除了 `jsp` 都是静态资源，静态资源会放在服务器中，如果找到，直接返回给用户



## 静态资源处理

在 `SpringMVC` 配置文件中加上这两个标签即可

```xml
<mvc:default-servlet-handler/>
<mvc:annotation-driven/>
```



# 二、地址处理

## RequestMapping

​	既可以标注在类上，也可以标注在方法上，标注在类上（如`/aa`）表示该类下的所有方法的访问都从 `/aa` 开始，相当于一级路径，方法上的属于二级路径



### 各种属性

1、`value`：请求路径（默认属性）

```java
@RequestMapping("/hello")
```

2、`method`：限定请求方式

```java
@RequestMapping(value = "/hello", method = RequestMethod.POST)
```

3、`params`：规定请求参数

形式一

```java
@RequestMapping(value = "/hello", params = {"username"})

// http://localhost:8080/stringmvc/hello?username=zhangsan
```

表示请求路径必须**带**一个名为 `username` 的参数

形式二

```java
@RequestMapping(value = "/hello", params = {"!username"})
```

表示请求路径必须**没有带**一个名为 `username` 的参数

形式三

```java
@RequestMapping(value = "/hello", params = {"username=123","pwd=1111"})

// http://localhost:8080/stringmvc/hello?username=zhangsan
```

简单来说，该属性支持简单的表达式

4、`header`：规定请求头，和 `params` 属性用法一样，例如可以规定浏览器的，设置`User-Agent` 即可

5、`consumes`：只接受请求内容是哪种类型，用来规定请求头中的 `Content-Type` 

6、`produces`：告诉浏览器返回的内容是什么，给相应头中设置 `Content-Type` 



### 模糊匹配

**更精确优先** 

`?`：能代替任意一个字符，零个、多个都不行

```java
@RequestMapping("/hello?")
```

`*`：能任意替换多个字符，一层路径

```
@RequestMapping("/a/*/hello?")
```

`**`：能替代多层路径

```
@RequestMapping("/a/**/hello?")
```



## PathVariable

路径上可以有占位符`{变量名}`，只能作用于一层路径

```java
@RequestMapping("/hello1/{id}")
public String hello(@PathVariable("id") String name) {}
```



# 三、请求参数处理

## RequestParam

> 获取请求参数的值

```java
@RequestMapping("/handle")
public String updateBook(@RequestParam(value = "username",     // 参数的 key
                                       required = true,        // 是否必须带该参数
                                       defaultValue = "aaaa")  // 默认值
                         String name) {}

http://localhost:8080/SpringMVC/handle?username=bbbbb
// 相当于 request.getParamter("username")
```



## RequestBody

封装前端传过来的 JSON 格式数据，一般封装成一个对象

```java
public class User{
    private String name;
    private String addr;
    ......
}
```

```java
@PostMapping
public void addUser(@RequestBody User user){
    System.out.println(user);
}
```

测试数据

```json
{
    "name":"tfc",
    "addr":"北凉王府"
}
```







## RequestHeader

> 获取请求头中某个 `key` 的值，若请求头中没有值就会报错（可以设置一个默认值）

```java
@RequestMapping("/handle")
public String updateBook(@RequestHeader(value = "User-Agent",     // 参数的 key
                                        required = true,        // 是否必须带该参数
                                        defaultValue = "aaaa")  // 默认值
                         String name) {}
```



## RequestCookie

> 获取 `Cookie` 的值

用法和 `RequestHeader`、`RequestParam` 一样的

```java
@RequestMapping("/handle")
public String updateBook(@RequestCookie(value = "JID",     // 参数的 key
                                       required = true,        // 是否必须带该参数
                                       defaultValue = "aaaa")  // 默认值
                         String name) {}
```



## 封装 POJO

​	若请求参数过多，一个个写太麻烦，可以写一个实体类，实体类中的属性就是对应参数的 `key`，**注意，实体类一定要有一个无参构造器** 

还支持级联：如果实体类中有另一个 `POJO` 对象属性，也可以一并封装

```jsp
<form action="book" method="post">
    书名：<input type="text" name="bookName">
    描述：<input type="text" name="describe">
    <input type="submit" value="提交">
</form>
```

```java
public class Book {
    private String bookName;
    private String describe;

    public Book() {
    }
}
```

```java
@RequestMapping("/book")
public String updateBook(Book book) {
    .....
}
```



## 直接获取原生 API

`SpringMVC` 也支持直接获取原生 `API` 

```java
@RequestMapping("/book")
public String updateBoo(HttpSession session, HttpServletRequest request) {}
```





# 四、Rest 风格地址

​	传统请求地址命名方式多为 `/book/del/1`，带有明确的添加、删除等信息，等到项目逐渐复杂后，命名就成了一个难事，为了解决这个问题，出现了 `Rest` 风格的地址，一言蔽之：通过请求方式来明确请求的行为，是添加、删除还是其他

```java
@RequestMapping(value = "/book/{id}", method = RequestMethod.GET)
public String getBook(@PathVariable("id") Integer id) {
    System.out.println("查询" + id + "号图书");
    return "success";
}

@RequestMapping(value = "/book/{id}", method = RequestMethod.POST)
public String addBook(@PathVariable("id") Integer id) {
    System.out.println("添加" + id + "号图书");
    return "success";
}

@RequestMapping(value = "/book/{id}", method = RequestMethod.DELETE)
public String deleteBook(@PathVariable("id") Integer id) {
    System.out.println("删除" + id + "号图书");
    return "success";
}

@RequestMapping(value = "/book/{id}", method = RequestMethod.PUT)
public String updateBook(@PathVariable("id") Integer id) {
    System.out.println("更新" + id + "号图书");
    return "success";
}
```

但是页面请求只支持 `GET`、`POST` 两种方式，没有 `PUT`、`DELETE` 其他方式的请求，解决办法如下：

1、用一个 `form` 表单，并且请求方式为 `post` 方式，在表单中带一个 `_method` 参数，参数的值为你需要的请求方式

```jsp
<form action="book/4" method="post">
    <label>
        <input name="_method" value="put"/>
    </label>
    <input type="submit" value="更新图书"/>
</form>
```

但高版本的 `Tomcat` 不支持这些，需要添加一个属性：`isErrorPage="true"` 

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isErrorPage="true" %>
```

2、配置一个过滤器，用来过滤 `DELETE、PUT` 等请求

```xml
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>  <!--这里需要用 /*-->
</filter-mapping>
```



# 五、字符乱码处理

每次写 `SpringMVC` 就把字符乱码处理一起写了

每次装了 `Tomcat` 就把 `server.xml` 中的字符编码改成 `UTF-8`，就在配置端口的标签中添一个属性 `URIEncoding="UTF-8"` 

**注意：字符乱码过滤器一般在所有过滤器之前** 

```xml
<!--解决乱码-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--encoding：解决 Post 请求乱码-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!--foreceEncoding：解决响应乱码-->
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



# 六、数据输出

## 1、用 Map 传递数据

结果放在 `Request` 域中

```java
@RequestMapping("/handle01")
public String test(Map<String, Object> map) {
    map.put("msg", "徐凤年");
    return "success";
}

// 取出数据
request：${requestScope.msg}
```



## 2、用 Model、ModelMap 传递数据

结果放在 `Request` 域中

```java
@RequestMapping("/handle02")
public String test1(Model model, ModelMap modelMap) {
    model.addAttribute("msg", "徐凤年");
    modelMap.addAttribute("aaa", "徐骁");
    return "success";
}

// 取出数据
request：${requestScope.msg}
request：${requestScope.aaa}
```



## 3、Map、Model、ModelMap 联系

​	其实三者的最终实现类都是 `Spring` 提供的 `BindingAwareModelMap` 类在起作用，也就是说，`BindingAwareModelMap` 中的数据都会放在请求域中



## 4、ModelAndView

该对象可以作为返回值，既包含视图信息，也包含数据信息，且数据也是保存在 `Request` 域中

```java
@RequestMapping("/handle03")
public ModelAndView test2() {
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("success"); // 设置返回的视图名

    // 添加数据
    modelAndView.addObject("view", "ssssssss");
    return modelAndView;
}
```



## 5、@SessionAttributes

该注解用来给 `Session` 中存放数据，**且该注解只能标注在类上** 

```java
@SessionAttributes(value = {"msg", "a", "b"}, types = {String.class})
@Controller
public class TestController {}
```

`value`：只要是 `key = “msg”...` 的数据都往 `Sesssion` 中放一份

`types`：只要是 `String...` 类型的数据都往 `Sesssion` 中放一份

但是该注解不推荐使用，推荐使用原生 `API` 操作 `Sesssion` 



# 七、请求转发、重定向

## 请求转发

若目标页面不在配置的视图解析路径中，可以用相对路径来返回目标页面，也可以用**请求转发**操作

```java
@RequestMapping("/handle04")
public String test04() {
    return "../../target";
}
```

​	`forward`：表示转发到一个页面，也可以转发到某个请求，带 `forward` 前缀的会单独解析，不会进行拼串。**这里一定要加上 `/`，否则容易出问题** 

```java
@RequestMapping("/handle04")
public String test04() {
    return "forward:/target.jsp"; // 请求转发，这里一定要加上 '/'，否则容易出问题
}
```



## 重定向

​	`redirect`：表示重定向，可以重定向到一个页面，也可重定向到某个请求，这里直接写 `/` 即可，`SpringMVC` 会自动为我们拼接上项目名（原生`API`需要手动添加项目名）

```java
@RequestMapping("/handle04")
public String test04() {
    return "redirect:/target.jsp";
}
```



















































