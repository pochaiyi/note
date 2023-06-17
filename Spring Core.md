# Spring Framework

> version 5.3.19

Spring Framework 包含多个独立的模块，每个模块都可以单独存在，或与其它模块联合使用。其中，IoC 容器和 AOP 面向切面编程是 Spring Framework 最基本、最核心的两个模块。

# 对象容器 Core Container

## IoC Container 的概念

Inversion of Control（IoC）控制反转的意思是把对象的控制权从代码移交给专门的容器。类似于池的思想，比如把线程的创建、运行、销毁都交给线程池管理。

传统编程中，程序需要什么对象，就原地 new 创建。这种方式简单直接，但如果项目的对象非常多，且存在复杂的关联关系，那这些对象的管理将困难重重，极易发生错误和内存溢出。

IoC 思想：首先编写配置信息，明确定义各种对象及它们的关联关系。Spring 读入这些数据后，为每个对象的定义信息创建 `BeanDefinition` 对象，对象工厂 `BeanFactory` 再使用 `BeanDefinition` 创建对象并维护。所以，IoC 也称为 IoC Container，程序运行时可以从 IoC 容器获取对象，而不用自己创建。

`ApplicationContext` 是 Spring 的核心，也是 `BeanFactory` 的子接口。它除实现最基本的容器外，还拓展有许多其它的功能。简单理解，它就是 Spring 暴露出来的整体接口。

## Bean 和 BeanDefinition

在 Spring 中，由 IoC 容器管理的对象称为 Spring Bean。所谓管理，就是对象的创建、装配、代理、销毁等，而如何进行管理，由加载的配置信息决定。

`BeanDefinition` 是配置信息的抽象，Spring 读入配置源，然后创建若干个 `BeanDefinition` 对象，它是 Spring Bean 的模板，主要包含以下信息：

- 全限定类名：通常就是 bean 的类型，工厂类除外。
- 相关属性，定义 bean 在容器内的行为，比如作用域。
- 依赖关系。
- 新建实例的初始值，使用构造器或 `setter` 方法进行设置。

`BeanFactory` 提供接口，允许直接注册现有对象。这种方式非常不规范，项目中常被禁止。

```
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
beanFactory.registerSingleton("beanName", new Object());
```

## Bean 的标识符

每个 bean 都有一个或多个标识符，这些标识符在容器内必须唯一。一个 bean 通常只有一个标识符，如果需要多个，一个之外的标识将被视为别名。

**Java 配置**

`@Component` 组件的标识符默认是首字母小写的类名，可以使用 `value` 属性自定义。

`@Bean` 方法注册的 bean，标识符默认为方法名，可以使用 `name` 属性自定义。`name` 可以接收多个值。

**XML 配置**

`<bean/>` 的标识符默认根据 bean 的全限定类名生成，可以使用 `id` 属性设置标识符，`name` 属性设置别名。别名可以有多个，使用 `,` 或 `;` 或空格分隔。

## Bean 作用域 Scope

bean 的作用域是指 bean 的有效范围，由 IoC 容器进行管理。作用域是 bean definition 的属性，与具体实例无关。Spring 支持 6 种作用域，其中 4 种只在 web 环境中有效。

**Java 配置**

对于 `@Bean` 方法和 `@Component` 组件，可以使用 `@Scope` 注解定义其作用域。

```
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Encryptor encryptor() {...}
```

**XML 配置**

对于 XML 的 `<bean/>` 标签，可以使用 `scope` 属性定义作用域。

```
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### 单例 singleton

单例是 bean 的默认作用域。容器在初始化时会为所有 singleton bean 创建实例（lazy-init 除外），并缓存起来。之后在容器范围内，凡是访问这个 bean 的请求，获取的都是同一个对象。

### 多例 prototype

对于 prototype bean，每次访问它，获取的都是一个新的实例。

与其它 scope 不同，容器不管理 prototype bean 的生命周期。容器在实例化、初始化、装配 prototype bean 后就直接把它交出去，不进行追踪。因此，只有 prototype bean 不会执行销毁回调（因为容器根本不管理它后面的生命周期）。如果想清除 prototype bean 占用的资源，可以通过 bean post-processor 实现，因为它持有需要被清除的对象的引用。

### web scope

`request`、`session`、`application` 和 `websocket` 4 个作用域只在 web 容器内有效，如果试图把它们用在普通的 IoC 容器，会引发 `IllegalStateException` 异常。

**request**

对每个 HTTP 请求，容器都会为其创建所有 request bean 的新实例，请求结束时将清除这些 request bean。

```
@RequestScope
@Component
public class LoginAction {...}
```

**session**

与 request 相似，只不过它的有效范围是在 session 对象。

```
@SessionScope
@Component
public class UserPreferences {...}
```

**application** 

与 singleton bean 不同的是：application bean 针对于 ServletContext 而非 ApplicationContext。

```
@ApplicationScope
@Component
public class AppPreferences {...}
```

**websocket**

Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`.

## 不同作用域的依赖

当 singleton bean 依赖 singleton bean，或 prototype bean 依赖  singleton bean 时，直接注入依赖即可。但如果是 singleton bean 依赖 prototype bean，或 prototype bean 依赖 prototype bean，那每次获取的 bean，它的 prototype 依赖都应该是新创建的实例，此时普通的注入方式不再适用。

### 代理对象注入

动态代理被依赖的 prototype bean，此时，真正被注入的是代理对象。之后程序每次获取外部 bean，代理都会为这个外部 bean 创建新的 prototype 依赖。

**单件设置**

可以使用 `@Scope` 的属性 `proxyMode` 为组件或 `@Bean` 方法设置动态代理注入，有 3 个可选值。

| value                          | description               |
| ------------------------------ | ------------------------- |
| `ScopedProxyMode.NO`           | 默认，不代理              |
| `ScopedProxyMode.TARGET_CLASS` | 基于继承的 CGLIB 动态代理 |
| `ScopedProxyMode.INTERFACES`   | 基于接口的 JDK 动态代理   |

```
@Component
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class LoginAction {...}
```

**批量设置**

还可以使用组件扫描 `@ComponentScan` 的属性 `scopedProxy` 进行批量设置，它会为所有被扫描到的组件的 prototype 依赖进行代理。`scopedProxy` 属性的可选值与 `@Scope` 的 `proxyMode` 属性相同。

## 懒加载 lazy-init

容器在实例化时会创建所有 singleton bean 和配置为预加载的 bean。懒加载设置将使这些 bean 在首次被访问时才进行创建。

**Java 配置**

对于 `@Bean` 方法和 `@Component` 组件，使用 `@Lazy` 设置懒加载。

```
@Bean
@Lazy
public Encryptor encryptor() {...}
```

**XML 配置**

`<bean/>` 的 `lazy-init` 属性用于设置懒加载，默认为 `false`：

```
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
```

`<beans/>` 的 `default-lazy-init` 属性用于设置默认的懒加载行为：

```
<beans default-lazy-init="true">    <!-- no beans will be pre-instantiated... --></beans>
```

## 间接依赖 dpends-on

如果想让不存在依赖关系的 bean 在某个 bean 之前创建，可以使用 dpends-on 设置。

**Java 配置**

对于 `@Bean` 方法和 `@Component` 组件，使用 `@DependsOn` 设置间接依赖。它的 `value` 属性接收字符串数组，这些字符串就是其间接依赖的 bean 的标识符。

```
@DependsOn("a", "b")
@Component
public class AppPreferences {...}
```

**XML 配置**

如果间接依赖多个 bean，使用 `,` 或 `;` 或空格分隔标识符。

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao"/>
<bean id="manager" class="ManagerBean" />
```

## 基于 XML 的配置信息

### XML 配置信息

以下是基于 XML 的配置信息的基本结构：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

`id` 是一个字符串，用于标识 `BeanDefinition`。

`class` 使用全限定类名指定 bean 的类型。

### 多个 XML 文件

可以把 bean definition 分散写在多个 XML 文件，这有利于分层管理。有两种实现方式：

* 在 `ApplicationContext` 的构造器中传入这些配置文件的 Resource 位置。
* 在 XML 文件中使用 `<import/>` 标签引入其它 XML 文件，如下所示：

```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

