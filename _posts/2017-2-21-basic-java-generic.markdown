---
layout:     post
title:      "Java基础学习(3)——泛型"
subtitle:   ""
date:       2017-02-21 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

泛型是jdk1.5更新的特性，是提供给javac编译器使用的，可以限定集合中的输入类型，让编译器挡住源程序中的非法输入，编译器编译带类型说明的集合时会去除掉“类型”信息，使程序运行效率不受影响，**对于参数化的泛型类型，getClass()方法的返回值和原始类型完全一样**(当编译完成后，jvm无法得知集合的类型信息)，由于编译生成的字节码会去掉泛型的类型信息，只要能**跳过编译器**，就可以**往某个泛型集合中加入其它类型的数据**。例如：用**反射**得到集合，再调用其add方法即可。（**去类型化**）

**有关术语**
+ ArrayList<E>中的E类型称为**类型变量**或**类型参数**
+ 整个ArrayList<Integer>称为**参数化的类型**
+ ArrayList<Integer>中的Integer称为**类型参数的实例**或**实际类型参数**
+ ArrayList<Integer>念做**“ArrayList typeof Integer”**
+ ArrayList称作**原始类型**

**参数化的类型与原始类型的兼容性**：
 + 参数化泛型可以引用一个原始类型的参数，编译报告警告，例如：
 
 ```java
   Collection<String> c = new Vector();
 ```
 
 + 原始类型可以引用一个参数化类型的变量，编译报告警告，例如：
 
```java
   Collection c = new Vector<String>();//原来的方法接受一个集合参数，新的类型也要能传进去
```

**参数化类型不考虑类型参数的继承关系**：
```java
Vector<String> v = new Vector<Object>();//错误
Vector<Object> v = new Vector<String>();//也错误
```

**在数组创建实例时，数组的元素不能使用参数化的类型。（不能把类型变量赋给数组）**
例如：

```java
Vector<Integer> vectorList[] = new Vector<Integer>[10];//错误
```

## 泛型的 ? 通配符

```java
public static void printCollection(Collection<?> collection){
		//collection.add(1);//通配符不可以调用和类型参数有关的方法
		System.out.println(collection.size());
		for(Object obj : collection){
			System.out.println(obj);
		}
}
```
? 通配符可以引用其他各种参数化的类型。? 通配符定义的变量主要用于引用，可以调用与参数化无关的方法，不能调用与参数化有关的方法。 

**通配符的扩展**
+ 限定通配符的上边界
```java 
Vector<? extends Number> x = new Vector<Integer>();//类型参数必须是Number或Number的子类
```
+ 限定通配符的下边界
```java
Vector<? super Integer> x = new Vector<Number>();//类型参数必须是Integer的父类
``` 

不能把 ? 赋给具体的类型，只能把具体的类型赋给 ? 

## 自定义泛型
Java中的泛型类型（或者泛型）类似于C++中的模版。但是这种相似性仅限于表面，Java语言中的泛型基本上完全是在编译器中实现，用于编译器执行类型检查和类型推断，然后生成普通的非泛型字节码，这种实现技术称为***擦除(erasure)***（编译器使用泛型类型信息保证类型安全，然后在生成字节码之前将其清除）。这是因为扩展虚拟机指令集来支持泛型被认为是无法接受的，这会为Java厂商升级jvm造成难以逾越的障碍。所以，java的泛型采用了可以完全在编译器中实现的擦除方法。

用于放置泛型的类型参数尖括号应出现在方法的**其他所有修饰符之后**和在**方法的返回类型之前**，也就是**紧邻返回值之前**。按照惯例，类型参数通常用单个大写字母表示。

```java
 swap(new String[]{"abc","xyz","itcast"},1,2);
//swap(new int[]{1,3,5,4,5},3,4);//错误，泛型方法的实际参数必须是引用类型

 private static <T> void swap(T[] a,int i,int j){
		T tmp = a[i];
		a[i] = a[j];
		a[j] = tmp;
	}
```

只有引用类型才能作为泛型方法的实际参数，对于add方法，使用**基本类型**的数据进行测试没有问题，这是因为**自动装箱和拆箱**了。swap(new int[3], 3, 5)语句会报告编译错误，这是因为编译器不会对new int[3]中的int进行自动拆箱和装箱了。因为new int[3]本身已经是对象了，你想要的有可能就是int数组呢？它装箱岂不是弄巧成拙了。

除了在应用泛型时可以使用extends，在定义泛型的时候也可以，并且可以使用&来指定多个边界，如`<V extends Serializable & Cloneable> void method(){}`。

**对异常采用泛型**

也可以用类型变量表示异常，称为参数化的异常，可以用于方法的throws列表中，但是不能用于catch子句中。

```java
private static <T extends Exception> method() throws T{
	try{
	}catch(Exception e){//不能catch T 
		throw (T)e;//catch到异常后包装成另外一个异常往外扔
	}
}
```

在泛型中可以同时有多个类型参数，在定义它们的尖括号中用逗号分，例如
```java
public static<K,V> V getValue(K key){
	return map.get(key);
}
```

## 自定义泛型方法

```java
copy1(new Vector<String>(),new String[10]);
copy2(new Date[10],new String[10]);		
//copy1(new Vector<Date>(),new String[10]);//错误

public static <T> void copy1(Collection<T> dest,T[] src){}
	
public static <T> void copy2(T[] dest,T[] src){}	
```

## 自定义泛型类

```java
class ClassTest<X extends Number, Y, Z> {    
    private X x;    
    private static Y y; //编译错误，不能用在静态变量中    
    public X getFirst() {
        //正确用法        
        return x;    
    }    
    public void wrong() {        
        Z z = new Z(); //编译错误，不能创建对象    
    }
}
```

**注意：**类里的静态方法不能使用泛型类的类型，因为该泛型类的实例化对象里的类型方法采用了泛型的类型，而静态方法不应该有类型级别的参数。

## 应用：通过反射获得泛型方法接收的参数的实际类型

```java
package com.crow.Generic;

/**
 * Created by CrowHawk on 17/2/16.
 */


import java.lang.reflect.*;
import java.util.*;

public class GenericTest {
    public static void main(String[] args) throws Exception{
        HashMap<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Tom", 1);
        hashMap.put("Jerry", 2);
        hashMap.put("Nick", 3);
        printHashmap(hashMap);

        //用反射的方法获取printHashmap方法的参数并打印
        Method printMethod = GenericTest.class.getMethod("printHashmap", HashMap.class);
        Type[] types = printMethod.getGenericParameterTypes();
        ParameterizedType parameterizedType = (ParameterizedType)types[0];
        Type[] actualTypes = parameterizedType.getActualTypeArguments();
        System.out.println("The paratype is " + parameterizedType.getRawType() + "<" + actualTypes[0] + "," + actualTypes[1] + ">");
    }


    public static void printHashmap(HashMap<String, Integer> hashMap){//遍历HashMap并打印其内容
        Set<Map.Entry<String, Integer>> entrySet = hashMap.entrySet();
        for(Map.Entry<String, Integer> entry: entrySet){
            System.out.println(entry.getKey() + "," + entry.getValue());
        }
        Iterator<Map.Entry<String, Integer>> iter = entrySet.iterator();
        while(iter.hasNext()){
            Map.Entry<String, Integer> entry = iter.next();
            System.out.print("Key is " + entry.getKey() + ",");
            System.out.println("Value is " + entry.getValue());
        }
    }
}
```
