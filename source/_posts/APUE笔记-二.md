title: APUE笔记-二
date: 2022-07-17 23:58:29
tags: [unix, linux, 读书笔记, APUE]
---

# 第五章 标准IO库

标准IO库是ISO C的标准，不仅仅UNIX系统提供了，很多其他操作系统也都实现了标准IO库

## 流和FILE对象

在第三章所有的文件IO都是围绕文件描述符的。对于标准IO库，其操作是围绕流（stream）进行的，打开或创建一个文件时，使一个流与一个文件相关联

标准IO文件流可用于单字节和多字节字符集。流的定向（stream's orientation）决定了读、写字符是单字节还是多字节的。

当一个流最初被创建时，它并没有定向。若在流上使用一个单字节IO函数，则将流设置为字节定向的。若在流上使用一个多字节IO函数，则将流设置为字宽定向的。

只有freopen和fwide这两个函数可以改变流的定向。

```cpp
#include <stdio.h>

#include <wchar.h>

int fwide(FILE* fp, int mode);

返回值: >0 : 宽定向；<0 字节定向；0 : 未定向
```

fwide并不能改变已定向流的定向。fwide无法返回出错。需要在调用fwide前清除errno，返回时再检查errno的值

一个流的结构（FILE*，通常称为文件指针）通常包含一些必要的信息，如实际IO的文件描述符，缓存区指针和长度，当前在缓冲区中的字符数和出错标志等

一个进程预定义了3个流，标准输入流，标准输出流，标准出错流(分别是stdin, stdout, stderr)可以自动被进程使用。分别对应文件描述符STDIN_FILENO, STDOUT_FILENO, STDERR_FILENO

## 缓冲

标准IO库提供缓冲的目的是尽可能减少read、write系统调用的次数。它对每个IO流自动进行缓冲管理，避免应用程序考虑应使用多大的缓冲区。

标准IO提供3中类型的缓冲

- 全缓冲。这种情况下，缓冲区满才进行实际的IO操作，磁盘文件通常是全缓冲的。当然我们可以通过fflush手动冲洗流的缓冲区
- 行缓冲。这种情况下，输入和输出遇到换行符执行IO操作。流涉及一个终端时通常使用行缓冲。行缓冲有以下两个限制
  - 缓冲区大小固定。缓冲区满时，即使没有换行符也进行IO操作
  - 任何时候，只要通过标准IO库要求从a）不带缓冲的流或b)一个行缓冲的流得到输入数据，那么就会冲洗所有行缓冲输出流
- 不带缓冲。标准IO库不对字符进行缓冲存储。标准错误流通常是不带缓冲的，这使得出错信息可以尽快的显示出来

ISO C要求

- 当且仅当标准输入和标准输出并不指向交互式设备时，它们才是全缓冲的
- 标准错误绝不会是全缓冲的

一般系统默认如下方案

- 标准错误流是不带缓冲的
- 若是指向终端的流，则是行缓冲的；否则是全缓冲的

我们可以通过如下函数修改系统默认的缓冲类型

```cpp
#include <stdio.h>

void setbuf(FILE* restrict fp, char* restrict buf);
int setvbuf(FILE* restrict fp, char* restrict buf, int mode, size_t size);

若成功，返回0；若出错，返回非0
```

以上函数应在流被打开后调用，且应在对流执行任何一个其他操作之前调用

可以使用setbuf打开或关闭缓冲机制。buf为NULL，则关闭缓冲；为了带缓冲进行IO，参数buf必须指向一个长度为BUFSIZ的缓冲区。一般在此之后流就是全缓冲的，但是如果一个流与一个终端设备相关，某些系统也可将其设置为行缓冲的

使用setvbuf可以通过mode精确的指定缓冲类型

- _IOFBF：全缓冲
- _IOLBF：行缓冲
  - 以上可以通过buf和size设置缓冲区和长度；如果buf为NULL，则标准IO库将自动分配合适的缓冲区，适当长度一般由BUFSIZ指定（GNU C使用stat结构中的st_blksize来决定最佳缓冲区长度）
- _IONBF：不带缓冲，忽略buf和size参数

