---
layout: post
title: Customize MySQL JDBC Driver
description: "This is how to customize mysql jdbc driver."
modified: 2014-03-13
tags: [mysql jdbc customize, mysql, jdbc]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true  
---

我们有时候需要对MySQL的驱动进行修改，加入自己的东西，如监控平台可能需要加入一些埋点信息。在修改代码方面可能毫无问题，但是在打包编译这块，往往会搞不定，在eclipse中打开源代码也会报各种错误。接下来，我会介绍如何在更改MySQL JDBC源代码后如何打包.

### 源码下载

1. 从[Oracle官方网站](http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.29.zip)下载最新版的MySQL JDBC Driver.
2. 本地解压后，目录如下：
  {% highlight html %}
  |--docs
      |--connector-html
      |--connector-j.pdf
      |--README.txt
  |--src
      |--com
      |--doc
      |--lib
      |--org
      |--testsuite
  |--mysql-connector-java-5.1.29-bin.jar
  |--build.xml
  |--CHANGES
  |--README
  |--README
  |--README.txt
  {% endhighlight %}
2. 在解压目录中，`build.xml`是与ant编译打包相关的文件，`src/com`是与MySQL相关的源代码，`src/lib`是MySQL源代码依赖的库

### 如何编译

1. 由于MySQL需要兼容JDK不同的版本，所以，如果你直接使用`ant`编译时无法编译通过的.
2. 你需要把你的系统环境的JDK版本设置成`JDK1.5`，在配置环境变量之前，你需要安装`JDK1.5`
  {% highlight html %}
  JAVA_HOME:D:\Program Files (x86)\Java\jdk1.5.0_22
  CLASSPATH:.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
  PATH:%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
  {% endhighlight %}
3. 为了兼容JDK1.6版本，你需要同时安装`JDK1.6`，并打开`build.xml`文件，修改配置如下：
  {% highlight xml %}
  <property name="com.mysql.jdbc.java6.java"  value="D:/Program Files (x86)/Java/jdk1.6.0_38/bin/java.exe" />
  <property name="com.mysql.jdbc.java6.javac" value="D:/Program Files (x86)/Java/jdk1.6.0_38/bin/javac.exe" />
  <property name="com.mysql.jdbc.java6.rtjar" value="D:/Program Files (x86)/Java/jdk1.6.0_38/jre/lib/rt.jar" />
  {% endhighlight %}
4. 如果修改代码需要依赖其他类库的话，只需要把`Jar包`放到`src/lib`目录下即可，`Jar包`必须编译成`JDK1.5`版本，如果是`JDK1.6`回报错误.
5. 打开`cmd`， 切换到MySQL工程目录，运行`ant`命令，查看编译结果.
6. 编译成功后，在MySQL工程目录下会新增`target`文件夹，编译后的jar包就在这个目录下.

### Eclipse中打开源代码

如果想要在Eclipse中打开并修改MySQL的源代码，可以按照以下步骤：

1. 在Eclipse中新建工程`mysql-connector-java`
2. 在本地目录下，将`MySQL的源代码`拷贝到`mysql-connector-java`工程下
3. 在Eclipse中，右键`mysql-connector-java`，选择`Build Path`->`Configure Build Path...`，在`Libraries`栏目下点击`Add JARs...`导入工程下`src/lib`下的所有Jar包.
4. 由于MySQL源代码需要支持JDK1.5和JDK1.6，所以在Eclipse中打开的源代码会包很多错误，这个不要管，只要`ANT`能够编译成功就行了，至于怎么做到的，我还不知道，请有经验的人解答.
5. 所以，可以通过在Eclipse中修改源代码，用ant来编译来实现定制MySQL Java Driver

我已经将Eclipse中的源代码上传至`Github`，如果需要，可以前去[下载](https://github.com/dingshuangxi888/mysql-connector-java-eclipse)我的版本.

### 上传至私服

可以将编译后的MySQL版本上传至私服中，以下是将Jar上传至`Nexus私服`的例子
  {% highlight html %}
  mvn deploy:deploy-file 
      -DgroupId=mysql 
      -DartifactId=mysql-connector-java 
      -Dversion=5.1.29 
      -Dpackaging=jar 
      -Dfile=./build/mysql-connector-java-5.1.29-SNAPSHOT/mysql-connector-java-5.1.29-SNAPSHOT-bin.jar 
      -Dclassifier=microscope-1-2-3-04 
      -Dtype= -Durl=http://10.101.0.2:8081/nexus/content/repositories/thirdparty/ 
      -DrepositoryId=thirdparty
  {% endhighlight %}