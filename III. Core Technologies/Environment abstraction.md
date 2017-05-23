## 环境的抽象

`Environment`是一个集成到容器之中的特殊抽象，它针对应用的环境建立了两个关键的概念：`profile`和`properties`.

*profile*是命名好的，其中包含了多个Bean的定义的一个逻辑集合，只有当指定的profile被激活的时候，其中的Bean才会激活。无论是通过XML定义的还是通过注解解析的Bean都可以配置到profile之中。而`Environment`对象的角色就是跟profile相关联，然后决定来激活哪一个profile，还有哪一个profile为默认的profile。

*properties*在几乎所有的应用当中都有着重要的作用，当然也可能存在多个数据源：property文件，JVM系统property，系统环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，Map等。`Environment`对象和property相关联，然后来给开发者一个方便的服务接口来配置这些数据源，并正确解析。

### Bean定义的profile

在容器之中，Bean定义profile是一种允许不同环境注册不同bean的机制。环境的概念就意味着不同的Bean对应不同的开发者，而且这个特性在以下场景使用十分便利：

* 解决一些内存中的数据源的问题，可以在不同环境访问不同的数据源，开发环境，QA测试环境，生产环境等。
* 仅仅在开发环境来使用一些监视服务
* 在不同的环境，使用不同的bean实现

下面参考一个例子，下面的应用需要一个`DataSource`，在一个测试的环境下，可能类似如下代码：

```
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在考虑如果应用部署到QA环境或者生产环境，假设应用的数据源是服务器上的JNDI目录的话，我们的`DataSource`可能会如下：

```
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题就是如何基于当前的环境来使用不同的配置。在以前，Spring的开发者开发了很多的方法来解决这个问题，通常都依赖于系统环境变量和XML中的`<import/>`标签以及占位符`${placeholder}`等来根据不同的环境解析当前的配置文件。现在Bean的profile属于容器的特性，也是该问题的解决方案之一。

如果我们泛化了我们一些特殊环境下引用的bean定义，我们可以将其中指定的Bean注入到特定的context之中，而不是所有的context之中了。很多开发者就希望能够在一种环境下使用Bean定义A，另一种情况下使用Bean定义B。

#### `@Profile`注解

`@Profile`注解允许开发者来表示一个组件是否适合在当前环境来进行注册，只有当前的Profile是激活的时候，对应的Bean才会被注册到上下文中。使用前面的例子，代码可以进行如下调整:

```
@Configuration
@Profile("dev")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```
```
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

`@Profile`注解可以当做元注解来使用。比如，下面所定义的`@Production`注解就可以来替代`@Profile("production")`:

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

`@Profile`注解也可以在方法级别使用，可以声明在包含`@Bean`注解的方法之上：

```
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean
    @Profile("production")
    public DataSource productionDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

> 如果配置了`@Configuration`的类同时配置了`@Profile`，那么所有的配置了`@Bean`注解的方法和`@Import`注解的相关的类都会被传递为该`Profile`。除非这个`Profile`激活了，否则其中的Bean定义都不会激活。如果配置为`@Component`或者`@Configuration`的类标记了`@Profile({"p1", "p2"})`,那么这个类当且仅当`Profile`是p1或者p2的时候才会激活。如果某个`Profile`的前缀是`!`这个非操作符，那么`@Profile`注解的类会只有当前的Profile没有激活的时候才能生效。举例来说，如果配置为`@Profile({"p1", "!p2"})`，那么注册的行为会在Profile为`p1`或者是Profile为非`p2`的时候才会激活。

#### XML中Bean定义的profile

在XML中相对应配置是`<beans/>`中的`profile`属性。我们在前面配置的信息可以被重写到XML文件之中如下：

```
<beans profile="dev"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

当然，也可以通过嵌套`<beans/>`标签来完成定义部分：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="dev">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

`spring-bean.xsd`已经被定义好，可以使用上面例子之中的这类标签。这将为XML文件的配置提供更多便利。

#### 激活profile

现在，我们已经更新了配置信息来使用环境抽象，但是我们还需要告诉Spring来激活具体哪一个`Profile`。如果我们直接启动应用的话，现在就回抛出`NoSuchBeanDefinitionException`异常，因为容器会找不到Spring的Bean`dataSource`。

有多种方法来激活一个Profile，最直接的方式就是通过编程的方式来直接调用`Environment`API，`ApplicationContext`中包含这个接口：

```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("dev");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

额外的，Profile还可以通过`spring.profiles.active`中的属性来指定，可以通过系统环境变量，JVM系统变量，servlet上下文中的参数,甚至是JNDI的一个参数等来写入。在集成测试中，激活Profile可以通过`spring-test`中的`@ActiveProfiles`来实现。

