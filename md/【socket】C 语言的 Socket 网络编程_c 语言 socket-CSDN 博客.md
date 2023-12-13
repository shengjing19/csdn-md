> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/bandaoyu/article/details/83312254)

**目录**

[Socket 网络编程](#t0)

[1、网络知识](#t1)

[网络中进程之间如何通信？](#t2)

[什么是 Socket？](#t3)

[socket 一词的起源](#t4)

 [怎么理解端口？](#t5)

 [怎么理解 socket ？](#t6)

[2. 客户 / 服务器模式](#t7)

[2.1 服务器端：](#t8)

[2.2 客户端：](#t9)

[4. 套接字函数](#t10)

[4.1 创建套接字──socket()](#t11)

[4.2 指定本地地址──bind()](#t12) 

[4.3 建立套接字连接──connect() 与 accept()](#4.3%20%E5%BB%BA%E7%AB%8B%E5%A5%97%E6%8E%A5%E5%AD%97%E8%BF%9E%E6%8E%A5%E2%94%80%E2%94%80connect%28%29%E4%B8%8Eaccept%28%29)

[4.4 监听连接──listen()](#t14) 

[4.5 数据传输──send() 与 recv()](#4.5%20%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E2%94%80%E2%94%80send%28%29%E4%B8%8Erecv%28%29%C2%A0) 

[4.6 关闭套接字──close](#t16)

[4.6 recv 和 read|send 和 write 的区别](#t17)

[编程实例](#t18)

[Linux 下 C 语言的 Socket 编程例子（多线程）](#t19)

[Windows 下 C 语言的 Socket 编程例子](#t20)

[【Socket】有很多 BUG，测试人员要注意。](#t21)

[网络字节序与主机字节序](#t22)

[3.1、socket() 函数](#t23)

[3.2、bind() 函数](#t24)

[3.3、listen()、connect() 函数](#3.3%E3%80%81listen%28%29%E3%80%81connect%28%29%E5%87%BD%E6%95%B0)

[3.4、accept() 函数](#t26)

[3.5、read()、write() 等函数](#3.5%E3%80%81read%28%29%E3%80%81write%28%29%E7%AD%89%E5%87%BD%E6%95%B0)

[3.6、close() 函数](#t28)

[4、socket 中 TCP 的三次握手建立连接详解](#t29)

[5、socket 中 TCP 的四次握手释放连接详解](#t30)

 [6、一个例子（实践一下）](#t31)

[7、动动手](#t32)

* * *

[Socket](https://so.csdn.net/so/search?q=Socket&spm=1001.2101.3001.7020) 网络编程
-----------------------------------------------------------------------------

### 1、网络知识

#### 网络中进程之间如何通信？

本地的进程间通信（IPC）有很多种方式，但可以总结为下面 4 类：

*   消息传递（管道、FIFO、消息队列）
    
*   同步（互斥量、条件变量、读写锁、文件和写记录锁、信号量）
    
*   共享内存（匿名的和具名的）
    
*   远程过程调用（Solaris 门和 Sun RPC）
    

但这些都不是本文的主题！我们要讨论的是网络中进程之间如何通信？首要解决的问题是如何唯一标识一个进程，否则通信无从谈起！在本地可以通过进程 PID 来唯一标识一个进程，但是在网络中这是行不通的。其实 TCP/IP 协议族已经帮我们解决了这个问题，**网络层的 “ip 地址”** 可以唯一**标识网络中的主机**，而**传输层的 “协议 + 端口”** 可以唯一标识主机中的**应用程序（进程）**。这样利用三元组（ip 地址，协议，端口）就可以标识网络的进程了，网络中的进程通信就可以利用这个标志与其它进程进行交互。

使用 TCP/IP 协议的应用程序通常采用应用编程接口：UNIX  BSD 的套接字（socket）和 UNIX System V 的 TLI（已经被淘汰），来实现网络进程之间的通信。就目前而言，几乎所有的应用程序都是采用 socket，而现在又是网络时代，网络中进程通信是无处不在，这就是我为什么说 “一切皆 socket”。

#### 什么是 Socket？

socket 一种特殊的文件，一些 socket 函数就是对其进行的操作（读 / 写 IO、打开、关闭），这些函数我们在后面进行介绍。

####  怎么理解端口？

我们平时说的端口一般都是指逻辑端口，比如浏览器用的 80 端口，FTP 工具用的 21 端口。由于网络工具众多，于是对网络端口做了编号，从 0 到 65535。

其中 0 - 1023 是公认的端口号，就是已经被一些知名的软件给占用了。留给我们程序里面使用的是 1024 - 65535。

程序 socket 通过绑定操作占领 x 端口，接下来其他程序将不能使用 x 端口。一旦 x 端口收到数据，系统都会转发给该程序，所以不会出现微信好友发送的数据，被 QQ 给收到了。可以简单地理解成，操作系统通过端口号，把不同的应用程序区分开。  
作者：linux  
链接：https://www.zhihu.com/question/535823141/answer/2538570947

####  怎么理解 socket ？

socket 套接字 莫名其妙，难道真是下面图片的而来？

套接字 (socket) 属于糟糕的翻译，因为它太生僻。“套接”生活中用得少，加个“字”，成了套接字。当你学了 socket 以后，你还是很难将名字与它的意思联系起来。香港翻译成：網路插座 感觉贴近一些

![](https://img-blog.csdnimg.cn/img_convert/dae18e74a0846c443dad581c9b2014fa.png)

### 2. 客户 / 服务器模式

  
在 TCP/IP 网络应用中，通信的两个进程间相互作用的主要模式是客户 / 服务器（Client/Server, C/S）模式，即客户向服务器发出服务请求，服务器接收到请求后，提供相应的服务。

客户 / 服务器模式的建立基于以下两点：  
（1）首先，建立网络的起因是网络中软硬件资源、运算能力和信息不均等，需要共享，从而造就拥有众多资源的主机提供服务，资源较少的客户请求服务这一非对等作用。  
（2）其次，网间进程通信完全是异步的，相互通信的进程间既不存在父子关系，又不共享内存缓冲区，因此需要一种机制为希望通信的进程间建立联系，为二者的数据交换提供同步，这就是基于客户 / 服务器模式的 TCP/IP。  
 

#### 2.1 服务器端：

  
其过程是首先服务器方要先启动，并根据请求提供相应服务：  
（1）打开一通信通道并告知本地主机，它愿意在某一公认地址上的某端口（如 FTP 的端口可能为 21）接收客户请求；  
（2）等待客户请求到达该端口；  
（3）接收到客户端的服务请求时，处理该请求并发送应答信号。接收到并发服务请求，要激活一新进程来处理这个客户请求（如 UNIX 系统中用 fork、exec）。新进程处理此客户请求，并不需要对其它请求作出应答。服务完成后，关闭此新进程与客户的通信链路，并终止。  
（4）返回第（2）步，等待另一客户请求。  
（5）关闭服务器

####   
2.2 客户端：

  
（1）打开一通信通道，并连接到服务器所在主机的特定端口；  
（2）向服务器发服务请求报文，等待并接收应答；继续提出请求......  
（3）请求结束后关闭通信通道并终止。

从上面所描述过程可知：  
（1）客户与服务器进程的作用是非对称的，因此代码不同。  
（2）服务器进程一般是先启动的。只要系统运行，该服务进程一直存在，直到正常或强迫终止。  
3. 基本 TCP 套接字编程  
基于 TCP 的套接字编程的所有客户端和服务器端都是从调用 socket 开始，它返回一个套接字描述符。客户端随后调用 connect 函数，服务器端则调用 bind、listen 和 accept 函数。套接字通常使用标准的 close 函数关闭，但是也可以使用 shutdown 函数关闭套接字。

下图为 TCP 套接字编程流程图：

![](https://img-blog.csdn.net/20151223151413413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 4. 套接字函数

####   
4.1 创建套接字──socket()

  
应用程序在使用套接字前，首先必须拥有一个套接字，系统调用 socket() 向应用程序提供创建套接字的手段，

其调用格式如下：  
SOCKET PASCAL FAR socket(int af, int type, int protocol);

该调用要接收三个参数：af、type、protocol。参数 af 指定通信发生的区域：AF_UNIX、AF_INET、AF_NS 等，而 DOS、WINDOWS 中仅支持 AF_INET，它是网际网区域。因此，地址族与协议族相同。参数 type 描述要建立的套接字的类型。

这里分三种：  
（1）一是 TCP 流式套接字 (SOCK_STREAM) 提供了一个面向连接、可靠的数据传输服务，数据无差错、无重复地发送，且按发送顺序接收。内设流量控制，避免数据流超限；数据被看作是字节流，无长度限制。文件传送协议（FTP）即使用流式套接字。  
（2）二是数据报式套接字 (SOCK_DGRAM) 提供了一个无连接服务。数据包以独立包形式被发送，不提供无错保证，数据可能丢失或重复，并且接收顺序混乱。网络文件系统（NFS）使用数据报式套接字。  
（3）三是原始式套接字 (SOCK_RAW) 该接口允许对较低层协议，如 IP、ICMP 直接访问。常用于检验新的协议实现或访问现有服务中配置的新设备。  
参数 protocol 说明该套接字使用的特定协议，如果调用者不希望特别指定使用的协议，则置为 0，使用默认的连接模式。根据这三个参数建立一个套接字，并将相应的资源分配给它，同时返回一个整型套接字号。因此，socket()系统调用实际上指定了相关五元组中的 “协议” 这一元。

####   
4.2 指定本地地址──bind() 

  
当一个套接字用 socket()创建后，存在一个名字空间 (地址族)，但它没有被命名。bind() 将套接字地址（包括本地主机地址和本地端口地址）与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关。

其调用格式如下：  
int PASCAL FAR bind(SOCKET s, const struct sockaddr FAR * name, int namelen);

参数 s 是由 socket()调用返回的并且未作连接的套接字描述符 (套接字号)。参数 name 是赋给套接字 s 的本地地址（名字），其长度可变，结构随通信域的不同而不同。namelen 表明了 name 的长度。如果没有错误发生，bind() 返回 0。否则返回 SOCKET_ERROR。

#### 4.3 建立套接字连接──connect() 与 accept()

  
这两个系统调用用于完成一个完整相关的建立，其中 connect() 用于建立连接。accept() 用于使服务器等待来自某客户进程的实际连接。

connect() 的调用格式如下：  
int PASCAL FAR connect(SOCKET s, const struct sockaddr FAR * name, int namelen);

参数 s 是欲建立连接的本地套接字描述符。参数 name 指出说明对方套接字地址结构的指针。对方套接字地址长度由 namelen 说明。  
如果没有错误发生，connect() 返回 0。否则返回值 SOCKET_ERROR。在面向连接的协议中，该调用导致本地系统和外部系统之间连接实际建立。  
由于地址族总被包含在套接字地址结构的前两个字节中，并通过 socket() 调用与某个协议族相关。因此 bind() 和 connect() 无须协议作为参数。

accept() 的调用格式如下：  
SOCKET PASCAL FAR accept(SOCKET s, struct sockaddr FAR* addr, int FAR* addrlen);

参数 s 为本地套接字描述符，在用做 accept() 调用的参数前应该先调用过 listen()。addr 指向客户方套接字地址结构的指针，用来接收连接实体的地址。addr 的确切格式由套接字创建时建立的地址族决定。addrlen 为客户方套接字地址的长度（字节数）。如果没有错误发生，accept() 返回一个 SOCKET 类型的值，表示接收到的套接字的描述符。否则返回值 INVALID_SOCKET。

accept() 用于面向连接服务器。参数 addr 和 addrlen 存放客户方的地址信息。调用前，参数 addr 指向一个初始值为空的地址结构，而 addrlen 的初始值为 0；调用 accept() 后，服务器等待从编号为 s 的套接字上接受客户连接请求，而连接请求是由客户方的 connect() 调用发出的。当有连接请求到达时，accept() 调用将请求连接队列上的第一个客户方套接字地址及长度放入 addr 和 addrlen，并创建一个与 s 有相同特性的新套接字号。新的套接字可用于处理服务器并发请求。

四个套接字系统调用，socket()、bind()、connect()、accept()，可以完成一个完全五元相关的建立。socket() 指定五元组中的协议元，它的用法与是否为客户或服务器、是否面向连接无关。bind() 指定五元组中的本地二元，即本地主机地址和端口号，其用法与是否面向连接有关：在服务器方，无论是否面向连接，均要调用 bind()，若采用面向连接，则可以不调用 bind()，而通过 connect() 自动完成。若采用无连接，客户方必须使用 bind() 以获得一个唯一的地址。

####   
4.4 监听连接──listen() 

  
此调用用于面向连接服务器，表明它愿意接收连接。listen() 需在 accept() 之前调用，

其调用格式如下：  
int PASCAL FAR listen(SOCKET s, int backlog);

参数 s 标识一个本地已建立、尚未连接的套接字号，服务器愿意从它上面接收请求。backlog 表示请求连接队列的最大长度，用于限制排队请求的个数，目前允许的最大值为 5。如果没有错误发生，listen() 返回 0。否则它返回 SOCKET_ERROR。  
listen() 在执行调用过程中可为没有调用过 bind() 的套接字 s 完成所必须的连接，并建立长度为 backlog 的请求连接队列。  
调用 listen() 是服务器接收一个连接请求的四个步骤中的第三步。它在调用 socket() 分配一个流套接字，且调用 bind() 给 s 赋于一个名字之后调用，而且一定要在 accept() 之前调用。

#### 4.5 数据传输──send() 与 recv() 

  
当一个连接建立以后，就可以传输数据了。常用的系统调用有 send() 和 recv()。  
send() 调用用于 s 指定的已连接的数据报或流套接字上发送输出数据，格式如下：

int PASCAL FAR send(SOCKET s, const char FAR *buf, int len, int flags);

参数 s 为已连接的本地套接字描述符。buf 指向存有发送数据的缓冲区的指针，其长度由 len 指定。flags 指定传输控制方式，如是否发送带外数据等。如果没有错误发生，send() 返回总共发送的字节数。否则它返回 SOCKET_ERROR。

recv() 调用用于 s 指定的已连接的数据报或流套接字上接收输入数据，格式如下：

int PASCAL FAR recv(SOCKET s, char FAR *buf, int len, int flags);

参数 s 为已连接的套接字描述符。buf 指向接收输入数据缓冲区的指针，其长度由 len 指定。flags 指定传输控制方式，如是否接收带外数据等。如果没有错误发生，recv() 返回总共接收的字节数。如果连接被关闭，返回 0。否则它返回 SOCKET_ERROR。

#### 4.6 关闭套接字──close

  
close() 关闭套接字 s，并释放分配给该套接字的资源；如果 s 涉及一个打开的 TCP 连接，则该连接被释放。

#### 4.6 recv 和 read|send 和 write 的区别

int recv(int sockfd,void *buf,int len,int flags)   
recv 比 read 的功能强大点，体现在 recv 提供的 flags 参数上，  
recv 最终的实现还是要调用 read。  
recv 和 read 都可以操作阻塞或非阻塞，阻塞非阻塞与 recv 和 read 没关系，它是 socket 的属性，函数 fcntl 可以设置。

[read 和 recv 等函数的区别 | 学步园](https://www.xuebuyuan.com/1794019.html "read和recv等函数的区别 | 学步园")

编程实例
----

### Linux 下 C 语言的 Socket 编程例子（[多线程](https://so.csdn.net/so/search?q=%E5%A4%9A%E7%BA%BF%E7%A8%8B&spm=1001.2101.3001.7020)）

2018-10-02 12:50:42

考虑到了关闭连接退出机制，多线程编程，以及线程参数的传递，值得学习

服务端

```
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netinet/ip.h>
#include <arpa/inet.h>
#include<unistd.h>
#include<errno.h>
#include<pthread.h>
#define MAXCONN 2
#define ERRORCODE -1
#define BUFFSIZE 1024
int count_connect = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
struct pthread_socket
{
	int socket_d;
	pthread_t thrd;
};
static void *thread_send(void *arg)
{
	char buf[BUFFSIZE];
	int sd = *(int *) arg;
	memset(buf, 0, sizeof(buf));
	strcpy(buf, "hello,welcome to you! \n");
	if (send(sd, buf, strlen(buf), 0) == -1)
	{
		printf("send error:%s \n", strerror(errno));
		return NULL;
	}
	while (1)
	{
		memset(buf, 0, sizeof(buf));
		read(STDIN_FILENO, buf, sizeof(buf));
		if (send(sd, buf, strlen(buf), 0) == -1)
		{
			printf("send error:%s \n", strerror(errno));
			break;
		}
	}
	return NULL;
}
static void* thread_recv(void *arg)
{
	char buf[BUFFSIZE];
	struct pthread_socket *pt = (struct pthread_socket *) arg;
	int sd = pt->socket_d;
	pthread_t thrd = pt->thrd;
	while (1)
	{
		memset(buf, 0, sizeof(buf));
		int rv = recv(sd, buf, sizeof(buf), 0); //是阻塞的
		if (rv < 0)
		{
			printf("recv error:%s \n", strerror(errno));
			break;
        	}
        	if (rv == 0) // 这种情况说明client已经关闭socket连接
        	{
            		break;
        	}
        	printf("%s", buf); //输出接受到内容
    	}
    	pthread_cancel(thrd);
    	pthread_mutex_lock(&mutex);
    	count_connect--;
    	pthread_mutex_unlock(&mutex);
    	close(sd);
    	return NULL;
}
 
static int create_listen(int port)
{
 
    	int listen_st;
    	struct sockaddr_in sockaddr; //定义IP地址结构
    	int on = 1;
    	listen_st = socket(AF_INET, SOCK_STREAM, 0); //初始化socket
    	if (listen_st == -1)
    	{
        	printf("socket create error:%s \n", strerror(errno));
        	return ERRORCODE;
    	}
    	if (setsockopt(listen_st, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)) == -1) //设置ip地址可重用
    	{
        	printf("setsockopt error:%s \n", strerror(errno));
        	return ERRORCODE;
    	}
    	sockaddr.sin_port = htons(port); //指定一个端口号并将hosts字节型传化成Inet型字节型（大端或或者小端问题）
    	sockaddr.sin_family = AF_INET;    //设置结构类型为TCP/IP
    	sockaddr.sin_addr.s_addr = htonl(INADDR_ANY);    //服务端是等待别人来连，不需要找谁的ip
    	//这里写一个长量INADDR_ANY表示server上所有ip，这个一个server可能有多个ip地址，因为可能有多块网卡
    	if (bind(listen_st, (struct sockaddr *) &sockaddr, sizeof(sockaddr)) == -1)
    	{
       		printf("bind error:%s \n", strerror(errno));
        	return ERRORCODE;
    	}
 
    	if (listen(listen_st, 5) == -1) //     服务端开始监听
    	{
        	printf("listen error:%s \n", strerror(errno));
        	return ERRORCODE;
    	}
    	return listen_st;
}
 
int accept_socket(int listen_st)
{
    	int accept_st;
    	struct sockaddr_in accept_sockaddr; //定义accept IP地址结构
    	socklen_t addrlen = sizeof(accept_sockaddr);
    	memset(&accept_sockaddr, 0, addrlen);
    	accept_st = accept(listen_st, (struct sockaddr*) &accept_sockaddr,&addrlen);
    	//accept 会阻塞直到客户端连接连过来 服务端这个socket只负责listen 是不是有客服端连接过来了
    	//是通过accept返回socket通信的
    	if (accept_st == -1)
    	{
        	printf("accept error:%s \n", strerror(errno));
        	return ERRORCODE;
    	}
   	printf("accpet ip:%s \n", inet_ntoa(accept_sockaddr.sin_addr));
    	return accept_st;
}
int run_server(int port)
{
    	int listen_st = create_listen(port);    //创建监听socket
    	pthread_t send_thrd, recv_thrd;
    	struct pthread_socket ps;
    	int accept_st;
    	if (listen_st == -1)
    	{
        	return ERRORCODE;
    	}
    	printf("server start \n");
    	while (1)
    	{
        	accept_st = accept_socket(listen_st); //获取连接的的socket
        	if (accept_st == -1)
        	{
            		return ERRORCODE;
        	}
        	if (count_connect >= MAXCONN)
        	{
            		printf("connect have already be full! \n");
            		close(accept_st);
            		continue;
        	}
        	pthread_mutex_lock(&mutex);
        	count_connect++;
        	pthread_mutex_unlock(&mutex);
        	if (pthread_create(&send_thrd, NULL, thread_send, &accept_st) != 0) //创建发送信息线程
        	{
            		printf("create thread error:%s \n", strerror(errno));
            		break;
 
        	}
        	pthread_detach(send_thrd);        //设置线程可分离性，这样的话主线程就不用join
        	ps.socket_d = accept_st;
        	ps.thrd = send_thrd;
        	if (pthread_create(&recv_thrd, NULL, thread_recv, &ps) != 0)//创建接收信息线程
        	{
            		printf("create thread error:%s \n", strerror(errno));
            		break;
        	}
        	pthread_detach(recv_thrd); //设置线程为可分离，这样的话，就不用pthread_join
    	}
    close(accept_st);
    close(listen_st);
    return 0;
}
//server main
int main(int argc, char *argv[])
{
    	if (argc < 2)
    	{
        	printf("Usage:port,example:8080 \n");
        	return -1;
    	}
    	int port = atoi(argv[1]);
    	if (port == 0)
    	{
        	printf("port error! \n");
    	} 
	else
    	{
        	run_server(port);
    	}
    return 0;
}
```

  客户端：

```
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netinet/ip.h>
#include <arpa/inet.h>
#include<unistd.h>
#include<errno.h>
#include<pthread.h>
 
#define BUFFSIZE 1024
#define ERRORCODE -1
 
static void *thread_send(void *arg)
{
	char buf[BUFFSIZE];
	int sd = *(int *) arg;
	while (1)
	{
		memset(buf, 0, sizeof(buf));
		read(STDIN_FILENO, buf, sizeof(buf));
		if (send(sd, buf, strlen(buf), 0) == -1)
		{
			printf("send error:%s \n", strerror(errno));
			break;
		}
	}
	return NULL;
}
static void *thread_recv(void *arg)
{
	char buf[BUFFSIZE];
	int sd = *(int *) arg;
	while (1)
	{
		memset(buf, 0, sizeof(buf));
		int rv = recv(sd, buf, sizeof(buf), 0);
		if (rv <= 0)
		{
			if(rv == 0) //server socket关闭情况
			{
				printf("server have already full !\n");
				exit(0);//退出整个客服端
			}
		printf("recv error:%s \n", strerror(errno));
		break;
		}
		printf("%s", buf);//输出接收到的内容
	}	
	return NULL;
}
int run_client(char *ip_str, int port)
{
	int client_sd;
	int con_rv;
	pthread_t thrd1, thrd2;
	struct sockaddr_in client_sockaddr; //定义IP地址结构
	client_sd = socket(AF_INET, SOCK_STREAM, 0);
	if (client_sd == -1)
	{
		printf("socket create error:%s \n", strerror(errno));
		return ERRORCODE;
	}
	memset(&client_sockaddr, 0, sizeof(client_sockaddr));
	client_sockaddr.sin_port = htons(port); //指定一个端口号并将hosts字节型传化成Inet型字节型（大端或或者小端问题）
	client_sockaddr.sin_family = AF_INET; //设置结构类型为TCP/IP
	client_sockaddr.sin_addr.s_addr = inet_addr(ip_str);//将字符串的ip地址转换成int型,客服端要连接的ip地址
	con_rv = connect(client_sd, (struct sockaddr*) &client_sockaddr,
	sizeof(client_sockaddr));
	//调用connect连接到指定的ip地址和端口号,建立连接后通过socket描述符通信
	if (con_rv == -1)
	{
		printf("connect error:%s \n", strerror(errno));
		return ERRORCODE;
	}
	if (pthread_create(&thrd1, NULL, thread_send, &client_sd) != 0)
	{
		printf("thread error:%s \n", strerror(errno));
		return ERRORCODE;
	}
	if (pthread_create(&thrd2, NULL, thread_recv, &client_sd) != 0)
	{
		printf("thread error:%s \n", strerror(errno));
		return ERRORCODE;
	}
	pthread_join(thrd2, NULL);// 等待线程退出
	pthread_join(thrd1, NULL);
	close(client_sd);
	return 0;
}
int main(int argc, char *argv[])
{
	if (argc < 3)
	{
		printf("Usage:ip port,example:127.0.0.1 8080 \n");
		return ERRORCODE;
	}
	int port = atoi(argv[2]);
	char *ip_str = argv[1];
	run_client(ip_str,port);
	return 0;
}
```

 g++ -pthread socket.c -o socketTest 

转自：https://blog.csdn.net/cike44/article/details/52756900/

**字节流套接口 (如 tcp 套接口) 上的 read 和 write 函数所表现的行为不同于通常的文件 IO。字节流套接口上的读或写输入或输出的字节数可能比要求的数量少，但这不是错误状况，原因是内核中套接口的缓冲区可能已达到了极限。此时所需的是调用者再次调用 read 或 write 函数，以输入或输出剩余的字节。** 

可以使用 readn 函数来实现循环读取以解决这个问题：

```
ssize_t      /* Read "n" bytes from a descriptor. */
readn(int fd, void *vptr, size_t n)
{
    size_t nleft;
    ssize_t nread;
    char *ptr;
    
    ptr = vptr;
    nleft = n;
 
    while (nleft > 0) {
        if ( (nread = read(fd, ptr, nleft)) < 0) {
            if (errno == EINTR)
                nread = 0;  /* and call read() again */
            else
                return(-1);
        } else if (nread == 0)
   
        break;    /* EOF */
 
        nleft -= nread;
        ptr += nread;
    }
    
    return(n - nleft);  /* return >= 0 */
}
```

[socket 编程中 recv() 和 read() 的使用与区别_HW_Coder0501 的博客 - CSDN 博客_recv](https://blog.csdn.net/hhhlizhao/article/details/73912578 "socket编程中recv()和read()的使用与区别_HW_Coder0501的博客-CSDN博客_recv")

### Windows 下 C 语言的 Socket 编程例子

2017-12-21 13:32:23

**TCP 服务端通信的常规步骤：**

*   使用 socket() 创建 TCP 套接字（socket）
*   将创建的套接字绑定到一个本地地址和端口上（Bind）
*   将套接字设为监听模式，准备接收客户端请求（listen）
*   等待客户请求到来: 当请求到来后，接受连接请求，返回一个对应于此次连接的新的套接字（accept）
*   用 accept 返回的套接字和客户端进行通信（使用 write() / send() 或 send() / recv() ）返回，等待另一个客户请求关闭套接字

![](https://img-blog.csdn.net/20181010145823118?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1bnhpYW5naHVhbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**TCP 客户端通信的常规步骤：**

*   创建套接字（socket）
*   使用 connect() 建立到达服务器的连接（connect)
*   客户端进行通信（使用 write() / send() 或 send() / recv() ）
*   使用 close() 关闭客户连接

**一。  TCP**

server 端：

 C++ Code 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>60<br>61<br>62<br>63<br>64<br>65<br>66<br>67<br>68<br>69<br>70<br>71<br>72<br>73<br>74<br>75</p></td><td></td><td><p>#include&nbsp;"stdafx.h"<br>#include&nbsp;<br>#include&nbsp;<br>#pragma&nbsp;comment(lib,"ws2_32.lib")<br>int&nbsp;main(int&nbsp;argc,&nbsp;char*&nbsp;argv[])<br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;// 初始化 WSA<br>&nbsp;&nbsp;&nbsp;&nbsp;WORD&nbsp;sockVersion&nbsp;=&nbsp;MAKEWORD(2,2);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSADATA&nbsp;wsaData;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(WSAStartup(sockVersion,&nbsp;&amp;wsaData)!=0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;// 创建套接字<br>&nbsp;&nbsp;&nbsp;&nbsp;SOCKET&nbsp;slisten&nbsp;=<strong>&nbsp;</strong><strong>socket</strong>(AF_INET,&nbsp;SOCK_STREAM,&nbsp;IPPROTO_TCP);<br>&nbsp;&nbsp;&nbsp;&nbsp;if(slisten&nbsp;==&nbsp;INVALID_SOCKET)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("socket&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;// 绑定 IP 和端口<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;sin;<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_family&nbsp;=&nbsp;AF_INET;<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_port&nbsp;=&nbsp;htons(8888);<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_addr.S_un.S_addr&nbsp;=&nbsp;INADDR_ANY;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(<strong>bind</strong>(slisten,&nbsp;(LPSOCKADDR)&amp;sin,&nbsp;sizeof(sin))&nbsp;==&nbsp;SOCKET_ERROR)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("bind&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;// 开始监听<br>&nbsp;&nbsp;&nbsp;&nbsp;if(<strong>listen</strong>(slisten,&nbsp;5)&nbsp;==&nbsp;SOCKET_ERROR)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("listen&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;// 循环接收数据<br>&nbsp;&nbsp;&nbsp;&nbsp;SOCKET&nbsp;sClient;<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;remoteAddr;<br>&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;nAddrlen&nbsp;=&nbsp;sizeof(remoteAddr);<br>&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;revData[255];&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(true)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("等待连接...\n");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sClient&nbsp;=&nbsp;<strong>accept</strong>(slisten,&nbsp;(SOCKADDR&nbsp;*)&amp;remoteAddr,&nbsp;&amp;nAddrlen);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if(sClient&nbsp;==&nbsp;INVALID_SOCKET)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("accept&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("接受到一个连接：%s&nbsp;\r\n",&nbsp;inet_ntoa(remoteAddr.sin_addr));<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 接收数据<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;ret&nbsp;=&nbsp;recv(sClient,&nbsp;revData,&nbsp;255,&nbsp;0);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if(ret&gt;&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;revData[ret]&nbsp;=&nbsp;0x00;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf(revData);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 发送数据<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*&nbsp;sendData&nbsp;=&nbsp;"你好，TCP 客户端！\n";<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;send(sClient,&nbsp;sendData,&nbsp;strlen(sendData),&nbsp;0);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;closesocket(sClient);<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;closesocket(slisten);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSACleanup();<br>&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>}</p></td></tr></tbody></table>

client 端：

 C++ Code 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47</p></td><td></td><td><p>#include &lt;winsock2.h&gt;<br>#include &lt;ws2tcpip.h&gt;<br>#include &lt;stdio.h&gt;<br>#include &lt;windows.h&gt;<br>#pragma&nbsp;&nbsp;comment(lib,"ws2_32.lib")<br>int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;*argv[])<br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;WORD&nbsp;sockVersion&nbsp;=&nbsp;MAKEWORD(2,&nbsp;2);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSADATA&nbsp;data;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(WSAStartup(sockVersion,&nbsp;&amp;data)&nbsp;!=&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;SOCKET&nbsp;sclient&nbsp;=&nbsp;<strong>socket</strong>(AF_INET,&nbsp;SOCK_STREAM,&nbsp;IPPROTO_TCP);<br>&nbsp;&nbsp;&nbsp;&nbsp;if(sclient&nbsp;==&nbsp;INVALID_SOCKET)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("invalid&nbsp;socket&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;serAddr;<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_family&nbsp;=&nbsp;AF_INET;<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_port&nbsp;=&nbsp;htons(8888);<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_addr.S_un.S_addr&nbsp;=&nbsp;inet_addr("127.0.0.1");<br>&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(<strong>connect</strong>(sclient,&nbsp;(sockaddr&nbsp;*)&amp;serAddr,&nbsp;sizeof(serAddr))&nbsp;==&nbsp;SOCKET_ERROR)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("connect&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;closesocket(sclient);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*sendData&nbsp;=&nbsp;"你好，TCP 服务端，我是客户端!\n";<br>&nbsp;&nbsp;&nbsp;&nbsp;send(sclient,&nbsp;sendData,&nbsp;strlen(sendData),&nbsp;0);<br>&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;recData[255];<br>&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;ret&nbsp;=&nbsp;recv(sclient,&nbsp;recData,&nbsp;255,&nbsp;0);<br>&nbsp;&nbsp;&nbsp;&nbsp;if(ret&gt;&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;recData[ret]&nbsp;=&nbsp;0x00;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf(recData);<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;closesocket(sclient);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSACleanup();<br>&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>}</p></td></tr></tbody></table>

**二. UDP**

SERVER 端

 C++ Code 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54</p></td><td></td><td><p>#include&nbsp;"stdafx.h"<br>#include&nbsp;<br>#include&nbsp;<br>#pragma&nbsp;comment(lib,&nbsp;"ws2_32.lib")&nbsp;<br>int&nbsp;main(int&nbsp;argc,&nbsp;char*&nbsp;argv[])<br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;WSADATA&nbsp;wsaData;<br>&nbsp;&nbsp;&nbsp;&nbsp;WORD&nbsp;sockVersion&nbsp;=&nbsp;MAKEWORD(2,2);<br>&nbsp;&nbsp;&nbsp;&nbsp;if(WSAStartup(sockVersion,&nbsp;&amp;wsaData)&nbsp;!=&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;SOCKET&nbsp;serSocket&nbsp;=&nbsp;socket(AF_INET,&nbsp;SOCK_DGRAM,&nbsp;IPPROTO_UDP);&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(serSocket&nbsp;==&nbsp;INVALID_SOCKET)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("socket&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;serAddr;<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_family&nbsp;=&nbsp;AF_INET;<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_port&nbsp;=&nbsp;htons(8888);<br>&nbsp;&nbsp;&nbsp;&nbsp;serAddr.sin_addr.S_un.S_addr&nbsp;=&nbsp;INADDR_ANY;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(bind(serSocket,&nbsp;(sockaddr&nbsp;*)&amp;serAddr,&nbsp;sizeof(serAddr))&nbsp;==&nbsp;SOCKET_ERROR)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("bind&nbsp;error&nbsp;!");<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;closesocket(serSocket);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;remoteAddr;<br>&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;nAddrLen&nbsp;=&nbsp;sizeof(remoteAddr);&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(true)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;recvData[255];&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;ret&nbsp;=&nbsp;recvfrom(serSocket,&nbsp;recvData,&nbsp;255,&nbsp;0,&nbsp;(sockaddr&nbsp;*)&amp;remoteAddr,&nbsp;&amp;nAddrLen);<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ret&gt;&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;recvData[ret]&nbsp;=&nbsp;0x00;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("接受到一个连接：%s&nbsp;\r\n",&nbsp;inet_ntoa(remoteAddr.sin_addr));<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf(recvData);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*&nbsp;sendData&nbsp;=&nbsp;"一个来自服务端的 UDP 数据包 \ n";<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sendto(serSocket,&nbsp;sendData,&nbsp;strlen(sendData),&nbsp;0,&nbsp;(sockaddr&nbsp;*)&amp;remoteAddr,&nbsp;nAddrLen);&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;closesocket(serSocket);&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;WSACleanup();<br>&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>}</p></td></tr></tbody></table>

 C++ Code 

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td><p>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37</p></td><td></td><td><p>#include&nbsp;"stdafx.h"<br>#include&nbsp;<br>#include&nbsp;<br>#pragma&nbsp;comment(lib,&nbsp;"ws2_32.lib")&nbsp;<br>int&nbsp;main(int&nbsp;argc,&nbsp;char*&nbsp;argv[])<br>{<br>&nbsp;&nbsp;&nbsp;&nbsp;WORD&nbsp;socketVersion&nbsp;=&nbsp;MAKEWORD(2,2);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSADATA&nbsp;wsaData;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;if(WSAStartup(socketVersion,&nbsp;&amp;wsaData)&nbsp;!=&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;SOCKET&nbsp;sclient&nbsp;=&nbsp;socket(AF_INET,&nbsp;SOCK_DGRAM,&nbsp;IPPROTO_UDP);<br>&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;sockaddr_in&nbsp;sin;<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_family&nbsp;=&nbsp;AF_INET;<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_port&nbsp;=&nbsp;htons(8888);<br>&nbsp;&nbsp;&nbsp;&nbsp;sin.sin_addr.S_un.S_addr&nbsp;=&nbsp;inet_addr("127.0.0.1");<br>&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;len&nbsp;=&nbsp;sizeof(sin);<br>&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*&nbsp;sendData&nbsp;=&nbsp;"来自客户端的数据包.\n";<br>&nbsp;&nbsp;&nbsp;&nbsp;sendto(sclient,&nbsp;sendData,&nbsp;strlen(sendData),&nbsp;0,&nbsp;(sockaddr&nbsp;*)&amp;sin,&nbsp;len);<br>&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;recvData[255];&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;ret&nbsp;=&nbsp;recvfrom(sclient,&nbsp;recvData,&nbsp;255,&nbsp;0,&nbsp;(sockaddr&nbsp;*)&amp;sin,&nbsp;&amp;len);<br>&nbsp;&nbsp;&nbsp;&nbsp;if(ret&gt;&nbsp;0)<br>&nbsp;&nbsp;&nbsp;&nbsp;{<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;recvData[ret]&nbsp;=&nbsp;0x00;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf(recvData);<br>&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;closesocket(sclient);<br>&nbsp;&nbsp;&nbsp;&nbsp;WSACleanup();<br>&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;<br>}</p></td></tr></tbody></table>

本文来至：[http://blog.csdn.net/ssun125/article/details/8525823](http://blog.csdn.net/ssun125/article/details/8525823 "http://blog.csdn.net/ssun125/article/details/8525823")

Socket send 函数和 recv 函数详解
-------------------------

2018-01-23 02:03:23

**1.send 函数**

**i****nt send(SOCKET s, const char FAR *buf, int len, int flags);**  

    不论是客户还是服务器应用程序都用 send 函数来向 TCP 连接的另一端发送数据。客户程序一般用 send 函数向服务器发送请求，而服务器则通常用 send 函数来向客户程序发送应答。

    该函数的第一个参数指定发送端套接字描述符；

    第二个参数指明一个存放应用程序要发送数据的缓冲区；

    第三个参数指明实际要发送的数据的字节数；

    第四个参数一般置 0。 

    这里只描述**同步 Socket 的 send 函数的执行流程**。当调用该函数时，

   （1）send 先比较待发送数据的长度 len 和套接字 s 的发送缓冲的长度， 如果 len 大于 s 的发送缓冲区的长度，该函数返回 SOCKET_ERROR；

   （2）如果 len 小于或者等于 s 的发送缓冲区的长度，那么 send 先检查协议是否正在发送 s 的发送缓冲中的数据，如果是就等待协议把数据发送完，如果协议还没有开始发送 s 的发送缓冲中的数据或者 s 的发送缓冲中没有数据，那么 send 就比较 s 的发送缓冲区的剩余空间和 len

   （3）如果 len 大于剩余空间大小，send 就一直等待协议把 s 的发送缓冲中的数据发送完

   （4）如果 len 小于剩余 空间大小，send 就仅仅把 buf 中的数据 copy 到剩余空间里（**注意并不是 send 把 s 的发送缓冲中的数据传到连接的另一端的，而是协议传的，send 仅仅是把 buf 中的数据 copy 到 s 的发送缓冲区的剩余空间里**）。

   如果 send 函数 copy 数据成功，就返回实际 copy 的字节数，如果 send 在 copy 数据时出现错误，那么 send 就返回 SOCKET_ERROR；如果 send 在等待协议传送数据时网络断开的话，那么 send 函数也返回 SOCKET_ERROR。

 **要注意 send 函数把 buf 中的数据成功 copy 到 s 的发送缓冲的剩余空间里后它就返回了，但是此时这些数据并不一定马上被传到连接的另一端**。如果协议在后续的传送过程中出现网络错误的话，那么下一个 Socket 函数就会返回 SOCKET_ERROR。（每一个除 send 外的 Socket 函数在执 行的最开始总要先等待套接字的发送缓冲中的数据被协议传送完毕才能继续，如果在等待时出现网络错误，那么该 Socket 函数就返回 SOCKET_ERROR）

注意：在 Unix 系统下，如果 send 在等待协议传送数据时网络断开的话，调用 send 的进程会接收到一个 SIGPIPE 信号，进程对该信号的默认处理是进程终止。

通过测试发现，异步 socket 的 send 函数在网络刚刚断开时还能发送返回相应的字节数，同时使用 select 检测也是可写的，但是过几秒钟之后，再 send 就会出错了，返回 - 1。select 也不能检测出可写了。

**2. recv 函数**

int recv(SOCKET s, char FAR *buf, int len, int flags);   

    不论是客户还是服务器应用程序都用 recv 函数从 TCP 连接的另一端接收数据。该函数的第一个参数指定接收端套接字描述符；

    第二个参数指明一个缓冲区，该缓冲区用来存放 recv 函数接收到的数据；

    第三个参数指明 buf 的长度；

    第四个参数一般置 0。

    这里只描述**同步 Socket** 的 recv 函数的执行流程。当应用程序调用 recv 函数时，

    （1）recv 先等待 s 的发送缓冲中的数据被协议传送完毕，如果协议在传送 s 的发送缓冲中的数据时出现网络错误，那么 recv 函数返回 SOCKET_ERROR，

    （2）如果 s 的发送缓冲中没有数据或者数据被协议成功发送完毕后，recv 先检查套接字 s 的接收缓冲区，如果 s 接收缓冲区中没有数据或者协议正在接收数据，那么 recv 就一直等待，直到协议把数据接收完毕。当协议把数据接收完毕，recv 函数就把 s 的接收缓冲中的数据 copy 到 buf 中（**注意协议接收到的数据可能大于 buf 的长度，所以 在这种情况下要调用几次 recv 函数才能把 s 的接收缓冲中的数据 copy 完。recv 函数仅仅是 copy 数据，真正的接收数据是协议来完成的**），

    recv 函数返回其实际 copy 的字节数。如果 recv 在 copy 时出错，那么它返回 SOCKET_ERROR；如果 recv 函数在等待协议接收数据时网络中断了，那么它返回 0。

注意：在 Unix 系统下，如果 recv 函数在等待协议接收数据时网络断开了，那么调用 recv 的进程会接收到一个 SIGPIPE 信号，进程对该信号的默认处理是进程终止。

【Socket】有很多 BUG，测试人员要注意。
------------------------

最近使用 Scoket 来测试链路和数据传输，发现 Scoket 有很多的 bug，第一：十六进制发送的时候，数据中间的空格和你再输入框内的数据间加了空格然后再删掉，然后把数据发送出去，接收到的数据长度和数据竟然前后不一样。

第二、Scoket 接收数据 16 进制显示总是在前面或者后面加一些额外的东西，比如在前面加 0A（回车符？) 时有时无。

编程知识
----

### 网络字节序与主机字节序

主机字节序就是我们平常说的大端和小端模式：不同的 CPU 有不同的字节序类型，这些字节序是指整数在内存中保存的顺序，这个叫做主机序。引用标准的 Big-Endian 和 Little-Endian 的定义如下：

　　a) Little-Endian 就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。

　　b) Big-Endian 就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

网络字节序：4 个字节的 32 bit 值以下面的次序传输：首先是 0～7bit，其次 8～15bit，然后 16～23bit，最后是 24~31bit。这种传输次序称作大端字节序。由于 TCP/IP 首部中所有的二进制整数在网络中传输时都要求以这种次序，因此它又称作网络字节序。字节序，顾名思义字节的顺序，就是大于一个字节类型的数据在内存中的存放顺序，一个字节的数据没有顺序的问题了。

所以：在将一个地址绑定到 socket 的时候，请先将主机字节序转换成为网络字节序，而不要假定主机字节序跟网络字节序一样使用的是 Big-Endian。由于这个问题曾引发过血案！公司项目代码中由于存在这个问题，导致了很多莫名其妙的问题，所以请谨记对主机字节序不要做任何假定，务必将其转化为网络字节序再赋给 socket。

### 4、socket 中 TCP 的三次握手建立连接详解

我们知道 tcp 建立连接要进行 “三次握手”，即交换三个分组。大致流程如下：

*   客户端向服务器发送一个 SYN J
*   服务器向客户端响应一个 SYN K，并对 SYN J 进行确认 ACK J+1
*   客户端再想服务器发一个确认 ACK K+1

只有就完了三次握手，但是这个三次握手发生在 socket 的那几个函数中呢？请看下图：

[​编辑](http://images.cnblogs.com/cnblogs_com/skynet/201012/201012122157467258.png "​编辑")

图 1、socket 中发送的 TCP 三次握手

从图中可以看出，当客户端调用 connect 时，触发了连接请求，向服务器发送了 SYN J 包，这时 connect 进入阻塞状态；服务器监听到连接请求，即收到 SYN J 包，调用 accept 函数接收请求向客户端发送 SYN K ，ACK J+1，这时 accept 进入阻塞状态；客户端收到服务器的 SYN K ，ACK J+1 之后，这时 connect 返回，并对 SYN K 进行确认；服务器收到 ACK K+1 时，accept 返回，至此三次握手完毕，连接建立。

> 总结：客户端的 connect 在三次握手的第二个次返回，而服务器端的 accept 在三次握手的第三次返回。

### 5、socket 中 TCP 的四次握手释放连接详解

上面介绍了 socket 中 TCP 的三次握手建立过程，及其涉及的 socket 函数。现在我们介绍 socket 中的四次握手释放连接的过程，请看下图：

[​编辑](http://images.cnblogs.com/cnblogs_com/skynet/201012/201012122157487616.png "​编辑")

图 2、socket 中发送的 TCP 四次握手

图示过程如下：

*   某个应用进程首先调用 close 主动关闭连接，这时 TCP 发送一个 FIN M；
    
*   另一端接收到 FIN M 之后，执行被动关闭，对这个 FIN 进行确认。它的接收也作为文件结束符传递给应用进程，因为 FIN 的接收意味着应用进程在相应的连接上再也接收不到额外数据；
    
*   一段时间之后，接收到文件结束符的应用进程调用 close 关闭它的 socket。这导致它的 TCP 也发送一个 FIN N；
    
*   接收到这个 FIN 的源发送端 TCP 对它进行确认。
    

这样每个方向上都有一个 FIN 和 ACK。

###  Socket 在 Linux 中的表示

带有 ID 的文件

*   0：标准输入文件，对应键盘
*   1：标准输出文件，对应显示器

一个文件描述符只是一个和打开的文件相关联的整数，背后代表的意思可能如下：

*   普通文件
*   FIFO
*   管道
*   终端
*   键盘
*   显示器
*   一个网络连接

socket() 的返回值就是文件描述符

*   read(): 读取远程计算机传来的数据
*   write(): 向远程计算机写入数据

### Socket 套接字类型

**流格式套接字（SOCK_STREAM）**

特征

*   数据在传输的过程中不会消失 (只要 SOCK_STREAM 不消失，数据就不会消失)
*   数据按顺序传输
*   数据的发送和接收不同步

流格式套接字内部有一个缓冲区，通过 socket 传输的数据将保存在这个缓冲区中。接收端手法哦数据后并不是立即读取，只要数据不超过缓冲区容量，接收端可以在缓冲区被填满之后一次性读取，也可以分多次读取。（接收端可以控制数据的读取）

**数据报套接字 (SOCK_DGRAM)**

无连接的套接字，用 SOCK_DGRAM 表示

只管传输数据，不做数据校验。

优缺点：传输效率快但是不能保证数据的有效性 (数据会丢失)

特点：

*   强调快速传输而非传输顺序
*   传输的数据可能丢失也可能损毁
*   限制每次传输的数据大小
*   数据的发送和接收是同步的 (也叫：存在数据边界)

QQ 视频聊天和语音聊天则使用 SOCK_DRRAM 来传输数据，首先先保证通信效率，降低延迟，视频和音频即使丢失一小部分数据，也不会对最终数据在终端的显示造成什么大的影响。

#### 计算机网络相关知识点

*   IP 地址
*   MAC 地址
*   端口号：一台计算机可以同时提供多种网络服务 (Web 服务，FTP 服务，SMTP 服务), 为区分不同的网络程序，计算机会为每个网络程序分配一个独一无二的端口号，不同计算机上的使用同一个端口通信的计算机可以通过这个端口号进行数据通信

#### Socket 缓冲区

*   输入缓冲区
*   输出缓冲区

write()/send() 先将数据写入到缓冲区内部，再根据 TCP 协议将缓冲区中的数据发送到目标机器，一旦将数据写入缓冲区，函数就返回，不管后面的机器发送数据的过程。

read()/recv() 函数从缓冲区读取数据，而不是从网络中读取数据。

缓冲区特性：

*   I/O 缓冲区再每个套接字中单独存在
*   I/O 缓冲区在创建套接字时自动生成
*   即使关闭套接字也会继续传送输出缓冲区中遗留的数据
*   关闭套接字将丢失输入缓冲区中的数据

获取输入输出黄忠的大小：getsockopt()

```
unsigned optVal;
int optLen = sizeof(int);
getsockopt(servSock, SOL_SOCKET, SO_SNDBUF, (char*)&optVal, &optLen);
printf("Buffer length: %d\n", optVal);
```

#### 阻塞 IO

使用 write()/send() 发送数据时

1.  先检查缓冲区，如果缓冲区的可用空间长度小于要发送的数据，则 write()/send() 会被阻塞，直到缓冲区中的数据被成功发送，缓冲区为空，才唤醒 write()/send() 继续写入数据
2.  如果程序在 TCP 协议下正在向网络发送数据，那么输出缓冲区会被锁定，不允许写入，write()/send() 函数会被阻塞，直到数据发送完毕，才对缓冲区解锁，才将 write()/send() 唤醒
3.  如果要写入的数据大于缓冲区的最大长度，则将分批写入
4.  直到所有的数据被写入缓冲区 write()/send() 才能返回

当使用 read()/recv() 读取数据时：

1.  先检查缓冲区，如果缓冲区中有数据，则读取；否则函数被阻塞，直到网络上有数据来
2.  如果要读取的数据长度小于缓冲区中的数据长度，则不能一次性将缓冲区中的所有数据读出，剩余数据将不断积压，直到有 read()/recv() 函数再次读取。
3.  直到读取到数据后，read()/recv() 函数才会返回，否则一直被阻塞

#### 什么是 TCP 的粘包问题

对于 Socket 方式的数据发送和接收方式而言，数据的接收和数据发送是无关的，不管数据通过 write()/send() 发送了多少次，都会尽可能多的发送数据，根据上面的 Socket 中的内容可以看出，在服务端，只要你缓冲区为空，我就唤醒 write()/send() 进行数据的发送；而在服务端我则是等待 read()/recv() 可用的情况才对缓冲区中的数据进行读取，如此便会出现服务端和客户端数据发送 / 接收数据速率不同步的问题。因此可能会出现数据的 ** 粘包问题。** 举例如下：例如客户端向服务器第一次发送 1，第二次发送 3，服务器可能当成 13 来处理。

可以在上面的章节 "Windows 系统 Socket 通信程序 Demo" 让服务器线程在接收客户端数据前等待足够长的一段时间，比如: Sleep(10000);。可以很容易观测到客户端程序多次发送的数据在服务器端形成了 "粘包问题"。

摘自：[你不懂 TCP/IP 编程，面试官连面试你的机会都不给你！ - 知乎](https://zhuanlan.zhihu.com/p/161089052 "你不懂TCP/IP编程，面试官连面试你的机会都不给你！ - 知乎")

### 6、一个例子（实践一下）

说了这么多了，动手实践一下。下面编写一个简单的服务器、客户端（使用 TCP）——服务器端一直监听本机的 6666 号端口，如果收到连接请求，将接收请求并接收客户端发来的消息；客户端与服务器端建立连接并发送一条消息。

服务器端代码：

```
#include<stdio.h> 
#include<stdlib.h> 
#include<string.h> 
#include<errno.h> 
#include<sys/types.h> 
#include<sys/socket.h> 
#include<netinet/in.h> 
#define MAXLINE 4096 
int main(int argc, char** argv) 
{ 
    int listenfd, connfd; 
    struct sockaddr_in servaddr; 
    char buff[4096]; int n; 
    if( (listenfd = socket(AF_INET, SOCK_STREAM, 0)) == -1 )
    { 
        printf("create socket error: %s(errno: %d)\n",strerror(errno),errno);
        exit(0); 
    } 
    memset(&servaddr, 0, sizeof(servaddr)); 
    servaddr.sin_family = AF_INET; 
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);             
    servaddr.sin_port = htons(6666); 
    if( bind(listenfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    { 
        printf("bind socket error: %s(errno: %d)\n",strerror(errno),errno);
        exit(0); 
    } 
    if( listen(listenfd, 10) == -1)
    { 
        printf("listen socket error: %s(errno: %d)\n",strerror(errno),errno);
        exit(0); 
    } 
    printf("======waiting for client's request======\n"); 
    while(1)
    { 
        if( (connfd = accept(listenfd, (struct sockaddr*)NULL, NULL)) == -1)
        { 
            printf("accept socket error: %s(errno: %d)",strerror(errno),errno); 
            continue; 
        } 
        n= recv(connfd, buff, MAXLINE, 0); 
        buff[n] = '\0'; 
        printf("recv msg from client: %s\n", buff); 
        close(connfd); 
    } 
    close(listenfd); 
}
```

客户端代码：

```
#include<stdio.h> 
#include<stdlib.h> 
#include<string.h> 
#include<errno.h> 
#include<sys/types.h> 
#include<sys/socket.h> 
#include<netinet/in.h> 
#define MAXLINE 4096 
int main(int argc, char** argv) 
{ 
    int sockfd, n; 
    char recvline[4096], sendline[4096]; 
    struct sockaddr_in servaddr; 
    if( argc != 2)
    { 
        printf("usage: ./client <ipaddress>\n"); 
        exit(0); 
    } 
    if( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
    { 
        printf("create socket error: %s(errno: %d)\n", strerror(errno),errno);
        exit(0); 
    }
    memset(&servaddr, 0, sizeof(servaddr)); 
    servaddr.sin_family = AF_INET; 
    servaddr.sin_port =htons(6666); 
    if( inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
    { 
        printf("inet_pton error for %s\n",argv[1]); 
        exit(0); 
    } 
    if( connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0)
    { 
        printf("connect error: %s(errno: %d)\n",strerror(errno),errno); 
        exit(0); 
    } 
    printf("send msg to server: \n"); fgets(sendline, 4096, stdin); 
    if( send(sockfd, sendline, strlen(sendline), 0) < 0) 
    { 
        printf("send msg error: %s(errno: %d)\n", strerror(errno), errno);
     exit(0); 
    } 
    close(sockfd); 
    exit(0); 
}
```

当然上面的代码很简单，也有很多缺点，这就只是简单的演示 socket 的基本函数使用。其实不管有多复杂的网络程序，都使用的这些基本函数。上面的服务器使用的是迭代模式的，即只有处理完一个客户端请求才会去处理下一个客户端的请求，这样的服务器处理能力是很弱的，现实中的服务器都需要有并发处理能力！为了需要并发处理，服务器需要 fork() 一个新的进程或者线程去处理请求等。

### 7、动动手

留下一个问题，欢迎大家回帖回答！！！是否熟悉 Linux 下网络编程？如熟悉，编写如下程序完成如下功能：

服务器端：

接收地址 192.168.100.2 的客户端信息，如信息为 “Client Query”，则打印 “Receive Query”

客户端：

向地址 192.168.100.168 的服务器端顺序发送信息 “Client Query test”，“Cleint Query”，“Client Query Quit”，然后退出。

题目中出现的 ip 地址可以根据实际情况定。

1、运行 SQLPLUS 工具

　　C:\Users\wd-pc>sqlplus

2、直接进入 SQLPLUS 命令提示符

　　C:\Users\wd-pc>sqlplus /nolog

3、以 OS 身份连接 

　　C:\Users\wd-pc>sqlplus / as sysdba   或

　　SQL>connect / as sysdba

4、普通用户登录

　　C:\Users\wd-pc>sqlplus scott/123456 　或

　　SQL>connect scott/123456  或

　　SQL>connect scott/123456@servername

5、以管理员登录

　　C:\Users\wd-pc>sqlplus sys/123456 as sysdba　或

　　SQL>connect sys/123456 as sysdba

6、切换用户

　　SQL>conn hr/123456 

　　注：conn 同 connect

 7、退出

　　exit

原文;[socket 编程——一个简单的例子_find12 的博客 - CSDN 博客_socket 编程](https://blog.csdn.net/qq_31918961/article/details/80546537 "socket编程——一个简单的例子_find12的博客-CSDN博客_socket编程")

接口函数 (API)
----------

### 3.1、socket() 函数

```
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};
 
/* Internet address. */
struct in_addr {
    uint32_t       s_addr;     /* address in network byte order */
};
```

socket 函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而 **socket()** 用于创建一个 socket 描述符（socket descriptor），它唯一标识一个 socket。这个 socket 描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

正如可以给 fopen 的传入不同参数值，以打开不同的文件。创建 socket 的时候，也可以指定不同的参数创建不同的 socket 描述符，socket 函数的三个参数分别为：

*   domain：即协议域，又称为协议族（family）。常用的协议族有，**AF_INET、AF_INET6、AF_LOCAL（或称 AF_UNIX，Unix 域 socket）、AF_ROUTE** 等等。协议族决定了 socket 的地址类型，在通信中必须采用对应的地址，如 AF_INET 决定了要用 ipv4 地址（32 位的）与端口号（16 位的）的组合、AF_UNIX 决定了要用一个绝对路径名作为地址。
*   type：指定 socket 类型。常用的 socket 类型有，**SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET** 等等（socket 的类型有哪些？）。
*   protocol：故名思意，就是指定协议。常用的协议有，**IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC** 等，它们分别对应 TCP 传输协议、UDP 传输协议、STCP 传输协议、TIPC 传输协议（这个协议我将会单独开篇讨论！）。

注意：并不是上面的 type 和 protocol 可以随意组合的，如 SOCK_STREAM 不可以跟 IPPROTO_UDP 组合。当 protocol 为 0 时，会自动选择 type 类型对应的默认协议。

当我们调用 **socket** 创建一个 socket 时，返回的 socket 描述字它存在于协议族（address family，AF_XXX）空间中，但没有一个具体的地址。如果想要给它赋值一个地址，就必须调用 bind() 函数，否则就当调用 **connect()、listen()** 时系统会自动随机分配一个端口。

### 3.2、bind() 函数

正如上面所说 bind() 函数把一个地址族中的特定地址赋给 socket。例如对应 **AF_INET、AF_INET6** 就是把一个 ipv4 或 ipv6 地址和端口号组合赋给 socket。

```
struct sockaddr_in6 { 
    sa_family_t     sin6_family;   /* AF_INET6 */ 
    in_port_t       sin6_port;     /* port number */ 
    uint32_t        sin6_flowinfo; /* IPv6 flow information */ 
    struct in6_addr sin6_addr;     /* IPv6 address */ 
    uint32_t        sin6_scope_id; /* Scope ID (new in 2.4) */ 
};
 
struct in6_addr { 
    unsigned char   s6_addr[16];   /* IPv6 address */ 
};
```

函数的三个参数分别为：

*   sockfd：即 socket 描述字，它是通过 socket() 函数创建了，唯一标识一个 socket。bind() 函数就是将给这个描述字绑定一个名字。
*   addr：一个 const struct sockaddr * 指针，指向要绑定给 sockfd 的协议地址。这个地址结构根据地址创建 socket 时的地址协议族的不同而不同，如 ipv4 对应的是： 
    
    ```
    #define UNIX_PATH_MAX    108
     
    struct sockaddr_un { 
        sa_family_t sun_family;               /* AF_UNIX */ 
        char        sun_path[UNIX_PATH_MAX];  /* pathname */ 
    };
    addrlen：对应的是地址的长度。
    ```
    
    ipv6 对应的是： 
    
    ```
    int listen(int sockfd, int backlog);
    int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    ```
    
    Unix 域对应的是： 
    
    ```
    #include <unistd.h>
     
           ssize_t read(int fd, void *buf, size_t count);
           ssize_t write(int fd, const void *buf, size_t count);
     
           #include <sys/types.h>
           #include <sys/socket.h>
     
           ssize_t send(int sockfd, const void *buf, size_t len, int flags);
           ssize_t recv(int sockfd, void *buf, size_t len, int flags);
     
           ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                          const struct sockaddr *dest_addr, socklen_t addrlen);
           ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                            struct sockaddr *src_addr, socklen_t *addrlen);
     
           ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
           ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
    ```
    

通常服务器在启动的时候都会绑定一个众所周知的地址（如 ip 地址 + 端口号），用于提供服务，客户就可以通过它来接连服务器；而客户端就不用指定，有系统自动分配一个端口号和自身的 ip 地址组合。这就是为什么通常服务器端在 listen 之前会调用 bind()，而客户端就不会调用，而是在 **connect**() 时由系统随机生成一个。

### 3.3、listen()、connect() 函数

如果作为一个服务器，在调用 socket()、bind() 之后就会调用 listen() 来监听这个 socket，如果客户端这时调用 connect() 发出连接请求，服务器端就会接收到这个请求。

```
#include <unistd.h>
int close(int fd);
```

listen 函数的第一个参数即为要监听的 socket 描述字，第二个参数为相应 socket 可以排队的最大连接个数。socket() 函数创建的 socket 默认是一个主动类型的，listen 函数将 socket 变为被动类型的，等待客户的连接请求。

connect 函数的第一个参数即为客户端的 socket 描述字，第二参数为服务器的 socket 地址，第三个参数为 socket 地址的长度。客户端通过调用 connect 函数来建立与 TCP 服务器的连接。

### 3.4、accept() 函数

TCP 服务器端依次调用 **socket()、bind()、listen()** 之后，就会监听指定的 socket 地址了。TCP 客户端依次调用 socket()、connect() 之后就想 TCP 服务器发送了一个连接请求。TCP 服务器监听到这个请求之后，就会调用 **accept()** 函数取接收请求，这样连接就建立好了。之后就可以开始网络 I/O 操作了，即类同于普通文件的读写 I/O 操作。

```
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

accept 函数的第一个参数为服务器的 socket 描述字，第二个参数为指向 struct sockaddr * 的指针，用于返回客户端的协议地址，第三个参数为协议地址的长度。如果 accpet 成功，那么其返回值是由内核自动生成的一个全新的描述字，代表与返回客户的 TCP 连接。

注意：accept 的第一个参数为服务器的 socket 描述字，是服务器开始调用 socket() 函数生成的，称为监听 socket 描述字；而 accept 函数返回的是已连接的 socket 描述字。一个服务器通常通常仅仅只创建一个监听 socket 描述字，它在该服务器的生命周期内一直存在。内核为每个由服务器进程接受的客户连接创建了一个已连接 socket 描述字，当服务器完成了对某个客户的服务，相应的已连接 socket 描述字就被关闭。

### 3.5、read()、write() 等函数

万事具备只欠东风，至此服务器与客户已经建立好连接了。可以调用网络 I/O 进行读写操作了，即实现了网咯中不同进程之间的通信！网络 I/O 操作有下面几组：

*   **read()/write()**
*   **recv()/send()**
*   **readv()/writev()**
*   **recvmsg()/sendmsg()**
*   **recvfrom()/sendto()**

我推荐使用 recvmsg()/sendmsg() 函数，这两个函数是最通用的 I/O 函数，实际上可以把上面的其它函数都替换成这两个函数。它们的声明如下：

```
#include <unistd.h>
 
       ssize_t read(int fd, void *buf, size_t count);
       ssize_t write(int fd, const void *buf, size_t count);
 
       #include <sys/types.h>
       #include <sys/socket.h>
 
       ssize_t send(int sockfd, const void *buf, size_t len, int flags);
       ssize_t recv(int sockfd, void *buf, size_t len, int flags);
 
       ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                      const struct sockaddr *dest_addr, socklen_t addrlen);
       ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                        struct sockaddr *src_addr, socklen_t *addrlen);
 
       ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
       ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

read 函数是负责从 fd 中读取内容. 当读成功时，read 返回实际所读的字节数，如果返回的值是 0 表示已经读到文件的结束了，小于 0 表示出现了错误。如果错误为 EINTR 说明读是由中断引起的，如果是 ECONNREST 表示网络连接出了问题。

write 函数将 buf 中的 nbytes 字节内容写入文件描述符 fd. 成功时返回写的字节数。失败时返回 - 1，并设置 errno 变量。 在网络程序中，当我们向套接字文件描述符写时有俩种可能。1)write 的返回值大于 0，表示写了部分或者是全部的数据。2) 返回的值小于 0，此时出现了错误。我们要根据错误类型来处理。如果错误为 EINTR 表示在写的时候出现了中断错误。如果为 EPIPE 表示网络连接出现了问题 (对方已经关闭了连接)。

其它的我就不一一介绍这几对 I/O 函数了，具体参见 man 文档或者 baidu、Google，下面的例子中将使用到 send/recv。

### 3.6、close() 函数

在服务器与客户端建立连接之后，会进行一些读写操作，完成了读写操作就要关闭相应的 socket 描述字，好比操作完打开的文件要调用 fclose 关闭打开的文件。

```
#include <unistd.h>
int close(int fd);
```

close 一个 TCP socket 的缺省行为时把该 socket 标记为以关闭，然后立即返回到调用进程。该描述字不能再由调用进程使用，也就是说不能再作为 read 或 write 的第一个参数。

注意：close 操作只是使相应 socket 描述字的引用计数 - 1，只有当引用计数为 0 的时候，才会触发 TCP 客户端向服务器发送终止连接请求。

TCP 和 Socket 关系 
----------------

**socket 跟 TCP/IP 并没有必然的联系**。Socket 编程接口在设计的时候，就希望也能适应其他的网络协议。所以，**socket 的出现只是可以更方便的使用 TCP/IP 协议栈而已，其对 TCP/IP 进行了抽象，形成了几个最基本的函数接口。**比如 create，listen，accept，connect，read 和 write 等等。

 [http://t.csdn.cn/eKY4z](http://t.csdn.cn/eKY4z "http://t.csdn.cn/eKY4z")