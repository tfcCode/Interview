

# 一、基本使用

1、全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/employeeMapper.xml"/>
    </mappers>
</configuration>
```

2、映射配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Employee">  <!--名称空间的名字随便起，但为了清晰含义，一般与类名相对应-->

    <select id="selectEmp" resultType="com.tfc.entity.Employee">
        select * from employee where id = #{id}
    </select>

</mapper>
```

3、测试

```java
@Test
public void test() throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    
    // 获取 sqlSession 实例，能执行已经映射的 sql 语句
    SqlSession sqlSession = sqlSessionFactory.openSession();
    Object selectEmp = sqlSession.selectOne("selectEmp", 1);
    System.out.println(selectEmp);

    sqlSession.close();
}
```



4、`Mapper` 配置文件与接口绑定

首先写一个接口，接口中是各种查询方法，查询语句写在配置文件中与之绑定

```java
public interface EmployeeDao {
    Employee getById(int id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tfc.dao.EmployeeDao"> <!--与接口绑定-->

    <!--与接口中的方法进行绑定-->
    <select id="getById" resultType="com.tfc.entity.Employee">
        select * from employee where id = #{id}
    </select>

</mapper>
```

测试

```java
public SqlSessionFactory getSessionFactory() throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    return sqlSessionFactory;
}
@Test
public void test1() throws IOException {
    SqlSessionFactory sqlSessionFactory = getSessionFactory();
    SqlSession sqlSession = sqlSessionFactory.openSession();
    EmployeeDao mapper = sqlSession.getMapper(EmployeeDao.class);
    Employee byId = mapper.getById(1);
    System.out.println(byId);
    
    sqlSession.close();
}
```



# 二、全局配置文件

> 全局配置文件约束

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
```



**各种标签的顺序严格按照官网给定的顺序，否则会报错** 

## properties

用来引入外部配置文件，例如数据库连接信息

```xml
<!--resource 表示类路径下， url 表示磁盘目录-->
<properties resource="db.properties"/>
```

用来配置数据库连接信息时，`key` 前要加一个 `jdbc`，例如 `jdbc.url` 



## settings

官方文档有详细说明：https://mybatis.org/mybatis-3/zh/index.html

以驼峰命名法举例

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```



### typeAliases（类型别名）

为类起一个别名，这样在需要全类名的时候就不用总是写全类名，写一个别名即可

```xml
<typeAliases>
    <!--单独起别名-->
    <typeAlias type="com.tfc.dao.EmployeeDao" alias="emp"/>
    <!--批量起别名，默认别名是类名的首字母小写（不区分大小写）-->
    <package name="com.tfc.dao"/>
</typeAliases>
```

在**批量**起别名时，如果有冲突，还可以在类上用 `@Alias` 注解指定一个别名

基本类型的别名 `Mybatis`  已经起好，基本类名在前面加一个`_`，包装类型是首字母小写



# 三、映射配置文件



## 增删改查

```java
public interface EmployeeDao {
    Employee getById(int id);

    void addEmp(Employee employee);
    void updateEmp(Employee employee);
    void deleteEmp(int id);
}
```



```xml
<select id="getById" resultType="com.tfc.entity.Employee">
    select * from employee where id = #{id}
</select>

<!--parameter 参数可以省略-->
<insert id="addEmp" parameterType="employee">
    insert into employee values(#{id},#{name},#{gender},#{email})
</insert>

<update id="updateEmp" parameterType="employee">
    update employee set name = #{name},email = #{email}, gender = #{gender} where id = #{id}
</update>

<delete id="deleteEmp" parameterType="_int">
    delete from employee where id = #{id}
</delete>
```

使用自增主键策略

使用 `useGeneratedKeys、keyProperty` 来实现

```xml
<insert id="addEmp" parameterType="employee" useGeneratedKeys="true" keyProperty="id">
    insert into employee values(#{id},#{name},#{gender},#{email})
</insert>
```

```java
// 不用自己设置主键，MyBatis 替我们完成了
Employee employee = new Employee(null, "姜泥", "女", "4444444");
mapper.addEmp(employee);
```



## 参数处理

1、单个参数时，`MyBatis` 不做任何处理

```java
void deleteEmp(int id);
```

```xml
<delete id="deleteEmp" parameterType="_int">
    delete from employee where id = #{idaaa} <!--只有一个参数时，任意标识都符合要求-->
</delete>
```



2、多个参数时，`MyBatis` 会将参数封装成一个 `map`，结构如下

> key：param1、...... paramN
>
> value：传进来的值

取值的时候要这样来取：`#{param1}` 

```java
Employee getByIdAndName(int id, String name);
```

```xml
<select id="getByIdAndName" resultType="Employee">
    select * from employee where id = #{param1} and name = #{param2}
</select>
```



