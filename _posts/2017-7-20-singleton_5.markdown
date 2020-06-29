---
layout:     post
title:      "设计模式(5)——单例模式"
subtitle:   ""
date:       2017-07-20 22:00:05
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 设计模式
    - 读书笔记
---

**单例（Singleton）模式**确保一个类只有一个实例，并提供一个全局访问点。

有一些对象其实我们只需要一个，比方说线程池、缓存、对话框、处理偏好设置和注册表的对象、日志对象、充当外设驱动程序的对象等等。使用单例模式可以确保它们只有一个实例会被创建，且提供了一个全局访问点，在需要时才创建对象，和全局对象一样方便，也不像全局变量一样在程序一开始就创建好对象。

单例模式的实现并不复杂，可以使用私有的构造函数+静态的getterInstance方法来完成。
```java
public class Singleton {
	private static Singleton uniqueInstance;
	
	private Singleton() {}

	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton();
		}
		return uniqueInstance;
	}
}
```
下面是单例模式的一个示例，该代码模拟了一个巧克力工厂用锅炉生产巧克力的过程，巧克力锅炉是一个只需要唯一实例的类。
```java
public class ChocolateBoiler {
	private boolean empty;
	private boolean boiled;
	private static ChocolateBoiler uniqueInstance;
  
	private ChocolateBoiler() {
		empty = true;
		boiled = false;
	}
  
	public static ChocolateBoiler getInstance() {
		if (uniqueInstance == null) {
			System.out.println("Creating unique instance of Chocolate Boiler");
			uniqueInstance = new ChocolateBoiler();
		}
		System.out.println("Returning instance of Chocolate Boiler");
		return uniqueInstance;
	}

	public void fill() {
		if (isEmpty()) {
			empty = false;
			boiled = false;
			// fill the boiler with a milk/chocolate mixture
		}
	}
 
	public void drain() {
		if (!isEmpty() && isBoiled()) {
			// drain the boiled milk and chocolate
			empty = true;
		}
	}
 
	public void boil() {
		if (!isEmpty() && !isBoiled()) {
			// bring the contents to a boil
			boiled = true;
		}
	}
  
	public boolean isEmpty() {
		return empty;
	}
 
	public boolean isBoiled() {
		return boiled;
	}
}
```
```java
public class ChocolateController {
	public static void main(String args[]) {
		ChocolateBoiler boiler = ChocolateBoiler.getInstance();
		boiler.fill();
		boiler.boil();
		boiler.drain();

		// will return the existing instance
		ChocolateBoiler boiler2 = ChocolateBoiler.getInstance();
	}
}
```

### 单例模式与多线程

在多线程环境中，使用get方法会出现取出多个实例的情况，与单例模式的初衷违背。
解决方案：

+ 声明synchronized关键字：运行效率低下，下一个线程想要取得对象，必须等上一个线程释放锁后，才可以继续执行。
+ 使用同步代码块：与第一种方法一样同步运行，效率低下
+ 针对某些重要的代码进行单独的同步：只对实例化对象的关键代码进行同步，还是有线程安全问题
+ 使用DCL双检查锁机制：对实例对象加上volatile关键字，在关键代码块上对加了volatile关键字的实例变量的类加上synchronized锁，可以成功解决“懒汉模式”遇到的多线程问题。
+ 使用静态内置类实现单例模式：可以实现和DCL相同的效果。

一个使用DCL的实例如下：
```java
public class DCL {
	private volatile static DCL uniqueInstance;
 
	private DCL() {}
 
	public static DCL getInstance() {
		if (uniqueInstance == null) {
			synchronized (DCL.class) {
				if (uniqueInstance == null) {
					uniqueInstance = new DCL();
				}
			}
		}
		return uniqueInstance;
	}
}
```
```java
public class DCLTest {
    public static void main(String[] args) {
        DCL dcl = DCL.getInstance();
    }
}
```
