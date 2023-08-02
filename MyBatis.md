# MyBatis 3

MyBatis 是一款持久层框架，支持动态 SQL、高级映射，以及存储过程。MyBatis 几乎省去所有 JDBC 代码，以及设置参数、获取结果集的工作，使用 XML 或注解把 Java 对象映射为数据库记录。

# 初步使用

纯粹 MyBatis 使用，没有集成 Spring 框架。

## 引入依赖

```
<dependencies>
    <!-- mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>mybatis.version</version>
    </dependency>
    
    <!-- mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>mysql.version</version>
    </dependency>
</dependencies>
```

## 创建会话

`SqlSession` 表示一次数据库连接，MyBatis 使用这个对象进行各种数据库操作。注意，`SqlSession` 不是线程安全对象，每次使用都应该重新创建并在使用后关闭。

`SqlSessionFactory` 是 `SqlSession` 工厂，`SqlSessionFactoryBuilder` 可以基于配置文件创建会话工厂。

```
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = 
    new SqlSessionFactoryBuilder().build(inputStream);
```

使用 XML 定义 MyBatis 配置，比如数据源、事务管理器。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
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

## SQL 映射

### XML 定义

使用 XML 定义 SQL 语句，名称空间 `namespace` 可以认为是映射文件的名字，属性 `id` 用于文档内标识语句。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

使用 `SqlSession` 对象执行前面定义的 SQL 语句，这种方式无法限制参数，以及返回类型。

```
// 通过"namespace.id"指定要执行的SQL语句
Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
```

### 接口映射

基于前边的内容，额外创建一个接口，这个接口的全限定类名与 XML 映射文件的名称空间一致，其中方法的名字与语句 `id` 一致，这就把 SQL 语句绑定到接口方法。

```
package org.mybatis.example;

public interface BlogMapper {
    public Blog selectBlog(Long id);
}
```

现在，`SqlSession` 通过接口执行 SQL 语句，这种方式可以限制参数、返回的数量和类型。

```
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

### 注解定义

使用注解直接在映射接口处定义 SQL 语句，复杂 SQL 不适合这种方式，可以混用注解和 XML 方式。

```
package org.mybatis.example;