如果其中一个参数是一个对象，需要用 `.` 来取出其中的数据，例如 `#{param.name}` 



这样操作很不方便，可以在方法上添加 `@Param` 注解指定一个清晰易懂的名字

```java
Employee getByIdAndName(@Param("id") int id, @Param("name") String name);
```

```xml
<select id="getByIdAndName" resultType="Employee">
    select * from employee where id = #{id} and name = #{name}
</select>
```



3、如果传入的参数恰好是封装的数据模型对象，可以直接传入

```xml
<update id="updateEmp" parameterType="employee">
    update employee set name = #{name},email = #{email}, gender = #{gender} where id = #{id}
</update>
```



4、如果参数很多，一个一个写很麻烦，可以用一个 `Map<key, value>` 来传递参数

```java
Employee getByIdAndName(Map<String, Object> map);
```

```java
Map<String, Object> map = new HashMap<>();
map.put("id", 1);
map.put("name", "徐凤年");
Employee a = mapper.getByIdAndName(map);
```

```xml
<select id="getByIdAndName" resultType="Employee">
    select * from employee where id = #{id} and name = #{name}
</select>
```



5、参数为 `String` 时，且在动态 `SQL` 中对 `String` 类型的参数进行 `!=null` 判断，此时不能直接用参数名进行判断，需要使用 `_parameter` 代替参数名

```java
List<Student> findByAddr(String addr);
```

```mysql
<select id="findByAddr" resultType="Student" parameterType="String">
    select * from student
    <where>
        <if test="_parameter != null">  # 使用 _parameter 代替参数名进行判断
            address = #{addr}
        </if>
    </where>
</select>
```



## #{}、${} 区别

1、`#{}` 是由预编译处理，也就是 `PrepareStatement`，防止 `SQL` 注入

2、`${}` 是直接将值取出来进行拼接，一般用来原生 `JDBC` 不支持占位符处理的地方，比如表名、分表、排序

```sql
select * from user order by  ${age}
```



`#{}` 还要一些属性可以指定数据类型，可以处理 `NULL` 的情况

```xml
<update id="updateEmp" parameterType="employee">
  update employee set name = #{name},email = #{email,jdbcType = NULL}, gender = #{gender} where id = #{id}
</update>
```



## 返回集合

```java
List<Employee> findAll();
```

```xml
<!--返回一个集合，结果类型要写集合中元素的类型-->
<select id="findAll" resultType="Employee">
    select * from employee;
</select>
```



## 返回 Map

1、返回单个 `Map`：`key` 是列名，`value` 是查出来的值

```java
Map<String, Object> getEmpReturnMap(int id);
```

```xml
<!--返回一个 Map-->
<select id="getEmpReturnMap" resultType="map">
    select * from employee where id = #{id}
</select>
```

2、返回多个数据，用 `Map` 封装

```java
@MapKey("id")  // 用哪个属性来作为 map 的 key
Map<Integer,Employee> getEmpRetuenMaps();
```

```xml
<select id="getEmpRetuenMaps" resultType="Employee">
    select * from employee
</select>
```



## 结果映射 resultMap

当数据库字段和实体类属性名称不一致时，可以手动设置，以便映射

```xml
<!-- id 是唯一标识符，type 是实体类类型-->
<resultMap id="empMap" type="Employee">
    <!--id 用来指定主键，底层会有优化，用 result 指定主键也可以-->
    <id column="id" property="id"/>
    
    <!--result 用来封装普通的字段，column 是数据库字段，property 是实体类中的属性字段-->
    <result column="name" property="name"/>
    
    <!--不需要将全部的字段都写出，写出关键的即可，其余的会自动一一映射-->
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
</resultMap>
```

```xml
<select id="getEmpByID" resultMap="empMap">
    select * from employee where id = #{id};
</select>
```



当实体类属性是一个对象时

```java
public class Employee {
    .....
    private Department department;
    .....
}
```

```java
Employee getEmpAndDept(int id);
```

方法一：级联方式 `department.id` 

```xml
<!--id  name   gender email  dept_id        d_id   departmentName-->
<!--这是字段是查询后结果中的字段，将这些字段封装到对象中即可-->
<resultMap id="empMapPlus" type="Employee">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <result column="d_id" property="department.id"/>
    <result column="departmentName" property="department.departmentName"/>
</resultMap>

<select id="getEmpAndDept" resultMap="empMapPlus">
    select * from employee e, department d where e.dept_id = d.d_id and e.id = #{id};
</select>
```

方法二：使用 `association` 标签

```xml
<!--id  name   gender email  dept_id        d_id   departmentName-->
<resultMap id="empMapPlus" type="Employee">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!--必须指定 javaType -->
    <association property="department" javaType="Department">
        <id column="d_id" property="id"/>
        <result column="departmentName" property="departmentName"/>
    </association>
</resultMap>
```



