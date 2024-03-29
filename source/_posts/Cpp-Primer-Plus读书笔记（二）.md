title: Cpp-Primer-Plus读书笔记（二）
date: 2017-07-09 14:47:52
tags: [c++, Cpp Primer Plus, 读书笔记]
---

# 第五章 循环和关系表达式

## for循环

for循环为执行重复的操作提供了循序渐进的步骤。for循环的组成部分完成下面这些步骤：

- 设置初始值
- 执行测试，看看循环是否应该继续执行
- 执行循环操作
- 更新用于测试的值

```cpp
for (initialization; test-expression; update-expression)
	body
```

C++ 语法将for看做一条语句，——虽然循环体可以包括一条或多条语句。循环只执行一次初始化，测试表达式决定循环体是否被执行，通常，这个表达式是关系表达式，即对两个值进行比较。这里可以使用任何表达式，C++将把结果强制转换为bool类型。

for循环是入口条件循环，这意味着在每轮循环之前，都将计算测试表达式得值，当测试表达式为false时，将不会执行循环体。

更新表达式在每轮循环结束时执行，此时循环体已经执行完毕。通常，它用来对跟踪循环次数的变量的值进行递增。

for循环的控制部分使用了3个表达式，任何值或任何有效的值和运算符的组合都是表达式。赋值也是一个表达式，C++将赋值表达式的值定义为左侧成员的值。

当判定表达式的值这种操作改变了内存中数据的值时，我们说表达式有副作用。判定赋值表达式的值会修改被赋值者的值。

从表达式到语句的转变很容易，只需要加分号即可。因此a=100是表达式，a=100;是语句。更准确的说，这是一条表达式语句。只要加上分号，所有的表达式都可以成为语句，但不一定有编程意义。如a+6;是合法的语句，但它没完成任何有用的工作，智能编译器甚至可能跳过这条语句

对任何表达式加上分号都可以成为语句，但是这句话反过来说就不对了。也就是说，从语句中删除分号并不一定能将它转换为表达式。对于返回语句，声明语句和for语句都不满足这种情况

C++在C的基础上添加了一项特性，要求对for循环语句做一些微妙的调整，原来的语法：

```cpp
for (expression; expression; expression)
	statement
```

但是C++的for循环允许这样做：

```cpp
for (int i = 0; i < 5; i++)
	printf("%d\n", i);
```

这很方便，但是不符合原来的语法，因为声明不是表达式。现在则使用下边的语句：

```cpp
for (for-init-statement condition; expression)
	statement
```

在for-init-statement中声明变量其实还有有用的一面，这也是该知道的。这种变量只存在与for循环中，当程序离开循环，这种变量将消失

在循环中经常使用递增运算符（++）和递减运算符（--）。这两种运算符执行两种极其常用的循环操作：将循环计数器加1或减1。这两个运算符都有两种变体。前缀版本位于操作数前面，如++x；后缀版本位于操作数后面，如x++。两个版本对操作数的影响是一样的，但是影响的时间不同。粗略的讲a++意味着使用a的当前值计算表达式，然后将a的值加1；而++b的意思是先将b的值加1，然后使用新的值来计算表达式。

递增运算符和递减运算符都是漂亮的小型运算符，不过千万不要失去控制，在同一条语句对同一个值递增或递减多次。问题在于规则使用后修改和修改后使用可能会变得模糊不清。也就是说，下面这条语句在不同的系统上将生成不同的结果：

```cpp
x = 2 * x++ * (3 - ++x);
```

对于这种语句，C++没有定义正确的行为

副作用是指在计算表达式时对某些东西进行了修改；顺序点是程序执行过程中的一个点，在这里，进入下一步之前将确保所有的副作用都进行了评估。在C++中语句中的分号就是一个顺序点，这意味着程序在处理下一条语句之前，赋值运算符、递增运算符和递减运算符执行的所有修改都必须完成。本章后边将讨论的有些操作也有顺序点。另外任何完整的表达式末尾都是一个顺序点。完整表达式是这样一个表达式：不是另一个更大表达式的子表达式

前缀格式和后缀格式的执行速度可能存在细微的差别，对于内置类型和当代的编译器而言，这看似不是什么问题。然而C++允许你针对类定义这些运算符，这种情况下，前缀格式的效率更高

可以将递增或递减运算符用于指针和基本变量。也可以结合使用这些运算符和\*运算符来修改指针指向的值。前缀递增、前缀递减和解除引用运算符的优先级相同，以从右到左的方式结合。后缀递增和后缀递减的优先级相同，但比前缀运算符的优先级高，这两个运算符以从左到右的方式进行结合。\*++p的含义如下：先将++应用于pt，然后将\*应用于被递增后的pt，另一方面++\*pt意味着先取得pt指向的值，然后将这个值加1。(\*p)++，圆括号指出首先对指针解除引用，然后再进行递增。\*pt++后缀运算符++的优先级更高，这意味着将运算符应用于pt而不是\*pt。

