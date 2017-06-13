## 6.10 Classpath scanning and managed components
本章中的大多数示例都使用 XML 配置元数据在 Spring 的容器中生产每一个BeanDefinition。之前的章节（6.9 节，“基于注解的容器配置”）表述了如何通过代码级的注解来提供大量的配置信息。尽管在那些示例中，“基础的”bean 的定义都是在 XML 文件中来明确定义的，而注解仅仅进行依赖注入。本节来说明另外一种通过扫描类路径的方式来隐式检测候选组件。候选组件是匹配过滤条件的类库，并有在容器中注册的对应的 bean 的定义。这就可以不用 XML 来执行 bean 的注册了，那么你就可以使用注解（比如`@Component`），AspectJ 风格的表达式，或者是你自定义的过滤条件来选择哪些类有在容器中注册 bean。

>从 Spring 3.0 开始，很多由 Spring JavaConfig 项目提供的特性作为了 Spring Framework 核心的一部分。这就允许你使用 Java 而不是传统的 XML 文件来定义 bean 了。看一看`@Configuration`，`@Bean`，`@Import` 和`@DependsOn` 注解的例子来了解如何使用它们的新特性。

### 6.10.1 @Component and further stereotype annotations
在 Spring 2.0 版之后，`@Repository`注解是任意满足它的角色或典型库（比如熟知的数据访问对象，DAO）的类的标记。这个标记的有多种用途，其中之一就是在  Section 19.2.2, `“Exception translation”`中描述的异常自动转化。

Spring 2.5 引入了更多的典型注解 ：`@Component`，`@Service` 和`@Controller`。`@Component` 是对受 Spring 管理组件的通用注解。`@Repository`，`@Service` 和 `@Controller` 是`@Component`的特殊用途，比如，分别对应了持久层，服务层和表现层。因此，你可以使用`@Component`注解你的组件类，但是如果使用`@Repository`，`@Service` 或 `@Controller` 注解来替代的话，那么你的类更合适由工具来处理或与切面进行关联。比如，这些老套的注解使得理想化的目标称为切入点。而且`@Repository`，`@Service` 和`@Controller`也可以在将来 Spring Framework 的发布中携带更多的语义。因此，如果对于服务层，你在`@Component` 或`@Service` 中间选择的话，那么`@Service`无疑是更好的选择。相似地，正如上面提到的，在持久层中，`@Repository` 已经支持作为自动异常转化的标记。

### 6.10.2 Meta-annotations
Spring提供了很多元注解。元注解简单的说就是能被应用到另一个注解上的注解。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

    // ....
}
```
元注解也可以被组合使用用于创建组合注解。例如Spring MVC的`@RestController`注解就是`@Controller`和`@ResponseBody`。

另外，组合注解可能从元注解中任意重新声明属性来允许用户自定义。这个会特别有用当你只想暴露一个源注解的子集。例如，下面是一个自定义的`@Scope`注解，将作用域名称硬编码到`@Session`注解上，但依然允许自定义`proxyMode`。
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Scope("session")
public @interface SessionScope {

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```
`@SessionScope`可以不声明`proxyMode`就使用，如下所示：
```
@Service
@SessionScope
public class SessionScopedUserService implements UserService {
    // ...
}
```
或者为`proxyMode`重载一个值，如下所示：
```
@Service
@SessionScope(proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedService {
    // ...
}
```
更多详情，请参阅`37. Spring Annotation Programming Model`

### 6.10.3 Automatically detecting classes and registering bean definitions

Spring可以自动检测固有的类并在`ApplicationContext`中注册 对应的`BeanDefinition`。比如，下面的两个类就是自动检测的例子：
```
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```
要自动检测这些类并注册对应的 bean，你需要添加`@ComponentScan`到你的`@Configuration`类上，其中的`base-package`元素是这两个类的公共父类包。（你可以任意选择使用逗号/分号/空格分隔的列表来将每个类引入到父包。）
```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

>为了更简洁，上面的示例可以使用注解的`value`属性，也就是`ComponentScan("org.example")`

下面的示例使用XML配置：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```
>`<context:component-scan>`隐式地开启了`<context:annotation-config>`功能，当使用了`<context:component-scan>`时通常没必要再包含`<context:annotation-config>`。

