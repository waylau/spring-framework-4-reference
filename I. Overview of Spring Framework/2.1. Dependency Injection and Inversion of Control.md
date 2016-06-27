依赖注入和控制反转
========================

Java 应用程序--运行在各个松散的领域,从受限的嵌入式应用程序,到 n 层架构的服务端企业级应用程序--通常由来自应用适当的对象进行组合合作。因此,对象在应用程序中是彼此依赖。

尽管 Java 平台提供了丰富的应用程序开发功能,但它缺乏来组织基本构建块成为一个完整的方法。这个任务留给了架构师和开发人员。虽然您可以使用设计模式,例如 Factory, Abstract Factory, Builder, Decorator, 和 Service Locator 来组合各种类和对象实例构成应用程序,这些模式是:给出一个最佳实践的名字,描述什么模式,哪里需要应用它,它要解决什么问题,等等。模式是形式化的最佳实践,但你必须在应用程序中*自己来实现*。

Spring Framework 的 *Inversion of Control* (IoC) 组件旨在通过提供正规化的方法来组合不同的组件成为一个完整的可用的应用。 Spring Framework 将规范化的设计模式作为一等的对象,您可以集成到自己的应用程序。许多组织和机构使用 Spring Framework 以这种方式来开发健壮的、可维护的应用程序。

*控制反转和依赖注入的背景*

*“问题是，[他们]哪些方面的控制被反转？”这个问题由 Martin Fowler在他的 Inversion of Control (IoC) [网站](http://martinfowler.com/articles/injection.html)在 2004 年提出。 Fowler 建议重新命名这个说法,使得他更加好理解,并且提出了 Dependency Injection（依赖注入） 这个新的说法。*

**译者注：**Dependency Injection 和 Inversion of Control 其实就是一个东西的两种不同的说法而已。本质上是一回事。Dependency Injection 是一个程序设计模式和架构模型， 一些时候也称作 Inversion of Control，尽管在技术上来讲，Dependency Injection 是一个 Inversion of Control 的特殊实现，Dependency Injection 是指一个对象应用另外一个对象来提供一个特殊的能力，例如：把一个数据库连接以参数的形式传到一个对象的结构方法里面而不是在那个对象内部自行创建一个连接。Inversion of Control 和 Dependency Injection 的基本思想就是把类的依赖从类内部转化到外部以减少依赖。
应用Inversion of Control，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用，传递给它。也可以说，依赖被注入到对象中。所以，Inversion of Control 是，关于一个对象如何获取他所依赖的对象的引用，这个责任的反转。
