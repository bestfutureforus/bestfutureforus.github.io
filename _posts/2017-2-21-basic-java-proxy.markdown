---
layout:     post
title:      "Java基础学习(4)——动态代理"
subtitle:   ""
date:       2017-02-21 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

## 代理

我们经常需要为已存在的多个**具有相同接口**的目标类的各个方法增加一些系统功能，例如，异常处理、日志、计算方法和运行时间、事务管理等等。

为此我们可以编写一个与目标类具有相同接口的**代理类**，代理类的每个方法调用目标类的相同方法，并在调用方法时加上系统功能的代码，在访问者看来，两者没有任何区别。通过代理类这中间一层，能有效控制对目标类对象的直接访问，也可以很好地隐藏和保护目标类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。这是一种***设计模式***，被称之为**代理**。代理是**AOP(Aspect oriented program,面向切面编程)**的关键技术。

如果采用工厂模式和配置文件的方式进行管理，则不需要修改客户端程序，**在配置文件中配置是使用目标类、还是代理类**，这样以后就很容易切换，譬如，想要日志功能时就配置代理类，否则配置目标类，这样，增加系统功能很容易，以后运行一段时间后，又想去掉系统功能也很容易。



## 动态代理

**静态代理**就是在源码阶段就为为系统中的各种接口的类增加代理功能，如果这样做，将需要非常多的代理类。
JVM可以**在运行期间动态生成类的字节码**，这种动态生的类往往被用作代理类，即**动态代理**。

+ JVM生成的**动态代理类必须实现*至少一个接口***， JVM生成的动态类只能用作**具有相同接口的目标类**的代理。
+ CGLIB库可以动态生成一个类的子类，一个类的子类也可以用作该类的代理，所以，如果要为一个没有实现接口的类生成动态代理类，可以使用CGLIB类。
+ 代理类的各个方法中通常除了要调用目标的相应方法和对外返回目标返回的结果外，还可以在代理方法中**调用目标方法前后**和**处理目标方法异常的catch块中**加上系统功能代码。

## 代理机制和特点

**Java动态代理主要涉及以下几个类：**
* `java.lang.reflect.Proxy` 
   Java动态代理机制的主类，提供了一组静态方法来为一组接口动态地生成代理类及其对象。其中最重要的一个静态方法如下，该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例

```java
static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)
```

* `java.lang.reflect.InvocationHandler`
   这是调用处理器接口，它自定义了一个**invoke 方法**，用于集中处理在动态代理类对象上的方法调用，通常**在该方法中实现对目标类的代理访问**。每次生成动态代理类对象时都需要指定一个实现了该接口的调用处理器对象（参见上文中列出的Proxy类的静态方法）。

```java
// 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象,第三个方法是调用参数。
// 调用处理器根据这三个参数进行预处理或分派到目标类实例上发射执行
Object invoke(Object proxy, Method method, Object[] args)
```

* `java.lang.ClassLoader`
   这是类加载器类，负责将类的字节码装载到JVM中并为其定义类对象，每次生成动态代理类对象时都需要指定一个类装载器对象（参见上文中列出的Proxy类的静态方法）。

**使用Java动态代理的具体步骤如下：**
1. 通过实现 InvocationHandler 接口创建自己的调用处理器
2. 通过 Proxy 类创建动态代理类实例，创建实例时调用处理器对象作为参数被传入

```java
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到目标类的分派转发
InvocationHandler handler = new InvocationHandlerImpl(..); 
// 通过 Proxy 直接创建动态代理类实例
Interface proxy = (Interface)Proxy.newProxyInstance( classLoader, 
	 new Class[] { Interface.class }, 
	 handler );
```

下面给出一个创建动态代理类实例的例子，可以直接运行

自定义接口`Advice`

```java
package com.crow.Proxy;

import java.lang.reflect.*;

/**
 * Created by CrowHawk on 17/2/20.
 */
public interface Advice {
    void forwardMethod(Method method);
    void backMethod(Method method);
}
```

实现了`Advice`接口的类`AdviceImpl`

```java
package com.crow.Proxy;

import java.lang.reflect.Method;

/**
 * Created by CrowHawk on 17/2/20.
 */
public class AdviceImpl implements Advice {
    long beginTime = 0;
    public void forwardMethod(Method method){
        System.out.println("end");
        beginTime = System.currentTimeMillis();
    }

    public void backMethod(Method method){
        System.out.println("start");
        long endTime = System.currentTimeMillis();
        System.out.println(method.getName() + " running time of " + (endTime - beginTime));
    }
}
```

ProxyTest.java

```java
package com.crow.Proxy;

import java.lang.reflect.*;
import java.util.*;
import java.util.Collection;

/**
 * Created by CrowHawk on 17/2/20.
 */
public class ProxyTest {
    public static void main(String args[]){
        ArrayList<String> target = new ArrayList<>();//创建目标类的实例对象
        AdviceImpl adviceImpl = new AdviceImpl();
        Collection proxy = (Collection) getProxy(target, adviceImpl);//创建动态类
        proxy.add("aa");
        System.out.println(proxy.size());
        System.out.println(proxy.getClass().getName());

    }

    public static Object getProxy(final Object target, final Advice advice){
        Object proxy = Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {//动态类通过Invocation类的invoke方法调用目标类所需的方法
                    public Object invoke(Object proxy, Method method, Object[] args) throws Exception {
                        advice.forwardMethod(method);
                        Object retVal = method.invoke(target, args);
                        advice.backMethod(method);
                        return retVal;
                    }
                }
        );
        return proxy;
    }

}
```

## 参考资料
+ [Java 动态代理机制分析及扩展，第 1 部分](http://www.ibm.com/developerworks/cn/java/j-lo-proxy1/index.html)
