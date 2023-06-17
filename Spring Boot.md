# Spring Boot

> version 2.6.1 

# 开始使用

基于 Maven 构建，通过父项目获得 Spring Boot 的各种功能。

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
</parent>
```

这是一个 Web 服务，引入相关依赖，主要是 Spring MVC 的东西。

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

编写代码，包括 Spring Boot 的启动方法和一个 Handler 方法。

```
@RestController
@EnableAutoConfiguration
public class MyApplication {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

> 注解 `@EnableAutoConfiguration` 告诉 Spring 的 Config 类根据依赖自动配置 Spring 组件。
>
> 方法 `main` 是项目的入口，它调用 `SpringApplication.run()` 引导 Spring Boot 的启动，第一个参数是项目的根组件，第二个参数是启动的命令行参数。

执行 `main` 方法，或通过 `run` 目标启动项目。

```
mvn spring-boot:run
```

把 Spring Boot 项目打包成能独立运行的 `Jar` 文件，引入打包插件。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

执行打包命令。

```
mvn package
```

运行生成的文件，效果应该与前面的直接运行相同。

```
java -jar target/project-0.0.1-SNAPSHOT.jar
```

# 基本概念

## 依赖管理

`spring-boot-starter-parent` 项目的每个版本，都会提供 Spring Boot 支持的所有依赖的版本列表。在实际开发中，大部分的依赖就不需要声明版本号，默认设置就能与项目完美兼容。

这减少了程序开发中各种依赖版本冲突的问题，也可以显式地声明版本号，这会覆盖 Spring Boot 管理的版本。

查看 `spring-boot-starter-parent` 内容，发现它有一个父项目 `spring-boot-dependencies`，再查看这个父项目。此时，可以发现这个根项目的 `<dependencyManagement>` 标签中管理有大量的依赖，这就是 Spring Boot 进行版本管理的原理。也就是简单的 POM 依赖管理而已。

## starter 包

Spring Boot 还提供大量 `Starter` 类型的包。这种包不是简单的第三方依赖，而是用于特定开发场景的整套依赖集合，比如 `spring-boot-starter-web` 就提供 Spring 与 web 开发相关的所有依赖。这种一站式的服务，引入后就能直接进行开发，极大的方便了依赖管理。

官方提供的 `Starter` 命名规范是 `spring-boot-starter-*`。自定义的 `Starter`，通常命名为 `*-spring-boot-starter`。

`spring-boot-starter-parent` 除提供版本管理功能外，还包含所有 Spring Boot 内置的 auto-configuration。因此它是所有 Spring Boot 项目的父模块。

## 自动配置 Auto-onfiguration

Spring Boot 会根据项目添加的依赖自动配置 Spring。在 Configuration 类上标注 `@SpringBootApplication` 或 `@EnableAutoConfiguration` 可以开启自动配置。自动配置注解通常是标注在 Spring Boot 程序的主类。

自动配置是非侵入式的。任何时候，都可以定义自己的配置以替换自动配置的特定部分。

如果需要了解程序使用了哪些 auto-configuration，可以使用 `--debug` 命令行参数启动程序，这会在控制台打印相关的日志信息。

如果不想使用程序的某些自动配置类，可以显式排除它们：

```
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApplication {

}
```

`@EnableAutoConfiguration` 也含有作用相同的 `exclude` 属性。如果类路径不存在目标类，可以使用 `excludeName` 属性指定要排除的自动配置类的完全限定名。

还可以使用 Spring Boot 的配置属性 `spring.autoconfigure.exclude` 指定要排除的自动配置类列表。

> 尽管自动配置类都是 `public`，但它们唯一能被使用的就是用于 `excludeName` 的类名。其它的部分如内部类、方法等，都只能在它的内部使用。强烈建议不要使用自动配置类的这些内容。

## @SpringBootApplication

第一章的范例工程使用 `@EnableAutoConfiguration` 标注 Spring Boot 入口，但实际开发中更多的是使用 
`@SpringBootApplication`。它的定义如下：

```
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
public @interface Spring BootApplication {...}
```

所以，`@SpringBootApplication` 主要有以下功能：

* `@EnableAutoConfiguration`，开启自动配置
* `@ComponentScan`，组件扫描，默认扫描标注类所在的包及其子包
* `@SpringBootConfiguration`，允许在容器中注册额外的 Bean 或导入额外的配置类，Spring 标准 `@Configuration` 的替代方案，可帮助在集成测试中进行配置检测。

以上功能除自动配置都不是 Spring Boot 必需的，可用 `@EnableAutoConfiguration` 组合其它注解以替代 `@SpringBootApplication`。如下所示：

```
@SpringBootConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ SomeConfiguration.class, AnotherConfiguration.class })
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

关于 `@SpringBootConfiguration`，它的定义如下：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

所以它其实是增强版的 `@Configuration`。

## Main Application Class

通常建议将 Spring Boot 的主程序类或称为入口类，放在程序的根目录。使用 `@SpringBootApplication` 标注它。这样，Spring Boot 程序会扫描整个项目的所有组件，因为 `@SpringBootApplication` 会扫描标注类的包及其子包。

# 核心功能

## SpringApplication

`SpringApplication` 类提供一种方便的方式从 `main()` 方法开始引导 Spring 程序启动。大部分情况，委托静态方法 `run()`。如下所示：

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.Spring BootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

### Startup Failure

如果程序启动失败，已注册的 `FailureAnalyzers` 会尝试提供专门的错误信息和解决问题的具体操作。比如，端口 8080 被占用，会打印以下消息：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that is listening on port 8080 or configure this application to listen on another port.
```

> Spring Boot 已经提供许多 `FailureAnalyzers` 是实现，也可以添加自定义的 `FailureAnalyzers`。

如果没有 `FailureAnalyzers` 能处理异常，你仍然可以显示完整的条件报告以更好地了解问题。为此，你需要为 `org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` 启用 `debug` 属性或 `DEBUG` 日志。比如，运行 `Jar` 时，可以按以下方式启用 `debug` 属性：

```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### Lazy Initialization

`SpringApplication` 允许程序懒加载。开启懒加载后，bean 只在被使用时加载，而非程序启动时，这有点类似 `prototype` 作用域。懒加载能减少程序启动时间。对于 web 程序，懒加载会使与 web 相关的 bean 在首次 HTTP 请求时才被初始化。

懒加载的一个副作用是：某些错误可能很晚才被发现。比如，某配置错误的 bean 是懒加载，那么只有当它被初始化时才会发生错误。此外，懒加载使得 JVM 无法确定足够的内存，因为不确定到底需要加载多少 bean。因此，默认不开启懒加载，并且建议开启懒加载前设置合适的 JVM 堆大小。

懒加载可以通过编程开启，使用 `SpringApplicationBuilder` 的 `lazyInitialization` 方法，或者 `SpringApplication` 的 `setLazyInitialization` 方法。或者，使用下面的配置属性：

```
spring.main.lazy-initialization=true
```

启用懒加载后，可以使用 `@Lazy(false)` 标注特定的 bean，以将它们的 `lazy` 属性设置为 `false`，这些 bean 会在程序启动时初始化。

### Customizing Banner

Spring Boot 启动时的 `banner` 图案可以进行修改，在类路径添加 `banner.txt` 文件，或者设置配置配置 `spring.banner.location` 为文件路径。如果文件编码不是 `UTF-8`，可以设置配置属性`spring.banner.charset`。

除文本文件外，还可以在类路径添加 `banner.gif`、`banner.jpg`、`banner.png` 这些图片文件，或者设置配置属性 `spring.banner.image.location`。图片会被转换为 ASCII 形式打印并覆盖文本 `banner`。

可以使用 `spring.main.banner-mode` 配置属性设置 `banner` 是否打印到 `System.out`（`console`），发送到配置的 `logger`（`log`），或根本不使用（`off`）。

打印的 `banner` 被注册为 `singleton` bean，名为 `Spring BootBanner`。

### Customizing SpringApplication

如果 `SpringApplication` 的默认配置不合你胃口，可以创建本地实例以自定义它。比如，下例将关闭 `banner`：

```
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }

}
```

> `SpringApplication` 的构造参数是 beans 配置源，通常是 `@Configuration`，也可以传入 `@Component`。

这样自定义 `SpringApplication` 的方式不常用，更多是使用 `application.properties/yaml` 文件。

### SpringApplicationBuilder

你如果想构建具有层次结构的 `ApplicationContext` 或者喜欢使用流畅的构建 API，可以考虑使用 `SpringApplicationBuilder`。`SpringApplicationBuilder` 允许将多个方法调用（包括用于创建层次结构上下文的 `parent` 和 `child` 方法）联合在一块，如下所示：

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

可以将 `SpringApplicationBuilder` 看作特别的 `SpringApplication`，不怎么常用。

> 创建层次结构的上下文具有一些限制。比如，web 组件必须包含在子上下文，父子上下文必须使用相同的 `Environment`。

### Web Environment

`SpringApplication` 尝试创建正确类型的 `ApplicationContext`，根据以下规则进行判断：

* 如果存在 Spring MVC，使用 `AnnotationConfigServletWebServerApplicationContext`；
* 如果存在 Spring WebFlux 且 Spring MVC 不存在，使用 `AnnotationConfigReactiveWebServerApplicationContext`；
* 使用 `AnnotationConfigApplicationContext`。

这意味着，如果程序同时存在 Spring WebFlux 和 Spring MVC，默认使用 Spring MVC。可以调用 `setWebApplicationType(WebApplicationType)` 覆盖此设置。

或者调用 `setApplicationContextClass(…)` 完全控制上下文类型。

### Accessing Application Arguments

如果需要访问传给 `SpringApplication.run()` 的应用参数，可以注入 `org.springframework.boot.ApplicationArguments`。`ApplicationArguments` 接口提供方法用以访问 `String[]` 参数以及已经解析的 `option` 参数和 `non-option` 参数。如下所示：

```
@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

> Spring Boot 还向 Spring `Environment` 注册一个 `CommandLinePropertySource`，这使得你能使用 `@Value` 注入单个应用参数。

### ApplicationRunner and CommandLineRunner

如果想在 `SpringApplication` 启动后运行一些特定的代码，可以实现 `ApplicationRunner` 或 `CommandLineRunner` 接口。它们的运行方式相同，且都只提供 `run` 方法，在 `SpringApplication.run(…)` 完全结束前调用。

> 这两个接口适合用于定义要在程序启动后，开始接收流量前运行的任务。