可以使用组合赋值运算符来进行赋值：

| 操作符  | 操作数（L为左操作数，R为右操作数） |
| ---- | ------------------ |
| +=   | 将L+R赋值给L           |
| -=   | 将L-R赋值给L           |
| *=   | 将L*R赋值给L           |
| /=   | 将L/R赋值给L           |
| %=   | 将L%R赋值给L           |

C++使用花括号来构造一条复合语句（代码块）。代码块由一对花括号和它们包含的语句组成，被视为一条语句，从而满足句法的要求

复合语句还有一个有趣的特性，如果在语句块中定义一个新的变量，则仅当程序执行该语句块中的语句时，该变量才存在。执行完毕该语句块后，变量将被释放。

在外部语句块中定义的变量在内部语句块中也是被定义了的。

如果在一个语句块中声明一个变量，而外部语句块中也有一个这种名称的变量，在声明位置到内部语句块结束的范围内，新的变量将隐藏旧变量，然后旧变量再次可见

逗号表达式允许将两个表达式放到C++句法只允许放一个表达式的地方。逗号并不总是逗号运算符，在声明中的逗号是分隔符

逗号表达式还有其他特性。首先，它确保先计算第一个表达式，然后计算第二个表达式，即逗号运算符是一个顺序点；其次，C++规定逗号表达式的值是第二部分的值

C++提供了6种关系运算符来对数字进行比较：

| 操作符  | 含义    |
| ---- | ----- |
| <    | 小于    |
| <=   | 小于或等于 |
| ==   | 等于    |
| >    | 大于    |
| >=   | 大于或等于 |
| !=   | 不等于   |

关系运算符的优先级比算符运算符低

不要混淆等于运算符（==）与赋值运算符（=）

C风格字符串比较，不能直接使用等于运算符进行比较，这样比较的是两个字符串的地址。对于string类字符串，可以直接使用等于运算符进行比较

## while循环

while循环是没有初始化和更新部分的for循环：

```cpp
while (test-condition)
	body
```

while也是一种入口条件循环，因此如果测试条件一开始就是false，则程序将不会执行循环体

while循环和for循环可以进行相互转换；省略for循环中的测试表达式时，测试结果将为true

在设计循环时，记住如下的原则：

- 指定循环终止的条件
- 在首次测试之前初始化条件
- 在条件再次被测试之前更新条件

在循环中，不要使用错误的分号，如：

```cpp
int i = 0;
int cnt = 10;
while (i < cnt);   // 多了一个分号
{
  printf("%d\n", i);
}
```

C++为类型建立别名的方式有两种，一种是使用预处理器：

```cpp
#define BYTE char
```

这样程序将使用char替换BYTE
第二种方式是使用C++的关键字typedef

```cpp
typedef char byte; // typedef type_name alias_name;
```

使用define和使用typedef的区别在于，在声明一系列变量时，define不在有效

do while循环不同于for循环和while循环，它是出口条件循环。这意味着这种循环将首先执行循环体，然后再判定测试表达式，决定是否继续执行循环。

```cpp
do
	body
while (test-expression);
```

三种循环各有应用场景，需要合适选择

C++新增了基于范围的for循环。这简化了一种常见的循环任务：对数组或容器的每个元素执行相同的操作，后者提供了修改元素的方法。

```cpp
vector<int> v = {1, 2, 3};
for (int x: v)
{
  printf("%d\n", x);
}
for (int &x: v)
{
  x = 1;
}
```

cin对象支持三种不同模式的单字符输入：

```cpp
char ch;
cin >> ch;		// 忽略空白
ch = cin.get(); // return char
cin.get(ch);	// return istream
```

可以使用文件尾条件（EOF）来表示输入的结束。检测到EOF后，cin将两位（eofbit和failbit）都设置为1，可以通过成员函数eof()来查看eofbit，如果eofbit或failbit被设置，fail函数返回true。eof和fail方法报告最近读取的结果，也就是说它们在事后报告，而不是预先报告。因此应将这两个测试放在读取后

istream提供了一个可以将istream转换为bool的函数，当cin出现在bool需要的地方时，该转换函数将被调用。如果最后一次读取成功，则转换得到的bool值为true；否则为false。因此输入可以如下：

```cpp
while (cin.get(ch))
{
  
}
```

二维数组更像是一个表格——既有数据行又有数据列。

# 第六章 分支语句和逻辑运算符

if语句让程序能够决定是否执行特定的语句块

```cpp
if (test-condition)
	statement
```

if else语句让程序决定执行两条语句中的哪一个

```cpp
if (test-condition)
	statement1
else
	statement2
```

if else中的两个操作必须都是一条语句，如果需要多条语句，则需要使用大括号将它们括起来，组成一个语句块

if else语句可以互相嵌套，组成多种选择

可以将variable == value转换成value == variable，以此来捕获将相等运算符误写为赋值运算符的错误

