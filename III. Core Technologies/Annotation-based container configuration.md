### 6.9 Annotation-based container configuration

> ##### 注解配置比XML配置更好嘛？  
基于注解配置的介绍就提出了这样的问题，这种方法要比XML‘更好’吗？简短的回答就是具体问题具体分析。完整的答案就是每种方法都有它的利与弊，通常是让开发人员来决定使用哪种策略更适合使用。由于定义它们的方式，注解在它们的声明中提供了大量的上下文，使得配置更加简短和简洁。然而，XML更擅长装配组件，而不需要触碰它们源代码或重新编译。一些开发人员更喜欢装配源码而其他人认为被注解的类不再是POJO了，此外，配置变得分散并且难以控制。

>无论怎么选择，Spring都可以容纳两种方式，甚至是它们的混合体。最值得指出的是通过JavaConfig（6.12节），Spring允许以非侵入式的方式来使用注解，而不需要触碰目标组件的源代码和工具，所有的配置方式都是SpringSource Tool Suite所支持的。

作为 XML 配置的另外一种选择，依靠字节码元数据的基于注解的配置来装配组件代替了尖括号式的声明。作为使用 XML 来表述 bean 装配的替换，开发人员可以将配置信息移入到组件类本身中，在相关的类，方法或字段声明上使用注解。正如在上面章节`“Example：RequiredAnnotationBeanPostProcessor”`中所提到的，使用`BeanPostProcessor`来连接注解是扩展 Spring IoC 容器的一种常用方式。比如，Spring 2.0 引入的用`@Required`注解来强制所需属性不能为空。在 Spring 2.5 中，可以使用相同的处理方法来驱动 Spring 的依赖注入。从本质上来说，`@Autowired` 注解提供了在` Section 6.4.5, “Autowiring collaborators”`中描述的相同能力，但却有更细粒度的控制和更广泛的适用性。Spring2.5 也添加了对 JSR-250 注解的支持，比如`@Resource`，`@PostConstruct` 和`@PreDestroy`。Spring 3.0添加了对 JSR-330 （对 Java 的依赖注入）注解的支持，包含在javax.inject 包下，比如`@Inject`，`@Qualifier`，`@Named` 和`@Provider`，当 JSR330 的 jar 包在类路径下时就可以使用。使用这些注解也需要在 Spring 容器中注册特定的`BeanPostProcessor`。

>注解注入会在 XML 注入之前执行，因此同时使用两种方式，那么后面的配置会覆盖前面装配的属性。