`CommandLineRunner` 接口能以字符串数组的形式访问应用参数，而 `ApplicationRunner` 需要使用前面提到的  `ApplicationArguments`。下面实现 `CommandLineRunner` 接口：

```
@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // Do something...
    }

}
```

如果有多个 `CommandLineRunner` 或 `ApplicationRunner`，且必须按特定的顺序运行。可以让它们实现 `org.springframework.core.Ordered` 或者使用 `org.springframework.core.annotation.Order` 注解。

## Externalized Configuration

Spring Boot 允许外部化配置，因此你可以使用相同的代码在不同的 Spring 环境下运行。可以使用各种外部配置源，包括 Java properties 文件，YAML 文件，环境变量，命令行参数。

因为外部配置的属性最终都会被添加到 Spring `Environment`，所以将它们称作环境属性或配置属性。

可以使用 `@Value` 直接注入环境属性，或使用 `Environment` 抽象访问，或通过 `@ConfigurationProperties` 绑定到结构化对象。

Spring Boot 使用特定的 `PropertySource` 读取顺序，以对属性值进行合理的覆盖。按以下顺序考虑属性源（后面的值覆盖前面）：

1. 默认属性，通过设置 `SpringApplication.setDefaultProperties` 指定；
2. `@Configuration` 类标注的 `@PropertySource`。注意，这种属性源在 Spring 上下文刷新前不会被添加到 `Environment`。所以某些需要在 refresh 前读取的属性不适合它。
3. config data，如 `application.properties` 文件；
4. `RandomValuePropertySource`，它仅有 `random.*` 中的属性；
5. 系统环境变量；
6. Java 系统属性（`System.getProperties()`）；
7. 来自 `java:comp/env` 的 JNDI 属性；
8. `ServletContext` 初始化参数；
9. `ServletConfig` 初始化参数；
10. 来自 `SPRING_APPLICATION_JSON` 的属性（嵌入环境变量或系统属性的内联 JSON）；
11. 命令行参数；
12. 测试的 `properties` 属性。在 `@Spring BootTest` 和用于测试特定程序片段的注解上可用；
13. 测试的 `@TestPropertySource`；
14. devtools 激活时，`$HOME/.config/spring-boot` 文件夹中 devtools 的全局设置属性。

将按以下顺序考虑 config data：

1. `Jar` 包内的应用属性（`application.properties|yaml`）；
2. `Jar` 包内特定的 profile 应用属性（`application-{profile}.properties|yaml`）；
3. `Jar` 包外的应用属性；
4. `Jar` 包外特定的 profile 应用属性；

> 应该使用统一的 config data 格式，如果 `.properties` 和 `.yaml` 同时存在，优先考虑 `.properties`。

### Accessing Command Line Properties

默认情况，`SpringApplication` 会将命令行 `option ` 参数（以 `--` 开头的参数，如 `--server.port=9000`）转换为 `property` 并添加到 Spring `Environment`。前面也提及过，命令行参数优先于 config data。

如果不想让 `option ` 参数被添加到 `Environment`，可以调用 `SpringApplication.setAddCommandLineProperties(false)` 禁用此行为。

### JSON Application Properties

环境变量和系统属性具有局限性，许多属性名无法使用。为解决此问题，Spring Boot 允许将属性块编码为单个 JSON 结构。

程序启动时，任何 `spring.application.json` 或 `SPRING_APPLICATION_JSON` 属性会被解析并添加到 `Environment`。

比如，`SPRING_APPLICATION_JSON` 属性可以在 UNIX shell 的命令行中提供为环境变量：

```
$ SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

上例最终会在 Spring `Environment` 添加 `my.name=test`。

还能使用 Java 系统属性提供相同的 JSON 数据：

```
$ java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

或者使用命令行参数：

```
$ java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

> 尽管 JSON 的 `null` 会添加到生成的属性源，但 `PropertySourcesPropertyResolver` 会将 `null` 视为缺失，这意味 JSON 的 `null` 值不能从低位覆盖前面的属性值。

### External Application Properties

程序启动时，Spring Boot 会自动从以下位置查找和加载 `application.properties` 和 `application.yaml`：

* 类路径
  * 根目录
  * `/config` 目录
* 当前应用目录
  * 当前目录
  * 当前目录的 `/config` 子目录
  * `/config` 子目录的直接子目录

根据上面的顺序，后面的值覆盖前面。加载的文档会作为 `PropertySources` 添加到 Spring `Environment`。

配置文件名默认是 `application`，如果你不喜欢，可以使用环境属性 `spring.config.name` 修改。比如，下例使应用将查找名为 `myproject.properties` 和 `myproject.yaml` 的配置文件：

```
$ java -jar myproject.jar --spring.config.name=myproject
```

还可以使用环境属性显式指定配置文件位置，多个位置使用 `,` 分隔。下例指定两个位置：

```
$ java -jar myproject.jar --spring.config.location=\
    optional:classpath:/default.properties,\
    optional:classpath:/override.properties
```

> 如果配置文件位置可选，可有可无，需要使用前缀 `optional:`。

> `spring.config.name`，`spring.config.location`，和 `spring.config.additional-location` 会很被使用以确定加载的文件。它们必须定义为环境属性，通常是环境变量、系统属性或命令行参数。

`spring.config.location` 指定的如果是目录的位置，必须以 `/` 结尾。运行时，将会在加载它之前追加 `spring.config.name` 生成的文件名。如果是文件，则直接导入。

> 目录或文件位置的值都会拓展检查特定的 profile 文件。比如，使用 `spring.config.location` 指定位置 `classpath:myconfig.properties`，此时还会加载 `classpath:myconfig-<profile>.properties`。

大部分情况，添加的每个 `spring.config.location` 都引用单个文件或目录。将按定义的顺序处理这些位置，后面的值覆盖前面。

如果有较复杂的位置设置，并且使用特定的 profile 配置文件，你可能需要提供进一步的提示，以便 Spring Boot 知道如何对它们进行分组。位置分组是相同级别位置的集合。比如，对所有类路径的位置进行分组，然后对其它外部位置进行分组。分组中的位置使用 `;` 分隔。详情参看 Profile Specific Files 章节。

`spring.config.location` 指定的位置会替换默认位置。比如，设置 `spring.config.location` 为 `optional:classpath:/custom-config/,optional:file:./custom-config/`，将按以下顺序查找配置文件：

* `optional:classpath:custom-config/`
* `optional:file:./custom-config/`

如果只是想添加额外的位置而非替换，可以使用 `spring.config.additional-location`。从额外位置读取的属性值会覆盖默认位置内容。比如，设置 `spring.config.additional-location` 为  `optional:classpath:/custom-config/,optional:file:./custom-config/`，将按以下顺序查找配置文件：

* `optional:classpath:/;optional:classpath:/config/`
* `optional:file:./;optional:file:./config/;optional:file:./config/*/`
* `optional:classpath:custom-config/`
* `optional:file:./custom-config/`

这种查找顺序的好处是：你可以先在默认文件 `application.properties` 提供默认属性，然后在运行时使用特定位置的配置文件选择性地覆盖这些属性值。

> 如果使用环境变量而非系统属性，大部分操作系统不支持 `.` 作为键名，可以使用 `_` 替代。比如使用 `SPRING_CONFIG_NAME` 替代 `spring.config.name`。

> 如果程序运行在 servlet 容器或应用服务器，可以使用 JNDI 属性（`java:comp/env`）或 servlet 容器初始化参数替代环境变量或系统属性。

#### Optional Locations

默认情况，指定的 config data 位置不存在文件时，Spring Boot 抛出 `ConfigDataLocationNotFoundExceptions` 异常，程序不会启动。

如果你想指定 config data 文件位置，但不在乎文件是否真的存在，你可以考虑使用 `optional:` 前缀。这个前缀能和 `spring.config.location`，`spring.config.additional-location` 以及 `spring.config.import` 一起使用。

比如，设置 `spring.config.import` 为 `optional:file:./myconfig.properties`，即使 `myconfig.properties` 文件不存在，程序也能正常启动。

如果想忽略所有 `ConfigDataLocationNotFoundExceptions` 异常继续启动程序，可以通过 “环境变量/系统属性“ 或 `SpringApplication.setDefaultProperties(…)` 设置 `spring.config.on-not-found` 为 `ignore`。

#### Wildcard Locations

如果配置文件位置的最后一个路径片段包含 `*` 字符，它将被视为通配符位置。加载配置时会拓展通配符，所以直接子目录也会被检查。

比如，有 Redis 配置和 MySQL 配置，想把它们分开设置，但它们又必须都放在 `application.properties` 文件。这就造成两个独立的配置文件分别挂载在不同目录，如 `/config/redis/application.properties` 和 `/config/mysql/application.properties`。这种情况，使用通配符位置 `config/*/` 能同时处理它们。

默认情况，Spring Boot 的检索位置包含 `config/*/`。这意味 `Jar` 包外 `/config` 目录的所有子目录都会被检索。

可以将通配符位置和 `spring.config.location` 和 `spring.config.additional-location` 一起使用。

> 通配符位置只能包含一个 `*`。以 `*/` 结尾搜索目录，以 `*/<filename>` 结尾搜索文件。含有通配符的位置将根据文件绝对路径按字母顺序排列。

> 通配符位置仅适用于外部目录，不适用 `classpath:`。这也解释虽然 Spring Boot 搜索位置包含 `config/*/`，但却只搜索 `Jar` 包外的 `/config` 目录的子目录。

#### Profile Specific Files

除 `application` 配置文件，Spring Boot 还会尝试加载约定命名为 `application-{profile}` 的 profile 文件。如果你的程序激活 `prod` profile 并使用 YAML 格式，将会同时考虑 `application.yml` 和 `application-prod.yml` 文件。

profile 文件从与标准 `application.properties` 文件相同的位置加载，并且 profile 的属性会覆盖标准配置文件。如果激活多个 profile 且重复设置相同的属性，则最后的 profile 获胜。比如，使用环境属性 `spring.profiles.active` 激活 `prod,live`，`application-prod.properties` 的属性会覆盖 `application-live.properties` 的同名属性。

> 最后获胜策略在位置分组时，覆盖规则有所不同。比如，现在激活 `prod,live` profile，有以下配置文件：
>
> ```
> /cfg
>   application-live.properties
> /ext
>   application-live.properties
>   application-prod.properties
> ```
>
> 如果配置 `spring.config.location=classpath:/cfg/,classpath:/ext/`，会优先处理 `/cfg` 目录的所有文件：
>
> 1. `/cfg/application-live.properties`
> 2. `/ext/application-prod.properties`
> 3. `/ext/application-live.properties`
>
> 如果配置 `classpath:/cfg/;classpath:/ext/` 位置分组时（`;`），`/cfg` 和 `/ext` 优先级相同，那么同名属性的覆盖顺序将不确定：
>
> 1. `/ext/application-prod.properties`
> 2. `/cfg/application-live.properties`
> 3. `/ext/application-live.properties`

`Environment` 拥有一组默认的 profile（默认是 `[default]`），如果未显式激活其它 profile，则会使用此 profile 文件，如 `application-default`。

> 属性文件只会加载一次，如果已经导入某个 profile 配置文件，则不会再次加载它。

#### Importing Additional Data

应用属性可以进一步使用 `spring.config.import` 属性引入其他位置的 config data。导入会在发现时被处理，并将导入文档视为声明导入的文档下插入的额外文档。

比如，在 `application.properties` 声明以下内容：

```
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

这将导入当前应用目录的 `dev.properties` 文件（如果存在）。`dev.properties` 定义的属性优先于声明导入的文档。上例中，`dev.properties` 能够重新赋值 `spring.application.name`。

一个导入无论声明多少次，都只会执行一次。properties/yaml 文档中，导入的定义顺序无关紧要。下面两个示例的效果是相同的：

```
spring.config.import=my.properties
my.property=value
```

```
my.property=value
spring.config.import=my.properties
```

可以在 `spring.config.import` 键下定义多个位置，这些位置将按照定义顺序进行处理，后面的导入优先级更高。

> 适当情况，还会考虑导入 profile 文件。比如上例就可能导入 `my-<profile>.properties`。

> Spring Boot 包含可插拔的 API 以支持各种不同类型的位置。默认情况，可以导入 Java Properties，YAML 和 "configuration trees"。
>
> 第三方 `Jar` 包可以提供额外的技术支持（不要求是本地文件）。比如，config data 可能储存在外部的 Zookeeper，Consul 等。
>
> 如果想支持自己的位置，可以查看 `org.springframework.boot.context.config` 包的 `ConfigDataLocationResolver` 和 `ConfigDataLoader` 类。

#### Importing Extensionless Files

某些云平台无法向卷挂载的文件添加拓展名。为导入这些没有拓展名的文件，需要给 Spring Boot 提示，以让它知道如何加载这些文件。你可以在方括号（`[]`）中添加拓展来进行提示。

比如，你想要导入 YAML 文件 `/etc/config/myconfig`，可以在 `application.properties` 定义以下内容：

```
spring.config.import=file:/etc/config/myconfig[.yaml]
```

#### Using Configuration Trees

When running applications on a cloud platform (such as Kubernetes) you often need to read config values that the platform supplies. It is not uncommon to use environment variables for such purposes, but this can have drawbacks, especially if the value is supposed to be kept secret.

As an alternative to environment variables, many cloud platforms now allow you to map configuration into mounted data volumes. For example, Kubernetes can volume mount both `ConfigMaps` and `Secrets`.

There are two common volume mount patterns that can be used:

1. A single file contains a complete set of properties (usually written as YAML).
2. Multiple files are written to a directory tree, with the filename becoming the ‘key’ and the contents becoming the ‘value’.

For the first case, you can import the YAML or Properties file directly using `spring.config.import` as described above. For the second case, you need to use the `configtree:` prefix so that Spring Boot knows it needs to expose all the files as properties.

As an example, let’s imagine that Kubernetes has mounted the following volume:

```
etc/
  config/
    myapp/
      username
      password
```

The contents of the `username` file would be a config value, and the contents of `password` would be a secret.

To import these properties, you can add the following to your `application.properties` or `application.yaml` file:

```
spring.config.import=optional:configtree:/etc/config/
```

You can then access or inject `myapp.username` and `myapp.password` properties from the `Environment` in the usual way.

> Filenames with dot notation are also correctly mapped. For example, in the above example, a file named `myapp.username` in `/etc/config` would result in a `myapp.username` property in the `Environment`.

> Configuration tree values can be bound to both string `String` and `byte[]` types depending on the contents expected.

If you have multiple config trees to import from the same parent folder you can use a wildcard shortcut. Any `configtree:` location that ends with `/*/` will import all immediate children as config trees.

For example, given the following volume:

```
etc/
  config/
    dbconfig/
      db/
        username
        password
    mqconfig/
      mq/
        username
        password
