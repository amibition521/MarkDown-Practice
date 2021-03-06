### Http协议相关

1.一次网络请求的过程

* 建立TCP连接  三次握手
* 发送请求命令   GET/sample/hello.jsp HTTP/1/1
* 发送请求头信息   user-agent ,host等信息
* Web服务器应答   版本号和协议状态码     HTTP/1.1  200 OK
* Web服务器发送应答头信息
* Web服务器发送数据
* Web服务器关闭TCP连接

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-ef43bfa84bb68de1.png)

2.TCP/IP协议

Transmission Control Protocol/Internet Protocol --- 传输控制协议

2.1四层

* 链路层

  通常包括操作系统中的设备驱动程序和计算机中对应的网络接口卡。

* 网络层

  网络层协议包括IP协议（网际协议），ICMP协议（Internet互联网控制报文协议），以及IGMP协议（Internet组管理协议）。

  IP是一种网络层协议，提供的是一种不可靠的服务，它只是尽可能快地把分组从源结点送到目的结点，但是并不提供任何可靠性保证。同时被TCP和UDP使用。TCP和UDP的每组数据都通过端系统和每个中间路由器中的IP层在互联网中进行传输。

  ICMP是IP协议的附属协议。IP层用它来与其他主机或路由器交换错误报文和其他重要信息。

  IGMP是Internet组管理协议。它用来把一个UDP数据报多播到多个主机。

* 传输层

  * TCP   --- 传输控制协议

    为两台主机提供高可靠性的数据通信。它所做的工作包括把应用程序交给它的数据分成合适的小块交给下面的网络层，确认接收到的分组，设置发送最后确认分组的超时时钟等。由于运输层提供了高可靠性的端到端的通信，因此应用层可以忽略所有这些细节。为了提供可靠的服务，TCP采用了超时重传、发送和接收端到端的确认分组等机制。

  * UDP ----用户数据报协议

    它只是把称作数据报的分组从一台主机发送到另一台主机，但并不保证该数据报能到达另一端。一个数据报是指从发送方传输到接收方的一个信息单元（例如，发送方指定的一定字节数的信息）。UDP协议任何必需的可靠性必须由应用层来提供。

* 应用层

  * HTTP
  * FTP --- File Transfer Protocol，文件传输协议
  * DNS服务 --- Domain Name System，域名系统

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-e533b0cdd7fca359.png)

HTTP请求和相应的过程：

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-ecf824604debcdf1.png)

3.TCP的三次握手

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-f2fe3775bd2678c2.png)

第一次握手：建立连接。客户端发送连接请求报文段，将SYN位置为1，Sequence Number为x；然后，客户端进入SYN_SEND状态，等待服务器的确认；

第二次握手：服务器收到SYN报文段。服务器收到客户端的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1)；同时，自己自己还要发送SYN请求信息，将SYN位置为1，Sequence Number为y；服务器端将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给客户端，此时服务器进入SYN_RECV状态；

第三次握手：客户端收到服务器的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

##### 为什么要三次握手

为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。

4.TCP的四次挥手

当客户端和服务器通过三次握手建立了TCP连接以后，当数据传送完毕，肯定是要断开TCP连接的啊。那对于TCP的断开连接，这里就有了神秘的“四次分手”。

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-c1c59148f8b26c43.png)

第一次分手：主机1（可以使客户端，也可以是服务器端），设置Sequence Number，向主机2发送一个FIN报文段；此时，主机1进入FIN_WAIT_1状态；这表示主机1没有数据要发送给主机2了；

第二次分手：主机2收到了主机1发送的FIN报文段，向主机1回一个ACK报文段，Acknowledgment Number为Sequence Number加1；主机1进入FIN_WAIT_2状态；主机2告诉主机1，我“同意”你的关闭请求；

第三次分手：主机2向主机1发送FIN报文段，请求关闭连接，同时主机2进入LAST_ACK状态；

