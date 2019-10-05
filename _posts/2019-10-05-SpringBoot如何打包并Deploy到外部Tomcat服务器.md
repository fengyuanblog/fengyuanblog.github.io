---
layout: post
post: true
title: SpringBoot打包并部署到外部Tomcat服务器
date: 2019-10-05
category: SpringBoot
tag: Java
author: Feng Yuan
---

* content
{: toc}

### **SpringBoot项目如何打包并部署到生产环境**

#### ***SpringBoot的运行方式和打包***

第一，SpringBoot中自带Tomcat服务器，可以直接通过内嵌的服务器直接通过IntelliJ启动运行，这样方便调试和开发；

第二，SpringBoot部署的时候，需要打包，打包方式一般可以打成jar包（**J**ava **AR**chive），或者war(**W**eb Application **AR**chive)。其中jar包仅仅包含运行java程序的基本类库，可以直接使用`java --jar xxx.jar`运行(其实，war包也可以通过同样的命令运行)，也可以用内嵌服务器直接运行。而war包则包含了完整的适用于tomcat的web文件夹结构，比如`WEB-INF`，`META-INF`等（jar包中不存在）。所以，部署到外部的tomcat只能打包成war包。

&nbsp;
<div align="center"><img src="/assets/img/2019/10/05/Jar_and_War.png" width="60%"/><p>Fig.1 Jar包和War包的区别</p></div>
&nbsp;
- 打成什么文件进行部署与项目业务有关，就像提供 rest 服务的项目需要打包成 jar文件，用命令运行很方便。。。而有大量css、js、html，且需要经常改动的项目，打成 war 包去运行比较方便，因为改动静态资源可以直接覆盖，很快看到改动后的效果，这是 jar 包不能比的（举个‘栗’子：项目打成 jar 包运行，一段时间后，前端要对其中某几个页面样式进行改动，使其更美观，那么改动几个css、html后，需要重新打成一个新的 jar 包，上传服务器并运行，这种改动频繁时很不友好，文件大时上传服务器很耗时，那么 war包就能免去这种烦恼，只要覆盖几个css与html即可）。
    &nbsp;
- war 包的本质是 IDE 工具把项目文件已某种结构进行编译与排列java类会编译成class的文件，html会放到目录中）
    - springboot 项目的war解压后，里面有3个文件夹，META-INF中有个文件记录了项目文件的引用、启动类、编译jdk版本等信息，java、html等项目文件放入WEB-INF中
   - WEB-INF目录如下，lib是项目引用的架包的目录，classes是java类编译成class文件和静态资源的地方
   - classes目录如下，com中按项目架包路径存放，templates中存放html，application.properties是配置文件，如果有css与js会创建一个static目录，里面包含css、js文件夹，分别放css与js文件，供templates目录中的html文件引用

第三，打包的工具使用maven-plugin的package或者install就可以打包了，SpringBoot默认打包成jar，如果需要打包成war包，需要在`pom.xml`中设置`<packaging>war</packaging>`。

&nbsp;
<div align="center"><img src="/assets/img/2019/10/05/SpringBoot_War_Packaging_Config.png" width="60%"/><p>Fig.2 配置打包成war</p></div>
&nbsp;

#### ***SpringBoot部署Tomcat的预处理***

第一步，为了让Tomcat可以运行SpringBoot，启动引导不能通过main函数了，需要通过SpringBootServletInitializer。所以需要修改修改启动Application文件继承SpringBootServletInitializer，重载configure方法。

{%highlight java%}
@SpringBootApplication
public class WarApplication extends SpringBootServletInitializer {

	public static void main(String[] args) {
		SpringApplication.run(WarApplication.class, args);
	}

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
		return builder.sources(WarApplication.class);
	}
}
{%endhighlight%}

第二步，移除SpringBoot内嵌的tomcat。这里有两种方法：

在`pom.xml`中添加`<scope>provided</scope>`语句，表明`spring-boot-starter-tomcat`以及`tomcat-embed-jasper`不会打包到`WEB-INF/lib`下，因为在外部tomcat运行时，这些由外部容器提供了。但是，设置成provided，maven仍然会将其打包到`WEB-INF/lib-provided`下，增加War包的体积。实际部署时候，不需要这么打包。好处是，方便调试，可以直接运行`java --jar xxx.war`启动这个war包，因为其中仍旧内嵌了tomcat。

{%highlight xml%}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
{%endhighlight%}

在`pom.xml`中设置exclusion直接排除掉tomcat，这样虽然无法再使用`java --jar xxx.war`启动这个war包，但是，缩小了war包的体积。

{%highlight xml%}
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<!-- 移除嵌入式tomcat插件 -->
	<exclusions>
	    <exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
	   </exclusion>
    </exclusions>
</dependency>
{%endhighlight%}

**实际部署中，推荐使用第二种方法**

对打成的war包进行重命名，通过添加`<finalName>OwnName</finalName>`来把目标的war包重命名成`OwnName`。

{%highlight xml%}
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
    <finalName>helloworld</finalName>
</build>
{%endhighlight%}

#### ***SpringBoot部署到Tomcat的方法***

方法一，*使用Tomcat自带的Manager部署war包*

使用默认的Tomcat Manager来部署war包，好处是方便，有GUI可以操作，可以实现热加载。但是在生产环境下，是不带这个manager的，所以这种方式在开发环境下是很方便，但是在生产环境下就不适用了。

方法二，*手工部署War包*

直接将打包好的war包放在tomcat的appBase目录下就好了，注意，之后需要重启tomcat server才可以，tomcat默认设置下会把war包自动的解压。在`conf/server.xml`中的<Host>选项中设置`unpackWARs="true"`和`autoDeploy="true"`来开启这种功能。比如，war包的名字是，`hello.war`那么会在appBase文件夹下自动unpack出一个`hello`文件夹，这个文件夹下面包含`WEB-INF`，`org`,`META-INF`三个文件夹。访问这个war包中的服务的时候，需要在url中加上`hello`前缀，比如:`http://localhost:8080/<resourceName>`需要改成`http://localhost:8080/hello/<resourceName>`。

但是，这种方法比较麻烦，不支持热部署，每次部署之后都需要重新启动tomcat。为了使用tomcat的热部署，可以在`tomcat/conf/server.xml`中的`<Host>`中添加`<context/>`标签，并设置`reloadable="true"`。如下：

{%highlight xml%}
<!-- 实现tomcat热部署和自定义ContextPath-->
<Context docBase="myPrj " path="/demo1" reloadable="true"/>
{%endhighlight%}

docBase: webapps下的你项目的包名，path：项目访问路径，reloadable: 是否开启热加载。