```admonish warning 
**本章施工中**
```

# 运行时间建模 { #chapmodelruntime }

## 学习目标
* 形式化的建模程序的运行时间, 特别是诸如 $O(n)$ 或 $O(n^3)$ 时间算法的概念 \
* 分别对多项式时间和指数时间进行建模的复杂性类 $\mathbf{P}$ 和 $\mathbf{EXP}$.  \
* 了解时间层级定理, 即对于任意 $k \geq 1$ 都存在我们可以在 $O(n^{k+1})$ 时间计算但不能在 $O(n^k)$ 时间计算的函数. 
* 代表非一致性计算 $\mathbf{P_{/poly}}$ 的复杂性类, 及 $\mathbf{P} \subseteq \mathbf{P_{/poly}}$ 这一结论. 

```admonish quote
当问题规模的度量标准合理, 且当规模取值任意大时, 对算法难度的阶进行渐近估计在理论上具有重要意义. 这种估计无法被操纵——即无法通过人为地使算法在较小规模下变得困难而扭曲结果. 

*-杰克·埃德蒙兹, "Paths, Trees, and Flowers", 1963.*
```

```admonish quote
*马克斯·纽曼*: “声称机器‘能够’做这做那固然很好, 但是……它做这些事情究竟需要花费多少时间呢？”

*艾伦·图灵*: “在我看来, 这个时间因素正是所有真正的技术难点所在. ”

*-BBC 广播座谈节目 "我们可以说计算机会思考吗？", 1952.*
```

在 [第12章](chapter_12.md) 中, 我们介绍了一些高效的算法, 并且对他们的运行时间做了一些假设, 不过并未对这些算法的运行时间进行精确数学定义. 我们将在本章节中借助我们之前已经介绍过的图灵机和 RAM（或等价的 NAND-TM 和 NAND-RAM）机完成这一工作. 
任何非平凡的算法都会在更大规模的输入上运行更长的时间, 因此算法的运行时间并不能用一个确定的数字来表示. 
因此, 我们想要确定的是算法需要运行的步数和输入长度的关系. 
我们特别关注以下两者之间的区别, 那些最多只需多项式时间（即对于某个常数 $c$, 时间为 $O(n^c)$）的算法, 与那些任何算法都至少需要指数时间（即对于某个 $c$, 时间为 $\Omega(2^{n^c})$）的问题. 
正如 [第12章](chapter_12.md) 中 Edmonds 的引言所提到的, 这两者之间的差异, 有时与可计算和不可计算之间的差异一样重要. 

```admonish pic id = 'picoverviewfig'
![overview](./images/chapter13/runtimeoverview.png)

{{pic}}{fig:picoverview} 本章得到结果的概览
```

```admonish info title = "本章：一个直观的概述"
在这一章中我们形式化的定义一个函数可以被在确定的步数下计算意味着什么. 
正如在 [第12章](chapter_12.md) 中所说的那样, 运行时间并不是一个数字, 我们关心的是随着输入规模增大, 算法运行步数会以怎样的规模增长. 
我们可以用图灵机或 RAM 机来给出一个形式化定义 - 事实上模型的选择并不影响这个问题的核心解决方案
本章我们将给出几个重要定义并证明一些重要的定理. 
我们将定义本书中使用的主要时间复杂性类, 并展示时间层级定理, 该定理表明：如果给予更多的资源（即针对每个输入规模允许更多的执行步数）, 我们就能够计算更多的函数
```

要将这一切用不那么数学化的语言表述出来, 我们将定义能在 $T(n)$ 步内将函数 $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 计算出来的含义, 其中 $T$ 是一个将输入长度 $n$ 映射到计算所需的步数的函数. 
使用这些定义, 我们将做以下事情（可参考 {{ref:fig:picoverview}}）

* 我们定义复杂性类 $\mathbf{P}$ 为可以在多项式时间内计算的布尔函数的集合, 复杂性类 $\mathbf{EXP}$ 为可以在指数时间内计算的函数的集合. 注意 $\mathbf{P} \subseteq \mathbf{EXP}$, 即如果我们能在多项式时间内计算一个函数, 那么当然也能在指数时间内计算他. 

* 我们证明, 用图灵机和RAM机计算一个函数所需的时间是多项式相关的. 这意味着, 无论使用图灵机还是 RAM 机（或 NAND-RAM 机）来定义, $\mathbf{P}$ 和 $\mathbf{EXP}$ 总是相同. 

* 我们给出一个高效且通用的 NAND-RAM 程序, 并使用它建立时间层级定理, 该定理意味着 $\mathbf{P}$ 是 $\mathbf{EXP}$ 的真子集. 

* 我们将此处定义的概念与 [第3章](chapter_3.md) 中定义的布尔电路和 NAND-CIRC 程序等非一致性模型联系起来. 我们将 $\mathbf{P_{/poly}}$ 定义为可以由一系列多项式大小的电路所计算的函数类. 我们证明了 $\mathbf{P} \subseteq \mathbf{P_{/poly}}$, 且 $\mathbf{P_{/poly}}$ 包含不可计算函数. 

## 13.1 形式化的定义运行时间

我们的计算模型（图灵机, NAND-TM 和 NAND-RAM 程序等）都是通过其运作方式都是对输入逐步执行一系列指令. 
我们可以通过测量算法 $M$ 在输入 $x$ 上执行的步数, 并将其表示为输入长度 $|x|$ 的函数, 从而定义算法 $M$ 在这些模型下的运行时间. 
我们首先定义基于图灵机的运行时间：

```admonish quote title=""
{{defc}}{def:time_tm_def}[运行时间（图灵机）] 

令 $T:\N \rightarrow \N$ 为某个实数到实数的映射. 
如果存在一台图灵机 $M$, 使得对于每一个充分大的 $n$ 和每一个 $x\in \{0,1\}^n$, 当给定输入 $x$ 时, 机器 $M$ 在执行最多 $T(n)$ 步后停机并输出 $F(x)$, 那么我们称函数 $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 是在 $T(n)$ 图灵机时间（Turing Machine Time, 简称 TM 时间）内可计算的. 
我们定义 $TIME_{\mathsf{TM}}(T(n))$ 为所有在 $T(n)$ 图灵机时间内可计算的布尔函数（即映射 $\{0,1\}^*$ 到 $\{0,1\}$ 的函数）的集合. 
```

```admonish bigidea
{{idec}}{ide:formaldefinetime}
对于函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 和 $T:\N  \rightarrow \N$, 我们可以形式化的定义 $F$ 能在至多 $T(n)$ 的时间内计算意味着什么, 其中 $n$ 为输入规模. 
```

```admonish pause title = "暂停一下"
{{ref:def:time_tm_def}} 并不复杂, 但这是本书中最为重要的定义之一. 照例, $TIME_{\mathsf{TM}}(T(n))$ 代表一类函数, 而不是机器类. 若 $M$ 是图灵机, 则像 “$M$ 属于 $TIME_{\mathsf{TM}}(n^2)$” 这样的表述并不正确. 
此处定义的 TM 时间（图灵机时间）概念在文献中有时被称为“单带图灵时间”（single-tape Turing machine time）, 这是因为有些文献会考虑拥有多条工作带的图灵机. 
```

放宽条件只考虑充分大的 $n$ 虽然本质上并不是很重要, 但却非常便利, 因为这使我们能够避免讨论一些无趣的边界情况. 
尽管“函数的运行时间”这一概念可以在任意函数上定义, 但在定义 $TIME_{\mathsf{TM}}(T(n))$ 类时, 我们只考虑布尔函数, 即那些只有一个 bit 输出的函数. 
这一选择并不重要, 是为了后续讨论的简洁与便利而做出的. 
事实上, 任何一个非布尔函数都有一个与之计算等价的布尔变体, 参见 {{ref:exe:boolex}}

```admonish quote title=""
{{exec}}{exe:timeboundexample}[时间界限的示例] 
证明 $TIME_{\mathsf{TM}}(10\cdot n^3) \subseteq TIME_{\mathsf{TM}}(2^n)$. 
```

```admonish pic id = 'examplefimeboundsfig'
![exampletimebounds](./images/chapter13/exampletimebounds.png)

{{pic}}{fig:examplefimebounds} 将 $T(n)=10n^3$ 和 $T'(n) = 2^n$ 比较（右图的 Y 轴采用对数标度）. 因为对于足够大的 $n$, $T'(n) \geq T(n)$, 则 $TIME_{\mathsf{TM}}(T(n)) \subseteq TIME_{\mathsf{TM}}(T'(n))$. 
```

```admonish solution collapsible=true, title = "对 {{ref:exe:timeboundexample}} 的解答" 
证明其实已经在 {{ref:fig:examplefimebounds}} 中展示了. 
假设 $F\in TIME_{\mathsf{TM}}(10\cdot n^3)$, 则存在数 $N_0$ 和计算模型 $M$, 满足对于任意 $n > N_0$, $x\in \{0,1\}^*$, 都有 $M(x)$ 会在最多 $10\cdot n^3$ 步内输出 $F(x)$ 的结果. 
因为 $10\cdot n^3 = o(2^n)$, 一定存在数 $N_1$ 满足对于任意 $n > N_1$, 都有 $10 \cdot n^3 < 2^n$. 
则对于任意 $n > \max\{ N_0, N_1 \}$, $M(x)$ 会在至多 $2^n$ 步内输出 $F(x)$ 的结果, 即证明了 $F \in TIME_{\mathsf{TM}}(2^n)$. 
```

### 13.1.1 多项式时间和指数时间

与可计算性的概念不同, 精确的运行时间可能会取决于我们所使用的计算模型. 然而, 事实上, 如果我们只关心“足够粗糙”的尺度（大部分情况下都是如此）, 那么模型的选择——无论是图灵机、RAM 机、NAND-TM/NAND-RAM 程序, 还是 C/Python 程序——都无关紧要了. 
这就是所谓的*扩展*Church-Turing论题 (*extended* Church-Turing Thesis). 
具体来说, 我们主要关心的是多项式时间与指数时间之前的区别. 

我们将关注以下两个主要的时间复杂性类:

* __多项式时间:__ 如果一个函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 属于类 $\mathbf{P} = \cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c)$, 则称其是 __多项式时间可计算__ 的. 也就是说, 若 $F\in \mathbf{P}$, 则存在一个计算 $F$ 的算法, 其运行时间关于输入长度至多是多项式的（换言之, 对于某个常数 $c$, 至多 $n^c$）. 

* __指数时间:__ 如果一个函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 属于类 $\mathbf{EXP} = \cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(2^{n^c})$, 则称其是 __指数时间可计算__ 的. 也就是说, 若 $F\in \mathbf{EXP}$, 则存在一个计算 $F$ 的算法, 其运行时间关于输入长度至多是指数的（换言之, 对于某个常数 $c$, 至多 $2^{n^c}$）. 

形式化的说, 他们是如下定义的. 

```admonish quote title=""
{{defc}}{def:PandEXP}[$\mathbf{P}$ 和 $\mathbf{EXP}$] 

设函数 $F:\{0,1\}^* \rightarrow \{0,1\}$. 

若存在一个多项式 $p:\N \rightarrow \R$, 和一个图灵机 $M$, 满足对于任意 $x\in \{0,1\}^*$, 当给出输入 $x$ 时, 图灵机将在至多 $p(|x|)$ 步内停机并输出 $F(x)$, 则我们称 $F\in \mathbf{P}$. 

若存在一个多项式 $p:\N \rightarrow \R$, 和一个图灵机 $M$, 满足对于任意 $x\in \{0,1\}^*$, 当给出输入 $x$ 时, 图灵机将在至多 $2^{p(|x|)}$ 步内停机并输出 $F(x)$, 则我们称 $F\in \mathbf{EXP}$. 
```