```

You can use `configtree:/etc/config/*/` as the import location:

```
spring.config.import=optional:configtree:/etc/config/*/
```

This will add `db.username`, `db.password`, `mq.username` and `mq.password` properties.

> Directories loaded using a wildcard are sorted alphabetically. If you need a different order, then you should list each location as a separate import

Configuration trees can also be used for Docker secrets. When a Docker swarm service is granted access to a secret, the secret gets mounted into the container. For example, if a secret named `db.password` is mounted at location `/run/secrets/`, you can make `db.password` available to the Spring environment using the following:

```
spring.config.import=optional:configtree:/run/secrets/
```

#### Property Placeholders

`application.properties` 和 `application.yml` 中的值在使用时会经过现有 `Environment` 过滤，所以你能在 config data 文件中引用前面定义的属性。可以在值的任何位置使用占位符 `${name}`。如下所示：

```
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

#### Working with Multi-Document Files

Spring Boot 允许将单个物理文件拆分为多个逻辑文档，每个文档都是独立添加的。文档按顺序从上到下处理，后面的文档覆盖前面。

对于 `application.yml` 文件，使用标准的 YAML 多文档标记。三个连续的连字符 `-` 表示一个文档的结尾和另一个文档的开头。下例展示两个逻辑文档：

```
spring:
  application:
    name: MyApp
---
spring:
  application:
    name: MyCloudApp
  config:
    activate:
      on-cloud-platform: kubernetes
```

对于 `application.properties`，使用特别的注释 `#---` 分隔文档：

```
spring.application.name=MyApp
#---
spring.application.name=MyCloudApp
spring.config.activate.on-cloud-platform=kubernetes
```

> 文档分隔符不能有任何前导空格，并且必须是精确的三个连字符。分隔符前后行不能是注释。

> 多文档属性文件不能使用 `@PropertySource` 或 `@TestPropertySource` 注解加载。

#### Activation Properties

如果想让某些属性仅当环境满足特定条件时才被使用，可以考虑使用 `spring.config.activate.*` 有条件地激活属性文档，它主要有以下两个激活属性：

| Property            | Note                                       |
| :------------------ | :----------------------------------------- |
| `on-profile`        | profile 表达式，只有匹配时才激活当前文档。 |
| `on-cloud-platform` | 必须检测到某云平台才激活当前文档。         |

下面的第二个逻辑文档，只有当程序运行在 `kubernetes` 云平台，且激活 `prod` 或 `staging` profile 时，才被激活：

```
myprop=always-set
#---
spring.config.activate.on-cloud-platform=kubernetes
spring.config.activate.on-profile=prod | staging
myotherprop=sometimes-set
```

### Encrypting Properties

Spring Boot 不提供对加密属性的内置支持，但它含有用于修改 Spring `Environment` 包含的值的勾子。`EnvironmentPostProcessor` 接口允许你在程序启动前操作 `Environment`。

### Working with YAML

YAML 是 JSON 的超集，是指定具有分层结构的配置数据的方便格式。当类路径中存在 `SnakeYAML` 库时，`SpringApplication` 类自动支持 YAML 格式以替代 properties。

> `spring-boot-starter` 模块自动提供 `SnakeYAML `。

#### Mapping YAML to Properties

YAML 文档的分层格式需要转换为平面结构，这样才能能用在 `Environment`。考虑下面的 YAML 内容：

```
environments:
  dev:
    url: https://dev.example.com
    name: Developer Setup
  prod:
    url: https://another.example.com
    name: My Cool App
```

为从 `Environment` 访问这些属性，它们被扁平化为以下形式：

```
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

同样，YAML 列表也需要扁平化，表现为带有 `[index]` 的属性键。考虑下面的 YAML 内容：

```
my:
 servers:
 - dev.example.com
 - another.example.com
```

上面的内容会被转换为以下形式：

```
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

> 可以使用 Spring Boot 的 `Binder` 把带有 `[index]` 标记的属性绑定到 `List` 或 `Set`。

> 不能使用 `@PropertySource` 或 `@TestPropertySource` 注解加载 YAML 文件。

#### Directly Loading YAML

Spring Framework 提供两个类用于加载 YAML 文档。`YamlPropertiesFactoryBean` 将 YAML 加载为 `Properties`，`YamlMapFactoryBean` 将 YAML 加载为 `Map`。你可以使用前者加载 YAML 为 `PropertySource`。

### Configuring Random Values

`RandomValuePropertySource` 可用于注入随机值，它能产生整数、长整、UUID 和字符串。如下所示：

```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number-less-than-ten=${random.int(10)}
my.number-in-range=${random.int[1024,65536]}
```

`random.int*` 的语法是 `OPEN value (,max) CLOSE`，其中 `OPEN,CLOSE` 可以是任意字符，`value,max` 是整数。如果提供 `max`，那么 `value` 就是最小数，`max` 就是最大数。如果不提供，最大数无限。

### Configuring System Environment Properties

Spring Boot 支持为环境属性设置前缀。这在系统环境被具有不同配置要求的多个 Spring Boot 程序共享时很有用。环境属性的前缀可以直接在 `SpringApplication` 设置。

比如，将前缀设为 `input`，则系统环境中的 `remote.timeout` 属性会被解析为 `input.remote.timeout`。

### Type-safe Configuration Properties

使用 `@Value("${property}")` 注入配置属性有时可能会很麻烦，尤其是在使用多个属性或数据是分层结构的时候。Spring Boot 提供另一种使用属性的方法，它允许强类型 bean 管理和验证应用的配置属性。

#### JavaBean properties binding

可以使用 `@ConfigurationProperties` 绑定声明标准 JavaBean 属性的 bean：

```
@ConfigurationProperties("my.service")
public class MyProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    // getters / setters...

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        // getters / setters...

    }

}
```

上面的 POJO 定义以下的属性：

- `my.service.enabled`，默认值 `false`；
- `my.service.remote-address`，可以从 `String` 强制转换；
- `my.service.security.username`，包含在嵌套的 `security` 对象，其名称由属性确定；
- `my.service.security.password`；
- `my.service.security.roles`，默认值为单个 `USER` 的 `String` 集合。

> 映射到 `@ConfigurationProperties` 类的属性是公共 API，但这不意味可以直接使用类的访问器。
>
> 这些属性可以通过 properties 文件，YAML 文件，环境变量和命令行参数等设置。

