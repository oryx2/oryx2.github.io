---
layout: post
title:  "加载war包里WEB-INF下的资源文件"
date:   2015-05-08 
categories: java classloader
---

通过classloader读取war包下WEB-INF里的文件失败

* 对于web,一般都通过servletcontext.getresource来读取，它是如何读到的？
web容器在解压war包时会产生一个baseResource ,路径为file:/web/src/main/webapp/
servletContext.getResource(“path”)获取文件时，通过baseResource+path得到绝对路径，用File读取文件。

* 另外一些eclipse插件如jettyrun,启动后通过classloader却能读到class以外的文件。
首先分析一下classloader.readerResource，jetty的classloader实现类为
org.eclipse.jetty.webapp.WebAppClassLoader，为URLClassLoader的扩展实现，资源的查询路径都存储在URLClassLoader中的ucp变量中(类型为 URLClassPath),可以通过以下方式获取：

{% highlight java %}
URLClassLoader classloader = (URLClassLoader)this.getClass().getClassLoader();
classloader.getURLs();

{% endhighlight%}

在ide中启动应用可以看到得到的url路径为
`file:/Users/oryx/hobby/workspace/java/ips/com.test.web/bin/`
web-inf目录为
`file:/Users/oryx/hobby/workspace/java/ips/com.test.web/bin/WEB-INF/a.properties`

此时web-inf目录下的资源在classloader搜索范围内，所以通过classloader是可以读取到文件的。

打成war在jetty中启动得到的url路径为
`file:/Users/oryx/hobby/jetty/webapps/com.test.web-1.0.0.RELEASE/WEB-INF/classes/`
web-inf目录为
`file:/Users/oryx/hobby/jetty/webapps/com.test.web-1.0.0.RELEASE/WEB-INF/a.properties`

此时web-inf下的资源文件在classloader范围以外，通过classloader显然是无法获取到的。

