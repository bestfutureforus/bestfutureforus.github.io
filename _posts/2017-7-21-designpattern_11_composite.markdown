---
layout:     post
title:      "设计模式(11)——组合模式"
subtitle:   ""
date:       2017-07-21 20:30:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 设计模式
    - 读书笔记
---

**组合（composite）模式**允许你将对象组合成树形结构来表现“整体／部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。

听起来可能有点抽象，我们在上一篇文章中迭代器模式的例子的基础上继续深入，来看看组合模式的奥妙。

在上一篇文章中，我们实现了西餐厅菜单、煎饼屋菜单和咖啡厅菜单，现在我们希望能够在西餐厅菜单上加上一份餐后甜点的“子菜单”。

这样一来我们又要修改我们的设计了，我们现在需要某种树形结构，可以容纳菜单、子菜单和菜单项。
我们需要确定能够在每个菜单的各个项之间游走，而且至少要像现在用迭代器一样方便。
我们也需要能够更有弹性地在菜单项之间游走。比方说，可能只需要遍历甜点菜单，或者可以遍历餐厅的整个菜单（包括甜点菜单在内）。

下面我们来看如何在菜单上应用组合模式。一开始，我们需要创建一个**组件接口**来作为菜单和菜单项的共同接口，让我们能够用统一的做法来处理菜单和菜单项。换句话说，我们可以针对菜单或菜单项调用相同的方法。菜单组件的角色是为叶节点和组合节点提供一个共同的接口。