需要注意的是，Profile的定义并不是一种互斥的关系，我们完全可以在同一时间激活多个Profile的。编程上来说，为`setActiveProfile()`方法提供多个Profile的名字即可：

```
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

也可以通过`spring.profiles.active`来指定`，`逗号分隔的多个Profile的名字：

```
-Dspring.profiles.active="profile1,profile2"
```

#### 默认profile

默认的Profile就表示默认启用的Profile。参考如下代码：

```
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有其他的Profile被激活，那么上面代码定义的`dataSource`就会被创建，这种方式就是为默认情况下提供Bean定义的一种方式。一旦任何一个Profile激活了，默认的Profile则不会激活。

默认的Profile的名字可以通过`Environment`中的`setDefaultProfiles()`方法或者是通过`spring.profiles.default`属性来更改。

### 属性源抽象

Spring的`Environment`的抽象提供了一些层次化的搜索选项，来配置的源信息。具体的内容，参考如下代码：

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("Does my environment contain the 'foo' property? " + containsFoo);
```

在上面的代码片段之中，我们看到一个high-level的查找Spring是否定义`foo`属性的一种方式。为了知道Spring中是否包含这个属性，`Environment`对象会针对`PropertySource`的集合进行查找。`PropertySource`是针对一些`key-value`的属性对的简单抽象，而Spring的`StandardEnvironment`是由两个`PropertySource`对象所组成的，一个代表的是JVM的系统属性（可以通过`System.getProperties()`来获取），而另一种则是系统的环境变量（通过`System.getenv()`来获取。）

> 这些默认的属性源都是`StandardEnvironment`的代表，在任何应用之中都可以使用。`StandardServletEnvironment`则是包含Servlet配置的环境信息，其中会包含很多Servlet的配置和Servlet上下文参数。`StandardPortletEnvironment`类似于`StandardServletEnvironment`，能够配置portlet上下文参数。可以参考其Javadoc了解更多信息。

具体的说，当使用`StandardEnvironment`的时候，调用`env.containsProperty("foo")`将返回一个`foo`的系统属性，或者是`foo`的运行时环境变量。

> 查询配置属性是按层次来查询的。默认情况下，系统属性优于系统环境变量，所以如果`foo`属性在两个环境中都有配置的话，那么在调用`env.getProperty("foo")`期间，系统属性值会优先返回。需要注意的是，属性的值是不会合并的，而是完全覆盖掉。
在一个普通的`StandardServletEnvironment`之中，查找的顺序如下，优先查找`* ServletConfig`参数（比如`DispatcherServlet`上下文），然后是`* ServletContext`参数（web.xml中的上下文参数），再然后是`* JNDI`环境变量，JVM系统变量（"-D"命令行参数）以及JVM环境变量（操作系统环境变量）。

最重要的是，整个的查找机制是可以配置的。也许开发者自己有些定义的配置源信息想集成到配置检索的系统中去。没问题，只要实现开发者自己的`PropertySource`并且将其加入到当前`Environment`的`PropertySources`之中即可：

```
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在上面的代码之中，`MyPropertySource`被添加到检索配置的第一优先级之中。如果存在一个`foo`属性，它将由于其他的`PropertySource`之中的`foo`属性优先返回。`MutablePropertySources`API提供一些方法来允许精确控制配置源。

### `@PropertySource`注解

`@PropertySource`注解提供了一种方便的机制来将`PropertySource`增加到Spring的`Environment`之中。
给定一个文件`app.properties`包含了key-value对`testbean.name=myTestBean`,下面的代码中，使用了`@PropertySource`调用`testBean.getName()`将返回`myTestBean`:

```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {
    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

任何的`@PropertySource`之中形如`${...}`的占位符，都可以被解析成`Environment`中的属性资源，比如:

```
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {
    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设上面的`my.placeholder`是我们已经注册到`Environment`之中的资源，举例来说，JVM系统属性或者是环境变量的话，占位符会解析成对象的值。如果没有的话，`default/path`会来作为默认值。如果没有指定默认值，而且占位符也解析不出来的话，就会抛出`IllegalArgumentException`。

### 占位符解析

从历史上来说，占位符的值是只能针对JVM系统属性或者环境变量来解析的。但是现在不是了，因为环境抽象已经继承到了容器之中，现在很容易通过容器将占位符解析集成。这意味着开发者可以任意的配置占位符：

* 开发者可以自由调整系统变量还有环境变量的优先级
* 开发者可以额外增加自己的属性源信息

具体的说，下面的XML配置不会在意`customer`属性在哪里定义，只有这个值在`Environment`之中有效即可：

```
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```