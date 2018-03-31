---
layout: "post"
title: "Network Memo"
date: "2017-08-26 18:32"
---

# 1 Internet

The Internet is a global system of interconnected computer networks that use the Internet protocol suite (TCP/IP) to link several billion devices worldwide.

参考：[https://en.wikipedia.org/wiki/History_of_the_Internet](https://en.wikipedia.org/wiki/History_of_the_Internet)

## 1.1 IETF and RFC

Internet Engineering Task Force (IETF) 负责管理因特网有关标准的开发。
因特网标准以Request For Comments (RFC) 文档形式在因特网上发表。

# 2 TCP/IP简介

The Internet protocol suite is the computer networking model and set of communications protocols used on the Internet and similar computer networks. It is commonly known as TCP/IP.

参考：
TCP/IP Illustrated, Volume 1 The Protocols:
http://www.cs.newpaltz.edu/~pletcha/NET_PY/the-protocols-tcp-ip-illustrated-volume-1.9780201633467.24290.pdf

TCP/IP Illustrated, Volume 1 The Protocols, 2nd Edition:
http://www.jtbookyard.com/uploads/6/2/9/3/6293106/tcp_ip_illustrated_volume_1_the_protocols_2nd_edit.pdf

计算机网络（第5版），谢希仁著

## 2.1 TCP/IP Four-layer Model

TCP/IP协议族分为四层：链路层、网络层、运输层和应用层，每一层各有不同的责任。

![](/images/network_IP_stack_connections.svg)

RFC 1122 covers the communications protocol layers: link layer, IP layer, and transport layer; its companion RFC 1123 covers the application and support protocols.
参考：https://en.wikipedia.org/wiki/Internet_protocol_suite#Abstraction_layers

## 2.2 TCP/IP vs OSI Model

![](/images/osi_model.png)

参考：[https://supportforums.cisco.com/discussion/11795526/osi-and-tcpip-model](https://supportforums.cisco.com/discussion/11795526/osi-and-tcpip-model)

# 3 链路层

链路层主要有三个目的：(1)为IP模块发送和接收IP数据报；(2)为ARP模块发送ARP请求和接收ARP应答；(3)为RARP发送请求和接收RARP应答。

## 3.1 PPP协议

Point-to-Point Protocol (PPP) is used over many types of physical networks including serial cable, phone line, trunk line, cellular telephone, specialized radio links, and fiber optic links such as SONET.
PPP is also used over Internet access connections, Internet service providers (ISPs) have used PPP for customer dial-up access to the Internet.

## 3.2 Ethernet

以太网有两种帧格式：IEEE 802.2/802.3 (RFC 1042) 和 DIX Ethernet (RFC 894)。
其中DIX Ethernet格式的使用更加广泛（DIX代表DEC，Intel和Xerox）。

![IEEE_VS_DIX](/images/iEEE_VS_DIX.png)

参考：[https://en.wikipedia.org/wiki/Ethernet_frame](https://en.wikipedia.org/wiki/Ethernet_frame)

## 3.3 最大传输单元MTU

DIX Ethernet和IEEE 802.3对数据帧和长度都有一个限制，其最大值分别为1500和1492字节。 链路层的这个特性称作 Maximum Transmission Unit (MTU)。
如果IP层有一个数据报要传，其数据的长度比链路层的MTU要大，那么IP层就需要进行分片（fragmentation），把数据报分成若干片，这样每一片都小于MTU。

Table 1: Typical MTUs

|               Network               | MTU (bytes) |
| ----------------------------------- | ----------- |
| Hyperchannel                        | 65535       |
| 16 Mbits/sec token ring (IBM)       | 17914       |
| 4 Mbits/sec token ring (IEEE 802.5) | 4464        |
| FDDI                                | 4352        |
| DIX Ethernet                        | 1500        |
| IEEE 802.3/802.2                    | 1492        |
| X.25                                | 576         |
| Point-to-Point (low delay)          | 296         |

参考：TCP/IP Illustrated Volume 1, 2.8 MTU

## 3.3.1 路径MTU

如果两台主机通信要通过多个网络，那么每个网络可能有不同的MTU。两台通信主机路径中的最小MTU，被称为路径MTU。
两台主机之间的路径MTU不一定是个常数。它取决于当时所选择的路由。而选路不一定是对称的（从A到B的路由可能与从B到A的路由不同）， 因此路径MTU在两个方向不一定是一致的。
参考：TCP/IP Illustrated Volume 1, 2.9 Path MTU

# 4 网络层

IP协议是网络层最重要的协议。 IP协议提供不可靠、无连续的数据报传送服务。
不可靠（unreliable）的意思是它不保证IP数据报能成功地到达目的地。如果发生某种错误，如路由器暂时用完了缓冲区，IP有一个简单的错误处理算法： 丢弃该数据报，然后发送ICMP消息报给信源端。
无连接（connectionless）的意思是IP并不维护任何关于后续数据报的状态信息，每个数据报的处理是相互独立的。这也说明，IP数据报可以不按发送顺序接收。

## 4.1 IP协议

[RFC 791](https://tools.ietf.org/html/rfc791) 对Internet Protocol (IP)协议进行了描述。
普通的IP数据头（不包含选项字段）为20个字节。

IP数据头的格式如下

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

参考：[https://tools.ietf.org/html/rfc791#page-11](https://tools.ietf.org/html/rfc791#page-11)

### 4.1.1 Protocol字段


Table 2: IP数据报中常用的协议字段值

| Protocol Number |                Protocol Name                |
| --------------- | ------------------------------------------- |
| 1               | ICMP (Internet Control Message Protocol)    |
| 2               | IGMP (Internet Group Management Protocol)   |
| 6               | TCP (Transmission Control Protocol)         |
| 17              | UDP (User Datagram Protocol)                |
| 41              | ENCAP (IPv6 encapsulation)                  |
| 89              | OSPF (Open Shortest Path First)             |
| 132             | SCTP (Stream Control Transmission Protocol) |

参考：IP数据报中协议字段值：[https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)

# 5 运输层UDP协议

TCP协议和UDP协议是运输层中重要的两个协议。
User Datagram Protocol ([UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol)) 是一个简单的面向数据报的运输层协议。UDP不提供可靠性，它把应用程序传给IP层的数据发送出去，但是并不保证它们能到达目的地。
[RFC 768](https://tools.ietf.org/html/rfc768) 是UDP的正式规范。

## 5.1 UDP首部格式

UDP的首部为固定的8字节。

```
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|     Source      |   Destination   |
|      Port       |      Port       |
+--------+--------+--------+--------+
|                 |                 |
|     Length      |    Checksum     |
+--------+--------+--------+--------+
|
|          data octets ...
+---------------- ...
```
由于IP层已经把IP数据报分配给TCP或UDP（根据IP首部协议字段值），因此TCP端口号由TCP来查看，而UDP端口号由UDP来查看。 TCP端口号和UDP端口号是相互独立的。


# 6 运输层TCP协议

[TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) 协议是运输层最重要的协议。[RFC 793](https://tools.ietf.org/html/rfc793) 对Transmission Control Protocol (TCP)协议进行了描述。

**TCP提供一种面向连接的、可靠的字节流服务**

面向连接意味着两个使用TCP的应用在彼此交换数据之前必须先建立一个TCP连接。
TCP有很多手段来提供可靠性：如检验和确保数据传输过程中无变化，选择确认机制，超时重传等等。
TCP的字节流服务是指：两个应用程序通过TCP连接交换8 bit字节构成的字节流（而不是比特流）。TCP不在字节流中插入记录标识符，如果一方的应用程序先传10字节，又传20字节，再传50字节，连接的另一方将无法了解发送方每次发送了多少字节。收方可能每次接收20字节，分4次接收这80个字节。此外，TCP对字节流的内容不作任何解释，TCP不知道传送的字节流是ASCII字符、EBCDIC字符还是二进制数据。

## 6.1 TCP Header Format

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Table 3: TCP头中标志位的说明

| TCP头中标志位 |                  说明                  |
| ------------- | -------------------------------------- |
| URG           | 紧急指针（ urgent pointer）有效。      |
| ACK           | 确认序号有效。                         |
| PSH           | 接收方应该尽快将这个报文段交给应用层。 |
| RST           | 重建连接。                             |
| SYN           | 同步序号用来发起一个连接。             |
| FIN           | 发端完成发送任务。                     |

参考：[https://tools.ietf.org/html/rfc793#page-15](https://tools.ietf.org/html/rfc793#page-15)

## 6.2 TCP连接的建立 (Three-way handshake)

TCP用如下方式建立一个连接：

1.请求端（通常称为客户）发送一个SYN报文段指明打算连接的服务器的端口，以及初始序号（ISN）。这是报文段1。
2.服务器发回包含服务器的初始序号的SYN报文段（报文段2）作为应答。同时，将确认序号设置为客户的ISN加1以对客户的SYN报文段进行确认。一个SYN将消耗一个序号。
3.客户必然将确认序号设置为服务器的ISN加1以对服务器的SYN报文段进行确认（报文段3）

**上面过程称为三次握手。**

![](/images/tcp-handshake.png)

### 6.2.1 为什么要三次握手（两次握手不行？）

为什么要三次握手？两次握手不行？
这主要是为了防止已失效的连接请求报文段突然又传送到了B，因而产生错误。
所谓的“已失效的连接请求报文段”是这样产生的。考虑一种正常情况：A发出连接请求，但因为连接请求报文丢失而未收到确认。于是A再重新发送一次连接请求。后来收到了确认，建立了连接。数据传输完毕后，就释放了连接。A一共发送了两个连接请求报文段，其中第一个丢失，第二个到达了B。没有“已失效的连接请求报文段”。
现假定出现一种异常情况，即A发出的第一个连接请求报文段并没丢失，而是在某个网络结点长时间滞留了，以致延误到连接释放以后的某个时间才达到B。本来这是一个早已失效的报文段，但B收到此失效报文段后，就误认为A又发出一次新的连接请求。于是就向A发出确认报文段，同意建立连接。假定不采用三次握手，那么只要B发出确认，B就进入连接建立状态。由于A并没有发出连接请求，因此，会忽略B的连接确认报文段，也不会向B发送数据，但是B却一直处于连接建立状态，一直在等待A发送数据，这样导致B的资源被白白浪费。
**采用三次握手可以防止上述现象的发生，例如在上述情况下，由于A并没有发出连接请求，因此，并不会发出确认信号，而B没有收到确认报文段，也就不会进入连接建立状态，这样就能避免A没有进入连接建立状态，而B处于连接建立状态的情况。**

### 6.2.2 同时打开 (Simultaneous Open)

TCP支持同时打开 (Simultaneous Open)，对于同时打开它仅建立一条连接而不是两条连接。

![simultaneous-open](/images/simultaneous-open.png)
Figure 5: TCP同时打开建立一条连接

### 6.3 TCP连接的释放（两个二次握手）

TCP的连接释放过程比较复杂。

![tcp-release](/images/tcp-release.png)
Figure 6: TCP的连接释放过程


A把连接释放报文段首部的FIN置1，其序号seq=u，它等于前面已传送过的数据的最后一个字节的序号加1。这时A进入 FIN-WAIT-1 状态，等待B的确认。B收到连接释放报文段后即发出确认，确认号是ack=u+1，而这个报文段自己的序号为v，等于B前面已传送过的数据的最后一个字节的序号加1。然后B就进入 CLOSE-WAIT 状态。这时从A到B这个方向的连接就释放了，TCP连接处于半关闭(half-close)状态。从B到A这个方向的连接并未关闭，如果B还有数据要发送，A仍需要接收。
A收到来自B的确认后，就进入 FIN-WAIT-2 状态，等待B发出的连接释放报文段。
如果B已经没有要向A发送的数据，其应用进程通知TCP释放连接。这时B发出的连接释放报文段必须使FIN=1，现假定B的序号为w （在半关闭状态B可能又发送了一些数据）。B还必须重复上次已发送过的确认号ack=u+1，这时B就进入 LAST-ACK 状态，等待A的确认。
A在收到B的连接释放报文段后，必须对些发出确认。在确认报文段中所ACK置1，确认号ack=w+1，而自己的序号是seq=u+1（前面发送过的FIN报文段也要消耗一个序号）。然后进行到 TIME-WAIT 状态。A再经过 2MSL (Maximum Segment Lifetime, RFC 793建议MSL设为2分钟)后才进入到 CLOSED 状态。
B只要收到A发出的确认，就进入到 CLOSED 状态。

参考：计算机网络（第5版），谢希仁著，5.9.2 TCP的连接释放

### 6.3.1 为什么要等待2MSL（或为什么需要TIME_WAIT状态）

为什么A在TIME-WAIT状态必须等待2MSL的时间呢？有两个理由。
第一，为了保证A发送的最后一个ACK报文段能够到达B。这个ACK报文段有可能丢失，因而使处于LAST-ACK状态的B收不到对已发送的FIN+ACK报文段的确认。B会超时重传这个FIN+ACK报文段，而A就能在2MSL时间内收到这个重传的FIN+ACK报文段。
第二，为了本连接持续的时间内所产生的所有报文段都从网络中消失，这样就可以使下一个新的连接中不会出现这种旧的连接请求报文段。

### 6.3.2 同时关闭 (Simultaneous Close)

TCP协议允许同时关闭 (Simultaneous Close)。

![tcp-close](/images/tcp-close.png)
Figure 7: TCP同时关闭连接

## 6.4 TCP的状态变迁图

![tcp-state ](/images/tcp-state.png)
Figure 8: TCP state transition diagram

参考：

UNIX网络编程第1卷：套接口API和XOpen传输接口API(第2版)，第2章
[http://www.tti.unipa.it/~gneglia/ip_networks06/slides/TCPIP_State_Transition_Diagram.pdf](http://www.tti.unipa.it/~gneglia/ip_networks06/slides/TCPIP_State_Transition_Diagram.pdf)

## 6.5 提供可靠性：“滑动窗口”、“确认机制”、“重传机制”

当底层通信系统仅仅提供不可靠的分组交付功能时，协议软件如何提供可靠的传输呢？答案十分复杂，但是大部分可靠协议使用一个名为“带重传的肯定确认”（positive acknowledgement with retransmission）的技术作为提供可靠性的基础。这种技术要求接收方收到数据之后向发送方回送确认（ACK）报文。发送方对发出的每个分组都保存一份记录，在发送下一个分组之前等待确认信息。发送方还在送出分组时启动一个定时器，并在定时器超时而确认信息还没有到的情况下重发刚才的分组。

![tcp-retransmission](/images/tcp-retransmission.png)
Figure 9: 简单“带重传的肯定确认”协议，发送方等待每个分组的确认信息。向下垂直方向表示时间增长

如果出现分组丢失时（超时还未收到确认），发送方会重新发送分组。

![tcp-mission-and-retransmission](/images/tcp-mission-and-retransmission.png)
Figure 10: 分组丢失引起了超时和重传

参考：《用TCP/IP进行网际互连（第一卷）——原理、协议与结构（第四版）》13.4 提供可靠性

### 6.5.1 滑动窗口

为实现可靠性，前面介绍了简单的“带重传的肯定确认”机制，在发送方送出一个分组后，等待相应的确认信息而不发送下一个分组。这样可行，但却没有充分利用网络带宽，在时延大的网络上，浪费带宽的问题更加突出。
TCP使用滑动窗口协议来更好地利用网络带宽，因为它允许发送方在等待确认之前可以发送多个分组。
下面是一个简单的滑动窗口协议示意图，假设窗口的大小为8个分组（TCP滑动窗口的窗口大小是可变的）：

![tcp-smooth-window](imagetcp-smooth-window.png)
Figure 11: (a) 窗口大小为8个分组的滑动窗口；(b) 收到对1号分组的确认后，窗口滑动，使得9号分组也能被发送。

TCP滑动窗口的窗口大小可以随时改变，这不仅提供可靠传输，而且还提供了流量控制。 **如果接收方的缓冲区快要满了，不能接收更多的分组，这时它就发出小的窗口通告值。在极端情况下，接收方使用0通告值来停止一般分组的传输。而在缓冲区空间又可用后，接收方通知一个非0的窗口值来再次触发数据流。**

### 6.5.2 累积式确认（Cummulative Acknowledgement）

**TCP的确认方法称为“累积式确认”。接收端给发送端的确认信息只会确认最后一个连续的包，而不能跳着确认。** 比如，发送端发了11,12,12,14,15共五个数据包，接收端收到了11,12，于是回ACK 13（表示序号13之前的所有数据都已经收到，期待收到序号为13的数据包），然后收到了14,15（此时数据包13没收到），这时ACK仍然为13。这就是累积式确认（cummulative acknowledgement）。

### 6.5.3 选择性确认（Selective Acknowledgment）

“累积式确认”有一个问题：假设发送方发送了100个包，而接收方仅第11个包这一个包没有收到，那么接收方也只能对前面10个包进行确认，这样发送方不知道后面的数据是否已经收到，如果要重传，就不知道重传哪些数据包才合适。

为了克服这个不足，TCP还有另外一种确认方式叫：选择性确认（Selective Acknowledgment, SACK）。 **SACK的基本思想是接收方汇报收到的所有“数据区间”（很可能不连续），这样发送方就知道哪些数据未收到，哪些已经收到，从而可以优化重传机制。**

SACK需要两端的协议都支持，在Linux系统下，通过tcp_sack参数可以打开“选择性确认”功能（Linux 2.2后默认打开）。

参考：
[man 7 tcp](http://man7.org/linux/man-pages/man7/tcp.7.html)
[RFC2883](https://tools.ietf.org/html/rfc2883)

### 6.5.4 超时重传

为了保证TCP的可靠性，必须有重传机制。从概念上讲，重传很简单，在规定时间内没有收到数据包的确认信息就重传数据包即可。但重传时间的选择却是TCP最复杂的问题之一。重传时间设置得过短，则可能使没有丢的数据包进行了不必要的重传，使网络负载不必要增大；重传时间设置得过长，则传输效率变低。
TCP采用了自适应的算法来确认重传时间（Retransmission TimeOut, RTO）。由于路由器和网络流量均会变化，因此我们认为合适的RTO可能经常会发生变化，TCP应该跟踪这些变化并相应地改变其RTO。
为了计算RTO，往往需要估计数据包往返时间（Round Trip Time, RTT），也就是一个数据包从发出去到收到该数据包确认所经过的时间。

()[https://tools.ietf.org/html/rfc793]中有一个利用RTT来计算RTO的例子：

    An Example Retransmission Timeout Procedure
    Measure the elapsed time between sending a data octet with a
    particular sequence number and receiving an acknowledgment that
    covers that sequence number (segments sent do not have to match
    segments received). This measured elapsed time is the Round Trip
    Time (RTT). Next compute a Smoothed Round Trip Time (SRTT) as:

    SRTT = ( ALPHA * SRTT ) + ((1-ALPHA) * RTT)

    and based on this, compute the retransmission timeout (RTO) as:

    RTO = min[UBOUND,max[LBOUND,(BETA*SRTT)]]

    where UBOUND is an upper bound on the timeout (e.g., 1 minute),
    LBOUND is a lower bound on the timeout (e.g., 1 second), ALPHA is
    a smoothing factor (e.g., .8 to .9), and BETA is a delay variance
    factor (e.g., 1.3 to 2.0).

摘自：[https://tools.ietf.org/html/rfc793](https://tools.ietf.org/html/rfc793)

### 6.5.5 快速重传

如果一连串收到3个或3个以上相同的ACK值，就非常可能是一个报文段丢失了。于是我们就重传丢失的数据报文段，而无需等待超时定时器溢出。这就是快速重传算法。

## 6.6 糊涂窗口综合症(Silly Windows Syndrome)

当发送端应用进程产生数据很慢、或接收端应用进程处理接收缓冲区数据很慢，或二者兼而有之；就会使应用进程间传送的报文段很小，特别是有效载荷很小。 **极端情况下，有效载荷可能只有1个字节，而传输开销有40字节（20字节的IP头+20字节的TCP头），类似这种现象就叫糊涂窗口综合症(Silly Windows Syndrome)。**
收发双方都有避免糊涂窗口综合症的责任。

参考：《用TCP/IP进行网际互连（第一卷）——原理、协议与结构（第四版）》13.32 避免糊涂窗口综合症

### 6.6.1 发送方避免糊涂窗口综合症——Nagle算法

有些应用产生的有效载荷可能很小，比如交互式程序Telnet和Rlogin等。在一个Rlogin连接上客户一般每次发送一个字节到服务器，这就产生了一些41字节长的分组（20字节IP首部、20字节TCP首部、1字节数据），这就是糊涂窗口综合症。在局域网上，这些小分组通常不会引起麻烦；但广域网上，这些小分组则会增加拥塞出现的可能。
避免TCP连接上出现很多小分组的一种解决办法是采用 **Nagle算法** 。Nagle算法的伪代码如下：

```
if there is new data to send
  if the window size >= MSS(maximum segment size) and available data is >= MSS
    send complete MSS segment now
  else
    if there is unconfirmed data still in the pipe
      enqueue data in the buffer until an acknowledge is received
    else
      send data immediately
    end if
  end if
end if
```

**Nagle算法要求一个TCP连接上最多只能有一个未被确认的未完成小分组，在该分组的确认到达之前不能发送其他的小分组。TCP收集这些少量的分组，并在ACK到来时以一个分组的方式发出去。
**

#### 6.6.1.1 禁止Nagle算法(TCP_NODELAY)

有时我们需要关闭Nagle算法，一个典型的例子是X窗口系统服务器：小消息（鼠标移动）必做无时延地发送，以便为进行某种操作的交互用户提供实时的反馈。
[RFC 1122 Requirements for Internet Hosts](https://tools.ietf.org/html/rfc1122) 明确指出TCP必须实现Nagle算法，且还应提供一种方法来关闭这个算法在某个连接上执行。

sock编程时，可以使用选项 TCP_NODELAY 来关闭Nagle算法：

```c
int one = 1;
setsockopt(descriptor, SOL_TCP, TCP_NODELAY, &one, sizeof(one));
```
### 6.6.2 接收方避免糊涂窗口综合症

接收端的TCP可能产生糊涂窗口综合症，如果它为消耗数据很慢的应用程序服务，例如，一次消耗一个字节。假定发送应用程序产生了1000字节的数据块，但接收应用程序每次只吸收1字节的数据。再假定接收端的TCP的输入缓存为4000字节。发送端先发送第一个4000字节的数据。接收端将它存储在其缓存中。现在缓存满了。它通知窗口大小为零，这表示发送端必须停止发送数据。接收应用程序从接收端的TCP的输入缓存中读取第一个字节的数据。在入缓存中现在有了1字节的空间。接收端的TCP宣布其窗口大小为1字节，这表示正渴望等待发送数据的发送端的TCP会把这个宣布当作一个好消息，并发送只包括一个字节数据的报文段。这样的过程一直继续下去。一个字节的数据被消耗掉，然后发送只包含一个字节数据的报文段。
对于这种糊涂窗口综合症，即应用程序消耗数据比到达的慢，有两种建议的解决方法。

(1) **Clark解决方法** ：Clark解决方法是只要有数据到达就发送确认，但宣布的窗口大小为零，直到或者缓存空间已能放入具有最大长度的报文段，或者缓存空间的一半已经空了。
(2) **延迟确认** ：即延迟一段时间后再发送确认，当一个报文段到达时并不立即发送确认。

参考：
[http://www.cnblogs.com/ggjucheng/archive/2012/02/03/2337046.html](http://www.cnblogs.com/ggjucheng/archive/2012/02/03/2337046.html)

## 6.7 拥塞控制（Congestion control）

TCP的滑动窗口模型使用大小可变的窗口，这能实现“端到端的流量控制”问题。实际上，存在两个独立的流量问题。第一，互联网协议需要源主机和最终目的主机之间“端到端的流量控制”。例如，当一台微机与一台大型机通信时，微机需要调用数据的流入速率，否则协议软件很快就超载了。第二，互联网协议需要一个流量控制机制得到中间系统（如路由器）能够控制一个源主机，以免它发送的通信量超过中间机器（如路由器）的承受能力。

**当中间机器（如路由器）超载时，这种状况称为拥塞（congestion），解决这个问题的机制称为拥塞控制机制。**

有很多拥塞控制算法，如慢启动、拥塞避免、拥塞发生时的快速重传等等。更多TCP拥塞控制算法可参考：[https://en.wikipedia.org/wiki/TCP_congestion_control](https://en.wikipedia.org/wiki/TCP_congestion_control)

## 6.8 TCP的4个定时器

TCP maintains 4 timers for each connection:

- **Retransmission Timer（超时重传定时器）**

 The timer is started during a transmission. A timeout causes a retransmission.

- **Persist Timer（坚持定时器）**

 Ensures that window size information is transmitted even if no data is transmitted.

- **Keepalive Timer（保活定时器）**

 Detects crashes on the other end of the connection.

- **2MSL Timer**

 Measures the time that a connection has been in the TIME_WAIT state.

### 6.8.1 Persist Timer

ACK的传输并不可靠，也就是说，TCP不对ACK报文段进行确认，TCP只确认那些包含有数据的ACK报文段。
**如果一个确认丢失了，则双方就有可能因为等待对方而使连接终止：接收方等待接收数据（因为它已经向发送方通告了一个非0的窗口），而发送方在等待允许它继续发送数据的窗口更新。为防止这种死锁情况的发生，发送方使用一个坚持定时器 (persist timer)来周期性地向接收方查询，以便发现窗口是否已增大。**

参考：TCP/IP Illustrated, Volume 1 The Protocols, Chapter 22 TCP Persist Timer

### 6.8.2 Keepalive Timer

保活定时器并不是TCP规范中的一部分，[RFC 1122](https://tools.ietf.org/html/rfc1122#page-101) 中提供了3个不使用保活定时器的理由：(1)在出现短暂有差错的情况下，这可能会使一个非常好的连接释放掉；(2)它们耗费不必要的带宽；(3)在按分组计费的情况下会在互联网上花掉更多的钱。
然而，很多TCP实现都提供了保活定时器。保活定时器是一个有争论的功能。 许多人认为如果需要，这个功能不应该在TCP中提供，而应该由应用程序来完成。
TCP关闭Keepalive功能时，如果没有应用级的定时器检测非活动状态，我们可以启动一个客户与服务器建立一个连接，双方不发送任何数据，然后离去数天或数月，这个连接依然保持着。 中间路由器可以崩溃或重启，只要两端的主机没有被重启，连接依然保持建立。
问题：当连接成功建立后，客户端拔去网线，客户端电脑重启后插入网线，启动客户端进程，会出现什么情况？
分析：首先如果TCP打开了keeplive选项，那么客户端关机时间久了以后，服务器就能感知到，如果keeplive关闭，或者客户端快速重启，那么这个时候，服务器没有发现异常。客户端重启过后是一个全新状态，由TCP状态转换图可知，客户端会发起SYN，而服务器收到客户端发来的SYN的话，发现在ESTABLISHED状态收到SYN是异常的，于是，返回RST，自己也复位改连接。客户端收到RST后，复位，重新连接。
在C程序中，可以使用函数 setsockopt 来使用keepalive定时器，完整的测试实例可参考：(http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/#examples)(http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/#examples)

参考：
TCP/IP Illustrated, Volume 1 The Protocols, Chapter 23 TCP Keepalive Timer
TCP Keepalive HOWTO: [http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/](http://www.tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/)

# 原文出处

- Author: cig01
- URL   ： [http://aandds.com/blog/network.html#orgd1da170](http://aandds.com/blog/network.html)

**本文摘自互联网，仅作个人学习使用,如发现侵犯您个人权限，请联系我删除 （email：jusonqiu@gmail.com）**
