Customing the nature of a bean
===

### Lifecycle callbacks

开发者通过实现Spring的`InitializeingBean`和`DisposableBean`接口，就可以让容器来管理Bean的生命周期。容器会在调用`afterPropertiesSet()`之后和`destroy()`之前会允许Bean在初始化和销毁Bean的时候执行一些操作。

> JSR-250的`@PostConstruct`和`@PreDestroy`注解就是现代Spring应用生命周期回调的最佳实践。使用这些注解意味着Bean不会再耦合在Spring特定的接口上。详细内容，后续将会介绍。
如果开发者不想使用JSR-250的注解，仍然可以考虑使用`init-method`和`destroy-method的`定义来解耦Spring接口。

内部来说，Spring框架使用`BeanPostProcessor`的实现来处理接口的回调，`BeanPostProcessor`能够找到并调用合适的方法。如果开发者需要定制一些Spring并不直接提供的生命周期行为，开发者可以考虑自行实现一个`BeanPostProcessor`。更多的信息可以参考后面的容器扩展点。

除了初始化和销毁回调，Spring管理的对象也实现了`Lifecycle`接口来让管理的对象在容器的生命周期内启动和关闭。

生命周期回调在本节会进行详细描述。

#### Initialization callbacks

`org.springframework.beans.factory.InitializingBean`接口允许Bean在所有的必要的依赖配置配置完成后来执行初始化Bean的操作。`InitializingBean`接口中特指了一个方法：

```
void afterPropertiesSet() throws Exception;
```

Spring团队是建议开发者不要使用`InitializingBean`接口的，因为这样会将代码耦合到Spring的特定接口之上。而通过使用`@PostConstruct`注解或者指定一个POJO的实现方法，会比实现接口要更好。在基于XML的配置元数据上，开发者可以使用`init-method`属性来指定一个没有参数的方法。使用Java配置的开发者可以使用`@Bean`之中的`initMethod`属性，比如如下：

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```
public class ExampleBean {

    public void init() {
        // do some initialization work
    }

}
```

与如下代码一样效果：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }

}
```

但是前一个版本的代码是没有耦合到Spring的。

#### Destruction callbacks

实现了`org.springframework.beans.factory.DisposableBean`接口的Bean就能通让容器通过回调来销毁Bean所引用的资源。`DisposableBean`接口包含了一个方法：

```
void destroy() throws Exception;
```

同InitializingBean相类似，Spring团队仍然不建议开发者来实现`DisposableBean`回调接口，因为这样会将开发者的代码耦合到Spring代码上。换种方式，比如使用`@PreDestroy`注解或者指定一个Bean支持的配置方法，比如在基于XML的配置元数据中，开发者可以在Bean标签上指定`destroy-method`属性。而在基于Java配置中，开发者也可以配置`@Bean`的`destroyMethod`来实现销毁回调。

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }

}
```

上面的代码配置和如下配置是等同的：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements DisposableBean {

    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }

}
```

但是第一段代码是没有耦合到Spring的。

> `<bean/>`标签的`destroy-method`可以被配置为特殊指定的值，来方便让Spring能够自动的检查到`close`或者`shutdown`方法（可以实现`java.lang.AutoCloseable`或者`java.io.Closeable`都会匹配。）这个特殊指定的值可以配置到`<beans/>`的`default-destroy-method`来让所有的Bean实现这个行为。

#### Default initialization and destroy methods

当开发者不使用Spring特有的`InitializingBean`和`DisposableBean`回调接口来实现初始化和销毁方法的时候，开发者通常定义的方法名字都是好似`init()`，`initialize()`或者是`dispose()`等等。那么，可以将这类的方法在项目中标准化，来让所有的开发者都使用一样的名字来确保一致性。

开发者可以配置Spring容器来针对每一个Bean都查找这种名字的初始化和销毁回调函数。也就是说，任何的一个应用开发者，都会在应用的类中使用一个叫`init()`的初始化回调，而不需要在每个Bean中定义`init-method="init"`这中属性。Spring IoC容器会在Bean创建的时候调用那个方法（就如前面描述的标准生命周期一样。）这个特性也将强制开发者为其他的初始化以及销毁回调函数使用同样的名字。

假设开发者的初始化回调方法名字为`init()`而销毁的回调方法为`destroy()`。那么开发者的类就会好似如下的代码：

```
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }

}
```

```
<beans default-init-method="init">

    <bean id="blogService" class="com.foo.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

`<beans/>`标签上面的`default-init-method`属性会让Spring IoC容器识别叫做`init`的方法来作为Bean的初始化回调方法。当Bean创建和装载之后，如果Bean有这么一个方法的话，Spring容器就会在合适的时候调用。

类似的，开发者也可以配置默认销毁回调函数，基于XML的配置就在`<beans/>`标签上面使用`default-destroy-method`属性。

