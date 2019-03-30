title: Lambda表达式
date: 2019-03-28 11:35:53
tags: [Java, C++, Python, Lambda表达式, 匿名函数]
---
# 前言
最近在看Elasticsearch的相关内容，在Java中遇到了Lambda表达式，因此想了解一下Lambda表达式，这里主要了解Java，C++，Python中的Lambda表达式

# 什么是Lambda表达式
在编程中，Lambda表达式就是指匿名函数（anonymous function），它是一类无需定义标识符（函数名）的函数
匿名函数通常作为参数传递给高阶函数或者作为高阶函数的返回值
如果函数只使用一次或者有限的次数，则使用匿名函数会比较方便
现在越来越多的编程语言支持匿名函数

闭包常常被用作匿名函数的同义词，严格来说，匿名函数仅仅是没有名字的函数，闭包是任何能够访问自由变量（free variable or non-local variable，不在局部和参数列表里定义的变量）的函数实例

# Java中的Lambda表达式
## 语法
Java 中的 Lambda 表达式通常使用 (argument) -> (body) 语法书写，例如：
```
(arg1, arg2...) -> { body }
(type1 arg1, type2 arg2...) -> { body }
```

在Lambda表达式中引用的自由变量，必须是final或者是实际上不会修改的变量

## 结构
1. 一个 Lambda 表达式可以有零个或多个参数
2. 参数的类型既可以明确声明，也可以根据上下文来推断。例如：(int a)与(a)效果相同
3. 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：(a, b) 或 (int a, int b) 或 (String a, int b, float c)
4. 空圆括号代表参数集为空。例如：() -> 42
5. 当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：a -> return a*a
6. Lambda 表达式的主体可包含零条或多条语句
7. 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
8. 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

## 一些例子
```
// 不需要参数，返回值为int的42
() -> 42

// 不需要参数，返回值为float的3.1415
() -> { return 3.1415 };

// 接受两个int参数，返回值为它们的和
(int a, int b) -> {  return a + b; }

// 不需要参数，没有返回值，输出一个字符串
() -> System.out.println("Hello World");

// 接受一个String参数，没有返回值，打印该字符串
(String s) -> { System.out.println(s); }
```

## 函数式接口
Java中只有一个方法声明的接口叫做函数式接口，例如java.lang.Runnable接口只声明了一个void run()方法
不指明函数式接口时，编译器可以进行自动转化
```
Runnable r = () -> System.out.println("Hello Java!");
new Thread(r).start();
new Thread(() -> System.out.println("Hello Java!")).start();
```

# C++中的Lambda表达式
在C++ 11中支持Lambda表达式
## 语法
Lambda表达式的语法如下
1. []中可以以值或引用捕获变量，来实现闭包
2. 捕获变量时需要注意，如果调用闭包时，引用了生存周期结束的变量，则发生未定义行为，C++的闭包不会延长被捕获引用的生存周期
3. Lambda表达式是纯右值表达式，其类型是独有的无名非联合非聚合类类型，被称为闭包类型
4. this是一个特殊的捕获，必须定义在类的非静态成员函数中，该Lambda具有访问该类的保护和私有成员的权限，在Lambda中访问该类的成员，不需要显式使用this->，即this->a和a等价
5. 不捕获变量的Lambda可以将函数赋值给对应的函数指针
6. Lambda表达式可以赋值给std::function

