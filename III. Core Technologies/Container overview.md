容器总览
====

org.springframework.context.ApplicationContext 代表 Spring IoC 容器，并负责实例化，配置和组装上述 bean 的接口。容器是通过对象实例化，配置，和读取配置元数据汇编得到对象的构建。配置元数据可以是用 XML，Java 注解，或 Java 代码来展示。它可以让你描述组成应用程序的对象和对象间丰富的相互依赖。

Spring ApplicationContext 接口提供了几种即装即用的实现方式。在独立应用中，通常以创建 ClassPathXmlApplicationContext 或FileSystemXmlApplicationContext 的实例。虽然 XML 一直是传统的格式来定义配置元数据，但也可以指示容器使用 Java 注解或代码作为元数据格式，并通过提供少量的XML配置以声明方式启用这些额外的元数据格式的支持。

在大多数应用场合，不需要明确的用户代码来实例化一个 Spring IoC 容器的一个或多个实例。例如，在 web 应用程序中，在应用程序的 web.xml文件中一个简单的样板网站的 XML 描述符通常就足够了（见[第6.15.4，“便捷的 ApplicationContext 实例化 Web 应用程序”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#context-create)）。如果您使用的是基于Eclipse的 [Spring Tool Suite](https://spring.io/tools/sts)开发环境，该样板配置可以很容易地用点击几下鼠标或键盘创建。

下面的图展示了 Spring 是如何工作的高级视图。您的应用程序的类是通过配置元数据来结合的，以便 ApplicationContext 需要创建和初始化后，你有一个完全配置和可执行的系统或应用程序。

Figure 6.1. The Spring IoC container

![](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/container-magic.png)

### 配置元数据

如上述图所示，Spring IoC 容器使用 配置元数据（ configuration metadata）;这个配置元数据代表了应用程序开发人员告诉 Spring 容器在应用程序中如何来实例化，配置和组装对象。

传统的配置元数据是一个简单而直观的 XML 格式，这是大多数本章用来传达关键概念和 Spring IoC 容器的功能。

*基于 XML 的元数据并不是配置元数据的唯一允许的形式。 Spring IoC容器本身是完全从此配置元数据实际写入格式脱钩。现在，许多开发商选择适合自己的 Spring 应用程序的[基于 Java 的配置](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-java)。*

更多其他格式的元数据见：

* [基于注解的配置](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-annotation-config)：Spring 2.5 的推出了基于注解的配置元数据支持。
* [基于Java的配置](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-java)：Spring3.0 开始，由 Spring JavaConfig 项目提供了很多功能成为核心 Spring 框架的一部分。因此，你可以通过使用Java，而不是 XML 文件中定义外部 bean 到你的应用程序类。要使用这些新功能，请参阅 @Configuration，@Bean，@Import 和 @DependsOn 注解。

Spring 配置至少一个，通常不止一个 bean 来由容器来管理。基于 XML 的配置元数据显示这些 bean 配置的`<bean/>` 包含于顶层元素`<beans/>`元素。 Java 配置通常使用 @Configuration 类中 @Bean 注解的方法。

这些 bean 定义对应于构成应用程序的实际对象。通常，您定义服务层对象，数据访问对象（DAO），展示对象，如 Struts Action 的情况下，基础设施的对象，如 Hibernate 的 SessionFactories，JMS Queues，等等。通常一个不配置细粒度域对象在容器中，因为它通常负责 DAO 和业务逻辑来创建和负载域对象。但是，你可以使用 Spring 和 AspectJ的集成配置在 IoC 容器的控制之外创建的对象。请参阅使用 AspectJ 在 Spring 中进行依赖关系注入域对象。

以下示例显示基于 XML 的配置元数据的基本结构：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <bean id="..." class="...">
	        <!-- collaborators and configuration for this bean go here -->
	    </bean>
	
	    <bean id="..." class="...">
	        <!-- collaborators and configuration for this bean go here -->
	    </bean>
	
	    <!-- more bean definitions go here -->
	
	</beans>

id 属性是一个字符串，唯一识别一个独立的 bean 定义。class 属性定义了 bean 的类型，并使用完整的类名。 id 属性的值是指协作对象。将XML 用于参照协作对象未在本例中示出;请参阅 [依赖](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-dependencies)以获取更多信息。

### 实例化容器

实例化 Spring IoC 容器是直截了当的。提供给 ApplicationContext构造器的路径就是实际的资源字符串，使容器装入从各种外部资源的配置元数据，如本地文件系统， Java CLASSPATH，等等。

	ApplicationContext context =
	    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});

