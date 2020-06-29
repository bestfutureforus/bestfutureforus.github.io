---
layout:     post
title:      "SpringMVC学习笔记(2)——商品列表查询入门程序"
subtitle:   ""
date:       2017-04-10 13:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 框架
---

在商品订单管理为例，做一个商品列表查询的简单案例。

# 开发环境和运行环境

+ 开发工具：IntelliJ IDEA Ultimate 15
+ 运行环境：Tomcat 8.5.11
+ 工程：使用maven构建的web工程
+ 依赖：spring和tomcat，在maven工程的pom.xml文件中配置

# 非注解的处理器映射器和适配器开发方式

### 配置前端控制器

在web.xml中配置前端控制器
```xml
<?xml version="1.0" encoding="utf-8" ?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!-- springmvc 前端控制器  -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- contextConfigLocation配置springmvc加载的配置文件(配置处理器映射器、适配器等等)
      若不配置，默认加载WEB-INF/servlet名称-servlet(springmvc-servlet.xml)
    -->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:/spring/springmvc.xml</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--
    第一种:*.action,访问以.action三结尾，由DispatcherServlet进行解析
    第二种:/,所有访问的地址由DispatcherServlet进行解析，对静态文件的解析需要配置不让DispatcherServlet进行解析，
            使用此种方式和实现RESTful风格的url
    第三种:/*,这样配置不对，使用这种配置，最终要转发到一个jsp页面时，仍然会由DispatcherServlet解析jsp地址，
            不能根据jsp页面找到handler，会报错
    -->
    <url-pattern>*.action</url-pattern>
  </servlet-mapping>
</web-app>
```

### 配置处理器适配器

在classpath下的/spring/springmvc.xml中配置处理器适配器

```xml
    <!-- 处理器适配器
     所有处理器适配器都实现了HandlerAdapter接口
     -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

### 开发Handler

需要实现 Controller接口，才能由`org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter`适配器执行。

```java
package com.crow.ssm.controller;


import com.crow.ssm.po.Items;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by CrowHawk on 17/3/30.
 */
public class ItemsController implements Controller {

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //调用service查找数据库，查询商品列表，这里使用静态数据模拟
        List<Items> itemsList = new ArrayList<Items>();

        //向list中填充静态数据
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("X Carbon");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone7");

        itemsList.add(items_1);
        itemsList.add(items_2);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList", itemsList);

        //指定视图
        modelAndView.setViewName("/items/itemsList");

        return modelAndView;
    }
}
```

### 视图编写

目录结构如下
![](http://pic.yupoo.com/crowhawk/GmpPuLwy/piHTc.jpg)
itemsList.jsp

```xml
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>查询商品列表</title>
</head>
<body>
<form action="${pageContext.request.contextPath }/item/queryItem.action" method="post">
    查询条件：
    <table width="100%" border=1>
        <tr>
            <td><input type="submit" value="查询"/></td>
        </tr>
    </table>
    商品列表：
    <table width="100%" border=1>
        <tr>
            <td>商品名称</td>
            <td>商品价格</td>
            <td>生产日期</td>
            <td>商品描述</td>
            <td>操作</td>
        </tr>
        <c:forEach items="${itemsList }" var="item">
            <tr>
                <td>${item.name }</td>
                <td>${item.price }</td>
                <td><fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/></td>
                <td>${item.detail }</td>

                <td><a href="${pageContext.request.contextPath }/item/editItem.action?id=${item.id}">修改</a></td>

            </tr>
        </c:forEach>

    </table>
</form>
</body>

</html>
```

### 配置Handler

将编写Handler在spring容器加载。

```xml
<!-- 配置Handler -->
    <bean id="itemsController" class="com.crow.ssm.controller.ItemsController"/>
```

### 配置处理器映射器

在classpath下的/spring/springmvc.xml中配置处理器适配器

```xml
    <!-- 处理器映射器
    将bean的name作为url进行查找，需要在配置Handler时指定beanname(就是url)
    所有的映射器都实现了HandlerMapping接口
     -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

### 配置视图解析器

在classpath下的/spring/springmvc.xml中配置解析jsp的视图解析器。

```xml
<!-- 视图解析器
    解析jsp,默认使用jstl,classpath下要有jstl的包
    -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
```
在springmvc.xml中视图解析器配置前缀和后缀：
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 配置jsp路径的前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 配置jsp路径的后缀 -->
        <property name="suffix" value=".jsp"/>
</bean>
```
则在程序中就不用指定前缀和后缀
```java
//指定视图
//下边的路径，如果在视图解析器中配置jsp的路径前缀和后缀，修改为items/itemsList
//modelAndView.setViewName("/WEB-INF/jsp/items/itemsList.jsp");

//下边的路径配置就可以不在程序中指定jsp路径的前缀和后缀
modelAndView.setViewName("items/itemsList");
```

### 部署调试
![](http://pic.yupoo.com/crowhawk/GmpXkQKI/131h2Y.jpg)

# 注解的处理器映射器和适配器开发方式

### 配置注解映射器和适配器

在springmvc.xml中配置注解映射器和适配器
```xml
    <!--注解映射器 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
    <!--注解适配器 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
    <!-- 使用 mvc:annotation-driven代替上边注解映射器和注解适配器配置
	mvc:annotation-driven默认加载很多的参数绑定方法，
	比如json转换解析器就默认加载了，如果使用mvc:annotation-driven不用配置上边的RequestMappingHandlerMapping和RequestMappingHandlerAdapter
	实际开发时使用mvc:annotation-driven
	 -->
	<!-- <mvc:annotation-driven></mvc:annotation-driven> -->
```

### 开发注解Handler

使用注解的映射器和注解的适配器（注解的映射器和注解的适配器必须配对使用）。

```java
package com.crow.ssm.controller;

import com.crow.ssm.po.Items;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by CrowHawk on 17/3/31.
 */
Controller
public class ItemsController3 {

    @RequestMapping("/queryItems")
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //调用service查找数据库，查询商品列表，这里使用静态数据模拟
        List<Items> itemsList = new ArrayList<Items>();

        //向list中填充静态数据
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("X Carbon");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone7");

        itemsList.add(items_1);
        itemsList.add(items_2);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList", itemsList);

        //指定视图
        modelAndView.setViewName("/items/itemsList");

        return modelAndView;
    }
}
```

### 在Spring容器中加载Handler

```xml
    <!-- <bean class="com.crow.ssm.controller.ItemsController3" /> -->
    <!-- 可以扫描controller、service、...
    这里让扫描controller，指定controller的包
     -->
    <context:component-scan base-package="com.crow.ssm.controller"></context:component-scan>
```

### 部署调试

![](http://pic.yupoo.com/crowhawk/GmpXkQKI/131h2Y.jpg)