</beans>
```

`resource` 是被导入文件的相对路径，因此 services.xml 应该与当前文件位于相同目录。前导斜杠会被忽略，鉴于路径是相对的，最好不使用斜杠，避免误会。

也可以使用绝对路径，比如 `file:C:/config/services.xml`、`classpath:/config/services.xml`。

### XML 定义 BeanDefinition

在 XML 文件，`<bean/>` 标签用于定义 bean definition，它有两种使用方式。

**方式一：**直接定义，`class` 属性指定 bean 的类型。这种方式 Spring 是使用反射创建对象，bean 可能需要有默认构造器。

```
<bean id="exampleBean" class="examples.ExampleBean"/>
```

**方式二：**使用工厂方法创建 bean，返回对象通常是其它类型，不探讨。

> **定义内部类的 bean**
>
> 此时对于 `class` 属性，使用 `$` 或 `.` 分隔外部类和内部类，比如 `com.SomeThing$OtherThing`。

### 构造器注入

构造器 DI 通过构造器的参数，把依赖注入 bean。`<bean/>` 的子标签 `<constructor-arg/>` 用于设置构造器参数。

普通情况，`<constructor-arg/>` 的顺序应该与构造参数的顺序一致：

```
<beans>
    <bean id="thingOne" class="x.y.ThingOne">
        <constructor-arg ref="thingTwo"/>
        <constructor-arg ref="thingThree"/>
    </bean>

    <bean id="thingTwo" class="x.y.ThingTwo"/>
    <bean id="thingThree" class="x.y.ThingThree"/>
</beans>
```

可以使用 `type` 属性指定参数类型，IoC 容器会自动匹配简单类型的参数：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

也可以使用 `index` 属性指定参数位序，基于 0：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

但最常用的还是使用 `name` 属性指定形参名：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

注意，如果想用 `name` 属性进行参数匹配，编译代码时必须启动调试，以便 Spring 可以获取构造器的形参名。或者，使用 `@ConstructorProperties` 标注构造方法并指定形参名列表：

```
@ConstructorProperties({"years", "ultimateAnswer"})
public ExampleBean(int years, String ultimateAnswer) {
    this.years = years;
    this.ultimateAnswer = ultimateAnswer;
}
```

### setter 注入

setter DI 是在 IoC 容器通过反射创建 bean 后，调用 bean 的 setter 方法注入依赖。sette DI 与构造器 DI 可以混合使用。

`<bean/>` 的子标签 `<property/>` 用于向 bean 注入属性，它的 `name` 属性指定注入的属性名。

#### 注入值

`<property/>` 的 `value` 属性用于注入简单值：

```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
</bean>
```

#### 注入引用

`<property/>` 的 `ref` 属性用于注入 bean，它的值是 bean 的标识符：

```
<property name="beanTwo" ref="yetAnotherBean"/>
```

#### 注入集合

`<list/>`，`<set/>`，`<map/>` 和 `<props/>` 都是 `<property/>` 的子标签，分别用于注入 `List`，`Set`，`Map` 和 `Properties` 类型的数据。

```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">[emailprotected]</prop>
            <prop key="support">[emailprotected]</prop>
            <prop key="development">[emailprotected]</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

`map` 的键和值、`set` 的值只能是以下类型：

```
bean | ref | idref | list | set | map | props | value | null
```

#### 注入空串和 null

`value` 的空值会被视为空字符串，下例会把 `email` 设置为空字符串：

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

`<null/>` 标签用于注入 `null` 值：

```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

### 自动装配

前面的构造器注入和 setter 注入，都需要使用 `ref` 显式指定依赖项。IoC 容器提供自动装配 bean 的功能，它根据属性的类型或名称，在容器查找匹配的 bean，然后将其注入。

`<bean/>` 的 `autowire` 属性用于设置自动装配模式，有 4 个选项：

| Mode          | Explanation                                                  |
| ------------- | ------------------------------------------------------------ |
| `no`          | 默认，不使用自动装配，依赖项必须由 `ref` 属性显式指定。      |
| `byName`      | 根据属性名装配。容器查找标识符与属性名相同的 bean，然后将其注入。 |
| `byType`      | 根据属性类型装配。容器查找类型与属性相同的 bean，然后将其注入。如果存在多个匹配项，则抛出异常。如果没有匹配项，则什么也不发生。 |
| `constructor` | 类似于 `byType`，但只适用于构造器参数。此时如果容器不存在类型匹配的 bean，会引发错误。 |

`byType` 和 `constructor` 两种模式都可以装配数组和集合，这会装配容器内所有类型匹配的 bean。如果是 `map` 类型且 key 为 `String`，IoC 容器会装配所有类型与 value 匹配的 bean，key 的值是 bean 的标识符。

> **禁止被自动装配**
>
> `<bean/>` 的 `autowire-candidate` 属性指定是否允许当前 bean 作为自动装配的候选项。如果为 `false`，表示当前 bean 不允许被其它 bean 自动装配。

### XML 源的容器实例化

`ApplicationContext` 有许多实现，`ClassPathXmlApplicationContext` 是专门用于加载类路径下 XML 配置信息的实现。`ApplicationContext::getBean(String, Class<T>)` 方法根据标识符从容器获取对象，第二个参数指定对象类型。

```
// create and configure beans
ApplicationContext context = 
					new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);
```

## 基于注解的自动装配

可以在 bean 类层面使用注解配置依赖关系，这能大量减少 XML 数据。

### XML 启动注解支持

要使注解配置在 XML 生效，需要注册大量的 `BeanPostProcessor`，Spring 提供一个简单标签用于批量注册这些与注解相关的处理器。

```
<context:annotation-config/>
```

这个标签会使 IoC 容器隐式注册以下处理器。

- `configurationClassPostProcessor`
- `AutowiredAnnotationBeanPostProcessor`
- `CommonAnnotationBeanPostProcessor`
- `PersistenceAnnotationBeanPostProcessor`
- `EventListenerMethodProcessor`

此时，XML 对已使用注解配置依赖的 bean，只需使用 `<bean/>` 标签注册即可。

### ByType @Autowired

`@Autowired` 是 byType 的自动装配，它可以标注 bean 类的属性、方法、构造器。

**构造器、setter 方法、带参方法**

`@Autowired` 标注构造器，则 IoC 容器将使用这个构造器创建实例，同时进行构造器注入。

> 从 Spring Framework 4.3 开始，如果 bean 类只有一个构造器，则不需要再显式标注 `@Autowired`，IoC 容器默认会使用它创建实例。但如果有多个构造器，且没有设置 `primary`/`default`，那必须使用 `@Autowired` 标注至少一个构造器。

`@Autowired` 标注 setter 方法，或有参数的任意方法，IoC 容器会在 bean 实例化后调用这些方法进行依赖注入。这些方法可以什么都不做，但必须有参数。

```
@Autowired
public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
	this.customerPreferenceDao = customerPreferenceDao;
}
```

**属性**

`@Autowired` 标注 bean 类的属性，这是最常用的注入方式。它的底层实现是在 bean 的后置处理器中使用反射注入属性。

```
@Autowired
private MovieCatalog movieCatalog;
```

**数组、集合**

`@Autowired` 标注数组、集合类型的属性，IoC 容器会注入所有类型匹配的 bean。

> 如果想控制这些 bean 的注入顺序，可以让 bean 类实现 `org.springframework.core.Ordered` 接口，或标注 `@Order` 或 `@Priority`。否则，它们的注入顺序将与 bean definition 的注册顺序一致。
>
> `@Order` 可以标注 bean 类和 `@Bean` 方法，它只影响注入顺序，与 singleton bean 的启动顺序没有任何关系，这是由依赖关系和 `@DependsOn` 所决定的。
>
> `@Primary` 只能标注 bean 类，可以使用 `@Order` 和 `@Primary` 的组合来替代它。

如果标注的是 `Map` 属性，且 `key` 为 `String` 类型，IoC 容器会注入所有类型与 `value` 匹配的 bean，key 的值为 bean 的标识符。

```
@Autowired
private MovieCatalog[] movieCatalogs;

