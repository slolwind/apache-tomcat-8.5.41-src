maven构建tomcat源码
去tomcat官网下载源码，这里面选择的版本是8.0

https://tomcat.apache.org/download-80.cgi

选择sourceCode下载 地址为：http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.41/src/apache-tomcat-8.5.41-src.zip

有两个种环境方式，zip和tar两种试 一种是windows一种是linux的

<meta charset="utf-8">

clipboard.png
1 两种方式构建 一种是通过ant进行构建 http://ant.apache.org/

需要下载 1.9.8以下版本的ant ，以windows版本为例

clipboard-1.png
这里选的是1.10.6稳定版

clipboard-2.png
环境变量需要对ant进行配置：新增变量名 ANT_HOME 对应相应的目录

创建path:

clipboard-4.png
clipboard-5.png
校验是否配置成功 ant -version

clipboard-6.png
在源码的目录直接运行ant命令 会对build.xml进行构建

clipboard-7.png
clipboard-8.png
具体可以看下根目录的文件：BUILDING.txt 说得很详细

至此整个源码构建完成。

第二个构建方案通过转换为maven的项目进行构建，maven其实与ant一样。与原理也是一样的

将源码导入idea中 并在根目录创建pom.xml 和calina_home目录 如下图

clipboard-9.png
pom.xml具体的内容如下：通过jdk1.8进行编译

<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0"

xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

<modelVersion>4.0.0</modelVersion>

<groupId>org.apache.tomcat</groupId>

<artifactId>Tomcat8.0</artifactId>

<name>Tomcat8.0</name>

<version>8.0</version>

<build>

<finalName>Tomcat8.0</finalName>

<sourceDirectory>java</sourceDirectory>

<resources>

<resource>

<directory>java</directory>

</resource>

</resources>

<testResources>

<testResource>

<directory>test</directory>

</testResource>

</testResources>

<plugins>

<plugin>

<groupId>org.apache.maven.plugins</groupId>

<artifactId>maven-compiler-plugin</artifactId>

<version>2.3</version>

<configuration>

<encoding>UTF-8</encoding>

<source>1.8</source>

<target>1.8</target>

</configuration>

</plugin>

</plugins>

</build>

<dependencies>

<dependency>

<groupId>junit</groupId>

<artifactId>junit</artifactId>

<version>4.12</version>

<scope>test</scope>

</dependency>

<dependency>

<groupId>org.easymock</groupId>

<artifactId>easymock</artifactId>

<version>3.4</version>

</dependency>

<dependency>

<groupId>ant</groupId>

<artifactId>ant</artifactId>

<version>1.7.0</version>

</dependency>

<dependency>

<groupId>wsdl4j</groupId>

<artifactId>wsdl4j</artifactId>

<version>1.6.2</version>

</dependency>

<dependency>

<groupId>javax.xml</groupId>

<artifactId>jaxrpc</artifactId>

<version>1.1</version>

</dependency>

<dependency>

<groupId>org.eclipse.jdt.core.compiler</groupId>

<artifactId>ecj</artifactId>

<version>4.5.1</version>

</dependency>

</dependencies>

</project>

此时idea不会自动对源项目代码转化为maven工程项目。需要手动引入。如下图

选择+号 导入pom即可 。

clipboard-10.png
此时idea 做自动检测到tomcat启动类 Bootstrap。

clipboard-11.png
需要对此添加一些VM运行参数。主要是构建的目标路径 catalina-home

-Dcatalina.home=catalina-home -Dcatalina.base=catalina-home -Djava.endorsed.dirs=catalina-home/endorsed -Djava.io.tmpdir=catalina-home/temp -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file=catalina-home/conf/logging.properties

clipboard-12.png
clean 与install 构建成功

clipboard-13.png
开始启动主导BootStrap.jva

这里面可能会遇到一些问题：如下图

Error:osgi: [apache-tomcat-8.5.20-src] Invalid value for Bundle-Version, @VERSION@ does not match [0-9]{1,9}(.[0-9]{1,9}(.[0-9]{1,9}(.[0-9A-Za-z_-]+)?)?)?

[图片上传失败...(image-1229e4-1559559372234)]


此时需要做的就是

如下 Bundle-Version: @VERSION@ 改为 Bundle-Version: 1.1

clipboard-16.png
Manifest-Version: 1.0

Export-Package: org.apache.tomcat.jdbc.naming;uses:="javax.naming,org.ap

ache.juli.logging,javax.naming.spi";version="@VERSION@",org.apache.tomc

at.jdbc.pool;uses:="org.apache.juli.logging,javax.sql,org.apache.tomcat

.jdbc.pool.jmx,javax.management,javax.naming,javax.naming.spi,org.apach

e.tomcat.jdbc.pool.interceptor";version="@VERSION@",org.apache.tomcat.j

dbc.pool.interceptor;uses:="org.apache.tomcat.jdbc.pool,org.apache.juli

.logging,javax.management.openmbean,javax.management";version="@VERSION@

",org.apache.tomcat.jdbc.pool.jmx;uses:="org.apache.tomcat.jdbc.pool,or

g.apache.juli.logging,javax.management";version="@VERSION@"

Bundle-Vendor: Apache Software Foundation

Bundle-Version: 1.1

Bundle-Name: Apache Tomcat JDBC Connection Pool

Bundle-ManifestVersion: 2

Bundle-SymbolicName: org.apache.tomcat.jdbc

Import-Package:

javax.management;version="0",

javax.management.openmbean;version="0",

javax.naming;version="0",

javax.naming.spi;version="0",

javax.sql;version="0",

org.apache.juli.logging;version="0"

重新启动成功！ 直接进入main方法

clipboard-17.png
启动需要花费一些时间。默认端口是8080

clipboard-18.png
下面可以进入tomcat的源码解析环节了。
