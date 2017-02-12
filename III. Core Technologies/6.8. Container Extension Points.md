6.8 Container Extension Points
===
通常，应用程序开发者，不需要继承`ApplicationContext`的实现类。相反，Spring IoC容器可以通过插入特殊的集成接口的实现进行拓展。新的一节中，描述了这些集成接口。

### 6.8.1 Customizing beans using a BeanPostProcessor
`BeanPostProcessor`定义了回调方法，通过实现这个回调方法，你可以提供你自己的(或者重写容器默认的)实例化逻辑，依赖分析逻辑等等。如果你想在Spring容器完成实例化，配置，和初始化bean之后，实例化一些自定义的逻辑，你可以插入一个或多个`BeanPostProcessor`的实现。

你可以配置多个`BeanPostProcessor`实例，你可以通过设置`order`属性来控制这些BeanPostProcessors执行的顺序。你可以设置这个属性仅当`BeanPostProcessor`实现了`Ordered`接口。如果你编写自己的`BeanPostProcessor`你也应该考虑实现`Ordered`接口。更多细节，请查看`BeanPostProcessor`和`Ordered`接口的javadocs，也可以看看下面的要点programmatic registration of `BeanPostProcessors`。

>`BeanPostProcessors`作用在一个bean(或者对象)的实例上;也就是说，Spring IoC实例化一个bean实例之后， BeanPostProcessors，才开始进行处理。
`BeanPostProcessors`作用范围是每一个容器。这仅仅和你正在使用容器有关。如果你在一个容器中定义了一个`BeanPostProcessor` ，它将仅仅后置处理那个容器中的beans。换言之，一个容器中的beans不会被另一个容器中的`BeanPostProcessor`处理，即使这两个容器，具有相同的父类。
为了改变实际的bean定义(例如， blueprint 定义的bean)，你反而需要使用`BeanFactoryPostProcessor`，就像在Section 5.8.2, “Customizing configuration metadata with a BeanFactoryPostProcessor”中描述的那样。

`org.springframework.beans.factory.config.BeanPostProcessor`接口，由两个回调方法组成。当这样的一个类注册为容器的一个后置处理器，由于每一个bean实例都是由容器创建的，这个后置处理器会在容器的初始化方法(比如InitializingBean的afterPropertiesSet()和任何生命的初始化方法)被调用之前和任何bean实例化回调之后从容器得到一个回调方法。后置处理器，可以对bean采取任何措施，包括完全忽略回调。一个bean后置处理器，通常会检查回调接口或者使用代理包装一个bean。一些Spring AOP基础设施类，为了提供包装式的代理逻辑，被实现为bean后置处理器。

`ApplicationContext`会自动地检测所有定义在配置元文件中，并实现了`BeanPostProcessor`接口的bean。该`ApplicationContext`注册这些beans作为后置处理器，使他们可以在bean创建完成之后被调用。bean后置处理器可以像其他bean一样部署到容器中。

注意，当在一个配置类上，使用@Bean工厂方法声明一个`BeanPostProcessor`，工厂方法返回的类型应该是实现类自身，或至少也要是 `org.springframework.beans.factory.config.BeanPostProcessor`接口，要清楚地表明这个bean的后置处理器本质特点。否则，在它完全创建之前，`ApplicationContext`将不能通过类型自动探测它。由于一个BeanPostProcessor，早期就需要被实例化，以适应上下文中其他bean的实例化，因此这个早期的类型检查是至关重要的。

>虽然推荐使用`ApplicationContext`的自动检测来注册`BeanPostProcessor`，但是也可以编程式地使用`ConfigurableBeanFactory`的`addBeanPostProcessor`方法来注册。 这对于在注册之前需要对条件逻辑进行评估，或者是在继承层次的上下文之间复制bean后置处理器是有很有用的。但是请注意，编程地添加的`BeanPostProcessors`不需要考虑`Ordered`接口。也就是注册的顺序决定了执行的顺序。也要注意，编程式注册的`BeanPostProcessors`，总是预先被处理----早于通过自动检测方式注册的，同时忽略任何明确的排序。

>实现了`BeanPostProcessor`接口的类是特殊的,会被容器特殊处理。所有`BeanPostProcessors`和他们直接引用的 beans都会在容器启动的时候被实例化,作为`ApplicationContext`特殊启动阶段的一部分。接着，所有的`BeanPostProcessors` 以一个有序的方式进行注册，并应用于容器中的一切bean。因为AOP自动代理本身被实现为`BeanPostProcessor`，这个`BeanPostProcessors`和它直接应用的beans都没有资格进行自动代理，这样就没有切面编织到他们里面。对于所有这样的bean，你会看到一个info日志："Bean foo is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)"。

>注意，如果你有beans使用自动装配或者`@Resource`装配到了你的`BeanPostProcessor`中，当根据依赖搜索匹配类型时，Spring也许会访问意外类型的bean；因此，使它们没有资格进行自动代理，或者其他类型的bean后置处理。例如，你使用`@Resource`注解一个依赖，其中字段或者set方法名，不是和bean声明的名字直接对应，同时没有name属性被使用，然后，Spring将会根据类型，访问其他beans进行匹配。