> 这种安排方式依赖于默认的空参构造器，并且 getter、setter 方法通常也是必须的。因为绑定使用标准的 Java Beans 属性描述器，就像在 Spring MVC 中一样。以下情况可以省略 setter：
>
> - Maps, as long as they are initialized, need a getter but not necessarily a setter, since they can be mutated by the binder.
> - Collections and arrays can be accessed either through an index (typically with YAML) or by using a single comma-separated value (properties). In the latter case, a setter is mandatory. We recommend to always add a setter for such types. If you initialize a collection, make sure it is not immutable (as in the preceding example).
> - If nested POJO properties are initialized (like the `Security` field in the preceding example), a setter is not required. If you want the binder to create the instance on the fly by using its default constructor, you need a setter.
>
> 有些人会使用 Lombok 自动添加 getter 和 setter。请确保 Lombok 没有为此类型生成特定的构造器，因为容器会自动使用它来实例化对象。
>
> 最后，绑定只会考虑标准的 Java Bean 属性，静态属性是不被支持的。

#### Constructor binding

上节的示例可以使用以下不可改变的方式重写：

```
@ConstructorBinding
@ConfigurationProperties("my.service")
public class MyProperties {

    // fields...

    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    // getters...

    public static class Security {

        // fields...

        public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        // getters...

    }

}
```

在此设置中，`@ConstructorBinding` 指示使用构造器绑定。这意味 binder 需要找到一个构造器，它的构造参数都是需要绑定的属性。在 Java 16 或更高版本中，构造器绑定能与 records 一起使用。这种情况，除非 records 含有多个构造器，否则不需要使用 `@ConstructorBinding`。

`@ConstructorBinding` 类的嵌套对象，如示例中的 `Security`，也会使用它们自己的构造器进行绑定。

可以使用 `@DefaultValue` 指定默认值，并且会使用相同的 conversion 将 `String` 值强制转换为缺失属性的目标类型。默认情况，如果没有属性绑定到 `Security`，`MyProperties` 的 `Security` 值为 `null`。如果希望即使没有绑定属性，也返回非 `null` 的实例，可以使用空的 `@DefaultValue`，如下所示：

```
public MyProperties(boolean enabled, InetAddress remoteAddress, @DefaultValue Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
}
```

> 要使用构造器绑定，必须使用 `@EnableConfigurationProperties` 或配置属性扫描以启用此功能。你不能对那些使用常规 Spring 机制创建的 bean 进行构造器绑定，比如 `@Component`，`@Bean` 或 `@Import`。

> 如果包含多个构造器，可以使用 `@ConstructorBinding` 直接标注用于绑定的构造器。

> 不建议将 `java.util.Optional` 与 `@ConfigurationProperties` 合用，`Optional` 毕竟主要用于返回类型。为与其它类型的属性保持一致，如果声明的 `Optional` 没有值，也会绑定 `null` 而非空 `Optional`。

#### Enabling @ConfigurationProperties-annotated types

Spring Boot 提供基础结构绑定 `@ConfigurationProperties` 类型并将它们注册为 bean。你可以使用 `@ConfigurationProperties` 逐个类启用，或者使用配置属性扫描（工作方式类似于组件扫描）。

有时，`@ConfigurationProperties` 标注的类可能不适合扫描。比如你正在开发自己的 auto-configuration 或者想有条件地启用它。这种时候最好使用 `@EnableConfigurationProperties` 列举需要处理的类型。这可以在任何 `@Configuration` 类完成，如下所示：

```
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {

}
```

要使用配置属性扫描，在程序中添加 `@ConfigurationPropertiesScan` 注解。通常是标注在 `@SpringBootApplication` 主类，当然也可以标注其它任意的 `@Configuration` 类。默认情况，扫描从标注类的包开始，如果想自定义扫描的包，可以进行以下操作：

```
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {

}
```

> 当使用配置属性属性扫描或者 `@EnableConfigurationProperties` 注册 `@ConfigurationProperties` bean 时，该 bean 约定命名为 `<prefix>-<fqn>`，`<prefix>` 是 `@ConfigurationProperties` 指定的环境前缀，`<fqn>` 是 bean 的完全限定类名。如果没有提供前缀，则只使用完全限定类名。
>
> 上面示例的 bean 名为 `com.example.app-com.example.app.SomeProperties`。

建议 `@ConfigurationProperties` 只处理环境，特别是不要从容器中注入其它 bean。对于极端情况，可以使用 setter 注入或其它的 `*Aware` 接口。如果仍然想使用构造器注入其它 bean，`@ConfigurationProperties` 类必须标注 `@Component` 且使用基于 JavaBean 的属性绑定。注意，既然已经是 `@Component`，就不能再使用构造器绑定配置属性。

#### Using @ConfigurationProperties-annotated types

这种配置样式特别适合用于 `SpringApplication` 拓展的 YAML。下面是 YAML 内容：

```
my:
  service:
    remote-address: 192.168.1.1
    security:
      username: "admin"
      roles:
      - "USER"
      - "ADMIN"
```

若要使用 `@ConfigurationProperties` bean，可以像普通的 bean 一样进行注入它们。如下所示：

```
@Service
public class MyService {

    private final SomeProperties properties;

    public MyService(SomeProperties properties) {
        this.properties = properties;
    }

    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        server.start();
        // ...
    }

    // ...

}
```

> `@ConfigurationProperties` 还允许你生成元数据文件，IDE 可以使用这些文件为你的密钥提供自动完成功能。

#### Third-party Configuration

`@ConfigurationProperties` 除用于类外，还能标注公共的 `@Bean` 方法。当你想将配置属性绑定到无法控制的第三方组件时，这很有用。

若要使用 `Environment` 属性配置 bean，请用 `@ConfigurationProperties` 标注它的注册方法，如下所示：

```
@Configuration(proxyBeanMethods = false)
public class ThirdPartyConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "another")
    public AnotherComponent anotherComponent() {
        return new AnotherComponent();
    }

}
```

任何包含 `another` 前缀的配置属性都会以类似前面 `SomeProperties` 的方式映射到 `AnotherComponent`。

#### Relaxed Binding

Spring Boot 使用宽松的规则绑定 `Environment` 属性到 `@ConfigurationProperties` bean，因此配置属性与类属性并不需要完全匹配名称。下划线分隔的或大写的配置属性都能成功匹配。考虑下面的 bean 定义：

```
@ConfigurationProperties(prefix = "my.main-project.person")
public class MyPersonProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

根据上面的内容，以下格式的属性都能成功映射到它：

| Property                            | Note                             |
| :---------------------------------- | :------------------------------- |
| `my.main-project.person.first-name` | Kebab 风格，使用 `-` 分隔符。    |
| `my.main-project.person.firstName`  | 驼峰命名。                       |
| `my.main-project.person.first_name` | 使用下划线 `_` 分隔符。          |
| `MY_MAINPROJECT_PERSON_FIRSTNAME`   | 大写格式，一般用于定义环境变量。 |

> `@ConfigurationProperties` 的 `prefix` 属性必须使用 kebab 风格，即小写和 `-` 分隔符。

| Property Source       | Simple                                              | List                                                       |
| :-------------------- | :-------------------------------------------------- | :--------------------------------------------------------- |
| Properties Files      | Camel case, kebab case, or underscore notation      | Standard list syntax using `[ ]` or comma-separated values |
| YAML Files            | Camel case, kebab case, or underscore notation      | Standard YAML list syntax or comma-separated values        |
| Environment Variables | Upper case format with underscore as the delimiter. | Numeric values surrounded by underscores                   |
| System properties     | Camel case, kebab case, or underscore notation      | Standard list syntax using `[ ]` or comma-separated values |

> 如果可能，建议使用小写的 kebab 格式储存属性，如 `my.person.first-name=Rod`。

##### Binding Maps

绑定 `Map` 属性时，可能需要使用 `[]` 标记以保留键的原始内容。如果键没有被 `[]` 括起，会自动删除任何不是字母、数字、`-`、`.` 的字符。

比如，考虑将以下配置属性绑定到 `Map<String,String>`：

```
my.map.[/key1]=value1
my.map.[/key2]=value2
my.map./key3=value3
```

> 对于 YAML，括号需要使用引号括起，以便正确解析键。

上面的配置属性会绑定到 `Map`，其中键为 `/key1`，`/key2` 和 `key3`。注意，`key3` 没有斜杠，没有被 `[]` 括起。

如果键包含 `.` 且被绑定到非标量值，可能也需要使用 `[]`。比如，绑定 `a.b=c` 到 `Map<String, Object>`，将返回 `{"a"={"b"="c"}}`，但如果使用 `[a.b]=c`，则返回 `{"a.b"="c"}`。

##### Binding from Environment Variables

大多数操作系统对环境变量的名称具有严格的限制。比如，Linux shell 变量名只能包含字母，数字和 `_`，按照惯例，Unix shell 变量名需要大写。

Spring Boot 宽松的绑定规则尽可能地被设计与这些命名规则兼容。

要将规范的属性名转换为环境变量名，需要遵循以下规则：

- 使用 `.` 替换 `_`；
- 删除 `-`；
- 大写。

比如，配置属性 `spring.main.log-startup-info` 对映环境变量 `SPRING_MAIN_LOGSTARTUPINFO`。

还能使用环境变量绑定对象 `List`，此时元素编号在变量名中应该使用 `_` 括起。比如，配置属性 `my.service[0].other` 对映环境变量 `MY_SERVICE_0_OTHER`。

#### Merging Complex Types

如果在多个位置配置 `List`，覆盖机制会替换前面整个 `List`。

比如，`MyPojo` 对象拥有 `name` 属性和默认值为 `null` 的 `description` 属性。下面暴露 `MyPojo` 列表：

```
@ConfigurationProperties("my")
public class MyProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

考虑以下配置：

```
my.list[0].name=my name
my.list[0].description=my description
#---
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

如果 `dev` profile 未激活，`MyProperties.list` 包含一个 `MyPojo` 对象，定义如上。如果激活 `dev`，`list` 依然只包含一个 `MyPojo` 对象（`name` 等于 `my another name`，`description` 等于 `null`）。

如果有多个 profile 配置 `List`，使用最高优先级的 profile。考虑以下配置：

```
my.list[0].name=my name
my.list[0].description=my description
my.list[1].name=another name
my.list[1].description=another description
#---
spring.config.activate.on-profile=dev
my.list[0].name=my another name
```

在上面的示例中，如果激活 `dev` profile，`MyProperties.list` 包含一个 `MyPojo` 对象（`name` 等于 `my another name`，`description` 等于 `null`）。对于 YAML，`,` 分隔的列表或 YAML 列表都能覆盖列表内容。

对于 `Map` 属性，可以绑定从多个源获得的属性。但是，对于同个源的属性，只会使用优先级最高的。下面暴露 `Map<String, MyPojo>` 映射：

```
@ConfigurationProperties("my")
public class MyProperties {