Boost库提供了自己的Lambda表达式语法，参见[Boost.Lambda](https://www.boost.org/doc/libs/1_62_0/doc/html/lambda.html)
```
[ captures ] ( params ) -> return_type { body }
// 忽略返回值，返回值的类型使用自动推断
[ captures ] ( params ) { body }
// 不接受任何参数
[ captures ] { body }
```

## 例子
```
// 接受x，y参数，返回它们的和
[](int x, int y) -> int { return x + y; }

// 下面是捕获的例子
[]        //no variables defined. Attempting to use any external variables in the lambda is an error.
[x, &y]   //x is captured by value, y is captured by reference
[&]       //any external variable is implicitly captured by reference if used
[=]       //any external variable is implicitly captured by value if used
[&, x]    //x is explicitly captured by value. Other variables will be captured by reference
[=, &z]   //z is explicitly captured by reference. Other variables will be captured by value

// Variables captured by value are constant by default. Adding mutable after the parameter list makes them non-constant.
// 按值捕获到的变量不能被改变，需要改变的话要在参数列表后加上mutalbe，按值捕获不会改变原值
a = 0;
auto f = [=] () mutable {
	printf("a = %d\n", a);
	a = 3;
	printf("a = %d\n", a);
};

printf("a = %d\n", a);
f();
printf("a = %d\n", a);

// 按引用捕获可以修改原值，a值会被改变成3
auto r = [&] {
	printf("a = %d\n", a);
	a = 3;
	printf("a = %d\n", a);
};

printf("a = %d\n", a);
r();
printf("a = %d\n", a);

class ClosureTest
{
public:
    ClosureTest() { r = 0; }
    int test();
    int result() { return r; }

private:
    int r;
};

// 捕获this
int ClosureTest::test()
{
    auto f = [&]{
        printf("%d, %d\n", this->r, r);
    };
    f();
    return 0;
}

// 将Lambda赋值给函数指针，不能捕获变量
void (*p)() = []{printf("test\n");};
p();

// 将Lambda赋值给std::function对象，可以进行捕获，void不能省略
a = 10;
std::function<int(int, int)> add = [](int a, int b){return a+b;};
std::function<void(int)> print = [](int a){printf("%d\n", a);};
std::function<void()> print = [&](){printf("%d\n", a);};
print(add(1, 2));
test();
```

# Python中的Lambda表达式
Python中可以使用lambda关键字定义匿名函数，它能用在任何需要函数的地方，在语法上它被限制只能有单个语句
```
lambda parameter_list: expression

// 例如参数x，返回x*x
lambda x: x * x

// 以上等价于
def f(x):
    return x * x

// 参数x，y，返回它们的和
lambda x, y: x + y

// 可以使用自由变量
def f(x):
    def g(y):
        return x + y
    return g  # Return a closure.

def h(x):
    return lambda y: x + y  # Return a closure.

# Assigning specific closures to variables.
a = f(1)
b = h(1)

# Using the closures stored in variables.
assert a(5) == 6
assert b(5) == 6

# Using closures without binding them to variables first.
assert f(1)(5) == 6  # f(1) is the closure.
assert h(1)(5) == 6  # h(1) is the closure.

// x是全局的
// 修改x后，会修改函数f的结果
x = 1
nums = [1, 2, 3]

def f(y):
    return x + y

// [2, 3, 4]
map(f, nums)
map(lambda y: x + y, nums)

x = 3
// [4, 5, 6]
map(f, nums)
map(lambda y: x + y, nums)
```

# Go的匿名函数
Go的匿名函数是闭包，他们可以引用自由变量
```
// 分别打出11，102，111
package main

import "fmt"

func main() {
	num := 10
	f := func(x int) int {
		return num + x
	}
	fmt.Println(f(1))
	num = 101
	fmt.Println(f(1))

	fmt.Println(test(f))
}

func test(f func(int) int) int {
	return f(10)
}
```

# 闭包的实现
闭包通常用一个包含函数代码的指针和创建闭包时函数词法环境（一系列可用的变量）的数据结构来实现
也就是说闭包引用的自由变量，需要在闭包调用的时候仍然能够访问

只在栈上分配变量的语言很难容易的实现完全的闭包，这些语言中，自动变量会在函数返回后自动的销毁
这也可以解释为什么通常支持闭包的语言也使用垃圾回收机制
替代方案是非局部变量的手动内存管理，不过这种方式可能会产生野指针的问题，C++ 11中的Lambda表达式和[GNU C中的嵌套函数](https://gcc.gnu.org/onlinedocs/gcc/Nested-Functions.html)都可能会产生这个问题

# 参考

1) [Anonymous_function][1]
2) [Closure_(computer_programming)][2]
3) [Java中的闭包之争][3]
4) [Free_variables_and_bound_variables][4]
5) [深入浅出 Java 8 Lambda 表达式][5]
6) [Lambda expressions][6]

[1]: https://en.wikipedia.org/wiki/Anonymous_function
[2]: https://en.wikipedia.org/wiki/Closure_(computer_programming)
[3]: https://sylvanassun.github.io/2017/07/30/2017-07-30-JavaClosure/
[4]: https://en.wikipedia.org/wiki/Free_variables_and_bound_variables
[5]: http://blog.oneapm.com/apm-tech/226.html
[6]: https://en.cppreference.com/w/cpp/language/lambda