逻辑运算符分为逻辑或，逻辑与，逻辑非

C++可以采用逻辑或运算符(||)，将两个表达式组合起来。如果原来表达式中的任何一个或全部都为true，则得到的表达式的值为true；否则，表达式的值为false

||运算符的优先级低于算术运算符

C++规定，||是一个顺序点，也就是说先修改左边的值，再对右侧的值进行判定。当能够确定整个表达式的值时，就不再往下运算了，也就是说||运算符有短路特性

对于逻辑与，仅当所有的表达式都为true时，得到的表达式才为true

&&运算符的优先级低于算术运算符

和||运算符一样，&&运算符也是顺序点，同样&&也有短路特性

对于取值范围，不要写成3<a<5，这样不能正确表示范围，编译器不会捕获这种错误，因为它仍然是合法的C++语句

逻辑非(!)对它后边的表达式取反

C++逻辑与和逻辑或运算符的优先级低于算术运算符，!运算符的优先级高于所有的关系运算符和算术运算符。因此如果对表达式取反，必须使用括号将其括起来

编写复杂的逻辑表达式，还是加上括号，降低出现错误的可能性

C++确保程序从左到右进行计算逻辑表达式，并在知道答案后立刻停止

并不是所有键盘都提供了用作逻辑运算符的符号，因此C++标准提供了另一种表示方式。标识符and，or，not都是C++保留字，这意味着不能将它们用作变量名等。它们不是关键字，因为它们都是已有语言特性的另一种表示方式。另外它们不是C的保留字。但C语言可以将它们用作运算符，只要包含了iso646.h头文件，C++并不要求头文件

C++从C语言继承了一个与字符相关的、非常方便的函数软件包，可以简化确定字母，数字等的工作。函数原型在cctype中。使用这些函数更加通用，例如在有的系统上，可能字母并不是连续编码的

| 函数名称       | 返回值                                      |
| ---------- | ---------------------------------------- |
| isalum()   | 如果参数是字母或数字，返回true                        |
| isalpha()  | 如果参数是字母，返回true                           |
| iscntrl()  | 如果参数是控制字符，返回true                         |
| isdigit()  | 如果参数是数字，返回true                           |
| isgraph()  | 如果参数是除空格外的打印字符，该函数返回true                 |
| islower()  | 如果参数是小写字母，返回true                         |
| isprint()  | 如果参数是打印字符(包括空格)，该函数返回true                |
| ispunct()  | 如果参数是标点符号，返回true                         |
| isspace()  | 如果参数是标准空白符，如空格，进纸，换行符，回车，水平制表符，或者垂直制表符，返回true |
| issupper() | 如果参数是大写字母，返回true                         |
| isxdigit() | 如果参数是16进制数字，0-9，a-f,A-F，返回true           |
| tolower()  | 如果参数是大写字母，则返回小写，否则返回该参数                  |
| toupper()  | 如果参数是小写字母，则返回大写，否则返回该参数                  |

C++有一个常被用来代替if else语句的运算符，这个运算符被称为条件运算符(?:)，它是C++中唯一一个需要3个操作数的运算符

```cpp
expression1?expression2:expression3;
```

如果expression1为true，则整个表达式的值为expression2，否则，表达式的值为expression3

switch语句提供了分支选择的另一种方法

```cpp
switch(integer expression)
{
  case label1: statement(s)
  case label1: statement(s)
  case label1: statement(s)
  ...
  defaule: statement(s)
}
```

程序将跳到integer expression的值标记的那一行。integer expression必须是结果值为整数的表达式，每个标签必须是整数常量表达式。如果不匹配任何标签，则程序跳转到default的那一行。default是可选的，如果被省略，有没有匹配的标签，则程序直接跳转到switch后边的语句执行

跳转到switch语句中的标签后，将会依次执行后边的代码，除非有明确的其他指示，如break指令

如果既可以使用switch和if else，选项不少于3项时，则可以考虑使用switch

break和continue语句都能使程序跳过部分代码。可以在switch语句中使用break，使程序跳转到switch后执行语句。continue和break都可以用到循环中。break使程序跳出循环，continue使程序跳过本次循环，开始一个新的循环，其不会跳过更新表达式

C++使得将读取屏幕输入输出的技巧应用于文件输入输出中。

可以使用ifstream和ofstream类创建对象，关联到相应的文件，对文件进行读取和写入。类似于使用cin和cout对象

# 第七章 函数——C++的编程模块

使用函数，必须完成如下工作：

- 提供函数定义
- 提供函数原型
- 调用函数

函数可以分为有返回值的函数和没有返回值的函数

```cpp
void func_name(parameter_list)
{
  statement(s);
  return;
}
type_name func_name(parameter_list)
{
  statement
  return value;	// value is type cast to type type_name
}
```

对于有返回值的函数，必须使用返回语句，以便将值返回给调用函数。结果的类型必须是type_name类型或者可以被转换成type_name类型。C++的函数返回值不能是数组。