第四次分手：主机1收到主机2发送的FIN报文段，向主机2发送ACK报文段，然后主机1进入TIME_WAIT状态；主机2收到主机1的ACK报文段以后，就关闭连接；此时，主机1等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，主机1也可以关闭连接了。

##### 为什么要四次分手

TCP协议是一种面向连接的、可靠的、基于字节流的运输层通信协议。TCP是全双工模式，这就意味着，当主机1发出FIN报文段时，只是表示主机1已经没有数据要发送了，主机1告诉主机2，它的数据已经全部发送完毕了；但是，这个时候主机1还是可以接受来自主机2的数据；当主机2返回ACK报文段时，表示它已经知道主机1没有数据发送了，但是主机2还是可以发送数据到主机1的；当主机2也发送了FIN报文段时，这个时候就表示主机2也没有数据要发送了，就会告诉主机1，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。

5.HTTP协议

一个基于请求和响应、无状态的、应用层的协议，常基于TCP/IP协议传输数据；

5.1四个基于

**请求与响应：**客户端发送请求，服务器端响应数据

**无状态的：**协议对于事务处理没有记忆能力，客户端第一次与服务器建立连接发送请求时需要进行一系列的安全认证匹配等，因此增加页面等待时间，当客户端向服务器端发送请求，服务器端响应完毕后，两者断开连接，也不保存连接状态，一刀两断！恩断义绝！从此路人！下一次客户端向同样的服务器发送请求时，由于他们之前已经遗忘了彼此，所以需要重新建立连接。

**应用层：**Http是属于应用层的协议，配合TCP/IP使用。

**TCP/IP：**Http使用TCP作为它的支撑运输协议。HTTP客户机发起一个与服务器的TCP连接，一旦连接建立，浏览器（客户机）和服务器进程就可以通过套接字接口访问TCP。

5.2 HTTP请求报文

一个HTTP请求报文由请求行（request line）、请求头部（header）、空行和请求数据4个部分组成，下图给出了请求报文的一般格式。

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-cd59a3899ef546e1.png)

##### 1.请求行

请求行分为三个部分：请求方法、请求地址和协议版本

**请求方法**

HTTP/1.1 定义的请求方法有8种：GET、POST、PUT、DELETE、PATCH、HEAD、OPTIONS、TRACE。

最常的两种GET和POST，如果是RESTful接口的话一般会用到GET、POST、DELETE、PUT。

**请求地址**

URL:统一资源定位符，是一种自愿位置的抽象唯一识别方法。

**协议版本**

协议版本的格式为：HTTP/主版本号.次版本号，常用的有HTTP/1.0和HTTP/1.1

#### 2.请求头

请求头部为请求报文添加了一些附加信息，由“名/值”对组成，每行一对，名和值之间使用冒号分隔。

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-539378eee14fa322.png)

请求头部的最后会有一个空行，表示请求头部结束，接下来为请求数据，这一行非常重要，必不可少。

#### 3.请求数据

6.HTTP响应报文

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-c6ee8f8526f59fc0.png)

##### 1.状态行

由3部分组成，分别为：协议版本，状态码，状态码描述。

其中协议版本与请求报文一致，状态码描述是对状态码的简单描述，所以这里就只介绍状态码。

**状态码**

状态代码为3位数字。
1xx：指示信息--表示请求已接收，继续处理。
2xx：成功--表示请求已被成功接收、理解、接受。
3xx：重定向--要完成请求必须进行更进一步的操作。
4xx：客户端错误--请求有语法错误或请求无法实现。
5xx：服务器端错误--服务器未能实现合法的请求。

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-8f3bf059bc4365e3.png)

2.响应头部

![image](https://github.com/amibition521/MarkDown-Practice/raw/master/interview/network/3985563-33ed95479f541a07.png)

##### 3.响应数据

用于存放需要返回给客户端的数据信息。

通过以上步骤，数据已经传递完毕，HTTP/1.1会维持持久连接，但持续一段时间总会有关闭连接的时候，这时候据需要断开TCP连接。
