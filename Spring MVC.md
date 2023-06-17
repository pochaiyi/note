# Spring MVC

> version 5.3.13

Spring Web MVC 是基于 Servlet API 的 web 框架，基于 MVC 架构，解耦合代码。它包含在 Spring Framework 中。Spring Framework 5.0 还引入与之平行的反应式 web 框架 Spring WebFlux。

Spring MVC 本身也是一个容器，使用 IoC 技术，管理各种组件。Spring MVC 的底层就是 Servlet，以 Servlet 为核心，接收、处理请求，返回处理结果给客户端。之前这个功能是由 Servlet 来实现的，现在使用 Spring MVC 代替 Servlet。

## 什么是 MVC？

Model View Controller，MVC 是一种软件设计模式，它将代码拆分为 模型 + 视图 + 控制器。这样做得好处其实就是典型得 “高内聚，低耦合”。

View 视图是指返回给客户端的内容，可以是 JSP、HTML、Json，这要看项目是如何设计的。如果是单体引用，可以直接返回 JSP，JSP 本质是 Servlet，它会将 JSP 页面的内容通过 Response 写出为 HTML 数据。如果是前后端分离项目，服务端就直接返回 Json 数据，虽然返回的是纯文本，但它在 MVC 中也叫视图，前端服务器再根据返回的 Json 数据显式内容。

Model 模型指业务规则，它拥有最多的代码，负责处理请求，然后返回数据给视图，视图需要处理这些数据，然后得到的才是前端需要的数据格式。Model 只负责返回原始数据，而数据如何处理是 View 的事，所以，相同的 Model 可以给多个不同的 View 提供数据，从而给不同的客户端提供相同数据的不同显式。

Controller 控制器本身不进行任何处理，所以我们在编码时不要为了图方便将业务逻辑直接写在 `@Controller` 组件中。它负责接收客户端发送的请求，至于请求发送给哪个控制器，在 Spring MVC 中是通过 `DispatcherServlet` 匹配请求路径来解决的。然后，根据请求路径、请求参数、报表等信息，调用相应的 Model 组件，再将 Modle 的结果返回给特定的 View。此外，`DispatcherServlet` 也应该归于控制器，只不过它是所有 `@Controller` 的控制器。

## 什么是三层架构

三层架构和 MVC 一样，也是一种软件设计模型，反正我这么理解，不要管什么架构、模式、思想的区别。它的目的也是 “高内聚，低耦合”，复用代码，提高开发效率。

三层架构将代码拆分为 表现层 Web 层 + 业务逻辑层 Service + 数据访问层 DAO。

DAO 层没什么要解释的，就是那些访问数据库的代码。Service 层就是业务逻辑的代码。而 Web 层相当于 Controller + View，负责请求的接收，调用 Service，返回数据给客户端。

所以，三层架构与 MVC 是不能一一对应的，它们在 Spring MVC 交混使用。

最后，还有一个实体类层，这没什么好解释的，它用于封装从数据库获得的数据，并在三个层之间传递数据。

# 前端控制器 DispatcherServlet

Spring MVC 和其它许多 web 框架一样，是围绕前端控制器模式设计的。其中心 `DispatcherServlet` 就是前端控制器，为所有请求提供统一的处理流程，但实际工作被委托给 IoC 的组件执行。`DispatcherServlet` 与其它 Servlet 一样，需要在 Java 配置或 `web.xml` 中根据 Servlet 规范进行声明和映射。

## 配置 DispatcherServlet

在 Servlet 3.0+ 的环境中，可以选择以代码的方式配置 Servlet 容器，或者使用原始的 `web.xml` 方式。

**WebApplicationInitializer**

`WebApplicationInitializer` 是 Spring MVC 的接口，MVC 会检测此接口的实现并自动将其用于 Servlet 3 容器的初始化。

下例使用 `WebApplicationInitializer` 接口配置 `DispatcherServlet`（被 Servlet 容器自动检测）：

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new
        		AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext
        		.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

**AbstractDispatcherServletInitializer**

`WebApplicationInitializer` 的子类 `AbstractDispatcherServletInitializer` 使得 `DispatcherServlet` 的配置更加简单，只需重写特定的方法。下例使用 `AbstractDispatcherServletInitializer` 配置 `DispatcherServlet`（被 Servlet 容器自动检测）：

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

`AbstractDispatcherServletInitializer` 还提供一种方便的方式来注册 `Filter`，并将它们自动映射到 `DispatcherServlet`，如下所示：

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

每个 `Filter` 根据其实体类型设置默认名称，并自动映射到 `DispatcherServlet`。

`AbstractDispatcherServletInitializer` 的方法 `isAsyncSupported` 用于开启 `DispatcherServlet` 和所有映射到它的 `Filter` 的异步支持。默认返回 `true`。

如果需要进一步自定义 `DispatcherServlet`，重写 `createDispatcherServlet` 方法。

**AbstractAnnotationConfigDispatcherServletInitializer**

还可以使用 `AbstractAnnotationConfigDispatcherServletInitializer` 抽象类配置 `DispatcherServlet`，如下所示：

```
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

## 具有层次结构的上下文

`DispatcherServlet` 含有一个 `WebApplicationContext` 属性，这就是它所关联的 IoC 容器，即 MVC 容器。MVC 容器也包含 `ServletContext` 和与其关联的 `Servlet` 的链接，这样它们就实现相互绑定。在需要时获取对方的资源进行工作。

通常只使用 MVC 容器已经足够完成工作，但有时需要与其它 Spring 整合，获得具有层次结构的 IoC 容器。Spring 容器会被 `DispatcherServlet` 实例共享，同时也被它的 MVC 容器继承。这样，MVC 容器便实现层次结构，且能使用 Spring 容器中的组件进行工作。MVC 容器通常只注册 Controller 和 MVC 特殊组件。

以下 Java 配置注册具有层次结构上下文的 `DispatcherServlet`：

```
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	// RootConfig 是父容器配置类
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

	// App1Config 是 MVC 容器配置类
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```

如果不需要实现上下文层次结构，重写 `getRootConfigClasses()` 返回 `null` 即可。

以下 `web.xml` 注册具有层次结构上下文的 `DispatcherServlet`：

```
<web-app>

    <listener>
        <listener-class>
        	org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>
        	org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

`ContextLoaderListener` 监听器用于配置 Spring 容器，上下文参数 `contextConfigLocation` 表示配置文件的位置。如果不需要实现上下文层次结构，删除监听器配置即可。

## MVC 的特殊组件

`DispatcherServlet` 将请求处理委托给特殊的 Bean 执行。特殊 Bean 指实现框架约定并由 MVC 容器管理的对象。主要就是 MVC 九大组件。这些特殊 Bean 通常都已内置，但也能拓展或替换它们。

下表列举 `DispatcherServlet` 的特殊组件：

| Bean type                                 | Explanation                                                  |
| :---------------------------------------- | :----------------------------------------------------------- |
| `HandlerMapping`                          | 将请求映射到处理器 Handler 以及对应的拦截器列表。映射基于一些条件，具体由 `HandlerMapping` 的实现决定。MVC 有两个主要实现 `RequestMappingHandlerMapping` 和 `SimpleUrlHandlerMapping`，前者支持 `@RequestMapping` 方法，后者维护 URI 路径到 Handler 的显式注册。 |
| `HandlerAdapter`                          | 帮助 `DispatcherServlet` 执行被请求映射到的 Handler 程序，可以将其理解为一个反射工具，用于调用特定的程序。这个组件的作用是封装调用的细节，方便 `DispatcherServlet` 的执行。 |
| `HandlerExceptionResolver`                | 异常的处理策略，可能将异常映射到 Handler 处理器、HTML 错误视图或其它目标。 |
| `ViewResolver`                            | 解析 Handler 返回的基于字符串的逻辑视图名称，从而选择匹配的视图返回给客户端。 |
| `LocaleResolver`，`LocaleContextResolver` | 解析客户端的 `Locale` 以及可能的时区，以便提供国际化的返回视图。 |
| `ThemeResolver`                           | 解析应用可以使用的主题。                                     |
| `MultipartResolver`                       | 使用 multipart 解析的包来解析 multi-part 请求，比如文件上传的请求。 |
| `FlashMapManager`                         | 存储和检索输入和输出的 `FlashMap`，它们可用于将属性从一个请求传递到另一个请求，通常是通过重定向。 |

可以在应用中声明自己的特殊 Bean，`DispatcherServlet` 会检查 `WebApplicationContext` 中所有的特殊的 Bean 类型。如果没有类型匹配的组件，将使用 `DispatcherServlet.properties` 默认配置，内容如下：