组合模式的类图如下：
![](https://pic.yupoo.com/crowhawk/063fa868/f2bd6ec6.png)

组件接口如下
```java
import java.util.*;

public abstract class MenuComponent {
   
	public void add(MenuComponent menuComponent) {
		throw new UnsupportedOperationException();
	}
	public void remove(MenuComponent menuComponent) {
		throw new UnsupportedOperationException();
	}
	public MenuComponent getChild(int i) {
		throw new UnsupportedOperationException();
	}
  
	public String getName() {
		throw new UnsupportedOperationException();
	}
	public String getDescription() {
		throw new UnsupportedOperationException();
	}
	public double getPrice() {
		throw new UnsupportedOperationException();
	}
	public boolean isVegetarian() {
		throw new UnsupportedOperationException();
	}

	public abstract Iterator createIterator();
 
	public void print() {
		throw new UnsupportedOperationException();
	}
}
```
现在我们来看菜单项类，这是组合类图里的叶类，它实现组合内元素的行为。
```java
import java.util.Iterator;

public class MenuItem extends MenuComponent {
 
	String name;
	String description;
	boolean vegetarian;
	double price;
    
	public MenuItem(String name, 
	                String description, 
	                boolean vegetarian, 
	                double price) 
	{ 
		this.name = name;
		this.description = description;
		this.vegetarian = vegetarian;
		this.price = price;
	}
  
	public String getName() {
		return name;
	}
  
	public String getDescription() {
		return description;
	}
  
	public double getPrice() {
		return price;
	}
  
	public boolean isVegetarian() {
		return vegetarian;
	}

	public Iterator createIterator() {
		return new NullIterator();
	}
 
	public void print() {
		System.out.print("  " + getName());
		if (isVegetarian()) {
			System.out.print("(v)");
		}
		System.out.println(", " + getPrice());
		System.out.println("     -- " + getDescription());
	}
}
```
我们已经有了菜单项，还需要组合类，即菜单。组合类可以持有菜单项或其他菜单。
```java
import java.util.Iterator;
import java.util.ArrayList;

public class Menu extends MenuComponent {
 
	ArrayList menuComponents = new ArrayList();
	String name;
	String description;
  
	public Menu(String name, String description) {
		this.name = name;
		this.description = description;
	}
 
	public void add(MenuComponent menuComponent) {
		menuComponents.add(menuComponent);
	}
 
	public void remove(MenuComponent menuComponent) {
		menuComponents.remove(menuComponent);
	}
 
	public MenuComponent getChild(int i) {
		return (MenuComponent)menuComponents.get(i);
	}
 
	public String getName() {
		return name;
	}
 
	public String getDescription() {
		return description;
	}

  
	public Iterator createIterator() {
		return new CompositeIterator(menuComponents.iterator());
	}
 
 
	public void print() {
		System.out.print("\n" + getName());
		System.out.println(", " + getDescription());
		System.out.println("---------------------");
  
		Iterator iterator = menuComponents.iterator();
		while (iterator.hasNext()) {
			MenuComponent menuComponent = 
				(MenuComponent)iterator.next();
			menuComponent.print();
		}
	}
}
```
我们还实现了一个组合迭代器，用来遍历组件内的菜单项，而且确保所有的子菜单（以及子子菜单……）都被包括进来。
```java
import java.util.*;
  
public class CompositeIterator implements Iterator {
	Stack stack = new Stack();
   
	public CompositeIterator(Iterator iterator) {//将要遍历的顶层组合的迭代器传入，把它抛进一个堆栈数据结构中
		stack.push(iterator);
	}
   
	public Object next() {
		if(hasNext()) {
			Iterator iterator = (Iterator) stack.peek();
			MenuComponent menuComponent = (MenuComponent) iterator.next();
			if(menuComponent instanceof Menu) {//如果元素是一个菜单，则有了另一个需要被包含进遍历中的组合，把它也压入栈中
				stack.push(menuComponent.createIterator());
			}
			return menuComponent;
		}
		else {
			return null;
		}
	}
  
	public boolean hasNext() {
		if(stack.isEmpty()) {
			return false;
		}
		else {
			if(((Iterator)stack.peek()).hasNext()) {
				return true;
			}
			else {
				stack.pop();
				return hasNext();
			}
		}
	}
   
	public void remove() {
		throw new UnsupportedOperationException();
	}
}
```
我们还需要实现一个空迭代器，当菜单项内没有东西可以遍历时，就返回这个空迭代器，它的hasNext()永远返回false
```java
import java.util.Iterator;
  
public class NullIterator implements Iterator {
   
	public Object next() {
		return null;
	}
  
	public boolean hasNext() {
		return false;
	}
   
	public void remove() {
		throw new UnsupportedOperationException();
	}
}
```
现在我们可以来实现女招待了，并且为她加上了一个可以确切地告诉我们哪些项目是素食的方法。
```java
import java.util.Iterator;
  
public class Waitress {
	MenuComponent allMenus;
 
	public Waitress(MenuComponent allMenus) {
		this.allMenus = allMenus;
	}
 
	public void printMenu() {
		allMenus.print();
	}
  
	public void printVegetarianMenu() {
		Iterator iterator = allMenus.createIterator();

		System.out.println("\nVEGETARIAN MENU\n----");
		while (iterator.hasNext()) {
			MenuComponent menuComponent = 
					(MenuComponent)iterator.next();
			try {
				if (menuComponent.isVegetarian()) {
					menuComponent.print();
				}
			} catch (UnsupportedOperationException e) {}
		}
	}
}
```
最后来测试一下
```java
public class MenuTestDrive {
	public static void main(String args[]) {

		MenuComponent pancakeHouseMenu = 
			new Menu("PANCAKE HOUSE MENU", "Breakfast");
		MenuComponent dinerMenu = 
			new Menu("DINER MENU", "Lunch");
		MenuComponent cafeMenu = 
			new Menu("CAFE MENU", "Dinner");
		MenuComponent dessertMenu = 
			new Menu("DESSERT MENU", "Dessert of course!");
  
		MenuComponent allMenus = new Menu("ALL MENUS", "All menus combined");
  
		allMenus.add(pancakeHouseMenu);
		allMenus.add(dinerMenu);
		allMenus.add(cafeMenu);
  
		pancakeHouseMenu.add(new MenuItem(
			"K&B's Pancake Breakfast", 
			"Pancakes with scrambled eggs, and toast", 
			true,
			2.99));
		pancakeHouseMenu.add(new MenuItem(
			"Regular Pancake Breakfast", 
			"Pancakes with fried eggs, sausage", 
			false,
			2.99));
		pancakeHouseMenu.add(new MenuItem(
			"Blueberry Pancakes",
			"Pancakes made with fresh blueberries, and blueberry syrup",
			true,
			3.49));
		pancakeHouseMenu.add(new MenuItem(
			"Waffles",
			"Waffles, with your choice of blueberries or strawberries",
			true,
			3.59));

		dinerMenu.add(new MenuItem(
			"Vegetarian BLT",
			"(Fakin') Bacon with lettuce & tomato on whole wheat", 
			true, 
			2.99));
		dinerMenu.add(new MenuItem(
			"BLT",
			"Bacon with lettuce & tomato on whole wheat", 
			false, 
			2.99));
		dinerMenu.add(new MenuItem(
			"Soup of the day",
			"A bowl of the soup of the day, with a side of potato salad", 
			false, 
			3.29));
		dinerMenu.add(new MenuItem(
			"Hotdog",
			"A hot dog, with saurkraut, relish, onions, topped with cheese",
			false, 
			3.05));
		dinerMenu.add(new MenuItem(
			"Steamed Veggies and Brown Rice",
			"A medly of steamed vegetables over brown rice", 
			true, 
			3.99));
 
		dinerMenu.add(new MenuItem(
			"Pasta",
			"Spaghetti with Marinara Sauce, and a slice of sourdough bread",
			true, 
			3.89));
   
		dinerMenu.add(dessertMenu);
  
		dessertMenu.add(new MenuItem(
			"Apple Pie",
			"Apple pie with a flakey crust, topped with vanilla icecream",
			true,
			1.59));
		dessertMenu.add(new MenuItem(
			"Cheesecake",
			"Creamy New York cheesecake, with a chocolate graham crust",
			true,
			1.99));
		dessertMenu.add(new MenuItem(
			"Sorbet",
			"A scoop of raspberry and a scoop of lime",
			true,
			1.89));

		cafeMenu.add(new MenuItem(
			"Veggie Burger and Air Fries",
			"Veggie burger on a whole wheat bun, lettuce, tomato, and fries",
			true, 
			3.99));
		cafeMenu.add(new MenuItem(
			"Soup of the day",
			"A cup of the soup of the day, with a side salad",
			false, 
			3.69));
		cafeMenu.add(new MenuItem(
			"Burrito",
			"A large burrito, with whole pinto beans, salsa, guacamole",
			true, 
			4.29));
 
		Waitress waitress = new Waitress(allMenus);
   
		waitress.printVegetarianMenu();
 
	}
}
```
下面是运行结果
![](https://pic.yupoo.com/crowhawk/9af655e0/eedf69e9.png)

### 设计分析

值得注意的是，组合模式不符合我们在先前的文章中所介绍的**单一责任制度**。组合模式让一个类有两个责任，不但需要管理层次结构，还要执行菜单的操作。可以这么说，组合模式以单一责任设计原则换取**透明性（transparency）**。所谓透明性，即是通过让组件的接口同时包含一些管理子节点和叶节点的操作，客户就可以将组合和叶节点一视同仁。也就是说，**一个元素究竟是组合还是叶节点，对客户是透明的**。
