title: APUE笔记-四
date: 2022-07-25 23:04:56
tags: [unix, linux, 读书笔记, APUE]
---

# 第十一章 线程

## 线程概念

典型的UNIX进程可以看成只有一个控制进程：一个进程在某一时刻只能做一件事情。有了多个控制线程以后，在程序设计时就可以把进程设计成在某一时刻能够做不止一件事，每个线程处理各自独立的任务。其好处如下

- 通过为每种事件类型分配单独的处理线程，可以简化处理异步时间的代码。每个线程会采用同步编程模式，其比异步编程模式简单得多
- 多个进程必须使用操作系统提供的复杂机制才能实现内存和文件描述符的共享，而多个线程自动的可以访问相同的存储地址和文件描述符
- 有些问题可以分解而提高整个程序吞吐量。当然只有在两个任务的处理过程互不依赖的情况下，两个任务才可以较差运行
- 交互程序同样可以使用多线程来改善响应时间，多线程可以把程序中处理用户输入输出的部分与其他部分分开

有些人把多线程的程序设计与多处理器或多核系统联系起来。但是即使程序运行在单处理器上也能得到多线程编程模型的好处。其并不影响程序结构。程序都可以通过使用线程得以简化。

每个线程都包含有表示执行环境所必须的信息，其中包括进程中标识线程的线程ID、一组寄存器值、栈、调度优先级和策略、信号屏蔽字、errno变量以及线程私有数据。一个进程的所有信息对该进程的所有线程都是共享的，包括可执行程序的代码、程序的全局内存和堆内存、栈以及文件描述符

## 线程标识

每个线程有一个线程ID。进程ID在整个系统中是唯一的，但线程ID不同，线程ID只有在它所属的进程上下文中才有意义

与pid_t是一个非负整数不同。线程ID是用pthread_t数据类型表示的，实现的时候可以用一个结构来代表，所以可移植的实现不能把它作为整数处理。因此必须使用一个函数来对两个线程ID进行比较

```cpp
#include <pthread.h>

int pthread_equal(pthread_t tid1, pthread_t tid2);

若相等，返回非0数值；否则，返回0
```

无法用可移植的方法打印pthread_t的值，但线程ID通常在调试过程中需要打印

线程可以通过调用pthread_self函数获得自身的线程ID

```cpp
#include <pthread.h>

pthread_t pthread_self(void);

返回调用线程的线程ID
```

## 线程创建

在传统的UNIX进程模型中，每个进程只有一个控制线程。从概念上讲，这与基于线程模型中每个进程只包含一个线程是相同的。在pthread的情况下，程序开始运行时，它也是以单进程中的单个控制线程启动的。在创建多个控制线程以前，程序的行为与传统的进程并没有什么区别。新增的线程可以通过调用pthread_create函数创建

```cpp
#include <pthread.h>

int pthread_create(pthread_t* restrict tidp, const pthread_attr_t* restrict attr, void* (*start_rtn)(void*), void* restrict arg);

若成功，返回0；否则返回错误编号
```

attr参数用于定制各种不同的线程属性。为NULL时创建一个具有默认属性的线程。新创建的线程从start_rtn函数的地址开始运行，参数为arg

线程创建时并不能保证哪个线程会先运行：是新创建的线程，还是调用线程。新创建的线程可以访问进程的地址空间，并继承调用线程浮点环境和信号屏蔽字，但是该线程的挂起信号集会被清除

注意pthread函数在调用失败时，通常会返回错误码，并不设置errno。每个线程提供errno的副本，这只是为了与使用errno的现有函数兼容。在线程中，从函数中返回错误码更清晰整洁，不需要依赖那些随着函数执行不断变化的全局状态，这样可以把错误范围限制在出错的函数中

pthread_create虽然会返回新线程ID，但是新建的线程并不能安全的使用它，如果新线程在主线程调用pthread_create返回之前就运行了，那么新线程看到的是未初始化的tidp内容，这个内容并不是正确的线程ID

## 线程终止

如果进程中的任意线程调用了exit、_Exit或_exit，那么整个进程就会终止。与此相似，如果默认的动作是终止进程，那么发送到线程的信号就会终止整个进程

单个线程有3中方式退出，因此可以在不终止整个进程的情况下，停止它的控制流

- 线程可以简单的从启动例程中返回，返回值是线程的退出码
- 线程可以被同一进程中的其他线程取消
- 线程调用pthread_exit

```cpp
#include <pthread.h>

void pthread_exit(void* rval_ptr);
```

进程中的线程可以通过调用pthread_join函数访问到退出指针

```cpp
#include <pthread.h>

int pthread_join(pthread_t thread, void** rval_ptr);

若成功，返回0；否则，返回错误编码
```

调用线程将一直阻塞，直到指定的线程退出。如果线程简单的从它的启动例程中返回，rval_ptr就包含了返回码。如果线程被取消，由rval_ptr指定的内存单元就设置为PTHREAD_CANCELED

可以通过调用pthread_detach自动把线程置于分离状态，这样资源就可以恢复。如果线程已处理分离状态，pthread_join调用就会失败，返回EINVAL，尽管这种行为是与具体实现相关的

如果对线程的返回值不感兴趣，可以将rval_ptr设置为NULL。在这种情况下，调用pthread_join的函数可以等待指定的线程终止，但并不获取线程的终止状态

线程可以通过调用pthread_cancel函数来取消同一进程中的其他线程

```cpp
#include <pthread.h>

int pthread_cancel(pthread_t tid);

若成功，返回0；若出错，返回错误编号
```

在默认情况下，pthread_cancel函数会使得由tid标识的线程的行为表现为如同调用了参数为PTHREAD_CANCELED的pthread_exit函数，但是，线程可以选择忽略取消或者控制如何被取消。注意pthread_cancel并不等待线程终止，它仅仅提出请求

线程可以安排它退出时需要调用的函数，这与进程在退出时可以用atexit函数安排退出是类似的。这样的函数称为线程清理处理程序（thread cleanup handler）。一个线程可以建立多个清理处理程序。处理程序记录再栈中，也就是说，它们的执行顺序与他们注册时相反

