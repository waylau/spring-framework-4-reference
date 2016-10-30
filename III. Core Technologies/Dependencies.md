Dependencies
====

一般情况下企业应用不会只有一个对象（或者是Spring Bean）。甚至最简单的应用都要多个对象来协同工作来让终端用户看到一个完整的应用的。下一部分将解释开发者如何从仅仅定义单独的Bean，到让这些Bean在一个应用中协同工作。

### Dependency Injection

*依赖注入*是一个让对象只通过构造参数，工厂方法的参数或者配置的属性来定义他们的依赖的过程。这些依赖也是对象所需要协同工作的对象。容器会在创建Bean的时候注入这些依赖。整个过程完全反转了由Bean自己控制实例化或者引用依赖，所以这个过程也称之为*控制反转*。

当使用了依赖注入的准则以后，会更易于管理和解耦对象之间的依赖，使得代码更加的简单。对象不再关注依赖，也不需要知道依赖类的位置。这样的话，开发者的类更加易于测试，尤其是当开发者的依赖是接口或者抽象类的情况，开发者可以轻易在单元测试中mock对象。

依赖注入主要使用两种方式，一种是基于构造函数的注入，另一种的基于Setter方法的依赖注入。

### Constructor-based dependency Injection

基于构造函数的依赖注入是由IoC容器来调用类的构造函数，构造函数的参数代表这个Bean所依赖的对象。跟调用带参数的静态工厂方法基本一样。下面的例子展示了一个类通过构造函数来实现依赖注入的。需要注意的是，这个类没有任何*特殊*的地方，只是一个简单的，不依赖于任何容器特殊接口，基类或者注解的普通类。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

**构造函数的参数解析**

构造函数的参数解析是通过参数的类型来匹配的。如果在Bean的构造函数参数不存在歧义，那么构造器参数的顺序也就是就是这些参数实例化以及装载的顺序。参考如下代码：

```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }

}
```

