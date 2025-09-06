# 6. 无限域函数，自动机与正则表达式

> [!tip] 本章的学习目标
> * 在*长度无界*的输入上定义函数，这种函数无法用一个由输入和输出构成的有限大小的表格描述
> * （前者）与语言的成员资格判定任务的等价性
> * 确定性有穷自动机（可选）：一个无界计算模型的简单案例
> * （前者）与正则表达式的等价性

> [!QUOTE] 
> “算法以有限回答无穷”
> 
> *—Stephen Kleene*

布尔电路的模型（或者说，NAND-CIRC编程语言）有一个非常明显的短板: 一个布尔电路只能计算一个*有限的*函数$f$。事实上，由于每个门配有两个输入，大小为$s$的电路至多能计算长度为$2s$的输入。

因此这一模型无法捕捉到算法作为对潜在的无穷函数进行的*统一处理*这一直观概念。

比方说，标准的小学乘法算法是一种可以对所有长度的数进行乘法运算的*统一*算法。然而，无法将这种算法表达为单一的电路，而是需要对每种输入配备一个不同的电路（或者说，NAND-CIRC语言）。（见图6.1）

> **图6.1 小学乘法**：
> ![multiplicationschool](./images/chapter6/multiplicationschool.png)
> 你一旦知道如何计算多位数乘法。你就可以对所有$n$位数这么做，但如果你想用布尔电路或者NAND-CIRC程序来描述乘法，对所有长度为$n$的输入，你都需要一个不同的程序/电路

本章拓展了计算任务的定义，使其考虑配备*无界*定义域$\{0,1\}^*$的函数。
其重点在于定义计算**哪些**任务，将**如何**计算的绝大部分留给之后的章节。其中将会认识到*图灵机*与其他在无界输入上进行计算的计算模型。
然而，这一章将认识到一个简单且受限的计算模型的样例——确定性有穷自动机(DFAs)。

> [!tip] 对本章内容进行的非数学化的概括：
> 本章将会讨论以任意长度字符串作为输入的函数。
> 本章主要关注*布尔*函数这种特例，其输出为单一的位。
> 除此之外仍然有无数多个输入长度无界的函数。因此这一的函数不能被任何一个单一的布尔电路计算。
> 这个章节的第二部分将会讨论*有穷自动机*，这种计算模型可以计算一个输入长度无界的函数。
> 确定性有穷自动机不像Python或其他通用编程语言一样强大。但它可以作为这些更加通用的计算模型的一个引子。
> 本章将会展示一个美妙的结果——能被有穷自动机计算的函数与能被*正则表达式*计算的函数精确地一致。
> 然而，读者仍然可以自由跳过自动机的部分而直接转向[第七章](./chapter_7.md)中对于*图灵机*的讨论。

## 6.1 输入长度无界的函数

直到现在，我们考虑的计算任务都将某些长度为$n$的字符串映射为某个长度为$m$的字符串。

然而，一般情况下的计算任务都会涉及到长度无界的输入
例如，接下来的Python函数会计算一个函数$XOR:\{0,1\}^* \rightarrow \{0,1\}$, 其中 $XOR(x)$ 为 $1$ 当且仅当$x$中$1$的数量为奇数。

（换言之，对每个 $x\in \{0,1\}^*$，$XOR(x) = \sum_{i=0}^{|x|-1} x_i \mod 2$）
$XOR$虽然简单，却无法被一个布尔电路计算。相反，对每个$n$，都需要通过不同的电路计算$XOR_n$($XOR$函数在$\{0,1\}^n$的限制)(e.g. 见图6.2)。 

```python
def XOR(X):
    '''接受一个0与1的列表X
       当1的个数为奇数时输出1
       否则输出0'''
    result = 0
    for i in range(len(X)):
        result = (result + X[i]) % 2
    return result
```

> **图 6.2**：
> ![xor5circprog](./images/chapter6/xor5circprog.png)
> 计算$5$位异或的NAND电路与NAND-CIRC程序。值得注意的是$XOR_5$的电路仅仅只是重复了四次计算$2$位异或的电路。
这本书的前面部分研究了*有限*函数$f:\{0,1\}^n \rightarrow \{0,1\}^m$的计算。这样一种函数$f$总是能通过列举所有的输入$x\in \{0,1\}^n$所对应的$2^n$个函数值来表示。本章考虑像$XOR$这样输入长度无界的函数。
尽管能用有限多个符号来描述$XOR$（事实上在上面已经做过了），它却能接受无穷多种可能的输入，因此无法把它所有的函数值都写下来。
这对其他蕴含着其他重要计算任务的函数也是同理，包括加法，乘法，排序，在图上寻找路径，由点拟合曲线，等等。

为了和有限情况作区分，有时将函数$F:\{0,1\}^* \rightarrow \{0,1\}$（或$F:\{0,1\}^* \rightarrow \{0,1\}^*$）称为*无限的*。
然而，这不意味着$F$可以接收一个无限长的输入。
它仅仅表明$F$可以接收任意长的输入，因此无法简单地把在一个表上把不同输入下$F$的全部输出都写下来。

> **想法6.1**
> 函数$F:\{0,1\}^* \rightarrow \{0,1\}^*$指明了一个将输入$x\in \{0,1\}^*$映射到$F(x)$的计算任务. 

如前所述，将我们的注意力限制在以二进制串为输入和输出的函数上是不失一般性的、因为其他的对象，像数字，列表，矩阵，照片，视频，以及别的种种，都可以用二进制串编码。

如前所述，有必要区分*规范*和*实现*。例如，考虑以下函数。

$$
TWINP(x) = \begin{cases} 
        1 & \exists_{p \in \N} \text{使得} p,p+2 \text{为质数且} p>|x| \\
        0 & \text{否则}
       \end{cases}
$$

