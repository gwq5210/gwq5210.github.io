title: APUE笔记(一)
date: 2017-07-12 01:22:08
tags:
---
# UNIX基础知识
所有的操作系统都为它们所运行的程序提供服务。典型的服务包括：执行新程序、打开文件、读文件、分配存储区以及获得当前时间等。

从严格意义上说，可以将操作系统定义为一种软件，它控制计算机硬件资源，提供程序运行环境。从广义上说，操作系统包含了内核和一些其他软件，这些软件使得计算机能够发挥作用，并使计算机具有自己的特性。

我们通常将这种软件成为内核（kernel），因为它相对较小，而且位于环境的核心。

内核的接口称之为系统调用（system call）。公共函数库构建在系统调用接口之上，应用程序既可使用公共函数库，也可以使用系统调用。shell是一个特殊的应用程序， 为运行其他应用程序提供了一个接口。

系统的口令文件由7个以冒号分割的字段组成，依次是：登录名、加密口令、数字用户ID、数字组ID、注释字段、起始目录以及shell程序。目前所有已知的系统已将加密口令移到另一个文件中。

shell是一个命令解释器，它读取用户输入，然后执行命令。

<!--more-->

UNIX文件系统是目录和文件的一种层次结构，所有的目录或文件以根（root，/）为起点。

目录是一个包含目录项的文件。每个目录项都包含文件名和文件属性。目录中的各个名字称为文件名。斜线（分割路径）和空字符（标记路径名结尾）不能出现在文件名中。创建新目录会自动创建点和点点目录，分别表示当前目录和上一级目录。根目录中，点点和点相同。

以斜线开头的路径名成为绝对路径，否则称为相对路径。

每个进程都有工作目录（working directory），有时称其为当前工作目录（current working directory）。所有相对路径都从工作目录开始解释。登录时，工作目录设置为起始目录（home direcotry）。

文件描述符（file descriptor）通常是一个小的非负整数，内核用于标识一个特定进程正在访问的文件。

当运行一个新程序时，所有的shell都为其打开3个文件描述符，即标准输入，标准输出，标准错误。

函数open，write，read，lseek和close提供了不带缓冲的IO。标准IO函数为那些不带缓冲的函数提供了一个带缓冲的接口，我们无需担心如何选取最佳的缓冲区大小。

程序（program）是一个存储在磁盘上某个目录中的可执行文件。程序的执行实例被称为进程（process）。UNIX系统确保每个进程都有一个唯一的数字标识符，称为进程ID，它是一个非负整数。

通常一个进程只有一个线程（thread）。但进程中可以有多个线程。一个进程内的所有线程共享同一地址空间，文件描述符，栈及进程相关属性。线程也使用线程ID标识。

UNIX系统函数出错时，函数通常返回一个负值， 而且整型变量errno通常被设置为具有特定信息的值。在支持线程的环境中，多个线程共享进程地址空间，每个线程都有属于它自己的局部errno。

使用errno需要注意，如果没有出错，其值不会被任何例程清除。因此仅当函数的返回值指明出错时，才检验其值。任何函数都不会将errno值设置为0。

errno定义的错误可以分为两类，致命的和非致命的。对于致命的错误无法进行恢复动作。对于非致命的错误，有时可以进行妥善的处理。与资源相关的非致命错误包括：EAGAIN、ENFILE、ENOBUFS、ENOLCK、ENOSPC、EWOULDBLOCK，有时ENOMEM也是非致命错误。当EBUSY指明共享资源正在使用时，也可将它作为非致命性出错处理。当EINTR中断一个慢速系统调用时，可将它作为非致命性错误处理。

口令文件登录项中的用户ID是一个数值，用来标识各个不同的用户。用户ID为0的用户为根用户或超级用户。我们称超级用户的特权为超级用户特权。

组被用于将若干用户集合到一起，便于共享资源。我们用组ID来标识用户组。组文件为/etc/group，它将组ID映射为组名。

使用数值的用户ID和组ID是历史原因，它们更节省磁盘空间，在权限校验时，更加节省资源。

多数UNIX系统还允许用户属于另外一个组，我们称之为附属组（supplementary group）。

信号（signal）用来通知进程发生了某种情况。
对于信号，进程有三种方式处理：
* 忽略信号
* 按照系统默认方式处理
* 提供一个函数，信号发生时调用该函数，这被称为捕捉该信号

在进程中，我们可以调用kill函数向另一个进程发送信号，前提是我们必须是这个进程的所有者或者超级用户。

UNIX使用两种不同的时间值。一是日历时间，该值是自协调世界时（Coordinated Universal Time，UTC）1970年1月1日00:00:00这个特定时间以来所经历的秒数累计值。系统基本数据类型使用time_t保存这个时间。二是进程时间。也被称为CPU时间，用于度量进程使用的中央处理器资源。进程时间以时钟时间为单位。系统基本数据类型clock_t保存这种时间值。我们可以通过sysconf函数得到每秒的时钟滴答数。
UNIX为一个进程维护了三个进程时间：
* 时钟时间，是进程运行的时间总量，其值与系统同时运行的进程数有关。
* 用户CPU时间，是执行用户指令所用的时间量。
* 系统CPU时间，是为该进程执行内核程序所经历的时间。

我们可以简单的使用命令time获取程序的执行时间。