@Autowired
public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
	this.movieCatalogs = movieCatalogs;
}
```

**Required**

`@Autowired` 根据 byType 从容器搜索 bean。默认情况，`@Autowired` 装配的 bean 是必须的，即如果在容器内找不到匹配的 bean，则会报错。对于数组、集合，至少要装配一个 bean。

可以修改 `@Autowired` 的 `required` 属性为 `false`，使 IoC 容器跳过 `@Autowired` 不满足的注入点。此时，在不满足的注入点，`@Autowired` 方法不会被调用，`@Autowired` 属性使用默认值。

在一个 bean 类，至多有一个构造器能被声明为 `required`，此时 IoC 容器使用它进行实例化，如果依赖项不满足，这个 bean 甚至不会被创建。如果有多个构造器被  `@Autowired` 标注且都申明为 `no-required`，IoC 容器会从中选则依赖项满足最多的一个用来实例化。如果没有 `@Autowired` 构造器，将调用 `primary`/`default` 构造器（如果有）进行实例化。如果只有一个构造器，那什么都不用做，默认使用它实例化。

> 如果 bean 类只有单个构造器，IoC 容器在它的多元素注入点（数组、集合）会有不同的策略。此时，如果这些点没有满足的依赖项，会为它们注入空实例。

```
@Autowired(required = false)
public void setMovieFinder(MovieFinder movieFinder) {
	this.movieFinder = movieFinder;
}
```

除设置 `required=false` 外，还可以使用 `java.util.Optional` 或 `@Nullable` 来表示依赖的非必须性。这里的 `@Nullable` 可以是任何包的任何类，比如 JSR-305 的 `javax.annotation.Nullable`。

```
@Autowired
public void setMovieFinder(Optional<MovieFinder> movieFinder) {...}

@Autowired
public void setMovieFinder(@Nullable MovieFinder movieMaker) {...}
```

**特殊接口的装配**

可以用 `@Autowired` 标注 Spring 内众所周知的接口类型，比如 `BeanFactory`、`ApplicationContext`、`Environment`。这些接口及其拓展将被自动解析而无需特别设置，下例就使用这种方式获得容器的引用。

```
@Autowired
private ApplicationContext context;
```

### 优先 @Primary

当 `@Autowired` 装配时，如果有多个类型满足的 bean，可以使用 `@Primary` 标注优先选择的 bean。下例情况，装配 `MovieCatalog` 类型时会优先选择 `firstMovieCatalog`。如果多个候选 bean 没有优先项，会发生异常。

```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() {...}

    @Bean
    public MovieCatalog secondMovieCatalog() {...}

    // ...
}
```

### 限定 @Qualifiers

`@Qualifier` 用于配合 `Autowired` 更加精确地从多个类型满足的候选项中选择目标 bean。正如其名字所示，它把限定值 qualifier values 与特定参数绑定，用于筛选候选项。

`@Qualifier` 可以标注属性、方法参数、构造器参数。它的 `value` 属性默认与 bean 的标识符绑定，也就是说，它会从类型匹配的候选项中，选择标识符与 `value` 值相同的 bean。

```
@Autowired
@Qualifier("main")
private MovieCatalog movieCatalog;
```

需要注意，`@Qualifiers` 是对已通过类型匹配的 bean 进行筛选，而不是根据 byName 去查找 bean。

> `@Autowired` 本身也具有限定匹配的能力：如果有多个候选项，且没有 `@Primary` 和 `@Qualifier` 的干扰，IoC 容器会尝试选择标识符与属性名、参数名相同的 bean，如果不存在匹配项，且这个依赖 `required`，那就要报错。

> **@Autowired 的泛型限定**
>
> 泛型可以作为隐含的限定条件。匹配时，IoC 容器会选择类型参数相同的 bean。

### ByName @Resource

`@Resource` 是 byName 的自动装配，它可以标注 bean 类的属性、单个参数的 setter 方法。IoC 容器会查找标识符与 `@Resource` 的 `name` 属性相等的 bean，然后将其注入。

如果 `@Resource` 没有显式设置 `name`，IoC 容器默认使用属性名或参数名进行查找。如果默认 byName 没有匹配结果，则转而尝试 byType 装配。

> 从效果看，`@Resource` 等价于 `@Autowired` + `@Qualifier`。但它们的实现原理不同，`@Resource` 是直接根据标识符查找，而后者先类型匹配，然后使用标识符进行筛选。

> 根据 `name` 从 IoC 容器查找 bean 的工作由 `CommonAnnotationBeanPostProcessor` 完成。

```
@Resource(name="myMovieFinder") 
public void setMovieFinder(MovieFinder movieFinder) {
	this.movieFinder = movieFinder;
}

@Resource
public void setMovieFinder(MovieFinder movieFinder) {
	this.movieFinder = movieFinder;
}
```

### 字面量值 @Value

`@Value` 用于注入字面量值，它可以标注 bean 类的属性和方法参数。

**直接注入**

IoC 容器会把 `@Value` 的 `value` 值作为属性或参数的值进行注入。

```
@Value(value = "this is filename")
private final String filename;
```

**外部属性**

IoC 容器根据 `@Value("${key}")` 句式指定的 key 从上下文的 properties 查找 value 进行注入。

```
public MovieRecommender(@Value("${catalog.name}") String catalog) {
	this.catalog = catalog;
}
```

> Spring 内置一个宽松的值解析器，它会尝试解析 key，如果失败，则直接把属性名或参数名作为值注入。

**数组**

Spring 内置的转换器支持简单的类型转换，使用逗号分隔的多个值会被转换为数组。

**默认值**

`@Value("${key:default}")` 句式会在分号 `:` 前的 key 解析失败时，使用分号后的默认值进行注入。

```
public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
	this.catalog = catalog;
}
```

## 类路径组件扫描

前面使用注解配置依赖关系已经节省很多 XML 配置，但依旧存在大量的 `<bean/>` 标签。可以考虑使用模板注解，它能替代 `<bean/>` 标签定义 bean definition。

### 模板注解和组件

**模板注解 Stereotype Annotation**

模板注解 Stereotype Annotation 用于标识 bean 类。这些类如果被组件扫描到，将会在 IoC 容器注册对应的 bean definition。现在，组件特指被 `@Component` 等模板注解标注的 bean 类。

`@Component` 是最基础的模板注解，在它的基础上，拓展有 `@Service`、`@Controller`、`@Repository` 等注解，它们的效果没有区别，但可以区分不同分层的组件。而且，它们在未来可能会增加额外不同的功能。

**组件内的 @Bean 方法**

`@Component` 组件内也可以定义 `@Bean` 方法，这样，一个组件类就能提供多个 bean definition。

`@Component` `@Bean` 方法与 `@Configuration` `@Bean` 方法的不同之处在于：CGLIB 不会拦截 `@Component` 类的方法调用，它们只能是 `lite` 模式。

### 组件扫描

使用模板注解定义组件后，还需要开启扫描，批量注册组件到 IoC 容器。相比如 `@Bean` 方法，这种方式更加简便。

#### 开启组件扫描

组件扫描，就是让 IoC 容器读取指定位置的类文件，检测组件并注册为 bean definition。

在 `@Configuration` 类，可以使用 `@ComponentScan` 开启组件扫描，它的 `basePackages` 和 `value` 属性用于指定需要扫描的类路径，多个路径使用 `,` 或 `;` 分割。

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {...}
```

以下是等价的 XML 配置：

```
<context:component-scan base-package="org.example"/>
```

> `<context:component-scan>` 隐式启动 `<context:annotation-config>`，后者会批量注册一堆与注解相关的后置处理器。如果不想自动注册这些处理器，可以设置属性 `annotation-config=false`。

#### 自定义扫描规则

组件扫描的默认规则是：扫描所有 `@Component` 组件。可以自定义过滤器来修改扫描规则，把它们添加到 `@ComponentScan` 的两个属性 `includeFilters`、`excludeFilters`。

> 可以设置 `@ComponentScan` 的属性 `useDefaultFilters=false` 来禁用默认过滤器，这样就不会再扫描  `@Component` 注解标注的组件。

每个过滤器都要设置 `type` 和 `expression` 这两个属性，下表描述这些过滤选项。

| Filter Type          | Example Expression           | Description                                                  |
| :------------------- | :--------------------------- | :----------------------------------------------------------- |
| annotation (default) | `org.example.SomeAnnotation` | An annotation to be *present* or *meta-present* at the type level in target components. |
| assignable           | `org.example.SomeClass`      | A class (or interface) that the target components are assignable to (extend or implement). |
| aspectj              | `org.example..*Service+`     | An AspectJ type expression to be matched by the target components. |
| regex                | `org\.example\.Default.*`    | A regex expression to be matched by the target components' class names. |
| custom               | `org.example.MyTypeFilter`   | A custom implementation of the `org.springframework.core.type.TypeFilter` interface. |

下例扫描 `org.example` 包中所有非 `@Repository` 并且类名符合正则表达式的组件：

```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```

### **元注解和混合注解**

