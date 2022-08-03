title: APUE笔记-六
date: 2022-08-03 20:54:10
tags: [unix, linux, 读书笔记, APUE]
---

# 第十六章 网络IPC：套接字

UNIX系统所提供的的经典进程间通信机制（IPC）：管道、FIFO、消息队列、信号量以及共享存储。这些机制允许在同一台计算机上运行的进程可以相互通信。网络进程间通信（network IPC）可以让在不同计算机（通过网络相连）上的进程相互通信

套接字网络进程间通信接口，进程用该接口能够和其他进程通信，无论它们是在同一台计算机上还是在不同的计算机上。实际上，这正是套接字接口的设计目标之一：同样的接口既可以用于计算机间通信，也可以用于计算机内通信。

## 套接字描述符

套接字是通信端点的抽象。正如使用文件描述符访问文件，应用程序使用套接字描述符访问套接字。套接字描述符在UNIX系统中被当作是一种文件描述符。事实上，许多处理文件描述符的函数（如read和write）可以用于处理套接字描述符。使用socket函数创建一个套接字

```cpp
#include <sys/socket.h>

int socket(int domain, int type, int protocol);

若成功，返回文件（套接字）描述符；若出错，返回-1
```

参数domain（域）确定通信的特性，包括地址格式。各个域都有自己表示地址的格式，而表示各个域的常数都以AF_开头，表示地址族（address family）。

大多数系统还定义了AF_LOCAL域，这是AF_UNIX的别名。AF_UNSPEC域可以代表“任何”域。套接字通信域如下

| 域         | 描述    |
| ---------- | ------- |
| AF_INET  | IPv4因特网域    |
| AF_INET6  | IPv6因特网域    |
| AF_UNIX  | UNIX域  |
| AF_UNSPEC  | 未执行   |

参数type确定套接字的类型，进一步确定通信特征。

| 类型         | 描述    |
| ---------- | ------- |
| SOCK_DGRAM  | 固定长度的、无连接的、不可靠的报文传递    |
| SOCK_RAW | IP协议的数据报接口    |
| SOCK_SEQPACKET  | 固定长度的、有序的、可靠的、面向连接的报文传递  |
| SOCK_STREAM  | 有序的、可靠的、双向的、面向连接的字节流   |

参数protocol通常是0，表示为给定的域和套接字类型选择默认协议。当对同一域和套接字类型支持多个协议时，可以使用protocol选择一个特定协议。在AF_INET通信域中，套接字类型SOCK_STREAM的默认协议是传输控制协议（Transmission Control Protocol，TCP）。在AF_INET通信域中，套接字类型SOCK_DGRAM的默认协议是UDP。

| 协议         | 描述    |
| ---------- | ------- |
| IPPROTO_IP  | IPv4网际协议    |
| IPPROTO_IPV6 | IPv6网际协议    |
| IPPROTO_ICMP  | 因特网控制报文协议（Internet Control Message Protocol）  |
| IPPROTO_RAW  | 原始IP数据包协议   |
| IPPROTO_TCP  | 传输控制协议   |
| IPPROTO_UDP  | 用户数据报协议（User Datagram Protocol）   |

对于数据报（SOCK_DGRAM）接口，两个对等进程之间通信时不需要逻辑连接。只需要向对等进程所使用的套接字送出一个报文

因此数据报提供了一个无连接的服务。另一方面，字节流（SOCK_STREAM）要求在交换数据之前，在本地套接字和通信的对等进程的套接字之间建立一个逻辑连接

数据报值自包含报文。发送数据报近似于给某人邮寄信件。你能邮寄很多信，但不能保证传递的次序，并且可能有些信件会丢失在路上。每封信件包含接收者地址，使这封信件独立于所有其他信件。每封信件可能送达不同的接收者

相反，使用面向连接的协议通信就像与对方打电话。首先需要通过电话建立一个连接，连接建立好之后，彼此能双向的通信。每个连接是端到端的通信链路。对话中不包含地址信息，就像呼叫两端存在一个点对点虚拟连接，并且连接本身暗示特定的源和目的地