    private final Map<String, MyPojo> map = new LinkedHashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

考虑以下配置：

```
my.map.key1.name=my name 1
my.map.key1.description=my description 1
#---
spring.config.activate.on-profile=dev
my.map.key1.name=dev name 1
my.map.key2.name=dev name 2
my.map.key2.description=dev description 2
```

如果 `dev` profile 未激活，`MyProperties.map` 包含一个条目。键为 `my name 1`，值为 `my description 1`。如果激活 `dev` profile，`map` 包含两个条目，键为 `dev name 1` 和 `dev name 2`，值为 `my description 1` 和 `dev description 2`。

`Map` 与 `List` 不同，`List` 会完全覆盖前面的 profile，而 `Map` 会合并多个 profile，只覆盖前面的特定部分。

> 这种合并规则适用于所有属性源的属性，而不仅是文件。

#### Properties Conversion

Spring Boot 绑定 `@ConfigurationProperties` bean 时，会尝试将外部的应用属性强制转换为正确的目标类型。如果需要自定义类型转换，可以提供名为 `conversionService` 的 `ConversionService` bean，或使用 `CustomEditorConfigurer` 自定义 property editor，或自定义 `Converters`（注册方法标注 `@ConfigurationPropertiesBinding`）。

> 因为此 bean 在 Spring Boot 生命周期的早期就被请求，所以请确保限制 `ConversionService` 使用的依赖。通常，所需的任何依赖在创建时期还未完全初始化。如果强制转换不需要自定义的 `ConversionService`，只依赖标注 `@ConfigurationPropertiesBinding` 的转换器，你可能会想重命名 `ConversionService`。

##### Converting Durations

Spring Boot 包含专门用于表示 `Duration` 的支持。如果想暴露 `java.time.Duration` 属性，应用属性可以使用以下格式：

- 常规的 `long`（默认单位毫秒，除非使用 `@DurationUnit` 指定）；
- `java.time.Duration` 使用的标准 ISO-8601 格式；
- 一种更具可读性的格式，其中值与单位在一起（`10s` 表示 10 秒）。

考虑以下示例：

```
@ConfigurationProperties("my")
public class MyProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    // getters / setters...

}
```

要指定会话超时为 30 秒，可以使用 `30`，`PT30S`，`30s`。读超时为 500 毫秒，可以使用 `500`，`PT0.5S`，`500ms`。

支持使用以下任意单位：

- `ns` 纳秒，千分之一微秒；
- `us` 微秒，千分之一毫秒；
- `ms` 毫秒，千分之一秒；
- `s` 秒；
- `m` 分；
- `h` 时；
- `d` 天。

默认单位时毫秒，可以使用 `@DurationUnit` 自定义，如上所示。

如果更喜欢构造器绑定，可以使用下面方式暴露相同的属性：

```
@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    // fields...

    public MyProperties(@DurationUnit(ChronoUnit.SECONDS) @DefaultValue("30s") Duration sessionTimeout,
            @DefaultValue("1000ms") Duration readTimeout) {
        this.sessionTimeout = sessionTimeout;
        this.readTimeout = readTimeout;
    }

    // getters...

}
```

> 如果要更新 `Long` 属性，注意使用 `@DurationUnit` 重新定义单位。这么做能提供透明的更新方式，同时支持更丰富的格式。

##### Converting periods

除 `Duration` 外，Spring Boot 还支持 `java.time.Period` 类型。如果想暴露 `Period` 属性，应用属性可以使用以下格式：

* 常规 `int`（默认单位天，除非使用 `@PeriodUnit` 指定）；
* `java.time.Period` 使用的标准 ISO-8601 格式；
* 一种更简单的格式，值和单位在一起（`1y3d` 表示 1 年 3 天）。

支持使用以下任意单位：

- `y` 年；
- `m` 月；
- `w` 周；
- `d` 天。

> `java.time.Period` 类型并不真正的存储周数，它只是 ”7 天“ 的缩写。

##### Converting Data Sizes

Spring Framework 含有一个自己的值类型 `DataSize`，以字节表示大小。如果想暴露 `DataSize` 属性，应用属性可以使用以下格式：

- 常规 `long`（默认单位字节，除非使用 `@DataSizeUnit` 指定）；
- 一种更具可读性的格式，值和单位在一起（`10MB` 表示 10 兆）。

考虑以下示例：

```
@ConfigurationProperties("my")
public class MyProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    // getters/setters...

}
```

要指定 10 兆的缓冲，可以使用 `10` 或 `10MB`，256 字节可以使用 `256` 或 `256B`。

支持使用以下任意单位：

- `B` 字节；
- `KB` 千字节；
- `MB` 兆，百万字节；
- `GB` 十亿字节；
- `TB` 兆兆字节。

默认单位为字节，可以使用 `@DataSizeUnit` 自定义，如上所示。

如果更喜欢构造器绑定，可以使用下面方式暴露相同的属性：

```
@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    // fields...

    public MyProperties(@DataSizeUnit(DataUnit.MEGABYTES) @DefaultValue("2MB") DataSize bufferSize,
            @DefaultValue("512B") DataSize sizeThreshold) {
        this.bufferSize = bufferSize;
        this.sizeThreshold = sizeThreshold;
    }

    // getters...

}
```

> 如果要更新 `Long` 属性，注意使用 `@DataSizeUnit` 重新定义单位。这么做能提供透明的更新方式，同时支持更丰富的格式。

#### @ConfigurationProperties Validation

无论何时 `@ConfigurationProperties` 类标注 `@Validated`，Spring Boot 都会尝试对它进行校验。你可以直接在类中使用 JSR-303 `javax.validation` 的注解（要确保类路径存在兼容的 JSR-303 实现）。如下所示：

```
@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    // getters/setters...

}
```

> 还可以使用 `@Validated` 标注创建 configuration properties 的 `@Bean` 方法来触发校验。

为确保即使找不到任何属性也会触发级联属性的校验，必须使用 `@Valid` 标注级联字段。如下所示：

```
@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // getters/setters...

    public static class Security {

        @NotEmpty
        private String username;

        // getters/setters...

    }

}
```

你可以添加自定义的 `Validator`，它应该命名为 `configurationPropertiesValidator` 。注册它的 `@Bean` 方法需要声明为静态，因为配置属性的校验发生在 Spring Boot 生命周期的早期，这么做可使校验器的实例化与配置类的生命周期分割而不受其限制，避免校验器迟加载问题。

> `spring-boot-actuator` 模块包含一个端点，它能暴露所有 `@ConfigurationProperties` bean。使用浏览器访问 `/actuator/configprops` 或打开等效的 JMX 端点，可以查看详细信息。

#### @ConfigurationProperties vs. @Value

`@Value` 是核心容器的功能，它不能提供与 type-safe configuration properties 相同的功能。下表总结 `@Value` 和 `@ConfigurationProperties` 各自支持的功能：

| Feature           | `@ConfigurationProperties` | `@Value` |
| :---------------- | :------------------------- | :------- |
| Relaxed binding   | Yes                        | Limited  |
| Meta-data support | Yes                        | No       |
| `SpEL` evaluation | No                         | Yes      |

> 如果你实在想使用 `@Value`，建议引用规范的属性名（kebab 格式）。这将允许 Spring Boot 使用与宽松绑定 `@ConfigurationProperties` 相同的逻辑。比如，`@Value("{demo.item-price}")` 将从 `application.properties` 文件中获取 `demo.item-price` 和 `demo.itemPrice`，或者从系统环境中获取 `DEMO_ITEMPRICE`。但你如果使用 `@Value("{demo.itemPrice}")` 注入属性，将不会考虑 `demo.item-price` 和 `DEMO_ITEMPRICE`。

如果你为自己的组件定义一组配置属性，建议将它们组合到标有 `@ConfigurationProperties` POJO 中。这将为你提供结构化的，类型安全的对象，可以将它直接注入到其它 bean。

在解析这些文件并填充环境时，不会处理来自应用属性文件的 `spEL` 表达式，但可以在 `@Value` 中编写 `spEL`。如果应用属性文件中的属性值为 `spEL` 表达式，在通过 `@Value` 使用它时会进行计算。

## Profiles

Spring Profiles 提供一种方法，用于分隔程序的配置，并使特定的部分仅在环境满足特定条件时才生效。任何 `@Configuration`，`@ConfigurationProperties` 或 `@Component` 都能标注 `@Profile`，以限制它们是否加载。如下所示：

```
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

> 如果 `@ConfigurationProperties` bean 是通过 `@EnableConfigurationProperties` 而非自动扫描注册，则只能在对应的 `@EnableConfigurationProperties` `@Configuration` 类上声明 `@Profile`。反之，如果使用扫描注册 `@ConfigurationProperties` bean，`@Profile` 就可以直接声明在 `@ConfigurationProperties` 类上。

你可以使用配置属性 `spring.profiles.active` 指定激活的 profiles，可以选择前面介绍的任意属性设置方式。比如，`application.properties`，如下所示：

```
spring.profiles.active=dev,hsqldb
```

或者使用命令行参数，`--spring.profiles.active=dev,hsqldb`。

如果没有显式激活任何 profile，则启用默认 profile，它的名字是 `default`。这个名字可以使用配置属性 `spring.profiles.default` 对其进行修改：

```
spring.profiles.default=none
```

### Adding Active Profiles

`spring.profiles.active` 属性遵守与其它配置属性相同的排序规则：最高的 `PropertySource` 获胜。这表示能在 `application.properties` 激活默认的 profiles，然后使用命令行参数进行覆盖。

有时，需要添加 profiles 而非替换。`SpringApplication` 入口点提供 `setAdditionalProfiles()` 方法，用于设置额外的 profiles，这种方式激活的 profiles 优先于 `spring.profiles.active` 激活的 profiles。在给定 profile 被激活时，下节讨论的 profiles 分组也可用于设置额外的 profiles。

### Profile Groups

有时，你自己定义的 profiles 可能太过细粒度和繁多，使用起来比较麻烦。

比如，现有 `proddb` 和 `prodmq` profiles，分别开启 database 和 messaging 功能。为将它们统一管理，Spring Boot 允许定义 profiles 分组。

下面定义 `production` profile 分组，它包含 `proddb` 和 `prodmq` profiles：

```
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

现在可以使用 `spring.profiles.active=production` 直接启用 `proddb` 和 `prodmq`。

### Programmatically Setting Profiles

可以以编程的方式设置激活的 profiles，只需在 `SpringApplication.run()` 之前调用 `SpringApplication.setAdditionalProfiles(…)`。还可以使用 Spring 的 `ConfigurableEnvironment` 接口激活 profiles。

### Profile-specific Configuration Files

特定的 profile 配置文件，包括 `application-{profile}.properties|yaml` 和 `@ConfigurationProperties` 引用的文件，都会被考虑为文件并加载。具体查看前面的 Profile Specific Files 章节。

## JSON

Spring Boot 提供 3 个 JSON 库的内置集成：

- Jackson
- Gson
- JSON-B

Jackson 是首选和默认库。

### Jackson

Auto-configuration for Jackson is provided and Jackson is part of `spring-boot-starter-json`. When Jackson is on the classpath an `ObjectMapper` bean is automatically configured. Several configuration properties are provided for customizing the configuration of the `ObjectMapper`.

### Gson

Auto-configuration for Gson is provided. When Gson is on the classpath a `Gson` bean is automatically configured. Several `spring.gson.*` configuration properties are provided for customizing the configuration. To take more control, one or more `GsonBuilderCustomizer` beans can be used.

### JSON-B

Auto-configuration for JSON-B is provided. When the JSON-B API and an implementation are on the classpath a `Jsonb` bean will be automatically configured. The preferred JSON-B implementation is Apache Johnzon for which dependency management is provided.

## Testing

Spring Boot 提供许多实用程序和注解用于测试你的程序。有两个模块负责提供测试支持：`spring-boot-test` 包含核心项目，`spring-boot-test-autoconfigure` 支持自动配置的测试。

大多数时候使用 `spring-boot-starter-test` 模块，它会导入 Spring Boot 测试模块，还有 JUnit Jupiter，AssertJ，AssertJ，Hamcrest 等有用的库。

> 如果你使用 JUnit 4 测试，可以使用 JUnit 5 的 vintage 引擎运行测试。如要使用 vintage 引擎，需要添加 `junit-vintage-engine` 依赖。如下所示：
>
> ```
> <dependency>
>     <groupId>org.junit.vintage</groupId>
>     <artifactId>junit-vintage-engine</artifactId>
>     <scope>test</scope>
>     <exclusions>
>         <exclusion>
>             <groupId>org.hamcrest</groupId>
>             <artifactId>hamcrest-core</artifactId>
>         </exclusion>
>     </exclusions>
> </dependency>
> ```

## Auto-configuration

auto-configuration 可以绑定在外部的第三方 `Jar` 包内，且它仍然会被 Spring Boot 检测并使用。因此，我们可以自由开发自己的自动配置。

auto-configuration 可以与 `starter` 相关联，通常，此时 `starter` 还会包含自动配置需要的其它库。

### Understanding Auto-configured Beans

auto-configuration 实际是标准的 `@Configuration` 类，它使用各种的 `@Conditional` 注解限制自己在哪种应用环境中生效。最常使用的条件注解是 `@ConditionalOnClass` 和 `@ConditionalOnMissingBean`。

### Locating Auto-configuration Candidates

Spring Boot 会检测类路径所有 `Jar` 包的 `META-INF/spring.factories` 文件。该文件应该在 `EnableAutoConfiguration` 键下列出自己的自动配置。如下所示：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

> 自动配置只能以这种方式加载。请确保它们被定义在特定的包空间，并且它们不是自动扫描的目标。此外，自动配置自己也不能开启自动扫描来注入外部组件，只能使用特定的 `@Import`。

如果你的配置类需要遵守某种使用顺序，可以使用 `@AutoConfigureAfter` 或 `@AutoConfigureBefore`，这两个注解会强制配置类在另一个类前或后使用。

如果想设置自动配置的使用顺序，但不要求它与特定类的相对顺序，可以使用 `@AutoConfigureOrder`。它拥有与普通的 `@Order` 相同的语义，不过它是专门为自动配置提供顺序。

与标准 `@Configuration` 相同，自动配置的使用顺序只会影响其包含的 bean 的定义顺序。这些 bean 在后续的创建顺序只和依赖关系、`@DependsOn` 相关。

### Condition Annotations

Spring Boot 提供许多 `@Conditional` 条件注解。可以使用它们标注 `@Configuration` 类或单个 `@Bean` 方法。这些注解包括：

- Class Conditions
- Bean Conditions
- Property Conditions
- Resource Conditions
- Web Application Conditions
- SpEL Expression Conditions

#### Class Conditions

`@ConditionalOnClass` 和 `@ConditionalOnMissingClass` 注解使得 `@Configuration` 类只在特定的类存在或不存在时才被包含。因为注解元数据是由 ASM 解析，所以你可以使用注解的 `value` 属性指定特定的类，即使它不在类路径。你也可以使用 `name` 属性指定类名。

这种机制不适用于返回类型为条件目标的 `@Bean` 方法。因为在方法的 condition 应用前，JVM 会加载类并处理方法的引用，如果类不存在，这些引用会失败。

为解决这种问题，可以使用单独的 `@Configuration` 类隔离条件。如下所示：

```
@Configuration(proxyBeanMethods = false)
// Some conditions ...
public class MyAutoConfiguration {

    // Auto-configured beans ...

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(SomeService.class)
    public static class SomeServiceConfiguration {
		
		//内部配置类限定：只有SomeService.class存在时才会处理此@Bean方法
        @Bean
        @ConditionalOnMissingBean
        public SomeService someService() {
            return new SomeService();
        }

    }

}
```

> 你如果使用 `@ConditionalOnClass` 或 `@ConditionalOnMissingClass` 作为元注解定义自己的组合注解，必须使用 `name` 属性，因为这种情况不会处理类的引用。

#### Bean Conditions

`@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 注解使得 bean 只有在特定的 bean 存在或不存在时才会被包含。可以使用注解的 `value` 属性指定 bean 的类型或者使用 `name` 属性指定 bean 的 name。还可以使用 `search` 属性限制搜索 bean 时考虑的 `ApplicationContext` 的层次结构。

当标注在 `@Bean` 方法时，目标类型默认为方法返回类型。如下所示：

```
@Configuration(proxyBeanMethods = false)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public SomeService someService() {
        return new SomeService();
    }

}
```

上例，当 `ApplicationContext` 不包含 `SomeService` 类型的 bean 时，会调用该方法。

> 你需要非常注意 bean definition 的添加顺序。因为 bean condition 是根据目前处理的内容来计算的。所以，建议只在 auto-configuration 中使用 `@ConditionalOnBean` 和 `@ConditionalOnMissingBean`，因为它们保证在所有用户自定义的 bean definition 添加才加载。

> `@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 不会阻止 `@Configuration` 的创建。在类级别标注和在每个 `@Bean` 方法标注的区别是，如果条件不匹配，前者会阻止将 `@Configuration` 注册为 bean。