所有的操作系统都提供多种服务的入口点，由此程序向内核请求服务。各种版本的UNIX实现都提供良好定义、数量有限、直接进入内核的入口点，这些入口点被称为系统调用。系统还定义了通用库函数，这些函数可能会使用一个或多个系统调用，但是它们并不是内核的入口点。

从实现者的角度，系统调用和库函数之间有根本的区别，但从用户角度看，其区别并不重要。我们应该理解，如果希望的话，我们可以替换库函数，但是系统调用通常是不能被替换的。

应用程序既可以调用系统调用也可以调用库函数。很多库函数则会调用系统调用。系统调用通常提供一种最小接口，而库函数通常提供比较复杂的功能。

# UNIX标准及实现
人们在UNIX环境编程和C程序设计语言的标准化方面做已经做了很多工作。

UNIX系统实现定了了很多幻数和常量。其中很多已经被硬编码到程序中，或用特定的技术确定。
以下两种类型的限制是必须的：
* 编译时限制，可以在头文件中定义，程序在编译时可以包含这些头文件
* 运行时限制，要求程序调用一个函数获得限制值

某些限制在给定的实现中是确定的，但在另一个实现中可能是变动的。为了解决这类问题，提供了如下三种限制：
* 编译时限制(头文件)
* 与文件或目录无关的运行时限制(sysconf函数)
* 与文件或目录有关的运行时限制(pathconf或fpathconf函数)

ISO C定义的所有编译时限制都在头文件<limits.h>中。这些限制在一个给定的系统中并不会改变。我们将会遇到的一个问题是系统是否提供带符号或无符号的字符值，可以通过其中定义的值确定。另一个ISO C的常量是FOPEN_MAX，这是具体实现保证可同时打开的标准IO流的最小个数，其在头文件<stdio.h>中定义，最小值是8。POSIX.1中的STREAM_MAX则应与其相同。头文件stdio.h中还定义了TMP_MAX，这是由tmpnam函数产生的唯一文件名的最大个数。在ISO C中虽然定义了常量FILENAME_MAX，但我们应该避免使用。POSIX.1提供了更好的代替常量NAME_MAX和PATH_MAX。

POSIX可以分为7类：
* 数值限制：LONG_BIT，SSIZE_MAX，WORD_BIT 
* 最小值
* 最大值，\_POSIX_CLOCKRES_MIN
* 运行时可以增加的值：CHARCLASS_NAME_MAX，COLL_WEIGHTS_MAX，LINE_MAX，NGROUPS_MAX，RE_DUP_MAX
* 运行时不变值
* 其他不变值
* 路径名可变值：FILESIZEBITS，LINK_MAX，MAX_CANON，MAX_INPUT,MAX_INPUT，NAME_MAX，PATH_MAX，PIPE_BUF，SYMLINK_MAX

最小值是不变的，它们不随系统改变。它们指定了这些特征最具约束性的值。一个符合POSIX.1的实现应当至少提供这样的值。为了保证移植性，严格符合标准的应用程序不应要求更大的值

不过某些最小值在实际中太小了。

运行时限制可以通过下面3个函数获得：
```
#include <unistd.h>
long sysconf(int name);
long pathconf(const char *pathname, int name);
long fpathconf(int fd, int name);
```
我们讨论这三个函数不同的返回值：
* 如果name参数并不是一个合适的常量，这三个函数都返回-1，并把errno设置为EINVAL
* 有些name返回一个变量值(大于等于0)，或者提示该值是不确定的，不确定的值通过返回-1来体现，但不改变errno的值
* \_SC_CLK_TCK返回值是每秒的时钟滴答数，用于times函数的返回值

对于后两个函数的第一个参数有很多限制。可以通过手册查询。

不确定的运行时限制：
* 路径名，可以通过PATH_MAX来获得，但是结果可能是不确定的
* 最大打开文件数，守护进程中一个常见的代码就是关闭所有打开的文件
```
#include <sys/param.h>
for (int i = 0; i < NOFILE; ++i)
{
  close(i);
}
```
但这种方式是不可移植的，所以可以通过sysconf来获取，尽可能提高移植性

如果我们需要编写可以移植的程序，这些程序可能依赖一些可选的支持的功能，我们需要一个可移植的方法来确定实现是否支持一个给定的选项

有三种处理选项的方法：
* 编译时选项定义在unistd.h中
* 与文件或目录无关的选项使用sysconf确定
* 与文件或目录有关的选项使用pathconf或fpathconf确定

对于每一个选项，有三种可能的平台支持状态。
* 如果符号常量没有定义或者定义值为-1，那么该平台在编译时不支持相应选项。但是有一种可能，在已支持该选项的新系统上运行老应用时，即使该选项在应用编译时未被支持，但如今新系统运行时检查会显示该选项已经支持
* 如果符号常量的定义值大于0，那么支持该选项
* 如果符号常量的值为0，则必须调用三个函数来确定是否支持相应的选项

如果在编译程序时，希望它只与POSIX的定义有关，而不与任何实现定义的常量冲突，那么需要定义常量\_POSIX_C_SOURCE，一旦定义了该常量，所有POSIX.1的头文件都是用此常量来排除任何实现专有的定义。可以在编译时指定该常量

头文件sys/types.h中定义了某些与实现有关的数据类型，它们被称为基本系统数据类型(primitive system data type)，它们使用C语言的typedef定义，大多使用\_t结尾，用这种方式定义了这些数据类型时，就不需要考虑因系统不同而变化的程序实现细节。

