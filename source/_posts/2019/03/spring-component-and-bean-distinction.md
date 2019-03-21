---
title: Spring中@Component与@Bean的区别
date: 2019-03-21 20:44:24
categories: "java"
tags:
- java
- spring framework
---

# 作用
`@Component`注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。

`@Bean`注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。

## @Bean

`@Bean`与配置类（使用`@Configuration`）一起工作，因此使用在基于配置中。也可用在配置类的方法中。告诉Spring将方法返回的任何内容添加到Spring Context中。

默认情况下，它将使用方法的名称作为`bean`的`id / name`（类似XML配置：`bean id=xxxx`）。另一种方法是，您可以在`@Bean`注释中指定它。

我们明确声明了`bean`。

## @Component
`@Component`用于我们的类，它只有在我们的SpringBoot应用程序启用了组件扫描并且包含了我们的类时才有效。

通过组件扫描，Spring将扫描整个类路径，并将所有`@Component`注释类添加到Spring Context（具有可调整的Filtering）。

我们让Spring发现了bean

# 区别
两个注释的结果是相同的，bean都会被添加到Spring上下文中。但是，有一些问题需要注意。

假设我们有一个需要在多个应用程序中共享的模块，这个模块包含了一些服务，但并非所有应用都需要这些服务。

如果在这些服务类上使用`@Component`并在应用程序中使用组件扫描，我们最终可能会检测到超过必要的bean数量，不需要的Bean也扫描加载了。这时候必须调整组件扫描的过滤或提供即使未使用的bean也可以运行的配置，否则，Spring应用程序上下文将无法启动。

在这种情况下，最好使用`@Bean`注释并仅实例化那些在每个应用程序中单独需要的`bean`。