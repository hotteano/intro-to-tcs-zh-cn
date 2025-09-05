```admonish warning title = ""
❗**页面施工中**: 目前状态: 翻译中.
```

# 定义计算 {#compchap }

```admonish quote title = ""
*“没有理由不借助机器来节省脑力劳动和体力劳动。”* —— Charles Babbage，1852   

*“如果有谁不以我的例子为戒，而尝试并成功地用不同的原理或更简单的机械手段，构造出一台在自身中体现数学分析执行部门全部功能的机器，那么我丝毫不担心将我的声誉交付于他，因为唯有他能完全理解我努力的性质及其成果的价值。”* —— Charles Babbage，1864 

*“要理解一个程序，你必须既成为机器，又成为程序。”* —— Alan Perlis，1982
```

## 学习目标 {.objectives  }
* 理解计算可以被精确建模。  
* 学习 **布尔电路** / **直线程序** 的计算模型。  
* 电路与直线程序的等价性。  
* $\AND$/$\OR$/$\NOT$ 与 $\text{NAND}$ 的等价性。  
* 物理世界中的计算实例。 

## 目录

<!-- toc -->

```admonish quote title = ""
![#babbagewheels .margin](./images/chapter3/wheels_babbage.png)
Charles Babbage的计算轮。图片取自 Harvard Mark I 计算机的“操作手册”。
```
```admonish quote title = ""
![#markIcomp .margin](./images/chapter3/PopularMechanics1944smaller.jpg)
摘自 **Popular Mechanics** 上的一篇关于 Harvard Mark I 计算机的[文章](http://sites.harvard.edu/~chsi/markone/about.html), 1944 年.
```

几千年来，人类一直在进行计算，不仅依靠纸笔，还使用过算盘、计算尺、各种机械装置，直到现代的电子计算机。从先验的角度来看，**计算**这一概念似乎总是依赖于所使用的具体工具。例如，你也许会认为，在现代笔记本电脑上用 **Python** 实现的乘法算法，与用纸笔进行乘法运算时的“最佳”算法会有所不同。  

然而，正如我们在[引言](chapter_0.md)中所看到的，一个在渐近意义上更优的算法，无论底层技术如何，最终都会优于较差的算法。这让我们看到希望：可以找到一种**独立于技术**的方式来刻画计算的概念。  