## 一对一分步查询

举例，员工信息、部门信息在两张表中，通过部门的主键相关联，若要查询员工的部门信息可以这样做：

1、先查询员工信息

2、根据员工的部门信息`dept_id` 去查询员工的部门信息

3、将部门信息封装到员工实体类中

```xml
<resultMap id="empByStep" type="Employee">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <!--select 是指定哪个查询语句  column 是将员工信息中的哪个字段信息传给查询语句-->
    <!--传递多个值时这样处理，column = {key1 = value1,key2 = value2}-->
    <association property="department"
                 select="com.tfc.dao.DepartmentDao.getDeptById"
                 column="dept_id"/>
</resultMap>
<select id="getEmyByIdStep" resultMap="empByStep">
    select * from employee where id = #{id};
</select>
```



> 延迟加载

​	若每次查询员工信息时都附带查询员工的部门信息，但部门信息并不是每次都有用，可以用分步查询的延迟加载，需要的时候再查询部门信息，而不是每次都查询，节省了数据库资源

方法：在主配置文件中添加两个配置

```xml
<!--开启后，关联的值可以在需要的时候加载-->
<setting name="lazyLoadingEnabled" value="true"/>
<!--开启表示所有属性都一次性加载，关闭后按需加载-->
<setting name="aggressiveLazyLoading" value="false"/>
```



## 一对多查询

例如：一个部门有多个员工

```xml
<resultMap id="myDept" type="Department">
    <id column="d_id" property="id"/>
    <result column="departmentName" property="departmentName"/>
    <!--用 collection 来封装集合，ofType 用来指定集合中的元素类型-->
    <collection property="employees" ofType="Employee">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="gender" property="gender"/>
        <result column="email" property="email"/>
    </collection>
</resultMap>
<select id="getDeptAndEmp" resultMap="myDept">
   select * from department d LEFT JOIN employee e on d.d_id = e.dept_id WHERE d.d_id = #{id}
</select>
```



## 一对多分步查询

```xml
<resultMap id="myDeptStep" type="Department">
    <id column="d_id" property="id"/>
    <result column="departmentName" property="departmentName"/>
    <!--select 是指定哪个查询语句  column 是将员工信息中的哪个字段信息传给查询语句-->
    <!--传递多个值时这样处理，column = {key1 = value1,key2 = value2}-->
    <collection property="employees"
                select="com.tfc.dao.EmployeeDao2.getEmpsByDeptId"
                column="id"/>
</resultMap>
<select id="getDeptAndEmpByStep" resultMap="myDeptStep">
    select d_id id, departmentName from department where d_id = #{id}
</select>
```



# 四、动态 SQL

`MyBatis` 中动态 `SQL` 的判断使用的表达式是用的是 `OGNL` 表达式

## if

```xml
<!--携带了哪个字段，查询条件就带上哪个字段的值-->
<select id="getEmpByConditionIf" resultType="Employee">
    select * from employee
    where
    <if test="id!=null">
        id = #{id}
    </if>
    <!--判断不为空可以用单引号-->
    <if test="name!=null and name!=''">
        and name = #{name}
    </if>
    <!--可以调用函数-->
    <if test="email!=null and email.trim()!=''">
        and email = #{email}   <!--多个条件 and 不可少-->
    </if>
</select>
```



## where

​	如果第一个条件判断失败，导致多出了一个 `and`，这时候 `SQL` 拼接就会出错，这时可以使用 `where` 标签来解决，但这种方式只会过滤第一个多余的 `and` 或 `or` 

```xml
<select id="getEmpByConditionIf" resultType="Employee">
    select * from employee
    <where>
        <if test="id!=null">
            id = #{id}    <!--即使 id 没有拼接上，也不会出错-->
        </if>
        <if test="name!=null and name!=''">
            and name = #{name}
        </if>
        <if test="email!=null and email.trim()!=''">
            and email = #{email}
        </if>
    </where>
</select>
```



## choose

```xml
<!--哪个有值，就用哪个查，依次判断-->
<select id="getEmpByConditionChoose" resultType="Employee">
    select * from employee
    <where>
        <choose>
            <when test="id!=null">
                id = #{id}
            </when>
            <when test="name!=null">
                name = #{name}
            </when>
            <otherwise>
                id = 1
            </otherwise>
        </choose>
    </where>
</select>
```



## set

```xml
<update id="updateEmp">
    update employee
    <!--使用 set 标签可以去除多余的逗号-->
    <set>
        <if test="name!=null">
            name = #{name},
        </if>
        <if test="email!=null">
            email = #{email}
        </if>
    </set>
    where id = #{id}
</update>
```



## foreach

1、批量查询

```java
List<Employee> getEmyByConditionForEach(List<Integer> nums);
```