Spring 提供的许多注解如 `@Component`、`@Service`、`@Controller` 等都可以作为元注解。像 `@Service`，它本身就被 `@Component` 标注。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {...}
```

这些注解可以自由组合，比如 Spring MVC 的 `@RestController` 就是由 `@Controller` 和 `@ResponseBody` 混合而成。

混合注解可以选择性地覆盖元注解的属性，比如 Spring 的 `@SessionScope` 便是把作用域名硬编码为 `session`，但仍允许自定义 `proxyMode` 属性。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {...}
```

## 基于 Java 的配置信息

现在，考虑使用 Java 配置完全替代 XML 文件。

### XML 和注解

XML 文件是最原始的配置信息，它把所有配置都集中在一个或几个 XML 文件，每个 XML 文件的内容非常多，很难编写，可读性也很低。所以，现在主要使用注解进行配置，主力框架 Spring Boot 也是完全基于注解的。当然，注解和 XML 并不冲突，可以在注解引入 XML 文件，也可以在 XML 启用注解信息。

### 配置类 @Configuration

`@Configuration` 标注类称为配置类，它标示这个类是 bean definition 的源。可以把配置类与 XML 文件等价，它能定义许多容器属性。而且 `@Configuration` 被 `@Component` 标注，所以配置类本身也是组件。

配置类中可以声明 `@Bean` 方法，这种方法其实就是工厂方法，它的返回值会被注册到 IoC 容器。

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### 工厂方法 @Bean

`@Bean` 用于标注工厂方法，它的返回值会被注册到 IoC 容器。`@Bean` 方法主要声明在配置类，也可以在普通组件，甚至普通类。

**返回类型**

`@Bean` 方法的返回类型就是 bean 的类型，也可以使用接口作为返回类型，但这会限制 bean 在容器内的预先可见类型。只有当 bean 完全实例化后才能知道其实际类型。

由于非 lazy-init 的 singleton bean 的实例化顺序与 `@Bean` 方法的声明顺序一致，所以 byType 匹配的结果前后可能会发生变化，这取决于声明接口返回类型的 `@Bean` 方法何时被实例化。

**方法参数实现依赖注入**

`@Bean` 方法可以声明多个参数，IoC 容器会查找匹配类型的 bean 对这些参数进行赋值。

```
@Bean
public TransferService transferService(AccountRepository accountRepository) {
	return new TransferServiceImpl(accountRepository);
}
```

**静态 @Bean 方法**

如果 `@Bean` 方法声明为静态，那这个 bean 的生命周期就与包含它的类无关，这能用于创建后置处理器。因为后置处理器的实例化优先级太高，它创建时可能很多 Spring 基础组件还未加载，导致一些服务不生效。

CGLIB 子类只能代理非静态方法，所以，静态 `@Bean` 方法是 `lite` 模式，对它的调用不会被代理，只会返回新的实例。

**@Bean 方法的可见性**

`@Bean` 方法的可见性不会对 bean definition 产生影响，在非 `@Configuration` 类，可以随意声明 `@Bean` 方法。但在 `full` `@Configuration` 类，它的 `@Bean` 方法必须是可重写的，不能声明为 `private`、`final`。

**继承**

`@Bean` 方法支持继承和接口实现，组件类或配置类的基类上的 `@Bean` 方法也会被扫描到。

**重载**

在单个类，可以为同一个 bean 定义多个重载 `@Bean` 方法，Spring 会选择可用依赖最多的方法。

### 内部 @Bean 方法注入：full 和 lite 模式

`@Configuration` 类本身也是 `@Component` 组件，Spring 默认会对配置类进行代理，它其中的 `@Bean` 方法也会被代理，这些 `@Bean` 方法如果在内部调用其它相同类的 `@Bean` 方法，这些方法调用会被代理成从 IoC 容器获取对象。这就是 `full` 模式，它使得 `@Bean` 方法能使用 `@Bean` 方法调用进行依赖注入。

而对于未被代理的 `@Bean` 方法，它们的内部 `@Bean` 方法调用，就只是普通的工厂方法调用，永远返回新的实例，这就是 `lite` 模式。

从 Spring 5.2 开始，`@Configuration` 可以设置属性 `proxyBeanMethods = false`，手动关闭当前配置类的代理，它的 `@Bean` 方法也就成为 `lite` 模式。此外，定义在非 `@Configuration` 类的 `@Bean` 方法不会被代理，它们也是 `lite` 模式。

Spring 主要使用的 CGLIB 代理基于继承实现。所以，`full` 模式的配置类和 `@Bean` 方法都不能被 `final` 或 `private` 修饰。

可以自动装配 `@Configuration` 类，调用这些配置类的 `@Bean` 方法与内部 `@Bean` 方法调用效果相同：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

### 引入外部组件 @Import

在 `@Configuration` 类，可以使用 `@Import` 引入外部其它的组件，类似于 `BeanFactory::register()` 方法。

```
@Configuration
@Import(ConfigA.class)
public class ConfigB {...}
```

### 注解源的容器实例化

Spring 3.0 引入的 `AnnotationConfigApplicationContext` 可以接受 `@Configuration` 类作为配置信息源，此外，它也可以只接受若干个组件类。

下例使用 `configuration` 类配置容器：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

下例使用 `@Component` 类配置容器：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

还可以不输入任何信息源，创建空容器，使用 `register()` 方法手动注册 bean：

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

下例使用 `AnnotationConfigApplicationContext` 的 `scan(String...)` 方法进行手动组件扫描：

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

### 注解、XML 混合配置

`@Configuration` 与 XML 文件不是互斥的，它们可以混合使用。但在合作时，需要确定以哪种配置方式为中心，简单说，就是 `ApplicationContext` 是加载 XML 文件还是加载配置类。

对于 `@Configuration` 类，可以使用 `@ImportResource` 导入外部 XML 文件：

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {
	//
}
```

对于 XML 文件，要么使用 `<bean/>` 标签注册 `@Configuration` 类，要么启用组件扫描。前者还必须单独使用 `<context:annotation-config/>` 标签启用对注解配置的支持。

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>

    <bean class="com.acme.AppConfig"/>
</beans>
```

```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
</beans>
```

## Bean 的生命周期

**bean 大概的生命周期**

* IoC 容器读取配置信息，生成 `BeanDefinition` 对象。

- IoC 容器通过反射调用构造器创建实例
- 使用 setter 方法设置对象属性
- 如果 bean 实现 aware 接口，执行接口方法设置相关依赖
- 后置处理器执行 `BeanPostProcessor::postProcessBeforeInitialization`
- 执行初始回调
  - `@PostConstruct` 标注的方法
  - `InitializingBean` 接口的 `afterPropertiesSet()` 方法

  - bean definition 的 init-method 属性指定的方法

- 后置处理器执行 `BeanPostProcessor::postProcessAfterInitialization`
- 执行销毁回调
  - `@PreDestroy` 标注的方法
  - `DisposableBean` 接口的 `destroy()` 方法

  - bean definition 的 destroy-method 属性指定的方法


**定制 bean 的生命周期，有两种方式**

* Lifecycle Callbacks：生命周期回调
* AwareInterface：特殊接口

### 初始回调，销毁回调

执行时机：

* 初始回调：bean 实例化并装配属性后

* 销毁回调：容器销毁前

设置 bean 的初始回调、销毁回调有 3 种方式：接口、配置、注解。但不论哪一种，bean 的类都必须实现两种回调方法。

如果 bean 使用不同方式配置多个回调，这些回调将按以下顺序执行：

* 初始回调

  * `@PostConstruct` 标注的方法

  * `InitializingBean` 接口的 `afterPropertiesSet()` 方法

  * bean definition 的 init-method 属性指定的方法

* 销毁回调

  - `@PreDestroy` 标注的方法

  - `DisposableBean` 接口的 `destroy()` 方法

  - bean definition 的 init-method 属性指定的方法

如果不同的方式设置相同的回调方法，这个方法只会执行一次。

> 注意，销毁回调时在容器销毁前执行，因为容器在销毁前会注销所有 bean。而且，容器必须显式调用销毁方法 `close()` 或 `sotp()`，这才能触发销毁回调。prototype bean 不会执行销毁回调。

> IoC 容器保证在 bean 设置完所有属性后立即执行初始回调，此时 bean 还未被 AOP 拦截应用。bean 完全初始化后才会调用拦截器链。

#### 实现回调接口

`InitializingBean` 和 `DisposableBean` 两个接口都只有一个方法，分别表示初始回调和销毁回调。IoC 容器一旦察觉到 bean 实现这两个接口，就自动执行相应的回调方法。