```
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping,\
	org.springframework.web.servlet.function.support.RouterFunctionMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter,\
	org.springframework.web.servlet.function.support.HandlerFunctionAdapter


org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

大部分情况，使用 MVC Config 进行特殊 Bean 配置是较好的方式。

## 处理请求过程

`DispatcherServlet` 处理请求的流程如下：

- 查找 `WebApplicationContext` 并将其绑定为 Request 的属性，以让其它处理请求的组件能够使用 MVC 容器。这个属性的 Key 默认为 `DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`。

- 绑定 `LocaleResolver` 到请求，以供处理本地化的组件使用，如果不需要本地化解析，则忽略。

- 绑定 `ThemeResolver` 到请求，以让后面的组件决定使用哪个主题，如果不使用主题，则忽略。

- 如果有配置 `MultipartResolver`，则会检查请求是否含有 `multiparts`。如果有，则将请求包装为 `MultipartHttpServletRequest` 以供后面的组件进一步处理。

- 查找匹配的 Hanlder，并运行 Hanlder 的调用链（包含拦截器），返回用于渲染的模型 Model。

  或者，可以直接返回响应，不需要返回 Model 给视图，这是在 `HandlerAdapter` 中完成的。

- 如果前面步骤返回 Model，则渲染视图。否则不返回视图，请求可能已经处理完。

`WebApplicationContext` 中声明的 `HandlerExceptionResolver` 实例用于处理请求处理过程中抛出的异常。这些异常处理器允许自定义处理逻辑。

可以通过在 `web.xml` 中设置 S	ervlet 初始化参数（`init-param`）自定义 `DispatcherServlet` 实例。下表列举支持的选项：

| Parameter                        | Explanation                                                  |
| :------------------------------- | :----------------------------------------------------------- |
| `contextClass`                   | 实现 `ConfigurableWebApplicationContext` 的类，由当前的 Servlet 负责实例化和配置它，默认为 `XmlWebApplicationContext`。 |
| `contextConfigLocation`          | 传递给 MVC 上下文的字符串，指定元数据配置的位置。可包含多个字符串，使用 `,` 分隔，以支持多个上下文。对于在多个 Location 中定义的 Bean，最新的位置优先级最高。 |
| `namespace`                      | `WebApplicationContext` 的名称空间，默认为 `[servlet-name]-servlet`。 |
| `throwExceptionIfNoHandlerFound` | 没有匹配的 Handler 时，是否抛出 `NoHandlerFoundException` 异常。这个异常可以被 `HandlerExceptionResolver` 捕获并处理。默认为 `false`，此时 `DispatcherServlet` 直接返回 404 给客户端，而不抛出任何异常。注意，如果配置有默认的 Servlet 处理，未处理的请求将总是被转发到默认 Serlvet，不会被返回 404。 |

## Path Matching

The Servlet API exposes the full request path as `requestURI` and further sub-divides it into `contextPath`, `servletPath`, and `pathInfo` whose values vary depending on how a Servlet is mapped. From these inputs, Spring MVC needs to determine the lookup path to use for handler mapping, which is the path within the mapping of the `DispatcherServlet` itself, excluding the `contextPath` and any `servletMapping` prefix, if present.

The `servletPath` and `pathInfo` are decoded and that makes them impossible to compare directly to the full `requestURI` in order to derive the lookupPath and that makes it necessary to decode the `requestURI`. However this introduces its own issues because the path may contain encoded reserved characters such as `"/"` or `";"` that can in turn alter the structure of the path after they are decoded which can also lead to security issues. In addition, Servlet containers may normalize the `servletPath` to varying degrees which makes it further impossible to perform `startsWith` comparisons against the `requestURI`.

This is why it is best to avoid reliance on the `servletPath` which comes with the prefix-based `servletPath` mapping type. If the `DispatcherServlet` is mapped as the default Servlet with `"/"` or otherwise without a prefix with `"/*"` and the Servlet container is 4.0+ then Spring MVC is able to detect the Servlet mapping type and avoid use of the `servletPath` and `pathInfo` altogether. On a 3.1 Servlet container, assuming the same Servlet mapping types, the equivalent can be achieved by providing a `UrlPathHelper` with `alwaysUseFullPath=true` via [Path Matching](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-path-matching) in the MVC config.

Fortunately the default Servlet mapping `"/"` is a good choice. However, there is still an issue in that the `requestURI` needs to be decoded to make it possible to compare to controller mappings. This is again undesirable because of the potential to decode reserved characters that alter the path structure. If such characters are not expected, then you can reject them (like the Spring Security HTTP firewall), or you can configure `UrlPathHelper` with `urlDecode=false` but controller mappings will need to match to the encoded path which may not always work well. Furthermore, sometimes the `DispatcherServlet` needs to share the URL space with another Servlet and may need to be mapped by prefix.

The above issues can be addressed more comprehensively by switching from `PathMatcher` to the parsed `PathPattern` available in 5.3 or higher, see [Pattern Comparison](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-pattern-comparison). Unlike `AntPathMatcher` which needs either the lookup path decoded or the controller mapping encoded, a parsed `PathPattern` matches to a parsed representation of the path called `RequestPath`, one path segment at a time. This allows decoding and sanitizing path segment values individually without the risk of altering the structure of the path. Parsed `PathPattern` also supports the use of `servletPath` prefix mapping as long as the prefix is kept simple and does not have any characters that need to be encoded.

## 拦截器 Interception

所有 `HandlerMapping` 实现都支持 Handler Interceptor，它们在实现特定功能的时候特别有用。拦截器必须实现 `org.springframework.web.servlet.HandlerInterceptor` 接口，其包含 3 个方法：

- `preHandle(..)`：Handler 之前运行
- `postHandle(..)`：Handler 之后运行
- `afterCompletion(..)`：整个请求结束后执行

`preHandle(..)` 返回一个布尔值，表示是否继续执行 Handler 调用链。返回 `true` 时，继续执行调用链，否则，`DispatcherServlet` 认为拦截器已经处理完请求，不再执行调用链剩余的拦截器和 Handler。

`postHandle` 对于 `@ResponseBody` 和 `ResponseEntity` 方法不太有用，因为此时响应已经在 `postHandle` 之前的 `HandlerAdapter` 中完成，这意味着在 `postHandle` 方法中操作响应已经太晚。对于这种情况，可以实现 `ResponseBodyAdvice` 接口，将其声明为 Controller Advice 或者直接在 `RequestMappingHandlerAdapter` 配置它。

## 异常 Exception

如果在请求映射时、或在请求处理过程中（如 Handler）抛出异常，`DispatcherServlet` 会委托 `HandlerExceptionResolver` 的调用链来处理异常并提供兜底的处理，这个兜底处理通常是 error 响应。

下表列举可用的 `HandlerExceptionResolver` 实现：

| `HandlerExceptionResolver`          | Description                                                  |
| :---------------------------------- | :----------------------------------------------------------- |
| `SimpleMappingExceptionResolver`    | 异常类名和错误视图名之间的映射，用于渲染浏览器应用的错误页面。 |
| `DefaultHandlerExceptionResolver`   | 解析 Spring MVC 抛出的异常，并映射为 HTTP 的状态码。另请参阅 `ResponseEntityExceptionHandler` 和 REST API 异常。 |
| `ResponseStatusExceptionResolver`   | 使用 `@ResponseStatus` 注解处理异常，并根据注解中的值将它们映射到 HTTP 状态码。 |
| `ExceptionHandlerExceptionResolver` | 使用 `@Controller` 或 `@ControllerAdvice` 中的 `@ExceptionHandler` 方法处理异常。 |

### 异常处理链

可以在 MVC 容器中注册多个 `HandlerExceptionResolver` 实现并设置 `order` 属性（如果需要）来组合异常处理调用链。`order` 值越高，距离中心就越远。

`HandlerExceptionResolver` 能返回：

* 表示错误视图的 `ModelAndView`。
* 空 `ModelAndView`，当异常已在 `HandlerExceptionResolver` 中被处理。
* `null`，异常仍未被处理，让后面的 `HandlerExceptionResolver` 继续尝试。如果最后异常依然存在，则异常会冒泡到 Servlet 容器。

MVC Config 自动配置内置的 `HandlerExceptionResolver`，用于处理 Spring MVC 异常，`@ResponseStatus` 标注的异常，和支持 `@ExceptionHandler` 方法。这些异常处理器可以自定义和替换。

### Servlet 容器的错误页面

如果异常未被任何 `HandlerExceptionResolver` 处理而继续传播，或响应状态码被设置为错误（ 4xx，5xx），Servlet 容器会返回默认的 HTML 错误页面。如果要自定义这个页面，可以在 `web.xml` 中设置错误页面映射，内容如下：

```
<error-page>
    <location>/error</location>
</error-page>
```

经过上面的配置，如果一个异常冒泡到 Servlet 容器或 Response 被设置错误响应码，Servlet 容器会捕获这个异常，然后在容器内对配置的 URL（`/error`）发送错误调度（Error Dispatch），即 Servlet 容器自己发出一个请求。这个错误请求后续由 `DispatcherServlet` 处理，可能映射到 `@Controller`，在其中返回 Json 或包含 Error View Name 的 Model。如下所示：

```
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```

Servlet API 不提供在 Java 中配置错误视图映射的方法，可以考虑使用 `WebApplicationInitializer` 和一个很少内容的 `web.xml` 来实现。

## View Resolution

Spring MVC 定义 `ViewResolver` 和 `View` 接口，使得我们不需特别的视图技术便能将模型（Model）渲染到浏览器。`ViewResolver` 提供视图名与视图 View 之间的映射。`View` 用于把数据交给视图技术前的准备工作。

下表提供 `ViewResolver` 层次结构的详细信息：

| ViewResolver                     | Description                                                  |
| :------------------------------- | :----------------------------------------------------------- |
| `AbstractCachingViewResolver`    | Subclasses of `AbstractCachingViewResolver` cache view instances that they resolve. Caching improves performance of certain view technologies. You can turn off the cache by setting the `cache` property to `false`. Furthermore, if you must refresh a certain view at runtime (for example, when a FreeMarker template is modified), you can use the `removeFromCache(String viewName, Locale loc)` method. |
| `UrlBasedViewResolver`           | Simple implementation of the `ViewResolver` interface that effects the direct resolution of logical view names to URLs without an explicit mapping definition. This is appropriate if your logical names match the names of your view resources in a straightforward manner, without the need for arbitrary mappings. |
| `InternalResourceViewResolver`   | Convenient subclass of `UrlBasedViewResolver` that supports `InternalResourceView` (in effect, Servlets and JSPs) and subclasses such as `JstlView` and `TilesView`. You can specify the view class for all views generated by this resolver by using `setViewClass(..)`. |
| `FreeMarkerViewResolver`         | Convenient subclass of `UrlBasedViewResolver` that supports `FreeMarkerView` and custom subclasses of them. |
| `ContentNegotiatingViewResolver` | Implementation of the `ViewResolver` interface that resolves a view based on the request file name or `Accept` header. |
| `BeanNameViewResolver`           | Implementation of the `ViewResolver` interface that interprets a view name as a bean name in the current application context. This is a very flexible variant which allows for mixing and matching different view types based on distinct view names. Each such `View` can be defined as a bean e.g. in XML or in configuration classes. |

### 视图处理 Handling

可以声明多个 `ViewResolver` 实现和设置 `order` 属性（如果需要）来组合视图解析调用链。`ViewResolver` 能返回 `null` 表示无法找到匹配的 `view`。但是在 JSP 和 `InternalResourceViewResolver` 的情况下，确定 JSP 是否存在的唯一方式是通过 `RequestDispatcher` 进行请求分发。所以，应该总是配置一个`InternalResourceViewResolver` 并作为最后位序的视图解析器。

配置视图解析（View Resolution）就像添加视图解析器到 Spring 一样简单。MVC Config 为添加 View Resolver 和 View controller 提供专门的 API。

### 请求重定向 Redirect

视图名的特殊前缀 `redirect:` 表示请求重定向。`UrlBasedViewResolver`（及其子类）将识别这个前缀。视图名的其余部分就是重定向的目标 URL。

实际效果与 Controller 返回 `RedirectView` 相同，但现在控制器自己可以根据视图名进行操作。逻辑视图名，比如 `redirect:/myapp/some/resource`，是相对于当前 Servlet Context，而形如  `redirect:https://myhost.com/some/arbitrary/path` 的 URL 表示绝对路径。