本章正是要做这件事。我们将把“从输入计算输出”定义为一系列基本操作的应用（见[下图](#compchapwhatvshowfig)）。借助这一框架，我们便能精确地表述诸如：“函数 $f$ 可以由模型 $X$ 计算”或“函数 $f$ 可以由模型 $X$ 在 $s$ 步操作内计算完成”这样的命题。  

```admonish quote title = ""
<a id="compchapwhatvshowfig"> ![compchapwhatvshowfig](./images/chapter3/compchapterwhatvshow.png) </a>
一个将字符串映射到字符串的函数，**规定**了一项计算任务，也就是说，它描述了输入与输出之间所期望的关系。在本章中，我们将定义一些模型，用来**实现**这些计算过程，从而达到所需的关系，也就是描述**如何**根据输入来计算输出。我们将看到若干此类模型的例子，包括布尔电路和直线型编程语言。
```

```admonish info title = "不那么严谨的概述"
阅读本章, 我们希望读者能够有以下收获：

* 我们可以使用 **逻辑运算**，如 $\AND$(与)、$\OR$(或) 和 $\NOT$(非)，从输入计算输出（见 [3.2节](#andornotsec)）。

* **布尔电路** 是一种通过组合基本逻辑运算来计算更复杂函数的方法（见 [3.3节](#booleancircuitsec)）。  
  我们既可以将布尔电路看作一种数学模型（基于有向无环图），也可以将其视为现实世界中可实现的物理装置。实现方式多种多样，不仅包括基于硅的半导体，还包括机械甚至生物机制（见 [3.5节](#physicalimplementationsec)）。

* 我们还可以把布尔电路描述为 **直线型程序**，即不包含循环结构的程序（没有 `while` / `for` / `do .. until` 等）（见 [3.4节](#starightlineprogramsec)）。

* 可以通过 $\text{NAND}$ 运算来实现 $\AND$、$\OR$ 和 $\NOT$ 运算（反之亦然）。  
  这意味着带有 $\AND$/$\OR$/$\NOT$ 门的电路，与带有 $\text{NAND}$ 门的电路在计算能力上是**等价的**，我们可以根据需要选择其中任一模型来描述计算（见 [3.6节](#nandsec)）。  
  先提前剧透一下，在 [下一章](chapter_4.md) 中我们将看到，这类电路可以计算**所有有限函数**。

本章的一个“大思想”是 **模型之间的等价性**（见 [equivalencemodels](#equivalencemodels)）。如果两个计算模型能够计算相同集合的函数，那么它们就是**等价的**。布尔电路（$\AND$/$\OR$/$\NOT$ 门）与 $\text{NAND}$ 电路的等价性只是一个例子，本书中我们还会多次遇到类似的普遍现象。
```



## 3.1 定义计算

“算法”一词来源于对穆罕默德·伊本·穆萨·花剌子密(Muhammad ibn Musa al-Khwarizmi)名字的拉丁化转写。al-Khwarizmi 是九世纪的一位波斯学者，他的著作向西方世界介绍了十进位值制数字系统，以及一次方程与二次方程的解法（见 [下图](#alKhwarizmi)）。然而，以今天的标准来看，al-Khwarizmi 对算法的描述的形式化程度相当不足。他没有使用如 $x,y$ 这样的**变量**，而是采用具体的数字（如 10 和 39），并依赖读者从这些例子中自行类推出一般情况——这与当今儿童学习算法时的教学方式颇为相似。  

以下是 al-Khwarizmi 对解形如 $x^2 + bx = c$ 方程的算法的描述：

```admonish quote title = "如何解形如‘平方与根的和等于某数’的方程"
举例来说：“一个平方加上它的十倍平方根等于三十九迪拉姆。” 换句话说，求这样一个平方数：它加上它自身的十倍平方根，结果是三十九。  

解法如下：  
1. 将根的数量减半，本例中十的一半是五。  
2. 将这个数（五）平方，得到二十五。  
3. 将平方结果加到三十九上，得到六十四。  
4. 取六十四的平方根，得到八。  
5. 从平方根中减去根数量的一半（五），余数为三。  

因此，这个平方根为三，对应的平方为九。  
```

```admonish quote title = ""
<a id = "alKhwarizmi">![alKhwarizmi](./images/chapter3/alKhwarizmi.jpg)</a>
代数学手稿中的文字页，展示了解两类二次方程的几何解法。馆藏号：MS. Huntington 214, 页码 fol. 004v-005r
```
```admonish quote title = ""
<a id="childrenalg">![An explanation for children of the two digit addition algorithm](./images/chapter3/addition_regrouping.jpg)</a>

面向儿童的两位数加法算法讲解
```

为了本书的目的，我们需要一种更加精确的方式来描述算法。幸运（或者说不幸）的是，至少目前，计算机在从实例中学习方面远远落后于学龄儿童。因此，在 20 世纪，人们提出了用于精确描述算法的形式化语言，即 **编程语言**。  

下面是用 **Python** 转写的 al-Khwarizmi 二次方程求解算法：

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

{{defc}}{defofalg}[算法的非正式定义] **算法**是一组指令，用于通过执行一系列“基本步骤”从输入计算出输出。如果对于每一个输入 $x$，按照算法 $A$ 的指令操作都能得到输出 $F(x)$，则称算法 $A$ **计算**函数 $F$。

在本章中，我们将使用 **布尔电路（Boolean Circuits）** 模型，更精确而正式地定义算法。我们将展示，布尔电路在计算能力上等价于用“极简”编程语言编写的 **直线程序（straight line programs）**，即不包含循环的编程语言。我们还将看到，具体选择哪种 **基本运算（elementary operations）** 并不重要，不同的选择都可以得到计算能力等价的模型（见[下图](#compchapoverviewfig)）。然而，要理解这一点，我们需要一些时间。我们将从讨论什么是“基本运算”开始，并说明如何将算法的描述映射为实际物理过程，使其在现实世界中从输入生成输出。

```admonish quote title = ""
<a id="compchapoverviewfig"> ![An overview of the computational models defined in this chapter. We will show several equivalent ways to represent a recipe for performing a finite computation. Specifically we will show that we can model such a computation using either a _Boolean circuit_ or a _straight line program_, and these two representations are equivalent to one another. We will also show that we can choose as our basic operations either the set $\{ \AND , \OR , \NOT \}$ or the set $\{ \text{NAND} \}$ and these two choices are equivalent in power. By making the choice of whether to use circuits or programs, and whether to use   $\{ \AND , \OR , \NOT \}$ or  $\{ \text{NAND} \}$ we obtain four equivalent ways of modeling finite computation. Moreover, there are many other choices of sets of basic operations that are equivalent in power.](./images/chapter3/compcharoverview.png)</a>
本章定义的计算模型概览。我们将展示几种等价的方式来表示执行有限计算的“操作方法”。具体而言，我们将证明，可以使用 **布尔电路（Boolean circuit）** 或 **直线程序（straight line program）** 来表示这样的计算，且这两种表示方式在计算能力上是等价的。我们还将展示，作为基本运算，我们可以选择集合 $\{ \AND , \OR , \NOT \}$ 或集合 $\{ \text{NAND} \}$，这两种选择在计算能力上也是等价的。通过选择使用电路还是程序，以及选择 $\{ \AND , \OR , \NOT \}$ 还是 $\{ \text{NAND} \}$，我们可以得到四种等价的有限计算建模方法。此外，还有许多其他基本操作集合的选择，它们在计算能力上同样是等价的。
```

## 3.2 使用与, 或, 非进行计算 { #andornotsec }

算法的表示需要将一个**较为复杂**的计算分解为一系列**更简单**的步骤。这些步骤可以通过多种不同的方式来执行，包括：

* 在纸上书写符号。  
* 改变电线中的电流。  
* 蛋白质与 DNA 链结合。  
* 集体中的个体对刺激做出反应（例如，蜂群中的蜜蜂，市场中的交易者）。  

为了形式化地定义算法，我们尝试“化繁为简”，挑出组成算法的"最小单位", 例如下列一组简单逻辑函数:

* $\OR:\{0,1\}^2 \rightarrow \{0,1\}$ 定义为

$$\OR(a,b) = \begin{cases} 0 & a=b=0 \\ 1 & \text{otherwise} \end{cases}$$

* $\AND:\{0,1\}^2 \rightarrow \{0,1\}$ 定义为

$$\AND(a,b) = \begin{cases} 1 & a=b=1 \\ 0 & \text{otherwise} \end{cases}$$

* $\NOT:\{0,1\} \rightarrow \{0,1\}$ 定义为

$$\NOT(a) = \begin{cases} 0 & a = 1 \\ 1 & a = 0 \end{cases}$$

函数 $\AND$、$\OR$ 和 $\NOT$ 是逻辑学以及许多计算机系统中使用的基本逻辑运算符。在逻辑学中, $\AND(a,b)$ 表示为 $a \wedge b$，$\OR(a,b)$ 表示为 $a \vee b$，$\NOT(a)$ 表示为 $\overline{a}$ 或 $\neg a$，我们也将采用这种表示法。

每一个函数 $\AND, \OR, \NOT$ 都以一个或两个单比特作为输入，并输出一个单比特。尽管这些运算看起来相当基本, 然而，计算的威力正来源于**将这些简单的运算组合在一起**。


~~~admonish example title="例: 用 $\\\AND$,$\\\OR$ 和 $\\\NOT$ 写出多数函数 $\\\text{MAJ}$"
考虑函数 $\text{MAJ}:\{0,1\}^3 \rightarrow \{0,1\}$，其定义如下：

$$
\text{MAJ}(x) = \begin{cases}
1 & x_0 + x_1 + x_2 \geq 2 \\
0 & \text{otherwise}
\end{cases} \;.
$$

也就是说，对于每个 $x\in \{0,1\}^3$，当且仅当 $x$ 的三个元素中至少有两个等于 $1$ 时，$\text{MAJ}(x)=1$。你能用 $\AND$、$\OR$ 和 $\NOT$ 写出一个计算 $\text{MAJ}$ 的公式吗？（此处建议你先停下来自己推导公式。提示：虽然某些函数需要用到 $\NOT$，但计算 $\text{MAJ}$ 不需要使用它。）

我们先用文字重新表述 $\text{MAJ}(x)$：“当且仅当存在一对不同的元素 $i,j$，且 $x_i$ 和 $x_j$ 都等于 $1$ 时，$\text{MAJ}(x)=1$。”  
换句话说，$\text{MAJ}(x)=1$ 当且仅当 **$x_0=1$ 且 $x_1=1$**，**或** **$x_1=1$ 且 $x_2=1$**，**或** **$x_0=1$ 且 $x_2=1$**。

由于三个条件 $c_0, c_1, c_2$ 的 $\OR$ 可以写作 $\OR(c_0, \OR(c_1, c_2))$，我们可以将其翻译为如下公式：

$$
\text{MAJ}(x_0,x_1,x_2) = \OR\left(\AND(x_0,x_1),\OR \bigl(\AND(x_1,x_2),\AND(x_0,x_2)\bigr)\right). {{numeq}}{eqmajandornot}
$$

回想一下，我们也可以将 $\OR(a,b)$ 写作 $a \vee b$，将 $\AND(a,b)$ 写作 $a \wedge b$。使用这种符号表示，公式 {{eqref: eqmajandornot}} 也可以写作：

$$\text{MAJ}(x_0,x_1,x_2) = ((x_0 \wedge x_1) \vee (x_1 \wedge x_2)) \vee (x_0 \wedge x_2)\;.$$

我们也可以将公式 {{eqref:eqmajandornot}} 以“编程语言”的形式表示: 将其表达为一组指令，用于在给定基本操作 $\AND, \OR, \NOT$ 的情况下计算 $\text{MAJ}$：

```python
def MAJ(X[0],X[1],X[2]):
    firstpair  = AND(X[0],X[1])
    secondpair = AND(X[1],X[2])
    thirdpair  = AND(X[0],X[2])
    temp       = OR(secondpair,thirdpair)
    return OR(firstpair,temp)
```
<iframe src="https://trinket.io/embed/python/5ead2eab1b" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>
~~~

### 3.2.1 $\AND$ 和 $\OR$ 的一些性质

与标准的加法和乘法类似，函数 $\AND$ 和 $\OR$ 满足**交换律**：$a \vee b = b \vee a$ 和 $a \wedge b = b \wedge a$，以及**结合律**：$(a \vee b) \vee c = a \vee (b \vee c)$ 和 $(a \wedge b) \wedge c = a \wedge (b \wedge c)$。  

于是如同加法和乘法的情况，我们通常可以省略括号，将 $((a \vee b) \vee c) \vee d$ 写作 $a \vee b \vee c \vee d$，对更多项的 $\AND$ 和 $\OR$ 同理。  

它们还满足分配律的一种变体：

{{exec}}{distributivelaw}[$\AND$ 与 $\OR$ 满足分配律] 
证明：对于任意 $a,b,c \in \{0,1\}$，都有
$$
  a \wedge (b \vee c) = (a \wedge b) \vee (a \wedge c)。
$$


```admonish solution collapsible=true, title = "解答" 
我们可以通过枚举 $a,b,c \in \{0,1\}$ 的所有 $8$ 种可能取值来证明这一点，但它也可以直接从标准的分配律推导出来。  

假设我们将任意正整数视为“真”，将零视为“假”。那么对于每个数 $u,v \in \mathbb{N}$，$u+v$ 为正当且仅当 $u \vee v$ 为真，而 $u \cdot v$ 为正当且仅当 $u \wedge v$ 为真。  

这意味着对于每个 $a,b,c \in \{0,1\}$，表达式 $a \wedge (b \vee c)$ 为真当且仅当 $a \cdot (b+c)$ 为正，而表达式 $(a \wedge b) \vee (a \wedge c)$ 为真当且仅当 $a \cdot b + a \cdot c$ 为正。  

根据标准的分配律 $a \cdot (b+c) = a \cdot b + a \cdot c$，因此前者表达式为真当且仅当后者表达式为真。
```



### 3.2.2 扩展例子: 计算异或($\XOR$) {#xoraonexample}

让我们看看如何用方才的基本运算得到一种新运算。定义 $\XOR:\{0,1\}^2 \rightarrow \{0,1\}$ 为函数 $\XOR(a,b) = a + b \mod 2$。也就是说，$\XOR(0,0) = \XOR(1,1) = 0$，$\XOR(1,0) = \XOR(0,1) = 1$。  

我们指出, 可以仅使用 $\AND$、$\OR$ 和 $\NOT$ 来构造 $\XOR$。

```admonish pause title = "暂停一下"
像往常一样，一个很好的练习是在继续阅读之前，先尝试自己用 $\AND$、$\OR$ 和 $\NOT$ 算法推导出 $\XOR$ 的实现方法。
```

以下算法使用 $\AND$、$\OR$ 和 $\NOT$ 来计算 $\XOR$：


{{algc}}{XORfromAONalg}[用 $\AND$, $\OR$ 与 $\NOT$ 计算 $\XOR$]

$
  \begin{array}{l}
  \mathbf{Input:}\ a,b \in \{0,1\} \\
  \mathbf{Output:}\ \XOR(a,b) \\
  \hline
  \mathbf{Step 1:}\ w_1 \leftarrow \AND(a,b) \\
  \mathbf{Step 2:}\ w_2 \leftarrow \NOT(w_1) \\
  \mathbf{Step 3:}\ w_3 \leftarrow\OR(a,b) \\
  \mathbf{Step 4: return}\ \AND(w_2,w_3)
  \end{array}
$

{{lemc}}{alganalaysis}
对于每个 $a,b \in \{0,1\}$，在输入 $a,b$ 时, {{ref: XORfromAONalg}} 输出 $a + b \mod 2$。


```admonish proof collapsible=true, title = "证明"
对于任意 $a,b$，有 $\XOR(a,b)=1$ 当且仅当 $a$ 与 $b$ 不同。
令 $w1 = \AND(a,b)$, $w2 = \NOT(\AND(a,b))$, $w3 = \OR(a,b)$. 则在输入 $a,b \in \{0,1\}$ 时，{{ref: XORfromAONalg}} 输出  
$$
\AND(w2, w3)
$$ 
* 如果 $a=b=0$，则 $w3 = \OR(a,b) = 0$，因此输出为 $0$。

* 如果 $a=b=1$，则 $\AND(a,b) = 1$，所以 $w2 = \NOT(\AND(a,b)) = 0$，输出为 $0$。

* 如果 $a=1$ 且 $b=0$（或反之），则 $w3 = \OR(a,b) = 1$ 且 $w1 = \AND(a,b) = 0$，此时算法输出  
$$
\AND(\NOT(w1), w3) = 1.
$$
```

我们也可以用编程语言来描述 {{ref: XORfromAONalg}}. 特别, 以下是 $\XOR$ 函数的 **Python** 实现:

```python
def AND(a,b): return a*b
def OR(a,b):  return 1-(1-a)*(1-b)
def NOT(a):   return 1-a

def XOR(a,b):
    w1 = AND(a,b)
    w2 = NOT(w1)
    w3 = OR(a,b)
    return AND(w2,w3)

# 一个测试
print([f"XOR({a},{b})={XOR(a,b)}" for a in [0,1] for b in [0,1]])
# ['XOR(0,0)=0', 'XOR(0,1)=1', 'XOR(1,0)=1', 'XOR(1,1)=0']
```




{{exec}}{xorthreebits}[在三个输入上计算 $\XOR$]
定义 $\XOR_3:\{0,1\}^3\to\{0,1\}$ 为 $\XOR_3(a,b,c)=a+b+c\pmod 2$. 也就是说，当 $a+b+c$ 为奇数时 $\XOR_3(a,b,c)=1$，否则 $\XOR_3(a,b,c)=0$。证明可以仅用 $\AND$、$\OR$ 和 $\NOT$ 三种逻辑运算来计算 $\XOR_3$。你可以将其表示为公式、使用诸如 Python 的编程语言实现，或构造相应的布尔电路。

~~~admonish solution collapsible=true, title = "解答"
模 2 加法具有与通常加法相同的 **结合律** ($(a+b)+c=a+(b+c)$) 和 **交换律** ($a+b=b+a$)。  
这意味着，如果我们定义 $a \oplus b = a+b \pmod 2$，那么  
$$
\XOR_3(a,b,c) = (a \oplus b) \oplus c
$$
换句话说，  
$$
\XOR_3(a,b,c) = \XOR(\XOR(a,b),c) \;.
$$

由于我们已经知道如何仅用 $\AND$、$\OR$ 和 $\NOT$ 来计算 $\XOR$，因此可以将其组合起来，用同样的基本运算实现 $\XOR_3$。在 Python 中，这可以写作如下程序：

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

# 一个小测试
print([f"XOR3({a},{b},{c})={XOR3(a,b,c)}" for a in [0,1] for b in [0,1] for c in [0,1]])
# ['XOR3(0,0,0)=0', 'XOR3(0,0,1)=1', 'XOR3(0,1,0)=1', 'XOR3(0,1,1)=0', 'XOR3(1,0,0)=1', 'XOR3(1,0,1)=0', 'XOR3(1,1,0)=0', 'XOR3(1,1,1)=1']
```
<iframe src="https://trinket.io/embed/python/0e71e3fcaa" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>
~~~

```admonish pause title="暂停一下"
尝试将上述例子推广，构造一种对任意正整数 $n$ 都适用的方法，用不超过 $4n$ 个基本步骤计算函数 $ \XOR_n:\{0,1\}^n \rightarrow \{0,1\}$。  
这里每一“基本步骤”指的是对某个已知输出或先前计算得到的值，应用集合 $\{\AND,\OR,\NOT\}$ 中的某个布尔运算。
```


### 3.2.3 非正式地定义“基本运算”和“算法”

我们已经看到，通过组合应用 $ \AND $、$ \OR $ 和 $ \NOT $ 可以得到一些有趣的函数。这启发我们将 $ \AND $、$ \OR $ 和 $ \NOT $ 视为我们的**基本运算**，从而给出如下关于**算法**的定义：

{{defc}}{semidefofalg}[算法的半形式化定义]一个**算法**由一系列步骤组成，每一步的形式是：“通过将 $ \AND $、$ \OR $ 或 $ \NOT $ 应用于先前计算得到的值（假定输入也已计算得到），来计算一个新值”。若对于函数 $F$ 的任意输入 $x$，当我们将 $x$ 作为算法 $A$ 的输入时，其最后一步计算出的值为 $F(x)$，则称算法 $A$ **计算**了函数 $F$。

这一定义引出了若干值得关注的问题：

1. 首先，这一定义确实过于非正式。我们既没有精确说明每一步到底做了什么，也没有明确“将 $x$ 作为输入”究竟是什么意思。

2. 其次，选择 $ \AND $、$ \OR $ 或 $ \NOT $ 看起来相当任意。为什么不是 $ \XOR $ 和 $ \text{MAJ} $？为什么不允许加法和乘法这样的运算？又或者其他逻辑结构，例如 `if/then` 或 `while`？

3. 第三，我们是否确信该定义真的与实际计算有关？如果有人给出了这种算法的描述，我们是否真的能够在现实中用它来计算相应的函数？

```admonish pause title="暂停一下"
这些问题将在很大程度上引导我们接下来的章节。因此，建议你重新阅读上述非正式定义，并思考自己对这些问题的看法。
```

本书的很大一部分内容将致力于回答上述问题。我们将看到：

1. 我们可以把算法的定义完全形式化，从而为“算法 $A$ 计算函数 $f$”这样的表述赋予精确的数学含义。

2. 虽然选择 $ \AND $ / $ \OR $ / $ \NOT $ 看似任意，我们本可以选择其他函数，但实际上这种选择影响不大。我们会看到，即使改用加法和乘法，或者几乎任何可以合理视为基本步骤的操作，我们依然能够得到相同的计算能力。

3. 事实证明，我们确实可以在现实世界中计算这种基于 $ \AND $ / $ \OR $ / $ \NOT $ 的算法。首先，这样的算法定义清晰，因此人类可以用纸和笔逐步执行。其次，这种计算可以通过多种方式**机械化**。我们已经看到，可以编写 Python 程序来对应执行这样的指令序列。而实际上，还可以通过被称为**晶体管**的元件，用电子信号直接实现 $ \AND $、$ \OR $ 和 $ \NOT $ 等操作。这正是现代电子计算机的工作方式。

在本章余下的内容以及本书后续部分，我们将开始回答这些问题。我们会看到更多简单操作组合出复杂操作的实例，包括加法、乘法、排序等。同时，我们还会讨论如何通过多种技术**物理实现** $ \AND $、$ \OR $ 和 $ \NOT $ 等基本操作。

## 3.3 布尔电路  {#booleancircuitsec }

```admonish quote title = ""
![logicgatesfig](./images/chapter3/logicgates.png)
逻辑运算或“门”的标准符号包括 $ \AND $、$ \OR $、$ \NOT $，以及在[3.6节](#nandsec)中讨论的 $ \text{NAND} $ 运算。
```
```admonish quote title = ""
<a id="smallandornotcircxorfig">![smallandornotcircxorfig](./images/chapter3/xorcircuitschemdraw.png)</a>
一个由 $ \AND $、$ \OR $ 和 $ \NOT $ 门构成的，用于计算 $ \XOR $ 函数的电路。
```

**布尔电路**提供了“组合基本运算”的精确定义。一个布尔电路（参见[下图](#boolancircfig)）由**门**和**输入**组成，并通过**导线**连接。  

**导线**传递的信号表示值 $0$ 或 $1$, 每个门对应 $\OR$、$\AND$ 或 $\NOT$ 运算. 一个 $\OR$ 门有两条输入导线和一条或多条输出导线, 如果这两条输入导线的信号分别为 $a$ 和 $b$（$a,b \in \{0,1\}$），则输出导线上的信号为 $\OR(a,b)$. $\AND$ 和 $\NOT$ 门的定义类似。  

**输入端**只有输出导线。如果我们将某个输入设为 $a \in \{0,1\}$，则该值会沿其所有输出导线传播. 我们还将一些门指定为**输出门**，其值对应于电路的计算结果. 例如，[上图](#smallandornotcircxorfig) 给出了一个用于计算 $\XOR$ 函数的电路，参考 [节3.2.2](#xoraonexample)。

对于一个 $n$ 输入的布尔电路 $C$，我们在输入端放置 $x \in \{0,1\}^n$ 的比特，然后沿导线传播信号，直到到达输出端，从而完成电路的计算，参见 [下图](boolancircfig)。

```admonish remark title="布尔电路的物理电路模拟" 
<a id= "booleancircimprem"></a>

布尔电路是一种 **数学模型**，不一定直接对应于物理对象，但它们可以被物理电路模拟。  

在电路中，信号通常通过导线上的电位（**电压**）来[表示](https://goo.gl/gntTQE)。例如，高于某一电压水平被解释为逻辑值 $1$，低于某一电压水平被解释为逻辑值 $0$。  

[3.5节](#physicalimplementationsec) 讨论了布尔电路的物理实现，包括使用电信号（如硅基电路）、生物实现以及机械实现的实例。
```

```admonish quote title=""
<a id="boolancircfig">![boolancircfig](./images/chapter3/booleancircuit.png)</a>
一个**布尔电路**由**门**组成，这些门通过**导线**彼此连接，并与**输入端**相连。  

左图显示了一个具有 $2$ 个输入和 $5$ 个门的电路，其中一个门被指定为输出门。  
右图展示了该电路在输入 $x \in \{0,1\}^2$（$x_0=1$，$x_1=0$）下的计算过程。  

每个门的值是通过对进入该门的导线上的值应用相应的函数（$\AND$、$\OR$ 或 $\NOT$）得到的。  
电路在给定输入下的输出为输出门的值。  

在此例中，该电路计算 $\XOR$ 函数，因此在输入 $10$ 下输出为 $1$。
```

{{exec}}{allequalex}[全相等函数]
定义函数 $ALLEQ:\{0,1\}^4 \rightarrow \{0,1\}$，其输入为 $x \in \{0,1\}^4$，当且仅当 $x_0 = x_1 = x_2 = x_3$ 时输出 $1$。 

```admonish solution collapsible=true, title="解答"
另一种描述函数 $ALLEQ$ 的方式是：当且仅当输入 $x \in \{0,1\}^4$ 满足 $x = 0^4$ 或 $x = 1^4$ 时，它输出 $1$。  
我们可以将条件 $x = 1^4$ 表述为 $x_0 \wedge x_1 \wedge x_2 \wedge x_3$，这可以用三个 $\AND$ 门计算。  
同样地，我们可以将条件 $x = 0^4$ 表述为 $\overline{x}_0 \wedge \overline{x}_1 \wedge \overline{x}_2 \wedge \overline{x}_3$，这可以用四个 $\NOT$ 门和三个 $\AND$ 门计算。  
$ALLEQ$ 的输出是这两个条件的 $\OR$，由此得到的电路包含 4 个 $\NOT$ 门、6 个 $\AND$ 门和 1 个 $\OR$ 门，如[下图](#allequalfig)所示。
```

```admonish quote title = ""
<a id="allequalfig"> ![A  Boolean circuit for computing the _all equal_ function $ALLEQ:\{0,1\}^4 \rightarrow \{0,1\}$ that outputs $1$ on $x\in \{0,1\}^4$ if and only if $x_0=x_1=x_2=x_3$.](./images/chapter3/allequalcirc2.png)</a>
一个用于计算 **all equal** 函数 $ALLEQ:\{0,1\}^4 \rightarrow \{0,1\}$ 的布尔电路。当且仅当 $x \in \{0,1\}^4$ 满足 $x_0 = x_1 = x_2 = x_3$ 时，它输出 $1$。
```


### 3.3.1 布尔电路: 形式化定义

我们之前非正式地将布尔电路定义为通过导线连接 $\AND$、$\OR$ 和 $\NOT$ 门，从输入生成输出的电路。  
然而，为了能够证明关于计算各种函数的布尔电路存在性或非存在性的定理，我们需要：

1. 将布尔电路作为**数学对象**进行形式化定义。  
2. 正式定义电路 $C$ **计算**函数 $f$ 的含义。

接下来我们将进行这一定义。我们把布尔电路定义为带标记的**有向无环图（DAG）**。图的**顶点**对应电路的门和输入端，图的**边**对应导线。电路中从输入或门 $u$ 到门 $v$ 的导线对应顶点间的有向边。输入顶点没有入边，而每个门根据其计算的函数具有适当数量的入边（即 $\AND$ 和 $\OR$ 门有两个入邻居，$\NOT$ 门有一个入邻居）。  

正式定义如下（参见[下图](#generalcircuitfig)）：

```admonish quote title=""
<a id="generalcircuitfig">![A _Boolean Circuit_ is a labeled directed acyclic graph (DAG). It has $n$ _input_ vertices, which are marked with `X[`$0$`]`,$\ldots$, `X[`$n-1$`]` and have no incoming edges, and the rest of the vertices are _gates_. _AND_, _OR_, and _NOT_ gates have two, two, and one incoming edges, respectively. If the circuit has $m$ outputs, then $m$ of the gates are known as _outputs_ and are marked with `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]`. When we evaluate a circuit $C$ on an input $x\in \{0,1\}^n$, we start by setting the value of the input vertices to $x_0,\ldots,x_{n-1}$ and then propagate the values, assigning to each gate $g$ the result of applying the operation of $g$ to the values of $g$'s in-neighbors. The output of the circuit is the value assigned to the output gates.](./images/chapter3/generalcircuit.png)</a>
**布尔电路** 是一个带标记的有向无环图 (DAG)。它有 $n$ 个 **输入** 顶点，这些顶点标记为 `X[`$0$`]`，$\ldots$，`X[`$n-1$`]`，且没有入边，其余顶点为 **门**。  
$\AND$、$\OR$ 和 $\NOT$ 门分别有两个、两个和一个入边。若电路有 $m$ 个输出，则 $m$ 个门被称为 **输出**，标记为 `Y[`$0$`]`，$\ldots$，`Y[`$m-1$`]`。  

在对输入 $x \in \{0,1\}^n$ 评估电路 $C$ 时，我们首先将输入顶点的值设置为 $x_0,\ldots,x_{n-1}$，然后将值向下传播，将每个门 $g$ 的值设置为对 $g$ 的入邻居的值应用 $g$ 的操作的结果。电路的输出即为分配给输出门的值。
```

{{defc}}{booleancircdef}[布尔电路]
设 $n,m,s$ 为正整数，且 $s \geq m$。一个具有 $n$ 个输入、$m$ 个输出和 $s$ 个门的**布尔电路**是一个带标记的有向无环图（DAG） $G=(V,E)$，其顶点数为 $s+n$，满足以下性质：

* 恰好有 $n$ 个顶点没有入邻居。这些顶点称为**输入端**，标记为 $X[0]$, $\ldots$, $X[n-1]$。每个输入端至少有一个出邻居。

* 其余 $s$ 个顶点称为**门**。每个门标记为 $\wedge$、$\vee$ 或 $\neg$。标记为 $\wedge$（$\AND$）或 $\vee$（$\OR$）的门有两个入邻居，标记为 $\neg$（$\NOT$）的门有一个入邻居。允许存在平行边。^[平行边意味着 AND 或 OR 门 $u$ 的两个入邻居可以是同一个门 $v$。由于对任意 $a \in \{0,1\}$ 有 $\AND(a,a)=\OR(a,a)=a$，在仅使用 AND/OR/NOT 门的电路中，这类平行边并不会计算出新的值。但在后面引入更一般门集合时，我们将看到平行边的用途。]

* 恰好有 $m$ 个门同时标记为 $Y[0]$, $\ldots$, $Y[m-1]$（除了其本来的 $\wedge$/$\vee$/$\neg$ 标记之外），称为**输出端**。

布尔电路的**规模**定义为其包含的门的数量 $s$。

```admonish pause
这是一个非平凡的数学定义，因此值得慢慢仔细阅读。  
正如所有数学定义一样，我们使用已知的数学对象——**有向无环图（DAG）**——来定义一个新的对象，即布尔电路。  

此时复习一些 DAG 的基本性质会很有帮助，特别是它们可以进行**拓扑排序**的事实，参见[1.6节](chapter_1.md#topsortsec)。
```

如果 $C$ 是一个具有 $n$ 个输入和 $m$ 个输出的电路，且 $x \in \{0,1\}^n$，则自然可以计算 $C$ 在输入 $x$ 下的输出：  
将输入顶点 $X[0]$, $\ldots$, $X[n-1]$ 赋值为 $x_0,\ldots,x_{n-1}$，然后对每个门应用其入邻居的值，最后输出对应于输出顶点的值。  

形式化定义如下：

{{defc}}{circuitcomputedef}[利用布尔电路计算函数]
设 $C$ 为一个具有 $n$ 个输入和 $m$ 个输出的布尔电路。  
对于每个 $x \in \{0,1\}^n$，$C$ 在输入 $x$ 上的 **输出**，记作 $C(x)$，定义为以下过程的结果：  

我们令 $h: V \rightarrow \N$ 为 $C$ 的 **最小分层**（又称 **拓扑排序**，见[定理1.22](chapter_1.md#thm:minilayerunique)）。  
令 $L$ 为 $h$ 的最大层数，对每个 $\ell=0,1,\ldots,L$，执行以下操作：

* 对每个位于第 $\ell$ 层的顶点 $v$（即 $v$ 满足 $h(v)=\ell$）执行：

  - 如果 $v$ 是输入顶点，标记为 `X[`$i$`]`，其中 $i \in [n]$，则将 $x_i$ 赋值给 $v$。

  - 如果 $v$ 是标记为 $\wedge$ 的门顶点，且有两个入邻居 $u,w$，则将 $u$ 和 $w$ 的值的 $\AND$ 赋给 $v$。（由于 $u$ 和 $w$ 是 $v$ 的入邻居，它们位于比 $v$ 更低的层，因此它们的值已经被赋值。）

  - 如果 $v$ 是标记为 $\vee$ 的门顶点，且有两个入邻居 $u,w$，则将 $u$ 和 $w$ 的值的 $\OR$ 赋给 $v$。

  - 如果 $v$ 是标记为 $\neg$ 的门顶点，且有一个入邻居 $u$，则将 $u$ 的值取反并赋给 $v$。

* 该过程的结果是一个 $y \in \{0,1\}^m$，其中对于每个 $j \in [m]$，$y_j$ 为标记为 `Y[`$j$`]` 的顶点的值。

设 $f: \{0,1\}^n \rightarrow \{0,1\}^m$，如果对于每个 $x \in \{0,1\}^n$，都有 $C(x) = f(x)$，则称电路 $C$ **计算** 函数 $f$。

```admonish remark title = "一些对布尔电路的吹毛求疵 (选读)"
<a id="#booleancircuitsremarks"></a>

在表述 {{ref: booleancircdef}} 时，我们做了一些技术性的选择，这些选择并不是非常重要，但对我们后续会很方便。  

允许存在平行边意味着一个 $\AND$ 或 $\OR$ 门 $u$ 可以让它的两个入邻居都是同一个门 $v$。  
由于对每个 $a \in \{0,1\}$ 都有 $\AND(a,a) = \OR(a,a) = a$，因此在仅使用 **$\AND/\OR/\NOT$** 门的电路中，这类平行边并不会带来新的计算值。  
然而，我们稍后会看到包含更一般门集合的电路。  

要求每个输入顶点至少有一个出邻居也不是特别重要，因为我们总可以添加“虚拟门”来使用这些输入。  
不过这个要求很方便，因为它保证了（由于每个门最多有两个入邻居）电路中的输入数量永远不会超过其规模的两倍。
```

## 3.4 直线程序 { #starightlineprogramsec }

我们已经看到两种使用 $\AND$、$\OR$ 和 $\NOT$ 来计算函数 $f$ 的方式：

* **布尔电路**，在 {{ref: booleancircdef}} 中定义，通过将 $\AND$、$\OR$ 和 $\NOT$ 门通过导线连接到输入来计算 $f$。

* 我们也可以使用 **直线程序** 来描述这样的计算，该程序的每一行形式为 `foo = AND(bar,blah)`、`foo = OR(bar,blah)` 和 `foo = NOT(bar)`，其中 `foo`、`bar` 和 `blah` 是变量名。（称其为 **直线程序**，因为它不包含循环或分支（例如 if/then）语句。）

为了更精确地描述第二种定义，我们现在定义一种与布尔电路等价的 **编程语言**。  
我们将这种编程语言称为 **AON-CIRC 编程语言**（"AON" 代表 $\AND/\OR/\NOT$；"CIRC" 代表 **circuit**）。

例如，以下是一个 AON-CIRC 程序，对于输入 $x \in \{0,1\}^2$，输出 $\overline{x_0 \wedge x_1}$（即对 $\AND(x_0,x_1)$ 应用 $\NOT$ 操作）：

```python
temp = AND(X[0],X[1])
Y[0] = NOT(temp)
```

AON-CIRC 并不是一种实用的编程语言：它仅用于教学目的，用来将计算建模为 $\AND$、$\OR$ 和 $\NOT$ 的组合。然而，它仍然可以很容易地在计算机上实现。

根据这个例子，你可能已经能够猜到如何编写程序来计算（例如）$x_0 \wedge \overline{x_1 \vee x_2}$，以及更一般地，如何将布尔电路翻译为 AON-CIRC 程序。但是，由于我们希望对 AON-CIRC 程序证明数学性质，我们需要精确定义 AON-CIRC 编程语言。  

编程语言的精确定义有时可能冗长且枯燥，例如，[C 语言规范](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1124.pdf) 就超过 500 页。但对于安全可靠的实现至关重要。幸运的是，AON-CIRC 编程语言足够简单，我们可以相对轻松地对其进行正式定义。

### 3.4.1 AON-CIRC 编程语言规范

一个 AON-CIRC 程序是一系列字符串，我们称之为“行”，满足以下条件：

* 每一行具有以下形式之一：`foo = AND(bar,baz)`、`foo = OR(bar,baz)` 或 `foo = NOT(bar)`，其中 `foo`、`bar` 和 `baz` 是 **变量标识符**。（我们遵循常见的 [编程语言惯例](https://goo.gl/QyHa3b)，使用 `foo`、`bar`、`baz` 等名称作为通用标识符的示例。）  
  行 `foo = AND(bar,baz)` 对应于将变量 `foo` 赋值为变量 `bar` 和 `baz` 的逻辑 $\AND$。类似地，`foo = OR(bar,baz)` 和 `foo = NOT(bar)` 分别对应逻辑 $\OR$ 和逻辑 $\NOT$ 操作。

* AON-CIRC 编程语言中的 **变量标识符** 可以由字母、数字、下划线和方括号的任意组合构成。有两类特殊变量：
  - 形式为 `X[`$i$`]` 的变量，其中 $i \in \{0,1,\ldots,n-1\}$，称为 **输入变量**。
  - 形式为 `Y[`$j$`]` 的变量，称为 **输出变量**。

* 一个有效的 AON-CIRC 程序 $P$ 包含输入变量 `X[`$0$`]`，$\ldots$，`X[`$n-1$`]` 和输出变量 `Y[`$0$`]`，$\ldots$，`Y[`$m-1$`]`，其中 $n,m$ 为自然数。我们称 $n$ 为程序 $P$ 的 **输入数**，$m$ 为 **输出数**。

* 在有效的 AON-CIRC 程序中，每一行右侧的变量必须是输入变量或在之前的行中已经被赋值的变量。

* 若 $P$ 是一个具有 $n$ 个输入和 $m$ 个输出的有效 AON-CIRC 程序，则对于每个 $x \in \{0,1\}^n$，程序 $P$ 在输入 $x$ 上的 **输出** 是字符串 $y \in \{0,1\}^m$，定义如下：
  - 将输入变量 `X[`$0$`]`，$\ldots$，`X[`$n-1$`]` 初始化为 $x_0,\ldots,x_{n-1}$。
  - 按顺序逐行执行 $P$ 的操作行，在每行中将左侧变量赋值为右侧操作的结果。
  - 执行结束后，令 $y \in \{0,1\}^m$ 为输出变量 `Y[`$0$`]`，$\ldots$，`Y[`$m-1$`]` 的值。

* 我们用 $P(x)$ 表示程序 $P$ 在输入 $x$ 上的输出。

* AON-CIRC 程序 $P$ 的 **规模** 是它包含的行数。（读者可能注意到，这与我们定义的电路规模——门的数量——是一致的。）

现在我们已经正式定义了 AON-CIRC 程序的规范，就可以定义 AON-CIRC 程序 $P$ **计算**一个函数 $f$ 的含义：

{{defc}}{AONcircdef}[使用AON-CIRC程序计算一个函数]
设 $f:\{0,1\}^n \rightarrow \{0,1\}^m$，且 $P$ 为一个具有 $n$ 个输入和 $m$ 个输出的有效 AON-CIRC 程序。  
如果对于每个 $x \in \{0,1\}^n$ 都有 $P(x) = f(x)$，则称 **$P$ 计算函数 $f$**。

以下已解练习给出了一个 AON-CIRC 程序的示例。

{{exec}}{aonforcmpsolved}考虑如下函数 $CMP:\{0,1\}^4 \rightarrow \{0,1\}$：对四个输入比特 $a,b,c,d \in \{0,1\}$，当且仅当由 $(a,b)$ 表示的数字大于由 $(c,d)$ 表示的数字时输出 $1$。  
即 $CMP(a,b,c,d) = 1$ 当且仅当 $2a + b > 2c + d$。

给出一个计算 $CMP$ 的 AON-CIRC 程序示例.

~~~admonish solution collapsible=true
编写这样的程序虽然繁琐，但并不困难。比较两个数字时，我们首先比较它们的最高有效位，然后依次比较下一位，以此类推。在数字仅有两位二进制的情况下，这些比较特别简单。由 $(a,b)$ 表示的数字大于由 $(c,d)$ 表示的数字，当且仅当满足以下任一条件：

1. $(a,b)$ 的最高有效位 $a$ 大于 $(c,d)$ 的最高有效位 $c$；  

或  

2. 两个最高有效位 $a$ 和 $c$ 相等，但 $b > d$。

另一种等价表述为：数字 $(a,b)$ 大于 $(c,d)$ 当且仅当 $a>c$ **或** ($a\ge c$ **且** $b>d$)。

对于二进制位 $\alpha, \beta$，条件 $\alpha > \beta$ 仅当 $\alpha = 1$ 且 $\beta = 0$，也就是 $\AND(\alpha, \NOT(\beta)) = 1$；条件 $\alpha \ge \beta$ 则为 $\OR(\alpha, \NOT(\beta)) = 1$。  

结合这些观察，可以得到用于计算 $CMP$ 的以下 AON-CIRC 程序：

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

我们也可以将这个 8 行程序表示为一个包含 8 个门的电路，见[下图](#aoncmpfig)。
~~~

```admonish quote title=""
<a id="aoncmpfig">![A circuit for computing the $CMP$ function. The evaluation of this circuit on $(1,1,1,0)$ yields the output $1$, since the number $3$ (represented in binary as $11$) is larger than the number $2$ (represented in binary as $10$).](./images/chapter3/comparecircuit.png)</a>
一个用于计算 $CMP$ 函数的电路。以输入 $(1,1,1,0)$ 运行该电路，输出为 $1$，因为数字 $3$（二进制表示为 $11$）大于数字 $2$（二进制表示为 $10$）。
```

### 3.4.2 证明AON-CIRC程序与布尔电路的等价性

我们现在正式证明 AON-CIRC 程序和布尔电路具有完全相同的计算能力：

{{thmc}}{slcircuitequivthm}[电路与直线程序的等价性]
设 $f:\{0,1\}^n \rightarrow \{0,1\}^m$, $s \ge m$ 为某个正整数。则 $f$ 可以由一个包含 $s$ 个门的布尔电路计算，当且仅当 $f$ 可以由一个包含 $s$ 行的 AON-CIRC 程序计算。

```admonish idea title="证明思路"
证明思路很简单——AON-CIRC 程序和布尔电路只是描述同一计算过程的不同方式。  
例如，布尔电路中的一个 $\AND$ 门对应于对两个已计算值执行 $\AND$ 操作。  
在 AON-CIRC 程序中，这对应于一行将两个已计算变量的 $\AND$ 结果存储到一个变量中的语句。
```

```admonish pause
{{ref: slcircuitequivthm}} 的证明本质上很简单，但其中包含的所有细节可能会让阅读起来有些繁琐。  
你最好先尝试自己推导一遍，再去阅读证明。  
我们的 [GitHub 仓库](https://github.com/boazbk/tcscode) 中提供了 {{ref: slcircuitequivthm}} 的“Python 证明”：实现了 `circuit2prog` 和 `prog2circuits` 函数，用于在布尔电路和 AON-CIRC 程序之间互相转换。
```

```admonish proof collapsible=true
设 $f:\{0,1\}^n \rightarrow \{0,1\}^m$。由于该定理是**“当且仅当”**的命题，要证明它，我们需要展示两个方向：  
1. 将计算 $f$ 的 AON-CIRC 程序转换为计算 $f$ 的布尔电路；  
2. 将计算 $f$ 的布尔电路转换为计算 $f$ 的 AON-CIRC 程序。  

我们先考虑第一个方向。设 $P$ 是一个计算 $f$ 的 AON-CIRC 程序。我们定义一个电路 $C$ 如下：该电路有 $n$ 个输入和 $s$ 个门。对于每个 $i \in [s]$，若第 $i$ 行运算为 `foo = AND(bar,blah)`，则电路中的第 $i$ 个门为 $\AND$ 门，其入邻居连接到对应的第 $j$ 和第 $k$ 个门，$j$ 和 $k$ 分别对应于在第 $i$ 行之前最后一次写入变量 `bar` 和 `blah` 的行号。（例如，如果 $i=57$，且 `bar` 最近一次被写入的是第 $35$ 行，`blah` 最近一次被写入的是第 $17$ 行，则门 $57$ 的两个入邻居为门 $35$ 和门 $17$。）  
如果 `bar` 或 `blah` 是输入变量，则将门连接到对应的输入顶点。  
如果 `foo` 是输出变量（形式为 `Y[j]`），则在对应门上添加相同标签，将其标记为输出门。  
对于 $\OR$ 或 $\NOT$ 操作的情况也类似，只是使用对应的 $\OR$ 或 $\NOT$ 门，并且 $\NOT$ 门只有一个入邻居。  

对于任意输入 $x \in \{0,1\}^n$，若运行程序 $P$，第 $i$ 行计算的值恰好等于在电路 $C$ 上对 $x$ 求值时第 $i$ 个门的值。因此，对所有 $x \in \{0,1\}^n$，有 $C(x)=P(x)$。

再看另一个方向。设 $C$ 是一个具有 $n$ 个输入、$s$ 个门的电路，计算函数 $f$。我们对门按照拓扑序排序，记为 $v_0,\ldots,v_{s-1}$。  
现在可以构造一个包含 $s$ 行运算的程序 $P$：  
对于每个 $i \in [s]$，若 $v_i$ 是一个 $\AND$ 门，其入邻居为 $v_j, v_k$，则在 $P$ 中添加一行 `temp_i = AND(temp_j,temp_k)`，除非某个顶点是输入顶点或输出门，此时改用 `X[.]` 或 `Y[.]`。  
由于我们按照拓扑顺序操作，保证入邻居 $v_j$ 和 $v_k$ 对应的变量已被赋值。  
$\OR$ 和 $\NOT$ 门同理。  

再次验证，对于每个输入 $x$，$P(x)=C(x)$，因此程序计算与电路相同的函数。  
（注意，由于 $C$ 是合法电路，根据 {{ref: booleancircdef}}，$C$ 的每个输入顶点至少有一个出邻居，并且恰有 $m$ 个输出门标记为 $0,\ldots,m-1$；因此所有变量 `X[0],\ldots,X[n-1]` 和 `Y[0],\ldots,Y[m-1]` 都会出现在程序 $P$ 中。）
```

```admonish quote title=""
<a id="aoncircequivfig">![Two equivalent descriptions of the same AND/OR/NOT computation as both an AON program and a Boolean circuit.](./images/chapter3/aoncircequiv.png)</a>
同一 $\AND/\OR/\NOT$ 计算的两种等效描述：既作为 AON 程序，也作为布尔电路。
```

## 3.5 Physical implementations of computing devices (digression) {#physicalimplementationsec }


_Computation_ is an abstract notion that is distinct from its physical _implementations_.
While most modern computing devices are obtained by mapping logical gates to semiconductor-based transistors, throughout history people have computed using a huge variety of mechanisms,  including mechanical systems, gas and liquid (known as _fluidics_), biological and chemical processes, and even living creatures (e.g., see [crabfig]() or  [this video](https://www.youtube.com/watch?v=czk4xgdhdY4) for how crabs or slime mold can be used to do computations).


In this section we will review some of these implementations, both so you can get an appreciation of how it is possible to directly translate Boolean circuits to the physical world, without going through the entire stack of architecture, operating systems, and compilers, as well as to emphasize that silicon-based processors are by no means the only way to perform computation.
Indeed, as we will see in [quantumchap](), a very exciting recent line of work involves using different media for computation that would allow us to take advantage of _quantum mechanical effects_ to enable different types of algorithms.

![Crab-based logic gates from the paper "Robust soldier-crab ball gate" by Gunji, Nishiyama and Adamatzky. This is an example of an AND gate that relies on the tendency of two swarms of crabs arriving from different directions to combine to a single swarm that continues in the average of the directions.](./images/chapter3/crab-gate.jpg){#crabfig .margin}

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Such a cool way to explain logic gates. <a href="https://t.co/6Wgu2ZKFCx">pic.twitter.com/6Wgu2ZKFCx</a></p>&mdash; Lionel Page (\@page_eco) <a href="https://twitter.com/page_eco/status/1188749430020698112?ref_src=twsrc%5Etfw">October 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### Transistors

A _transistor_ can be thought of as an electric circuit with two inputs, known as the _source_ and the _gate_ and an output, known as the _sink_.
The gate controls whether current flows from the source to the sink.
In a _standard transistor_, if the gate is "ON" then current can flow from the source to the sink and if it is "OFF" then it can't.
In a _complementary transistor_ this is reversed: if the gate is "OFF" then current can flow from the source to the sink and if it is "ON" then it can't.

![We can implement the logic of transistors using water. The water pressure from the gate closes or opens a faucet between the source and the sink.](./images/chapter3/transistor_water.png){#transistor-water-fig .margin  }

There are several ways to implement the logic of a transistor.
For example, we can use faucets to implement it using water pressure (e.g. [transistor-water-fig]()). This might seem as merely a curiosity, but there is a field known as [fluidics](https://en.wikipedia.org/wiki/Fluidics) concerned with implementing logical operations using liquids or gasses. Some of the motivations include operating in extreme environmental conditions such as in space or a battlefield, where standard electronic equipment would not survive.

The standard implementations of transistors use electrical current.
One of the original implementations used   _vacuum tubes_.
As its name implies, a vacuum tube is a tube containing nothing (i.e., a vacuum) and where a priori electrons could freely flow from the source (a wire) to the sink (a plate). However, there is a gate (a grid)  between the two, where modulating its voltage can block the flow of electrons.

Early vacuum tubes were roughly the size of lightbulbs (and looked very much like them too).
In the 1950's they were supplanted by _transistors_, which implement the same logic using _semiconductors_ which are materials that normally do not conduct electricity but whose conductivity can be modified and controlled by inserting impurities ("doping") and applying an external electric field (this is known as the _field effect_).
In the 1960's computers started to be implemented using _integrated circuits_ which enabled much greater density.
In 1965, Gordon Moore predicted that the number of transistors per integrated circuit would double every year (see [moorefig]()), and that this would lead to "such wonders as home computers —or at least terminals connected to a central computer— automatic controls for automobiles, and personal portable communications equipment".
Since then, (adjusted versions of) this so-called "Moore's law" have been running strong, though exponential growth cannot be sustained forever, and some physical limitations are already [becoming apparent](http://www.nature.com/news/the-chips-are-down-for-moore-s-law-1.19338).

![The number of transistors per integrated circuit from 1959 till 1965 and a prediction that exponential growth will continue for at least another decade. Figure taken from "Cramming More Components onto Integrated Circuits", Gordon Moore, 1965](./images/chapter3/gordon_moore.png){#moorefig .margin  }

![Cartoon from Gordon Moore's article "predicting" the implications of radically improving transistor density.](./images/chapter3/moore_cartoon.png){#moore-cartoon-fig .margin  }

![The exponential growth in computing power over the last 120 years. Graph by Steve Jurvetson, extending a prior graph of Ray Kurzweil.](./images/chapter3/1200px-Moore's_Law_over_120_Years.png){#kurzweil-fig .margin  }




### Logical gates from transistors

We can use transistors to implement various Boolean functions such as $\AND$, $\OR$, and $\NOT$.
For each two-input gate $G:\{0,1\}^2 \rightarrow \{0,1\}$,  such an implementation would be a system with two input wires $x,y$ and one output wire $z$, such that if we identify high voltage with "$1$" and low voltage with "$0$", then the wire  $z$ will be equal to "$1$" if and only if applying $G$ to the values of the wires $x$ and $y$ is $1$ (see [logicgatestransistorsfig]() and [transistor-nand-fig]()).
This means that if there exists a AND/OR/NOT circuit to compute a function $g:\{0,1\}^n \rightarrow \{0,1\}^m$, then we can compute $g$ in the physical world using transistors as well.

![Implementing logical gates using transistors. Figure taken from [Rory Mangles' website](http://www.northdownfarm.co.uk/rory/tim/basiclogic.htm).](./images/chapter3/dtl_logic.png){#logicgatestransistorsfig   .margin  }

![Implementing a \text{NAND} gate  (see [3.6节](#nandsec)) using transistors.](./images/chapter3/nand_transistor.png){#transistor-nand-fig .margin  }





### Biological computing

Computation can be based on [biological or chemical systems](http://www.nature.com/nrg/journal/v13/n7/full/nrg3197.html).
For example the [_lac_ operon](https://en.wikipedia.org/wiki/Lac_operon) produces the enzymes needed to digest lactose only if the conditions $x \wedge (\neg y)$ hold where $x$ is "lactose is present" and $y$ is "glucose is present".
Researchers have managed to [create transistors](http://science.sciencemag.org/content/340/6132/554?iss=6132), and from them  logic gates, based on DNA molecules (see also [transcriptorfig]()).
Projects such as the [Cello programming language](https://www.cidarlab.org/cello) enable converting Boolean circuits into DNA sequences that encode operations that can be executed in bacterial cells, see [this video](https://youtu.be/-1fqgrF7fXU). 
One motivation for DNA computing is to achieve increased parallelism or storage density; another is to create "smart biological agents" that could perhaps be injected into bodies, replicate themselves, and fix or kill cells that were damaged by a disease such as cancer.
Computing in biological systems is not restricted, of course, to DNA:
even larger systems such as [flocks of birds](https://www.cs.princeton.edu/~chazelle/pubs/cacm12-natalg.pdf) can be considered as computational processes.

![Performance of DNA-based logic gates. Figure taken from paper of [Bonnet et al](http://science.sciencemag.org/content/early/2013/03/27/science.1232758.full), Science, 2013.](./images/chapter3/transcriptor.jpg){#transcriptorfig .margin  }

### Cellular automata and the game of life

_Cellular automata_ is a model of a system composed of a sequence of _cells_, each of which can have a finite state.
At each step, a cell updates its state based on the states of its _neighboring cells_ and some simple rules.
As we will discuss later in this book (see [cellularautomatasec]()), cellular automata such as Conway's "Game of Life" can be used to simulate computation gates.

![An AND gate using a "Game of Life" configuration. Figure taken from [Jean-Philippe Rennard's paper](http://www.rennard.org/alife/CollisionBasedRennard.pdf).](./images/chapter3/game_of_life_and.png){#gameoflifefig .margin  }


### Neural networks

One computation device that we all carry with us is our own _brain_.
Brains have served humanity throughout history, doing computations that range from distinguishing prey from predators, through making scientific discoveries and artistic masterpieces, to composing witty 280 character messages.
The exact working of the brain is still not fully understood, but one common mathematical model for it is a (very large) _neural network_.

A neural network can be thought of as a Boolean circuit that instead of $\AND$/$\OR$/$\NOT$ uses some other gates as the basic basis.
For example, one particular basis we can use are _threshold gates_.
For every vector $w= (w_0,\ldots,w_{k-1})$ of integers and integer $t$ (some or all of which could be negative),
the _threshold function corresponding to $w,t$_ is the function
$T_{w,t}:\{0,1\}^k \rightarrow \{0,1\}$ that maps $x\in \{0,1\}^k$ to $1$ if and only if $\sum_{i=0}^{k-1} w_i x_i \geq t$.
For example, the threshold function $T_{w,t}$ corresponding to $w=(1,1,1,1,1)$ and $t=3$ is simply the majority function $\text{MAJ}_5$ on $\{0,1\}^5$.
Threshold gates can be thought of as an approximation for _neuron cells_ that make up the core of human and animal brains. To a first approximation, a neuron has $k$ inputs and a single output, and the neuron "fires" or "turns on" its output when those signals pass some threshold.

Many machine learning algorithms use _artificial neural networks_ whose purpose is not to imitate biology but rather to perform some computational tasks, and hence are not restricted to a threshold or other biologically-inspired gates.
Generally, a neural network is often described as operating on signals that are real numbers, rather than $0/1$ values, and where the output of a gate on inputs $x_0,\ldots,x_{k-1}$ is obtained by applying $f(\sum_i w_i x_i)$ where $f:\R \rightarrow \R$ is an [activation function](https://goo.gl/p9izfA) such as rectified linear unit (ReLU), Sigmoid, or many others (see [activationfunctionsfig]()).
However, for the purposes of our discussion, all of the above are equivalent (see also [NANDsfromActivationfunctionex]()).
In particular we can reduce the setting of real inputs to binary inputs by representing a real number in the binary basis, and multiplying the weight of the bit corresponding to the $i^{th}$ digit by $2^i$.

![Common activation functions used in Neural Networks, including rectified linear units (ReLU), sigmoids, and hyperbolic tangent. All of those can be thought of as continuous approximations to simplify the step function. All of these can be used to compute the \text{NAND} gate (see [NANDsfromActivationfunctionex]()). This property enables neural networks to (approximately) compute any function that can be computed by a Boolean circuit.](./images/chapter3/activationfuncs.png){#activationfunctionsfig .margin }


### A computer made from marbles and pipes

We can implement computation using many other physical media, without any electronic, biological, or chemical components. Many suggestions for _mechanical_ computers have been put forward, going back at least to Gottfried Leibniz's computing machines from the 1670s and Charles Babbage's 1837 plan for a mechanical ["Analytical Engine"](https://en.wikipedia.org/wiki/Analytical_Engine).
As one example, [marblefig]() shows a simple implementation of a $\text{NAND}$ (negation of AND, see [3.6节](#nandsec)) gate using marbles going through pipes. We represent a logical value in $\{0,1\}$ by a pair of pipes, such that there is a marble flowing through exactly one of the pipes.
We call one of the pipes the "$0$ pipe" and the other the "$1$ pipe", and so the identity of the pipe containing the marble determines the logical value.
A NAND gate corresponds to a mechanical object with two pairs of incoming pipes and one pair of outgoing pipes, such that for every $a,b \in \{0,1\}$, if two marbles are rolling toward the object in the $a$ pipe of the first pair and the $b$ pipe of the second pair, then a marble will roll out of the object in the $\text{NAND}(a,b)$-pipe of the outgoing pair.
In fact, there is even a commercially-available educational game that uses marbles as a basis of computing, see [turingtumblefig]().



![A physical implementation of a NAND gate using marbles. Each wire in a Boolean circuit is modeled by a pair of pipes representing the values $0$ and $1$ respectively, and hence a gate has four input pipes (two for each logical input) and two output pipes. If one of the input pipes representing the value $0$ has a marble in it then that marble will flow to the output pipe representing the value $1$. (The dashed line represents a gadget that will ensure that at most one marble is allowed to flow onward in the pipe.) If both the input pipes representing the value $1$ have marbles in them, then the first marble will be stuck but the second one will flow onwards to the output pipe representing the value $0$.](./images/chapter3/marble.png){#marblefig .margin  }

![A "gadget" in a pipe that ensures that at most one marble can pass through it. The first marble that passes causes the barrier to lift and block new ones.](./images/chapter3/gadget.png){#gadgetfig .margin  }

![The game ["Turing Tumble"](https://www.turingtumble.com/) contains an implementation of logical gates using marbles.](./images/chapter3/turingtumble.png){#turingtumblefig .margin  }




## 3.6 The NAND function { #nandsec }

The $\text{NAND}$ function is another simple function that is extremely useful for defining computation.
It is the function mapping $\{0,1\}^2$ to $\{0,1\}$ defined by:

$$\text{NAND}(a,b) = \begin{cases} 0 & a=b=1 \\ 1 & \text{otherwise} \end{cases}\;.$$

As its name implies, $\text{NAND}$ is the NOT of AND (i.e., $\text{NAND}(a,b)= NOT(AND(a,b))$), and so we can clearly compute $\text{NAND}$ using $\AND$  and $\NOT$.
Interestingly, the opposite direction holds as well:

> ### {.theorem title="NAND computes AND,OR,NOT" #univnandonethm}
We can compute $\AND$, $\OR$, and $\NOT$ by composing only the $\text{NAND}$ function.

> ### {.proof data-ref="univnandonethm"}
We start with the following observation. For every $a\in \{0,1\}$, $\AND(a,a)=a$. Hence, $\text{NAND}(a,a)=NOT(AND(a,a))=NOT(a)$.
This means that $\text{NAND}$ can compute $\NOT$.
By the principle of "double negation",  $\AND(a,b)=NOT(NOT(AND(a,b)))$, and hence we can use $\text{NAND}$ to compute $\AND$ as well.
Once we can compute $\AND$ and $\NOT$, we can compute $\OR$ using ["De Morgan's Law"](https://goo.gl/TH86dH):  $\OR(a,b)=NOT(AND(NOT(a),NOT(b)))$ (which can also be written as $a \vee b = \overline{\overline{a} \wedge \overline{b}}$) for every $a,b \in \{0,1\}$.

> ### { .pause }
[univnandonethm]()'s proof is very simple, but you should make sure that __(i)__ you understand the statement of the theorem, and __(ii)__ you follow its proof. In particular, you should make sure you understand why De Morgan's law is true.

We can use $\text{NAND}$ to compute many other functions, as demonstrated in the following exercise.

> ### {.solvedexercise title="Compute majority with NAND" #majbynandex}
Let $\text{MAJ}: \{0,1\}^3 \rightarrow \{0,1\}$ be the function that on input $a,b,c$ outputs $1$ iff $a+b+c \geq 2$. Show how to compute $\text{MAJ}$ using a composition of $\text{NAND}$'s.

::: {.solution data-ref="majbynandex"}
Recall that {{eqref: eqmajandornot}} states that

$$
\text{MAJ}(x_0,x_1,x_2) = OR\left(\, AND(x_0,x_1)\;,\; OR \bigl( AND(x_1,x_2) \;,\; AND(x_0,x_2) \bigr) \, \right) \;. {{numeq}}{eqmajandornotrestated}
$$

We can use [univnandonethm]()  to replace all the occurrences of $\AND$ and $\OR$   with $\text{NAND}$'s.
Specifically, we can use the equivalence $\AND(a,b)=NOT(NAND(a,b))$, $\OR(a,b)=NAND(NOT(a),NOT(b))$, and $\NOT(a)=NAND(a,a)$ to replace the right-hand side of
{{eqref: eqmajandornotrestated}} with an expression involving only $\text{NAND}$, yielding that $\text{MAJ}(a,b,c)$ is equivalent to the (somewhat unwieldy) expression

$$
\begin{gathered}
NAND \biggl(\, NAND\Bigl(\, NAND\bigl(NAND(a,b),NAND(a,c)\bigr), \\
NAND\bigl(NAND(a,b),NAND(a,c)\bigr)\, \Bigr),\\
NAND(b,c) \, \biggr)
\end{gathered}
$$

The same formula can also be expressed as a circuit with NAND gates, see [majnandcircfig]().
:::

![A circuit with NAND gates to compute the Majority function on three bits](./images/chapter3/majfromnand.png){#majnandcircfig .margin  }  





### NAND Circuits

We define _NAND Circuits_ as circuits in which all the gates are NAND operations.
Such a circuit again corresponds to a directed acyclic graph (DAG) since all the gates correspond to the same function (i.e., NAND), we do not even need to label them, and all gates have in-degree exactly two.
Despite their simplicity, NAND circuits can be quite powerful.


::: {.example title="$\text{NAND}$ circuit for $\XOR$" #xornandexample}
Recall the $\XOR$ function which maps $x_0,x_1 \in \{0,1\}$ to $x_0 + x_1 \mod 2$.
We have seen in [xoraonexample](xoraonexample) that we can compute $\XOR$ using $\AND$, $\OR$, and $\NOT$, and so by [univnandonethm]() we can compute it using only $\text{NAND}$'s.
However, the  following is a direct construction of computing $\XOR$ by a sequence of NAND operations:

1. Let $u = NAND(x_0,x_1)$.
2. Let $v = NAND(x_0,u)$
3. Let $w = NAND(x_1,u)$.
4. The $\XOR$ of $x_0$ and $x_1$ is $y_0 = NAND(v,w)$.

One can verify that this algorithm does indeed compute $\XOR$ by enumerating all the four choices for $x_0,x_1 \in \{0,1\}$.
We can also represent this algorithm graphically as a circuit, see [cornandcircfig]().
:::


![A circuit with NAND gates to compute the XOR of two bits.](./images/chapter3/nandcircxor.png){#cornandcircfig .margin  }  

In fact, we can show the following theorem:

> ### {.theorem title="NAND is a universal operation" #NANDuniversamthm}
For every Boolean circuit $C$ of $s$ gates, there exists a NAND circuit $C'$ of at most $3s$ gates that computes the same function as $C$.

> ### {.proofidea data-ref="NANDuniversamthm"}
The idea of the proof is to just replace every $\AND$, $\OR$ and $\NOT$ gate with their NAND implementation following the proof of [univnandonethm]().

::: {.proof data-ref="NANDuniversamthm"}
If $C$ is a Boolean circuit, then since, as we've seen in the proof of  [univnandonethm](),  for every $a,b \in \{0,1\}$

* $\NOT(a) = NAND(a,a)$

* $\AND(a,b) = NAND(NAND(a,b),NAND(a,b))$

* $\OR(a,b) = NAND(NAND(a,a),NAND(b,b))$

we can replace every gate of $C$ with at most three $\text{NAND}$ gates to obtain an equivalent circuit $C'$. The resulting circuit will have at most $3s$ gates.
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


[incrementalg]() describes precisely how to compute the increment operation, and can be easily transformed into _Python_ code that performs the same computation, but it does not seem to directly yield a NAND circuit to compute this.
However, we can transform this algorithm line by line to a NAND circuit.
For example, since for every $a$, $\text{NAND}(a,NOT(a))=1$, we can replace the initial statement $c_0=1$ with $c_0 = NAND(x_0, NAND(x_0,x_0))$.
We already know how to compute $\XOR$ using NAND and so we can use this to implement the operation $y_i \leftarrow XOR(x_i,c_i)$.
Similarly, we can write the "if" statement as saying $c_{i+1} \leftarrow AND(c_i,x_i)$,  or in other words $c_{i+1} \leftarrow  NAND( NAND(c_i,x_i), NAND(c_i,x_i))$.
Finally, the assignment $y_n = c_n$ can be written as $y_n = NAND( NAND(c_n,c_n), NAND(c_n,c_n))$.
Combining these observations yields for every $n\in \N$, a $\text{NAND}$ circuit to compute $INC_n$.
For example, [nandincrememntcircfig]() shows what this circuit looks like for $n=4$.



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


Once again, [additionfromnand]() can be translated into a NAND circuit.
The crucial observation is that the "if/then" statement simply corresponds to
$c_{i+1} \leftarrow \text{MAJ}_3(u_i,v_i,v_i)$ and we have seen in [majbynandex]() that the function $\text{MAJ}_3:\{0,1\}^3 \rightarrow \{0,1\}$ can be computed using $\text{NAND}$s.



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

Formally, just like we did in [AONcircdef]() for AON-CIRC, we can define the notion of computation by a NAND-CIRC program in the natural way:


::: {.definition title="Computing by a NAND-CIRC program" #NANDcomp}
Let $f:\{0,1\}^n \rightarrow \{0,1\}^m$ be some function, and let $P$ be a NAND-CIRC program. We say that $P$ _computes_ the function $f$ if:

1. $P$ has $n$ input variables `X[`$0$`]`$,\ldots,$`X[`$n-1$`]` and $m$ output variables `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]`.

2. For every $x\in \{0,1\}^n$, if we execute $P$ when we assign to `X[`$0$`]`$,\ldots,$`X[`$n-1$`]` the values $x_0,\ldots,x_{n-1}$, then at the end of the execution, the output variables `Y[`$0$`]`,$\ldots$,`Y[`$m-1$`]` have the values $y_0,\ldots,y_{m-1}$ where $y=f(x)$.
:::

As before we can show that NAND circuits are equivalent to NAND-CIRC programs (see [progandcircfig]()):

> ### {.theorem title="NAND circuits and straight-line program equivalence" #NANDcircslequivthm}
For every $f:\{0,1\}^n \rightarrow \{0,1\}^m$ and $s \geq m$, $f$ is computable by a NAND-CIRC program of $s$ lines if and only if $f$ is computable by a NAND circuit of $s$ gates.


![A NAND program and the corresponding circuit. Note how every line in the program corresponds to a gate in the circuit.](./images/chapter3/nandcircuitequiv.png){#progandcircfig   .margin  }


We omit the proof of [NANDcircslequivthm]() since it follows along exactly the same lines as the equivalence of Boolean circuits and AON-CIRC program  ({{ref: slcircuitequivthm}}).
Given [NANDcircslequivthm]() and [NANDuniversamthm](), we know that we can translate every $s$-line AON-CIRC program $P$ into an equivalent NAND-CIRC program of at most $3s$ lines.
In fact, this translation can be easily done by replacing every line of the form `foo = AND(bar,blah)`, `foo = OR(bar,blah)` or `foo = NOT(bar)` with the equivalent 1-3 lines that use the `NAND` operation.
Our [GitHub repository](https://github.com/boazbk/tcscode) contains a "proof by code": a simple Python program `AON2NAND` that transforms an AON-CIRC into an equivalent NAND-CIRC program.



> ### {.remark title="Is the NAND-CIRC programming language Turing Complete? (optional note)" #NANDturingcompleteness}
You might have heard of a term called "Turing Complete" that is sometimes used to describe programming languages. (If you haven't, feel free to ignore the rest of this remark: we define this term precisely in [chapequivalentmodels]().)
If so, you might wonder if the NAND-CIRC programming language has this property.
The answer is __no__, or perhaps more accurately, the term "Turing Completeness" is not really applicable for the NAND-CIRC programming language.
The reason is that, by design, the NAND-CIRC programming language can only compute _finite_ functions $F:\{0,1\}^n \rightarrow \{0,1\}^m$ that take a fixed number of input bits and produce a fixed number of outputs bits.
The term "Turing Complete" is only applicable to programming languages for _infinite_ functions that can take inputs of arbitrary length.
We will come back to this distinction later on in this book.

## 3.7 Equivalence of all these models


If we put together {{ref: slcircuitequivthm}}, [NANDuniversamthm](), and [NANDcircslequivthm](), we obtain the following result:

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
We omit the formal proof, which is obtained by combining {{ref: slcircuitequivthm}}, [NANDuniversamthm](), and [NANDcircslequivthm](). The key observation is that the results we have seen allow us to translate a program/circuit that computes $f$ in one of the above models into a program/circuit that computes $f$ in another model by increasing the lines/gates by at most a constant factor (in fact this constant factor is at most $3$).


{{ref: slcircuitequivthm}} is a special case of a more general result.
We can consider even more general models of computation, where instead of AND/OR/NOT or NAND, we use other operations (see [othergatessec]() below).
It turns out that Boolean circuits are equivalent in power to such models as well.
The fact that all these different ways to define computation lead to equivalent models shows that we are "on the right track".
It justifies the seemingly arbitrary choices that we've made of using AND/OR/NOT or NAND as our basic operations, since these choices do not affect the power of our computational model.
Equivalence results such as [equivalencemodelsthm]() mean that we can easily translate between Boolean circuits, NAND circuits, NAND-CIRC programs and the like.
We will use this ability later on in this book, often shifting to the most convenient formulation without making a big deal about it.
Hence we will not worry too much about the distinction between, for example, Boolean circuits and NAND-CIRC programs.



In contrast, we will continue to take special care to distinguish between _circuits/programs_ and _functions_ (recall [functionprogramidea]()).
A function corresponds to a _specification_ of a computational task, and it is a fundamentally different object than a program or a circuit, which corresponds to the _implementation_ of the task.


### Circuits with other gate sets  {#othergatessec }

There is nothing special about AND/OR/NOT or NAND. For every set of functions $\mathcal{G} = \{ G_0,\ldots,G_{k-1} \}$, we can define a notion of circuits that use elements of  $\mathcal{G}$ as gates, and a notion of a "$\mathcal{G}$ programming language" where every line involves assigning to a variable `foo` the result of applying some $G_i \in \mathcal{G}$ to previously defined or input variables.
Specifically, we can make the following definition:

::: {.definition title="General straight-line programs" #genstraight-lineprogs}
Let $\mathcal{F} = \{ f_0,\ldots, f_{t-1} \}$ be a finite collection of Boolean functions, such that
$f_i:\{0,1\}^{k_i} \rightarrow \{0,1\}$ for some $k_i \in \N$.
An _$\mathcal{F}$ program_ is a sequence of lines, each of which assigns to some variable the result of applying some $f_i \in \mathcal{F}$ to $k_i$ other variables. As above, we use `X[`$i$`]` and `Y[`$j$`]` to denote the input and output variables.

We say that $\mathcal{F}$ is a _universal set of operations_ (also known as a universal gate set) if there exists a $\mathcal{F}$ program to compute the function $\text{NAND}$.
:::

AON-CIRC programs correspond to $\{AND,OR,NOT\}$ programs, NAND-CIRC programs corresponds to $\mathcal{F}$ programs for the set  $\mathcal{F}$ that only contains the $\text{NAND}$ function,   but we can also define  $\{ IF, ZERO, ONE\}$ programs (see below), or use any other set.

We can also define _$\mathcal{F}$ circuits_, which will be directed graphs in which each _gate_ corresponds to applying a function $f_i \in \mathcal{F}$, and will each have $k_i$ incoming wires and a single outgoing wire. (If the function $f_i$ is not _symmetric_, in the sense that the order of its input matters then we need to label each wire entering a gate as to which parameter of the function it corresponds to.)
As in {{ref: slcircuitequivthm}}, we can show that $\mathcal{F}$ circuits and $\mathcal{F}$ programs are equivalent.
We have seen that for $\mathcal{F} = \{ AND,OR, NOT\}$, the resulting circuits/programs are equivalent in power to the NAND-CIRC programming language, as we can compute $\text{NAND}$ using $\AND$/$\OR$/$\NOT$ and vice versa.
This turns out to be a special case of a general phenomenon — the _universality_ of $\text{NAND}$ and other gate sets — that we will explore more in-depth later in this book.

::: {.example title="IF,ZERO,ONE circuits" #IZOcircuits}
Let $\mathcal{F} = \{ IF , ZERO, ONE \}$ where $ZERO:\{0,1\} \rightarrow \{0\}$ and $ONE:\{0,1\} \rightarrow \{1\}$ are the constant zero and one functions,^[One can also define these functions as taking a length zero input. This makes no difference for the computational power of the model.] and $IF:\{0,1\}^3 \rightarrow \{0,1\}$ is the function that on input $(a,b,c)$ outputs $b$ if $a=1$ and $c$ otherwise.
Then $\mathcal{F}$ is universal.

Indeed, we can demonstrate that $\{ IF, ZERO, ONE \}$ is universal using the following formula for $\text{NAND}$:

$$
NAND(a,b) = IF(a,IF(b,ZERO,ONE),ONE) \;.
$$
:::


There are also some sets $\mathcal{F}$ that are more restricted in power. For example it can be shown that if we use only AND or OR gates (without NOT) then we do _not_ get an equivalent model of computation.
The exercises cover several examples of universal and non-universal gate sets.


### Specification vs. implementation  (again) {#specvsimplrem}

![It is crucial to distinguish between the _specification_ of a computational task, namely _what_ is the function that is to be computed and the _implementation_ of it, namely the algorithm, program, or circuit that contains the instructions defining _how_ to map an input to an output. The same function could be computed in many different ways.](./images/chapter3/specvsimpl.png){#specvsimplfig }


As we discussed in [secimplvsspec](), one of the most important distinctions in this book is that of _specification_ versus _implementation_ or separating "what" from "how" (see [specvsimplfig]()).
A _function_ corresponds to the _specification_ of a computational task, that is _what_ output should be produced for every particular input.
A _program_ (or circuit, or any other way to specify _algorithms_) corresponds to the _implementation_ of _how_ to compute the desired output from the input.
That is, a program is a set of instructions on how to compute the output from the input.
Even within the same computational model there can be many different ways to compute the same function.
For example, there is more than one NAND-CIRC program that computes the majority function, more than one Boolean circuit to compute the addition function, and so on and so forth.

Confusing specification and implementation  (or equivalently _functions_ and _programs_) is a common mistake, and one that is unfortunately encouraged by the common programming-language terminology of referring to parts of programs as "functions".
However, in both the theory and practice of computer science, it is important to maintain this distinction, and it is particularly important for us in this book.


> ### { .recap }
* An _algorithm_ is a recipe for performing a computation as a sequence of "elementary" or "simple" operations.
* One candidate definition for "elementary" operations is the set $\AND$, $\OR$ and $\NOT$.
* Another candidate definition for an "elementary" operation is the $\text{NAND}$ operation. It is an operation that is easily implementable in the physical world in a variety of methods including by electronic transistors.
* We can use $\text{NAND}$ to compute many other functions, including majority, increment, and others.
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

::: {.exercise title="\text{MAJ},NOT, 1 is universal" #majnotex}
Let $\text{MAJ}:\{0,1\}^3 \rightarrow \{0,1\}$ be the majority function.
Prove that $\{ \text{MAJ},NOT, 1 \}$ is a universal set of gates.
:::


::: {.exercise title="\text{MAJ},NOT  is not universal" #majnotextwo}
Prove that $\{ \text{MAJ},NOT  \}$ is not a universal set. See footnote for hint.^[_Hint:_ Use the fact that $\text{MAJ}(\overline{a},\overline{b},\overline{c}) = \overline{\text{MAJ}(a,b,c)}$ to prove that every $f:\{0,1\}^n \rightarrow \{0,1\}$ computable by a circuit with only $\text{MAJ}$ and $\NOT$ gates satisfies $f(0,0,\ldots,0) \neq f(1,1,\ldots,1)$. Thanks to Nathan Brunelle and David Evans for suggesting this exercise.]
:::

::: {.exercise title="NOR is universal" #norex}
Let $\text{NOR}:\{0,1\}^2 \rightarrow \{0,1\}$ defined as $\text{NOR}(a,b) = NOT(OR(a,b))$. Prove that $\{ NOR \}$ is a universal set of gates.
:::


::: {.exercise title="Lookup is universal" #lookupex}
Prove that $\{ LOOKUP_1,0,1 \}$ is a universal set of gates where $0$ and $1$ are the constant functions and $LOOKUP_1:\{0,1\}^3 \rightarrow \{0,1\}$ satisfies $LOOKUP_1(a,b,c)$ equals $a$ if $c=0$ and equals $b$ if $c=1$.
:::


> ### {.exercise title="Bound on universal basis size (challenge)" #universal-bound}
Prove that for every subset $B$ of the functions from $\{0,1\}^k$ to $\{0,1\}$,
if $B$ is universal then there is a $B$-circuit of at most $O(1)$ gates to compute the $\text{NAND}$ function (you can start by showing that there is a $B$ circuit of at most $O(k^{16})$ gates).^[Thanks to Alec Sun and Simon Fischer for comments on this problem.]


::: {.exercise title="Size and inputs / outputs" #nandcircsizeex}
Prove that for every NAND circuit of size $s$ with $n$ inputs and $m$ outputs, $s \geq \min \{ n/2 , m \}$. See footnote for hint.^[_Hint:_ Use the conditions of {{ref: booleancircdef}} stipulating that every input vertex has at least one out-neighbor and there are exactly $m$ output gates. See also [booleancircuitsremarks]().]
:::



> ### {.exercise title="Threshold using NANDs" #threshold-nand-ex}
Prove that there is some constant $c$ such that for every $n>1$, and integers $a_0,\ldots,a_{n-1},b \in \{-2^n,-2^n+1,\ldots,-1,0,+1,\ldots,2^n\}$, there is a NAND circuit with at most $c n^4$ gates that computes the _threshold_ function $f_{a_0,\ldots,a_{n-1},b}:\{0,1\}^n \rightarrow \{0,1\}$ that on input $x\in \{0,1\}^n$ outputs $1$ if and only if $\sum_{i=0}^{n-1} a_i x_i > b$.

::: {.exercise title="NANDs from activation functions" #NANDsfromActivationfunctionex}
We say that a function $f:\mathbb{R}^2 \rightarrow \mathbb{R}$ is a _NAND approximator_ if it has the following property: for every $a,b \in \mathbb{R}$, if $\min\{|a|,|1-a|\}\leq 1/3$ and $\min \{ |b|,|1-b| \}\leq 0.1$ then $|f(a,b) - NAND(\lfloor a \rceil, \lfloor b \rceil)| \leq 0.1$ where we denote by $\lfloor x \rceil$ the integer closest to $x$. That is, if $a,b$ are within a distance $1/3$ to $\{0,1\}$ then we want $f(a,b)$ to equal the $\text{NAND}$ of the values in $\{0,1\}$ that are closest to $a$ and $b$ respectively. Otherwise, we do not care what the output of $f$ is on $a$ and $b$.

In this exercise you will show that you can construct a NAND approximator from many common activation functions used in deep neural networks. As a corollary you will obtain that deep neural networks can simulate NAND circuits. Since NAND circuits can also simulate deep neural networks, these two computational models are equivalent to one another.


1. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(DReLU(L'(a,b)))$ where $L':\mathbb{R}^2 \rightarrow \mathbb{R}$ is an _affine_ function (of the form $L'(a,b)=\alpha a + \beta b + \gamma$ for some $\alpha,\beta,\gamma \in \mathbb{R}$), $L$ is an affine function (of the form $L(y) = \alpha y + \beta$ for $\alpha,\beta \in \mathbb{R}$), and $DReLU:\mathbb{R} \rightarrow \mathbb{R}$, is the function defined as $DReLU(x) = \min(1,\max(0,x))$. Note that $DReLU(x) = 1-ReLU(1-ReLU(x))$  where $ReLU(x)=\max(x,0)$ is the rectified linear unit activation function.

2. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(sigmoid(L'(a,b)))$ where $L',L$ are affine as above and $sigmoid:\mathbb{R} \rightarrow \mathbb{R}$ is the function defined as $sigmoid(x) = e^x/(e^x+1)$.

3. Show that there is a NAND approximator $f$ defined as $f(a,b) = L(tanh(L'(a,b)))$ where $L',L$ are affine as above and $tanh:\mathbb{R} \rightarrow \mathbb{R}$ is the function defined as $tanh(x) = (e^x-e^{-x})/(e^x+e^{-x})$.

4. Prove that for every NAND-circuit $C$ with $n$ inputs and one output that computes a function $g:\{0,1\}^n \rightarrow \{0,1\}$, if we replace every gate of $C$ with a NAND-approximator and then invoke the resulting circuit on some $x\in \{0,1\}^n$, the output will be a number $y$ such that $|y-g(x)|\leq 1/3$.
:::


::: {.exercise title="Majority with NANDs efficiently" #majwithNAND}
Prove that there is some constant $c$ such that for every $n>1$, there is a NAND circuit of at most $c\cdot n$ gates that computes the majority function on $n$ input bits $\text{MAJ}_n:\{0,1\}^n \rightarrow \{0,1\}$. That is $\text{MAJ}_n(x)=1$ iff $\sum_{i=0}^{n-1}x_i > n/2$. See footnote for hint.^[One approach to solve this is using recursion and analyzing it using the so called  "Master Theorem".]
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
One of the earliest people to realize the engine's potential and far-reaching implications was Ada Lovelace (see the notes for [chaploops]()).




Boolean algebra was first investigated by Boole and DeMorgan in the 1840's [@Boole1847mathematical, @DeMorgan1847].
The definition of Boolean circuits and connection to electrical relay circuits was given in Shannon's Masters Thesis [@Shannon1938].
(Howard Gardener called Shannon's thesis "possibly the most important, and also the most famous, master's thesis of the [20th] century".)
Savage's book [@Savage1998models], like this one, introduces the theory of computation starting with Boolean circuits as the first model.
Jukna's book [@Jukna12] contains a modern in-depth exposition of Boolean circuits, see also [@wegener1987complexity].


The NAND function was shown to be universal by Sheffer [@Sheffer1913], though this also appears in the earlier work of Peirce, see [@Burks1978charles].
Whitehead and Russell used NAND as the basis for their logic in their magnum opus _Principia Mathematica_ [@WhiteheadRussell1912].
In her Ph.D thesis, Ernst [@Ernst2009phd] investigates empirically the minimal NAND circuits for various functions.
Nisan and Shocken's book [@NisanShocken2005]  builds a computing system starting from NAND gates and ending with high-level programs and games ("NAND to Tetris"); see also the website [nandtotetris.org](https://www.nand2tetris.org/).

We defined the _size_ of a Boolean circuit in {{ref: booleancircdef}} to be the number of gates it contains. This is one of two conventions used in the literature. The other convention is to define the size as the number of _wires_ (equivalent to the number of gates plus the number of inputs).
This makes very little difference in almost all settings, but can affect the circuit size complexity of some "pathological examples" of functions such as the constant zero function that do not depend on much of their inputs.