假设`Bar`和`Baz`在继承层次上不相关，也没有什么歧义的话，下面的配置完全可以工作正常，开发者不需要再去`<constructor-arg>`元素中指定构造函数参数的索引或类型信息。

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```

当引用另一个Bean的时候，如果类型确定的话，匹配会工作正常（如上面的例子）.当使用简单的类型的时候，比如说`<value>true</value>`，Spring IoC容器是无法判断值的类型的，所以是无法匹配的。考虑代码如下：

```
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }

}
```

在上面代码这种情况下，容器可以通过使用构造函数参数的`type`属性来实现简单类型的匹配。比如：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

或者使用`index`属性来指定构造参数的位置，比如：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

这个索引也同时是为了解决构造函数中有多个相同类型的参数无法精确匹配的问题。需要注意的是，索引是基于0开始的。
开发者也可以通过参数的名称来去除二义性。

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

需要注意的是,做这项工作的代码必须启用了调试标记编译,这样Spring才可以从构造函数查找参数名称。开发者也可以使用`@ConstructorProperties`注解来显式声明构造函数的名称，比如如下代码：

```
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }

}
```

### Setter-based dependency injection

基于Setter函数的依赖注入则是容器会调用Bean的无参构造函数，或者无参数的工厂方法，然后再来调用Setter方法来实现的依赖注入。

下面的例子展示了使用Setter方法进行的依赖注入，下面的类对象只是简单的POJO对象，不依赖于任何Spring的特殊的接口，基类或者注解。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

`ApplicationContext`所管理Bean对于基于构造函数的依赖注入，或者基于Setter方式的依赖注入都是支持的。同时也支持使用Setter方式在通过构造函数注入依赖之后再次注入依赖。开发者在`BeanDefinition`中可以使用`PropertyEditor`实例来自由选择注入的方式。然而，大多数的开发者并不直接使用这些类，而是跟喜欢XML形式的`bean`定义，或者基于注解的组件（比如使用`@Component`，`@Controller`等）或者在配置了`@Configuration`的类上面使用`@Bean`的方法。

> **基于构造函数还是基于Setter方法？**
因为开发者可以混用两者，所以通常比较好的方式是通过构造函数注入*必要的依赖*通过Setter方式来注入一些*可选的依赖*。其中，在Setter方法上面的`@Required`注解可用来构造必要的依赖。
Spring队伍推荐基于构造函数的注入，因为这种方式会促使开发者将组件开发成*不可变对象*而且确保了注入的依赖不为`null`。而且，基于构造函数的注入的组件被客户端调用的时候也是完全构造好的。当然，从另一方面来说，过多的构造函数参数也是非常差的代码方式，这种方式说明类貌似有了太多的功能，最好重构将不同职能分离。
基于Setter的注入只是用于可选的依赖，但是也最好配置一些合理的默认值。否则，需要对代码的依赖进行非NULL的检查了。基于Setter方法的注入有一个便利之处在于这种方式的注入是可以进行重配置和重新注入的。
依赖注入的两种风格适合大多数的情况，但是有时使用第三方的库的时候，开发者可能并没有源码，而第三方的代码也没有setter方法，那么就只能使用基于构造函数的依赖注入了。

### Dependency resolution process

容器对Bean的解析如下：

* 创建并根据描述的元数据来实例化`ApplicationContext`。配置元数据可以通过XML， Java 代码，或者注解。
* 每一个Bean的依赖通过构造函数参数或者属性或者静态工厂方法的参数等来表示。这些依赖会在Bean创建的的时候注入和装载。
* 每一个属性或者构造函数的参数都是实际定义的值或者引用容器中其他的Bean。
* 每一个属性或者构造参数可以根据其指定的类型转换而成。Spring也可以将String转成默认的Java内在的类型，比如`int`,`long`,`String`,`boolean`等。

Spring容器会在容器创建的时候针对每一个Bean进行校验。然而，Bean的属性在Bean没有真正创建的时候是不会配置进去的。单例类型的Bean是容器创建的时候配置成预实例状态的。Bean的`Scope`在后续有介绍。其他的Bean都只有在请求的时候，才会创建。显然创建Bean对象会有一个依赖的图。这个图表示Bean之间的依赖关系，容器根据此来决定创建和配置Bean的顺序。

> **循环依赖**
如果开发者主要使用基于构造函数的依赖注入，那么很有可能出现一个循环依赖的场景。
比如说：类A在构造函数中依赖于类B的实例，而类B的构造函数依赖类A的实例。如果你这么配置类A和类B相互注入的话，Spring IoC容器会发现这个运行时的循环依赖，并且抛出`BeanCurrentlyInCreationException`。
开发者可以通过使用Setter方法来配置依赖注入，这样可以解决这个问题。或者就不使用基于构造函数的依赖注入，仅仅使用基于Setter方法的依赖注入。换言之，尽管不推荐，但是开发者可以将循环依赖配置为基于Setter方法的依赖注入。

开发者可以相信Spring能正确处理Bean。Spring能够在加载的过程中发现配置的问题，比如引用到不存在的Bean或者是循环依赖。Spring会尽可能晚的在Bean创建的时候装载属性或者解析依赖。这也意味着Spring容器加载正确后会在Bean注入依赖出错的时候抛出异常。比如，Bean抛出缺少属性或者属性不合法。这延迟的解析也是为什么`ApplicationContext`的实现会令单例Bean处于预实例化状态。这样，通过`ApplicationContext`的创建，可以在真正使用Bean之前消耗一些内存代价发现配置的问题。开发者也可以覆盖默认的行为让单例Bean延迟加载，而不是处于预实例化状态。
如果不存在循环依赖的话，Bean所引用的依赖会优先完全构造依赖的。举例来说，如果Bean A依赖于Bean B，那么Spring IoC容器会先配置Bean B，然后调用Bean A的Setter方法来构造Bean A。换言之，Bean先会实例化，然后注入依赖，然后才是相关的生命周期方法的调用。

### Examples of dependency injection

下面的例子会使用基于XML配置的元数据，然后使用Setter方式进行依赖注入。代码如下：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }

}
```