下面的示例显示了如何在`ApplicationContext`中编写，注册，使用`BeanPostProcessor`。

### Example: Hello World, BeanPostProcessor-style
第一个示例演示了基础的使用。示例中演示了一个自定义的`BeanPostProcessor`实现，在容器创建bean后调用了每个bean的`toString()`方法，并把结果输出到控制台上。

以下是自定义`BeanPostProcessor`实现类的定义：
```
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean,
            String beanName) throws BeansException {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean,
            String beanName) throws BeansException {
        System.out.println("Bean ''" + beanName + "'' created : " + bean.toString());
        return bean;
    }

}
```
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        http://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```
注意`InstantiationTracingBeanPostProcessor`是简单地定义。它甚至没有名字，因为它能像其它bean一样被依赖注入。（上面示例中也定义了一个使用Groovy脚本支持的bean。Spring动态语言支持详见Chapter 34, Dynamic language support）

下面简单的Java应用执行了前面代码和配置：
```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```
上面应用运行输出结果如下：
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961

### Example: The RequiredAnnotationBeanPostProcessor

自定义`BeanPostProcessor`实现与回调接口或注解配合使用，是一种常见的扩展Spring IoC容器的手段。一个例子就是`RequiredAnnotationBeanPostProcessor`-这是一个`BeanPostProcessor`实现，确保用（任意）注解标记的那些JavaBean属性确实被注入一个值。

### 6.8.2 Customizing configuration metadata with a BeanFactoryPostProcessor

下一个我们要看的扩展点是`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。这个接口的语义与`BeanPostProcessor`类似，但有一个主要的不同点：`BeanFactoryPostProcessor`操作bean的配置元数据；也就是说，Spring的IoC容器允许
`BeanFactoryPostProcessor`来读取配置元数据并在容器实例化任何bean(除了`BeanFactoryPostProcessor`)之前可以修改它。

你可以配置多个`BeanFactoryPostProcessor`实例，你可以通过设置`order`属性来控制这些`BeanFactoryPostProcessor`执行的顺序。你可以设置这个属性仅当`BeanFactoryPostProcessor`实现了`Ordered`接口。如果你编写自己的`BeanFactoryPostProcessor`你也应该考虑实现`Ordered`接口。更多细节，请查看`BeanPostProcessor`和`Ordered`接口的javadocs。

>如果你想修改真实的bean实例（也就是说，从配置元数据中创建的对象），那么你需要使用`BeanPostProcessor`（在上面 6.8.1 节， “Customizing beans using a BeanPostProcessor”中描述）来代替。在`BeanFactoryPostProcessor`（ 比 如 使 用
BeanFactory.getBean()）中来使用这些bean的实例虽然在技术上是可行的，但这么来做会引起bean过早实例化，违反标准的容器生命周期。这也会引发一些副作用，比如绕过bean的后置处理。

>`BeanFactoryPostProcessors`作用范围是每一个容器。这仅仅和你正在使用容器有关。如果你在一个容器中定义了一个`BeanFactoryPostProcessor` ，它将仅仅后置处理那个容器中的beans。换言之，一个容器中的beans不会被另一个容器中的`BeanFactoryPostProcessor`处理，即使这两个容器，具有相同的父类。

当在`ApplicationContext`中声明时，bean工厂后置处理器会自动被执行，这就可以对定义在容器中的配置元数据进行修改。Spring包含了一些预定义的bean工厂后置处理器，
比如`PropertyOverrideConfigurer`和`PropertyPlaceholderConfigurer`。自定义的`BeanFactoryPostProcessor`也可以用来，比如，注册自定义的属性编辑器。

`ApplicationContext`会自动检测任意部署其中，且实现了`BeanFactoryPostProcessor`接口的bean。在适当的时间，它用这些bean作为bean工厂后置处理器。你可以部署这些后置处理器bean作为你想用的任意其它的bean。

>和`BeanPostProcessor`一样，通常你不会想配置 `BeanFactoryPostProcessor`来进行延迟初始化。如果没有其它bean引用`Bean(Factory)PostProcessor`，那么后置处理器就不会被初始化了。因此，标记它为延迟初始化就会被忽略，即便你在`<beans/>`元素声明中设置`default-lazy-init`属性为`true`，那么`Bean(Factory)PostProcessor`
也会正常被初始化。

### Example: the Class name substitution PropertyPlaceholderConfigurer
从使用了标准Java `Properties`格式的bean定义的分离的文件中，你可以使用`PropertyPlaceholderConfigurer`来具体化属性值。这么做允许部署应用程序来自定义指定的环境属性，比如数据库的连接URL和密码，不会有修改容器的主XML定义文件或其它文件的复杂性和风险。

考虑一下下面这个基于 XML 的配置元数据代码片段，这里的 `DataSource`就使用了占位符来定义。这个示例展示了从`Properties`文件中配置属性的方法。在运行时，`PropertyPlaceholderConfigurer`就会用于元数据并为数据源替换一些属性。指定替换的值作为`${property-name}`形式中的占位符，这里应用了 Ant/log4j/JSP EL 的风格。
```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```
而真正的值是来自于标准的 Java Properties 格式的文件：
>jdbc.driverClassName=org.hsqldb.jdbcDriver   
>jdbc.url=jdbc:hsqldb:hsql://production:9002  
>jdbc.username=sa  
>jdbc.password=root

因此，字符串${jdbc.username}在运行时会被值’sa’替换，对于其它占位符来说也是相同的，匹配到了属性文件中的键就会用其值替换占位符。`PropertyPlaceholderConfigurer`在bean定义的属性中检查占位符。此外，对占位符可以自定义前缀和后缀。

使用 Spring 2.5 引入的`context`命名空间，也可以使用专用的配置元素来配置属性占位符。在`location`属性中，可以提供一个或多个以逗号分隔的列表。
```
<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
```
`PropertyPlaceholderConfigurer`不仅仅查看在`Properties`文件中指定的属性。默认情况下，如果它不能在指定的属性文件中找到属性，它也会检查Java`System`属性。
你可以通过设置`systemPropertiesMode`属性，使用下面整数的三者之一来自定义这种行为：
- never(0)：从不检查系统属性
- fallback(1)：如果没有在指定的属性文件中解析到属性，那么就检查系统属性。这是默认的情况。
- override(2)：在检查指定的属性文件之前，首先去检查系统属性。这就允许系统属性覆盖其它任意的属性资源。

查看`PropertyPlaceholderConfigurer`的JavaDoc文档来获取更多信息。

>你可以使用`PropertyPlaceholderConfigurer`来替换类名，在运行时，当你不得不去选择一个特定的实现类时，这是很有用的。比如：

```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <value>classpath:com/foo/strategy.properties</value>
    </property>
    <property name="properties">
        <value>custom.strategy.class=com.foo.DefaultStrategy</value>
    </property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果类在运行时不能解析成一个有效的类，那么在即将创建时，bean 的解析就失败了，这是`ApplicationContext`在对非延迟初始化bean的`preInstantiateSingletons()`阶段发生的。

