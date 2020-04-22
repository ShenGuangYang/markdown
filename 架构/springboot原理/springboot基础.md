# SpringBoot

​	SpringBoot 是为了帮助使用Spring框架使用者能快速高效的构建一个基于spring 以及spring生态体系的应用解决方案。它是基于“约定由于配置”的理念，是服务于spring框架，主要是简化配置文件。



## SpringBootApplication

默认的springboot项目启动类会声明SpringBootApplication注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

}

```

`SpringBootApplication` 是个复合注解，它下面有`SpringBootConfiguration`、

`EnableAutoConfiguration`、`ComponentScan `三个注解



### SpringBootConfiguration

SpringBootConfiguration 是一个复合注解，里面Configuration注解

带有Configuration注解的类表示是一个配置类，扫描里面带有@bean注解的方法，加载到Ioc容器中

### ComponentScan

ComponentScan主要是扫描指定包下带有@Component、@Repository、@Service、@Controller 的标识类，并加载到IoC容器中。



### EnableAutoConfiguration

EnableAutoConfiguration代码如下

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};

}
```

AutoConfigurationPackage代码如下

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```



查看代码，我们可以看到，这两个注解都是使用了@Import注解

Import等同于xml配置文件中`<import resource="...">` ，就是把多个分开的容器配置文件合并到一个容器中。

EnableAutoConfiguration分别导入了AutoConfigurationImportSelector、AutoConfigurationPackages.Registrar两个类。

- `AutoConfigurationImportSelector`实现的`ImportSelector`接口来实现动态注入

- `AutoConfigurationPackages.Registrar`实现`ImportBeanDefinitionRegistrar`接口进行动态注入























