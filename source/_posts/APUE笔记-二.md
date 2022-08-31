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

![setbuf和setvbuf函数](https://gwq5210.github.io/images/setbuf和setvbuf函数.png)

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

![fopen的type](https://gwq5210.github.io/images/fopen的type.png)

使用字符b可以使得标准IO系统区分二进制和文本文件。UNIX内核并不区分这两种文件，所以在UNIX系统下并无作用

对于fdopen，type参数的作用稍有区别。因为该描述符已被打开，所有fdopen为写打开并不截断文件。标准IO追加写方式也不能用于创建该文件（该文件一定存在，fd引用一个文件）

当以读和写类型（type中+号）打开一个文件时，有如下限制

- 如果中间没有fflush、fseek、fsetpos或rewind，则在输出后边不能直接跟随输入
- 如果中间没有fseek、fsetpos或rewind或者一个输入操作没有到达文件尾端，则在输入操作之后不能直接跟随输出

![打开流的不同方式](https://gwq5210.github.io/images/打开流的不同方式.png)

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

![标准IO库的效率](https://gwq5210.github.io/images/标准IO库的效率.png)

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

![passwd文件中的字段](https://gwq5210.github.io/images/passwd文件中的字段.png)

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

![时间函数之间的关系](https://gwq5210.github.io/images/时间函数之间的关系.png)

# 第七章 进程环境

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

![程序的启动和终止](https://gwq5210.github.io/images/程序的启动和终止.png)

内核使程序执行的唯一方法是调用一个exec函数。进程自愿终止的唯一方法是显式或隐式的（通过调用exit）调用_exit或_Exit。进程也可非自愿的由一个信号使其终止

## 命令行参数

执行一个程序时，调用exec的进程可以将命令行参数传递给该新程序。这是UNIX shell的一部分常规操作。参数`argv[argc]`被要求为NULL

## 环境表

每个程序都接收到一张环境表，与参数一样环境表也是一个字符串指针数组，其中每个指针包含一个以null字节结尾的C字符串地址。全局变量environ包含了该指针数组的地址

```cpp
extern char** environ;
```

environ称为环境指针（environment pointer），指针数组称为环境表，各指针指向的字符串称为环境字符串

环境字符串的格式为

```cpp
name=value
```

![环境指针](https://gwq5210.github.io/images/环境指针.png)

大多数环境预定义名全部由大写字母组成，这只是一个惯例。历史上UNIX系统支持main函数带3个参数，第三个参数是环境表指针。但通常我们应使用environ全局变量，而不是使用第3个参数。更好的方法是getenv和putenv函数，但如果要查看整个环境，则必须访问environ全局变量

## C程序的存储空间布局

历史沿袭至今，C程序一直由以下部分组成

- 正文段。这是由CPU执行的机器指令部分。通常正文段是共享的和只读的（防止程序由于意外而修改其指令）
- 初始化数据段。通常将此段称为数据段，它包含了程序中需明确的赋初值的变量（已初始化的全局变量和静态变量）。如C程序中任何函数外的声明int maxcount = 99;则此变量和初值存放在初始化数据段中
- 未初始化数据段。通常将此段称为bss段，在程序开始执行之前，内核将此段中的数据初始化为0或空指针。包含未初始化的全局变量和静态变量
- 栈。自动变量以及每次函数调用时所需保存的信息都存放在此段中。每次函数调用时，其返回地址以及调用者的环境信息都存放在栈中。然后最近被调用的函数在栈上为其自动变量和临时变量分配空间。通过此方式使用栈，C递归函数可以工作。一次函数的调用产生一个新的栈帧，因此一次函数的调用不会影响另一次函数调用实例中的变量
- 堆。通常在堆中进行动态内存分配。由于历史上形成的惯例，堆位于未初始化段和栈之间

![典型的存储空间安排](https://gwq5210.github.io/images/典型的存储空间安排.png)

程序中还有其他的段，如包含符号表的段、包含调试信息的段，包含动态共享库链接表的段，这些部分并不装载到进程执行的程序映像中

未初始化段的内容并不存放在磁盘程序文件中，其原因是内核在程序开始运行前都将它们设置为0，需要存放在磁盘文件中的段只有正文段和初始化数据段

size(1)命令可以报告正文段、数据段和bss段的以字节为单位的长度

## 共享库

共享库使得可执行文件不再需要包含共用的函数库，而只需要在所有进程都可引用的存储区保存这种库例程的一个副本。

程序第一次执行或者第一次调用某个函数库时，用动态链接方法将程序与共享库函数相链接。这减少了每个可执行文件的长度，但增加了一些运行时间开销。这种时间开销发生在该程序第一次被执行时，或者每个共享库函数第一次被调用时。

共享库的另一个优点是可以用库函数的新版本代替老版本而无需对使用该库的程序重新链接编辑（假定参数的数目和类型都没有发生改变）。

gcc可使用-static阻止使用共享库

## 存储空间分配

ISO C说明了3个可用于存储空间动态分配的函数

- malloc，分配指定字节数的存储区。此存储区中的初始值不确定
- calloc，为指定数量指定长度的对象分配存储空间。该空间中的每一个bit都初始化为0
- realloc，增加或减少以前分配区的长度。当增加长度时，可能需要将以前分配区的内容移动到另一个足够大的区域，以便在尾端提供增加的存储区，而新增加区域内的初始值则不确定

```cpp
#include <stdlib.h>

void* malloc(size_t size);
void* calloc(size_t nobj, size_t size);
void* recalloc(void* ptr, size_t newsize);

若成功，返回非空指针；若出错，返回NULL

void free(void* ptr);
```

这3个分配函数所返回的指针一定是适当对齐的，使其可用于任何数据对象

realloc的最后一个参数是存储区的新长度，不是新旧存储区长度之差。作为一个特例，如果ptr是一个空指针，则realloc的功能与malloc相同，用于分配一个指定长度为newsize的存储区

这些分配例程通常用sbrk(2)系统调用实现，该系统调用扩充或缩小进程的堆。虽然sbrk可以扩充或缩小进程的存储空间，但是大多数malloc和free实现都不减少进程的存储空间。释放的空间可供以后再分配，但将它们保持在malloc池中而不返回给内核

大多数实现所分配的存储空间比所要求的的要稍大一些，额外的空间用来记录管理信息——分配块的长度、指向下一个分配块的指针等。这就意味着，如果超过一个已分配区的尾端或者在已分配区的起始位置之前进行写操作，则会改写另一块的管理记录信息。这种类型的错误是灾难性的，但是这种错误不会很快暴露出来，所以也就很难发现

常见的错误是：释放一个已经释放的块；调用free时所用的指针不是3个alloc函数返回的指针

常见的代替分配方法为jemalloc和tcmalloc

## 环境变量

UNIX内核并不查看环境变量字符串，它们的解释完全取决于各个应用程序。

getenv可以取环境变量值

```cpp
#include <stdlib.h>

char* getenv(const char* name);

返回指向与name关联的value的指针；若未找到，返回NULL
```

此函数返回一个指针，它指向`name=value`字符串中的value

我们也希望可以改变现有变量的值，或者是增加新的环境变量。我们能影响的只是当前进程及其后生成和调用的任何子进程的环境，但不能影响父进程的环境，这通常是一个shell进程

```cpp
#include <stdlib.h>

int putenv(char* str);

若成功，返回0；若出错，返回非0

int setenv(const char* name, const char* value, int rewrite);
int unsetenv(const char*name);

若成功，返回0；若出错，返回-1
```

- putenv取形式为`name=value`的字符串，将其放到环境表中，如果name已经存在，则先删除原来的定义。其参数str指针直接放到环境变量中
- setenv将name设置为value。若环境中name已经存在，那么a）若rewrite非0，则首先删除其现有定义；b)若rewrite为0，则不删除其现有定义（name不设置为新的value，而且也不出错）。与putenv不同setenv必须分配存储空间
- unsetenv删除name的定义，即使不存在这种定义也不算出错

## 函数setjmp和longjmp

在C中，goto语句是不能跨越函数的，而执行这种类型跳转功能的函数是setjmp和longjmp。这两个函数对于处理发生在很深层嵌套函数调用中的出错情况是非常有用的

如果对嵌套层次很深的错误进行判断，逐层返回，会比较麻烦。可以使用setjmp和longjmp来实现非局部跳转。其不是由普通C语言goto语句在一个函数内实施的跳转，而是在栈上跳过若干调用帧，返回到当前函数调用路径上的某一个函数中

```cpp
#include <setjmp.h>

int setjmp(jmp_buf env);

若直接调用，返回0；若从longjmp返回，则为非0

void longjmp(jmp_buf env, int val);
```

在希望返回到的位置调用setjmp。jmp_buf是某种形式的数组，其中存放在调用longjmp时能用来恢复栈状态的所有信息。

longjmp参数val将成为setjmp处返回的值，因为一个setjmp可以有多处longjmp，通过val值就可以进行区分

当longjmp发生跳转时，自动变量和寄存器变量的值会如何变化，这是不确定的，大多数实现并不回滚这些自动变量和寄存器变量的值，而所有的标准则称它们的值是不确定的。如果你有一个自动变量，而又不想使其回滚，则可以定义其为volatile属性。声明为全局变量、静态变量的值在执行longjmp时保持不变。

## 函数getrlimit和setrlimit

每个进程都有一组资源限制，其中一些是可以用getrlimit和setrlimit函数查询和更改

```cpp
#include <sys//resource.h>

int getrlimit(int resource, struct rlimit* rlptr);
int setrlimit(int resource, const struct rlimit* rlptr);

若成功，返回0；若出错，返回非0
```

这两个函数的每一次调用都指定一个资源以及一个指向下列结构的指针

```cpp
struct rlimit {
  rlim_t rlim_cur;  // soft limit: current limit
  rlim_t rlim_max;  // hard limit: maximum value for rlim_cur
};
```

更改资源时，需要遵循下列规则

- 任何一个进程都可以将一个软限制更改为小于或等于其硬限制值
- 任何一个进程都可以降低其硬限制值，但它必须大于或等于其软限制值。这种降低，对普通用户而言是不可逆的
- 只有超级用户进程才可以提高硬限制值

常量RLIM_INFINITY指定一个无限量的值

![对资源限制的支持](https://gwq5210.github.io/images/对资源限制的支持.png)

资源限制影响到调用进程并由其子进程继承。shell具有内置ulimit命令，用于修改限制值