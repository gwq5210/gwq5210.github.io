title: mathjax简单教程（翻译）
date: 2018-02-04 00:36:22
tags: [mathjax教程,LaTeX]
---
# mathjax简单教程（翻译）
原文地址：[mathjax basic tutorial and quick reference](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)

 1. 在任何的mathjax公式上，都可以使用右键点击公式选择"Show Math As > TeX Commands"来查看公式是怎么写出来的（包括这里）。
 2. 使用\$...\$来产生行内公式，使用\$\$...\$\$来产生多行公式。公式$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$（行内）和公式$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}\tag{displayed}$是不一样的。
 3. 使用\alpha,\beta,...,\omega来产生希腊字母：$\alpha$,$\beta$,$\omega$。使用\Gamma,\Delta,...,\Omega来产生大写的：$\Gamma$,$\Delta$,...,$\Omega$。
 4. 使用^和_来产生下标和上标。如x\_i^2产生：$x_i^2$。

<!-- more -->

 5. 组。下标，上标和其他运算符只能应用到下一个"组"。单个符号或者被大括号{...}包含起来的公式被视为一个组。如10^10产生$10^10$，而不是$10^{10}$，这样写可以得到预期的结果：10^{10}。使用大括号可以界定公式中那些是上标或者下标：x^5^6是错误的，{x^y}^z是${x^y}^z$，x^{y^z}是$x^{y^z}$。显然x_i^2：$x_i^2$和x_{i^2}：$x_{i^2}$是不同的。
 6. 符号()[]产生对应的符号，如(3+4)[4+4]产生：$(3+4)[4+4]$。但使用\{和\}来产生大括号：$\{\}$。
