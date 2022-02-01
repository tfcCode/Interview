# 通用 Mapper



## 一、配置

### 1、代码方式

```java
MapperHelper mapperHelper = new MapperHelper();

//特殊配置
Config config = new Config();
//具体支持的参数看后面的文档
config.setXXX(XXX);

//设置配置
mapperHelper.setConfig(config);

// 注册自己项目中使用的通用Mapper接口，这里没有默认值，必须手动注册
mapperHelper.registerMapper(Mapper.class);

//配置完成后，执行下面的操作
mapperHelper.processConfiguration(session.getConfiguration());
```



### 2、XML 方式

* 基本配置

```xml
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.isea533.mybatis.mapper"/>
    <property name="properties">
        <value>
            mappers=tk.mybatis.mapper.common.Mapper
        </value>
    </property>
</bean>
```

* 唯一不同的是将 **org** 改成了 **tk**

* 通用 Mapper 的各项属性通过 `properties` 属性进行配置



## 二、使用

### 1、接口继承 `Mapper<T>`

* 必须制定泛型

```java
public interface AccountMapper extends Mapper<Account> {

}
```

* 一旦继承了 `Mapper<T>`，继承的 `Mapper` 就拥有了 `Mapper<T>` 所有的通用方法

注意

* **简单的增删改查==只需要继承接口==就够了**
* 如果需要其他复杂的查询还是和以前一样：写接口方法，使用注解或 xml 文件



### 2. 泛型（实体类） `<T>` 的类型必须符合要求

实体类按照如下规则和数据库表进行转换，注解全部是 JPA 中的注解

1. 表名默认使用类名,驼峰转下划线（只对大写字母进行处理）
    * 如 `UserInfo ` 默认对应的表名为 `user_info`
2. 表名可以使用 `@Table(name = "tableName")` 进行指定
    * 对不符合第一条默认规则的可以通过这种方式指定表名
3. 字段默认和 `@Column` 一样，都会作为表字段
    * 表字段默认为 Java 对象的 `Field` 名字驼峰转下划线形式
4. 可以使用 `@Column(name = "fieldName")` 指定不符合第 3 条规则的字段名
5. 使用 `@Transient` 注解可以**忽略字段**
    * 添加该注解的字段不会作为表字段使用
6. **建议一定是有一个 `@Id` 注解作为主键的字段**
    * **可以有多个 `@Id` 注解的字段作为联合主键**
7. **默认情况下，实体类中如果不存在包含 `@Id` 注解的字段**
    * **所有的字段都会作为主键字段进行使用（==这种效率极低==）**
8. 实体类可以继承使用
    * 可以参考测试代码中的 `tk.mybatis.mapper.model.UserLogin2` 类
9. 由于基本类型，如 int 作为实体类字段时，会有默认值 0，而且无法消除
    * 所以实体类中建议不要使用基本类型.
10. `@NameStyle` 注解
    * 用来配置 对象名/字段、表名/字段 之间的转换方式，该注解优先于全局配置 `style`，可选值
        * `normal`：使用 **实体类名/属性名** 作为 **表名/字段名**
        * `camelhump`：**这是默认值**，驼峰转换为下划线形式
        * `uppercase`：转换为大写
        * `lowercase`：转换为小写



### 3、在实体类中使用注解来标明字段

```java
public class Account {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private Double money;
    ....
}
```

* 推荐必须使用一个 `@Id` 注解，不然所有字段都会默认成为主键，效率极低



### 4、使用继承过来的方法

```java
public interface AccountMapper extends Mapper<Account> {
  // 没有自己写的方法
}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountMapper mapper;

    @Override
    public List<Account> findAll() {
        return mapper.selectAll();   // 使用继承过来的方法
    }
}
```



























