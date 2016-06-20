Bean 总览
====

Spring IoC 容易管理一个或者多个 bean。 bean 由应用到到容器的配置元数据创建,例如,在 XML 中定义 `<bean/>` 的形式。

容器内部,这些 bean 定义表示为 BeanDefinition 对象,其中包含(其他信息)以下元数据:

* 限定包类名称:典型的实际实现是定义 bean 的类。
* bean 行为配置元素,定义了容器中的Bean应该如何行为(范围、生命周期回调,等等)。
* bean 需要引用其他 bean 来完成工作,这些引用也称为合作者或依赖关系。
* 其他配置设置来设置新创建的对象,例如,连接使用 bean 的数量管理连接池,或者池的大小限制。

以下是每个 bean 定义的属性。

Table 6.1. The bean definition

属性 | 解释
---- | ----
class | Section 6.3.2, “Instantiating beans”
name | Section 6.3.1, “Naming beans”
scope | Section 6.5, “Bean scopes”
constructor arguments | Section 6.4.1, “Dependency Injection”
properties | Section 6.4.1, “Dependency Injection”
autowiring mode | Section 6.4.5, “Autowiring collaborators”
lazy-initialization mode | Section 6.4.4, “Lazy-initialized beans”
initialization method | the section called “Initialization callbacks”
destruction method | the section called “Destruction callbacks”

除了包含的信息里面 bean 定义的如何创建一个特定的bean, ApplicationContext 的实现还允许由用户注册现有创建在容器之外的现有对象。这是通过访问 ApplicationContext 的 BeanFactory 的 getBeanFactory() 方法 返回 BeanFactory 的实现 DefaultListableBeanFactory 。DefaultListableBeanFactory 支持这种通过 registerSingleton(..) 和registerBeanDefinition(..) 方法来注册。然而,典型的应用程序只能通过元数据定义的 bean 来定义。

需要尽早注册 Bean 元数据和手动使用单例的实例,这是为了使容器正确推断它们在自动装配和其他内省的步骤。虽然覆盖现有的元数据和现有的单例实例在某种程度上是支持的,新 bean 在运行时(同时动态访问工厂)注册不是官方支持,可能会导致并发访问 bean 容器中的异常和/或不一致的状态。

### 命名bean

每个 bean 都有一个或多个标识符。这些标识符在容器托管 bean 必须是唯一的。bean 通常只有一个标识符,但如果它需要不止一个,可以考虑额外的别名。

