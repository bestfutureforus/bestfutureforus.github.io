---
layout:     post
title:      "SpringMVC学习笔记(3)——Spring+SpringMVC+MyBatis框架整合"
subtitle:   ""
date:       2017-04-11 10:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 框架
---

**SpringMVC+Spring+MyBatis**框架的组合，是标准的MVC程序架构模式，将整个系统划分为**显示层**、**Controller层**、**Service层**、**Dao层**四层，使用SpringMVC负责请求的转发和视图管理，Spring实现业务对象管理, MyBatis作为数据对象持久化引擎。彼此分工，极大地提高了Web应用的开发效率。本文依然以商品列表查询为例，介绍SSM框架的整合使用。

## SSM框架的系统层次说明

+ **持久层：DAO层（mapper）**
> DAO层主要做数据持久层的工作，负责**与数据库进行联络**的一些任务都封装在此。
DAO层的设计首先是设计DAO的接口，然后在Spring的配置文件中定义此接口的实现类，然后就可在模块中调用此接口来进行数据业务的处理，而不用关心此接口的具体实现类是哪个类，显得结构非常清晰
DAO层的数据源配置，以及有关数据库连接的参数都在Spring的配置文件中进行配置。

+ **业务层：Service层**
> Service层主要负责**业务模块的逻辑应用设计**。 
首先设计接口，再设计其实现的类，接着再在Spring的配置文件中配置其实现的关联。这样我们就可以在应用中调用Service接口来进行业务处理。
Service层的业务实现，具体要调用到已定义的DAO层的接口，
封装Service层的业务逻辑有利于通用的业务逻辑的独立性和重复利用性，程序显得非常简洁。

+ **表现层：Controller层（Handler层）**
> Controller层负责具体的**业务模块流程**的控制。
在此层里面要**调用Service层的接口来控制业务流程**，控制的配置也同样是在Spring的配置文件里面进行，针对具体的业务流程，会有不同的控制器，我们具体的设计过程中可以将流程进行抽象归纳，设计出可以重复利用的子单元流程模块，这样不仅使程序结构变得清晰，也大大减少了代码量。

+ **View层**
> View层与控制层结合比较紧密，需要二者结合起来协同工发。View层主要负责前台jsp页面的表示。

+ **各层联系**
> DAO层，Service层这两个层次都可以单独开发，互相的耦合度很低，完全可以独立进行，这样的一种模式在开发大项目的过程中尤其有优势
Controller，View层因为耦合度比较高，因而要结合在一起开发，但是也可以看作一个整体独立于前两个层进行开发。这样，在层与层之前我们只需要知道接口的定义，调用接口即可完成所需要的逻辑单元应用，一切显得非常清晰简单。

+ **Service逻辑层设计**
> Service层是建立在DAO层之上的，建立了DAO层后才可以建立Service层，而Service层又是在Controller层之下的，因而Service层应该既调用DAO层的接口，又要提供接口给Controller层的类来进行调用，它刚好处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。

## SSM框架整合

**整合思路：**
1. **整合dao层**
	MyBatis和Spring整合，通过Spring管理Mapper接口。
	使用Mapper的扫描器自动扫描Mapper接口在Spring中进行注册。

2. **整合Service层**
	通过Spring管理 Service接口。
	使用配置方式将Service接口配置在Spring配置文件中，实现事务控制。

3. **整合SpringMVC**
    由于SpringMVC是Spring的模块，不需要整合。

### 开发环境和运行环境

+ 开发工具：IntelliJ IDEA Ultimate 15
+ 运行环境：Tomcat 8.5.11
+ 工程：使用maven构建的web工程
+ 依赖：在maven工程的pom.xml文件中配置

