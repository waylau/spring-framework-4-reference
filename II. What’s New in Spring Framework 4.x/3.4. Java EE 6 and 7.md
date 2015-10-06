Java EE 6 和 7
========================

Java EE 6 或以上版本是 Spring4 的底线,与 JPA2.0 和 Servlet3.0规范有着特殊的意义。为了保持与 Google App Engine 和旧的应用程序服务器兼容,Spring4 可以部署在 Servlet2.5 运行环境。但是我们强烈的建议您在 Spring 测试和模拟测试的开发环境中使用 Servlet3.0+。

*如果你是WebSphere 7的用户，一定要安装JPA2.0功能包。在WebLogic 10.3.4或更高版本，安装附带的JPA2.0补丁。这样就可以将这两种服务器变成Spring4兼容的部署环境。*

从长远的观点来看，Spring4.0 现在支持 Java EE 7 级别的适用性规范：尤其是 JMS 2.0, JTA 1.2, JPA 2.1, Bean Validation 1.1 和JSR-236 并发工具类。像往常一样，支持的重点是独立的使用这些规范。例如在 Tomcat 或者独立环境中。但是，当把 Spring 应用部署到 Java EE 7 服务器时它同样适用。

注意，Hibernate 4.3 是 JPA 2.1 的提供者，因此它只支持 Spring4。同样适用用于作为 Bean Validation 1.1 提供者的 Hibernate Validator 5.0。这两个都不支持 Spring3.2。