```xml
<!--
    collection：指定要遍历的集合（也就是传进来的参数）
          item：将每个值赋给一个变量，起一个变量名
     separator：指定元素是分隔符
          open：遍历出来的结果拼接的首字符
         close：遍历出来的结果拼接的尾字符
         index：遍历 list 时是索引，遍历 map 时是 key，item 是对应的值
-->
<select id="getEmyByConditionForEach" resultType="Employee">
    select * from employee where id in
    # collection 的值必须是 list
    <foreach collection="list" item="item_id" separator="," open="(" close=")">
        #{item_id}
    </foreach>
</select>
```



2、批量插入

```java
void insertEmps(@Param("emps") List<Employee> emps);
```

```xml
<insert id="insertEmps">
    insert into employee(name,gender,email,dept_id)
    values
    <foreach collection="emps" item="emp" separator=",">
        (#{emp.name},#{emp.gender},#{emp.email},#{emp.dept_id})
    </foreach>
</insert>
```



## bind

用来给 `OGNL` 表达式绑定一个变量，以便使用

```xml
<select id="find" resultType="Employee">
    <bind name="_id" value="1"/>
    select * from employee
    <where>
        id = #{_id}
    </where>
</select>
```



## sql

抽取公共 `SQL` 片段

```xml
<sql id="common">
    id,name,email,gender
</sql>
<insert id="a">
    insert into employee values (
        <include refid="common"/>
    )
</insert>
```



# 五、MyBatis 内置参数

1、`_parameter` 

> 表示传进来的整个参数，若是单个参数，直接用；若是多个参数，会封装成一个 `map` 



2、`_databaseId` 

> 如果配置了 `databaseIdProvider`，该属性就是当前数据库的别名
>
> 这样就利用别名来判断当前是哪个数据库，这样就不用再分开写两个 `SQL` 了

```xml
<select id="">
    <if test="_databaseId=='mysql'">
        select * from employee
    </if>
    <if test="_databaseId=='oracle'">
        select * from dept
    </if>
</select>
```



# 六、缓存

## 一级缓存

> `sqlSession`  级别的缓存

​		默认开启的，无法关闭。也叫本地缓存，与数据库同一次期间查询的数据会放在本地缓存中，如果以后需要获取相同的数据，直接从缓存中拿，不需要再次查询数据库



> 一级缓存失效

1、`sqlSession` 不同

2、两次查询期间进行了增、删、改操作

* 原因：执行增、删、改的时候可能改动过当前数据，所以需要重新查询数据库

3、手动清除一级缓存

* `sqlSession.clearCache();` 

4、查询条件不同



## 二级缓存

> 也叫全局缓存，基于 `namespace` 级别的缓存，一个 `namespace` 对应一个二级缓存

​		工作机制：一次会话中，查询数据会放在一级缓存中，如果会话关闭，则会将一级缓存中的数据保存到二级缓存中，新的会话就会到二级缓存中获取数据



> 使用细节

1、开启二级缓存

```xml
<setting name="cacheEnabled" value="true"/>
```

2、在映射配置文件中使用 `<cache>` 标签开启二级缓存

```xml
<cache eviction="" flushInterval="" readOnly="" size="" type=""/>
```

`eviction`：缓存策略

* `LRU`：最近最少使用，移出最长时间没有使用过的缓存（默认）
* `FIFO`：先进先出，按对象进入的顺序移出
* `SOFT`：软引用，移除基于垃圾回收器状态和软引用规则的对象
* `WEAK`：弱引用，更积极的移除基于垃圾回收器状态和弱引用规则的对象

`flushInterval`：缓存刷新间隔

* 默认不清空，以毫秒为单位

`readOnly`：缓存是否只读

* `true`：所有数据都是只读，为了加快速度，会直接将引用交给用户。不安全，但速度快
* `false`：利用序列化技术克隆一份新的数据返回用户。安全，但速度较慢

`size`：缓存元素个数

`type`：自定义缓存的类型（全类名）

3、`POJO` 需要实现序列化接口



## 和缓存有关的设置

1、关闭全局配置中的缓存关的是二级缓存，一级缓存一直存在

```xml
<setting name="cacheEnabled" value="true"/>
```

2、每个 `select` 标签中都有一个 `useCache` 属性，该属性影响的是二级缓存

3、每个**增删改**标签都有一个 `flushCache` 属性，默认是 `true`，表示每次增删改之后都会立即清除所有（一级、二级）缓存，但查询标签中该属性默认是 `false` 

4、`sqlSession.clearCache()` 清除的是一级缓存

5、`localCacheScope`：本地缓存作用域

* 默认是 `SESSION`，表示所有数据保存在当前会话中
* 若是 `STATEMENT`，表示不存储任何数据，相当于禁用一级缓存











