在数学上，这是一个良定义的函数。对每个$x$,$TWINP$都会有一个非$0$即$1$的函数值。然而，截至目前，尚未已知能计算该函数的Python程序。[孪生素数猜想](https://en.wikipedia.org/wiki/Twin_prime)主张对每个$n$都有一个$p>n$使得$p,p+2$均为素数。
如果该猜想成立，那么$T$（译者注:此处应指$TWINP$）是很容易计算的—— `def T(x): return 1`是奏效的
然而，自1849年起，数学家们对该猜想的证明均无功而返。
这说明，不论知不知道$TWINP$函数的*实现*，上面的定义提供的都是它的*规范*

### 6.1.1 改变输入和输出

许多有趣的函数都接受不止一个输入，例如函数：

$$
MULT(x,y) = x \cdot y
$$

接受一个二进制表示的整数对$x,y \in \N$，并输出积$x \cdot y$的二进制表示.
然而，因为能够将一对字符串表达为一个单一的字符串，我们将把像MULT这样的函数视为从$\{0,1\}^*$到$\{0,1\}^*$的映射。
一般不考虑像把一对整数精确地表达为串的方式这样的底层细节，因为近乎所有的选择对我们的目标而言都是等价的。

我们想计算的另一个函数是

$$
PALINDROME(x) = \begin{cases}
        1 & \forall_{i \in [|x|]} x_i = x_{|x|-i} \\
        0 & \text{否则}
        \end{cases}
$$

$PALINDROME$ 以一个单一的位作为输出。以一个单一的位为输出的函数成为*布尔函数*。
布尔函数是计算理论的中心，因此将在这本书中经常性地被讨论。
需要注意的是，即使布尔函数只有一个单一位用于输出，其输入可以是任意长度的。
因此它们仍然无法通过一个由函数值组成的有限表格描述，仍然是一个无限函数。

*“布尔化”函数*。有时从一个非布尔函数中构造一个布尔函数的变体是非常方便的。
例如，下列函数是$MULT$的一个布尔函数变体：

$$
BMULT(x,y,i) = \begin{cases}
                x\cdot y \text{的第i位} & i <|x \cdot y| \\
                0     & \text{否则}
               \end{cases}
$$

如果能够通过例如Python，C，JAVA等任何一门编程语言计算$BMULT$，也可以计算$MUL$，反之亦然。

> **问题 6.1** 一般函数的布尔化
> 说明对每个函数$F:\{0,1\}^* \rightarrow \{0,1\}^*$，都有一个布尔函数$BF:\{0,1\}^* \rightarrow \{0,1\}$使得一个能够计算$BF$的Python程序可以被转移为一个计算$F$的程序，反之亦然。

> **解 6.1.1**
> 对每个函数$F:\{0,1\}^* \rightarrow \{0,1\}^*$, 可以定义。
>
> $$
BF(x,i,b) = \begin{cases}
            F(x)_i & i<|F(x)|, b=0 \\
            1      & i<|F(x)|, b=1 \\
            0      & i \geq |F(x)|
            \end{cases}
$$
>
> 其输入满足 $x \in \{0,1\}^*, i \in \N, b\in \{0,1\}$，而输出为$F(x)$的第i位（如果$b=0$且$i<|F(x)|$）。
>
> 如果$b=1$，则$BF(x,i,b)$当且仅当$i<|F(x)|$时为$1$，通过这一点可以计算$F(x)$的长度。
> 从$F$出发计算$BF$是十分直接的。
> 另一方面，给定一个计算$BF$的Python函数$BF$，可以通过如下方法计算$F$。
>
> ```python
> def F(x):
>    res = []
>    i = 0
>    while BF(x,i,1):
>        res.append(BF(x,i,0))
>        i += 1
>    return res
> ```

### 6.1.2 形式语言

对每个布尔函数$F:\{0,1\}^* \rightarrow \{0,1\}$，可以定义集合 $L_F = \{ x | F(x) = 1 \}$。这样的集合被称为*语言*。
这个名字源于*形式语言理论*，像Noam Chomsky这样的语言学家致力于该理论。。
一个*形式语言*是$L \subseteq \{0,1\}^*$（更一般地说$L \subseteq \Sigma^*$，其中$\Sigma$是一个有限的字母表）（译者注：$\Sigma$中的元素称为*字母*，原著中提到其元素时使用的术语是字母表符号`alphabet symbol`，翻译时为了简洁使用字母这一个更加简单但是可能会引起混淆的术语）。
一个语言$L$上的*成员资格问题*或*判定问题*，是断定对于给定的$x\in \{0,1\}^*$，是否有$x\in L$。
如果能够计算函数$F$，也就能够判定语言$L_F$的成员资格，反之亦然。
因此，许多像[Sipser，1997](https://scholar.google.com/scholar?hl=en&q=Sipser+Introduction+to+the+theory+of+computation)这样的教材都将计算一个布尔函数的任务称为“判定一个语言”
本书主要用*函数*的记号来描述计算任务，这种方法更容易推广到不止一位输出的计算任务。
然而，因为语言的术语在文献中更加流行，有时也会提到它们。

### 6.1.3 函数的限制

如果$F:\{0,1\}^* \rightarrow \{0,1\}$是一个布尔函数而$n\in \N$，则$F$在输入长度为$n$上的限制记作$F_n$，是一个有限函数$f:\{0,1\}^n \rightarrow \{0,1\}$使得对每个$x\in \{0,1\}^n$均有$f(x) = F(x)$。
这就是说$F_n$是定义在$\{0,1\}^n$上的有限函数，但在这些输入上与$F$保持一致。
因为$F_n$是一个有限函数，所以它可以被一个布尔电路计算。以下定理表明了这一点。

> **定理6.1** 每个无限函数的电路族
令$F:\{0,1\}^* \rightarrow \{0,1\}$.。则有一个电路族$\{ C_n \}_{n\in \{1,2,\ldots\}}$使得对每个$n>0$，$C_n$能够计算$F$在输入长度为$n$上的限制$F_n$

> 证明：
这是布尔电路通用性的一个立即推论。
事实上，因为$F$把$\{0,1\}^n$映射到$\{0,1\}$，定理4.15表明一定有一个布尔函数$C_n$来计算它。
事实上，这个电路的大小为至多$c \cdot 2^n / n$个门，其中$c \leq 10$为常数。$\square$

特别地，定理6.1表明甚至对于前面描述过的$TWINP$函数，这样的电路族也存在，即使尚未已知的程序可以对其进行计算。
这实际上并不令人惊讶：对每个特定的$n \in N$，$TWINP_n$要么是常0函数要么是常1函数，其中任何一者都可以用一个简单的布尔电路计算。
因为计算$TWINP$的电路族一定存在，用Python或其他任何编程语言计算$TWINP$的难度源于这样一个事实——我们不知道对每个特定的$n$，电路族中的$C_n$应该是什么。

## 6.2 确定性有穷自动机（可选）

我们目前所有的计算模型——布尔电路和无分支程序——都只对*有限*函数有效。

在[第七章](./chapter_7.md)中，将会介绍*图灵机*，这是输入长度无界函数的中心计算模型。
然而，本节将会介绍一个更加基本的模型——*确定性有穷自动机*(DFA)

自动机可以视作通往图灵机的一个优秀的垫脚石，尽管它们在这本书的后面部分并不会大量地被用到，所以读者可以自由跳过到[第七章](./chapter_7.md).

DFAs在能力上与*正则表达式*是等价的：正则表达式是识别模式的一个强力工具，在实践中广泛应用。
本书对自动机的处理是相对简略的。有大量的资源可以帮助你更加熟悉DFAs。
详细地说，第一章中Sisper的著作[Sipser, 1997](https://scholar.google.com/scholar?hl=en&q=Sipser+Introduction+to+the+theory+of+computation)包含对这个内容的绝佳的说明。
这里有许多的在线自动机模拟器网站，也有将自动机和正则表达式互化的翻译器。
(例如[此处](http://ivanzuzak.info/noam/webapps/fsm2regex/)和[此处](https://cyberzhg.github.io/toolbox/nfa2dfa)).

从高视角上看，一个*算法*是通过以下步骤的组合从输入计算输出的方法:

1. 从输入读入一位
2. 更新*状态*(工作记忆)
3. 停止并产生一个输出

例如，回忆以下计算$XOR$函数的Python程序

```python
def XOR(X):
    '''接受一个0与1的列表X
       当1的个数为奇数时输出1
       否则输出0'''
    result = 0
    for i in range(len(X)):
        result = (result + X[i]) % 2
    return result
```
每一步中，程序读入一个位`X[i]`并且根据它更新自己的`result`状态（在`X[i]`为1时翻转`result`，否则保持原样）。当它遍历完输入后，程序输出`result`。
在计算机科学中，这样一个程序称为*单遍常数内存算法*，因为它只遍历一次输入，而它的工作记忆是有限的。
（事实上，在这个案例中，`result`非$0$即$1$）
这样一个算法称为*确定性有穷自动机*或*DFA*（DFAs的另一个名字是*有限自动机*）。
我们可以把这样一种算法视作一个拥有$C$个状态的“机器”，其中$C$为常数。
这样一种机器从某个初始状态开始，然后从输入$x \in \{0.1\}^*$中一次读取一个位
只要这个机器读入了一个位$\sigma \in \{0,1\}$，它就会根据$\sigma$和先前的状态转换到一个新的状态。
机器的输出决定于最终状态。每个单遍常数内存算法都和这样一个机器一致。
如果这个算法使用了$c$位内存，那么其内存中的内容就能用一个长度为$c$的串表达。因此这样一个算法的任意一个执行点都在至多$2^c$个状态之中。

我们可以通过一个$C \cdot 2$条规则的列表来指明一个拥有$C$个状态的DFA（译者注：更准确地说，是$C \cdot |\Sigma|$条，但之后考虑的均为$\Sigma=\{0,1\}$，因此$|\Sigma|=2$）。
每条规则都有这样的形式：“如果DFA位于状态$v$而读入的输入位为$sigma$则新状态为$v'$”。
在计算的最后，会有一个具有形式“如果最终状态为下列中的一者 ... 则输出$1$，否则输出$0$”的规则。
举例而言，上述的Python程序可以用一个两个状态的自动机来计算$XOR$：

* 初始化为状态$0$。
* 对每个状态$s \in \{0,1\}$和读取的输入位$\sigma$，如果$\sigma=1$则将状态转移为$1-s$，否则停留在状态$s$。
* 最终当且仅当$s=1$时输出$1$。

我们也可以用一个带标号的$C$个顶点的*图*来描述$C$个状态的DFA。随每个状态$s$和位$\sigma$，我们添加一条带有标号$\sigma$的从$s$到$s'$的有向边，使得若DFA位于状态$s$且读入$\sigma$，则DFA转移到状态$s'$。（如果状态不变，则这个边是一个指向原状态的圈；相似地，如果$s$在$\sigma=0$和$\sigma=1$两种情况下都转移为状态$s'$，则图上会有两条平行的边）同时也会标明在最后使自动机输出$1$的状态集$\mathcal{S}$。这个集合称为*接受状态*集。

图6.3给出了XOR自动机的图形表示

> **图 6.3**：
> ![xorautomaton](./images/chapter6/xorautomaton.png)
> 一个计算XOR函数的有穷自动机。其有两个状态$0$和$1$, 当它读入$\sigma$时，它从$v$转移到$v \oplus \sigma$.

形式化地讲，一个DFA由 **(1)** $C \cdot 2$条规则构成的表格，该表格用*转移函数*$T$表示。$T$将状态$s \in [C]$和位$\sigma \in \{0,1\}$映射到状态$s' \in [C]$。DFA将会在输入$\sigma$下从状态$s$转移到$s'$；和 **(2)** 接受状态集$\mathcal{S}$

> **定义6.2** 确定性有穷自动机
> 一个在$\{0,1\}$上定义的$C$个状态的确定性有穷自动机是一个对$(T, \mathcal{S})$。其中$T:[C]\times \{0,1\} \rightarrow [C]$ 而 $\mathcal{S} \subseteq [C]$。
> 有限函数$T$称为DFA的*转移函数*。集合$\mathcal{S}$称为*接受状态*集。
>
> 令$F:\{0,1\}^* \rightarrow \{0,1\}$为无限域$\{0,1\}^*$上的布尔函数。
> 对于任意$n \in \N$和$x\in \{0,1\}^n$，定义$s_0=0$且对任意$i \in [n]$，$s_{i+1} = T(s_i,x_i)$，若有
$$
s_n \in \mathcal{S}   \Leftrightarrow F(x)=1
$$
则称$(T,\mathcal{S})$计算函数$F:\{0,1\}^* \rightarrow \{0,1\}$。

> **休息一下，仔细思考**
> 确保你没有混淆自动机的*转移函数*(定义6.2中的$T$)与其所*计算*的函数(定义6.2中的$F$)。前者是一个有限函数，指明了自动机所遵循的规则的表格；后者是一个无限函数。

> [!remark] 其他教材中的定义
> 确定性有穷自动机可以通过几种等价的方法定义。
> 特别地，Sisper在[Sipser，1997](https://scholar.google.com/scholar?hl=en&q=Sipser+Introduction+to+the+theory+of+computation)将DFA定义为五元组$(Q,\Sigma,\delta,q_0,F)$，其中$Q$为状态集，$\Sigma$为字母表，$\delta$为转移函数，$q_0$是初始状态，$F$为接受状态集。
该书中状态集总是如下形式$Q=\{0,\ldots,C-1 \}$而初状态总是$q_0 = 0$，但这对这些模型的计算能力没有影响。
> 因此，我们将注意力局限在字母表$\Sigma$与$\{0,1\}$相等的情况。

> **练习6.2**  “识别$(010)^*的DFA$”
> 证明计算下列函数$F$的DFA存在：

$$
F(x) = \begin{cases} 
         1 & 3 \text{ 整除 } |x| \text{ 且 } \forall_{i\in [|x|/3]} x_{3i} x_{3i+1} x_{3i+2} = 010  \\
         0 & \text{否则}
         \end{cases}
$$

> **解6.2**
> 当要求构造一个DFA时，首先通过更加一般的形式化的方法，构造一个单遍常数内存算法通常是有效的。（例如使用伪代码或者一个python程序）。一旦得到了这样一个算法，就可以机械式地将其翻译为一个DFA。
以下是计算$F$的一个简单Python程序：

```python
def F(X):
    '''当且仅当X是零个或多个[0,1,0]的拼接时返回1'''
    if len(X) % 3 != 0:
        return False
    ultimate = 0
    penultimate = 1
    antepenultimate = 0
    for idx, b in enumerate(X):
        antepenultimate = penultimate
        penultimate = ultimate
        ultimate = b
        if idx % 3 == 2 and ((antepenultimate, penultimate, ultimate) != (0,1,0)):
            return False
    return True
```

既然我们维护了三个布尔变量，工作记忆就可以是$2^3 = 8$种配置中的一个，因此上述程序可以直接翻译为一个$8$状态DFA。
尽管这对解决问题没有必要，通过检查结果DFA，会发现可以通过合并一些状态得到一个$4$状态自动机，该自动机在图6.4中描述。图6.5中描述了在一个特定输入上这个DFA的运行。


> **图 6.4**：
> ![DFA010a](./images/chapter6/DFA010a.png)
> 一个仅在输入$x\in \{0,1\}^*$为零个或多个$010$的拼接时输出$1$的DFA。
> 状态$0$既是初始状态又是唯一的接受状态。
> 表格表示了转移函数$T$。它将当前状态和读到的符号映射到一个新状态。

### 对自动机的剖析（有限vs无界）

既然我们已在考虑输入长度无界的计算任务，将算法中拥有*固定长度*的组件和随输入增长的组件区分开是非常关键的。对于DFAs而言，是下列部分：

**固定大小组件：** 给定一个DFA $A$，下列量是固定的，与输入大小无关：

* $A$中*状态*数$C$。
* *转移函数*$T$（有$2C$种输入，因此可以用一个$2C$行的表格描述，每一项都是$[C]$中的一个数字）。
* 接收状态集$\mathcal{S} \subseteq [C]$。该集合可以用一个$\{0,1\}^C$中的串描述，以指明哪些状态位于$\mathcal{S}$中而哪些没有。

以上这些意味着可以通过有限多个符号完全地描述一个自动机。这是我们要求的任何一种“算法”的概念都拥有的一个共同性质：我们应当能够写下如何从输入生成输出的完整规范。

**无界大小组件：** 一下关于DFA的量不以任何常数作为上界。我们强调对于任何给定的输入，它们仍然是有限的。 

* 提供给DFA的输入$x\in \{0,1\}^*$的大小。输入长度总是有限的，但是不能预先设定上界。
* DFA执行的步数可以随输入长度而增长。事实上，DFA进行单次便利，因此对于一个输入$x\in \{0,1\}^*$，它精确地执行$|x|$步。

> **图 6.5**：
> ![DFA010execution](./images/chapter6/DFA010execution.png)
> 图6.4中DFA的执行过程。状态数和转移函数的大小是有界的，但是输入可以是任意长的。
如果DFA位于状态$s$且读取值$\sigma$，则其转移到状态$T(s,\sigma)$。在执行的最后，当且仅当最终状态位于$\mathcal{S}$时DFA接受该输入。

### DFA可计算函数

如果有一个$DFA$可以计算$F$，就称一个函数$F:\{0,1\}^* \rightarrow \{0,1\}$是*DFA可计算的*。
在[第四章](./chapter_4.md)中，我们发现每个有限函数都可以被某些布尔电路计算，因此，在此刻，你可能会希望每个函数都可以被*某些*DFA计算。
然而，有很多并*不是*这种情况。我们马上就会发现一些简单的无法被DFA计算的无限函数。但对于初学者，我们先证明这样的函数是存在的。

> **定理6.4 DFA可计算的函数是可数的**
让$DFACOMP$为全体使得存在一个DFA计算$F$的布尔函数$F:\{0,1\}^* \rightarrow \{0,1\}$的集合。
则$DFACOMP$可数。


> 证明思想：
每个DFA都能用一个有限长度的串来描述，从而产生一个从$\{0,1\}^*$到$DFACOMP$的满射：更准确地说，这个函数将一个描述自动机$A$的串对应到$A$计算的函数。

> 证明:
每个DFA都能用一个表示转移函数$T$和接收状态集的串描述，而每个DFA $A$都计算*某些*函数$F:\{0,1\}^* \rightarrow \{0,1\}$。
因此可以定义如下函数$StDC:\{0,1\}^* \rightarrow DFACOMP$:
$$
StDC(a) = \begin{cases}
           F & a \text{ 表示自动机 } A \text{ 且 } F \text{ 是 } A \text{ 计算的函数 } \\
           ONE & \text{否则}
           \end{cases}
$$
其中$ONE:\{0,1\}^* \rightarrow \{0,1\}$是对于所有输入均输出$1$的常函数（也是$DFACOMP$中的一个函数）。
因此根据定义，每个$DFACOMP$中的函数$F$都可以被*某些*自动机计算，而$StDC$是从$\{0,1\}^*$到$DFACOMP$的满射，这就意味着$DFACOMP$可数。(见[节 2.4.2](./chapter_2.md))$\square$

因为*所有*布尔函数的集合是不可数的，所以有如下推论：

> **定理6.5** DFA不可计算函数的存在性
> 存在一个布尔函数$F:\{0,1\}^* \rightarrow \{0,1\}$不能被*任何的*DFA计算。

> 证明:
> 如果每个布尔函数$F$都可以被一些DFA计算，那么$DFACOMP$就与集合$ALL$（所有布尔函数的集合）相等。但根据定理2.12，后者不可数，又与定理6.4相矛盾。$\square$

## 6.3 正则表达式

*搜索*一段文本是计算中的一个常见任务。从本质上说，*搜索问题*非常简单。
我们有一个串集$X = \{ x_0, \ldots, x_k \}$（例如硬盘上的文件，或数据库中的学生记录），而用户想要找到一个所有被某些模式*匹配*的$x \in X$构成的子集。
（例如，所有名称以串`.txt`结尾的文件）
在最一般的情况下，我们允许用户通过指定一个（可计算的）*函数*$F:\{0,1\}^* \rightarrow \{0,1\}$来指明模式，其中$F(x)=1$与$x$的模式匹配相一致。
这就是说，用户提供一个用像*Python*这样的编程语言编写的*程序*$P$，而系统返回所有使$P(x)=1$的$x \in X$。
举例而言，我们可以搜索所有包含串`important document`的文本文件，或是（让$P$与一个基于神经网络的分类器相一致）所有包含猫的图片。
然而，我们不希望系统会因为尝试求程序$P$的值而陷入死循环中！
因此，典型的搜索文件和数据库的系统*不*允许用户用功能齐全的编程语言来指定模式。
相反，这样的系统使用*受限计算模型*。这种模型一方面*足够丰富*，可以捕捉许多实践中需要的查询（例如，所有以`.txt`结尾的文件名，或者所有形如`(617)xxx-xxxx`的电话号码），但另一方面受到的*限制*又足以使大型文件中的查询变得非常高效，并避免陷入死循环中。

这种计算模型中最流行的一种是[正则表达式](https://goo.gl/2vTAFU)。如果你使用过一个高级的文本编辑器，一个命令行终端，或者进行过任何种类的对文本文件的大批量操作，那么你很有可能对正则表达式有所耳闻。

在字母表$\Sigma$上定义的*正则表达式*由$\Sigma$上的元素通过连接操作，$|$操作（与*或*一致）和$*$操作（与重复零到多次一致）组合而成。
举例而言，接下来在字母表$\{0,1\}$上定义的正则表达式与所有使每个数位重复至少两次的串$x\in \{0,1\}^*$构成的集合一致：
$$
(00(0^*)|11(1^*))^*
$$

下列定义在字母表$\{ a,\ldots,z,0,\ldots,9 \}$上的正则表达式与所有这样的串形成的集合一致——该串由至少一个$a$-$d$的字母形成的序列连接上至少一个数位形成的序列组成（无前导零）。

$$
(a|b|c|d)(a|b|c|d)^*(1|2|3|4|5|6|7|8|9)(0|1|2|3|4|5|6|7|8|9)^* (6.1)
$$

形式化地说，正则表达式由以下递归定义所定义：


> **定义6.6** 正则表达式
> 字母表$\Sigma$上定义的*正则表达式*$e$是$\Sigma \cup \{ (,),|,*,\emptyset, "" \}$上的一个串，并具有下列形式之一
>
> 1. $e = \sigma$，其中$\sigma \in \Sigma$
>
> 2. $e = (e' | e'')$，其中$e', e''$为正则表达式
>
> 3. $e = (e')(e'')$其中$e',e''$为正则表达式（当不会混淆时，通常省略括号并写为$e' \; e''$。）
>
> 4. $e = (e')^*$其中$e'$为正则表达式
> 最终还有两个“边界条件”：$e = \emptyset$ and $e = ""$。这些正则表达式分别与不接受任何串和只接受空串一致。

在能从上下文中推断出来时，我们也会忽略括号。我们也使用或运算和连接运算左结合的惯例，并且给$*$运算最高的优先级，然后是连接，最后是或。
因此，举例来说，我们写的是$00^*|11$而不是$((0)(0^*))|((1)(1))$。

每个正则表达式$e$都与一个函数$\Phi_{e}:\Sigma^* \rightarrow \{0,1\}$一致，其中若$x$*匹配*正则表达式，则$\Phi_{e}(x)=1$。
举例说，若$e = (00|11)^*$ 则 $\Phi_e(110011)=1$ 而 $\Phi_e(101)=0$（你知道为什么吗）

> **休息一下，仔细思考**
> $\Phi_{e}$的形式化定义是那种写比掌握麻烦的类型。因此第一时间自己搞清楚其定义，再检查其是否与下列的定义相符可能会更加简单。

> **定义6.7** 匹配正则表达式
> 令$e$为字母表$\Sigma$上的正则表达式
> 函数$\Phi_{e}:\Sigma^* \rightarrow \{0,1\}$ 定义如下:
>
> 1. 若$e = \sigma$，则当且仅当$x=\sigma$时$\Phi_{e}(x)=1$。
>
> 2. 若$e = (e' | e'')$，则$\Phi_{e}(x) = \Phi_{e'}(x) \vee \Phi_{e''}(x)$，其中$\vee$为或运算符。
>
> 3. 若$e = (e')(e'')$，则当且仅当存在$x',x'' \in \Sigma^*$使得$x$为$x'$和$x''$的连接，且$\Phi_{e'}(x')=\Phi_{e''}(x'')=1$时，$\Phi_{e}(x) = 1$。 
>
> 4. 若$e= (e')*$，则当且仅当存在$k\in \N$和$x_0,\ldots,x_{k-1} \in \Sigma^*$使得$x$为$x_0 \cdots x_{k-1}$的连接，且对每个$i\in [k]$，均有$\Phi_{e'}(x_i)=1$时$\Phi_{e}(x)=1$。
>
> 5. 最终, 对边界条件 $\Phi_{\emptyset}$是常$0$函数, 而$\Phi_{""}$ 只在输入空串时输出$1$。
>
> 对一个串$x \in \Sigma^*$，若$\Sigma$上的正则表达式$e$使$\Phi_{e}(x)=1$，就说$e$*匹配*$x$。

> **休息一下，仔细思考**
> 上述的定义本身并不是什么难事，但很麻烦。所以你应该在此处停下并再看一次上述定义，直到你理解为什么该定义与我们对正则表达式的直观概念是相一致的。这不仅对理解正则表达式本身（在许多应用中经常使用）很重要，对更好地理解一般的递归定义也一样。

若一个布尔函数在输出$1$时，所有的输入串都能够被某些正则表达式匹配，就说这个布尔函数是“正则的”。

> **定义6.8** 正则函数/语言
> 令$\Sigma$为一个有限集，而$F:\Sigma^*\rightarrow \{0,1\}$为一个布尔函数。若存在某个正则表达式$e$，$F=\Phi_{e}$，就称$F$是*正则*的。
类似的，对每个形式语言$L \subseteq \Sigma^*$，称$L$是正则的当且仅当存在某个正则表达式$e$使得$x\in L$当且仅当$e$匹配$x$

> **样例6.9** 一个正则函数
> 令$\Sigma=\{ a,b,c,d,0,1,2,3,4,5,6,7,8,9 \}$而$F:\Sigma^* \rightarrow \{0,1\}$使得$F(x)$当且仅当$x$是一个或多个$a$-$d$组成的序列接上一个或多个数位组成的序列（无前导零）
> 则$F$就是一个正则函数，因为$F=\Phi_e$，其中
$$
e = (a|b|c|d)(a|b|c|d)^*(1|2|3|4|5|6|7|8|9)(0|1|2|3|4|5|6|7|8|9)^*
$$
>
> 即式(6.1)
>
> 举例而言，如果要验证$\Phi_e(abc12078)=1$，注意到$(a|b|c|d)$匹配$a$，$(a|b|c|d)^*$匹配$bc$，$(1|2|3|4|5|6|7|8|9)$匹配$1$， $(0|1|2|3|4|5|6|7|8|9)^*$匹配$2078$。其中这些式子又可以被归结为一些更简单的表达式。例如$(a|b|c|d)^*$匹配$bc$，因为$b$和$c$被表达式$a|b|c|d$所匹配。

正则表达式可以在任意有限字母表$\Sigma$上定义。但是和之前一样，我们主要关注*二进制情况*，其中$\Sigma=\{0,1\}$。绝大部分（如果不是所有的话）关于正则表达式的理论和实践的真知灼见都可以从研究二进制情况得到。

### 6.3.1 匹配正则表达式的算法

除非能计算以下问题，否则正则表达式在搜索方面并不会很有用：给定一个正则表达式$e$，串$x$是否被$e$匹配。幸运的是，这样一个算法存在。
准确地说，存在一个算法（你可以想成“Python程序”，尽管稍后就会用*图灵机*来形式化算法的概念），该算法输入一个正则表达式$e$和串$x \in \{0,1\}^*$，当且仅当$e$匹配$x$时输出$1$（即，输出$\Phi_e(x)$）

实际上，定义6.7已经指明了一个*计算*$\Phi_{e}$的递归算法。准确地说，操作——连接，或，星号（译者注：准确的说法是闭包）——可以被视作这样一个过程：对测试某个表达式$e$是否匹配$x$的任务，将其归约到测试$e$的某个子表达式是否匹配$x$的某个子串。因为这些子表达式总是比原式短，所以这个判定$e$是否匹配$x$的递归算法最终会在最基础的表达式上停止：与空串或者当个符号一致。

**算法6.10** 正则表达式匹配
$$
\begin{align}
&\input\text{在}\Sigma^*\text{上定义的正则表达式}e\text{,}x\in \Sigma^* \\
&\output \Phi_e(x) \\
&\proc{Match}(e,x) \\
&\quad\if{e = \emptyset} \return{0} \textbf{;} \\
&\quad\if{x = ""} \return{\mathsf{MatchEmpty}(e)} \textbf{;} \\
&\quad\if{e \in \Sigma} \return{1} \text { iff } x = e \textbf{;} \\
&\quad\if{e = (e' | e'')} \return{\{\textsf{Match}(e',x) \text{或} \textsf{Match}(e'', x)\}} \textbf{;} \\
&\quad\if{e = (e')(e'')} \\
&\qquad\for{i \in [|x|]} \\
&\qqquad\if {\textsf{Match}(e',x_0 \cdots x_{i-1}) \text{且} \textsf{Match}(e'',x_i \cdots x_{|x|-1})} \return{1} \textbf{;}\\
&\qquad \endfor\\
&\quad \endif\\
&\quad\if{e = (e')^*}\\
&\qquad{e'=""} \return{\textsf{Match}("",x)} \textbf{;}\\
&\qquad \text{\#} ("") \text{与} "" \text{相同} \\
&\qquad\for{i \in [|x|]} \\
&\qqquad\text{\#}x_0\cdots x_{i-1}\text{比}x\text{短}\\
&\qqquad\if{\textsf{Match}(e,x_0 \cdots x_{i-1}) \text{且} \textsf{Match}(e',x_i \cdots x_{|x|-1})}\return{1} \textbf{;}\\
&\qquad\endfor \\
&\quad\endif \\
&\quad\textbf{return }0 \\
&\endproc
\end{align}
$$

以上代码假定已经编写了一个过程$\textsf{MatchEmpty}$，其当且仅当$e$匹配空串$""$时输出$1$。

一个关键的观察结果为，在对正则表达式的递归定义中，无论$e$是由一个还是两个表达式$e',e''$组成的，这两个正则表达式都比$e$ *小*
最终（当其长度为$1$）时，它们一定和单个字母的非递归情形一致。
相应地，算法6.10中的递归调用总是和一个更短的表达式或者（在表达式具有形式$(e')^*$的情况下）一个更短的输入串相一致。
因此，当输入具有形式$(e,x)$时，通过在$\min \{ |e|, |x| \}$上做递归，可以证明算法6.10的正确性。
归纳奠基是$x=""$或$e$为单独的一个字母，$""$或$\emptyset$。
在表达式具有形式$e=(e'|e'')$或$e=(e')(e'')$时，用更短的表达式$e',e''$做递归调用
在表达式具有形式$e=(e')^*$时，在一个更短的字符串$x$与同样的表达式，或更短的表达式$e'$与一个字符串$x'$上做递归调用，其中的$x'$长度小于等于$x$。

（译者注：事实上，以上过程仅仅证明了这个算法是会结束的，但是并没有证明正确性。但上面的过程确实给出了证明其正确性的骨架，因此剩下的工作繁而不难）

> **练习6.3** 匹配空串
> 给出一个匹配空串的算法。该算法输入为正则表达式$e$，且满足当且仅当$\Phi_e("")=1$时输出$1$

> **解6.3**
> 可以通过以下观察结果给出这样一个递归算法
> 1. 具有形式 $""$或$(e')^*$的表达式总是匹配空串
> 2. 具有形式 $\sigma$，其中$\sigma\in\Sigma$是一个字母，不匹配空串
> 3. 正则表达式$\emptyset$不匹配空串
> 4. 具有形式$e'|e''$的表达式当且仅当$e'$或$e''$匹配空串时才匹配
> 5. 具有形式$(e')(e'')$的表达式当且仅当$e'$和$e''$都匹配空串时才匹配
> 
> 根据以上的观察结果，可以给出下列算法来判断$e$是否匹配空串
> 
> **算法6.11** 匹配空串
$$
\begin{align}
&\input\text{在}\Sigma^*\text{上定义的正则表达式}e\text{,}x\in \Sigma^*\text{（译者注：该算法并未要求输入串}x\in\Sigma^*\text{。此处应为作者笔误）} \\
&\output \text{当且仅当}e\text{匹配空串时输出}1 \\
&\proc{MatchEmpty}(e) \\
&\quad\if\{e=""\}\return{1} \textbf{;}\\
&\quad\if\{e=\emptyset\text{或}e\in\Sigma\}\return{0} \textbf{;}\\
&\quad\if\{e=(e'|e'')\}\return{\textsf{MatchEmpty}(e')\text{或}\textsf{MatchEmpty}(e'')} \textbf{;}\\
&\quad\if\{e=(e')(e'')\}\return{\textsf{MatchEmpty}(e')\text{ and }\textsf{MatchEmpty}(e'')} \textbf{;}\\
&\quad\if\{e=(e')^*\}\return{1} \textbf{;}\\
&\endproc
\end{align}
$$

## 6.4 高效匹配正则表达式（可选）

算法6.10并不高效
举例而言，给定一个包含连接或“*”操作的表达式和一个长度为$n$的串，它需要$n$次递归调用。因此，在最劣情况下，算法6.10花费的时间是输入串$x$长度的*指数*级别。
幸运的是，有快得多的算法可以在*线性*时间（即$O(n)$）内匹配正则表达式。
鉴于还没提到时间和空间复杂度的话题，我们将在不给出计算模型的情况下，使用高级术语描述这个算法，并使用口语化的$O(n)$运行时间的概念，就像在编程入门课程和白板编程面试中做的那样。
我们将会在[第13章](./chapter_13.md)中介绍时间复杂度的形式化定义

> *定理6.12* 在线性时间内匹配正则表达式
> 给定一个正则表达式$e$，则存在$O(n)$时间的算法计算$\Phi_{e}$

定理6.12中$O(n)$术语所隐含的常数取决于表达式$e$
因此，另一个描述定理6.12的方法是对于每个表达式$e$，都会有一个常数$c$和一个算法$A$使得在$n$位输入上计算$\Phi_e$最多需要$c\cdot n$步
因为在实践中，通常希望对一个短的正则表达式$e$和大的文档$x$计算$\Phi_e(x)$，所以这是有意义的。
定理6.12告诉我们，可以在运行时间随文档大小线性增大的情况下计算$\Phi_e(x)$，即使运行时间可能更依赖于正则表达式的大小

我们通过给出一个高效的递归算法来证明定理6.12。该算法将判定$e$是否匹配串$x \in \{0,1\}$的任务归约到判定相关表达式$e'$是否匹配$x_0,\ldots,x_{n-2}$。
该算法使得表达式的运行时间拥有形式$T(n) = T(n-1) + O(1)$,解得$T(n)=O(n)$。

**正则表达式的限制**：定理6.12背后的算法，其中心定义是正则表达式的*限制*的概念
其思想为：对每个正则表达式$e$和字母$\sigma$，有可能定义一个正则表达式$e[\sigma]$使得$e[\sigma]$匹配$x$当且仅当$e$匹配匹配串$x\sigma$。
例如，如果$e$是正则表达式$(01)^*(01)$（即$01$出现一次或多次），那么$e[1]$与$(01)^*0$等价而$e[0]$为$\emptyset$。（你能发现是为什么吗）

算法6.13计算给定正则表达式$e$和字母$\sigma$的限制$e[\sigma]$。
该算法总会结束，因为其递归调用时传递的表达式总比输入的表达式小。
其正确性可以通过对正则表达式$e$的长度进行归纳证明，归纳奠基是$e$为$""$，$\emptyset$，或一个单独的字母$\tau$时。

**算法6.13** 限制正则表达式
$$
\begin{align}
&\input \text{在}\Sigma\text{上定义的正则表达式}e\text{，符号}\sigma\in\Sigma\\
&\output \text{正则表达式}e'=e[\sigma]\text{，使得}\Phi_{e'}(x)=\Phi_e(x \sigma)\text{对每个}x\in\Sigma^*\text{成立}\\
&\proc{Restrict}(e,\sigma)\\
&\quad\if{e="" \text{或} e=\emptyset} \return{\emptyset} \textbf{;}\\
&\quad\if{e=\tau \text{其中} \tau \in \Sigma}\return{""}\text{若}\tau=\sigma\text{否则}\return{\emptyset}\textbf{;}\\
&\quad\if{e=(e'|e'')}\return{(\mathsf{Restrict}(e',\sigma) | \mathsf{Restrict}(e'',\sigma))}\textbf{;}\\
&\quad\if{e=(e')^*}\return{(e')^* (\mathsf{Restrict}(e',\sigma))}\textbf{;}\\
&\quad\if{e=(e')(e'')\text{且}\Phi_{e''}("")=0} \return{(e')(\mathsf{Restrict}(e'',\sigma))}\textbf{;}\\
&\quad\if{e=(e')(e'')\text{且}\Phi_{e''}("")=1} \return{(e' \mathsf{Restrict}(e'',\sigma)) \; | \; \mathsf{Restrict}(e',\sigma)}\textbf{;}\\
&\endproc
\end{align}
$$

通过限制的概念，可以定义如下匹配正则表达式的递归算法

**算法6.14** 在线性时间内匹配正则表达式
$$
\begin{align}
&\input\text{在}\Sigma^*\text{上定义的正则表达式}e\text{，}x\in\Sigma^n\text{其中}n\in\N\\
&\output\Phi_e(x)\\
&\proc{FMatch}(e,x)\\
&\quad\if{x=""}\return{\mathsf{MatchEmpty}(e)}\mathbf{;}\\
&\quad\text{令}e'\leftarrow \mathsf{Restrict}(e,x_{n-1})\\
&\quad\return{\mathsf{FMatch}(e',x_0 \cdots x_{n-2})}\\
&\endproc
\end{align}
$$

根据限制的定义，对于每个$\sigma \in \Sigma$和$x'\in \Sigma^*$，表达式$e$匹配$x'\sigma$当且仅当$e[\sigma]$匹配$x'$。
因此对每个$e$和$x\in \Sigma^n$，$\Phi_{e[x_{n-1}]}(x_0\cdots x_{n-2}) = \Phi_e(x)$和算法6.14确实给出了正确的结果。
剩下的唯一任务就是分析其*运行时间*。
需要注意的是，算法6.14在归纳奠基$x=""$时使用练习6.3中的$\text{\textsf{MatchEmpty}}$过程。
然而，因为这个过程的运行时间只依赖于$e$，与原输入的长度无关，所以没有问题。

简单起见，我们将注意力限制在字母表$\Sigma$与$\{0,1\}$相等的情况。
定义$C(\ell)$为，给定最大符号数$\ell$，输入定义在$\{0,1\}$上的符号数不超过最大符号数的正则表达式，算法6.13所能进行的最大操作次数。
可以发现$C(\ell)$的值是关于$\ell$的多项式。然而这对我们的定理并不重要，因为我们只关心计算$\Phi_e(x)$时运行时间对$x$长度的依赖而不关心其对$e$长度的依赖。

算法6.14是输入表达式$e$和串$x\in \{0,1\}^n$的递归算法。其计算过程为在最多运行$C(|e|)$后，以某些表达式$e'$和长度为$n-1$的串$x'$为输入调用自身。
它将在$n$步运行后结束，此时它到达一个长度为$0$的串。
因此，对长度为$n$的输入，用算法6.13计算$Phi_e$的运行时间$T(e,n)$满足以下递归方程：

$$
T(e,n) = \max \{ T(e[0],n-1) , T(e[1],n-1)  \} + C(|e|) (6.2)
$$

（在归纳奠基$n=0$时，$T(e,0)$是某个只与$e$有关的常数。）

为了对式6.2有直观印象，我们展开一层递归，将$T(e,n)$写作
$$
\begin{aligned}
T(e,n) &= \max \{ T(e[0][0],n-2) + C(|e[0]|), \\ 
&T(e[0][1],n-2) + C(|e[0]|), \\
&T(e[1][0],n-2) + C(|e[1]|),  \\
&T(e[1][1],n-2) + C(|e[1]|) \} + C(|e|)
\end{aligned}
$$

如此继续，可以发现$T(e,n) \leq n \cdot C(L) + O(1)$，其中$L$是这么做时会遇到的最长的表达式$e'$的长度。
因此，如下声明足以说明算法6.14在运行时间是$O(n)$：

**声明**: 令$e$是定义在$\{0,1\}$上的正则表达式，则有$L(e)\in\N$使得对符号序列$\alpha_0,\ldots,\alpha_{n-1}$，再定义$e'=e[\alpha_0][\alpha_1]\cdots[\alpha_{n-1}]$（即，将$e$限制在$\alpha_0$上，然后是$\alpha_1$，以此类推），则$|e'| \leq L(e)$。

> [!QUOTE] 
> **对声明的证明**：对于一个定义在$\{0,1\}$上的正则表达式$e$和$\alpha\in \{0,1\}^m$，我们用$e[\alpha]$来指代表达式$e[\alpha_0][\alpha_1]\cdots [\alpha_{m-1}]$，其通过将$e$限制在$\alpha_0$上，再是$\alpha_1$，以此类推得到。
> 令$S(e) = \{ e[\alpha] | \alpha \in \{0,1\}^* \}$。
> 通过说明对每个$e$，集合$S(e)$是有限的，因此$L(e)$也一样，其为$e' \in S(e)$中$e'$的最大长度，从而证明该声明。
> 
> 我们通过在$e$的结构上做归纳证明这一点。如果$e$是符号，空串，或者空集，则可以直截了当地说明$S(e)$能含有的最多的表达式就是只有这个表达式本身，$""$和$\emptyset$。对其余情况，我们分为两类：**(i)** $e = e'^*$ 和 **(ii)** $e = e'e''$，其中$e',e''$是更小的表达式(因此根据归纳假设$S(e')$和$S(e'')$有限). 
>
> 在情况 **(i)** 中，若$e = (e')^*$则$e[\alpha]$要么等于$(e')^* e'[\alpha]$要么在$e'[\alpha]=\emptyset$时为空集合。因为$e'[\alpha]$在集合$S(e')$中，所以$S(e)$中不同表达式的个数最多为$|S(e')|+1$。
> 在情况 **(ii)** 中，若$e = e' e''$，则$e$在串$\alpha$上的所有限制要么具有形式$e' e''[\alpha]$，要么具有形式$e' e''[\alpha] | e'[\alpha']$，其中$\alpha'$为使得$\alpha = \alpha' \alpha''$成立的串，其中$e''[\alpha'']$匹配空串。
> 因为 $e''[\alpha] \in S(e'')$ 和 $e'[\alpha'] \in S(e')$，所以具有形式$e[\alpha]$的可能不同的表达式的数量最多有$|S(e'')| + |S(e'')|\cdot |S(e')|$个。这就完成了对该声明的证明。

最重要的是，在一个正则表达式$e$上运行算法6.14时，会遇到的所有表达式都在有限集$S(e)$中，不论输入$x$多大。因此算法6.14的运行时间满足等式$T(n) = T(n-1) + C'$，其中$C'$是依赖于$e$的常数。
最终解得$O(n)$，O记号中隐含的常数可以（且将会）依赖于$e$，并且，重要的是，不依赖于输入$x$的长度。

### 6.4.1 用DFAs匹配正则表达式

定理6.12非常令人印象深刻，但是我们可以做得更好。
准确的说，不管$x$有多长，都可以通过维护一个常数大小的内存并进行对$x$的*单次遍历*来计算$\Phi_e(x)$。
也就是说，这个算法将会从输入$x$的开头扫描到结尾，然后判定$x$是否被$e$匹配。
在一般的案例中，我们尝试用一个短的正则表达式匹配一个巨大的文件或文档，这些文件或文档甚至没法整个装在电脑的内存里。因此这种算法是否重要。
当然，如前所述，一个单遍常数内存算法仅仅就是一个确定性有穷自动机。
就像在定理6.17中将要看到的那样，一个函数能被正则表达式计算*当且仅当*它能被一个DFA计算。
我们从证明“仅当”开始：

> **定理6.15** 匹配正则表达式的DFA
> 
> 令$e$为正则表达式。则有输入$x \in\{0,1\}^*$的计算$\Phi_e(x)$算法，其对$x$进行单次遍历并维护一个常数大小的内存。

> 证明思路：
> 算法6.16给出了一个匹配正则表达式的单遍常数内存算法来检查正则表达式是否匹配一个串。其思路在于使用“记忆化搜索”的方法，将算法6.14这一个递归算法用[动态规划](https://goo.gl/kgLdX1)的算法替代。如果你还没有上过算法课，你可能不知道这些技巧，这没有关系；尽管这个更高效的算法对正则表达式的实践应用十分关键，对这本书却并不是很重要。

**算法6.16** 匹配正则表达式
$$
\begin{align}
&\input\text{定义在}\Sigma^*\text{上的正则表达式}e\text{，串}x\in\Sigma^n\text{其中}n\in\N\\
&\output\Phi_e(x)\\
&\proc{DFAMatch}(e,x)\\
&\quad\text{令}S\leftarrow S(e)\text{为线性时间匹配定理的证明中定义的集合}\{ e[\alpha] | \alpha\in \Sigma^* \}\\
&\quad\for{e' \in S} \\
&\qquad\text{当}\Phi_{e'}("")=1\text{时，令}v_{e'} \leftarrow 1\text{否则为}v_{e'} \leftarrow 0 \\
&\quad\endfor \\
&\quad\for{i \in [n]} \\
&\qquad\text{对每个}e' \in S\text{，令}last_{e'} \leftarrow v_{e'} \\
&\qquad\text{对每个}e' \in S\text{，令}v_{e'} \leftarrow last_{e'[x_i]} \\
&\quad\endfor \\
&\quad\return{v_e}\\
&\endproc
\end{align}
$$

> 证明：
算法6.16判定给定的串$x\in \Sigma^*$是否被正则表达式$e$所匹配。
对每个正则表达式，这个算法都有恒定数量的布尔变量（更准确地说，对每个$e' \in S(e)$有一个变量$v_{e'}$和$\last_{e'}$。该算法利用了一个事实：对每个$e' \in S(e)$，$e'[x_i]$都在$S(e)$中。
其对输入串进行单次遍历。因此与一个DFA一致。
我们通过归纳输入长度$n$来证明其正确性。
准确地说，我们将论证，在读入$x_i$之前，对每个$e; \in S(e)$，变量$v_{e'}$与$\Phi_{e'}(x_0 \cdots x_{i-1})$相等。
因为初始对每个$e' \in S(e)$，让$v_{e'} = \Phi_{e'}("")$,所以$i=0$的情况成立
对$i>0$的情况，归纳法证明其成立。归纳假设表明对每个$e' \in S(e)$，都有$last_{e'} = \Phi_{e'}(x_0 \cdots x_{i-2})$。而根据集合$S(e')$的定义，对每个$e' \in S(e)$，$x_{i-1} \in \Sigma$和$e'' = e'[x_{i-1}]$，$e'' = e'[x_{i-1}]$位于$S(e)$中而$\Phi_{e'}(x_0 \cdots x_{i-1}) = \Phi_{e''}(x_0 \cdots x_i)$。$\square$

### 6.4.2 正则表达式和自动机的等价性

回忆 以下，若存在某个正则表达式$e$，布尔函数$F:\{0,1\}^* \rightarrow \{0,1\}$与$Phi_e$相等，则称其为 *正则的* 。（等价地，若存在某个正则表达式$e$，语言$L \subseteq \{0,1\}^*$满足当且仅当$x\in L$时$e$匹配$x$，则称其为 *正则的* ）。下述定理是自动机理论的核心：

> **定理6.17** DFA与正则表达式的等价性
>
> 令$F:\{0,1\}^* \rightarrow \{0,1\}$。则$F$正则当且仅当存在DFA$(T,\mathcal{S})$计算$F$。

> 证明思路：
>
> 一个方向由定理6.15证明，其说明对每个正则表达式$e$，函数$\Phi_e$可以被一个DFA计算（见样例图6.6）。
> 在另一个方向上，我们说明给定一个DFA$(T,\mathcal{S})$，对每个$v,w \in [C]$都可以找到这样一个正则表达式：当且仅当DFA从状态$v$出发，在读取$x$后最终会到达$w$时，该正则表达式才匹配串$x\in \{0,1\}^*$。

> **图6.6**：
> ![automaton](./images/chapter6/automaton.png)
> 计算函数$\Phi_{(01)^*}$的确定性有穷自动机。

> **图6.7**：
> ![dfatoreg1](./images/chapter6/dfatoreg1.png)
> 给定一个$C$状态DFA，对于每个$v,w \in [C]$和数$t\in\{0,\ldots,C\}$，定义函数$F^t_{v,w}:\{0,1\}^* \rightarrow \{0,1\}$，其输入为$x\in \{0,1\}^*$。当且仅当DFA从状态$v$出发，在给定输入为$x$的情况下，最后会到达状态$w$，且过程中仅通过了中间状态$\{0,\ldots,t-1\}$，则函数值为$1$。

> 证明：
> 既然定理6.15已经证明了“仅当”方向，现在只需要证明“当”方向。
> 令$A=(T,\mathcal{S})$为一个$C$状态DFA，其计算函数$F$，需要证明$F$是正则的。
>
> 对每个$v,w \in [C]$，令$F_{v,w}:\{0,1\}^* \rightarrow \{0,1\}$为这样的函数：当且仅当DFA $A$从状态$v$出发，读入输入$x\in \{0,1\}^*$后会到达状态$w$，则其将$x$映射到$1$。
> 现在将要证明$F_{v,w}$对每个$v,w$都正则。这将证明该定理。因为根据定义6.2，$F(x)$等于对所有$F_{0,w}(x)$取或，其中$w\in \mathcal{S}$。
> 因此一旦能够为每个具有形式$F_{v,w}$的函数写出一个正则表达式，（通过使用$|$操作）也就可以得到$F$的正则表达式。
>
> 为了给出函数$F_{v,w}$的正则表达式，现在从定义函数$F_{v,w}^t$开始：对每个$v,w \in [C]$和$0 \leq t \leq C$，$F_{v,w}^t(x)=1$当且仅当自动机从$v$出发接受输入$x$后到达$w$且*所有的中间状态都在集合$[t]=\{0,\ldots, t-1\}$中*。（见图6.7）
>
> 这就是说，尽管$v,w$可能会在$[t]$之外，$F_{v,w}^t(x)=1$当且仅当在输入$x$（从$v$出发）时自动机运行过程中永不进入$[t]$之外的状态并在$w$结束。
> 当$t=0$时$[t]$就是空集，因此$F^0_{v,w}(x)=1$当且仅当自动机在输入$x$时直接从$v$转移到$w$而不经过任何的中间状态。
> 当$t=C$时所有的状态都在$[t]$中，因此$F_{v,w}^t=F_{v,w}$。
>
> 现在通过归纳$t$来证明这个定理，说明对所有$v,w$和$t$，$F^t_{v,w}$正则。
>
> 对于**归纳奠基**$t=0$，对所有的$v,w$，$F^0_{v,w}$都正则，因为它可以被表示为表达式$""$，$\emptyset$，$0$，$1$或$0|1$中的一个。
>
> 准确地说，若$v=w$，则$F^0_{v,w}(x)=1$当且仅当$x$为空串。
> 若$v\neq w$，则$F^0_{v,w}(x)=1$当且仅当$x$为单个字母$\sigma \in \{0,1\}$且$T(v,\sigma)=w$。
>
> 因此在这种情况中，$F^0_{v,w}$与四个正则表达式$0|1$，$0$，$1$和$\emptyset$中的一个相一致，并取决于$A$从$v$转移到$w$时读取的是$0$或$1$，还是仅为两个符号中的一者，或者都不是。
>
> **归纳步骤**：刚刚已经说明了归纳奠基，现在通过归纳法来证明一般情况。
> 归纳假设为对每个$v',w' \in [C]$，都有正则表达式$R_{v',w'}^t$计算$F_{v',w'}^t$。
> 需要证明的是对每个$v,w$，$F_{v,w}^{t+1}$正则。
> 如果自动机从$v$到$w$时访问了中间状态$[t+1]$，则其访问了第$t$个状态零次或多次。
> 
> 如果标号为$x$的路径使得自动机从$v$到$w$的过程中不需要访问第$t$个状态，则$x$被正则表达式$R_{v,w}^t$匹配；
> 如果标号为$x$的路径使得自动机从$v$到$w$的过程中需要访问第$t$个状态$k>0$次，则我们可以将该路径视为：
>
> * 首先，在只访问位于$[t-1]$的中间状态的情况下从$v$到$t$。
> * 然后，在只访问位于$[t-1]$的中间状态的情况下从$t$回到自身$k-1$次。
> * 最后，在只访问位于$[t-1]$的中间状态的情况下从$t$到$w$。
>
> 因此在该情况下，字符串被正则表达式$R_{v,t}^t(R_{t,t}^t)^* R_{t,w}^t$匹配。（又见图6.8）
> 因此我们可以使用以下正则表达式计算$F_{v,w}^{t+1}$：
> $$
> R_{v,w}^t \;|\; R_{v,t}^t(R_{t,t}^t)^* R_{t,w}^t\;
> $$
>
> 归纳步骤证明完毕，进而定理得证明。$\square$
> 
> **图6.8**：
> ![dfatoreginductivefig](./images/chapter6/dfatoreginductivefig.png)
> 若对于每个$v',w' \in [C]$，均有与$F_{v',w'}^{t}$相一致的正则表达式$R_{v',w'}^{t}$，则可以得到一个与$F_{v',w'}^{t+1}$相一致的正则表达式$R_{v',w'}^{t+1}$。关键的观察结果在于，从$v$到$w$的可能经过$\{0,\ldots, t \}$的路径，要么完全不通过$t$——这种情况被$R_{v,w}^{t}$所捕捉；要么从$v$到$t$，然后回到$t$零或多次，最终从$t$到$w$——这种情况被$R_{v,t}^{t}(R_{t,t}^{t})^* R_{t,w}^t$所捕捉。

### 6.4.3 正则表达式的闭包性质

If $F$  and $G$ are regular functions computed by the expressions $e$ and $f$ respectively, then the expression $e|f$ computes the function
$H = F \vee G$ defined as $H(x) = F(x) \vee G(x)$. 
Another way to say this is that the set of regular functions is _closed under the OR operation_.
That is, if $F$ and $G$ are regular then so is $F \vee G$.
An important corollary of [dfaregequivthm](){.ref} is that this set is also closed under the NOT operation:

### {.lemma title="Regular expressions closed under complement" #regcomplementlem}
If $F:\{0,1\}^* \rightarrow \{0,1\}$ is regular then so is the function $\overline{F}$, where $\overline{F}(x) = 1 - F(x)$ for every $x\in \{0,1\}^*$.

::: {.proof data-ref="regcomplementlem"}
If $F$ is regular then by [reglintimethm](){.ref} it can be computed by a DFA $A$. But we can then construct a DFA $\overline{A}$  which does the same computation but flips the set of accepted states. The DFA $\overline{A}$ will compute  $\overline{F}$.
By [dfaregequivthm](){.ref}  this implies that $\overline{F}$ is regular as well.
:::

Since $a \wedge b = \overline{\overline{a} \vee \overline{b}}$, [regcomplementlem](){.ref} implies that the set of  regular functions is closed under the AND operation as well. Moreover, since OR, NOT and AND are a universal basis, this set is also closed under NAND, XOR, and any other finite function.
That is, we have the following corollary:

> ### {.theorem title="Closure of regular expressions" #closurereg}
Let $f:\{0,1\}^k \rightarrow \{0,1\}$ be any finite Boolean function, and  let $F_0,\ldots,F_{k-1} : \{0,1\}^* \rightarrow \{0,1\}$ be regular functions.
Then the function $G(x) = f(F_0(x),F_1(x),\ldots,F_{k-1}(x))$ is regular.



::: {.proof data-ref="closurereg"}
This is a direct consequence of the closure of regular functions under OR and NOT (and hence AND), combined with [circuit-univ-thm](){.ref}, that states that every $f$ can be computed by a Boolean circuit (which is simply a combination of the AND, OR, and NOT operations).
:::










## 6.5 正则表达式的限制与泵引理 


The efficiency of regular expression matching makes them very useful.
This is why operating systems and text editors often restrict their search interface to regular expressions and do not allow searching by specifying an arbitrary function.
However, this efficiency comes at a cost.
As we have seen, regular expressions cannot compute every function. 
In fact, there are some very simple (and useful!) functions that they cannot compute.
Here is one example:

> ### {.lemma title="Matching parentheses" #regexpparn}
Let $\Sigma = \{\langle ,\rangle \}$ and  $MATCHPAREN:\Sigma^* \rightarrow \{0,1\}$ be the function that given a string of parentheses, outputs $1$ if and only if every opening parenthesis is matched by a corresponding closed one.
Then there is no regular expression over $\Sigma$ that computes $MATCHPAREN$.

[regexpparn](){.ref} is a consequence of the following result, which is known as the _pumping lemma_:

::: {.theorem title="Pumping Lemma" #pumping}
Let $e$ be a regular expression over some alphabet $\Sigma$. Then there is some number $n_0$ such that for every $w\in \Sigma^*$ with $|w|>n_0$ and $\Phi_{e}(w)=1$,  we can write $w=xyz$ for strings $x,y,z \in \Sigma^*$  satisfying the following conditions:

1. $|y| \geq 1$.

2. $|xy| \leq n_0$. 

3. $\Phi_{e}(xy^kz)=1$ for every $k\in \N$.
:::

![To prove the "pumping lemma" we look at a word $w$ that is much larger than the regular expression $e$ that matches it. In such a case, part of $w$ must be matched by some sub-expression of the form $(e')^*$, since this is the only operator that allows matching words longer than the expression. If we look at the "leftmost" such sub-expression and define $y^k$ to be the string that is matched by it, we obtain the partition needed for the pumping lemma.](../figure/pumpinglemma.png){#pumpinglemmafig   }

> ### {.proofidea data-ref="pumping"}
The idea behind the proof is the following.  Let $n_0$ be twice the number of symbols that are used in the expression $e$, then the only way that there is some $w$ with $|w|>n_0$ and $\Phi_{e}(w)=1$ is that $e$ contains the $*$ (i.e. star) operator and that there is a non-empty substring $y$ of $w$ that was matched by $(e')^*$ for some sub-expression $e'$ of $e$.  We can now repeat $y$ any number of times and still get a matching string. See also [pumpinglemmafig](){.ref}.

::: { .pause }
The pumping lemma is a bit cumbersome to state, but one way to remember it is that it simply says the following: _"if a string matching a regular expression is long enough, one of its substrings must be matched using the $*$ operator"_.
:::


::: {.proof data-ref="pumping"}
To prove the lemma formally, we use induction on the length of the expression.
Like all induction proofs, this will be somewhat lengthy, but at the end of the day it directly follows the intuition above that _somewhere_ we must have used the star operation. Reading this proof, and in particular understanding how the formal proof below corresponds to the intuitive idea above, is a very good way to get more comfortable with inductive proofs of this form.

Our inductive hypothesis is that for an $n$ length expression,  $n_0=2n$ satisfies the conditions of the lemma.
The __base case__ is when the expression is a single symbol $\sigma \in \Sigma$ or that the expression is $\emptyset$ or $""$.
In all these cases the conditions of the lemma are satisfied simply because $n_0=2$, and there exists no string $x$ of length larger than $n_0$ that is matched by the expression.

We now prove the __inductive step__.   Let $e$ be a regular expression with $n>1$ symbols.
We set $n_0=2n$ and let $w\in \Sigma^*$ be a string satisfying $|w|>n_0$.
Since $e$ has more than one symbol, it has  one of the forms __(a)__ $e' | e''$, __(b)__, $(e')(e'')$, or __(c)__ $(e')^*$ where in all these cases the subexpressions $e'$ and $e''$ have fewer symbols than $e$ and hence satisfy the induction hypothesis.


In the case __(a)__, every string $w$ matched by $e$ must be matched by either $e'$ or $e''$.
If $e'$ matches $w$ then, since $|w|>2|e'|$, by the induction hypothesis there exist $x,y,z$ with $|y| \geq 1$ and $|xy| \leq 2|e'| <n_0$  such that  $e'$ (and therefore also $e=e'|e''$) matches $xy^kz$  for every $k$. The same arguments works in the case that $e''$ matches $w$.


In the case __(b)__, if $w$ is matched by $(e')(e'')$ then we can write $w=w'w''$ where $e'$ matches $w'$ and $e''$ matches $w''$.
We split to subcases.
If $|w'|>2|e'|$ then by the induction hypothesis there exist $x,y,z'$ with $|y| \geq 1$, $|xy| \leq 2|e'| < n_0$ such that $w'=xyz'$ and $e'$ matches $xy^kz'$ for every $k\in \N$.
This completes the proof since if we set $z=z'w''$ then we see that $w=w'w''=xyz$ and $e=(e')(e'')$ matches $xy^kz$ for every $k\in \N$.
Otherwise, if $|w'| \leq 2|e'|$ then since $|w|=|w'|+|w''|>n_0=2(|e'|+|e''|)$, it must be that  $|w''|>2|e''|$.
Hence by the induction hypothesis there exist $x',y,z$ such that $|y| \geq 1$, $|x'y| \leq 2|e''|$ and $e''$ matches $x'y^kz$ for every $k\in \N$.
But now if we set $x=w'x'$ we see that $|xy| = |w'| + |x'y| \leq 2|e'| + 2|e''| =n_0$ and on the other hand  the expression $e=(e')(e'')$ matches $xy^kz = w'x'y^kz$ for every $k\in \N$.

In case __(c)__, if $w$ is matched by $(e')^*$ then $w= w_0\cdots w_t$ where for every $i\in [t]$, $w_i$ is a nonempty string matched by $e'$.
If $|w_0|>2|e'|$, then we can use the same approach as in the concatenation case above.
Otherwise, we simply note that if $x$ is the empty string, $y=w_0$, and $z=w_1\cdots w_t$ then $|xy| \leq n_0$ and $xy^kz$ is matched by $(e')^*$ for every $k\in \N$.
:::

> ### {.remark title="Recursive definitions and inductive proofs" #recursiveproofs}
When an object is _recursively defined_ (as in the case of regular expressions) then it is natural to prove properties of such objects by _induction_.
That is, if we want to prove that all objects of this type have property $P$, then it is natural to use an inductive step that says that if $o',o'',o'''$ etc have property $P$ then so is an object $o$ that is obtained by composing them.


Using the pumping lemma, we can easily prove [regexpparn](){.ref} (i.e., the non-regularity of the "matching parenthesis" function):

::: {.proof data-ref="regexpparn"}
Suppose, towards the sake of contradiction, that there is an expression $e$ such that $\Phi_{e}= MATCHPAREN$.
Let $n_0$ be the number obtained from  [pumping](){.ref} and let
$w =\langle^{n_0}\rangle^{n_0}$ (i.e., $n_0$ left parenthesis followed by $n_0$ right parenthesis). Then we see that if we write $w=xyz$ as in [pumping](){.ref}, the condition $|xy| \leq n_0$ implies that $y$ consists solely of left parenthesis. Hence the string $xy^2z$ will contain more left parenthesis than right parenthesis.
Hence $MATCHPAREN(xy^2z)=0$ but by the pumping lemma $\Phi_{e}(xy^2z)=1$, contradicting our assumption that $\Phi_{e}=MATCHPAREN$.
:::

The pumping lemma is a very useful tool to show that certain functions are _not_ computable by a regular expression.
However, it is _not_ an "if and only if" condition for regularity: there are non-regular functions that still satisfy the pumping lemma conditions.
To understand the pumping lemma, it is crucial to follow the order of quantifiers in [pumping](){.ref}.
In particular, the number $n_0$ in the statement of  [pumping](){.ref} depends on the regular expression (in the proof we chose $n_0$ to be twice the number of symbols in the expression).
So, if we want to use the pumping lemma to rule out the existence of a regular expression $e$ computing some function $F$, we need to be able to choose an appropriate input $w\in \{0,1\}^*$ that can be arbitrarily large and satisfies $F(w)=1$.
This makes sense if you think about the intuition behind the pumping lemma: we need $w$ to be large enough as to force the use of the star operator.


![A cartoon of a proof using the pumping lemma that a function $F$ is not regular. The pumping lemma states that if $F$ is regular then _there exists_ a number $n_0$ such that _for every_ large enough $w$ with $F(w)=1$, _there exists_ a partition of $w$ to $w=xyz$ satisfying certain conditions such that _for every_ $k\in \N$, $F(xy^kz)=1$. You can imagine a pumping-lemma based proof as a game between you and the adversary. Every _there exists_ quantifier corresponds to an object you are free to choose on your own (and base your choice on previously chosen objects). Every _for every_ quantifier corresponds to an object the adversary can choose arbitrarily (and again based on prior choices) as long as it satisfies the conditions. A valid proof corresponds to a strategy by which no matter what the adversary does, you can win the game by obtaining a contradiction which would be a choice of $k$ that would result in $F(xy^kz)=0$, hence violating the conclusion of the pumping lemma.](../figure/pumpinglemmaproof.png){#pumpingprooffig  .full  }

::: {.solvedexercise title="Palindromes is not regular" #palindromenotreg}
Prove that the following function over the alphabet $\{0,1,; \}$ is not regular: $PAL(w)=1$  if and only if $w = u;u^R$ where $u \in \{0,1\}^*$ and $u^R$ denotes $u$ "reversed": the string $u_{|u|-1}\cdots u_0$.
(The _Palindrome_ function is most often defined without an explicit separator character $;$, but the version with such a separator is a bit cleaner, and so we use it here. This does not make much difference, as one can easily encode the separator as a special binary string instead.)
:::

::: {.solution data-ref="stringreversed"}
We use the pumping lemma.
Suppose toward the sake of contradiction that there is a regular expression $e$ computing $PAL$, and let $n_0$ be the number obtained by the pumping lemma ([pumping](){.ref}).
Consider the string $w = 0^{n_0};0^{n_0}$.
Since the reverse of the all zero string is the all zero string, $PAL(w)=1$.
Now, by the pumping lemma, if $PAL$ is computed by $e$, then we can write $w=xyz$ such that $|xy| \leq n_0$, $|y|\geq 1$ and $PAL(xy^kz)=1$ for every $k\in \N$. In particular, it must hold that $PAL(xz)=1$, but this is a contradiction, since $xz=0^{n_0-|y|};0^{n_0}$ and so its two parts are not of the same length and in particular are not the reverse of one another.
:::

For yet another example of a pumping-lemma based proof, see [pumpingprooffig](){.ref} which illustrates a cartoon of the proof of the non-regularity of the function $F:\{0,1\}^* \rightarrow \{0,1\}$ which is defined as $F(x)=1$ iff $x=0^n1^n$ for some $n\in \N$ (i.e., $x$ consists of a string of consecutive zeroes, followed by a string of consecutive ones of the same length).



## 6.6 对正则表达式语义问题的回答


Regular expressions have applications beyond search.
For example, regular expressions are often used to define _tokens_ (such as what is a valid variable identifier, or keyword) in the design of
_parsers_, _compilers_ and _interpreters_ for programming languages.
Regular expressions have other applications too:  for example, in recent years, the world of networking moved from fixed topologies to "software defined networks".
Such networks are routed by programmable switches that can implement _policies_ such as "if packet is secured by SSL then forward it to A, otherwise forward it to B".
To represent such policies we need a language that is on one hand sufficiently expressive to capture the policies we want to implement, but on the other hand sufficiently restrictive so that we can quickly execute them at network speed and also be able to answer questions such as "can C see the packets moved from A to B?".
The  [NetKAT network programming language](https://goo.gl/oeJNuw) uses a variant of regular expressions to achieve precisely that.
For this application, it is important that we are not merely able to answer whether an expression $e$ matches a string $x$ but also answer
_semantic questions_ about regular expressions such as "do expressions $e$ and $e'$ compute the same function?" and "does there exist a string $x$ that 
is matched by the expression $e$?".
The following theorem shows that we can answer the latter question:


> ### {.theorem title="Emptiness of regular languages is computable" #regemptynessthm}
There is an algorithm that given a regular expression $e$, outputs $1$ if and only if $\Phi_{e}$ is the constant zero function.

> ### {.proofidea data-ref="regemptynessthm"}
The idea is that we can directly observe this from the structure of the expression. The only way a regular expression $e$ computes the constant zero function is if $e$ has the form $\emptyset$ or is obtained by concatenating $\emptyset$ with other expressions.

::: {.proof data-ref="regemptynessthm"}
Define a regular expression to be "empty" if it computes the constant zero function.
Given a regular expression $e$, we can determine if $e$ is empty using  the following rules:

* If $e$ has the form $\sigma$ or $""$ then it is not empty.

* If $e$ is not empty then $e|e'$ is not empty for every $e'$.

* If $e$ is not empty then $e^*$ is not empty.

* If $e$ and $e'$ are both not empty then $e\; e'$  is not empty.

* $\emptyset$ is empty.

Using these rules, it is straightforward to come up with a recursive algorithm to determine emptiness. 
:::

Using [regemptynessthm](){.ref}, we can obtain an algorithm that determines whether or not two regular expressions $e$ and $e'$ are _equivalent_, 
in the sense that they compute the same function.

> ### {.theorem title="Equivalence of regular expressions is computable" #regequivalencethm}
Let $REGEQ:\{0,1\}^* \rightarrow \{0,1\}$ be the function that on input (a string representing) a pair of regular expressions $e,e'$, $REGEQ(e,e')=1$ if and only if $\Phi_{e} = \Phi_{e'}$. Then there is an algorithm that computes  $REGEQ$.



> ### {.proofidea data-ref="regequivalencethm"}
The idea is to show that given a pair of regular expressions $e$ and $e'$ we can find an expression $e''$ such that $\Phi_{e''}(x)=1$ if and only if $\Phi_e(x) \neq \Phi_{e'} (x)$. Therefore $\Phi_{e''}$ is the constant zero function if and only if $e$ and $e'$ are equivalent, and thus we can test for emptiness of $e''$ to determine equivalence of $e$ and $e'$.


::: {.proof data-ref="regequivalencethm"}
We will prove  [regequivalencethm](){.ref} from [regemptynessthm](){.ref}. (The two theorems are in fact equivalent: it is easy to prove [regemptynessthm](){.ref} from [regequivalencethm](){.ref}, since checking for emptiness is the same as checking equivalence with the expression $\emptyset$.)
Given two regular expressions $e$ and $e'$, we will compute an expression $e''$ such that $\Phi_{e''}(x) =1$ if and only if $\Phi_e(x) \neq \Phi_{e'}(x)$.
One can see that $e$ is equivalent to $e'$ if and only if $e''$ is empty.

We start with the observation that for every bit $a,b \in \{0,1\}$, $a \neq b$ if and only if
$$
(a \wedge \overline{b}) \; \vee \;  (\overline{a} \wedge b) \;.
$$

Hence we need to construct $e''$ such that for every $x$,

$$
\Phi_{e''}(x) = (\Phi_{e}(x) \wedge \overline{\Phi_{e'}(x)}) \; \vee  \; (\overline{\Phi_{e}(x)} \wedge \Phi_{e'}(x)) \;.
\label{eqemptyequivreg}
$$

To construct the expression $e''$, we will show how given any pair of expressions $e$ and $e'$, we can construct expressions $e\wedge e'$ and $\overline{e}$ that compute the functions $\Phi_{e} \wedge \Phi_{e'}$ and $\overline{\Phi_{e}}$ respectively. (Computing the expression for $e \vee e'$ is straightforward using the $|$ operation of regular expressions.)

Specifically, by [regcomplementlem](){.ref}, regular functions are closed under negation, which means that for every regular expression $e$, there is an expression $\overline{e}$ such that $\Phi_{\overline{e}}(x) = 1 - \Phi_{e}(x)$ for every $x\in \{0,1\}^*$.
Now, for every two expressions $e$ and $e'$, the expression
$$
e \wedge e' = \overline{(\overline{e} | \overline{e'})}
$$
computes the AND of the two expressions.
Given these two transformations, we see that for every regular expressions $e$ and $e'$ we can find a regular expression $e''$ satisfying [eqemptyequivreg](){.eqref} such that $e''$ is empty if and only if  $e$ and $e'$ are equivalent.
:::


> ### { .recap }
* We model computational tasks on arbitrarily large inputs using _infinite_ functions $F:\{0,1\}^* \rightarrow \{0,1\}^*$. 
* Such functions take an arbitrarily long (but still finite!) string as input, and cannot be described by a finite table of inputs and outputs.
* A function with a single bit of output is known as a _Boolean function_, and the task of computing it is equivalent to deciding a _language_ $L\subseteq \{0,1\}^*$.
* _Deterministic finite automata_ (DFAs) are one simple model for computing (infinite) Boolean functions.
* There are some functions that cannot be computed by DFAs. 
* The set of functions computable by DFAs is the same as the set of languages that can be recognized by regular expressions.


## 6.7 练习


::: {.exercise title="Closure properties of regular functions" #closureregex}
Suppose that $F,G:\{0,1\}^* \rightarrow \{0,1\}$ are regular. For each one of the following definitions of the function $H$, either prove that $H$ is always regular or give a counterexample for regular $F,G$ that would make $H$ not regular.

1. $H(x) = F(x) \vee G(x)$.

2. $H(x) = F(x) \wedge G(x)$

3. $H(x) = NAND(F(x),G(x))$.

4. $H(x) = F(x^R)$ where $x^R$ is the reverse of $x$: $x^R = x_{n-1}x_{n-2} \cdots x_o$ for $n=|x|$.

5. $H(x) = \begin{cases}1 & x=uv \text{ s.t. } F(u)=G(v)=1 \\  0 & \text{otherwise} \end{cases}$

6. $H(x) = \begin{cases}1 & x=uu \text{ s.t. } F(u)=G(u)=1 \\  0 & \text{otherwise} \end{cases}$


7. $H(x) = \begin{cases}1 & x=uu^R \text{ s.t. } F(u)=G(u)=1 \\  0 & \text{otherwise} \end{cases}$
:::



::: {.exercise  #regularno}
One among the following two functions that map $\{0,1\}^*$ to $\{0,1\}$ can be computed by a regular expression, and the other one cannot. For the one that can be computed by a regular expression, write the expression that does it. For the one that cannot, prove that this cannot be done using the pumping lemma.

* $F(x)=1$ if $4$ divides $\sum_{i=0}^{|x|-1} x_i$ and  $F(x)=0$ otherwise.

* $G(x) = 1$ if and only if $\sum_{i=0}^{|x|-1} x_i \geq |x|/4$ and $G(x)=0$ otherwise.
:::

::: {.exercise title="Non-regularity" #nonregex}
1. Prove that the following function $F:\{0,1\}^* \rightarrow \{0,1\}$ is not regular. For every $x\in \{0,1\}^*$, $F(x)=1$ iff $x$ is of the form $x=1^{3^i}$ for some $i>0$. 

2.  Prove that the following function $F:\{0,1\}^* \rightarrow \{0,1\}$ is not regular. For every $x\in \{0,1\}^*$, $F(x)=1$ iff  $\sum_j x_j = 3^i$ for some $i>0$. 
:::


## 6.8 参考文献


The relation of regular expressions with finite automata is a beautiful topic, on which we only touch upon in this text.
It is covered more extensively in [@SipserBook, @hopcroft, @kozen1997automata].
These texts also discuss topics such as _non-deterministic finite automata_ (NFA) and the relation between context-free grammars and pushdown automata.

The automaton of [dfazeroonefig](){.ref} was generated using the [FSM simulator](http://ivanzuzak.info/noam/webapps/fsm_simulator/) of Ivan Zuzak and Vedrana Jankovic.
Our proof of [reglintimethm](){.ref} is closely related to the [Myhill-Nerode Theorem](https://goo.gl/mnKVMP). One direction of the Myhill-Nerode theorem can be stated as saying that if $e$ is a regular expression then there is at most a finite number of strings $z_0,\ldots,z_{k-1}$ such that $\Phi_{e[z_i]} \neq \Phi_{e[z_j]}$ for every $0 \leq i\neq j < k$.