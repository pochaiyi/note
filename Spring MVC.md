# Spring MVC

MVC，Model View Controller，是一种架构模式，拆分程序：模型 + 视图 + 控制器，这样的目的是：把数据和视图分离、控制逻辑和处理逻辑分离、视图选择和视图实现分离，使得程序简洁、易读、可配置。

# 前端控制器 DispatcherServlet

`DispatcherServlet` 是 Servlet MVC 的核心，担任前端控制器的角色，负责接收和包装请求，，然后按步骤把请求交给组件处理，同时根据处理结果动态改变流程。并且，天然支持 IoC 容器，得到 Spring 支持。

由此可见，Servlet MVC 基于 Servlet API，最终会部署在 Servlet 容器。

**Servlet 和 Servlet 容器**

Servlet 是 SUN 公司提出的 Java Web 开发规范，它只定义如何处理请求，没有"解析网络报文"那些重复而又无聊的逻辑，把业务逻辑从底层实现分离，使动态网站的开发更加简单。

凡是符合 Servlet 规范，或者说，实现 Servlet 接口的类，都是 Servlet 实现。

但是，Servlet 没有实现网络传输，那谁负责接收网络数据，封装 Request 对象呢？这些都是服务器的工作，它会接收请求，根据 URL 映射转交给特定的 Servlet 处理，再把返回作为响应发送给客户端。用户只需把 Servlet 的编译结果放在服务器的目录，可能还要修改一些配置，就能正常运行 Servlet 实例。

支持 Serlvet 规范的服务器很多，比如 Tomcat、Jboss、Jetty、WebLogic，它们也称为 Serlvet 容器。

## 请求的处理流程

`DispatcherServlet` 处理请求的逻辑主要放在 `doDispatch()` 方法，以下是大致流程：

* Serlvet 容器最先收到请求，根据映射把请求转给 DispatcherServlet 对象；
* DispatcherServlet 根据 HandlerMapping 映射，获取处理器执行链，包括一个处理器+若干拦截器；
* 把处理器包装为适配器 HanderAdapter，这一步用于支持多种类型的处理器；
* 正序执行所有拦截器的前置方法，失败时跳过后面三步，直接执行拦截器的结束方法；
* 通过适配器调用处理器方法，得到 ModelAndView 对象；
* 倒序执行所有拦截器的后置方法，这里能修改 ModelAndView 对象；
* 视图解析器 ViewResolver 解析逻辑视图名，创建视图对象；
* 视图 view 使用 model 数据进行渲染，得到最终的响应内容；
* 倒序执行前置方法成功执行的拦截器的结束方法；
* DispatcherServlet 返回，Servlet 据此发送响应给客户端。

可见，`DispatcherServlet` 真的只是一个流程管理器，没有具体的业务处理逻辑。它所用到的组件，包括处理器映射器、处理器、适配器、拦截器、视图解析器、异常处理器等，都能替换和配置，这类对象有九种，这里不都具体讨论。这种机制使 Spring MVC 具有极高的拓展性。

## Servlet 容器配置

`DispatcherServlet` 需要放在 Servlet 容器才能运行，通常还需为其编写一些配置，使得 Serlvet 容器能够配置映射规则、过滤器、监视器等。可用 Java 代码或 XML 文件编写配置信息。

### XML 文件配置

IDEA 的 MVC 项目，固定位置 */web/WEB-INF/web.xml*。

```
<!-- Servlet -->
<servlet>
    <servlet-name>spring-mvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value> <!-- MVC容器配置文件位置 -->
    </init-param>
    <load-on-startup>1</load-on-startup> <!-- 启动优先级 -->
</servlet>

<!-- Servlet URL Pattern -->
<servlet-mapping>
    <servlet-name>spring-mvc</servlet-name>
    <url-pattern>/</url-pattern> <!-- 映射路径 -->
</servlet-mapping>

<!-- Filter -->
<filter>
    <filter-name>filter-name</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>

<!-- Filter URL Pattern -->
<filter-mapping>
    <filter-name>filter-name</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Java 代码配置

`WebApplicationInitializer` 是 Spring MVC 提供的接口，表示 Servlet 容器配置。注意，Servlet 容器可能没有这个接口，那就无法识别这个配置类。

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = 
        					new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = 
        					servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

## 层次结构的上下文

Spring MVC，或者说 `DispatcherServlet`，有一个自己的 IoC 容器，处理请求用到的组件，其实都由这个容器提供。虽然 MVC 容器足以完成大部分工作，但是，有时可能还需与其它 IoC 容器组合。如果 MVC 启动时检测到另一个 IoC 容器，将会把它作为 MVC 容器的父容器，这样，MVC 就能使用 Root 容器的对象。

**XML 文件配置**

监视器 `ContextLoaderListener ` 负责检测和注册第三方 IoC 容器。

```
<web-app>
    <listener>
        <listener-class>
        	org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

	<!-- Root Conetext Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

	...
