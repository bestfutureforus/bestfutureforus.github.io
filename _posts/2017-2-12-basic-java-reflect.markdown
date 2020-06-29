---
layout:     post
title:      "Java基础学习(1)——反射"
subtitle:   ""
date:       2017-02-12 12:00:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

> 反射就是把Java类中的各种成分映射成相应的Java类（主要用于框架开发）

---

### 反射的基石-->Class类
Java程序中的各个类属于同一事物，描述这类事务的Java类名就是**Class**。
Class类描述了哪些信息？

+ 类的名字
+ 类的访问属性
+ 类所属于的包名
+ 字段名称的列表
+ 方法名称的列表
+ ...

如何得到各个字节码对应的实例对象(Class类型)?

1、 `类名.class`

```java
Class cls1 = Date.class;//获取字节码
```
	
2、 `对象.getClass()`

```java	
new Date().getClass();//获取字节码
```

3、 用静态方法`Class.forName("类名")`去得到字符串对应的类的字节码

```java	
Class.forName("java.util.Date")//forName是Class类的静态方法，写类名时一定要写出包名
```
	
+ 若该类的字节码已经加载到内存中，直接返回
+ 若jvm里还没有该类的字节码，则用类加载器去加载，加载以后把字节码缓存起来，方法返回该字节码   
	
作反射的时候主要用第三种方法,因为编写框架的时候不知道用户定义的类的名字，在运行的时候传递一个字符串，字符串中包括一个类名，在程序运行时临时传进来

```java
String str1 = "abc";
Class cls1 = str1.getClass();
Class cls2 = String.class;
Class cls3 = Class.forName("java.lang.String");//一定要写出包名
```		
	
三种方法获取的字节码相同 

**注意1：字节码的比较都使用 == 而不是 equals**

**注意2：一个Class对象实际上表示的是一个类型，而这个类型未必一定是一种类，也有可能是Java基础类型。**例如int不是类，但int.class是一个Class类型的对象。


---

### 反射



#### Constructor类: 构造方法的反射


+ Constructor类代表某个类中的一个构造方法
    
- **用反射的方法创建一个对象**
 
**class -> constructor -> new object**

```java
Constructor constructor = String.class.getConstructor(StringBuffer.class);//得到构造方法的时候要注意参数类型StringBuffer
String str = (String)constructor.newInstance(new StringBuffer("Hello world!")//查阅Java API，该方法返回值为Object类型，需要强制转换为String
```

  
#### Field类: 成员变量的反射
 
+ Field类代表某个类中的一个成员变量
 
 [Crow's Github](https://github.com/CrowHawk/JavaLearning/tree/master/Basic-java/src/com/crow/Reflect)给出了一个通过Field类修改类的实例域的Demo
 截取其中部分代码如下:
 
```java		
public static void refFieldChange(FieldReflect fr, String fieldName) throws Exception{
//使用Field类改变类的实例域,如果是int型,全部设为3,如果是String类型,将String中的"b"改为"c"
	Field field = fr.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);//暴力反射
        if(field.getType() == int.class){
            field.set(fr, 3);
        }
        else if(field.getType() == String.class) {
            String string = (String) field.get(fr);
            string.replace("b", "c");
            field.set(fr, string);
        }
        else {
            throw new IOException();
        }
}
```
 
应注意:			
fr.getField()只能得到可见字段，若遇到private等不可见字段，应使用getDeclaredField()方法

```java	
Filed fieldY = pt1.class.getDeclaredField(fieldName);
```
	
field.get(fr)也只能得到可见字段，private不可获取，此时可采用**暴力反射**方法，先执行

```java		
field.setAccessible(true);//暴力反射
```

再使用field.get(fr)
	
	
---

#### Method类: 成员方法的反射

```java	
//str1.charAt
Method methodCharAt = String.class.getMethod("charAt", int.class);
System.out.println(methodCharAt.invoke(str1,1));
```
	
先用Method获得某个类的某个方法，再用得到的这个方法去作用于某个对象

方法`methodCharAt.invoke(null, 1)`就是静态方法，因为静态方法调用的时候不需要对象

##### 对接收数组参数的成员方法进行反射
例如:根据用户提供的类名，去执行该类中的main方法，其中main方法接收的参数为数组。

```java
class TestArguments{
	public static void main(String[] args){
		for(String arg : args){
			System.out.println(arg);
		}
	}
}
```

一般的调用方法是

```java	
TestArguments.main(new String[]{"Hello","world","!"});
```	

使用反射方法(用Method类)是

```java
Method mainMethod = Class.forname("startingClassName").getMethod("main", String[].class);
mainMethod.invoke(null, new Object[]{new String[]{"Hello", "world", "!"}});
```

**注意:main方法要接受一个类型为String[]的参数。为了兼容不具备可变参数的老版本代码，给main函数输入一个参数（一个三元数组），会自动拆分为三个参数，会出现wrong number of arguments错误，因此要用Object[]包装起来，只拆分为一个参数，即一个三元数组。**
	
---
	
#### 数组的反射

##### 数组与Object的关系及其反射类型
具有相同的**数据类型**和**维度**的数组 的 **反射** 是相同的
		
```java
int [] a1 = new int[]{1,2,3};
int [] a2 = new int[4];
int[][] a3 = new int[2][3];
String [] a4 = new String[]{"a","b","c"}; 
		            
Object aObj1 = a1;   //正确，因为Int数组是Object
Object aObj2 = a4;
//Object[] aObj3 = a1; 错误，因为基本类型Int不是Object
Object[] aObj4 = a3;  //正确，因为Int数组是Object
Object[] aObj5 = a4;
```

---


通过反射可以得到数组中每一个元素的类型:  

```java	
Object[] clsArray = new Object[]("a", 1);
clsArray[0].getClass().getName();
```

（但我暂时不知道Java如何得到数组的类型，还望知道的读者不吝赐教）

##### 数组反射应用实例: 打印数组里的所有元素

```java
private static void printObject(Object obj) {//打印数组里的所有元素
Class clazz = obj.getClass();
	if(clazz.isArray()){
		int len = Array.getLength(obj);
		for(int i=0;i<len;i++){
			System.out.println(Array.get(obj, i));
		}
	}
	else{
		System.out.println(obj);
	}	
}
```