但上边的不会自动适应公式的大小，所以如果你写下(\frac12)，那么括号会很小：$(\frac12)$，使用\left(...\right)来使括号自动适应公式的大小：\left(\frac12\right)产生$\left(\frac12\right)$。
\left和\right可以作用于下边种类的括号：(和)：$\left(x\right)$，[和]：$\left[x\right]$，\{和\} ：$\left\{x\right\}$，|：$\left|x\right|$，\langle和\ rangle：$\left\langle{x}\right\rangle$，\lceil和\rceil：$\left\lceil{x}\right\rceil$，\lfloor和\rfloor：$\left\lfloor{x}\right\rfloor$。还有一种不可见的括号，用.来指代：\left.\frac12\right\}表示$\left.\frac12\right\}$。
 7. 使用\sum和\int来代表求和，积分。下标表示下限，上标表示上限，所以\sum_1^n表示$\sum_1^n$。上限和下限超过一个符号不要忘记加上{...}。如\sum_{i=0}^\infty i^2表示$\sum_{i=0}^\infty{i^2}$。类似的还有：\prod$表示\prod$，\int表示$\int$，\bigcup表示$\bigcup$，\bigcap表示$\bigcap$，\iint表示$\iint$。
 8. 分数有两种产生方式。\frac作用于下边紧邻的两个组；如\frac ab产生$\frac ab$。可以使用{...}来产生更复杂的表达式；如\frac{a+1}{b+1}产生$\frac{a+1}{b+1}$。如果分子和分母很复杂，你可能更喜欢\over，它分割一个组内的两部分产生分数；如{a+1 \over b+1}产生${a+1 \over b+1}$。
 9. 字体。
  - 使用\mathbb或\Bbb产生"blackboard bold"：$\mathbb{CHNQRZ}$
  - 使用\mathface产生blodface：$\mathbf{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathbf{abcdefghijklmnopqrstuvwxyz}$
  - 使用\mathtt产生"typewriter"字体：$\mathtt{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathtt{abcdefghijklmnopqrstuvwxyz}$
  - 使用\mattrm产生roman字体：$\mathrm{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathrm{abcdefghijklmnopqrstuvwxyz}$
  - 使用\mathcal产生"calligraphic"字母：
$\mathcal{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathcal{abcdefghijklmnopqrstuvwxyz}$
  - 使用\mathsrc产生script字母：
$\mathscr{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathscr{abcdefghijklmnopqrstuvwxyz}$
  - 使用\mathfrak产生"Fraktur"(老式德国风格)字母：
$\mathfrak{ABCDEFGHIJKLMNOPQRSTUVWXYZ}$$\mathfrak{abcdefghijklmnopqrstuvwxyz}$
 10. 使用\sqrt产生根号，它会自动适应公式的大小。如\sqrt{x^3}产生$\sqrt{x^3}$；\sqrt[3]{\frac xy}产生$\sqrt[3]{\frac xy}$。对于复杂的表达式可以使用{...}^{1/2}。
 11. 一些如lim，sin，max，ln的特殊函数可以使用\lim，\sin等产生。如\sin x产生$\sin x$，而不是sin x产生$sin x$。\lim使用下标产生趋近符号：\lim_{x\to 0}产生$\lim_{x\to 0}$。
 12. 有大量的特殊符号和记号，可以参考[this shorter listing](http://pic.plover.com/MISC/symbols.pdf)或[this exhaustive listing](http://mirror.math.ku.edu/tex-archive/info/symbols/comprehensive/symbols-a4.pdf)。一些常用的列在下边：
  - \lt,\gt,\le,\ge,\neq分别表示$\lt$,$\gt$,$\le$,$\ge$,$\neq$。你可以使用\not来在大多数符号上画上斜线，但通常看起来很差劲，如\not\lt表示$\not\lt$。
  - \times,\div,\pm,\mp分别表示$\times$,$\div$,$\pm$,$\mp$。\cdot表示一个居中的点：$x\cdot y$。
  - \cup,\cap,\setminus,\subset,\subseteq,\subsetneq,\supset,\in,\notin,\emptyset,\varnothing分别表示$\cup\, \cap\, \setminus\, \subset\, \subseteq \,\subsetneq \,\supset\, \in\, \notin\, \emptyset\, \varnothing$。
  - {n+1 \choose 2k}或者\binom{n+1}{2k}表示${n+1 \choose 2k}$。
  - \to,\rightarrow,\leftarrow,\Rightarrow,\Leftarrow,\mapsto分别表示$\to\, \rightarrow\, \leftarrow\, \Rightarrow\, \Leftarrow\, \mapsto$。
  - \land,\lor,\lnot,\forall,\exists,\top,\bot,\vdash,\vDash分别表示$\land\, \lor\, \lnot\, \forall\, \exists\, \top\, \bot\, \vdash\, \vDash$。
  - \star\, \ast\, \oplus\, \circ\, \bullet分别表示$\star\, \ast\, \oplus\, \circ\, \bullet$。
  - \approx\, \sim \, \simeq\, \cong\, \equiv\, \prec分别表示$\approx\, \sim \, \simeq\, \cong\, \equiv\, \prec$。
  - \infty\, \aleph_0, \nabla\, \partial, \Im\, \Re分别表示$\infty\, \aleph_0, \nabla\, \partial, \Im\, \Re$。
  - 可以使用\pmod来产生同余式，如a\equiv b\pmod n产生$a\equiv b\pmod n$。
  - \ldots可以产生这里的点：$a_1, a_2, \ldots ,a_n$；\cdots产生这里的点：$a_1+a_2+\cdots+a_n$。
  - 一些希腊字母具有不同的形式：\epsilon\, \varepsilon分别表示$\epsilon\, \varepsilon$，\phi\, \varphi分别表示$\phi\, \varphi$，\ell表示$\ell$等。
 13. 在mathjax的公式中加入空格，不会改变公式中空格的数量。如a␣b和a␣␣␣␣b都产生$a b$。可以使用\,加入一个空格$a\,b$，使用\;加入一个宽空格$a\;b$，使用\quad和\qquad产生大量的空格$a\quad b,a\qquad b$。
可以使用\text{...}在公式中加入普通文本：$\{x\in s\mid x\text{ is extra large}\}$。还可以在\text{...}中嵌套\$...\$。
 14. 强调和区别标志。\hat用于单个符号：$\hat x$,\widehat用于一个公式：$\widehat{xy}$。如果将它弄的特别宽，看起来就十分丑。类似的，\bar产生$\bar x$，\overline产生$\overline{xyz}$，\vec产生$\vec x$，\overrightarrow产生$\overrightarrow{xy}$。可以使用\dot和\ddot产生点和双点，如\frac d{dx}x\dot x =  \dot x^2 +  x\ddot x产生$\frac d{dx}x\dot x =  \dot x^2 +  x\ddot x$。
 15. 使用\来转义mathjax中使用的特殊字符，如\$表示$\$$，\{表示$\{$，\_表示$\_$等等。
教程到此结束。