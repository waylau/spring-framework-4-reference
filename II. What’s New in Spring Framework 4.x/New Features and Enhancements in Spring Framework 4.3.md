Spring 4.3 的新功能和增强
====

## 核心容器改进

* 核心容器额外提供了更丰富的元数据来改进编程。
* 默认 Java 8 的方法检测为 bean 属性的 getter/setter 方法。
* 如果目标 bean 只定义了一个构造函数,则它无需要指定`@Autowired`注解
* `@Configuration`类支持构造函数注入。
* 任何 SpEL 表达式用于指定`@EventListener`的 `condition` 引用到  bean（例如`@beanName.method()`）。
* 组成注解现在可以用一个包含元注解中的数组属性的数组组件类型的元素来覆盖。例如，`@RequestMapping`的的`String[] path` 可以在组成注解用 `String path` 覆盖。
* `@Scheduled`和`@Schedules`现在是作为元注解用来通过属性覆盖来创建自定义的组成注解。
* `@Scheduled`适当支持任何范围内的 bean。

## 数据访问改进

`jdbc:initialize-database` 和 `jdbc:embedded-database` 支持可配置的分离器被应用到每个脚本。

## 缓存改进

Spring 4.3 允许在一个给定的 key 并发调用时实现要同步，使得相应的值只计算一次。这是一个可选的功能，通过设置`@Cacheable`的新的 `sync` 属性来启用。此功能引入了`Cache`接口的一个重大更改，即`get(Object key, Callable<T> valueLoader)`方法已添加。

Spring 4.3 还改进了缓存抽象如下：

* SpEL 表达式对于缓存相关的注解，现在可以引用 bean（即`@beanName.method())`）。
* `ConcurrentMapCacheManager`和`ConcurrentMapCache`现在通过一个新的`storeByValue`属性支持缓存实体的序列化。
`@Cacheable`，`@CacheEvict`，`@CachePut`和`@Caching`现在是作为元注解用来通过属性覆盖来创建自定义的组成注解。

## JMS 改进

* `@SendTo`现在可以在类级别指定一个共同回复目标。
* `@JmsListener` 和 `@JmsListeners`现在是作为元注解用来通过属性覆盖来创建自定义的组成注解。

## Web 改进

* 内建支持 [HTTP HEAD 和 HTTP OPTIONS.](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-requestmapping-head-options)
* 新的组合注解 `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, 和 `@PatchMapping` 用于 `@RequestMapping`。
    * 详见 [`@RequestMapping` 组合变种](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-requestmapping-composed)
* 新的`@RequestScope`, `@SessionScope`, 和 `@ApplicationScope`用于 web 范围的组合注解
    *  [Request scope](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-scopes-request), [Session scope](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-scopes-session), 和 [Application scope](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-scopes-application) 
* 新的 `@RestControllerAdvice` 注解是 `@ControllerAdvice` 和 `@ResponseBody` 的语义结合
* `@ResponseStatus`现在在类级别被支持，并被所有方法继承
* 新的 `@SessionAttribute` 注解用于访问 session 属性 (见[例子](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-sessionattrib-global))
* 新的 `@RequestAttribute` 注解用于访问请求属性 (见[例子](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-requestattrib))
* `@ModelAttribute` 允许通过 `binding=false` 来避免数据绑定(见引用)
* 错误和自定义抛出，将被统一到 MVC 异常处理器中处理
* HTTP 消息转换编码一致处理，包括默认 UTF-8 用于多部分文本内容
* 静态资源处理使用配置的`ContentNegotiationManager`用于媒体类型计算
* `RestTemplate` 和 `AsyncRestTemplate` 支持通过`DefaultUriTemplateHandler` 来实现严格的URI变量编码
* `AsyncRestTemplate`支持请求拦截

## WebSocket 消息改进

`@SendTo`和`@SendToUser`现在可以在类级被指定为共享共同的目的地。

## 测试改进

* 为了支持 Spring TestContext Framework ，现在需要 JUnit 4.12  或者更高的版本
* 新的`SpringRunner` 关联于 `SpringJUnit4ClassRunner`
* 测试相关的注解，现在可以在接口上声明了。例如，基于 Java 8 的接口上使用测试接口
* 空声明的  `@ContextConfiguration` 现在将会完全忽略，如果检测到默认的 XML 文件, Groovy 脚本, 或 `@Configuration` 类型
* `@Transactional` 测试方法不再需要`public` (如, 在 TestNG 和 JUnit 5)
* `@BeforeTransaction` 和 `@AfterTransaction`不再需要`public`，并且在 基于 Java 8 的接口的默认方法上声明
* 在Spring TestContext Framework 的`ApplicationContext`的缓存现在有界为32默认最大规模和最近最少使用驱逐策略。最大的大小可以通过设置称为`spring.test.context.cache.maxSize` 一个 JVM 系统属性或 Spring 配置。
* `ContextCustomizer` API 用于自定义测试 `ApplicationContext` 在 bean 定义加载到上下文后但在上下文被刷新前。定制工具可以在全球范围由第三方进行注册，而无需要实现一个自定义的 `ContextLoader`。
* `@Sql` 和 `@SqlGroup` 现在作为元注解通过覆盖属性来创建自定义组合注解
* `ReflectionTestUtils`现在在 set 或 get 一个字段时，会自动解开代理。
* 服务器端的 Spring MVC 测试支持具有多个值的响应头。
* 服务器端的 Spring MVC 测试解析表单数据的请求内容和填充请求参数。
* 服务器端的 Spring MVC 测试支持 mock 式的断言来调用处理程序方法。
* 客户端 REST 测试支持允许指定多少次预期的请求以及期望的声明顺序是否应该被忽略（参见[15.6.3，“客户端REST测试”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#spring-mvc-test-client)）。
* 客户端 REST 测试支持请求主体表单数据的预期。

## 支持新的类库和服务器

* Hibernate ORM 5.2 (同样很好的支持 4.2/4.3 和 5.0/5.1，不推荐  3.6 )
* Jackson 2.8 (在Spring 4.3，最低至 Jackson 2.6+ )
* OkHttp 3.x (仍然并行支持 OkHttp 2.x)
* Netty 4.1
* Undertow 1.4
* Tomcat 8.5.2 以及 9.0 M6