通常函数通过将返回值复制到指定的CPU寄存器或内存单元中来将其返回。

函数原型描述了函数到编译器的接口，也就是说，它将函数返回值的类型以及参数的类型和数量告诉编译器。通常在函数原型的参数列表中，可以包括变量名，也可以不包括。原型中的变量名相当于占位符，因此不必与函数定义中的变量名相同。

在C++中，括号为空与在括号中使用关键字void是等效的，但是在C中，括号意味着不指出参数——这意味着将在后边定义参数列表。但在C++中不指定参数列表时应使用省略号。

原型有以下几点作用：

- 编译器正确的处理函数返回值
- 编译器检查使用的参数数目是否正确
- 编译器检查使用的参数类型是否正确，如果不正确，则转换成正确的类型(如果可能的话)

C++通常按值传递参数。在函数中声明的变量(包括参数)是该函数私有的。C++标准用参数(argument)表示实参，使用参量(parameter)来表示形参。

在C++中当且仅当用于函数头或函数原型中，int *arr和int arr[]的含义才是相同的。

可以用const来声明一个指针常量，让指针指向一个常量对象，这样可以防止使用指针来修改所指向的值。但是指针本身是可以修改的，即指向另外的常量对象。如果只涉及一级间接关系，则可以将非const指针赋值给const指针，但是进入两级间接关系时，这种方式是不允许的。
在指针参数中，尽可能使用const，可以避免无意对数据的修改，使得能够处理const和非const参数
另一种是声明一个常量指针，即指针本身的指向是不能修改的。

```cpp
const int *p = NULL;
int * const p = NULL;

const int **pp;
int *p;
const int a = 0;
pp = &p; 不允许，假设允许
*pp = &a; 合法，都是const，但是将p设置为a的地址
*p = 1; 合法，但是修改了const a
```

对于在参数中的二维数组或多维数组，必须指出除第一维的数组长度。否则，函数将无法处理。因为整个数组的指针是需要指定数组长度的

可以通过三种方式将C-风格字符串传递给函数：

- char数组
- 用引号括起来的字符串常量
- 被设置为字符串地址的char指针

对于string字符串和模板类array，可以像使用基本类型一样使用。

使用结构作为参数时，可以像处理基本类型一样处理结构。然而，这种默认的按值传递结构的方式有一个缺点，如果结构比较大，则复制结构将增加内存消耗，我们可以通过传递结构指针来减少消耗。

C++函数可以调用自己(然而，与C语言不同的是C++不允许main函数调用自己)，这种功能被称为递归。递归需要在合适的条件下终止，否则函数将一直递归下去，直到耗尽内存。

与数据项相似，函数也有地址，函数的地址是存储其机器语言代码的内存的开始地址。

函数名就是函数的地址。

通常，如果要声明函数指针，可以首先编写这种函数的原型，然后使用(*pf)替换函数名，这样pf就是这类函数的指针。优先级要求使用括号，才能定义正确的函数指针。

如果pf是函数指针，可以有两种方式调用该函数(*pf)()和pf()，其实其中存在着逻辑冲突。但是C++允许这两种语法同时存在。

函数指针数组：

```cpp
int f();
int (*pf)();
int (*pf_arr[10])();
```

可以使用typedef简化函数指针的定义

```cpp
typedef int *(*pf)();
int *f();
pf func = f;
```

# 第八章 函数探幽

## 内联函数

常规函数和内联函数的区别不在于编写方式，而在于C++编译器如何将它们组合到程序中。

使用函数需要一定的开销。执行到函数调用指令时，程序将在函数调用后立即存储该指令的内存地址，并将函数参数复制到堆栈中，跳到标记函数起点的内存单元，执行函数代码，也许还要将返回值放入寄存器中，然后跳跃到地址被保存的指令处。对于内联函数，编译器将使用相应的函数代码替换函数调用。这样程序无需来回跳跃，执行速度稍快，代价是需要占用更多的内存。应有选择性的使用内联函数。

为了定义内联函数，我们必须在函数声明前或者函数定义前加上关键字inline。但是编译器不一定会满足这种需求。

在C语言中使用宏也可以实现类似的功能，但是宏只是文本替换，可能造成问题。

## 引用变量

引用是已定义的变量的别名。

必须在声明引用变量时进行初始化。引用实际上更接近const指针。

引用经常被作为函数参数，这种传递参数的方法叫做按引用传递。

引用变量作用函数参数，如果不需要修改参数的值，可以将其声明为const引用。

如果引用参数是const，则编译器将在下面两种情况下生成临时变量：

- 实参的类型正确，但不是左值
- 实参的类型不正确，但可以转换为正确的类型

左值参数是可以被引用的数据对象。常规变量和const变量都可以视为左值，一个可以被修改，一个不能被修改。

对于常量引用，将不会生成临时变量。如果是const引用参数，如果实参不匹配，则其行为类似于按值传递。

我们应尽可能的使用const：