在上面的例子当中，Setter方法的声明和XML文件中相一致，下面的例子是基于构造函数的依赖注入

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }

}
```

在Bean定义之中的构造函数参数就是用来构造`ExampleBean`的依赖。

下面的例子，是通过静态的工厂方法来返回Bean实例的。

```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }

}
```

工厂方法的参数，也是通过 `<constructor-arg/>`标签来指定的，和基于构造函数的依赖注入是一致的。之前有提到过，返回的类型不需要跟`exampleBean`中的`class`属性一致的，`class`指定的是包含工厂方法的类。当然了，上面的例子是一致的。使用`factory-bean`的实例工厂方法构造Bean的，这里就不多描述了。

### 7.4.2 Dependencies and configuration in detail

如前文所述，开发者可以通过定义Bean的依赖的来引用其他的Bean或者是一些值的，Spring基于XML的配置元数据通过支持一些子元素`<property/>`以及`<constructor-arg/>`来达到这一目的。

### Straight values (primitives, Strings, and so on)

元素`<property/>`有`value`属性来对人友好易读的形式配置一个属性或者构造参数。Spring的便利之处就是用来将这些字符串的值转换成指定的类型。

```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的例子使用的p命名空间，是更为简单的XML配置。

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

上面的XML更为的简洁，但是因为属性的类型是在运行时确定的，而非设计时确定的，所以除非使用[IntelliJ IDEA](http://www.jetbrains.com/idea/)或者[Spring Tool Suite](https://spring.io/tools/sts)这些工具才能在定义Bean的时候自动完成属性配置。当然很推荐使用这些IDE。

开发者也可以定义一个`java.util.Properties`实例，比如：

```
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring的容器会将`<value/>`里面的文本通过使用JavaBean的`PropertyEditor`机制转换成一个`java.util.Properties`实例。这也是一个捷径，也是一些Spring团队更喜欢使用嵌套的`<value/>`元素而不是`value`属性风格。

**idref元素**
`idref`元素是一种简单的提前校验错误的方式，通过id来关联容器中的其他的Bean的方式。

```
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean" />
    </property>
</bean>
```
上述的Bean的定义在运行时，和如下定义是完全一致的。

```
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean" />
</bean>
```

第一种方式是更值得提倡的，因为使用了`idref`标签，会是的容器在*部署阶段*就针对Bean进行校验，确保Bean一定存在。而第二个版本的话，是没有任何校验的。只有实际上引用了Bean `client`，在实例化client的时候才会发现。如果`client`是一个`prototype`的Bean，那么类似拼写之类的错误会在容器部署以后很久才能发现。

> `idref`元素的`local`属性在4.0以后的xsd中已经不再支持了，而是使用了`bean`引用。如果更新了版本的话，需要将`idref local`引用都转换成 `idref bean`即可。

### References to other beans (collaborators)

`ref`元素在`<constructor-arg/>`或者`<property/>`中的一个终极标签。开发者可以通过这个标签配置一个Bean来引用另一个Bean。当需要引用一个Bean的时候，被引用的Bean会先实例化，然后配置属性，也就是引用的依赖。如果该Bean是单例Bean的话，那么该Bean会早由容器初始化。最终的引用另一个对象的所有引用。Bean的范围以及校验取决于开发者是否通过`bean`,`local`,`parent`这些属性来指定对象的id或者name属性。

通过指定Bean的`bean`属性中的`<ref/>`来指定依赖是最常见的一种方式，可以引用容器或者父容器中的Bean，无论是否在同一个XML文件定义都可以引用。其中`bean`属性中的值可以和其他引用Bean中的`id`属性一致，或者和其中的一个`name`属性一致的。

```
<ref bean="someBean"/>
```

通过指定Bean的`parent`属性会创建一个引用到当前容器的父容器之中。`parent`属性的值可以跟跟目标Bean的`id`属性一致，或者和目标Bean的`name`属性中的一个一致，目标Bean必须是当前引用目标Bean容器的父容器。开发者一般只有在存在层次化容器，并且希望通过代理来包裹父容器中一个存在的Bean的时候才会用到这个属性。

```
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> 与`idref`标签一样，`ref`元素中的`local`标签在xsd 4.0以后已经不再支持了，开发者可以通过将已存在的`ref local`改为`ref bean`来完成更新Spring。

### Inner beans

定义在`<bean/>`元素的`<property/>`或者`<constructor-arg/>`元素之内的Bean叫做*内部Bean*

```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部Bean的定义是不需要指定id或者名字的。如果指定了，容器也不会用之作为分别Bean的区分标识。容器同时也会无视内部Bean的`scope`标签：内部Bean *总是* 匿名的，而且 *总是* 随着外部的Bean同时创建的。开发者是无法将内部的Bean注入到外部Bean以外的其他Bean的。

当然，也有可能从指定范围接收到破坏性回调。比如：一个请求范围的内部Bean包含了一个单例的Bean，那么内部Bean实例会绑定到包含的Bean，而包含的Bean允许访问到`request`的`scope`生命周期。这种场景不常见，内部Bean通常只是共享它的外部Bean。


### Collections

在`<list/>`,`<set/>`,`<map/>`和`<props/>`元素中，开发者可以配置Java集合类型`List`,`Set`,`Map`以及`Properties`的属性和参数。

```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

当然，map的key或者value，或者是集合的value都可以配置为下列之中的一些元素：

```
bean | ref | idref | list | set | map | props | value | null
```

**集合合并**
Spring的容器也支持来合并集合。开发者可以定义一个父样式的`<list/>`,`<map/>`,`<set/>`或者`<props/>`，同时有子样式的`<list/>`,`<map/>`,`<set/>`或者`<props/>`继承并且覆盖父集合。也就是说，子集合的值是父元素和子元素集合的合并值。比如下面的例子。

```
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

可以发现，我们在`child`Bean上使用了`merge=true`属性。当`child`Bean由容器初始化且实例化的时候，其实例中包含的`adminEmails`集合就是`child`的`adminEmails`以及`parent`的`adminEmails`集合。如下：

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

`child`的`Properties`集合的值继承了`parent`的`<props/>`，`child`的值也支持重写`parent`的值。

这个合并的行为和`<list/>`,`<map/>`以及`<set/>`之类的集合类型的行为是类似的。`<list/>`的特定的例子中，与`List`集合类型类似，有隐含的`ordered`概念的。所有的父元素里面的值，是在所有孩子元素的值之前的。但是像`Map`,`Set`或者`Properties`的集合类型，是不存在顺序的。

**集合合并的限制**

开发者是不能够合并不同类型的集合的（比如`Map`和`List`合并），如果开发者这么做，会抛出异常。`merge`的属性是必须特指到更低级或者继承者，子节点的定义上。特指`merge`属性到父集合的定义上是冗余的，而且在合并上也没有任何效果。

**强类型集合**
在Java 5以后，开发者可以使用强类型的集合了。也就是，开发者可以声明一个`Collection`类型，然后这个集合只包含`String`元素（举例来说）。如果开发者通过Spring来注入强类型的`Collection`到Bean中，开发者就可以利用Spring的类型转换支持来做到。

```
public class Foo {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当`foo`的属性`accounts`准备注入的时候，`accounts`的泛型信息`Map<String, Float>`就会通过反射拿到。这样，Spring 的类型转换系统能够识别不同的类型，如上面的例子`Float`然后将字符串的值`9.99, 2.75`以及`3.99`转换成对应的`Float`类型。

### Null and empty string values

Spring将会将属性的空参数，直接当成空字符串来处理。下面的基于XML的元数据配置就会将email属性配置为`String`的值为`""`

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的例子和下列JAVA代码是一致的。

```
exampleBean.setEmail("")
```

而`<null/>`元素来处理`null`的值。如下：

```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```
上面的代码和下面的Java代码是一样的效果：

```
exampleBean.setEmail(null)
```

### XML shortcut with the p-namespace

p命名空间令开发者可以使用`bean`的属性，而不用使用嵌套的`<property/>`元素，就能描述开发者想要注入的依赖。

Spring是支持基于XML的格式化的命名空间扩展的。本节讨论的`beans`的配置都是基于XML的，p命名空间并不是定义在XSD文件，而是定义在Spring Core之中的。

下面展示了两种XML片段是同样的解析结果：第一个使用的标准的XML格式，而第二种使用了p命名空间。

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="foo@bar.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="foo@bar.com"/>
</beans>
```
上面的例子在Bean中展示了email属性的定义。这种定义告知Spring这是一个属性的声明。如前面所描述，p命名空间并没有标准模式定义，所以你可以配置属性的名字为依赖名字。

下面的例子包括了2个Bean定义，引用了另外的Bean：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

从上述的例子中可以看出，`john-modern`不止包含一个属性，也同时使用了特殊的格式来声明一个引用指向另一个Bean。第一个Bean定义使用的是`<property name="spouse" ref="jane"/>`来创建的Bean引用到另外一个Bean，而第二个Bean的定义使用了`p:spouse-ref="jane"`来作为一个指向Bean的引用。在这个例子中`spouse`是属性的名字，而`-ref`部分表名这个依赖不是直接的类型，而是引用另一个Bean。

> p命名空间并不同标准的XML格式一样灵活。比如，声明属性的引用可能和一些以`Ref`结尾的属性相冲突，而标准的XML格式就不会。Spring团队推荐开发者能够和团队商量一下，要使用哪一种方式，而不要同时使用3种方法。


### XML shortcut with the c-namespace

与p命名空间类似，c命名空间是在Spring 3.1首次引入的，c命名空间允许内联的属性来配置构造参数而不用使用`constructor-arg`元素。
下面就是一个使用了c命名空间的例子：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>

    <!-- traditional declaration -->
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
        <constructor-arg value="foo@bar.com"/>
    </bean>

    <!-- c-namespace declaration -->
    <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

</beans>
```

`c:`命名空间使用了和`p:`命名空间相类似的方式（使用了`-ref`来配置引用）。而且，同样的，c命名空间也不是定义在XSD的模式之中（但是在Spring Core之中）。

在少数的例子之中，构造函数的参数名字并不可用（通常，如果字节码没有debug信息的编译），开发者可以使用下面的例子：

```
<!-- c-namespace index declaration -->
<bean id="foo" class="x.y.Foo" c:_0-ref="bar" c:_1-ref="baz"/>
```

> 根据XML语法，索引的概念的存在要求使用`_`作为XML属性名字不能以数字开始。

实际上，构造函数的解析机制在匹配参数是很高效的，除非必要，Spring团队推荐在配置中使用命名空间。

### Compound property names

开发者可以在配置属性的时候配置混合的属性，只要所有的组件路径（除了最后一个属性名字）不能为`null`。
参考如下的定义。

```
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

`foo`有一个`fred`的属性，而其中`fred`属性有一个`bob`属性，而`bob`属性之中有一个`sammy`属性，那么最后这个`sammy`属性会配置为`123`。想要上述的配置能够生效，`fred`属性需要有一个`bob`属性且在`fred`构造只是构造之后不为`null`，否则会抛出`NullPointerException`。

### 7.4.3 Using depends-on

如果一个Bean是另一个Bean的依赖的话，通常来说这个Bean也就是另一个Bean的属性之一。多数情况下，开发者可以在配置XML元数据的时候使用`<ref/>`标签。然而，有时Bean之间的依赖关系不是直接关联的。比如：需要调用类的静态实例化器来触发，类似数据库驱动注册。`depends-on`属性会使明确的强迫依赖的Bean在引用之前就会初始化。下面的例子使用`depends-on`属性来让表示单例Bean上的依赖的。

```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

如果想要依赖多个Bean，可以提供多个名字作为`depends-on`的值，以逗号，空格，或者分号分割，如下：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

> Bean中的`depends-on`属性可以同时指定一个初始化时间的依赖以及一个相应的销毁时依赖（单例Bean情况）。独立的定义了`depends-on`属性的Bean会优先销毁，优于被`depends-on`的Bean来销毁，这样`depends-on`可以控制销毁的顺序。

### 7.4.4 Lazy-initialized beans

默认情况下，`ApplicationContext`会在实例化的过程中创建和配置所有的单例Bean。总的来说，这个预初始化是很不错的。因为这样能及时发现环境上的一些配置错误，而不是系统运行了很久之后才发现。如果这个行为不是迫切需要的，开发者可以通过将Bean标记为延迟加载就能阻止这个预初始化。延迟初始化的Bean会通知IoC不要让Bean预初始化而是在被引用的时候才会实例化。
在XML中，可以通过`<bean/>`元素的`lazy-init`属性来控制这个行为。如下：

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当将Bean配置为上面的XML的时候，`ApplicationContext`之中的`lazy`Bean是不会随着`ApplicationContext`的启动而进入到预初始化状态的，而那些非延迟加载的Bean是处于预初始化的状态的。

然而，如果一个延迟加载的类是作为一个单例非延迟加载的Bean的依赖而存在的话，`ApplicationContext`仍然会在`ApplicationContext`启动的时候加载，因为作为单例Bean的依赖，会随着单例Bean的实例化而实例化。
开发者可以通过使用`<beans/>`的`default-lazy-init`属性来在容器层次控制Bean是否延迟初始化，比如：

```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 7.4.5 Autowiring collaborators

Spring容器可以根据Bean之间的依赖关系*自动装配*。开发者可以令Spring通过`ApplicationContext`来来自动解析这些关联。自动的装载有很多的优点：

* 自动装载能够明显的减少指定的属性或者是构造参数。
* 自动装载可以扩展开发者的对象。比如说，如果开发者需要加一个依赖，依赖就能够不需要开发者特别关心更改配置就能够自动满足。这样，自动装载在开发过程中是极度高效的，不用明确的选择装载的依赖会使系统更加的稳定。

当使用基于XML的元数据配置的时候，开发者可以指定自动装配的方式。通过配置`<bean/>`元素的`autowire`属性就可以了。自动装载有如下四种方式，开发者可以指定每个Bean的装载方式，这样Bean就知道如何加载自己的依赖。

|模式|解释|
|----|----|
|no|（默认）不装载。Bean的引用必须通过`ref`元素来指定。对于比较大项目的部署，不建议修改默认的配置，因为特指会加剧控制。在某种程度上来说，默认的形式也说明了系统的结构。|
|byName|通过名字来装配。Spring会查找所有的Bean直到名字和属性相同的一个Bean来进行装载。比如说，如果Bean配置为根据名字来自动装配，它包含了一个属性名字为`master`(也就是包含一个`setMaster(..)`方法)，Spring就会查找名字为`master`的Bean，然后用之装载|
|byType|如果需要自动装配的属性的类型在容器之中存在的话，就会自动装配。如果容器之中存在不止一个类型匹配的话，就会抛出一个重大的异常，说明开发者最好不要使用byType来自动装配那个Bean。如果没有匹配的Bean存在的话，不会抛出异常，只是属性不会配置。|
|构造函数|类似于*byType*的注入，但是应用的构造函数的参数。如果没有一个Bean的类型和构造函数参数的类型一致，那么仍然会抛出一个重大的异常|

通过 *byType* 或者 *构造函数* 的自动装配方式，开发者可以装载数组和强类型集合。在如此的例子之中，所有容器之中的匹配指定类型的Bean会自动装配到Bean上来完成依赖注入。开发者可以自动装配key为`String`的强类型的`Map`。自动装配的`Map`值会包含所有的Bean实例值来匹配指定的类型，`Map`的key会包含关联的Bean的名字。

### Limitations and disadvantages of autowiring

自动装载如果在整个的项目的开发过程中使用，会工作的很好。但是如果不是全局使用，而只是用之来自动装配几个Bean的话，会很容易迷惑开发者。

下面是一些自动装配的劣势和限制

* 精确的`property`以及`constructor-arg`参数配置，会覆盖掉自动装配的配置。开发不能够自动装配所谓的简单属性，比如`Primitive`类型或者字符串。
* 自动装配并有精确装配准确。尽管如上面的表所描述，Spring会尽量小心来避免不必要的错误装配，但是Spring管理的对象关系仍然不如文档描述的那么精确。
* 装配的信息对开发者可见性不好，因为这一切都由Spring容器管理。
* 容器中的可能会存在很多的Bean匹配Setter方法或者构造参数。比如说数组，集合或者Map等。然而依赖却希望仅仅一个匹配的值，含糊的信息是无法解析的。如果没有独一无二的Bean，那么就会抛出异常。

在后面的场景，开发者有如下的选择

* 放弃自动装配有利于精确装配
* 可以通过配置`autowire-candidate`属性为`false`来阻止自动装配
* 通过配置`<bean/>`元素的`primary`属性为`true`来指定一个bean为主要的候选Bean
* 实现更多的基于注解的细粒度的装配配置。


### Excluding a bean from autowiring

在每个Bean的基础之上，开发者可以阻止Bean来自动装配。在基于XML的配置中，可以配置`<bean/>`元素的`autowire-candidate`属性为`false`来做到这一点。容器在读取到这个配置后，会让这个Bean对于自动装配的结构中不可见（包括注解形式的配置比如`@Autowired`）

开发者可以通过模式匹配而不是Bean的名字来限制自动装配的候选者。最上层的`<beans/>`元素会在`default-autowire-candidates`属性中来配置多种模式。比如，限制自动装配候选者的名字以*Repository*结尾，可以配置`*Repository`。如果需要配置多种模式，只需要用逗号分隔开即可。当然Bean中如果配置了`autowire-candidate`的话，这个信息拥有更高的优先级。

上面的这些技术在配置那些不需要自动装配的Bean是很有效的。当然这并不是说这类Bean本身无法自动装配其他的Bean，而是说这些Bean不在作为自动装配依赖的候选了。

### 7.4.6 Method injection

在大多数的应用场景下，大多数的Bean都是单例的。当这个单例的Bean需要和另一个单例的或者非单例的Bean联合使用的时候，开发者只需要配置依赖的Bean为这个Bean的属性即可。但是有时会因为不同的Bean生命周期的不同而产生问题。假设单例的Bean A在每个方法调用中使用了非单例的Bean B。容器只会创建Bean A一次，而只有一个机会来配置属性。那么容器就无法给Bean A每次都提供一个新的Bean B的实例。

一个解决方案就是放弃一些IoC。开发者可以通过实现`ApplicationContextAware`接口令Bean A可以看到`ApplicationContext`，从而通过调用`getBean("B")`来在Bean A 需要新的实例的时候来获取到新的B实例。参考下面的例子：

```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

上面的代码并不是让人十分满意，因为业务的代码已经与Spring框架耦合在了一起。Spring提供了一个稍微高级的点特性方法注入的方式，可以用来处理这种问题。

### Lookup method injection

查找方法注入就是容器一种覆盖容器管理Bean的方法，来返回查找的另一个容器中的Bean的能力。查找方法通常就包含前面场景提到的Bean。Spring框架通过使用CGLIB库生成的字节码来动态生成子类来覆盖父类的方法实现方法注入。

> * 为了让这个动态的子类方案正常，那么Spring容器所需要继承的这个Bean不能是`final`的，而覆盖的方法也不能是`final`的。
* 针对这个类的单元测试因为存在抽象方法，所以必须实现子类来测试
* 组件扫描的所需的具体方法也需要具体类。
* 一个关键的限制在于查找方法与工厂方法是不能协同工作的，尤其是不能和配置类之中的`@Bean`的方法，因为容器不在负责创建实例，而是创建一个运行时的子类。
* 最后，被注入的到方法的对象不能被序列化。

看到前面的代码片段中的`CommandManager`类，我们发现发现Spring容器会动态的覆盖`createCommand()`方法。`CommandManager`类不在拥有任何的Spring依赖，如下：

```
package fiona.apple;

// no more Spring imports!

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

在包含需要注入的方法的客户端类当中，注入的方法需要有如下的函数签名 

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法为抽象，那么动态生成的子类会实现这个方法。否则，动态生成的子类会覆盖类中的定义的原方法。例如：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="command" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="command"/>
</bean>
```

上面的commandManager在当它需要一个command Bean实例的时候就会调用自己的方法`createCommand()`。开发者一定要谨慎配置`command` Bean的为prototype类型的Bean。如果所需的Bean为单例的，那么这个方法注入返回的将都是同一个实例。

### Arbitrary method replacement

从前面的描述中，我们知道查找方法是有能力来覆盖任何由容器管理的Bean的方法的。开发者最好跳过这一部分，除非一定需要使用这个功能。

通过配置基于XML的配置元数据，开发者可以使用`replaced-method`元素来替换一个存在的方法的实现。考虑如下情况：

```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...

}
```

一个实现了`org.springframework.beans.factory.support.MethodReplacer`接口的类会提供一个新方法的定义。

```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

如果需要覆盖Bean的方法需要配置XML如下：

```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

开发者可以使用更多的`<replaced-method>`中的`<arg=type/>`元素来指定需要覆盖的方法。当需要覆盖的方法存在重载方法时，指定参数才是必须的。为了方便起见，字符串的类型是会匹配如下类型，完全等同于`java.lang.String`。

```
java.lang.String
String
Str
```

因为通常来说参数的个数已经足够区别不同的方法了，这种快捷的写法可以省去很多的代码。