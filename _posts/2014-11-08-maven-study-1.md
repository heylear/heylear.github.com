--- 
layout: post 
title: "【转】Apache Maven 入门(上)" 
date: 2014-11-08  
categories: 构建
tags: maven java
--- 

写这个 maven 的入门篇是因为之前在一个开发者会的动手实验中发现挺多人对于 maven 不是那么了解，所以就有了这个想法。
这个入门篇分上下两篇。本文着重动手，用 maven 来构建运行 hellow world 程序，体会一下不用任何 IDE ，只用 maven 是咋回事。然后下篇就讲解一下 maven 的核心概念。写这两篇文章特意回避了复杂的示例，也不使用 IDE ，目的是排除干扰，着重于 maven 本身。

本文的源代码可从这里下载。  

### Apache Maven 是做什么用的？

* Maven 是一个项目管理和构建自动化工具。但是对于我们程序员来说，我们最关心的是它的项目构建功能。所以这里我们介绍的就是怎样用 maven 来满足我们项目的日常需要。

* Maven 使用惯例优于配置的原则 。它要求在没有定制之前，所有的项目都有如下的结构：
 

| 目录        | 作用   |
| --------   | -----  |
| ${basedir}     | pom.xml和所有的子目录 |
| ${basedir}/src/main/java        |   项目的 java源代码   |
| ${basedir}/src/main/resources        |    项目的资源，比如说 property文件    |
| ${basedir}/src/test/java          |	项目的测试类，比如说 JUnit代码	|
| ${basedir}/src/test/resources		|	测试使用的资源		|
 

一个 maven 项目在默认情况下会产生 JAR 文件，另外 ，编译后 的 classes 会放在 ${basedir}/target/classes 下面， JAR 文件会放在 ${basedir}/target 下面。
这时有人会说了 ， Ant 就没有那么多要求 ，它允许你可以自由的定义项目的结构。在这里不想引起口水战哈， 我个人觉得 maven 的这些默认定义很方便使用。 
好了 ，接下来我们来安装 maven 。

 

### Maven 的安装
 

在安装 maven 前，先保证你安装了 JDK 。 JDK 6 可以从 Oracle 技术网上下载：

<http://www.oracle.com/technetwork/cn/java/javase/downloads/index.html>。

Maven 官网的下载链接是 : 

<http://maven.apache.org/download.html> 。

该页的最后给出了安装指南。


安装完成后，在命令行运行下面的命令：  
   
`$ mvn -v`
``` 
Apache Maven 3.0.3 (r1075438; 2011-03-01 01:31:09+0800)
Maven home: /home/limin/bin/maven3
Java version: 1.6.0_24, vendor: Sun Microsystems Inc.
Java home: /home/limin/bin/jdk1.6.0_24/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.35-28-generic-pae", arch: "i386", family: "unix"
```

如果你看到类似上面的输出的话，就说明安装成功了。 
接下来我们用 maven 来建立最著名的“Hello World!”程序 :)
注意：如果你是第一次运行 maven，你需要 Internet 连接，因为 maven 需要从网上下载需要的插件。    
我们要做的第一步是建立一个 maven 项目。在 maven 中，我们是执行 maven 目标 (goal) 来做事情的。
maven 目标和 ant 的 target 差不多。在命令行中执行下面的命令来建立我们的 hello world 项目
  

  `~$mvn archetype:generate -DgroupId=com.mycompany.helloworld -DartifactId=helloworld -Dpackage=com.mycompany.helloworld -Dversion=1.0-SNAPSHOT`


* archetype:generate 目标会列出一系列的 archetype 让你选择。 Archetype 可以理解成项目的模型。 Maven 为我们提供了很多种的项目模型，包括从简单的 Swing 到复杂的 Web 应用。我们选择默认的 maven-archetype-quickstart ，是编号 #106 

* 连打两个回车，这时候让你确定项目属性的配置，这些属性是我们在命令行中用 -D 选项指定的。该选项使用 -Dname=value 的格式。回车确认，就完成了项目的建立
  

* maven 的 archetype 插件建立了一个 helloworld 目录，这个名字来自 artifactId 。在这个目录下面，有一个 Project Object Model(POM) 文件 pom.xml 。这个文件用于描述项目，配置插件和管理依赖关系。源代码和资源文件放在 src/main 下面，而测试代码和资源放在 src/test 下面。

Maven 已经为我们建立了一个 App.java 文件：

Java代码:
```
    package com.mycompany.helloworld;   
      
    /**  
     * Hello world!  
     *  
     */   
    public class App {   

      
        public static void main( String[] args ) {   
            System.out.println( "Hello World!" );   
        }   
    }  
``` 

 正是我们需要的 Hello World 代码。所以我们可以构建和运行这个程序了。用下面简单的命令构建：

`~$cd helloworld`

`~$mvn package `


* 当你第一次运行 maven 的时候，它会从网上的 maven 库 (repository) 下载需要的程序，存放在你电脑的本地库 (local repository) 中，所以这个时候你需要有Internet 连接。Maven 默认的本地库是 ~/.m2/repository/ ，在 Windows 下是 %USER_HOME%\.m2\repository\ 。
 

* 这个时候， maven 在 helloworld 下面建立了一个新的目录 target/ ，构建打包后的 jar 文件 helloworld-1.0-SNAPSHOT.jar 就存放在这个目录下。编译后的 class 文件放在 target/classes/ 目录下面，测试 class 文件放在 target/test-classes/ 目录下面。

 为了验证我们的程序能运行，执行下面的命令：

 `~$java -cp target/helloworld-1.0-SNAPSHOT.jar com.mycompany.helloworld.App`