这里直接列出我的pom.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.crow</groupId>
    <artifactId>springmvc-mybatis-start</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- jar 版本设置 -->
        <spring.version>4.2.4.RELEASE</spring.version>
    </properties>

    <dependencies>

        <!-- servlet&jsp配置 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!-- spring框架-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.2.4</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>

        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.18</version>
        </dependency>

        <!-- JSP tag -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>

        <!-- hibernate 校验 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.4.Final</version>
        </dependency>

        <!-- 文件上传 -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>

        <!-- json 转换-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.7.2</version>
        </dependency>

        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.13</version>
        </dependency>

    </dependencies>

</project>
```

+ 工程结构：
![](http://pic.yupoo.com/crowhawk/Gms1aZ7G/gtQio.jpg)

### 整合Dao层

将MyBatis和Spring整合

#### MyBatis配置文件 sqlMapConfig.xml

配置别名：用于批量扫描Pojo包
不需要配置mappers标签，但一定要保证mapper.java文件与mapper.xml文件同名。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 全局setting配置，根据需要添加 -->

    <!-- 配置别名 -->
    <typeAliases>
        <!-- 批量扫描别名 -->
        <package name="com.crow.ssm.po"/>
    </typeAliases>

    <!-- 配置mapper
    由于使用spring和mybatis的整合包进行mapper扫描，这里不需要配置了。
    必须遵循：mapper.xml和mapper.java文件同名且在一个目录
     -->

    <!-- <mappers>

    </mappers> -->
</configuration>
```

#### Spring配置文件 applicationContext-dao.xml

配置内容：
+ 数据源
+ SqlSessionFactory
+ mapper扫描器

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd

    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <!-- 加载db.properties文件中的内容，db.properties文件中key命名要有一定的特殊规则 -->
    <context:property-placeholder location="classpath:db.properties"/>
    <!-- 配置数据源 ，dbcp -->

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxActive" value="30"/>
        <property name="maxIdle" value="5"/>
    </bean>

    <!-- 从整合包里找，org.mybatis:mybatis-spring:1.2.4 -->
    <!-- sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 加载mybatis的全局配置文件 -->
        <property name="configLocation" value="classpath:mybatis/sqlMapConfig.xml"/>
        <!-- <property name="mapperLocations" value="classpath:com/crow/ssm /mapper/*.xml"/>-->
    </bean>

    <!-- mapper扫描器 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 扫描包路径，如果需要扫描多个包，中间使用半角逗号隔开 -->
        <property name="basePackage" value="com.crow.ssm.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- <property name="sqlSessionFactory" ref="sqlSessionFactory" />
        会导致数据源配置不管用，数据库连接不上。
        且spring 4弃用
        -->
    </bean>

</beans>
```

#### 逆向工程生成po类及mapper(单表增删改查)

![](http://pic.yupoo.com/crowhawk/Gms5K5gH/medium.jpg)

#### POJO的包装类ItemsMapperCustom.java

针对综合查询mapper，一般情况会有关联查询，建议自定义mapper
```java
public interface ItemsMappperCustom{
    public List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception;
}
```

#### ItemsMapperCustom.xml

sql语句：
```sql
SELECT * FROM items  WHERE items.name LIKE '%笔记本%'
```
ItemsMapperCustom.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.crow.ssm.mapper.ItemsMapperCustom">

    <!-- 定义商品查询的sql片段，就是商品查询条件 -->
    <sql id="query_items_where">
        <!-- 使用动态sql，通过if判断，满足条件进行sql拼接 -->
        <!-- 商品查询条件通过ItemsQueryVo包装对象 中itemsCustom属性传递 -->
        <if test="itemsCustom!=null">
            <if test="itemsCustom.name!=null and itemsCustom.name!=''">
                items.name LIKE '%${itemsCustom.name}%'
            </if>
        </if>

    </sql>

    <!-- 商品列表查询 -->
    <!-- parameterType传入包装对象(包装了查询条件)
        resultType建议使用扩展对象
     -->
    <select id="findItemsList" parameterType="com.crow.ssm.po.ItemsQueryVo"
            resultType="com.crow.ssm.po.ItemsCustom">
        SELECT items.* FROM items
        <where>
            <include refid="query_items_where"></include>
        </where>
    </select>

</mapper>
```

### 整合service