>对类路径包的扫描需要类路径下存在对应的目录实体。当你使用 Ant 来构建 JAR 包时，要保证你没有激活 JAR 目标中的 files-only 开关。同时，在一些环境中基于安全策略，类路径下的文件不会被暴露出来，如果单独的apps在JDK 1.7.0_45和更高的版本（这需要‘Trusted-Library’计划在你的manifests中，see http://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources）


此外，当你使用`component-scan`时，`AutowiredAnnotationBeanPostProcessor` 和
`CommonAnnotationBeanPostProcessor`二者是隐式包含着的。这就意味着两个组件被自动检测之后就装配在一起了-而不需要在 XML 中提供其它任何 bean 的配置元数据。

>你 可 以 将 annotation-config 属 性 置 为 false 来 关 闭`AutowiredAnnotationBeanPostProcessor`和
`CommonAnnotationBeanPostProcessor` 注册。

### 6.10.4 Using filters to customize scanning
默认情况下，使用`@Component`，`@Repository`，`@Service`，`@Controller` 注解或使用了进行自定义的`@Component` 注解的类本身仅仅检测候选组件。你可以修改并扩展这种行为，仅仅应用自定义的过滤器就可以了。在 `@ComponentScan` 注解中添加
`include-filter` 或 `exclude-filter` 参数就可以了（或者作为component-scan元素的include-filter 或 exclude-filter 子元素）。每个过滤器元素需要`type`和`expression`属性。下面的表格描述了过滤选项。

###### 表 6.5过滤器类型

| 过滤器类型 | 表达式示例 | 描述  |
| -------- |-------------| -----|
|annotation（注解默认）|  org.example.SomeAnnotation|使用在目标组件的类级别上|
|assignable（分配）|  org.example.SomeClass|目标组件分配去（扩展/实现）的类（接口）|
|aspectj | org.example..*Service+ |AspectJ 类型表达式来匹配目标组件|
|regex（正则表达式）| org\.example\.Default.*|正则表达式来匹配目标组件类的名称|
|custom（自定义）|  org.example.MyTypeFilter|自定义`org.springframework.core.type.TypeFilter` 接口的实现类|

下面的示例代码展示了 XML 配置忽略所有`@Repository`注解并使用“sub”库来替代。
```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```
一样的可以使用XML配置：
```
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```
>你可以使用注解的`useDefaultFilters=false`或`<component-scan/>`元素中的`use-default-filter="false"`属性来关闭默认的过滤
器。这会关闭自动检测`@Component`，`@Repository`，`@Service`或`@Controller`注解的类。

### 6.10.5 Defining bean metadata within components
Spring 组件可以为容器提供 bean 定义的元数据。你可以在`@Configuration`注解的类中使用`@Bean`注解来达成这一目的。这里有一个简单的示例：
```
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }

}
```
这个类是个 Spring 组件，它的`doWork()`方法中包含特定应用代码。同时，它也提供了 bean 的定义并且由工厂方法来指向`publicInstance()`方法。`@Bean`注解定义了工厂
方法和其它 bean 定义的属性，比如通过`@Qualifier`注解表示的限定符。其它方法级的注解可以使用的是`@Scope`，`@Lazy`和自定义限定符注解。

>除了组件初始化的角色，`@Lazy`注解也可以跟着`@Autowired`或`@Inject`放置到注入点。在这种上下文中，它将生成一个延迟解析的代理注入。

自动装配字段和方法也是支持的，这在之前讨论过，而且还有对自动装配`@Bean`方法的支持：
```
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters

    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    @Scope(BeanDefinition.SCOPE_SINGLETON)
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }

}
```
这个示例为另外一个名为`privateInstance`的bean的`Age`属性自动装配了`String`方法参数`country`。Spring 的表达式语言元素通过`#{<expression>}`表示定义了属性的值。对于`@Value`注解，当解析表达式文本时，表达式解析器会预先配置来查看 bean 的名称。  
Spring组件中的`@Bean`方法会被不同方式处理，而不会像Spring的`@Configuration`类中的同仁那样。不同的是`@Component`类没有使用 CGLIB 来加强并拦截字段和方法的调用。CGLIB代理是调用`@Configuration` 类中的`@Bean`方法或字段来创建 bean 元数据引用协作对象的手段。方法没有使用通常的 Java 语义来调用。相比之下，调用普通的`@Component`类中的`@Bean`方法或字段有标准的 Java 语义，没有特殊的CGLIB处理或其他的限制应用。