```admonish pause title = "暂停一下"
请务必花点时间, 确保你透彻理解了这些定义.
特别需要注意的是, 学生们有时会误以为 $\mathbf{EXP}$ 类指的是那些不在 $\mathbf{P}$ 中的函数.
然而, 事实并非如此. 如果 $F$ 属于 $\mathbf{EXP}$, 这意味着它能够在指数时间内被计算出来.
这并不意味着它不能同时在多项式时间内被计算.
```

```admonish quote title=""
{{exec}}{exe:diffdefofP}[$\mathbf{P}$ 的另一定义] 

证明 {{ref:def:PandEXP}} 中定义的 $\mathbf{P}$ 与 $\cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c)$ 等价. 
```

```admonish solution collapsible=true, title = "对 {{ref:exe:diffdefofP}} 的解答" 
为了证明这两个集合相等, 我们可证明 $\mathbf{P} \subseteq \cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c)$ 以及 $\cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c) \subseteq \mathbf{P}$. 

我们从前一个包含关系开始. 假设 $F \in \mathbf{P}$. 那么存在某个多项式 $p:\N \rightarrow \R$ 和一台图灵机 $M$, 使得 $M$ 能计算 $F$, 并且对于每一个输入 $x$, $M$ 都在至多 $p(|x|)$ 步内停机. 我们可以将多项式 $p:\N \rightarrow \R$ 写成 $p(n) = \sum_{i=0}^d a_i n^i$ 的形式, 其中 $a_0,\ldots,a_d \in \R$, 并且我们假设 $a_d$ 非零（否则我们就让 $d$ 对应使得 $a_d$ 非零的最大数）. 这个 $d$ 即为 $p$ 的次数（degree）. 由于 $n^d = o(n^{d+1})$, 无论系数 $a_d$ 是多少, 对于足够大的 $n$, 都有 $p(n) < n^{d+1}$. 这意味着图灵机 $M$ 在处理长度为 $n$ 的输入时, 会在少于 $n^{d+1}$ 步内停机, 因此 $F \in TIME_{\mathsf{TM}}(n^{d+1}) \subseteq \cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c)$. 

对于第二个包含关系, 假设 $F \in \cup_{c\in \{1,2,3,\ldots \}} TIME_{\mathsf{TM}}(n^c)$. 那么存在某个正整数 $c \in \N$ 使得 $F \in TIME_{\mathsf{TM}}(n^c)$, 这意味着存在一台图灵机 $M$ 和某个数值 $N_0$, 使得 $M$ 能计算 $F$, 并且对于每一个 $n>N_0$, $M$ 在处理长度为 $n$ 的输入时, 都在至多 $n^c$ 步内停机. 设 $T_0$ 为 $M$ 在处理长度至多为 $N_0$ 的输入时所花费的最大步数. 那么, 如果我们定义多项式 $p(n) = n^c + T_0$, 我们就会发现 $M$ 在处理每一个输入 $x$ 时都在至多 $p(|x|)$ 步内停机, 因此 $M$ 的存在证明了 $F\in \mathbf{P}$. 
```


因为指数时间比多项式时间大得多, $\mathbf{P}\subseteq \mathbf{EXP}$ 类.
我们在 [第12章](chapter_12.md) 中列出的所有问题都属于 $\mathbf{EXP}$, 不过如我们所见, 对于他们中的一些问题存在更高效的算法，这证明了他们实际上属于更小的 $\mathbf{P}$ 类.


```admonish table title=""
| $\mathbf{P}$             | $\mathbf{EXP}$ (但目前不知道属于 $\mathbf{P}$) |
|--------------------------|---------------------------|
| 最短路            | 最长路              |
| 最小割                  | 最大割                   |
| 2SAT                     | 3SAT                      |
| 解线性方程组               | 解二次方程组                 |
| 零和博弈                  | 纳什均衡                      |
| 行列式              | 积和式                 |
| 素数判定                | 整数分解                 |

这是一个来自 [第12章](chapter_12.md) 的表格. 表格中的所有问题都属于 $\mathbf{EXP}$ 类但只有左列中的问题目前已知属于 $\mathbf{P}$ 类. (换言之, 他们有多项式时间的算法). 参见 {{ref:fig:PvsEXP}}.
```

```admonish pic id = 'PvsEXPfig'
![PvsEXPfig](./images/chapter13/PvsEXP.png)

{{pic}}{fig:PvsEXP} 一些在 $\mathbf{P}$ 类中的问题和一些在 $\mathbf{EXP}$ 类中但不知道在不在 $\mathbf{P}$ 类中的问题的例子. 因为 $\mathbf{P}$ 和 $\mathbf{EXP}$ 都是布尔函数的类, 在此图中，我们始终指的是这些问题的布尔变体 (即只关心是/否).
```

```admonish remark title="问题的布尔版本"
{{remc}}{rem:booleanversion}
[第12章](chapter_12.md) 中定义的许多问题都对应于非布尔函数 (即输出超过一个 bit 的函数), 而 $\mathbf{P}$ 和 $\mathbf{EXP}$ 是布尔函数的集合.
然而, 对于每一个非布尔函数 $F$, 我们总是可以通过定义 $G(x,i)$ 为 $F(x)$ 的第 $i$ 个比特, 来定义一个与之等价的布尔函数 $G$ (参见 {{ref:exe:boolex}}).
因此, 上表以及 {{ref:fig:PvsEXP}} 中所指的, 都是这些问题的计算等价布尔变体.
```


## 13.2 使用 RAM 机 / NAND-RAM 建模运行时间

图灵机虽然是一种简洁的理论计算模型, 但它与现实世界的计算架构并不十分吻合. 当我们考虑哪些函数是"可计算的"这一问题时, 图灵机与实际计算机之间的这种差异关系不大; 但在涉及"效率"的语境下, 这种差异就会产生影响. 甚至是本科算法课程中的基础内容——如"归并排序", 也无法在图灵机上以 $O(n\log n)$ 的时间实现 (参见 {{ref:bibnotesrunningtime}}). RAM 机 (或等价的 NAND-RAM 程序) 更接近实际的计算架构, 也更符合我们在算法课程或白板编程面试中所说的 $O(n)$ 或 $O(n \log n)$ 算法的含义. 我们可以像定义图灵机那样, 定义针对 NAND-RAM 程序的运行时间.



```admonish quote title=""
{{defc}}{def:time_def}[运行时间 (RAM)] 

设 $T:\N \rightarrow \N$ 是某个将自然数映射到自然数的函数.
我们称函数 $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 是 **在 $T(n)$ RAM 时间内可计算的** (简称 RAM 时间), 如果存在一个 NAND-RAM 程序 $P$, 使得对于每一个足够大的 $n$ 和每一个 $x\in \{0,1\}^n$, 当给定输入 $x$ 时, 程序 $P$ 在执行至多 $T(n)$ 行指令后停机, 并输出 $F(x)$.

我们定义 $TIME_{\mathsf{RAM}}(T(n))$ 为在 $T(n)$ RAM 时间内可计算的布尔函数 (即映射 $\{0,1\}^*$ 到 $\{0,1\}$ 的函数) 的集合.
```

因为 NAND-RAM 程序更加符合我们对运行时间的直观理解, 我们将把 NAND-RAM 作为我们讨论运行时间的默认模型, 并因此使用不带任何下标的 $TIME(T(n))$ 来表示 $TIME_{\mathsf{RAM}}(T(n))$.
然而, 事实证明, 只要我们只关心指数时间和多项式时间之间的区别, 模型的选择并没有太大影响.
原因是图灵机可以模拟 NAND-RAM 程序, 且其开销至多是多项式级别的 (参见 {{ref:fig:RAMTMsimulation}}):


```admonish quote title=""
{{thmc}}{thm:polyRAMTM}[连接图灵机和 RAM 机]

$T:\N \rightarrow \N$ 为一个函数, 满足对任意 $n$ 都有 $T(n) \geq n$, 且映射 $n \mapsto T(n)$ 可以由一台图灵机在 $O(T(n))$ 时间内计算得出.
那么: 

$$
TIME_{\mathsf{TM}}(T(n)) \subseteq TIME_{\mathsf{RAM}}(10\cdot T(n)) \subseteq TIME_{\mathsf{TM}}(T(n)^4). {{numeq}}{eq:tmrambisimulation}
$$
```


```admonish pause title = "暂停一下"
{{ref:thm:polyRAMTM}} 中的一些技术细节并不重要, 如要求 $n \mapsto T(n)$ 可以在 $O(T(n))$ 时间内被计算出来的条件, 或者 {{ref:eq:tmrambisimulation}} 中的常数 $10$ 和 $4$ (这些常数并非紧致的, 是可以被改进的)
特别的, 我们在实践中遇到的所有非病态的时间界限函数, 如 $T(n)=n$, $T(n)=n\log n$, $T(n)=2^n$ 等, 都满足 {{ref:thm:polyRAMTM}} 的条件 (另见 {{ref:rem:nicefunctions}})

该定理的核心信息是: 图灵机和 RAM 机是"大致等价"的, 在这个意义上, 其中一个可以模拟另一个, 且只产生多项式级别的开销.
同样地, 虽然证明过程涉及一些技术细节, 但它并不深奥也不困难, 仅仅是沿用了我们在 [定理8.1](./chapter_8.md#thm:RAMTMequivalencethm) 中看到的用图灵机模拟 RAM 机的方法, 只是做了更仔细的"簿记" (即状态维护) 工作.
```


```admonish pic id = 'RAMTMsimulationfig'
![RAMTMsimulationfig](./images/chapter13/RAMTMsimulation.png)

{{pic}}{fig:RAMTMsimulation} {{ref:thm:polyRAMTM}} 的证明表明, 我们可以用 $T$ 步的 NAND-RAM 程序来模拟 $T$ 步的图灵机, 并且可以用 $o(T^4)$ 步的图灵机来模拟 $T$ 步的 NAND-RAM 程序.
因此, $TIME_{\mathsf{TM}}(T(n)) \subseteq TIME_{\mathsf{RAM}}(10\cdot T(n)) \subseteq TIME_{\mathsf{TM}}(T(n)^4)$.
```