一如往常，你可以注册它们作为独立的 bean，但是它们也可以通过包含下面的基于 XML的 Spring 配置代码片段被隐式地注册（注意要包含 context 命名空间）：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```
（隐式注册的后置处理器包含`AutowiredAnnotationBeanPostProcessor`，`CommonAnnotationBeanPostProcessor`，`PersistenceAnnotationBeanPostProcessor`，以及上述的`RequiredAnnotationBeanPostProcessor`）

>`<context:annotation-config/>`仅仅查找定义在同一上下文中的 bean 的注解。这就意味着，如果你为`DispatcherServlet`将`<context:annotation-config/>`放置在`WebApplicationContext`中，那么它仅仅检查控制器中的`@Autowired` bean，而不是你的服务层 bean，可以参看 21.2 节，“DispatcherServlet”来查看更多信息。

### 6.9.1 @Required
`@Required` 注解应用于bean属性的setter方法，就像下面这个示例：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
这个注解只是表明受影响的 bean 的属性必须在 bean 的定义中或者是自动装配中通过明确的属性值在配置时来填充。如果受影响的 bean 属性没有被填充，那么容器就会抛出异常；这就允许了急切而且明确的失败，要避免 NullPointerException。我们推荐你放置断言到 bean 的类中，比如，放置到初始化方法中。这么做可以强制使用所需的外部类的引用和值。

### 6.9.2 @Autowired
正如预期的那样，你可以使用`@Autowired` 注解到“传统的”setter 方法中：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
>JSR 330的`@Inject` 注解可以代替上面示例中的Spring的`@Autowired`注解。

你也可以将注解应用于任意名称和(或)多个参数的方法：
```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```
你也可以将它用于构造方法和字段：
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```
也可以提供`ApplicationContext`中特定类型的所有 bean，通过添加注解到期望哪种类型的数组的字段或者方法上：
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...

}
```
同样，也可以用于特定类型的集合：
```
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...

}
```
>你的beans可以实现`org.springframework.core.Ordered`接口或使用以下注解之一:`@Order`或`@Priority`注解，当你想让列表或数组项能按照指定的顺序排序。

甚至特定类型的 Map 也可以自动装配，如果期望的键的类型是 `String`的话。Map 值会包含所有期望类型的 bean，而键会包含对应 bean 的名字：
```
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...

}
```
默认情况下，当出现零个候选bean的时候，自动装配就会失败；默认的行为是将被注解的方法，构造方法和字段作为需要的依赖关系。这种行为也可以通过下面这样的做法来改变。
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required=false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
>每一个类中仅有一个被注解的构造器可以标记为必须的，但是可以注解多个非必须的构造器。在那种情况下，每个构造方法都要考虑到而且 Spring 使用贪婪模式选择依赖可以被最满足的那个构造方法，也就是参数最多的那个构造方法。  
推荐使用`@Autowired`的required属性而不是@Required注解。required属性表示了属性对于自动装配目的不是必须的，如果它不能被自动装配，那么属性就会忽略了。另一方面，`@Required`更健壮一些，它强制了由容器支持的各种方式的属性设置。如果没有注入任何值，就会抛出对应的异常。

你也可以针对我们熟知的解决依赖关系的接口来使用`@Autowired`：`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`，`ApplicationEventPublisher`和`MessageSource`。这些接口和它们的扩展接口，比
如 `ConfigurableApplicationContext` 或`ResourcePatternResolver`也会被自动解析，而不需要特殊设置的必要。
```
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...

}
```

>`@Autowired`，`@Inject`，`@Resource` 和 `@Value` 注解是由Spring的`BeanPostProcessor`实现类来控制的，反过来说，你不能在`BeanPostProcessor`或 `BeanFactoryPostProcessor`类型（任意之一）应用这些注解。这些类型必须明确地通过 XML 或使用 Spring 的`@Bean`方法来‘装配’。

### 6.9.3 Fine-tuning annotation-based autowiring with @Primary
因为通过类型的自动装配可能有多个候选者，那么在选择过程中通常是需要更多的控制的。达成这个目的的一种做法就是 Spring的`@Primary`注解。当一个单值的依赖有多个候选者bean时，`@Primary`指定了一个优先提供的特殊bean。当多个候选者bean中存在一个确切的指定了'primary'的bean，它将会自动装载这个bean。

让我们假设我们有以下的配置，其中定义了`firstMovieCatalog` 和primary `MovieCatalog`。
```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...

}
```
对于上面的配置，下面的`MovieRecommender`将会使用`firstMovieCatalog`自动注解。
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...

}
```
相应的bean定义如下所示：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary=true>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
### 6.9.4 Fine-tuning annotation-based autowiring with qualifiers
因为通过类型的自动装配可能有多个候选者，那么在选择过程中通常是需要更多的控制的。达成这个目的的一种做法就是 Spring的`@Qualifier`注解。你可以用特定的参数来关联限定符的值，缩小类型的集合匹配，那么特定的 bean 就为每一个参数来选择。最简单的情形，这可以是普通描述性的值：
```
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...

}
```
`@Qualifier`注解也可以在独立的构造方法参数或方法参数中来指定：
```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```
对应的 bean 的定义如下所示。限定符值是“main”的 bean 会用限定了相同值的构造方法参数来装配。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
对于后备匹配，bean名称被认为是默认的限定符值。因此你可以定义一个bean id为“main”，来替代嵌套的限定符元素，这也会达到相同的匹配结果。然而，尽管你使用这种规约来通过名称关联特定的 bean，`@Autowired` 从根本上来说就是关于可选语义限定符的类型驱动注入。这意味着限定符的值，即便有 bean 的名称作为后备，通常在类型匹配时也会缩小语义；它们不会在语义上表达对唯一bean的id的引用。好的限定符的值是“main”
或“EMEA”或“persistent”，特定组件的表达特性是和 bean的 id 独立的，在比如前面示例之一的匿名 bean 的情况下它是可能自动被创建的。

