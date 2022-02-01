# 一、Spring Boot 入门案例

## 1、maven 配置

```xml
<!--配置父依赖，之后就不需要自己来控制版本-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

## 2、新建一个类

#### 1、使用 @SpringBootApplication 注解

​		表示这是一个主程序

#### 2、启动

```java
SpringApplication.run(HelloWorldApplication.class, args);
```

#### 3、编写逻辑代码

#### 3、打包

* 添加插件依赖

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* 打包

    ​	打开 maven 管理（右侧边框），在 lifecycle 中点击 package

## 探究 pom.xml 文件

#### 1、父依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
</parent>

它的父依赖
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
这个里面的 spring-boot-dependencies 是真正来管理所有的依赖
```

这是 Spring Boot 的版本仲裁中心，这里面的依赖的版本不需要我们来控制

==这里面没有的依赖需要我们自己手动控制== 

#### 2、导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter**-==web==

* **spring-boot-starter**：Spring Boot 的场景启动器，帮我们导入所需模块正常运行的所有依赖

* spring boot 将所有的功能场景都抽取出来，做成一个个 stater（启动器），只需要在项目里引入相应的 stater 即可使用相应的功能

## 探究主程序类

```java
/**
 * @SpringBootApplication 注解用来标注这是一个主程序类，是一个 Spring Boot 应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {

        // 启动 Spring 应用
        SpringApplication.run(HelloWorldApplication.class,args);
    }
}
```

* **@SpringBootApplication** 标注在类上，说明这个类是 Spring Boot 的主配置类，Spring Boot 应该运行这个类中的 main 方法来启动 Spring Boot 应用

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

* **@SpringBootConfiguration**

1. 标志某个类上，表明这是一个 Spring Boot 配置类

* **@EnableAutoConfiguration** 

1. 开启自动配置功能

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

**@AutoConfigurationPackage**：自动配置包

​	Spring 的底层注解 @Import 给容器中导入一个组件，将**主配置类**所在包以及子包下的所有组件扫描到Spring 容器中

**@Import({AutoConfigurationImportSelector.class})** 

​			给容器中导入一些组件

* **AutoConfigurationImportSelector**：导入组件选择器，会给容器中导入很多自动配置类，也就是导入这个场景所需的所有组件，并配置好这些组件









