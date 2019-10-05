---
layout: post
post: true
title: 同一个Tomcat上运行多个不同（或相同）的Service
date: 2019-10-05
category: Tomcat
tag: Java_Tomcat_SpringBoot
author: Feng Yuan
---

* content
{: toc}

### **使用一个Tomcat，来运行多个不同的Web Services**

在`tomcat/conf/server.xml`文件中，`<Service></Service>`标签标示了在同一个Tomcat下不同的Service。为什么要使用同一个Tomcat处理多个Web Apps呢？当然，最好的方式是搭建Tomcat集群，使用不同的Tomcat，每个Tomcat在不同的Host上跑。但是，可以用一个Tomcat跑多个WebApp的方式模拟出这种场景。适用于利用Nginx做反向代理的负载均衡。

下面就定义了三个不同的Web Service，分别跑在三个不同的端口上：8081,8082,8083。


{%highlight xml%}
<Service name="Catalina">
  <!-- Another Service Running on a different port -->
    <Connector port="8081" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />

    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <Engine name="Catalina" defaultHost="tomcathost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
        </Realm>
        <Host name="tomcathost"  appBase="/media/work/Workspace/Personal_Projects/tomcatroot" unpackWARs="true" autoDeploy="true">
        <!-- Project Name and Context Prefix -->
            <Context docBase="demo/helloworld.war" path="/demo/helloworld" reloadable="true" />
            <Valve  className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                    prefix="tomcat_access_log." suffix=".txt"
                    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        </Host>
        <Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For"
                protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https" />
    </Engine>
</Service>

<Service name="Catalina">
  <!-- Another Service Running on a different port -->
    <Connector port="8082" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />

    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <Engine name="Catalina" defaultHost="tomcathost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
        </Realm>
        <Host name="tomcathost"  appBase="/media/work/Workspace/Personal_Projects/tomcatroot" unpackWARs="true" autoDeploy="true">
        <!-- Project Name and Context Prefix -->
            <Context docBase="demo/helloworld.war" path="/demo/helloworld" reloadable="true" />
            <Valve  className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                    prefix="tomcat_access_log." suffix=".txt"
                    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        </Host>
        <Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For"
                protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https" />
    </Engine>
</Service>

<Service name="Catalina">
  <!-- Another Service Running on a different port -->
    <Connector port="8083" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />

    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <Engine name="Catalina" defaultHost="tomcathost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
        </Realm>
        <Host name="tomcathost"  appBase="/media/work/Workspace/Personal_Projects/tomcatroot" unpackWARs="true" autoDeploy="true">
        <!-- Project Name and Context Prefix -->
            <Context docBase="demo/helloworld.war" path="/demo/helloworld" reloadable="true" />
            <Valve  className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                    prefix="tomcat_access_log." suffix=".txt"
                    pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        </Host>
        <Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For"
                protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https" />
    </Engine>
</Service>
{%endhighlight%}
&nbsp;


注意在Nginx处理反向代理的过程中，Upstream的设置，每一个server的地址可以使用`hostname:port`，也可以使用`ip:port`。如果使用`hostname:port`的格式，Nginx在发送给代理服务器之前，需要DNS把hostname转换成ip地址，所以最好是使用ip地址。如果Tomcat在配置过程中，同一个Service下定义了多个不同的hostname来区分Service，就容易出问题。Nginx转成ip地址后，传给Tomcat, Tomcat会发现没有对应的hostname，只能使用default的hostname，会造成所有的请求都导向了默认的web app，而不是不同的app。所以，不同的app最好是区分开来。
