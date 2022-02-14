# Spring Boot 的配置

## 配置文件

#### 1、全局配置文件

* application.properties

* application.yml

    * 全称 YMAL（Aint‘s Markup Language）：是一种标记语言
    * 以数据为中心

* 配置示例

    * **application.properties**

    ```properties
    server.port=80801
    ```

    * **application.yml**

    ```yaml
    server:
      port: 80801  // 空格不能省略
    ```

    

**文件名是固定的**

#### 2、配置文件的作用

* 修改 Spring Boot 自动配置的默认值

## yaml 语法

#### 1、基本语法

```java
key:(空格)value   用一对键值对表示
```

* 以空格来表示层级关系

    * 只要是左对齐的一列书，都属于同一层级

    ```yaml
    server:
    	prot: 8081(空格不能省略)
    	path: /hello
    ```

    **属性和值是大小写敏感的**

#### 2、值的写法

##### 字面量：普通的值（数字，字符串，布尔）

* key: value :  字面直接来写
    * 字符串不用加单引号或双引号
        * 双引号 “ ”：不因转义特殊字符，是什么就是是什么
        * 单引号 ’ ‘：会转义特殊字符

##### 对象、Map（其实就是键值对）

* key: value ：在下一行写属性以及属性的值，注意缩进

* 假设有一个 Person 类，属性有个 name，age

    * ```yaml
        proson：
        	name: tfc
        	age: 21
        ```

* 行内写法

    * ```yaml
        proson: {name: tfc,age: 21}
        ```

##### 数组

* 用 **- 值** 表示数组中的一个元素

* ```yaml
    pets:
    	- cat
    	- dog
    	- pig
    ```

* 行内写法

```
pets: [cat,dog,pig]
```

## 获取配置文件中的值

* **导入配置文件处理器，配置时就会有提示**

```xml
<!--导入配置文件处理器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



```yaml
person:
  lastName: 徐凤年
  age: 23
  boss: true
  birth: 2020/3/30
  maps: {k1: 1,k2: 2}
  lists:
    - 徐晓
    - 陈芝豹
  dog:
    name: 柯基
    age: 2