```
public class AnotherExampleBean implements InitializingBean {
	
    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
    
    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

#### Definition 配置回调

可以在 bean 的类中自定义方法，然后在 bean definition 指定其为初始回调，或销毁回调。

**Java 配置**

`@Bean` 的属性 `initMethod` 和 `destroyMethod` 用于指定 bean 的初始回调和销毁回调。

```
@Bean(initMethod = "init", destroyMethod = "cleanup")
public BeanOne beanOne() {
	return new BeanOne();
}
```

**XML 配置**

对于 XML 配置，`<bean/>` 的 `init-method` 和 `destroy-method` 属性用于指定回调方法。

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>

<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

#### 注解标注回调

使用 `@PostConstruct` 和 `@PreDestroy` 标注 bean 类的方法，效果与配置回调相同。

```
public class ExampleBean {

	@PostConstruct
    public void init() {
        // do some initialization work
    }
    
    @PreDestroy
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

### 容器生命周期相关的回调

Spring Bean 回调和 Lifecycle 回调的执行顺序：

```
A bean initA bean isRunning : false
A bean Lifecycle start
A bean is working
A bean isRunning : true
A bean Lifecycle stop
A bean destroy
```

#### Lifecycle

`Lifecycle` 接口有 3 个与容器生命周期相关的回调方法，任何 bean 都可以实现这个接口。

```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

`ApplicationContext` 也是 `Lifecycle` 的实现，它在收到 `start`、`stop` 调用时，会把这个信号传播到容器内所有的 `Lifecycle` 实现。所以实现 `Lifecycle` 的 bean，会拥有与容器生命周期相关的回调。

容器在启动时会把 `start` 的信号传递给所有 lifecycle bean，这说明此时这些 bean 至少都已经完成实例化。所以，`Lifecycle::start` 的执行时机是所有 bean 都实例化、初始化之后。

至于 `close`，通常是先执行 `Lifecycle::close` 再执行 bean 的销毁回调。但容器在 hot refresh 时，不会传播 `close` 信号，此时只执行销毁回调。

#### LifecycleProcessor

`LifecycleProcessor` 是 `Lifecycle` 的扩展，它增加两个方法来响应容器的重启和关闭。

```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

`onRefresh` 和 `onClose` 的执行时机是在容器 refreshing 和 closing 的时候。`onClose` 只是简单的驱动关闭过程，就像显式调用 `sotp()` 一样，但它只发生在容器 closing 时。

#### SmartLifecycle

lifecycle bean 的 `start`、`stop` 执行顺序没有规定，如果想控制这个过程，可以考虑使用 `SmartLifecycle`。

```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

```
public interface Phased {

