---
layout: post
title: jconsole远程监控JVM
date: 2019-08-27 13:32:20 +0800
description: jconsole远程监控JVM）
tags: [jconsole 监控 JVM]
img: jconsole.jpg
---


# 服务器添加配置文件

  * export JAVA_OPTS='-Djava.rmi.server.hostname=你的服务器地址（公网ip） -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8888 -Dcom.sun.management.jmxremote.rmi.port=8888 -Dcom.sun.management.jmxremote.authenticate=true -Dcom.sun.management.jmxremote.ssl=false'

# 参数分析

* 配置远程调用主机地址,即jar包运行所在系统的IP地址,不配置则默认使用hosts文件中的值：-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8888
* 指定JMX的运行端口,jconsole需要对接的既是该端口：-Dcom.sun.management.jmxremote.rmi.port=8888
* 是否需要进行身份验证：-Dcom.sun.management.jmxremote.authenticate=true
* 是否允许使用ssl方式接入：-Dcom.sun.management.jmxremote.ssl=false

# 配置文件启用

* 导入环境变量后，用命令重新加载配置文件：source /etc/profile

# 添加认证文件

* cd /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/management/；
* 如果有jmxremote.password文件就进行编辑，如果没有就用jmxremote.password.template文件拷贝生成一个这样文件；
* 添加用户名与密码，这里默认的用户有monitorRole和controlRole，对应的密码分别是QED和R&D。

# 启用工程

* java $JAVA_OPTS -jar xxx.jar &

# 开启监控

* 客户端打开jconsole，输入对应的信息
* 最后成功的监控
![jconsole]({{site.baseurl}}/assets/img/jconsole.jpg)
