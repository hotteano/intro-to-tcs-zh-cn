```admonish warning title = ""
❗**页面施工中**: 目前状态: 翻译完成, 修复格式中. 正文开放debug, 需要精修.

待办: 

- ⬛️将所有numthm环境用灰色admonish(quote)框起.
- ⬛️标点符号统一为英文.
- ⬛️`<a>`标签换成`<span>`.
- ⬛️修复对functionprogramidea, secimplvsspec(第2章)的引用.
- ⬛️修复结尾传记部分的文献引用.
```

# 计算与表示 {#chaprepres }

```admonish quote title = ""
_"字母表是一项伟大的发明, 使人们能够轻松地储存并学习他人经过艰难努力才获得的知识 —— 也就是说, 可以通过书本学习, 而非通过与真实世界直接且可能痛苦的接触来学习. "_ B.F. Skinner 

_"这首歌的名字叫作 'HADDOCK'S EYES'."_ 骑士说道.

_"哦, 这就是歌的名字吗？"_ 爱丽丝如此问, 努力装作有兴趣. 

_"不, 你没明白, "_ 骑士有些恼火. _"这首歌只是名字被叫作这个. 这首歌的名字其实是 'THE AGED AGED MAN'. "_  

_"那我应该说, '这首歌被叫做这个'？"_ 爱丽丝认真想了想.   

_"不, 你不该那么说: 那完全是另一回事! 这首歌 **被叫作** 'WAYS AND MEANS', 但你知道, 那只是它被叫作这个而已! "_  

_"那么, 这首歌究竟 **是什么** 呢？"_ 爱丽丝问道, 此时她已经完全被搞糊涂了.   

_"我正要说到这点, "_ 骑士回答道. _"这首歌其实 **是** `A-SITTING ON A GATE', 而曲调是我自创的. "_  

