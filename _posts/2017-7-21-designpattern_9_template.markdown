---
layout:     post
title:      "设计模式(9)——模板方法模式"
subtitle:   ""
date:       2017-07-21 14:00:01
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 设计模式
    - 读书笔记
---

**模板（template）方法模式**在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模版方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。
这个模式用来创建一个算法的模板，模板就是一个方法。更具体地说，这个方法将算法定义成一组步骤，其中的任何步骤都可以是抽象的，由子类负责实现。这可以确保算法的结构保持不变，同时由子类提供部分实现。

下面直接用一个实例来加以说明。

在一个咖啡馆里，有咖啡和茶两种饮料，它们的冲泡方法分别如下：

**咖啡**
1. 把水煮沸
2. 用沸水冲泡咖啡
3. 把咖啡倒进杯子
4. 加糖和牛奶

**茶**
1. 把水煮沸
2. 用沸水浸泡茶叶
3. 把茶倒进杯子
4. 加柠檬

可以看到，它们的第1和第3步其实是一样的，第2和第4步不同，但也有些相似。我们可以抽取出一个共同点，设计出一个抽象类：
```java
public abstract class CaffeineBeverage {
  
	final void prepareRecipe() {
		boilWater();
		brew();
		pourInCup();
		addCondiments();
	}
 
	abstract void brew();
  
	abstract void addCondiments();
 
	void boilWater() {
		System.out.println("Boiling water");
	}
  
	void pourInCup() {
		System.out.println("Pouring into cup");
	}
}
```
然后再通过继承分别实现咖啡和茶
```java
public class Coffee extends CaffeineBeverage {
	public void brew() {
		System.out.println("Dripping Coffee through filter");
	}
	public void addCondiments() {
		System.out.println("Adding Sugar and Milk");
	}
}
```
```java
public class Tea extends CaffeineBeverage {
	public void brew() {
		System.out.println("Steeping the tea");
	}
	public void addCondiments() {
		System.out.println("Adding Lemon");
	}
}
```

### “钩子”

以上是模板方法模式的一个最基本应用，下面我们要再深入一点，介绍一下**“钩子”**。

**钩子是一种被声明再抽象类中的方法，但只有空的或者默认的实现。**钩子的存在，可以让子类有能力对算法的不同点进行挂钩。要不要挂钩，由子类自己决定。

接着刚刚的例子，有了钩子，我们可以决定要不要覆盖方法。如果不提供自己的方法，抽象类会提供一个默认的实现。
```java
public abstract class CaffeineBeverageWithHook {
 
	void prepareRecipe() {
		boilWater();
		brew();
		pourInCup();
		if (customerWantsCondiments()) {
			addCondiments();
		}
	}
 
	abstract void brew();
 
	abstract void addCondiments();
 
	void boilWater() {
		System.out.println("Boiling water");
	}
 
	void pourInCup() {
		System.out.println("Pouring into cup");
	}
 
	boolean customerWantsCondiments() {
		return true;
	}
}
```
为了使用钩子，我们需要在子类中覆盖它。接着刚刚那个例子，我们可以用钩子控制咖啡因饮料是否执行某部分算法，说得更明确一点，就是饮料中是否要加进调料。
```java
import java.io.*;

public class CoffeeWithHook extends CaffeineBeverageWithHook {
 
	public void brew() {
		System.out.println("Dripping Coffee through filter");
	}
 
	public void addCondiments() {
		System.out.println("Adding Sugar and Milk");
	}
 
	public boolean customerWantsCondiments() {

		String answer = getUserInput();

		if (answer.toLowerCase().startsWith("y")) {
			return true;
		} else {
			return false;
		}
	}
 
	private String getUserInput() {
		String answer = null;

		System.out.print("Would you like milk and sugar with your coffee (y/n)? ");

		BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
		try {
			answer = in.readLine();
		} catch (IOException ioe) {
			System.err.println("IO error trying to read your answer");
		}
		if (answer == null) {
			return "no";
		}
		return answer;
	}
}
```
```java
import java.io.*;

public class TeaWithHook extends CaffeineBeverageWithHook {
 
	public void brew() {
		System.out.println("Steeping the tea");
	}
 
	public void addCondiments() {
		System.out.println("Adding Lemon");
	}
 
	public boolean customerWantsCondiments() {

		String answer = getUserInput();

		if (answer.toLowerCase().startsWith("y")) {
			return true;
		} else {
			return false;
		}
	}
 
	private String getUserInput() {
		// get the user's response
		String answer = null;

		System.out.print("Would you like lemon with your tea (y/n)? ");

		BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
		try {
			answer = in.readLine();
		} catch (IOException ioe) {
			System.err.println("IO error trying to read your answer");
		}
		if (answer == null) {
			return "no";
		}
		return answer;
	}
}
```
测试代码
```java
public class BeverageTestDrive {
	public static void main(String[] args) {
 
		Tea tea = new Tea();
		Coffee coffee = new Coffee();
 
		System.out.println("\nMaking tea...");
		tea.prepareRecipe();
 
		System.out.println("\nMaking coffee...");
		coffee.prepareRecipe();

 
		TeaWithHook teaHook = new TeaWithHook();
		CoffeeWithHook coffeeHook = new CoffeeWithHook();
 
		System.out.println("\nMaking tea...");
		teaHook.prepareRecipe();
 
		System.out.println("\nMaking coffee...");
		coffeeHook.prepareRecipe();
	}
}
```
运行结果
![](https://pic.yupoo.com/crowhawk/280a9642/e55354e9.png)
在这个例子中，钩子可以作为条件控制，影响抽象类中的算法流程，这是非常cool的。
我们可以总结出钩子的几种用法
+ 让子类实现算法中可选的部分，或者在钩子对于子类的实现不重要时，子类可以对此钩子置之不理。
+ 让子类能够有机会对模板方法中某些即将发生的（或刚刚发生的）步骤作出反应。比方说，名为`justReOrderedList()`的钩子方法允许子类在内部列表重新组织后执行某些动作（例如在屏幕上重现显示数据）。
+ 钩子可以让子类为其抽象类作一些决定。

### 设计原则

模板方法体现了一个新的设计原则，称为**好莱坞原则：别调用（打电话给）我们，我们会调用（打电话给）你**。

好莱坞原则可以给我们一种防止**“依赖腐败”**的方法。当高层组件依赖底层组件，而低层组件又依赖高层组件，而高层组件又依赖边侧组件，而边侧组件又依赖低层组件时，依赖腐败就发生了。

在好莱坞原则之下，我们允许低层组件将自己挂钩到系统上，但是高层组件会决定什么时候和怎样使用这些低层组件。换句话说。高层组件对待低层组件的方式是“别调用我们，我们会调用你”。

除了模板方法模式，在本系列之前的文章中所介绍的观察者模式和工厂方法模式也体现了好莱坞原则。
