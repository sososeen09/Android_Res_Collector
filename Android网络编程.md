#Android网络编程
网络分层，每一层都建立在它的下层之上，并对自己的上层提供一定的服务，把实现细节对上一次屏蔽。
一台设备的n层与另一台设备的n层采用n层协议进行通讯，接收方和发送方的同层的协议必须一致，否则无法识别。

网络协议是网络上所有设备（网络服务器、计算器和交换机，路由器、防火墙等）之间的通讯协议，它规定了这些通讯时信息必须采用的格式和这个格式的意义。通过网络协议，网络各种设备才能够交换信息。

## 网络协议的层次结构如下：

- 结构中的每一层都规定有明确的任务及接口标准。
- 把用户的应用程序作为最高层
- 除了最高层，中间的每一层都为上层提供服务，同时又是下一层的用户
- 把物理通信线路作为最低层，它使用最高层传过来的参数，是提供服务的基础

开放系统互联参考模型（OSI/RM模型）七层模型（1978年），从高到低：

- 应用层：网络服务与最终用户的一个接口。协议有HTTP、FTP、TFTP 、SMTP、 SNMP、 DNS、 TELNET 、HTTPS、 POP3、 DHCP
- 表示层：数据的表示、安全、压缩。（在五层模型里面已经合并到了应用层），格式有，JPEG、ASCll、DECOIC、加密格式等
- 会话层：建立、管理、终止会话。（在五层模型里面已经合并到了应用层），对应主机进程，指本地主机与远程主机正在进行的会话
- 传输层：定义传输数据的协议端口号，以及流控和差错校验。协议有：TCP UDP，数据包一旦离开网卡即进入网络传输层
- 网络层：进行逻辑地址寻址，实现不同网络之间的路径选择。协议有：ICMP IGMP IP（IPV4 IPV6） ARP RARP
- 数据链路层：建立逻辑连接、进行硬件地址寻址、差错校验等功能。（由底层网络定义协议）将比特组合成字节进而组合成帧，用MAC地址访问介质，错误发现但不能纠正。
- 物理层：建立、维护、断开物理连接。（由底层网络定义协议）