    int getPhase();
}
```

容器启动时，对于实现 `SmartLifecycle` 的 bean，先调用其 `isAutoStartup` 方法，如果返回 `true`，容器将执行它的 `start` 方法。

并且，`SmartLifecycle` 可以定制 `start` 和 `close` 的相对执行顺序。`getPhase` 的返回值越高，执行时机就越靠近 bean 的使用时间，`phase` 默认为 0。

容器 refresh 后，会检查 `SmartLifecycle` 的 `isAutoStartup()` 返回值，如果返回 `true`，容器也会执行它的 `start` 方法，此过程同样遵守 `phase` 顺序。

`stop(Runnable)` 方法会在 `SmartLifecycle` 完成关闭过程后被异步执行。`LifecycleProcessor` 的默认实现 `DefaultLifecycleProcessor` 会等待各 `phase` 的 bean 执行这个任务 30 秒，这个等待时间可以修改，比如自定义一个标识符为 `lifecycleProcessor` 的 `processor`。

### 关闭容器

基于 web 环境的 `ApplicationContext` 已经实现在程序退出时自动执行 shutdown。而对于非 web 的容器，如果不显式调用 `stop()` 或 `close()`，但想让容器在程序退出时自动执行关闭过程，可以向虚拟机注册钩子函数。它会在 JVM 关闭时执行 `DefaultLifecycleProcessor` 的 `onClose` 方法，从而触发 `Lifecycle::close` 和销毁回调。

`ConfigurableApplicationContext::registerShutdownHook`  向虚拟机注册关闭钩子函数：

```
ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
// add a shutdown hook for the above context...
ctx.registerShutdownHook();
```

### AwareInterface

`AwareInterface` 是一类 Spring 提供的特殊接口，任何 bean 都可以实现这些接口，它们的作用是使 bean 能够与 Spring 进行交互，比如获得 `ApplicationContext` 的引用。

但 `AwareInterface` 会把程序代码与 Spring 耦合在一起，这当然不符合控制反转的原则，通常只有作为基础结构的 bean 会使用这些接口。

`AwareInterface` 的执行时机是在 bean 装配所有属性后，执行初始回调前。

**ApplicationContextAware**

`ApplicationContextAware` 能让 bean 得到 `ApplicationContext` 的引用。

```
public interface ApplicationContextAware {

void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

**BeanNameAware**

`BeanNameAware` 能让 bean 得到其在容器内的标识符。

```
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

## 容器的拓展

IoC 容器已经为我们提供 bean 的创建、装配等功能，如果想定制容器的功能，可以通过添加一些实现特殊接口的 bean 来完成，这些特殊接口的方法会在特定的时机运行。

这其实就是模板方法设计模式，Spring 把能拓展容器的点固定。主要有 3 个拓展点：

* BeanPostProcessor
* BeanFactoryPostProcessor
* FactoryBean

### BeanPostProcessor

根据名字，可以知道 `BeanPostProcessor` 用于定制与 bean 相关的功能。它可以修改容器对 bean 的实例化、依赖解析的默认逻辑，也就是干预 bean 的生命周期。

可以向容器注册多个 `BeanPostProcessor`，如果它们还实现 `Ordered` 接口，那还能控制它们的执行顺序。

```
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

**执行时机**

`BeanPostProcessor` 是对 bean 进行操作，所以必然是在 bean 实例化之后执行。它的两个方法分别在 bean 的初始回调前后执行。`BeanPostProcessor` 可以对 bean 做任何事情，AOP 其实就是通过 `BeanPostProcessor` 来增强 bean 的。

**执行顺序**

如果 `BeanPostProcessor` 实现 `Ordered` 接口，`order` 值越大，执行顺序就越靠近初始回调，默认为 0。

**注册方式**

`BeanPostProcessor` 的注册与普通 bean 完全相同，容器会自动检测 `BeanPostProcessor`。如果是使用工厂方法提供，必须标注返回类型为 `BeanPostProcessor`，因为容器无法在创建实例前获取它的类型，而后置处理器需要尽早被实例化然后应用到其它 bean。

还可以在 Java 代码层面使用 `ConfigurableBeanFactory::addBeanPostProcessor` 方法添加处理器。这种方式添加的 `BeanPostProcessor`，它们的执行顺序不依据 `order` 值，而是根据 add 顺序，且它们都先于其它方式添加的处理器执行。

**BeanPostProcessor 和 AOP 动态代理**

所有 `BeanPostProcessor` 和它们依赖的 bean 都会在容器启动时先被实例化，然后按照 `order` 顺序或 add 顺序应用于普通 bean。AOP 本身依靠 `BeanPostProcessor` 对 bean 进行代理增强，因此，`BeanPostProcessor` 以及它们依赖的 bean 都不能被 AOP 增强。

### BeanFactoryPostProcessor

学习 `BeanPostProcessor` 后，`BeanFactoryPostProcessor` 对于我们便不难理解。与 `BeanPostProcessor` 不同的是，`BeanFactoryPostProcessor` 操作的对象是 `BeanDefinition`。

```
@FunctionalInterface
public interface BeanFactoryPostProcessor {
void postProcessBeanFactory(ConfigurableListableBeanFactory var1) throws BeansException;
}
```

**执行时机**

`BeanFactoryPostProcessor` 会在除 `BeanPostProcessor` 之外的 bean 实例化之前，读取和修改它们的配置信息。所以，`BeanFactoryPostProcessor` 是在 bean 的实例化之前执行。

需要注意，`BeanFactoryPostProcessor` 可以使用 `ConfigurableListableBeanFactory::getBean` 获取实例，这会导致 bean 的提前实例化。

**执行顺序**

`BeanFactoryPostProcessor` 的执行顺序也是通过 `order` 进行控制，默认为 0。

**注册方式**

`BeanFactoryPostProcessor` 的注册方式与普通 bean 相同。容器会检测 `BeanFactoryPostProcessor`，将其尽早实例化。

### FactoryBean

`FactoryBean` 的作用是向容器注册现成的对象，这与使用 `BeanDefinition` 构建 bean 不同。它常用注册比较复杂的对象，MyBatis 便是使用这种方式与 Spring 进行整合。

```
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	// 对象实例
    @Nullable
    T getObject() throws Exception;

	// 对象类型
    @Nullable
    Class<?> getObjectType();

	// 是否单例
    default boolean isSingleton() {
        return true;
    }
}
```

# 切面编程 AOP

Aspect-oriented Programming，AOP 面向切面编程，是一种不同于面向对象的编程思想。OOP 的关注点是类，而 AOP 的关注点是面。所谓的面，是跨越类型，在特定的点增加的自定义逻辑。AOP 好处是不需要修改类，就能对类的方法进行增强。

## AOP 的相关概念

下面是与 AOP 切面编程相关的术语：

- aspect：切面。跨越类型，在特定点增加的自定义逻辑，即切入点+通知。
- join point：连接点。可以被增强的点，在 Spring AOP，就是方法调用。
- advice：通知。切面在连接点的行为，根据执行时机分为多个类型。Spring AOP 把通知实现为拦截器。
- pointcut：切入点。实际增强的连接点。通知会应用于所有与切入点表达式匹配的连接点。
- introduction：引入。为类型声明额外的方法或字段，Spring AOP 允许向被增强的对象引入新接口。
- target：目标。被切面增强的对象。
- proxy：代理。目标对象的代理，Spring AOP 使用 JDK 或 CGLIB 实现。
- weaving：织入。把切面与目标对象连接以创建代理的动作。

根据执行时机的不同，通知 advice 有以下几种类型：

- before advice：切入点之前执行的通知，不能阻止继续执行到切入点，除非抛出异常。
- after returning advice：切入点正常返回后执行的通知。
- after throwing advice：切入点抛出异常后执行的通知。
- after advice：切入点结束后执行的通知，不论是正常返回还是抛出异常。
- around advice：切入点前后都能定义逻辑的通知，可以阻止继续执行到切入点。

## 切面编程 @AspectJ

AspectJ 是基于 Java 的 AOP 实现，可以说它是目前 Java 生态中最优秀的 AOP 框架。AspectJ 的原理是编译时增强，Spring AOP 的原理是动态代理。

Spring AOP 提供一种使用注解+普通类来定义切面的编程风格，这就是 `@AspectJ` 切面编程。这种风格使得切面的编写非常简单，可读性也较高，是 Spring AOP 的主流应用方式。Spring AOP 使用 AspectJ 的切入点表达式来定义切入点，所以需要依赖 AspectJ 的库以解析表达式。

Spring 已经支持集成 AspectJ，这里暂不介绍。

## 启用 @AspectJ 支持

要使用 `@AspectJ` 切面，需要专门设置开启相关支持。此时，Spring 会根据 `@AspectJ` 切面类来配置 AOP，并对被增强的类进行动态代理，以拦截切入点的调用。

**Java 配置**

使用 `@EnableAspectJAutoProxy` 标注配置类。

```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {...}
```

**XML 配置**

```
<aop:aspectj-autoproxy/>
```

## 切面类 @Aspect

标注有 `@Aspect` 的 bean 类就是切面类，Spring AOP 根据 aspectJ bean 进行配置。注意，切面也是 bean，所以如果使用组件扫描，切面类还需要标注 `@Component` 注解。此外，切面不能作为其它通知的目标。

```
@Aspect
@Component
public class NotVeryUsefulAspect {...}
```

切面 = 切入点 + 通知，所以，切入点和通知都以方法的形式定义在切面类。

## 切入点 Pointcut

Spring AOP 只支持对方法增强，也就是说，切入点只能是 bean 的方法调用。前面已经提到，`@AspectJ` 风格使用 AspectJ 的库解析和匹配切入点。所以，这里自然就使用 AspectJ 的切入点表达式来指示切入点。

### 切入点方法 @Pointcut

`@Pointcut` 用于标注方法，它标识当前方法是切入点的定义，这个方法的返回类型必须是 `void`。切入点的声明分为两个部分：签名、表达式。`@Pointcut` 的 `value` 属性提供切入点表达式，签名则由方法定义提供。

下例定义一个名为 anyOldTransfer 的切入点，它匹配所有名为 transfer 的方法：

```
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```

对于常用的切入点，可以使用专门的切面类集中保存。在需要的地方，使用全限定方法路径引用。

```
@Aspect
public class CommonPointcuts {

    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}
}
```

### 切入点表达式

AspectJ 的切入点表达式支持多种指示器 Pointcut Designator，不同指示器的语法和语义相差非常大，以适应不同的指示需求。Spring AOP 只支持部分指示器，这里只对常用的进行介绍。

指示器分类：

* execution：精确匹配方法。
* within：指示指定路径的所有方法，类路径。
* @annotation：指示标注指定注解的所有方法。

此外，Spring AOP 还定义有一个自己的指示器：

* bean：指示标识符匹配的 bean 的所有方法。它支持 `*` 通配符，以及 `&&`、`||` 和 `!` 运算符。

#### 精确匹配 execution

`execution` 是最常用的切入点指示器，它的表达式语法如下：

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)throws-pattern?)
```

可以看到，execution 表达式由 6 个匹配模式组成，以 `?` 结尾表示该模式可以省略。所以，只有返回类型、方法名、方法参数是必须的。

| 模式                    | 说明                                                         | 必须 |
| ----------------------- | ------------------------------------------------------------ | ---- |
| modifiers-pattern?      | 访问修饰符，省略则匹配任意修饰符。                           | 否   |
| ret-type-pattern        | 返回类型，`*` 匹配任意类型                                   | 是   |
| declaring-type-pattern? | 类的全限定路径，用 `.` 分隔包名和连接 name-pattern，可用 `.`，`*` 进行模糊匹配 | 否   |
| name-pattern            | 方法名，可用 `*` 进行模糊匹配                                | 是   |
| param-pattern           | 方法参数，`()` 匹配无参，`(..)` 任意参数，`(*)` 一个任意类型的参数 | 是   |
| throws-pattern?         | 方法抛出的异常类型                                           | 否   |

以下是 execution 指示器的使用示例：

- 指示所有 `public` 方法

  ```
  execution(public * *(..))
  ```

- 指示名字以 "set" 开头的所有方法

  ```
  execution(* set*(..))
  ```

- 指示 `service` 包的所有方法

  ```
  execution(* com.xyz.service.*.*(..))
  ```

#### 其它指示器的示例

within 指示器：

- 指示 service 包的所有方法

  ```
  within(com.xyz.service.*)
  ```

- 指示 service 包及其子包的所有方法

  ```
  within(com.xyz.service..*)
  ```

@annotation 指示器：

* 指示标注 `@Transactional` 的所有方法：

  ```
  @annotation(org.springframework.transaction.annotation.Transactional)
  ```


bean 指示器：

* 指示标识符等于 "tradeService" 的 bean 的所有方法：

  ```
  bean(tradeService)
  ```

* 指示标识符匹配 "*service" 的 bean 的所有方法：

  ```
  bean(*service)
  ```

#### 混合切入点表达式

不同的指示器可以相互合作，使用逻辑运算符 `&&`，`||` 和 `!` 连接。

* 指示 service 包的所有 `public` 方法

  ```
  execution(public * *(..)) && within(com.xyz.service..*)
  ```

#### 切入点表达式引用

切入点表达式可以通过方法路径，引用其它 `@Pointcut` 方法定义的表达式。方法路径可以是全限定路径，如果引用方法在相同的类，则可以省略类路径。

```
com.xyz.myapp.CommonPointcuts.anyPublicOperation()
```

```
anyPublicOperation()
```

## 通知 Advice

通知也是以方法的形式定义在 `@AspectJ` 切面类，它需要与切入点关联，以确定需要增强的方法。方法的内部逻辑，就是通知增强的内容。

`@Before`、`@AfterReturning`、`@AfterThrowing`、`@After`、`@Around` 这 5 个注解用于标注各种通知方法，它们的 `value` 属性用于设置切入点，可以使用全限定路径引用 `@Pointcut` 方法，也可以原地定义切入点表达式。

### Before

before advice 在切入点之前执行，它只是执行通知方法，不能影响切入点的运行。除非它抛出异常，后续的执行自然会被中断。`@Before` 标注前置通知方法。

```
@Aspect
public class BeforeExample {