```cpp
#include <pthread.h>

void pthread_cleanup_push(void (*rtn)(void*), void* arg);
void pthread_cleanup_pop(int execute);
```

pthread_cleanup_push函数注册线程清理处理程序，当线程执行以下动作时，清理函数rtn被执行，参数为arg

- 调用pthread_exit时
- 响应取消请求时
- 用非零execute参数调用pthread_cleanup_pop时

如果execute参数设置为0，清理函数将不被调用。不管发生上述哪种情况，pthread_cleanup_pop都将删除上次pthread_cleanup_push调用建立的清理处理程序

这些函数有一些限制，由于它们可以实现为宏，所以必须在与线程相同的作用域中以匹配对的形式使用。pthread_cleanup_push的宏定义可以包含{，在这种情况下，在pthread_cleanup_pop的定义中要有对应的匹配字符

在Single UNIX Specification中，如果函数在调用pthread_cleanup_push和pthread_cleanup_pop之间返回，会产生未定义行为。唯一的可移植方法是调用pthread_exit

线程和进程函数之间有相似之处，如下

![进程和线程原语比较](https://gwq5210.com/images/进程和线程原语比较.png)

在默认情况下，线程的终止状态会保存直到对该线程调用pthread_join。如果线程已经被分离，线程的底层存储资源可以在线程终止时立即被回收。在线程分离后，我们不能用pthread_join函数等待它的终止状态，因为对分离状态的线程调用pthread_join会产生未定义行为。可以调用pthread_detach分离线程

```cpp
#include <pthread.h>

int pthread_detach(pthread_t tid);

若成功，返回0；若出错，返回错误编号
```

我们将学习通过修改传给pthread_create函数的线程属性，创建一个已处于分离状态的线程

## 互斥量

可以使用pthread的互斥接口来保护数据，确保同一时间只有一个线程访问数据。互斥量（mutex）从本质上是一把锁，在访问共享资源前对互斥量进行加锁，在访问完成后解锁互斥量。对互斥量进行加锁以后，任何其他视图再次对互斥量加锁的线程都会被阻塞知道当前线程释放该互斥锁。如果释放互斥量时有一个以上的线程阻塞，那么所有该锁上的阻塞线程都会变成可运行状态，第一个变成运行的线程就可以对互斥量加锁，其他线程就会看到互斥量依然是锁着的，只能回去再次等待它重新变为可用。在这种方式下，每次只有一个线程可以向前执行。

只有将所有线程都设计成遵守相同的数据访问规则的，互斥机制才能正常工作。操作系统并不会为我们做的数据访问串行化。如果允许其中的某个线程在没有得到锁的情况下也可以访问共享资源，那么即使其他线程在使用共享资源前都申请锁，也还是会出现数据不一致的问题

互斥变量是用pthread_mutex_t数据类型表示的。在使用互斥变量以前，必须首先对它进行初始化，可以把它设置为常量PTHREAD_MUTEX_INITIALIZER（只适用于静态分配的互斥量），也可以通过调用pthread_mutex_init函数进行初始化。如果动态分配互斥量（例如通过调用malloc函数），在释放内存前需要调用pthread_mutex_destroy

```cpp
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t* restrict mutex, const pthread_mutexattr_t* restrict attr);
int pthread_mutex_destroy(pthread_mutex_t* mutex);

若成功，返回0；若出错，返回错误编号
```

要用默认的属性初始化互斥量，只需要把attr设置为NULL。

对互斥量进行加锁，需要调用pthread_mutex_lock。如果互斥量已经上锁，调用线程将阻塞直到互斥量被解锁。对互斥量解锁，需要调用pthread_mutex_unlock

```cpp
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t* mutex);
int pthread_mutex_trylock(pthread_mutex_t* mutex);
int pthread_mutex_unlock(pthread_mutex_t* mutex);

若成功，返回0；若出错，返回错误编号
```

如果线程不希望被阻塞，它可以使用pthread_mutex_trylock尝试对互斥量进行加锁。如果调用pthread_mutex_trylock时互斥量处于为锁住状态，那么pthread_mutex_trylock将锁住互斥量，不会出现阻塞直接返回0，否则pthread_mutex_trylock就会失败，不能锁住互斥量，返回EBUSY

### 避免死锁

如果线程视图对同一个互斥量加锁两次，那么自身就会陷入死锁状态，但是使用互斥量时，还有其他不太明显的方式也能产生死锁。

可以通过仔细控制互斥量加锁的顺序来避免死锁的发生。即不同的线程总是以相同的顺序加锁互斥量

但有时候，应用程序的结构使得对互斥量进行排序是很困难的。如果涉及了太多的锁和数据结构，可用的函数并不能把它转换成简单的层次，那么就需要采用另外的方法。在这种情况下，可以先释放占用的锁，然后过一段时间再试。这种情况可以使用pthread_mutex_trylock接口避免死锁。如果已经占有某些锁而且pthread_mutex_trylock接口返回成功，那么就可以前进。但是如果不能获取锁，可以先释放已经占用的锁，做好清理工作，然后过一段时间再重新尝试

如果锁的粒度太粗，就会出现很多线程阻塞等待相同的锁，这可能并不能改善并发性。如果锁的粒度太细，那么过多的锁开销就会使系统性能收到影响，而且代码变得复杂。作为一个程序员，需要在满足锁需求的情况下，在代码复杂性和性能之间找到正确的平衡

### 函数pthread_mutex_timedlock

当线程视图获取一个已加锁的互斥量时，pthread_mutex_timedlock互斥量原语允许绑定线程阻塞时间。pthread_mutex_timedlock与pthread_mutex_lock是基本等价的，但是在到达超时时间值时，pthread_mutex_timedlock不会对互斥量进行加锁，而是返回错误码ETIMEDOUT

```cpp
#include <pthread.h>

int pthread_mutex_timedlock(pthread_mutex_t* restrict mutex, const struct timespec* restrict timespec);

若成功，返回0；若出错，返回错误编号
```

超时指定愿意等待的绝对时间（与相对时间比较而言，指定在时间X之间可以阻塞等待，而不是说愿意阻塞Y秒）。这个超时时间是用timespec结构来表示的，它用秒和纳秒来描述时间

注意，阻塞的时间可能会有所不同，造成不同的原因有很多种：开始时间可能在某秒的中间位置，系统时钟的精度可能不足以精确支持我们指定的超时时间值，或者在程序继续运行前，调度延迟可能会增加时间值

## 读写锁

读写锁（reader-writer-lock）与互斥量类似，不过读写锁允许更高的并行性。互斥量要么是锁住状态，要么就是不加锁状态，而且一次只有一个线程可以对其加锁。读写锁可以有3种状态：读模式下加锁状态，写模式下加锁状态，不加锁状态。一次只有一个线程可以占有写模式的读写锁，但是多个线程可以同时占有读模式的读写锁

在读写锁是写加锁状态时，在这个锁被解锁之前，所有尝试读对这个锁加锁的线程都会被阻塞。当读写锁在读加锁状态时，所有尝试以读模式对它进行加锁的线程都可以得到访问权，但是任何希望以写模式对此锁进行加锁的线程都会阻塞，知道所有的线程释放它们的读锁为止。

虽然各操作系统对读写锁的实现各不相同，但当读写锁处于读模式锁住的状态，而这时有一个线程视图以写模式获取锁时，读写锁通常会阻塞随后的读模式锁请求。这样可以避免读模式锁长期占用，而等待的写模式锁请求一直得不到满足

读写锁非常适用于对数据结构读的次数远大于写的情况。当读写锁在写模式下时，它所保护的数据结构就可以被安全的修改，因为一次只有一个线程可以在写模式下拥有这个锁。当读写锁在读模式下时，只要线程先获取了读模式下的读写锁，读锁所保护的数据结构就可以被多个获得读模式锁的线程读取

读写锁也叫做共享互斥锁（shared-execlusive lock）。当读写锁是以读模式锁住时，就可以说成是以共享模式锁住的。当它是写模式锁住的时候，就可以说成是互斥模式锁住的。

像互斥量一样，读写锁在使用之前必须初始化，在释放它们底层的内存之前必须销毁

```cpp
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t* restrict rwlock, const pthread_rwlockattr_t* restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t* rwlock);

若成功，返回0；若出错，返回错误编号
```

读写锁通过调用pthread_rwlock_init进行初始化。如果希望读写锁有默认属性，可以传一个NULL指针给attr

可以使用PTHREAD_RWLOCK_INITIALIZER对静态分配的读写锁进行初始化

在释放读写锁占用的内存之前，需要调用pthread_rwlock_destroy做清理工作。如果pthread_rwlock_init为读写锁分配了资源，pthread_rwlock_destroy将释放这些资源。如果在调用pthread_rwlock_destroy之前就释放了读写锁占用的内存空间，那么分配给这个锁的资源就会丢失

要在读模式下锁定读写锁，需要调用pthread_rwlock_rdlock。要在写模式下锁定读写锁，需要调用pthread_rwlock_wrlock。不过以何种方式锁住读写锁，都可以调用pthread_rwlock_unlock进行解锁

```cpp
#include <pthread.h>

int pthread_rwlock_rdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t* rwlock);

若成功，返回0；若出错，返回错误编号
```

各种实现可能会对共享模式下可获取的读写锁的次数进行限制，所以需要检查pthread_rwlock_rdlock的返回值。尽管pthread_rwlock_wrlock和pthread_rwlock_unlock有错误返回，而且从技术上来讲，在调用函数时应该总是检查错误返回，但是如果锁设计合理的话，就不需要检查它们。错误返回值的定义只是针对不正确使用读写锁的情况（如未经初始化的锁），或试图获取已拥有的锁而可能产生死锁的情况。但是需要注意，有些特定的实现可能会定义另外的错误返回

Single UNIX Specification还定义了读写锁原语的条件版本

```cpp
#include <pthread.h>

int pthread_rwlock_tryrdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t* rwlock);

若成功，返回0；若出错，返回错误编号
```

可以获取锁时，两个函数返回0，否则，它们返回EBUSY。这两个函数可以用于我们前面讨论的遵守某种锁层次但还不能完全避免死锁的情况

### 带有超时的读写锁

```cpp
#include <pthread.h>
#include <time.h>

int pthread_rwlock_timedrdlock(pthread_rwlock_t* restrict rwlock, const struct timespec* restrict tsptr);
int pthread_rwlock_timedwrlock(pthread_rwlock_t* restrict rwlock, const struct timespec* restrict tsptr);

若成功，返回0；若出错，返回错误编号
```

其行为与它们的不计时版本类似。如果不能获取锁，超时时间到达时，这两个函数将返回ETIMEDOUT错误。超时指定的是绝对时间，而不是相对时间

## 条件变量

条件变量是线程可用的另一种同步机制。条件变量给多线程提供了一个汇合的场所。条件变量与互斥量一起使用允许线程已无竞争的方式，等待特定的条件发生。

条件本身是由互斥量保护的，线程在改变条件状态之前必须首先锁住互斥量。其他线程在获得互斥量之前不会察觉到这种变化，因互斥量必须在锁定以后才能计算条件。

在使用条件变量之前必须先对它进行初始化。由pthread_cond_t数据类型表示的条件变量可以用两种方式进行初始化，可以把常量PTHREAD_COND_INITIALIZER赋给静态分配的条件变量，但是如果条件变量是动态分配的，则需要使用pthread_cond_init函数对它进行初始化

在释放条件变量底层的内存空间前，可以使用pthread_cond_destroy函数对条件变量进行销毁

```cpp
#include <pthread.h>

int pthread_cond_init(pthread_cond_t* cond, const pthread_condattr_t* restrict attr);
int pthread_cond_destroy(pthread_cond_t* cond);

若成功，返回0；若出错，返回错误编号
```

当attr参数为NULL时，创建一个具有默认属性的条件变量

我们使用pthread_cond_wait等待条件变量变为真。如果在给定的时间内条件不能满足，那么会返回错误编码

```cpp
#include <pthread.h>

int pthread_cond_wait(pthread_cond_t* restrict cond, pthread_mutex_t* restrict mutex);
int pthread_cond_timedwait(pthread_cond_t* restrict cond, pthread_mutex_t* restrict mutex, const struct timespec* restrict tsptr);

若成功，返回0；若出错，返回错误编号
```

传递给pthread_cond_wait的互斥量对条件变量进行保护。调用者把锁住的互斥量传给函数，函数然后自动把调用线程放到等待条件的线程列表上，对互斥量解锁。这就关闭了条件检查和线程进入休眠状态等待条件改变这两个操作之间的时间通道，这样线程就不会错过条件的任何变化。pthread_cond_wait返回时，互斥量再次被锁住。pthread_cond_timedwait可以指定等待的超时，其是一个绝对时间，如果超时到期条件还没有出现，那么pthread_cond_timedwait将重新获取互斥量，然后返回错误ETIMEDOUT。从pthread_cond_wait和pthread_cond_timedwait调用成功返回时，线程需要重新计算条件，因为另一个线程可能已经在运行并改变了条件

有两个函数可以用于通知线程条件已经满足。pthread_cond_signal函数至少能唤醒一个等待该条件的线程，pthread_cond_broadcast函数则能唤醒等待该条件的所有线程。POSIX规范为了简化pthread_cond_signal的实现，允许它在实现时唤醒一个以上的线程

```cpp
#include <pthread.h>

int pthread_cond_signal(pthread_cond_t* cond);
int pthread_cond_broadcast(pthread_cond_t* cond);

若成功，返回0；若出错，返回错误编号
```

在给等待线程发送信号时，不需要占有互斥量。我们在while循环中检查条件，所以不存在这样的问题：线程醒来，发现队列仍是空，然后返回继续等待。如果代码不能容忍这种竞争，就需要再给线程发信号的时候，占有互斥量

## 自旋锁

自旋锁与互斥量类似，但它不是通过休眠使进程阻塞，而是在获取锁之前一直处于忙等（自旋）阻塞状态。自旋锁可用于以下情况：锁被持有的时间短，而且线程并不希望在重新调度上花费太多的成本。

自旋锁通常作为底层原语用于实现其他类型的锁。根据他们所基于的系统体系结构，可以通过使用测试并设置指令有效地实现。当然这里说的有效还是会导致CPU资源的浪费：当线程自旋等待锁变为为可用时，CPU不能做其他的事情。这也是自旋锁只能够被持有一小段时间的原因。

当自旋锁在非抢占式内核中时是非常有用的：除了提供互斥机制以外，他们会阻塞中断，这样中断处理程序就不会让系统陷入死锁状态，因为他需要获取已被加锁的自旋锁（把中断想成是另一种抢占）。在这种类型的内核中，中断处理处理程序不能休眠，因此他们能用的同步原语只能是自旋锁。

但是，在用户层自旋锁并不是非常有用，除非运行在不允许抢占的实时调度类中。运行在分时调度类中的用户层线程在两种情况下可以被取消调度：当他们的时间片到期时，或者具有更高优先级的线程就绪变成可运行时。在这些情况下，如果线程拥有自旋锁，他就会进入休眠状态，阻塞在锁上的其他线程自旋的时间可能会比预期的时间更长。

很多互斥量的实现非常高效，以至于应用程序采用互斥锁的性能与曾经采用过自旋锁的性能基本是相同的。事实上，有些互斥量的实现在试图获取互斥量的时候会自旋一小段时间，只有在自旋计数达到某一阈值的时候才会休眠。这些因素，加上现代处理器的进步，使得上下文切换越来越快，也使得自旋锁只在某些特定的情况下有用。

自旋锁的接口与互斥量的接口类似，这使得它可以比较容易的从一个替换成另一个。可以用pthread_spin_init函数对自旋锁进行初始化。用pthread_spin_destroy函数进行自旋锁的销毁

```cpp
#include <pthread.h>

int pthread_spin_init(pthread_spinlock_t* lock, int pshared);
int pthread_spin_destroy(pthread_spinlock_t* lock);

若成功，返回0；若出错，返回错误编号
```

只有一个属性是自旋锁特有的，这个属性只在支持线程进程同步（Thread Process-Shared Synchronization）选项的平台上才用得到。pshared参数表示进程共享属性，表明自旋锁是如何获取的。如果它设为PTHREAD_PROCESS_SHARED，则自旋锁能被可以访问底层内存的线程锁获取，即便那些线程属于不同的进程，情况也是如此。否则pshared参数设置为PTHREAD_PROCESS_PRIVATE，自旋锁就只能被初始化该锁的进程内部的线程所访问

可以用pthread_spin_lock或pthread_spin_trylock对自旋锁进行加锁，前者在获取锁之前一直自旋，后者如果不能获取锁，就立即返回EBUSY错误。注意pthread_spin_trylock不能自旋。不管以何种方式加锁，自旋锁都可以调用pthread_spin_unlock函数解锁

```cpp
#include <pthread.h>

int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t * lock);

若成功，返回0；若出错，返回错误编号
```

注意，如果自旋锁当前在解锁状态的话，pthread_spin_lock函数不要自旋就可以对它加锁。如果线程应对它加锁了，结果就是未定义的。调用pthread_spin_lock会返回EDEADLK错误（或其他错误），或者调用可能会永久自旋。具体行为依赖于实际的实现。视图对没有加锁的自旋锁进行解锁，结果也是未定义的

不管是pthread_spin_lock还是pthread_spin_trylock，返回值为0的话就表示自旋锁被加锁。需要注意，不要调用再持有自旋锁情况下可能会进入休眠状态的函数。如果调用了这些函数，会浪费CPU资源，因为其他线程需要获取自旋锁需要等待的时间就延长了。

## 屏障

屏障（barrier）是用户协调多个线程并行工作的同步机制。屏障允许每个线程等待，直到所有的合作线程都到达某一点，然后从该点继续执行。我们已经看到一种屏障，pthread_join函数就是一种屏障，允许一个线程等待，直到另一个线程退出。

但是屏障对象的概念更广，他们允许任意数量的线程等待，直到所有的线程完成处理工作，而线程不需要退出，所有的线程到达屏障后可以接着工作。

可以使用pthread_barrier_init函数对屏障进行初始化，用pthread_barrier_destroy函数进行销毁

```cpp
#include <pthread.h>

int pthread_barrier_init(pthread_barrier_t* restrict barrier, const pthread_barrierattr_t* restrict attr, unsigned int count);
int pthread_barrier_destroy(pthread_barrier_t* barrier);

若成功，返回0；若出错，返回错误编号
```

初始化屏障时，可以使用count参数指定，在允许所有线程继续运行之前，必须到达屏障的线程数目。使用attr参数指定屏障对象的属性。当attr参数为NULL时，使用默认属性初始化屏障。如果使用pthread_barrier_init函数为屏障分配资源，那么在销毁屏障时可以调用pthread_barrier_destroy函数释放相应的资源。

可以使用pthread_barrier_wait函数来表明线程已完成工作，准备等待所有其他线程赶上来。

```cpp
#include <pthread.h>

int pthread_barrier_wait(pthread_barrier_t* barrier);

若成功，返回0或者PTHREAD_BARRIER_SERIAL_THREAD；若出错，返回错误编号
```

调用pthread_barrier_wait的线程在屏障计数未满足条件时，会进入休眠状态。如果该线程是最后一个调用pthread_barrier_wait的线程就满足了屏障计数，所有的线程都被唤醒

对于一个任意线程，pthread_barrier_wait函数返回了PTHREAD_BARRIER_SERIAL_THREAD。剩下的线程返回值是0.这使得一个线程可以作为主线程，他可以工作在其他所有线程已完成的工作结果上

一旦达到屏障计数值，而且线程处于非阻塞状态，屏障就可以被重用。但是除非在调用了pthread_barrier_destroy函数之后，有调用了pthread_barrier_init函数对计数用另外的数进行初始化，否则屏障计数不会改变

# 第十二章 线程控制

## 线程限制

线程有一些限制，如创建的键（线程私有数据）的最大数目，线程栈的最小字节数，进程可以创建的线程的最大数目等

![线程配置限制的实例](https://gwq5210.com/images/线程配置限制的实例.png)

## 线程属性

pthread接口允许我们通过设置每个对象关联的不同属性来细调线程和同步对象的行为。通常管理这些属性的函数都遵循相同的模式

- 每个对象与他自己类型的属性对象进行关联（线程与线程属性关联，互斥量与互斥量属性关联等等）。一个属性对象可以代表多个属性。属性对象对应用程序来说是不透明的。这意味着应用程序并不需要了解有关属性对象内部结构的详细细节，这样可以增强应用程序的可移植性，取而代之的是需要提供相应的函数来管理这些属性对象。
- 有一个初始化函数，把属性设置为默认值。
- 还有一个销毁属性对象的函数，如果初始化函数分配了与属性对象关联的资源，销毁函数负责释放这些资源。
- 每个属性都有一个从属性对象中获取属性值的函数。由于函数成功时返回零，失败时返回错误编号，所以可以通过把属性值存储在函数的某一参数指定的内存单元中，把属性值返回给调用者
- 每个属性都有一个设置属性值的函数，在这种情况下属性值作为参数按值传递。

![线程属性](https://gwq5210.com/images/线程属性.png)

线程分离属性的有效值为：PTHREAD_CREATE_DETACHED或PTHREAD_CREATE_JOINABLE

对于进程来说，虚地址空间的大小是固定的，因为进程中只有一个栈，所以它的大小通常不是问题。但对于线程来说，同样大小的虚拟地址空间，必须被所有的线程栈站共享。如果应用程序，使用了许多线程，以致这些线程栈的累积大小超过了可用的虚拟地址空间，就需要减少默认的线程栈大小。另一方面，如果线程调用的函数分配了大量的自动变量或者调用的函数，涉及许多很深的栈帧（stack frame），那么需要的栈大小，可能要比默认的大。

如果线程栈的虚地址空间都用完了，那可以使用malloc或者mmap来为可替代的栈分配空间，并用pthread_attr_setstack函数来改变新建线程的栈位置。由stackaddr参数指定的地址可以用作线程栈的内存范围中的最低可寻址地址，改地址与处理器结构相应的边界应对齐。当然，这要假设malloc和mmap所用的虚地址范围与线程栈当前使用的虚地址范围不同。stackaddr线程属性被定义为栈的最低内存地址，但这并不一定是栈的开始位置。对于一个给定的处理器结构来说，如果栈是从高地址向低地址方向增长的，那么stackaddr线程属性将是栈的结尾位置，而不是开始位置

可以修改默认栈的大小，但最小不能小于PTHREAD_STACK_MIN

线程属性guardsize控制着线程栈末尾之后用于避免栈溢出的扩展内存大小。这个属性默认值的由具体的实现来定义的，但常用值是系统页大小。如果线程的栈指针溢出到警戒区，应用程序就可能通过信号接收到出错信息

线程还有一些其他的pthread_attr_t结构中没有表示的属性：可撤销状态和可撤销类型

## 互斥量属性

互斥量值得注意的3个属性是：进程共享属性、健壮属性以及类型属性

在进程中，多个线程可以访问同一个同步对象。这是默认的行为，在这种情况下，进程共享互斥量属性需设置为PTHREAD_PROCESS_PRIVATE。存在这样的机制，允许相互独立的多个进程把同一个内存数据块映射到它们各自独立的地址空间中。就像多个线程访问共享数据一样，多个进程访问共享数据通常也需要同步。如果进程共享互斥量属性设置为PTHREAD_PROCESS_SHARED，从多个进程彼此之间共享的内存数据块中分配的互斥量就可以用于这些进程的同步

进程共享互斥量属性设置为PTHREAD_PROCESS_PRIVATE时，允许pthread线程库提供更有效的互斥量实现，这在多线程应用程序中是默认的情况。在多个进程共享多个互斥量的情况下，pthread线程库可以限制开销较大的互斥量实现

互斥量健壮属性与多个进程间共享的互斥量有关，这意味当持有互斥量的进程终止时，需要解决互斥量状态恢复的问题。这种情况发生时，互斥量处于锁定状态，恢复起来很困难。其他阻塞在这个锁的进程，将会一直阻塞下去。

健壮属性取值有两种可能的情况，默认值是PTHREAD_MUTEX_STALLED，这意味着时有互斥量的进程终止时不需要采取特别的动作。在这种情况下，使用互斥量后的行为是未定义的，等待该互斥量解锁的应用程序会被有效地“拖住”。

另一个取值是PTHREAD_MUTEX_ROBUST。这个值将导致线程调用pthread_mutex_lock获取锁，而该锁被另一个进程持有，但他终止时并没有对该锁进行解锁，此时线程会阻塞，从pthread_mutex_lock返回的值为EOWNERDEAD而不是0。你应用程序可以通过这个特殊的返回值获知，若有可能（要保护状态的细节以及如何进行恢复会已经不同的应用程序而异），不管他们保护的互斥量状态如何，都需要进行恢复。使用健壮的互斥量改变了我们使用pthread_mutex_lock的方式，因为现在必须检查三个返回值，而不是之前的两个：不需要恢复的成功，需要恢复的成功以及失败。但是即使不用健壮的互斥量，也可以只检查成功或者失败。Linux 3.2.0支持健壮的线程互斥量

如果应用状态无法恢复，在线程对互斥量解锁以后，该互斥量将处于永久不可用的状态。为了避免这样的问题，线程可以调用函数pthread_mutex_consistent函数，指明该互斥量相关的状态在互斥量解锁之前是一致的。如果线程没有先调用pthread_mutex_consistent函数就对互斥量进行了解锁，那么其他试图获取该互斥量的阻塞线程就会得到错误吗ENOTRECOVERABLE。如果发生这种情况，互斥量将不再可用。线程通过提前调用pthread_mutex_consistent，能让互斥量正常工作，这样他就可以持续被使用。

类型互斥量属性控制着互斥量的锁定特性

- PTHREAD_MUTEX_NORMAL: 一种标准互斥量类型,不做任何特殊的错误检查或死锁检测。
- PTHREAD_MUTEX_ERRORCHECK: 此互斥量类型提供错误检查
- PTHREAD_MUTEX_RECURSIVE: 此互斥量类型允许同一线程在互斥量解锁之前对该互斥量进行多次加锁。递归互斥量维护锁的计数，在解锁次数和加锁次数不相同的情况下，不会释放锁。所以如果对一个递归互斥量加锁两次，然后解锁一次，那么这个互斥量将依然处于加锁状态，对他再次解锁以前不能释放该锁。
- PTHREAD_MUTEX_DEFAULT: 此互斥量类型可以提供默认特性和行为。操作系统在实现它的时候，可以把这种类型自由的映射到其他互斥量类型中的一种。

![互斥量类型行为](https://gwq5210.com/images/互斥量类型行为.png)

互斥量用于保护与条件变量关联的条件。在阻塞线程之前，pthread_cond_wait和pthread_cond_timedwait函数释放与条件变量相关的互斥量，这就允许其他线程获取互斥量、改变条件、释放互斥量以及给条件变量发信号。既然改变条件时，必须占有互斥量，使用递归互斥量就不是一个好主意，如果递归互斥量被多次加锁，然后用在调用pthread_cond_wait函数中，那么条件永远不会得到满足，因为pthread_cond_wait所做的解锁操作并不能释放互斥量。

如果需要把现有的单线程接口放到多线程环境中，递归互斥量是非常有用的，但由于现有程序兼容性的限制，不能对函数接口进行修改。然而，使用递归锁可能很难处理，因此应该只在没有其他可行方案的时候才使用他们。

## 读写锁属性

读写锁支持的唯一属性是进程共享属性。它与互斥量的进程共享属性是相同的，就像互斥量的进程共享属性一样，有一个函数用于读取和设置读写所的进程共享属性。

## 条件变量属性

Single UNIX Specification目前定义了条件变量的两个属性：进程共享属性和时钟属性。

与其他的同步属性一样，条件变量的进程共享属性控制着条件变量是可以被单进程的多个线程使用，还是可以被多进程的线程使用。

时钟属性控制计算pthread_cond_timedwait函数超时参数tsptr时采用的是哪个时钟。奇怪的是Single UNIX Specification并没有为其他有超时等待函数的属性对象定义时钟属性

## 屏障属性

目前定义的屏障属性只有进程共享属性

## 重入

线程在遇到重入问题时与信号处理程序是类似的。在这两种情况下，多个控制线程在相同的时间有可能调用相同的函数。如果一个函数在相同的时间点可以被多个线程安全的调用，就称该函数是线程安全的。在Single UNIX Specification中定义的所有函数中，除了下房列出的函数，其他函数都保证是线程安全的。

![不能保证线程安全的函数](https://gwq5210.com/images/不能保证线程安全的函数.png)

如果一个函数对多个线程来说是可重入的，就说这个函数是线程安全的。但这并不能说明对信号处理程序来说，该函数也是可重入的。如果函数对异步信号处理程序的重入是安全的，那么就可以说函数是异步信号安全的。

## 线程特定数据

线程特定数据（thread-specific data），也称为线程私有数据（thread-private data），是存储和查询某个特定线程相关数据的一种机制。我们把这种数据称为线程特定数据或线程私有数据的原因是，我们希望每个线程可以访问他自己单独的数据副本，而不需要担心与其他线程的同步访问问题。

线程模型促进了进程中数据和属性的共享，许多人在设计线程模型时会遇到各种麻烦。那么为什么有人想在这样的模型中设计阻止共享的借口呢？这其中有两个原因

- 有时候需要维护基于每个线程（per-thread）的数据，因为线程ID并不能保证是最小而连续的整数，所以就不能简单的分配 一个每线程数据数组，用线程ID作为数组的索引。即使线程ID确实是最小而连续的整数，我们可能还希望有一些额外的保护，防止某个线程的数据，与其他线程的数据相混淆。
- 它提供了让基于进程的接口适应多线程环境的机制。一个很明显的实例就是errno。以前的接口（线程出现以前）把errno定义为进程上下文中全局可访问的整数。系统调用和库例程在调用或执行失败时设置errno，把它作为操作失败时的附属结果。为了让线程也能够使用那些原本基于进程的系统调用和库例程，errno被重新定义为线程私有数据。这样一个线程做了重置errno的操作，也不会影响进程中其他线程的errno值。

我们知道一个进程中的所有线程都可以访问这个进程的整个地址空间，除了使用寄存器外，一个线程没有办法阻止另一个线程访问他的数据。线程特定数据也不例外。虽然底层的实现部分并不能阻止这种访问能力，但管理线程特定数据的函数，可以提高线程间的数据独立性，使得线程不太容易访问到其他线程的线程特定数据。

在分配线程特定数据之前，需要创建与该数据关联的键。这个键用于获取对线程特定数据的访问。使用pthread_key_create创建一个键

```cpp
#include <pthread.h>

int pthread_key_create(pthread_key_t* keyp, void (*destructor)(void*));

若成功，返回0；若出错，返回错误编号
```

创建的键存储在keyp指向的内存单元中，这个键可以被进程中所有的线程使用，但每个线程把这个键与不同的线程特定数据地址进行关联。创建新键时，每个线程的数据地址设为空值

除了创建键以外，pthread_key_create可以为该键关联一个可选择的析构函数。当这个线程退出时，如果数据地址空间已经被设置为非空值，那么析构函数就会被调用，他唯一的参数就是该数据地址，如果传入的析构函数为空就表明没有析构函数与这个键相关联，当线程调用pthread_exit或线程执行返回，正常退出时，析构函数就会被调用。同样线程取消时，只有在最后的清理处理程序返回之后，析构函数才会被调用。如果线程调用了exit、_exit、_Exit或abort，或者出现其他非正常的退出时，就不会调用析构函数。

线程经常使用malloc为线程特定数据分配内存，析构函数通常释放已分配的内存。如果线程在没有释放内存之前就退出了，那么这块内存就会丢失，即线程所属进程就出现了内存泄露。

线程可以为线程特定数据分配多个键，每个键都可以有一个析构函数与他关联，每个键的析构函数可以互不相同，当然所有键也可以使用相同的析构函数，每个操作系统实现对进程可分配的键的数量进行限制。

线程退出时线程特定数据的析构函数，将按照操作系统实现中定义的顺序被调用。析构函数可能会掉用另一个函数，该函数可能会创建新的线程特定数据，并且把这个数据与当前的键关联起来。当所有的析构函数都调用完成以后，系统会检查是否还有非空的线程特定数据值与当前的键关联，如果有的话再次调用析构函数。这个过程将会一直重复，直到所有的键都为空线程特定数据值，或者已经做了PTHREAD_DESTRUCTOR_ITERATIONS中定义的最大次数的尝试

对所有的线程我们都可以调用pthread_key_delete来取消键与线程特定数据值之间的关联关系

```cpp
#include <pthread.h>

int pthread_key_delete(pthread_key_t key);

若成功，返回0；若出错，返回错误编号
```

注意，调用pthread_key_delete并不会激活与键关联的析构函数。要释放任何与键关联的线程特定数据值的内存，需要在应用程序中采取额外的步骤

需要确保分配的键并不会由于初始化阶段的竞争而发生变动。下面的代码会导致两个线程都调用pthread_key_create

```cpp
void destructor(void*);

pthread_key_t key;
int init_done = 0;

int threadfunc(void* arg) {
  if (!init_done) {
    init_done = 1;
    err = pthread_key_create(&key, destructor);
  }
}
```

有些线程可能看到一个键值，而其他线程看到的可能是另一个不同的键值，这取决于系统是如何调度线程的，解决这种竞争的办法是使用pthread_once

```cpp
#include <pthread.h>

pthread_once_t initflag = PTHREAD_ONCE_INIT;
int pthread_once(pthread_once_t* initflag, void (*initfn)(void));

若成功，返回0；若出错，返回错误编号
```

initflag必须是一个非本地变量（如全局变量或静态变量），而且必须初始化为PTHREAD_ONCE_INIT

如果每个线程都调用pthread_once，系统就能保证初始化例程initfn只被调用一次，即系统首次调用pthread_once时。创建键时避免出现冲突的一个正确方法如下

```cpp
void destructor(void*);

pthread_key_t key;
pthread_once_t init_done = PTHREAD_ONCE_INIT;

void thread_init(void) {
  err = pthread_key_create(&key, destructor);
}

int threadfunc(void* arg) {
  pthread_once(&init_done, thread_init);
}
```

键一旦创建成功以后，就可以通过调用pthread_setspecific函数把键和线程特定数据关联起来。可以通过pthread_getspeciffic函数获得线程特定数据的地址

```cpp
void* pthread_getspecific(pthread_key_t key);

返回线程特定数据值；若没有值与该键关联，返回NULL

int pthread_setspeciffic(pthread_key_t key, const void* value);

若成功，返回0；若出错，返回错误编号
```

如果没有线程特定数据与键关联，pthread_getspecific将返回一个空指针，我们可以用这个返回值来确定是否需要调用pthread_setspecific

## 取消选项

有两个线程属性并没有包含在pthread_attr_t结构中，它们是可取消状态和可取消类型。这两个属性影响pthread_cancel函数调用时所呈现的行为

可取消状态可以是PTHREAD_CANCEL_ENABLE也可以是PTHREAD_CANCEL_DISABLE。

pthread_cancel调用并不等待线程终止。在默认情况下，线程在取消请求发出以后还是继续运行，直到线程到达某个取消点。取消点是线程检查它是否被取消的一个位置，如果取消了，则按照请求行事。取消点的函数如下（还有一些可选的取消点）

![定义的取消点](https://gwq5210.com/images/定义的取消点.png)

线程启动时默认的可取消状态是PTHREAD_CANCEL_ENABLE。当状态设置为了PTHREAD_CANCEL_DISABLE时，对pthread_cancel的调用并不会杀死线程。相反，取消请求对这个线程来说还处于挂起状态，当取消状态再次变为PTHREAD_CANCEL_ENABLE时，线程将在下一个取消点上对所有挂起的请求进行处理

如果应用程序在很长一段时间都不会调用取消点的函数，那么可以调用pthread_testcancel函数在应用程序中添加自己的取消点。调用pthread_testcancel时，如果某个取消请求正处于挂起状态，而且取消并没有设置为无效，那么线程就会被取消。但是，如果取消被置为无效，pthread_testcancel调用就没有任何效果了

我们所描述的默认取消类型也称为推迟取消。调用pthread_cancel以后，在现场到达取消点之前，并不会出现真正的取消。取消类型可以是PTHREAD_CANCEL_DEFERRED或PTHREAD_CANCEL_ASYNCHRONOUS。异步取消与推迟取消不同，使用异步取消时，线程可以在任意时间撤销，不是非得遇到取消点才能被取消

## 线程和信号

即使在基于进程的编程范型中，信号的处理有时候也是很复杂的。把线程引入编程范型，就使信号的处理变得更加复杂

每个线程都有自己的信号屏蔽字，但是信号的处理是进程中所有线程共享的。这意味着单个线程可以阻止某些信号，但是当没某个线程修改了与某个给定信号的相关处理行为以后，所有的线程都必须共享这个处理行为的改变。这样，如果一个线程选择忽略某个给定信号，那么另一个线程就可以通过以下两种方式撤销上述线程的信号选择：恢复信号的默认处理行为或者为信号设置一个新的信号处理程序

进程中的信号是递送到单个线程的。如果一个信号与硬件故障相关，那么该信号一般会被发送到引起该事件的线程中去，而其他的信号则被发送到任意一个线程。

sigprocmask的行为在多线程的进程中并没有定义，线程必须使用pthread_sigmask

```cpp
#include <pthread.h>

int pthread_sigmask(int how, sigset_t* restrict set, sigset_t* restrict oset);

若成功，返回0；若出错，返回错误编号
```

pthread_sigmask函数与sigprocmask基本相同，不过pthread_sigmask工作在现场中，而且失败时返回错误码，不再像sigprocmask那样设置errno并返回-1。set参数包含线程用于修改信号屏蔽字的信号集。how参数可以取以下3个值之一：SIG_BLOCK，把信号集添加到线程信号屏蔽字中，SIG_SETMASK，用信号集替换线程的信号屏蔽字，SIG_UNBLOCK，从线程信号屏蔽字中移除信号集。如果oset参数不为空，线程之前的信号屏蔽字就存储在它指向的sigset_t结构中。线程可以通过把set参数设置为NULL，并把oset设置为sigset_t结构的地址，来获取当前的信号屏蔽字。这种情况中的how参数会被忽略

线程可以调用sigwait等待一个或多个信号的出现

```cpp
#include <signal.h>

int sigwait(const sigset_t* restrict set, int* restrict signop);

若成功，返回0；若出错，返回错误编号
```

set参数指定了线程等待的信号集。返回时，signop指向的整数将包含发送的信号值

如果信号集的某个信号在sigwait调用的时候处于挂起状态，那么sigwait将无阻塞的返回。在返回之前，sigwait将从进程中移除那些处于挂起等待状态的信号。如果具体实现支持排队信号，并且信号的多个实例被挂起，那么sigwait将会移除该信号的一个实例，其他的实例还要继续排队。

为了避免错误行为发生，线程在调用sigwait之前，必须阻塞那些正在等待的信号。sigwait函数会原子的取消信号集的阻塞状态，直到有新的信号被递送。在返回之前，sigwait将恢复线程的信号屏蔽字。如果信号在sigwait被调用的时候没有被阻塞，那么在线程完成对sigwait的调用之前会出现一个时间窗，在这个时间窗中，信号就可以被发送给线程

使用sigwait的好处在于它可以简化信号处理，允许把异步产生的信号用同步的方式处理。为了防止信号中断线程，可以把信号加到每个线程的信号屏蔽字中。然后可以安排专用线程处理信号。这些专用线程可以进行函数调用，不需要担心在信号处理程序中调用哪些函数是安全的，因为这些函数调用来自正常的线程上下文，而非会中断线程正常执行的传统信号处理程序

如果多个线程在sigwait的调用中，因等待同一个信号而阻塞，那么信号在递送的时候，就只有一个线程可以从sigwait中返回。如果一个信号被捕获（例如进程通过使用sigaction建立了一个信号处理程序），而且一个线程正在sigwait调用中等待同一信号，那么这时将由操作系统实现来决定以何种方式递送信号。操作系统实现可以让sigwait返回，也可以激活信号处理程序，但这两种情况不会同时发生

要把信号发送给进程可以调用kill。要把信号发送给线程，可以调用pthread_kill

```cpp
#include <pthread.h>

int pthread_kill(pthread_t tid, int signo);

若成功，返回0；若出错，返回错误编号
```

可以传一个0值的signo来检查线程是否存在。如果信号的默认处理动作是终止该进程，那么把信号传递给某个线程仍然会杀死整个进程。

注意闹钟定时器是进程资源，并且所有的线程共享相同的闹钟。所以，进程中的多个线程不可能互不干扰（或互不合作）的使用闹钟定时器

## 线程和fork

当线程调用fork时，就为子进程创建了整个地址空间的副本。子进程与父进程是完全不同的进程，只要两者都没有对内存内容做出改动，父进程和子进程之间还可以共享内存页的副本。

子进程通过继承整个地址空间的副本，还从子进程那里继承了每个互斥量、读写锁和条件变量的状态。如果父进程包含一个以上的线程，子进程在fork返回以后，如果紧接着不是马上调用exec的话，就需要清理锁状态。

在子进程内部，只存在一个线程，它是由父进程中调用fork的线程的副本构成的。如果父进程中的线程占有锁，子进程将同样占有这些锁。问题是子进程并不包含占有锁的线程的副本，所以子进程没办法知道它占有了哪些锁、需要释放哪些锁

如果子进程从fork返回以后马上调用其中一个exec函数，就可以避免这样的问题。这种情况下，旧的地址空间就被丢弃，所以锁的状态无关紧要。但是如果子进程需要继续做处理工作的话，这种策略就行不通，还需要使用其他的策略

在多线程的进程中，为了避免不一致状态的问题，POSIX.1声明，在fork返回和子进程调用其中一个exec函数之间，子进程只能调用异步信号安全的函数。这就限制了在调用exec之前子进程能做什么，但不涉及子进程中锁状态的问题。

如果要清除锁状态，可以通过调用pthread_atfork函数建立fork处理程序

## 线程和IO

pread和pwrite函数在多线程环境下是非常有用的，因为进程中的所有线程共享相同的文件描述符。其偏移量的设定和数据读取或写入是一个原子操作。可以用来解决并发线程对同一文件的读写操作问题