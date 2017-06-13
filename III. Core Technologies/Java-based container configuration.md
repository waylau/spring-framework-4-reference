### 6.12 Java-based container configuration
#### 6.12.1 Basic concepts: @Bean and @Configuration
Spring 中新的 Java 配置支持的核心就是`@Configuration`注解的类和`@Bean`注解的方法。

`@Bean`注解用来指定一个方法实例，配置和初始化一个新对象交给Spring IOC容器管理。对于那些熟悉Spring`<beans>`XML配置的人来说，`@Bean`注解和`<bean>`元素扮演相同的角色。你可以使用在`@Component`类中使用`@Bean`注解方法，但更常用的，是在`@Configuration`类中使用。

`@Configuration`注解的类表示它的主要目的是作为bean定义的来源。另外，`@Configuration`类允许内部bean依赖通过简单地调用同一类内的其他`@Bean`方法进行定义。最简单的`@Configuration`类可能是这样的：
```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

}
```
上面`AppConfig`类与下面Spring XML中的<beans/>配置是等同的：
```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

>Full @Configuration vs 'lite' @Beans mode?
当`@Bean`方法不是在`@Configuration`中声明时，被称为‘lite’ mode。例如，bean方法在`@Component`或甚至在简单的PO类中声明，都被认为是’lite‘。   
与full `@Configuration`不同的是，lite`@Bean`方法无法声明内部bean依赖。通常，当在`lite`mode下时一个`@Bean`方法不应该调用另一个`@Bean`方法。   
只有在`@Configuration`类下使用`@Bean`方法是被推荐的，确保'full'mode经常使用到。这将阻止被同一个`@Bean`方法意外地多次调用，帮助减少在`lite`mode下出现的难以追踪的bugs。

`@Bean`和`@Configuration`注解将会在下面的章节中被深入地探讨。首先，我们先来看看使用基于 Java 的配置创建 Spring 容器的各种方
式。

#### 6.12.2 Instantiating the Spring container using AnnotationConfigApplicationContext
下面的章节讲解了 Spring 的 `AnnotationConfigApplicationContext`，是在 Spring3.0 中新加入的。这个全能的`ApplicationContext`实现类不仅仅可以接受`@Configuration`类作为输入，也可以是普通的`@Component`类，还有使用 JSR-330 元数据注解的类。
当`@Configuration`类作为输入时，`@Configuration`类本身作为 bean 被注册了，并且类内所有声明的`@Bean`方法也被作为 bean 注册了。
当`@Component` 和 JSR-330 类作为输入时，它们被注册为 bean，并且被假设如`@Autowired` 或`@Inject` 的 DI 元数据在类中需要的地方使用。

##### 简单构造
与使用Spring XML配置作为输入实例化`ClassPathXmlApplicationContext`过程类似，当实例化`AnnotationConfigApplicationContext`时`@Configuration`类可能作为输入。这就允许在 Spring 容器中完全可以不使用 XML：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
正如上面所提到的，`AnnotationConfigApplicationContext`不仅仅局限于和`@Configuration`类合作。任意`@Component`或 JSR-330 注解的类都可以作为构造方法的输入。比如：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
上面假设`MyServiceImpl`，`Dependency1`和 `Dependency2` 使用了 Spring 依赖注入注解，比如`@Autowired`。

#### 使用 register(Class<?>...)编程式构建容器
`AnnotationConfigApplicationContext`可以使用无参构造方法来实例化，之后使用`register()`方法来配置。这个方法当编程式地构建`AnnotationConfigApplicationContext`时尤其有用。
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
#### 使用 scan(String..)开启组件扫描
要开启组件扫描，仅仅需要在你的`@Configuration`类上加上如下注解：
```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```
有经验的 Spring 用户肯定会熟悉下面这个 Spring 的 `context:`命名空间中的常用 XML声明：
```
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```
在上面的示例中，`com.acme`包会被扫描到，去查找任意`@Component`注解的类，那些类就会被注册为 Spring 容器中的 bean。`AnnotationConfigApplicationContext` 暴露出 `scan(String ...)`方法，允许相同的组件扫描功能：
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

>记得`@Configuration`类是使用`@Component`进行元数据注解的，所以它们是组件扫描的候选者！在上面的示例中，假设 `AppConfig` 是声明在 `com.acme` 包（或是其中的子包）中的，那么会在调用 `scan()`方法时被找到，在调用 `refresh()`方法时，所有它的`@Bean`方法就会被处理并注册为容器中的 bean。

#### 支持Web应用的`AnnotationConfigWebApplicationContext`
`WebApplicationContext` 是 `AnnotationConfigApplicationContext` 的变种，适用于 `AnnotationConfigWebApplicationContext`。当配置 Spring 的 Servlet 监听器`ContextLoaderListener`，Spring MVC 的 `DispatcherServlet` 等时，这个实现类就可能被用到了。下面的代码是在 `web.xml` 中的片段，配置了典型的 Spring MVC 的 Web 应用程序。注意 `contextClass` 上下文参数和初始化参数的使用：
```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 6.12.3 Using the @Bean annotation
`@Bean`是一个方法级的注解，与XML的`<bean/>`元素功能相同。该注解支持一些`<bean/>`上的属性，如:`init-method`, `destroy-method`, `autowiring` 和 `name`。
你可以在`@Configuration`或`@Component`类里使用`@Bean`注解。