> 声明 `@Bean` 方法时，应该提供尽可能多的返回类型信息。比如，返回实例实现某接口，`@Bean` 方法应该返回真实类型而非接口。在使用 bean condition 时，这很重要，因为它的计算只能根据方法签名的类型信息。

#### Other Conditions

参看文档，或查阅其它资料。

### Testing Auto-configuration

auto-configuration 可能会被多种因素影响：用户配置（`@Bean` definition 和自定义 `Environment`），条件注解（某库存在或不存在）等。具体而言，每个测试都应该创建一个定义完好的 `ApplicationContext`，这个上下文表示所有自定义的组合。`ApplicationContextRunner` 提供实现这个目标的最好方式。

`ApplicationContextRunner` 通常定义为测试类的一个字段，它将所有基础的、通用的配置整合。下面示例保证 `MyServiceAutoConfiguration` 始终有效：

```
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
        .withConfiguration(AutoConfigurations.of(MyServiceAutoConfiguration.class));
```

> 如果需要定义多个 auto-configurations，不需要对它们的声明进行排序，因为程序运行时它们的调用顺序完全相同。

每个测试都能使用 runner 表示特定的用例。比如，下例调用用户配置 `UserConfiguration` 并检查 auto-configuration 是否正确回退。`run` 能提供一个可与 `AssertJ` 一起使用的回调上下文：

```
@Test
void defaultServiceBacksOff() {
    this.contextRunner.withUserConfiguration(UserConfiguration.class).run((context) -> {
        assertThat(context).hasSingleBean(MyService.class);
        assertThat(context).getBean("myCustomService").isSameAs(context.getBean(MyService.class));
    });
}

@Configuration(proxyBeanMethods = false)
static class UserConfiguration {

    @Bean
    MyService myCustomService() {
        return new MyService("mine");
    }

}
```

还可以轻松地自定义 `Environment`，如下所示：

```
@Test
void serviceNameCanBeConfigured() {
    this.contextRunner.withPropertyValues("user.name=test123").run((context) -> {
        assertThat(context).hasSingleBean(MyService.class);
        assertThat(context.getBean(MyService.class).getName()).isEqualTo("test123");
    });
}
```

runner 还能用于展示 `ConditionEvaluationReport`，这个报告能在 `INFO` 或 `DEBUG` 级别打印。下例示范如何使用 `ConditionEvaluationReportLoggingListener` 在 auto-configuration 测试中打印报告：

```
class MyConditionEvaluationReportingTests {

    @Test
    void autoConfigTest() {
        new ApplicationContextRunner()
            .withInitializer(new ConditionEvaluationReportLoggingListener(LogLevel.INFO))
            .run((context) -> {
                    // Test something...
            });
    }

}
```

#### Simulating a Web Context

如果你想在 servlet 或 reactive web 上下文中测试 auto-configuration，请使用 `WebApplicationContextRunner` 或者 `ReactiveWebApplicationContextRunner`。

#### Overriding the Classpath

还可以测试当类路径不存在特定类或包，运行时会发生什么。Spring Boot 载有一个 `FilteredClassLoader`，runner 可以轻松地使用它。下例断言 `MyService` 不存在时， auto-configuration 会被准确禁用：

```
@Test
void serviceIsIgnoredIfLibraryIsNotPresent() {
    this.contextRunner.withClassLoader(new FilteredClassLoader(MyService.class))
            .run((context) -> assertThat(context).doesNotHaveBean("myService"));
}
```