| 类型           | 说明              |
| ------------ | --------------- |
| clock_t      | 时钟滴答计数器         |
| comp_t       | 压缩的时钟滴答         |
| dev_t        | 设备号，主和次         |
| fd_set       | 文件描述符集          |
| fpos_t       | 文件位置            |
| gid_t        | 数值组ID           |
| ino_t        | i节点编号           |
| mode_t       | 文件类型，文件创建模式     |
| nlink_t      | 目录项的链接计数        |
| off_t        | 文件长度和偏移量，带符号的   |
| pid_t        | 进程ID和进程组ID，带符号的 |
| pthread_t    | 线程ID            |
| ptrdiff_t    | 两个指针相减的结果       |
| rlim_t       | 资源限制            |
| sig_atomic_t | 能原子性访问的数据类型     |
| sigset_t     | 信号集             |
| size_t       | 对象长度，如字符串，不带符号的 |
| ssize_t      | 返回字节计数器的函数，带符号的 |
| time_t       | 日历时间的秒计数器       |
| uid_t        | 数值用户ID          |
| wchar_t      | 能表示所有不同的字符码     |

# 文件IO
UNIX系统大多数的文件IO只需要5个函数：open，read，write，lseek，close。对于read和write不同缓冲区对读取性能有一定影响。这些函数常被称为不带缓冲的IO。属于不带缓冲指的是每个read和write都调用内核中的一个系统调用。如果在多个进程之间共享资源，我们需要了解原子操作。

对内核而言，所有打开的文件都通过文件描述符引用。文件描述符是一个非负整数。

## open函数
调用open或openat可以打开或创建一个文件：
```
#include <fcntl.h>

int open(const char *path, int oflag, ... /* mode_t mode */);
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */);
```
path参数时要打开或创建的文件的名字。oflag参数可用来说明此函数的多个选项。用下面一个或多个常量进行或来构成oflag参数。
* O_RDONLY，只读打开
* O_WRONLY，只写打开
* O_RDWR，读写打开
* O_EXEC，只执行打开
* O_SEARCH，只搜索打开(应用于目录)

以上5个常量中必须指定一个且只能指定一个。下列常量是可选的：
* O_APPEND，每次写都追加到文件的结尾
* O_CLOEXEC，把FD_CLOEXEC常量设置为文件描述符标志。
* O_CREAT，若此文件不存在则创建它。使用此选项open或openat函数需同时说明mode参数，用mode指定该新文件的访问权限位。
* O_DIRECTORY，如果path引用的不是目录，则出错。
* O_EXCL，如果同时指定了O_CREAT，而文件已经存在，则出错。用此可以测试一个文件是否存在，如果不存在则创建文件，这使得测试和创建两者成为一个原子操作。
* O_NOCTTY，如果path引用的是终端设备，则不将该设备分配作为此进程的控制终端。
* O_NOFOLLOW，如果path引用的是一个符号链接，则出错。
* O_NOBLOCK，如果path引用的是一个FIFO，一个块特殊文件或一个字符特殊文件，则此选项为文件的本次打开操作和后续的IO操作设置非阻塞方式。
* O_SYNC，使每次write等待物理IO操作完成，包括由该write操作引起的文件属性更新所需的IO。
* O_TRUNC，如果此文件存在，而且为只写或读写成功打开文件，则将其长度截断为0。
* O_TTY_INIT，如果打开一个还未打开的终端设备，设置非标准的termios参数值，使其符合single UNIX Specification。
* O_DSYNC，使每次write要等待物理IO操作完成，但是如果该写操作并不影响读取刚写的数据，则不需要等待文件属性被更新。文件属性包括文件大小等信息。
* O_RSYNC，使每一个以文件描述符作为参数进行的read操作等待，直至所有对文件同一部分挂起的写操作都完成。

对于每一个选项，具体可以阅读man手册。

由open或openat函数返回的文件描述符一定是最小的未被使用的描述符的值。

fd参数把open和openat参数区分开：
* path参数指定的是绝对路径名，在这种情况下，fd参数被忽略，openat函数相当于open函数。
* path参数指定的是相对路径名，fd参数指出了相对路径名在文件系统中的开始地址。fd参数时通过打开相对路径名所在的目录来获取。
* path参数指定了相对路径名，fd参数具有特殊值AT_FDCWD。在这种情况下，路径名在当前工作目录获取，open函数在操作上与open函数类似。

openat函数希望解决两个问题，一是让线程可以使用相对路径名打开目录中的文件，而不再只能打开当前工作目录。二是可以避免time-of-check-to-time-of-use(TOCTTOU)错误，其基本思想是，如果有两个基于文件的函数调用，其中第二个调用依赖于第一个调用的结果，那么程序是脆弱的，因为两个调用并不是原子操作，在两个函数调用之间的文件可能改变了，这样也就造成了第一个调用的结果不再有效，使得最终程序的结果是错误的。

使用open需要注意文件名截断。现在大多数系统支持的文件名长度可以为255。

## creat函数
另外，creat函数也可以创建一个文件：
```
#include <fcntl.h>

int creat(const char *path, mode_t mode);
// 等效于
open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);
```
它只以写的方式打开所创建的文件

## close函数
函数close关闭一个打开的文件：
```
#include <unistd.h>

int close(fd);
```
关闭一个文件时，还会释放该进程加在该文件上的所有记录锁。当一个进程终止，内核自动关闭它所有的打开文件。