- 可以避免无意修改数据
- 可以处理const和非const实参
- 能够正确的生成并使用临时变量

函数可以返回引用，其实际上是被引用变量的别名。应避免返回函数终止时不再存在的内存单元的引用。

返回引用类型的函数是左值，我们可以对其进行赋值，如果不希望这样做，可以将返回类型声明为const引用。但常规返回类型是右值，这种返回值位于临时内存单元中，运行到下一条语句，它们可能已经不存在了

```cpp
func() = 1; // 合法
```

C风格字符串可以传递给const string引用。

对于集成的对象，基类的引用可以指向派生类对象，无需进行强制转换。

使用引用参数的原因主要有两个：

- 能够修改调用函数中的数据对象
- 通过传递引用而不是整个数据对象，可以提高程序的运行速度

对于使用传递的值而不做修改的函数：

- 如果数据对象很小，如内置数据类型，则按值传递
- 如果数据对象是数组，则使用指针，这是唯一的选择，并将指针声明为指向const的指针
- 如果数据对象是较大的结构，则使用const指针或const引用，可以提高程序效率。节省复制结构所需的时间和空间
- 如果数据对象是类对象，则使用const引用。传递类对象参数的标准方式是按引用传递

对于修改调用函数中数据的函数：

- 如果数据对象是内置数据类型，则使用指针。当然也可以使用引用
- 如果数据对象是数组，则只能使用指针
- 如果数据对象是结构，则使用指针或引用
- 如果数据对象是类对象，则使用引用

## 默认参数

默认参数指的是当函数调用中省略了实参时自动使用的一个值。

通过函数原型来设置默认参数。对于带参数列表的函数，必须从右往左添加默认值。如果不指定实参，则使用默认值，如果指定实参，则使用实参的值。

## 函数重载

函数重载可以使用多个同名的函数。其关键是函数的参数列表，也称为函数的特征标。

如果两个函数的参数的个数和类型相同，同时参数的排列顺序也相同，则它们的特征标相同。函数重载的条件是函数的特征标不同。

我们可以对引用参数进行重载：

```cpp
double x = 1.0;
const double y = 2.0;
void stove(double &r);	// 匹配可修改的左值，x
void stove(const double &r);	// 匹配const左值，y
void stove(double &&r);			// 匹配右值，x+y，如果没有定义，将匹配上一个函数
```

仅当函数基本执行相同任务，但使用不同形式的数据时，才应该采用函数重载。

编译器对每个重载函数进行名称修饰，以此来定位每个重载的函数。

## 函数模板

函数模板是通用的函数描述，它使用泛型来定义函数，其中的泛型可以用具体的类型替换。通过将类型作为参数传递给模板，可使编译器生成该类型的函数。

模板的一个用法是希望将一个算法应用到不同的数据类型中。例如排序算法。

```cpp
template <typename T> // template <class T>
void swap(T &a, T &b);
```

在最终生成的代码里不包含模板，只包含实际生成的函数。

可以对模板定义进行重载，前提也是特征标不同。

编写的模板函数可能无法处理所有的类型。例如，可能有的类没有+运算符，但是我们在模板定义中使用了。一种解决方案是重载+号运算符，另一种是为特定类型提供具体化的模板定义。

我们可以对模板函数提供一个函数具体化的定义——称为显式具体化，其中包含所需要的代码。当编译器找到与函数调用匹配的具体化定义时，将使用该定义，而不再寻找模板。
具体方法为：

- 对于给定的函数名，可以有非模板函数，模板函数和显式具体化模板函数以及它们的重载版本
- 显式具体化的原型和定义应该以template<>开头，并通过名称来指出类型
- 具体化优先于常规模板，而非模板函数优先于具体化和常规模板

```cpp
template <typename T>
void swap(T &a, T &b);

// 一下两种方式都可以
template <> swap<int>(int &a, int &b);
template <> swap(int &a, int &b);
```

另外还必须理解实例化和具体化。在代码中包含函数模板本身并不会生成函数定义，它只是一个用于生成函数定义的方案。编译器使用模板为特定类型生成函数定义时，得到的是模板实例。这种方式被称为隐式实例化，编译器通过函数调用来知道如何进行实例化。

现在C++还允许显式实例化。这意味着可以直接命令编译器创建特定的实例。声明所需的种类——用<>符号指示类型，并在声明前加上关键字template。

```cpp
template void swap<int>(int &a, int &b);
```

要区别显式具体化和显式实例化，前者并不生成函数定义。试图在同一个文件或抓换单元中使用同一种类型的显式实例和显式具体化将出错。

可以在程序中使用函数来创建显式实例化：

```cpp
int x = 0;
char m = 1;
swap<int>(x, m);
```

隐式实例化，显式实例化，显式具体化统称为具体化。它们表示的都是使用具体类型的函数定义，而不是通用描述。

对于函数重载，函数模板和函数模板重载，C++需要一个定义良好的策略，来决定为函数调用使用哪一个函数定义，尤其是有多个参数时。这个过程称为重载解析。