##### 声明一个bean
要声明一个bean，可以简单地使用`@Bean`注解到一个方法上。你可以在`ApplicationContext`容器中使用这个方法的返回值来注册一个bean定义。默认的，bean的名称将会与方法的名称相同。下面是一个`@Bean`声明的简单示例：
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
上述配置是完全与下面的Spring XML相等的：
```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```
两者的声明都在`ApplicationContext`中创建了一个名为`transferService`的bean，绑定了一个`TransferServiceImpl`类型的实例对象：
>transferService -> com.acme.TransferServiceImpl

#### Bean依赖
`@Bean`注解的方法可以有任意数量的参数依赖来构建bean。比如，如果我们的`TransferService`要求一个`AccountRepository`，我们可以具体化该依赖通过一个方法参数：
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

}
```
这个解析机制与基于构造器的依赖注入非常相似，详情查看“Constructor-based dependency injection”章节。

#### 接收生命周期回调
任何使用`@Bean`注解定义的类都支持定期的生命周期回调，可以使用 JSR-250 的`@PostConstruct`和`@PreDestroy`注解，详情查看`6.9.8 @PostConstruct and @PreDestroy`

定期的生命周期回调得到了很好地支持。如果一个bean实现了`InitializingBean`, `DisposableBean`, or `Lifecycle`接口，他们各自的方法将会被容器调用。

标准的一套`*Aware`接口，例如，`BeanFactoryAware`, `BeanNameAware`, `MessageSourceAware`, `ApplicationContextAware`等等，也一样得到了很好的支持。

`@Bean`注解支持指定任意的初始化和销毁的回调方法，非常类似于Spring XML中bean元素的`init-method`和`destroy-method`属性：
```
public class Foo {
    public void init() {
        // initialization logic
    }
}

public class Bar {
    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```
默认地，使用Java配置的bean定义有public的`close`或`shutdown`方法时，在销毁回调时会被自动地调用。所以如果你有public的`close`或`shutdown`方法，而你不想它在容器关闭时被调用，那你可以简单地加上`@Bean(destroyMethod="")`到你的bean定义上，以此来使默认的`(inferred)`行为失效。

你可以想默认地通过JNDI获取你需要的在应用之外管理的资源作为它的生命周期。特别地，一定要始终做一个`DataSource`，因为它是已知的有问题的：
```
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```
当然，上面`Foo`的示例，它等同于在构造期间调用`init()`方法：
```
@Configuration
public class AppConfig {
    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...
}
```

>当你完全使用Java配置，你对你的对象可以做任何你想做的，不要总必须依赖于容器的生命周期！

#### 指定bean的作用域
##### 使用`@Scope`注解
你可以指定使用`@Bean`注解的bean定义应该有个指定的作用域。你可以使用任何在6.5章节 Bean scopes中提到的标准的作用域。

默认的作用域是`singleton`，但你可以使用`@Scope`注解进行覆盖：
```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }

}
```
##### @Scope 和 scoped-proxy
Spring 提供了一个便利的途径，通过作用域代理处理作用域依赖。当使用XML配置时，最简单的方法创建这样一个代理就是使用`<aop:scoped-proxy/>`元素。在Java中使用`@Scope`注解带`proxyMode`属性来配置你的beans是等价的。默认的是没有代理（`ScopedProxyMode.NO`），但你可以指定`ScopedProxyMode.TARGET_CLASS`或`ScopedProxyMode.INTERFACES`。
```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```
##### 自定义bean命名
默认地，配置类使用`@Bean`方法的名称来作为注册bean的名称。这个方法可以被重写，当然，使用的是`name`属性。
```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }

}
```
#### Bean 混淆现象
在 6.3.1 , “Naming beans” 章节讨论过，有时需要对单个bean使用多个名称才能满足需要，否则就会有bean混淆现象。`@Bean`注解的`name`属性为了达到该目的可以接收一个字符串数组。
```
@Configuration
public class AppConfig {

    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }

}
```
#### Bean 描述
有时提供一个更详细的bean的文本描述是很有帮助的。这是特别有用当beans用于（可能通过JMX）监测的目的。

使用`@Description`注解来对`@Bean`添加一个描述：
```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }

}
```
#### 6.12.4 Using the @Configuration annotation
`@Configuration`是一个类级别的注解，用于表明此对象是一个bean定义的资源。`@Configuration`类通过public的`@Bean`注解的方法来声明beans。调用`@Configuration`类的`@Bean`方法也可以被用于定义inter-bean依赖。详情查看`6.12.1, “Basic concepts: @Bean and @Configuration”章节`。

#### Injecting inter-bean dependencies
当`@Beans`上有其它bean的依赖，简单演示一个bean方法调用另一个：
```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }

}
```
在上面的例子中，`foo`bean通过构造器注入接收了一个`bar`引用。

>这个声明inter-bean依赖的方法只会在`@Bean`声明在`@Configuration`类下时才会起效。你不能使用简单的`@Component`类声明inter-bean依赖。

#### 查找方法注入
如前所述，查找方法注入是一个高级特性，你应该很少会用到。当一个singleton作用域的bean有一个prototype作用域bean的依赖时，这将很有用。对于Java配置，提供了一种自然的方式实现这一模式：
```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();

        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
    return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