![OSI模型](http://d.hiphotos.baidu.com/baike/w%3D268%3Bg%3D0/sign=ade49c9cb6003af34dbadb660d11a161/d50735fae6cd7b89056646160f2442a7d9330e18.jpg)

低4层完成数据传输服务，上面3层面向用户，每一层至少制定两项标准：

- 服务协议：给出该层所提供的服务的准确定义
- 服务规范：详细描述了该协议的动作和各种有关规范，以保证服务的提供

##TCP/IP模型
由于OSI模型难以实现，现实中广泛应用的是TCP/IP协议，分为四层
![](https://github.com/sososeen09/Android_Res_Collector/blob/master/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE.png)

- **应用层**：大多数普通和网络相关的程序与其它程序通信所使用的层。包含HTTP、HTTPS、文件传输协议FTP、接收电子邮件的POP3和IMAP一些、发送电子邮件使用的SMTP协议，以及远程登录使用的SSH和Telnet协议。
- **传输层**：响应来自应用层的服务请求，并向网络层发出服务请求。提供两台主机之间透明的数据传输，通常用于端到端链接、流量控制或者错误恢复。两个最重要的协议是TCP（Transmission Control Protocol，传输控制协议）和UDP（User Datagram Protocol，用户数据报协议）
- **网络层**：网络层提供端到端的数据包交付，负责从把数据包发送到目的地，任务包括网络路由、差错控制和IP编址，重要的有IP协议（IPV4、IPV6）、ICMP（Internet Control Message Protocaol，Internet控制报文协议）和IPSec（Internet Protocol Security，Internet协议安全）
- **网络接口层**：有时也称链路层或者数据链路层，是TCP/IP参考模型的最底层，负责通过网络发送和接收IP数据报；允许主机连入网络时使用多种现成的流行的技术，如以太网、令牌网、帧中继、ATM、X.25、DDN、SDH、WDM等。

### IP协议
互联网协议（Internet Protocol，IP）用于报文交换网络的一种面向数据的协议。是TCP/IP协议中网络层的重要协议，任务是根据源主机和目的主机的地址传送数据。

### TCP协议
传输控制协议，面向连接的、可靠的、基于字节流的传输层通信协议。

不同主机的应用层之间经常需要可靠的、像管道一样的连接，但是网络层不提供这样的流机制，其只能提供不可靠的包交换，所以传输层就自然出现了。

应用层向传输层发送用于网间传输的、用8位字节表示的数据流，然后TCP协议把数据流分成适当长度的报文段（受该计算机连接的网络的数据链路层的最大传送单元MTU的限制）。

TCP为了不丢包，给每个包一个序号，同时序号也保证了传送到接收端实体的包能按序接收。然后接收端实体为已经成功收到的包发回一个响应的确认（ACK）；如果发送端实体在合理的往返时延（RTT）内未收到确认，那么对应的数据包（假设丢失了）将会被重传。

TCP协议用一个校验和（Checksum）函数来检验数据是否有错误，在发送和接收时都要计算校验和。

![3次握手建立连接](http://img.blog.csdn.net/20150816204217054?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![4次挥手断开连接](http://img.blog.csdn.net/20150816205206997?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
### UDP协议
用户数据报协议是面向无连接的传出层协议，提供面向事务的简单不可靠信息传输服务。UDP协议基本上是IP协议与上层协议的接口，UDP协议适用于端口分别运行在同一台设备上的多个应用程序中。

不可靠、不需要建立连接，只能传输少量的数据

## Andorid中使用TCP、UDP协议

为了使程序员不必费心于上述底层具体细节，同过Socket对网络纠错、包大小、包重传等进行了封装。Socket是应用层与TCP/IP协议簇通信的中间软件抽象层，是一组接口。把复杂的TCP/IP协议簇隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。Socket用于描述IP地址和端口，是一个通信链的句柄。应用程序通产通过套接字向网络发出请求或者应答网络请求。

Socket套接字、也可称为“插座”，客户端的Socket是插头、服务端的ServerSocket是插座。

基本操作：
- 连接远程机器
- 发送数据
- 接收数据
- 关闭连接
- 绑定端口
- 监听到达数据
- 在绑定的端口上接受来自远程机器的连接

Socket一般有两种类型，TCP套接字和UDP套接字。TCP把小子分解成数据包（数据报，datagrams）并在接收端以正确的顺序把他们组装起来。TCP还处理对遗失数据包的重传请求，位于上层的应用层处理的事情少。UDP不提供装配和重传这些功能，它只是向前传送信息包，位于上层的应用层必须确保消息是完整的，并且以正确的顺序装配。

### 使用TCP通信

### 使用UDP通信
**UDP服务器端工作步骤**：
- 使用DatagramSocket(int port)创建一个数据报套接字，并绑定到指定端口上
- 调用DatagramPacket（byte[] buf,int length），建立一个字节数组以接收UDP包。
- 调用DatagramSocket类的receive()，接受UDP包
- 关闭数据报套接字

**UDP客户端主要步骤**：
- 调用DatagramSocket()创建一个数据包套接字
- 调用DatagramPacket（byte[] buf,int offset,int length,InetAddress address,int port)，建立要发送的UDP包
- 调用DatagramSocket类的send发送UDP包
- 关闭数据报套接字


## HTTP协议（HyperText Transport Protocol 超文本传输协议）
互联网应用最为广泛的一种网络协议，所有的WWW文件都必须遵守这个标准。

特点：
- 支持C/S(客户端/服务器)模式
- 简单快速
- 灵活
- 无连接
- 无状态

**请求报文**：一个HTTP请求报文由请求行、请求报头、空行和请求数据4个部分组成。

**响应报文**：也是由4个部分组成，状态行、消息报头、空行、响应正文



### Android中的网络请求库
HttpUrlConnection和HttpClient

## 加密
加密是通过加密算法和加密密匙将明文转为密文的过程，解密是其逆过程。加密有：
- 对称加密（DES、AES等），DES是数据加密标准（Data Encryption Standard）的简称，AES是高级加密标准（Anvanced Encryption Standard）
- 非对称加密（RSA）
- 单向加密（MD5）：Message-digest Algorithm 5信息摘要算法