当存在一些Bean的类有了一些回调函数，而和配置的默认回调函数不同的时候，开发者可以通过特指的方式来覆盖掉默认的回调函数。以XML为例，就是通过使用`<bean>`标签的`init-method`和`destroy-method`来覆盖掉`<beans/>`中的配置。

Spring容器会做出如下保证，Bean会在装载了所有的依赖以后，立刻就开始执行初始化回调。这样的话，初始化回调只会在直接的Bean引用装载好后调用，而AOP拦截器还没有应用到Bean上。首先目标Bean会完全初始化好，然后，AOP代理以及其拦截链才能应用。如果目标Bean以及代理是分开定义的，那么开发者的代码甚至可以跳过AOP而直接和引用的Bean交互。因此，在初始化方法中应用拦截器会前后矛盾，因为这样做耦合了目标Bean的生命周期和代理/拦截器，还会因为同Bean直接交互而产生奇怪的现象。

#### Combining lifecycle mechanisms

在Spring 2.5之后，开发者有三种选择来控制Bean的生命周期行为：

* `InitializingBean`和`DisposableBean`回调接口
* 自定义的`init()`以及`destroy`方法
* 使用`@PostConstruct`以及`@PreDestroy`注解

开发者也可以在Bean上联合这些机制一起使用

> 如果Bean配置了多个生命周期机制，而且每个机制配置了不同的方法名字，那么每个配置的方法会按照后面描述的顺序来执行。然而，如果配置了相同的名字，比如说初始化回调为`init()`，在不止一个生命周期机制配置为这个方法的情况下，这个方法只会执行一次。

如果一个Bean配置了多个生命周期机制，并且含有不同的方法名，执行的顺序如下：

* 包含`@PostConstruct`注解的方法
* 在`InitializingBean`接口中的`afterPropertiesSet()`方法
* 自定义的`init()`方法

销毁方法的执行顺序和初始化的执行顺序相同：

* 包含`@PreDestroy`注解的方法
* 在`DisposableBean`接口中的`destroy()`方法
* 自定义的`destroy()`方法

#### Startup and shutdown callbacks

`Lifecycle`接口中为任何有自己生命周期需求的对象定义了一些基本的方法（比如启动和停止一些后台进程）:

```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();

}
```

任何Spring管理的对象都可实现上面的接口。那么当`ApplicationContext`本身收到了启动或者停止的信号时，比如运行时的停止或者重启等场景，`ApplicationContext`会通知到所有上下文中包含的生命周期对象，`ApplicationContext`通过将代理到`LifecycleProcessor`来串联上下文中的`Lifecycle`的实现对象。

```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();

}
```

从上面代码我们可以发现`LifecycleProcessor`是`Lifecycle`接口的扩展。`LifecycleProcessor`增加了另外的两个方法来针对上下文的刷新和关闭做出反应。

> 常规的`org.springframework.context.Lifecycle`接口只是为明确的开始/停止通知提供一个契约，而并不表示在上下文刷新会自动开始。考虑实现`org.springframework.context.SmartLifecycle`接口则可以取代在某个Bean的自动启动过程（包括启动阶段）。同时，停止通知并不能保证在销毁之前出现：在正常的关闭情况下，所有的`Lifecycle`Bean都会在销毁回调准备好之前收到停止通知，然而，在上下文存活期的热刷新或者停止刷新尝试的时候，只会调用销毁方法。

启动和关闭调用是很重要的。如果不同的Bean之间存在`depends-on`的关系的话，被依赖的一方需要更早的启动，而且关闭的更早。然而，有的时候直接的依赖是未知的，而开发者仅仅知道哪一种类型需要更早进行初始化。在这种情况下，`SmartLifecycle`接口定义了另一种选项，就是其父接口`Phased`中的`getPhase()`方法。

```
public interface Phased {

    int getPhase();

}
```

```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);

}
```

当启动时，拥有最低的`phased`的对象优先启动，而当关闭时，是相反的顺序。因此，如果一个对象实现了`SmartLifecycle`然后令其`getPhase()`方法返回了`Integer.MIN_VALUE`的话，就会让该对象最早启动，而最晚销毁。显然，如果`getPhase()`方法返回了`Integer.MAX_VALUE`就说明了该对象会最晚启动，而最早销毁。当考虑到使用`phased`的值得时候，也同时需要了解正常没有实现`SmartLifecycle`的`Lifecycle`对象的默认值，这个值为0。因此，任何负值将标兵对象会在标准组件启动之前启动，在标准组件销毁以后再进行销毁。

`SmartLifecycle`接口也定义了一个`stop`的回调函数。任何实现了`SmartLifecycle`接口的函数都必须在关闭流程完成之后调用回调中的`run()`方法。这样做可以使能异步关闭。而`LifecycleProcessor`的默认实现`DefaultLifecycleProcessor`会等到配置的超时时间之后再调用回调。默认的每一阶段的超时时间为30秒。开发者可以通过定义一个叫做`lifecycleProcessor`的Bean来覆盖默认的生命周期处理器。如果开发者需要配置超时时间，可以通过如下代码进行配置：

