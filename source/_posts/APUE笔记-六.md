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

既然一个套接字端点表示为一个文件描述符，那么只要建立连接，就可以使用read和write来通过套接字通信。在套接字描述符上使用read和write是非常有意义的，因为这意味着可以将套接字描述符传递给那些原先为处理本地文件而设计的函数。而且还可以安排将套接字描述符传递给子进程，而该子进程执行的程序并不了解套接字

尽管可以通过read和write交换数据，但这就是这两个函数所能做的一些。如果想指定选项，从多个客户端接收数据包，或者发送带外数据，就需要使用6个位数据传递而设计的套接字函数中的一个

3个函数用来发送数据，3个用于接收数据

最简单的是send，它和write很像，但是可以指定标志来改变传输数据的方式

```cpp
#include <sys/socket.h>

ssize_t send(int sockfd, const void* buf, size_t nbytes, int flags);

若成功，返回发送的字节数；若出错，返回-1
```

类似write，使用send时套接字必须已经连接。参数buf和nbytes的含义与write中的一致。send支持第4个参数flags。标志如下

![send套接字调用标志](https://gwq5210.com/images/send套接字调用标志.png)

即使send成功返回，也并不表示连接的另一端的进程就一定接收了数据。我们所能保证的只是当send成功返回时，数据已经被无错误的发送到网络驱动程序上

对于支持报文边界的协议，如果尝试发送的单个报文的长度超过协议所支持的最大长度，那么send会失败，并将errno设置为EMSGSIZE。对于字节流协议，send会阻塞直到整个数据传输完成。函数sendto和send很类似。区别在于sendto可以在无连接的套接字上指定一个目标地址

```cpp
#include <sys/socket.h>

ssize_t sendto(int sockfd, const void* buf, size_t nbytes, int flags, const struct sockaddr* addr, socklen_t destlen);

若成功，返回发送的字节数；若出错，返回-1
```

对于面向连接的套接字，目标地址是被忽略的，因为连接中隐含了目标地址。对于无连接的套接字，除非先调用connect设置了目标地址，否则不能使用send。sendto提供了发送报文的另一种方式

通过套接字发送数据时，还有一个选择。可以调用带有msghdr结构的sendmsg来指定多重缓冲区传输数据，这和writev函数很相似

```cpp
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr* msg, int flags);

若成功，返回发送的字节数；若出错，返回-1
```

POSIX.1定义了msghdr结构，至少有以下成员

```cpp
struct msghdr {
  void* msg_name;  // optional address
  socklen_t msg_namelen;  // address size in bytes
  struct iovec* msg_iov;  // array of I/O buffers
  int msg_iovlen;  // number of elements in array
  void* msg_control;  // ancillary data
  socklen_t msg_controllen;  // number of ancillary bytes
  int msg_flags;  // flags for received message
};
```

函数recv和read相似，但是recv可以指定标志来控制如何接收数据

```cpp
#include <sys/socket.h>

ssize_t recv(int sockfd, void* buf, size_t nbytes, int flags);

若成功，返回数据的字节长度；若无可用数据或对等方已经按序结束，返回0；若出错，返回-1
```

支持的标志如下

![recv套接字调用标志](https://gwq5210.com/images/recv套接字调用标志.png)

当指定MSG_PEEK标志时，可以查看下一个要读取的数据但不真正取走它。当再次调用read或其中一个recv函数时，会返回刚才查看的数据

对于SOCK_STREAM套接字，接收的数据可以比预期的少。MSG_WAITALL标志会阻止这种行为，直到所请求的数据全部返回，recv函数才会返回。对于SOCK_DGRAM和SOCK_SEQPACKET套接字，MSG_WAITALL标志没有改变什么行为，因为这些基于报文的套接字类型一次读取就返回整个报文

如果发送者已经调用shutdown来结束传输，或者网络协议支持按默认的顺序关闭并且发送端已经关闭，那么当所有的数据接收完毕后，recv会返回0

如果有兴趣定位发送者，可以使用recvfrom来得到数据发送者的源地址

```cpp
#include <sys/socket.h>

ssize_t recvfrom(int sockfd, void* restrict buf, size_t len, int flags, struct sockaddr* restrict addr, socklen_t* restrict addrlen);

返回数据的字节长度；若无可用数据或对等方已经按序结束，返回0；若出错，返回-1
```

如果addr非空，它将包含数据发送者的套接字端点地址。当调用recvfrom时，需要设置addrlen参数指向一个整数，该整数包含addr所指向的套接字缓冲区的字节长度。返回时，该整数设置为该地址的实际字节长度

因为可以获得发送者的地址，recvfrom通常用于无连接的套接字。否则，recvfrom等同于recv

为了将接收到的数据送入多个缓冲区，类似于readv，或者想接收辅助数据，可以使用recvmsg

```cpp
#include <sys/socket.h>

ssize_t recvmsg(int sockfd, struct msghdr* msg, int flags);

返回数据的字节长度；若无可用数据或对等方已经按序结束，返回0；若出错，返回-1
```

recvmsg用msghdr结构指定接收数据的输入缓冲区。可以设置参数flags来改变recvmsg的默认行为。返回时，msghdr结构中的msg_flags字段被设置为所接收数据的各种特征。进入recvmsg时msg_flags被忽略。recvmsg中返回的各种可能值如下

![从recvmsg中返回的msg_flags标志](https://gwq5210.com/images/从recvmsg中返回的msg_flags标志.png)

对于无连接的套接字，数据包到达时可能已经没有次序，因此如果不能将所有的数据放在一个数据包里，则在应用程序中就必须关心数据包的次序。数据包的最大尺寸是通信协议的特征。另外对于无连接的套接字，数据包可能会丢失。如果应用程序不能容忍这种丢失，必须使用面向连接的套接字

容忍数据包丢失意味着两种选择。一种选择是，如果想和对等方可靠通信，就必须对数据包编号，并且在发现数据包丢失时，请求对等应用程序重传，还必须标识重复数据包并丢弃它们，因为数据包可能会延迟或疑似丢失，可能请求重传之后，它们又出现了

另一种选择是，通过让用户再次尝试那个命令来处理错误。对于简单的应用程序，这可能就足够了，但对于复杂的应用程序，这种选择通常不行。因此，一般在这种情况下使用面向连接的套接字比较好

面向连接的套接字的缺陷在于需要更多的时间和工作来建立一个连接，并且每个连接都需要消耗较多的系统资源

## 套接字选项

套接字机制提供了两个套接字选项接口来控制套接字行为。一个接口用来设置选项，另一个接口可以查询选项的状态。可以获取或设置以下3中选项

- 通用选项，工作在所有套接字类型上
- 在套接字层次管理的选项，但是依赖于下层协议的支持
- 特定于某协议的选项，每个协议独有的

可以使用setsockopt函数来设置套接字选项

```cpp
#include <sys/socket.h>

int setsockopt(int sockfd, int level, int option, const void* val, socklen_t len);

若成功，返回0；若出错，返回-1
```

参数level表示了选项应用的协议。如果选项是通用的套接字层次选项，则level设置成SOL_SOCKET。否则，level设置成控制这个选项的协议编号。对于TCP选项，level是IPPROTO_TCP，对于IP，level是IPPROTO_IP。下图是Single UNIX Specification中定义的通用套接字层次选项

![套接字选项](https://gwq5210.com/images/套接字选项.png)

参数val根据选项的不同指向一个数据结构或一个整数。一些选项是on/off开关。如果整数非0，则启用选项。如果整数为0，则禁止选项。参数len指定了val指向的对象的大小

可以使用getsockopt函数来查看选项的当前值

```cpp
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int option, void* restrict val, socklen_t* restrict lenp);

若成功，返回0；若出错，返回-1
```

参数lenp是一个指向整数的指针。在调用getsockopt之前，设置该整数为复制选项缓冲区的长度。如果选项的实际长度大于此值，则选项会被截断。如果实际长度正好小于此值，那么返回时将此值更新为实际长度

## 带外数据

带外数据（out-of-band data）是一些通信协议所支持的可选功能，与普通数据相比，它允许更高优先级的数据传输。带外数据先行传输，即使传输队列已经有数据。TCP支持带外数据，但是UDP不支持。套接字接口对带外数据的支持很大程度上受TCP带外数据具体实现上的影响

TCP将带外数据称为紧急数据（urgent data）。TCP仅支持一个字节的紧急数据，但是允许紧急数据在普通数据传递机制数据流之外传输。为了产生紧急数据，可以在3个send函数中的任何一个里指定MSG_OOB标志。如果带MSG_OOB标志发送的字节数超过一个时，最后一个字节将被视为紧急数据字节

如果通过套接字安排了信号的产生，那么紧急数据被接收时，会发送SIGURG信号。在fcntl中使用F_SETOWN命令来设置一个套接字的所有权。如果fcntl中的第三个参数为正值，那么它指定的就是进程ID。如果为非-1的负值，那么它代表的就是进程组ID。因此，可以通过调用以下函数安排进程接收套接字的信号。

```cpp
fcntl(sockfd, F_SETOWN, pid);
```

F_GETOWN命令可以用来获得当前套接字的所有权。对于F_GETOWN命令，负值代表进程组ID，正值代表进程ID。因此调用`owner = fcntl(sockfd, F_GETOWN, 0)`将返回owner，如果owner为正值，则等于配置为接收套接字信号的进程的ID。如果owner为负值，其绝对值为接收套接字信号的进程组ID

TCP支持紧急标记（urgent mark）的概念，即在普通数据流中紧急数据所在的位置。如果采用套接字选项SO_OOBINLINE，那么可以在普通函数中接收紧急数据。为帮助判断是否已经到达紧急标记，可以使用函数sockatmark

```cpp
#include <sys/socket.h>

int sockatmark(int sockfd);

若在标记处，返回1；若没在标记处，返回0；若出错，返回-1
```

当下一个要读取的字节在紧急标志处时，sockatmark返回1

当带外数据出现在套接字读取队列时，select函数会返回一个文件描述符并且有一个待处理的异常条件。可以在普通数据流上接收紧急数据，也可以在其中一个recv函数中采用MSG_OOB标志在其他队列数据之前接收紧急数据。TCP队列仅用一个字节的紧急数据。如果在接收当前的紧急数据字节之前又有新的紧急数据到来，那么已有的字节会被丢弃

## 非阻塞和异步IO

通常，recv函数没有数据可用时会阻塞等待。同样的，当套接字输出队列没有足够空间来发送消息时，send函数会阻塞。在套接字非阻塞模式下，行为会改变。在这种情况下，这些函数不会阻塞而是会失败，将errno设置为EWOULDBLOCK或者EAGAIN。当这种情况发生时，可以使用poll或select来判断能否接收或者传输数据

套接字的异步IO机制成为基于信号的IO，通常用的很少。

# 第十七章

## UNIX域套接字

UNIX域套接字用于在同一台计算机上运行的进程之间的通信。虽然因特网域套接字可用于同一目的，但UNIX域套接字的效率更高。UNIX域套接字仅仅复制数据，它们并不执行协议处理，不需要添加或删除网络报头，无需计算校验和，不需要产生顺序号，无需发送确认报文

UNIX域套接字提供流和数据报两种接口。UNIX域数据报服务是可靠的，既不会丢失报文也不会传递出错。UNIX域套接字就像是套接字和管道的混合。可以使用它们面向网络的域套接字接口或者使用socketpair函数来创建一对无命名的，相互连接的UNIX域套接字

```cpp
#include <sys/socket.h>

int socketpair(int domain, int type, int protocol, int sockfd[2]);

若成功，返回0；若出错，返回-1
```

虽然接口足够通用，允许socketpair用于其他域，但一般来说操作系统仅对支持UNIX域提供支持

一对相互连接的UNIX域套接字可以起到全双工管道的作用：两端对读和写开放。我盟将其称为fd管道（fd-pipe），以便与普通的半双工管道区分开来

![套接字对](https://gwq5210.com/images/套接字对.png)

## 命名UNIX域套接字

可以命名UNIX域套接字，并可将其用于告示服务。但是要注意，UNIX域套接字使用的地址格式不同于因特网域套接字

UNIX域套接字的地址由sockaddr_un结构表示。一般在<sys/un.h>中定义，在Linux 3.2.0中定义如下

```cpp

// Linux 3.2.0 and Solaris 10
struct sockaddr_un {
  sa_family_t sun_family;  // AF_UNIX
  char sun_path[108];  // pathname
};

// FreeBSD 8.0 and Mac OS X 10.6.8
struct sockaddr_un {
  unsigned char sun_len;  // sockaddr length
  sa_family_t sun_family;  // AF_UNIX
  char sun_path[108];  // pathname
};
```

sun_path成员包含一个路径名。当我们将一个地址绑定到一个UNIX域套接字时，系统会用该路径名创建一个S_IFSOCK类型的文件。该文件仅用于向客户进程告示套接字名字。该文件无法打开，也不能由应用程序用于通信。

如果我们试图绑定同一地址时，该文件已经存在，那么bind请求会失败。当关闭套接字时，并不会自动删除文件，所以必须确保在应用程序退出前，对该文件执行解除链接操作