Lewis Carroll, _爱丽丝镜中奇遇_  
```

## 学习目标 
* 区分规约与实现, 亦即区分数学函数与算法/程序.
* 将对象表示为字符串(通常由 0 和 1 构成).
* 常见对象(如自然数、向量、列表与图)的表示实例.
* 前缀无歧义编码.
* Cantor定理：实数无法被有限长字符串精确表示.

## 目录

<!-- toc -->

---

```admonish quote title=""
<span id="computationinputtooutputfig">![computationinputtooutputfig](./images/chapter2/input_output.png)</span>
我们对计算最基本的理解, 是把它看作一种将输入转化为输出的过程.
```

```admonish quote title=""
<span id="zerosandonesgreenfig">![zerosandonesgreenfig](./images/chapter2/zeroes-ones.jpg)</span>
我们用由 0 和 1 组成的字符串来表示数字, 文本, 图像, 网络以及许多其他对象. 当然, 将这些 0 和 1 本身以绿色字体写在黑色背景上也是可选的.
```

从初步的角度看, **计算** 是一个将 **输入** 映射为 **输出** 的过程.

在谈论计算时, 一个关键点是要区分两个问题: __需要完成的任务是什么__(即规约), 以及 __如何去实现这一任务__(即实现方式).
例如, 正如我们已经看到的, 计算两个整数的乘积这一任务, 并不只有唯一的一种实现方式.

在本章中, 我们将聚焦于 "**是什么**" 部分, 即如何定义计算任务. 而这首先要求我们明确定义 **输入与输出**.
要囊括所有可能的输入和输出似乎颇具挑战性, 因为如今计算已经被应用在各种各样的对象上, 不仅是数字, 还可以是文本, 图像, 视频, 例如社交网络的连接图, MRI 扫描结果, 基因组数据, 甚至是其它程序.

我们将尝试把所有这些对象表示为 **由 0 和 1 组成的字符串**, 也就是诸如 $0011101$, $1011$, 或任意有限个 $0$ 与 $1$ 组成的序列. (当然, 这样的选择只是出于方便, 0 和 1 并非 "神圣" 而不可替代: 我们完全可以用任何其他有限集合的符号来表示.)

![我们用由 0 和 1 组成的字符串来表示数字, 文本, 图像, 网络以及许多其他对象. 当然, 将这些 0 和 1 本身以绿色字体写在黑色背景上也是可选的.](./images/chapter2/zeroes-ones.jpg)

如今, 我们已经对数字化的表示习以为常, 因而并不会对这种编码的存在感到惊讶, 但这实际上是一个深刻的结果, 并带来了许多重要的影响.
许多动物也能够表达某种恐惧或欲望, 但人类独特之处在于 **语言**: 我们使用有限的一组基本符号来描述潜在无限范围的体验.
语言使得信息能够跨越时间与空间进行传递, 并让社会能够涵盖大量的人群, 随时间积累出共享的知识体系.

在过去的几十年里, 我们见证了一场关于数字化表示与传递的革命: 我们现在几乎可以完美地捕捉视觉与听觉的体验, 并几乎瞬间将其传播给无限的受众. 更重要的是, 一旦信息以数字形式存在, 我们便能够对其进行 **计算**, 并从中获取以往无法触及的数据洞见. 这场革命的核心, 是一个简单却深刻的观察: 我们能够用有限的一组符号 (事实上仅需两个符号 0 和 1) 来表示无穷多样的对象.

在后续的章节中, 我们通常会默认这种表示方法的存在, 因此会使用诸如 "程序 $P$ 以 $x$ 为输入" 这样的表述, 即便 $x$ 可能是一个数字、向量、图, 或者其他任意对象. 不过我们真正的意思是, $P$ 的输入实际上是 $x$ 的 **二进制字符串表示**.
在本章中, 我们会更深入地探讨如何构造这样的表示方法. 

本章的主要要点如下:

* 我们可以使用 **二进制字符串** 来表示所有我们想作为输入和输出的对象. 例如, 可以利用 **二进制基** 将整数和有理数表示为二进制字符串 (参见 naturalnumsec{.ref} 和 morerepressec{.ref}).

* 我们可以通过 **组合** 简单对象的表示, 来构造复杂对象的表示. 这样一来, 就可以表示整数或有理数的列表, 并进一步用来表示矩阵、图像和图等对象. **前缀无歧义编码** (prefix-free encoding) 是实现这种组合的一种方式 (参见 prefixfreesec{.ref}).

* 一个 **计算任务** 指定了从输入到输出的映射 -- 即一个 **函数**. 区分 "what" 与 "how", 或者说 **规约** (specification) 与 **实现** (implementation), 至关重要 (参见 secimplvsspec{.ref}). 一个函数仅仅定义了哪个输入对应哪个输出, 而并没有规定 **如何** 从输入计算出输出. 正如我们在乘法的例子中所看到的, 计算同一个函数可能存在多种方式.

* 虽然所有可能的二进制字符串的集合是无限的, 它仍然无法表示 **一切**. 特别地, 并不存在将 **实数** (绝对精确地) 表示为二进制字符串的方法. 这一结果也被称为 **Cantor定理** (Cantor's Theorem) (参见 cantorsec{.ref}), 通常表述为 "实数是不可数的". 这也暗示了无限还存在 _不同的层次_, 不过在本书中我们不会深入讨论这一话题 (参见 generalizepowerset{.ref}).

本章讨论的两个 "核心思想" 是: representtuplesidea{.ref} -- 我们可以通过组合简单对象的表示来表示更复杂的对象; 以及 functionprogramidea{.ref} -- 区分 函数 的 "what" 与 程序 的 "how" 至关重要. 后者将是本书中反复提到的一个主题.
:::

## 定义表示

每当我们在计算机中存储数字、图像、声音、数据库或其他对象时, 实际上存储在计算机内存中的只是这些对象的 **表示**.  
此外, "表示" 的概念并不限于电子计算机, 当我们写下文字或画一幅图时, 我们同样是在将思想或体验 **表示** 为符号序列 (这些符号也完全可以是由 0 和 1 构成的字符串), 甚至我们的脑中也并非储存真实的感官输入, 而是仅仅存储它们的 **表示**.  

为了在计算中使用数字、图像、图或其他对象作为输入, 我们需要精确定义如何将这些对象表示为二进制字符串.  
一个 **表示方案** (representation scheme) 就是将对象 $x$ 映射到一个二进制字符串 $E(x) \in \{0,1\}^*$ 的方法, 例如, 自然数的一个表示方案就是一个函数 $E:\N \rightarrow \{0,1\}^*$.  
当然, 我们不能把所有的数字都表示成相同的字符串 (比如 "$0011$"), 一个最基本的要求是, 如果两个数 $x$ 和 $x'$ 不同, 那么它们必须被表示为不同的字符串, 换句话说, 我们要求编码函数 $E$ 是 **一一对应** 的 (one-to-one).  

### 表示自然数

现在我们来展示如何将自然数表示为二进制字符串.  
多年来, 人们已经尝试了各种方式来表示数字, 包括绳结计数, 雅玛数字, 罗马数字, 我们熟悉的十进制, 以及许多其它方法. 我们当然可以使用其中任意一种将一个数字表示为字符串 (参见 [bitmapdigitsfig](){.ref}), 然而, 出于计算上的方便, 我们采用 **二进制基** 作为默认的自然数字符串表示法.  

例如, 我们将数字 6 表示为字符串 $110$, 因为 $1\cdot 2^{2} + 1 \cdot 2^1 + 0 \cdot 2^0 = 6$.

类似地, 我们将数字 35 表示为字符串 $y = 100011$, 它满足 $\sum_{i=0}^5 y_i \cdot 2^{|y|-i-1} = 35$.  

更多示例见下表.  


![将数字 0, 1, 2, ..., 9 的每个数字表示为一个 12×8 的位图图像, 该图像可以被视为属于 $\{0,1\}^\{96\}$ 的一个字符串. 使用这个方案, 我们可以把具有 $n$ 位十进制数字的自然数 $x$ 表示为属于 $\{0,1\}^\{96n\}$ 的一个字符串. 图片来源: [A. C. Andersen 的博客文章](http://blog.andersen.im/2010/12/autonomous-neural-development-and-pruning/).](./images/chapter2/digitsbitmap.png)


| **十进制表示** | **二进制表示** |
| ----------------------------------------- | ---------------------------------------- |
| 0                                         | 0                                        |
| 1                                         | 1                                        |
| 2                                         | 10                                       |
| 5                                         | 101                                      |
| 16                                        | 10000                                    |
| 40                                        | 101000                                   |
| 53                                        | 110101                                   |
| 389                                       | 110000101                                |
| 3750                                      | 111010100110                             |

表格: 使用二进制基表示数字. 左列包含自然数在十进制下的表示, 右列包含相同数字在二进制下的表示.  

如果 $n$ 是偶数, 那么 $n$ 的二进制表示的最低有效位为 $0$; 如果 $n$ 是奇数, 那么该位为 $1$.  
就像数字 $\floor{n/10}$ 对应于"去掉"最低有效的十进制位 (例如, $\floor{457/10}=\floor{45.7}=45$), 数字 $\floor{n/2}$ 对应于"去掉"最低有效的 **二进制** 位.  

因此, 二进制表示可以形式化定义为以下函数 $NtS:\N \rightarrow \{0,1\}^*$ ($NtS$ 表示 "natural numbers to strings"):  

$$
NtS(n) = \begin{cases}
            0    &  n=0 \\
            1    &  n=1 \\
            NtS(\floor{n/2}) parity(n) & n>1
\end{cases} \label{ntseq}
$$

其中, $parity:\N \rightarrow \{0,1\}$ 是函数, 定义为: 如果 $n$ 为偶数, 则 $parity(n)=0$; 如果 $n$ 为奇数, 则 $parity(n)=1$.  
像往常一样, 对于字符串 $x,y \in \{0,1\}^*$, $xy$ 表示字符串 $x$ 与 $y$ 的连接.  

函数 $NtS$ 是 **递归定义** 的: 对于每个 $n>1$, 我们通过较小的数字 $\floor{n/2}$ 的表示来定义 $rep(n)$.  
同样, 也可以用非递归方式定义 $NtS$, 参见 [binaryrepex](){.ref}.  

在本书的大部分内容中, 将数字表示为二进制字符串的具体选择并不重要: 我们只需要知道这样的表示是存在的.  
事实上, 对于许多用途, 我们甚至可以使用更简单的表示方法, 将自然数 $n$ 映射为长度为 $n$ 的全零字符串 $0^n$.  


::: {.remark title="二进制表示的Python实现 (选读)" #pythonbinary}
我们可以在 **Python** 中实现如下的二进制表示:

```python
def NtS(n):# 自然数(Natural number) to 字符串(String)
    if n > 1:
        return NtS(n // 2) + str(n % 2)
    else:
        return str(n % 2)

print(NtS(236))
# 11101100

print(NtS(19))
# 10011
```

我们一样可以使用 **Python** 实现逆向的转换: 将一个字符串映射回它表示的自然数.

```python
def StN(x):# 字符串 to 自然数
    k = len(x)-1
    return sum(int(x[i])*(2**(k-i)) for i in range(k+1))

print(StN(NtS(236)))
# 236


```

<iframe src="https://trinket.io/embed/python/fe91c91550" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>

:::

::: {.remark title="编程示例" #programmingrem}
在本书中, 我们有时会使用 **代码示例**, 如 [pythonbinary](){.ref}, 但它们的目的始终是强调某些计算可以被具体实现, 而不是为了展示 Python 或任何其他编程语言的特性.  
实际上, 本书传达的一个信息是, 所有编程语言在某种精确定义的意义下都是 **等价的**, 因此我们完全可以使用 JavaScript、C、COBOL、Visual Basic, 甚至 [BrainF*ck](https://goo.gl/LKKNFK)具体实现计算.

本书 **不是** 编程指南. 不熟悉 Python 或无法理解如 [pythonbinary](){.ref} 中的代码示例不会影响本书内容的学习.

:::

### 表示的意义(讨论)

初学时, 我们自然会认为 $236$ 是"实际"的数字, 而 $11101100$ 只是它的表示.  
然而, 对于中世纪的大多数欧洲人来说, `CCXXXVI` 才是"实际"的数字, 而 $236$(如果他们甚至听说过的话)则是奇怪的印度-阿拉伯位置记数法表示. ^[尽管巴比伦人早已发明了位置记数法, 我们今天使用的十进制位置记数法是印度数学家约在公元三世纪发明的, 再由阿拉伯数学家在八世纪采用与发展. 它在欧洲首次受到显著关注是在 1202 年 Fibonacci(又名 Leonardo of Pisa)出版的著作 _"Liber Abaci"_ 中,  但直到十五世纪, 它才在日常使用中取代罗马数字.]  
或许未来当我们的 AI 机器人统治者出现时, 它们可能会认为 $11101100$ 才是"实际"的数字, 而 $236$ 只是它们在向人类下达命令时需要使用的表示方法.  

那么, 什么才是"实际"的数字呢? 这是数学哲学家们自古以来一直思考的问题.  
柏拉图认为, 数学对象存在于某种理想的存在领域中 (在某种程度上比我们通过感官感知的世界更"真实", 因为后者不过是理想领域的影子).  
在柏拉图的视角中, 符号 $236$ 仅仅是某个理想对象的记号, 为了向 [已故音乐家](https://goo.gl/b93h83) 致敬, 我们可以称之为 "通常由 $236$ 表示的数字".  

而奥地利哲学家路德维希·维特根斯坦则认为, 数学对象根本不存在, 唯一存在的只有构成 $236$、$11101100$ 或 `CCXXXVI` 的实际纸上符号.  
在维特根斯坦看来, 数学仅仅是对没有固有意义的符号进行形式操作.  
你可以将"实际"的数字理解为(有些递归地)"$236$、$11101100$ 和 `CCXXXVI` 以及所有旨在表示同一对象的过去和未来的表示方式共同指向的那个东西".  

阅读本书时, 你可以自由选择自己的数学哲学, 只要你能区分数学对象本身与表示它们的各种具体方式, 无论是墨迹斑点、屏幕上的像素、零和一, 还是任何其他形式.

## 自然数以外对象的表示

我们已经看到, 自然数可以表示为二进制字符串. 而现在我们将展示, 这对于其他类型的对象也同样适用, 包括(可能为负的)整数、有理数、向量、列表、图以及许多其他对象.  

在很多情况下, 为一条数据选择"合适的"字符串表示是非常复杂的任务, 寻找"最佳"表示(例如, 最紧凑, 保真度最高, 最易操作、鲁棒性强(抗干扰能力强), 信息量最大等)一直都是研究的热点.  

但目前, 我们先专注于展示一些简单的表示方法, 用于将各种对象作为计算的输入和输出.

### 表示带有负数的全体整数

既然我们可以将自然数表示为字符串, 我们也可以基于此表示 **整数** 的全集 (即集合 $\Z=\{ \ldots, -3 , -2 , -1 , 0 , +1, +2, +3,\ldots \}$ 的成员), 只需增加一位用于表示符号.  

为了表示一个(可能为负的)数字 $m$, 我们在自然数 $|m|$ 的表示前加上一个比特 $\sigma$, 若 $m \geq 0$ 则 $\sigma=0$, 若 $m<0$ 则 $\sigma=1$.  

形式上, 我们将函数 $ZtS:\Z \rightarrow \{0,1\}^*$ 定义如下:

$$
ZtS(m) = \begin{cases}
0\;NtS(m) & m \geq 0  \\
1\;NtS(-m) & m < 0
\end{cases}
$$

其中, $NtS$ 的定义如 [ntseq](){.eqref} 所示.  

虽然表示的编码函数必须是一一对应的, 但不必是 **满射**.  
例如, 在上述表示法中, 没有任何数字被表示为空字符串, 但这仍然是有效的表示方法, 因为每个整数都能被唯一地表示为某个字符串.  

给定一个字符串 $y\in \{0,1\}^*$, 我们如何判断它"应该"表示一个(非负的)自然数还是一个(可能为负的)整数?  
更进一步, 即便我们知道 $y$ "应该"是一个整数, 我们又如何知道它使用的是哪种表示方案?  
事实上, 除非上下文提供该信息, 否则我们不一定知道. (在编程语言中, 编译器或解释器会根据变量的 **类型** 决定对应变量的比特序列的表示方法.)  

我们可以将同一个字符串 $y$ 视作表示自然数、整数、一段文本、一幅图像, 或者一个绿色的小妖精.  
每当我们说类似 "令 $n$ 为字符串 $y$ 表示的数字" 这样的句子时, 我们假设固定某种规范表示方案, 比如上文所示的那些.  
具体选择哪种表示方案通常无关紧要, 只需要确保在使用时保持一致即可.


### 补码表示(选读)

[repnegativeintegerssec](){.ref} 中使用特定的"符号位"来表示整数的方法被称为 **有符号数表示法 (Signed Magnitude Representation)**, 曾在一些早期计算机中使用.  
然而, [二进制补码表示](https://en.wikipedia.org/wiki/Two%27s%5Fcomplement) 在实际中更为常见.  

整数 $k$ 在集合 $\{ -2^n , -2^n+1, \ldots, 2^n-1 \}$ 的 **二进制补码表示** 是长度为 $n+1$ 的字符串 $ZtS_n(k)$, 定义如下:

$$
ZtS_n(k) = \begin{cases} NtS_{n+1}(k) & 0 \leq k \leq 2^n-1 \\
                     NtS_{n+1}(2^{n+1}+k) & -2^n \leq k \leq -1 \end{cases} \;,
$$

其中, $NtS_\ell(m)$ 表示数字 $m \in \{0,\ldots, 2^{\ell}\}$ 的标准二进制表示, 作为长度为 $\ell$ 的字符串, 并根据需要用前导零填充.  
例如, 如果 $n=3$, 则 $ZtS_3(1)=NtS_4(1)=0001$, $ZtS_3(2)=NtS_4(2)=0010$, $ZtS_3(-1)=NtS_4(16-1)=1111$, 而 $ZtS_3(-8)=NtS_4(16-8)=1000$.  
如果 $k$ 是大于或等于 $-2^n$ 的负数, 那么 $2^{n+1}+k$ 是一个位于 $2^n$ 和 $2^{n+1}-1$ 之间的数字.  
因此, 该数字 $k$ 的二进制补码表示是长度为 $n+1$ 的字符串, 其首位为 $1$.  

换句话说, 我们将一个可能为负的数字 $k \in \{ -2^n,\ldots, 2^n-1 \}$ 表示为非负数 $k \mod 2^{n+1}$ (参见 [twoscomplementfig](){.ref}).  
这意味着, 如果两个可能为负的数字 $k$ 和 $k'$ 不太大 (即 $ k + k' \in \{ -2^n,\ldots, 2^n-1 \}$), 那么我们可以通过将 $k$ 和 $k'$ 的表示当作非负整数来进行模 $2^{n+1}$ 加法, 从而得到 $k+k'$ 的表示.  
二进制补码表示的这一特性是其主要优势, 因为根据微处理器的架构, 它们通常可以非常高效地执行模 $2^w$ 的算术运算(对于某些 $w$ 值, 如 32 或 64).  

许多系统将检查值是否过大留给程序员, 无论数字大小如何, 系统都会执行这种模运算.  
因此, 在某些系统中, 两个大的正数相加可能得到一个 **负数** (例如, 将 $2^n-100$ 与 $2^n-200$ 相加可能得到 $-300$, 因为 $(2^{n+1}-300) \mod 2^{n+1} = -300$, 参见 [twoscomplementfig](){.ref}).  


![In the _two's complement representation_  we represent a potentially negative integer $k \in \{ -2^n ,\ldots, 2^n-1 \}$ as an $n+1$ length string using the binary representation of the integer $k \mod 2^{n+1}$. On the left-hand side: this representation for $n=3$ (the red integers are the numbers being represented by the blue binary strings). If a microprocessor does not check for overflows, adding the two positive numbers $6$ and $5$ might result in the negative number $-5$ (since $-5 \mod 16 = 11$. The right-hand side is a `C` program that will on some $32$ bit architecture print a negative number after adding two positive numbers. (Integer overflow in `C` is considered _undefined behavior_ which means the result of this program, including whether it runs or crashes, could differ depending on the architecture, compiler, and even compiler options and version.)](./images/chapter2/twoscomplement.png)
### 有理数及字符串表示对

我们可以通过表示两个数字 $a$ 和 $b$ 来表示分数形式的有理数 $a/b$.  
然而, 仅仅将 $a$ 和 $b$ 的表示简单连接起来是行不通的.  
例如, 数字 $4$ 的二进制表示是 $100$, 数字 $43$ 的二进制表示是 $101011$, 但将它们简单连接得到的字符串 $100101011$ 也可以看作是 $18$ 的表示 $10010$ 与 $11$ 的表示 $1011$ 的连接.  
因此, 如果使用这种简单连接方式, 我们将无法判断字符串 $100101011$ 是表示 $4/43$ 还是 $18/11$.  

我们通过给 **字符串对** 提供通用表示来解决这个问题.  
如果使用纸笔, 我们只需使用一个分隔符号如 $\|$, 将表示数字 $10$ 和 $110001$ 的一对数字表示为长度为 9 的字符串 "$10\|110001$".  
换句话说, 存在一个一一对应的映射 $F$, 将 **字符串对** $x,y \in \{0,1\}^*$ 映射为一个在字母表 $\Sigma = \{0,1,\|\}$ 上的单个字符串 $z$ (即 $z \in \Sigma^*$).  
使用分隔符类似于英语中使用空格和标点来分隔单词.  
通过增加少量冗余, 我们可以在数字领域实现同样的效果.  
我们可以将三元素集合 $\Sigma$ 映射到三元素集合 $\{00,11,01\} \subset \{0,1\}^2$ 并保持一一对应, 从而将长度为 $n$ 的字符串 $z \in \Sigma^*$ 编码为长度为 $2n$ 的字符串 $w \in \{0,1\}^*$.  

我们对有理数的最终表示通过以下步骤组合得到:  

1. 将一个(可能为负的)有理数表示为一对整数 $a,b$, 使得 $r = a/b$.  
2. 将整数表示为二进制字符串.  
3. 将步骤 1 和 2 结合, 得到有理数作为字符串对的表示.  
4. 将 $\{0,1\}$ 上的字符串对表示为 $\Sigma = \{0,1,\|\}$ 上的单个字符串.  
5. 将 $\Sigma$ 上的字符串表示为更长的 $\{0,1\}$ 字符串.  


::: {.example title="将一个有理数表示为字符串" #represnumberbypairs}
考虑有理数 $r=-5/8$.  
我们将 $-5$ 表示为 $1101$, $+8$ 表示为 $01000$, 因此可以将 $r$ 表示为字符串对 $(1101,01000)$, 并将该字符串对表示为字母表 $\{0,1,\|\}$ 上长度为 10 的字符串 $1101\|01000$.  

现在, 通过映射 $0 \mapsto 00$, $1 \mapsto 11$, $\| \mapsto 01$, 我们可以将该字符串表示为字母表 $\{0,1\}$ 上长度为 20 的字符串 $s=11110011010011000000$.  

:::

同样的思想可以用来表示字符串三元组、四元组, 甚至更多, 作为单个字符串.  
实际上, 这是一个非常通用的原则的实例, 我们会在计算机科学的理论与实践中反复使用它(例如, 在面向对象编程中):  

::: { .bigidea #representtuplesidea }
如果我们可以将类型为 $T$ 的对象表示为字符串, 那么我们也可以将类型为 $T$ 的对象元组表示为字符串.  
:::

重复同样的思想, 一旦我们可以表示类型为 $T$ 的对象, 我们也可以表示这些对象的 **列表的列表**, 甚至是列表的列表的列表, 如此类推.  
当我们讨论 [prefixfreesec](){.ref} 中的 **前缀无歧义编码 (prefix free encoding)** 时, 我们会再次回到这一点.  


## 实数的表示

**实数集** $\R$ 包含所有正数、负数、分数，以及像 $\pi$ 或 $e$ 这样的 **无理数**.  
每个实数都可以用有理数近似, 因此我们可以用一个接近 $x$ 的有理数 $a/b$ 来表示实数 $x$.  
例如, 我们可以用 $22/7$ 来表示 $\pi$, 误差约为 $10^{-3}$. 若希望误差更小(例如约 $10^{-4}$)，可以使用 $311/99$, 以此类推.  

![实数 $x\in \R$ 的浮点表示](./images/chapter2/floatingpoint.png)  
实数通过近似有理数来表示是一个可行的表示方案.  

然而, 在计算机应用中, 通常更常用 **浮点表示法** (参见 [floatingpointfig](){.ref}) 来表示实数.  
在浮点表示法中, 我们用一对 $(b,e)$ 表示 $x \in \R$, 其中 $b$ 和 $e$ 是某些规定长度的(可能为正或负的)整数, 并且 $b \times 2^{e}$ 最接近 $x$.  
浮点表示是 [科学计数法](https://goo.gl/MUJnVE) 的二进制版本, 即将一个数字 $y \in \R$ 表示为 $b \times 10^e$ 的近似.  
称之为"浮点"是因为可以将 $b$ 看作指定一串二进制数字, $e$ 描述这串数字中"二进制小数点"的位置.  

正是浮点表示的使用, 导致许多编程系统中, 表达式 `0.1+0.2` 的输出为 `0.30000000000000004` 而不是 `0.3`.  
更多信息可见: [这里](http://floating-point-gui.de/), [这里](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html), [这里](https://randomascii.wordpress.com/2012/04/05/floating-point-complexities/).  


![XKCD cartoon on floating-point arithmetic.](./images/chapter2/e_to_the_pi_minus_pi.png)

读者可能会(合理地)担心, 浮点表示法(或有理数表示法)只能 **近似** 表示实数.  
在许多(但不是全部)计算应用中, 可以将精度调得足够高, 以至于不会影响最终结果.  

但有时我们仍需要谨慎. 事实上, 浮点数错误有时可能造成严重后果.  
例如, 浮点舍入误差曾导致美国爱国者导弹未能拦截伊拉克飞毛腿导弹, 造成 28 人死亡 ([详细报道](http://embeddedgurus.com/barr-code/2014/03/lethal-software-defects-patriot-missile-failure/)), 以及在计算 [英国养老金发放金额](https://catless.ncl.ac.uk/Risks/5/74) 时出现过的 1 亿英镑的错误.

## Cantor定理, 可数集, 以及实数的字符串表示

::: {.quote}
_"对于任意一组水果, 我们可以制作的水果沙拉数量总可以比水果数量更多. 如果不是这样, 我们可以给每个沙拉贴上一个不同水果的标签, 最后再考虑这样一个沙拉, 它包含所有未被标签所指的水果, 那么某个水果恰好在这个沙拉的标签中当且仅当它不在其中."_ — [Martha Storey](https://twitter.com/JDHamkins/status/1266278627018043392)

:::

鉴于浮点数对实数的近似问题, 一个自然的问题是: 是否可以将实数 **精确地** 表示为字符串.  
不幸的是, 下述定理表明这是不可能的:  

不存在一一对应的函数 $RtS:\R \rightarrow \{0,1\}^*$.^[其中 $RtS$ 代表 "real numbers to strings".]  

__可数集.__ 我们说一个集合 $S$ 是 **可数的**, 如果存在一个满射 $C:\N \rightarrow S$, 或者换句话说, 我们可以将 $S$ 写成序列  
$C(0), C(1), C(2), \ldots$.  
由于二进制表示给出了从 $\{0,1\}^*$ 到 $\N$ 的满射, 并且两个满射的复合仍然是满射, 集合 $S$ 是可数的当且仅当存在从 $\{0,1\}^*$ 到 $S$ 的满射.
利用函数的基本性质(见 [functionsec](){.ref}), 一个集合可数当且仅当存在从 $S$ 到 $\{0,1\}^*$ 的一一函数.  

因此, 我们可以将 [cantorthm](){.ref} 重述如下:  

**实数是不可数的**. 也就是说, 不存在从 $\N$ 到 $\R$ 的满射 $NtR:\N \rightarrow \R$.  

[cantorthmtwo](){.ref} 由 [Georg Cantor](https://en.wikipedia.org/wiki/Georg_Cantor) 于 1874 年证明.  
这一结果(以及相关结论)震惊了当时的数学家. 通过证明不存在从 $\R$ 到 $\{0,1\}^*$(或 $\N$)的一一映射, Cantor 展示了这两个无限集合有"不同的无限形式", 并且实数集 $\R$ 在某种意义上比无限集合 $\{0,1\}^*$ "更大".  
"无限的层次"这一概念当时让数学家和哲学家深感困惑. 哲学家 Ludwig Wittgenstein(前面提到过)称 Cantor 的结果为"完全的胡扯"且"可笑", 其他人甚至认为更糟: Leopold Kronecker 称 Cantor 是"腐蚀青年的人", 而 Henri Poincaré 说 Cantor 的思想"应从数学中彻底剔除".
不过事实证明 Cantor 看得更远. 如今 Cantor 的工作已被普遍接受为集合论和数学基础的基石.  
正如 David Hilbert 在 1925 年所说, _"无人能将我们从 Cantor 为我们创造的天堂中驱逐出去"_.  
也正如我们稍后将在本书中看到的, Cantor 的思想在计算理论中也起着重要作用.  

我们已经讨论了 [cantorthm](){.ref} 的重要性, 让我们来看看它的证明. 这将分两步进行:  

1. 定义一个无限集合 $\mathcal{X}$, 对于它证明不可数更加容易(即证明不存在从 $\mathcal{X}$ 到 $\{0,1\}^*$ 的一一函数更容易).  
2. 证明存在一个一一函数 $G$ 将 $\mathcal{X}$ 映射到 $\mathbb{R}$.  

利用反证法, 这两条事实结合起来可以推出 [cantorthm](){.ref}.  
具体来说, 如果假设(为了反证)存在某个一一函数 $F$ 将 $\mathbb{R}$ 映射到 $\{0,1\}^*$,  
那么通过将 $F$ 与步骤 2 中的函数 $G$ 复合得到的函数 $x \mapsto F(G(x))$ 就是从 $\mathcal{X}$ 到 $\{0,1\}^*$ 的一一函数,  
这与步骤 1 中的结论矛盾!  

为了将这个想法完整地转化为 [cantorthm](){.ref} 的证明, 我们需要:  

* 定义集合 $\mathcal{X}$.  
* 证明不存在从 $\mathcal{X}$ 到 $\{0,1\}^*$ 的一一函数.  
* 证明存在从 $\mathcal{X}$ 到 $\R$ 的一一函数.  

接下来我们将精确地做到这些:  
我们将定义集合 $\{0,1\}^\infty$, 它将扮演 $\mathcal{X}$ 的角色,  
然后陈述并证明两个引理, 说明该集合满足我们所需的两个性质.


::: {.definition #bitsinfdef}
将 $\{0,1\}^\infty$ 定义为集合 $\{ f \;|\; f:\N \rightarrow \{0,1\} \}$.
:::

简单来说, $\{0,1\}^\infty$ 是一个 **函数的集合**, 并且一个函数 $f$ 属于 $\{0,1\}^\infty$ 当且仅当它的定义域是 $\N$ 而值域是 $\{0,1\}$.  
我们可以将 $\{0,1\}^\infty$ 理解为所有无限长 **比特序列** 的集合, 因为函数 $f:\N \rightarrow \{0,1\}$ 正好一一对应于无限序列 $(f(0), f(1), f(2), \ldots)$.

下面两个引理说明, $\{0,1\}^\infty$ 可以作为 $\mathcal{X}$ 来证明 [cantorthm](){.ref}:  

* 不存在从 $\{0,1\}^\infty$ 到 $\{0,1\}^*$ 的一一映射 $FtS$.^[$FtS$ 代表 "functions to strings".]  
* 存在从 $\{0,1\}^\infty$ 到 $\R$ 的一一映射 $FtR$.^[$FtR$ 代表 "functions to reals".]  

如上所示, [sequencestostrings](){.ref} 和 [sequencestoreals](){.ref} 结合起来即可推出 [cantorthm](){.ref}.  
为了更正式地重复这一论证, 为了反证, 假设存在一一函数 $RtS:\R \rightarrow \{0,1\}^*$.  
由 [sequencestoreals](){.ref}, 存在一一函数 $FtR:\{0,1\}^\infty \rightarrow \R$.  
因此, 根据假设, 由于两个一一函数的复合仍是一一函数（见 [onetoonecompex](){.ref}）,  
函数 $FtS:\{0,1\}^\infty \rightarrow \{0,1\}^*$ 定义为 $FtS(f)=RtS(FtR(f))$ 将是一一函数,  
这与 [sequencestostrings](){.ref} 矛盾.  
参见 [proofofcantorfig](){.ref} 获取该论证的图示说明.


![We prove [cantorthm](){.ref} by combining [sequencestostrings](){.ref} and [sequencestoreals](){.ref}.  [sequencestoreals](){.ref}, which uses standard calculus tools, shows the existence of a one-to-one map $FtR$ from the set $\{0,1\}^\infty$ to the real numbers. So, if a hypothetical one-to-one map $RtS:\R \rightarrow \{0,1\}^*$ existed, then we could compose them to get a one-to-one map $FtS:\{0,1\}^\infty \rightarrow \{0,1\}^*$. Yet this contradicts [sequencestostrings](){.ref}- the heart of the proof- which rules out the existence of such a map.](./images/chapter2/proofofcantor.png)

现在只剩下证明这两个引理.
我们先从证明 [sequencestostrings](){.ref} 开始, 这实际上是 [cantorthm](){.ref} 的核心部分.

![We construct a function $\overline{d}$ such that $\overline{d} \neq StF(x)$ for every $x\in \{0,1\}^*$ by ensuring that $\overline{d}(n(x)) \neq StF(x)(n(x))$ for every $x\in \{0,1\}^*$ with lexicographic order $n(x)$. We can think of this as building a table where the columns correspond to numbers $m\in \N$ and the rows correspond to $x\in \{0,1\}^*$ (sorted according to $n(x)$). If the entry in the $x$-th row and the $m$-th column corresponds to $g(m))$ where $g=StF(x)$ then $\overline{d}$ is obtained by going over the "diagonal" elements in this table (the entries corresponding to the $x$-th row and $n(x)$-th column) and ensuring that $\overline{d}(n(x)) \neq StF(x)(n(x))$. ](./images/chapter2/diagonalization.png)

__Warm-up: "Baby Cantor".__ [sequencestostrings](){.ref} 的证明相当微妙. 一种获得直觉的方法是考虑以下有限版本的陈述: "不存在一个满射函数 $f:\{0,\ldots,99\} \rightarrow \{0,1\}^{100}$". 当然我们知道这是正确的, 因为集合 $\{0,1\}^{100}$ 比集合 $[100]$ 更大, 但让我们来看一个不太直接的证明: 对于任意 $f:\{0,\ldots,99\} \rightarrow \{0,1\}^{100}$, 我们可以定义字符串 $\overline{d} \in \{0,1\}^{100}$ 如下: $\overline{d} = (1-f(0)_0, 1-f(1)_1 , \ldots, 1-f(99)_{99})$. 如果 $f$ 是满射, 那么必然存在某个 $n\in [100]$ 使得 $f(n) =\overline{d}$, 但我们声称不存在这样的 $n$. 实际上, 如果存在这样的 $n$, 那么 $\overline{d}$ 的第 $n$ 个分量应当等于 $f(n)_n$, 但根据定义这个分量等于 $1-f(n)_n$. 另见此陈述的 [“proof by code”](https://trinket.io/python/4cff7e58f4).

<iframe src="https://trinket.io/embed/python/4cff7e58f4" width="100%" height="600" frameborder="0" marginwidth="0" marginheight="0" allowfullscreen></iframe>

::: {.proof data-ref="sequencestostrings"}
我们将证明不存在一个 **满射** 函数 $StF:\{0,1\}^* \rightarrow \{0,1\}^\infty$.  
这将推出该引理, 因为对于任意两个集合 $A$ 和 $B$, 当且仅当存在一个从 $B$ 到 $A$ 的一一映射时, 才存在一个从 $A$ 到 $B$ 的满射 (见 [onetooneimpliesonto](){.ref}).  

这个证明技巧被称为 "diagonal argument" (对角线论证), 详情可见 [diagrealsfig](){.ref}.  
为了得到矛盾, 我们假设存在这样一个函数 $StF:\{0,1\}^* \rightarrow \{0,1\}^\infty$. 然后我们通过构造一个函数 $\overline{d}\in \{0,1\}^\infty$, 使得对每个 $x\in \{0,1\}^*$ 都有 $\overline{d} \neq StF(x)$, 来证明 $StF$ 不是满射.

考虑二进制字符串的字典序排列 (即 "", $0$, $1$, $00$, $01$, $\ldots$).  
对于每个 $n\in \N$, 我们令 $x_n$ 为此顺序中的第 $n$ 个字符串.  
也就是说 $x_0 = ""$, $x_1 = 0$, $x_2 = 1$ 等等.  
对每个 $n\in \N$, 我们定义函数 $\overline{d} \in \{0,1\}^\infty$ 如下:

$$
\overline{d}(n) = 1 - StF(x_n)(n)
$$

也就是说, 为了计算 $\overline{d}$ 在输入 $n\in\N$ 时的值, 我们首先计算 $g= StF(x_n)$, 其中 $x_n \in \{0,1\}^*$ 是字典序中的第 $n$ 个字符串.  
由于 $g \in \{0,1\}^\infty$, 它是一个将 $\N$ 映射到 $\{0,1\}$ 的函数.  
值 $\overline{d}(n)$ 被定义为 $g(n)$ 的取反.  

函数 $\overline{d}$ 的定义有些微妙.  
一种理解方式是将函数 $StF$ 想象为由一张无限长的表格指定, 其中每一行对应一个字符串 $x\in \{0,1\}^*$ (字符串按字典序排列), 并包含序列 $StF(x)(0), StF(x)(1), StF(x)(2),\ldots$.  
然后, 我们取该表格中的 **对角线** 元素如下:

$$
StF("")(0),StF(0)(1),StF(1)(2),StF(00)(3), StF(01)(4),\ldots
$$

这些元素对应于表格中第 $n$ 行第 $n$ 列的 $StF(x_n)(n)$, 对于 $n=0,1,2,\ldots$.  
我们上面定义的函数 $\overline{d}$ 将每个 $n\in \N$ 映射到第 $n$ 个对角线元素的取反值.  

为了完成 $StF$ 不是满射的证明, 我们需要说明对每个 $x\in \{0,1\}^*$ 都有 $\overline{d} \neq StF(x)$.  
事实上, 令 $x\in \{0,1\}^*$ 为某个字符串, 并令 $g = StF(x)$.  
如果 $n$ 是 $x$ 在字典序中的位置, 则根据构造有 $\overline{d}(n) = 1-g(n) \neq g(n)$, 这意味着 $g \neq \overline{d}$, 这正是我们需要的.
:::

::: {.remark title="推广到字符串或实数以外" #generalizepowerset}
[sequencestostrings](){.ref} 实际上与自然数或字符串没有太大关系.  
仔细审视这个证明可以发现, 它实际上说明对于 **任意** 集合 $S$, 不存在一个一一映射 $F:\{0,1\}^S \rightarrow S$, 其中 $\{0,1\}^S$ 表示所有以 $S$ 为定义域的布尔函数的集合 $\{ f \;|\; f:S \rightarrow \{0,1\} \}$.  
由于我们可以将子集 $V \subseteq S$ 与其特征函数 $f=1_V$ 对应 (即 $1_V(x)=1$ 当且仅当 $x\in V$), 我们也可以将 $\{0,1\}^S$ 看作 $S$ 的所有 **子集** 的集合.  
这个子集集合有时被称为 $S$ 的 **幂集**, 记作 $\mathcal{P}(S)$ 或 $2^S$.  

[sequencestostrings](){.ref} 的证明可以推广, 说明不存在一个集合与其幂集之间的一一映射.  
特别地, 这意味着集合 $\{0,1\}^\R$ “比” $\R$ 更大.  
Cantor 利用这些思想构建了无限的无穷层级.  
这些无穷的数量远大于 $|\N|$ 甚至 $|\R|$.  
他将 $\N$ 的基数记作 $\aleph_0$, 并将下一个更大的无限数记作 $\aleph_1$ ($\aleph$ 是希伯来字母表的第一个字母).  
Cantor 还提出了 [连续统假设](https://en.wikipedia.org/wiki/Continuum_hypothesis), 即 $|\R|=\aleph_1$.  
我们将在本书后续回到这个假设背后的精彩故事.  
[Aaronson 的这节讲座](https://www.scottaaronson.com/democritus/lec2.html) 提到了一些相关问题 (另见 [Berkeley CS 70 lecture](http://www.fa19.eecs70.org/static/notes/n10.pdf)).

:::

为了完成 [cantorthm](){.ref} 的证明, 我们需要证明 [sequencestoreals](){.ref}.  
这个证明虽然需要一些微积分基础, 但使用了的地方都比较直接易懂.  
不过如果你之前处理实数列极限的经验不多, 那么下面的证明还是可能会有些难以理解.  
当然, 这部分并非 Cantor 论证的核心, 此类极限对于本书后续内容也不重要, 因此你完全可以选择相信 [sequencestoreals](){.ref} 并跳过这些繁琐的证明.

::: {.proofidea data-ref="sequencestoreals"}
我们定义 $FtR(f)$ 为介于 $0$ 和 $2$ 之间的数, 其十进制展开为 $f(0).f(1) f(2) \ldots$, 换句话说, $FtR(f) = \sum_{i=0}^{\infty} f(i) \cdot 10^{-i}$.  
如果 $f$ 和 $g$ 是 $\{0,1\}^\infty$ 中的两个不同函数, 那么必然存在某个输入 $k$ 使它们在该输入上不一致.  
取最小的这样的 $k$, 那么数字 $f(0).f(1) f(2) \ldots f(k-1) f(k) \ldots$ 与 $g(0).g(1) g(2) \ldots g(k) \ldots$ 在小数点后的第 $0$ 到 $k-1$ 位完全相同, 并在第 $k$ 位上不同.  
因此这些数字必然不同.  
具体来说, 如果 $f(k)=1$ 且 $g(k)=0$, 则第一个数字大于第二个; 否则 ($f(k)=0$ 且 $g(k)=1$) 第一个数字小于第二个.  
在证明中我们需要稍微注意, 因为某些数字可以被 **无限展开**, 例如, 数字 $\frac{1}{2}$ 有两种十进制展开: $0.5$ 和 $0.49999\cdots$.  
但在这里不会出现这个问题, 因为按上述定义, 我们使用的数字的十进制展开中永远不会包含数字 $9$.

:::

::: {.proof data-ref="sequencestoreals"}
对于每个 $f \in \{0,1\}^\infty$, 我们定义 $FtR(f)$ 为其十进制展开为 $f(0).f(1)f(2)f(3)\ldots$ 的数字.  
形式上,

$$
FtR(f) = \sum_{i=0}^\infty f(i) \cdot 10^{-i} \label{eqcantordecimalexpansion}
$$

在微积分中有一个已知结论(这里我们不重复证明): [eqcantordecimalexpansion](){.eqref} 右侧的级数在 $\mathbb{R}$ 中收敛到一个确定的极限.  

现在我们证明 $FtR$ 是一一映射.  
设 $f,g$ 是 $\{0,1\}^\infty$ 中的两个不同函数.  
由于 $f$ 和 $g$ 不同, 必然存在某个输入它们的值不同, 我们令 $k$ 为最小的这样的输入, 并且不失一般性地假设 $f(k)=0$ 且 $g(k)=1$.  
(否则, 如果 $f(k)=1$ 且 $g(k)=0$, 我们可以简单地交换 $f$ 和 $g$ 的角色.)  
数字 $FtR(f)$ 和 $FtR(g)$ 在小数点后的前 $k-1$ 位完全相同.  
由于这第 $k$ 位在 $FtR(f)$ 中为 $0$ 而在 $FtR(g)$ 中为 $1$, 我们声称 $FtR(g)$ 比 $FtR(f)$ 至少大 $0.5 \cdot 10^{-k}$.  
要理解这一点, 注意 $FtR(g)-FtR(f)$ 的差值在以下情况下最小: 对于所有 $\ell>k$, $g(\ell)=0$ 且 $f(\ell)=1$, 此时(由于 $f$ 和 $g$ 在前 $k-1$ 位相同)


$$
FtR(g)-FtR(f) = 10^{-k} - 10^{-k-1} - 10^{-k-2} - 10^{-k-3} - \cdots \label{eqcantordecimalexpansion2}
$$

由于无穷级数 $\sum_{i=0}^{\infty} 10^{-i}$ 收敛到 $10/9$, 可得对于每一对这样的 $f$ 和 $g$, $FtR(g) - FtR(f) \geq 10^{-k} - 10^{-k-1}\cdot (10/9) > 0$.  
特别地, 我们看到对于每一对不同的 $f,g \in \{0,1\}^\infty$, $FtR(f) \neq FtR(g)$, 从而函数 $FtR$ 是一一映射.

:::

::: {.remark title="十进制展开的使用(选读)" #decimal}
在上面的证明中, 我们使用了级数 $1 + 1/10 + 1/100 + \cdots$ 收敛到 $10/9$ 的事实, 将其代入 [eqcantordecimalexpansion2](){.eqref} 可得 $FtR(g)$ 与 $FtR(h)$ 的差值至少为 $10^{-k} - 10^{-k-1}\cdot (10/9) > 0$.  
虽然我们为 $FtR$ 选择的十进制表示是任意的, 但我们不能用二进制表示代替.  
如果使用 **二进制** 展开而非十进制, 相应的级数 $1 + 1/2 + 1/4 + \cdots$ 收敛到 $2/1=2$, 并且由于 $2^{-k} = 2^{-k-1} \cdot 2$, 我们无法推导出 $FtR$ 是一一映射.  
事实上, 确实存在一些不同的序列对 $f,g\in \{0,1\}^\infty$ 满足 $\sum_{i=0}^\infty f(i)2^{-i} = \sum_{i=0}^\infty g(i)2^{-i}$.  
(例如, 序列 $1,0,0,0,\ldots$ 与序列 $0,1,1,1,\ldots$ 就具有此性质.)

:::

### 推论: 布尔函数全体不可数.

Cantor 定理得出如下推论, 我们将在本书中多次使用: 所有 **布尔函数**（将 $\{0,1\}^*$ 映射到 $\{0,1\}$ 的函数）构成的集合是不可数的.  

设 $ALL$ 为所有函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 的集合.  
则 $ALL$ 是不可数的. 等价地, 不存在一个满射 $StALL:\{0,1\}^* \rightarrow ALL$.  

这是 [sequencestostrings](){.ref} 的直接推论, 因为我们可以用二进制表示构造一个从 $\{0,1\}^\infty$ 到 $ALL$ 的一一映射. 因此, $\{0,1\}^\infty$ 的不可数性意味着 $ALL$ 的不可数性.


::: {.proof data-ref="uncountalbefuncthm"}
由于 $\{0,1\}^\infty$ 是不可数的, 我们只需展示一个从 $\{0,1\}^\infty$ 到 $ALL$ 的一一映射, 便可得到该结论.  
原因在于, 这样的映射存在意味着如果 $ALL$ 是可数的, 从而存在一个从 $ALL$ 到 $\N$ 的一一映射, 那么就会存在一个从 $\{0,1\}^\infty$ 到 $\N$ 的一一映射, 与 [sequencestostrings](){.ref} 矛盾.  

现在我们展示这个一一映射. 我们简单地将一个函数 $f \in \{0,1\}^\infty$ 映射到函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 如下.  
我们令 $F(0)=f(0)$, $F(1)=f(1)$, $F(10)=f(2)$, $F(11)=f(3)$ 等等.  
也就是说, 对于每个 $x\in \{0,1\}^*$, 如果它在二进制下表示自然数 $n$, 我们定义 $F(x)=f(n)$.  
如果 $x$ 不表示这样的数字（例如, 它有前导零）, 则我们令 $F(x)=0$.  

这个映射是一一映射, 因为如果 $f \neq g$ 是 $\{0,1\}^\infty$ 中的两个不同元素, 那么必然存在某个输入 $n\in \N$ 使 $f(n) \neq g(n)$.  
于是, 如果 $x\in \{0,1\}^*$ 是表示 $n$ 的字符串, 我们看到 $F(x) \neq G(x)$, 其中 $F$ 是 $f$ 映射到的 $ALL$ 中的函数, 而 $G$ 是 $g$ 映射到的函数.
:::

### 可数性的等价条件.

上述结果建立了多种等价的方式来表述集合可数的事实.  
具体来说, 以下陈述都是等价的:  

1. 集合 $S$ 是可数的  
2. 存在一个从 $\N$ 到 $S$ 的满射  
3. 存在一个从 $\{0,1\}^*$ 到 $S$ 的满射  
4. 存在一个从 $S$ 到 $\N$ 的一一映射  
5. 存在一个从 $S$ 到 $\{0,1\}^*$ 的一一映射  
6. 存在一个从某个可数集合 $T$ 到 $S$ 的满射  
7. 存在一个从 $S$ 到某个可数集合 $T$ 的一一映射


::: { .pause }
你确定你会证明上述所有等价陈述了吗?
:::

## 数字以外元素的表示

当然, 数字并不是我们唯一可以表示为二进制字符串的对象.  
用于表示某个集合 $\mathcal{O}$ 中对象的 **表示方案** 由一个将 $\mathcal{O}$ 中对象映射为字符串的 **编码** 函数和一个将字符串解码回 $\mathcal{O}$ 中对象的 **解码** 函数组成.  
形式上, 我们作如下定义:  

设 $\mathcal{O}$ 为任意集合. 对 $\mathcal{O}$ 的 **表示方案** 是一个函数对 $E,D$, 其中 $E:\mathcal{O} \rightarrow \{0,1\}^*$ 是全域一一函数, $D:\{0,1\}^* \rightarrow_p \mathcal{O}$ 是一个（可能是局部定义的）函数, 并且满足 $D$ 和 $E$ 使得 $D(E(o))=o$ 对每个 $o\in \mathcal{O}$ 成立.  
$E$ 称为 **编码** 函数, $D$ 称为 **解码** 函数.  

注意, 对每个 $o\in \mathcal{O}$ 都有 $D(E(o))=o$ 的条件意味着 $D$ 是 **满射**（你能看出为什么吗？）.  
事实上, 构造一个表示方案时, 我们只需要找到一个 **编码** 函数.  
也就是说, 每个一一的编码函数都有对应的解码函数, 如下引理所示:  

假设 $E: \mathcal{O} \rightarrow \{0,1\}^*$ 是一一映射. 那么存在一个函数 $D:\{0,1\}^* \rightarrow \mathcal{O}$ 使得 $D(E(o))=o$ 对每个 $o\in \mathcal{O}$ 成立.  

设 $o_0$ 为 $\mathcal{O}$ 中任意一个元素.  
对于每个 $x \in \{0,1\}^*$, 要么不存在, 要么仅存在一个 $o\in \mathcal{O}$ 使 $E(o)=x$（否则 $E$ 将不是一一映射）.  
我们将 $D(x)$ 定义为在第一种情况取 $o_0$, 在第二种情况取该唯一对象 $o$.  
根据定义, 对每个 $o\in \mathcal{O}$ 都有 $D(E(o))=o$.


::: {.remark title="全域解码函数" #totaldecoding}
虽然表示方案的解码函数通常可以是一个 **局部** 函数, 但 [decodelem](){.ref} 的证明表明, 每个表示方案都有一个 **全域** 解码函数. 这一观察有时是很有用的.

:::

### 有限表示

如果 $\mathcal{O}$ 是 **有限** 的, 那么我们可以将 $\mathcal{O}$ 中的每个对象表示为长度至多为某个数 $n$ 的字符串.  
那么 $n$ 的取值是多少呢？  
我们记 $\{0,1\}^{\leq n}$ 为长度至多为 $n$ 的字符串集合 $\{ x\in \{0,1\}^* : |x| \leq n \}$.  
集合 $\{0,1\}^{\leq n}$ 的大小等于

$$
|\{0,1\}^0| + |\{0,1\}^1| + |\{0,1\}^2| + \cdots + |\{0,1\}^n| = \sum_{i=0}^n 2^i = 2^{n+1}-1.
$$

这使用 [等比数列](https://en.wikipedia.org/wiki/Geometric_progression) 的标准求和公式即可得到.  

为了将 $\mathcal{O}$ 中的对象表示为长度至多为 $n$ 的字符串, 我们需要构造一个从 $\mathcal{O}$ 到 $\{0,1\}^{\leq n}$ 的一一映射. 而当且仅当 $|\mathcal{O}| \leq 2^{n+1}-1$, 我们才能做到这一点, 如以下引理所示:  

对于任意两个非空有限集合 $S,T$, 当且仅当 $|S| \leq |T|$ 时, 存在一个一一映射 $E:S \rightarrow T$.  

设 $k=|S|$ 且 $m=|T|$, 并将 $S$ 和 $T$ 的元素分别写为 $S = \{ s_0 , s_1, \ldots, s_{k-1} \}$ 和 $T= \{ t_0 , t_1, \ldots, t_{m-1} \}$.  
我们需要证明, 存在一个一一映射 $E: S \rightarrow T$ 当且仅当 $k \leq m$.  

对"当"方向, 如果 $k \leq m$, 我们可以简单地定义 $E(s_i)=t_i$ 对每个 $i\in [k]$.  
显然, 对于 $i \neq j$, 有 $t_i = E(s_i) \neq E(s_j) = t_j$, 因此该函数是一一映射.  

对"仅当"方向, 假设 $k>m$ 且 $E: S \rightarrow T$ 是某个函数. 那么 $E$ 不可能是一一映射.  
事实上, 对 $i=0,1,\ldots,m-1$, 我们“标记” $T$ 中的元素 $t_j=E(s_i)$.  
如果 $t_j$ 已经被标记过, 那么我们就找到了两个映射到同一元素 $t_j$ 的 $S$ 中的对象.  
否则, 由于 $T$ 有 $m$ 个元素, 当我们标记到 $i=m-1$ 时, $T$ 中的所有对象都已被标记.  
因此, 在这种情况下, $E(s_m)$ 必须映射到一个已经被标记过的元素.  
(这一观察有时被称为“鸽巢原理”: 假设有 $m$ 个巢和 $k>m$ 只鸽子, 则必有两只鸽子在同一个巢中.)


### 前缀无歧义编码

在展示有理数的表示方案时, 我们使用了一个"技巧": 将字母表 $\{ 0,1, \|\}$ 编码, 以便将字符串元组表示为单个字符串.  
这是 **前缀无歧义编码** 的一个特例.  

前缀无歧义编码的思想如下, 如果我们的表示具有如下性质: 表示对象 $o$ 的字符串 $x$ 不是表示不同对象 $o'$ 的字符串 $y$ 的 **前缀** (即初始子串), 那么我们可以仅通过将列表中所有成员的表示串联起来, 来表示一个对象列表.  
例如, 因为在英文中每个句子都以标点符号结束, 如句号, 感叹号或问号, 没有句子可以成为另一个句子的前缀, 因此我们可以仅通过将句子一个接一个地串联来表示一个句子列表. (英文中存在一些复杂情况, 例如缩写中的句点 (如 "e.g.")或句子引号包含标点, 但高层次上前缀自由表示句子的原理仍然成立.)  

事实上, 我们可以将 **每一个** 表示转换为前缀无歧义形式.  
这为 [representtuplesidea](){.ref} 提供了依据, 并允许我们将类型 $T$ 对象的表示方案转换为类型 $T$ 对象 **列表** 的表示方案.  
通过重复同样的技术, 我们还可以表示类型 $T$ 对象的列表的列表, 以此类推.  

但首先, 让我们正式定义前缀无歧义性:


::: {.definition title="前缀无歧义编码" #prefixfreedef}
对于两个字符串 $y,y'$, 如果 $|y| \leq |y'|$ 并且对每个 $i<|y|$, 有 $y'_i = y_i$, 我们称 $y$ 是 $y'$ 的一个 **前缀**.  

设 $\mathcal{O}$ 为非空集合, $E:\mathcal{O} \rightarrow \{0,1\}^*$ 为一个函数.  
如果对每个 $o\in\mathcal{O}$, $E(o)$ 非空, 并且不存在一对不同的对象 $o, o' \in \mathcal{O}$ 使得 $E(o)$ 是 $E(o')$ 的前缀, 我们称 $E$ 是 **前缀无歧义** 的.
:::

回忆一下, 对于每个集合 $\mathcal{O}$, 集合 $\mathcal{O}^*$ 包含所有有限长度的元组（即 **列表**）的 $\mathcal{O}$ 中元素.  
下述定理表明, 如果 $E$ 是 $\mathcal{O}$ 的前缀自由编码, 则通过串联编码, 我们可以得到 $\mathcal{O}^*$ 的一个有效的(一一)表示:


::: {.theorem title="前缀无歧义蕴含元组可编码" #prefixfreethm}
假设 $E:\mathcal{O} \rightarrow \{0,1\}^*$ 是前缀无歧义的.  
则以下映射 $\overline{E}:\mathcal{O}^* \rightarrow \{0,1\}^*$ 是一一映射: 对每个 $(o_0,\ldots,o_{k-1}) \in \mathcal{O}^*$, 我们定义

$$
\overline{E}(o_0,\ldots,o_{k-1}) = E(o_0)E(o_1) \cdots E(o_{k-1}) \;.
$$

:::

[prefixfreethm](){.ref} 可能有点难以理解, 但一旦你理解了它的含义, 实际上证明起来相当直接.  
因此, 我强烈建议你在此处停下来, 确保你理解了该定理的陈述. 你也应该尝试自己证明它, 然后再继续阅读.


![If we have a prefix-free representation of each object then we can concatenate the representations of $k$ objects to obtain a representation for the tuple $(o_0,\ldots,o_{k-1})$.](./images/chapter2/repres_list.png)

证明的思路很简单.  
例如, 假设我们想从表示 $x= \overline{E}(o_0,o_1,o_2)=E(o_0)E(o_1)E(o_2)$ 中解码三元组 $(o_0,o_1,o_2)$.  
我们首先找到 $x$ 的第一个前缀 $x_0$, 它是某个对象的表示.  
然后解码该对象, 从 $x$ 中去掉 $x_0$ 得到新的字符串 $x'$, 再继续找到 $x'$ 的第一个前缀 $x_1$, 以此类推（参见 [prefix-free-tuples-ex](){.ref}）.  
$E$ 的前缀自由性质保证了 $x_0$ 实际上就是 $E(o_0)$, $x_1$ 是 $E(o_1)$, 依此类推.


::: {.proof data-ref="prefixfreethm"}
现在我们给出正式证明.  
使用反证法, 假设存在两个不同的元组 $(o_0,\ldots,o_{k-1})$ 和 $(o'_0,\ldots,o'_{k'-1})$, 使得

$$
\overline{E}(o_0,\ldots,o_{k-1})= \overline{E}(o'_0,\ldots,o'_{k'-1}) \;. \label{prefixfreeassump}
$$

我们将字符串 $\overline{E}(o_0,\ldots,o_{k-1})$ 记为 $\overline{x}$.  

设 $i$ 为第一个使得 $o_i \neq o'_i$ 的索引.  
（如果对所有 $i$ 都有 $o_i=o'_i$, 由于假设这两个元组不同, 则其中一个元组的长度必须大于另一个. 在这种情况下, 不失一般性, 我们假设 $k'>k$ 并令 $i=k$.）  
在 $i<k$ 的情况下, 我们看到字符串 $\overline{x}$ 可以用两种不同的方式表示:

$$
\overline{x} = \overline{E}(o_0,\ldots,o_{k-1}) = x_0\cdots x_{i-1} E(o_i) E(o_{i+1}) \cdots E(o_{k-1})
$$

以及

$$
\overline{x} = \overline{E}(o'_0,\ldots,o'_{k'-1}) = x_0\cdots x_{i-1} E(o'_i) E(o'_{i+1}) \cdots E(o'_{k'-1})
$$

其中 $x_j = E(o_j) = E(o'_j)$ 对所有 $j<i$ 成立.  
令 $\overline{y}$ 为从 $\overline{x}$ 中去掉前缀 $x_0 \cdots x_{i-1}$ 后得到的字符串.  
我们看到 $\overline{y}$ 可以写成两种形式: $\overline{y}= E(o_i)s$ 对某个字符串 $s\in \{0,1\}^*$, 也可以写成 $\overline{y} = E(o'_i)s'$ 对某个 $s'\in \{0,1\}^*$.  
但这意味着 $E(o_i)$ 与 $E(o'_i)$ 中的一个必须是另一个的前缀, 这与 $E$ 的前缀自由性矛盾.  

若 $i=k$ 且 $k'>k$, 我们通过如下方式得到矛盾: 在这种情况下

$$
\overline{x} = E(o_0)\cdots E(o_{k-1}) = E(o_0) \cdots E(o_{k-1}) E(o'_k) \cdots E(o'_{k'-1})
$$

这意味着 $E(o'_k) \cdots E(o'_{k'-1})$ 必须对应于空字符串 $\text{""}$.  
但在这种情况下, $E(o'_k)$ 也必须是空字符串, 而空字符串显然是任意其他字符串的前缀, 这与 $E$ 的前缀自由性矛盾.

:::

::: {.remark title="列表表示的前缀无歧义性" #prefixfreelistsrem}
即使集合 $\mathcal{O}$ 中对象的表示 $E$ 是前缀无歧义的, 也并不意味着这些对象的 **列表** 的表示 $\overline{E}$ 也会是前缀无歧义的. 例如: 对于任意三个对象 $o,o',o''$, 列表 $(o,o')$ 的表示将是列表 $(o,o',o'')$ 的表示的前缀.  
然而, 如下的 [prefixfreetransformationlem](){.ref} 所示, 我们可以将 **每一个** 表示转换为前缀无歧义的, 因此如果需要表示列表的列表、列表的列表的列表等, 我们就可以使用该转换.

:::

### 构造前缀无歧义表示

有一些自然的表示是前缀无歧义的.  
例如, 每个 **固定输出长度** 的表示（即一一函数 $E:\mathcal{O} \rightarrow \{0,1\}^n$）自动是前缀无歧义的, 因为只有当 $x$ 和 $x'$ 相等时, 长度相同的 $x'$ 才可能有 $x$ 作为前缀.  

此外, 我们用来表示有理数的方法也可以用来证明如下结论:  

设 $E:\mathcal{O} \rightarrow \{0,1\}^*$ 为一一函数.  
则存在一个一一的前缀无歧义编码 $\overline{E}$, 对每个 $o\in \mathcal{O}$ 有 $|\overline{E}(o)| \leq 2|E(o)|+2$.  

为了完整起见, 我们将在下方给出证明. 不过你可以在这里停下来, 尝试用我们表示有理数时使用的相同技巧自己证明它.


::: {.proof data-ref="prefixfreetransformationlem"}
证明的核心思想是使用映射 $0 \mapsto 00$, $1 \mapsto 11$ 来"加倍"字符串 $x$ 中的每一位, 然后通过在其后拼接 $01$ 来标记字符串的结束.  
如果我们以这种方式对字符串 $x$ 进行编码, 它可以确保 $x$ 的编码绝不会是不同字符串 $x'$ 的编码的前缀.  
形式上, 我们对每个 $x\in \{0,1\}^*$ 定义函数 $PF:\{0,1\}^* \rightarrow \{0,1\}^*$ 如下:

$$
PF(x)=x_0 x_0 x_1 x_1 \ldots x_{n-1}x_{n-1}01.
$$

如果 $E:\mathcal{O} \rightarrow \{0,1\}^*$ 是 $\mathcal{O}$ 的（可能不是前缀无歧义的）表示, 我们可以通过定义 $\overline{E}(o)=PF(E(o))$ 将其转换为前缀无歧义的表示 $\overline{E}:\mathcal{O} \rightarrow \{0,1\}^*$.  

为了证明该引理, 我们需要证明 __(1)__ $\overline{E}$ 是一一函数, 并且 __(2)__ $\overline{E}$ 是前缀无歧义的.  
事实上, 前缀无歧义是比一一更强的条件（如果两个字符串相等, 则其中一个必然是另一个的前缀）, 因此只需证明 __(2)__ 即可, 我们现在来证明它.  

设 $o \neq o'$ 为 $\mathcal{O}$ 中两个不同的对象.  
我们将证明 $\overline{E}(o)$ 不是 $\overline{E}(o')$ 的前缀, 或换句话说, $PF(x)$ 不是 $PF(x')$ 的前缀, 其中 $x = E(o)$, $x'=E(o')$.  
由于 $E$ 是一一函数, 所以 $x \neq x'$. 我们分三种情况讨论, 取决于 $|x|<|x'|$, $|x|=|x'|$, 或 $|x|>|x'|$.  

- 如果 $|x|<|x'|$, 则 $PF(x)$ 中位置 $2|x|,2|x|+1$ 的两位为 $01$, 而 $PF(x')$ 中对应位将等于 $00$ 或 $11$（取决于 $x'$ 的第 $|x|$ 位）, 因此 $PF(x)$ 不可能是 $PF(x')$ 的前缀.  
- 如果 $|x|=|x'|$, 由于 $x \neq x'$, 必然存在某个位置 $i$ 使它们不同, 这意味着 $PF(x)$ 和 $PF(x')$ 在位置 $2i,2i+1$ 上不同, 同样 $PF(x)$ 不是 $PF(x')$ 的前缀.  
- 如果 $|x|>|x'|$, 则 $|PF(x)|=2|x|+2>|PF(x')|=2|x'|+2$, 因此 $PF(x)$ 比 $PF(x')$ 长, 不可能是其前缀.  

在所有情况下, 我们可以预见 $PF(x)=\overline{E}(o)$ 都不是 $PF(x')=\overline{E}(o')$ 的前缀, 从而完成了证明.

:::

[prefixfreetransformationlem](){.ref} 的证明并不是将任意表示转换为前缀无歧义形式的唯一方法，也不一定是最优方法.  
[prefix-free-ex](){.ref} 就要求你构造一个更高效的前缀无歧义转换, 满足 $|\overline{E}(o)| \leq |E(o)| + O(\log |E(o)|)$.


### "基于Python的证明" (选读)

[prefixfreethm](){.ref} 和 [prefixfreetransformationlem](){.ref} 的证明是 **构造性的**, 意味着它们给出了：

* 将任意对象 $O$ 的表示的编码和解码函数转换为前缀无歧义的编码和解码函数的方法, 以及
* 将单个对象的前缀无歧义编码和解码扩展到 **对象列表** 的编码和解码的方法（通过串联实现）。

具体来说, 我们可以将任意一对 Python 函数 `encode` 和 `decode` 转换为函数 `pfencode` 和 `pfdecode`, 对应于前缀无歧义的编码和解码.
同样, 给定单个对象的 `pfencode` 和 `pfdecode`, 我们可以将它们扩展到列表的编码.
下面展示了如何对上文定义的 `NtS` 和 `StN` 函数进行这种处理。

我们从 [prefixfreetransformationlem](){.ref} 的“Python 证明”开始：一种将任意表示转换为 **前缀无歧义** 表示的方法.
下面的函数 `prefixfree` 接受一对编码和解码函数作为输入, 并返回一个三元组函数，其中包含 **前缀无歧义** 的编码和解码函数, 以及一个检查字符串是否为对象有效编码的函数.


```python
# 接受 encode 和 decode 函数，分别将对象映射为比特列表以及反向映射，
# 并返回 pfencode 和 pfdecode 函数，
# 以前缀无歧义的方式将对象映射为比特列表以及反向映射。
# 同时返回一个 pfvalid 函数，用于判断一个比特列表是否为有效编码

def prefixfree(encode, decode):
    def pfencode(o):
        L = encode(o)
        return [L[i//2] for i in range(2*len(L))]+[0,1]
    def pfdecode(L):
        return decode([L[j] for j in range(0,len(L)-2,2)])
    def pfvalid(L):
        return (len(L) % 2 == 0 ) and all(L[2*i]==L[2*i+1] for i in range((len(L)-2)//2)) and L[-2:]==[0,1]

    return pfencode, pfdecode, pfvalid

pfNtS, pfStN , pfvalidN = prefixfree(NtS,StN)

NtS(234)
# 11101010
pfNtS(234)
# 111111001100110001
pfStN(pfNtS(234))
# 234
pfvalidM(pfNtS(234))
# true
```

注意, 上述 Python 函数 `prefixfree` 接受两个 **Python 函数** 作为输入, 并输出三个 Python 函数作为结果. (无歧义的情况下, 我们会使用 “Python 函数” 或 “子程序” 这个术语来区分 Python 程序片段和数学意义上的函数.)  
在本书中, 你不需要掌握 Python, 但你需要熟悉函数作为独立的数学对象的概念, 可以被用作其他函数的输入或输出.

下面我们给出 [prefixfreethm](){.ref} 的 “Python 证明”. 具体来说, 我们展示一个函数 `represlists`, 它接受一个前缀无歧义表示方案作为输入 (通过编码、解码和有效性检测函数实现), 并输出一个用于表示该类对象 **列表** 的表示方案. 如果我们希望使这个表示也是前缀无歧义的, 那么可以再将其放入上面的 `prefixfree` 函数中.


```python
def represlists(pfencode,pfdecode,pfvalid):
    """
    接受函数 pfencode, pfdecode 和 pfvalid,  
    并返回函数 encodelists, decodelists,  
    它们可以分别对该类对象的 **列表** 进行编码和解码.   
    """

    def encodelist(L):
        """Gets list of objects, encodes it as list of bits"""
        return "".join([pfencode(obj) for obj in L])

    def decodelist(S):
        """Gets lists of bits, returns lists of objects"""
        i=0; j=1 ; res = []
        while j<=len(S):
            if pfvalid(S[i:j]):
                res += [pfdecode(S[i:j])]
                i=j
            j+= 1
        return res

    return encodelist,decodelist


LtS , StL = represlists(pfNtS,pfStN,pfvalidN)

LtS([234,12,5])
# 111111001100110001111100000111001101
StL(LtS([234,12,5]))
# [234, 12, 5]
```

### 字母和文本的表示

我们可以用一个字符串来表示一个字母或符号, 然后如果这种表示是前缀无关的, 我们就可以通过简单地连接每个符号的表示来表示一个符号序列.  
其中一种表示是 [ASCII](https://en.wikipedia.org/wiki/ASCII), 它用 7 位的字符串表示 128 个字母和符号.  
由于 ASCII 表示是固定长度的, 它自动是前缀无关的 (你能看出原因吗?).  
[Unicode](https://en.wikipedia.org/wiki/Unicode) 是一种将 (在撰写本文时) 约 128,000 个符号表示为介于 0 和 1,114,111 之间的数字的表示方法 (称为 **code points**).  
对于这些 code points 有几种前缀无关的表示方法, 一种流行的方法是 [UTF-8](https://en.wikipedia.org/wiki/UTF-8), 它将每个 code point 编码为长度在 8 到 32 之间的字符串.


<!-- (For example, the UTF-8 encoding for the "confused face" emoji 😕 is `11110000100111111001100010010101`) -->

![The word ](./images/chapter2/braille.png){#braillefig .class .margin }

::: {.example title="Braille 编码(盲文)" #braille}
**Braille 编码**(盲文) 是另一种将字母和其他符号编码为二进制字符串的方法. 具体来说, 在盲文中, 每个字母被编码为一个属于 $\{0,1\}^6$ 的字符串, 该字符串通过排列成两列三行的凸起点来书写, 参见 [braillefig](){.ref}.  
（一些符号需要用超过一个六位字符串来编码, 因此盲文使用了更通用的前缀无关编码.）

[Louis Braille](https://goo.gl/Y2BkEe) 是一个法国男孩, 因事故在 5 岁时失明. 盲文由 Braille 于 1821 年发明, 当时他只有 12 岁 (尽管他在一生中不断改进和完善它). 

:::

::: {.example title="C语言中对象的表示(选读)" #Crepresentation}
我们可以使用编程语言来探究我们的计算环境如何表示各种数值.  
在允许直接访问内存的 “不安全” 编程语言（如 C语言）中，这种操作最为简单.

使用一个 [简单的 C 程序](https://goo.gl/L8oMzn), 我们可以得到各种数值的表示方法.  
可以看到, 对于整数, 乘以 2 对应于每个字节内部的 “左移”.  
相比之下, 对于浮点数, 乘以 2 对应于表示中指数部分加 1.  
在我们使用的架构中, 负数使用 [二进制补码](https://goo.gl/wov5fa) 方法表示.  
C语言通过确保字符串末尾有一个零字节, 来以前缀无关的形式表示字符串.


```c
int      2    : 00000010 00000000 00000000 00000000
int      4    : 00000100 00000000 00000000 00000000
int      513  : 00000001 00000010 00000000 00000000
long     513  : 00000001 00000010 00000000 00000000 00000000 00000000 00000000 00000000
int      -1   : 11111111 11111111 11111111 11111111
int      -2   : 11111110 11111111 11111111 11111111
string   Hello: 01001000 01100101 01101100 01101100 01101111 00000000
string   abcd : 01100001 01100010 01100011 01100100 00000000
float    33.0 : 00000000 00000000 00000100 01000010
float    66.0 : 00000000 00000000 10000100 01000010
float    132.0: 00000000 00000000 00000100 01000011
double   132.0: 00000000 00000000 00000000 00000000 00000000 10000000 01100000 01000000
```

:::

### 向量, 矩阵及图片的表示

一旦我们可以表示数字和数字列表, 我们就可以表示 **向量**(本质上就是数字的列表).  
同样, 我们可以表示列表的列表, 因此特别地, 可以表示 **矩阵**.  
为了表示一张图像, 我们可以通过一个长度为3的数字列表表示每个像素的颜色, 分别对应红色、绿色和蓝色的强度.  
（我们可以只使用三种原色, 因为 [大多数](https://en.wikipedia.org/wiki/Tetrachromacy) 人类视网膜中只有三种类型的视锥细胞; 而如果要表示 [螳螂虾](https://goo.gl/t7JBfC) 可见的颜色, 我们需要 16 种原色.）  
因此, 一张包含 $n$ 个像素的图像可以表示为一个包含 $n$ 个长度为三的列表的列表.  
视频可以表示为图像的列表.  
当然, 这些表示方法相当浪费, 对于图像和视频通常使用 [更](https://en.wikipedia.org/wiki/JPEG) [紧凑](https://goo.gl/Vs8UhU) 的表示方法, 虽然本书不会涉及这些内容.

### 图的表示

一个 **图** 在 $n$ 个顶点上可以表示为一个 $n \times n$ 的 **邻接矩阵**, 其第 $(i,j)$ 个元素为 1 当且仅当边 $(i,j)$ 存在, 否则为 0.  
也就是说, 我们可以将一个 $n$ 顶点的有向图 $G=(V,E)$ 表示为一个字符串 $A \in \{0,1\}^{n^2}$, 使得 $A_{i,j}=1$ 当且仅当边 $\overrightarrow{i\;j} \in E$.  
我们可以通过将每条无向边 $\{i,j\}$ 替换为两条有向边 $\overrightarrow{i\; j}$ 和 $\overleftarrow{i\; j}$ 来将无向图转换为有向图.

另一种图的表示方法是 **邻接表** 表示. 也就是说, 我们将图的顶点集合 $V$ 与集合 $[n]$ 对应, 其中 $n=|V|$, 并将图 $G=(V,E)$ 表示为 $n$ 个列表组成的列表, 其中第 $i$ 个列表包含顶点 $i$ 的出邻居.  
对于某些应用, 这些表示方法之间的差异可能很大, 虽然对于我们而言通常无关紧要.


![Representing the graph $G=(\{0,1,2,3,4\},\{ (1,0),(4,0),(1,4),(4,1),(2,1),(3,2),(4,3) \})$ in the adjacency matrix and adjacency list representations.](./images/chapter2/representing_graphs.png)

### 列表和嵌套列表的表示

如果我们有一种方法将集合 $\mathcal{O}$ 中的对象表示为二进制字符串, 那么我们可以通过应用前缀无关变换来表示这些对象的列表.  
此外, 我们可以使用类似上述的技巧来处理 **嵌套** 列表.  
其思想是, 如果我们有某种表示 $E:\mathcal{O} \rightarrow \{0,1\}^*$, 那么我们可以使用五元素字母表 $\Sigma = \{$ `0`,`1`,`[` , `]` , `,` $\}$ 上的字符串来表示来自 $\mathcal{O}$ 的嵌套列表.  

例如, 如果 $o_1$ 表示为 `0011`, $o_2$ 表示为 `10011`, $o_3$ 表示为 `00111`, 那么我们可以将嵌套列表 $(o_1,(o_2,o_3))$ 表示为字母表 $\Sigma$ 上的字符串 `"[0011,[10011,00111]]"`.  

通过将 $\Sigma$ 的每个元素本身编码为三位二进制字符串,  
我们可以将任意对象集合 $\mathcal{O}$ 的表示转换为一种表示, 使得可以表示这些对象的（潜在嵌套）列表.


### 一些注释

我们通常会将一个对象与其字符串表示等同起来.  
例如, 如果 $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 是某个将字符串映射到字符串的函数, 且 $n$ 是一个整数, 我们可能会说 “$F(n)+1$ 是质数”, 这意味着如果我们将 $n$ 表示为字符串 $x$, 那么由字符串 $F(x)$ 表示的整数 $m$ 满足 $m+1$ 是质数.  
（你可以看到, 这种将对象与其表示等同的约定可以为我们节省大量繁琐的形式化表达.）  

同样地, 如果 $x, y$ 是某些对象, 且 $F$ 是一个以字符串为输入的函数, 那么 $F(x,y)$ 表示将 $F$ 应用于有序对 $(x,y)$ 的表示的结果.  
我们对任意 $k$ 元组对象使用相同的符号表示函数的调用.

这种将对象与其字符串表示等同的约定, 是我们人类一直在使用的.  
例如, 当人们说 “$17$ 是质数” 时, 他们真正的意思是, 十进制表示为字符串 "`17`" 的整数是质数.


::: {.quote}
当我们说

_“$A$ 是一个计算自然数乘法的算法”_  

时, 我们真正的意思是  

_“$A$ 是一个计算函数 $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 的算法, 满足对于每一对 $a,b \in \N$, 如果 $x \in \{0,1\}^*$ 是表示有序对 $(a,b)$ 的字符串, 那么 $F(x)$ 将是表示它们乘积 $a \cdot b$ 的字符串”_.

天呐!
:::

## 将计算任务定义为数学函数

抽象地讲, **计算过程** 是一种将输入（二进制字符串）转换为输出（二进制字符串）的过程.  
这种从输入到输出的变换可以通过现代计算机、遵循指令的人、某些自然系统的演化或其他任何手段完成.

在后续章节中, 我们将转向对计算过程的数学定义, 但正如上文所讨论的, 目前我们关注 **计算任务**. 也就是说, 我们关注的是 **规约** 而非 **实现**.  
同样地, 在抽象层面上, 一个计算任务可以指定输出需要满足的任意输入输出关系.  
然而, 在本书的大部分内容中, 我们将专注于最简单、最常见的任务: **计算函数**.  

下面是一些例子:

* 给定两个整数 $x, y$ 的表示, 计算它们的乘积 $x \times y$. 使用上面的表示方法, 这对应于从 $\{0,1\}^*$ 到 $\{0,1\}^*$ 的函数计算. 我们已经看到, 解决这个计算任务的方法不止一种, 事实上, 我们仍然不知道该问题的最优算法.
* 给定一个整数 $z > 1$ 的表示, 计算其 **因式分解**; 即, 找出质数列表 $p_1 \leq \cdots \leq p_k$ 使得 $z = p_1 \cdots p_k$. 这同样对应于从 $\{0,1\}^*$ 到 $\{0,1\}^*$ 的函数计算. 对于该问题的复杂性, 我们的认知差距甚至更大.
* 给定图 $G$ 的表示和两个顶点 $s$ 与 $t$, 计算 $G$ 中从 $s$ 到 $t$ 的最短路径长度, 或者计算从 $s$ 到 $t$ 的 **最长路径**（不重复顶点）的长度. 这两个任务都对应于从 $\{0,1\}^*$ 到 $\{0,1\}^*$ 的函数计算, 但它们的计算难度却差别极大.
* 给定一个 Python 程序的代码, 判断是否存在输入会使程序进入无限循环. 该任务对应于从 $\{0,1\}^*$ 到 $\{0,1\}$ 的 **部分函数** 计算, 因为并非每个字符串都对应语法有效的 Python 程序. 我们会看到, 我们 **确实** 理解该问题的计算状态(见下文的状态机), 但答案相当令人惊讶.
* 给定图像 $I$ 的表示, 判断 $I$ 是猫的照片还是狗的照片. 这对应于从 $\{0,1\}^*$ 到 $\{0,1\}$ 的某个（部分）函数的计算.

计算任务的一个重要特例是计算 **布尔函数**, 其输出为单比特 $\{0,1\}$.  
计算这类函数对应于回答 **是/否** 问题, 因此该任务也被称为 **判定问题**.  
给定任意函数 $F:\{0,1\}^* \rightarrow \{0,1\}$ 和 $x \in \{0,1\}^*$, 计算 $F(x)$ 的任务对应于判定 $x$ 是否属于集合 $L$, 其中 $L = \{ x : F(x)=1 \}$ 被称为与函数 $F$ 对应的 **语言**.（语言这个术语源于计算理论与诺姆·乔姆斯基发展的形式语言学之间的历史联系.）  
因此, 许多文献将这类计算任务称为 **判定一个语言**.


![A subset $L \subseteq \{0,1\}^*$ can be identified with the function $F:\{0,1\}^* \rightarrow \{0,1\}$ such that $F(x)=1$ if $x\in L$ and $F(x)=0$ if $x\not\in L$. Functions with a single bit of output are called _Boolean functions_, while subsets of strings are called _languages_. The above shows that the two are essentially the same object, and we can identify the task of deciding membership in $L$ (known as _deciding a language_ in the literature) with the task of computing the function $F$.](./images/chapter2/booleanfunc.png)

对于每一个特定函数 $F$, 可能存在多种 **算法** 来计算 $F$.  
我们将关注如下问题:

* 对于给定函数 $F$, 是否可能 **不存在算法** 来计算 $F$?
* 如果存在算法, 哪一个是最优的? 是否可能 $F$ 在某种意义上是 “有效不可计算”的, 即计算 $F$ 的每个算法都需要极其庞大的资源?
* 如果我们无法回答这个问题, 能否在不同函数 $F$ 和 $F'$ 之间证明某种等价性, 即它们要么都容易（有快速算法）, 要么都困难?
* 一个函数难以计算是否可能是 **好事**? 我们能否将其应用于密码学等领域?

为了回答这些问题, 我们需要对 **算法** 的概念进行数学定义, 这将在 [compchap](){.ref} 中完成.


### 注意区分 **函数** 和 **程序**!

你应始终注意 **规约** 与 **实现** 之间可能产生的混淆, 或等价地, **数学函数** 与 **算法/程序** 之间的混淆.  
编程语言（包括 Python）使用 **函数** 这个术语来表示（部分）程序, 这只会增加混乱.  
这种混淆还源于数千年的数学历史, 在历史上人们通常通过一种计算方法来定义函数.

例如, 考虑自然数上的乘法函数.  
这是函数 $MULT:\N\times \N \rightarrow \N$, 将一对自然数 $(x,y)$ 映射为它们的乘积 $x \cdot y$.  
正如我们提到的, 它可以通过多种方式实现:

```python
def mult1(x,y):
    res = 0
    while y>0:
        res += x
        y   -= 1
    return res

def mult2(x,y):
    a = str(x) # represent x as string in decimal notation
    b = str(y) # represent y as string in decimal notation
    res = 0
    for i in range(len(a)):
        for j in range(len(b)):
            res += int(a[len(a)-i])*int(b[len(b)-j])*(10**(i+j))
    return res

print(mult1(12,7))
# 84
print(mult2(12,7))
# 84
```

无论是 `mult1` 还是 `mult2`, 给定相同的自然数输入对, 都会产生相同的输出.  
（不过当数字变大时, `mult1` 所需时间会长得多.）  
因此, 尽管它们是两个不同的 **程序**, 它们计算的是相同的 **数学函数**.  
区分 **程序或算法** $A$ 与 $A$ **计算的函数** $F$ 对本课程至关重要 (参见 [functionornotfig](){.ref}).


![A function is a mapping of inputs to outputs. A program is a set of instructions on how to obtain an output given an input. A program computes a function, but it is not the same as a function, popular programming language terminology notwithstanding.](./images/chapter2/functionornot.png){#functionornotfig .margin  }

::: { .bigidea #functionprogramidea }
**函数** 与 **程序** 并不相同.  
程序是用来 **计算** 一个函数的.
:::

区分 **函数** 与 **程序**（或其他计算方式，包括 **电路** 和 **机器**）是本课程的一个核心主题.  
因此，这也是我（以及许多其他教师）在作业和考试中经常提出的问题主题（暗示一下，暗示一下）.


::: {.remark title="超越于函数的计算 (进阶主题, 选读)" #beyonfdunc}
函数能够涵盖相当多的计算任务，但我们也可以考虑更一般的情形.  
首先, 我们可以且将要讨论 **部分函数**, 它们并不在所有输入上都有定义.  
在计算部分函数时, 我们只需关注函数定义域内的输入.  
换句话说, 我们可以在假设有人“承诺”所有输入 $x$ 都使得 $F(x)$ 有定义的前提下, 设计部分函数 $F$ 的算法（否则我们不关心结果）.  
因此, 这种任务也被称为 **承诺问题 (promise problems)**.

另一种推广是考虑 **关系**, 它可能有多个可接受的输出.  
例如, 考虑求解给定方程组的任意解的任务.  
一个 **关系** $R$ 将字符串 $x \in \{0,1\}^*$ 映射为一个 **字符串集合** $R(x)$（例如, $x$ 可能描述一组方程, 此时 $R(x)$ 对应于 $x$ 的所有解的集合）.  
我们也可以将关系 $R$ 与字符串对 $(x,y)$ 的集合对应起来, 其中 $y \in R(x)$.  
如果一个计算过程对于每个 $x \in \{0,1\}^*$ 都输出某个 $y \in R(x)$, 则称它求解了关系 $R$.

在本书后续章节, 我们将考虑更一般的任务, 包括 **交互式任务**（如在游戏中寻找良好策略）、使用概率概念定义的任务等.  
然而, 在本书的大部分内容中, 我们将专注于 **计算函数** 的任务, 并且常常是 **布尔函数**, 输出仅为单比特.  
事实证明, 在这个任务背景下可以研究大量计算理论, 所获得的见解在更一般的情形中同样适用.

:::
* 我们可以使用二进制字符串来表示希望计算的对象.  
* 一个集合 $\mathcal{O}$ 的表示方案是从 $\mathcal{O}$ 到 $\{0,1\}^*$ 的一一映射.  
* 我们可以使用前缀无歧义编码将集合 $\mathcal{O}$ 的表示“升级”为集合中元素列表的表示.  
* 一个基本的计算任务是 **计算函数** $F:\{0,1\}^* \rightarrow \{0,1\}^*$ 的任务. 这个任务不仅包括乘法、因式分解等算术计算, 还涵盖了科学计算、人工智能、图像处理、数据挖掘等众多领域中的其他任务.  
* 我们将研究如何找到（或至少给出界限）计算各种有趣函数 $F$ 的 **最优算法** 的问题.

## 习题

::: {.exercise}
Which one of these objects can be represented by a binary string?

a. An integer $x$

b. An undirected graph $G$.

c. A directed graph $H$

d. All of the above.
:::

::: {.exercise title="Binary representation" #binaryrepex}

a. Prove that the function $NtS:\N \rightarrow \{0,1\}^*$ of the binary representation defined in [ntseq](){.eqref} satisfies that for every $n\in \N$, if $x = NtS(n)$ then $|x| =1+\max(0,\floor{\log_2 n})$ and $x_i = \floor{x/2^{\floor{\log_2 n}-i}} \mod 2$.

b. Prove that $NtS$ is a one to one function by coming up with a function $StN:\{0,1\}^* \rightarrow \N$ such that $StN(NtS(n))=n$ for every $n\in \N$.
:::

::: {.exercise title="More compact than ASCII representation" #compactrepletters}
The ASCII encoding can be used to encode a string of $n$ English letters as a $7n$ bit binary string, but in this exercise, we ask about finding a more compact representation for strings of English lowercase letters.

1. Prove that there exists a representation scheme $(E,D)$ for strings over the 26-letter alphabet $\{ a, b,c,\ldots,z \}$ as binary strings such that for every $n>0$ and length-$n$ string $x \in \{ a,b,\ldots,z \}^n$, the representation $E(x)$ is a binary string of length at most  $4.8n+1000$. In other words, prove that for every $n$, there exists a one-to-one function $E:\{a,b,\ldots, z\}^n \rightarrow \{0,1\}^{\lfloor 4.8n +1000 \rfloor}$.
2. Prove that there exists _no_ representation scheme for strings over the alphabet $\{ a, b,\ldots,z \}$ as binary strings such that for every length-$n$ string $x \in \{ a,b,\ldots, z\}^n$, the representation $E(x)$ is a binary string of length  $\lfloor 4.6n+1000 \rfloor$. In other words, prove that there exists some $n>0$ such that there is no one-to-one function $E:\{a,b,\ldots,z \}^n \rightarrow \{0,1\}^{\lfloor 4.6n + 1000 \rfloor}$.
3. Python's `bz2.compress` function is a mapping from strings to strings,  which uses the _lossless_ (and hence _one to one_) [bzip2](https://en.wikipedia.org/wiki/Bzip2) algorithm for compression.  After converting to lowercase, and truncating spaces and numbers, the text of Tolstoy's "War and Peace" contains $n=2,517,262$. Yet, if we run `bz2.compress` on the string of the text of "War and Peace" we get a string of length $k=6,274,768$ bits, which is only $2.49n$ (and in particular much smaller than $4.6n$). Explain why this does not contradict your answer to the previous question.
4. Interestingly, if we try to apply `bz2.compress` on a _random_ string, we get much worse performance. In my experiments, I got a ratio of about $4.78$ between the number of bits in the output and the number of characters in the input.  However, one could imagine that one could do better and that there exists a company called "Pied Piper" with an algorithm that can losslessly compress a string of $n$ random lowercase letters to fewer than $4.6n$ bits.^[Actually that particular fictional company uses a metric that focuses more on compression _speed_ then _ratio_, see [here](https://blogs.dropbox.com/tech/2016/06/lossless-compression-with-brotli/) and [here](https://www.jefftk.com/p/weissman-scores-useful).] Show that this is not the case by proving that for _every_ $n>100$ and one to one function $Encode:\{a,\ldots,z\}^{n} \rightarrow \{0,1\}^*$, if we let $Z$ be the random variable  $|Encode(x)|$ (i.e., the length of $Encode(x)$) for $x$ chosen uniformly at random from the set $\{a,\ldots,z\}^n$, then the expected value of $Z$ is at least $4.6n$.
   :::

::: {.exercise title="Representing graphs: upper bound" #representinggraphsex}
Show that there is a string representation of directed graphs with vertex set $[n]$ and degree at most $10$ that uses at most $1000 n\log n$ bits. More formally, show the following: Suppose we define for every $n\in\mathbb{N}$, the set $G_n$ as the set containing all directed graphs (with no self loops) over the vertex set $[n]$ where every vertex has degree at most $10$. Then, prove that for every sufficiently large $n$, there exists a one-to-one function $E:G_n \rightarrow \{0,1\}^{\lfloor 1000 n \log n \rfloor}$.
:::

::: {.exercise title="Representing graphs: lower bound" #represgraphlbex}

1. Define $S_n$ to be the set of one-to-one and onto functions mapping $[n]$ to $[n]$. Prove that there is a one-to-one mapping from $S_n$ to $G_{2n}$, where $G_{2n}$ is the set defined in [representinggraphsex](){.ref} above.
2. In this question you will show that one cannot improve the representation of [representinggraphsex](){.ref} to length $o(n \log n)$. Specifically, prove for every sufficiently large  $n\in \mathbb{N}$ there is _no_ one-to-one function $E:G_n \rightarrow \{0,1\}^{\lfloor 0.001 n \log n \rfloor +1000}$.
   :::

::: {.exercise title="Multiplying in different representation" #multrepres }
Recall that the grade-school algorithm for multiplying two numbers requires $O(n^2)$ operations. Suppose that instead of using decimal representation, we use one of the following representations $R(x)$ to represent a number $x$ between $0$ and $10^n-1$. For which one of these representations you can still multiply the numbers in $O(n^2)$ operations?

a. The standard binary representation: $B(x)=(x_0,\ldots,x_{k})$ where $x = \sum_{i=0}^{k} x_i 2^i$ and $k$ is the largest number s.t. $x \geq 2^k$.

b. The reverse binary representation: $B(x) = (x_{k},\ldots,x_0)$ where $x_i$ is defined as above for $i=0,\ldots,k-1$. \

c. Binary coded decimal representation: $B(x)=(y_0,\ldots,y_{n-1})$ where $y_i \in \{0,1\}^4$ represents the $i^{th}$ decimal digit of $x$ mapping $0$ to $0000$, $1$ to $0001$, $2$ to $0010$, etc. (i.e. $9$ maps to $1001$)

d.  All of the above.
:::

::: {.exercise }
Suppose that $R:\N \rightarrow \{0,1\}^*$ corresponds to representing a number $x$ as a string of $x$ $1$'s, (e.g., $R(4)=1111$, $R(7)=1111111$, etc.).
If $x,y$ are numbers between $0$ and $10^n -1$, can we still multiply $x$ and $y$ using $O(n^2)$ operations if we are given them in the representation $R(\cdot)$?
:::

::: {.exercise }
Recall that if $F$ is a one-to-one and onto function mapping elements of a finite set $U$ into a finite set $V$ then the sizes of $U$ and $V$ are the same. Let $B:\N\rightarrow\{0,1\}^*$ be the function such that for every $x\in\N$, $B(x)$ is the binary representation of $x$.

1. Prove that $x < 2^k$ if and only if $|B(x)| \leq k$.
2. Use 1. to compute the size of the set $\{ y \in \{0,1\}^* : |y| \leq k \}$ where $|y|$ denotes the length of the string $y$.
3. Use 1. and 2. to prove that $2^k-1 = 1 + 2 + 4+ \cdots + 2^{k-1}$.
   :::

::: {.exercise title="Prefix-free encoding of tuples" #prefix-free-tuples-ex}
Suppose that $F:\N\rightarrow\{0,1\}^*$ is a one-to-one function that is _prefix-free_ in the sense that there is no $a\neq b$ s.t.  $F(a)$ is a prefix of $F(b)$.

a. Prove that $F_2:\N\times \N \rightarrow \{0,1\}^*$, defined as $F_2(a,b) = F(a)F(b)$ (i.e., the concatenation of $F(a)$ and $F(b)$) is a one-to-one function.

b. Prove that $F_*:\N^*\rightarrow\{0,1\}^*$ defined as $F_*(a_1,\ldots,a_k) = F(a_1)\cdots F(a_k)$ is a one-to-one function, where $\N^*$ denotes the set of all finite-length lists of natural numbers.
:::

::: {.exercise title="More efficient prefix-free transformation" #prefix-free-ex}
Suppose that $F:O\rightarrow\{0,1\}^*$ is some (not necessarily prefix-free) representation of the objects in the set $O$, and $G:\N\rightarrow\{0,1\}^*$ is a prefix-free representation of the natural numbers.  Define $F'(o)=G(|F(o)|)F(o)$ (i.e., the concatenation of the representation of the length $F(o)$ and $F(o)$).

a. Prove that $F'$ is a prefix-free representation of $O$.

b. Show that we can transform any representation to a prefix-free one by a modification that takes a $k$ bit string into a string of length at most $k+O(\log k)$.

c. Show that we can transform any representation to a prefix-free one by a modification that takes a $k$ bit string into a string of length at most $k+ \log k + O(\log\log k)$.^[Hint: Think recursively how to represent the length of the string.]
:::

::: {.exercise title="Kraft's Inequality" #prefix-free-lb}
Suppose that $S \subseteq \{0,1\}^*$ is some finite prefix-free set, and let $n$ some number larger than $\max \{ |x| : x\in X \}$.

a. For every $x\in S$, let $L(x) \subseteq \{0,1\}^n$ denote all the length-$n$ strings whose first $k$ bits are $x_0,\ldots,x_{k-1}$. Prove that __(1)__ $|L(x)|=2^{n-|x|}$ and __(2)__ For every distinct $x,x' \in S$,  $L(x)$ is disjoint from $L(x')$.

b. Prove that $\sum_{x\in S}2^{-|x|} \leq 1$. (_Hint:_ first show that $\sum_{x \in S} |L(x)| \leq 2^n$.)

c. Prove that there is no prefix-free encoding of strings with less than logarithmic overhead. That is, prove that there is no function $PF:\{0,1\}^* \rightarrow \{0,1\}^*$ s.t. $|PF(x)| \leq |x|+0.9\log |x|$ for every sufficiently large $x\in \{0,1\}^*$ and such that the set $\{ PF(x) : x\in \{0,1\}^* \}$ is prefix-free. The factor $0.9$ is arbitrary; all that matters is that it is less than $1$.
:::

Prove that for every two one-to-one functions $F:S \rightarrow T$ and $G:T \rightarrow U$, the function $H:S \rightarrow U$ defined as $H(x)=G(F(x))$ is one to one.

::: {.exercise title="Natural numbers and strings" #naturalsstringsmapex}

1. We have shown that the natural numbers can be represented as strings. Prove that the other direction holds as well: that there is a one-to-one map $StN:\{0,1\}^* \rightarrow \N$. ($StN$ stands for "strings to numbers.")
2. Recall that Cantor proved that there is no one-to-one map $RtN:\R \rightarrow \N$. Show that Cantor's result implies [cantorthm](){.ref}.
   :::

::: {.exercise title="Map lists of integers to a number" #listsinttonumex}
Recall that for every set $S$, the set $S^*$ is defined as the set of all finite sequences of members of $S$ (i.e., $S^* = \{ (x_0,\ldots,x_{n-1}) \;|\; n\in\mathbb{N} \;,\; \forall_{i\in [n]} x_i \in S \}$ ). Prove that there is a one-one-map from $\mathbb{Z}^*$ to $\mathbb{N}$ where $\mathbb{Z}$ is the set of $\{ \ldots, -3 , -2 , -1,0,+1,+2,+3,\ldots \}$ of all integers.
:::

## Bibliographical notes

The study of representing data as strings, including issues such as _compression_ and _error corrections_ falls under the purview of _information theory_, as covered in the classic textbook of Cover and Thomas [@CoverThomas06].
Representations are also studied in the field of _data structures design_, as covered in texts such as  [@CLRS].

The question of whether to represent integers with the most significant digit first or last is known as [Big Endian vs. Little Endian](https://betterexplained.com/articles/understanding-big-and-little-endian-byte-order/) representation.
This terminology comes from Cohen's [@cohen1981holy]  entertaining and informative paper about the conflict between adherents of both schools which he compared to the warring tribes in Jonathan Swift's _"Gulliver's Travels"_.
The two's complement representation of signed integers was suggested in von Neumann's classic report [@vonNeumann45] that detailed the design approaches for a stored-program computer, though similar representations have been used even earlier in abacus and other mechanical computation devices.

The idea that we should separate the _definition_ or _specification_ of a function from its _implementation_ or _computation_ might seem "obvious," but it took quite a lot of time for mathematicians to arrive at this viewpoint.
Historically, a function $F$ was identified by rules or formulas showing how to derive the output from the input.
As we discuss in greater depth in  [chapcomputable](){.ref}, in the 1800s this somewhat informal notion of a function started "breaking at the seams," and eventually mathematicians arrived at the more rigorous definition of a function as an arbitrary assignment of input to outputs.
While many functions may be described (or computed) by one or more formulas, today we do not consider that to be an essential property of functions, and also allow functions that do not correspond to any "nice" formula.

We have mentioned that all representations of the real numbers are inherently _approximate_. Thus an important endeavor is to understand what guarantees we can offer on the approximation quality of the output of an algorithm, as a function of the approximation quality of the inputs. This question is known as the question of determining the [numerical stability](https://en.wikipedia.org/wiki/Numerical_stability) of given equations.
The  [Floating-Point Guide website](https://floating-point-gui.de/) contains an extensive description of the floating-point representation, as well the many ways in which it could subtly fail, see also the website [0.30000000000000004.com](http://0.30000000000000004.com/).

Dauben [@Dauben90cantor] gives a biography of Cantor with emphasis on the development of his mathematical ideas. [@halmos1960naive] is a classic textbook on set theory, also including Cantor's theorem. Cantor's Theorem is also covered in many texts on discrete mathematics, including [@LehmanLeightonMeyer, @LewisZax19].

The adjacency matrix representation of graphs is not merely a convenient way to map a graph into a binary string, but it turns out that many natural notions and operations on matrices are useful for graphs as well. (For example, Google's PageRank algorithm relies on this viewpoint.)  The notes of [Spielman&#39;s course](http://www.cs.yale.edu/homes/spielman/561/) are an excellent source for this area, known as _spectral graph theory_. We will return to this view much later in this book when we talk about _random walks_.