- 第一步，创建候选函数列表。其中包含与被调函数的名称相同的函数和模板函数
- 第二步，使用候选函数列表创建可行函数列表。这些都是参数数目正确的函数，为此有一个隐式转换序列，其中包括实参类型与相应的形参类型完全匹配的情况。
- 第三步，确定是否有最佳的可行函数，如果有则使用它，否则该函数调用出错。

为了确定是否有最佳的可行函数，需要查看为使函数调用参数与可行的候选函数的参数匹配所需要进行的转换。通常，最佳到最差的顺序如下：

- 完全匹配，但常规函数优于模板
- 提升转换(char转换为int或float转换为double)
- 标准转换(int转换为char，long转换为double)
- 用户定义的转换，如类声明中定义的转换

进行完全匹配时，C++允许某些无关紧要的转换。Type本身可以是char &这样的类型。Type(argument-list)意味着用作实参的函数名与用作形参的函数指针只要返回类型和参数列表相同，就是匹配的

| 从实参                 | 到形参                    |
| ------------------- | ---------------------- |
| Type                | Type &                 |
| Type &              | Type                   |
| Type []             | *Type                  |
| Type(argument-list) | Type(*)(argument-list) |
| Type                | const Type             |
| Type                | volatile Type          |
| Type *              | const Type             |
| Type *              | volatile Type*         |

例如：

```cpp
int a = 10;
f(a);
// 下面都是完全匹配的
void f(int);
void f(const int);
void f(int &);
void (const int &);
```

有的时候即使两个函数都完全匹配，仍可以完成重载解析。首先，指向非const数据的指针和引用优先与非const指针和引用参数匹配。在上面示例中，如果只定义了函数3和4，则使用函数3。这种区别只适用于指针和引用指向的数据，如果只定义了函数1和2，将出现二义性错误。

另一种情况是其中一个是非模板函数，而另一个不是。这种情况下非模板函数优先于模板函数(包括显式具体化)。如果两个完全匹配的函数都是模板函数，则较具体的模板函数优先。例如：显式具体化将优于使用模板隐式生成的具体化。

术语最具体不一定意味着显式具体化。而是指编译器推断使用哪种类型时执行的转换最少。

```cpp
template <typename T> void f(T t);
template <typename T> void f(T *t);

int a = 10;
f(&a);
```

模板1将T解释为`int *`，模板2将T解释为int，因此两个隐式实例`f<int *>(int *)`和`f<int>(int *)`，后者被认为是更具体的，模板2已经指出函数参数时指向T的指针。

用于找出最具体模板的规则被称为函数模板的部分排序规则。

简而言之，重载解析将寻找最匹配的函数。如果只存在这样一个函数，则选择它；如果存在多个这样的函数，但其中只有一个是非模板函数，则选择该函数；如果存在多个合适的函数，且它们都为模板函数，但其中一个函数比其他函数更具体，则选择该函数。如果有多个同样合适的非模板或模板函数，但没有一个函数比其他函数更具体，则函数调用时不确定的，因此是错误的。当然，如果不存在匹配的函数，则也是错误的。

我们也可以显式的指出需要使用模板函数，即使已经有匹配的非模板函数了。

```cpp
int a = 0;
f<>(a);
f<int>(a);
```

将有多个参数的函数调用与有多个参数的原型进行匹配时，情况将非常复杂。编译器必须考虑所有参数的匹配情况。如果找到比其他可行函数都合适的函数，则选择该函数。一个函数要比其他函数都合适，其所有参数的匹配程度都必须不比其他函数差，同时至少有一个参数的匹配程度比其他函数都高

编写模板时，并不一定能知道使用的类型

```cpp
template <typename T1, typename T2
void f(T1 a, T2 b)
{
  type? c = a + b;
}
```

C++11提供了关键字decltype，给其的参数可以是表达式

```cpp
decltype(a + b) c = a + b;
```

编译器一般通过如下方式确定类型：

- 如果expression是一个没有用括号括起来的标识符，则类型与该标识符类型相同，包括const等限定符
- 如果expression是一个函数调用，则类型与函数的返回类型相同，这种情况并不会实际调用函数
- 如果expression是一个左值，则类型为指向其类型的引用。一种情况是expression是用括号括起来的标识符。括号并不会改变表达式的值和左值性
- 如果前面条件都不满足，则类型与expression的类型相同

可以结合typedef和decltype

```cpp
type decltype(a + b) type;
type c;
```

另一中函数声明语法

```cpp
template <typename T1, typename T2
? f(T1 a, T2 b)
// auto f(T1 a, T2 b) -> decltype(a + b)
{
  return a + b;
}
```

这种情况，并不能使用decltype解决，x，y并不在作用域内。
新增的语法可以用作声明和定义：

```cpp
auto f(int a, double b) - > double;
auto func(double a) -> double
{
  return a;
}
```