SOCK_STREAM套接字提供字节流服务，所以应用程序分辨不出报文的界限。这意味着从SOCK_STREAM套接字读数据时，它也许不会返回所有由发送进程所写的字节数。最终可以获得发送过来的所有数据，但也许要通过若干次函数调用才能得到。

SOCK_SEQPACKET套接字和SOCK_STREAM套接字很类似，只是从该套接字得到的是基于报文的服务而不是字节流服务。这意味着从SOCK_SEQPACKET套接字接收的数据量与对方发送的一致。流控制传输协议（Stream Control Transmission Protocol，SCTP）提供了因特网域上的顺序数据包服务

SOCK_RAW套接字提供一个数据报接口，用于直接访问下面的网络层（即因特网域中的IP层）。使用这个接口时，应用程序负责构造自己的协议头部，这是因为传输协议（如TCP和UDP）被绕过了。当创建一个原始套接字时，需要有超级用户特权，这样可以防止恶意应用程序绕过内建安全机制来创建报文

调用socket与调用open类似。在两种情况下，均可获得用于IO的文件描述符。当不再需要该文件描述符时，调用close来关闭对文件或套接字的访问，并且释放该描述符以便重新使用

虽然套接字描述符本质上是一个文件描述符，但不是所有参数为文件描述符的函数都可以接受套接字描述符。未指定和由实现定义的行为通常意味着该函数对套接字描述符无效。

