# Mybatis
官方文档：https://mybatis.org/mybatis-3/zh/index.html

## 简介
MyBatis 是一款优秀的**持久层框架**，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

## 入门
### 安装
maven构建项目，在pom.xml文件中简单添加：
```xml=
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```
然后从XML中构建
### 构建SqlSessionFactory
* 从XML中构建SqlSessionFactory 配置mybatis配置文件：
```xml=
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```
idea则可以配置mybatis的模板文件，将上面的文件添加到模板文件中去：
![](https://i.imgur.com/5IIxN8L.png)
然后右键新建文件就可以直接新建mybatis配置模板文件了。
* 不使用XML构建SqlSessionFactory 
```java=
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```
### 获取SqlSession
从SqlSessionFactory获取SqlSession，目前两种方法：
一是通过session.select/update/delete...执行，而是通过定义Mapper接口，在接口中添加相关的注解以及方法：

```java=
// 通过XML中定义的唯一id（命名空间 + id）来执行SQL
// 这里若只有一个mapper.xml不会产生命名空间冲突，直接使用getDomainById即可。
VulnBeans c=  session.selectOne("com.example.demo.BlogMapper.getDomainById",100);
System.out.println(c);

// 通过Mapper，定义接口方法（通过注解）来执行SQL
// 通过调用java对象来执行SQL，明确传入参数与返回类型
MyMapper myMapper = session.getMapper(MyMapper.class);
VulnBeans d = myMapper.selectDomainById(100);
System.out.println(d);
```
第一种方法需要的配置Mapper.xml文件：
```xml=
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.BlogMapper">
    <select id="getDomainById" parameterType="int" resultType="com.example.demo.VulnBeans">
        SELECT * FROM scan_project_domain WHERE id = #{id};
    </select>
    <select id="getIpById" parameterType="int" resultType="com.example.demo.VulnBeans">
        SELECT * FROM scan_project_ip WHERE id = #{id};
    </select>
</mapper>
```
第二种方法需要的Mapper.java类：（第二种方法优点在于可以像一般的java对象调用，将已映射的 select 语句匹配到对应名称、参数和返回类型）
```java=
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface MyMapper {
    @Select("select * from scan_project_domain where id = #{id}")
    VulnBeans selectDomainById(int id);
}
```
两种方法都需要在总的mybatis-config.xml中注册：
```xml=
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
                <property name="url" value="jdbc:mysql://localhost:3306/workspace"/>
                <property name="username" value="root"/>
                <property name="password" value="XXXXXX"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="BlogMapper.xml"/>
        <!-- 引用com.example.demo下的MyMapper.java作为mapper -->
        <mapper class="com.example.demo.MyMapper"/>
        <!-- 引用该包下所有的接口作为mappers -->
        <package name="com.example.demo"/>
    </mappers>
</configuration>
```
### 作用域与生命周期
* SqlSessionFactoryBuilder
这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 

* SqlSessionFactory
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是**在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”**。因此 SqlSessio
nFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用**单例模式或者静态单例模式**。比如创建一个辅助类，实现单例模式。

* SqlSession
**每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的**，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对**不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。** 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，**每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它**。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中，例如：
```java=
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

* 映射器实例
映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的。虽然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但方法作用域才是映射器实例的最合适的作用域。 也就是说，**映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃**。 映射器实例并不需要被显式地关闭。尽管在整个请求作用域保留映射器实例不会有什么问题，但是你很快会发现，在这个作用域上管理太多像 SqlSession 的资源会让你忙不过来。 因此，最好将映射器放在方法作用域内。就像下面的例子一样：
```java=
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // 你的应用逻辑代码
}
```
### 测试类举例
```java=
public class TestMybatis {
    public static void main(String[] args) throws IOException {
        String resource = "Mybatis-example.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();

        // 通过XML中定义的唯一id（命名空间 + id）来执行SQL
        VulnBeans c=  session.selectOne("com.example.demo.BlogMapper.getDomainById",100);
        System.out.println(c);

        // 通过Mapper，定义接口方法（通过注解）来执行SQL
        MyMapper myMapper = session.getMapper(MyMapper.class);
        VulnBeans d = myMapper.selectDomainById(100);
        System.out.println(d);

        session.commit();
        session.close();
    }
}
```
## 配置
### XML配置文件
一般采用默认配置，只配置数据库连接<environments>以及映射器<mappers>遇到问题再去看文档。
### XML映射文件
* select
```
id	在命名空间中唯一的标识符，可以被用来引用这条语句。
parameterType	将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。
resultType	期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。
resultMap	对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。
flushCache	将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。
useCache	将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。
timeout	这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。
fetchSize	这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。
statementType	可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。
resultSetType	FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。
databaseId	如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。
resultOrdered	这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：false。
resultSets	这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。
```
* insert, update以及delete
* sql
这个元素可以用来**定义可重用的 SQL 代码片段，以便在其它语句中使用**。 参数可以静态地（在加载的时候）确定下来，并且可以在不同的 include 元素中定义不同的参数值。比如：
```xml=
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```
* 参数
之前见到的所有语句都使用了简单的参数形式。但实际上，参数是 MyBatis 非常强大的元素。对于大多数简单的使用场景，你都不需要使用复杂的参数，比如：
```xml=
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```
鉴于参数类型（parameterType）会被自动设置为 int，这个参数可以随意命名。原始类型或简单数据类型（比如 Integer 和 String）因为没有其它属性，会用它们的值来作为参数。 然而，如果传入一个复杂的对象，例如User 类型的参数对象传递到了语句中，会查找 id、username 和 password 属性，然后将它们的值传入预处理语句的参数中。。比如：
```xml=
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```
**字符串替换：#换为$，不安全，直接传入会导致SQL注入！**
比如mapper中实现方法，添加注解：
```java=
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```
然后调用该mapper：
```java=
User userOfId1 = userMapper.findByColumn("id", 1L);
User userOfNameKid = userMapper.findByColumn("name", "kid");
User userOfEmail = userMapper.findByColumn("email", "noone@nowhere.com");
```
* 结果映射
    * resultType
    ```xml=
    <!-- mybatis-config.xml 中 , 类型别名，省略写全包名-->
    <typeAlias type="com.someapp.model.User" alias="User"/>
    <!-- SQL 映射 XML 中 ，别名来统一sql字段名与java中字段名-->
    <select id="selectUsers" resultType="User">
      select id as id, 
      user_name as username, 
      hashed_password, hashedPassword
      from some_table
      where id = #{id}
    </select>
    ```
    * resultMap
    定义一个resultMap：
    ```xml=
    <resultMap id="userResultMap" type="User">
      <id property="id" column="user_id" />
      <result property="username" column="user_name"/>
      <result property="password" column="hashed_password"/>
    </resultMap>
    ```
    引用该resultMap：
    ```xml=
    <select id="selectUsers" resultMap="userResultMap">
      select user_id, user_name, hashed_password
      from some_table
      where id = #{id}
    </select>
    ```
* 高级结果映射（复杂的resultMap XML编写）
### 动态SQL
* if
```xml=
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```
* choose、when、otherwise
```xml=
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```
* trim、where、set
where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。
```xml=
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```
* foreach
多个参数，例如id=1,2,3，这样使用foreach而不拼接降低SQL注入产生概率。
```xml=
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```
*...
### Java API
### SQL语句构建器
### 日志