```
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

前文提到的，`LifecycleProcessor`接口定义了回调方法来刷新和关闭上下文。关闭的话，如果`stop()`方法已经明确调用了，那么就会驱动关闭的流程，但是如果是上下文正在关闭就不会发生这种情况。而刷新的回调会使能`SmartLifecycle`的另一个特性。当上下文刷新完毕（所有的对象已经实例化并初始化），那么就会调用回调，默认的生命周期处理器会检查每一个`SmartLifecycle`对象的`isAutoStartup()`返回的Bool值。如果为真，对象将会自动启动而不是等待明确的上下文调用，或者调用自己的`start()`方法(不同于上下文刷新，标准的上下文实现是不会自动启动的)。`phased`的值以及`depends-on`关系会决定对象启动和销毁的顺序。

#### Shutting down the Spring IoC container gracefully in non-web applications


> 这一部分只是针对于非Web的应用。Spring的基于web的`ApplicationContext`实现已经有代码在web应用关闭的时候能够自动的的关闭Spring IoC容器。

如果开发者在非web应用环境使用Spring IoC容器的话， 比如，在桌面客户端的环境下，开发者需要在JVM上注册一个关闭的钩子，来确保在关闭Spring IoC容器的时候能够调用相关的销毁方法来释放掉引用的资源。当然，开发者也必须要正确的配置和实现那些销毁回调。

开发者可以在`ConfigurableApplicationContext`接口调用`registerShutdownHook()`来注册销毁的钩子：

```
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {

        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext(
                new String []{"beans.xml"});

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...

    }
}
```

### ApplicationContextAware and BeanNameAware

当`ApplicationContext`在创建实现了`org.springframework.context.ApplicationContextAware`接口的对象时，该对象的实例会包含一个到`ApplicationContext`的引用。

```
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;

}
```

这样Bean就能够通过编程的方式操作和创建`ApplicationContext`了。通过`ApplicationContext`接口，或者通过将引用转换成已知的接口的子类，比如`ConfigurableApplicationContext`就能够提供一些额外的功能。其中的一个用法就是可以通过编程的方式来获取其他的Bean。有的时候这个能力很有用。当然，Spring团队推荐最好不要这样做，因为这样会耦合代码到Spring上，同时也没有遵循IoC的风格。`ApplicationContext`的其它的方法可以提供一些到诸如资源的访问，发布应用事件，或者进入`MessageSource`之类的功能。这些信息在后续的针对`ApplicationContext`的描述中会讲到。

在Spring 2.5版本中，自动装载也是获得`ApplicationContext`的一种方式。传统的构造函数和通过类型的装载方式（前文[依赖](Dependencies.md)中有相关描述）可以通过构造函数或者是setter方法的方式注入，开发者也可以通过注解注入的方式。

当`ApplicationContext`创建了一个实现了`org.springframework.beans.factory.BeanNameAware`接口的类，那么这个类就可以针对其名字进行配置。

```
public interface BeanNameAware {

    void setBeanName(string name) throws BeansException;

}
```

这个回调的调用处于属性配置完以后，但是初始化回调之前。比如`InitializingBean`的`afterPropertiesSet()`方法以及自定义的初始化方法等。

### Other Aware interfaces

除了上面描述的两种Aware接口，Spring还提供了一系列的`Aware`接口来让Bean告诉容器，这些Bean需要一些具体的*基础设施*信息。最重要的一些`Aware`接口都在下面表中进行了描述：

|名字|注入的依赖|
|----|----------|
|`ApplicationContextAware`|声明的`ApplicationContext`|
|`ApplicationEventPlulisherAware`|`ApplicationContext`中的事件发布器|
|`BeanClassLoaderAware`|加载Bean使用的类加载器|
|`BeanFactoryAware`|声明的`BeanFactory`|
|`BeanNameAware`|Bean的名字|
|`BootstrapContextAware`|容器运行的资源适配器`BootstrapContext`，通常仅在JCA环境下有效|
|`LoadTimeWeaverAware`|加载期间处理类定义的weaver|
|`MessageSourceAware`|解析消息的配置策略|
|`NotificationPublisherAware`|Spring JMX通知发布器|
|`PortletConfigAware`|容器当前运行的`PortletConfig`，仅在web下的Spring `ApplicationContext`中可见|
|`PortletContextAware`|容器当前运行的`PortletContext`，仅在web下的Spring `ApplicationContext`中可见|
|`ResourceLoaderAware`|配置的资源加载器|
|`ServletConfigAware`|容器当前运行的`ServletConfig`，仅在web下的Spring `ApplicationContext`中可见|
|`ServletContextAware`|容器当前运行的`ServletContext`，仅在web下的Spring `ApplicationContext`中可见|


再次的声明，上面这些接口的使用是违反IoC原则的，除非必要，最好不要使用。
