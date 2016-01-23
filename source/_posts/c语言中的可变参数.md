title: c语言中的可变参数
date: 2016-01-22 22:54:30
tags: [可变参数, c语言]
categories: 编程
---

c语言中可变参数列表有关的内容在stdarg.h头文件中。这个头文件中的内容可以让函数实现类似scanf函数的功能，接收不确定个数的参数。
使用方法稍微复杂一些，必须按照如下步骤进行：
1.在函数原型中使用省略号。
2.在函数定义中创建一个va_list类型的变量。
3.用宏将该变量初始化为一个参数列表。
4.用宏访问这个参数列表。
5.用宏完成清理工作。

下面详细介绍这些步骤，函数原型中的参数列表中至少有一个后跟省略号的参数，这个参数成为parmN，如：

<!--more-->

```
// 合法
void f1(int n, ...);
// 合法
int f2(int n, const char *s, ...);
// 无效，省略号不是最后一个参数
char f3(char c1, ..., char c2);
// 无效，没有任何参数
double f4();
```

接下来，在头文件stdarg.h中声明的va_list类型代表一种数据对象，该数据对象用于存放参数列表中省略号部分代表的参数。可变参数函数的定义应该像下边这样：

```
doube sum(int lim, ...)
{
        va_list ap;        // 声明用于存放参数的变量
```

然后，函数将使用stdargs.h中定义的宏va_start()把参数列表复制到va_list变量中。宏va_start()有两个参数：va_list类型的变量和变量parmN。如前所述，这里两个参数分别为va和lim，所以，va_start()函数的调用如下所示：

```
va_start(vp, lim);        // 把vp初始化为参数列表
```

下一步是访问参数列表中的内容，这涉及宏va_arg()的使用，该宏接受两个参数：一个va_list类型的变量和一个类型名。
第一次调用va_arg()时，它返回参数列表的第一项，下次调用返回第二项，一次类推。参数类型制定返回值的类型。例如，如果参数列表中第一个参数为double类型，第二个为int类型，那么可使用下列语句：

```
double tic;
int toc;
...
tic = va_arg(ap, double);        // 取得第一个参数
toc = va_arg(ap, int);        // 取得第二个参数
```

这里需要注意的是，实际参数的类型必须与说明的类型相匹配，如果第一个参数为10.0，那么前面的tic部分的代码正常工作；但是如果参数为10，代码就可能无法正常工作。这里不会像赋值过程中那样进行double到int的自动转换。
最后，应该使用va_end()完成清理工作。该宏接受一个va_list变量作为参数：

```
va_end(ap);        // 清理工作
```

此后，只有用va_start()重新对ap初始化后，才能使用变量ap。
因为va_arg()不提供后退回先前参数的方法，所以保存va_list变量的副本是有用的，C99为此专门添加了宏va_copy()。该宏的两个参数均为va_list类型变量，它将第二个参数复制到第一个参数中：

```
va_list ap;
va_list apcopy;
double tic;
int toc;
...
va_start(ap, lim);        // 把ap初始化为参数列表
va_copy(apcopy, ap);       // apcopy是ap一个副本
tic = va_arg(ap, double);        // 取得第一个参数
toc = va_arg(ap, int);        // 取得第二个参数
```

这时，虽然已从ap中删除了前两项，但是还可以从apcopy中重新获取这两项。
下面是一个例子：

```
/*************************************************************************
    > File Name: varargs.c
    > Author: gwq
    > Mail: gwq5210@qq.com 
    > Created Time: 2015年01月07日 星期三 20时13分16秒
 ************************************************************************/

#include <stdio.h>
#include <stdarg.h>

double sum(int num, ...)
{
    va_list ap;

    int i = 0;
    double res = 0.0;
    va_start(ap, num);
    for (i = 0; i < num; ++i) {
        res += va_arg(ap, double);
    }
    va_end(ap);
    return res;
}

int main(int argc, char *argv[])
{
    double s = 0.0;

    s = sum(3, 1.1, 2.5, 13.3);
    printf("sum(3, 1.1, 2.5, 13.3) = %.2f\n", s);
    // 如果不对应，会出现错误，这与放的位置有关
    //s = sum(3, 1, 2.5, 13.3);
    //printf("sum(3, 1, 2.5, 13.3) = %.2f\n", s);
    s = sum(6, 1.1, 2.1, 13.1, 4.1, 5.1, 6.1);
    printf("sum(6, 1.1, 2.5, 13.1, 4.1, 5.1, 6.1) = %.2f\n", s);

    return 0;
}
```

上边也测试了，传进来的参数不对应的情况，发生了错误。
下面是另一个程序，说明了va_arg传入的类型还可以是自定义的类型：

```
/*************************************************************************
    > File Name: testargs.c
    > Author: gwq
    > Mail: gwq5210@qq.com 
    > Created Time: 2015年01月07日 星期三 20时21分44秒
 ************************************************************************/

#include <stdio.h>
#include <stdarg.h>

struct point_t {
    int x, y;
};

struct point_t add(int num, ...)
{
    va_list ap;
    va_start(ap, num);

    int i = 0;
    struct point_t p;
    struct point_t res;
    res.x = 0;
    res.y = 0;
    for (i = 0; i < num; ++i) {
        p = va_arg(ap, struct point_t);
        res.x += p.x;
        res.y += p.y;
    }
    va_end(ap);
    return res;
}

int main(int argc, char *argv[])
{
    struct point_t p;
    struct point_t sum;
    p.x = 1;
    p.y = 2;

    sum = add(3, p, p, p);

    printf("add(3, p, p, p) = {%d, %d}\n", sum.x, sum.y);

    return 0;
}
```

下面说说stdio.h中与va_list相关的几个函数：
这几个函数有vprintf,vfprintf,vsprintf,vsnprintf。
函数原型分别为：

```
int vprintf(const char *format, va_list ap);
int vfprintf(FILE *stream, const char *format, va_list ap);
int vsprintf(char *str, const char *format, va_list ap);
int vsnprintf(char *str, size_t size, const char *format, va_list ap);
```

这几个函数分别与printf,fprintf,sprintf,snprintf类似，不过，他们接受一个va_list类型的变量，而不是参数列表。
利用这几个函数可以实现方便的函数如：

```
/*************************************************************************
    > File Name: execute_cmd.c
    > Author: gwq
    > Mail: gwq5210@qq.com 
    > Created Time: 2015年01月07日 星期三 20时44分17秒
 ************************************************************************/

#include <stdio.h>
#include <stdlib.h>
#include <stdarg.h>

#define BUFSIZE 1024

int execute_cmd(const char *fmt, ...)
{
    char cmd[BUFSIZE];
    va_list ap;
    va_start(ap, fmt);
    vsprintf(cmd, fmt, ap);
    va_end(ap);
    return system(cmd);
}

int main(int argc, char *argv[])
{
    execute_cmd("echo %s", "dir");
    execute_cmd("ls %s", "/");

    return 0;
}
```

这里边的大部分内容都是从书上摘下来的，程序部分经过了修改。

参考：
1）C Primer Plus（第五版）中文版 16.13
2）hustoj项目源代码