</web-app>
```

**Java 代码配置**

`AbstractAnnotationConfigDispatcherServletInitializer` 是 `WebApplicationInitializer` 的拓展，配置更加方便。

```
public class MyWebAppInitializer extends 
					AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class }; // RootConfig是第三方容器的根配置类
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { AppConfig.class }; // App1Config是MVC容器的根配置类
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app/*" }; // 匹配路径
    }
}
```

# 过滤器 Filter

## 初步了解

过滤器是 Servlet 容器的概念，它和 Serlvet 处于同个层级。请求到达服务器，首先匹配过滤器，过滤器可以修改请求对象，等到所有过滤器都放行，请求才会被 Servlet 处理。

Spring MVC 提供许多过滤器，以下是常见的几个：

* `CorsFilter`，支持 `@CrossOrigin` 等 MVC 的 CORS 设置。
* `FormContentFilter`，接收 PUT、DELETE 等方法提交的表单。
* `CharacterEncodingFilter`，设置请求和响应的编码。

## 接口抽象

Servlet API 的 `javax.servlet.Filter` 接口，表示过滤器，可以实现这个接口，创建自己的过滤器。

```
public interface Filter {

    default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) 
    	throws IOException, ServletException;

    default void destroy() {}
}
```

## 注册方式

最简单的方式，把过滤器注册 Spring Bean，可用 `@Order` 设置过滤器的优先级。但是，这种方式无法设置过滤器的拦截规则，默认 `/*` 会拦截所有请求。

第二种方式，使用 `@WebFilter` 标注过滤器，注解属性 `urlPatterns` 设置拦截规则。但是，这种方式无法设置过滤器的优先级。另外，需要使用 `@ServletComponentScan` 扫描、注册过滤器/监视器。

第三种方式，最常用，使用 `FilterRegistrationBean` 包装过滤器，然后把这个类注册 Spring Bean，可以设置拦截规则和优先级。

注意，这里涉及的类、注解，都由 Spring 提供。

## 实践 CORS

实现 `Filter` 接口，设置 CORS 相关的响应字段，允许浏览器跨域请求资源。

```
public class MyCorsFilter implements Filter {

    @Override
    public void doFilter(
    		ServletRequest servletRequest, 
    		ServletResponse servletResponse, 
    		FilterChain filterChain) 
    				throws IOException, ServletException {
        // 配置CORS规则
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Headers", "*");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setHeader("Access-Control-Allow-Methods", "POST,GET,OPTIONS,DELETE");
        // 固定调用，继续执行下一个过滤器
        filterChain.doFilter(servletRequest, response);
    }
}
```

使用 `FilterRegistrationBean` 注册过滤器。

```
@Bean
FilterRegistrationBean<MyCorsFilter> myFilterFilterRegistrationBean() {
    FilterRegistrationBean<MyCorsFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new MyCorsFilter());
    bean.setUrlPatterns(Arrays.asList("/*")); // 拦截所有请求路径
    bean.setOrder(-1); // 优先级，CORS应该最高
    return bean;
}
```

# 拦截器 Interceptor

拦截器是 Spring MVC 的概览，它有 3 个方法，会在 `doDispatch()` 内被调用：

* `preHandle()`：处理器方法前，返回布尔值，表示是否继续执行后面的拦截器和处理器。
* `postHandle()`：处理器方法后。
* `afterCompletion()`：视图渲染后，只有 `preHandle()` 返回 True 的拦截器会执行。

# 控制器 Controller

控制器，其实就是前面 `doDispatch()` 使用 HandlerMapping 得出的处理器。Spring MVC 3.0 开始，支持基于注解的开发方式，需要 `RequestMappingHandlerMapping` 和 `RequestMappingHandlerAdapter` 支持。

`@Controller` 标注类，它有 `@Component` 元注解，MVC 认为 `@Controller` 对象是控制器。

## 请求映射

`@RequestMapping` 标注处理器的类和方法，处理器方法的 `@RequestMapping` 继承类的 `@RequestMapping` 属性设置，除了 consumes 和 produces 属性。

| 注解属性 | 说明，这些属性用于筛选请求。                                 |
| -------- | ------------------------------------------------------------ |
| value    | 请求路径，支持 Ant 风格和正则表达式，可设多个。              |
| method   | 请求方法，值为 `RequestMethod` 静态常量，可设多个。          |
| params   | 请求参数，可用 `=` 限制参数值，支持 `!` 运算符，可设多个，值与值的关系是且。 |
| headers  | 请求头字段，可用 `=` 限制参数值，支持 `!` 运算符，可设多个，值与值的关系是且。 |
| consumes | 请求的媒体类型，可设多个。                                   |
| produces | 响应的媒体类型，可设多个。                                   |

`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping` 是 `@RequestMapping` HTTP 方法变体。

## 数据绑定

处理器方法支持的参数类型，这些参数都由 Spring MVC 提供，可以直接声明使用：

* `ServletRequest`/`HttpServletRequest` 和 `ServletResponse`/`HttpServletResponse`
* `InputStream`/`OutputStream` 和 `Reader`/`Writer`
* `WebRequest`/`NativeWebRequest`
* `HttpSession`
* `Model`、`Map`、`ModelMap`：模型数据，用于渲染视图，它们其实是同一个对象。
* `Errors`/`BindingResult`：错误对象。
* Command Object：普通类型，Spring MVC 创建实例，并把请求参数绑定到属性。

### 请求参数

`@RequestParam` 标注方法参数，用于绑定请求参数，可以进行简单的类型转换。如果参数是简单类型，并且没有被其它参数解析器处理，默认这个参数被 `@RequestParam` 标注，它会尝试绑定同名的请求参数。

| 注解属性     | 说明                  |
| ------------ | --------------------- |
| value        | 参数名                |
| required     | 是否必须，默认 `true` |
| defaultValue | 默认值                |

如果方法参数是数组或列表，它会接收所有同名的请求参数；如果方法参数是 `Map<String,String>`，并且没有指定参数名，它会接收所有请求参数。

### 路径片段

`@PathVariable` 标注方法参数，用于绑定 URL 片段，它的 vlaue 属性，指定路径片段的名字，如果参数名和标记名相同，可以省略 value 属性。

```
@GetMapping("/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
	// ...
}
```

### Header 字段

`@RequestHeader` 标注方法参数，用于绑定请求头字段，它有 value、required、defaultValue 3 个属性，可以进行简单的类型转换。能把以 `,` 分隔的多个字符串绑定到数组、列表。

如果 `@RequestHeader` 参数是 `Map<String, String>` 或 `HttpHeaders`，它会接收所有的请求头字段。

```
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

### Cookie 字段

`@CookieValue` 标注方法参数，用于绑定 Cookie 字段，它有 value、required、defaultValue 3 个属性。

```
@GetMapping("/demo")
public void handle(@CookieValue("SESSIONID") String cookie) { 
    //...
}
```

### Model 属性

`@ModelAttribute` 标注方法、方法参数、方法返回类型，正如其名，它和 Model 属性有关。如果参数不是简单类型，并且没有被其它参数解析器处理，默认这个参数被 `@ModelAttribute` 标注。

标注处理器的普通方法，往后调用处理器方法，都会先执行同类的 `@ModelAttribute` 方法，把它们的返回值设为 Model 属性，属性名是 value 值（如果有），或者类型简单名。

标注处理器方法的参数，往后执行处理器方法，都先尝试把同名 Model 属性绑定到方法参数，如果没有这个属性则使用默认构造器创建对象，同时绑定请求参数到对象属性，最后还把对象设为 Model 属性。

标注处理器方法的返回类型，处理器方法执行完后会把返回值设为 Model 属性。

### Session 属性

`@SessionAttributes` 标注类，用于在多次请求间共享某些 Model 属性，属性 value 指定哪些名字的 Model 属性会被存入会话，属性 type 指定哪些类型的 Model 属性会被存入会话，如果 value 和 type 同时设置，则只有名字和类型都满足的 Model 属性才会被存入会话。

处理器方法的执行过程：

* 首先，检查 `@SessionAttributes`，把会话的所有属性加入当前 Model；
* 然后，执行 `@ModelAttribute` 方法，如果 Model 已有同名属性，略过继续使用会话数据；
* 遇到 `@ModelAttribute` 方法参数，尝试绑定 Model 属性，没有则用反射创建对象。

调用 `SessionStatus::setComplete()` 清空会话数据，至于 `SessionStatus` 对象，它是处理器方法的可用参数类型，能够直接声明使用。

### Body 绑定报文

`@RequestBody` 标注参数，根据 `Content-Type` 选择 `HttpMessageConverter ` 转换请求报文为参数对象。

```
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
	// ...
}
```

`@ResponseBody` 标注方法，根据 `Accept`  选择 `HttpMessageConverter ` 转换返回对象为响应报文。

```
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

### Entity 读写实体

Entity 包括 Body 和 Headers，可以读取请求头字段，或者设置响应头字段和响应码

```
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    Account account = entity.getBody();
    HttpHeaders headers = entity.getHeaders();
    // ...
}
```

```
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

## 数据处理

### 数据校验

处理器方法绑定的数据，可能需要满足某些要求才能使用。Spring MVC 内置 Validation 校验框架，但是使用起来非常不便。Spring 3 开始，支持 JSR-303 校验框架，提供完美的集成，居然远比内置框架好用。

Spring Boot 使用 JSR-303 校验框架，首先引入依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

按需使用 JSR-303 注解标注对象属性，比如 `@NotNull`、`@Max`、`@Size`、`@Pattern`。

```
public class User {

    @NotNull
    @Min(1)
    private Long id;

    @Size(min = 2)
    private String name;

    @Min(1)
    private Integer age;

	...
}
```

使用 `@Valid` 标注处理器方法参数，表示这个参数绑定的数据需要进行校验。

```
@PostMapping("/user")
public void postUser(@Valid @RequestBody User user) {
    System.out.println(user);
}
```

另外，还能自定义校验注解，参考原生注解的实现即可。

### 类型转换

**初步了解**

`Converter` 接口是最简单的转换器，用于一对一类型转换，它有许多强大的拓展，暂不学习。

```
public interface Converter<S, T> {
    @Nullable
    T convert(S source);
}
```

`ConversionService` 表示类型转换系统的入口，Spring 通过它使用各种转换器。`ConverterRegistry` 自然表示转换器的配置，声明注册、删除转换器的方法。

`FormattingConversionService` 实现 `GenericConversionService` 和 `FormatterRetistry`，可见它是转换器和格式化器的集合，能把 `Printer` 和 `Parser` 转换为 `GenericConverter`。

Spring Boot 默认注册所有 `Converter`、`Printer`、`Parser` 到 `FormatterRegistry `。

**简单实践**

创建 `Card` 类型的转换器。

```
public class CardConverter implements Converter<String, Card> {

    @Override
    public Card convert(String source) {
        if (StringUtils.hasLength(source)) {
            String[] seg = source.split(",");
            if (seg.length == 2) {
                Card card = new Card();
                card.setId(Long.parseLong(seg[0]));
                card.setName(seg[1]);
                return card;
            }
        }
        return null;
    }
}
```

重写 `WebMvcConfigurer#addFormatters()` 添加转换器，或者把它注册 Spring 组件。

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new CardConverter());
    }
}
```

现在，处理器方法如要绑定 `User` 对象，将会尝试使用 `CardConverter` 进行类型转换。

```
@PostMapping("/card")
@ResponseBody
public Card postUser(Card card) {
    System.out.println(card);
    return card;
}
```

### 格式化器

**初步了解**

`Formatter` 接口用于把特定格式的字符串转换为对象，或者按特定格式输出对象。

```
public interface Formatter<T> extends Printer<T>, Parser<T> {
	
	T parse(String text, Locale locale) throws ParseException;

	String print(T object, Locale locale);
}
```

**简单实践**

创建 `Date` 类型的格式化器。

```
public class DateFormatter implements Formatter<Date> {

    private final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy#MM#dd");

    @Override
    public Date parse(String src, Locale locale) throws ParseException {
        return dateFormat.parse(src);
    }

    @Override
    public String print(Date date, Locale locale) {
        return dateFormat.format(date);
    }
}
```

重写 `WebMvcConfigurer#addFormatters()` 添加格式化器，或者把它注册 Spring 组件。

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new DateFormatter());
    }
}
```

处理器方法，`Good` 有一个 `Date` 属性，可以接收 `yyyy#MM#dd` 格式的字符串。

```
@PostMapping("/good")
public void postGood(Good good, Writer writer) throws IOException {
    System.out.println(good.toString());
    writer.write(good.toString());
}
```

### 消息转换

**初步了解**

`HttpMessageConverter` 表示消息转换器，用于转换请求或响应的报文，前面两节转换的是请求参数。

```
public interface HttpMessageConverter<T> {

	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	List<MediaType> getSupportedMediaTypes();

	T read(Class<? extends T> clazz, HttpInputMessage inputMessage);

	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage);
}
```

**原生支持**

Spring MVC 内置许多消息转换器，支持 JSON、XML、字符串等类型的报文。

* `ByteArrayHttpMessageConverter`
* `StringHttpMessageConverter`
* `ResourceHttpMessageConverter`
* `SourceHttpMessageConverter`
* `FormHttpMessageConverter`
* `Jaxb2RootElementHttpMessageConverter`
* `MappingJacksonHttpMessageConverter`
* ...

**简单实践**

创建 `User` 类型的消息转换器。

```
@Component
public class UserHttpBodyConverter implements HttpMessageConverter<User> {

    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        if (mediaType != null) {
            return clazz == User.class && mediaType.includes(MediaType.TEXT_PLAIN);
        }
        return true;
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        if (mediaType != null) {
            return clazz == User.class && mediaType.includes(MediaType.TEXT_PLAIN);
        }
        return true;
    }

    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return Collections.singletonList(MediaType.TEXT_PLAIN);
    }

    @Override
    public User read(Class<? extends User> clazz, HttpInputMessage inputMessage) 
    		throws IOException, HttpMessageNotReadableException {
        BufferedReader br = 
        		new BufferedReader(new InputStreamReader(inputMessage.getBody()));
        StringBuilder sb = new StringBuilder();
        String res = null;
        while ((res = br.readLine()) != null) {
            sb.append(res);
        }
        res = sb.toString();
        String[] elements = res.split(",");
        if (elements.length == 3) {
            User user = new User();
            user.setId(Long.parseLong(elements[0]));
            user.setName(elements[1]);
            user.setAge(Integer.parseInt(elements[2]));
            return user;
        }
        return null;
    }

    @Override
    public void write(User user, MediaType contentType, HttpOutputMessage outputMessage) 
    		throws IOException, HttpMessageNotWritableException {
        String res = user.toString();
        PrintWriter pw = 
        		new PrintWriter(new OutputStreamWriter(outputMessage.getBody()));
        pw.println(res);
        pw.flush();
    }
}
```

重写 `WebMvcConfigurer#extendMessageConverters()` 添加消息转换器，或者把它注册 Spring 组件。

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new UserHttpBodyConverter());
    }
}
```

处理器方法，使用消息转换器读取和写出 `User` 对象。

```
@PostMapping("/user")
@ResponseBody
public User postUser(@Valid @RequestBody User user) {
    System.out.println(user);
    return user;
}
```

## 异常处理

`@ExceptionHandler` 标注方法，用于捕获处理器方法抛出的异常，属性 value 表示能捕获的异常类型列表，还可以用方法参数匹配异常。

另外，`@ExceptionHandler` 还能匹配抛出异常的祖先类型，当然，精确匹配优先，如果匹配祖先类型，继承关系越近越优先。

`@ExceptionHandler` 方法可用的参数类型、返回类型，与 `@RequestMapping` 方法相同。它的返回，跟处理器方法的返回作用相同。

##  控制器通知

`@ControllerAdvice` 标注类，其中定义的 `@ModelAttribute`、`@InitBinder` 和 `@ExceptionHandler` 方法将用于全局的处理器。可以理解为，`@ControllerAdvice` 是处理器的通知。

`@ControllerAdvice` 默认对全局的处理器生效，可用 value 属性指定拦截哪些包和子包的处理器，当然，还有其它属性用于缩小拦截范围，暂不赘述。

# Java Config MVC

## 开启 MVC

`@EnableWebMvc` 标注配置类，启用基于注解的 MVC 功能，这会注册大量相关的组件。

```
@Configuration
@EnableWebMvc
public class App {...}
```

## 配置 MVC

接口 `WebMvcConfigurer` 表示 Spring MVC 配置，可以通过重写其特定方法来配置 Spring MVC，比如添加自定义的拦截器。

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    // 添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
    }
        
    // 添加消息转换器
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
    }
    
    // 开启静态资源的默认处理
    @Override
    ... configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

# 跨域请求 CORS

## 相关概念

为了安全，浏览器禁止页面的脚本访问不同源的资源，所谓同源，指协议、域名、端口都相同。为什么不安全？比如网页使用浏览器的凭证访问银行账户的资源。

Cross-Origin Resource Sharing，CORS，是一项 W3C 规范，用于支持跨域访问，它的实现需要浏览器和服务端的共同支持，目前大多浏览器都实现 CORS。

整个 CORS 通信过程，对于浏览器用户和脚本程序，完全透明。浏览器一旦发现 AJAX 请求跨域，就会自动为其添加一些头信息，有时还要先发出一个预检请求。所以，CORS 通信的关键在于服务端，只要服务端返回允许跨域请求的字段，就能进行 CORS 通信。

**简单请求**

浏览器把跨域请求分为简单请求和其它请求，简单请求需要满足以下条件：

* 请求方法：GET、POST、HEAD；
* 请求头字段：`Accept`、`Accept-Language`、`Content-Language`、`Last-Event-ID`、`Content-Type`；
* `Content-Type`：`application/x-www-form-urlencoded` 或 `multipart/form-data` 或 `text/plain`；

对于简单请求，浏览器会在请求头自动添加 `Origin` 字段，包含请求源的协议、域名、端口，服务端根据这个字段决定是否返回请求的数据。如果允许跨域，响应头应该有以下几个字段：

* `Access-Control-Allow-Origin`：必须，允许的请求源，值是 `Origin` 或 `*`。
* `Access-Control-Expose-Headers`：可选，列出 AJAX 可以额外获得的头信息字段。
* `Access-Control-Allow-Credentials`：可选，是否允许 Cookie 字段。

如果不允许跨域，服务端也会返回 200 响应，但是没有上面任何字段，浏览器抛出一个错误。

**其它请求**

对于其它请求，浏览器会先发出一个预检请求，询问服务端是否允许此次跨域，只有得到肯定答复，浏览器才会真正发出 CORS 请求，否则抛出一个错误，这个 CORS 请求和普通请求的发送方式相同。

> `PUT`、`DELETE` 和 `Content-Type=application/json` 都是其它请求，比较常见。

预检请求的方法是 `OPTIONS`，它有以下请求头字段：

* `Origin`：包含请求源的协议、域名、端口。
* `Access-Control-Request-Method`：跨域请求会用到的方法。
* `Access-Control-Request-Headers`：跨域请求会额外发送的请求头字段。

服务端收到预检请求，如果允许跨域，返回的响应头应该有以下字段：

* `Access-Control-Max-Age`：可选，本次预检的有效期，单位秒。
* `Access-Control-Allow-Origin`：必须，允许的请求源。
* `Access-Control-Allow-Methods`：必须，允许的请求方法。
* `Access-Control-Allow-Headers`：可选，允许的请求头字段。
* `Access-Control-Allow-Credentials`：可选，是否允许 Cookie 字段。

如果不允许，服务端也会返回 200 响应，但是没有上面任何字段。

## 跨域策略

Spring MVC 的 `HandlerMapping` 实现提供 CORS 支持。`@CrossOrigin` 标注处理器、处理器方法，可以配置局部的跨域策略，默认允许任何请求源、请求方法、请求头字段，有效期 30 分钟，但不允许 Cookie 字段，可用注解属性修改。

```
@Controller
public class AccountController {

    @CrossOrigin
    @GetMapping("/account")
    public Account retrieve(Long id) {
        // ...
    }
}
```

使用 Java Config 修改 `HandlerMapping` 配置，全局有效。

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowedHeaders("*")
                .exposedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

创建过滤器，手动设置 CORS，其实就是添加一些响应头字段。

```
@Bean
public CorsFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.setMaxAge(3600L);
    config.addAllowedOrigin("*");
    config.addAllowedHeader("GET", "POST", "PUT", "DELETE");
    config.addAllowedMethod("*");
    config.setAllowCredentials(true);
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}
```