让spring管理service接口。

#### 定义service接口及其实现类

Service接口
```java
public interfae ItemsService{
    public List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception;
}
```

ServiceImpl实现类
因为在applicationContext-dao.xml中已经使用了mapper扫描器，这里可以直接通过注解的方式将itemsMapperCustom自动注入。

```java
public class ItemsServiceImpl implements ItemsService{

    @Autowired
    private ItemsMapperCustom itemsMapperCustom;

    @Override
    public List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception{
        return itemsMapperCustom.findItemsList(itemsQueryVo);
    }
}
```

#### Spring配置文件applicationContext-service.xml

在Spring容器中配置service

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <!-- 商品管理的service -->
    <bean id="itemsService" class="com.crow.ssm.service.impl.ItemsServiceImpl"/>

</beans>
```

#### 事务控制(applicationContext-transaction.xml)

在applicationContext-transaction.xml中使用Spring声明式事务控制方法。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 事务管理器
            对mybatis操作数据库事务控制，spring使用jdbc的事务控制类
        -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 数据源
        dataSource在applicationContext-dao.xml中配置了
         -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 传播行为 -->
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!-- aop -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.crow.ssm.service.impl.*.*(..))"/>
    </aop:config>
</beans>
```

### 整合SpringMVC

#### springmvc.xml

创建springmvc.xml文件，配置处理器映射器、适配器、视图解析器。

```xml
<context:component-scan base-package="cn.itcast.ssm.controller"></context:component-scan>

<!-- 使用 mvc:annotation-driven 加载注解映射器和注解适配器配置-->
<mvc:annotation-driven></mvc:annotation-driven>

<!-- 视图解析器 解析jsp解析，默认使用jstl标签，classpath下的得有jstl的包
 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 配置jsp路径的前缀 -->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!-- 配置jsp路径的后缀 -->
    <property name="suffix" value=".jsp"/>
</bean>
```

#### 配置前端控制器

参考[SpringMVC学习笔记(2)——商品列表查询入门程序](https://crowhawk.github.io/2017/04/10/SpringMVC_2/)

#### 编写Controller(Handler)

```java
@Congtroller

@RequestMapping("/items") //窄化路径
public class ItemsController {
    @Autowired
    private ItemsService itemsService;

    //商品查询列表
    @RequestMapping("/queryItems")
    //实现 对queryItems方法和url进行映射，一个方法对应一个url
    //一般建议将url和方法写成一样
    public ModelAndView queryItems(HttpServletRequest request, ItemsQueryVo itemsQueryVo) throws Exception {
        //测试forward后request是否可以共享
        //System.out.println(request.getParameter("id"));

        //调用service查找数据库，查询商品列表
        List<ItemsCustom> itemsList = itemsService.findItemsList(itemsQueryVo);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList", itemsList);

        //指定视图
        //下边的路径，如果在视图解析器中配置jsp的路径前缀和后缀，修改为items/itemsList
        //modelAndView.setViewName("/WEB-INF/jsp/items/itemsList.jsp");
        //下边的路径配置就可以不在程序中指定jsp路径的前缀和后缀
        modelAndView.setViewName("items/itemsList");

        return modelAndView;
    }
}
```

#### 编写jsp

参考[SpringMVC学习笔记(2)——商品列表查询入门程序](https://crowhawk.github.io/2017/04/10/SpringMVC_2/)

#### 加载spring容器

将mapper、service、controller加载到spring容器中。
建议使用通配符加载配置文件`applicationContext-dao.xml`，`applicationContext-service.xml`，`applicationContext-transaction.xml`。
在web.xml中，添加spring容器监听器，加载spring容器。

```xml
  <!-- 加载spring容器 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext-*.xml</param-value>
    <!--  <param-value>classpath:spring/applicationContext-*.xml</param-value>-->
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

---

以上便完成了SSM框架的整合，部署调试得运行结果
![](http://pic.yupoo.com/crowhawk/GmpXkQKI/131h2Y.jpg)

