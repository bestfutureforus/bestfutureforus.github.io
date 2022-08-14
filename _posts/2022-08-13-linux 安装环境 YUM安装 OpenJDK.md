---
layout: post
title: "JDK安装"
date: 2022-04-13
description: "JDK安装"
tag: hexo
---   
## JDK安装

#1.查看JDK版本

    yum search java | grep jdk


#2.安装openSDK

    uname -a 查看系统版本 x86_x64

    yum install java-1.8.0-openjdk-devel.x86_64

#3.查看安装路径

    which java  查看路径

    超链接指向：ls -lrt /etc/alternatives/java

    实际为：/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.amzn2.0.2.x86_64/jre/bin/java

#4. vim /etc/profile

    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.amzn2.0.2.x86_64

    JRE_HOME=$JAVA_HOME/jre

    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib

    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

    export PATH  JAVA_HOME JRE_HOME CLASS_PATH

#5.环境变量生效

    source /etc/profile

#6. nohup sh start.sh >/dev/null 2>&1 &

    1、nohup &

    nohup 表示永久运行，& 表示后台运行。

    2、>/dev/null 2>&1

    /dev/null 代表空设备文件，也就是不输出任何信息到终端。

    操作系统中有三个常用的流：　　0：标准输入流 stdin　　1：标准输出流 stdout　　2：标准错误流 stderr

    ">/dev/null" 等价于 "1>/dev/null"，表示标准输出（1）输出到 /dev/null 中，即终端不输出标准输出信息；

    "2>&1" 中的 “&” 是等价于的意思，表示 标准错误（2）输出的位置 等价于 标准输出（1）的位置，即等价于 “2>/dev/null”， 即终端不输出标准错误信息。

    因此，">/dev/null 2>&1" 表示 标准错误信息和标准输出信息，在终端上均不输出。



    


    




