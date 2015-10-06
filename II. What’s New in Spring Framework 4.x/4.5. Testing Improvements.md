测试 改进
========

* Groovy脚本现在能使用`ApplicationContext`配置，在TestContext框架中整合测试。[the section called “Context configuration with Groovy scripts”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-ctx-management-groovy)
* 现在通过新的`TestTransaction`接口，使用编程化来开始结束测试管理事务。[the section called “Programmatic transaction management”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-tx-programmatic-tx-mgt)
* 现在SQL脚本的执行可以通过`Sql`和`SqlConfig`注解申明在每一个类和方法中。[the section called “Executing SQL scripts”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-executing-sql)
* 测试属性值可以通过配置`@TestPropertySource`注解来自动覆盖系统和应用的属性值。[the section called “Context configuration with test property sources”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-ctx-management-property-sources)。
* 默认的`TestExecutionListeners`现在可以自动被发现。[the section called “Automatic discovery of default TestExecutionListeners” ](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-tel-config-automatic-discovery)。
* 自定义的`TestExecutionListeners`现在可以通过默认的监听器自动合并。[the section called “Merging TestExecutionListeners”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-tel-config-merging)。 
* 在TestContext框架中的测试事务方面的文档已经通过更多解释和其他事例得到改善。[the section called “Transaction management”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-tx)。 
* `MockServletContext`、`MockHttpServletRequest`和其他servlet接口mocks等诸多改善。
* `AssertThrows`重构后，`Throwable`代替`Exception`。
* Spring MVC测试中，使用JSONPath选项返回JSON格式可以使用[JSON Assert](https://github.com/skyscreamer/JSONassert)来断言,非常像之前的XML和XMLUnit。
* `MockMvcBuilder`现在可以在`MockMvcConfigurer`的帮助下创建。`MockMvcConfigurer`的加入使得Spring Security的设置更加简单，同时使用于任何第三方的普通封装设置或则仅仅在本项目中。
* `MockRestServiceServer`现在支持`AsyncRestTemplate`作为客户端测试。



* 发布新的应用上下文实践`SessionSubscribeEvent`和
`SessionUnsubscribeEvent`,用于STOMP客户端的订阅和取消订阅.
* 新的"websocket"级别.查看[Section 21.4.13, “WebSocket Scope”.](http://docs.spring.io/spring/docs/current/spring-framework-
