### 6.11 Using JSR 330 Standard Annotations
从 Spring 3.0 开始，Spring 提供了对 JSR-330 标准注解（依赖注入）的支持。这些注解可以和 Spring 注解以相同方式被扫描到。你仅仅需要在类路径下添加相关的 jar 包即可。

如果你使用Maven，那么标准Maven仓库[http://repo1.maven.org/maven2/javax/inject/javax.inject/1/] 中`javax.inject`的artifact是可用的。你仅仅需要在 pom.xml 中添加如下的依赖即可：
```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```
### 6.11.1 Dependency Injection with @Inject and @Named
取代`@Autowired`，`javax.inject.Inject` 还可以像下面这样使用：
```
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
与`@Autowired`一样，可以在类级别，字段级别，方法级别和构造方法参数级别使用`@Inject`。如果你想对被注入的依赖使用限定符名称，你应该按如下方式使用`@Named`注解：
```
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
### 6.11.2 @Named: a standard equivalent to the @Component annotation
取代`@Component`，`javax.inject.Named`还可以像下面这样使用：
```
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
使用`@Component`而不指定组件的名称是很常用的方式。`@Named`可以被用于相同的
情况：
```
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```
当使用`@Named`时，可以使用相同的方式使用Spring注解的组件扫描：
```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```
### 6.11.3 Limitations of the standard approach
当使用标准注解时，了解一些很重要的特性是不可用的是很有必要的，在下面的表格中给出：
表 6.6 Spring注解和标准注解的对比

|Spring | javax.inject.* | javax.inject  限制/ 注释|
|---------|-------------|-------------|
|@Autowired | @Inject | @Inject 没有 'required' 属性|
|@Component | @Named| |
|@Scope( " singleton " )| @Singleton|jsr-330 默认范围和 Spring 的 prototype相似。但是，要保持和 Spring 一般默认值一致，在 Spring 容器中 jsr-330 的 bean 声明默认是 singleton 的。要使用另外的范围，你应该使用 Spring 的@Scope 注解。javax.inject 也供@Scope 注解。不过这仅仅用于创建你自己的注解。|
|@Qualifier | @Named| |
|@Value | - | 不等同|
|@Required|  -  |不等同|
|@Lazy | -  |不等同|