如果一个 Controller 方法标注 `@ResponseStatus`，则其注解值优先于 `RedirectView` 设置的响应状态码。

### 请求转发 Forwarding

视图名的特殊前缀 `forward:` 表示请求转发，它将被 `UrlBasedViewResolver`（及其子类）解析。这将创建一个 `InternalResourceView` 用于执行 `RequestDispatcher.forward()`。所以，这个前缀对 `InternalResourceViewResolver` 和 `InternalResourceView`（JSP）没有用。但是如果您使用其他视图技术，却仍希望强制 Servlet/JSP 引擎处理资源的转发，此前缀会很有帮助。

### 内容协商 Content Negotiation

`ContentNegotiatingViewResolver` 本身不解析视图，而是委托给其它视图解析器并选择与客户端请求的表现形式匹配的视图。视图表现形式能通过 `Accept` 请求头或查询参数获取（`"/path?format=pdf"`），`Accept` 可以包含通配符（`text/*`）。

`ContentNegotiatingViewResolver` 将请求媒体类型和它所有 `ViewResolver` 关联的 `View` 支持的媒体类型进行比较，选择合适的 `view` 来处理请求。匹配表中第一个 `view` 会被选中，用于处理数据。如果 `ViewResolver` 无法提供合适的视图，则将参考 `DefaultViews` 属性指定的视图列表。

## Multipart 处理

`org.springframework.web.multipart` 包的 `MultipartResolver` 接口是用专门用于解析 multipart 请求（主要是文件上传请求）的策略。它有基于 `Commons FileUpload` 或 Servlet 3.0 multipart request parsing 的两种实现。

要启用 multipart 处理，需要在 MVC 容器注册名为 `multipartResolver` 的 `MultipartResolver` 实例。`DispatcherServlet` 检测到它并将其应用于传入的请求。当遇到媒体类型为 `multipart/form-data` 的 POST 请求时，`MultipartResolver` 会解析其内容，并将 `HttpServletRequest` 包装为 `MultipartHttpServletRequest`，以提供对被解析内容的访问以及将部分解析内容暴露为请求参数，

### Apache Commons FileUpload

要使用 `Apache Commons FileUpload`，需要配置 `CommonsMultipartResolver` 类型的 Bean，并命名为 `multipartResolver`。当然，还需要引入 `commons-fileupload` 依赖：

```
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="#{1024*1024*10}"/>
    <property name="defaultEncoding" value="UTF-8"/>
</bean>
```

`Commons FileUpload` 传统上仅适用于 POST 请求，但接受任何 `multipart/` 的媒体类型。

### Servlet 3.0

使用 Servlet 3.0 的 multipart 解析，需要显式启用，有两种方式：

* 在 `web.xml` 的 Servlet 声明中添加 `<multipart-config>`
* 或使用 Java 配置，在 Servlet Registration 上设置 `MultipartConfigElement`，内容如下：

```
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```

启用成功后，就可以直接配置一个名为 `multipartResolver ` 的 `StandardServletMultipartResolver ` 类型的 Bean。

```
<bean class="org.springframework.web.multipart.support.StandardServletMultipartResolver" id="multipartResolver">
</bean>
```

这种方式默认解析任何 HTTP 方法的任何 `multipart/` 的媒体类型，但并非所有 Servlet 容器都支持此功能。

# 过滤器 Filter

过滤器是 Servlet 容器的概念，拦截器是 Spring 的概念。

`spring-web` 模块内置以下过滤器：

- `Form Data`
- `Forwarded Headers`
- `Shallow ETag`
- `CORS`

## 访问表单数据

浏览器只能通过 GET 或 POST 来提交表单数据，非浏览器用户还可以使用 PUT，PATCH 和 DELETE 方法。Servlet API 仅支持使用 `ServletRequest.getParameter*()` 系列方法从 POST 请求访问表单数据。

为访问其它提交方法的表单数据，`spring-web` 提供 `FormContentFilter` 过滤器，拦截媒体类型为 `application/x-www-form-urlencoded` 的 PUT，PATCH 和 DELETE 请求，从请求体中读取表单数据，并包装 请求使得可以通过 `ServletRequest.getParameter*()` 系列方法访问表单数据。

## 转发头 Forwarded Headers

当请求通过代理时（如负载均衡），它的 Host，Port 和 Scheme 等信息可能发生变化，这使得难以从服务端角度创建指向正确的 Host，Port 和 Scheme 客户端的链接。RFC-7239 定义的 `Forwarded` HTTP 请求头可用于代理请求，以提供原始请求信息。

`ForwardedHeaderFilter` 过滤器用于修改请求：根据 `Forwarded` 请求头信息修改 Host，Port 和 Scheme，或者删除这些请求头以消除进一步的影响。过滤器依赖于包装的请求，所以 `ForwardedHeaderFilter` 应该先于其它过滤器执行。

对于转发请求还有安全的考虑，应用并不清楚额外添加的请求头是否是恶意的。这也是为什么需要删除外部请求中不信任的 `Forwarded` 请求头。还能设置 `ForwardedHeaderFilter` 的属性 `removeOnly=true`，这样，它只会删除但不使用 `Forwarded` 请求头。

为支持异步请求和错误调度，需要将此过滤器映射到 `DispatcherType.ASYNC` 和 `DispatcherType.ERROR`。如果使用 Spring Framework 的 `AbstractAnnotationConfigDispatcherServletInitializer`，所有过滤器会自动为所有 Dispatch Type 注册。但如果使用 `web.xml` 或通过 Spring Boot 的 `FilterRegistrationBean` 来注册过滤器，请确保除 `DispatcherType.REQUEST` 外还包括 `DispatcherType.ASYNC` 和 `DispatcherType.ERROR`。

## CORS

Spring MVC 通过 Controller 上的注解为 CROS 配置提供细粒度的支持。但与 Spring Security 一起使用时，建议使用内置的 `CorsFilter`，且必须将它配置在 Spring Security 的过滤器调用链之前。

## Encoding

POST 请求上传的中文可能会乱码，可以使用编码过滤器解决此问题：

```
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 控制器 Controller

Spring MVC 提供基于注解的编码模式，`@Controller` 和 `@RestController` 组件使用注解来表示请求映射，请求输入，异常处理等。控制器具有灵活的方法签名，不需拓展特定的类或实现特定的接口。

## 控制器声明

`@Controller` 标注的类会被组件扫描检测到，因为它是被 `@Component` 元注解标注的。

```
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

```
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

`@RestController` 是 `@Controller` 和 `@ResponseBody` 的组合注解，表示这个控制器的所有方法都继承 `@ResponseBody`，直接将返回写入响应并使用 HTML 模板渲染。

某些情况，可能需要使用 AOP 代理控制器。官方建议使用 Class-Based 的代理。

## 请求映射

`@RequestMapping` 可以将请求映射到控制器的方法。它具有各种属性，用于匹配 URL，HTTP 方法，请求参数，请求头和媒体类型。使用它标注 Controller 可以提供公共的映射，或标注方法提供特定的映射。

以下是 `@RequestMapping` 的 HTTP 方法变体，它们都是组合注解：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

### URI 匹配

可以使用 URL 模式映射到 `@RequestMapping` 方法，有两种方式：

* `PathPattern`：与 URL 路径匹配的预编译模式，同时也预编译为 `PathContainer`。专为 web 使用而设计，可有效处理编码和路径参数，且匹配效率高。
* `AntPathMatcher`：与字符串路径匹配的字符串模式。这也是 Spring 配置从类路径、文件系统或其它位置查找资源的传统方式。它的效率较低，且难以处理字符串路径中的编码等问题。

`PathPattern` 是 web 应用推荐的方式，也是 Spring WebFlux 的唯一选择。在 5.3 之前，`AntPathMatcher` 是 Spring MVC 唯一选择且现在仍是默认方式。`PathPattern` 可以在 MVC Config 启用。

`PathPattern` 支持与 `AntPathMatcher` 相同的语法，它限制只能在末尾使用 `**` 匹配多个片段从而减少歧义。`PathPattern` 还支持使用 `{}` 捕获字符串路径中的内容，并转换为 URI 变量。

示例:

- `"/resources/ima?e.png"`：`?` 匹配任意一个字符
- `"/resources/*.png"`：匹配任意个任意字符，包括 0 个
- `"/resources/**"`：匹配任意个路径片段
- `"/projects/{project}/versions"`：匹配一个路径片段并捕获变量
- `"/projects/{project:[a-z]+}/versions"`：使用正则匹配并捕获变量

捕获的 URI 变量可以通过 `@PathVariable` 方法参数访问：

```
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

> **Pattern Comparison**
>
> 当 URL 被多个模式匹配时，必须选择最佳匹配项。可以通过以下方法之一完成，具体取决于使用的 URI Pattern：
>
> - `PathPattern.SPECIFICITY_COMPARATOR`
> - `AntPathMatcher.getPatternComparator(String path)`
>
> 两者都能将最优项排序到顶部。选择规则：具有较少 URI 变量（1分），具有 n 个通配符（n分），评分高的模式精确度高。相同得分时，选择较长的模式。等分等长时，选择 URI 变量多于通配符的模式。默认映射 `/**` 被排除在评分外，它总是排在最后。

### URI 路径变量

还可以在类上声明 URI 变量：

```
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI 变量会自动转换为适当的类型，或抛出 `TypeMismatchException` 异常。默认支持简单类型的转换，也可以注册对其它类型转换的支持。

`@PathVariable` 可以显式指定 URI 变量名，比如 `@PathVariable("customId")`。如果变量名相同，并且代码是通过调试信息或 Java 8 的 `-parameters` 编译器标记编译的，可以不指定 URI 变量名，也能正确匹配。

`{varName:regex}` 使用正则表达式来匹配 URI 变量，比如 `@GetMapping("/{name:[a-z-]+}/")`。

URL 路径模式还支持嵌入 `${…}`，这些占位符在启动时通过 `PropertyPlaceHolderConfigurer` 检索本地、系统、环境的属性资源。

### 后缀匹配

从 5.3 开始，Spring MVC 默认不再执行 `.*` 后缀匹配，映射到 `/person` 的控制器也隐式映射 `/person.*`。因此，不再使用拓展标记表示返回类型。首选使用 HTTP 的 `Accept` 请求头表示接受的返回类型。

### content-type 映射

根据请求体的媒体类型缩小映射范围，如下所示：

```
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

`consume` 还支持否定表达，如 `!text/plain`。可以在类级别声明 `consume` 属性，方法级别的 `consume` 会覆盖类级别的设置。

