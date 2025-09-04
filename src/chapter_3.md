# 定义计算 {#compchap }
> *“没有理由不借助机器来节省脑力劳动和体力劳动。”* —— Charles Babbage，1852   

> *“如果有谁不以我的例子为戒，而尝试并成功地用不同的原理或更简单的机械手段，构造出一台在自身中体现数学分析执行部门全部功能的机器，那么我丝毫不担心将我的声誉交付于他，因为唯有他能完全理解我努力的性质及其成果的价值。”* —— Charles Babbage，1864 

> *“要理解一个程序，你必须既成为机器，又成为程序。”* —— Alan Perlis，1982 
## 学习目标 {.objectives  }
* 理解计算可以被精确建模。  
* 学习 **布尔电路** / **直线程序** 的计算模型。  
* 电路与直线程序的等价性。  
* AND/OR/NOT 与 NAND 的等价性。  
* 物理世界中的计算实例。  

## 目录

<!-- toc -->

> ![#babbagewheels .margin](./images/chapter3/wheels_babbage.png)
> Charles Babbage的计算轮。图片取自 Harvard Mark I 计算机的“操作手册”。

> ![#markIcomp .margin](./images/chapter3/PopularMechanics1944smaller.jpg)
> 摘自 **Popular Mechanics** 上的一篇关于 Harvard Mark I 计算机的[文章](http://sites.harvard.edu/~chsi/markone/about.html), 1944 年.

几千年来，人类一直在进行计算，不仅依靠纸笔，还使用过算盘、计算尺、各种机械装置，直到现代的电子计算机。  
从先验的角度来看，**计算**这一概念似乎总是依赖于所使用的具体工具。  
例如，你也许会认为，在现代笔记本电脑上用 **Python** 实现的乘法算法，与用纸笔进行乘法运算时的“最佳”算法会有所不同。  

然而，正如我们在[引言](chapter_0.md)中所看到的，一个在渐近意义上更优的算法，无论底层技术如何，最终都会优于较差的算法。  
这让我们看到希望：可以找到一种**独立于技术**的方式来刻画计算的概念。  

本章正是要做这件事。我们将把“从输入计算输出”定义为一系列基本操作的应用（见[下图](#compchapwhatvshowfig)）。  
借助这一框架，我们便能精确地表述诸如：“函数 $f$ 可以由模型 $X$ 计算”或“函数 $f$ 可以由模型 $X$ 在 $s$ 步操作内计算完成”这样的命题。  


> <a id="compchapwhatvshowfig"> ![compchapwhatvshowfig](./images/chapter3/compchapterwhatvshow.png) {#compchapwhatvshowfig} </a>
> 一个将字符串映射到字符串的函数，**规定**了一项计算任务，也就是说，它描述了输入与输出之间所期望的关系。在本章中，我们将定义一些模型，用来**实现**这些计算过程，从而达到所需的关系，也就是描述**如何**根据输入来计算输出。我们将看到若干此类模型的例子，包括布尔电路和直线型编程语言。


::: {.nonmath}
阅读本章, 读者主要有以下收获：

* 我们可以使用 **逻辑运算**，如 $AND$(与)、$OR$(或) 和 $NOT$(非)，从输入计算输出（见 [3.2节](#andornotsec)）。

* **布尔电路** 是一种通过组合基本逻辑运算来计算更复杂函数的方法（见 [3.3节](#booleancircuitsec)）。  
  我们既可以将布尔电路看作一种数学模型（基于有向无环图），也可以将其视为现实世界中可实现的物理装置。实现方式多种多样，不仅包括基于硅的半导体，还包括机械甚至生物机制（见 [3.5节](#physicalimplementationsec){.ref}）。

* 我们还可以把布尔电路描述为 **直线型程序**，即不包含循环结构的程序（没有 `while` / `for` / `do .. until` 等）（见 [3.4节](#starightlineprogramsec){.ref}）。

* 可以通过 $NAND$ 运算来实现 $AND$、$OR$ 和 $NOT$ 运算（反之亦然）。  
  这意味着带有 $AND$/$OR$/$NOT$ 门的电路，与带有 $NAND$ 门的电路在计算能力上是**等价的**，我们可以根据需要选择其中任一模型来描述计算（见 [3.6节](#nandsec){.ref}）。  
  先提前剧透一下，在 [下一章](chapter_4.md){.ref} 中我们将看到，这类电路可以计算**所有有限函数**。

本章的一个“大思想”是 **模型之间的等价性**（见 [equivalencemodels](#equivalencemodels){.ref}）。  
如果两个计算模型能够计算相同集合的函数，那么它们就是**等价的**。  
布尔电路（$AND$/$OR$/$NOT$ 门）与 $NAND$ 电路的等价性只是一个例子，本书中我们还会多次遇到类似的普遍现象。

:::


## 3.1 定义计算

“算法”一词来源于对穆罕默德·伊本·穆萨·花剌子密(Muhammad ibn Musa al-Khwarizmi)名字的拉丁化转写。  
al-Khwarizmi 是九世纪的一位波斯学者，他的著作向西方世界介绍了十进位值制数字系统，以及一次方程与二次方程的解法（见 [下图](#alKhwarizmi)）。  
然而，以今天的标准来看，al-Khwarizmi 对算法的描述相当**不够形式化**。他没有使用如 $x,y$ 这样的**变量**，而是采用具体的数字（如 10 和 39），并依赖读者从这些例子中自行类推出一般情况——这与当今儿童学习算法时的教学方式颇为相似。  

以下是 al-Khwarizmi 对解形如 $x^2 + bx = c$ 方程的算法的描述：

> **“如何解形如‘平方与根的和等于某数’的方程”**  
> 举例来说：“一个平方加上它的十倍平方根等于三十九迪拉姆。”  
> 换句话说，求这样一个平方：它的平方加上它自身的十倍平方根，结果是三十九。  
>
> 解法如下：  
> 1. 将根的数量减半，本例中十的一半是五。  
> 2. 将这个数（五）平方，得到二十五。  
> 3. 将平方结果加到三十九上，得到六十四。  
> 4. 取六十四的平方根，得到八。  
> 5. 从平方根中减去根数量的一半（五），余数为三。  
>
> 因此，这个平方根为三，对应的平方为九。  

> <a id = "alKhwarizmi">![alKhwarizmi](./images/chapter3/alKhwarizmi.jpg)</a>
> 代数学手稿中的文字页，展示了解两类二次方程的几何解法。馆藏号：MS. Huntington 214, 页码 fol. 004v-005r


> <a id="childrenalg">![An explanation for children of the two digit addition algorithm](./images/chapter3/addition_regrouping.jpg)</a>
> 面向儿童的两位数加法算法讲解

为了本书的目的，我们需要一种更加精确的方式来描述算法。幸运（或者说不幸）的是，至少目前，计算机在从实例中学习方面远远落后于学龄儿童。因此，在 20 世纪，人们提出了用于精确描述算法的形式化方法，即 **编程语言**。  

下面是用 **Python** 描述的 al-Khwarizmi 二次方程求解算法：

```python
from math import sqrt
# 使用 Python 的 sqrt 函数来计算平方根

def solve_eq(b, c):
    # 根据 al-Khwarizmi 的方法求解 x^2 + b*x = c
    # al-Khwarizmi 在 b=10, c=39 的例子中演示了这个方法

    val1 = b / 2.0  # “将根的数量减半”
    val2 = val1 * val1  # “将这个数平方”
    val3 = val2 + c  # “将平方结果加到 c 上”
    val4 = sqrt(val3)  # “取和的平方根”
    val5 = val4 - val1  # “从平方根中减去根数量的一半”
    return val5  # “这就是所求的平方根”

# 测试：求解 x^2 + 10*x = 39
print(solve_eq(10, 39))
# 输出 3.0
```
我们可以非正式地定义算法如下：

::: {.quote}
> __算法的非正式定义：__ **算法**是一组指令，用于通过执行一系列“基本步骤”从输入计算出输出。
> 如果对于每一个输入 $x$，按照算法 $A$ 的指令操作都能得到输出 $F(x)$，则称算法 $A$ **计算**函数 $F$。
:::

在本章中，我们将使用**布尔电路（Boolean Circuits）** 模型，将这一非正式定义变得精确。我们将展示，布尔电路在计算能力上等价于用“极简”编程语言编写的 **直线程序（straight line programs）**，即不包含循环的编程语言。我们还将看到，具体选择哪种**基本操作（elementary operations）**并不重要，不同的选择都可以得到计算能力等价的模型（见[下图](#compchapoverviewfig)）。然而，要理解这一点，我们需要一些时间。我们将从讨论什么是“基本操作”开始，并说明如何将算法的描述映射为实际物理过程，使其在现实世界中从输入生成输出。

> <a id="compchapoverviewfig"> ![An overview of the computational models defined in this chapter. We will show several equivalent ways to represent a recipe for performing a finite computation. Specifically we will show that we can model such a computation using either a _Boolean circuit_ or a _straight line program_, and these two representations are equivalent to one another. We will also show that we can choose as our basic operations either the set $\{ AND , OR , NOT \}$ or the set $\{ NAND \}$ and these two choices are equivalent in power. By making the choice of whether to use circuits or programs, and whether to use   $\{ AND , OR , NOT \}$ or  $\{ NAND \}$ we obtain four equivalent ways of modeling finite computation. Moreover, there are many other choices of sets of basic operations that are equivalent in power.](./images/chapter3/compcharoverview.png)</a>
> 本章定义的计算模型概览。我们将展示几种等价的方式来表示执行有限计算的“操作方案”。具体来说，我们将证明，可以使用**布尔电路（Boolean circuit）**或**直线程序（straight line program）**来建模这样的计算，这两种表示方式在计算能力上是等价的。我们还将展示，作为基本操作，我们可以选择集合 $\{ AND , OR , NOT \}$ 或集合 $\{ NAND \}$，这两种选择在计算能力上同样等价。通过选择使用电路还是程序，以及选择 $\{ AND , OR , NOT \}$ 或 $\{ NAND \}$，我们可以得到四种等价的有限计算建模方法。此外，还有许多其他基本操作集合的选择，它们在计算能力上同样是等价的。

## 3.2 使用与, 或, 非进行计算 { #andornotsec }

算法将一个**复杂**的计算分解为一系列**更简单**的步骤。这些步骤可以通过多种不同的方式来执行，包括：

* 在纸上书写符号。  
* 改变电线中的电流。  
* 蛋白质与 DNA 链结合。  
* 集体中的个体对刺激做出反应（例如，蜂群中的蜜蜂，市场中的交易者）。  

为了形式化地定义算法，我们尝试“化繁为简”，挑出组成算法的"最小单位", 例如下列一组简单逻辑函数:

* $OR:\{0,1\}^2 \rightarrow \{0,1\}$ 定义为

$$OR(a,b) = \begin{cases} 0 & a=b=0 \\ 1 & \text{otherwise} \end{cases}$$

* $AND:\{0,1\}^2 \rightarrow \{0,1\}$ 定义为

$$AND(a,b) = \begin{cases} 1 & a=b=1 \\ 0 & \text{otherwise} \end{cases}$$

* $NOT:\{0,1\} \rightarrow \{0,1\}$ 定义为

$$NOT(a) = \begin{cases} 0 & a = 1 \\ 1 & a = 0 \end{cases}$$

函数 $AND$、$OR$ 和 $NOT$ 是逻辑学以及许多计算机系统中使用的基本逻辑运算符。在逻辑学中, $AND(a,b)$ 表示为 $a \wedge b$，$OR(a,b)$ 表示为 $a \vee b$，$NOT(a)$ 表示为 $\overline{a}$ 或 $\neg a$，我们也将采用这种表示法。

每一个函数 $AND, OR, NOT$ 都以一个或两个单比特作为输入，并输出一个单比特。显然，它们已经达到了非常基础的层次。然而，计算的威力正来源于**将这些简单的函数组合在一起**。


::: {.example title="用 $AND$,$OR$ 和 $NOT$ 写出多数函数 MAJ" #majorityfunctionex}

考虑函数 $MAJ:\{0,1\}^3 \rightarrow \{0,1\}$，其定义如下：

$$
MAJ(x) = \begin{cases}
1 & x_0 + x_1 + x_2 \geq 2 \\
0 & \text{otherwise}
\end{cases} \;.
$$

也就是说，对于每个 $x\in \{0,1\}^3$，当且仅当 $x$ 的三个元素中至少有两个等于 $1$ 时，$MAJ(x)=1$。你能用 $AND$、$OR$ 和 $NOT$ 写出一个计算 $MAJ$ 的公式吗？（此处建议你先停下来自己推导公式。提示：虽然某些函数需要用到 $NOT$，但计算 $MAJ$ 不需要使用它。）

我们先用文字重新表述 $MAJ(x)$：“当且仅当存在一对不同的元素 $i,j$，且 $x_i$ 和 $x_j$ 都等于 $1$ 时，$MAJ(x)=1$。”  
换句话说，$MAJ(x)=1$ 当且仅当 **$x_0=1$ 且 $x_1=1$**，**或** **$x_1=1$ 且 $x_2=1$**，**或** **$x_0=1$ 且 $x_2=1$**。

由于三个条件 $c_0, c_1, c_2$ 的 $OR$ 可以写作 $OR(c_0, OR(c_1, c_2))$，我们可以将其翻译为如下公式：

$$
MAJ(x_0,x_1,x_2) = OR\left(AND(x_0,x_1),OR \bigl(AND(x_1,x_2),AND(x_0,x_2)\bigr)\right). {{numeq}}{eqmajandornot}
$$

回想一下，我们也可以将 $OR(a,b)$ 写作 $a \vee b$，将 $AND(a,b)$ 写作 $a \wedge b$。使用这种符号表示，公式 {{eqref: eqmajandornot}} 也可以写作：

$$MAJ(x_0,x_1,x_2) = ((x_0 \wedge x_1) \vee (x_1 \wedge x_2)) \vee (x_0 \wedge x_2)\;.$$

我们也可以将公式 {{eqref:eqmajandornot}} 以“编程语言”的形式表示: 将其表达为一组指令，用于在给定基本操作 $AND, OR, NOT$ 的情况下计算 $MAJ$：

```python
def MAJ(X[0],X[1],X[2]):
    firstpair  = AND(X[0],X[1])
    secondpair = AND(X[1],X[2])
    thirdpair  = AND(X[0],X[2])
    temp       = OR(secondpair,thirdpair)
    return OR(firstpair,temp)
```
:::

<iframe src="https://trinket.io/embed/python/5ead2eab1b" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>


### AND和OR的一些性质

与标准的加法和乘法类似，函数 $AND$ 和 $OR$ 满足**交换律**：$a \vee b = b \vee a$ 和 $a \wedge b = b \wedge a$，以及**结合律**：$(a \vee b) \vee c = a \vee (b \vee c)$ 和 $(a \wedge b) \wedge c = a \wedge (b \wedge c)$。  

于是如同加法和乘法的情况，我们通常可以省略括号，将 $((a \vee b) \vee c) \vee d$ 写作 $a \vee b \vee c \vee d$，对更多项的 $AND$ 和 $OR$ 同理。  

它们还满足分配律的一种变体：

{.solvedexercise title="$AND$ 与 $OR$ 满足分配律" #distributivelaw}
证明：对于任意 $a,b,c \in \{0,1\}$，都有

$$
  a \wedge (b \vee c) = (a \wedge b) \vee (a \wedge c)。
$$


{.solution data-ref="distributivelaw"}
我们可以通过枚举 $a,b,c \in \{0,1\}$ 的所有 $8$ 种可能取值来证明这一点，但它也可以直接从标准的分配律推导出来。  

假设我们将任意正整数视为“真”，将零视为“假”。那么对于每个数 $u,v \in \mathbb{N}$，$u+v$ 为正当且仅当 $u \vee v$ 为真，而 $u \cdot v$ 为正当且仅当 $u \wedge v$ 为真。  

这意味着对于每个 $a,b,c \in \{0,1\}$，表达式 $a \wedge (b \vee c)$ 为真当且仅当 $a \cdot (b+c)$ 为正，而表达式 $(a \wedge b) \vee (a \wedge c)$ 为真当且仅当 $a \cdot b + a \cdot c$ 为正。  

根据标准的分配律 $a \cdot (b+c) = a \cdot b + a \cdot c$，因此前者表达式为真当且仅当后者表达式为真。




### 扩展例子: 计算异或(XOR) {#xoraonexample }

让我们看看如何用相同的基本构建块得到一个不同的函数。定义 $XOR:\{0,1\}^2 \rightarrow \{0,1\}$ 为函数 $XOR(a,b) = a + b \mod 2$。也就是说，$XOR(0,0) = XOR(1,1) = 0$，$XOR(1,0) = XOR(0,1) = 1$。  

我们声称可以仅使用 $AND$、$OR$ 和 $NOT$ 来构造 $XOR$。


::: { .pause }
像往常一样，一个很好的练习是在继续阅读之前，先尝试自己用 **AND**、**OR** 和 **NOT** 算法推导出 **XOR** 的实现方法。
:::


以下算法使用 $AND$、$OR$ 和 $NOT$ 来计算 $XOR$：

> {{alg}}{XORfromAONalg}[用 $AND$, $OR$ 与 $NOT$ 计算 $XOR$]
> $$
  \begin{array}{l}
  \mathbf{Input:}\ a,b \in \{0,1\} \\
  \mathbf{Output:}\ XOR(a,b) \\
  \hline
  \mathbf{Step 1:}\ w_1 \leftarrow AND(a,b) \\
  \mathbf{Step 2:}\ w_2 \leftarrow NOT(w_1) \\
  \mathbf{Step 3:}\ w_3 \leftarrow OR(a,b) \\
  \mathbf{Step 4: return}\ AND(w_2,w_3)
  \end{array}
  $$

{{lem}}{alganalaysis}
对于每个 $a,b \in \{0,1\}$，在输入 $a,b$ 时，算法 [XORfromAONalg](){.ref} 输出 $a + b \mod 2$。

::: {.proof data-ref="alganalaysis"}
对于任意 $a,b$，有 $XOR(a,b)=1$ 当且仅当 $a$ 与 $b$ 不同。  
在输入 $a,b \in \{0,1\}$ 时，[XORfromAONalg](){.ref} 输出  
$$
AND(w2, w3)
$$  
其中 $w2 = NOT(AND(a,b))$ 且 $w3 = OR(a,b)$。

* 如果 $a=b=0$，则 $w3 = OR(a,b) = 0$，因此输出为 $0$。

* 如果 $a=b=1$，则 $AND(a,b) = 1$，所以 $w2 = NOT(AND(a,b)) = 0$，输出为 $0$。

* 如果 $a=1$ 且 $b=0$（或反之），则 $w3 = OR(a,b) = 1$ 且 $w1 = AND(a,b) = 0$，此时算法输出  
$$
OR(NOT(w1), w3) = 1。
$$

:::

We can also express [XORfromAONalg](){.ref} using a programming language.
Specifically, the following is a _Python_ program that computes the $XOR$ function:

```python
def AND(a,b): return a*b
def OR(a,b):  return 1-(1-a)*(1-b)
def NOT(a):   return 1-a

def XOR(a,b):
    w1 = AND(a,b)
    w2 = NOT(w1)
    w3 = OR(a,b)
    return AND(w2,w3)

# Test out the code
print([f"XOR({a},{b})={XOR(a,b)}" for a in [0,1] for b in [0,1]])
# ['XOR(0,0)=0', 'XOR(0,1)=1', 'XOR(1,0)=1', 'XOR(1,1)=0']
```




::: {.solvedexercise title="Compute $XOR$ on three bits of input" #xorthreebits}
Let $XOR_3:\{0,1\}^3 \rightarrow \{0,1\}$ be the function defined as $XOR_3(a,b,c) = a + b +c \mod 2$. That is, $XOR_3(a,b,c)=1$ if $a+b+c$ is odd, and $XOR_3(a,b,c)=0$ otherwise.
Show that you can compute $XOR_3$ using AND, OR, and NOT.
You can express it as a formula, use a programming language such as Python, or use a Boolean circuit.
:::

::: {.solution data-ref="xorthreebits"}
Addition modulo two satisfies the same properties of _associativity_ ($(a+b)+c=a+(b+c)$) and _commutativity_ ($a+b=b+a$) as standard addition.
This means that, if we define $a \oplus b$ to equal $a + b \mod 2$,
then
$$
XOR_3(a,b,c) = (a \oplus b) \oplus c
$$
or in other words
$$
XOR_3(a,b,c) = XOR(XOR(a,b),c) \;.
$$

Since we know how to compute $XOR$ using AND, OR, and NOT, we can compose this to compute $XOR_3$ using the same building blocks.
In Python this corresponds to the following program:

```python
def XOR3(a,b,c):
    w1 = AND(a,b)
    w2 = NOT(w1)
    w3 = OR(a,b)
    w4 = AND(w2,w3)
    w5 = AND(w4,c)
    w6 = NOT(w5)
    w7 = OR(w4,c)
    return AND(w6,w7)

# Let's test this out
print([f"XOR3({a},{b},{c})={XOR3(a,b,c)}" for a in [0,1] for b in [0,1] for c in [0,1]])
# ['XOR3(0,0,0)=0', 'XOR3(0,0,1)=1', 'XOR3(0,1,0)=1', 'XOR3(0,1,1)=0', 'XOR3(1,0,0)=1', 'XOR3(1,0,1)=0', 'XOR3(1,1,0)=0', 'XOR3(1,1,1)=1']
```

<iframe src="https://trinket.io/embed/python/0e71e3fcaa" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>

:::


> ### { .pause }
Try to generalize the above examples to obtain a way to compute $XOR_n:\{0,1\}^n \rightarrow \{0,1\}$ for every $n$ using at most $4n$ basic steps involving applications of a function in $\{ AND, OR , NOT \}$ to outputs or previously computed values.


### Informally defining "basic operations" and "algorithms"

We have seen that we can obtain at least some examples of interesting functions by composing together applications of $AND$, $OR$, and $NOT$.
This suggests that we can use $AND$, $OR$, and $NOT$ as our "basic operations", hence obtaining the following definition of an "algorithm":


::: {.quote}
__Semi-formal definition of an algorithm:__ An _algorithm_ consists of a sequence of steps of the form "compute a new value by applying $AND$, $OR$, or $NOT$ to previously computed values (assuming that the input was also previously computed)".

An algorithm $A$ _computes_ a function $F$ if for every input $x$ to $F$, if we feed $x$ as input to the algorithm, the value computed in its last step is $F(x)$.
:::


There are several concerns that are raised by this definition:

1. First and foremost, this definition is indeed too informal. We do not specify exactly what each step does, nor what it means to "feed $x$ as input".

2. Second, the choice of $AND$, $OR$ or $NOT$ seems rather arbitrary. Why not $XOR$ and $MAJ$? Why not allow operations like addition and multiplication? What about any other logical constructions such `if`/`then` or `while`?

3. Third, do we even know that this definition has anything to do with actual computing? If someone gave us a description of such an algorithm, could we use it to actually compute the function in the real world?


> ### { .pause }
These concerns will to a large extent guide us in the upcoming chapters. Thus you would be well advised to re-read the above informal definition and see what you think about these issues.


A large part of this book will be devoted to addressing the above issues. We will see that:

1. We can make the definition of an algorithm fully formal, and so give a precise mathematical meaning to statements such as "Algorithm $A$ computes function $f$".

2. While the choice of $AND$/$OR$/$NOT$ is arbitrary, and we could just as well have chosen other functions, we will also see this choice does not matter much. We will see that  we would obtain the same computational power if we instead used addition and multiplication, and essentially every other operation that could be reasonably thought of as a basic step.

3. It turns out that we can and do compute such "$AND$/$OR$/$NOT$-based algorithms" in the real world. First of all, such an algorithm is clearly well specified, and so can be executed by a human with a pen and paper. Second, there are a variety of ways to _mechanize_ this computation. We've already seen that we can write Python code that corresponds to following such a list of instructions. But in fact we can directly implement operations such as $AND$, $OR$, and $NOT$ via electronic signals using components known as _transistors_. This is how modern electronic computers operate.

In the remainder of this chapter, and the rest of this book, we will begin to answer some of these questions.
We will see more examples of the power of simple operations to compute more complex operations including addition, multiplication, sorting and more.
We will also discuss how to _physically implement_ simple operations such as $AND$, $OR$ and $NOT$ using a variety of technologies.



## 3.3 布尔电路  {#booleancircuitsec }

![Standard symbols for the logical operations or "gates" of $AND$, $OR$, $NOT$, as well as the operation $NAND$ discussed in [3.6节](#nandsec){.ref}.](./images/chapter3/logicgates.png){#logicgatesfig .margin }


![A circuit with $AND$, $OR$ and $NOT$ gates  for computing the $XOR$ function.](./images/chapter3/xorcircuitschemdraw.png){#smallandornotcircxorfig  .margin  }




_Boolean circuits_ provide a precise notion of  "composing basic operations together".
A Boolean circuit (see [boolancircfig](){.ref}) is composed of _gates_ and _inputs_ that are connected by _wires_.
The _wires_  carry a signal that represents either the value $0$ or $1$.
Each gate corresponds to either the _OR_, _AND_, or _NOT_ operation.
An _OR gate_ has two incoming wires, and one or more outgoing wires.
If these two incoming wires carry the signals $a$ and $b$ (for $a,b \in \{0,1\}$), then the signal on the outgoing wires will be $OR(a,b)$.
_AND_ and _NOT_ gates are defined similarly.
The _inputs_ have only outgoing wires.
If we set a certain input to a value $a\in \{0,1\}$, then this value is propagated on all the wires outgoing from it.
We also designate some gates as _output gates_, and their value corresponds to the result of evaluating the circuit.
For example,  [smallandornotcircxorfig](){.ref} gives such a circuit for the $XOR$ function, following [xoraonexample](){.ref}.
We evaluate an $n$-input Boolean circuit $C$ on an input $x\in \{0,1\}^n$ by placing the bits of $x$ on the inputs, and then propagating the values on the wires until we reach an output, see [boolancircfig](){.ref}.






::: {.remark title="Physical realization of Boolean circuits" #booleancircimprem}
Boolean circuits are a _mathematical model_ that does not necessarily correspond to a physical object, but they can be implemented physically.
In physical implementations of circuits, the signal is [often implemented](https://goo.gl/gntTQE) by electric potential, or _voltage_, on a wire, where for example voltage above a certain level is interpreted as a logical value of $1$, and below a certain level is interpreted as a logical value of $0$.
[physicalimplementationsec](){.ref} discusses physical implementations of Boolean circuits (with examples including using electrical signals such as in silicon-based circuits, as well as biological and mechanical implementations).
:::






![A _Boolean Circuit_ consists of  _gates_ that are connected by _wires_ to one another and the _inputs_. The left side depicts a circuit with $2$ inputs and $5$ gates, one of which is designated the output gate. The right side depicts the evaluation of this circuit on the input $x\in \{0,1\}^2$ with $x_0=1$ and $x_1=0$. The value of every gate is obtained by applying the corresponding function ($AND$, $OR$, or $NOT$) to values on the wire(s) that enter it. The output of the circuit on a given input is the value of the output gate(s). In this case, the circuit computes the $XOR$ function and hence it outputs $1$ on the input $10$.](./images/chapter3/booleancircuit.png){#boolancircfig  }




::: {.solvedexercise title="All equal function" #allequalex}
Define $ALLEQ:\{0,1\}^4 \rightarrow \{0,1\}$ to be the function that on input $x\in \{0,1\}^4$ outputs $1$ if and only if $x_0=x_1=x_2=x_3$. Give a Boolean circuit for computing $ALLEQ$.
:::


::: {.solution data-ref="allequalex"}
Another way to describe the function $ALLEQ$ is that it outputs $1$ on an input $x\in \{0,1\}^4$ if and only if $x = 0^4$ or $x=1^4$.
We can phrase the condition $x=1^4$ as $x_0 \wedge x_1 \wedge x_2 \wedge x_3$ which can be computed
using three AND gates.
Similarly we can phrase the condition $x=0^4$ as $\overline{x}_0 \wedge \overline{x}_1 \wedge \overline{x}_2 \wedge \overline{x}_3$ which can be computed using four NOT gates and three AND gates.
The output of $ALLEQ$ is the OR of these two conditions, which results in the circuit of 4 NOT gates, 6 AND gates, and one OR gate presented in [allequalfig](){.ref}.
:::

![A  Boolean circuit for computing the _all equal_ function $ALLEQ:\{0,1\}^4 \rightarrow \{0,1\}$ that outputs $1$ on $x\in \{0,1\}^4$ if and only if $x_0=x_1=x_2=x_3$.](./images/chapter3/allequalcirc2.png){#allequalfig .margin }

### Boolean circuits: a formal definition

We defined Boolean circuits informally as obtained by connecting _AND_, _OR_, and _NOT_ gates via wires so as to produce an output from an input.
However, to be able to prove theorems about the existence or non-existence of Boolean circuits for computing various functions we need to:

1. Formally define a Boolean circuit as a mathematical object.

2. Formally define what it means for a circuit $C$ to compute a function $f$.


We now proceed to do so.
We will define a Boolean circuit as a labeled _Directed Acyclic Graph (DAG)_.
The _vertices_ of the graph correspond to the gates and inputs of the circuit, and the _edges_ of the graph correspond to the wires.
A wire from an input or gate $u$ to a gate $v$ in the circuit corresponds to a directed edge between the corresponding vertices.
The inputs are vertices with no incoming edges, while each gate has the appropriate number of incoming edges based on the function it computes. (That is,  _AND_ and _OR_ gates have two in-neighbors, while _NOT_ gates have one in-neighbor.)
The formal definition is as follows (see also [generalcircuitfig](){.ref}):

![A _Boolean Circuit_ is a labeled directed acyclic graph (DAG). It has $n$ _input_ vertices, which are marked with `X[`$0$`]`,$\ldots$, `X[`$n-1$`]` and have no incoming edges, and the rest of the vertices are _gates_. _AND_, _OR_, and _NOT_ gates have two, two, and one incoming edges, respectively. If the circuit has $m$ outputs, then $m$ of the gates are known as _outputs_ and are marked with `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]`. When we evaluate a circuit $C$ on an input $x\in \{0,1\}^n$, we start by setting the value of the input vertices to $x_0,\ldots,x_{n-1}$ and then propagate the values, assigning to each gate $g$ the result of applying the operation of $g$ to the values of $g$'s in-neighbors. The output of the circuit is the value assigned to the output gates.](./images/chapter3/generalcircuit.png){#generalcircuitfig }

::: {.definition title="Boolean Circuits" #booleancircdef}
Let $n,m,s$ be positive integers with $s \geq m$. A _Boolean circuit_ with $n$ inputs, $m$ outputs, and $s$ gates, is a labeled directed acyclic graph (DAG) $G=(V,E)$ with $s+n$ vertices satisfying the following properties:

* Exactly $n$ of the vertices have no in-neighbors. These vertices are known as _inputs_ and are labeled with the $n$ labels `X[`$0$`]`, $\ldots$, `X[`$n-1$`]`. Each input has at least one out-neighbor.

* The other $s$ vertices are known as _gates_. Each gate is labeled with $\wedge$, $\vee$ or $\neg$. Gates labeled with $\wedge$ (_AND_) or $\vee$ (_OR_) have two in-neighbors. Gates labeled with $\neg$ (_NOT_) have one in-neighbor. We will allow parallel edges.^[Having parallel edges means an AND or OR gate $u$ can have both its in-neighbors be the same gate $v$. Since $AND(a,a)=OR(a,a)=a$ for every $a\in \{0,1\}$, such parallel edges don't help in computing new values in circuits with AND/OR/NOT gates. However, we will see circuits with more general sets of gates later on.]

* Exactly $m$ of the gates are also labeled with the $m$ labels   `Y[`$0$`]`, $\ldots$, `Y[`$m-1$`]` (in addition to their label $\wedge$/$\vee$/$\neg$). These are known as _outputs_.

The _size_ of a Boolean circuit is the number $s$ of gates it contains.
:::



::: { .pause }
This is a non-trivial mathematical definition, so it is worth taking the time to read it slowly and carefully. As in all mathematical definitions, we are using a known mathematical object --- a directed acyclic graph (DAG) --- to define a new object, a Boolean circuit.
This might be a good time to review some of the basic properties of DAGs and in particular the fact that they can be _topologically sorted_, see [topsortsec](){.ref}.
:::

If $C$ is a circuit with $n$ inputs and $m$ outputs, and $x\in \{0,1\}^n$, then we can compute the output of $C$ on the input $x$ in the natural way: assign the input vertices `X[`$0$`]`, $\ldots$, `X[`$n-1$`]` the values $x_0,\ldots,x_{n-1}$,  apply each gate on the values of its in-neighbors, and then output the values that correspond to the output vertices.
Formally, this is defined as follows:

::: {.definition title="Computing a function via a Boolean circuit" #circuitcomputedef}
Let $C$ be a Boolean circuit with $n$ inputs and $m$ outputs.
For every $x\in \{0,1\}^n$, the _output_ of $C$ on the input $x$, denoted by $C(x)$, is defined as the result of the following process:


We let $h:V \rightarrow \N$ be the _minimal layering_ of $C$ (aka _topological sorting_, see [minimallayeruniquethm](){.ref}).
We let $L$ be the maximum layer of $h$, and for $\ell=0,1,\ldots,L$  we do the following:

* For every $v$ in the $\ell$-th layer (i.e., $v$ such that $h(v)=\ell$) do:

  - If $v$ is an input vertex labeled with `X[`$i$`]` for some $i\in [n]$, then we assign to $v$ the value $x_i$.

  - If $v$ is a gate vertex labeled with $\wedge$ and with two in-neighbors $u,w$ then we assign to $v$ the _AND_ of the values assigned to $u$ and $w$. (Since $u$ and $w$ are in-neighbors of $v$, they are in a lower layer than $v$, and hence their values have already been assigned.)

  - If $v$ is a gate vertex labeled with $\vee$ and with two in-neighbors $u,w$ then we assign to $v$ the OR of the values assigned to $u$ and $w$.

  - If $v$ is a gate vertex labeled with $\neg$ and with one in-neighbor $u$ then we assign to $v$ the negation of the value assigned to $u$.

* The result of this process is the value $y\in \{0,1\}^m$ such that for every $j\in [m]$, $y_j$ is the value assigned to the vertex with label `Y[`$j$`]`.

Let $f:\{0,1\}^n \rightarrow \{0,1\}^m$. We say that the circuit $C$ _computes_ $f$ if for every $x\in \{0,1\}^n$, $C(x)=f(x)$.
:::


::: {.remark title="Boolean circuits nitpicks (optional)" #booleancircuitsremarks}
In phrasing [booleancircdef](){.ref}, we've made some technical choices that are not very important, but will be convenient for us later on.
Having parallel edges means an AND or OR gate $u$ can have both its in-neighbors be the same gate $v$.
Since $AND(a,a)=OR(a,a)=a$ for every $a\in \{0,1\}$, such parallel edges don't help in computing new values in circuits with AND/OR/NOT gates.
However, we will see circuits with more general sets of gates later on.
The condition that every input vertex has at least one out-neighbor is also not very important because we can always add "dummy gates" that touch
these inputs. However, it is convenient since it guarantees that (since every gate has at most two in-neighbors) the number of inputs
in a circuit is never larger than twice its size.
:::


## 3.4 直线程序 { #starightlineprogramsec }

We have seen two ways to describe how to compute a function $f$ using _AND_, _OR_ and _NOT_:


* A _Boolean circuit_, defined in [booleancircdef](){.ref},  computes $f$ by connecting via wires _AND_, _OR_, and _NOT_ gates to the inputs.

* We can also describe such a computation using a _straight-line program_ that has lines of the form `foo = AND(bar,blah)`, `foo = OR(bar,blah)` and `foo = NOT(bar)` where `foo`, `bar` and `blah` are variable names. (We call this a _straight-line program_ since it contains no loops or branching (e.g., if/then) statements.)

To make the second definition more precise, we will now define a _programming language_ that is equivalent to Boolean circuits.
We call this programming language the **AON-CIRC programming language** ("AON" stands for _AND_/_OR_/_NOT_; "CIRC" stands for _circuit_).

For example, the following is an AON-CIRC program that on input $x \in \{0,1\}^2$, outputs $\overline{x_0 \wedge x_1}$ (i.e., the $NOT$ operation applied to $AND(x_0,x_1)$:

```python
temp = AND(X[0],X[1])
Y[0] = NOT(temp)
```

AON-CIRC is not a practical programming language: it was designed for pedagogical purposes only, as a way to model computation as the composition of $AND$, $OR$, and $NOT$.
However, it can still be easily implemented on a computer.


Given this example, you might already be able to guess how to write a program for computing (for example) $x_0 \wedge \overline{x_1 \vee x_2}$, and in general how to translate a Boolean circuit into an AON-CIRC program.
However, since we will want to prove mathematical statements about AON-CIRC programs, we will need to precisely define the AON-CIRC programming language.
Precise specifications of programming languages can sometimes be long and tedious,^[For example the [C programming language specification](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1124.pdf) takes more than 500 pages.] but are crucial for secure and reliable implementations.
Luckily, the AON-CIRC programming language is simple enough that we can define it formally with relatively little pain.



### Specification of the AON-CIRC programming language

An AON-CIRC program is a sequence of strings, which we call "lines", satisfying the following conditions: 


* Every line has one of the following forms: `foo = AND(bar,baz)`,  `foo = OR(bar,baz)`, or  `foo = NOT(bar)` where `foo`, `bar` and `baz` are _variable identifiers_. (We follow the common [programming languages convention](https://goo.gl/QyHa3b)  of using names such as `foo`, `bar`, `baz` as stand-ins for generic identifiers.) The line `foo = AND(bar,baz)` corresponds to the operation of assigning to the variable `foo` the logical AND of the values of the variables `bar` and `baz`. Similarly  `foo = OR(bar,baz)` and `foo = NOT(bar)` correspond to the logical OR and logical NOT operations. 


* A _variable identifier_ in the AON-CIRC programming language can be any combination of letters, numbers,  underscores, and brackets. There are two special types of variables:
  - Variables of the form `X[`$i$`]`, with $i \in \{0,1,\ldots, n-1\}$  are known as _input_ variables.
  - Variables of the form `Y[`$j$`]` are known as _output_ variables.

* A valid AON-CIRC program $P$ includes input variables of the form `X[`$0$`]`,$\ldots$,`X[`$n-1$`]` and output variables of the form `Y[`$0$`]`,$\ldots$, `Y[`$m-1$`]` where $n,m$ are natural numbers. We say that $n$ is the number of _inputs_ of the program $P$ and $m$ is the number of outputs. 

* In a valid AON-CIRC program, in every line the variables on the right-hand side of the assignment operator must either be input variables or variables that have already been assigned a value in a previous line.


* If $P$ is a valid AON-CIRC program of $n$ inputs and $m$ outputs, then for every $x\in \{0,1\}^n$ the _output_ of $P$ on input $x$ is the string $y\in \{0,1\}^m$ defined as follows:
  - Initialize the input variables `X[`$0$`]`,$\ldots$,`X[`$n-1$`]` to the values $x_0,\ldots,x_{n-1}$
  - Run the operator lines of $P$ one by one in order, in each line assigning to the variable on the left-hand side of the assignment operators the value of the operation on the right-hand side.
  - Let $y\in \{0,1\}^m$ be the values of the output variables `Y[`$0$`]`,$\ldots$, `Y[`$m-1$`]` at the end of the execution.

* We denote the output of $P$ on input $x$ by $P(x)$.

* The _size_ of an AON circ program $P$ is the number of lines it contains. (The reader might note that this corresponds to our definition of the size of a circuit as the number of gates it contains.)


Now that we formally specified AON-CIRC programs, we can define what it means for an AON-CIRC program $P$ to compute a function $f$:

::: {.definition title="Computing a function via AON-CIRC programs" #AONcircdef}
Let $f:\{0,1\}^n \rightarrow\{0,1\}^m$, and $P$ be a valid AON-CIRC program with $n$ inputs and $m$ outputs.
We say that _$P$ computes $f$_ if $P(x)=f(x)$ for every $x\in \{0,1\}^n$.
:::

The following solved exercise gives an example of an AON-CIRC program.

::: {.solvedexercise title="" #aonforcmpsolved}
Consider the following function $CMP:\{0,1\}^4 \rightarrow \{0,1\}$ that on four input bits $a,b,c,d\in \{0,1\}$, outputs $1$ iff the number represented by $(a,b)$ is larger than the number represented by $(c,d)$.
That is $CMP(a,b,c,d)=1$ iff $2a+b>2c+d$.

Write an AON-CIRC program to compute $CMP$.
:::

::: {.solution data-ref="aonforcmpsolved"}
Writing such a program is tedious but not truly hard.
To compare two numbers we first compare their most significant digit, and then go down to the next digit and so on and so forth.
In this case where the numbers have just two binary digits, these comparisons are particularly simple.
The number represented by $(a,b)$ is larger than the number represented by $(c,d)$ if and only if one of the following conditions happens:

1. The most significant bit $a$ of $(a,b)$   is larger than the most significant bit $c$ of $(c,d)$.

or

2. The two most significant bits $a$ and $c$ are equal, but $b>d$.


Another way to express the same condition is the following:
the number $(a,b)$ is larger than $(c,d)$ iff  $a>c$ __OR__ ($a\ge c$ __AND__ $b>d$).

For binary digits $\alpha,\beta$, the condition $\alpha>\beta$ is simply that $\alpha=1$ and $\beta=0$ or $AND(\alpha,NOT(\beta))=1$, and the condition $\alpha\ge\beta$ is simply $OR(\alpha, NOT(\beta))=1$.
Together these observations can be used to give the following AON-CIRC program to compute $CMP$:

```python
# Compute CMP:{0,1}^4-->{0,1}
# CMP(X)=1 iff 2X[0]+X[1] > 2X[2] + X[3]
temp_1 = NOT(X[2])
temp_2 = AND(X[0],temp_1)
temp_3 = OR(X[0],temp_1)
temp_4 = NOT(X[3])
temp_5 = AND(X[1],temp_4)
temp_6 = AND(temp_5,temp_3)
Y[0] = OR(temp_2,temp_6)
```

We can also present this 8-line program as a circuit with 8 gates, see [aoncmpfig](){.ref}.
:::


![A circuit for computing the $CMP$ function. The evaluation of this circuit on $(1,1,1,0)$ yields the output $1$, since the number $3$ (represented in binary as $11$) is larger than the number $2$ (represented in binary as $10$).](./images/chapter3/comparecircuit.png){#aoncmpfig .margin}



### Proving equivalence of AON-CIRC programs and Boolean circuits


We now formally prove that AON-CIRC programs and Boolean circuits have exactly the same power:

> ### {.theorem title="Equivalence of circuits and straight-line programs" #slcircuitequivthm}
Let $f:\{0,1\}^n \rightarrow \{0,1\}^m$ and $s \geq m$ be some number. Then $f$ is computable by a Boolean circuit with $s$ gates  if and only if $f$ is computable by an AON-CIRC program of $s$ lines.



> ### {.proofidea data-ref="slcircuitequivthm"}
The idea is simple - AON-CIRC programs and Boolean circuits are just different ways of describing the exact same computational process.
For example, an _AND_ gate in a Boolean circuit corresponds to computing the _AND_ of two previously-computed values.
In an AON-CIRC program this will correspond to the line that stores in a variable the `AND` of two previously-computed variables.

::: { .pause }
This proof of [slcircuitequivthm](){.ref} is simple at heart, but all the details it contains can make it a little cumbersome to read. You might be better off trying to work it out yourself before reading it.
Our  [GitHub repository](https://github.com/boazbk/tcscode)  contains a  "proof by Python" of [slcircuitequivthm](){.ref}: implementation of functions `circuit2prog` and `prog2circuits` mapping Boolean circuits to AON-CIRC programs
and vice versa.
:::

::: {.proof data-ref="slcircuitequivthm"}
Let $f:\{0,1\}^n \rightarrow \{0,1\}^m$. Since the theorem is an "if and only if" statement, to prove it we need to show both directions: translating an AON-CIRC program that computes $f$ into a circuit that computes $f$, and translating a circuit that computes $f$ into an AON-CIRC program that does so.

We start with the first direction. Let $P$ be an AON-CIRC program that computes $f$. We define a circuit $C$ as follows: the circuit will have $n$ inputs and $s$ gates. For every $i \in [s]$, if the $i$-th operator line has the form `foo = AND(bar,blah)` then the $i$-th gate in the circuit will be an AND gate that is connected to gates $j$ and $k$ where $j$ and $k$ correspond to the last lines before $i$ where the variables `bar` and `blah` (respectively) were written to. (For example, if $i=57$ and the last line `bar` was written to is $35$ and the last line `blah` was written to is $17$ then the two in-neighbors of gate $57$ will be gates $35$ and $17$.)
If either `bar` or `blah` is an input variable then we connect the gate to the corresponding input vertex instead.
If `foo` is an output variable of the form `Y[`$j$`]` then we add the same label to the corresponding gate to mark it as an output gate.
We do the analogous operations if the $i$-th line involves an `OR` or a `NOT` operation (except that we use the corresponding _OR_ or _NOT_ gate, and in the latter case have only one in-neighbor instead of two).
For every input $x\in \{0,1\}^n$, if we run the program $P$ on $x$, then the value written that is computed in the $i$-th line is exactly the value that will be assigned to the $i$-th gate if we evaluate the circuit $C$ on $x$. Hence $C(x)=P(x)$ for every $x\in \{0,1\}^n$.

For the other direction, let $C$ be a circuit of $s$ gates and $n$ inputs that computes the function $f$. We sort the gates according to a topological order and write them as $v_0,\ldots,v_{s-1}$.
We now can create a program $P$ of $s$ operator lines as follows.
For every $i\in [s]$, if $v_i$ is an AND gate with in-neighbors  $v_j,v_k$ then we will add a line to $P$ of the form `temp_`$i$ ` = AND(temp_`$j$`,temp_`$k$`)`, unless one of the vertices is an input vertex or an output gate, in which case we change this to the form `X[.]` or `Y[.]` appropriately.
Because we work in topological ordering, we are guaranteed that the in-neighbors $v_j$ and $v_k$ correspond to variables that have already been assigned a value.
We do the same for OR and NOT gates.
Once again, one can verify that for every input $x$, the value $P(x)$ will equal $C(x)$ and hence the program computes the same function as the circuit.
(Note that since $C$ is a valid circuit, per [booleancircdef](){.ref}, every input vertex of $C$ has at least one out-neighbor and there are exactly $m$ output gates labeled $0,\ldots,m-1$;
hence all the variables  `X[0]`, $\ldots$, `X[`$n-1$`]` and `Y[0]` ,$\ldots$, `Y[`$m-1$`]` will appear in the  program $P$.)
:::


![Two equivalent descriptions of the same AND/OR/NOT computation as both an AON program and a Boolean circuit.](./images/chapter3/aoncircequiv.png){#aoncircequivfig .margin  }


## 3.5 Physical implementations of computing devices (digression) {#physicalimplementationsec }


_Computation_ is an abstract notion that is distinct from its physical _implementations_.
While most modern computing devices are obtained by mapping logical gates to semiconductor-based transistors, throughout history people have computed using a huge variety of mechanisms,  including mechanical systems, gas and liquid (known as _fluidics_), biological and chemical processes, and even living creatures (e.g., see [crabfig](){.ref} or  [this video](https://www.youtube.com/watch?v=czk4xgdhdY4) for how crabs or slime mold can be used to do computations).


In this section we will review some of these implementations, both so you can get an appreciation of how it is possible to directly translate Boolean circuits to the physical world, without going through the entire stack of architecture, operating systems, and compilers, as well as to emphasize that silicon-based processors are by no means the only way to perform computation.
Indeed, as we will see in [quantumchap](){.ref}, a very exciting recent line of work involves using different media for computation that would allow us to take advantage of _quantum mechanical effects_ to enable different types of algorithms.

![Crab-based logic gates from the paper "Robust soldier-crab ball gate" by Gunji, Nishiyama and Adamatzky. This is an example of an AND gate that relies on the tendency of two swarms of crabs arriving from different directions to combine to a single swarm that continues in the average of the directions.](./images/chapter3/crab-gate.jpg){#crabfig .margin}

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Such a cool way to explain logic gates. <a href="https://t.co/6Wgu2ZKFCx">pic.twitter.com/6Wgu2ZKFCx</a></p>&mdash; Lionel Page (\@page_eco) <a href="https://twitter.com/page_eco/status/1188749430020698112?ref_src=twsrc%5Etfw">October 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### Transistors

A _transistor_ can be thought of as an electric circuit with two inputs, known as the _source_ and the _gate_ and an output, known as the _sink_.
The gate controls whether current flows from the source to the sink.
In a _standard transistor_, if the gate is "ON" then current can flow from the source to the sink and if it is "OFF" then it can't.
In a _complementary transistor_ this is reversed: if the gate is "OFF" then current can flow from the source to the sink and if it is "ON" then it can't.

![We can implement the logic of transistors using water. The water pressure from the gate closes or opens a faucet between the source and the sink.](./images/chapter3/transistor_water.png){#transistor-water-fig .margin  }

There are several ways to implement the logic of a transistor.
For example, we can use faucets to implement it using water pressure (e.g. [transistor-water-fig](){.ref}). This might seem as merely a curiosity, but there is a field known as [fluidics](https://en.wikipedia.org/wiki/Fluidics) concerned with implementing logical operations using liquids or gasses. Some of the motivations include operating in extreme environmental conditions such as in space or a battlefield, where standard electronic equipment would not survive.

The standard implementations of transistors use electrical current.
One of the original implementations used   _vacuum tubes_.
As its name implies, a vacuum tube is a tube containing nothing (i.e., a vacuum) and where a priori electrons could freely flow from the source (a wire) to the sink (a plate). However, there is a gate (a grid)  between the two, where modulating its voltage can block the flow of electrons.

Early vacuum tubes were roughly the size of lightbulbs (and looked very much like them too).
In the 1950's they were supplanted by _transistors_, which implement the same logic using _semiconductors_ which are materials that normally do not conduct electricity but whose conductivity can be modified and controlled by inserting impurities ("doping") and applying an external electric field (this is known as the _field effect_).
In the 1960's computers started to be implemented using _integrated circuits_ which enabled much greater density.
In 1965, Gordon Moore predicted that the number of transistors per integrated circuit would double every year (see [moorefig](){.ref}), and that this would lead to "such wonders as home computers —or at least terminals connected to a central computer— automatic controls for automobiles, and personal portable communications equipment".
Since then, (adjusted versions of) this so-called "Moore's law" have been running strong, though exponential growth cannot be sustained forever, and some physical limitations are already [becoming apparent](http://www.nature.com/news/the-chips-are-down-for-moore-s-law-1.19338).

![The number of transistors per integrated circuit from 1959 till 1965 and a prediction that exponential growth will continue for at least another decade. Figure taken from "Cramming More Components onto Integrated Circuits", Gordon Moore, 1965](./images/chapter3/gordon_moore.png){#moorefig .margin  }

![Cartoon from Gordon Moore's article "predicting" the implications of radically improving transistor density.](./images/chapter3/moore_cartoon.png){#moore-cartoon-fig .margin  }

![The exponential growth in computing power over the last 120 years. Graph by Steve Jurvetson, extending a prior graph of Ray Kurzweil.](./images/chapter3/1200px-Moore's_Law_over_120_Years.png){#kurzweil-fig .margin  }




### Logical gates from transistors

We can use transistors to implement various Boolean functions such as $AND$, $OR$, and $NOT$.
For each two-input gate $G:\{0,1\}^2 \rightarrow \{0,1\}$,  such an implementation would be a system with two input wires $x,y$ and one output wire $z$, such that if we identify high voltage with "$1$" and low voltage with "$0$", then the wire  $z$ will be equal to "$1$" if and only if applying $G$ to the values of the wires $x$ and $y$ is $1$ (see [logicgatestransistorsfig](){.ref} and [transistor-nand-fig](){.ref}).
This means that if there exists a AND/OR/NOT circuit to compute a function $g:\{0,1\}^n \rightarrow \{0,1\}^m$, then we can compute $g$ in the physical world using transistors as well.

![Implementing logical gates using transistors. Figure taken from [Rory Mangles' website](http://www.northdownfarm.co.uk/rory/tim/basiclogic.htm).](./images/chapter3/dtl_logic.png){#logicgatestransistorsfig   .margin  }

![Implementing a NAND gate  (see [3.6节](#nandsec){.ref}) using transistors.](./images/chapter3/nand_transistor.png){#transistor-nand-fig .margin  }





### Biological computing

Computation can be based on [biological or chemical systems](http://www.nature.com/nrg/journal/v13/n7/full/nrg3197.html).
For example the [_lac_ operon](https://en.wikipedia.org/wiki/Lac_operon) produces the enzymes needed to digest lactose only if the conditions $x \wedge (\neg y)$ hold where $x$ is "lactose is present" and $y$ is "glucose is present".
Researchers have managed to [create transistors](http://science.sciencemag.org/content/340/6132/554?iss=6132), and from them  logic gates, based on DNA molecules (see also [transcriptorfig](){.ref}).
Projects such as the [Cello programming language](https://www.cidarlab.org/cello) enable converting Boolean circuits into DNA sequences that encode operations that can be executed in bacterial cells, see [this video](https://youtu.be/-1fqgrF7fXU). 
One motivation for DNA computing is to achieve increased parallelism or storage density; another is to create "smart biological agents" that could perhaps be injected into bodies, replicate themselves, and fix or kill cells that were damaged by a disease such as cancer.
Computing in biological systems is not restricted, of course, to DNA:
even larger systems such as [flocks of birds](https://www.cs.princeton.edu/~chazelle/pubs/cacm12-natalg.pdf) can be considered as computational processes.

![Performance of DNA-based logic gates. Figure taken from paper of [Bonnet et al](http://science.sciencemag.org/content/early/2013/03/27/science.1232758.full), Science, 2013.](./images/chapter3/transcriptor.jpg){#transcriptorfig .margin  }

### Cellular automata and the game of life

_Cellular automata_ is a model of a system composed of a sequence of _cells_, each of which can have a finite state.
At each step, a cell updates its state based on the states of its _neighboring cells_ and some simple rules.
As we will discuss later in this book (see [cellularautomatasec](){.ref}), cellular automata such as Conway's "Game of Life" can be used to simulate computation gates.

![An AND gate using a "Game of Life" configuration. Figure taken from [Jean-Philippe Rennard's paper](http://www.rennard.org/alife/CollisionBasedRennard.pdf).](./images/chapter3/game_of_life_and.png){#gameoflifefig .margin  }


### Neural networks

One computation device that we all carry with us is our own _brain_.
Brains have served humanity throughout history, doing computations that range from distinguishing prey from predators, through making scientific discoveries and artistic masterpieces, to composing witty 280 character messages.
The exact working of the brain is still not fully understood, but one common mathematical model for it is a (very large) _neural network_.

A neural network can be thought of as a Boolean circuit that instead of $AND$/$OR$/$NOT$ uses some other gates as the basic basis.
For example, one particular basis we can use are _threshold gates_.
For every vector $w= (w_0,\ldots,w_{k-1})$ of integers and integer $t$ (some or all of which could be negative),
the _threshold function corresponding to $w,t$_ is the function
$T_{w,t}:\{0,1\}^k \rightarrow \{0,1\}$ that maps $x\in \{0,1\}^k$ to $1$ if and only if $\sum_{i=0}^{k-1} w_i x_i \geq t$.
For example, the threshold function $T_{w,t}$ corresponding to $w=(1,1,1,1,1)$ and $t=3$ is simply the majority function $MAJ_5$ on $\{0,1\}^5$.
Threshold gates can be thought of as an approximation for _neuron cells_ that make up the core of human and animal brains. To a first approximation, a neuron has $k$ inputs and a single output, and the neuron "fires" or "turns on" its output when those signals pass some threshold.

Many machine learning algorithms use _artificial neural networks_ whose purpose is not to imitate biology but rather to perform some computational tasks, and hence are not restricted to a threshold or other biologically-inspired gates.
Generally, a neural network is often described as operating on signals that are real numbers, rather than $0/1$ values, and where the output of a gate on inputs $x_0,\ldots,x_{k-1}$ is obtained by applying $f(\sum_i w_i x_i)$ where $f:\R \rightarrow \R$ is an [activation function](https://goo.gl/p9izfA) such as rectified linear unit (ReLU), Sigmoid, or many others (see [activationfunctionsfig](){.ref}).
However, for the purposes of our discussion, all of the above are equivalent (see also [NANDsfromActivationfunctionex](){.ref}).
In particular we can reduce the setting of real inputs to binary inputs by representing a real number in the binary basis, and multiplying the weight of the bit corresponding to the $i^{th}$ digit by $2^i$.

![Common activation functions used in Neural Networks, including rectified linear units (ReLU), sigmoids, and hyperbolic tangent. All of those can be thought of as continuous approximations to simplify the step function. All of these can be used to compute the NAND gate (see [NANDsfromActivationfunctionex](){.ref}). This property enables neural networks to (approximately) compute any function that can be computed by a Boolean circuit.](./images/chapter3/activationfuncs.png){#activationfunctionsfig .margin }


### A computer made from marbles and pipes

We can implement computation using many other physical media, without any electronic, biological, or chemical components. Many suggestions for _mechanical_ computers have been put forward, going back at least to Gottfried Leibniz's computing machines from the 1670s and Charles Babbage's 1837 plan for a mechanical ["Analytical Engine"](https://en.wikipedia.org/wiki/Analytical_Engine).
As one example, [marblefig](){.ref} shows a simple implementation of a NAND (negation of AND, see [3.6节](#nandsec){.ref}) gate using marbles going through pipes. We represent a logical value in $\{0,1\}$ by a pair of pipes, such that there is a marble flowing through exactly one of the pipes.
We call one of the pipes the "$0$ pipe" and the other the "$1$ pipe", and so the identity of the pipe containing the marble determines the logical value.
A NAND gate corresponds to a mechanical object with two pairs of incoming pipes and one pair of outgoing pipes, such that for every $a,b \in \{0,1\}$, if two marbles are rolling toward the object in the $a$ pipe of the first pair and the $b$ pipe of the second pair, then a marble will roll out of the object in the $NAND(a,b)$-pipe of the outgoing pair.
In fact, there is even a commercially-available educational game that uses marbles as a basis of computing, see [turingtumblefig](){.ref}.



![A physical implementation of a NAND gate using marbles. Each wire in a Boolean circuit is modeled by a pair of pipes representing the values $0$ and $1$ respectively, and hence a gate has four input pipes (two for each logical input) and two output pipes. If one of the input pipes representing the value $0$ has a marble in it then that marble will flow to the output pipe representing the value $1$. (The dashed line represents a gadget that will ensure that at most one marble is allowed to flow onward in the pipe.) If both the input pipes representing the value $1$ have marbles in them, then the first marble will be stuck but the second one will flow onwards to the output pipe representing the value $0$.](./images/chapter3/marble.png){#marblefig .margin  }

![A "gadget" in a pipe that ensures that at most one marble can pass through it. The first marble that passes causes the barrier to lift and block new ones.](./images/chapter3/gadget.png){#gadgetfig .margin  }

![The game ["Turing Tumble"](https://www.turingtumble.com/) contains an implementation of logical gates using marbles.](./images/chapter3/turingtumble.png){#turingtumblefig .margin  }




## 3.6 The NAND function { #nandsec }

The $NAND$ function is another simple function that is extremely useful for defining computation.
It is the function mapping $\{0,1\}^2$ to $\{0,1\}$ defined by:

$$NAND(a,b) = \begin{cases} 0 & a=b=1 \\ 1 & \text{otherwise} \end{cases}\;.$$

As its name implies, $NAND$ is the NOT of AND (i.e., $NAND(a,b)= NOT(AND(a,b))$), and so we can clearly compute $NAND$ using $AND$  and $NOT$.
Interestingly, the opposite direction holds as well:

> ### {.theorem title="NAND computes AND,OR,NOT" #univnandonethm}
We can compute $AND$, $OR$, and $NOT$ by composing only the $NAND$ function.

> ### {.proof data-ref="univnandonethm"}
We start with the following observation. For every $a\in \{0,1\}$, $AND(a,a)=a$. Hence, $NAND(a,a)=NOT(AND(a,a))=NOT(a)$.
This means that $NAND$ can compute $NOT$.
By the principle of "double negation",  $AND(a,b)=NOT(NOT(AND(a,b)))$, and hence we can use $NAND$ to compute $AND$ as well.
Once we can compute $AND$ and $NOT$, we can compute $OR$ using ["De Morgan's Law"](https://goo.gl/TH86dH):  $OR(a,b)=NOT(AND(NOT(a),NOT(b)))$ (which can also be written as $a \vee b = \overline{\overline{a} \wedge \overline{b}}$) for every $a,b \in \{0,1\}$.

> ### { .pause }
[univnandonethm](){.ref}'s proof is very simple, but you should make sure that __(i)__ you understand the statement of the theorem, and __(ii)__ you follow its proof. In particular, you should make sure you understand why De Morgan's law is true.

We can use $NAND$ to compute many other functions, as demonstrated in the following exercise.

> ### {.solvedexercise title="Compute majority with NAND" #majbynandex}
Let $MAJ: \{0,1\}^3 \rightarrow \{0,1\}$ be the function that on input $a,b,c$ outputs $1$ iff $a+b+c \geq 2$. Show how to compute $MAJ$ using a composition of $NAND$'s.

::: {.solution data-ref="majbynandex"}
Recall that {{eqref: eqmajandornot}} states that

$$
MAJ(x_0,x_1,x_2) = OR\left(\, AND(x_0,x_1)\;,\; OR \bigl( AND(x_1,x_2) \;,\; AND(x_0,x_2) \bigr) \, \right) \;. {{numeq}}{eqmajandornotrestated}
$$

We can use [univnandonethm](){.ref}  to replace all the occurrences of $AND$ and $OR$   with $NAND$'s.
Specifically, we can use the equivalence $AND(a,b)=NOT(NAND(a,b))$, $OR(a,b)=NAND(NOT(a),NOT(b))$, and $NOT(a)=NAND(a,a)$ to replace the right-hand side of
{{eqref: eqmajandornotrestated}} with an expression involving only $NAND$, yielding that $MAJ(a,b,c)$ is equivalent to the (somewhat unwieldy) expression

$$
\begin{gathered}
NAND \biggl(\, NAND\Bigl(\, NAND\bigl(NAND(a,b),NAND(a,c)\bigr), \\
NAND\bigl(NAND(a,b),NAND(a,c)\bigr)\, \Bigr),\\
NAND(b,c) \, \biggr)
\end{gathered}
$$

The same formula can also be expressed as a circuit with NAND gates, see [majnandcircfig](){.ref}.
:::

![A circuit with NAND gates to compute the Majority function on three bits](./images/chapter3/majfromnand.png){#majnandcircfig .margin  }  





### NAND Circuits

We define _NAND Circuits_ as circuits in which all the gates are NAND operations.
Such a circuit again corresponds to a directed acyclic graph (DAG) since all the gates correspond to the same function (i.e., NAND), we do not even need to label them, and all gates have in-degree exactly two.
Despite their simplicity, NAND circuits can be quite powerful.


::: {.example title="$NAND$ circuit for $XOR$" #xornandexample}
Recall the $XOR$ function which maps $x_0,x_1 \in \{0,1\}$ to $x_0 + x_1 \mod 2$.
We have seen in [xoraonexample](){.ref} that we can compute $XOR$ using $AND$, $OR$, and $NOT$, and so by [univnandonethm](){.ref} we can compute it using only $NAND$'s.
However, the  following is a direct construction of computing $XOR$ by a sequence of NAND operations:

1. Let $u = NAND(x_0,x_1)$.
2. Let $v = NAND(x_0,u)$
3. Let $w = NAND(x_1,u)$.
4. The $XOR$ of $x_0$ and $x_1$ is $y_0 = NAND(v,w)$.

One can verify that this algorithm does indeed compute $XOR$ by enumerating all the four choices for $x_0,x_1 \in \{0,1\}$.
We can also represent this algorithm graphically as a circuit, see [cornandcircfig](){.ref}.
:::


![A circuit with NAND gates to compute the XOR of two bits.](./images/chapter3/nandcircxor.png){#cornandcircfig .margin  }  

In fact, we can show the following theorem:

> ### {.theorem title="NAND is a universal operation" #NANDuniversamthm}
For every Boolean circuit $C$ of $s$ gates, there exists a NAND circuit $C'$ of at most $3s$ gates that computes the same function as $C$.

> ### {.proofidea data-ref="NANDuniversamthm"}
The idea of the proof is to just replace every $AND$, $OR$ and $NOT$ gate with their NAND implementation following the proof of [univnandonethm](){.ref}.

::: {.proof data-ref="NANDuniversamthm"}
If $C$ is a Boolean circuit, then since, as we've seen in the proof of  [univnandonethm](){.ref},  for every $a,b \in \{0,1\}$

* $NOT(a) = NAND(a,a)$

* $AND(a,b) = NAND(NAND(a,b),NAND(a,b))$

* $OR(a,b) = NAND(NAND(a,a),NAND(b,b))$

we can replace every gate of $C$ with at most three $NAND$ gates to obtain an equivalent circuit $C'$. The resulting circuit will have at most $3s$ gates.
:::

::: { .bigidea #equivalencemodels }
Two models are _equivalent in power_ if they can be used to compute the same set of functions.
:::



### More examples of NAND circuits (optional)

Here are some more sophisticated examples of NAND circuits:


__Incrementing integers.__ Consider the task of computing, given as input a string $x\in \{0,1\}^n$ that represents a natural number $X\in \N$, the representation of $X+1$.
That is, we want to compute the function $INC_n:\{0,1\}^n \rightarrow \{0,1\}^{n+1}$ such that for every $x_0,\ldots,x_{n-1}$, $INC_n(x)=y$  which satisfies $\sum_{i=0}^n y_i 2^i = \left( \sum_{i=0}^{n-1} x_i 2^i \right)+1$. (For simplicity of notation, in this example we use the representation where the least significant digit is first rather than last.)

The increment operation can be very informally described as follows: _"Add $1$ to the least significant bit and propagate the carry"_.
A little more precisely, in the case of the binary representation, to obtain the increment of $x$, we scan $x$ from the least significant bit onwards, and flip all $1$'s to $0$'s until we encounter a bit equal to $0$, in which case we flip it to $1$ and stop.


Thus we can compute the increment of $x_0,\ldots,x_{n-1}$ by doing the following:


``` {.algorithm title="Compute Increment Function" #incrementalg}
INPUT: $x_0,x_1,\ldots,x_{n-1}$ representing the number $\sum_{i=0}^{n-1} x_i\cdot 2^i$ # we use LSB-first representation
OUTPUT:$y \in \{0,1\}^{n+1}$ such that $\sum_{i=0}^n y_i \cdot 2^i =  \sum_{i=0}^{n-1} x_i\cdot 2^i + 1$

Let $c_0 \leftarrow 1$ # we pretend we have a "carry" of $1$ initially
For{$i=0,\ldots, n-1$}
Let $y_i \leftarrow XOR(x_i,c_i)$.
If{$c_i=x_i=1$}
$c_{i+1}=1$
else
$c_{i+1}=0$
endif
Endfor
Let $y_n \leftarrow c_n$.
```


[incrementalg](){.ref} describes precisely how to compute the increment operation, and can be easily transformed into _Python_ code that performs the same computation, but it does not seem to directly yield a NAND circuit to compute this.
However, we can transform this algorithm line by line to a NAND circuit.
For example, since for every $a$, $NAND(a,NOT(a))=1$, we can replace the initial statement $c_0=1$ with $c_0 = NAND(x_0,NAND(x_0,x_0))$.
We already know how to compute $XOR$ using NAND and so we can use this to implement the operation $y_i \leftarrow XOR(x_i,c_i)$.
Similarly, we can write the "if" statement as saying $c_{i+1} \leftarrow AND(c_i,x_i)$,  or in other words $c_{i+1} \leftarrow  NAND(NAND(c_i,x_i),NAND(c_i,x_i))$.
Finally, the assignment $y_n = c_n$ can be written as $y_n = NAND(NAND(c_n,c_n),NAND(c_n,c_n))$.
Combining these observations yields for every $n\in \N$, a $NAND$ circuit to compute $INC_n$.
For example, [nandincrememntcircfig](){.ref} shows what this circuit looks like for $n=4$.



![NAND circuit with computing the _increment_ function on $4$ bits.](./images/chapter3/incrementfromnand.png){#nandincrememntcircfig  .margin }




__From increment to addition.__
Once we have the increment operation, we can certainly compute addition by repeatedly incrementing (i.e., compute $x+y$ by performing $INC(x)$ $y$ times).
However, that would be quite inefficient and unnecessary.
With the same idea of keeping track of carries we can implement the "grade-school" addition algorithm and compute the function $ADD_n:\{0,1\}^{2n} \rightarrow \{0,1\}^{n+1}$ that on input $x\in \{0,1\}^{2n}$ outputs the binary representation of the sum of the numbers represented by $x_0,\ldots,x_{n-1}$ and $x_{n},\ldots,x_{2n-1}$:


``` {.algorithm title="Addition using NAND" #additionfromnand}
INPUT: $u \in \{0,1\}^n$, $v\in \{0,1\}^n$ representing numbers in LSB-first binary representation.
OUTPUT: LSB-first binary representation of $x+y$.

Let $c_0 \leftarrow 0$
For{$i=0,\ldots,n-1$}
    Let $y_i \leftarrow u_i + v_i \mod 2$
    If{$u_i + v_i + c_i \geq 2$}
    $c_{i+1}\leftarrow 1$
    else
    $c_{i+1} \leftarrow 0$
    endif
Endfor
Let $y_n \leftarrow c_n$
```


Once again, [additionfromnand](){.ref} can be translated into a NAND circuit.
The crucial observation is that the "if/then" statement simply corresponds to
$c_{i+1} \leftarrow MAJ_3(u_i,v_i,v_i)$ and we have seen in [majbynandex](){.ref} that the function $MAJ_3:\{0,1\}^3 \rightarrow \{0,1\}$ can be computed using $NAND$s.



### The NAND-CIRC Programming language { #nandcircsec }

Just like we did for Boolean circuits, we can define a programming-language analog of NAND circuits.
It is even simpler than the AON-CIRC language since we only have a single operation.
We define the _NAND-CIRC Programming Language_ to be a programming language where every line (apart from the input/output declaration) has the following form:

```python
foo = NAND(bar,blah)
```

where `foo`, `bar` and `blah` are variable identifiers.

::: {.example title="Our first NAND-CIRC program" #NANDprogramexample}
Here is an example of a NAND-CIRC program:

```python
u = NAND(X[0],X[1])
v = NAND(X[0],u)
w = NAND(X[1],u)
Y[0] = NAND(v,w)
```
:::



> ### { .pause }
Do you know what function this program computes? Hint: you have seen it before.

Formally, just like we did in [AONcircdef](){.ref} for AON-CIRC, we can define the notion of computation by a NAND-CIRC program in the natural way:


::: {.definition title="Computing by a NAND-CIRC program" #NANDcomp}
Let $f:\{0,1\}^n \rightarrow \{0,1\}^m$ be some function, and let $P$ be a NAND-CIRC program. We say that $P$ _computes_ the function $f$ if:

1. $P$ has $n$ input variables `X[`$0$`]`$,\ldots,$`X[`$n-1$`]` and $m$ output variables `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]`.

2. For every $x\in \{0,1\}^n$, if we execute $P$ when we assign to `X[`$0$`]`$,\ldots,$`X[`$n-1$`]` the values $x_0,\ldots,x_{n-1}$, then at the end of the execution, the output variables `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]` have the values $y_0,\ldots,y_{m-1}$ where $y=f(x)$.
:::

As before we can show that NAND circuits are equivalent to NAND-CIRC programs (see [progandcircfig](){.ref}):

> ### {.theorem title="NAND circuits and straight-line program equivalence" #NANDcircslequivthm}
For every $f:\{0,1\}^n \rightarrow \{0,1\}^m$ and $s \geq m$, $f$ is computable by a NAND-CIRC program of $s$ lines if and only if $f$ is computable by a NAND circuit of $s$ gates.


![A NAND program and the corresponding circuit. Note how every line in the program corresponds to a gate in the circuit.](./images/chapter3/nandcircuitequiv.png){#progandcircfig   .margin  }


We omit the proof of [NANDcircslequivthm](){.ref} since it follows along exactly the same lines as the equivalence of Boolean circuits and AON-CIRC program  ([slcircuitequivthm](){.ref}).
Given [NANDcircslequivthm](){.ref} and [NANDuniversamthm](){.ref}, we know that we can translate every $s$-line AON-CIRC program $P$ into an equivalent NAND-CIRC program of at most $3s$ lines.
In fact, this translation can be easily done by replacing every line of the form `foo = AND(bar,blah)`, `foo = OR(bar,blah)` or `foo = NOT(bar)` with the equivalent 1-3 lines that use the `NAND` operation.
Our [GitHub repository](https://github.com/boazbk/tcscode) contains a "proof by code": a simple Python program `AON2NAND` that transforms an AON-CIRC into an equivalent NAND-CIRC program.



> ### {.remark title="Is the NAND-CIRC programming language Turing Complete? (optional note)" #NANDturingcompleteness}
You might have heard of a term called "Turing Complete" that is sometimes used to describe programming languages. (If you haven't, feel free to ignore the rest of this remark: we define this term precisely in [chapequivalentmodels](){.ref}.)
If so, you might wonder if the NAND-CIRC programming language has this property.
The answer is __no__, or perhaps more accurately, the term "Turing Completeness" is not really applicable for the NAND-CIRC programming language.
The reason is that, by design, the NAND-CIRC programming language can only compute _finite_ functions $F:\{0,1\}^n \rightarrow \{0,1\}^m$ that take a fixed number of input bits and produce a fixed number of outputs bits.
The term "Turing Complete" is only applicable to programming languages for _infinite_ functions that can take inputs of arbitrary length.
We will come back to this distinction later on in this book.

## 3.7 Equivalence of all these models


If we put together [slcircuitequivthm](){.ref}, [NANDuniversamthm](){.ref}, and [NANDcircslequivthm](){.ref}, we obtain the following result:

::: {.theorem title="Equivalence between models of finite computation" #equivalencemodelsthm}
For every sufficiently large $s,n,m$  and $f:\{0,1\}^n \rightarrow \{0,1\}^m$, the following conditions are all equivalent to one another:

* $f$ can be computed by a Boolean circuit (with $\wedge,\vee,\neg$ gates) of at most $O(s)$   gates.

* $f$ can be computed by an AON-CIRC straight-line program of at most $O(s)$ lines.

* $f$ can be computed by a NAND circuit of at most $O(s)$ gates.

* $f$ can be computed by a NAND-CIRC straight-line program of at most $O(s)$ lines.
:::

By "$O(s)$" we mean that the bound is at most $c\cdot s$ where $c$ is a constant that is independent of $n$.
For example, if $f$ can be computed by a Boolean circuit of $s$ gates, then it can be computed by a NAND-CIRC program of at most $3s$ lines, and if $f$ can be computed by a NAND circuit of $s$ gates, then it can be computed by an AON-CIRC program of at most $2s$ lines.




> ### {.proofidea data-ref="equivalencemodelsthm"}
We omit the formal proof, which is obtained by combining [slcircuitequivthm](){.ref}, [NANDuniversamthm](){.ref}, and [NANDcircslequivthm](){.ref}. The key observation is that the results we have seen allow us to translate a program/circuit that computes $f$ in one of the above models into a program/circuit that computes $f$ in another model by increasing the lines/gates by at most a constant factor (in fact this constant factor is at most $3$).


[slcircuitequivthm](){.ref} is a special case of a more general result.
We can consider even more general models of computation, where instead of AND/OR/NOT or NAND, we use other operations (see [othergatessec](){.ref} below).
It turns out that Boolean circuits are equivalent in power to such models as well.
The fact that all these different ways to define computation lead to equivalent models shows that we are "on the right track".
It justifies the seemingly arbitrary choices that we've made of using AND/OR/NOT or NAND as our basic operations, since these choices do not affect the power of our computational model.
Equivalence results such as [equivalencemodelsthm](){.ref} mean that we can easily translate between Boolean circuits, NAND circuits, NAND-CIRC programs and the like.
We will use this ability later on in this book, often shifting to the most convenient formulation without making a big deal about it.
Hence we will not worry too much about the distinction between, for example, Boolean circuits and NAND-CIRC programs.



In contrast, we will continue to take special care to distinguish between _circuits/programs_ and _functions_ (recall [functionprogramidea](){.ref}).
A function corresponds to a _specification_ of a computational task, and it is a fundamentally different object than a program or a circuit, which corresponds to the _implementation_ of the task.


### Circuits with other gate sets  {#othergatessec }

There is nothing special about AND/OR/NOT or NAND. For every set of functions $\mathcal{G} = \{ G_0,\ldots,G_{k-1} \}$, we can define a notion of circuits that use elements of  $\mathcal{G}$ as gates, and a notion of a "$\mathcal{G}$ programming language" where every line involves assigning to a variable `foo` the result of applying some $G_i \in \mathcal{G}$ to previously defined or input variables.
Specifically, we can make the following definition:

::: {.definition title="General straight-line programs" #genstraight-lineprogs}
Let $\mathcal{F} = \{ f_0,\ldots, f_{t-1} \}$ be a finite collection of Boolean functions, such that
$f_i:\{0,1\}^{k_i} \rightarrow \{0,1\}$ for some $k_i \in \N$.
An _$\mathcal{F}$ program_ is a sequence of lines, each of which assigns to some variable the result of applying some $f_i \in \mathcal{F}$ to $k_i$ other variables. As above, we use `X[`$i$`]` and `Y[`$j$`]` to denote the input and output variables.

We say that $\mathcal{F}$ is a _universal set of operations_ (also known as a universal gate set) if there exists a $\mathcal{F}$ program to compute the function $NAND$.
:::

AON-CIRC programs correspond to $\{AND,OR,NOT\}$ programs, NAND-CIRC programs corresponds to $\mathcal{F}$ programs for the set  $\mathcal{F}$ that only contains the $NAND$ function,   but we can also define  $\{ IF, ZERO, ONE\}$ programs (see below), or use any other set.

We can also define _$\mathcal{F}$ circuits_, which will be directed graphs in which each _gate_ corresponds to applying a function $f_i \in \mathcal{F}$, and will each have $k_i$ incoming wires and a single outgoing wire. (If the function $f_i$ is not _symmetric_, in the sense that the order of its input matters then we need to label each wire entering a gate as to which parameter of the function it corresponds to.)
As in [slcircuitequivthm](){.ref}, we can show that $\mathcal{F}$ circuits and $\mathcal{F}$ programs are equivalent.
We have seen that for $\mathcal{F} = \{ AND,OR, NOT\}$, the resulting circuits/programs are equivalent in power to the NAND-CIRC programming language, as we can compute $NAND$ using $AND$/$OR$/$NOT$ and vice versa.
This turns out to be a special case of a general phenomenon — the _universality_ of $NAND$ and other gate sets — that we will explore more in-depth later in this book.

::: {.example title="IF,ZERO,ONE circuits" #IZOcircuits}
Let $\mathcal{F} = \{ IF , ZERO, ONE \}$ where $ZERO:\{0,1\} \rightarrow \{0\}$ and $ONE:\{0,1\} \rightarrow \{1\}$ are the constant zero and one functions,^[One can also define these functions as taking a length zero input. This makes no difference for the computational power of the model.] and $IF:\{0,1\}^3 \rightarrow \{0,1\}$ is the function that on input $(a,b,c)$ outputs $b$ if $a=1$ and $c$ otherwise.
Then $\mathcal{F}$ is universal.

Indeed, we can demonstrate that $\{ IF, ZERO, ONE \}$ is universal using the following formula for $NAND$:

$$
NAND(a,b) = IF(a,IF(b,ZERO,ONE),ONE) \;.
$$
:::


There are also some sets $\mathcal{F}$ that are more restricted in power. For example it can be shown that if we use only AND or OR gates (without NOT) then we do _not_ get an equivalent model of computation.
The exercises cover several examples of universal and non-universal gate sets.


### Specification vs. implementation  (again) {#specvsimplrem}

![It is crucial to distinguish between the _specification_ of a computational task, namely _what_ is the function that is to be computed and the _implementation_ of it, namely the algorithm, program, or circuit that contains the instructions defining _how_ to map an input to an output. The same function could be computed in many different ways.](./images/chapter3/specvsimpl.png){#specvsimplfig }


As we discussed in [secimplvsspec](){.ref}, one of the most important distinctions in this book is that of _specification_ versus _implementation_ or separating "what" from "how" (see [specvsimplfig](){.ref}).
A _function_ corresponds to the _specification_ of a computational task, that is _what_ output should be produced for every particular input.
A _program_ (or circuit, or any other way to specify _algorithms_) corresponds to the _implementation_ of _how_ to compute the desired output from the input.
That is, a program is a set of instructions on how to compute the output from the input.
Even within the same computational model there can be many different ways to compute the same function.
For example, there is more than one NAND-CIRC program that computes the majority function, more than one Boolean circuit to compute the addition function, and so on and so forth.

Confusing specification and implementation  (or equivalently _functions_ and _programs_) is a common mistake, and one that is unfortunately encouraged by the common programming-language terminology of referring to parts of programs as "functions".
However, in both the theory and practice of computer science, it is important to maintain this distinction, and it is particularly important for us in this book.


> ### { .recap }
* An _algorithm_ is a recipe for performing a computation as a sequence of "elementary" or "simple" operations.
* One candidate definition for "elementary" operations is the set $AND$, $OR$ and $NOT$.
* Another candidate definition for an "elementary" operation is the $NAND$ operation. It is an operation that is easily implementable in the physical world in a variety of methods including by electronic transistors.
* We can use $NAND$ to compute many other functions, including majority, increment, and others.
* There are other equivalent choices, including the sets $\{AND,OR,NOT\}$ and $\{ IF, ZERO, ONE \}$.
* We can formally define the notion of a function $F:\{0,1\}^n \rightarrow \{0,1\}^m$ being computable using the _NAND-CIRC Programming language_.
* For every set of basic operations, the notions of being computable by a circuit and being computable by a straight-line program are equivalent.

## Exercises


::: {.exercise title="Compare $4$ bit numbers" #comparenumbersex}
Give a Boolean circuit (with AND/OR/NOT gates) that computes the function $CMP_8:\{0,1\}^8 \rightarrow \{0,1\}$ such that $CMP_8(a_0,a_1,a_2,a_3,b_0,b_1,b_2,b_3)=1$ if and only if the number represented by $a_0a_1a_2a_3$ is larger than the number represented by $b_0b_1b_2b_3$.
:::

::: {.exercise title="Compare $n$ bit numbers" #compareasymnumbersex}
Prove that there exists a constant $c$ such that for every $n$ there is a Boolean circuit (with AND/OR/NOT gates) $C$ of at most $c\cdot n$ gates  that computes the function $CMP_{2n}:\{0,1\}^{2n} \rightarrow \{0,1\}$ such that $CMP_{2n}(a_0\cdots a_{n-1} b_0 \cdots b_{n-1})=1$ if and only if the number represented by $a_0 \cdots a_{n-1}$ is larger than the number represented by $b_0 \cdots b_{n-1}$.
:::


::: {.exercise title="OR,NOT is universal" #ornotex}
Prove that the set $\{ OR, NOT \}$ is _universal_, in the sense that one can compute NAND using these gates.
:::

::: {.exercise title="AND,OR is not universal" #andorex}
Prove that for every $n$-bit input circuit $C$ that contains only AND and OR gates, as well as gates that compute the constant functions $0$ and $1$, $C$ is _monotone_, in the sense that if $x,x' \in \{0,1\}^n$, $x_i \leq x'_i$ for every $i\in [n]$, then $C(x) \leq C(x')$.

Conclude that the set $\{ AND, OR, 0, 1\}$ is _not_ universal.
:::


::: {.exercise title="XOR is not universal" #xorex}
Prove that for every $n$-bit input circuit $C$ that contains only XOR gates, as well as gates that compute the constant functions $0$ and $1$, $C$ is _affine or linear modulo two_, in the sense that there exists some $a\in \{0,1\}^n$ and $b\in \{0,1\}$ such that for every $x\in \{0,1\}^n$, $C(x) = \sum_{i=0}^{n-1}a_ix_i + b \mod 2$.

Conclude that the set $\{ XOR , 0 , 1\}$ is _not_ universal.
:::

::: {.exercise title="MAJ,NOT, 1 is universal" #majnotex}
Let $MAJ:\{0,1\}^3 \rightarrow \{0,1\}$ be the majority function.
Prove that $\{ MAJ,NOT, 1 \}$ is a universal set of gates.
:::


::: {.exercise title="MAJ,NOT  is not universal" #majnotextwo}
Prove that $\{ MAJ,NOT  \}$ is not a universal set. See footnote for hint.^[_Hint:_ Use the fact that $MAJ(\overline{a},\overline{b},\overline{c}) = \overline{MAJ(a,b,c)}$ to prove that every $f:\{0,1\}^n \rightarrow \{0,1\}$ computable by a circuit with only $MAJ$ and $NOT$ gates satisfies $f(0,0,\ldots,0) \neq f(1,1,\ldots,1)$. Thanks to Nathan Brunelle and David Evans for suggesting this exercise.]
:::

::: {.exercise title="NOR is universal" #norex}
Let $NOR:\{0,1\}^2 \rightarrow \{0,1\}$ defined as $NOR(a,b) = NOT(OR(a,b))$. Prove that $\{ NOR \}$ is a universal set of gates.
:::


::: {.exercise title="Lookup is universal" #lookupex}
Prove that $\{ LOOKUP_1,0,1 \}$ is a universal set of gates where $0$ and $1$ are the constant functions and $LOOKUP_1:\{0,1\}^3 \rightarrow \{0,1\}$ satisfies $LOOKUP_1(a,b,c)$ equals $a$ if $c=0$ and equals $b$ if $c=1$.
:::


> ### {.exercise title="Bound on universal basis size (challenge)" #universal-bound}
Prove that for every subset $B$ of the functions from $\{0,1\}^k$ to $\{0,1\}$,
if $B$ is universal then there is a $B$-circuit of at most $O(1)$ gates to compute the $NAND$ function (you can start by showing that there is a $B$ circuit of at most $O(k^{16})$ gates).^[Thanks to Alec Sun and Simon Fischer for comments on this problem.]


::: {.exercise title="Size and inputs / outputs" #nandcircsizeex}
Prove that for every NAND circuit of size $s$ with $n$ inputs and $m$ outputs, $s \geq \min \{ n/2 , m \}$. See footnote for hint.^[_Hint:_ Use the conditions of [booleancircdef](){.ref} stipulating that every input vertex has at least one out-neighbor and there are exactly $m$ output gates. See also [booleancircuitsremarks](){.ref}.]
:::



> ### {.exercise title="Threshold using NANDs" #threshold-nand-ex}
Prove that there is some constant $c$ such that for every $n>1$, and integers $a_0,\ldots,a_{n-1},b \in \{-2^n,-2^n+1,\ldots,-1,0,+1,\ldots,2^n\}$, there is a NAND circuit with at most $c n^4$ gates that computes the _threshold_ function $f_{a_0,\ldots,a_{n-1},b}:\{0,1\}^n \rightarrow \{0,1\}$ that on input $x\in \{0,1\}^n$ outputs $1$ if and only if $\sum_{i=0}^{n-1} a_i x_i > b$.

::: {.exercise title="NANDs from activation functions" #NANDsfromActivationfunctionex}
We say that a function $f:\mathbb{R}^2 \rightarrow \mathbb{R}$ is a _NAND approximator_ if it has the following property: for every $a,b \in \mathbb{R}$, if $\min\{|a|,|1-a|\}\leq 1/3$ and $\min \{ |b|,|1-b| \}\leq 0.1$ then $|f(a,b) - NAND(\lfloor a \rceil, \lfloor b \rceil)| \leq 0.1$ where we denote by $\lfloor x \rceil$ the integer closest to $x$. That is, if $a,b$ are within a distance $1/3$ to $\{0,1\}$ then we want $f(a,b)$ to equal the $NAND$ of the values in $\{0,1\}$ that are closest to $a$ and $b$ respectively. Otherwise, we do not care what the output of $f$ is on $a$ and $b$.

In this exercise you will show that you can construct a NAND approximator from many common activation functions used in deep neural networks. As a corollary you will obtain that deep neural networks can simulate NAND circuits. Since NAND circuits can also simulate deep neural networks, these two computational models are equivalent to one another.


1. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(DReLU(L'(a,b)))$ where $L':\mathbb{R}^2 \rightarrow \mathbb{R}$ is an _affine_ function (of the form $L'(a,b)=\alpha a + \beta b + \gamma$ for some $\alpha,\beta,\gamma \in \mathbb{R}$), $L$ is an affine function (of the form $L(y) = \alpha y + \beta$ for $\alpha,\beta \in \mathbb{R}$), and $DReLU:\mathbb{R} \rightarrow \mathbb{R}$, is the function defined as $DReLU(x) = \min(1,\max(0,x))$. Note that $DReLU(x) = 1-ReLU(1-ReLU(x))$  where $ReLU(x)=\max(x,0)$ is the rectified linear unit activation function.

2. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(sigmoid(L'(a,b)))$ where $L',L$ are affine as above and $sigmoid:\mathbb{R} \rightarrow \mathbb{R}$ is the function defined as $sigmoid(x) = e^x/(e^x+1)$.

3. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(tanh(L'(a,b)))$ where $L',L$ are affine as above and $tanh:\mathbb{R} \rightarrow \mathbb{R}$ is the function defined as $tanh(x) = (e^x-e^{-x})/(e^x+e^{-x})$.

4. Prove that for every NAND-circuit $C$ with $n$ inputs and one output that computes a function $g:\{0,1\}^n \rightarrow \{0,1\}$, if we replace every gate of $C$ with a NAND-approximator and then invoke the resulting circuit on some $x\in \{0,1\}^n$, the output will be a number $y$ such that $|y-g(x)|\leq 1/3$.
:::


::: {.exercise title="Majority with NANDs efficiently" #majwithNAND}
Prove that there is some constant $c$ such that for every $n>1$, there is a NAND circuit of at most $c\cdot n$ gates that computes the majority function on $n$ input bits $MAJ_n:\{0,1\}^n \rightarrow \{0,1\}$. That is $MAJ_n(x)=1$ iff $\sum_{i=0}^{n-1}x_i > n/2$. See footnote for hint.^[One approach to solve this is using recursion and analyzing it using the so called  "Master Theorem".]
:::

::: {.exercise title="Output at last layer" #outputlastlayer}
Prove that for every $f:\{0,1\}^n \rightarrow \{0,1\}$, if there is a Boolean circuit $C$ of $s$ gates
that computes $f$ then there is a Boolean circuit $C'$ of at most $s$ gates such that in the minimal layering of $C'$, the output gate of $C'$ is placed in the last layer. See footnote for hint.^[_Hint:_ Vertices in layers beyond the output can be safely removed without changing the functionality of the circuit.]
:::

## Biographical notes

The excerpt from Al-Khwarizmi's book is from "The Algebra of Ben-Musa", Fredric Rosen, 1831.


Charles Babbage (1791-1871) was a visionary scientist, mathematician, and inventor (see [@swade2002the, @collier2000charles]).
More than a century before the invention of modern electronic computers, Babbage realized that computation can be in principle mechanized.
His first design for a mechanical computer was the _difference engine_ that was designed to do polynomial interpolation.
He then designed the _analytical engine_ which was a much more general machine and the first prototype for a programmable general-purpose computer.
Unfortunately, Babbage was never able to complete the design of his prototypes.
One of the earliest people to realize the engine's potential and far-reaching implications was Ada Lovelace (see the notes for [chaploops](){.ref}).




Boolean algebra was first investigated by Boole and DeMorgan in the 1840's [@Boole1847mathematical, @DeMorgan1847].
The definition of Boolean circuits and connection to electrical relay circuits was given in Shannon's Masters Thesis [@Shannon1938].
(Howard Gardener called Shannon's thesis "possibly the most important, and also the most famous, master's thesis of the [20th] century".)
Savage's book [@Savage1998models], like this one, introduces the theory of computation starting with Boolean circuits as the first model.
Jukna's book [@Jukna12] contains a modern in-depth exposition of Boolean circuits, see also [@wegener1987complexity].


The NAND function was shown to be universal by Sheffer [@Sheffer1913], though this also appears in the earlier work of Peirce, see [@Burks1978charles].
Whitehead and Russell used NAND as the basis for their logic in their magnum opus _Principia Mathematica_ [@WhiteheadRussell1912].
In her Ph.D thesis, Ernst [@Ernst2009phd] investigates empirically the minimal NAND circuits for various functions.
Nisan and Shocken's book [@NisanShocken2005]  builds a computing system starting from NAND gates and ending with high-level programs and games ("NAND to Tetris"); see also the website [nandtotetris.org](https://www.nand2tetris.org/).

We defined the _size_ of a Boolean circuit in [booleancircdef](){.ref} to be the number of gates it contains. This is one of two conventions used in the literature. The other convention is to define the size as the number of _wires_ (equivalent to the number of gates plus the number of inputs).
This makes very little difference in almost all settings, but can affect the circuit size complexity of some "pathological examples" of functions such as the constant zero function that do not depend on much of their inputs.
