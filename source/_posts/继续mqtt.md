title: "继续mqtt"
date: 2015-05-15 10:31:13
categories:
 - work
tags:
 - mqtt
 - mosquitto
---
&emsp;手头上正在做有关智能家居的项目。项目基于[mqtt](http://mqtt.org/)协议开发，使用开源项目[mosquitto](http://mosquitto.org/)为主体搭建。
&emsp;过程中有很多心得以及走过的坑，都记录在为知笔记上了。想一下子转到这边来，还有点麻烦，就算了。以后的填坑心得都记录于此吧。毕竟准备以此为工作技术blog的。
&emsp;昨天其它的同事提的需求是：本地mqtt server与云端mqtt server桥接，使app端与家庭设备端能在lan和wan范围内都能正常通信。这就涉及到mosquitto中的bridge部分了。这部分早先简单看了一下，但是不是太明白，现在已基本忘了，得重新捡拾起来review并实现需求。
&emsp;go go go...