```

* **使用 @ConfigurationProperties(prefix = "person") 注解** 

```java
/**
 * 将配置文件中的每一个属性的值，都映射到这个组件中
 * @ConfigurationProperties：告诉 Spring Boot 将本类中所有的属性和配置文件中的配置绑定
 *      prefix：对配置文件中哪个属性进行一一映射
 * 只有这个组件是容器中的组件的时候才能使用容器体统的 @ConfigurationProperties 功能（@Component）
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private int age;
    private boolean obss;
    private Date birth;

    private Map<String, Integer> maps;
    private List<Object> lists;

    private Dog dog;
```

* 使用 **@Value 注解** 

```java
public class Person {

    @Value("${person.last-name}")
    private String lastName;

    @Value("#{11*2}")  // SpEL 表达式
    private int age;

    @Value("true")
    private boolean boss;
```

* **@Value** 与 **@ConfigurationProperties** 比较

|                              | **@ConfigurationProperties** | **@Value** |
| :--------------------------: | :--------------------------: | :--------: |
|             功能             |   批量注入配置文件中的属性   | 一个个指定 |
|           松散绑定           |             支持             |   不支持   |
|             SpEL             |            不支持            |    支持    |
| JSR303数据校验（@Validatad） |             支持             |   不支持   |
| 复杂类型（对象，Map、List）  |             支持             |   不支持   |

* JSR303 数据校验示例

```java
@ConfigurationProperties(prefix = "person")
@Validated   // 开启校验注解支持
public class Person {

    @Email  // 必须是邮箱格式
    private String lastName;
```

* 使用情景

    * 如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用 @Value

    * 如果说，我们专门编写了一个 javaBean来和配置文件进行映射，我们就直接

        使用 **@Configuration Properties**

## 使用 .properties 文件编写配置

* 需要将文件的编码格式改为 UTF-8

```properties
person.last-name=徐凤年
person.age=23
person.birth=2020/3/30
person.boss=true
person.maps.k1=1
person.maps.k2=2
person.lists=a,b,c
person.dog.name=dog
person.dog.age=2
```

## 常用注解

#### @PropertySource

* 作用：**加载指定的配置文件**，@ConfigurationProperties 默认是从全局配置文件中获取内容
* 使用
    * 在类级别上使用 **@PropertySource(value = {"classpath:person.properties"})**

#### @ImportResource

* 作用：导入 Spring 的配置文件，让配置文件里的内容生效

    * Spring Boot 里面没有 Spring 的配置文件，自己编写的配置文件也不能生效

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean id="helloService" class="com.tfc.springbootquick.service.HelloService"/>
    
    </beans>
    ```

    

* **一般用法**：在一个配置类上使用该注解

```java
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBootQuickApplication {
```

* **==Spring Boot 推荐用法==**

    * 使用配置类（全注解）

    ```java
    @Configuration
    public class MyConfig {
        @Bean
        public HelloService getHelloService() {
            System.out.println("使用 @Bean 给容器添加组件");
            return new HelloService();
        }
    }
    ```

## 配置文件的占位符

* **RandomValuePropertySource：配置文件中使用随机数**

|                             种类                             |
| :----------------------------------------------------------: |
| ${random.value}、${random.int}、${random.int(10)}、${random.int[1024,2048]}、${random.long} |

```yaml
person:
  last-name: 徐凤年${random.uuid}
  age: ${random.int}
```



* 使用占位符获取之前配置过的值，如果没有，就使用指定默认值

```yaml
  dog:
    name: ${person.hello:hello}_柯基
```

## Profile

#### 1、多 Profile 文件

* 在主配置文件编写的时候，文件名可以是 **application-{profile}.properties/yml**
* 默认是用的是 **application.properties** 配置

#### 2、yml 支持多文档块方式

```yaml
server:
  port: 8081
spring:
  profiles:
    active: dev    //激活指定的文档块
---
server:
  port: 8082
spring:
  profiles: dev
---
server:
  port: 8083
spring:
  profiles: prod
```



#### 3、激活指定的 profile

1. 在默认配置文件（**application.properties**）中使用 **spring.profiles.active=dev** 
2. 命令行方式
    1. 在 IDEA 中添加一个命令行参数（Program arguments）  **--spring.profiles.active=dev**
    2. 如果已经打包
        * 使用命令 **java -jar  包全名  --spring.profiles.active=dev** 启动
3. 使用虚拟机参数（VM options）

* **-Dspring.profiles.active=dev**

## 配置文件加载位置

* spring boot 会**依次扫描**以下位置中的 application.properties 或 application.yml **作为默认配置文件** 
* **优先级由高到低，依次扫描**，内容相同的高优先级的覆盖低优先级，内容不同的项目补充
* -classpath 表示类路径下：**就是 resource 目录下** 

| 类型                     | 位置                                                        |
| :----------------------- | :---------------------------------------------------------- |
| -file：./**config**/     | 项目名/**config**/application.properties                    |
| -file：./                | 项目名/application.properties                               |
| -classpath：/**config**/ | 项目名/src/main/resources/**config**/application.properties |
| -classpath：/**          | 项目名/src/main/resources/application.properties            |

* Spirng Boot 会从这四个位置全部加载主配置文件，最后组合到一起，也就是 ==互补配置==

#### 使用 spring.config.location 来改变配置文件位置

* 项目**打包好以后**，我们可以使用**命令行参数的形式**，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成**互补配置**
    * 命令：**java  -jar  包全名  --spring.config.location= 文件绝对路径**
* 打包方式：**只会将 src 目录下的内容进行打包**



## 外部配置的加载顺序

1. 命令行参数

* **java  -jar  包全名  --server.port=端口号**

1. 来自 java:comp/env 的 NDI 属性

1. Java系统属性（ System.getProperties()）
2. 操作系统环境变量
3.  RandomValuePropertySource配置的 randon.* 属性值



##### 优先加载带 profile 的

1. jar包外部的 application-{ proflle}. properties或 application.yml（带 spring. proflle）配置文件
2. jar包**内**部的 application-{ proflle}. properties或 application.yml（带 spring. proflle）配置文件

##### 再加载不戴 profile 的

1. jar包外部的 application.properties 或 application.yml 不带( spring.profile）配置文件
2. jar包**内**部的 application.properties 或 application.yml 不带( spring.profile）配置文件



1. @Configuration 注解类上的 @Propertysource
2. 通过 SpringApplication. setDefaultProperties 指定的默认属性

* **以上这些优先级由高到低**

## 自动配置原理

* SpringBoot 启动的时候加载主配置类，开启了自动配置功能 ==**@EnableAutoConfiguration**==

    * **@EnableAutoConfiguration**作用
        * 利用 ==**AutoConfigurationImportSelector**== 给容器中导入一些组件
            * 可以通过 ==**selectImports**== 方法来查看导入了哪些组件
                * selectImports 调用了同类中另一个方法 **==getAutoConfigurationEntry==**
                * getAutoConfigurationEntry 方法中有这样一句代码，**作用是获取候选的配置**
                    List<String> configurations =
                                              this.==**getCandidateConfigurations**==(annotationMetadata, attributes);
                    * **getCandidateConfigurations** 方法的作用
                        * 利用 SpringFactoriesLoader.loadFactoryNames 方法来扫描所有 jar 包类路径下的 **"META-INF/spring.factories"**
                        * 把扫描到的文件的内容包装成 **properties** 对象
                        * 从 properties 对象中获取到 ==**EnableAutoConfiguration.class**==(类名)对应的值，把他们添加到容器中
    * **一句话：该注解会将类路径下 META-INF/spring.factories 文件中所配置的 ==EnableAutoConfiguration.class== 的值添加到了容器中**

    ```
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
    org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
    org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
    org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
    org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
    ......
    ```

    * 像这样的 **AutoConfiguration** 类都是容器中的一个组件，都被添加到了容器中，用来做自动配置

* 每一个自动配置类进行自动配置功能，以 **HttpEncodingAutoConfiguration** 为例解释自动配置原理

    ```java
    @Configuration   
    @EnableConfigurationProperties({HttpProperties.class}) 
    @ConditionalOnWebApplication(type = Type.SERVLET)
    @ConditionalOnClass({CharacterEncodingFilter.class})
    @ConditionalOnProperty(   // 判断配置文件中是否存在某个配置（spring.http.encoding/enable）
        prefix = "spring.http.encoding", // matchIfMissing = true 表示若不存在值也为 true
        value = {"enabled"},
        matchIfMissing = true
    )
    public class HttpEncodingAutoConfiguration {}
    ```

    * **@Configuration**：表示这是一个配置类

    * **@EnableConfigurationProperties**：表示启动 ==**ConfigurationProperties**== 功能

        ```java
        @ConfigurationProperties(prefix = "spring.http")
        public class HttpProperties {
        ```

        * @ConfigurationProperties：从配置文件中获取指定的值和 bean 的属性进行绑定
            * 所有在配置文件中能配置的属性都是在 ***Properties 类中封装着

    * **@ConditionalOnWebApplication**：底层是 Spring 的 @Conditional，作用是根据不同的条件来判断整个配置类是否生效（这个例子即：判断是不是一个 web 应用，是就生效）
    * **@ConditionalOnClass**：判断当前项目有没有这个类（**CharacterEncodingFilter**.**class**）
    * ==**一句话解释：根据各种不同的条件来判断当前配置类是否生效，一旦生效就会往容器中添加各种组件，这些组件的属性是从对应的 properties 类中获取的，这些类里面的每一个属性又是和配置文件绑定的**==

* 精髓
    * SpringBoot 启动会加载大量自动配置的类
    * 看看我们需要的功能 SpringBoot 有没有帮我们写好默认自动配置类
    * 如果有，再来看这个自动配置类中到底配置了哪些组件，有没有我们需要的，没有就需要自己写了
    * 给容器中自动配置类添加组件的时候，会从 properties 类中获取某些属性，我们可以在自己的配置文件中指定这些属性的值
        * xxxAutoConfiguration ：自动配置类
        * xxxProperties：封装配置文件中相关的属性



## 自动配置报告

#### 怎样知道哪些自动配置类生效了

* 在配置文件中使用 **==debug=true==** 来让控制台打印生效的自动配置类的报告
    * **Positive matches**：生效
    * **Negative matches**：未生效

