# 第九章 内存模型和名称空间

C++为在内存中存储数据方面提供了多种选择

## 单独编译

和C语言一样，C++也允许甚至鼓励程序员将组件函数放在独立的文件中。我们可以单独编译这些文件，然后将这些文件链接起来。

我们一般可以将程序分为三部分：

- 头文件：包含结构声明和使用这些结构的函数的原型
- 源代码文件：包含与结构有关的函数的代码
- 源代码文件：包含调用与结构相关的函数的代码

在头文件中，我们一般包含如下内容：

- 函数原型
- 使用#define或const定义的符号常量
- 结构声明
- 类声明
- 模板声明
- 内联函数

通常我们需要保证一个头文件只包含一次。

## 存储持续性、作用域和链接性

C++使用3种(在C++11中是4种)不同的方案来存储数据，这些方案的区别在于数据保留在内存中的时间：

- 自动存储持续性：在函数定义中声明的包括参数的存储持续性为自动的
- 静态存储持续性：在函数定义外定义的变量和使用关键字static定义的变量的存储持续性都为静态。
- 线程存储持续性(C++11)：我们使用关键字thread_local声明这种变量，其生命周期与所属的线程一样长
- 动态存储持续性：用new来分配，和delete来释放。也被称为堆或自由存储

作用域描述了名称在文件中的多大范围可见。链接性描述了名称如何在不同单元间共享。链接性为外部的名称可以在文件间共享，链接性为内部的名称只能由一个文件中的函数共享。

C++变量的作用于有多种，作用域为局部的变量只在定义它的代码块中可用。作用域为全局的变量在定义位置到文件结束之间都可用。在函数原型作用域中使用的名称只在包含参数列表的括号中可用。在类中声明的成员的作用域为整个类。在名称空间中声明的变量的作用域为整个名称空间。全局作用域是名称空间作用域的特例。

C++函数的作用域可以是整个类或整个名称空间(包括全局的)，但不能是局部的。不同的C++存储方式是通过存储持续性、作用域和链接性来描述的。

### 自动存储持续性

默认情况下，在函数中声明的函数参数和变量的存储持续性为自动，作用域为局部，没有链接性。如果代码块中定义了变量，则该变量的存在时间和作用域将被限制在该代码块中。如果存在同名变量，则较小作用域的变量会覆盖较大作用域的变量。

C++11中的auto关键字用于自动类型推断。和之前的含义截然不同。C++11中的register关键字替代了之前的auto关键字，来指出变量是自动的，不再建议编译器使用CPU寄存器来存储自动变量。

自动变量一般存储在栈中，栈是LIFO。这种设计简化了参数传递，参数一般从右往左进入栈中，这样也实现了可变参数。

### 静态存储变量

静态存储变量有三种链接性：

- 外部链接性，可在其他文件中访问，在代码块的外边声明它
- 内部链接性，只能在当前文件中访问，在代码块的外边声明它，并使用static限定符
- 无链接性，只能在当前函数或代码块中访问，在代码块中声明它，并使用static限定符

这三种链接性都在整个程序执行期间存在。编译器分配固定的内存块来存储所有的静态变量。如果静态变量没有显示的初始化，编译器将把它设置为0。

所有的静态持续变量都有下述初始化特征：未被初始化的静态变量的所有位都被设置为0，这种变量被称为零初始化的。

| 存储描述     | 持续性  | 作用域  | 链接性  | 如何声明                |
| -------- | ---- | ---- | ---- | ------------------- |
| 自动       | 自动   | 代码块  | 无    | 在代码块中               |
| 寄存器      | 自动   | 代码块  | 无    | 在代码块中，使用关键字register |
| 静态，无连接性  | 静态   | 代码块  | 无    | 在代码块中，使用关键字static   |
| 静态，外部链接性 | 静态   | 文件   | 外部   | 不在任何函数中             |
| 静态，内部链接性 | 静态   | 文件   | 内部   | 不再任何函数中，使用关键字static |

除了默认的零初始化外，还可对静态变量进行常量表达式初始化和动态初始化。零初始化和常量表达式初始化被统称为静态初始化。这意味着在编译器处理文件时初始化变量。动态初始化意味着变量在编译后初始化。

C++11新增了关键字constexpr，这增加了创建常量表达式的方式。

### 静态持续性、外部链接性

链接性为外部的变量通常简称为外部变量，它们的存储持续性为静态，作用域为整个文件。外部变量是在函数外定义的，因此对所有函数而言都是外部的。

单定义规则指出变量只能定义一次，C++提供了两种变量声明。一种是定义声明，简称定义，它给变量分配存储空间，另一种是引用声明，简称声明，它不给变量分配存储空间，它引用已有的变量；引用声明使用关键字extern，且不进行初始化，否则为定义。

单定义规则并不意味这不能定义相同名字的变量。因为定义可以被覆盖。

### 静态持续性、内部链接性