### Example: the PropertyOverrideConfigurer
`PropertyOverrideConfigurer`，另外一种bean工厂后置处理器，类似于`PropertyPlaceholderConfigurer`，但不像后者，对于所有bean的属性，原始定义可以有默认值或没有值。如果一个`Properties`覆盖文件没有特定bean的属性配置项，那么就会使用默认的上下文定义。

注意，bean定义是不知道被覆盖的，所以从XML定义文件中不能立即明显反应覆盖配置被使用中。在多个`PropertyOverrideConfigurer`实例的情况下，为相同 bean 的属性定义不同的值，那么最后一个有效，源于它的覆盖机制。

属性文件配置行像这种格式：
```
beanName.property=value
```
例如：
```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```
这个示例文件可以用于包含了dataSource bean的容器，它有driver和url属性。

复合属性名也是支持的，除了最终的属性被覆盖，只要路径中的每个组件都是非空的（假设由构造方法初始化）。在这个例子中...
```
foo.fred.bob.sammy=123
```
`foo`bean的`fred`属性的`bob`属性的`sammy`属性的值设置为标量123。

>指定的覆盖值通常是文字值；它们不会被转换成bean的引用。当XML中的bean定义的原始值指定了 bean 引用时，这个约定也适用。

使用 Spring 2.5 引入的`context`命名空间，可以使用专用的配置元素来配置属性覆盖：
```
<context:property-override location="classpath:override.properties"/>
```

### 6.8.3 Customizing instantiation logic with a FactoryBean

实现了`org.springframework.beans.factory.FactoryBean`接口的对象是它们自己的工厂。

`FactoryBean`接口就是Spring IoC容器实例化逻辑的可插拔点。如果你的初始化代码很复杂，那么相对于（潜在地）大量详细的 XML 而言，最好是使用 Java 语言来表达。你可以创建自己的`FactoryBean`，在类中编写复杂的初始化代码，之后将你自定义的`FactoryBean`插入到容器中。
`FactoryBean`接口提供下面三个方法：
- `Object getObject()`：返回工厂创建对象的实例。这个实例可能被共享，那就是看这个工厂返回的是单例还是原型实例了。
- `boolean isSingleton()`：如果 `FactoryBean` 返回单例的实例，那么该方法返回`true`，否则就返回`false`。
- `Class getObjectType()`：返回由`getObject()`方法返回的对象类型，或者事先不知道类型时返回`null`。

`FactoryBean`的概念和接口被用于 Spring Framework 中的很多地方；随 Spring 发行，有超过50个`FactoryBean`接口的实现类。

当你需要向容器请求一个真实的`FactoryBean`实例，而不是它生产的 bean，调用`ApplicationContext`的`getBean()`方法时在bean的id之前加连字符`（&）`。所以对于一个给定 id 为 `myBean`的`FactoryBean`，调用容器的`getBean("myBean")`方法返回的是`FactoryBean`的产品；而调用`getBean("&myBean")`方法则返回`FactoryBean`实例本身。