在基于 xml 的配置元数据,您可以使用 id 和 / 或名称属性指定 bean 标识符(。id 属性允许您指定一个 id。通常这些名字字母数字(“myBean”、“fooService”,等等),但可以包含特殊字符。如果你想介绍其他别名 bean,您还可以指定属性名称,由逗号分隔(,),分号(;),或白色空格。作为一个历史因素的要注意,在 Spring 3.1 版本之前,id 属性被定义为 xsd:ID类型,它限制可能的字符。3.1,它被定义为一个 xsd:string 类型。注意,bean id 独特性仍由容器执行,虽然不再由 XML 解析器。

你不需要提供一个 bean 的名称或id。如果没有显式地提供名称或id, 容器生成一个唯一的名称给 bean 。然而,如果你想引用 bean 的名字,通过使用 ref 元素或使用 [Service Locator（服务定位器）](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-servicelocator)风格查找,你必须提供一个名称。不使用名称的原因是，[内部 bean](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-inner-beans) 和[自动装配的合作者](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-autowire)。


*bean 名约定*

*约定是使用标准 Java 实例字段名称命名 bean 时的约定。也就是说,bean 名称开始以小写字母开头,后面采用“骆峰式”。例如“accountManager”、“accountService’,‘userDao’,‘loginController’,等等。*

*一致的beans命名可以让您的配置容易阅读和理解，如果你正在使用Spring AOP，当你通过 bean 名称应用到 advice 时，这会对你帮助很大。*

#### bean 的别名

在对 bean 定义时，除了使用 id 属性指定一个唯一的名称外，为了提供多个名称，需要通过 name 属性加以指定，所有这个名称都指向同一个bean，在某些情况下提供别名非常有用，比如为了让应用每一个组件都能更容易的对公共组件进行引用。然而，在定义 bean 时就指定所有的别名并不总是很恰当。有时我们期望能够在当前位置为那些在别处定义的bean引入别名。在XML配置文件中，可以通过`<alias/>`元素来完成 bean 别名的定义，例如：

	<alias name="fromName" alias="toName"/>

在这种情况下，如果容易中存在名为 fromName 的 bean 定义，在增加别名定义后，也可以用 toName 来引用。

例如，在子系统 A 中通过名字 subsystemA-dataSource 配置的数据源。在子系统B中可能通过名字 subsystemB-dataSource 来引用。当两个子系统构成主应用的时候，主应用可能通过名字 myApp-dataSource 引用数据源，将全部三个名字引用同一个对象，你可以将下面的别名定义添加到应用配置中：

	<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
	<alias name="subsystemA-dataSource" alias="myApp-dataSource" />

现在每个子系统和主应用都可以通过唯一的名称来引用相同的数据源，并且可以保证他们的定义不与任何其他的定义冲突。

*基于 Java 的配置*

*如果你想使用基于 Java 的配置，@Bean 注解可以用来提供别名，详细信息请看 [Section 6.12.3, “Using the @Bean annotation”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-java-bean-annotation)*

### 实例化bean

bean 定义基本上就是用来创建一个或多个对象的配置，当需要一个 bean 的时候，容器查看配置并且根据 bean 定义封装的配置元数据创建（或获取）一个实际的对象。

如果你使用基于 XML 的配置，你可以在`<bean/>`元素通过 class 属性来指定对象的类型。这个 class 属性，实际上是 BeanDefinition 实例中的一个 Class 属性。这个 class 属性通常是必须的（例外情况，查看 [“使用实例工厂方法实例化” 章节](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-class-instance-factory-method)和 [Section 6.7, “Bean定义的继承”）](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-child-bean-definitions)，使用 Class 属性的两种方式：

* 通常情况下，直接通过反射调用构造方法来创建 bean，和在 Java 代码中使用 new 有点像。
* 通过静态工厂方法创建，类中包含静态方法。通过调用静态方法返回对象的类型可能和 Class 一样，也可能完全不一样。

*内部类名。如果你想配置使用静态的内部类，你必须用内部类的二进制名称。例如，在 com.example 包下有个 Foo 类，这里类里面有个静态的内部类Bar，这种情况下bean定义的class属性应该…com.example.Foo$Bar
注意，使用$字符来分割外部类和内部类的名称。*

#### 通过构造函数实例化

当你使用构造方法来创建 bean 的时候，Spring 对类来说并没有什么特殊。也就是说，正在开发的类不需要实现任何特定的接口或者以特定的方式进行编码。但是，根据你使用那种类型的 IoC 来指定 bean，你可能需要一个默认（无参）的构造方法。

Spring IoC 容器可以管理几乎所有你想让它管理的类，它不限于管理POJO。大多数 Spring 用户更喜欢使用 POJO（一个默认无参的构造方法和setter,getter方法）。但在容器中使用非 bean 形式(non-bean style)的类也是可以的。比如遗留系统中的连接池，很显然它与 JavaBean规范不符，但 Spring 也能管理它。

当使用基于XML的元数据配置文件，可以这样来指定 bean 类：

	<bean id="exampleBean" class="examples.ExampleBean"/>
	
	<bean name="anotherExample" class="examples.ExampleBeanTwo"/>

给构造方法指定参数以及为bean实例化设置属性将在后面的[依赖注入](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-collaborators)中详细说明。

#### 使用静态工厂方法实例化

当采用静态工厂方法创建 bean 时，除了需要指定 class 属性外，还需要通过 factory-method 属性来指定创建 bean 实例的工厂方法。Spring将调用此方法(其可选参数接下来介绍)返回实例对象，就此而言，跟通过普通构造器创建类实例没什么两样。

下面的 bean 定义展示了如何通过工厂方法来创建bean实例。注意，此定义并未指定返回对象的类型，仅指定该类包含的工厂方法。在此例中，createInstance() 必须是一个 static 方法。

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

给工厂方法指定参数以及为bean实例设置属性的详细内容请查阅依赖和配置详解。

使用实例工厂方法实例化

与通过 静态工厂方法 实例化类似，通过调用工厂实例的非静态方法进行实例化。 使用这种方式时，class属性置为空，而factory-bean属性必须指定为当前(或其祖先)容器中包含工厂方法的bean的名称，而该工厂bean的工厂方法本身必须通过factory-method属性来设定。

```xml
<!-- 工厂bean，包含createInstance()方法 -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 其他需要注入的依赖项 -->
</bean>

<!-- 通过工厂bean创建的ben -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以有多个工厂方法，如下代码所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 其他需要注入的依赖项 -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private static AccountService accountService = new AccountServiceImpl();

    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }

}
```

这种做法表明工厂bean本身也可以通过依赖注入（DI）进行管理配置。查看依赖和配置详解。

*在Spring文档中，factory bean是指在Spring容器中配置的工厂类通过 实例 或 静态 工厂方法来创建对象。相比而言, FactoryBean (注意大小写) 代表了Spring中特定的 FactoryBean*