![setbuf和setvbuf函数](https://gwq5210.com/images/setbuf和setvbuf函数.png)

一般而言我们应该由系统选择缓冲区的长度，并自动分配缓冲区，流关闭时，将自动释放缓冲区

任何时候我们可以强制冲洗一个流

```cpp
#include <stdio.h>

int fflush(FILE* fp);

若成功，返回0；若出错，返回EOF
```

此函数使所有未写的数据都传送至内核。如果fp是NULL，则导致所有的输出流被冲洗

## 打开流

以下3个函数打开一个标准流

```cpp
#include <stdio.h>

FILE* fopen(const char* restrict pathname, const char* restrict type);
FILE* freopen(const char* restrict pathname, const char* restrict type, FILE* restrict fp);
FILE* fdopen(int fd, const char* restrict type);

若成功，返回文件指针；若出错，返回NULL
```

说明如下

- fopen打开路径名为pathname的指定文件
- freopen在一个指定流上打开一个指定文件，如若流已经被打开，则先关闭该流。若流已经定向，则使用freopen清除该定向。此函数一般用于将一个指定文件打开为一个预定义的流：标准输入、标准输出、标准错误
- fdopen函数取一个已有的文件描述符（如从open、dup、dup2、fcntl、pipe、socket、socketpair或accept函数得到），并使一个标准的IO流与此文件描述符相结合

参数type有如下组合

![fopen的type](https://gwq5210.com/images/fopen的type.png)

使用字符b可以使得标准IO系统区分二进制和文本文件。UNIX内核并不区分这两种文件，所以在UNIX系统下并无作用

对于fdopen，type参数的作用稍有区别。因为该描述符已被打开，所有fdopen为写打开并不截断文件。标准IO追加写方式也不能用于创建该文件（该文件一定存在，fd引用一个文件）

当以读和写类型（type中+号）打开一个文件时，有如下限制

- 如果中间没有fflush、fseek、fsetpos或rewind，则在输出后边不能直接跟随输入
- 如果中间没有fseek、fsetpos或rewind或者一个输入操作没有到达文件尾端，则在输入操作之后不能直接跟随输出

![打开流的不同方式](https://gwq5210.com/images/打开流的不同方式.png)

在w和a类型创建一个文件时，我们无法指定文件的访问权限位，我们可以通过调整umask值来限制这些权限或创建之后修改权限

调用fclose可以关闭一个打开的流

```cpp
#include <stdio.h>

int fclose(FILE* fp);

若成功，返回0；若出错，返回EOF
```

在该文件被关闭之前，冲洗缓冲区中的输出数据，缓冲区中的任何输入数据被丢弃。如果分配和缓冲区，则自动释放缓冲区

进程正常终止时（直接调用exit或从main函数返回），则所有带未写缓冲数据的标准IO流都被冲洗，所有打开的标准IO流都被关闭

## 读和写流

有三种不同的非格式化IO可以对流进行读写操作

- 每次一个字符的IO。如果流带缓冲，则标准IO函数处理所有缓冲
- 每次一行的IO。
- 直接IO，或者称为二进制IO。fread和fwrite支持这种类型的IO

### 输入函数

以下函数可以一次读取一个字符

```cpp
#include <stdio.h>

int getc(FILE* fp);  // 可以实现为宏
int fgetc(FILE* fp);  // 不能实现为宏
int getchar(void); // 等同于fgetc(stdin);

若成功，返回下一个字符；若已到文件结尾或出错，返回EOF
```

返回值会将unsigned char转换为int类型。无符号即使最高位为1也不会返回负值。要求int返回值是因为除了返回所有可能的字符，还要加上一个已出错或到达文件尾端的标记值。EOF被要求是一个负值，通常为-1

为了区分出错或到达文件尾端，需要使用以下函数进行区分

```cpp
#include <stdio.h>

int ferror(FILE* fp);
int feof(FILE* fp);

若条件为真，返回非0；否则，返回0

void clearerr(FILE* fp);
```

大多数实现在文件指针中维护了两个标志：出错标志和文件结束标志；调用clearerr可以清除这两个标志

从流中读取数据后，可以调用ungetc再将字符压入流中

```cpp
#include <stdio.h>

int ungetc(int c, FILE* fp);

若成功，返回c；若出错，返回EOF
```

压送回流的字符又可以从流中读出，但读出字符的顺序与压送回流的顺序相反。回送的字符不一定必须是上次读到的字符，不能回送EOF。但是当已经达到文件尾端时，仍可以回送一个字符。调用ungetc只是将字符写入流缓冲区中

### 输出函数

```cpp
#include <stdio.h>

int putc(int c, FILE* fp);  // 可以实现为宏
int fputc(int c, FILE* fp);  // 不能实现为宏
int putchar(int c); // 等同于fputc(stdin);

若成功，返回c；若出错，返回EOF
```

## 每次一行IO

由以下函数提供每次输入一行的功能

```cpp
#include <stdio.h>

char* fgets(char* restrict buf, int n, FILE* restrict fp);
char* gets(char* restrict buf);  // 可能造成缓冲区溢出，应使用fgets(buf, n, stdin)替换

若成功，返回buf；若已到达文件末尾或出错，返回NULL
```

fgets读取不超过n-1个字符，buf总是以null字节结尾。gets不将换行符写入buf，而fgets则将换行符写入buf

fputs和puts提供每次输出一行的功能

```cpp
#include <stdio.h>

int fputs(const char* restrict str, FILE* restrict fp);
int puts(const char* str);  // 等价于fputs(str, stdout); fputs("\n", stdout);

若成功，返回非负值；若出错，返回EOF
```

应尽量使用fget和fputs组合，以便记得在每行终止时必须处理换行符

## 标准IO的效率

![标准IO库的效率](https://gwq5210.com/images/标准IO库的效率.png)

## 二进制IO

使用以下函数处理二进制IO

```cpp
#include <stdio.h>

size_t fread(void* restrict ptr, size_t size, size_t nobj, FILE* restrict fp);
size_t fwrite(const void* restrict ptr, size_t size, size_t nobj, FILE* restrict fp);

返回读或写的对象数
```

对于读，如果出错或到达文件尾端，则此数字可以小于nobj，这时应调用ferror或feof判断是哪种情况。对于写如果返回值小于nobj则出错

二进制IO需要对读写的数据有限制，否则可能产生兼容性的问题。如结构体对齐问题，导致数据错误

## 定位流

有三种方法定位标准IO流

- ftell和fseek，用long进行定位
- ftello和fseeko，用off_t进行定位
- fgetpos和fsetpos，由ISO C引入，使用fpos_t进行定位；移植到非UNIX系统的应用程序应当使用这两个函数

```cpp
#include <stdio.h>

long ftell(FILE* fp);

若成功，返回当前文件位置指示；若出错，返回-1L

int fseek(FILE* fp, long offset, int whence);  // whence与lseek的参数相同

若成功，返回0；若出错，返回-1

void rewind(FILE* fp);
```

可以使用rewind将流设置到文件的起始位置

除了偏移类型是off_t外，ftello和fseeko函数与上述函数相同

```cpp
#include <stdio.h>

int fgetpos(FILE* restrict fp, fpos_t* restrict pos);
int fsetpos(FILE* fp, const fpos_t* pos);

若成功，返回0；若出错，返回非0
```

## 格式化IO

格式化IO有scanf和printf函数族来实现，同时也提供了vscanf和vprintf函数族来处理可变参数

有关格式化IO的详细信息，这里不过多介绍，需要时可以参考手册或书籍

## 标准IO库的文件描述符

fileno可以获取对应流关联的文件描述符，其不是ISO C标准，不可移植

```cpp
#include <stdio.h>

int fileno(FILE* fp);

返回与流相关联的文件描述符
```

## 临时文件

ISO C标准库提供了两个函数来帮助创建临时文件

```cpp
#include <stdio.h>

char* tmpnam(char* ptr);

返回指向唯一路径名的指针

FILE* tmpfile(void);

若成功，返回文件指针；若出错，返回NULL
```

tmpnam函数产生一个与现有文件名不通的有效路径名字符串，最多调动TMP_MAX次。若ptr是NULL，则所产生的路径名存放在静态存储区中，指向该静态区的指针作为返回值返回。若ptr不为空，则长度至少为L_tmpnam，并返回ptr

tmpfile创建一个临时二进制文件（wb+），在关闭该文件或程序结束时将自动删除这中文件。其原理一般是调用tmpnam函数产生一个唯一路径名，然后用该路径名创建一个文件，并立即unlink它

另外mkdtemp和mkstemp也可以创建临时目录或文件。

一般应使用mkstemp（文件不会自动删除）和tmpfile，因为使用tmpnam和tempnam至少有一个缺点：因为在调用其获取唯一文件名和创建文件之间，可能有其他进程使用相同名字创建文件，而mkstemp和tmpfile不存在这个问题

## 内存流

内存流一般不会用到，不做介绍

## 标准IO库的替代软件

标准IO库有一些缺点。效率不高，通常需要复制两次数据：内核到标准IO库缓冲区、标准IO库缓冲区到用户程序缓冲区。

mmap函数也可以用来读写文件

# 第六章 系统数据文件和信息

UNIX系统有很多与系统有关的数据文件，由于历史原因这些数据文件都是ASCII文本文件，本章简单介绍这些文件。除此之外还介绍系统标识函数、时间和日期函数

## 口令文件

UNIX系统口令文件包含如下字段，其定义在<pwd.h>中的passwd结构中

![passwd文件中的字段](https://gwq5210.com/images/passwd文件中的字段.png)

可以通过getpwuid和getpwunam获取uid或name，同时也提供了函数来读取/etc/passwd文件，具体可查看文档

目前，加密口令一般存放在阴影口令文件中，一般为/etc/shadow，可以通过函数来读取加密口令

组文件一般为/etc/group。UNXI系统一般支持附属组，一个用户可以加入多个附属组中，用户附属组可以通过函数获取

## 登录账户记录

大多数UNIX会提供utmp——记录当前登录到系统的各个用户，wtmp——跟踪各个登录和注销事件，其一般为二进制文件，存放相应的结构体

## 系统标识

可以通过uname获取系统标识，一般包括如下字段

```cpp
struct utsname {
  char sysname[];  // name of the operating system
  char nodename[];  // name of the node
  char release[];  // current release of operating system
  char version[];  // current version of release
  char machine[];  // name of hardware type
};
```

历史上，BSD派生的系统提供gethostname函数，它只返回主机名，改名字通常就是TCP/IP网络上主机的名字

## 时间和日期例程

time函数返回自协调世界时公元1970年1月1日 00:00:00这一特定时间以来经过的秒数

clock_gettime函数可用于获取指定时钟时间，clock_getres用于获取指定时钟精度，clock_settime对特定的时钟设置时间

gettimeofday可以获取微秒级的时间

其余函数可以将秒数转换为tm结构体，如下

```cpp
struct tm {
  int tm_sec;                   /* Seconds.     [0-60] (1 leap second) */
  int tm_min;                   /* Minutes.     [0-59] */
  int tm_hour;                  /* Hours.       [0-23] */
  int tm_mday;                  /* Day.         [1-31] */
  int tm_mon;                   /* Month.       [0-11] */
  int tm_year;                  /* Year - 1900.  */
  int tm_wday;                  /* Day of week. [0-6] */
  int tm_yday;                  /* Days in year.[0-365] */
  int tm_isdst;                 /* DST.         [-1/0/1]*/
};
```

![时间函数之间的关系](https://gwq5210.com/images/时间函数之间的关系.png)

# 进程环境

## main函数

C程序总是从main函数开始执行，其原型如下

```cpp
// argc为命令行参数的个数
// argv是指向参数的各个指针构成的数组
int main(int argc, char* argv[]);
```

内核（使用exec函数，后续将会介绍）执行C程序时，在调用main函数前会先调用一个特殊的启动例程。可执行程序文件将此启动例程指定为程序的起始地——这是由链接编辑器指定的，而链接编辑器由C编译器调用。启动例程从内核取得命令行参数和环境变量值，然后为按照上述方式调用main函数做好准备

## 进程终止

有八种方式终止进程（termination），前5种为正常终止，后3种为异常终止

- 从main函数返回
- 调用exit
- 调用_exit或_Exit
- 最后一个线程从启动例程返回
- 从最后一个线程调用pthread_exit
- 调用abort
- 接到一个信号
- 最后一个线程对取消请求做出响应

启动例程一般是从main函数返回后立即调用main函数

```cpp
// 语义为如下C语言调用，但启动例程一般用汇编语言编写
exit(main(argc, argv));
```

### 退出函数

_exit和_Exit立即进入内核，exit则先执行一些清理，然后返回内核

```cpp
#include <stdlib.h>
void exit(int status);
void _Exit(int status);

#include <unistd.h>

void _exit(int status);
```

exit函数总是执行一个标准IO库的相关清理工作：对于所有打开流调用fclose函数

3个函数都带一个整型参数，称为终止状态（exit status）。大多数UNIX系统shell都提供检查进程终止状态的方法（echo $?）

main函数推荐声明为int，并且需要返回整型值，否则，可能终止状态是未定义的

main函数返回一个整型值与用该值调用exit是等价的，即exit(0)等价于return 0

### 函数atexit

ISO C规定一个进程可以登记多至32个函数，这些函数将由exit自动调用，这些函数称为终止处理程序（exit handler），使用atexit来登记这些函数

```cpp
#include <stdio.h>

int atexit(void (*func)(void));

若成功，返回0；若出错，返回非0
```

其参数是一个函数地址，其无参数也无返回值。exit调用这些函数的顺序与它们登记时候的顺序相反。同一函数若登记多次也会被调用多次

exit首先调用各种终止处理程序，然后关闭所有打开的流。若程序调用exec函数族的任一函数，则将清除所有已安装的终止处理程序

![程序的启动和终止](https://gwq5210.com/images/程序的启动和终止.png)

内核使程序执行的唯一方法是调用一个exec函数。进程自愿终止的唯一方法是显式或隐式的（通过调用exit）调用_exit或_Exit。进程也可非自愿的由一个信号使其终止

## 命令行参数
