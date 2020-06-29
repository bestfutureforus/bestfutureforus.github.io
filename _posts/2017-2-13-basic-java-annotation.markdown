---
layout:     post
title:      "Java基础学习(2)——注解"
subtitle:   ""
date:       2017-02-13 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---


> 注解可以用一个词来描述，就是元数据，即描述数据的数据，是一种代码级别的说明

注解类似于注释，但不同于注释的是，注解不是单纯的对代码功能的说明，而是实现程序功能的组成部分。**一个注解就是一个类**,每次使用一次注解就是创建一个相应的实例对象。在开发Java EE应用时，总是需要导入各种配置文件，这些配置文件需要与Java源代码保持同步，否则在运行时就会出错，使用注解则可以只在一个地方管理和维护信息，其他部分所需信息均自动生成。

## Java内置的标准注解

+ `@Override`，表示当前的方法定义将覆盖超类中的方法。
+ `@SuppressWarnings`，关闭不当编译器警告信息。
+ `@Deprecated`，编译器将对注解为@Deprecated的元素发出警告，注解@Deprecated是不赞成使用的、被弃用的代码。

以上三者我们在平时编写和阅读代码时都会遇到，其中以前两者更为常见。
	
加了注解就相当于为程序加了某种标记，开发工具和其他程序可以用反射来了解你的类和各种元素上有无何种标记，若你有什么标记，就去干相应的事。标记可以在包，类，字段，方法，方法的参数以及局部变量上。

## 元注解
元注解就是在注解类上加的注解

**@Retention**

表示需要在什么级别保存该注解信息

注解的生命周期：
java源文件 ——> class文件 ——> 字节码阶段

@Retention元注解有三种取值(参考Java API枚举类java.lang.annotation.RetentionPolicy)：
	
	 RetentionPolicy.SOURCE //只存在于java源文件中，会被编译器丢弃
	 RetentionPolicy.CLASS//保留到class文件中，但会被jvm丢弃
	 RetentionPolicy.RUNTIME//一直保留到jvm在运行期间，可以通过反射机制获取注解的信息

某一注解对于@Retention的具体取值应视该注解的实际情况而定
举例:
+ @Override应保留到SOURCE阶段
+ @SupperessWarnings保留到RUNTIME阶段
+ @Deprecated保留到RUNTIME阶段

**@Target**

表示该注解可以用于什么地方
@Target元注解的取值有(参考Java API枚举类java.lang.annotation.ElementType)：
	
	ElementType.CONSTRUCTOR//构造器的声明
	ElementType.FIELD//域声明（包括enum实例）
	ElementType.LOCAL_VARIABLE//局部变量声明
	ElementType.METHOD//方法声明
	ElementType.PACKAGE//包声明
	ElementType.PARAMETER//参数声明
	ElementType.TYPE//类、接口（包括注解类型）或enum声明

**@Documented**

@Documented是一个标记注解，没有成员。该元注解用于将其他注解包含在Javadoc中，从而被文档化。

**@Inherited**

@Inherited元注解也是一个标记注解，被该注解标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

## 自定义注解

用关键字@interface 定义一个注解标记，使用@interface 关键字实际上的意思就是该接口继承自java.lang.annotation.Annotation接口。

定义注解格式：

```java
public @interface 注解名 {定义体}
```
注意：
+ 注解方法不能有参数。
+ 注解的值必须是确定的，且不能使用null作为值。
+ 注解方法的返回类型局限于原始类型，String，Enum，Annotation，或以上类型构成的数组。
+ 注解方法可以包含默认值，使用default就可实现。
+ 注解只有一个元素的时候，该元素名称必须是value，并且在使用该注解的时候可以省略”value=”。
+ 注解可以包含与其绑定的元注解。

自定义注解实例:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Assignment {
    String assignee();
    int effort();
    double finished() default 0;
}
```
## 参考资料

[深入理解Java：注解（Annotation）自定义注解入门](http://www.cnblogs.com/peida/archive/2013/04/24/3036689.html)

[Java注解教程：自定义注解示例，利用反射进行解析](http://www.importnew.com/14479.html)