>你可能声明`@Bean`为`static`，允许包含它们的配置类没有创建为实例时进行调用。这使得定义后置处理器特别有意义，例如，`BeanFactoryPostProcessor` 或 `BeanPostProcessor`，由于这样的bean将在容器生命周期早期初始化，因此应该避免在该点触发配置的其他部分。  
注意，调用静态的`@Bean`方法将不会被容器拦截，即使是在`@Configuration`类里（看上面）。这是由于技术上的局限性：CGLIB 的子类仅仅可以重载非静态的方法。因此，直接调用另一个`@Bean`方法将会有标准的Java语义，从而直接从工厂方法返回一个独立的实例。  
在spring容器中，`@Bean`方法的Java语言可视性并没有直接对bean的定义产生影响。你可以自由地声明你自己的工厂方法填充到非`@Configuration`类中，也可以是静态方法。当然，在`@Configuration`类中合格的`@Bean`方法应该是可重写的，也就是说，你不应该声明为`private`或`final`类型。
最后，`@Bean`也可以用在给定组件或配置类的基类上，以及Java 8的默认方法声明的被组件或配置类实现的接口。这使得更灵活地组成复杂的配置结构，甚至在Spring 4.2，多重继承可以通过java 8的默认方法实现。
### 6.10.6 Naming autodetected components
当组件被自动检测作为扫描进程的一部分时，它的bean名称是由`BeanNameGenerator`策略来生成并告知扫描器的。默认情况下，Spring 构造型的注解（`@Component`，`@Repository`，`@Service` 和`@Controller`）包含`name`值将会提供该值给对应 bean 定义的名称。
如果注解不包含`name`值或任何被检测到的组件（比如那些被自定义过滤器发现的），默认的 bean 的名称生成器返回未大写的非限定符类名。比如，如果下面的两个组件被检测到了，那么名称可能是 myMovieLister 和 movieFinderImpl：
```
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```
>如果你不想使用默认的 bean 命名策略，你可以提供自定义的 bean 命名策略。首先，实现`BeanNameGenerator`接口，要保证包含默认的无参构造器。之后，在配置扫描器时，要提供类的完全限定名。
```
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    ...
}
```
```
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```
作为通用的规则，要考虑使用注解指定名称时，其它组件可能会有对它的明确的引用。另一方面，当容器负责装配时，自动生成名称是可行的。
### 6.10.7 Providing a scope for autodetected components
一般情况下，Spring 管理的组件，自动检测组件默认和最多使用的作用域是单例。然而，有时你需要其它作用域，Spring 2.5 提供了一个新的`@Scope`注解。仅仅需要提供作用域的名称到注解上：
```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```
>为作用域提供自定义策略而不是基于注解的方式，实现 `ScopeMetadataResolver`接口，并保证包含了默认的无参构造方法。之后，当配置扫描器时，要提供类的完全限定名：
```
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    ...
}
<beans>
    <context:component-scan base-package="org.example"
            scope-resolver="org.example.MyScopeResolver" />
</beans>
```
当使用确定的非单例作用域时，它可能必须要为该作用域的对象生成代理。这个原因在“Scoped beans as dependencies”章节中描述过了。出于这样的目的，在`component-scan`元素中可以使用 `scoped-proxy` 属性。三种可能的值是：`no（无）`，`interface（接口）`和 `targetClass（目标类）`。比如，下面的配置就会启动标准的 JDK 动态代理：
```
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
<beans>
    <context:component-scan base-package="org.example"
        scoped-proxy="interfaces" />
</beans>
```
### 6.10.8 Providing qualifier metadata with annotations
`@Qualifier`注解在 “Fine-tuning annotation-based autowiring with qualifiers”章节中讨论过了。那章节中演示了`@Qualifier`注解的使用和当你需要处理自动装配候选者时，自定义限定符注解来提供细粒度控制。因为那些示例是基于 XML 的 bean 定义的，限定符元数据在候选者 bean 定义中提供，并使用了 XML 中的 bean 元素的 `qualifier` 和 `meta` 子元素。
当对自动检测组件使用基于类路径扫描时，你可以在候选者类中使用类型级别的注解提供限定符元数据。下面的三个示例就展示了这个技术：
```
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```
>对于大多数基于注解的方式，要记得注解元数据会绑定到类定义本身中去，而使用 XML就允许对多个相同类型的 bean 在它们的限定符元数据中提供变化，因为那些元数据是对于每个实例而不是每个类提供的。
