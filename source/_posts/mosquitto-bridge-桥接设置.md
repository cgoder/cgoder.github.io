title: "mosquitto bridge 桥接设置"
date: 2015-05-18 11:18:41
tags: [mqtt mosquitto smarthome] 
categories: work
---

下面是mosquitto桥接模式的一些参数简单设置。
上次提到其它同事项目中有这个需求，实现的同时顺便记录一下，以备查。
<!--more-->

----------

bridge桥接配置文件mosquitto.conf内容如下：

     user root
 	pid_file /var/run/mosquitto.pid
 	port 61883
 	persistence true
 	persistence_location /skydir/sshome/
 	connection LOCAL-SERVER
 	remote_clientid MSTARTV_REMOTE
 	local_clientid MSTARTV_LOCAL
 	address 4.12.12.6:61883
 	topic # both 0 sshome/ sshome/
 	bridge_protocol_version mqttv311

##1.connection
bridge时用于表示此桥接的别名。
格式为:

 	connection LOCAL-SERVER

log如:

 	1431916959: Opening ipv4 listen socket on port 61883.
 	1431916959: Connecting bridge LOCAL-SERVER (4.12.12.6:61883)

##2.remote_clientid
用于bridge时local_mqtt_server在remote_mqtt_server上的别名显示。
格式为：

    remote_clientid BRIDGE_REMOTE

注意：如果不设置。系统为默认分配一个形如localhost.***connection***的别名。
log如：

    May 18 10:42  mosquitto[23367]: New connection from 18.10.13.7 on port 61883.
    May 18 10:42 mosquitto[23367]: New client connected from 18.10.13.7 as **<font color=red>localhost.LOCAL-SERVER</font>** (c0, k60).
    May 18 10:42 mosquitto[23367]: localhost.LOCAL-SERVER 0 sshome/#

##3.local_clientid
同remote_clientid。用于bridge时local_mqtt_server在本地mqtt client的标识。此字符串为local_mqtt_server在本地数据库的别名。但只存在于本地。
格式为：

    local_clientid BRIDGE_LOCAL

##4.address
bridge的桥接地址和端口。标示远端服务器的IP地址和端口。
格式为：

    address 8.8.8.8:61883。

##5.topic
bridge时用于桥接local_mqtt_server与remote_mqtt_server时，信息转发用的topic。
例如:

    topic # both 0 sshome/ sshome/

##6.案例.实践
PC上开一个sub客户端。连接remote_mqtt_server，并订阅某个topic。

 	C:\Program Files (x86)\mosquitto>mosquitto_sub.exe -h 4.12.12.6 -p 61883 -i TzWinTester -u user -P user -t sshome
 	C:\Program Files (x86)\mosquitto>mosquitto_sub.exe -h 4.12.12.6 -p 61883 -i TzWinTester -u user -P user -t sshome/sshome
 	
 	test msg. pc-mstartv-server-pc.
 	hello,server! what can i do for you?
 	hello,server! what can i do for you?
 	hey,are you ok、

PC上另开一个pub客户端。连接local_mqtt_server,以向某个topic发送信息。

 	C:\Program Files (x86)\mosquitto>mosquitto_pub.exe -h 192.168.1.114 -p 61883 -i PCTESTER -t sshome -m "hello,server! what can i do for you?"
 	C:\Program Files (x86)\mosquitto>mosquitto_pub.exe -h 192.168.1.114 -p 61883 -i PCTESTER -t sshome -m "hello,server! what can i do for you?"
 	C:\Program Files (x86)\mosquitto>mosquitto_pub.exe -h 192.168.1.114 -p 61883 -i PCTESTER -t sshome -m "hey,are you ok、"

本地local_mqtt_server运行。

 	mosquitto version 1.4 (build date 2015-05-06 14:33:09+0800) starting
 	1431916959: Config loaded from sshome.conf.
 	1431916959: Opening ipv6 listen socket on port 61883.
 	1431916959: Opening ipv4 listen socket on port 61883.
 	1431916959: Connecting bridge LOCAL-SERVER (42.121.120.62:61883)

