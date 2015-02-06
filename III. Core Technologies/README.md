Part III. 核心技术
============================
该部分的参考文档涵盖了Spring Framework中所有绝对不可或缺的技术。

这其中最重要的部分就是Spring Framework中的控制反转（IoC)容器。Spring Framework中IoC容器是紧随着Spring中面向切面编程（AOP)技术的全面应用的来完整实现的。Spring Framework有它自己的一套AOP框架，这套框架从概念上很容易理解，而且成功解决了Java企业级应用中AOP需求80%的核心要素。

同样Spring与AspectJ（目前从功能上来说是最丰富，而且也无疑是Java企业领域最成熟的AOP实现）的集成也涵盖在内。

最后，Spring团队相当推崇采用测试驱动开发（TDD)的方法来进行软件开发，所以Spring对于集成测试的支持也被涵盖在内（与单元测试的最佳实践一起）。Spring团队发现对于IoC的正确使用使得单元测试和集成测试都变得简单了（如类中使用的setter方法和合适的构造方法使得它们在测试中更加方便地连接到一起，而不用设置服务定位注册器等）...专门致力于测试的那一章节也会很有希望地使你认同这一观点。

* [Chapter 5, IoC容器](5. The IoC container.md)
* [Chapter 6, 资源](6. Resources.md)
* [Chapter 7, 验证，数据绑定和类型转换](7. Validation, Data Binding, and Type Conversion.md)
* [Chapter 8, Spring表达式语言（SpEL)](8. Spring Expression Language-SpEL.md)
* [Chapter 9, Spring中的面向切面编程](9. Aspect Oriented Programming with Spring.md)
* [Chapter 10, Spring AOP APIs](10. Spring AOP APIs.md)
* [Chapter 11, 测试](11. Testing.md)