# The “Spring Web MVC Framework”

> **Web**
>
> Spring Boot 非常适合 web 程序的开发。因为我们可以使用内嵌的 Tomcat，Jetty，Undertow 或 Netty 创建独立的 HTTP 服务器，而不需要将程序再部署到运行的服务器。这样，就能像运行普通 Java 程序般运行 web 程序。大多数 web 程序会使用 `spring-boot-starter-web` 模块来启动和运行。对于 reactive  web 程序，可以使用模块 `spring-boot-starter-webflux`。
>
> ***
>
> **Servlet Web Applications**
>
> 如果想构建 servlet web 程序，可以使用 Spring Boot 对 Spring MVC 或 Jersey 的 auto-configuration。

Spring Web MVC framework（Spring MVC）是功能丰富的 “模型 视图 控制器” web 框架。Spring MVC 使得用户可以创建特殊的 bean `@Controller` 或 `@RestController` 来处理进入的 HTTP 请求。controller 中的方法使用 `@RequestMapping`  映射 HTTP。

下例使用典型的 `@RestController`，它会返回 JSON 格式响应 ：

```
@RestController 
@RequestMapping("/users")
public class MyRestController {

    private final UserRepository userRepository;

    private final CustomerRepository customerRepository;

    public MyRestController(UserRepository userRepository, CustomerRepository customerRepository) {
        this.userRepository = userRepository;
        this.customerRepository = customerRepository;
    }

    @GetMapping("/{user}")
    public User getUser(@PathVariable Long userId) {
        return this.userRepository.findById(userId).get();
    }

    @GetMapping("/{user}/customers")
    public List<Customer> getUserCustomers(@PathVariable Long userId) {
        return this.userRepository.findById(userId).map(this.customerRepository::findByUser).get();
    }

    @DeleteMapping("/{user}")
    public void deleteUser(@PathVariable Long userId) {
        this.userRepository.deleteById(userId);
    }

}
```

## Spring MVC Auto-configuration

Spring Boot 为 Spring MVC 提供的 auto-configuration 适用于大部分的程序。这个 auto-configuration 在 Spring 默认设置的基础上添加以下功能：

- 包含 `ContentNegotiatingViewResolver` 和 `BeanNameViewResolver` beans；
- 支持提供静态资源，包括对 WebJar 的支持；
- 自动注册 `Converter`，`GenericConverter` 和 `Formatter` beans；
- 支持 `HttpMessageConverters`；
- 自动注册 `MessageCodesResolver`；
- 支持静态首页 `index.html`；
- 自动使用 `ConfigurableWebBindingInitializer` beans。

如果你想保留 auto-configuration 的设置，又想添加自己的 MVC 设置（interceptors，formatters 等），你可以添加自己的继承 `WebMvcConfigurer` 的 `@Configuration` 类，但不需要标注 `@EnableWebMvc`。

如果你想提供自定义的 `RequestMappingHandlerMapping`，`RequestMappingHandlerAdapter` 或者 `ExceptionHandlerExceptionResolver`，又想保留 auto-configuration 的设置，你可以声明一个 `WebMvcRegistrations` 类型的 bean，使用它提供这些实例。

如果你想完全自己设置 Spring MVC，可以添加自己的标有 `@EnableWebMvc` 的 `@Configuration`，或者添加自己的标有 `@Configuration` 元注解的 `DelegatingWebMvcConfiguration`，详细信息在 `@EnableWebMvc` 文档。

> Spring MVC 使用与 Spring Boot 转换 config data 属性时不同的 `ConversionService`。所以，Spring MVC 不支持 `Period`，`Duration`，`DataSize` 类型转换，`@DurationUnit` 和 `@DataSizeUnit` 注解会被忽略。
>
> 如果你想自定义 Spring MVC 使用的 `ConversionService`，你可以提供一个 `WebMvcConfigurer` bean，它含有方法 `addFormatters`，可使用它注册任何转换器。或者委托 `ApplicationConversionService` 的静态方法。

## HttpMessageConverters

Spring MVC 使用 `HttpMessageConverter` 接口转换 HTTP 请求和响应。默认设置十分合理，可以开箱即用。比如，对象可以自动转换为 JSON（Jackson）或者 XML（Jackson XML 拓展，或者 JAXB）。默认情况，字符串会被编码为 `UTF-8`。

如果你想添加自定义的转换器，可以使用 Spring Boot 的 `HttpMessageConverters` 类，如下所示：

```
@Configuration(proxyBeanMethods = false)
public class MyHttpMessageConvertersConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = new AdditionalHttpMessageConverter();
        HttpMessageConverter<?> another = new AnotherHttpMessageConverter();
        return new HttpMessageConverters(additional, another);
    }

}
```

context 中的任何 `HttpMessageConverter` bean 都会被添加到转换器列表，你也可以使用相同的方式覆盖默认设置。

## Custom JSON Serializers and Deserializers

如果你使用 Jackson 序列化和反序列化 JSON 数据，你也许会想定义自己的 `JsonSerializer` 和 `JsonDeserializer` 类。自定义序列化器通常是通过模块注册到 Jackson，但 Spring Boot 提供另一种方式，使用 `@JsonComponent` 更方便地直接注册 bean。

你可以直接使用 `@JsonComponent` 标注 `JsonSerializer`，`JsonDeserializer` 或 `KeyDeserializer` 的实现。你也可以标注含有 serializers/deserializers 内部类的类，如下所示：

```
@JsonComponent
public class MyJsonComponent {

    public static class Serializer extends JsonSerializer<MyObject> {

        @Override
        public void serialize(MyObject value, JsonGenerator jgen, SerializerProvider serializers) throws IOException {
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
        }

    }

    public static class Deserializer extends JsonDeserializer<MyObject> {

        @Override
        public MyObject deserialize(JsonParser jsonParser, DeserializationContext ctxt)
                throws IOException, JsonProcessingException {
            ObjectCodec codec = jsonParser.getCodec();
            JsonNode tree = codec.readTree(jsonParser);
            String name = tree.get("name").textValue();
            int age = tree.get("age").intValue();
            return new MyObject(name, age);
        }

    }

}
```

`ApplicationContext` 中的所有 `@JsonComponent` bean 都会自动注册到 Jackson。因为 `@JsonComponent` 被 `@Component` 元注解标注，所以它会被自动扫描检测。

Spring Boot 还提供 `JsonObjectSerializer` 和 `JsonObjectDeserializer` 基类，可替代标准的 Jackson 版本来序列化对象。

前面示例可以使用 `JsonObjectSerializer`/`JsonObjectDeserializer` 重写，如下所示：

```
@JsonComponent
public class MyJsonComponent {

    public static class Serializer extends JsonObjectSerializer<MyObject> {

        @Override
        protected void serializeObject(MyObject value, JsonGenerator jgen, SerializerProvider provider)
                throws IOException {
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
        }

    }

    public static class Deserializer extends JsonObjectDeserializer<MyObject> {

        @Override
        protected MyObject deserializeObject(JsonParser jsonParser, DeserializationContext context, ObjectCodec codec,
                JsonNode tree) throws IOException {
            String name = nullSafeValue(tree.get("name"), String.class);
            int age = nullSafeValue(tree.get("age"), Integer.class);
            return new MyObject(name, age);
        }

    }

}
```

## MessageCodesResolver

Spring MVC 具有生成错误代码的策略 `MessageCodesResolver`，它从绑定的错误中呈现错误消息。如果设置 `spring.mvc.message-codes-resolver-format` 属性为 `PREFIX_ERROR_CODE` 或 `POSTFIX_ERROR_CODE`，Spring Boot 会为你创建一个此类型的实例。

## Static Content

默认情况，Spring Boot 从类路径或 `ServletContext` 根下的文件夹 `/static` 或 `/public` 或 `/resources` 或 `/META-INF/resources` 中提供静态资源。这其实是使用 Spring MVC 的 `ResourceHttpRequestHandler` 实现的，所以你可以添加自己的 `WebMvcConfigurer` 并重写 `addResourceHandlers` 方法以自定义此行为。

在独立 web 程序中，启用容器的 default servlet 并将它作为保底。如果 Spring 无法处理当前的请求，它能提供 `ServletContext` 根路径下的内容。当然，绝大部分时候都不会使用它，因为开发者通常会考虑到所有可能接收的请求。

默认情况，资源映射到 `/**`，但你可以使用 `spring.mvc.static-path-pattern` 属性调整它。比如，下例将所有资源重新定位到 `/resources/**`：

```
spring.mvc.static-path-pattern=/resources/**
```

注意，这里是是修改资源的映射位置，而非资源在程序中的物理位置。

你也可以使用 `spring.web.resources.static-locations` 属性自定义静态资源的位置（将默认值替换为目录列表）。servlet 容器的根路径 `"/"` 会自动添加到静态资源位置列表。

除前面提到的标准静态资源位置，Spring Boot 还为 `Webjars`内容提供支持。任何路径为 `/webjars/**` 的资源，如果它们以 `Webjars` 格式打包，将会从 `Jar` 包中提供。

> 如果你的程序打包为 `Jar`，请不要使用 `src/main/webapp` 目录。虽然这个目录是通用标准，但它只在 `war` 包中有效。如果你生成 `Jar` 包，大部分构建工具会默默忽略它。

Spring Boot also supports the advanced resource handling features provided by Spring MVC, allowing use cases such as cache-busting static resources or using version agnostic URLs for Webjars.

To use version agnostic URLs for Webjars, add the `webjars-locator-core` dependency. Then declare your Webjar. Using jQuery as an example, adding `"/webjars/jquery/jquery.min.js"` results in `"/webjars/jquery/x.y.z/jquery.min.js"` where `x.y.z` is the Webjar version.

> If you use JBoss, you need to declare the `webjars-locator-jboss-vfs` dependency instead of the `webjars-locator-core`. Otherwise, all Webjars resolve as a `404`.

To use cache busting, the following configuration configures a cache busting solution for all static resources, effectively adding a content hash, such as `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`, in URLs:

```
spring.web.resources.chain.strategy.content.enabled=true
spring.web.resources.chain.strategy.content.paths=/**
```

> Links to resources are rewritten in templates at runtime, thanks to a `ResourceUrlEncodingFilter` that is auto-configured for Thymeleaf and FreeMarker. You should manually declare this filter when using JSPs. Other template engines are currently not automatically supported but can be with custom template macros/helpers and the use of the `ResourceUrlProvider`.

