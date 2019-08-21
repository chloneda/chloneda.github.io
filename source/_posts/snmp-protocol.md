---
title: 聊聊SNMP协议
tags: SNMP
categories: SNMP
keywords: SNMP
comments: false
---
**注**：本文出自博主：[chloneda](https://chloneda.github.io/)

# SNMP概述
SNMP(Simple Network Management Protocol):简单网络管理协议,是基于TCP/IP五层协议中的应用层协议。由于其简单可靠，提供了一种监控和管理网络设备的系统方法，因此受到了众多厂商的欢迎，成为了目前最为广泛的网管协议。

SNMP的基本思想：为不同种类、厂家、型号的设备，定义一个统一的接口和协议，使得管理员可以是使用统一的外观面对这些需要管理的网络设备。通过网络，管理员可以管理位于不同物理空间的设备，从而大大提高网络管理的效率，简化网络管理员的工作。

# Snmp版本
SNMP目前共有v1，v2，v3这三个版本，三个版本的联系与区别。

- SNMP v1：是SNMP协议的最初版本，存在较多安全缺陷，现在这个版本是使用的比较少了。 
- SNMP v2：也采用团体名认证,在兼容SNMPv1的同时又扩充了SNMPv1的功能，具体是扩展了数据类型、支持分布式网络管理、可以实现大量数据的传输，提高了效率和性能、丰富了故障处理能力及增加了集合处理功能。
- SNMP v3：是最新版本的SNMP。它相对于V2版本，在安全性上得到了重要提升，增加了对认证和密文传输的支持。

# SNMP核心概念

## SNMP管理架构

SNMP管理架构应包含四个部分进行网络管理：**SNMP管理站**、**SNMP代理**、**MIB(管理信息库)**和**SNMP管理协议**。

**SNMP管理站**（management station）：通常被称作为网络管理工作站（NMS），负责收集维护各个SNMP元素的信息，通过UDP协议向SNMP代理发送各种命令，当SNMP代理收到命令后，对收集的信息进行处理，并返回SNMP管理站需要的参数。而此时被管理对象中一定要有代理进程，这样才能响应管理站发来的请求。

**SNMP代理**(Agent)：运行在各个被管理的网络节点之上，负责统计该节点的各项信息，并且负责与SNMP管理站交互，接收并执行管理站的命令，上传各种本地的网络信息。

![snmp_structure](/uploads/snmp_Structure.png)

**注**：协议栈中带有阴影的部分是原主机或路由器所具有的，而没有阴影的部分是为实现网络管理而增加的。
　　
**MIB(管理信息库)**：是对象的集合，它代表网络中可以管理的资源和设备。每个对象基本上是一个数据变量，它代表被管理的对象的一方面的信息。

**SNMP管理协议**：用于管理站与SNMP代理之间的通信规则，其SNMP报文格式请查看底下章节。

管理站和代理之间利用 SNMP 报文进行通信，而 SNMP 报文又使用 UDP协议来传送，由于采用UDP协议，不需要在代理和管理站之间保持连接。
　
## SNMP报文格式

管理站（NMS）和代理（Agent）之间交换的管理信息构成了SNMP报文，报文的基本格式如下图：

![snmp_MessageFormat.png](/uploads/snmp_MessageFormat.png)

**下面将对该SNMP报文格式逐个进行说明！**

SNMP消息报文包含两个部分：SNMP报头和PDU(协议数据单元)。

**SNMP报头**
- 版本：版本号字段，规则为版本号减 1，如SNMP V1则应写入 0。如果SNMP代理使用不相同的协议，会直接抛弃与自己协议版本不同的数据报。
- 共同体(community)：即团体名，作为管理进程和代理进程之间的明文口令，相当于密码，默认为public，该字段可读可写，用来限制NMS对Agent的访问。如果团体名没有得到NMS/Agent的认可，该报文将被丢弃。
- PDU 类型：填入 0～4 中的一个数字，其对应关系如图。

![snmp_PDUType.png](/uploads/snmp_PDUType.png)

PDU类型请参考**PDU(协议数据单元)**章节！

**Get/Set 首部**
- 请求标识符(request ID)，这是由管理进程设置的一个整数值。代理进程在发送 get-response 报文时也要返回此请求标识符。管理进程可同时向许多代理发出 get 报文，这些报文都使用 UDP 传送，先发送的有可能后到达。设置了请求标识符可使管理进程能够识别返回的响应报文对于哪一个请求。
- 差错状态（error status）：由代理进程回答时填入 0～5 中的一个数字，见表 3 的描述。

![snmp_ErrorStatus.png](/uploads/snmp_ErrorStatus.png)

- 差错索引(error index)：当出现 noSuchName、badValue 或 readOnly 的差错时，由代理进程在回答时设置的一个整数，它指明有差错的变量在变量列表中的偏移。

**Trap 首部**
- 企业（enterprise）：填入 trap 报文的网络设备的对象标识符。此对象标识符肯定是在图 3 的对象命名树上的 enterprise 结点{1.3.6.1.4.1}下面的一棵子树上。
- Trap 类型：此字段正式的名称是 generic-trap，共分为表 4 中的 7 种。

![snmp_TrapType.png](/uploads/snmp_TrapType.png) 

当使用上述类型 2、3、5 时，在报文后面变量部分的第一个变量应标识响应的接口。
- 特定代码(specific-code)：指明代理自定义的时间（若 trap 类型为 6），否则为 0。
- 时间戳(timestamp)：指明自代理进程初始化到 trap 报告的事件发生所经历的时间，单位为 10ms。例如时间戳为 1908 表明在代理初始化后 1908ms 发生了该时间。
- 变量绑定(variable-bindings)：指明一个或多个变量的名和对应的值。在 get 或 get-next 报文中，变量的值应忽略。

## PDU(协议数据单元)

SNMP v1 版本规定了 5 种核心 PDU(协议数据单元)，用来在管理进程和代理之间信息的交换。
- get-request 操作：从代理进程处提取一个OID值。
- get-next-request 操作：从代理进程(MIB中)处提取紧跟当前参数值的下一个OID值，常被用于检索表数据，也被用于不能指定名称的变量,可以浏览MIB树。
- set-request 操作：设置代理进程的一个或多个参数值。
- get-response 操作：返回的一个或多个参数值。这个操作是由代理进程发出的，它是前面三种操作的响应操作。
- trap 操作：代理进程主动发出的报文，通知管理进程有某些事情发生。

前面的 3 种操作是由管理进程向代理进程发出的，后面的 2 个操作是代理进程发给管理进程的，其中代理进程端是用 161 端口接收 get 或 set 报文，而在管理进程端是用 162 端口来接收 trap 报文。

![snmp_fivePDU.png](/uploads/snmp_fivePDU.png) 

另外，在SNMP v2版本又增加了 3 种PDU(协议数据单元)，它们分别是：
- inform-request 操作：允许路由器向SNMP管理器发送通知请求。
- getBulk-request 操作：从代理进程处提取多个参数值,该操作会根据最大重试值执行一连串的GetNext操作，减少管理站与代理之间的交互,提高效率。
- report 操作：

### SNMP的操作类型
其实上述 8 种PDU(协议数据单元)按照功能不同，可以归结为三类操作。

- 查询、设置SNMP变量,如get-request、get-next-request、set-request、getBulk-request、inform-request。
- 应答请求,如 get-response。
- 事件报告，如trap。

实际上，在SNMP中，SNMP管理站对被管理设备的SNMP变量的操作只能有 **读** 与 **写** 两种基本动作。

## ASN.1(抽象语法标记)
ASN.1：高级的数据描述语言。描述数据的类型、结构、组织、及编码方法。包括符号和语法两部分。SNMP使用ASN.1描述PDU和MIB(管理信息库)。

关于ASN.1详细信息请查看这篇博文：[SNMP从入门到开发：进阶篇](http://blog.chinaunix.net/uid-23069658-id-3251526.html)

## BER(基本编码规则)
**BER(Basic Encoding Rule)**，中文名称：基本编码规则。描述具体的ASN.1对象如何编码为比特流在网络上传输。SNMP使用BER(Basic Encoding Rule)作为编码方案，数据首先先经过BER编码，再经由传输层协议(一边是UDP)发往接收方。接收方在SNMP端口上收到PDU后，经过BER解码后，得到具体的SNMP操作数据。

BER的数据都由三个域构成:标识域(tag)+长度域(length)+值域(value)。

## SMI(管理信息结构)
**SMI(Structure of Managerment Intormation)**，中文名称：管理信息结构，是SNMP的描述方法。规定了使用ASN.1子类型、符号。ASN.1功能强大，但SNMP只用到了其中很小一部分，对于这一部分内容的描述，限定了范围，即为SMI。SMI规定了使用到的ASN.1类型、宏、符号等。SMI是ASN.1的一个子集和超集。

## MIB(管理信息库)

**MIB(Management Information Base)**，中文名称：管理信息库，由网络管理协议访问的管理对象数据库。MIB是对象的集合，它代表网络中可以管理的资源和设备。每个对象基本上是一个数据变量，它代表被管理的对象的一方面的信息。

管理信息库 MIB 指明了网络元素所维持的变量（即能够被管理进程查询和设置的信息）。MIB 给出了一个网络中所有可能的被管理对象的集合的数据结构。SNMP 的管理信息库采用和域名系统 DNS 相似的树型结构，它的根在最上面，根没有名字。底下的图 画的是管理信息库的一部分，它又称为对象命名（object naming tree）。

MIB采用分层树形结构，对象命名树的顶级对象有三个，即 ISO、ITU-T 和这两个组织的联合体。在 ISO 的下面有 4 个结点，其中的饿一个（标号 3）是被标识的组织。在其下面有一个美国国防部（Department of Defense）的子树（标号是 6），再下面就是 Internet（标号是 1）。在只讨论 Internet 中的对象时，可只画出 Internet 以下的子树（图中带阴影的虚线方框），并在 Internet 结点旁边标注上{1.3.6.1}即可。

在 Internet 结点下面的第二个结点是 mgmt（管理），标号是 2。再下面是管理信息库，原先的结点名是 mib。1991 年定义了新的版本 MIB-II，故结点名现改为 mib-2，其标识为{1.3.6.1.2.1}，或{Internet(1) .2.1}。这种标识为对象标识符。

![snmp_Mib.png](/uploads/snmp_Mib.png)

最初的结点 mib 将其所管理的信息分为 8 个类别，如图，现在的 mib-2 所包含的信息类别已超过 40 个。

![snmp_MibInfoType.png](/uploads/snmp_MibInfoType.png)

应当指出，MIB 的定义与具体的网络管理协议无关，这对于厂商和用户都有利。厂商可以在产品（如路由器）中包含 SNMP 代理软件，并保证在定义新的 MIB 项目后该软件仍遵守标准。用户可以使用同一网络管理客户软件来管理具有不同版本的 MIB 的多个路由器。当然，一个没有新的 MIB 项目的路由器不能提供这些项目的信息。这里要提一下MIB中的对象{1.3.6.1.4.1}，即enterprises（企业），其所属结点数已超过 3000。例如IBM为{1.3.6.1.4.1.2}，Cisco为{1.3.6.1.4.1.9}，Novell为{1.3.6.1.4.1.23}等。世界上任何一个公司、学校只要用电子邮件发往iana-mib@isi.edu进行申请即可获得一个结点名。这样各厂家就可以定义自己的产品的被管理对象名，使它能用SNMP进行管理。

## ASN.1、BER、SMI、MIB、PDU的关系
关于ASN.1、BER、SMI、MIB、PDU的关系如下图所示。
![snmp_SMI_MIB.png](/uploads/snmp_SMI_MIB.png)

## OID(对象标识符)
**OID(Object Identifier)**，中文名称：对象标识符，被管理设备的每个管理资源和对象都有自己的OID(Object Identifier)，管理对象通过树状结构进行组织，OID由树上的一系列整数组成，整数之间用点( . )分隔开，树的叶子节点才是真正能够被管理的对象。

## 常用OID
这里总结了一些常用的OID，当需要时可以及时查询。
[SNMP监控常用OID查询](http://oid-info.com/cgi-bin/display?tree=1.3.6)
[SNMP监控一些常用OID的总结](https://blog.csdn.net/a9254778/article/details/51200502)
[下载不同厂商的MIB包](http://www.oidview.com/mibs/2021/UCD-SNMP-MIB.html)
[查看不同厂商的OID代号](https://www.alvestrand.no/objectid/1.3.6.1.4.1.html)

# SNMP协议实现
SNMP协议的Java实现是SNMP4J，其jar包可以在[SNMP官方网站](http://www.snmp4j.org/)上下载。开发前请简单了解一下SNMP4J，具体细节请看这篇博文：[SNMP4J介绍](https://www.cnblogs.com/xdp-gacl/p/4187089.html)；更多SNMP4J示例请参考[Github](https://github.com/chloneda/snmp/)。


# 本文小结
SNMP支持Window、Linux、MacOS系统的安装，关于SNMP的安装步骤请自行查询。


---