当你了解 Spring 的 IoC 容器，你可能想知道更多关于 Spring 的Resource 抽象，如[第7章，资源](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#resources)，它提供了一种方便的从一个 URI 语法定义的位置读取一个InputStream 描述。具体地，资源路被用作在[第7.7节，“应用环境和资源的路径”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#resources-app-ctx)中所述构建的应用程序的上下文。

下面的例子显示了服务层对象（services.xml中）配置文件：
	
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <!-- services -->
	
	    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
	        <property name="accountDao" ref="accountDao"/>
	        <property name="itemDao" ref="itemDao"/>
	        <!-- additional collaborators and configuration for this bean go here -->
	    </bean>
	
	    <!-- more bean definitions for services go here -->
	
	</beans>

下面的例子显示了数据访问对象 daos.xml 文件：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <bean id="accountDao"
	        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
	        <!-- additional collaborators and configuration for this bean go here -->
	    </bean>
	
	    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
	        <!-- additional collaborators and configuration for this bean go here -->
	    </bean>
	
	    <!-- more bean definitions for data access objects go here -->
	
	</beans>

在上面的例子中，服务层由类 PetStoreServiceImpl，以及类型的两个数据访问对象 JpaAccountDao 和 JpaItemDao（基于JPA对象/关系映射标准）组成。property name 元素是指 JavaBean 属性的名称，而 ref 元素引用另一个 bean 定义的名称。 id 和 ref 元素之间的这种联系表达了合作对象之间的依赖关系。对于配置对象的依赖关系的详细信息，请参阅[依赖](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-dependencies)。

#### 撰写基于XML的配置元数据

它可以让 bean 定义跨越多个 XML 文件,这样做非常有用。通常，每个单独的 XML 配置文件代表你的架构一个逻辑层或模块。

您可以使用应用程序上下文构造从所有这些 XML 片段加载 bean 定义。这个构造函数的多个 Resource 位置，作为上一节中被证明。另外，使用  `<import/>`元素的一个或多个出现，从另一个或多个文件加载 bean 定义。 例如：
	
	<beans>
	    <import resource="services.xml"/>
	    <import resource="resources/messageSource.xml"/>
	    <import resource="/resources/themeSource.xml"/>
	
	    <bean id="bean1" class="..."/>
	    <bean id="bean2" class="..."/>
	</beans>

services.xml，messageSource.xml 及 themeSource.xml 在前面的例子中，外部 bean 定义是由三个文件加载而来。所有位置路径是相对于导入文件的，因此 services.xml 是必须和导入文件在同一目录或类路径中的位置，而 messageSource.xml 和 themeSource.xml 来必须在导入文件的 resources 以下位置。正如你所看到的，前面的斜线被忽略，但考虑到这些路径是相对的，它更好的形式是不使用斜线。该文件的内容被导入，包括顶级`<beans />`元素，必须根据  Spring Schema 是有效的XML bean 定义。

*这是可能的，但不推荐，引用在使用相对“../”的路径的父目录中的文件。这样将创建一个文件，该文件是当前应用程序之外的依赖。特别是，该引用不推荐“classpath：” URL（例如，“classpath:../services.xml”），在运行时解决过程中选择了“就近”的类路径的根，然后查找到它的父目录。类路径配置的变化可能导致不同的，不正确的目录的选择。
您可以随时使用完全合格的资源位置，而不是相对路径：例如，file:C:/config/services.xml" 或 "classpath:/config/services.xml"。但是，要知道，你这是是在耦合应用程序的配置到特定的绝对位置。通常优选间接的方式应对这种绝对路径，例如，通过“${…​}”在运行时解决了对JVM系统属性占位符。*

### 使用容器

ApplicationContext 是能够保持 bean 定义以及相互依赖关系的高级工厂接口。使用方法 T getBean(String name, Class<T> requiredType)就可以取得 bean 的实例。

ApplicationContext 中可以读取 bean 定义并访问它们，如下所示：

	// create and configure beans
	ApplicationContext context =
	    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
	
	// retrieve configured instance
	PetStoreService service = context.getBean("petStore", PetStoreService.class);
	
	// use configured instance
	List<String> userList = service.getUsernameList();

您可以使用 getBean() 方法来获取 bean 的实例。 ApplicationContext 接口有一些其他的方法来获取 bean，但理想的应用程序代码不应该使用它们。事实上，你的应用程序代码不应该调用的getBean() 方法，因此在 Spring 的 API 没有依赖性的。例如，Spring如何与Web框架的集成提供了依赖注入各种Web框架类，如控制器和 JSF 管理的bean。

