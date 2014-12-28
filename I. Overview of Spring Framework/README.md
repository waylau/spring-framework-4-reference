Part I. Overview of Spring Framework 总览
========================
Spring Framework 是一个轻量级的解决方案，可以一站式构建企业级应用。然而，Spring 是模块化的，允许你使用的你需要的部分，而不必把其余带进来。您可以使用 IoC 容器，顶层有任何 Web 框架，但你也可以只使用 [Hibernate 集成代码](../IV. Data Access/15.3. Hibernate.md) 或 [JDBC 抽象层](../IV. Data Access/14.1. Introduction to Spring Framework JDBC.md)。Spring Framework 支持声明式事务管理，通过 RMI 或 Web 服务远程访问你的逻辑，并支持各种方式持久化你的数据。它提供了一个全功能的 [MVC 框架](../V. The Web/17.1. Introduction to Spring Web MVC framework.md)，使您能够将 AOP 透明地集成到您的软件。

Spring 的设计是非侵入性的，也就是说你的领域逻辑代码一般对框架本身无依赖性。在你的集成层（如数据访问层），在数据访问技术和 Spring 的库一些依赖将存在。然而，它应该很容易从其他的代码基分离这些依赖。

本文档是 Spring Framework 功能的参考指南。如果你有关于这个文档的任何要求，意见或问题，请发送到[用户邮件列表](https://groups.google.com/forum/#!forum/spring-framework-contrib)。对 Framework 本身的问题应该到 StackOverflow 请问（见<https://spring.io/questions>）

译者注：对本文档的翻译有任何问题，请在<https://github.com/waylau/spring-framework-4-reference/issues>上面提问