限定符也可以用于类型集合，正如上面讨论过的，比如`Set<MovieCatalog>`。在这种情况下，根据声明的限定符，所有匹配的 bean 都会被注入到集合中。这就说明了限定符不必是唯一的；它们只是构成了筛选条件。比如，你可以使用相同的限定符“action”来定义多个`MovieCatalog` bean；它们全部都会通过`@Qualifier("action")`注解注入到`Set<MovieCatalog>`中。
>如果你想通过名称来实现注解驱动注入，不要首选`@Autowired`，即便在技术上来说能够通过`@Qualifier`值指向一个 bean的名称。相反，使用 JSR-250 的`@Resource`注解，在语义上定义了通过它的唯一名称去确定一个具体的目标组件，声明的类型和匹配过程无关。  
由于这种语义差别的特殊后果，被定义为集合或 map 类型的bean，是不能通过`@Autowired`来进行注入的，因为类型匹配用于它们不太合适。对于这样的 bean 使用`@Resource`，通过唯一的名称指向特定的集合或 map 类型的 bean。`@Autowired` 可以用于字段，构造方法和多参数方法，在参数级允许通过限定符注解缩小匹配范围。相比之下，`@Resource` 仅支持字段和仅有一个参数的bean属性setter方法。因此，如果你的注入目标是构造方法或多参数的方法，那么就坚持使用限定符。

你可以创建自定义的限定符注解。只需定义一个注解并在你的定义中提供`@Qualifier`注解即可。
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```
之后你可以在自动装配的字段和参数上提供自定义的限定符：
```
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;
    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...

}
```
之后，为候选 bean 提供信息，你可以添加`<qualifier/>`标签来作为`<bean/>`标签的子元素，然后指定`type`和`value`值来匹配你自定义的限定符注解。这种类型匹配是基于注解类的完全限定名。否则，作为一种简便的形式，如果没有名称冲突存在的风险，你可以使用短类名。这两种方法都会在下面的示例中来展示。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        _<qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
在 6.10 节， “Classpath scanning and managed components”中，你会看到基于注解的替代，在 XML 中来
提供限定符元数据。特别是在6.10.8, “Providing qualifier metadata with annotations”中。

在一些示例中，使用无值的注解就足够了。这当注解服务于多个通用目的时是很有用的，而且也可以用于集中不同类型的依赖关系。比如，当没有因特网连接时，你可以提供脱机目录来用于搜索。首先定义简单的注解：
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```
之后将注解添加到要被自动装配的字段或属性上：
```
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...

}
```
现在来定义 bean 的限定符的`type`：
```
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```
你也可以定义自定义的限定符注解来接受命名属性，或替代简单的`value`属性。如果多个属性值被指定在字段或参数自动装配，那么要考虑 bean 的定义必须匹配自动装配候选者的所有属性值。示例，考虑下面的注解定义：
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();

}
```
在本例中，`Format`是枚举类型：
```
public enum Format {
    VHS, DVD, BLURAY
}
```
要自动装配的字段使用自定义的限定符和包含`genre`和`format`两个属性的值来注解。
```
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...

}
```
最后，bean 的定义应该包含匹配的限定符值。这个示例也展示了 bean 的`<meta>`属性可能用于替代`<qualifier/>`子元素。如果可用，`<qualifier/>`和它的属性优先，但是如果目前没有限定符，自动装配机制就会在`<meta/>`标签提供的值上失效，就像下面这个示例中的最后两个 bean。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```
### 6.9.5 Using generics as autowiring qualifiers
除了`@Qualifier`注解外，也可以使用java泛型来作为绝对的限定形式。举例，假设你有以下的配置：
```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }

}
```
假设上面的bean实现了泛型接口，也就是，`Store<String>`和`Store<Integer>`，你可以使用`@Autowire`注解到`Store`接口，然后泛型可以被当作限制符使用：
```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```
泛型限制符也应用于自动装配Lists,Maps和Arrays:
```
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```
### 6.9.6 CustomAutowireConfigurer
`CustomAutowireConfigurer`是`BeanFactoryPostProcessor` 的一种，它使得你可以注册自定义的限定符注解类型，即便它们没有使用 Spring 的`@Qualifier` 注解也是可以的。
```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```
`AutowireCandidateResolver`决定它的自动装配的候选者通过：
- 每个bean定义上的`autowire-candidate`值
- `<beans>`上任何可用的`default-autowire-candidates`模式
- `@Qualifier`注解的存在以及任何使用`CustomAutowireConfigurer`注册的自定义注解。

