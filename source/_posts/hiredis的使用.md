title: hiredis的使用
date: 2018-01-27 16:39:49
tags: [Redis, hiredis, Redis API]
---
# 前言
近期开发的项目使用到了Redis，本文主要记录下使用Redis API遇到的一些问题。
如果想了解Redis相关的细节，可以看[《Redis设计与实现》](http://redisbook.com/)。
想了解Redis相关的命令，可以参考[Redis命令参考](http://redisdoc.com/)

# hiredis简介
hiredis是[Redis](https://redis.io/)的一个极简c客户端库。
它实现了Redis协议的最小集，但同时也提供类printf的API，使用既方便有简介。
hiredis提供了多种API，它同时支持同步，异步和回复解析API。在此基础上，能够实现Redis的流水线操作。

<!-- more -->

# 数据结构
首先来看数据结构，主要分为两部分。保存Redis连接上下文的redisContext和保存Redis命令回复的RedisReply。
下边分别介绍：
## 连接上下文redisContext
首先看结构定义：
```
/* Context for a connection to Redis */
typedef struct redisContext {
    int err; /* Error flags, 0 when there is no error */
    char errstr[128]; /* String representation of error when applicable */
    int fd;
    int flags;
    char *obuf; /* Write buffer */
    redisReader *reader; /* Protocol reader */

    enum redisConnectionType connection_type;
    struct timeval *timeout;

    struct {
        char *host;
        char *source_addr;
        int port;
    } tcp;

    struct {
        char *path;
    } unix_sock;

} redisContext;
```
可以看到结构体中包含了一些必要的字段。
obuf保存的是格式化后的Redis协议字符串，我们可以把这个字段打出来，可以理解Redis协议。
后边介绍到具体接口再来看这个结构体。

## 回复结构体redisReply
结构定义如下：
```
/* This is the reply object returned by redisCommand() */
typedef struct redisReply {
    int type; /* REDIS_REPLY_* */
    long long integer; /* The integer when type is REDIS_REPLY_INTEGER */
    size_t len; /* Length of string */
    char *str; /* Used for both REDIS_REPLY_ERROR and REDIS_REPLY_STRING */
    size_t elements; /* number of elements, for REDIS_REPLY_ARRAY */
    struct redisReply **element; /* elements vector for REDIS_REPLY_ARRAY */
} redisReply;
```
从结构体定义可以看出，它可以表示不同类型的消息。
最常用的包括：字符串，数组，整数。具体的在后边介绍。

## 总结
以上两个结构体可以帮助我们理解hiredis库的内部实现，我们总会遇到库的一些坑。只有理解库的实现，才能够避免踩到这些坑。

# 同步接口
hiredis提供了如下三个同步API：
```
redisContext *redisConnect(const char *ip, int port);
void *redisCommand(redisContext *c, const char *format, ...);
void freeReplyObject(void *reply);
```
可以看出使用同步接口分为三个步骤，创建连接，发送Redis命令，得到结果后释放回复类。

## 连接Redis
redisConnect函数使用之前提到的redisContext来保存连接的状态。
结构体redisContext的成员err非零表示连接出错，errstr保存具体的错误信息。
调用redisConnect函数后，应该检查err来确认连接是否真的建立。
```
redisContext *c = redisConnect("127.0.0.1", 6379);
if (c == NULL || c->err) {
    if (c) {
        printf("Error: %s\n", c->errstr);
        // handle error
    } else {
        printf("Can't allocate redis context\n");
    }
}
```
值得注意的是redisContext不是线程安全的。

## 向Redis发送命令
redisCommand是类printf的，可以向已经建立连接的Redis发送命令。

这个函数使用空格来区分Redis的参数。%s指明一个参数，该参数是一个\0结尾的字符串。如果想使用二进制安全的版本，则可以使用%b，其需要两个参数，一个字符指针，一个表明这个字符串长度的size_t参数。

值得注意的是%s表明一个Redis命令的参数，即使这个字符串中包含空格，回车等字符。仔细阅读hiredis的代码你就会发现这一点。

下面是一些例子：
```
reply = redisCommand(context, "SET foo bar");
reply = redisCommand(context, "SET foo %s", value);
reply = redisCommand(context, "SET foo %b", value, (size_t) valuelen);
reply = redisCommand(context, "SET key:%s %s", myid, value);

// 这个将会出错，因为%s被解释为一个参数，所以Redis收到的就是一个参数的命令‘SET foo bar’，这当然会出错
reply = redisCommand(context, "%s", "SET foo bar");
```

## 获得命令的回复
redisCommand返回值是一个redisReply的结构体，保存了命令的回复。
如果出错，则返回值为NULL并且redisContext的err字段设置为非0。
Once an error is returned the context cannot be reused and you should set up a new connection.

redisReply结构体中的type字段表明了返回的数据类型。
有如下几种枚举
```
// 表明回复是一个状态，状态的具体信息在str和len中
REDIS_REPLY_STATUS
// 表明回复是一个错误，错误信息在str和len中
REDIS_REPLY_ERROR:
// 表明回复是一个整数，整数保存在interger中，其实long long类型
REDIS_REPLY_INTEGER:
// 返回一个nil对象，表明没有数据
REDIS_REPLY_NIL:
// 回复一个字符串，字符串在str和len中
REDIS_REPLY_STRING:
// 表明回复是一个数组，保存在elements，element表明了数组的大小。可能回复是嵌套数组
REDIS_REPLY_ARRAY:
```

redisCommand获得的reply应该使用freeReplyObject函数释放内存。

Important: the current version of hiredis (0.10.0) frees replies when the asynchronous API is used. This means you should not call freeReplyObject when you use this API. The reply is cleaned up by hiredis after the callback returns. This behavior will probably change in future releases, so make sure to keep an eye on the changelog when upgrading (see issue #39).

## 断开连接
使用完Redis后，使用redisFree来断开网络连接和释放对应的内存
```
void redisFree(redisContext *c);
```

## 变种的redisCommand
由于redisCommand使用了可变参数技术，其就会有对应的va_list版本。
下边是这一系列参数的原型
```
/* Write a command to the output buffer. Use these functions in blocking mode
 * to get a pipeline of commands. */
int redisvAppendCommand(redisContext *c, const char *format, va_list ap);
int redisAppendCommand(redisContext *c, const char *format, ...);
int redisAppendCommandArgv(redisContext *c, int argc, const char **argv, const size_t *argvlen);

/* Issue a command to Redis. In a blocking context, it is identical to calling
 * redisAppendCommand, followed by redisGetReply. The function will return
 * NULL if there was an error in performing the request, otherwise it will
 * return the reply. In a non-blocking context, it is identical to calling
 * only redisAppendCommand and will always return NULL. */
void *redisvCommand(redisContext *c, const char *format, va_list ap);
void *redisCommand(redisContext *c, const char *format, ...);
void *redisCommandArgv(redisContext *c, int argc, const char **argv, const size_t *argvlen);
```

我们暂时忽略redisAppendCommand这一系列函数，我们在讲到流水线操作时会讲解它们。

可以看到这些函数与我们使用的其他可变参数函数没有什么区别。

另外一个需要介绍的函数是redisCommandArgv。
它不再使用类printf的方法来指定参数，而使用argc和argv来指明参数。另外argvlen参数还给该函数提供了使用二进制字符串的方法。如果argvlen是NULL，那么该函数将使用strlen来确定每个参数的长度。再该函数中Redis命令总是第一个参数。

## 流水线操作
我们下边解释hiredis的内部执行流程来理解为什么可以在同步的API中实现流水线操作。

当我们调用redisCommand等函数时，hiredis首先讲这些命令转化成Redis的协议。然后将转化后的命令写入输出缓冲区中，写入完成后，调用redisGetReply来获取命令的结果。该函数的流程如下：
1. 输入缓冲区非空，尝试解析单个回复并返回。如果不能解析则进行2
2. 输入缓冲区是空，将输出缓冲区的所有内容发送到Redis，然后从socket读取单个回复并返回

为了实现流水线操作，第一步就是使用redisAppendCommand或redisAppendCommandArgv填充输出缓冲区，函数原型参见上小结。

第二步我们需要调用redisGetReply，当然我们可以调用redisGetReply多次来获取多个命令的回复。我们需要判断redisGetReply的返回值来确定是否调用成功。成功返回REDIS_OK，失败返回REDIS_ERR。

下面是一个简单的例子：
```
redisReply *reply;
redisAppendCommand(context,"SET foo bar");
redisAppendCommand(context,"GET foo");
redisGetReply(context,&reply); // reply for SET
freeReplyObject(reply);
redisGetReply(context,&reply); // reply for GET
freeReplyObject(reply);
```

## 错误
当函数返回NULL或REDIS_ERR的时候，redisContext的err字段将会设置成如下枚举：
```
REDIS_ERR_IO: There was an I/O error while creating the connection, trying to write to the socket or read from the socket. If you included errno.h in your application, you can use the global errno variable to find out what is wrong.

REDIS_ERR_EOF: The server closed the connection which resulted in an empty read.

REDIS_ERR_PROTOCOL: There was an error while parsing the protocol.

REDIS_ERR_OTHER: Any other error. Currently, it is only used when a specified hostname to connect to cannot be resolved.
```
redisContext的errstr字段将会指明具体的错误信息。

# 异步API
TODO