When loading resources dynamically with, for example, a JavaScript module loader, renaming files is not an option. That is why other strategies are also supported and can be combined. A "fixed" strategy adds a static version string in the URL without changing the file name, as shown in the following example:

```
spring.web.resources.chain.strategy.content.enabled=true
spring.web.resources.chain.strategy.content.paths=/**
spring.web.resources.chain.strategy.fixed.enabled=true
spring.web.resources.chain.strategy.fixed.paths=/js/lib/
spring.web.resources.chain.strategy.fixed.version=v12
```

With this configuration, JavaScript modules located under `"/js/lib/"` use a fixed versioning strategy (`"/v12/js/lib/mymodule.js"`), while other resources still use the content one (`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`).

See `WebProperties.Resources` for more supported options.

> This feature has been thoroughly described in a dedicated blog post and in Spring Framework’s reference documentation.

## Welcome Page

Spring Boot 支持静态和模板技术的欢迎页。它首先会搜索配置的静态内容位置中的 `index.html` 文件。如果没有找到，再搜索 `index` 模板。

## Path Matching and Content Negotiation

Spring MVC 可以通过查看请求路径并将它与定义的映射（`@RequestMapping`）匹配，以此将 HTTP 请求映射到处理器。

Spring Boot 默认禁用后缀匹配模式，这意味像 `"GET /projects/spring-boot.json"` 的请求将无法匹配到 `@GetMapping("/projects/spring-boot")` 映射。这被认为是当前 Spring MVC 程序的最佳实践。后缀匹配在过去是 HTTP 客户端用于向程序补充接受类型的方式，因为 "Accept" 请求头可能不正确。但现在内容协商更加可靠。

还有其它的方式来处理无法总是发送正确 "Accept" 请求头的 HTTP 客户端。除后缀模式，我们可以使用请求查询参数以保证像 `"/projects/spring-boot?format=json"` 的请求被映射到 `@GetMapping("/projects/spring-boot")`：

```
spring.mvc.contentnegotiation.favor-parameter=true
```

或者你更喜欢其它的参数名：

```
spring.mvc.contentnegotiation.favor-parameter=true
spring.mvc.contentnegotiation.parameter-name=myparam
```

大部分媒体类型都是支持开箱即用的，但你也可以定义新的媒体类型：

```
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

后缀匹配模式已被弃用，并在将来的版本中删除。如果你知道这些，但仍然想在程序中使用后缀匹配模式，则需要进行以下配置：

```
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

或者，与其开启所有后缀模式，不如只支持已经注册的后缀模式：

```
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true
```

从 Spring Framework 5.3 开始，Spring MVC 支持多种策略的实现，用于匹配请求路径和 controller 处理器。以前只支持 `AntPathMatcher` 策略，现在增加对 `PathPatternParser` 的支持。Spring Boot 提供一个配置属性，用于选择新的策略：

```
spring.mvc.pathmatch.matching-strategy=path-pattern-parser
```

> `PathPatternParser` 是优化的实现，但它限制了一些路径变体的使用，并且不兼容后缀匹配模式（`spring.mvc.pathmatch.use-suffix-pattern`，`spring.mvc.pathmatch.use-registered-suffix-pattern`）和映射带有 servlet 前缀的 `DispatcherServlet`（`spring.mvc.servlet.path`）。

## ConfigurableWebBindingInitializer

Spring MVC 使用一个 `WebBindingInitializer` 为特定的请求初始化 `WebDataBinder`。如果你创建自己的 `ConfigurableWebBindingInitializer` `@Bean`，Spring Boot 会自动将它配置到 Spring MVC 使用。

## Template Engines

除 REST web 服务，还可以使用 Spring MVC 提供动态的 HTML 内容。Spring MVC 支持多种模板技术，包括 Thymeleaf，FreeMarker 和 JSP 等。另外，其它的许多模板引擎都含有自己的 Spring MVC 集成。

Spring Boot 包含支持以下模板引擎的 auto-configuration：

- FreeMarker
- Groovy
- Thymeleaf
- Mustache

> 应该尽可能避免使用 JSP。已知当它与内嵌的 servlet 容器一起使用时，具有许多限制。

当你使用以上任意的模板技术和默认设置时，将自动从 `src/main/resources/templates` 获取模板。

>  根据程序的运行方式，你的 IDE 可能会对类路径进行不同的排序。在 IDE 中从主方法运行程序可能会导致与使用 Maven 或 Gradle 或 `Jar` 包不同的运行顺序，这可能导致 Spring Boot 无法找到预期的模板。如果你遇到此问题，可以在 IDE 中对类路径进行重新排序，将模块的类和资源放在前面。

## Error Handling

默认情况，Spring Boot 提供 `/error` 映射以合理的方式处理所有错误，并在 servlet 容器中将它注册为全局错误页面。对于机器客户端，它返回 JSON 响应，其中包含错误详情，HTTP 状态码和异常信息。对于浏览器客户端，它返回 “白页” 错误视图，包含相同的数据，但会渲染为 HTML 格式。如果想自定义它，请添加解析为 `error` 的 `View`。

如果想自定义默认的错误处理行为，Spring Boot 提供许多 `server.error` 属性可以设置，具体参看 [“Server Properties”](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.server)。

如果想完全控制错误处理行为，可以实现 `ErrorController`，并注册它为此类型的 bean。或者添加 `ErrorAttributes` 类型的 bean，使用现有的机制但替换内容。

> `BasicErrorController` 可以作为自定义 `ErrorController` 类的基类。当你想为新的媒体类型添加处理器时，这会非常有用，默认只处理 `text/html` 并为所有内容提供返回。为此，拓展 `BasicErrorController`，添加标有 `@RequestMapping` 的公共方法，这个注解包含属性 `produces`，然后创建你的新类型的 bean。

你还可以创建 `@ControllerAdvice` 类，自定义为特定的控制器和异常类型返回的 JSON 文档。如下所示：

```
@ControllerAdvice(basePackageClasses = SomeController.class)
public class MyControllerAdvice extends ResponseEntityExceptionHandler {

    @ResponseBody
    @ExceptionHandler(MyException.class)
    public ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new MyErrorBody(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer code = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        HttpStatus status = HttpStatus.resolve(code);
        return (status != null) ? status : HttpStatus.INTERNAL_SERVER_ERROR;
    }

}
```

上面示例，如果相同包的控制器 `SomeController` 抛出异常 `MyException`，则会使用自定义错误类型的 JSON 表示而非 `ErrorAttributes` 表示。

某些情况，控制器级别处理的错误不会被指标基础结构记录。程序可以将这些处理的异常设置为请求属性，以确保它们被请求指标记录。如下所示：

```
@Controller
public class MyController {

    @ExceptionHandler(CustomException.class)
    String handleCustomException(HttpServletRequest request, CustomException ex) {
        request.setAttribute(ErrorAttributes.ERROR_ATTRIBUTE, ex);
        return "errorView";
    }

}
```

### Custom Error Pages

如果你想为给定的 HTTP 状态码返回自定义 HTML 错误页面，你可以在 `/error` 文件夹添加文件。错误页面可以是静态的 HTML（添加到任意的静态资源文件夹），也可以由模板技术生成。文件的名字应该是确切的状态码或序列掩码。

比如，要映射 `404` 到一个静态的 HTML 文件，你的文件夹结构可能如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要使用 FreeMarker 模板映射 `5xx` 的状态码，你的文件夹结构可能如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftlh
             +- <other templates>
```

对于更加复杂的映射，可以添加实现 `ErrorViewResolver` 接口的 bean，如下所示：

```
public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        if (status == HttpStatus.INSUFFICIENT_STORAGE) {
            // We could add custom model values here
            new ModelAndView("myview");
        }
        return null;
    }

}
```

你也可以使用常规的 Spring MVC 功能，比如 `@ExceptionHandler` 方法和 `@ControllerAdvice`。`ErrorController` 最后会捕获所有未被处理的异常。

### Mapping Error Pages outside of Spring MVC

对于没有使用 Spring MVC 的程序，你可以使用 `ErrorPageRegistrar` 接口直接注册 `ErrorPages`。这个抽象直接在顶级内嵌的 servlet 容器工作，即使你没有 Spring MVC 的 `DispatcherServlet`。

```
@Configuration(proxyBeanMethods = false)
public class MyErrorPagesConfiguration {

    @Bean
    public ErrorPageRegistrar errorPageRegistrar() {
        return this::registerErrorPages;
    }

    private void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }

}
```

如果你注册的 `ErrorPage` 的路径最终会被 `Filter` 处理（这很常见，如 Jersey 和 Wicket），则必须将此 `Filter` 显式注册为 `ERROR` 调度器。如下所示：

```
@Configuration(proxyBeanMethods = false)
public class MyFilterConfiguration {

    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> registration = new FilterRegistrationBean<>(new MyFilter());
        // ...
        registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
        return registration;
    }

}
```

注意，默认的 `FilterRegistrationBean` 并不包含 `ERROR` 调度器类型。

### Error handling in a war deployment

当部署到 servlet 容器，Spring Boot 会使用它的 error page filter 将带有错误状态码的请求转发到合适的错误页面。这种过滤器是必要的，因为 servlet 规范没有提供 API 注册错误页面。根据你的 war 文件部署的容器和你的程序使用的技术，可能需要一些额外的配置。

只有当响应未被提交时，error page filter 才能把请求转发到正确的错误页。默认情况，WebSphere Application Server 8.0 以及更高版本会在完成 servlet 的 service 方法后立刻提交响应，你应该通过设置属性 `com.ibm.ws.webcontainer.invokeFlushAfterService` 为 `false` 禁止此行为。

如果你正在使用 Spring Security，且想要访问错误页面的主体，你必须设置 Spring Security 的过滤器在错误调度时使用。为此，设置 `spring.security.filter.dispatcher-types` 属性为 `async, error, forward, request`。

## CORS Support

Cross-origin resource sharing（CORS） 是由大部分浏览器实现的 W3C 规范。它使你能以灵活的方式授权跨域请求，而不是使用一些安全性低，功能弱的方法（如 IFRAME 和 JSONP）。

Spring MVC 从 4.2 开始支持 CORS。在 Spring Boot 中使用 `@CrossOrigin` 标注控制器（类或方法）不需要任何特殊的设置，开箱即用。还能注册 `WebMvcConfigurer` bean，并重写 `addCorsMappings(CorsRegistry)` 方法来定义全局的 CORS 设置。如下所示：

```
@Configuration(proxyBeanMethods = false)
public class MyCorsConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {

            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }

        };
    }

}
```

