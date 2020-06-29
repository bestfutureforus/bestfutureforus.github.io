---
layout:     post
title:      "设计模式(3)——装饰者模式"
subtitle:   ""
date:       2017-07-20 22:00:03
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 设计模式
    - 读书笔记
---

**装饰者（Decorator）模式**动态地将责任附加到对象上，若要扩展功能，装饰者提供了比继承更有弹性的替代方法。
+ 装饰者和被装饰对象有相同的超类型。
+ 你可以用一个或多个装饰者包装一个对象。
+ 既然装饰者和被装饰者对象有相同的超类型，所以在任何需要原始对象（被包装的）的场合，可以用装饰过的对象代替它。
+ **装饰者可以在所委托被装饰者的行为前后，加上自己的行为，以达到特定的目的。**
+ 对象可以在任何时候被装饰，所以可以在运行时动态地、不限量地用你喜欢的装饰者来装饰对象。

装饰者模式的类图如下
![](http://pic.yupoo.com/crowhawk/GBCAH6Me/T4EiB.jpg)

这里是我的Github中观察者模式的[示例代码](https://github.com/CrowHawk/DesignPattern-Learning/tree/master/Decorator/src)。
该代码模拟了咖啡店的菜单。以饮料Beverage类为基类，向饮料中加入不同的调料就会成为不同的咖啡，加入调料的过程就是装饰者模式。

此外，Java I/O的源码实现也用到了装饰者模式，这是一个笔者使用装饰者模式自定义的IOStream类，把输入流的所有大写字符换成小写。
```java
import java.io.FilterInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * Created by CrowHawk on 17/7/10.
 */
public class LowerCaseInputStream extends FilterInputStream {//JavaI/O设计时使用了装饰者模式，编写自己的JavaI/O装饰者，把输入流的所有大写字符换成小写
    public LowerCaseInputStream(InputStream in) {
        super(in);
    }

    public int read() throws IOException {
        int c = super.read();
        return (c == -1 ? c : Character.toLowerCase((char)c));
    }

    public int read(byte[] b, int offset, int len) throws IOException {
        int result = super.read(b, offset, len);
        for (int i = offset; i < offset + result; i++) {
            b[i] = (byte)Character.toLowerCase((char)b[i]);
        }
        return result;
    }
}
```
```java
import com.crow.JavaIO.LowerCaseInputStream;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * Created by CrowHawk on 17/7/10.
 */
public class InputTest {
    public static void main(String[] args) throws IOException {
        int c;
        try {
            InputStream in = new LowerCaseInputStream(new BufferedInputStream(new FileInputStream("test.txt")));

            while((c = in.read()) >= 0) {
                System.out.print((char)c);
            }

            in.close();
        }catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

装饰者模式用到了我们的第五条设计原则：**类应该对扩展开放，对修改关闭**。我们的目标是允许类容易扩展，在不修改现有代码的情况下，就可搭配新的行为。这样的设计具有弹性，可以应对改变，可以接受新的功能来应对改变的需求。
