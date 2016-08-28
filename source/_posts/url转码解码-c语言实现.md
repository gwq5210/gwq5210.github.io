title: url转码解码-c语言实现
date: 2016-01-22 22:07:10
tags: [url转码, c语言]
categories: 编程
---

url提交信息为何要转码呢？因为url本身会有很多特殊字符。而提交的参数中如果再有特殊字符的话，url就不能区分哪些是参数内容，哪些是分隔符。尤其是unicode，gb18030，big5等多字节的编码，不知道里面会隐藏什么字节，因此必须全部转码。

好像C语言里面没有现成的转码函数。参考[这里][1]，从php的转码中引出两个函数，进行转码。

字符'a'-'z','A'-'Z','0'-'9','.','-','*'和'_' 都不被编码，维持原值；

空格' '被转换为加号'+'。

其他每个字节都被表示成"%XY"的格式，X和Y分别代表一个十六进制位。编码为UTF-8。

<!--more-->

下面是代码实现：
```
/*************************************************************************
	> File Name: url.c
	> Author: gwq
	> Mail: gwq5210@qq.com 
	> Created Time: 2014年12月19日 星期五 14时05分41秒
 ************************************************************************/

#include <stdio.h>
#include <string.h>

#define BURSIZE 1024

int hex2dec(char c)
{
	if ('0' <= c && c <= '9') {
		return c - '0';
	} else if ('a' <= c && c <= 'f') {
		return c - 'a' + 10;
	} else if ('A' <= c && c <= 'F') {
		return c - 'A' + 10;
	} else {
		return -1;
	}
}

char dec2hex(short int c)
{
	if (0 <= c && c <= 9) {
		return c + '0';
	} else if (10 <= c && c <= 15) {
		return c + 'A' - 10;
	} else {
		return -1;
	}
}


/*
 * 编码一个url
 */
void urlencode(char url[])
{
	int i = 0;
	int len = strlen(url);
	int res_len = 0;
	char res[BURSIZE];
	for (i = 0; i < len; ++i) {
		char c = url[i];
		if (('0' <= c && c <= '9') ||
				('a' <= c && c <= 'z') ||
				('A' <= c && c <= 'Z') || c == '/' || c == '.') {
			res[res_len++] = c;
		} else {
			int j = (short int)c;
			if (j < 0)
				j += 256;
			int i1, i0;
			i1 = j / 16;
			i0 = j - i1 * 16;
			res[res_len++] = '%';
			res[res_len++] = dec2hex(i1);
			res[res_len++] = dec2hex(i0);
		}
	}
	res[res_len] = '\0';
	strcpy(url, res);
}

/*
 * 解码url
 */
void urldecode(char url[])
{
	int i = 0;
	int len = strlen(url);
	int res_len = 0;
	char res[BURSIZE];
	for (i = 0; i < len; ++i) {
		char c = url[i];
		if (c != '%') {
			res[res_len++] = c;
		} else {
			char c1 = url[++i];
			char c0 = url[++i];
			int num = 0;
			num = hex2dec(c1) * 16 + hex2dec(c0);
			res[res_len++] = num;
		}
	}
	res[res_len] = '\0';
	strcpy(url, res);
}

int main(int argc, char *argv[])
{
	char url[BURSIZE] = "http://'www.baidu.com/a b";
	urlencode(url);
	printf("%s\n", url);

	return 0;
}
```

## 参考
1) [bnuoj vjudge][2]
2) [wudishine的专栏][3]

  [1]: http://blog.csdn.net/wudishine/article/details/39025407
  [2]: https://github.com/BNUACM/bnuoj-vjudge.git
  [3]: http://abloz.com/2011/09/07/c-language-the-url-to-the-code-submit-content-to-the-site.html 