    @Before("dataAccessOperation()")
    public void doAccessCheck() {...}
}
```

### After Returning

after returning advice 在切入点正常返回后执行，如果切入点发生异常则不执行。`@AfterReturning` 标注返回通知方法。

```
@Aspect
public class AfterReturningExample {

    @AfterReturning("dataAccessOperation()")
    public void doAccessCheck() {...}
}
```

`@AfterReturning` 方法可以接收切入点的返回值，只需声明一个与切入点返回类型一致的方法参数，然后设置注解的 `value` 属性为参数名。

```
@Aspect
public class AfterReturningExample {
    
    @AfterReturning(pointcut="dataAccessOperation()", returning="retVal")
    public void doAccessCheck(Object retVal) {...}
}
```

### After Throwing

after throwing advice 在切入点抛出异常返回后执行。`@AfterThrowing` 标注异常通知方法。类似的，异常通知支持接收切入点抛出的异常，只需增加一个类型一致的参数，以及设置注解属性 `throwing` 为参数名。

```
@Aspect
public class AfterThrowingExample {
   
    @AfterThrowing(pointcut="dataAccessOperation()", throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {...}
}
```

### After Finally

after advice 在切入点结束后执行，不论切入点是正常返回还是抛出异常。`@After` 标注最终通知。

```
@Aspect
public class AfterFinallyExample {

    @After("dataAccessOperation()")
    public void doReleaseLock() {...}
}
```

### Around

around advice 是最为强大的通知，它能在切入点前后定义逻辑，甚至能阻止继续执行到切入点。`@Around` 标注环绕通知方法。

`@Around` 方法必须有一个类型为 `ProceedingJoinPoint` 的参数，且一定放在首位。它表示切入点，调用 `ProceedingJoinPoint::proceed()` 以运行切入点，这个方法可以反复调用。`proceed()` 能接收 `Object[]` 参数，表示切入点的入参。

around advice 的结果会作为切入点的结果返回给调用者。比如，使用 around advice 实现缓存，先在 Redis 检索，如果有则直接返回，否则，执行切入点并将结果缓存和返回。

```
@Aspect
public class AroundExample {