`MediaType` 枚举类提供常用的媒体类型，如 `APPLICATION_JSON_VALUE` 和 `APPLICATION_XML_VALUE`。

### Accept 映射

将 `Accept` 请求头和指定的媒体类型列表进行比较，缩小映射范围。如下所示：

```
@GetMapping(path = "/pets/{petId}", produces = "application/json") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

可以指定一个媒体类型的集合。支持否定表达，如 `!text/plain`。可以在类级别声明 `produces` 属性，方法级别的 `produces` 会覆盖类级别的设置。

### Parameter，header 映射

根据请求参数和请求头属性来缩小映射范围。可以指定某属性存在或不存在（`!`），还能指定值。如下所示：

```
@GetMapping(path = "/pets/{petId}", params = "!myParam") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

```
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

`headers` 能实现与 `consumes` 和 `produce` 同样的功能，但推荐使用专门的配置。

### 组合注解

Spring MVC 支持使用组合注解用于请求映射，这些注解被已经设置属性值的 `@RequestMapping` 元注解标注以用于特定的映射场景。`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping` 和 `@PatchMapping` 都是组合注解，用于特定的 HTTP 请求方法，`@RequestMapping`  默认映射所有 HTTP 方法。如果需要，可以参考这些注解来自定义组合注解。

Spring MVC 还支持自定义请求映射属性来自定义映射逻辑，这需要继承 `RequestMappingHandlerMapping` 并重写方法 `getCustomMethodCondition `。

## Handler 方法

Handler 方法，用于处理请求。它拥有灵活的签名，几乎没有什么限制。

### 方法参数

下表列举 Handler 方法支持的参数类型，任何参数都不支持反应式类型。`java.util.Optional` 类型的参数与具有 `required` 属性的注解（如 `@RequestParam`）结合，等价于 `required=false`。

| Controller method argument                                   | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `WebRequest`, `NativeWebRequest`                             | Generic access to request parameters and request and session attributes, without direct use of the Servlet API. |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | Choose any specific request or response type — for example, `ServletRequest`, `HttpServletRequest`, or Spring’s `MultipartRequest`, `MultipartHttpServletRequest`. |
| `javax.servlet.http.HttpSession`                             | Enforces the presence of a session. As a consequence, such an argument is never `null`. Note that session access is not thread-safe. Consider setting the `RequestMappingHandlerAdapter` instance’s `synchronizeOnSession` flag to `true` if multiple requests are allowed to concurrently access a session. |
| `javax.servlet.http.PushBuilder`                             | Servlet 4.0 push builder API for programmatic HTTP/2 resource pushes. Note that, per the Servlet specification, the injected `PushBuilder` instance can be null if the client does not support that HTTP/2 feature. |
| `java.security.Principal`                                    | Currently authenticated user — possibly a specific `Principal` implementation class if known.Note that this argument is not resolved eagerly, if it is annotated in order to allow a custom resolver to resolve it before falling back on default resolution via `HttpServletRequest#getUserPrincipal`. For example, the Spring Security `Authentication` implements `Principal` and would be injected as such via `HttpServletRequest#getUserPrincipal`, unless it is also annotated with `@AuthenticationPrincipal` in which case it is resolved by a custom Spring Security resolver through `Authentication#getPrincipal`. |
| `HttpMethod`                                                 | The HTTP method of the request.                              |
| `java.util.Locale`                                           | The current request locale, determined by the most specific `LocaleResolver` available (in effect, the configured `LocaleResolver` or `LocaleContextResolver`). |
| `java.util.TimeZone` + `java.time.ZoneId`                    | The time zone associated with the current request, as determined by a `LocaleContextResolver`. |
| `java.io.InputStream`, `java.io.Reader`                      | For access to the raw request body as exposed by the Servlet API. |
| `java.io.OutputStream`, `java.io.Writer`                     | For access to the raw response body as exposed by the Servlet API. |
| `@PathVariable`                                              | For access to URI template variables.                        |
| `@MatrixVariable`                                            | For access to name-value pairs in URI path segments.         |
| `@RequestParam`                                              | For access to the Servlet request parameters, including multipart files. Parameter values are converted to the declared method argument type. Note that use of `@RequestParam` is optional for simple parameter values. See “Any other argument”, at the end of this table. |
| `@RequestHeader`                                             | For access to request headers. Header values are converted to the declared method argument type. |
| `@CookieValue`                                               | For access to cookies. Cookies values are converted to the declared method argument type. |
| `@RequestBody`                                               | For access to the HTTP request body. Body content is converted to the declared method argument type by using `HttpMessageConverter` implementations. |
| `HttpEntity<B>`                                              | For access to request headers and body. The body is converted with an `HttpMessageConverter`. |
| `@RequestPart`                                               | For access to a part in a `multipart/form-data` request, converting the part’s body with an `HttpMessageConverter`. |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access to the model that is used in HTML controllers and exposed to templates as part of view rendering. |
| `RedirectAttributes`                                         | Specify attributes to use in case of a redirect (that is, to be appended to the query string) and flash attributes to be stored temporarily until the request after redirect. |
| `@ModelAttribute`                                            | For access to an existing attribute in the model (instantiated if not present) with data binding and validation applied.Note that use of `@ModelAttribute` is optional (for example, to set its attributes). See “Any other argument” at the end of this table. |
| `Errors`, `BindingResult`                                    | For access to errors from validation and data binding for a command object (that is, a `@ModelAttribute` argument) or errors from the validation of a `@RequestBody` or `@RequestPart` arguments. You must declare an `Errors`, or `BindingResult` argument immediately after the validated method argument. |
| `SessionStatus` + class-level `@SessionAttributes`           | For marking form processing complete, which triggers cleanup of session attributes declared through a class-level `@SessionAttributes` annotation. |
| `UriComponentsBuilder`                                       | For preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping. |
| `@SessionAttribute`                                          | For access to any session attribute, in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. |
| `@RequestAttribute`                                          | For access to request attributes.                            |
| Any other argument                                           | If a method argument is not matched to any of the earlier values in this table and it is a simple type (it is resolved as a `@RequestParam`. Otherwise, it is resolved as a `@ModelAttribute`. |

###  返回类型

下表列举 Handler 方法支持的返回类型，任何返回值都不支持反应式类型。

| Controller method return value                               | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                              | The return value is converted through `HttpMessageConverter` implementations and written to the response. |
| `HttpEntity<B>`, `ResponseEntity<B>`                         | The return value that specifies the full response (including HTTP headers and body) is to be converted through `HttpMessageConverter` implementations and written to the response. |
| `HttpHeaders`                                                | For returning a response with headers and no body.           |
| `String`                                                     | A view name to be resolved with `ViewResolver` implementations and used together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument. |
| `View`                                                       | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument . |
| `java.util.Map`, `org.springframework.ui.Model`              | Attributes to be added to the implicit model, with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `@ModelAttribute`                                            | An attribute to be added to the model, with the view name implicitly determined through a `RequestToViewNameTranslator`.Note that `@ModelAttribute` is optional. See "Any other return value" at the end of this table. |
| `ModelAndView` object                                        | The view and model attributes to use and, optionally, a response status. |
| `void`                                                       | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse`, an `OutputStream` argument, or an `@ResponseStatus` annotation. The same is also true if the controller has made a positive `ETag` or `lastModified` timestamp check.If none of the above is true, a `void` return type can also indicate “no response body” for REST controllers or a default view name selection for HTML controllers. |
| `DeferredResult<V>`                                          | Produce any of the preceding return values asynchronously from any thread — for example, as a result of some event or callback. |
| `Callable<V>`                                                | Produce any of the above return values asynchronously in a Spring MVC-managed thread. |
| `ListenableFuture<V>`, `java.util.concurrent.CompletionStage<V>`, `java.util.concurrent.CompletableFuture<V>` | Alternative to `DeferredResult`, as a convenience (for example, when an underlying service returns one of those). |
| `ResponseBodyEmitter`, `SseEmitter`                          | Emit a stream of objects asynchronously to be written to the response with `HttpMessageConverter` implementations. Also supported as the body of a `ResponseEntity`. |
| `StreamingResponseBody`                                      | Write to the response `OutputStream` asynchronously. Also supported as the body of a `ResponseEntity`. |
| Reactive types — Reactor, RxJava, or others through `ReactiveAdapterRegistry` | Alternative to `DeferredResult` with multi-value streams (for example, `Flux`, `Observable`) collected to a `List`.For streaming scenarios (for example, `text/event-stream`, `application/json+stream`), `SseEmitter` and `ResponseBodyEmitter` are used instead, where `ServletOutputStream` blocking I/O is performed on a Spring MVC-managed thread and back pressure is applied against the completion of each write. |
| Any other return value                                       | Any return value that does not match any of the earlier values in this table and that is a `String` or `void` is treated as a view name (default view name selection through `RequestToViewNameTranslator` applies), provided it is not a simple type, as determined by BeanUtils#isSimpleProperty. Values that are simple types remain unresolved. |

### 类型转化 Type Conversion

如果 Handler 参数声明为非 `String` 类型，表示基于字符串的请求输入的带某些注解（如 `@RequestParam`）的参数，需要进行类型转换。对于这种情况，将使用配置的类型转换器完成。默认执行简单类型的转换，可以通过 `WebDataBinder` 或向 `FormattingConversionService` 注册 `Formatters` 自定义类型转换逻辑。

类型转换的一个实际问题是如何处理空字符串。如果将此值转为 `null`，则该值被视为缺失。如果想要允许注入 `null`，可以使用注解标记 `required` 或者标注参数 `@Nullable`。从 5.3 开始，类型转换后会强制非 `null`，所以必须配置。

### 矩阵变量 Matrix Variables

RFC-3986 提出路径片段中的 name-value 键值对，Spring MVC 将它们称为矩阵变量，也可以称为 URI 路径参数。矩阵变量可以出现在任何路径片段中，变量间使用 `;` 分隔，值之间使用 `,` 分隔，比如 `/cars;color=red,green;year=2012`。还可以通过为相同名称重复赋值来设置多个值，比如  `color=red;color=green;color=blue`；

如果 URL 中预期含有矩阵变量，请求映射必须使用 URI 变量来装饰它们。也就是说，矩阵变量只能作为 URI 变量的附加品。而且，请求映射的成功与否独立于矩阵变量的顺序和存在。

使用矩阵变量，需要显式开启支持，有两种方式：

* Java 代码，通过`removeSemicolonContent=false` 配置  `UrlPathHelper`。
* XML，设置 `<mvc:annotation-driven enable-matrix-variables="true"/>`。

矩阵变量必须与路径变量合用，通过 `@MatrixVariable` 方法参数访问矩阵变量：

```
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

所有的 URI 变短都有可能携带矩阵变量，下例使用矩阵变量名和 URI 变量名进行消歧：

```
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

矩阵变量的定义是可选的，可以设置默认值以处理没有矩阵变量的情况，如下所示：

```
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

测试只设置 `required=false` 时还是会抛出异常，所以默认值必须一起设置。

可以使用 `MultiValueMap` 类型参数获取所有的矩阵变量：

```
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

### @RequestParam

`@RequestParam` 可以将请求参数（查询参数或 GET 表单字段）绑定到方法参数。如下所示：

```
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

默认情况，被这种注解标注的参数是必须的，可以设置 `required=false` 使得此参数可选，或者使用 `java.util.Optional` 包装。

如果参数类型不是 `String`，将自动进行类型转换。

将参数声明为数组或列表，可以解析名称相同的多个请求参数值。

当 `@RequestParam` 参数声明为 `Map<String,String>` 或 `MultiValueMap<String,String>`，且注解未指定参数名，将自动填充所有的请求参数。

`@RequestParam` 是可选的。当 Handler 参数是简单类型，且未被其它参数解析器处理，默认这个参数标有 `@RequestParam`。所以，有时可以直接使用参数来获取请求参数。

### @RequestHeader

`@RequestHeader` 可以将 HTTP 请求头绑定到方法参数。下面是当前请求的请求头内容：

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

使用 `@RequestHeader` 获取请求头的 `Accept-Encoding` 和 `Keep-Alive` 属性：

```
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

如果参数类型不是 `String`，将自动进行类型转换。

当 `@RequestHeader` 参数声明为 `Map<String, String>` 或 `MultiValueMap<String, String>` 或  `HttpHeaders`，默认填充所有请求头信息。

支持将使用 `,` 分隔的字符串转换为数组集合或其它类型转换支持的类型。比如，`@RequestHeader("Accept")` 参数可以被声明为 `String` 或 `String[]` 或 `List<String>`。

### @CookieValue

`@CookieValue` 可以将请求的 Cookie 绑定到方法参数。下面是当前请求的 Cookie 值：

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

使用 `@CookieValue` 获取 Cookie 值：

```
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

如果参数类型不是 `String`，将自动进行类型转换。

### @ModelAttribute

`@ModelAttribute` 可以将 Model 的属性绑定到方法参数，或者使其实例化（如果不存在）。Model 的属性还会被与其同名的请求参数覆盖，同名是指请求参数名与 Model 属性名相等。这就是数据绑定，它使得我们不需要专门处理和转型单个请求参数。下例示范操作：

```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) {
    // method logic...
}
```

上面方法参数获取 `Pet` 实例的流程如下：

- 根据参数名和类型检索 Model，因为 `@ModelAttribute` 方法可能提前添加这个属性到 Model 中。
- 检索 Session，如果 Controller 标有 `@SessionAttributes` 且声明 `pet` 属性名或 `Pet` 类型。
- 尝试通过 `Converter` 获取，如果域模型的属性名与请求值（比如请求参数，URI 变量等）的名称匹配。具体看下段内容。
- 使用 `Pet` 类型的默认构造器实例化。
- 使用 `Pet` 类型的 Primary 构造器实例化，其参数与请求参数匹配。构造器参数名可通过注解 `@ConstructorProperties` 获取，或者从运行时字节码中保留的信息获取。

上面第三种方式使用 `Converter<String, T>` 获取实例。当域模型的属性名与请求值，如请求参数或 URI 变量，的名称匹配，并且有 `Converter` 能将 `String` 转为属性类型时，将应用此方案。下例中，Model 属性名是 “account”，与 URI 变量匹配，且有 `Converter<String, Account>`：

```
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

获取实例后，还要进行数据绑定。`WebDataBinder` 将请求参数名与目标对象的字段名进行匹配，对象字段会被配对的请求参数值在转型后填充。

数据绑定可能会抛出异常 `BindException`。可以在数据绑定的参数后紧跟着声明 `BindingResult` 类型的参数，用于收集错误信息，此时方法不会抛出异常。如下所示：

```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

某些时候，想要访问没有被数据绑定的实例。可以提前向 Controller 注入此模型，然后通过第一种方式直接获取实例。或者，使用 `@ModelAttribute(binding=false)`，如下所示：

```
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 
    // ...
}
```

通过 `javax.validation.Valid` 注解或 Spring 的 `@Validated`，可以在数据绑定后进行数据校验，如下所示：

```
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

`@ModelAttribute` 是可选的。当参数不是简单类型，且没有被其它参数解析器处理，默认此参数标有 `@ModelAttribute`。所以，可以使用实体类型的参数直接获取名称匹配的表单字段，可能的过程是：使用默认构造器实例化 -> 将匹配的表单字段填充到模型属性 -> 将实例引用赋予参数。

### @SessionAttributes

`@SessionAttributes` 用于在请求间的 Servlet HTTP Session 中存储属性。它是类级别的。注解通常会列举需要存储在 Session 中的属性的名称和类型，以供后面的请求使用。使用 `value` 列举名称，`type` 列举类型。

下例示范使用 `@SessionAttributes`：

```
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

之后，在首次请求时，名为 `pet` 的属性会添加到 Model。同时，这个属性还会自动保存到 Session，它会一直存在，直到另个 Handler 方法使用 `SessionStatus` 方法参数清除存储：

```
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
        status.setComplete(); 
        // ...
    }
}
```

### @SessionAttribute

`@SessionAttribute` 可以将提前创建的全局管理的（控制器之外，如 `Filter`）Session 属性绑定到方法参数，这个 Session 属性可能不存在。如下所示：

```
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

如果要添加或删除 Session 属性，考虑使用 `org.springframework.web.context.request.WebRequest` 或 `javax.servlet.http.HttpSession` 类型的方法参数。

### @RequestAttribute

与 `@SessionAttribute` 类似，`@RequestAttribute` 用于访问提前创建的请求参数，比如 `Filter` 或 `HandlerInterceptor`，如下所示

```
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```

### Redirect Attributes

重定向是返回给客户端指示，让其重新访问某个指定的 URL，那么上次请求的有用信息将会丢失，`Redirect Attributes` 是解决此问题的一种方案。

Spring MVC 中，所有模型属性都能在重定向 URL 中做为 URI 变量。至于其余的属性，基本类型或基本类型的集合或数组，被自动追加为重定向 URL 的查询参数。

如果 Model 实例是专为重定向准备，将属性追加为查询参数是可行的方案。但有时 Model 中包含用于视图渲染的属性，为避免这些属性出现在重定向 URL，可在 `@RequestMapping` 方法中声明 `RedirectAttributes` 类型参数，用它指定哪些属性能被 `RedirectView` 访问。此时，如果发生重定向，则使用 `RedirectAttributes` 指定的内容，否则使用 Model 所有内容。

`RequestMappingHandlerAdapter` 提供一个名为 `ignoreDefaultModelOnRedirect` 的标记，可以指示 Handler 方法重定向时不使用 Model。MVC 的 Java 配置和 XML 名称空间默认将此设置为 `false`。

对于重定向 URL，默认可以访问当前请求的 URI 变量，不需要 Model 或 `RedirectAttributes` 特别设置。下例示范使用当前请求的 URI 变量来拓展重定向 URL：

```
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

另一种向重定向添加数据的方式是 `Flash Attributes`，它将数据存放在 Session，而非暴露在 URL。

### Flash Attributes

Flash 属性为请求提供另一种方式来保存数据以供另个请求使用。比如，在重定向之前临时保存 Flash 属性（通常是 Session），以便在重定向后给下个请求使用并立即删除。

Spring MVC 有两个抽象类用于支持 Flash 属性。`FlashMap` 负责保存 Flash 属性，`FlashMapManager` 负责储存、检索和管理 `FlashMap` 实例。Spring MVC 默认支持 Flash 属性，不需显式启用，但如果不使用，将永远不会导致 HTTP Session 的创建。每个请求都有一个 “input” `FlashMap`，其包含上个请求（如果有）传递的属性。和一个 “output” `FlashMap`，其包含要传给下个请求的属性。两个 `FlashMap` 实例可以在 Spring MVC 的任何位置通过 `RequestContextUtils` 访问。

Controller 通常不直接操作 `FlashMap`，而是使用 `@RequestMapping` 方法的 `RedirectAttributes` 类型参数添加 Flash 属性。`RedirectAttributes` 添加的 Flash 属性会自动同步到 “output” `FlashMap`，重定向后，“input” `FlashMap` 的属性会自动添加到目标 Controller 的 Model。

> **Matching requests to flash attributes**
>
> Flash 属性的概念在许多 web 框架中都有，并且已经证明其存在异步问题。根据定义，Flash 属性将会保存直到下个请求为止。然而，下个请求能是异步的，不会立即执行，此时 Flash 属性可能在异步请求执行前被删除。为此，`RedirectView` 使用重定向 URL 的路径和查询参数标记 `FlashMap`，`FlashMapManager` 会在请求进入时根据这些信息查找对应的 “input” `FlashMap`。这并不能完全消除并发问题，但已大大减少其发生的机会。

### Multipart

启用 `MultipartResolver` 后，将对媒体类型为 `multipart/form-data` 的 POST 请求的内容进行解析。包装请求后，可以像访问请求参数般访问被解析的内容。下例示范访问普通表单字段和上传文件：

```
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

将方法参数声明为 `List<MultipartFile>` 可用于解析参数名相同的多个文件。当 `@RequestParam` 参数被声明为 `Map<String, MultipartFile>` 或 `MultiValueMap<String, MultipartFile>` 类型，且注解未指定参数名，将自动填充请求中所有的文件。

使用 Servlet 3.0 Multipart Parsing 时，可能需要将 `MultipartFile` 替换为 `javax.servlet.http.Part` 类型。

请求中的 `multipart` 内容也能被数据绑定。下例请求与上例相同，`MyForm` 对象创建后，上传的文件会被绑定到 `MyForm` 的 `file` 属性：

```
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

对于 RESTFUL，非浏览器客户端也能提交 multipart 请求，下面是 Json 格式的文件：

```
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

对于上面的内容，可以使用 `@RequestParam` 直接以 `String` 方式访问 "meta-data" 部分的内容，但更多时候想获取它的反序列化结果。`@RequestPart` 可以将 multipart 内容被 `HttpMessageConverter` 转换后的结果绑定到 方法参数，如下所示：

```
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

`@RequestPart` 和 `javax.validation.Valid` 或 Spring 的 `@Validated` 结合，两者都能启用标准 Bean 校验。校验错误会抛出异常 `MethodArgumentNotValidException`，此错误会被转换为 400（BAD_REQUEST）响应。或者，可以使用 `Errors` 或 `BindingResult` 方法参数在本地处理异常，如下所示：

```
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```

### @RequestBody

`@RequestBody` 可以读取请求体内容，并通过 `HttpMessageConverter` 将其反序列化为指定的对象，如下所示：

```
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

可以使用 MVC Config 的 `Message Converters` 来配置或自定义消息转换。

`@RequestBody` 和 `javax.validation.Valid` 或 Spring 的 `@Validated` 结合，启用标准 Bean 校验。校验错误会抛出异常 `MethodArgumentNotValidException`，此错误会被转换为 400（BAD_REQUEST）响应。或者，可以使用 `Errors` 或 `BindingResult` 方法参数在本地捕获异常，如下所示：

```
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```

### HttpEntity

`HttpEntity` 或多或少与 `@RequestBody` 相似，但它是基于暴露请求头和请求体的容器对象。如下所示：

```
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

也就是说，它不但能访问请求体，还能访问请求头。

### @ResponseBody

`@ResponseBody` 标注方法，通过 `HttpMessageConverter` 将返回值序列化并输入到响应体。如下所示：

```
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

`@ResponseBody` 也可以标注类，此时它将被 Controller 的所有方法继承。`@RestController` 是 `@Controller` 和 `@ResponseBody` 的组合注解。

可以使用 MVC 配置的 `MessageConverters` 来配置或自定义消息转换。

可以将 `@ResponseBody` 与 Json 序列化视图结合使用，具体参看 Json 内容。

### ResponseEntity

`ResponseEntity` 类似于 `@ResponseBody`，但它还能设置响应码和响应头信息。如下所示：

```
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

### Json

目前主流的 Json 处理工具有三种：

- `Jackson`
- `Gson`
- `FastJson`

Spring MVC 对 `Jackson` 和 `Gson` 都提供相应的支持，如果使用它们作为 Json 转换器，只需添加依赖即可，返回的对象，集合，Map 等都会自动转为 Json 字符串。但如果使用 `FastJson`，除添加依赖外，还需要自己手动配置 `HttpMessageConverter` 转换器。前两个也是使用 `HttpMessageConverter` 转换器，但 Spring MVC 为它们提供自动注册的处理。

#### Jackson

Spring 提供对 `Jackson` 的支持。引入 `Jackson` 依赖：

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>version</version>
</dependency>
```

添加依赖后，Handler 直接返回的对象，集合等，都会自动转为 Json。如果与 `@ResponseBody` 合作，那么便会将 Json 格式的返回直接写入响应体。

添加 `Jackson` 后就能自动返回 Json，这其实依赖于 `HttpMessageConverter` 接口，它是 HTTP 消息转换器，既然是消息转换器，它提供两种功能：将 Handler 的返回对象转为 Json，将前端提交的 Json 转为对象。

`HttpMessageConverter` 只是接口，具体实现由各个 Json 工具提供。`Jackson` 提供的实现类类名是  `MappingJackson2HttpMessageConverter`。它的初始化由 Spring MVC 完成。除非有自定义配置的需求，否则不需要自己提供 `MappingJackson2HttpMessageConverter`。

#### Gson

`Gson` 是 Google 推出的 Json 解析器，在 Android 开发中使用较多。不过，web 开发也支持此工具，Spring MVC 还针对 `Gson` 提供相关的自动化配置。与 `Jackson` 相同，只需添加 `Gson` 依赖，就能直接使用 `Gson` 来解析 Json。

引入 `Gson` 依赖：

```
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>version</version>
</dependency>
```

如果项目中同时存在 `Jackson` 和 `Gson` 依赖，默认使用 `Jackson`。为什么呢？

在 `org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter` 的构造器中，加载顺序是先加载 `Jackson` 的 `HttpMessageConverter`，后加载 `Gson` 的 `HttpMessageConverter`。

添加依赖后，就能直接返回 Json。可以通过自定义 `HttpMessageConverter` 来自定义 `Gson` 配置。

#### Serialization View

Spring MVC 内置支持 `Jackson` 的序列化视图（Serialization View），这种技术允许只渲染对象的部分属性，用于序列化对象时排除密码等敏感信息。它与 `@ResponseBody` 或 `ResponseEntity` 合作使用，可以用 `Jackson` 的 `@JsonView` 激活序列化视图类。如下所示：

```
@RestController
public class UserController {

    @GetMapping("/user")
    // 激活不带密码的视图
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

	// 不带密码的视图
    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

	// 带密码的视图
    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

`@JsonView` 虽然可以声明多个视图类，但只能在 Controller 的每个方法上指定单个视图类。如果需要激活多个视图，考虑使用复合接口。

如果想用编程的方式实现，可以用 `MappingJacksonValue` 包装返回值并设置序列化视图。如下所示：

```
@RestController
public class UserController {

    @GetMapping("/user")
    public MappingJacksonValue getUser() {
        User user = new User("eric", "7!jd#h23");
        MappingJacksonValue value = new MappingJacksonValue(user);
        // 设置 serialization view
        value.setSerializationView(User.WithoutPasswordView.class);
        return value;
    }
}
```

对于依赖视图解析的 Controller，可以将序列化视图类添加到 Model，如下所示：

```
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

## Model

`@ModelAttribute` 有好几种使用方式：

* 标注 `@RequestMapping` 方法参数，创建或从 Model 中获取对象，并通过 `WebDataBinder` 将这个对象与请求绑定。
* 标注 `@Controller` 或 `@ControllerAdvice` 的方法，在 `@RequestMapping` 方法调用前初始化 Model。
* 标注 `@RequestMapping` 方法，标记此方法的返回值是 Model 属性。

前面已介绍第一种方式，现在介绍第二种。Controller 可以有任意个的 `@ModelAttribute` 方法，这些方法都会在 `@RequestMapping` 方法调用前执行。`@ModelAttribute` 方法还可以通过 `@ControllerAdvice` 在 Controller 间共享。

`@ModelAttribute` 方法具有灵活的方法签名。它支持大部分 `@RequestMapping` 的参数，除 `@ModelAttribute` 注解本身和与请求体相关的内容。

下面示范在 `@ModelAttribute` 方法中使用 `Model` 参数显式向 Model 添加属性。：

```
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

`@ModelAttribute` 方法的返回值也会自动添加为 Model 属性：

```
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

当没有显式指定 Model 属性名时，可以使用方法级别的 `@ModelAttribute` 的 name 属性指定，或者由 Spring MVC 根据类型赋予默认名称。

`@ModelAttribute` 还能标注 `@RequestMapping` 方法，方法的返回值会自动添加为 Model 属性。这通常没有必要，因为 HTML Controller 默认会这么做，除非返回值是字符串（解释为视图名）。下例使用 `@ModelAttribute` 显示指定 Model 属性名：

```
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

## DataBinder

`@Controller` 或 `@ControllerAdvice` 类可以声明 `@InitBinder` 方法，用于实例化 `WebDataBinder`。这些实例具有许多功能：

* 将请求参数（表单或查询参数）绑定到 Model 对象。
* 将基于 `String` 的请求值（请求参数，URI 变量，请求头，Cookie 等）转换为目标参数类型。
* 当渲染 HTML 表单时，将 Model 对象格式化为字符串。

`@InitBinder` 方法可以注册特定于 Controller 的 `java.beans.PropertyEditor` 或 Spring 的 `Converter` 和 `Formatter` 组件。此外，还能使用 MVC Config 向 `FormattingConversionService` 注册全局的 `Converter` 和 `Formatter` 组件。后者更常见。

`@InitBinder` 支持大部分 `@RequestMapping` 的参数，除 `@ModelAttribute` 参数。通常只声明 `WebDataBinder` 参数（数据绑定），并返回 `void`。如下所示：

```
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

还可以在 `initBinder()` 中向 `WebDataBinder` 注册 `Formatter`：

```
@Controller
public class FormController {

    @InitBinder 
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

## Exceptions

`@Controller` 或 `@ControllerAdvice` 类可以有 `@ExceptionHandler` 方法，用于处理特定的 Controller 抛出的异常。如下所示：

```
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

异常处理方法可以匹配传播的异常类型，如抛出的 `IOException`。也可以匹配被包装的 Cause 异常，如包装在 `IllegalStateException` 中的 `IOException`。从 5.3 开始，异常处理方法便能匹配任何层级的异常，而之前只能匹配直接抛出的异常。

对于异常类型的匹配，可以直接用方法参数声明目标异常，如上面所示。当匹配多个异常时，通常 Root 异常优先于 Cause 异常。

或者，可以在 `@ExceptionHandler` 中声明匹配的异常类型，如下所示：

```
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(IOException ex) {
    // ...
}
```

还能使用多个具体异常限制匹配类型，而在方法中声明通用的异常类型参数。如下所示，这种方式常用：

```
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(Exception ex) {
    // ...
}
```

Root 异常与 Cause 异常在类型匹配的区别：对于上面的示例，`@ExceptionHandler` 方法传入的异常类型被限制为 `FileSystemException` 或 `RemoteException`。然而，如果某个匹配的异常实例被包装在 `Exception` 实例中，那么传入方法的实例将会是这个包装异常，即 Root 异常。此时只能通过 `ex.getCause()` 获取 Cause 异常。只有当 `FileSystemException` 或 `RemoteException` 作为顶级异常（Root 异常）时，传入的实例才会是它们。

多个 `@ControllerAdvice` 时，建议使用 `order` 进行排序并声明 Root  异常。虽然 Root 异常比 Cause 异常优先级高，但这只是对于定义在 controller 或 `@ControllerAdvice` 类中的异常处理方法。这意味高优先级 `@ControllerAdvice` 中的 cause 异常匹配比优先级低的 `@ControllerAdvice` 的任何异常匹配的优先级都要高。

最后，`@ExceptionHandler` 方法还可以将捕获的异常重新抛出来，这个异常将继续在后面的异常处理链中传播，就好像从未被匹配到过一样。

Spring MVC 对 `@ExceptionHandler` 的支持建立在 `DispatcherServlet` 的 `HandlerExceptionResolver` 机制。

`@ExceptionHandler` 方法支持以下参数类型：

| Method argument                                              | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Exception type                                               | For access to the raised exception.                          |
| `HandlerMethod`                                              | For access to the controller method that raised the exception. |
| `WebRequest`, `NativeWebRequest`                             | Generic access to request parameters and request and session attributes without direct use of the Servlet API. |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | Choose any specific request or response type (for example, `ServletRequest` or `HttpServletRequest` or Spring’s `MultipartRequest` or `MultipartHttpServletRequest`). |
| `javax.servlet.http.HttpSession`                             | Enforces the presence of a session. As a consequence, such an argument is never `null`. Note that session access is not thread-safe. Consider setting the `RequestMappingHandlerAdapter` instance’s `synchronizeOnSession` flag to `true` if multiple requests are allowed to access a session concurrently. |
| `java.security.Principal`                                    | Currently authenticated user — possibly a specific `Principal` implementation class if known. |
| `HttpMethod`                                                 | The HTTP method of the request.                              |
| `java.util.Locale`                                           | The current request locale, determined by the most specific `LocaleResolver` available — in effect, the configured `LocaleResolver` or `LocaleContextResolver`. |
| `java.util.TimeZone`, `java.time.ZoneId`                     | The time zone associated with the current request, as determined by a `LocaleContextResolver`. |
| `java.io.OutputStream`, `java.io.Writer`                     | For access to the raw response body, as exposed by the Servlet API. |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access to the model for an error response. Always empty. |
| `RedirectAttributes`                                         | Specify attributes to use in case of a redirect — (that is to be appended to the query string) and flash attributes to be stored temporarily until the request after the redirect. |
| `@SessionAttribute`                                          | For access to any session attribute, in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. |
| `@RequestAttribute`                                          | For access to request attributes.                            |

`@ExceptionHandler` 方法支持以下返回类型：

| Return value                                    | Description                                                  |
| :---------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                 | The return value is converted through `HttpMessageConverter` instances and written to the response. |
| `HttpEntity<B>`, `ResponseEntity<B>`            | The return value specifies that the full response (including the HTTP headers and the body) be converted through `HttpMessageConverter` instances and written to the response. |
| `String`                                        | A view name to be resolved with `ViewResolver` implementations and used together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument (described earlier). |
| `View`                                          | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (descried earlier). |
| `java.util.Map`, `org.springframework.ui.Model` | Attributes to be added to the implicit model with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `@ModelAttribute`                               | An attribute to be added to the model with the view name implicitly determined through a `RequestToViewNameTranslator`.Note that `@ModelAttribute` is optional. See “Any other return value” at the end of this table. |
| `ModelAndView` object                           | The view and model attributes to use and, optionally, a response status. |
| `void`                                          | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse` an `OutputStream` argument, or a `@ResponseStatus` annotation. The same is also true if the controller has made a positive `ETag` or `lastModified` timestamp check .If none of the above is true, a `void` return type can also indicate “no response body” for REST controllers or default view name selection for HTML controllers. |
| Any other return value                          | If a return value is not matched to any of the above and is not a simple type, by default, it is treated as a model attribute to be added to the model. If it is a simple type, it remains unresolved. |

> **REST API Exceptions**
>
> REST 风格的服务要求在响应正文中包含错误详细信息。Spring 不会自动执行此操作，因为错误信息特定于应用。不过，可以在 `@RestController` 的 `@ExceptionHandler` 方法中返回 `ResponseEntity` 类型结果来设置响应码和响应体内容。这也能应用于 `@ControllerAdvice`。

##  Controller Advice

前面介绍的 `@ExceptionHandler`，`@InitBinder` 和 `@ModelAttribute` 方法仅适用于特定的 Controller。但如果将它们声明在 `@ControllerAdvice` 或 `@RestControllerAdvice` 中，它们将适用于所有 Controller。除此之外，从 5.3 开始，`@ControllerAdvice` 中的 `@ExceptionHandler` 方法可处理任何 Controller 和任何 Handler 抛出的异常。

`@ControllerAdvice` 被 `@Component` 标注，所以它能被 Spring 自动扫描和注册。`@RestControllerAdvice` 被 `@ControllerAdvice` 和 `@ResponseBody` 标注，它所有的 `@ExceptionHandler` 方法的返回值都将通过消息转换器渲染而非 HTML 视图。

Spring MVC 启动时，`RequestMappingHandlerMapping` 和 `ExceptionHandlerExceptionResolver` 检测 `@ControllerAdvice` 组件并在运行时应用它们。`@ControllerAdvice` 的 `@ExceptionHandler` 方法在 Controller 本地方法执行后调用，它的 `@ModelAttribute` 和 `@InitBinder` 方法在本地方法之前调用。

`@ControllerAdvice` 具有一些属性可用于缩小其能应用的 Controller 和 Handler 的范围，如下所示：

```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

上面范围的选择是在运行时解析，如果应用太广泛可能会对性能有负面影响。

# CORS

## 同源策略

浏览器为安全考虑，在全局禁止页面加载或执行与自身来源不同的域的任何脚本。同源指协议、域名、端口三者都相同。安全问题？比如，某个打开的网页使用浏览器的凭证去访问其它网页的资源，可能是银行账户等。凡 URL 的协议、域名、端口任一与当前页面地址不同的请求即为跨域请求。

## CROS

Cross-Origin Resource Sharing，CORS 是一项 W3C 规范，用于允许浏览器发出跨域请求。它的实现需要浏览器和服务端同时支持，目前大部分浏览器都已实现 CORS。

整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但这对用户都是透明的。

因此，实现 CORS 通信的关键是服务端。只要服务端实现 CORS 接口，就可以跨域通信。

Spring MVC 支持 CORS。

### 两种请求

浏览器将 CROS 的请求分为两类：简单请求和非简单请求。

同时满足以下两种条件的请求是简单请求：

* 请求方法是 HEAD 或 GET 或 POST。
* HTTP 的头信息不超出以下几种字段：
  * `Accept`
  * `Accept-Language`
  * `Content-Language`
  * `Last-Event-ID`
  * `Content-Type` 限于 3 种值：`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

这是为兼容表单，因为历史上表单一直可以发出跨域请求。凡是不同时满足上面两个条件，就属于非简单请求。

浏览器对这两种请求的处理，是不一样的。

### 简单请求

对于简单请求，浏览器直接发出 CORS 请求，具体就是在头信息中增加一个 `Origin` 字段。`Origin` 表示本次请求来自哪个源（协议 + 域名 + 端口）。服务端根据这个值，决定是否同意这次请求。

如果 `Origin` 指定的源，不在允许范围内，服务端会返回一个正常的 HTTP 回应。浏览器发现这个回应的头信息没有包含 `Access-Control-Allow-Origin` 字段，就知道该请求不被允许，抛出一个错误。这种错误无法通过状态码识别，因为 HTTP 回应的状态码可能是 200。

如果 `Origin` 指定的域名在允许范围内，服务器返回的响应，会多出几个头信息字段：

* `Access-Control-Allow-Origin`：必须存在，它的值要么是请求源，要么是 `*`，表示允许任何源的请求。
* `Access-Control-Allow-Credentials`：可选，它是布尔值，表示 CORS 请求是否允许包含 Cookie。
* `Access-Control-Expose-Headers`：可选，指定 `XMLHttpRequest` 对象可以获取的额外头信息字段。

需要注意，如果要传送 Cookie，`Access-Control-Allow-Origin` 就不能使用 `*`，必须明确设置一个或多个域，否则浏览器很可能不会发送 Cookie。

### 非简单请求

非简单请求是那些对服务端有特殊要求的请求，比如请求方法是 `PUT` 或 `DELETE`，或者 `Content-Type` 字段的类型是 `application/json`。

**预检请求**

非简单请求的 CORS 请求，会在正式通信之前，增加一次 HTTP 查询请求，即预检请求。

浏览器先询问服务器，当前网页所在的域是否在服务器的允许名单中，以及可以使用哪些 HTTP 方法和头信息字段。只有得到肯定答复，浏览器才会发出正式的请求，否则就报错。

预检请求的请求方法是 `OPTIONS`，表示这个请求用于询问。头信息里面，关键字段是 `Origin`，表示请求来自哪个源。除 `Origin` 字段，预检请求的头信息还包含两个特殊字段：

* `Access-Control-Request-Method`：必须存在，列出浏览器的 CORS 请求会用到哪些 HTTP 方法。
* `Access-Control-Request-Headers`：值是一个 `,` 分隔的字符串，列出浏览器 CORS 请求会额外发送的头信息字段。

服务端收到预检请求以后，会检查 `Origin`、`Access-Control-Request-Method` 和 `Access-Control-Request-Headers` 字段，确认是否允许跨域请求，再做出回应。

如果允许跨域，则响应中会包含前面提到的 `Access-Control-Allow-Origin` 字段。如果服务端否定预检请求，也会返回一个正常的 HTTP 响应，但不包含任何与 CROS 相关的头信息字段，这时浏览器就认定服务端不支持发送这个跨域请求，报错。

服务端响应包含的其它 CROS 相关的字段：

* `Access-Control-Allow-Methods`：必须，使用 `,` 分隔的字符串，表示允许跨域请求的 HTTP 方法。
* `Access-Control-Allow-Headers`：如果浏览器请求中包含这个字段，那响应也必须有这个字段。它也是使用 `,` 分隔的字符串，表示服务端支持的所有头信息字段。
* `Access-Control-Max-Age`：可选，指定本次预检请求的有效期，单位为秒。
* `Access-Control-Allow-Credentials`：与前面的定义相同。

**真实请求**

在第一次预检请求通过之后，浏览器后续的真实请求都与普通请求相同，有一个 `Origin` 头信息字段，响应也会有一个 `Access-Control-Allow-Origin` 头信息字段。

## Spring MVC 的处理过程

Spring MVC 的 `HandlerMapping` 实现为 CORS 提供内置支持。当成功将请求映射到 Handler 后，`HandlerMapping` 会检查当前请求和 Handler 关于 CORS 的设置，并采取进一步措施。预检请求会被直接处理，而简单请求和真实请求会被拦截、验证、设置需要的 CROS 响应头信息。

要启用跨域请求，需要显式进行一些设置。如果未找到匹配的 CORS 设置，预检请求将被拒绝，简单请求和真实请求的响应头不会添加 CORS，所以浏览器会拒绝它们。

每个 `HandlerMapping` 都能使用基于 URL 的 `CorsConfiguration` 映射单独进行配置。大部分情况，应用程序使用 Java 或 XML 配置来声明此映射，这会将单个映射传递到所有 `HandlerMapping` 实例。

可以将 `HandlerMapping` 级别的全局 CORS 配置与 Handler 级别的局部 CORS 配置结合。比如，Controller 可以使用类级别或方法级别的 `@CrossOrigin` 注解（或实现 `CorsConfigurationSource`）。全局与局部结合的规则是累加的，但对于只接收单个值的属性，局部配置将覆盖全局配置。

如果想了解更多信息或进行高级的自定义，查看源码 `AbstractHandlerMapping`，`DefaultCorsProcessor`，`CorsConfiguration`，`CorsProcessor`。

## 局部配置 @CrossOrigin

`@CrossOrigin` 用于启用局部的 Controller 或 Handler 的跨域请求：

```
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

`@CrossOrigin` 默认允许：

* 所有域
* 所有头信息
* 所有 HTTP 方法

`maxAge` 被设为 30 分钟。

`allowCredentials` 默认不启用，因为它会建立信任级别并暴露用户的敏感信息，如 Cookie，只能在适当的时候开启。如果启用，必须将 `allowOrigins` 设置为一个或多个特定的域，或者使用 `allowOriginPatterns` 动态设置。

`@CrossOrigin` 可用于类级别或方法级别。

```
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com")
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

## 全局配置

除配置特定的 Controller 或 Handler 外，还能定义全局的 CORS 配置。任何 `HandlerMapping` 都能单独设置基于 URL 的 `CorsConfiguration` 映射。可以使用 Java 或 XML 实现此操作。

全局配置默认允许：

* 所有域
* 所有头信息
* `GET`，`HEAD` 和 `POST` 方法

`maxAge` 被设为 30 分钟。

`allowCredentials` 默认不启用，如果启用，必须将 `allowOrigins` 设置为一个或多个特定的域，或者使用 `allowOriginPatterns` 动态设置。

### Java 配置类

通过 `CorsRegistry` 配置全局 CORS：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

### XML 配置文件

使用 `<mvc:cors>` 标签配置全局 CORS：

```
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="https://domain1.com, https://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="true"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="https://domain1.com" />

</mvc:cors>
```

## CORS 过滤器

可以通过内置的 `CorsFilter` 启用 CORS。如果将 `CorsFilter` 与 Spring Security 结合，需要知道 Spring Security 也内置对 CORS 的支持。

要配置 Filter，需要把 `CorsConfigurationSource` 传递给它的构造器，如下所示：

```
CorsConfiguration config = new CorsConfiguration();

// Possibly...
// config.applyPermitDefaultValues()

config.setAllowCredentials(true);
config.addAllowedOrigin("https://domain1.com");
config.addAllowedHeader("*");
config.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);

CorsFilter filter = new CorsFilter(source);
```

# MVC 配置

MVC 的 Java 配置和 XML 配置提供适用于大部分应用的默认配置，以及自定义这些配置的 API。对于不能通过 API 修改的自定义项，可以查看 Advanced Java Config 和 Advanced XML Config。如果想要了解 XML 或 Java 配置创建的 Bean，可以查看 Special Bean 和 Web MVC Config 节。

## 启用 MVC

Java 配置，使用 `@EnableWebMvc` 启用 MVC：

```
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

XML 配置，使用 `<mvc:annotation-driven>` 标签启用 MVC：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

启用 MVC 后会向 MVC 容器注册许多 Spring MVC 基础 Bean 并扫描类路径上可获取的依赖项。

## MVC 配置 API

Java 配置，实现 `WebMvcConfigurer` 接口，其包含许多自定义 MVC 配置的方法，如下所示：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

XML 配置，可以使用 `<mvc:annotation-driven/>` 的属性和子元素自定义 MVC，具体内容查看 `Spring MVC XML schema`。

## 类型转换 Conversion

MVC 默认会注册各种数字和日期类型的 `Formatter`，并支持通过标注字段的 `@NumberFormat` 和 `@DateTimeFormat` 注解进行自定义格式。

使用 Java 配置注册自定义的 `Formatter` 和 `Converter`：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

默认情况，Spring MVC 在解析和格式化日期值时会考虑请求的 Locale。这适用于日期表现为基于字符串的表单字段的情况。然而，对于日期和时间字段，浏览器可能会使用自定义的格式。对于这种情况，可以按如下方式添加匹配的格式化器：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

## addArgumentResolvers



## 数据校验 Validation

默认情况，如果类路径上存在 `Bean Validation`，则 `LocalValidatorFactoryBean` 将自动注册其为全局 `Validator`，以便在 Controller 方法参数上使用 `@Valid` 和 `@Validated`。

Java 配置，注册自定义的全局 `Validator`：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

还可以在 Controller 本地注册 `Validator` 实例，如下所示：

```
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}
```

如果需要在某处注入 `LocalValidatorFactoryBean`，可以创建这个类型的 Bean 并标注 `@Primary`，以避免与 MVC 配置的 Bean 发生冲突。

##  拦截器 Interceptors

Java 配置，注册自定义的拦截器：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

## 内容类型 Content Types

可以配置 Spring MVC 如何从请求中确定其接受的媒体类型，比如 `Accept` 请求头，URL 拓展符，请求参数等。默认只会检查 `Accept` 请求头。

如果想使用基于 URL 的接受内容媒体类型解析方案，比如请求参数的拓展符。有关信息查阅 Suffix Match 和 Suffix Match and RFD。

Java 配置，可以按如下方式自定义接受内容媒体类型的解析方式：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

## 消息转换器 Message Converters

Java 配置，自定义 `HttpMessageConverter` 有两种方式：重写 `configureMessageConverters()` 方法，覆盖 Spring MVC 默认创建的 `Converters`；重写 `extendMessageConverters()` 方法，自定义默认的 `Converters`或添加额外的 `Converters`。

下例使用自定义的 `ObjectMapper` 添加 XML 和 Jackson Json 的 `Converters` 并覆盖默认配置：

```
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

上面示例，使用 `Jackson2ObjectMapperBuilder` 为 `MappingJackson2HttpMessageConverter` 和 `MappingJackson2XmlHttpMessageConverter` 创建公共的配置，比如开启缩进，自定义日期格式和注册 `jackson-module-parameter-names`（用于支持访问参数的名称，Java 8 新增）。

builder 自定义 Jackson 的默认属性如下：

* 禁用 `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`
* 禁用 `MapperFeature.DEFAULT_VIEW_INCLUSION`

如果在类路径上检测到以下模块，也会进行自动注册：

* `jackson-datatype-joda`：支持 Joda-Time 类型；
* `jackson-datatype-jsr310`：支持 Java 8 `Date` 和 `Time` 类型；
* `jackson-datatype-jdk8`：支持其它 Java 8 类型，如 `Optional`；
* `jackson-module-kotlin`：支持 Kotlin 类和 data 类。

## View Controllers

This is a shortcut for defining a `ParameterizableViewController` that immediately forwards to a view when invoked. You can use it in static cases when there is no Java controller logic to run before the view generates the response.

The following example of Java configuration forwards a request for `/` to a view called `home`:

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

The following example achieves the same thing as the preceding example, but with XML, by using the `<mvc:view-controller>` element:

```
<mvc:view-controller path="/" view-name="home"/>
```

If an `@RequestMapping` method is mapped to a URL for any HTTP method then a view controller cannot be used to handle the same URL. This is because a match by URL to an annotated controller is considered a strong enough indication of endpoint ownership so that a 405 (METHOD_NOT_ALLOWED), a 415 (UNSUPPORTED_MEDIA_TYPE), or similar response can be sent to the client to help with debugging. For this reason it is recommended to avoid splitting URL handling across an annotated controller and a view controller.

## View Resolvers

The MVC configuration simplifies the registration of view resolvers.

The following Java configuration example configures content negotiation view resolution by using JSP and Jackson as a default `View` for Json rendering:

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

The following example shows how to achieve the same configuration in XML:

```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

Note, however, that FreeMarker, Tiles, Groovy Markup, and script templates also require configuration of the underlying view technology.

The MVC namespace provides dedicated elements. The following example works with FreeMarker:

```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

In Java configuration, you can add the respective `Configurer` Bean, as the following example shows:

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```

## 默认 Servlet 处理静态资源请求

Spring MVC 允许将 `DispatcherServlet` 映射到 `/`，这会覆盖 Servlet 容器的 `Default Servlet` 映射，不过仍然有办法使用 `Default Servlet` 来处理静态资源的请求。

可以通过内置的 `DefaultServletHttpRequestHandler` 解决，它是映射 `/**` 的 `HandlerMapping`，会把接收到的所有请求转发到 `Default Servlet`。其 `order` 值为 `Integer.MAX_VALUE`，相对于其它映射器，它的优先级最低。此时访问静态资源，前面的映射器如果都不能匹配，最终将由 `Default Servlet` 来处理。

`DefaultServletHttpRequestHandler` 默认不启用，下例使用 Java 配置开启此映射器：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

同样功能的 XML 配置内容如下：

```
<mvc:default-servlet-handler/>
```

需要注意的是，不同的 Servlet 容器，其 `Default Servlet` 的名称可能不同，而对于 `Default Servlet` 的请求转发是根据名称进行的。`DefaultServletHttpRequestHandler` 在 Servlet 容器启动时，会使用大部分容器的已知名称列表来检测 `Default Servlet`。如果 `Default Servlet` 自定义为其它名称，或者其名称未知，则需要显式提供 `Default Servlet` 的名称，如下所示：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```

同样功能的 XML 配置内容如下：

```
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```

## 路径匹配

可以自定义路径匹配和 URL 处理相关的选项，具体信息参考 `PathMatchConfigurer` 的文档 。

下例示范如何在 Java 配置自定义路径匹配：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setPatternParser(new PathPatternParser())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    private PathPatternParser patternParser() {
        // ...
    }
}
```

## 高级设置

`@EnableWebMvc` 导入 `DelegatingWebMvcConfiguration`，它的作用：

* 为 Spring MVC 应用提供默认的 Spring 配置
* 检测并委托给 `WebMvcConfigurer` 实现以自定义该配置

对于更高级的配置，可以将 `@EnableWebMvc` 删除，直接从 `DelegatingWebMvcConfiguration` 进行拓展，而非继承 `WebMvcConfigurer`，如下所示：

```
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...
}
```

现在，既能保留 `WebConfig` 的现有方法，还能覆盖基类中的 Bean 声明，而且仍然能在类路径中拥有任意数量的其它 `WebMvcConfigurer` 实现。

MVC 名称空间没有高级模式。如果需要自定义 Bean 中无法修改的属性，可以使用 `ApplicationContext` 中的 `BeanPostProcessor` 生命周期勾子函数，如下所示：

```
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```