## lseek函数
每个打开的文件都有一个与其相关联的当前文件偏移量(current file offset)。它通常是一个非负整数，用于度量从文件开始处计算的字节数。通常读写操作都从当前文件便宜量开始，并使偏移量增加所读写的字节数。按照系统默认情况，当打开一个文件时，除非指定O_APPEND选项，否则该偏移量被设置为0。

lseek可以显式的设置偏移量：
```
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```
对offset参数的解释与whence的值有关：
* 若whence是SEEK_SET，则将该文件的偏移量设置为距文件开始处offset个字节
* 若whence是SEEK_CUR，则将该文件偏移量设置为其当前值加上offset，offset可正可负
* 若whence是SEEK_END，则该文件偏移量设置为文件长度加上offset，offset可正可负

lseek成功执行返回新的文件偏移量：
```
off_t curr_pos;
curr_pos = lseek(fd, 0, SEEK_CUR);
```
这种方法可以确定打开文件的当前偏移量，也可以来确定涉及的文件是否可以设置偏移量。如果文件描述符指向的是一个管道，FIFO，或网络套接字，则lseek返回-1，并将errno设置为ESPIPE。

通常文件的当前偏移量应该是一个非负整数，但是某些设备也可能允许负的偏移量，但对于普通文件，必须是非负值。在比较lseek返回值时应该注意，不要测试它是否小于0，而要测试它是否等于-1。

lseek仅将当前的文件偏移量记录在内核中，它并不引起任何IO操作，该偏移量用于下一次读或写。

文件的偏移量可以大于当前文件大小，这种情况下，将会引起文件空洞，没被写过的字节都被读为0，文件中的空洞并不要求在磁盘上占用存储区，具体与实现有关。

lseek使用的偏移量用off_t类型表示，允许具体实现根据各自的平台自行选择合适的数据类型。

## read函数
read函数从打开的文件中读取数据：
```
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t nbytes);
```
如果read成功，返回读取到的字节数。如已到达文件末端，返回0。
多种情况使得实际读取到的字节数少于要求读取的字节数：
* 读取普通文件时，在读到要求字节数之前已经达到了文件尾端。
* 当从终端设备读时，通常依次最多读取一行。
* 当从网络读时，网络中的缓冲机制可能造成返回值小于所要求读的字节数。
* 当从管道或FIFO读时，如果管道包含的字节少于所需的数量，那么read将返回实际可用的字节数。
* 当从某些面向记录的设备(如磁带)读时，一次最多返回一个记录
* 当一信号造成中断，而已经读了部分数量时。

读操作从当前文件偏移量开始，在成功返回之前，该偏移量将增加实际读到的字节数。

## write函数
write函数向打开的文件写入数据。
```
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t nbytes);
```
返回值通常与nbytes相同，否则表示出错，write出错的一个常见错误是磁盘已经写满，或者超过了一个给定进程的文件长度限制。

对于普通文件，写操作从文件的当前偏移量处开始。如果在打开文件时指定了O_APPEND选项，则在每次写操作前，将文件的偏移量设置在文件的结尾处，因此这种方式不能写中间的某个部分。在一次写成功后，文件偏移量增加实际写入的字节数。

## IO的效率
read函数和write的函数选出合适的buf大小是很重要的。一般来说在4096字节(磁盘块长度)效率比较高。大多数文件系统为改善性能都采取某种预读技术。检测到正在顺序读取时，系统就试图读取比应用所要求更多的数据，并假想应用很快就会读这些数据。

## 文件共享
UNIX支持在不同进程间共享打开文件。

内核使用三种数据结构表示打开文件，它们之间的关系决定了在文件共享方面一个进程对另一个进程可能产生的影响。
* 每个进程在进程表项中都有一个记录项，记录项包含一张打开文件描述符表，可将其视为一个矢量，每个描述符占用一项。与文件描述符相关联的是文件描述符标志(close_on_exec)和指向一个文件表项的指针。
* 内核为所有打开文件维持一张文件表。每个文件表项包含：文件状态标志(读，写，添写，同步和非阻塞等)、当前文件偏移量、指向该文件的v节点指针。
* 每个打开文件(或设备)都有一个v节点结构。v节点包含了文件类型和对此文件进行各种操作的函数指针。对于大多数文件，v节点还包含了该文件的i节点(索引节点)。这些信息是在打开文件时从磁盘读入内存的。对于不同的实现可能不同，例如linux不包含v节点，而是使用了通用的i节点。