    @Around("dataAccessOperation()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
}
```

### 通知的执行顺序

如果一个切入点配置有多个通知，Spring AOP 使用与 AspectJ 相同的优先级规则来确定这些通知的执行顺序：

* "in the way in"，优先级高的通知先执行。
* "On the way out"，优先级高的通知后执行。

从 Spring 5.2.7 开始，相同 `@Aspect` 类中，通知的优先级顺序为：

* `@Around` > `@Before` > `@After` > `@AfterReturning`  >`@AfterThrowing`

对于同个 `@Aspect` 类中类型相同的通知，它们在相同切入点的执行顺序是不确定的，考虑把它们合并，或拆分到不同的切面。

定义在不同切面的通知，它们在相同切入点的执行顺序是不确定的。可以使切面实现 `Ordered` 接口，或使用 `@Order` 标注切面，以定义优先级。order 值越低，优先级越高。

### 切入点 JoinPoint

任何通知方法都可以声明一个 `org.aspectj.lang.JoinPoint` 参数，并把它放在首位。`JoinPoint` 表示切入点，通知可以通过它获取切入点的相关信息。

around advice 的 `ProceedingJoinPoint` 参数是 `JoinPoint` 的子类型。

> **org.aspectj.lang.JoinPoint**
>
> * `Object[] getArgs()`：返回切入点的参数。
> * `Object getThis()`：返回代理对象。
> * `Object getTarget()`：返回目标对象。
> * `Signature getSignature()`：返回方法描述。

## 代理机制

Spring AOP 基于动态代理实现。默认情况，如果目标对象实现至少一个接口，则使用 JDK 动态代理，否则，使用 CGLIB 动态代理。spring-core 模块已经包含 CGLIB 库。

CGLIB 会代理目标对象的所有方法，而 JDK 只会代理接口方法。如果想强制使用 CGLIB 代理，可以修改相关配置。不过，必须考虑到 CGLIB 基于继承，它不能代理 `final` 方法和 `private` 类。

> 由于 Spring AOP 代理的特性，目标对象内部的方法调用不会被拦截，只能拦截外部对目标对象的调用。对于 JDK 代理，只能拦截代理接口的 `public` 方法。对于 CGLIB，可以拦截代理的 `public` 和 `protected` 方法，还可以拦截包可见的方法。

**Java 配置**

设置 `@EnableAspectJAutoProxy` 的属性 `proxyTargetClass = true`：

```
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class AppConfig {

}
```

**XML 配置**

设置 `<aop:config/>` 或 `<aop:aspectj-autoproxy/>` 的相关属性：

```
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

```
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

# 事务管理 Transaction

Declarative Transaction Management 声明式事务管理能以对现有代码影响较小的方式进行事务管理。它基于 AOP 实现，相关的 aspects 已被预先提供。可以直接参照模板使用声明式事务，不需要了解 AOP，简单到只需要标注几个注解，设置一些属性即可。

Spring Framework 的声明式事务管理具有以下特点：

* 支持任何环境，比如通过调整配置在 JDBC、MyBatis 中使用事务。

* 可应用于任何类。
* 可以修改回滚规则。回滚规则是指在发生哪些 Exception 或 Throwable 时进行回滚。
* 允许使用 Spring AOP 自定义事务管理的行为。
* 不支持在远程调用传播事务。

## 相关接口和原理

Spring 的事务管理主要有 3 个接口

* `TransactionManagement`：事务管理器
* `TransactionDefinition`：事务的定义信息
* `TransactionStatus`：事务的运行状态

**TransactionManagement**
事务管理器是事务的管理者，它为所有不同的平台提供统一的事务接口，具体实现由各个平台自己决定，类似于 JDBC 的思想。

**TransactionDefinition**

事务的定义信息用于描述当前事务的行为，主要有隔离级别、传播级别、回滚规则、超时、只读。事务管理器根据这些信息来管理事务。

`@Transactional` 注解的作用，一是标志事务方法，二是提供事务定义信息。

**TransactionStatus**

`TransactionStatus` 接口的作用是获取当前事务的运行状态。

## 启用事务管理器

使用 `@EnableTransactionManagement` 标注配置类。

```
@Configuration
@EnableTransactionManagement
public class AppConfig {...}
```

## 事务管理器配置

下例代码创建事务管理器。`@Transactional` 默认使用标识符为 transactionManager 的事务管理器，可以使用 `value` 或 `transactionManager` 属性指定事务管理器的标识符，如果没有匹配项，则报错。

```
@Bean
public DataSourceTransactionManager transactionManager(){
    DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
    // 必须配置数据源
    transactionManager.setDataSource(druidDataSource());
    return transactionManager;
}

@Bean
public DruidDataSource druidDataSource(){
    DruidDataSource druidDataSource = new DruidDataSource();
    druidDataSource.setUrl("jdbc:mysql://localhost:3306/test");
    druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    druidDataSource.setUsername("root");
    druidDataSource.setPassword("root");
    return druidDataSource;
}
```

下表列举 `@EnableTransactionManagement` 属性的默认值和作用。

| Attribute          | Default                     | Description                                                  |
| :----------------- | :-------------------------- | :----------------------------------------------------------- |
| `mode`             | `proxy`                     | `proxy` 模式使用 Spring AOP 处理需要被事务管理的 bean，这种方式只对从外部发出的方法调用生效。`aspectj` 模式使用 AspectJ 编译时织入实现 AOP。 |
| `proxyTargetClass` | `false`                     | 仅在 `proxy` 模式有效。`true` 强制使用基于继承的动态代理 CGLIB 实现 AOP。`false` 使用 JDK 或 CGLIB 代理。 |
| `order`            | `Ordered.LOWEST_PRECEDENCE` | 事务通知的 `order` 值，默认最低优先级，值为 `Integer.MAX_VALUE`。 |

## 使用事务 @Transactional

### 可标注的点

凡是被 `@Transactional` 标注的类、接口、方法，它们的方法调用都会被 Spring 进行事务管理。标注类时，事务不会应用于从基类继承的方法，可以在本地重新声明这些方法，或对父类也进行事务管理。

如果有多个重复的事务设置，比如某个方法以及包含它的类都被 `@Transactional` 标注，距离近的设置优先级更高。

### 默认事务行为

`@Transactional` 关于事务的默认设置：

- 事务传播级别：`REQUIRED`
- 事务隔离级别：`DEFAULT`
- 读写事务
- 事务超时时间默认为底层事务系统的默认设置，如果底层系统不支持超时设置，则无。
- 任何运行时非检查型异常和 `error` 都会触发回滚，检查型异常默认不触发回滚。

### 注解相关属性

下表列举 `@Transactional` 属性的默认值和作用。

| Property                 | Type                  | Description                                                  |
| :----------------------- | :-------------------- | :----------------------------------------------------------- |
| value                    | `String`              | aliasFor transactionManager property                         |
| transactionManager       | `String`              | 使用的事务管理器的标识符，可选。默认使用标识符为 transactionManager 的管理器。 |
| propagation              | `enum`: `Propagation` | 事务传播级别，可选。                                         |
| `isolation`              | `enum`: `Isolation`   | 隔离级别，可选。只在 `REQUIRED` 或 `REQUIRES_NEW` 传播级别生效。 |
| `timeout`                | `int` 单位秒          | 事务超时时间，可选。只在 `REQUIRED` 或 `REQUIRES_NEW` 传播级别生效。默认 -1，表示使用底层事务系统的默认设置。 |
| `readOnly`               | `boolean`             | 读写事务，和只读事务。只在 `REQUIRED` 或 `REQUIRES_NEW` 传播级别生效。 |
| `rollbackFor`            | `Throwable` 数组      | 导致回滚的异常类型数组，可选。注意，这几个归滚相关的属性，是拓展默认规则，它不会取消默认回滚策略。 |
| `rollbackForClassName`   | `Throwable` 类名数组  | 导致回滚的异常类型名称数组，可选。                           |
| `noRollbackFor`          | `Throwable` 数组      | 不会导致回滚的异常类型数组，可选。                           |
| `noRollbackForClassName` | `Throwable` 类名数组  | 不会导致回滚的异常类型名称数组，可选。                       |

## 事务传播级别（7种）

事务传播级别用于解决逻辑事务和物理事务之间的问题。逻辑事务指由 Spring 管理的事务，物理事务指底层系统提供的事务支持，比如 JDBC 的事务。逻辑事务提供更丰富的控制，通常讨论的也是逻辑事务。

> 这是什么意思呢？比如方法 A 和方法 B 都被 `@Transactional` 标注，如果在方法 A 中调用方法 B，那么 A 和 B 的执行是在同一个事务，还是在两个事务呢？这要根据具体环境而定。

| Propagation     | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| `required`      | 如果有事务正在运行，当前方法加入这个事务，否则开启新事务并在其中运行。 |
| `requires_new`  | 直接开启新事务并在其中运行，挂起已有事务。                   |
| `supports`      | 如果有事务正在运行，当前方法加入这个事务，否则当前方法不使用事务。 |
| `not_supported` | 当前方法不支持事务，如果有事务正在运行则将其挂起。           |
| `mandatory`     | 上下文必须存在事务，当前方法加入这个事务，否则抛出异常。     |
| `never`         | 当前方法不支持事务，如果有事务正在运行则抛出异常。           |
| `nested`        | 如果有事务正在运行，则创建嵌套事务并在其中运行，否则开启新事务并在其中运行。这种设置可以进行局部回滚，外部事务不会察觉内部发生的回滚。 |

> 需要注意，"如果当前有事务正在运行" 的意思是调用当前方法的方法也被 `@Transactional` 标注且正处于事务当中。如果外部事务方法调用两个 `@Transactional` 方法 A 和 B，A 先于 B 执行，它开启一个新事务，但 A 的事务与 B 毫无关系，它并不是 B 的外部事务。

默认情况，子事务接受外部事务的设置（传播级别、隔离级别、超时等），忽略本地配置。如果想在加入具有不同隔离级别的现有事务时拒绝它的隔离设置，可以设置事务管理器 `validateExistingTransactions=true`，这个设置还能拒绝外部事务不同的只读标记。

传播级别为 `REQUIRED` 时，会为每个事务方法创建一个逻辑事务作用域。每个这样的逻辑事务可以独立设置仅回滚属性，外部事务作用域在逻辑上独立于内部事务作用域。但在 `REQUIRED` 看来，所有这些作用域都映射在同一个物理事务，内部事务的仅回滚标记会影响外部事务的实际提交。对于外部事务来说，内部事务的回滚是意外的，会抛出 `UnexpectedRollbackException` 异常。

## 事务隔离级别（5种）

`@Transactional` 的属性 `isolation` 用于设置当前事务的隔离级别，它的值为枚举类型 `Isolation`，有 5 个可选值，比 MySQL 多一个默认级别。

| isolation                    | Description              |
| ---------------------------- | ------------------------ |
| `Isolation.DEFAULT`          | 数据库默认的事务隔离级别 |
| `Isolation.READ_UNCOMMITTED` | 读未提交                 |
| `Isolation.READ_COMMITTED`   | 读已提交                 |
| `Isolation.REPEATABLE_READ`  | 可重复读                 |
| `Isolation.SERIALIZABLE`     | 串行化                   |

# Environment

`Environment` 是集成在容器内的接口，它对应用环境的两个方面进行抽象：profiles 和 properties。

## Profiles

`profile` 表示一组 bean definition，仅当这个 `profile` 被激活时，它包含的 bean definition 才会注册到容器。bean definition 不论使用什么方式定义，它都一定属于某个 `profile`。`Environment` 在 profiles 方面的作用是决定那些 `profile` 被激活。

### 定义 @Profile

`@Profile` 可以标注组件类、配置类、`@Bean` 方法，它的作用不是分组，而是指示 bean 在什么环境下可用。`@Profile` 的 `value` 属性接受符串分组，值可以是简单的 profile name，此时它表示 bean 在这些 `profile` 任意一个被激活时可用。值还可以是 profile expression，表达式可以使用 `!`、`&`、`|` 运算符，此时它表示 bean 在哪些 `profile` 激活或哪些 `profile` 未激活时可用。

> 混合使用 `&` 和 `|` 时，必须用括号明确逻辑，比如 "production & (us-east | eu-central)"。

`@Profile` 标注配置类时，它包含的所有 `@Bean`、`@Import`、组件扫描都不可用，除非所属的 profile 被激活。

```
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {...}
    
    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {...}
}
```

### 激活 profile

激活 `profile` 有多种方式，最简单直接的就是调用 `Environment` 接口：

```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
```

还可以设置 `spring.profiles.active` 这个 `property` 来激活 profile。设置 `propertie` 的方式很多，下例使用启动参数设置。

```
-Dspring.profiles.active="profile1,profile2"
```

### 默认 profile

IoC 容器有一个默认的 `profile`，它的 profile name 就是 "default"。没有专门指定的 bean definition 都属于 default。如果环境没有显式激活任何 `profile`，则默认激活 default，但如果有任何一个 `profile` 被激活，就不会自动激活 default。

可以调用 `Environment::setDefaultProfiles` 方法修改默认 `profile` 的 profile name，或者，设置 `spring.profiles.default` 这个 `property`。

## Properties

`propertie` 就是 key-value 键值对的集合，它在许多应用特别是 Spring Boot 担任重要的角色。它的数据可能来自各种源：.properties 文件、JVM 系统属性、系统环境变量、JNDI 等。`Environment` 在 properties 方面的作用是提供访问这些属性的接口。

### 访问 propertie

下例查询当前环境是否含有 hi-property 这个 `property`：

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("hi-property");
```

`Environment` 会在一堆 `PropertySource` 中进行查找，`PropertySource` 是 key-value 数据源的抽象。

Spring 的 `StandardEnvironment` 配置有两个 `PropertySource` 对象：

* JVM 属性，通过 `System::getProperties()` 获取。
* 系统环境变量，通过 `System::getenv()` 获取。

`StandardServletEnvironment` 还含有其它属性，比如 `servlet config`、`JndiPropertySource`。

`Environment` 查找 `property` 的过程是分层的，`StandardEnvironment` 的检索顺序如下：

1. `ServletConfig` 参数（DispatcherServlet）
2. `ServletContext` 参数（web.xml context-param）
3. `JNDI` 环境变量
4. `JVM` 系统属性，包含 `-D` 命令行参数
5. `JVM` 系统环境，包含系统变量

### 注入 properties 源

如果想向环境注入其它数据源，可以实现并实例化 `PropertySource`，然后将其添加到 `Environment` 的 `PropertySources` 属性。自定义添加的 `PropertySource` 拥有最高查找优先级。

```
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

更简单常用的，使用 `@PropertySource` 标注配置类，它的 `value` 属性指定数据源路径：

```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {...}
```