例如, 通过将 $T(n)=n^a$ 代入 {{ref:thm:polyRAMTM}}, 并利用 $10n^a = o(n^{a+1})$ 这一事实, 我们看到 $TIME_{\mathsf{TM}}(n^a) \subseteq TIME_{\mathsf{RAM}}(n^{a+1}) \subseteq TIME_{\mathsf{TM}}(n^{4a+4})$. 这意味着 (根据 {{ref:exe:diffdefofP}}): 
$$
\mathbf{P} = \cup_{a = 1,2,\ldots} TIME_{\mathsf{TM}}(n^a) = \cup_{a = 1,2,\ldots} TIME_{\mathsf{RAM}}(n^a) \;.
$$
也就是说, 我们完全可以将 $\mathbf{P}$ 定义为由 *NAND-RAM 程序* (而不是图灵机) 在输入长度的多项式时间内计算的函数类. 同样地, 通过将 $T(n)=2^{n^a}$ 代入 {{ref:thm:polyRAMTM}}, 我们看到 $\mathbf{EXP}$ 类也可以定义为由 NAND-RAM 程序在至多 $2^{p(n)}$ 时间内计算的函数集, 其中 $p$ 为某个多项式. 对于许多其他模型, 包括元胞自动机, C/Python/Javascript 程序, 并行计算机以及许多其他模型, 已知都存在类似的等价结果. 这证明了选择 $\mathbf{P}$ 作为捕捉独立于技术的"易处理性"概念是合理的 (参见 {{ref:ECTTsec}} 关于此问题的更多讨论). 图灵机和 NAND-RAM (以及其他模型) 之间的这种等价性, 允许我们根据手头的任务选择我们最喜欢的模型 (即"鱼与熊掌兼得"), 即使在研究效率问题时也是如此---只要我们只关心*多项式时间*和*指数时间*之间的差距. 当我们想要*设计*一个算法时, 我们可以利用 NAND-RAM 提供的额外能力和便利. 当我们想要*分析*一个程序或证明一个*否定性结果*时, 我们可以将注意力局限于图灵机.




```admonish bigidea
{{idec}}{ide:polyvsnot} 

只要我们仅关注**多项式时间**与**指数时间**之间的区别, 所有 "合理的" 计算模型都是等价的.
```


上文中的形容词 **"合理的"** 指的是所有已实现的、可扩展的计算模型, 而 **量子计算机** 可能是唯一的例外. 参见 {{ref:ECTTsec}} 和 [第23章](chapter_23.md).


```admonish proof collapsible=true title="{{ref:thm:polyRAMTM}}的证明思路"
证明 $TIME_{\mathsf{TM}}(T(n)) \subseteq TIME_{\mathsf{RAM}}(10 \cdot T(n))$ 这一方向并不困难, 因为 NAND-RAM 程序 $P$ 可以通过在数组中存储图灵机 $M$ 的状态转移表（如 [定理9.1](./chapter_9.md#thm:universaltmthm) 的证明中所做的那样）, 以常数级的开销模拟 $M$. 模拟图灵机的每一步都可以在常数 $c$ 步 RAM 操作内完成, 且可以证明这个常数 $c$ 小于 $10$.

因此, 该定理的核心在于证明 $TIME_{\mathsf{RAM}}(T(n)) \subseteq TIME_{\mathsf{TM}}(T(n)^4)$. 这一证明紧随 [定理8.1](./chapter_8.md#thm:RAMTMequivalencethm) 的证明思路, 在那里我们已经证明了任何由 NAND-RAM 程序 $P$ 可计算的函数 $F$, 同样可以由图灵机（或等价的 NAND-TM 程序）$M$ 来计算. 为了证明 {{ref:thm:polyRAMTM}}, 我们沿用完全相同的证明过程, 只需核实 $M$ 模拟 $P$ 的开销是多项式级别的即可.

该证明包含许多细节, 但并不深奥. 因此, 相比于证明过程, 理解该定理的 **陈述** 要重要得多.
```

```admonish proof collapsible=true title="{{ref:thm:polyRAMTM}}的证明"
我们仅关注非平凡方向的 $TIME_{\mathsf{RAM}}(T(n)) \subseteq TIME_{\mathsf{TM}}(T(n)^4)$.
令 $F\in TIME_{\mathsf{RAM}}(T(n))$.
$F$ 可由某个 NAND-RAM 程序 $P$ 在 $T(n)$ 的时间内计算, 且我们需要证明它同样可以被一个图灵机 $M$ 在 $T(n)^4$ 的时间内被计算.
这等价于证明 $F$ 可以被一个 NAND-TM 程序在 $T(n)^4$ 时间内被计算, 因为对于任意 NAND-TM 程序 $Q$, 都存在一台模拟它的图灵机 $M$, 使得 $Q$ 的每一次迭代都对应 $M$ 的一个单步操作.


如前文所述, 我们沿用 [定理8.1](./chapter_8.md#thm:RAMTMequivalencethm) 的证明方法 (使用 NAND-TM 程序模拟 NAND-RAM 程序), 并且使用一样的模拟方法, 但更仔细地核算每步模拟所需要消耗的步数.
回想一下, NAND-RAM 的模拟是通过"剥离"其特性, 直到只剩下 NAND-TM 为止.

我们不会提供所有证明的细节, 但将展示证明 NAND-RAM 的每个特性都能以至多多项式开销被 NAND-TM 模拟的核心思路:

1. 回想一下, 每个 NAND-RAM 变量或数组元素包含的整数范围在 0 到 T 之间, 其中 T 是目前已经执行的行数.
因此, 如果 P 是一个在 $T(n)$ 时间内计算 $F$ 的 NAND-RAM 程序, 那么在长度为 $n$ 的输入下, P 所使用的所有整数的大小至多为 $T(n)$.
这意味着索引 `i` 能到达的最大值至多是 $T(n)$, 因此 $P$ 的每个变量都可以看作是一个拥有至多 $T(n)$ 个索引的数组, 每个索引存放一个大小至多为 $T(n)$ 的自然数.
令 $\ell = \ceil{\log T(n)}$ 为编码此类数字所需要的对比特数 (我们可以在模拟开始时先计算出 $T(n)$ 和 $\ell$).
2. 我们可以将一个长度 $\leq T(n)$, 包含范围在 $\{0,\ldots, T(n)-1 \}$ 内数字的 NAND-RAM 数组, 编码为一个包含 $T(n)\ell =O(T(n)\log T(n))$ 个比特的布尔 (即 NAND-TM) 数组.
我们也可以像 [定理8.1](./chapter_8.md#thm:RAMTMequivalencethm) 的证明那样, 将其视为一个*二维数组*.
一个包含数字的 NAND-RAM 标量则简单地编码为一个长度为 $\ell$ 的较短 NAND-TM 数组.
3. 我们可以使用长度为 $T(n)\ell = O(T(n) \log T(n))$ 的一维数组来模拟二维数组.
所有关于整数的算术运算都是用"小学数学算法", 其耗时是整数比特数 $\ell$ 的多项式级别的, 在本例中即为 $poly(\log T(n))$.
因此, 我们可以用一个使用随机访问内存但仅有*布尔值*的一维数组, 在 $O(T(n)poly(\log T(n)))$ 步内模拟 $T(n)$ 步的 NAND-RAM 模型.
4. 最昂贵的步骤是将随机访问内存转化为 NAND-TM/图灵机 的顺序内存模型.
正如我们在 [定理8.1](./chapter_8.md#thm:RAMTMequivalencethm) 证明中所做的, 我们可以通过以下步骤模拟访问数组 `Foo` 中由数组 `Bar` 编码的某个位置:

   1. 将 `Bar` 复制到某个临时数组 `Temp`
   2. 维护一个数组 `Index`, 其初始除第一位为 $1$ 外其余位置为 $0$.
   3. 重复以下步骤直到 `Temp` 编码了数字 $0$: _(最多重复 $T(n)$ 次)_
	  - 将 `Temp` 编码的数值减 $1$. _(消耗步数为 $\ell = \ceil{\log T(n)}$ 的多项式级)_
	  - 减小 `i` 直到其等于 $0$. _(消耗 $O(T(n))$ 步)_
	  - 扫描 `Index` 直到直到值为 $1$ 的位置, 将其改成 $0$, 向后移动一步并写下 $1$. _(消耗 $O(T(n))$ 步)_
   4. 完成后, 如果我们扫描 `Index` 直到找到 `Index[i]`$=1$ 的点, 那么 `i` 就包含了原先由 `Bar` 编码的值. _(消耗 $O(T(n))$ 步)_

   每次此类操作的总代价为 $O(T(n)^2 + T(n)poly(\log T(n))) = O(T(n)^2)$ 步.

综上所述，我们使用 $O(T(n)^2 poly(\log T(n)))$ 步 NAND-TM 来模拟 NAND-RAM 的**单步**操作，因此总模拟时间为 $O(T(n)^3 poly(\log T(n)))$。对于足够大的 $n$，这个值小于 $T(n)^4$.
```



```admonish info
{{remc}}{rem:nicefunctions}[好的时间界限]

在讨论一般的时间界限时, 我们需要确保排除掉一些"病态"的情况, 比如函数 $T$ 没有给算法留足够读取输入的时间, 或者时间界限函数本身就是不可计算的.

我们称函数 $T:\N \rightarrow \N$ 是一个 *好的时间界限函数* (或简称为好函数), 如果它满足以下条件:
+ 对于任意 $n\in \N$, 都有 $T(n) \geq n$ (即 $T$ 预留了足够的读入时间)
+ 对于任意 $n' \geq n$, 都有 $T(n') \geq T(n)$ (即 $T$ 允许在更长的输入上花费更长的时间)
+ 映射 $F(x) = 1^{T(|x|)}$ (即把长度为 $n$ 的字符串映射为长度为 $T(n)$ 的全 $1$ 序列) 可以被一个 NAND-RAM 程序在 $O(T(n))$ 时间内计算出来.

我们在应用中遇到所有"正常的"时间复杂度界限, 如 $T(n)= 100 n$, $T(n) =  n^2 \log n$,$T(n) = 2^{\sqrt{n}}$ 等, 都是好的.
因此, 从现在起, 我们只关心当 $T(n)$ 是"好函数"时的复杂性类 $TIME(T(n))$.
可计算性的条件一般是很容易被满足的.
比如, 对于像 $T(n) = n^3$ 这样的代数函数, 我们可以在关于 $T(n)$ 的比特数的多项式时间内 (即关于 $T(n)$ 的对数多项式级) 计算出 $T(n)$ 的二进制表示.
因此, 在这种情况下, 写出字符串 $1^{T(n)}$ 的时间将会是 $T(n) + poly(\log T(n)) = O(T(n))$.
```


## 13.3 扩展Church-Turing论题 (讨论) { #ECTTsec }

{{ref:thm:polyRAMTM}} 表明, 图灵机和 RAM 机/ NAND-RAM 程序这几个计算模型在运行时间上是多项式等价的.
其他多项式等价模型的例子有:

* 所有标准的编程语言, 包括 C/Python/JavaScript/Lisp/等.
* $\lambda$ 算子 (参见 {{ref:bibnotesrunningtime}})
* 元胞自动机
* 并行计算机
* 生物计算设备, 如基于 DNA 的计算机.