云端remote_mqtt_server运行。

 	May 18 12:00:58 mosquitto[23367]: localhost.LOCAL-SERVER 0 **sshome/#**
 	May 18 12:00:58 mosquitto[23367]: New client connected from 180.109.130.79 as localhost.LOCAL-SERVER (c0, k60).



运行过程:
>1.启动local_mqtt_server。
2.启动remote_mqtt_server。
3.运行PC上的sub客户端，根据local_mqtt_server运行时载入的配置文件mosquitto.conf中的bridge参数设定，指定云端remote_mqtt_server IP/port，并订阅相应的topic，如**"sshome/sshome"**。模拟设备连接云端mqtt服务器。
4.运行PC上的pub客户端，将信息"hey,are you ok、"发布到loca_mqtt_server上的topic。此topic为bridge配置里设定的topic。如**"sshome"**。模拟设备连接本地mqtt服务器。
5.此时，信息"hey,are you ok、"即通过local_mqtt_server与remote_mqtt_server的桥接，从本地设备转发到云端设备上。
6.服务器转发的内容如下:

>  	id	hdr	mid	topic	payload	ctime
 	124982 (d0, q0, r0) 0 **sshome/sshome** "hey,are you ok" 2015-05-18 12:05:51



注意：sub客户端订阅的topic为**"sshome/sshome"**,而不是mosquitto.conf中的**"sshome/"**。
详细说明见[mosquitto的官方手册](http://mosquitto.org/man/mosquitto-conf-5.html)。

    topic pattern [[[ out | in | both ] qos-level] local-prefix remote-prefix]
> Define a topic pattern to be shared between the two brokers. Any topics matching the pattern (which may include wildcards) are shared. The second parameter defines the direction that the messages will be shared in, so it is possible to import messages from a remote broker using in, export messages to a remote broker using out or share messages in both directions. If this parameter is not defined, the default of out is used. The QoS level defines the publish/subscribe QoS level used for this topic and defaults to 0.

> The local-prefix and remote-prefix options allow topics to be remapped when publishing to and receiving from remote brokers. This allows a topic tree from the local broker to be inserted into the topic tree of the remote broker at an appropriate place.

> For incoming topics, the bridge will prepend the pattern with the remote prefix and subscribe to the resulting topic on the remote broker. When a matching incoming message is received, the remote prefix will be removed from the topic and then the local prefix added.

> For outgoing topics, the bridge will prepend the pattern with the local prefix and subscribe to the resulting topic on the local broker. When an outgoing message is processed, the local prefix will be removed from the topic then the remote prefix added.

> When using topic mapping, an empty prefix can be defined using the place marker "". Using the empty marker for the topic itself is also valid. The table below defines what combination of empty or value is valid.

> 	 	 Topic	Local_Prefix	Remote_Prefix	Validity
 	1	value	value	value	valid
 	2	value	value	""	valid
 	3	value	""	value	valid
 	4	value	""	""	valid (no remapping)
 	5	""	value	value	valid (remap single local topic to remote)
 	6	""	value	""	invalid
 	7	""	""	value	invalid
 	8	""	""	""	invalid

> To remap an entire topic tree, use e.g.:
topic # both 2 local/topic/ remote/topic/
This option can be specified multiple times per bridge.

> Care must be taken to ensure that loops are not created with this option. If you are experiencing high CPU load from a broker, it is possible that you have a loop where each broker is forever forwarding each other the same messages.

> See also the cleansession option if you have messages arriving on unexpected topics when using incoming topics.

> Example Bridge Topic Remapping. 
The configuration below connects a bridge to the broker at test.mosquitto.org. It subscribes to the remote topic $SYS/broker/clients/total and republishes the messages received to the local topic test/mosquitto/org/clients/total

> connection test-mosquitto-org
address test.mosquitto.org
cleansession true
topic clients/total in 0 test/mosquitto/org $SYS/broker/