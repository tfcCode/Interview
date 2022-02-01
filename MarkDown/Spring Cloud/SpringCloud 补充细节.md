# 一、Nacos Discovery

这个是 Spring Cloud Alibaba 提供的组件，需要引入 Spring Cloud Alibaba

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



> 使用服务发现步骤

1、引入服务发现依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2、添加配置

* 服务名
* Nacos 注册中心地址（需要先下载 Nacos 并启动）

Nacos 管理界面地址：http://localhost:8848/nacos

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 注册中心地址
  application:
    name: mall-member  # 服务名
```

3、在启动类上添加 `@EnableDiscoveryClient`

```java
@EnableDiscoveryClient
@SpringBootApplication
public class MallCouponApplication {...}
```



# 二、OpenFeign 服务调用

1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2、编写一个接口，告诉 Spring Cloud 这个接口需要进行远程调用

* 在接口上使用`@FeignClient`注解

```java
@FeignClient("mall-coupon")
public interface CouponFeignService {
    @RequestMapping("/coupon/coupon/member/list")
    public R memberCoupons();
}
```

解释：如果某个服务调用`memberCoupons()`方法，会调用`mall-coupon`服务的`/coupon/coupon/member/list`地址下的方法

3、在主程序上开启远程调用功能

```java
@EnableFeignClients("com.tfc.mall.member.feign")    // 远程调用
@EnableDiscoveryClient
@SpringBootApplication
public class MallMemberApplication {...}
```

4、测试

```java
// 远程调用接口
@Autowired
CouponFeignService feignService;    // 远程调用服务

@RequestMapping("/coupons")
public R feignTest() {
    ...
    R r = feignService.memberCoupons();   // 调用远程服务
    ...
}
```



# 三、Nacos Config

1、引入依赖

* 高版本的 Spring Boot 还需要引入`spring-cloud-starter-bootstrap`依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<!--高版本需要-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.1.0</version>
</dependency>
```

2、在类路径下创建一个`bootstrap.properties`文件，该文件被读取的优先级较高

* 添加以下内容

```properties
spring.application.name=mall-coupon                   // 服务名
spring.cloud.nacos.config.server-addr=127.0.0.1:8848  // 配置中心地址
```

3、启动自动刷新数据

* 若没有生效，记得重启服务器

```java
@RefreshScope // 启动自动刷新数据
@RestController
@RequestMapping("coupon/coupon")
public class CouponController {...}
```

4、在 Nacos 图形化界面的中添加配置

Data ID的名字有有一定规则，规则如下

```xml
${prefix} - ${spring.profiles.active} . ${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值

    - 也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置

- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)

    **注意，当 active profile 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}`.`${file-extension}`** 

- `file-extension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension`来配置。目前只支持 `properties` 类型

* group
    * `group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置



















