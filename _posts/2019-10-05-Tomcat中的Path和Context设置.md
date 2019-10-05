---
layout: post
post: true
title: Tomcat中关于Context-Path的迷思
date: 2019-10-05
category: Tomcat
tag: Java_Tomcat_SpringBoot
author: Feng Yuan
---

* content
{: toc}

### **Tomcat以及Spring框架中URL如何定位资源**

可以发现，在Tomcat中有以下几个地方可以设置Path这个参数，`tomcat/conf/server.xml`中的`<Context/>`元素,`tomcat/conf/Catalina/localhost/xxx.xml`文件中的`<Context/>`元素,在Spring项目中，在`application.properties`中可以设置`server.servlet.context-path`。

最终的Spring程序中的URL定位资源的path是遵循如下原则的：

*不同的Tomcat的配置互不干扰*

`application.properties`中的设置仅仅在SpringBoot内嵌的tomcat中起作用，这个文件的作用就是配置开发环境下的Tomcat参数。当部署在外部Tomcat中的时候，虽然这个文件会被打包到WAR包中，但是，并不会影响外部Tomcat的配置。也就是说，不同的Tomcat的配置互不影响，各自独立。

*同一个Tomcat中，遵循：由大到小，显式设置高于隐式设置*

在同一个Tomcat中，也有多个地方可以设置Context Path参数

**由大到小原则：** URL定位资源，首先要遵从Tomcat的设置，之后，连接上Web服务程序中的Resource路径定义。比如，Spring中使用了注解`@RequestMapping('/resource')`来匹配URL，那么，浏览器中的URL实际需要写：`http://localhost:8080/<Server-defined-Context-Path>/resource`。

**显式高于隐式原则：** 上面中的`<Server-defined-Context-Path>`部分由JavaWeb程序运行的Tomcat容器决定。官方的说明如下：

>[The path] attribute must only be used when statically defining a Context in server.xml. In all other circumstances, the path will be inferred from the filenames used for either the .xml context file or the docBase.

也就是说，明确的通过`server.xml`中的<Context/>标签进行的Path设置，要优先于其他的隐式指定方式。

第一优先级，*显式设置方式*，利用`tomcat/conf/server.xml`中的`<Context/>`元素：

{%highlight xml%}
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context docBase="/absolute/path/to/project/war/unpacked/folder" path="/app1" reloadable="true"/>
</Host>
{%endhighlight%}
&nbsp;

其中，docBase表示的是WebApp的文件夹在OS的文件系统上的绝对路径（也可以是相对于appBase的相对路径），换句话说，就是指向war展开后的目录。path表示的URL中的`<Server-defined-Context-Path>`部分，在上述设置下，URL变成`http://localhost:8080/app1/resource`。

第二优先级，*隐式设置方式*，利用`tomcat/conf/Catalina/localhost/xxx.xml`文件中的`<Context/>`元素。

这种情况下，URL的Server部分通过**文件名**来隐式确定。比如，使用了`app2.xml`这个文件名，内容如下所示：

{%highlight xml%}
<?xml version='1.0' encoding='utf-8'?>
<Context docBase="/absolute/path/to/project/war/unpacked/folder" path="/ignored" privileged="true" reloadable="true"/>
{%endhighlight%}
&nbsp;

这个元素中的`path`不起作用，只有`docBase`起作用，指向实际的war展开后的folder，`path`部分被忽略。实际的URL通过`app2.xml`确定成`http://localhost:8080/app2/resource`，而**不是**`http://localhost:8080/ignored/resource`。

以上两种方式，可以同时存在（Tomcat会同时允许两个URL），互不冲突，因为是直接明确的写入了Tomcat的配置文件夹里面的。

如果**没有**如上明确的配置，那么`<Server-defined-Context-Path>`就通过WebApp的War包的路径进行定位。`app3.war`这个文件**必须**放在`appBase`文件夹下（也就是根目录下），然后，自动展开。比如，war包展开到如下路径：`appBase/app3`（这个`app3`中包含了`WEB-INF`，`META-INF`，等文件夹）,那么，URL就会定位到: `http://localhost:8080/app3/resource`。隐式方式不够智能，因为，如果不放在`appBase`这个根目录下面，而是更深层的目录里面，Tomcat**不会**扫描进去，也就会找不到目录，出现`Not Found`（这种情况通过显式设置，指明docBase就可以访问到了）。

这种方式是最后的隐式方式，*只有*在没有明确的configure服务器的Context Path的时候才会起作用。

隐式方式的问题也很明显：
1. 只能放在Tomcat的`appBase`指定的根目录下（放深了就找不到了）；
2. war包的名字跟URL的prefix绑定（一样了）；war包的名字默认带了version，SNAPSHOT等名字，很复杂，如果不注意改，将会出现在URL中；

基于以上的问题，配置Tomcat的时候，最好是通过`server.xml`中进行显示设置，最高优先级，可以通过docBase指定WebApp文件夹的位置，也可以指定URL的prefix，两者可以独立设置，互不影响。
