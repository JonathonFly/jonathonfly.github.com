---
layout: post
category : Java
tagline: 
tags : []
---
{% include JB/setup %}

由于Myeclipse 10启动的速度实在是无法忍受，我毅然决定放弃Myeclipse，转而使用eclipse。(eclipse 4.5 Juno对JDK要求在1.7及以上，为了跟现有的项目使用的JDK配套，于是下了个eclipse 4.3 Kepler)

在配置好eclipse tomcat后，启动时报如下错误信息：
    
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/1.png" style="max-width:100%;">

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/2.png" style="max-width:100%;">


Problems的错误如下：

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/3.png" style="max-width:100%;">

打开Properties的Project Facts，看到Dynamic Web Module的Version为3.0，而项目中的web.xml中，webapp的version为2.5，此处版本不一致，导致了Problem：Can't change version of project fact Dynamic Web Module to 2.5。

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/4.png" style="max-width:100%;">

将web.xml中webapp版本改为3.0，重新启动tomcat。

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/5.png" style="max-width:100%;">

tomcat启动成功，但是又出现了PermGen space OutOfMemoryError.

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/6.png" style="max-width:100%;">

双击打开tomcat server，然后open launch configuration，点击Arguments标签，在VM arguments的末尾添加一些JVM参数，如：-Xms128m -Xmx1024m -XX:MaxPermSize=512m。

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/7.png" style="max-width:100%;">

再次启动tomcat，成功！

<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-9/8.png" style="max-width:100%;">