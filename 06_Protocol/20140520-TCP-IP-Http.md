TCP/IP、Http、Socket的区别
=====================

网络由下往上分为**物理层**、**数据链路层**、**网络层**、**传输层**、**会话层**、**表示层**和**应用层**。

![Alt text](ProtocolImages/OSI-TCP.gif)

- **IP协议**对应于**网络层**
- **TCP协议**对应于**传输层**
- **HTTP协议**对应于**应用层**

三者从本质上来说没有可比性。

**socket**则是**对TCP/IP协议的封装和应用**（程序员层面上）。也可以说，**TPC/IP协议是传输层协议**，主要解决数据如何在网络中传输，而**HTTP**是**应用层协议**，主要解决如何包装数据。

**TCP/IP**和**HTTP**协议的**关系**：
- 在传输数据时，可以只使用（传输层）TCP/IP协议，但是那样的话，如果没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用到应用层协议。
- 应用层协议有很多，比如HTTP、FTP、TELNET等，也可以自己定义应用层协议。WEB使用HTTP协议作应用层协议，以封装HTTP文本信息，然后使用TCP/IP做传输层协议将它发到网络上。

**socket**是什么：
- **socket**是**对TCP/IP协议的封装**，Socket本身并不是协议，而是**一个调用接口（API）**，通过Socket，我们才能使用TCP/IP协议。
- 实际上，**Socket**跟**TCP/IP**协议**没有必然的联系**。
- Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以说，Socket的出现只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口，比如create、listen、connect、accept、send、read和write等等。
    
**socket**和**TCP/IP**协议的**关系**：
- **TCP/IP**只是一个**协议栈**。
- **TCP/IP**也要提供可供程序员做网络开发所用的**接口**，这就是**Socket编程接口**。
  
CSDN上有个比较形象的描述：
- *HTTP是轿车，提供了封装或者显示数据的具体形式；*
- *Socket是发动机，提供了网络通信的能力。*
  
实际上，传输层的TCP是基于网络层的IP协议的，而应用层的HTTP协议又是基于传输层的TCP协议的，而Socket本身不算是协议，它只是提供了一个针对TCP或者UDP编程的接口。

####1、什么是TCP连接的三次握手
- 第一次握手：客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；
- 第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；
- 第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。
  
**握手过程**中传送的**包**里**不包含数据**，三次**握手完毕后**，客户端与服务器才正式**开始传送数据**。
理想状态下，TCP连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP 连接都将被一直保持下去。断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次握手”。

####2、利用Socket建立网络连接的步骤
建立**Socket连接**至少需要一对**套接字**，其中一个运行于客户端，称为**ClientSocket**，另一个运行于服务器端，称为**ServerSocket**。

**套接字之间的连接过程**分为三个步骤：**服务器监听**，**客户端请求**，**连接确认**。
- 1） **服务器监听**：服务器端套接字并**不定位具体的客户端套接字**，而是处于**等待连接**的状态，**实时监控**网络状态，等待客户端的连接请求。
- 2） **客户端请求**：指客户端的套接字提出**连接请求**，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。
- 3） **连接确认**：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

####3、HTTP链接的特点
- **HTTP协议**即**超文本传送协议**(Hypertext Transfer Protocol)，是Web联网的基础，也是手机联网常用的协议之一，HTTP协议是建立在TCP协议之上的一种应用。
- **HTTP连接**最显著的**特点**：**客户端**发送的每次**请求**都需要**服务器**回送**响应**，在**请求结束**后，会**主动释放连接**。从建立连接到关闭连接的过程称为“**一次连接**”。

####4、TCP和UDP的区别
- 1） **TCP**是**面向链接**的，虽然说网络的不安全不稳定特性决定了多少次握手都不能保证连接的可靠性，但TCP的三次握手在最低限度上（实际上也很大程度上保证了）保证了连接的**可靠性**；而**UDP不是面向连接**的，UDP传送数据前并不与对方建立连接，对接收到的数据也不发送确认信号，发送端不知道数据是否会正确接收，当然也不用重发，所以说**UDP**是**无连接的**、**不可靠的**一种数据传输协议。
- 2） **UDP**的**开销更小数据传输速率更高**，因为不必进行收发数据的确认，所以UDP的**实时性更好**。