使用Java配置支持，你可以创建一个`CommandManager`的子类，其虚方法`createCommand()`将会被重写，通过此种方式，来查找一个新的（prototype）的command对象。
```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```
#### Further information about how Java-based configuration works internally
下面的例子展示了`@Bean`注解的方法被调用了2次：
```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }

}
```
`clientDao()`在`clientService1()`中被调用一次，在`clientService2()`也被调用一次。从这个方法创建一个`ClientDaoImpl`实例并返回它，你通常期望会生成2个实例（每个service一个）。那肯定是有问题的：在Spring中，实例化beans默认拥有一个singleton作用域。这就是神奇的地方：所有的`@Configuration`类在启动的阶段使用CGLIB进行子类化。在子类中，子方法首先检查容器中任何缓存（scoped）的beans在它调用父方法及创建新的实例之前。注意，Spring3.2已经不再需要添加CGLIB到你的classpath中了，因为CGLIB类已经被打包到`org.springframework`，并被包含到`spring-core`JAR包中了。

>根据你的bean的作用域的不同，行为可能会有所不同。我们这里仅谈论了singltons。

>由于CGLIB在启动期间动态添加特性，所以这里有几个限制：
- 配置类必须不能为final
- 应该有一个无参的构造器

#### 6.12.5 Composing Java-based configurations
##### 使用`@Import`注解
就像 Spring 的 XML 文件中使用的`<import/>`元素帮助模块化配置一样，`@Import`注解允许从其它配置类中加载`@Bean`的配置：
```
@Configuration
public class ConfigA {

     @Bean
    public A a() {
        return new A();
    }

}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }

}
```
现在，当实例化上下文时，不需要指定`ConfigA.class` 和 `ConfigB.class` 了，仅仅 `ConfigB` 需要被显式提供：
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```
这种方式简化了容器的实例化，仅仅是一个类需要被处理，而不是需要开发人员在构造时记住很多大量的`@Configuration`类。
##### 在引入的@Bean定义中注入依赖
上面的示例是可行的，但是太过简单。在很多实际场景中，bean 会有依赖其它配置的类的依赖。当使用 XML 时，这本身不是什么问题，因为没有调用编译器，而且我们可以仅仅声明 `ref="someBean"`并且相信 Spring 在容器初始化时可以完成。当然，当使用`@Configuration`的类时，Java 编译器在配置模型上放置约束，对其它 bean 的引用必须是符合 Java 语法的。

幸运的是，解决这个问题非常简单。我们已经讨论过了，`@Bean`方法可以有任意个参数声明bean依赖。让我们考虑一个更真实的场景，有个`@Configuration`类，每个bean都依赖其他的配置中声明的bean：
```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
有另外一种方法可以达到一样的结果。记住`@Configuration`类最终是容器中的另外一个bean：也就是说它们可以像其它 bean 那样利用`@Autowired`注入元数据！

>要确定这种方式的依赖注入是最简单的那种。`@Configuration`类在容器初始化时就进行处理了，强制要求依赖使用这种方式进行注入可能导致意外的早期初始化问题。如果可能，就采用如上述例子所示的基于参数的注入。
同时，也要特别小心通过`@Bean`的`BeanPostProcessor` `BeanFactoryPostProcessor`定义。它们应该被声明为static的`@Bean`方法，不会触发实例化它们的包含类。否则，`@Autowired`和`@Value`将在配置类上不生效，因为它太早被创建为一个实例了。

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
在上面的场景中，使用`@Autowired`工作正常，提供所需的模块化，但是准确地决定在哪儿声明自动装配的 bean 还是有些含糊。比如，作为开发者来看待 `ServiceConfig`，你如何准确知道`@Autowired AccountRepository` 在哪里声明的？它没有显式地出现在代码中，这可能很不错。要记得 `SpringSource Tool Suite` 提供工具可以生成展示所有对象是如何装配起来的-那可能就是你所需要的。而且，你的 Java IDE 可以很容器发现所有的声明，还有使用的 `AccountRepository` 类型，也会很快地给你展示出`@Bean`方法的位置和返回的类型。