不同实现可能有不同，但是是有必要保存这些信息。
![打开文件的内核数据结构](http://blogfiles-10055310.cossh.myqcloud.com/1.png?sign=ySOhpUdnj/w4D1X/x8Nmuwi8Ij1hPTEwMDU1MzEwJms9QUtJRDZHMTR1emhxOFlRYUIwOFBCZ204OFc5WVdNeHBrcG4zJmU9MTUwMjc5NDA1NSZ0PTE1MDAyMDIwNTUmcj0xODY0NTczNDE4JmY9LzEucG5nJmI9YmxvZ2ZpbGVz)

文件描述符标志和文件状态标志在作用范围方面有区别，前者只用于一个进程的某一个描述符，而后者则应用于指向该给定文件表项的任何进程中所有的描述符。

## 原子操作
一般而言，原子操作(atomic operation)是由多步操作组成的一个操作。如果该操作原子地执行，则要么执行完所有步骤，要么一步也不执行，不可能只执行所有步骤的一个子集。

多个进程对同一个文件追加数据，可以在打开文件时指定O_APPEND标志。这使得每次写操作之前内核都将进程的当前文件偏移量设置到该文件的末尾。对于对于这种write操作在不同进程间是不是原子的，后续有文章来验证。

pread和pwrite允许原子性的定位并执行IO。
```
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
size_t pwrite(int fd, const void *buf, size_t nbytes, off_t offset);
```
调用pread相当于调用lseek后调用read，但是pread又与这种顺序调用有如下重要区别：
* 调用pread时，无法中断其定位和读操作
* 不更新当前文件偏移量

pwrite也有类似的问题

## 函数dup和dup2
这两个函数可用来复制一个现有的文件描述符：
 ```
 #include <unistd.h>

 int dup(int fd);
 int dup2(int fd, int fd2);
 ```
 由dup返回的新文件描述符一定是当前可用文件描述符中的最小数值。对于dup2，可以用fd2参数指定新描述符的值，如果fd2已经打开，则先将其关闭。如若fd等于fd2，则dup2返回fd2而不关闭它。否则，fd2的FD_CLOEXEC文件描述符就被清除，这样fd2在进程调用exec时是打开状态。

 这些函数返回的新描述符与参数fd共享同一个文件表项。
 ![dup后的内核数据结构](http://blogfiles-10055310.cossh.myqcloud.com/2.png?sign=dygXsTP3Dg9p73JTwXKlTIDJ7hRhPTEwMDU1MzEwJms9QUtJRDZHMTR1emhxOFlRYUIwOFBCZ204OFc5WVdNeHBrcG4zJmU9MTUwMjc5NTg1NiZ0PTE1MDAyMDM4NTYmcj0yMTMyOTUzOTY2JmY9LzIucG5nJmI9YmxvZ2ZpbGVz)

 赋值描述符的另一个方法是使用fcntl函数
 ```
 dup(fd);	// fcntl(fd, F_DUPFD, 0);
 dup2(fd, fd2);	// close(fd2); fcntl(fd, F_DUPFD, fd2);
 ```
 后一种情况，dup2并不完全等同于close加上fcntl
* dup2是一个原子操作
* dup2和fcntl有一些不同的errno

## 函数sync，fsync和fdatasync
传统的UNIX系统实现在内核中设有高速缓冲区高速缓存或页高速缓存，大多数磁盘IO都通过缓冲区进行。当我们向文件写入数据时，内核通常先将数据复制到缓冲中，然后排入队列，晚些时候再写入磁盘。这种方式被称为延迟写

通常，当内核需要重用缓冲区来存放其他磁盘块数据时，它会把所有延迟写数据块写入磁盘。为了保证磁盘实际文件与缓冲区中内容的一致性，UNIX提供了sync，fsync和fdatasync三个函数：
```
#include <unistd.h>

int fsync(int fd);
int fdatasync(int fd);

void sync(void);
```
sync只是将所有修改过的缓冲块排入写队列，然后就返回，它并不等待实际写磁盘结束。通常称为update的系统守护进程周期性的调用(一般30s)sync函数。这就保证了定期冲洗内核的块缓冲区。sync命令也调用sync函数

fsync函数只对由文件描述符fd指定的一个文件起作用，并且等待磁盘操作结束才返回。fdatasync函数类似于fsync，但它只影响文件的数据部分。fsync还会同时更新文件属性的更新。

## 函数fcntl
fcntl可以改变已经打开文件的属性：
```
#include <unistd.h>

int fcntl(int fd, int cmd, ... /* int arg */);
```
fcntl函数有如下5个功能：
* 复制一个已有的文件描述符(F_DUPFD或F_DUPFD_CLOEXEC)
* 获取或设置文件描述符标志(F_GETFD或F_SETFD)
* 获取或设置文件状态标志(F_GETFL或F_SETFL)
* 获取或设置异步IO所有权(F_GETOWN或F_SETOWN)
* 获取或设置记录锁(F_GETLK，F_SETLK或F_SETLKW)

F_DUPFD，复制文件描述符fd。新的描述符作为函数返回值返回。它是尚未打开的各描述符中大于或等于第三个参数值中各值得最小值。
F_DUPFD_CLOEXEC，复制文件描述符，设置文件描述符关联的FD_CLOEXEC文件描述符的值，返回新文件描述符。
F_GETFD，对应于fd的文件描述符标志作为函数值返回，当前只定义了一个文件描述符标志FD_CLOEXEC
F_SETFD，对于fd设置文件描述符标志
F_GETFL，对应于fd的文件状态标志作为函数值返回。5个访问方式标志(O_RDONLY，O_WRONLY，O_RDWR，O_EXEC以及O_SEARCH)并不各占一位，这5个值互斥。因此首先必须使用屏蔽字O_ACCMODE取得访问方式，然后再进行比较。
F_SETFL，将文件状态标志设置为第三个参数的值，可以更改的标志是：O_APPEND，O_NONBLOCK，O_SYNC，O_DSYNC，O_RSYNC，O_FSYNC和O_ASYNC。
F_GETOWN，获取当前接收SIGIO和SIGURG信号的进程ID或进程组ID。
F_SETOWN，设置接收SIGIO和SIGURG信号的进程ID或进程组ID。正的arg指定一个进程ID，负的arg表示等于arg绝对值的一个进程组ID。

fcntl如果出错返回-1，如果成功返回相应的值。

我们必须小心的使用设置标志，我们需要先取出现有的，设置想要设置的标志，然后再进行设置。

## 函数ioctl
ioctl函数一直是IO操作的杂物箱。终端IO是使用ioctl最多的地方。
```
#include <unistd.h>
// #include <sys/ioctl.h>

int ioctl(int fd, int request, ...);
```
不能用本章其他io函数表示的IO操作通常都能用ioctl表示。

## /dev/fd
较新的系统都提供名为/dev/fd的目录，打开文件/dev/fd/n等效于复制描述符n。

在linux系统中，它把文件描述符映射成指向物理底层文件的符号链接。

# 文件和目录
对于文件除了文件本身，还有和文件相关的其他属性。另外还有函数能够修改这些属性

## 函数stat，fstat，fstatat和lstat
```
#include <sys/stat.h>

int stat(const char *restrict pathname, staruct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
```
stat将返回与文件pathname有关的信息结构。fstat函数获得已在描述符fd上打开文件的有关信息。lstat函数类似stat，但是当命名文件是一个符号链接时，lstat返回符号链接本身的有关信息，而不是由符号链接引用的文件的信息。fstatat函数为一个相对于当前打开目录（由fd参数指向）的路径名返回文件统计信息。flag参数控制着是否跟随着符号链接。当AT_SYMLINK_NOFOLLOW标志被设置时，fstatat函数不会跟随符号链接，而是返回符号链接本身的信息。否则，在默认情况下，返回的是符号链接所指向的文件的信息。如果fd参数的值是AT_FDCWD，并且pathname参数是一个相对路径名，fstatat会计算相对于当前目录的pathname参数。如果pathname是一个绝对路径名，fd参数就会被忽略。这两种情况下，根据flag的取值，fstatat函数的作用就跟stat或lstat函数一样。

第二个参数buf是一个指针，它指向一个我们必须提供的结构。函数来填充buf指向的结构。结构的实际定义可能随系统具体实现不同，但其基本形式是：
```
struct stat
{
  mode_t		st_mode;	// file type & mode (permissions)
  ino_t			  st_ino;		// i-node number (serial number)
  dev_t			 st_dev;		// device number (file system)
  dev_t			 st_rdev;		// device number for special files
  nlink_t		   st_nlink;		// number of links
  uid_t			  st_uid;		// user ID of owner
  gid_t			 st_gid;		// group ID of owner
  off_t			  st_size;		// size in bytes, for regular files
  struct timespec	st_atime;	// time of last access
  struct timespec	st_mtime;	// time of last modification
  struct timespec	st_ctime;	// time of last file status change
  blksize_t			st_blksize;	// best IO block size
  blkcnt_t			st_blocks;	// number of disk blocks allocated
};
// timespec按照秒和纳秒定义了时间
struct timespec
{
  time_t tv_sec;
  long tv_nsec;
};
```

stat结构中的大部分成员都是基本系统数据类型。我们将说明此结构的每个成员以了解文件属性。ls命令使用stat函数。

## 文件类型
文件类型包括
* 普通文件（regular file）。这是最常用的文件类型。UNIX系统并不区分文本文件和二进制文件。但是对于可执行文件，内核必须理解其格式，以便能够确定程序文本和数据的加载位置。
* 目录文件（direcory file）。这种文件包含了其他文件的名字与指向这些文件的有关信息。通常只有内核可以直接写目录文件。
* 块特殊文件（block special file）。这种类型的文件提供对设备的带缓冲的访问，每次访问以固定的长度为单位进行。
* 字符特殊文件（character special file）。这种类型的文件提供对设备不带缓冲的访问，每次访问长度可变。系统中的设备要么是字符特殊文件，要么是块特殊文件。
* FIFO。可用于进程间通信，又被称作命名管道（named pipe）。
* 套接字（socket）。这种类型的文件用用于进程间的网络通信。也可以用于在一台宿主机上进程间的非网络通信。
* 符号链接（symbolic link）。这种类型的文件指向另一个文件。

文件类型信息包含在stat结构的st_mode成员中。可以使用宏来确定文件类型。

| 宏          | 文件类型    |
| ---------- | ------- |
| S_ISREG()  | 普通文件    |
| S_ISDIR()  | 目录文件    |
| S_ISCHR()  | 字符特殊文件  |
| S_ISBLK()  | 块特殊文件   |
| S_ISFIFO() | 管道或FIFO |
| S_ISLNK()  | 符号链接    |
| s_ISSOCK() | 套接字     |

POSIX.1允许将进程间通信的对象说明为文件。例如消息队列，信号量，共享内存对象。

对于宏的实现，一般是将st_mode与屏蔽字S_IFMT进行&运算，然后与对应常量比较，判断是否为特定文件类型。
```
#define S_ISDIR(mode) (((mode) & SIFMT) == SIFDIR)
```

## 设置用户ID和设置组ID
与一个进程相关的ID有6个或更多：
* 实际用户ID、实际组ID——我们实际上是谁，在登录时取自口令文件。在一个登录会话期间并不会改变，但是超级用户进程可以改变它们。
* 有效用户ID、有效组ID、附属组ID——用于文件访问权限检查
* 保存的设置用户ID、保存的设置组ID——由exec函数保存

每个文件有一个所有者和组所有者，分别由st_uid和st_gid指出。

执行一个程序文件时，进程的有效用户ID通常就是实际用户ID，有效组ID通常就是实际组ID。但是可以st_mode中设置一个特殊标志，含义为当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID。同样还有类似的设置组ID。分别被称为设置用户ID位和设置组ID位。

例如passwd修改登录口令，是一个设置用户ID程序。因为口令文件只有超级用户才有写入权限。这种程序会获得额外权限，需要谨慎处理。

设置用户ID和设置组ID可以使用S_ISUID和S_ISGID测试。

## 文件访问权限
st_mode也包含了对文件的文件权限位。这里的文件是前面提到的任何类型的文件。所有文件类型都有访问权限。

每个文件有9个访问权限位，可以分为三组

|st_mode屏蔽|含义|
|S_IRUSR|用户读|
|S_IWUSR|用户写|
|S_IXUSR|用户执行|
|S_IRGRP|组读|
|S_IWGRP|组写|
|S_IXGRP|组执行|
|S_IROTH|其他读|
|S_IWOTH|其他写|
|S_IXOTH|其他执行|

前三行用户指的是文件的所有者。我们可以使用chmod来修改文件的访问权限。

对于访问权限以各种方式由不同函数使用。
* 第一个规则是我们用名字打开任一类型的文件时，对该名字包含中的每一个目录，包括它可能隐含的当前工作目录都应具有执行权限。这也是为什么对于目录其执行权限位常被称为搜索位的原因。
* 对于一个文件的读权限决定了我们是否能够打开现有文件进行读操作，对应了open函数的o_RDONLY和O_RDWR标志
* 对于一个文件的写权限决定了我们是否能够打开现有文件进行写操作，对应了open函数的O_WRONLY和O_RDWR标志
* 为了在open函数中对一个文件指定O_TRUNC标志，必须对该文件具有写权限
* 为了在一个目录中创建一个新文件，必须对该目录具有写权限和执行权限
* 为了删除一个现有文件，必须对包含该文件的目录具有写权限和执行权限。对该文件本身则不需要有读写权限
* 如果7个exec函数中的任何一个执行某个文件，都必须对该文件具有执行权限，该文件必须是一个普通文件

进程每次打开，创建或删除一个文件，内核就进行文件访问权限测试，这种测试可能涉及文件的所有者，进程的有效ID及进程的附属组ID。
* 若进程的有效ID是0（超级用户），则允许访问。
* 若进程的有效用户ID等于文件的所有者ID，那么如果所有者适当的访问权限位被设置，则允许访问，否则拒绝访问。
* 若进程的有效组ID或进程的附属组ID之一等于文件的组ID，那么如果组适当的访问权限位被设置，则允许访问，否则拒绝访问。
* 若其他用户适当的访问权限位被设置，则允许访问；否则拒绝访问。

## 新文件和目录的所有权
进程创建的新文件的用户ID设置为进程的有效用户ID。对于组ID，POSIX.1允许实现选择以下之一作为新文件的组ID：
* 新文件的组ID可以是进程的有效组ID
* 新文件的组ID可以是它所在目录的组ID

有些实现默认使用第二种方式，例如Mac OS 10.6.8。linux 3.2.0默认情况下新文件的组ID取决于它所在目录的设置组ID位是否被设置，如果设置，则使用第二种方式，否则使用第一种方式。

## 函数access和faccessat
当打开一个文件时，内核使用进程的有效用户ID和有效组ID为基础执行权限访问测试。进程有时候也希望按照实际用户ID和实际组ID来测试其访问能力。access和faccessat函数按照实际用户ID和实际组ID进行访问权限测试。
```
#include <unistd.h>

int access(const char *pathname, int mode);
int faccess(int fd, const char *pathname, int mode, int flag);
```
如果mode设置为F_OK，则测试文件是否存在。否则是三个常量的按位或：R_OK，W_OK、X_OK。

faccessat函数在下列情况的作用于access函数相同：一种是pathname参数为绝对路径，另一种是fd参数的取值为AT_FDCWD且参数pathname为相对路径。否则，faccessat计算相对于打开目录（fd指定）的pathname。

flag参数可以改变faccessat的行为，如果flag设置为AT_EACCESS，访问检查用的是调用进程的有效用户ID和有效组ID。

## 函数umask
每个进程都有一个相关联的文件模式创建屏蔽字，umask函数可以为进程设置文件模式创建屏蔽字，返回之前的值
```
#include <sys/stat.h>

mode_t umaks(mode_t mode);
```
进程创建一个文件或目录时就一定会使用文件模式创建屏蔽字，在其中为1的位，在文件mode中相应的位一定会被关闭。

在shell中，我们可以使用umask命令来修改这个值。以便修改创建文件的默认权限。

## 函数chmod、fchmod和fchmodat
这三个函数用来修改现有文件的访问权限
```
#include <sys/stat.h>

int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
```
fchmodat函数在两种情况下与chmod函数一样：一种是pathname参数为绝对路径；另一种是fd参数取值为AT_FDCWD且pathname参数为相对路径。否则fchmodat相对于fd打开的目录来计算pathname。flag参数可以改变fchmodat的行为，当设置了AT_SYMLINK_NOFOLLOW时，fchmodat不跟随符号链接。

为了改变一个文件的权限位，进程的有效用户ID必须等于文件所有者ID，或者该进程具有超级用户权限

参数mode由以下值按位或

|mode|说明|
|S_ISUID|执行时设置用户ID|
|S_ISGID|执行时设置组ID|
|S_ISVTX|保存正文（粘着位）|
|S_IRWXU|用户读写执行|
|S_IRUSR|用户读|
|S_IWUSR|用户写|
|S_IXUSR|用户执行|
|S_IRWXG|组读写执行|
|S_IRGRP|组读|
|S_IWGRP|组写|
|S_IXGRP|组执行|
|S_IRWXO|其他读写执行|
|S_IROTH|其他读|
|S_IWOTH|其他写|
|S_IXOTH|其他执行|

chmod函数在下列条件下自动清除两个权限位：
* Solaris等系统对于普通文件的粘着位赋予了特殊含义在这些系统上，如果我们尝试设置普通文件的粘着位(S_ISVTX)，而且又没有超级用户权限，那么mode中的粘着位自动被关闭。这意味着只有超级用户才能设置普通文件的粘着位。这样做是防止恶意用户设置粘着位，影响系统性能。
* 新创建文件的组ID可能不是调用进程所属的组。新文件的组ID可能是父目录的组ID，特别的如果新文件的组ID不等于进程的有效组ID或附属组ID中的一个，而且进程没有超级用户权限，那么设置组ID位会被自动关闭。这就防止了用户创建了一个设置组ID文件，而该文件是并非该用户所属的组拥有的。

有的系统，如Linux 3.2.0等增加了另一个安全性功能试图阻止误用某些保护位。如果没有超级用户权限的进程写一个文件，则设置用户ID位和设置组ID位会被自动清除。如果恶意用户找到一个它们可以写的设置用户ID位和设置组ID位文件，即使它们可以修改此文件，它们也失去了特殊的授权。

## 粘着位
粘着位（S_ISVTX，sticky bit）有着有趣的历史，在早期版本的UNIX中，如果一个可执行程序的这一位被设置了，那么当程序第一次执行，在其终止时，程序正文部分的一个副本仍然保存在交换区中，正文指指令部分。这样程序下次执行就能较快的载入内存。原因是：通常的UNIX文件系统中，文件的各数据块可能是随机存放的，相比较而言，交换区是被作为一个连续文件来存放的。后来的UNIX系统将其称为保存正文位(saved-text bit)。现在的系统大多数都配置了虚拟存储系统以及快速文件系统，所以不再需要此技术。

现在，我们扩展了粘着位的使用范围。如果对一个目录设置了粘着位，只有对该目录具有写权限的用户并且满足下列条件之一，才能删除或重命名该目录下的文件：
* 拥有此文件
* 拥有此目录
* 是超级用户

例如目录/tmp就是这种目录，这样，一个用户就无法删除或重命名其他用户的文件。

## chown、fchown、fchownat和lchown
下列函数用来修改文件的用户ID和组ID，如果参数中任意一个是-1，则对应ID不变：
```
#include <unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int fchownat(int fd, const char *pathname, uid_t owner, gid_t group, int flag);
int lchown(const char *pathname, uid_t owner, gid_t group);
```
除了引用的文件时符号链接外，4个函数都类似。在符号链接的情况下，lchown和fchownat(设置了AT_SYMLINK_NOFOLLOW)更改符号链接本身的所有者，而不是符号链接指向的文件的所有者。

fchown用于改变fd参数指向的打开文件的所有者，它在一个已打开的文件上操作，因此它不能用于改变符号链接的所有者。

fchownat函数与chown或和lchown函数在下列情况下相同：一种是pathname是绝对路径，另一种是fd参数值取值为AT_FDCWD且pathname参数为相对路径。在这两种情况下，如果flag参数设置了AT_SYMLINK_NOFOLLOW，fchownat与lchown函数相同，否则与chown函数相同。其他情况下，fchownat函数计算相对于打开目录的pathname。

一般来说，只有超级用户或文件所有者才能更改一个文件的所有者。POSIX.1允许两种操作中选择一种。可以使用_POSIX_CHOWN_RESTRICTED常量来确定是否有限制，我们可以根据pathconf或fpathconf查询。此选项与文件有关，对于不同的文件可能值不一样。

如果其对指定的文件生效：
* 只有超级用户才能更改此文件的用户ID
* 如果进程拥有此文件，参数owner等于-1或文件的用户ID，并且参数group等于进程的有效组ID或进程的附属组ID之一，那么一个非超级用户进程可以更改此文件的组ID。

如果这些函数由非超级用户进程调用，那么在成功返回时，该文件的设置用户ID位和设置组ID位都被清除。

## 文件长度
stat结构成员st_size表示以字节为单位的长度。此字段只对普通文件、目录文件和符号链接有意义。

对于普通文件，长度可以是0，在开始读这种文件时，将得到文件结束指示。对于目录，文件长度通常是一个数的整数倍。对于符号链接，文件长度是在文件名中的实际字节数。

现今大多数现在的UNIX系统提供字段st_blksize和st_blocks，其中，第一个是对文件IO比较适合的块长度，第二个是所分配的实际512字节(不同系统单位可能不同)的块数。标准IO库一次也尝试读写st_blksize个字节。

我们可以在文件中建立空洞，这些空洞并不占用磁盘空间，读取空洞导致读取到的内容为0。

## 文件截断
有时我们需要在文件尾端处截去一些数据以缩短文件。将一个文件长度截断为0是一个特例，在打开文件时使用O_TRUNC标志可以做到这一点。为了截断文件可以调用函数truncate和ftruncate函数。
```
#include <unistd.h>

int truncate(const char *pathname, off_t length);
int ftruncate(int fd, off_t length);
```
这两个函数将一个现有文件截断到长度length。要么缩短文件，要么产生一个空洞。

## 文件系统