public interface BlogMapper {
    @Select("select * from Blog where id = #{id}")
    Blog blog = mapper.selectBlog(101);
}
```

## 自动提交

默认情况，会话需要手动提交 SQL 语句。

```
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
  sqlSession.commit();
}
```

使用可选参数，指定会话自动提交。

```
try(SqlSession sqlSession = sqlSessionFactory.openSession(true)){
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

# XML 配置

MyBatis 基于 XML 文件进行配置，以下是文档结构，各个元素的定义应该严格遵守前后顺序：

- Configuration 配置，顶级标签
  - Properties 属性
  - Settings 设置
  - TypeAliases 类型别名
  - TypeHandlers 类型处理器
  - ObjectFactory 对象工厂
  - Plugins 插件
  - Environments 环境配置
    - Environment 环境变量
      - DataSource 数据源
      - TransactionManager 事务管理器
  - DatabaseIdProvider 数据库厂商标识
  - Mappers 映射器

## 属性 Properties

属性就是键值对，文档的任何地方都能根据键来引用值，便于统一修改。

**定义属性**

使用 `resource` 属性引用外部 properties 文件，或者使用 property 子标签定义属性。

```
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

**引用属性**

使用 `${name}` 格式引用前面定义的属性的值。

```
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

**加载顺序**

MyBatis 按照以下顺序加载属性源：

- 首先读取 property 子标签定义的属性；
- 然后读取 `resource` 属性引用的 properties 文件，覆盖已有同名属性；
- 最后读取作为方法参数传递的属性，覆盖已有同名属性。

## 设置 Settings

这是极为重要的调整设置，影响 MyBatis 众多的运行行为，以下是常见设置的含义和默认值。

| 设置名                     | 描述                                                   | 有效值               | 默认值  |
| :------------------------- | :----------------------------------------------------- | :------------------- | :------ |
| `cacheEnabled`             | 开启所有映射文件配置的缓存                             | true \| false        | true    |
| `lazyLoadingEnabled`       | 全局启用延迟加载                                       | true \| false        | false   |
| `aggressiveLazyLoading`    | 调用任何方法都会加载所有延迟加载对象，默认是按需加载。 | true \| false        | false   |
| `useColumnLabel`           | 使用列标签代替列名                                     | true \| false        | true    |
| `useGeneratedKeys`         | 使用 JDBC 自动生成主键                                 | true \| false        | False   |
| `autoMappingBehavior`      | 自动映射级别                                           | NONE\|PARTIAL\|FULL  | PARTIAL |
| `defaultExecutorType`      | 默认的执行器                                           | SIMPLE\|REUSE\|BATCH | SIMPLE  |
| `defaultStatementTimeout`  | 等待数据库响应的超时秒数                               | 任意正整数           | null    |
| `mapUnderscoreToCamelCase` | 使用驼峰命名的自动映射                                 | true \| false        | False   |
| `jdbcTypeForNull`          | 空值的默认 JDBC 类型                                   | JdbcType 常量        | OTHER   |

## 类型别名 TypeAliases

就是 Java 类型的简写，仅用于配置文件和映射文件，意在降低冗余的全限定类名书写。

```
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
</typeAliases>
```

指定一个包，MyBatis 会搜索包内的  Java Bean 并使用首字母小写的类名作为它们的别名。

```
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

如果不想使用默认别名，可用 `@Alias` 注解自定义别名。

```
@Alias("author")
public class Author {
    ...
}
```

对于 Java 基本类型和一些常用 Java 类型，MyBatis 内置它们的类型别名，比如 `string`、`_long`、`_int`。

## 类型处理器 TypeHandlers

MyBatis 设置 PreparedStatement 语句中的参数，以及从结果集中获取一个值，都会用类型处理器把获取到的值转换成合适的 Java 类型。

MyBatis 内置许多类型处理器，可以重写已有的类型处理器，或者创建自己的类型处理器，从而处理不支持的或非标准的类型。具体通过实现 `TypeHandler` 接口，或者继承 `BaseTypeHandler` 完成。

自定义类型处理器，用于处理 String 类型的属性以及 VARCHAR 类型的参数和结果。

```
// ExampleTypeHandler.java
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

注册自定义的类型处理器，它会覆盖已有 `<String, VARCHAR>` 处理器。

```
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

通过类型处理器的泛型，MyBatis 得知该处理器处理的 Java 类型，这个行为可以通过两种方式修改：

* 类型处理器的配置标签的 `javaType` 属性；
* 使用 `@MappedTypes` 标注类型处理器，这个设置会被配置标签覆盖。

指定关联的 JDBC 类型，同样有两种方式：

* 类型处理器的配置标签的 `jdbcType` 属性；
* 使用 `@MappedJdbcTypes` 标注类型处理器，这个设置会被配置标签覆盖。

执行预处理语句，可以根据映射接口的方法参数得知 Java 类型，选择合适的处理器。对于结果映射，通过映射接口的返回类型只能知道 Java 类型，但是无法得知 JDBC 类型，所以如果没有设置 `jdbcType` 属性，MyBatis 将会根据 `<JavaType, Null>` 选择一个处理器。MyBatis 不会查询数据库元数据来获取 JDBC 类型，所以如果想要使用特定处理器，必须显式指定字段类型，结果映射以及 SQL 语句字段占位符都支持 `jdbcType` 属性。

## 枚举类型的类型处理

用于映射枚举类型 `Enum`，MyBatis 提供 `EnumTypeHandler` 和 `EnumOrdinalTypeHandler` 两种实现，默认使用前者，它把 `Enum`  值转换成对应的名字，后者则根据序数值映射成对应的整型数值。

> `EnumTypeHandler` 和 `EnumOrdinalTypeHandler` 都是泛型类型处理器，可以处理任何 `Enum` 类型。

`EnumOrdinalTypeHandler` 需要显式启用，添加标签。

```
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="java.math.RoundingMode"/>
</typeHandlers>
```

现在，自动映射器将会自动使用 `EnumOrdinalTypeHandler` 处理枚举类型，如果想局部使用名字映射，那就必须显式使用 `typeHandler` 属性为那些 SQL 语句指定类型处理器。

```
<mapper namespace="org.apache.ibatis.submitted.rounding.Mapper">  
  <resultMap type="org.apache.ibatis.submitted.rounding.User" id="usermap2">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="funkyNumber" property="funkyNumber"/>
    <result column="roundingMode" property="roundingMode" 
    					typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
  </resultMap>
  <select id="getUser2" resultMap="usermap2">
    select * from users2
  </select>
  <insert id="insert2">
      insert into users2 (id, name, funkyNumber, roundingMode) values (
        #{id}, #{name}, #{funkyNumber}, 
        #{roundingMode, typeHandler=org.apache.ibatis.type.EnumTypeHandler}
      )
  </insert>
</mapper>
```

## 对象工厂 ObjectFactory

MyBatis 使用对象工厂来实例化结果对象，默认的对象工厂只用于实例化目标类，要么通过无参构造器，要么通过存在的参数映射来调用有参构造器。如果想要修改默认行为，可以自定义对象工厂。

`ObjectFactory` 接口表示对象工厂，它有两个方法分别表示使用无参或有参的构造器来创建实例。

## 插件 Plugins

MyBatis 允许在映射语句执行过程中的某一点进行拦截调用，默认可以使用插件拦截以下方法的调用：

* `Executor (update,query,flushStatements,commit,rollback,getTransaction,close,isClosed)`
* `ParameterHandler (getParameterObject, setParameters)`
* `ResultSetHandler (handleResultSets, handleOutputParameters)`
* `StatementHandler (prepare, parameterize, batch, update, query)`

拦截器是比较底层的拓展，最好稍微了解一下那些可被重写的方法的行为，因为修改或重写已有的方法可能会破坏核心模块。`Interceptor` 接口表示拦截器，实现它并重写想要拦截的方法，即可创建拦截器。

## 环境配置 Environments

MyBatis 可以配置多个环境，这种机制可把 SQL 映射应用到多种数据库，现实情况也的确有这么做的需求，比如开发、测试、生产几个环境分许需要使用不同的配置。

**环境选择**

注意，尽管可以配置多个环境，但是每个 `SqlSessionFactory` 实例只能使用一种环境。所以，如果想连接多个数据库，就要创建相同数量的 `SqlSessionFactory` 实例。

通过 `SqlSessionFactoryBuilder#build()` 可选参数指定使用那种环境，以下两个方法可以指定环境：

* `new SqlSessionFactoryBuilder().build(reader, environment);`
* `new SqlSessionFactoryBuilder().build(reader, environment, properties);`

如果没有显式指定，`SqlSessionFactory` 将会加载默认的环境。

**环境配置 Environment**

`environments` 元素表示环境，`default` 属性指定默认环境，`environment` 子元素表示单个环境配置。

```
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

**事务管理器 TransactionManager**

MyBatis 内置两种事务管理器

* **JDBC**

  它会直接使用 JDBC 提交和回滚的功能，依赖从数据源获得的连接来管理事务作用域。默认情况，为了与某些驱动程序兼容，它会在关闭连接时自动提交。对于某些驱动程序自动提交没有必要，3.5.10 开始可以设置属性关闭自动提交。

  ```
  <transactionManager type="JDBC">
    <property name="skipSetAutoCommitOnClose" value="true"/> <!-- 关闭自动提交 -->
  </transactionManager>
  ```

* **MANAGED**

  它几乎什么都不做，从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期。默认情况，它会关闭连接，可以设置属性禁止这个行为。

  ```
  <transactionManager type="MANAGED">
    <property name="closeConnection" value="false"/> <!-- 禁止关闭连接 -->
  </transactionManager>
  ```

如果正在使用 Spring + MyBatis，没有必要配置事务管理器，因为 Spring 会使用自带的管理器覆盖这些配置。

**数据源 DataSource**

`DataSource` 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源，虽然数据源配置可选，但是如果想要使用延迟加载特性，就必须配置它。

MyBatis 内置三种数据源类型。**UNPOOLED** 数据源会在每次请求时打开和关闭连接，显然，这比较慢，但对连接可用性要求不高的简单应用来说，这是一个不错的选择。这种数据源的性能表现依赖于使用的数据库，对某些数据库来说，使用连接池与否并不重要。UNPOOLED 数据源需要配置以下 6 种属性：

| 属性                               | 说明                                   |
| ---------------------------------- | -------------------------------------- |
| `driver`                           | JDBC 驱动的 Java 类全限定名            |
| `url`                              | 数据库 JDBC URL 地址                   |
| `username`                         | 登录用户                               |
| `password`                         | 登录密码                               |
| `defaultTransactionIsolationLevel` | 默认的连接事务隔离级别                 |
| `defaultNetworkTimeout`            | 等待数据库返回的默认超时实际，单位毫秒 |

**POOLED** 数据源管理一个数据库连接池，节省反复创建连接的开销和耗时，可以快速响应请求。POOLED 数据源需要配置所有 UNPOOLED 数据源的属性，另外，还有以下池相关的属性需要配置：

| 属性                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `poolMaximumActiveConnections`           | 最大活动连接数，默认 10。                                    |
| `poolMaximumIdleConnections`             | 最大空闲连接数。                                             |
| `poolMaximumCheckoutTime`                | 强制返回前池中连接被检出时间，默认 20000 毫秒。              |
| `poolTimeToWait`                         | 获取连接等待超时，默认 20000 毫秒，超时将会打印状态日志并重新尝试。 |
| `poolMaximumLocalBadConnectionTolerance` | 某个线程如果获取到一个坏连接，数据源允许它重试，次数不超过 `poolMaximumLocalBadConnectionTolerance` 与 `poolMaximumIdleConnections` 之和，默认 3。 |

**JNDI** 数据源，这个实现是为了能在 EJB 或应用服务器这类容器中使用，暂不赘述。

## 数据库厂商标识 DatabaseIdProvider

MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持基于映射语句 `databaseId` 属性。默认加载带有匹配当前数据库 `databaseId` 属性和所有不带 `databaseId` 属性的语句。如果两条语句相同，将会舍弃不带 `databaseId` 属性的那条。

添加 `databaseIdProvider` 元素，开启多厂商特性。

```
<databaseIdProvider type="DB_VENDOR" />
```

MyBatis 默认会把 `databaseId` 设为 `DatabaseMetaData#getDatabaseProductName()` 返回，通常这些字符串都非常长，而且相同产品的不同版本的返回也可能不同，可用 property 子标签为厂商设置别名。

```
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

提供别名之后，`MyBatis` 会把 `databaseId` 设为上面子标签中第一个 `name` 匹配的值，如果没有匹配属性，那就设为 `null`。这里例子，如果 `getDatabaseProductName()` 返回 `Oracle`，它的 `databaseId` 就是 `oracle`。

## 映射器 Mappers

指定映射文件的位置，可以使用相对于类路径的资源引用，或者完全限定资源定位符，或者类名和包名。

```
<mappers>
  <!-- 相对于类路径的资源引用 -->
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <!-- 完全限定资源定位符 -->
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <!-- 映射器接口实现类的完全限定类名 -->
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <!-- 注册包内的全部映射接口为映射器 -->
  <package name="org.mybatis.builder"/>
</mappers>
```

默认情况，映射接口和映射文件应该在同一个包，可以分别指定它们的位置以绑定文件到不同包的接口。

```
<mappers>
  <package name="org.mybatis.builder"/> <!-- 接口 -->
  <mapper resource="mapper/*.xml"/> <!-- 文件 -->
</mappers>
```

# XML 映射

## 查询语句

定义一个简单 select 语句，接收一个 `int` 或 `Integer` 参数，返回一个 `HashMap`  对象。`#{id}` 表示设置预处理语句的参数。任何结果集都可以用 `Map` 接收，如果只有一条记录，键是列名，值是列值；如果有多条记录，键是标识符属性，值是对象，对象类型需要指定，标识符属性可在接口方法用 `@MapKey` 指定。

```
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

以下是一些 select 标签的属性。

| 属性            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `id`            | 标识符，应该与映射接口的绑定方法同名。                       |
| `parameterType` | 参数类型，可选，MyBatis 能够自动识别。                       |
| `resultType`    | 结果集的每条记录的映射类型。                                 |
| `resultMap`     | 引用自定义的结果映射，与 `resultType` 互斥。                 |
| `useCache`      | 每次都把执行结果放到二级缓存，Select 默认 `true`。           |
| `flushCache`    | 每次被调用都会清空本地缓存和二级缓存，默认 `false`。         |
| `timeout`       | 等待数据库返回的最长时间，单位秒。                           |
| `statementType` | 可选 `STATEMENT`，`PREPARED` 或 `CALLABLE`，默认使用预编译。 |

## 其它语句

没什么特别，参考示例即可。

```
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>
```

```
<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>
```

```
<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

对于 insert 和 updat 操作，如果想要获取主键的值，可以使用以下属性。

| 属性               | 描述                                           |
| :----------------- | :--------------------------------------------- |
| `useGeneratedKeys` | 获取数据库生成的主键并回设对象，默认 `false`。 |
| `keyProperty`      | 指定映射主键的属性，多个属性使用逗号分隔。     |
| `keyColumn`        | 指定表示主键的列名，多个列使用逗号分隔，可选。 |

## 存储过程

使用 select 标签，设置 `statementType="CALLABLE"`，用 `{}` 包裹调用语句，OUT 参数会被返回结果覆盖。

```
<select id="callProcedure" statementType="CALLABLE">
	{ call procedure_name(
		#{param1,mode=IN,jdbcType=INTEGER},
		#{param2,mode=IN,jdbyTtype=INTEGER},
		#{param3,mode=OUT,jdbyType=CURSOR,javaType=ResultSet,resultMap=param3map},
		...
	)}
</select>
```

## 参数设置

**预编译 #{}**

如果只有一个简单类型参数，可以使用任意名字引用。如果有多个简单类型参数，应该先用 `@Param` 在接口方法命名各个参数，避免潜在问题。

```
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```

如果参数是一个对象，可以直接使用属性名引用，非常方便。

```
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```

参数还能指定数据类型，虽然大多时候 MyBatis 能够自动识别。如果参数是 `HashMap`，需要指定 `javaType` 确保使用正确的类型处理器。如果字段允许为 `null`，并且参数也是 `null`，需要指定 `jdbcType`。

```
#{name,javaType=int,jdbcType=NUMERIC}
```

**硬拼接 ${}**

使用 `#{}` 填充 SQL 语句，其实是向 `PreparedStatement` 设置参数，这样做更安全、更迅速，但是只能用于设置查询字段、更新字段。如果想填充其它内容，比如表名、Group By字段、Order By 字段，只能使用字符串拼接。

```
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```

这种方式不安全，有 SQL 注入风险。

## 结果映射

### 自动映射

默认情况，如果映射类型是对象，MyBatis 将根据列名和属性名把结果集映射成 Java 对象，忽略大小写。数据库列名通常是单词加下划线，而 Java 属性更习惯驼峰命名，设置 `mapUnderscoreToCamelCase=true` 可以启用这两种命名风格之间的自动映射。

自动映射有三种级别：

* `NONE`：禁止自动映射；
* `FULL`：映射所有属性，类似 `id` 这种字段会被嵌套对象重复映射，明显不对。
* `PARTIAL`：默认，只对嵌套对象之外的属性进行映射。

结果映射可以和自动映射混用，通过 `autoMapping` 属性开启或禁止自动映射，默认开启。这时，没被结果映射处理的列会被自动映射。

```
<resultMap id="userResultMap" type="User" autoMapping="false">
  <result property="password" column="hashed_password"/>
</resultMap>
```

### 结果映射

`resultMap` 标签用于自定义结果集映射规则，标签属性 `id`、`type`、`autoMapping` 指定标识符、映射类型和自动映射。映射的具体细节使用各种子标签描述。

**ID & Result**

`id` 和 `result` 标签使用 Setter 方式把列值映射为 Java 属性，标签属性 `property` 和 `column` 指定匹配的属性名和列名。

```
<id property="id" column="post_id"/>
```

```
<result property="subject" column="post_subject"/>
```

`id` 标签对应的属性会被 Mybatis 标记为对象标识符，用于对象比较，正确使用可以提高整体性能。

**Constructor**

如果映射结果是不可变对象，那就只能使用构造器的方式来设置属性，标签属性 `name`、`column` 指定匹配的属性名和列名，`idArg` 标签对应的属性会被标记为对象标识符。

```
<constructor>
   <idArg column="id" javaType="int" name="id" />
   <arg column="age" javaType="_int" name="age" />
   <arg column="username" javaType="String" name="username" />
</constructor>
```

### 关联对象

如果属性是复杂对象，使用 `association` 标签进行关联映射，参考示例进行简单应用。

**嵌套定义映射**

```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
  </association>
</resultMap>
```

**引用结果映射**

```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author" resultMap="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```

**引用查询语句**

关联对象是某个查询的返回，结果集包含查询所需的参数。`association` 标签的属性 `column` 指定传递哪些结果集的列给查询语句，`fetchType` 设置延迟加载关联对象，值为 `lazy` 或 `eager`。

```
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" select="selectAuthor"/>
</resultMap>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>
```

### 集合属性

如果属性是一个集合，使用 `collection` 标签进行映射，参考示例进行简单应用。

**嵌套定义映射**

`collection` 标签的属性 `ofType` 指定集合元素的类型。

```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
```

**引用结果映射**

```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post" resultMap="blogPostResult"/>
</resultMap>

<resultMap id="blogPostResult" type="Post">
  <id property="id" column="id"/>
  <result property="subject" column="subject"/>
  <result property="body" column="body"/>
</resultMap>
```

**引用查询语句**

```
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" 
  					column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>

<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>
```

## 缓存设置

MyBatis 拥有事务性的查询缓存机制，用户可以进行相关配置。

### 本地缓存，一级缓存

每次新建一个会话，MyBatis 就会为其创建一个关联的本地缓存，会话执行任何查询，返回的结果都会被放入关联的本地缓存。修改语句、提交/回滚事务、关闭会话，都会导致缓存清空。

通过 MyBatis 配置 `localCacheScope`，设置本地缓存的作用域：

* `SESSION`：默认，缓存对整个会话有效。
* `STATEMENT`：缓存只对当前执行的 SQL 语句生效，执行完后清空缓存，相当于禁用缓存。

### 全局缓存，二级缓存

二级缓存（second level cache）也称全局缓存，是在 namespace 级别缓存数据。每个会话关闭前都会把本地缓存的数据放入对应的全局缓存。执行查询语句，首先查看全局缓存，然后查看本地缓存，最后才查库。

修改 Myabatis 配置，允许全局缓存。

````
<setting name="cacheEnabled" value="true"/>
````

映射文件添加 cache 标签，启用当前名称空间的全局缓存。

```
<cache/>
```

全局缓存的默认策略：

* 缓存所有 select 语句的结果；
* 任何 insert、update 和 delete 语句都会清空缓存；
* 使用最近最少算法，清除多余的缓存；
* 不会定时清空缓存，即没有刷新间隔；
* 最多保存 1024 个对象的引用；
* 读写缓存，也就是说，返回的缓存对象并不是共享内容，可以修改，不会影响其它线程。

使用 cache 标签的属性自定义缓存策略。

```
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

映射语句的 `useCache` 和 `flushCache` 属性，也会覆盖名称空间的缓存设置。另外，还能使用 cache-ref 标签引用其它名称空间的全局缓存。

# 动态 SQL

动态 SQL 指能根据不同条件拼接 SQL 语句，这样可以节省代码，MyBatis 3 主要使用 OGNL 表达式条件判断。

## 条件插入 IF

根据条件选择是否插入语句，可以嵌套使用。

```
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

## 分支插入 Choose

类似 Java 的 Switch 语句，根据条件从多个语句选择一个插入，可以设置默认语句。

```
<select id="findActiveBlogLike" resultType="Blog">
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

## 动态条件 Where

**where**

动态设置条件语句。包含多个条件标签，如果子标签返回内容，将会自动在开头添加 WHERE 关键字，同时删除返回内容的头部的 AND 或 OR 关键字。

```
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
  </where>
</select>
```

**set**

动态设置赋值字段。包含多个条件标签，如果子标签返回内容，将会自动在开头添加 SET 关键字，同时删除返回内容的尾部的逗号。

```
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

**trim**

自定义子标签有内容时，前缀、后缀的处理方式，使用以下属性：

* prefix：添加前缀，仅当子标签有内容。
* prefixOverrides：删除子标签返回内容的指定前缀。
* suffix：添加后缀，仅当子标签有内容。
* suffixOverrides：删除子标签返回内容的指定后缀。

与 where 标签等价的 trim 标签。

```
<trim prefix="WHERE" prefixOverrides="AND|OR">
  ...
</trim>
```

与 set 标签等价的 trim 标签。

```
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

## 集合遍历 foreach

遍历参数指向的集合。属性 `item` 表示集合元素，`index` 表示元素索引，`open` 表示开始符号，`close` 表示结尾符号，`separator` 表示元素分隔符。对于 `Map` 集合，`index` 表示键，`item` 表示值。

```
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list" open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

## 注解定义动态 SQL

如果要使用注解定义动态 SQL，需要使用 script 标签，很少应用。

```
@Update(
  {"<script>",
    "update Author",
    "  <set>",
    "    <if test='username != null'>username=#{username},</if>",
    "    <if test='password != null'>password=#{password},</if>",
    "    <if test='email != null'>email=#{email},</if>",
    "    <if test='bio != null'>bio=#{bio}</if>",
    "  </set>",
    "where id=#{id}",
  "</script>"})
void updateAuthorValues(Author author);
```