*扩展Church-Turing论题* 指出, 这一表述对于所有物理上可实现的计算模型均成立.
换言之, 扩展Church-Turing论题认为, 对于任意一个可以*扩展的计算设备* $C$ (该设备具有有限的描述, 但原则上可以用于处理任意长度的输入), 都存在某个常数 $a$, 使得对于 $C$ 在长度为 $n$ 的输入上使用 $S(n)$ 量的物理资源所能计算的每一个函数 $F:\{0,1\}^* \rightarrow \{0,1\}$, $F$ 都属于 $TIME(S(n)^a)$.
这是对一般的Church-Turing论题 (在 [第 8.8 节](./chapter_8.md#churchturingdiscussionsec) 中被讨论) 的加强.
普通论题仅指出所有物理上可实现模型的"可计算函数集"是相同的, 但不要求不同模型之间模拟的开销至多为多项式级别.

目前所有关于可扩展计算模型和编程语言的构建都遵循扩展Church-Turing论题, 即它们都可以被图灵机 (以及 NAND-TM 或 NAND-RAM 程序) 以多项式级开销进行模拟.
因此, $\mathbf{P}$ 和 $\mathbf{EXP}$ 类对于模型的选择具有鲁棒性.
我们可以使用任何我们喜欢的编程语言, 或者算法的高层描述, 来确定一个问题是否属于 $\mathbf{P}$.

与Church-Turing论题本身一样, 扩展Church-Turing论题也处于渐近设定之下, 并不直接产生可实验验证的预测.
然而, 它可以用更具体的开销界限来实例化, 从而产生可实验验证的预测, 例如我们在 {{ref:PECTTsec}} 中提到的*物理扩展Church-Turing论题*.

在过去一百多年对计算的研究和机械化进程中, 尚未有人制造出能违反扩展Church-Turing论题的可扩展计算设备.
然而, *量子计算* (如果得以实现) 将对扩展丘奇-图灵论题提出严峻挑战 (见 [第23章](./chapter_23.md)).
但是, 即便量子计算的愿景完全实现, 扩展Church-Turing论题在"精神层面"上依然是正确的: 
虽然我们需要修正该论题以纳入量子计算的可能性, 但其宏观框架保持不变.
我们依然能够对计算进行数学建模; 依然可以将程序视为字符串并拥有通用程序; 依然拥有时间层级和不可计算性结果; 并且依然没有理由怀疑 ("普通") Church-Turing论题.
此外, 量子计算的前景似乎并不会改变我们所关心的许多 (虽非全部!) 具体问题的运行时间复杂度.
特别是, 就我们目前所知, 在 [第12章](chapter_12.md) 提到的所有示例问题中, 只有*整数分解*这一个问题的复杂度, 会因为将模型修改为包含量子计算机而受到影响.






## Efficient universal machine: a NAND-RAM interpreter in NAND-RAM

We have seen in [universaltmthm](){.ref} the "universal Turing machine".
Examining that proof, and combining it with  [polyRAMTM-thm](){.ref} , we can see that the program $U$ has a _polynomial_ overhead, in the sense that it can simulate $T$ steps of a given NAND-TM (or NAND-RAM) program $P$ on an input $x$ in $O(T^4)$ steps.
But in fact, by directly simulating NAND-RAM programs we can do better with only a _constant_ multiplicative overhead.
That is, there is a _universal NAND-RAM program_ $U$ such that for every NAND-RAM program $P$, $U$ simulates $T$ steps of $P$ using only $O(T)$ steps. (The implicit constant in the $O$ notation can depend on the program $P$ but does _not_ depend on the length of the input.)


::: {.theorem title="Efficient universality of NAND-RAM" #univ-nandpp}
There exists a NAND-RAM program $U$ satisfying the following:

1. _($U$ is a universal NAND-RAM program.)_ For every NAND-RAM program $P$ and input $x$,  $U(P,x)=P(x)$ where by $U(P,x)$ we denote the output of $U$ on a string encoding the pair $(P,x)$.

2. _($U$ is efficient.)_ There are some constants $a,b$ such that for every NAND-RAM program $P$, if $P$ halts on input $x$ after at most $T$ steps, then $U(P,x)$ halts after at most $C\cdot T$ steps where $C \leq a |P|^b$. 
:::


> ### { .pause }
As in the case of [polyRAMTM-thm](){.ref}, the proof of [univ-nandpp](){.ref} is not very deep and so it is more important to understand its _statement_.
Specifically, if you understand how you would go about writing an interpreter for NAND-RAM using a modern programming language such as Python, then you know everything you need to know about the proof of this theorem.


![The universal NAND-RAM program $U$ simulates an input NAND-RAM program $P$ by storing all of $P$'s variables inside a single array `Vars` of $U$. If $P$ has $t$ variables, then the array `Vars` is divided into blocks of length $t$, where the $j$-th coordinate of the $i$-th block contains the $i$-th element of the $j$-th array of $P$. If the $j$-th variable of $P$ is scalar, then we just store its value in the zeroth block of `Vars`.](../figure/universalrammachine.png){#universalrammachinefig .margin}

::: {.proof data-ref="univ-nandpp"}
To present a universal NAND-RAM program in full we would need to describe a precise representation scheme, as well as the full NAND-RAM instructions for the program.
While this can be done, it is more important to focus on the main ideas, and so we just sketch the proof here.
A specification of NAND-RAM is given in the [appendix](http://tiny.cc/introtcsappendix), and for the purposes of this simulation, we can simply use the representation of the NAND-RAM code as an ASCII string.

The program $U$ gets as input a NAND-RAM program $P$ and an input $x$ and simulates $P$ one step at a time.
To do so, $U$ does the following:

1. $U$ maintains variables  `program_counter`, and `number_steps` for the current line to be executed and the number of steps executed so far.

2. $U$ initially scans the code of $P$ to find the number $t$ of unique variable names that $P$ uses. It will translate each variable name into a number between $0$ and $t-1$ and use an array `Program` to store $P$'s code where  for every line $\ell$, `Program[`$\ell$`]` will store the $\ell$-th line of $P$ where the variable names have been translated to numbers. (More concretely, we will use a constant number of arrays to separately encode the operation used in this line, and the variable names and indices of the operands.)


3. $U$ maintains a single array `Vars` that contains all the values of $P$'s variables. We divide `Vars` into blocks of length $t$. If $s$ is a number corresponding to an array variable `Foo` of $P$, then we store `Foo[0]` in `Vars[`$s$`]`, we store `Foo[1]` in `Var_values[`$t+s$`]`, `Foo[2]` in `Vars[`$2t + s$`]` and so on and so forth (see [universalrammachinefig](){.ref}). Generally, if the $s$-th variable of $P$ is a scalar variable, then its value will be stored in location `Vars[`$s$`]`.
If it is an array variable then the value of its $i$-th element will be stored in location `Vars[`$t\cdot i + s$`]`.

4. To simulate a single step of $P$, the program $U$ recovers from `Program` the line corresponding to `program_counter`  and executes it. Since NAND-RAM has a constant number of arithmetic operations, we can implement the logic of which operation to execute using a sequence of a constant number of  if-then-else's.  Retrieving from `Vars` the values of the operands of each instruction can be done using a constant number of arithmetic operations. 

The setup stages take only a constant (depending on $|P|$ but not on the input $x$) number of steps.
Once we are done with the setup, to simulate a single step of $P$, we just need to retrieve the corresponding line and do a constant number of "if elses" and accesses to `Vars` to simulate it.
Hence the total running time to simulate $T$ steps of the program $P$ is at most $O(T)$ when suppressing constants that depend on the program $P$.
:::

### Timed Universal Turing Machine

One corollary of the efficient universal machine is the following.
Given any Turing machine $M$, input $x$, and "step budget" $T$, we can simulate the execution of $M$ for $T$ steps in time that is polynomial in $T$.
Formally, we define a function $TIMEDEVAL$ that takes the three parameters $M$, $x$, and the time budget, and outputs $M(x)$ if $M$ halts within at most $T$ steps, and outputs $0$ otherwise.
The timed universal Turing machine computes $TIMEDEVAL$ in polynomial time (see [timeduniversaltmfig](){.ref}). 
(Since we measure time as a function of the input length, we define $TIMEDEVAL$ as taking the input $T$ represented in _unary_: a string of $T$ ones.)

> ### {.theorem title="Timed Universal Turing Machine" #timeduniversalTM}
Let $TIMEDEVAL:\{0,1\}^* \rightarrow \{0,1\}^*$ be the function defined as 
$$TIMEDEVAL(M,x,1^T) = \begin{cases} M(x) & M \text{ halts within $\leq T$ steps on $x$} \\ 0 & \text{otherwise}\end{cases} \;.$$
Then $TIMEDEVAL \in \mathbf{P}$.

![The _timed_ universal Turing machine takes as input a Turing machine $M$, an input $x$, and a time bound $T$, and outputs $M(x)$ if $M$ halts within at most $T$ steps. [timeduniversalTM](){.ref} states that there is such a machine that runs in time polynomial in $T$. ](../figure/timeduniversaltm.png){#timeduniversaltmfig  .margin }


::: {.proof #proofoftimeduniversalTM data-ref="timeduniversalTM"}
We only sketch the proof since the  result follows fairly directly from [polyRAMTM-thm](){.ref} and [univ-nandpp](){.ref}. By  [polyRAMTM-thm](){.ref}  to show that $TIMEDEVAL \in \mathbf{P}$, it suffices to give a polynomial-time NAND-RAM program to compute $TIMEDEVAL$.

Such a program can be obtained as follows. Given a Turing machine $M$, by [polyRAMTM-thm](){.ref} we can transform it in time polynomial in its description into a functionally-equivalent NAND-RAM program $P$ such that the execution of $M$ on $T$ steps can be simulated by the execution of $P$ on $c\cdot T$ steps.
We can then run the universal NAND-RAM machine of [univ-nandpp](){.ref} to simulate $P$ for $c\cdot T$ steps, using $O(T)$ time, and output $0$ if the execution did not halt within this budget.
This shows that $TIMEDEVAL$ can be computed by a NAND-RAM program in time polynomial in $|M|$ and linear in $T$, which means $TIMEDEVAL \in \mathbf{P}$.
:::


## The time hierarchy theorem

Some functions are _uncomputable_,  but are there functions that can be computed, but only at an exorbitant cost?
For example, is there a function that _can_ be computed in time $2^n$, but _can not_ be computed in time $2^{0.9 n}$?
It turns out that the answer is __Yes__:

> ### {.theorem title="Time Hierarchy Theorem" #time-hierarchy-thm}
For every nice function $T:\N \rightarrow \N$, there is a function $F:\{0,1\}^* \rightarrow \{0,1\}$
in $TIME(T(n)\log n) \setminus TIME(T(n))$.

There is nothing special about $\log n$, and we could have used any other efficiently computable function that tends to infinity with $n$.

::: { .bigidea #timehierarchy}
If we have more time, we can compute more functions.
:::


::: {.remark title="Simpler corollary of the time hierarchy theorem" #hierarchytoyrem}
The generality of the time hierarchy theorem can make its proof a little hard to read. 
It might be easier to follow the proof if you first try to prove by yourself the easier statement $\mathbf{P} \subsetneq \mathbf{EXP}$.

You can do so by showing that the following function $F:\{0,1\}^* :\rightarrow \{0,1\}$ is in $\mathbf{EXP} \setminus \mathbf{P}$: for every Turing machine $M$ and input $x$, $F(M,x)=1$ if and only if $M$ halts on $x$ within at most $|x|^{\log |x|}$ steps.
One can show that $F \in TIME(n^{O(\log n)}) \subseteq \mathbf{EXP}$ using the universal Turing machine (or the efficient universal NAND-RAM program of [univ-nandpp](){.ref}). On the other hand, we can use similar ideas to those used to show the uncomputability of $HALT$ in [haltalternativesec](){.ref} to prove that $F \not\in \mathbf{P}$.
:::




![The _Time Hierarchy Theorem_ ([time-hierarchy-thm](){.ref}) states that all of these classes are _distinct_.](../figure/timehierarchythm.png){#timehierarchythmfig}


> ### {.proofidea data-ref="time-hierarchy-thm"}
In the proof of [halt-thm](){.ref} (the uncomputability of the Halting problem), we have shown that the function $HALT$ cannot be computed in any finite time. An examination of the proof shows that it gives something stronger.
Namely, the proof shows that if we fix our computational budget to be $T$ steps, then not only can we not distinguish between programs that halt and those that do not, but we cannot even distinguish between programs that halt within at most $T'$ steps and those that take more than that (where $T'$ is some number depending on $T$).
Therefore, the proof of [time-hierarchy-thm](){.ref} follows the ideas of the uncomputability of the halting problem, but again with a more careful accounting of the running time.



::: {.proof data-ref="time-hierarchy-thm"}
Our proof is inspired by the proof of the uncomputability of the halting problem.
Specifically, for every function $T$ as in the theorem's statement, we define the _Bounded Halting_ function $HALT_T$ as follows.
The input to $HALT_T$ is a pair $(P,x)$ such that $|P| \leq \log \log |x|$ and $P$ encodes some NAND-RAM program.
We define

$$
HALT_T(P,x) = \begin{cases}1, & P \text{ halts on } x \text{ within } \leq 100\cdot T(|P|+|x|) \text{ steps} \\
0, & \text{otherwise} \end{cases} \;.
$$
(The constant $100$ and the function $\log \log n$ are rather arbitrary, and are chosen for convenience in this proof.)

[time-hierarchy-thm](){.ref} is an immediate consequence of the following two claims:

__Claim 1:__ $HALT_T \in TIME(T(n)\cdot \log n)$

and

__Claim 2:__ $HALT_T \not\in TIME(T(n))$.

Please make sure you understand why indeed the theorem follows directly from the combination of these two claims. We now turn to proving them.

__Proof of claim 1:__ We can easily check in linear time whether an input has the form $P,x$ where $|P| \leq \log\log |x|$.
Since $T(\cdot)$ is a nice function, we can evaluate it in $O(T(n))$ time. Thus, we can compute $HALT_T(P,x)$ as follows:

1. Compute $T_0=T(|P|+|x|)$ in $O(T_0)$ steps.

2. Use the universal NAND-RAM program of [univ-nandpp](){.ref} to simulate $100\cdot T_0$ steps of $P$ on the input $x$ using at most  $poly(|P|)T_0$ steps. (Recall that we use $poly(\ell)$ to denote a quantity that is bounded by $a\ell^b$ for some constants $a,b$.)

3. If $P$ halts within these $100\cdot T_0$ steps then output $1$, else output $0$. 

The length of the input is $n=|P|+|x|$.
Since $|x| \leq n$ and $(\log \log |x|)^b = o(\log |x|)$ for every $b$, the running time will be $o(T(|P|+|x|) \log n)$ and hence the above algorithm demonstrates that $HALT_T \in TIME(T(n)\cdot \log n)$, completing the proof of Claim 1. 


__Proof of claim 2:__ This proof is the heart of [time-hierarchy-thm](){.ref}, and is very reminiscent of the proof that $HALT$ is not computable.
Assume, for the sake of contradiction, that there is some NAND-RAM program $P^*$ that computes $HALT_T(P,x)$ within $T(|P|+|x|)$ steps. We are going to show a contradiction by creating a program $Q$ and showing that under our assumptions, if $Q$ runs for less than $T(n)$ steps when given (a padded version of)  its own code as input then it actually runs for more than $T(n)$ steps and vice versa. (It is worth re-reading the last sentence twice or thrice to make sure you understand this logic. It is very similar to the direct proof of the uncomputability of the halting problem where we obtained a contradiction by using an assumed "halting solver" to construct a program that, given its own code as input, halts if and only if it does not halt.)


We will define $Q^*$ to be the program that on input a string $z$   does the following:

1. If $z$ does not have the form $z=P1^m$ where $P$ represents a NAND-RAM program and $|P|< 0.1 \log\log m$ then return $0$. (Recall that $1^m$ denotes the string of $m$ ones.) 

2. Compute $b= P^*(P,z)$ (at a cost of at most $T(|P|+|z|)$ steps, under our assumptions).

3. If $b=1$ then $Q^*$ goes into an infinite loop, otherwise it halts.

Let $\ell$ be the length description of $Q^*$ as a string, and let $m$ be larger than $2^{2^{1000 \ell}}$.
We will reach a contradiction by splitting into cases according to whether or not $HALT_T(Q^*,Q^*1^m)$ equals $0$ or $1$.


On the one hand, if $HALT_T(Q^*,Q^*1^m)=1$, then under our assumption that $P^*$ computes $HALT_T$, $Q^*$ will go into an infinite loop on input $z=Q^*1^m$, and hence in particular $Q^*$ does _not_ halt within $100 T(|Q^*|+m)$ steps on the input $z$. But this contradicts our assumption that $HALT_T(Q^*,Q^*1^m)=1$.

This means that it must hold that $HALT_T(Q^*,Q^*1^m)=0$. But in this case, since we assume $P^*$ computes $HALT_T$, $Q^*$ does not do anything in phase 3 of its computation, and so the only computation costs come in phases 1 and 2 of the computation.
It is not hard to verify that Phase 1 can be done in linear and in fact less than $5|z|$ steps.
Phase 2 involves executing $P^*$, which under our assumption requires $T(|Q^*|+m)$ steps.
In total we can perform both phases in less than $10 T(|Q^*|+m)$ in steps, which by definition means that $HALT_T(Q^*,Q^*1^m)=1$, but this is of course a contradiction. This completes the proof of Claim 2 and hence of [time-hierarchy-thm](){.ref}.
:::

::: {.solvedexercise title="$\mathbf{P}$ vs $\mathbf{EXP}$" #PvsEXPexercise}
Prove that $\mathbf{P} \subsetneq \mathbf{EXP}$.
:::

::: {.solution data-ref="PvsEXP"}
This statement follows directly from the time hierarchy theorem, but it can be an instructive exercise to prove it directly, see [hierarchytoyrem](){.ref}.
We need to show that there exists $F \in \mathbf{EXP} \setminus \mathbf{P}$.
Let $T(n) = n^{\log n}$ and $T'(n) = n^{\log n / 2}$.
Both are nice functions.
Since $T(n)/T'(n) = \omega(\log n)$, by [time-hierarchy-thm](){.ref} there exists some $F$ in $TIME(T(n)) \setminus TIME(T'(n))$.
Since for sufficiently large $n$, $2^n > n^{\log n}$,  $F \in TIME(2^n) \subseteq \mathbf{EXP}$.
On the other hand, $F \not\in \mathbf{P}$. Indeed, suppose otherwise that there was a constant $c>0$ and a  Turing machine computing $F$ on $n$-length input in at most $n^c$ steps for all sufficiently large $n$. Then since for $n$ large enough $n^c < n^{\log n/2}$, it would have followed that $F \in TIME(n^{\log n /2})$ contradicting our choice of $F$.
:::



The time hierarchy theorem tells us that there are functions we can compute in $O(n^2)$ time but not $O(n)$, in $2^n$ time, but not $2^{\sqrt{n}}$, etc..
In particular there are most definitely functions that we can compute in time $2^n$ but not $O(n)$.
We have seen that we have no shortage of natural functions for which the best _known_ algorithm requires roughly $2^n$ time, and that many people have invested significant effort in trying to improve that.
However,  unlike in the finite vs. infinite case, for all of the examples above at the moment we do not know how to rule out even an $O(n)$ time algorithm.
We will however see that there is a single unproven conjecture that would imply such a result for most of these problems.

![Some complexity classes and some of the functions we know (or conjecture) to be contained in them.](../figure/time_complexity_map.png){#complexityclassinclusionfig .margin  }



The time hierarchy theorem relies on the existence of an efficient universal NAND-RAM program, as proven in [univ-nandpp](){.ref}.
For other models such as Turing machines we have similar time hierarchy results showing that there are functions computable in time $T(n)$ and not in time $T(n)/f(n)$ where $f(n)$ corresponds to the overhead in the corresponding universal machine. 


## Non-uniform computation {#nonuniformcompsec }



We have now seen two measures of "computation cost" for functions.
In [secdefinesizeclasses](){.ref} we defined the complexity of computing _finite_ functions using circuits / straightline programs.
Specifically,  for a finite function $g:\{0,1\}^n \rightarrow \{0,1\}$ and number $s\in \N$,  $g\in SIZE_n(s)$ if there is a circuit of at most $s$ NAND gates (or equivalently  an $s$-line NAND-CIRC program) that computes $g$.
To relate this to the classes $TIME(T(n))$ defined in this chapter we first need to extend the class $SIZE_n(s)$ from finite functions to functions with unbounded input length.

::: {.definition title="Non-uniform computation" #nonuniformdef}
Let $F:\{0,1\}^* \rightarrow \{0,1\}$ and $T:\N \rightarrow \N$ be a nice time bound.
For every $n\in \N$, define $F_{\upharpoonright n} : \{0,1\}^n \rightarrow \{0,1\}$ to be the _restriction_ of $F$ to inputs of size $n$. That is, $F_{\upharpoonright n}$ is the function mapping $\{0,1\}^n$ to $\{0,1\}$ such that for every $x\in \{0,1\}^n$, $F_{\upharpoonright n}(x)=F(x)$.

We say that $F$ is _non-uniformly computable in at most $T(n)$ size_, denoted by $F \in SIZE(T)$ if there exists a sequence $(C_0,C_1,C_2,\ldots)$ of NAND circuits such that:

* For every $n\in \N$, $C_n$ computes the function $F_{\upharpoonright n}$

* For every sufficiently large $n$, $C_n$ has at most $T(n)$ gates.
:::

In other words, $F \in SIZE(T)$ iff for every $n \in \N$, it holds that $F_{\upharpoonright n} \in SIZE_n(T(n))$.
The non-uniform analog to the class $\mathbf{P}$ is the class $\mathbf{P_{/poly}}$ defined as 

$$
\mathbf{P_{/poly}} = \cup_{c\in \N} SIZE(n^c)  \; . \label{eqppolydef}
$$
There is a big difference between non-uniform computation and uniform complexity classes such as $TIME(T(n))$ or $\mathbf{P}$.
The condition $F\in \mathbf{P}$ means that there is a _single_ Turing machine $M$ that computes $F$ on all inputs in polynomial time.
The condition $F\in \mathbf{P_{/poly}}$ only means that for every input length $n$ there can be a _different_ circuit $C_n$ that computes $F$ using polynomially many gates on inputs of these lengths.
As we will see, $F\in \mathbf{P_{/poly}}$ does not necessarily imply that $F\in \mathbf{P}$.
However, the other direction is true:

![We can think of an infinite function $F:\{0,1\}^* \rightarrow \{0,1\}$ as a collection of finite functions $F_0,F_1,F_2,\ldots$ where $F_{\upharpoonright n}:\{0,1\}^n \rightarrow \{0,1\}$ is the restriction of $F$ to inputs of length $n$. We say $F$ is in $\mathbf{P_{/poly}}$ if for every $n$, the function $F_{\upharpoonright n}$  is computable by a polynomial-size NAND-CIRC program, or equivalently, a polynomial-sized Boolean circuit.](../figure/Ppoly.png){#Ppolyfig .margin  }


::: {.theorem title="Non-uniform computation contains uniform computation" #non-uniform-thm}
There is some $a\in \N$ s.t. for every nice $T:\N \rightarrow \N$ and  $F:\{0,1\}^* \rightarrow \{0,1\}$,
$$TIME(T(n)) \subseteq SIZE(T(n)^a)\;.$$
:::

In particular,  [non-uniform-thm](){.ref} shows that for every $c$, $TIME(n^c) \subseteq SIZE(n^{ca})$ and hence $\mathbf{P} \subseteq \mathbf{P_{/poly}}$.



::: {.proofidea data-ref="non-uniform-thm"}
The idea behind the proof is to "unroll the loop".
Specifically, we will use the programming language variants of non-uniform and uniform computation: namely NAND-CIRC and NAND-TM. 
The main difference between the two is that NAND-TM has _loops_. However, for every fixed $n$, if we know that a NAND-TM program runs in at most $T(n)$ steps, then we can replace its loop by simply "copying and pasting" its code $T(n)$ times, similar to how in Python we can replace code such as

```python
for i in range(4):
	print(i)
```

with the "loop free" code

```python
print(0)
print(1)
print(2)
print(3)
```

To make this idea into an actual proof we need to tackle one technical difficulty, and this is to ensure that the NAND-TM program is _oblivious_ in the sense that the value of the index variable `i`  in the $j$-th iteration of the loop will depend only on $j$ and not on the contents of the input. We make a digression to do just that in [obliviousnandtm](){.ref} and then complete the proof  of [non-uniform-thm](){.ref}.
:::


### Oblivious NAND-TM programs  {#obliviousnandtm }

Our approach for proving [non-uniform-thm](){.ref} involves "unrolling the loop". 
For example,  consider the following NAND-TM to compute the $XOR$ function on inputs of arbitrary length:

```python
temp_0 = NAND(X[0],X[0])
Y_nonblank[0] = NAND(X[0],temp_0)
temp_2 = NAND(X[i],Y[0])
temp_3 = NAND(X[i],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
MODANDJUMP(X_nonblank[i],X_nonblank[i])
```

Setting (as an example) $n=3$, we can attempt to translate this NAND-TM program into a NAND-CIRC program for computing  $XOR_3:\{0,1\}^3 \rightarrow \{0,1\}$ by simply "copying and pasting" the loop three times (dropping the `MODANDJMP` line):


```python
temp_0 = NAND(X[0],X[0])
Y_nonblank[0] = NAND(X[0],temp_0)
temp_2 = NAND(X[i],Y[0])
temp_3 = NAND(X[i],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
temp_0 = NAND(X[0],X[0])
Y_nonblank[0] = NAND(X[0],temp_0)
temp_2 = NAND(X[i],Y[0])
temp_3 = NAND(X[i],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
temp_0 = NAND(X[0],X[0])
Y_nonblank[0] = NAND(X[0],temp_0)
temp_2 = NAND(X[i],Y[0])
temp_3 = NAND(X[i],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
```

However, the above is still not a valid NAND-CIRC program since it contains references to the special variable `i`.
To make it into a valid NAND-CIRC program, we replace references to `i` in the first iteration with $0$, references in the second iteration with $1$, and references in the third iteration with $2$.
(We also create a variable `zero` and use it for the first time any variable is instantiated, as well as remove assignments to non-output variables that are never used later on.)
The resulting program is a standard "loop free and index free" NAND-CIRC program that computes $XOR_3$ (see also [unrolledcircuitfig](){.ref}):

```python
temp_0 = NAND(X[0],X[0])
one = NAND(X[0],temp_0)
zero = NAND(one,one)
temp_2 = NAND(X[0],zero)
temp_3 = NAND(X[0],temp_2)
temp_4 = NAND(zero,temp_2)
Y[0] = NAND(temp_3,temp_4)
temp_2 = NAND(X[1],Y[0])
temp_3 = NAND(X[1],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
temp_2 = NAND(X[2],Y[0])
temp_3 = NAND(X[2],temp_2)
temp_4 = NAND(Y[0],temp_2)
Y[0] = NAND(temp_3,temp_4)
```

![A NAND circuit for $XOR_3$ obtained by "unrolling the loop" of the NAND-TM program for computing $XOR$ three times.](../figure/unrolled_circuit.png){#unrolledcircuitfig .margin }



Key to this transformation was the fact that in our original NAND-TM program for $XOR$, regardless of whether the input is $011$, $100$, or any other string, the index variable `i` is guaranteed to equal $0$ in the first iteration, $1$ in the second iteration,  $2$ in the third iteration, and so on and so forth.
The particular sequence $0,1,2,\ldots$ is immaterial: the crucial property is that the NAND-TM program for $XOR$ is  _oblivious_ in the sense that the value of the index `i` in the $j$-th iteration depends only on $j$ and does not depend on the particular choice of the input. 
Luckily, it is possible to transform every NAND-TM program into a functionally equivalent oblivious program with at most quadratic overhead. (Similarly we can transform any Turing machine into a functionally equivalent oblivious Turing machine, see [oblivious-ex](){.ref}.)

> ### {.theorem title="Making NAND-TM oblivious" #obliviousnandtmthm}
Let $T:\N \rightarrow \N$ be a nice function and let $F\in TIME_{\mathsf{TM}}(T(n))$.
Then there is a NAND-TM program  $P$ that computes $F$ in $O(T(n)^2)$ steps and satisfying the following.
For every $n\in \N$ there is  a sequence $i_0,i_1,\ldots, i_{m-1}$ such that for every $x\in \{0,1\}^n$, if $P$ is executed on input $x$ then in the  $j$-th iteration  the variable `i` is equal to $i_j$.

In other words, [obliviousnandtmthm](){.ref} implies that if we can compute $F$ in $T(n)$ steps, then we can compute it in $O(T(n)^2)$ steps with a program $P$ in which the position of `i` in the $j$-th iteration depends only on $j$ and the length of the input, and not on the contents of the input.
Such a program can be easily translated into a NAND-CIRC program of $O(T(n)^2)$ lines by "unrolling the loop".

> ### {.proofidea data-ref="obliviousnandtmthm"}
We can translate any NAND-TM program $P'$ into an oblivious program $P$ by making $P$ "sweep" its arrays. That is, the index `i` in $P$ will always move all the way from position $0$ to position $T(n)-1$ and back again.
We can then simulate the program $P'$ with at most $T(n)$ overhead: if $P'$ wants to move `i` left when we are in a rightward sweep then we simply wait the at most $2T(n)$ steps until the next time we are back in the same position while sweeping to the left. 


![We simulate a $T(n)$-time NAND-TM program $P'$ with an _oblivious_ NAND-TM program $P$ by adding special arrays `Atstart` and `Atend` to mark positions $0$ and $T-1$ respectively. The program $P$ will simply "sweep" its arrays from right to left and back again. If the original program $P'$ would have moved `i` in a different direction then we wait $O(T)$ steps until we reach the same point back again, and so $P$ runs in $O(T(n)^2)$ time.](../figure/obliviousnandtm.png){#obliviousnandtmfig  .margin }


::: {.proof data-ref="obliviousnandtmthm"}
Let $P'$ be a NAND-TM program computing $F$ in $T(n)$ steps.
We construct an oblivious NAND-TM program $P$ for computing $F$ as follows (see also [obliviousnandtmfig](){.ref}).

1. On input $x$, $P$ will compute $T=T(|x|)$ and set up arrays `Atstart` and `Atend` satisfying 
 `Atstart[`$0$`]`$=1$ and `Atstart[`$i$`]`$=0$ for $i>0$ and `Atend[`$T-1$`]`$=1$ and `Atend[`i`]`$=0$ for all $i \neq T-1$.  We can do this because $T$ is a nice function. Note that since this computation does not depend on $x$ but only on its length, it is oblivious. 

 2. $P$ will also have a special array `Marker` initialized to all zeroes.

2. The index variable of $P$ will change direction of movement to the right whenever `Atstart[i]`$=1$ and to the left whenever `Atend[i]`$=1$. 

3. The program $P$  simulates the execution of $P'$. However, if the `MODANDJMP` instruction in $P'$ attempts to move to the right when $P$ is moving left (or vice versa) then $P$ will set `Marker[i]` to $1$ and  enter into a special "waiting mode". In this mode $P$ will wait until the next time in which `Marker[i]`$=1$ (at the next sweep) at which points $P$ zeroes `Marker[i]` and continues with the simulation. In the worst case this will take $2T(n)$ steps (if $P$ has to go all the way from one end to the other and back again.)
   
4. We also modify $P$ to ensure it ends the computation after simulating exactly $T(n)$ steps of $P'$, adding "dummy steps" if $P'$ ends early.

We see that $P$ simulates  the execution of $P'$ with an overhead of $O(T(n))$ steps of $P$ per one step of $P'$, hence completing the proof.
:::


[obliviousnandtmthm](){.ref} implies [non-uniform-thm](){.ref}. Indeed, if $P$ is a $k$-line oblivious NAND-TM program computing $F$ in time $T(n)$ then for every $n$ we can obtain a NAND-CIRC program of $(k-1)\cdot T(n)$ lines by simply making $T(n)$ copies of $P$ (dropping the final `MODANDJMP` line).
In the $j$-th copy we replace all references of the form `Foo[i]`  to `foo_`$i_j$ where $i_j$ is the value of `i` in the $j$-th iteration.

### "Unrolling the loop": algorithmic transformation of Turing Machines to circuits  { #unrollloopsec  }

The proof of [non-uniform-thm](){.ref} is _algorithmic_, in the sense that the proof yields a polynomial-time algorithm that given a Turing machine $M$ and parameters $T$ and $n$, produces a circuit  of $O(T^2)$ gates that agrees with $M$ on all inputs $x\in \{0,1\}^n$ (as long as $M$ runs for less than $T$ steps these inputs.)
We record this fact in the following theorem, since it will be useful for us later on:

![The function $UNROLL$ takes as input a Turing machine $M$, an input length parameter $n$, a step budget parameter $T$, and outputs a circuit $C$ of size $poly(T)$ that takes $n$ bits of inputs and outputs $M(x)$ if $M$ halts on $x$ within at most $T$ steps.](../figure/unrollloop_alg.png){#unrollloopfig .margin }

::: {.theorem title="Turing-machine to circuit compiler" #nand-compiler}
There is algorithm $UNROLL$ such that for every Turing machine $M$ and numbers $n,T$, 
$UNROLL(M,1^T,1^n)$ runs for $poly(|M|,T,n)$ steps and outputs a NAND circuit $C$ with  $n$ inputs, $O(T^2)$ gates, and one output, such that 
$$
C(x) = \begin{cases}y & M \text{ halts in $\leq T$ steps and outputs $y$} \\ 0 & \text{otherwise} \end{cases}\;.
$$
:::



::: {.proof #proofofnandcompiler data-ref="nand-compiler"}
We only sketch the proof  since it follows by directly translating the proof of [non-uniform-thm](){.ref} into an algorithm together with the simulation of Turing machines by NAND-TM programs (see also [unrolldescriptionfig](){.ref}).
Specifically, $UNROLL$ does the following:

1. Transform the Turing machine $M$ into an equivalent NAND-TM program $P$. 

2. Transform the NAND-TM program $P$ into an equivalent oblivious program $P'$ following the proof of [obliviousnandtmthm](){.ref}. The program $P'$ takes $T' = O(T^2)$ steps to simulate $T$ steps of $P$.

3. "Unroll the loop" of $P'$ by obtaining a NAND-CIRC program of $O(T')$ lines (or equivalently a NAND circuit with $O(T^2)$ gates)  corresponding to the execution of $T'$ iterations of $P'$.
:::


![We can transform a Turing machine $M$, input length parameter $n$, and time bound $T$ into an $O(T^2)$-sized NAND circuit that agrees with $M$ on all inputs $x\in \{0,1\}^n$ on which $M$ halts in at most $T$ steps. The transformation is obtained by first using the equivalence of Turing machines and NAND-TM programs $P$, then turning $P$ into an equivalent _oblivious_ NAND-TM program $P'$ via [obliviousnandtmthm](){.ref}, then "unrolling" $O(T^2)$ iterations of the loop of $P'$ to obtain an $O(T^2)$ line  NAND-CIRC program  that agrees with $P'$ on length $n$ inputs, and finally translating this program into an equivalent circuit.](../figure/unrolldescription.png){#unrolldescriptionfig }

::: { .bigidea #unrollloop}
By "unrolling the loop" we can transform an algorithm that takes $T(n)$ steps to compute $F$ into a circuit that uses $poly(T(n))$ gates to compute the restriction of $F$ to $\{0,1\}^n$.
:::




::: { .pause }
Reviewing the transformations described in [unrolldescriptionfig](){.ref}, as well as solving the following two exercises is a great way to get more comfort with non-uniform complexity and in particular with $\mathbf{P_{/poly}}$ and its relation to $\mathbf{P}$.
:::


::: {.solvedexercise title="Alternative characterization of $\mathbf{P}$" #characterizationofp}
Prove that for every $F:\{0,1\}^* \rightarrow \{0,1\}$, $F\in \mathbf{P}$ if and only if there is a polynomial-time Turing machine $M$ such that  for every $n\in \N$, $M(1^n)$ outputs a description of an $n$ input circuit $C_n$ that computes the restriction $F_{\upharpoonright n}$ of $F$ to inputs in $\{0,1\}^n$.
:::

::: {.solution data-ref="characterizationofp"}
We start with the "if" direction.
Suppose that there is a polynomial-time Turing machine $M$ that on input $1^n$ outputs a circuit $C_n$ that computes $F_{\upharpoonright n}$. Then the following is a polynomial-time Turing machine $M'$ to compute $F$. On input $x\in \{0,1\}^*$, $M'$ will:

1. Let $n=|x|$ and compute $C_n = M(1^n)$.

2. Return the evaluation of $C_n$ on $x$.

Since we can evaluate a Boolean circuit on an input in polynomial time, $M'$ runs in polynomial time and computes $F(x)$ on every input $x$.

For the "only if" direction, if $M'$ is a Turing machine that computes $F$ in polynomial-time, then (applying the equivalence of Turing machines and NAND-TM as well as [obliviousnandtmthm](){.ref}) there is also an oblivious NAND-TM program $P$ that computes $F$ in  time $p(n)$ for some polynomial $p$.
We can now define $M$ to be the Turing machine that on input $1^n$ outputs the NAND circuit obtained by "unrolling the loop" of $P$ for $p(n)$ iterations.
The resulting NAND circuit computes $F_{\upharpoonright n}$ and  has $O(p(n))$ gates.
It can also be transformed to a Boolean circuit with $O(p(n))$ AND/OR/NOT gates.
:::


::: {.solvedexercise title="$\mathbf{P_{/poly}}$ characterization by advice" #adviceppoly}
Let $F:\{0,1\}^* \rightarrow \{0,1\}$. Then $F\in\mathbf{P_{/poly}}$ if and only if there exists a polynomial $p:\N \rightarrow \N$, a polynomial-time Turing machine  $M$ and a sequence $\{ a_n \}_{n\in \N}$ of strings, such that for every $n\in \N$:

* $|a_n| \leq p(n)$  \
* For every $x\in \{0,1\}^n$, $M(a_n,x)=F(x)$.
:::

::: {.solution data-ref="adviceppoly"}
We only sketch the proof.
For the "only if" direction, if $F\in \mathbf{P_{/poly}}$ then we can use for $a_n$  simply the description of the corresponding circuit $C_n$ and for $M$ the program that computes in polynomial time the evaluation of a circuit on its input. 

For the "if" direction, we can use the same "unrolling the loop" technique of [non-uniform-thm](){.ref} to show that if $P$ is a polynomial-time NAND-TM program, then for every $n\in \N$, the map $x \mapsto P(a_n,x)$ can be computed by a polynomial-size NAND-CIRC program $Q_n$.
:::




### Can uniform algorithms  simulate non-uniform ones?

[non-uniform-thm](){.ref} shows that every function in $TIME(T(n))$ is in $SIZE(poly(T(n)))$.
One can ask if there is an inverse relation.
Suppose that $F$ is such that $F_{\upharpoonright n}$ has a "short" NAND-CIRC program for every $n$.
Can we say that it must be in $TIME(T(n))$ for some "small" $T$?
The answer is an emphatic __no__.
Not only is $\mathbf{P_{/poly}}$ not contained in $\mathbf{P}$, in fact $\mathbf{P_{/poly}}$ contains functions that are _uncomputable_!


> ### {.theorem title="$\mathbf{P_{/poly}}$ contains uncomputable functions" #Ppolyuncomputable}
There exists an _uncomputable_ function $F:\{0,1\}^* \rightarrow \{0,1\}$ such that $F \in \mathbf{P_{/poly}}$.


> ### {.proofidea data-ref="Ppolyuncomputable"}
Since $\mathbf{P_{/poly}}$ corresponds to non-uniform computation, a function $F$ is in $\mathbf{P_{/poly}}$ if for every $n\in \N$, the restriction $F_{\upharpoonright n}$ to inputs of length $n$ has a small circuit/program, even if the circuits for different values of $n$ are completely different from one another. In particular, if $F$ has the property that for every equal-length inputs $x$ and $x'$, $F(x)=F(x')$ then this means that $F_{\upharpoonright n}$ is either the constant function zero or the constant function one for every $n\in \N$.
Since the constant function has a (very!) small circuit, such a function $F$ will always be in $\mathbf{P_{/poly}}$ (indeed even in smaller classes).
Yet by a reduction from the Halting problem, we can obtain a function with this property that is uncomputable.

::: {.proof data-ref="Ppolyuncomputable"}
Consider the following "unary halting function" $UH:\{0,1\}^* \rightarrow \{0,1\}$ defined as follows.
We let $S:\N \rightarrow \{0,1\}^*$ be the function that on input $n\in \N$, outputs the string that corresponds to the binary representation of the number $n$ without the most significant $1$ digit.
Note that $S$ is _onto_.
For every $x\in \{0,1\}^*$, we define $UH(x)=HALTONZERO(S(|x|))$.
That is, if $n$ is the length of $x$, then $UH(x)=1$ if and only if the string $S(n)$ encodes a NAND-TM program that halts on the input $0$.


$UH$ is uncomputable, since otherwise we could compute $HALTONZERO$ by transforming the input program $P$ into the integer $n$ such that $P=S(n)$ and then running $UH(1^n)$ (i.e., $UH$ on the string of $n$ ones).
On the other hand, for every $n$, $UH_n(x)$ is either equal to $0$ for all inputs $x$ or equal to $1$ on all inputs $x$, and hence can be computed by a NAND-CIRC program of a _constant_ number of lines.
:::


The issue here is of course _uniformity_.
For a function $F:\{0,1\}^* \rightarrow \{0,1\}$, if $F$ is in $TIME(T(n))$ then we have a _single_ algorithm that can compute $F_{\upharpoonright n}$ for every $n$.
On the other hand,  $F_{\upharpoonright n}$ might be in  $SIZE(T(n))$ for every $n$ using a completely different algorithm for every input length.
For this reason we typically use $\mathbf{P_{/poly}}$ not as a model of _efficient_ computation but rather as a way to model _inefficient computation_.
For example, in cryptography people often define an encryption scheme to be secure if breaking it for a key of length $n$ requires more than a polynomial number of NAND lines.
Since $\mathbf{P} \subseteq \mathbf{P_{/poly}}$, this in particular precludes a polynomial time algorithm for doing so, but there are technical reasons why working in a non-uniform model makes more sense in cryptography.
It also allows to talk about security in non-asymptotic terms such as a scheme having "$128$ bits of security".

While it can sometimes be a real issue, in many natural settings the difference between uniform and non-uniform computation does not seem so important.
In particular, in all the examples of problems not known to be in $\mathbf{P}$ we discussed before: longest path, 3SAT, factoring, etc., these problems are also not known to be in $\mathbf{P_{/poly}}$ either.
Thus, for "natural" functions, if you pretend that $TIME(T(n))$  is roughly the same as $SIZE(T(n))$, you will be right more often than wrong.


![Relations between $\mathbf{P}$, $\mathbf{EXP}$, and $\mathbf{P_{/poly}}$. It is known that $\mathbf{P} \subseteq \mathbf{EXP}$, $\mathbf{P} \subseteq \mathbf{P_{/poly}}$ and that $\mathbf{P_{/poly}}$ contains uncomputable functions (which in particular are outside of $\mathbf{EXP}$). It is not known whether or not $\mathbf{EXP} \subseteq \mathbf{P_{/poly}}$ though it is believed that $\mathbf{EXP} \not\subseteq \mathbf{P_{/poly}}$.](../figure/PEXPPpolyrelations.png){#PEXPPpolyrelationsfig}



### Uniform vs. Non-uniform computation: A recap

To summarize, the two models of computation we have described so far are:


* **Uniform models:** _Turing machines_, _NAND-TM programs_,  _RAM machines_, _NAND-RAM programs_, _C/JavaScript/Python_, etc.  These models include loops and unbounded memory  hence a single program can compute a function with unbounded input length. 


* **Non-uniform models:** _Boolean Circuits_ or _straightline programs_  have no loops and can only compute finite functions. The time to  execute them is exactly the number of lines or gates they contain. 

For a function $F:\{0,1\}^* \rightarrow \{0,1\}$ and some nice time bound $T:\N \rightarrow \N$, we know that:

* If $F$ is uniformly computable in time $T(n)$ then there is a sequence of circuits $C_1,C_2,\ldots$ where $C_n$ has $poly(T(n))$ gates and computes $F_{\upharpoonright n}$ (i.e., restriction of $F$ to $\{0,1\}^n$) for every $n$.

* The reverse direction is not necessarily true - there are examples of functions $F:\{0,1\}^n \rightarrow \{0,1\}$ such that $F_{\upharpoonright n}$ can be computed by even a constant size circuit but  $F$ is uncomputable.

This means that non-uniform complexity is more useful to establish _hardness_ of a function than its _easiness_.




::: { .recap }
* We can define the time complexity of a function using NAND-TM programs, and similarly to the notion of computability, this appears to capture the inherent complexity of the function.

* There are many natural problems that have polynomial-time algorithms, and other natural problems that we'd love to solve, but for which the best known algorithms are exponential.

* The definition of polynomial time, and hence the class $\mathbf{P}$, is robust to the choice of model, whether it is Turing machines, NAND-TM, NAND-RAM, modern programming languages, and many other models.

* The time hierarchy theorem shows that there are _some_ problems that can be solved in exponential, but not in polynomial time. However, we do not know if that is the case for the natural examples that we described in this lecture.

* By "unrolling the loop" we can show that every function computable in time $T(n)$ can be computed by a sequence of NAND-CIRC programs (one for every input length) each of size at most $poly(T(n))$.
:::


## Exercises


::: {.exercise title="Equivalence of different definitions of $\mathbf{P}$ and $\mathbf{EXP}$." #definitionofP}
Prove that the classes $\mathbf{P}$ and $\mathbf{EXP}$ defined in [PandEXPdef](){.ref} are equal to $\cup_{c\in \{1,2,3,\ldots \}} TIME(n^c)$ and $\cup_{c\in \{1,2,3,\ldots \}} TIME(2^{n^c})$ respectively.
(If $S_1,S_2,S_3,\ldots$ is a collection of sets then the set $S = \cup_{c\in \{1,2,3,\ldots \}} S_c$ is the set of all elements $e$ such that there exists some $c\in \{ 1,2,3,\ldots \}$ where $e\in S_c$.)
:::


::: {.exercise title="Robustness to representation" #robsutrepresex }
[polyRAMTM-thm](){.ref} shows that the classes $\mathbf{P}$ and $\mathbf{EXP}$ are _robust_ with respect to variations in the choice of the computational model.
This exercise shows that these classes are also robust with respect to our choice of the representation of the input.

Specifically, let $F$ be a function mapping graphs to $\{0,1\}$, and let $F', F'':\{0,1\}^* \rightarrow \{0,1\}$ be the functions defined as follows. For every $x\in \{0,1\}^*$:

* $F'(x)=1$ iff $x$ represents a graph $G$ via the adjacency matrix representation such that $F(G)=1$.

* $F''(x)=1$ iff $x$ represents a graph $G$ via the adjacency list representation such that $F(G)=1$.

Prove that $F' \in \mathbf{P}$ iff $F'' \in \mathbf{P}$.


More generally, for every function $F:\{0,1\}^* \rightarrow \{0,1\}$, the answer to the question of whether $F\in \mathbf{P}$ (or whether $F\in \mathbf{EXP}$) is unchanged by switching representations, as long as transforming one representation to the other can be done in polynomial time (which essentially holds for all reasonable representations).
:::



::: {.exercise title="Boolean functions" #boolex}
For every function $F:\{0,1\}^* \rightarrow \{0,1\}^*$, define $Bool(F)$ to be the function mapping $\{0,1\}^*$ to $\{0,1\}$ such that on input a (string representation of a) triple $(x,i,\sigma)$ with $x\in \{0,1\}^*$, $i \in \N$ and $\sigma \in \{0,1\}$,

$$
Bool(F)(x,i,\sigma) = \begin{cases} F(x)_i & \sigma =0, i<|F(x)| \\
									1      & \sigma = 1,i<|F(x)| \\
								 0   & \text{otherwise} \end{cases}
$$
where $F(x)_i$ is the $i$-th bit of the string $F(x)$.

Prove that for every $F:\{0,1\}^* \rightarrow \{0,1\}^*$, $Bool(F) \in \mathbf{P}$ if and only if there is a Turing Machine $M$ and a polynomial $p:\N \rightarrow \N$ such  that for every $x\in \{0,1\}^*$, on input $x$, $M$ halts within $\leq p(|x|)$ steps and outputs $F(x)$.
:::


> ### {.exercise title="Composition of polynomial time" #poly-time-comp-ex}
Say that a (possibly non-Boolean) function $F:\{0,1\}^* \rightarrow \{0,1\}^*$ is _computable in polynomial time_, if there is a Turing Machine $M$ and a polynomial $p:\N \rightarrow \N$ such  that for every $x\in \{0,1\}^*$, on input $x$, $M$ halts within $\leq p(|x|)$ steps and outputs $F(x)$.
Prove that for every pair of functions  $F,G:\{0,1\}^* \rightarrow \{0,1\}^*$ computable in polynomial time, their _composition_ $F\circ G$, which is the function $H$ s.t. $H(x)=F(G(x))$, is also computable in polynomial time.


> ### {.exercise title="Non-composition of exponential time" #exp-time-comp-ex}
Say that a (possibly non-Boolean) function $F:\{0,1\}^* \rightarrow \{0,1\}^*$ is _computable in exponential  time_, if there is a Turing Machine $M$ and a polynomial $p:\N \rightarrow \N$ such  that for every $x\in \{0,1\}^*$, on input $x$, $M$ halts within $\leq 2^{p(|x|)}$ steps and outputs $F(x)$.
Prove that there is some $F,G:\{0,1\}^* \rightarrow \{0,1\}^*$ s.t. both $F$ and $G$ are computable in exponential time, but $F\circ G$ is not computable in exponential time.


::: {.exercise title="Oblivious Turing Machines" #oblivious-ex}
We say that a Turing machine $M$ is _oblivious_ if there is some function $T:\N\times \N \rightarrow \Z$ such that for every input $x$ of length $n$, and $t\in \N$ it holds that:

* If $M$ takes more than $t$ steps to halt on the input $x$, then in the $t$-th step $M$'s head will be in the position $T(n,t)$. (Note that this position depends only on the _length_ of $x$ and not its contents.)

* If $M$ halts before the $t$-th step then $T(n,t) = -1$.

Prove that if $F\in \mathbf{P}$ then there exists an _oblivious_ Turing machine $M$ that computes $F$ in polynomial time. See footnote for hint.^[_Hint:_ This is the Turing machine analog of [obliviousnandtmthm](){.ref}. We replace one step of the original TM $M'$ computing $F$ with a "sweep" of the obliviouss TM $M$ in which it goes $T$ steps to the right and then $T$ steps to the left.]
:::


::: {.exercise  #graphedgeex}
Let $EDGE:\{0,1\}^* \rightarrow \{0,1\}$ be the function such that on input a string representing a triple $(L,i,j)$, where $L$ is the adjacency list representation of an $n$ vertex graph $G$, and $i$ and $j$ are numbers in $[n]$, $EDGE(L,i,j)=1$ if the edge $\{i,j \}$ is present in the graph. $EDGE$ outputs $0$ on all other inputs.

1. Prove that $EDGE \in \mathbf{P}$.


2. Let $PLANARMATRIX:\{0,1\}^* \rightarrow \{0,1\}$ be the function that on input an adjacency matrix $A$ outputs $1$ if and only if the graph represented by $A$ is _planar_ (that is, can be drawn on the plane without edges crossing one another). For this question, you can use without proof the fact that $PLANARMATRIX \in \mathbf{P}$. Prove that $PLANARLIST \in \mathbf{P}$ where $PLANARLIST:\{0,1\}^* \rightarrow \{0,1\}$ is the function that on input an adjacency list $L$ outputs $1$ if and only if $L$ represents a planar graph.
:::


::: {.exercise title="Evaluate NAND circuits" #evalnandcircuit}
Let $NANDEVAL:\{0,1\}^* \rightarrow \{0,1\}$ be the function such that for every string representing a pair $(Q,x)$ where $Q$ is an $n$-input $1$-output
NAND-CIRC (not NAND-TM!) program  and $x\in \{0,1\}^n$, $NANDEVAL(Q,x)=Q(x)$.  On all other inputs $NANDEVAL$ outputs $0$.

Prove that $NANDEVAL \in \mathbf{P}$.
:::

::: {.exercise title="Find hard function" #hardfunc}
Let $NANDHARD:\{0,1\}^* \rightarrow \{0,1\}$ be the function such that on input a string representing a  pair $(f,s)$ where

* $f \in \{0,1\}^{2^n}$ for some $n\in \mathbb{N}$
* $s\in \mathbb{N}$

$NANDHARD(f,s)=1$ if there is no NAND-CIRC program $Q$ of at most $s$ lines that computes the function $F:\{0,1\}^n \rightarrow \{0,1\}$ whose truth table is the string $f$.
That is, $NANDHARD(f,s)=1$ if for every NAND-CIRC program $Q$ of at most $s$ lines, there exists some $x\in \{0,1\}^{n}$ such that $Q(x) \neq f_x$ where $f_x$ denote the $x$-the coordinate of $f$, using the binary representation to identify $\{0,1\}^n$ with the numbers $\{0,\ldots,2^n -1 \}$.

1. Prove that $NANDHARD \in \mathbf{EXP}$.

2. (Challenge) Prove that there is an algorithm $FINDHARD$ such that if $n$ is sufficiently large, then $FINDHARD(1^n)$ runs in time $2^{2^{O(n)}}$ and outputs a string $f \in \{0,1\}^{2^n}$ that is the truth table of a function that is not contained in  $SIZE(2^n/(1000n))$. (In other words, if $f$ is the string output by $FINDHARD(1^n)$ then if we let $F:\{0,1\}^n \rightarrow \{0,1\}$ be the function such that $F(x)$ outputs the $x$-th coordinate of $f$, then $F\not\in SIZE(2^n/(1000n))$.^[__Hint:__ Use Item 1, the existence of functions requiring exponentially hard NAND programs, and the fact that there are only finitely many functions mapping $\{0,1\}^n$ to $\{0,1\}$.]
:::





::: {.exercise  #scheduleprogex}
Suppose that you are in charge of scheduling courses  in computer science in University X. In University X, computer science students wake up late, and have to work on their startups in the afternoon, and take long weekends with their investors. So you only have two possible slots: you can schedule a course either Monday-Wednesday 11am-1pm or Tuesday-Thursday 11am-1pm.


Let $SCHEDULE:\{0,1\}^* \rightarrow \{0,1\}$ be the function that takes as input a list of courses $L$ and a list of _conflicts_ $C$ (i.e., list of pairs of courses that cannot share the same time slot) and outputs  $1$ if and only if there is a "conflict free" scheduling of the courses in $L$, where no pair in $C$ is scheduled in the same time slot.

More precisely, the list $L$ is a list of strings $(c_0,\ldots,c_{n-1})$ and the list $C$ is a list of pairs of the form $(c_i,c_j)$. $SCHEDULE(L,C)=1$ if and only if there exists a partition of $c_0,\ldots,c_{n-1}$ into two parts so that there is no pair $(c_i,c_j) \in C$ such that both $c_i$ and $c_j$ are in the same part.

Prove that $SCHEDULE \in \mathbf{P}$.  As usual, you do not have to provide the full code to show that this is the case, and can describe operations as a high level, as well as appeal to any data structures or other results mentioned in the book or in lecture. Note that to show that a function $F$ is in $\mathbf{P}$ you need to both __(1)__ present an algorithm $A$ that computes $F$ in polynomial time, __(2)__ _prove_ that $A$ does indeed run in polynomial time, and does indeed compute the correct answer.

Try to think whether or not your algorithm extends to the case where there are _three_ possible time slots.
:::




## Bibliographical notes {#bibnotesrunningtime }

Because we are interested in the _maximum_ number of steps for inputs of a given length, running-time as we defined it is often known as _worst case complexity_. The _minimum_ number of steps (or "best case" complexity) to compute a function on length $n$ inputs is typically not a meaningful quantity since essentially every natural problem will have some trivially easy instances.
However, the _average case complexity_ (i.e., complexity on a "typical" or "random" input) is an interesting concept which we'll return to when we discuss _cryptography_.
That said, worst-case complexity is the most standard and basic of the complexity measures, and will be our focus in most of this book.




Some lower bounds for single-tape Turing machines are given in [@maass1985combinatorial].

For defining efficiency in the  $\lambda$ calculus, one needs to be careful about the order of application of the reduction steps, which can matter for computational efficiency, see for example [this paper](https://lmcs.episciences.org/1627).



The notation $\mathbf{P_{/poly}}$ is used for historical reasons.
It was introduced by Karp and Lipton, who considered this class as corresponding to functions that can be computed by polynomial-time Turing machines that are given for any input length $n$ an _advice string_ of length polynomial in $n$.





