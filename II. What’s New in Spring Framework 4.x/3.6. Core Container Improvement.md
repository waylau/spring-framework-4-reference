核心容器改进
=============

有几处对核心容器的常规改进：

* Spring 现在注入 Bean 的时候把 [*泛型类型当成一种形式的限定符*](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-generics-as-qualifiers)。例如：如果你使用S pring Data `Repository` 你可以方便的插入特定的实现：`@Autowired Repository<Customer> customerRepository`。
* 如果你使用 Spring 的元注解支持，你现在可以[开发自定义注解来公开源注解的特定属性](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#beans-meta-annotations)。
* 当[自动装配到lists和arrays时](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#beans-autowired-annotation)，Beans 现在可以进行排序 了。支持`@Order`注解和`Ordered`接口两种方式。
`@Lazy`注解现在可以用在注入点以及`@Bean`定义上。
* [引入`@Description`注解](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#beans-java-bean-description),开发人员可以使用基于Java 方式的配置。
* 根据条件[筛选 Beans](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#beans-java-conditional)的广义模型通过`@Conditional`注解加入。这和`@Profile`支持的类似，但是允许以编程式开发用户定义的策略。
* [基于CGLIB的代理类](http://docs.spring.io/spring/docs/4.2.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#aop-pfb-proxy-types)不在需要默认的构造方法。这个支持是由 [objenesis](http://code.google.com/p/objenesis/)库提供。这个库重新打包到 Spring 框架中，作为Spring框架的一部分发布。通过这个策略，针对代理实例被调用没有构造可言了。
* 框架现在支持管理时区。例如`LocaleContext`。
