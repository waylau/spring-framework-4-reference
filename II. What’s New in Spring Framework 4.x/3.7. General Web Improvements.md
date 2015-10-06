常规Web改进
=========

现在仍然可以部署到 Servlet 2.5 服务器，但是 Spring4.0 现在主要集中在 Servlet 3.0+ 环境。如果你使用 [Spring MVC 测试框架](http://spring.oschina.mopaas.com/aop-api.html#spring-mvc-test-framework)，你需要将 Servlet 3.0 兼容的 JAR 包放到 测试的 *classpath* 下。

除了稍后会提到的 WebSocket 支持外，下面的常规改进已经加入到Spring 的 Web 模块：

* 你可以在Spring MVC应用中使用[新的](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-ann-restcontroller)`@RestController`[注解](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-ann-restcontroller)，不在需要给`@RequestMapping`的方法添加`@ResponseBody`注解。
* AsyncRestTemplate 类已被添加进来，当开发 REST 客户端时，[允许非阻塞异步支持](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#rest-async-resttemplate)。
* 当开发 Spring MVC 应用时，Spring现在提供了[全面的时区支持](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#mvc-timezone) 。
