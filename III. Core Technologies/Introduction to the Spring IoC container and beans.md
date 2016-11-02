介绍  Spring IoC 容器和 bean
====

 
本章涵盖了 Spring Framework 中控制反转原则（IoC）的实现。IoC 也就是被大家所熟知的依赖注入（DI）。他是一个对象定义他的依赖的一个过程，所谓依赖，就是和它一起工作的对象，这个过程只能通过构造函数的参数，工厂方法的参数，或者已经被构造或者从工厂方法返回的对象的 setter 方法设置其属性来实现。容器先创建bean，然后再注入这些依赖。然后容器在创建Bean时注入这些依赖。这个过程从根本上来讲是反向的，由于bean自己控制实例，或者直接通过类的结构，类似于 Service Locator 模式来定位他的依赖。

org.springframework.beans 和 org.springframework.context 包是 Spring Framework 的 IoC 容器的根本。BeanFactory 接口提供了一种更先进的配置机制来管理任意类型的对象。ApplicationContext 是 BeanFactory 的一个子接口。ApplicationContext 使得和 Spring  的AOP 功能集成变得更简单；添加了信息资源处理（国际化中使用），事件发布；还添加了应用程序层的特殊上下文 ，如用于 web 应用程序的 WebApplicationContext。


简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext 添加了更多企业应用功能。ApplicationContext完整扩展了BeanFactory, 这些内容在介绍Spring的IoC容器的章节里面会专门讲到。更多使用 BeanFactory 请参阅
[章节 6.16, “The BeanFactory”](../III. Core Technologies/6.16. Additional Capabilities of the ApplicationContex.md).


在Spring中，由Spring IoC容器管理的，构成你的程序骨架的这些对象叫做bean。
bean对象是指经过IoC容器实例化，组装和管理的对象。此外，bean 就是你应用程序中诸多对象之一。bean 和 bean 的依赖被容器所使用的配置元数据所反射。
