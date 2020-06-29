---
layout:     post
title:      "SpringMVC学习笔记(1)——SpringMVC介绍"
subtitle:   ""
date:       2017-04-10 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 框架
---

# MVC架构模式

如何设计一个程序的结构，这是一门专门的学问，叫做**“架构模式”（architectural pattern）**，属于编程的方法论。 MVC模式就是架构模式的一种。
MVC是三个单词的首字母缩写，它们是**Model（模型）**、**View（视图）**和**Controller（控制）**。该模式可以把不论简单或复杂的程序，都从结构上划分为三层。

1. 最上面的一层，是直接面向最终用户的**"视图层"（View）**。它是提供给用户的操作界面，是程序的外壳。
2. 最底下的一层，是核心的**"数据层"（Model）**，也就是程序需要操作的数据或信息。
3. 中间的一层，就是**"控制层"（Controller）**，它负责根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行相应的操作，产生最终结果。

每一部分都相对独立，职责单一，在实现过程中可以专注于自身的核心逻辑。MVC是对系统复杂性的一种合理的梳理与切分，它的思想实质就是“关注点分离”。具体原理可用下图很好地概括：
![](http://pic.yupoo.com/crowhawk/GmoLuzsV/11nwbB.jpg)

# SpringMVC简介

Spring Web MVC是一种基于Java的实现了Web MVC设计模式的**请求驱动类型**的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用**请求-响应模型**。SpringMVC是Spring框架的一个模块，SpringMVC和Spring无需通过中间整合层进行整合。Spring MVC 分离了**控制器**、**模型对象**、**分派器**以及**处理程序对象**的角色，这种分离让它们更容易进行定制。

Spring 的 Web MVC 框架是围绕`DispatcherServlet` 设计的，它把请求分派给处理程序，同时带有可配置的*处理程序映射*、*视图解析*、*本地语言*、*主题解析*以及*上载文件*支持。应用控制器其实拆为**处理器映射器(Handler Mapping)**进行处理器管理和**视图解析器(View Resolver)**进行视图管理，**页面控制器**是非常简单的 `Controller` 接口，只有一个方法 `ModelAndView handleRequest(request, response)`。Spring 提供了一个**控制器层次结构**，可以派生子类。如果应用程序需要处理用户输入表单，那么可以继承 `AbstractFormController`。如果需要把多页输入处理到一个表单，那么可以继承 `AbstractWizardFormController`。

# SpringMVC框架原理

### SpringMVC请求处理的流程

![](http://pic.yupoo.com/crowhawk/Gmpd92DH/medium.jpg)
**具体执行步骤如下：**
1. 首先用户发送请求给**前端控制器（Front Controller）**，前端控制器根据请求信息（如URL）来决定选择哪一个**页面控制器（Controller）**进行处理并把请求委托给它，即以前的控制器的控制逻辑部分；
2. **页面控制器（Controller）**接收到请求后，进行功能处理，首先需要**收集和绑定请求参数到一个对象**，这个对象在SpringMVC中叫**命令对象**，并进行验证，然后**将命令对象委托给业务对象进行处理**；处理完毕后返回一个**ModelAndView**（模型数据和逻辑视图名）；
3. 前端控制器收回控制权，然后根据返回的**逻辑视图（View）**名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；
4. 前端控制器再次收回控制权，将响应返回给用户；至此整个结束。

### SpringMVC具体架构

该架构图只包括了SpringMVC的核心架构，没有包含文件上传、拦截器等功能，这些将在后续文章里介绍。
![](http://pic.yupoo.com/crowhawk/GmpmZ8CM/U334n.jpg)
**核心架构的具体流程步骤如下：**
1. **首先用户发送请求给DispatcherServlet（前端控制器）**，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
2. **DispatcherServlet访问HandlerMapping（处理器映射器）**， HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. **DispatcherServlet调用HandlerAdapter（处理器适配器）**，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. **HandlerAdapter调用处理器相应功能处理方法**，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名）；
5. **DispatcherServlet将ModelAndView的逻辑视图名发送给ViewResolver（视图解析器）**， ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6. **View进行视图渲染**，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. **返回控制权给DispatcherServlet**，由DispatcherServlet返回响应给用户，到此一个流程结束。

**涉及到的组件：**
+ **前端控制器DispatcherServlet（不需要程序员开发）**
**作用**：接收请求，响应结果，相当于转发器，中央处理器。
有了DispatcherServlet减少了其它组件之间的耦合度。
+ **处理器映射器HandlerMapping(不需要程序员开发)**
**作用**：根据请求的url查找Handler
+ **处理器适配器HandlerAdapter**
**作用**：按照特定规则（HandlerAdapter要求的规则）去执行Handler
+ **处理器Handler(需要程序员开发)**
编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
+ **视图解析器View resolver(不需要程序员开发)**
**作用**：进行视图解析，根据逻辑视图名解析成真正的视图（view）
+ **视图View(需要程序员开发jsp)**
View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）