![文件描述符函数使用套接字时的行为](https://gwq5210.com/images/文件描述符函数使用套接字时的行为.png)

套接字通信是双向的。可以采用shutdown函数来禁止一个套接字的IO

```cpp
#include <sys/socket.h>

int shutdown(int sockfd, int how);

若成功，返回0；若出错，返回-1
```

如果how是SHUT_RD（关闭读端），那么无法从套接字读取数据。如果how是SHUT_WR（关闭写端），那么无法使用套接字发送数据。如果how是SHUT_RDWR，则既无法读取数据，又无法发送数据。

能够关闭（close）一个套接字，为何还使用shutdown呢？这里有若干理由。首先，只有最后一个活动引用关闭时，close才释放网络断点。这意味着如果复制一个套接字（如采用dup），要直到关闭了最后一个引用它的文件描述符才会释放这个套接字。而shutdown允许使一个套接字处于不活动状态，和引用它的文件描述符数目无关。其次，有时可以很方便的关闭套接字双向传输中的一个方向。例如，如果想让所通信的进程能够确定数据传输何时结束，可以关闭该套接字的写端，然而通过该套接字读端仍可以继续接收数据

## 寻址

需要知道如何标识一个目标通信进程。进程标识由两部分组成。一部分是计算机的网络地址，它可以帮助标识网络上我们想与之通信的计算机；另一部分是该计算机上用端口号表示的服务，它可以帮助标识特定的进程

### 字节序

与同一台计算机上的进程通信时，一般不用考虑字节序。字节序是一个处理器架构特性，用于指示像整数这样的大数据类型内部的字节如何排序。

如果处理器架构支持大端（bit-endian）字节序，那么最大字节地址出现在最低有效字节（Least Significant Byte，LSB）上。小端（little-endian）字节序则相反：最低有效字节包含最小字节地址。
注意不管字节如何排序，最高有效字节（Most Significant Byte，MSB）总是在左边，最低有效字节总是在右边。因此如果想给一个32位证书赋值0x04030201，不管字节序如何，最高有效字节都将包含4，最低有效字节都将包含1.如果接下来想将一个字符指针（cp）强制转换到这个整数地址，就会看到字节序带来的不同。在小端字节序的处理器上，cp[0]指向最低有效字节因而包含1，cp[3]指向最高有效字节因而包含4。大端序则相反。

网络协议指定了字节序，因此异构计算机系统能够交换协议信息而不会被字节序所混淆。TCP/IP协议栈使用大端字节序。应用程序交换格式化数据时，字节序问题就会出现。对于TCP/IP，地址用网络字节序来表示，所以应用程序有时需要在处理器的字节序和网络字节序之间转换。

对于TCP/IP应用程序，有4个用来在处理器字节序和网络字节序之间实施转换的函数

```cpp
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostint32);

返回以网络字节序表示的32位整数

uint16_t htons(uint16_t hostint16);

返回以网络字节序表示的16位整数

uint32_t ntohl(uint32_t netint32);

返回以主机字节序表示的32位整数

uint16_t ntohs(uint16_t netint16);

返回以主机字节序表示的16位整数
```

h表示主机字节序，n表示网络字节序。l表示长（4字节）整数，s表示短（2字节）整数。这些函数常常实现为宏

## 地址格式

一个地址标识一个特定通信域的套接字端点，地址格式与这个特定的通信域相关。为何不同格式地址能够传入到套接字函数，地址会被强制转换成一个通用的地址结构sockaddr。套接字实现可以自由地添加额外的成员并且定义sa_data成员的大小。例如在Linux中和FreeBSD中，其结构定义如下

```cpp
struct sockaddr {
  as_family_t sa_family;  // address family
  char sa_data[];  // variable-length address
};

// Linux结构定义
struct sockaddr {
  sa_family_t sa_family; // address family
  char sa_data[14];  // variable-length address
};

// FreeBSD结构定义
struct sockaddr {
  unsigned char sa_len;  // total length
  sa_family_t sa_family; // address family
  char sa_data[14]; // variable-length address
};
```

因特网地址定义在<netinet/in.h>头文件中。在IPv4因特网域（AF_INET）中，套接字地址用结构sockaddr_in表示：

```cpp
struct in_addr {
  in_addr_t s_addr; // IPv4 address（定义为uint32_t）
};

struct sockaddr_in {
  sa_family_t sin_family; // address family
  in_port_t sin_port; // port number (in_port_t定义为uint16_t)
  struct in_addr sin_addr; // IPv4 address
};
```

与AF_INET域相比较，IPv6因特网域（AF_INET6）套接字地址用结构sockaddr_in6表示

```cpp
struct in6_addr {
  uint8_t s6_addr[16];  // IPv6 address
};

struct sockaddr_in6 {
  sa_family_t sin6_family; // address family
  in_port_t sin6_port; // port number
  uint32_t sin6_flowinfo;  // traffic class and flow info
  struct in6_addr sin6_addr; // IPv6 address
  uint32_t sin6_scope_id;  // set of interfaces for scope
};
```

这些都是Single UNIX Specification要求的定义。每个实现可以自由添加更多的字段。例如，在Linux中，sockaddr_in定义如下

```cpp
struct sockaddr_in {
  sa_family_t sin_family;  // address family
  in_port_t sin_port;  // port number
  struct in_addr sin_addr;  // IPv4 address
  unsigned char sin_zero[8];  // filler，填充字段，应该全部被置为0
};
```

注意，尽管sockaddr_in和sockaddr_in6的结构相差比较大，但它们均被强制转换成sockaddr结构输入到套接字例程中。UNIX域套接字地址的结构与上述两个因特网域套接字地址格式稍有不同

有时，需要打印出能被人理解而不是计算机所理解的地址格式。BSD网络软件包含函数inet_addr和inet_ntoa，用于二进制地址格式与点分十进制字符表示（a.b.c.d）之间的相互转换。但是这个函数仅适用于IPv4地址。有两个新函数inet_ntop和inet_pton具有相似的功能，而且同时支持IPv4和IPv6地址

```cpp
#include <arpa/inet.h>

const char* inet_ntop(int domain, const void* restrict addr, char* restrict str, socklen_t size);

若成功，返回地址字符串指针；若出错，返回NULL

int inet_pton(int domain, const char* restrict str, void* restrict addr);

若成功，返回1；若格式无效，返回0；若出错，返回-1
```

函数inet_ntop将网络字节序的二进制地址转换成文本字符串格式。inet_pton将文本字符串格式转换成网络字节序的二进制地址。参数domain仅支持两个值：AF_INET和AF_INET6。对于inet_ntop，参数size指定了保存文本字符串的缓冲区（str）的大小。两个常数用于简化工作：INET_ADDDRSTRLEN定义了足够大的空间来存放一个标识IPv4地址的文本字符串；INET6_ADDDRSTRLEN定义了足够大的空间来存放一个表示IPv6地址的文本字符串。对于inet_pton，如果domain是AF_INET，则缓冲区addr需要足够大的空间来存放一个32位地址，如果domain是AF_INET6，则需要足够大的空间来存放一个128位地址

## 地址查询

理想情况下，应用程序不需要了解一个套接字地址的内部结构。如果一个程序简单的传递一个类似于sockaddr结构的套接字地址，并且不依赖于任何协议相关的特性，那么可以与提供相同类型服务的许多不同协议协作

历史上，BSD网络软件提供了访问各种网络配置信息的接口。我们需要深入了解一些细节，并引入新的函数来查询寻址信息

这些函数返回的网络配置信息被存放在许多地方。这个信息可以存放在静态文件（如/etc/hosts和/etc/services）中，也可以由名字服务管理，如域名系统（Domain Name System，DNS）或者网络信息服务（Network Infomation Service，NIS）。无论这个信息放在何处，都可以用同样的函数访问它

通过调用gethostent，可以找到给定计算机系统的主机信息

```cpp
#include <netdb.h>

struct hostent* gethostent(void);

若成功，返回指针；若出错，返回NULL

void sethostent(int stayopen);
void endhostent(void);
```

如果主机数据库文件没有打开，gethostent会打开它。函数gethostent返回文件中的下一个条目。函数sethostent会打开文件，如果文件已经被打开，那么将其回绕。当stayopen参数设置成非0值时，调用gethostent之后，文件将亦然是打开的。函数endhostent可以关闭文件。

当gethostent返回时，会得到一个指向hostent结构的指针，该结构可能包含一个静态的数据缓冲区，每次调用gethostent，缓冲区都会被覆盖。hostent结构至少包含以下成员，返回的地址采用网络字节序

```cpp
struct hostent {
  char* h_name;  // name of host
  char** h_aliases;  // pointer to alternate host name array
  int h_addrtype;  // address type
  int h_length;  // length in bytes of address
  char** h_addr_list;  // pointer to array of network addresses
};
```

可以使用一套相似的接口来获得网络名字和网络编号

```cpp
#include <netdb.h>

struct netent* getnetbyaddr(uint32_t net, int type);
struct netent* getnetbyname(const char* name);
struct netent* getnetent(void);

若成功，返回指针；若出错，返回NULL

void setnetent(int stayopen);
void endnetent(void);
```

netent结构至少包含以下字段

```cpp
struct netent {
  char* n_name;  // network name
  char** n_aliases; // alternate network name array pointer
  int n_addrtype;  // address type
  uint32_t n_net;  // network number
};
```

网络编号按照网络字节序返回。地址类型是地址族常量之一（如AF_INET）

我们可以用以下函数在协议名字和协议编号之间进行映射

```cpp
#include <netdb.h>

struct protoent* getprotobyname(const char* name);
struct protoent* getprotobynumber(int proto);
struct protoent* getprotoent(void);

若成功，返回指针；若出错，返回NULL

void setprotoent(int stayopen);
void endprotoent(void);
```

POSIX.1定义的protoent结构至少包含以下成员

```cpp
struct protoent {
  char* p_name;  // protocol name
  char** p_aliases;  // pointer to alternate protocol name array
  int p_proto;  // protocol number
};
```

服务是由地址的端口号部分表示的。每个服务由一个唯一的众所周知的端口号来支持。可以使用函数getservbyname将一个服务名映射到一个端口号，使用函数getservbyport将一个端口号映射到一个服务名，使用函数getservent顺序扫描服务数据库

```cpp
#include <netdb.h>

struct servent* getservbyname(const char* name, const char* proto);
struct servent* getservbyport(int port, const char* proto);
struct servent* getservent(void);

若成功，返回指针；若出错，返回NULL

void setservent(int stayopen);
void endservent(void);
```

servent结构至少包含以下成员

```cpp
struct servent {
  char* s_name;  // service name
  char** s_aliases;  // pointer to alternate service name array
  int s_port;  // port number
  char* s_proto;  // name of protocol
};
```

POSIX.1定义了若干新函数，允许一个应用程序将一个主机名和一个服务名映射到一个地址，或者反之。这些函数替代了较老的函数gethostbyname和gethostbyaddr

getaddrinfo函数允许将一个主机名和一个服务名映射到一个地址

```cpp
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char* restrict host, const char* restrict service, const struct addrinfo* restrict hint, struct addrinfo** restrict res);

若成功，返回0；若出错，返回非0错误码

void freeaddrinfo(struct addrinfo* ai);
```

需要提供主机名、服务名，或者两者都提供。如果仅仅提供一个名字，另外一个应该是一个空指针。主机名可以是一个节点名或点分格式的主机地址

getaddrinfo函数返回一个链表结构addrinfo。可以用freeaddrinfo来释放一个或多个这种结构，这取决于用ai_next字段链接起来的结构有多少

addrinfo结构的定义至少包含以下成员

```cpp
struct addrinfo {
  int ai_flags;  // customize behavior
  int ai_family;  // address family
  int ai_socktype;  // socket type
  int ai_protocol; // protocol
  socklen_t ai_addrlen;  // length in bytes of address
  struct sockaddr* ai_addr;  // address
  char* ai_canonname;  // canonical name of host
  struct addrinfo* ai_next;  // next in list
};
```

可以提供一个可选的hint来选择符合特定条件的地址。hint是一个用于过滤地址的模板，包括ai_family、ai_flags、ai_protocol和ai_socktype字段。剩余的整数字段必须设置为0，指针字段必须为空。

ai_flags字段中的标志，可以用这些标志来自定义如何处理地址和名字

| 标志         | 描述    |
| ---------- | ------- |
| AI_ADDRCONFIG  | 查询配置的地址类型（IPv4或IPv6）    |
| AI_ALL | 查询IPv4和IPv6地址（仅用于AI_V4MAPPED）    |
| AI_CANONNAME  | 需要一个规范的名字（与别名相对）  |
| AI_NUMERICHOST  | 以数字格式指定主机地址，不翻译   |
| AI_NUMERICSERV  | 将服务指定为数字端口号，不翻译   |
| AI_PASSIVE  | 套接字地址用于监听绑定   |
| AI_V4MAPPED  | 如果没有找到IPv6地址，返回映射到IPv6格式的IPv4地址   |

如果getaddrinfo失败，不能使用perror或strerror来生成错误消息，而是要调用gai_strerror将返回的错误码转换成错误消息

```cpp
#include <netdb.h>

const char* gai_strerror(int error);

返回指向描述错误的字符串的指针
```

getnameinfo函数将一个地址转换成一个主机名和一个服务名

```cpp
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr* restrict addr, socklen_t alen, char* restrict host, socklen_t hostlen, char* restrict service, socklen_t servlen, int flags);

若成功，返回0；若出错，返回非0值
```

套接字地址（addr）被翻译成一个主机名和一个服务名。如果host非空，则指向一个长度为hostlen字节的缓冲区用于存放返回的主机名。同样如果service非空，则指向一个长度为servlen字节的缓冲区用于存放返回的主机名

flags参数提供了一些控制翻译的方式

![getnameinfo函数的标志](https://gwq5210.com/images/getnameinfo函数的标志.png)

## 将套接字与地址关联

将一个客户端的套接字关联上一个地址没有多少新意，可以让系统选一个默认的地址。然而，对于服务器，需要给一个接收客户端请求的服务器套接字关联上一个众所周知的地址。客户端应有一种方法来发现连接服务器所需要的地址，最简单的方法就是服务器保留一个地址并且注册在/etc/services或者某个名字服务中

使用bind函数来关联地址和套接字

```cpp
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr* addr, socklen_t len);

若成功，返回0；若出错，返回-1
```

对于使用的地址有以下一些限制

- 在进程正在运行的计算机上，指定的地址必须有效；不能指定一个其他机器的地址
- 地址必须和创建套接字时的地址族所支持的格式相匹配
- 地址中的端口号必须不小于1024，除非该进程具有相应的特权（即超级用户）
- 一般只能将一个套接字端点绑定到一个给定的地址上，尽管有些协议允许多重绑定

对于因特网域，如果指定IP地址为INADDR_ANY（<netinet/in.h>中定义的），套接字端点可以被绑定到所有的系统网络接口上。这意味着可以接收这个系统所安装的任何一个网卡的数据包。如果调用connect或listen，但没有将地址绑定到套接字上，系统会选一个地址绑定到套接字上

可以调用getsockname函数来发现绑定到套接字上的地址

```cpp
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr* restrict addr, socklen_t* restrict alenp);

若成功，返回0；若出错，返回-1
```

调用getsockname之前，将alenp设置为一个指向整数的指针，该整数指定缓冲区sockaddr的长度。返回时，该整数会被设置成返回地址的大小。如果地址和提供的缓冲区长度不匹配，地址会被自动截断而不报错。如果当前没有地址绑定到该套接字，则其结果是未定义的。

如果套接字已和对等方连接，可以调用getpeername函数来找到对方的地址

```cpp
#include <sys/socket.h>

int getpeername(int sockfd, struct sockaddr* restrict addr, socklen_t* restrict alenp);

若成功，返回0；若出错，返回-1
```

除了返回对等方的地址，函数getpeername和getsockname一样

## 建立连接

如果要处理一个面向连接的网络服务（SOCK_STREAM或SOCK_SEQPACKET），那么在开始交换数据以前，需要在请求服务的进程套接字（客户端）和提供服务的进程套接字（服务器）之间简历一个连接。使用connect函数来建立连接

```cpp
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr* addr, socklen_t len);

若成功，返回0；若出错，返回-1
```

在connect中指定的地址是我们想与之通信的服务器地址。如果sockfd没有绑定到一个地址，connect会给调用者绑定一个默认地址

当尝试连接服务器时，出于一些原因，连接可能会失败。要想一个连接请求成功，要连接的计算机必须是开启的，并且正在运行，服务器必须绑定到一个想与之连接的地址上，并且服务器的等待连接队列要有足够的空间。因此应用程序必须能够处理connect返回的错误，这些错误可能是由一些瞬时条件引起的。因此可以对connect进行重试connect来处理瞬时错误，可以使用指数补偿算法，每次休眠时间指数级增加

如果套接字描述符处于非阻塞模式，那么在连接不能马上建立时，connect将会返回-1并且将errno设置为特殊的错误码EINPROGRESS。应用程序可以使用poll或者select来判断文件描述符何时可写。如果可写，连接完成。

connect函数还可以用于无连接的网络服务（SOCK_DGRAM）。这看起来有点矛盾，实际上却是一个不错的选择。如果用SOCK_DGRAM套接字调用connect，传送的报文的目标地址会设置成connect调用中所指定的地址，这样每次传送报文时就不需要再提供地址。另外，仅能接收来自指定地址的报文

服务器调用listen函数来宣告它愿意接受连接请求

```cpp
#include <sys/socket.h>

int listen(int sockfd, int backlog);

若成功，返回0；若出错，返回-1
```

参数backlog提供了一个提示，提示系统该进程所要入队列的未完成连接请求数量。其实际值由系统决定，但上限由<sys/socket.h>中的SOMAXCONN指定

一旦队列满，系统就会拒绝对于的连接请求，所以backlog的值应该基于服务器期望负载和处理量来选择，其中处理量是指接受连接请求与启动服务的数量

一旦服务器调用了listen，所用的套接字就能接收连接请求。使用accept函数获得连接请求并建立连接

```cpp
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr* restrict addr, socklen_t* restrict len);

若成功，返回文件（套接字）描述符；若出错，返回-1
```

函数accept所返回的文件描述符是套接字描述符，该描述符连接到调用connect的客户端。这个新的套接字描述符和原始套接字（sockfd）具有相同的套接字类型和地址族。传给accept的原始套接字没有关联到这个连接，而是继续保持可用状态并接收其他连接请求

如果不关心客户端标识，可以将参数addr和len设置为NULL。否则，在调用accept之前，将addr参数设为足够大的缓冲区来存放地址，并且将len指向的整数设置为这个缓冲区的字节大小。返回时，accept会在缓冲区填充客户端的地址，并且更新指向len的整数来反映改地址的大小

如果没有连接请求在等待，accept会阻塞直到一个请求到来。如果sockfd处于非阻塞模式，accept会返回-1，并将errno设置为EAGAIN或EWOULDBLOCK。

如果服务器调用accept，并且当前没有连接请求，服务器会阻塞直到一个请求到来。另外，服务器可以使用poll或select来等待一个请求的到来。在这种情况下，一个带有等待连接请求的套接字会以可读的方式出现

## 数据传输