将static限定符用于作用域为整个文件的变量时，该变量的链接性定义为内部的。我们不能在两个文件中分别定义同名的变量，这违反了单定义规则；但是一个内部静态变量能够隐藏在另一个文件中定义的外部静态变量。

## 静态存储持续性、无链接性

静态局部变量的值在程序运行期间一直存在，初始化只初始化一次。

### 说明符和限定符

有些称为存储说明符或cv限定符的关键字提供了其他有关存储的信息。下边是存储说明符：

- auto，C++11中不再是说明符
- register
- static
- extern
- thread_local
- mutable

关键字mutable的含义将根据const来解释。cv限定符包含const和volatile。const我们已经介绍过了。volatile表明即使一个程序代码没有对内存单元进行修改，其值也可能发生变化，例如访问某些硬件。这样编译器就不会对这个变量进行特殊优化。

mutable指出即使结构或类为const，某个成员也可以被修改。

在C++中，但不是在C语言中，const限定符对默认存储类型稍有影响。在默认情况下，全局变量的链接性为外部的，但const全局变量的链接性为内部的。对于const变量，我们需要使用extern指出其链接性为外部的。

### 函数和链接性

函数的链接性可选的范围比变量小。默认情况下，函数的链接性为外部的，可以在文件间共享。对于函数来说extern关键字是可选的。使用关键字static来指出函数链接性为内部的，只能在当前文件中使用，必须同时在原型和函数定义中使用该关键字。

单定义规则也适用于非内联函数。内联函数不受这个规则的约束，我们允许将内联函数定义在头文件中。要求所有内联函数的定义都必须相同。

### 语言链接性

语言链接性对函数也有影响。链接程序要求每个不同的函数都有不同的符号名。C语言中，一个名称只对应一个函数，因此这很容易实现，编译器可能将func这样的函数翻译为_func。这被称为C语言链接性。但在C++中，函数重载导致一个名称对应多个函数，我们需要为这些函数生成不同的名称，例如加上参数列表的类型信息，这被称为C++语言链接性。这就导致了链接程序寻找函数时的方法不同。如果要在C++中使用C库中预编译的函数，我们可以使用函数原型来指出希望的约定：

```cpp
extern "C" void func(int); // c
extern void func(int);	// c++
extern "C++" func(int);  // c++
```

### 存储方案和动态分配

对于new或malloc分配的内存，我们有自己控制这些内存生存周期的能力。

new和delete将被转换为对应的函数，我们可以提供自己的函数来替换现有的new函数

new还提供一种变体，被称为定位new运算符。我们能够指定存放的位置。我们必须包含new头文件。

```cpp
char buffer[1024];
int *p3 = new (buffer)int[10];
```

定位new运算符并不跟踪哪些内存已经被使用，两次使用将返回相同的地址，先前的数据将被覆盖。我们不能使用delete来释放这些内存。

对于定位new函数，我们不可以替换，但可以进行重载。

## 名称空间

C++提供的名称空间工具，可以更好的控制名称的作用域。

### 传统的C++名称空间

声明区域是可以进行声明的区域。潜在作用域从声明点开始，到其声明区域的结尾。变量在潜在作用域内并不一定都是可见的。

C++可以通过定义一种新的声明区域来创建命名的名称空间，提供一个声明名称的区域。名称空间可以是全局的，也可以位于另一个名称空间中，但不能位于代码块中。在默认情况下，名称空间中声明的名称的链接性是外部的，除非它引用了常量。存在一个全局名称空间，它对应于文件声明区域，全局变量在这个名称空间中。

任何名称空间中的名称不会与其他名称空间中的名称发生冲突。我们使用作用域解析运算符::来使用名称空间中的名称。

同样我们也可以使用using声明或using编译指令来简化对名称空间的使用。前者是特定标识符可用，后者使整个名称空间可用。

using声明将特定的名称添加到它所属的声明区域中。

using声明和using编译指令是不一样的，使用using声明就好像是声明了相应的名称一样，如果一个名称已经在函数中声明了，则不能再使用using声明导入相同的名称。然而使用using编译指令将进行名称解析，局部变量将隐藏名称空间版本的同名变量。

名称空间可以进行嵌套，也可以在名称空间中使用using声明和using编译指令。using编译指令是可传递的。还可以对名称空间创建别名：

```cpp
namespace test = other_namespace;
```

我们可以创建未命名的名称空间，其中的名称的潜在作用域为从声明点到该声明区域末尾。这提供了链接性为内部的静态变量的替代品。

### 指导原则

对于名称空间，下面是指导原则：

- 使用在已命名的名称空间中声明的变量，而不是使用外部全局变量
- 使用在已命名的名称空间中声明的变量，而不是使用静态全局变量
- 如果开发了一个函数库，将其放在一个名称空间中
- 谨慎使用using编译指令
- 不要在头文件使用using编译指令
- 导入名称时，首先使用作用域解析运算符或using声明的方法
- 对于using声明，首选将其作用域设置为局部而不是全局