当多个 bean 作为自动装配的候选者，决定“主要”候选者的方式如下：如果候选者中一个 bean 的定义有`primary`属性精确地设置为`true`，那么它就会被选择。

### 6.9.7 @Resource
Spring 也支持使用 JSR 250 的`@Resource`注解在字段或 bean 属性的 setter 方法上的注入。这在 Java EE 5 和 6 中是一个通用的模式，比如在 JSF 1.2 中管理的 bean或 JAX-WS 2.0端点。Spring 也为其所管理的对象支持这种模式。

`@Resource`使用name属性，默认情况下 Spring 解析这个值作为要注入的 bean的名称。换句话说，如果遵循 by-name 语义，正如在这个示例所展示的：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```
如果没有明确地指定name值，那么默认的名称就从字段名称或 setter方法中派生出来。如果是字段，它会选用字段名称；如果是setter方法，它会选用 bean 的属性名称。所以下面的示例中名为“movieFinder”的 bean 通过 setter 方法来注入：
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```
>使用注解提供的name被被`CommonAnnotationBeanPostProcessor`感知的
`ApplicationContext`解析成 bean 的名称。名称可以通过 JNDI方式来解析，只要你明确地配置了 Spring 的 `SimpleJndiBeanFactory`。然而，还是推荐你基于默认行为并仅使用Spring 的 JNDI 查找能力来保存间接的水平。

在使用`@Resource`并没有明确指定名称的独占情况下，和`@Autowired`相似，`@Resource`使用主要类型匹配而不是指定特定名称bean来解决了熟知的可解析的依赖关系：`BeanFactory`，`ApplicationContext`，`ResourceLoader`，`ApplicationEventPublisher`，`MessageSource`接口。

因此，在下面的示例中，`customerPreferenceDao`字段首先寻找名为`customerPreferenceDao`的bean，之后回到匹配 `CustomerPreferenceDao`的主类型。“context”字段基于已知的可解析的依赖类型`ApplicationContext`注入。
```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...

}
```
### 6.9.8 @PostConstruct and @PreDestroy
`CommonAnnotationBeanPostProcessor`不但能识别`@Resource`注解，而且还能识别 JSR-250 生命周期注解。在 Spring 2.5 中引入，对这些注解的支持提供了在initialization callbacks和destruction callbacks章节中描述的另一种选择。只要`CommonAnnotationBeanPostProcessor`在Spring的`ApplicationContext`中注册，一个携带这些注解之一的方法就同时被调用了，和 Spring 生命周期接口方法或明确地声明回调方法相对应。在下面的示例中，在初始化后缓存会预先填充，在销毁后会清理。
```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }

}
```
