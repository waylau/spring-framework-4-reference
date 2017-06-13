Bean definition inheritance
====

bean的定义可以包含很多配置信息，包括构造方法参数，属性值和容器特定的信息，如初始化方法，静态工厂方法名称等。子bean定义从继承父bean定义的配置元数据。子bean可以覆盖或者添加一些它所需要的值。使用父子bean定义可以节省很多配置输入。实际上，这是一种模板形式。

如果你编程式地使用`ApplicationContext`接口，子bean的定义可以通过`ChildBeanDefinition`类来表示。很多用户不使用这个级别的方法，而是在类似于`ClassPathXmlApplicationContext`中声明式地配置bean的定义。当你使用基于XML配置时，你可以在子bean中用`parent`属性，该属性的值用来标识父bean。
```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```
子bean如果没有指定class，它将使用父bean定义的class，但也可以进行重载。在后一种情况中，子bean必须与父bean兼容，也就是说，它必须接受父bean的属性值。

子bean定义从父类继承作用域，构造器参数，属性值，和可以重写的方法，除此之外，还可以增加新的值。你指定的任何作用域，初始化方法，销毁方法，和/或者静态工厂方法设置，都会覆盖相应的父bean设置。

其余的设置总是取自子bean定义：depends on, autowire mode, dependency check, singleton, scope, lazy init。

上面的例子，通过使用`abstract`属性明确地表明这个父bean定义是抽象的。如果，父bean定义没有明确地指出所属的类，那么标记父bean定义为为`abstract`是必须的，如下：
```
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```
这个父bean不能自主实例化，因为它是不完整的，同时它也明确地被标注为`abstract`；像这样， 一个bean定义为`abstract`的，它只能作为一个纯粹的bean模板，为子bean定义，充当父bean定义。尝试独立地使用这样一个`abstract`的父bean，把他作为另一个bean的引用，或者根据这个父bean的id显式调用`getBean()`方法，将会返回一个错误。类似地，容器内部的`preInstantiateSingletons()` 方法，也忽略定义为抽象的bean定义。

>`ApplicationContext`默认会预实例化所有单例bean。因此，如果你打算把一个(父)bean定义仅仅作为模板来使用，同时给它指定了`class`属性，你必须确保设置`abstract`属性为true,否则，应用程序上下文，会(尝试)预实例化这个`abstract`bean。