在这种歧义不能接受，你想直接在 IDE 中从一个`@Configuration` 类导航到另一个，要考虑自动装配配置类的本身：
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
在上面的情形中，定义`AccountRepository`是完全明确的。而`ServiceConfig`却紧紧耦合到`RepositoryConfig`中了；这就需要我们来权衡了。这种紧耦合可以使用基于接口或抽象基类的`@Configuration`类来减轻。考虑下面的代码：
```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();

}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }

}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class}) // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
现在`ServiceConfig` 和 `DefaultRepositoryConfig` 的耦合就比较松了，并且内建的 IDE 工具也一直有效：对于开发人员来说会更加简单地获取 `RepositoryConfig` 实现类的类型层次。以这种方式，导航`@Configuration` 类和它们的依赖就和普通的基于接口代码的导航过程没有任何区别了。

#### Conditionally include @Configuration classes or @Bean methods
基于任意的系统状态，使完整的`@Configuration`类或单独的`@Bean`方法有条件性地生效或失效是很有用的。一个常见的例子就是当一个指定的profile已经在Spring环境中生效时使用`@Profile`注解来激活beans（详见`Section 6.13.1, “Bean definition profiles”`）。

`@Profile`注解是通过使用一个非常灵活的称为`@Conditional`的注解实现的。`@Conditional`注解表示具体的` org.springframework.context.annotation.Condition`实现者应该在`@Bean`被注册前先被访问。

`Condition`接口的实现简单地提供了一个`matches(…​)`方法，用来返回`true`或`false`。例如，以下就是使用`@Profile`具体的`Condition`实现：
```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```
#### 结合 Java和 和 XML
Spring 的`@Configuration`类并不是完全100%地支持 Spring XML 替换的。一些基本特性，比如 Spring XML 的命名空间会保持在一个理想的方式下去配置容器。在 XML 更便于使用或是必须要使用的情况下，你也有另外一个选择：以“XML为中心”的方式来实例化容器，比如，`ClassPathXmlApplicationContext`，或者以“Java 为中心”的方式，使用`AnnotationConfigurationApplicationContext` 和`@ImportResource`注解来引入需要的 XML。

##### 以“XML为中心”使用@Configuration
以XML配置文件结合包含`@Configuration`类的点对点的方式来启动 Spring 容器是更好的选择。比如，在使用了 Spring XML配置的大型的代码库中，根据需要从已有的 XML 文件中创建`@Configuration`类是很简单的。在下面，你会发现在这种“XML为中心”情形下，使用`@Configuration` 类的选项。
记住，`@Configuration`类最终仅仅是容器中的bean。在这个示例中，我们创建了名为`AppConfig`的`@Configuration`类，并且将它包含在 `system-test-config.xml`文件中作为`<bean/>`的定义。因为开启了`<context:annotation-config/>`配置，容器会识别`@Configuration`注解，并以合适的方式处理声明在`AppConfig`中`@Bean`方法。
```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }

}
```
###### system-test-config.xml:
```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
> jdbc.properties:   
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb   
jdbc.username=sa   
jdbc.password=   

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```
>在上面的`system-test-config.xml`文件中，`AppConfig`的`<bean/>`定义没有声明`id`元素。这么做也是可以接受的，就不必让其它 bean 去引用它了，同时也就不可能从容器中通过明确的名称来获取它了。同样地，`DataSource`bean，通过类型自动装配，那么明确的 bean`id`就不严格要求了。

因为`@Configuration`是使用`@Component`来元数据注解的，被`@Configuration`注解的类是自动作为组件扫描的候选者的。使用上面相同的场景，我们可以重新来定义`system-test-config.xml`文件来利用组件扫描的优点。注意这种情况下，我们不需要明确地声明`<context:annotation-config/>`，因为`<context:component-scan/>`开启了相同的功能。

##### system-test-config.xml:
```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
##### 使用了@ImportResource导入XML的@Configuration类为中心
在`@Configuration`类作为配置容器主要机制的应用程序中，使用一些 XML 还是必要的。在这些情况中，仅仅使用`@ImportResource`来定义 XML 就可以了。这么来做就实现了“Java为中心”的方式来配置容器并保持 XML 在最低限度。
```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }

}
```
```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```
>jdbc.properties  
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb   
jdbc.username=sa   
jdbc.password=  
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```
