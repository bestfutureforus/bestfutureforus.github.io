---
layout:     post
title:      "设计模式(10)——迭代器模式"
subtitle:   ""
date:       2017-07-21 15:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 设计模式
    - 读书笔记
---

**迭代器（iterator）模式**提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。

如果你有一个统一的方法访问聚合中的每一个对象，你就可以编写多态的代码和这些聚合搭配，使用。迭代器模式把在元素之间游走的责任交给迭代器，而不是聚合对象。这不仅让聚合的接口和实现变得更简洁，也可以让聚合更专注在它所应该专注的事情上面（也就是管理对象集合），而不必去理会遍历的事情。

下面用一个具体实例来加以说明。
<p>我们模拟了一个西餐厅、一个煎饼屋和一个咖啡厅，现在我们想要让顾客在同一个地方能够同时点这三种餐馆的菜色。</p>

在原来的菜单实现中，西餐厅的菜单用`数组`来记录菜单项，煎饼屋的菜单用`ArrayList`来记录菜单项，而咖啡厅的菜单用`Hashtable`来记录菜单项。

我们抽取出的菜单接口为
```java
import java.util.Iterator;

public interface Menu {
	public Iterator createIterator();
}
```
菜单项的类为
```java
public class MenuItem {
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
}
```
<p>我们需要创建一个Java版本的女招待，她需要能应对顾客的需要打印定制的菜单，甚至告诉你是否某个菜单项是素食的，而无需询问厨师。</p>

我们需要找出一个方法，让这些餐馆的菜单实现一个相同的接口。在之前的文章中讲到，我们有一个重要的设计原则就是**封装变化的部分**。很明显，在这里发生变化的是由不同的集合类型所造成的遍历。我们需要借助迭代器模式来解决这一问题。

迭代器模式依赖于一个名为迭代器的接口
```java
public interface Iterator {
	boolean hasNext();
	Object next();
}
```
对于煎饼屋的`ArrayList`菜单和咖啡厅的`Hashtable`菜单，Java均已为其实现了默认的迭代器，而对于西餐厅的`数组`菜单，则需要我们自己实现一个迭代器：
```java
import java.util.Iterator;
  
public class DinerMenuIterator implements Iterator {
	MenuItem[] list;
	int position = 0;
 
	public DinerMenuIterator(MenuItem[] list) {
		this.list = list;
	}
 
	public Object next() {
		MenuItem menuItem = list[position];
		position = position + 1;
		return menuItem;
	}
 
	public boolean hasNext() {
		if (position >= list.length || list[position] == null) {
			return false;
		} else {
			return true;
		}
	}
  
	public void remove() {
		if (position <= 0) {
			throw new IllegalStateException
				("You can't remove an item until you've done at least one next()");
		}
		if (list[position-1] != null) {
			for (int i = position-1; i < (list.length-1); i++) {
				list[i] = list[i+1];
			}
			list[list.length-1] = null;
		}
	}
}
```
相应的西餐厅菜单实现为
```java
import java.util.Iterator;

public class DinerMenu implements Menu {
	static final int MAX_ITEMS = 6;
	int numberOfItems = 0;
	MenuItem[] menuItems;
  
	public DinerMenu() {
		menuItems = new MenuItem[MAX_ITEMS];
 
		addItem("Vegetarian BLT",
			"(Fakin') Bacon with lettuce & tomato on whole wheat", true, 2.99);
		addItem("BLT",
			"Bacon with lettuce & tomato on whole wheat", false, 2.99);
		addItem("Soup of the day",
			"Soup of the day, with a side of potato salad", false, 3.29);
		addItem("Hotdog",
			"A hot dog, with saurkraut, relish, onions, topped with cheese",
			false, 3.05);
		addItem("Steamed Veggies and Brown Rice",
			"A medly of steamed vegetables over brown rice", true, 3.99);
		addItem("Pasta",
			"Spaghetti with Marinara Sauce, and a slice of sourdough bread",
			true, 3.89);
	}
  
	public void addItem(String name, String description, 
	                     boolean vegetarian, double price) 
	{
		MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
		if (numberOfItems >= MAX_ITEMS) {
			System.err.println("Sorry, menu is full!  Can't add item to menu");
		} else {
			menuItems[numberOfItems] = menuItem;
			numberOfItems = numberOfItems + 1;
		}
	}
 
	public MenuItem[] getMenuItems() {
		return menuItems;
	}
  
	public Iterator createIterator() {
		return new DinerMenuIterator(menuItems);
		//return new AlternatingDinerMenuIterator(menuItems);
	}
 
	// other menu methods here
}
```
为避免篇幅太过冗长，还有其他餐馆的菜单和女招待代码就不在此列出，有兴趣的可[点击此处](https://github.com/CrowHawk/DesignPattern-Learning/tree/master/Iterator/src)查看。

### 设计原则——单一责任原则

如果我们允许我们的聚合实现它们内部的集合，以及相关的操作和遍历的方法，这样我们就允许一个类不但要完成自己的事情（管理某种聚合），还同时要负担更多的责任（例如遍历）时，我们就给了这个类两个变化的原因：
+ 如果这个集合改变的话，这个类也必须跟着改变
+ 如果我们遍历的方式改变的话，这个类也必须跟着改变

所以，我们的老朋友“改变”，又一次成为了我们设计原则的中心——**单一责任原则：一个类应该只有一个引起变化的原因。**
