### How-to指南

本章节将回答一些常见的"我该怎么做"类型的问题，这些问题在我们使用Spring Boot时经常遇到。这绝不是一个详尽的列表，但它覆盖了很多方面。

如果遇到一个特殊的我们没有覆盖的问题，你可能想去查看[stackoverflow.com](http://stackoverflow.com/tags/spring-boot)，看是否有人已经给出了答案;这也是一个很好的提新问题的地方（请使用`spring-boot`标签）。

我们也乐意扩展本章节;如果想添加一个'how-to'，你可以给我们发一个[pull请求](http://github.com/spring-projects/spring-boot/tree/master)。

### Spring Boot应用

### 解决自动配置问题

Spring Boot自动配置总是尝试尽最大努力去做正确的事，但有时候会失败并且很难说出失败原因。

在每个Spring Boot `ApplicationContext`中都存在一个相当有用的`ConditionEvaluationReport`。如果开启`DEBUG`日志输出，你将会看到它。如果你使用`spring-boot-actuator`，则会有一个`autoconfig`的端点，它将以JSON形式渲染该报告。可以使用它调试应用程序，并能查看Spring Boot运行时都添加了哪些特性（及哪些没添加）。

通过查看源码和javadoc可以获取更多问题的答案。以下是一些经验：
- 查找名为`*AutoConfiguration`的类并阅读源码，特别是`@Conditional*`注解，这可以帮你找出它们启用哪些特性及何时启用。
将`--debug`添加到命令行或添加系统属性`-Ddebug`可以在控制台查看日志，该日志会记录你的应用中所有自动配置的决策。在一个运行的Actuator app中，通过查看`autoconfig`端点（`/autoconfig`或等效的JMX）可以获取相同信息。
- 查找是`@ConfigurationProperties`的类（比如[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)）并看下有哪些可用的外部配置选项。`@ConfigurationProperties`类有一个用于充当外部配置前缀的`name`属性，因此`ServerProperties`的值为`prefix="server"`，它的配置属性有`server.port`，`server.address`等。在运行的Actuator应用中可以查看`configprops`端点。
- 查看使用`RelaxedEnvironment`明确地将配置从`Environment`暴露出去。它经常会使用一个前缀。
- 查看`@Value`注解，它直接绑定到`Environment`。相比`RelaxedEnvironment`，这种方式稍微缺乏灵活性，但它也允许松散的绑定，特别是OS环境变量（所以`CAPITALS_AND_UNDERSCORES`是`period.separated`的同义词）。
- 查看`@ConditionalOnExpression`注解，它根据SpEL表达式的结果来开启或关闭特性，通常使用解析自`Environment`的占位符进行计算。

### 启动前自定义Environment或ApplicationContext

每个`SpringApplication`都有`ApplicationListeners`和`ApplicationContextInitializers`，用于自定义上下文（context）或环境(environment)。Spring Boot从`META-INF/spring.factories`下加载很多这样的内部使用的自定义。有很多方法可以注册其他的自定义：

* 以编程方式为每个应用注册自定义，通过在SpringApplication运行前调用它的`addListeners`和`addInitializers`方法来实现。
* 以声明方式为每个应用注册自定义，通过设置`context.initializer.classes`或`context.listener.classes`来实现。
* 以声明方式为所有应用注册自定义，通过添加一个`META-INF/spring.factories`并打包成一个jar文件（该应用将它作为一个库）来实现。

`SpringApplication`会给监听器（即使是在上下文被创建之前就存在的）发送一些特定的`ApplicationEvents`，然后也会注册监听`ApplicationContext`发布的事件的监听器。查看Spring Boot特性章节中的[Section 22.4, “Application events and listeners” ](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-application-events-and-listeners)可以获取一个完整列表。