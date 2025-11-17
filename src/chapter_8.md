```admonish warning 
**本章施工中**
```

<!-- toc -->

# 等价的计算模型 { #chapequivalentmodels }

## 学习目标 { .objectives }
- 了解RAM机(RAM Machine)与λ演算(λ Calculus)
- 掌握这些模型与图灵机及其他模型的等价关系
- 认识元胞自动机(Cellular Automata)与各种图灵机格局
- 理解Church-Turing论题

```admonish quote
计算机科学的所有问题都可以通过增加一层间接寻址来解决

*——大卫·惠勒(David Wheeler)*
```

```admonish quote
由于后续我们将使用函数表达式进行计算, 必须区分函数与形式, 并需要相应的表示法. 这一区分及其描述记法由Church提出, 我们仅作细微调整.

*——约翰·麦卡锡(John McCarthy), 1960年(摘自描述LISP编程语言的论文)*
```

到目前为止, 我们已经定义了使用图灵机计算函数的概念, 但这与实际的计算方法并不完全吻合. 本章将通过证明可计算函数的定义在各种计算模型下保持不变, 来论证这一选择的合理性. 这一概念被称为*图灵完备性*(Church completeness)或*图灵等价性*(Church equivalence), 是计算机科学中最基本的事实之一. 实际上, 被广泛认同的*Church-Turing论题*做了出了如下主张: 任何对可计算函数的"合理"定义, 都等价于通过图灵机可计算的概念. 我们将在[8.8节](#88-church-turing论题讨论)讨论Church-Turing论题以及"合理"的可能定义. 

本章讨论的主要计算模型包括: 

- **RAM机**: 图灵机与具备随机存取存储器(RAM, Random Access Memory)的标准计算架构并不对应, RAM机的数学模型更接近实际计算机, 但我们将看到它在计算能力上与图灵机等价. 我们还将讨论RAM机的一种编程语言变体, 称之为NAND-RAM. 图灵机与RAM机的等价性使得我们能够证明诸多流行编程语言的图灵等价性, 包括现实中使用的所有通用编程语言, 如C、Python、JavaScript等. 
- **元胞自动机**: 许多自然的和人工的系统都可以被建模为简单组件的集合, 每个组件根据其自身状态及其直接邻居的状态, 按照简单的规则进行演化. 一个著名的例子是[康威的生命游戏](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)(Conway's Game of Life). 为了证明元胞自动机与图灵机等价, 我们将引入*图灵机格局*(configurations of Turing machines). 这些格局还有其他应用, 特别是在[第11章](./chapter_11.md)用于证明哥德尔不完备定理——数学中的一个核心结果. 
- **λ演算**: λ演算是一种表达计算模型, 起源于20世纪30年代, 不过它与当今广泛使用的函数式编程语言密切相关. 证明λ演算与图灵机等价涉及一种名为"Y组合子"(Y Combinator)的消除递归的巧妙方法. 

```admonish info title="本章的一个非数学化概览"
本章中我们将研究不同模型间的等价性. 如果两个计算模型能够计算的函数构成的集合是相同的, 则称它们是*等价*的(也称之为*图灵等价*). 例如, 我们已经看到图灵机与NAND-TM程序是等价的, 因为我们可以将每个图灵机转换为计算相同函数的NAND-TM程序, 同样地, 也可以将每个NAND-TM程序转换为计算相同函数的图灵机. 

本章我们将证明这种等价性远不止于图灵机. 我们开发的技术使我们能够证明所有通用编程语言(即Python、C、Java等)都是*图灵完备的*, 即它们能够模拟图灵机, 因此能够计算所有图灵机可计算的函数. 我们还将证明其反向亦成立——图灵机可以用来模拟用任何这些语言编写的程序, 因此能够计算这些语言可计算的任何函数. 这意味着所有这些编程语言都是*图灵等价*的: 即它们在计算能力上等价于图灵机, 并且彼此等价. 这是一个强大的原理, 是计算机科学广泛影响的基础. 此外, 它使我们能够"鱼和熊掌兼得"——既然所有这些模型都是等价的, 我们可以为手头的任务选择方便的模型. 为了实现这种等价性, 我们定义了一种新的计算模型, 称为RAM机. RAM机比图灵机更接近现代计算机的架构, 但在计算能力上仍然与图灵机等价. 

最后, 我们将证明图灵等价性远不止于传统编程语言, 作为极其简单的自然系统的数学模型的*元胞自动机*也是图灵等价的, 并且我们还将看到λ演算的图灵等价性——λ演算是一种用于表达函数的逻辑系统, 是Lisp、OCaml等*函数式编程语言*的基础. 

本章成果概览见{{ref:fig:turingcomplete}}. 
```

```admonish pic id="turingcompletefig"
![](./images/chapter8/turingcomplete.png)

{{pic}}{fig:turingcomplete} 一些图灵等价模型. 所有这些模型在计算能力上都与图灵机(或等价的NAND-TM程序)等价, 因为它们能够计算完全相同的函数类. 所有这些模型都是用于计算接受无界长度输入的无限函数的模型. 相比之下, 布尔电路/NAND-CIRC程序只能计算有限函数, 因此不是图灵完备的. 
```

## 8.1 RAM机与NAND-RAM

图灵机(以及NAND-TM程序)的一个局限性在于, 我们每次只能访问数组或磁带的一个位置. 如果磁头位于磁带的第$22$位, 而我们想要访问第$957$个位置, 那么我们至少需要$923$步才能到达该位置. 相比之下, 几乎每种编程语言都提供了直接访问内存位置的形式化方法. 实际的物理计算机也提供了可以被视为一个大型数组`Memory`的*随机存取存储器*(RAM), 给定索引$p$(即内存地址或*指针*), 我们可以读取和写入`Memory`的第$p$个位置. ("随机存取存储器"这一名称实际上用词有误, 因为它与概率无关, 但既然这是计算理论与实践中的标准术语, 我们也将沿用这一说法). 

中这种内存访问进行建模的计算模型是*RAM机*(有时也称为*字RAM模型*(Word RAM Model)), 如{{ref:fig:ramvsturing}}所示. RAM机的内存是一个大小无界的数组, 其中每个单元可以存储一个*字*(Word), 我们将其视为$\{0,1\}^w$的字符串, 同时(等价地)也视为$[2^w]$中的一个数字. 例如, 许多现代计算架构使用64位的字, 每个内存位置保存一个$\{0,1\}^{64}$中的字符串, 这也可以视为一个介于$0$到$2^{64}-1=18,446,744,073,709,551,615$之间的数字. 参数$w$被称为*字长*(Word Size). 在实践中, $w$通常是一个固定数字(比如64), 但在理论研究中, 我们将$w$建模为一个可以依赖于输入长度或步骤数的参数. (你可以将$2^w$大致视为我们在计算中使用的最大内存地址)除了内存数组, RAM机还包含恒定数量的*寄存器*(Register)$r_0,\dots,r_{k-1}$, 每个寄存器也能保存一个字. 

```admonish pic id="ramvsturingfig"
![ramvsturing](./images/chapter8/ramvsturing.png)

{{pic}}{fig:ramvsturing} RAM机包含有限数量的局部寄存器(每个寄存器保存一个整数)和一个无界的内存数组. 它可以对寄存器执行算术运算, 还可以将内存中由寄存器$r'$中的数字索引的地址的内容加载到寄存器$r$中. 
```

RAM机可以执行的操作包括: 

- **数据移动**:  将内存中某个单元的数据加载到寄存器中, 或将寄存器的内容存储到内存的某个单元. RAM机可以直接访问内存的任何单元, 而无需像图灵机那样将"磁头"移动到该位置. 也就是说, RAM机可以在一步中将由寄存器$r_j$索引的内存单元的内容加载到寄存器$r_i$中, 或将寄存器$r_i$的内容存储到由寄存器$r_j$索引的内存单元中. 
- **计算**:  RAM机可以对寄存器执行计算, 例如算术运算、逻辑运算和比较. 
- **控制流**:  与图灵机一样, 接下来执行什么指令的选择可以取决于RAM机的状态, 这由其寄存器的内容捕获. 

```admonish pic id="rammachinefig"
![](./images/chapter8/rammachine.png)

{{pic}}{fig:rammachine} RAM机和图灵机的不同方面. RAM机可以在其局部寄存器中存储整数, 并且可以读取和写入由其寄存器指定的内存位置. 相比之下, 图灵机只能访问其磁头位置的内存, 且磁头在每一步最多只能向右或向左移动一个位置.
```

我们不会给出RAM机的正式定义, 但参考文献部分([第8.10节](#810-参考文献))包含了这些定义的来源. 正如NAND-TM编程语言模拟图灵机一样, 我们也可以定义一种模拟RAM机的*NAND-RAM编程语言*. NAND-RAM编程语言通过添加以下特性扩展了NAND-TM: 

- NAND-RAM的变量允许是(非负)*整数值*的, 而不仅仅是NAND-TM中的布尔值. 也就是说, 标量变量`foo`保存的是$\N$中的非负整数(而不仅仅是$\{0,1\}$中的一位), 数组变量`Bar`保存的是一个整数数组. 与RAM机的情况一样, 我们不允许无界大小的整数. 具体来说, 每个变量保存一个介于$0$和$T-1$之间的数字, 其中$T$是程序到目前为止已执行的步骤数. (你现在可以忽略此限制: 如果我们想要保存更大的数字, 可以简单地执行虚拟指令；这在后面的章节中会有用)
- 我们允许对数组进行*索引访问*. 如果`foo`是标量而`Bar`是数组, 则`Bar[foo]`引用由`foo`的值索引的`Bar`的位置. (注意这意味着我们不再需要特殊的索引变量`i`)
- 正如编程语言中常见的情况, 我们假设对于布尔运算(如`NAND`), 零值整数被视为*假*, 非零值整数被视为*真*. 
- 除了`NAND`之外, NAND-RAM还包括所有基本的算术运算: 加、减、乘、(整数)除, 以及比较(等于、大于、小于等). 
- NAND-RAM将条件语句`if`/`then`作为语言的一部分. 
- NAND-RAM包含循环结构, 例如`while`和`do`, 作为语言的一部分. 

NAND-RAM编程语言的完整描述见[附录](http://tiny.cc/introtcsappendix). 然而, 关于NAND-RAM你需要了解的最重要的事实是你实际上并不需要太多了解NAND-RAM, 因为它在能力上等同于图灵机: 

```admonish quote title=""
{{thmc}}{thmc:t81}(图灵机(即 NAND-TM 程序)与 RAM 机(即 NAND-RAM 程序)的等价性)

对于每个函数 $F:\{0,1\}^* \rightarrow \{0,1\}^*$, $F$ 可由 NAND-TM 程序计算, 当且仅当 $F$ 可由 NAND-RAM 程序计算. 
```

由于NAND-TM程序等价于图灵机, 而NAND-RAM程序等价于RAM机, {{tref:thmc:t81}}表明所有这四种模型彼此之间是等价的. 

```admonish pic id="nandramproofoverviewfig"
![](./images/chapter8/nandramproofoverview.png)

{{pic}}{fig:nandramproofoverview} 使用NAND-TM模拟NAND-RAM的{{tref:thmc:t81}}证明步骤概览. 我们首先使用[7.4.1节]()中的内部循环语法糖, 使得能够将整数从数组加载到NAND-TM的索引变量`i`. 一旦我们能这样做, 我们就可以在NAND-TM中模拟索引访问. 然后, 我们利用$\N^2$到$\N$的嵌入, 在NAND-TM中模拟二维位数组. 最后, 我们使用二进制表示将整数的一维数组编码为位的二维数组, 从而完成使用NAND-TM对NAND-RAM的模拟.
```

```admonish tip collapsible=true title="证明思路"
显然, NAND-RAM只会比NAND-TM更强大, 因此如果一个函数$F$可由NAND-TM程序计算, 那么它也能由NAND-RAM程序计算. 具有挑战性的方向是将NAND-RAM程序$P$转换为等价的NAND-TM程序$Q$. 要完整描述这个证明, 我们需要涵盖NAND-RAM语言的完整形式化规范, 并展示如何将其每一个特性实现为NAND-TM之上的语法糖. 

这可以做到, 但详细检查所有操作相当繁琐. 因此, 我们将着重描述此转换背后的主要思想. (另见{{ref:fig:nandramproofoverview}})NAND-RAM在两个方面推广了NAND-TM: **(a)** 增加了对数组的索引访问(即`Foo[bar]`语法), 以及 **(b)** 从布尔值变量过渡到整数值变量. 该转换有两个步骤: 
1. *位数组的索引访问*:  我们首先展示如何处理 **(a)**. 即, 我们展示如何在NAND-TM中实现操作`Setindex(Bar)`, 使得如果`Bar`是编码了某个整数$j$的数组, 则在执行`Setindex(Bar)`后, `i`的值将等于$j$. 这将允许我们通过`Setindex(Bar)`后跟`Foo[i]`来模拟`Foo[Bar]`这种形式的语法. 
2. *二维位数组*:  接着, 我们展示如何使用"语法糖"来为NAND-TM增加*二维数组*的功能. 即, 拥有*两个索引*`i`和`j`以及*二维数组*, 使得我们可以使用语法`Foo[i][j]`来访问`Foo`的(`i`,`j`)位置. 
3. *整数数组*:  最后, 我们将一个*整数*的一维数组`Arr`编码为一个*位*的二维数组`Arrbin`. 思路很简单: 如果$a_{i,0},\ldots,a_{i,\ell}$是`Arr[`$i$`]`的一个二进制(无前缀)表示, 那么`Arrbin[`$i$`][`$j$`]`将等于$a_{i,j}$. 

一旦我们有了整数数组, 我们就可以使用我们常用的函数语法糖、`GOTO`等来实现NAND-RAM的算术和控制流操作. 
```

上述方法并非获得{{tref:thmc:t81}}证明的唯一途径, 例如可参见[练习8.1](). 

```admonish note title="备注8.2: RAM机/NAND-RAM与汇编语言(可选)"
RAM机与现实中的微处理器(例如Intel x86系列中的那些)非常对应, 这些微处理器也包含一个大的*主内存*和数量固定的少量寄存器. 这当然并非偶然: 与图灵机相比, RAM机旨在更贴近地模拟实际计算系统的体系结构, 这种体系结构在很大程度上遵循了 ([von Neumann, 1945](https://scholar.google.com/scholar?hl=en&q=von+Neumann+First+Draft+of+a+Report+on+the+EDVAC)) 报告中描述的所谓[冯·诺依曼架构](https://en.wikipedia.org/wiki/Von_Neumann_architecture). 
因此, NAND-RAM在其大致轮廓上类似于x86或NIPS等汇编语言. 这些汇编语言都具有以下指令: **(1)** 将数据从寄存器移动到内存, **(2)** 对寄存器执行算术或逻辑计算, 以及 **(3)** 条件执行和循环(在汇编语言语境中通常称为"分支"和"跳转"的"if"和"goto"). 

RAM机与实际微处理器之间的主要区别(相应地, 也是NAND-RAM与汇编语言之间的主要区别)在于, 实际微处理器具有固定的字长$w$, 因此所有寄存器和内存单元保存的都是$[2^w]$中的数字(或等价地, $\{0,1\}^w$中的字符串). 这个数字$w$在不同的处理器中可能不同, 但常见的值要么是$32$, 要么是$64$. 作为理论模型, RAM机没有这个限制, 我们反而让$w$作为我们运行时间的对数(这也大致对应于其在实践中的值). 现实中的微处理器也具有固定数量的寄存器(例如, x86-64中有14个通用寄存器), 但这与RAM机相比差别不大. 可以证明, 只有两个寄存器的RAM机与拥有任意大的常数数量寄存器的完整RAM机具有同等能力. 

当然, 现实中的微处理器也具有许多RAM机所不具备的特性, 包括并行性、内存层次结构以及许多其他特性. 然而, RAM机确实在初步近似下捕捉了实际计算机的特征, 因此(正如我们将看到的), 算法在RAM机上的运行时间(例如, $O(n)$对比$O(n^2)$)与其实际运行的效率高度相关. 
```

## 8.2 具体细节(可选) { #nandtmgorydetailssec  }

我们将不展示{{tref:thmc:t81}}的完整形式化证明, 而是聚焦于最重要的部分: 实现索引访问, 以及用一维数组模拟二维数组. 即便如此, 描述这些部分也已经相当繁琐, 这对于任何写过编译器的人都不足为奇. 因此, 你可以随意略读本节. 重点不在于记住所有细节, 而在于明白*原则上*将一个NAND-RAM程序转换为等价的NAND-TM程序是*可能*的, *你*自己如果想做也能完成. 

### 8.2.1 NAND-TM中的索引访问

在NAND-TM中, 我们只能访问数组在索引变量`i`位置处的元素, 而NAND-RAM拥有整数值变量, 并能使用它们对数组进行*索引访问*, 写作`Foo[bar]`. 为了在NAND-TM中实现索引访问, 我们将使用某种无前缀编码(参见[2.5.2节](./chapter_2.md#前缀无歧义编码))在数组中编码整数, 然后提供一个过程`Setindex(Bar)`来将`i`设置为`Bar`编码的值. 我们可以通过先执行`Setindex(Bar)`再执行`Foo[i]`来模拟`Foo[Bar]`的效果. 

`Setindex(Bar)`的实现可以通过以下方式完成: 

1. 初始化一个数组`Atzero`, 使得`Atzero[`$0$`]`$=1$并且对所有$j>0$, `Atzero[`$j$`]`$=0$. (这可以在NAND-TM中轻松完成, 因为所有未初始化的变量默认值为零)
2. 通过递减`i`直到达到`Atzero[i]`$=1$的点来将`i`设置为零. 
3. 令`Temp`为一个编码数字$0$的数组. 
4. 我们使用`GOTO`来模拟一个内部循环, 形式如下: 当`Temp`$\neq$`Bar`时, 递增`Temp`. 
5. 在循环结束时, `i`等于由`Bar`编码的值. 

在NAND-TM代码中(使用一些语法糖), 我们可以按如下方式实现上述操作: 

```python
# 假设Atzero是一个数组, 满足Atzero[0]=1
# 且对所有j>0, Atzero[j]=0

# 将i设置为0. 
LABEL("zero_idx")
dir0 = zero
dir1 = one
# 对应i <- i-1
GOTO("zero_idx",NOT(Atzero[i]))
...
# 将temp清零
#(下面的代码假设使用一种特定的无前缀编码, 其中10是"结束标记")
Temp[0] = 1
Temp[1] = 0
# 将i设置为Bar, 假设我们知道如何递增和比较
LABEL("increment_temp")
cond = EQUAL(Temp,Bar)
dir0 = one
dir1 = one
# 对应i <- i+1
INC(Temp)
GOTO("increment_temp",cond)
# 如果执行到这里, i就是Bar所编码的数字
...
# 程序的最终指令
MODANDJUMP(dir0,dir1)
```

### 8.2.2 NAND-TM中的二维数组

为了实现二维数组, 我们希望将它们嵌入到一个一维数组中. 思路是通过一个*一一对应*的函数$\text{embed}:\N\times\N\to\N$, 从而将二维数组`Two`中的位置$(i,j)$嵌入到一维数组`One`的位置$\text{embed}(i,j)$中. 

由于集合$\N\times\N$看上去"远大于"集合$\N$, 先验地来看, 这样的一个双射可能并不明显存在. 然而, 一旦你深入思考, 你就会发现构建它并不算太难. 例如, 你可以让一个孩子用剪刀和胶水将一张10英寸乘10英寸的纸转换成一条1英寸乘100英寸的纸带. 这本质上就是一个从$[10]\times [10]$到$[100]$的双射. 我们可以推广这一点, 得到一个从$[n]\times[n]$到$[n^2]$的双射, 更一般地, 得到一个从$\N\times\N$到$\N$的双射. 

具体来说, 下面的$\text{embed}$函数可以做到这一点(见{{ref:fig:pairingfunction}}): 
$$
\text{embed}(x,y)=\frac{1}{2}(x+y)(x+y+1)+x
$$

```admonish quote id="pairingfunctionfig"
![](./images/chapter8/pairing_function.png)

{{pic}}{fig:pairingfunction} 映射$\text{embed}(x,y)=\frac{1}{2}(x+y)(x+y+1)+x$对于$x,y\in[10]$的示意图, 可以看出对于每一对不同的$(x,y)$和$(x',y')$, 都有$\text{embed}(x,y)\ne\text{embed}(x',y')$
```

[习题8.3]()要求你证明$\text{embed}$确实是一个双射, 并且可以由一个NAND-TM程序计算. (后者可以通过简单地遵循小学所学的乘法、加法和除法算法来完成)这意味着我们可以将形式为`Two[Foo][Bar] = something`(即, 访问二维数组中由一维数组`Foo`和`Bar`编码的整数对应的位置)替换为如下形式的代码: 

```python
Blah = embed(Foo,Bar)
Setindex(Blah)
Two[i] = something
```

### 8.2.3 其他细节

一旦我们有了二维数组和索引访问, 用NAND-TM模拟NAND-RAM就只是在NAND-TM中实现算术运算和比较的标准算法的问题了. 虽然这很繁琐, 但并不困难, 最终的结果表明每个NAND-RAM程序$P$都可以被一个等价的NAND-TM程序$Q$模拟, 从而完成了{{tref:thmc:t81}}的证明. 

```admonish note title="备注8.3: NAND-RAM中的递归(进阶)"
*递归*是许多编程语言中都出现的一个概念, 但我们没有将其包含在NAND-RAM程序中. 然而, 递归(以及一般的函数调用)可以在NAND-RAM中使用[栈数据结构](https://goo.gl/JweMj)来实现. *栈*是一种包含一系列元素的数据结构, 我们可以按照"后进先出"的顺序向其中"压入"元素和从中"弹出"元素. 

我们可以使用一个整数数组`Stack`和一个标量变量`stackpointer`(表示栈中的项目数量)来实现一个栈. 我们通过以下方式实现`push(foo)`: 

~~~python
Stack[stackpointer]=foo
stackpointer += one
~~~

并通过以下方式实现`bar = pop()`: 

~~~python
bar = Stack[stackpointer]
stackpointer -= one
~~~

我们通过将$F$的参数压入栈中来实现对$F$的函数调用. $F$的代码将从栈中"弹出"参数, 执行计算(可能涉及进行递归或非递归调用), 然后将其返回值"压入"栈中. 由于栈的"后进先出"特性, 直到所有递归调用完成, 我们才会将控制权返回给调用过程. 

我们可以使用非递归语言实现递归这一事实并不令人惊讶. 实际上, *机器语言*通常不具有递归(或一般的函数调用)功能, 因此编译器使用栈和`GOTO`来实现函数调用. 你可以在网上找到关于您最喜欢的编程语言(无论是[Python](http://interactivepython.org/runestone/static/pythonds/Recursion/StackFramesImplementingRecursion.html)、[JavaScript](https://javascript.info/recursion)还是[Lisp/Scheme](https://mitpress.mit.edu/sites/default/files/sicp/full-text/sicp/book/node110.html))中如何通过栈实现递归的教程. 
```

## 8.3 图灵等价性(讨论)

```admonish pic id="fortranprogfig"
![](./images/chapter8/FortranProg.jpg)

{{pic}}{fig:fortranprog} 表示一条Fortran语句的打孔卡片
```

任何标准编程语言, 如`C`、`Java`、`Python`、`Pascal`、`Fortran`, 其操作都与NAND-RAM非常相似. (事实上, 它们最终都可以由具有固定数量寄存器和大型内存阵列的机器来执行)因此, 使用{{tref:thmc:t81}}, 我们可以用NAND-TM程序来模拟任何此类编程语言中的程序. 反过来, 在任何上述编程语言中编写一个NAND-TM的解释器是一个相当简单的编程练习. 因此, 我们也可以使用这些编程语言来模拟NAND-TM程序(进而通过[定理7.11]()来模拟图灵机). 这种在计算能力上等同于图灵机/NAND-TM的特性被称为*图灵等价*(有时也称为*图灵完备*). 因此, 我们熟悉的所有编程语言都是图灵等价的. {{footnote: 一些编程语言可以访问的内存量有固定的(即使非常大)上限, 这正式地阻止了它们适用于计算无限函数并因此模拟图灵机. 我们在本次讨论中忽略此类问题, 并假定可以访问某种容量没有固定上限的存储设备. }}

### 8.3.1 "两全其美"的范式

图灵机与RAM机之间的等价性使我们能够为手头的任务选择最方便的语言: 

- 当我们想要*证明*一个关于所有程序/算法的定理时, 我们可以使用图灵机(或NAND-TM), 因为它们更简单且易于分析. 特别是, 如果我们想证明某个函数无法被计算, 那么我们将使用图灵机. 
- 当我们想要证明某个函数*可以被计算*时, 我们可以使用RAM机器或NAND-RAM, 因为它们更容易编程, 并且更接近于我们习惯的高级编程语言. 事实上, 我们通常会以非正式的方式描述NAND-RAM程序, 并相信读者能够填充细节并将简略的描述转换为精确的程序. (这就像人们通常使用非正式的或"伪代码"的算法描述方式, 并相信他们的受众知道在需要时将这些描述转换为代码一样)

我们对图灵机/NAND-TM和RAM机/NAND-RAM的使用, 与人们在实践中使用高级和低级编程语言的方式非常相似. 当人们想要制造一个执行程序的设备时, 为一种非常简单和"低级"的编程语言来实现是很方便的. 当人们想要描述一个算法时, 使用尽可能高级的形式体系是方便的. 

```admonish pic id="have_your_cake_and_eat_it_toofig"
![](./images/chapter8/have_your_cake_and_eat_it_too-img-intro.png)

{{pic}}{fig:have_your_cake_and_eat_it_too} 通过拥有两种等价语言NAND-TM和NAND-RAM, 我们可以"鱼与熊掌兼得": 当我们想证明程序不能做某事时, 使用NAND-TM；当我们想证明程序能做某事时, 使用NAND-RAM或其他高级语言
```

```admonish bigidea id="bd10"
利用图灵机和RAM机之间的等价性, 我们可以"鱼与熊掌兼得". 

当我们想证明某事无法完成时, 可以使用更简单的模型(如图灵机)；当我们想证明某事可以完成时, 可以使用功能丰富的模型(如RAM机). 
```

### 8.3.2 浅谈抽象层次

```admonish quote
程序员处于一个独特的位置……他必须能够思考概念层次结构, 其深度是单个思维以前从未需要面对的.

*——Edsgar Dijkstra, 《论真正教授计算机科学的残酷性》, 1988年. *
```

在任何计算理论课程中的某个时刻, 教师和学生都需要进行*那次*谈话. 也就是说, 我们需要讨论描述算法时的*抽象层次*. 在算法课程中, 通常用英语描述算法{{footnote: 译者注: 在本翻译版中会使用中文}}, 假设读者能够"填充细节", 并在需要时能够将此类算法转化为实现. 例如, {{tref:algc:a81}}是广度优先搜索算法的高级描述. 

```admonish quote title=""
{{algc}}{algc:a81}(广度优先搜索)
$$
\begin{aligned}
&\mathbf{Input: }图G, 顶点u,v\\
&\mathbf{Output: }当u与v在图中联通时, 返回"\text{connected}", 否则返回"\text{disconnected}"\\
&初始化一个空队列Q\\
&将u放入Q中\\
&\textbf{while}\{Q不为空\}\\
&\quad 将队列顶部的顶点w从Q中移除\\
&\quad\textbf{if}\{w=v\}\textbf{ return }\text{connected}\textbf{ endif}\\
&\quad 标记w\\
&\quad 将w的所有未被标记的邻居加入Q\\
&\textbf{endwhile}\\
&\textbf{return }\text{disconnected}
\end{aligned}
$$
```

如果我们想提供关于如何在Python或C(或NAND-RAM/NAND-TM)等编程语言中实现广度优先搜索的更多细节, 我们会描述如何用数组实现队列数据结构, 以及同样如何用数组标记顶点. 我们称这种"中间层次"的描述为*实现级别*(implementation level)或*伪代码*描述. 最后, 如果我们想精确地描述实现, 我们会给出程序的全部代码(或另一个完全精确的表示形式, 例如元组列表的形式). 我们称之为*形式化*或*低级*(low level)描述. 

```admonish pic id="levelsofdescriptionfig"
![](./images/chapter8/levelsofdescription.png)

{{pic}}{fig:levelsofdescription} 我们可以用不同的粒度/细节和精确度级别来描述一个算法. 在最高级别, 我们只用文字描述想法, 省略所有关于表示和实现的细节. 在中间级别(也称为实现或伪代码), 我们提供足够的实现细节, 使他人能够推导出它, 但我们仍然不提供完整代码. 最低级别是实际代码或数学描述被完整阐述的地方. 这些不同的细节层次都有其用途, 在它们之间转换是计算机科学家最重要的技能之一
```

虽然我们开始时是在完全形式化的层面上描述NAND-CIRC、NAND-TM和NAND-RAM程序, 随着本书的深入, 我们将转向实现级别和高级别的描述. 毕竟, 我们的目标不是使用这些模型进行实际计算, 而是分析计算的一般现象. 也就是说, 如果你不理解高级描述如何转化为实际实现, "深入底层"通常是一个极好的练习. 计算机科学家最重要的技能之一就是能够在抽象层次结构中上下移动. 

类似的区别也适用于将对象表示为字符串的概念. 有时, 为了精确起见, 我们会给出一个*低级规范*(low level specification), 确切说明一个对象如何映射到二进制字符串. 例如, 我们可能将$n$个顶点的图的编码描述为长度为$n^2$的二进制字符串, 通过说明我们将顶点集为$[n]$的图$G$映射到字符串$x\in \{0,1\}^{n^2}$, 其中$x$的第$n\cdot i + j$个坐标是$1$当且仅当边$\overrightarrow{i ; j}$存在于$G$中. 我们也可以使用*中间*或*实现*级别的描述, 只需简单说明我们使用邻接矩阵表示法来表示图. 

最后, 因为图(以及一般对象)的各种表示之间的转换可以通过NAND-RAM(因此也可以通过NAND-TM)程序完成, 所以在进行高级别讨论时, 我们也会避免关于表示的讨论. 例如, 图连通性是一个可计算函数, 这一事实无论我们是用邻接表、邻接矩阵、边对列表等表示图都是成立的. 因此, 在精确表示无关紧要的情况下, 我们通常会谈论我们的算法将对象$X$(可以是图、向量、程序等)作为输入, 而不指定$X$如何被编码为字符串. 

**定义"算法"**: 到目前为止, 我们一直非正式地使用"算法"这个术语. 然而, 图灵机和一系列等效模型产生了一种精确且形式化地定义算法的方法. 因此, 在本书中, 每当我们提到算法时, 我们指的是它是图灵等效模型(如图灵机、NAND-TM、RAM机等)中的一个实例. 由于所有这些模型的等价性, 在许多情况下, 我们使用哪一个并不重要. 

### 8.3.3 图灵完备性与等价性的形式化定义(可选) {#turingcompletesec }

一个*计算模型*是某种定义*程序*(由字符串表示)计算(部分)函数的方式. 一个*计算模型*$\mathcal{M}$是*图灵完备*的, 如果我们可以将每个图灵机(或等价的NAND-TM程序)$N$映射到$\mathcal{M}$中的一个程序$P$, 使得$P$计算与$N$相同的函数. 它是*图灵等价*的, 如果另一个方向也成立(即, 我们可以将$\mathcal{M}$中的每个程序映射到一个计算相同函数的图灵机). 我们可以形式化地定义这个概念如下. (这个形式化定义对于本书的其余部分并不关键, 只要你理解图灵等价的一般概念就可以跳过它；这个概念在文献中有时被称为[哥德尔数](https://goo.gl/rzuNPu)(Gödel numbering)或[可接纳数](https://goo.gl/xXJoUG)(admissible numbering))

```admonish quote title=""
{{defc}}{defc:d85}(图灵完备性与等价性(可选))

令$\mathcal{F}$为所有从$\{0,1\}^*$到$\{0,1\}^*$的部分函数的集合. 一个*计算模型*是一个映射$\mathcal{M}:\{0,1\}^* \rightarrow \mathcal{F}$. 

我们说一个程序$P \in \{0,1\}^*$ $\mathcal{M}$-*计算*一个函数$F\in \mathcal{F}$, 如果$\mathcal{M}(P) = F$. 

一个计算模型$\mathcal{M}$是*图灵完备*的, 如果存在一个可计算映射$ENCODE_{\mathcal{M}}:\{0,1\}^*\rightarrow\{0,1\}^*$, 使得对于每个图灵机$N$(表示为字符串), $\mathcal{M}(ENCODE_{\mathcal{M}}(N))$等于由$N$计算的部分函数. 

一个计算模型$\mathcal{M}$是*图灵等价*的, 如果它是图灵完备的, 并且存在一个可计算映射$DECODE_{\mathcal{M}}:\{0,1\}^*\rightarrow\{0,1\}^*$, 使得对于每个字符串$P\in \{0,1\}^*$, $N=DECODE_{\mathcal{M}}(P)$是一个计算函数$\mathcal{M}(P)$的图灵机的字符串表示. 
```

一些图灵等价模型的例子(其中一些我们已经见过, 一些将在下面讨论)包括: 

- 图灵机
- NAND-TM程序
- NAND-RAM程序
- λ演算
- 生命游戏(将程序和输入/输出映射到起始和结束格局)
- 编程语言, 如Python/C/Javascript/OCaml...(允许无限存储)

## 8.4 元胞自动机 {#cellularautomatasec }

许多物理系统可以被描述为由大量相互作用的基元组件组成. 一种模拟此类系统的方法是使用*元胞自动机*. 这是一个由大量(甚至无限)细胞组成的系统. 每个细胞只有有限个可能的状态. 在每个时间步, 细胞通过将某个简单规则应用于自身及其邻居的状态来更新到新状态. 

```admonish pic id="conwaysgridsfig"
![](./images/chapter8/conwaysgrids.png)

{{pic}}{fig:conwaysgrids} 康威生命游戏的规则. 图片来自[此博客文章](https://mblogscode.wordpress.com/2017/06/07/python-simulation-coding-conways-game-of-life/)
```

元胞自动机的一个典型例子是[康威的生命游戏](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)(Conway's Game of Life). 在此自动机中, 细胞排列在一个无限二维网格中. 每个细胞只有两种状态: "死亡"(我们可以编码为$0$并标识为$\varnothing$)或"存活"(我们可以编码为$1$). 细胞的下一个状态取决于其先前状态及其8个垂直、水平和对角线邻居的状态(参见{{ref:fig:conwaysgrids}}). 死亡细胞只有在恰好有三个存活邻居时才会变为存活. 存活细胞只有在有两个或三个存活邻居时继续存活. 尽管细胞数量可能无限, 但我们可以通过仅跟踪存活细胞来使用有限长度字符串编码状态. 如果我们在具有有限数量存活细胞的格局中初始化系统, 那么在所有未来步骤中存活细胞的数量将保持有限. 生命游戏的[维基百科页面](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)上有一些产生非常有趣演化的格局的美丽图形和动画. 

```admonish pic id="onetwodimensionalcafig"
![](./images/chapter8/onetwodimensionalca.png)

{{pic}}{fig:onetwodimensionalca} 在二维元胞自动机中, 每个细胞位于某个整数$i,j \in \Z$的位置$i,j$上. 细胞的状态是某个值$A_{i,j} \in \Sigma$, 其中$\Sigma$是某个有限字母表. 在给定时间步, 细胞的状态根据应用于$(i,j)$及其所有邻居$(i \pm 1, j\pm 1)$状态的某个函数进行调整. 在一维元胞自动机中, 每个细胞位于位置$i\in \Z$上, 且$i$在下一个时间步的状态$A_i$取决于其当前状态及其两个邻居$i-1$和$i+1$的状态
```

由于生命游戏中的细胞排列在无限二维网格中, 它是二维元胞自动机的一个例子. 我们也可以考虑*一维元胞自动机*的更简单设置, 其中细胞排列在一条无限直线上, 参见{{ref:fig:onetwodimensionalca}}. 事实证明, 即使这个简单模型也足以实现图灵完备性. 我们现在将正式定义一维元胞自动机, 然后证明它们的图灵完备性. 

```admonish quote title=""
{{defc}}{defc:d86}(一维元胞自动机)

设$\Sigma$是一个包含符号$\varnothing$的有限集合. 一个在字母表$\Sigma$上的*一维元胞自动机*由一个*转移规则*$r:\Sigma^3 \rightarrow \Sigma$描述, 该规则满足$r(\varnothing,\varnothing,\varnothing) = \varnothing$. 

自动机$r$的一个*格局*(configuration)是一个函数$A:\Z \rightarrow \Sigma$. 如果具有规则$r$的自动机处于格局$A$, 那么它的下一个格局, 记为$A' = NEXT_r(A)$, 是函数$A'$, 使得对于每个$i\in \Z$, 有$A'(i) = r(A(i-1),A(i),A(i+1))$. 换句话说, 自动机$r$在点$i$的下一个状态是通过将规则$r$应用于$A$在$i$及其两个邻居的值得到的. 
```

**有限格局**: 如果自动机$r$的格局中只有有限个索引$i_0,\ldots,i_{j-1}$在$\Z$中使得$A(i_j) \neq \varnothing$, 则我们称该格局是_有限的_. (即, 对于每个$i \not\in { i_0, \ldots, i_{j-1}}$, 有$A(i)=\varnothing$)这样的格局可以使用一个有限字符串表示, 该字符串编码索引$i_0,\ldots,i_{n-1}$和值$A(i_0),\ldots,A(i_{n-1})$. 由于$r(\varnothing,\varnothing,\varnothing)=\varnothing$, 如果$A$是有限格局, 则$NEXT_r(A)$也是有限的. 我们只关心在有限格局中初始化的元胞自动机, 因此在其整个演化过程中保持有限格局. 

### 8.4.1 一维元胞自动机的图灵完备性

我们可以编写一个程序(例如使用NAND-RAM)来模拟任何元胞自动机从初始有限格局的演化, 只需存储状态不等于$\varnothing$的细胞值并重复应用规则$r$. 因此, 元胞自动机可以被图灵机模拟. 更令人惊讶的是, 反过来也成立. 例如, 尽管其规则简单, 我们可以使用生命游戏模拟图灵机(参见{{ref:fig:turinggol}}). 

```admonish pic id="turinggolfig"
![](./images/chapter8/turing_gol.jpg)

{{pic}}{fig:turinggol} 模拟图灵机的生命游戏格局. 图片由[Paul Rendell](http://rendell-attic.org/gol/tm.htm)提供
```

事实上, 即使一维元胞自动机也可以是图灵完备的: 

```admonish quote title=""
{{thmc}}{thmc:t87}(一维自动机是图灵完备的)

对于每个图灵机$M$, 存在一个一维元胞自动机, 可以在每个输入$x$上模拟$M$. 
```

为了使"模拟图灵机"的概念更精确, 我们需要定义图灵机的*格局*. 我们将在下面的[8.4.2节](#842-图灵机格局与状态转移函数)中这样做, 但高层面上, 图灵机的*格局*是一个字符串, 编码了其在计算中给定步骤的完整状态. 即, 其磁带所有(非空)单元的内容、其当前状态以及磁头位置. 

{{tref:thmc:t87}}的证明的关键思想是, 在图灵机$M$的计算中的每个点, $M$的磁带中唯一能改变的单元是磁头所在的位置, 并且该单元改变的值是其当前状态和$M$的有限状态的函数. 这一观察使我们能够将图灵机$M$的格局编码为一个元胞自动机$r$的有限格局, 并确保此编码格局在$r$的规则下的一步演化对应于图灵机$M$执行中的一步. 

### 8.4.2 图灵机格局与状态转移函数 {#turingmachinesconfigsec }

为了将上述思想转化为{{tref:thmc:t87}}的严格证明(甚至陈述! ), 我们需要精确定义图灵机的*格局*这一概念. 这个概念在后续章节中对我们也有用. 

```admonish pic id="turingmachineconffig"
![](./images/chapter8/turingmachineconf.png)

{{pic}}{fig:turingmachineconf} 具有字母表$\Sigma$和状态空间$[k]$的图灵机$M$的_格局_将其在执行中特定步骤的状态编码为一个在字母表$\overline{\Sigma} = \Sigma \times ({\cdot } \cup [k])$上的字符串$\alpha$. 字符串的长度为$t$, 其中$t$满足$M$的磁带在所有位置$t$及更大处包含$\varnothing$, 且$M$的磁头位于小于$t$的位置. 如果$M$的磁头在第$i$个位置, 那么对于$j \neq i$, $\alpha_j$编码$M$磁带的第$j$个单元的值, 而$\alpha_i$编码此值以及$M$的当前状态. 如果机器写入值$\tau$, 更改状态为$t$, 并向右移动, 那么在下一个格局中, 位置$i$将包含值$(\tau,\cdot)$, 位置$i+1$将包含值$(\alpha_{i+1},t)$
```

```admonish quote title=""
{{defc}}{defc:d88}(图灵机的格局)

设$M$是一个具有磁带字母表$\Sigma$和状态空间$[k]$的图灵机. $M$的一个*格局*是一个字符串$\alpha \in \overline{\Sigma}^*$, 其中$\overline{\Sigma} = \Sigma \times \left( {\cdot} \cup [k] \right)$, 满足存在恰好一个坐标$i$, 使得对于某个$\sigma \in \Sigma$和$s\in [k]$, 有$\alpha_i = (\sigma,s)$. 对于所有其他坐标$j$, $\alpha_j = (\sigma',\cdot)$, 其中$\sigma'\in \Sigma$. 

$M$的格局$\alpha \in \overline{\Sigma}^*$对应于其执行的以下状态: 

- $M$的磁带对于所有$j<|\alpha|$包含$\alpha_{j,0}$, 对于所有至少为$|\alpha|$的位置包含$\varnothing$, 其中我们令$\alpha_{j,0}$为值$\sigma$, 使得$\alpha_j = (\sigma,t)$, 其中$\sigma \in \Sigma$且$t \in {\cdot } \cup [k]$. (换句话说, 由于$\alpha_j$是一个字母表符号$\sigma$和一个$[k]$中的状态或符号$\cdot$的对, $\alpha_{j,0}$是这个对的第一个分量$\sigma$)
- $M$的磁头位于唯一位置$i$, 其中$\alpha_i$具有形式$(\sigma,s)$, $s\in [k]$, 且$M$的状态等于$s$. 
```

```admonish pause title = "暂停一下"
{{tref:defc:d88}}有一些技术细节, 但实际上并不深奥或复杂. 尝试花点时间停下来思考*你*如何将图灵机在执行中给定点的状态编码为一个字符串. 

思考你需要知道哪些组件才能从此点继续执行, 以及使用有限符号列表编码它们的简单方法是什么. 特别是, 考虑到我们未来的应用, 尝试思考一种编码, 使得将步骤$t$的格局映射到步骤$t+1$的格局尽可能简单. 
```

{{tref:defc:d88}}有点繁琐, 但无论怎么讲格局只是一个字符串, 编码了图灵机在执行中给定点的*快照*. (用操作系统术语, 它是一个"核心转储"(core dump))这样的快照需要编码以下组件: 

1. 当前磁头位置. 
2. 大容量存储器的完整内容, 即磁带. 
3. "本地寄存器"的内容, 即机器的状态. 

我们如何编码格局的精确细节并不重要, 但我们确实想记录以下简单事实: 

```admonish quote title=""
{{lemc}}{lemc:l89}

设$M$是一个图灵机, 令$NEXT_M:\overline{\Sigma}^* \rightarrow \overline{\Sigma}^*$是将$M$的格局映射到执行下一步格局的函数. 那么对于每个$i \in \N$, $NEXT_M(\alpha)i$的值仅依赖于坐标$\alpha_{i-1},\alpha_i,\alpha_{i+1}$. 
```

(为简化记号, 上面我们使用约定: 如果$i$"越界", 例如$i<0$或$i>|\alpha|$, 则我们假设$\alpha_i = (\varnothing,\cdot)$)我们将{{tref:lemc:l89}}的证明留作[练习8.7](). 证明背后的思想很简单: 如果磁头既不在位置$i$, 也不在位置$i-1$和$i+1$, 那么$i$处的下一步格局将与之前相同. 否则, 我们可以从$i$或其邻居的格局中"读取"图灵机的状态和磁头位置的磁带值, 并用其更新$i$处的新状态应该是什么. 完成完整证明并不难, 但这样做是确保你熟悉格局定义的好方法. 

**完成{{tref:thmc:t87}}的证明**: 我们现在可以更正式地重述{{tref:thmc:t87}}, 并完成其证明: 

```admonish quote title=""
{{thmc}}{thmc:t810}(一维自动机是图灵完备的(形式化陈述))

对于每个图灵机$M$, 如果我们用$\overline{\Sigma}$表示其格局字符串的字母表, 那么存在一个在字母表$\overline{\Sigma}^*$上的一维元胞自动机$r$, 使得
$$
NEXT_M(\alpha)=NEXT_r(\alpha)
$$
对于$M$的每个格局$\alpha \in \overline{\Sigma}^*$(再次使用约定: 如果$i$"越界", 则我们考虑$\alpha_i=\varnothing$). 
```

```admonish proof collapsible=true, title = "证明"
我们将$\overline{\Sigma}$的元素$(\varnothing,\cdot)$对应于自动机$r$的$\varnothing$元素. 在这种情况下, 由{{tref:lemc:l89}}, 将$M$的格局映射到下一个格局的函数$NEXT_M$实际上是一维自动机的有效规则. 
```

从{{tref:thmc:t810}}的证明中产生的自动机具有大的字母表, 而且其大小依赖于被模拟的机器$M$. 事实证明, 人们可以获得一个具有固定大小字母表的自动机, 该字母表独立于被模拟的程序, 实际上自动机的字母表可以是最小集合$\{0,1\}$! {{ref:fig:rule110big}}展示了这样的一个图灵完备的自动机. 

```admonish quote id="rule110bigfig"
![](./images/chapter8/Rule110Big.jpg)

{{pic}}{fig:rule110big} 一维自动机的演化. 图中的每一行对应一个格局. 初始格局对应顶行, 仅包含一个"存活"细胞. 此图对应Stephen Wolfram的"规则110"自动机, 它是图灵完备的. 图片取自[Wolfram MathWorld](http://mathworld.wolfram.com/Rule110.html)
```

```admonish note title="备注8.11: NAND-TM程序的格局"
我们可以使用与{{tref:defc:d88}}相同的方法来定义*NAND-TM程序*的格局. 这样的格局需要编码: 

1. 变量`i`的当前值. 
2. 对于每个标量变量`foo`, `foo`的值. 
3. 对于每个数组变量`Bar`, 值`Bar[`$j$`]`对于每个$j \in {0,\ldots, t-1}$, 其中$t-1$是指标变量`i`在计算中曾达到的最大值. 
```

## 8.5 λ演算与函数式编程语言 { #lambdacalculussec }

[λ演算](https://goo.gl/B9HwT8)是定义可计算函数的另一种方式. 它有Alonzo Church在1930年代提出, 大约与Alan Turing提出图灵机同时. 有趣的是, 尽管图灵机不用于实际计算, λ演算却催生了函数式编程语言, 如Lisp、ML和Haskell, 并间接地促进了许多其他编程语言的发展. 在本节中, 我们将介绍λ演算并展示其能力等价于NAND-TM程序(因此也等价于图灵机). 我们的[Github仓库](https://github.com/boazbk/tcscode)包含一个Jupyter Notebook, 其中有一个λ演算的Python实现, 你可以通过实验来更好地理解这个话题. 

**λ算子**: λ演算的核心是定义"匿名"函数的一种方式. 例如, 有一个函数$f$的定义为
$$
f(x)=x\times x
$$
我们可以将其写为
$$
\lambda x.x\times x
$$
因此$(\lambda x.x\times x)(7)=49$. 也就是说, 你可以将$\lambda x. exp(x)$(其中$exp$是某个表达式)视为指定匿名函数$x \mapsto exp(x)$的一种方式. 匿名函数使用$\lambda x.f(x)$、$x \mapsto f(x)$或其他密切相关的表示法, 出现在许多编程语言中. 例如, 在*Python*中我们可以使用`lambda x: x*x`来定义平方函数, 而在*JavaScript*中我们可以使用`x => x*x`或`(x) => x*x`. 在*Scheme*中我们会将其定义为`(lambda (x) (* x x))`. 显然, 函数的参数名称无关紧要, 因此$\lambda y.y\times y$与$\lambda x.x \times x$相同, 因为两者都对应平方函数. 

**省略括号**: 为了减少表示上的杂乱, 在书写λ演算表达式时我们经常省略函数求值的括号. 因此, 与其将函数$f$应用于输入$x$的结果写为$f(x)$, 我们也可以简单地写为$f\; x$. 因此我们可以写$(\lambda x.x\times x) 7=49$. 在本章中, 我们将同时使用$f(x)$和$f\; x$表示法进行函数应用. 函数求值是结合性的, 并从左到右绑定, 因此$f;g;h$与$(f g) h$相同. 

### 8.5.1 函数的高阶应用

λ演算的一个核心特性是函数都是"一等公民", 即我们可以将函数作为其他函数的参数. 比如说, 你能猜到下面这个表达式等于什么数字吗?

$$
(((\lambda f.(\lambda y.(f\; (f\; y))))(\lambda x.x\times x))\; 3) {{numeq}}{eq:lambdaexampleeq}
$$

```admonish pause title = "暂停一下"
{{eqref:eq:lambdaexampleeq}}可能看上去有点吓人, 但在你看下面的解答之前, 尝试将其分解为各个组成部分, 并一次计算一个部分. 完成这个例题将极大地有助于理解λ演算
```

让我们一步一步地计算{{eqref:eq:lambdaexampleeq}}. 尽管允许匿名函数是λ演算的优势, 但添加名称对于理解复杂表达式非常有帮助. 因此, 我们令$F=\lambda f.(\lambda y.(f\; (f\; y)))$与$g=\lambda x.x\times x$. 

因此, {{eqref:eq:lambdaexampleeq}}可以写作
$$
((F\; g)\; 3)
$$
在输入函数$f$时, $F$输出函数$\lambda y.(f\; (f\; y))$, 换而言之, $F\; f$是函数$y\mapsto f(f(y))$. 我们的函数$g$是简单的$g(x)=x^2$, 因此$(F\; g)$是将$y$映射到$(y^2)^2=y^4$的函数. 因此$((F\; g)\; 3)=3^4=81$. 

```admonish question
{{exec}}{exe:lambdaexptwoex}[λ表达式求值练习]

下面的这个λ表达式等于什么数字?

$$
((\lambda x.(\lambda y.x)) \; 2)\; 9 \;. {{numeq}}{eq:lambdaexptwoeq}
$$
```

```admonish solution collapsible=true title="对{{ref:exe:lambdaexptwoex}}的解答"
$\lambda y.x$是一个函数, 其在输入$y$时忽略其输入并返回$x$.

因此, $(\lambda x.(\lambda y.x))\; 2$的结果是函数$y\mapsto 2$(或者使用λ符号写作函数$\lambda y.2$).

因此, {{eqref:eq:lambdaexptwoeq}}等价于$(\lambda y. 2) 9 = 2$.
```

### 8.5.2 通过柯里化实现多参数函数 { #curryingsec }
在形如$\lambda x.e$的λ表达式中, 表达式$e$本身也可以包含λ运算符. 比如如下函数
$$
\lambda x. (\lambda y. x+y) {{numeq}}{eq:eqlambdaexampleone}
$$
将$x$映射到函数$y\mapsto x+y$.

特别地, 若我们使用$a$调用函数{{eqref:eq:eqlambdaexampleone}}得到某个函数$f$, 再以$b$调用$f$, 便可获得值$a+b$. 可以看出, 对应于$a\mapsto (b\mapsto a+b)$的单参数函数{{eqref:eq:eqlambdaexampleone}}亦可视为双参数函数$(a,b)\mapsto a+b$. 一般地, 我们可以使用λ表达式$\lambda x.(\lambda y.f(x,y))$来模拟双参数函数$(x,y)\mapsto f(x,y)$的效果, 这一技巧被称为[*柯里化*(Currying)](https://en.wikipedia.org/wiki/Currying). 我们将使用$\lambda x,y.e$作为$\lambda x.(\lambda y.e)$的简写形式. 若$f=\lambda x.(\lambda y.e)$, 则$(f\; a)\; b$对应于对$f\; a$进行求值后, 将所得函数作用于$b$, 从而获得将$e$中$x$出现处替换为$a$, $y$出现处替换为$b$的结果. 根据结合律, 该结果等价于$(f\; a\; b)$, 有时我们也写作$f(a,b)$.

```admonish pic id="curryingfig"
![](./images/chapter8/currying.png)

{{pic}}{fig:currying} 在"柯里化"转换中, 我们可以通过λ表达式$\lambda x.(\lambda y. f(x,y))$实现双参数函数$f(x,y)$的效果: 当输入$x$时, 该表达式会输出一个单参数函数$f_x$, 其中$x$已被"硬编码"至函数内, 且满足$f_x(y)=f(x,y)$. 这一过程可通过电路图直观展示, 详见[Chelsea Voss的网站](https://tromp.github.io/cl/diagrams.html).
```

### 8.5.3 λ演算的形式化描述
我们现在提供λ演算的形式描述. 我们从包含单个变量的"基本表达式"开始, 例如$x$或$y$, 并构建更复杂的表达式, 形为$(e\; e')$和$\lambda x.e$, 其中$e,e'$是表达式, $x$是变量标识符. 形式上, λ表达式的定义如下:
```admonish quote title=""
{{defc}}{def:lambdaexpdef}[λ表达式]

一个*λ表达式*要么是一个单独的变量标识符, 要么是以下形式之一的表达式$e$:
- **应用(Application)**: $e = (e' \; e'')$, 其中$e'$和$e''$是λ表达式.
- **抽象(Abstraction)**: $e = \lambda x.(e')$, 其中$e'$是λ表达式.
```

{{ref:def:lambdaexpdef}}是一个*递归定义*, 因为我们在λ表达式的定义中使用了其自身. 这可能起初看起来令人困惑, 但事实上你从小学起就已经知道递归定义. 考虑我们如何定义*算术表达式*: 它是一个表达式, 要么只是一个数字, 要么具有形式$(e + e')$, $(e - e')$, $(e \times e')$, 或 $(e \div e')$, 其中$e$和$e'$是其他算术表达式.

*自由变量和绑定变量*: λ表达式中的变量可以是*自由的*(free)或*绑定*(bound)到一个$\lambda$运算符(在[1.4.7节](./chapter_1.md#boundvarsec)的意义上). 在单变量λ表达式$var$中, 变量$var$是自由的. 在应用表达式$e = (e' \; e'')$中, 自由和绑定变量的集合与底层表达式$e'$和$e''$的相同. 在抽象表达式$e = \lambda var.(e')$中, $e'$中$var$的所有自由出现(free occurences)都被绑定到$e$的$\lambda$运算符. 如果你觉得自由和绑定变量的概念令人困惑, 你可以通过为所有变量使用唯一标识符来避免所有这些问题.

*优先级和括号*: 我们将使用以下规则来允许我们省略一些括号. 函数应用从左向右结合, 因此$f\; g\; h$与$(f\; g)\; h$相同. 函数应用的优先级高于λ运算符, 因此$\lambda x.f\; g\; x$与$\lambda x.((f\; g)\; x)$相同. 这类似于我们在算术运算中使用优先级规则来允许我们使用更少的括号, 比如表达式$(7 \times 3) + 2$可以写成$7\times 3 + 2$. 如[8.5.2节](#curryingsec)所述, 我们还使用简写$\lambda x,y.e$表示$\lambda x.(\lambda y.e)$, 以及简写$f(x,y)$表示$(f\; x)\; y$. 这与使用λ表达式模拟多输入函数的"柯里化"转换很好地配合.

**λ表达式的等价性**: 正如我们在{{ref:exe:lambdaexptwoex}}中看到的,规则$(\lambda x. exp)\; exp'$等价于$exp[x \rightarrow exp']$使我们能够修改λ表达式并获得更简单的*等价形式*. 另一个我们可以使用的规则是参数名称无关紧要, 因此$\lambda y.y$与$\lambda z.z$相同. 这些规则一起定义了λ表达式的*等价性*概念:

```admonish quote title=""
{{defc}}{def:lambdaequivalence}[λ表达式的等价性]

两个λ表达式是*等价的*, 如果它们可以通过重复应用以下规则变成相同的表达式:
1. **求值(即$\beta$归约)**: 表达式$(\lambda x.exp) exp'$等价于$exp[x \rightarrow exp']$.
2. **变量重命名(即$\alpha$转换)**: 表达式$\lambda x.exp$等价于$\lambda y.exp[x \rightarrow y]$.
```

如果$exp$是一个形式为$\lambda x.exp'$的λ表达式, 那么它自然对应于将任何输入$z$映射到$exp'[x \rightarrow z]$的函数. 因此, λ演算自然隐含了一个计算模型. 由于在λ演算中, 输入本身可以是函数, 我们需要决定以什么顺序求值一个表达式, 例如

$$
(\lambda x.f)(\lambda y.g\; z) {{numeq}}{eq:lambdaexpeq}
$$

对此有两种自然约定:
- **按名调用(Call-by-name, 即"惰性求值")**: 我们通过先将右侧表达式$(\lambda y.g; z)$作为输入代入左侧函数来求值{{eqref:eq:lambdaexpeq}}, 得到$f[x \rightarrow (\lambda y.g; z)]$然后从此继续.
- **按值调用(Call-by-value, 即"立即求值")**: 我们先对右侧进行求值并得到$h=g[y \rightarrow z]$, 然后将其代入左侧得到$f[x \rightarrow h]$来求值{{eqref:eq:lambdaexpeq}}.
- 
因为λ演算只有*纯*函数, 没有"副作用", 所以在许多情况下顺序无关紧要. 事实上, 可以证明如果我们在两种策略中都得到一个确定的不可约表达式(irreducible expression)(例如, 一个数字), 那么它将是同一个. 然而, 为具体起见, 我们将始终使用"按名调用"(即惰性求值)顺序. (编程语言Haskell也做出了相同的选择, 尽管许多其他编程语言使用立即求值)形式上, 使用"按名调用"求值λ表达式的过程由以下过程描述:

```admonish quote title=""
{{defc}}{def:simplifylambdadef}[λ表达式的简化]

令$e$为一个λ表达式. $e$的*简化*是以下递归过程的结果:
1. 如果$e$是一个单独变量$x$, 那么$e$的简化是$e$.
2. 如果$e$具有形式$e= \lambda x.e'$, 那么$e$的简化是$\lambda x.f'$其中$f'$是$e'$的简化.
3. *求值/$\beta$归约*: 如果$e$具有形式 $e=(\lambda x.e' \; e'')$, 那么$e$的简化是$e'[x \rightarrow e'']$的简化,这表示将$e'$中绑定到$\lambda$运算符的所有$x$的出现替换为$e''$.
4. *重命名/$\alpha$转换*: $e$的*规范简化*(canonical simplification)通过取$e$的简化并重命名变量得到, 使得表达式中的第一个绑定变量是$v_0$, 第二个是$v_1$, 依此类推.

我们说两个λ表达式$e$和$e'$是*等价的*, 记为$e \cong e'$, 如果它们具有相同的规范简化.
```

```admonish question
{{exec}}{exe:lambdaeuivexer}[λ表达式等价判断练习]

证明以下两个表达式$e$和$f$是等价的:

$$e = \lambda x.x$$

$$f = (\lambda a.(\lambda b.b)) (\lambda z.z\; z)$$
```

```admonish solution collapsible=true title="对{{ref:exe:lambdaeuivexer}}的解答"
$e$的规范简化就是$\lambda v_0.v_0$. 为了计算$f$的规范简化, 我们首先使用$\beta$归约将$\lambda z.z\; z$代入$(\lambda b.b)$中的$a$, 但由于$a$在这个函数中根本未被使用, 我们简单地得到$\lambda b.b$, 它同样简化为$\lambda v_0.v_0$.
```

### 8.5.4 λ演算中的无限循环

与图灵机和NAND-TM程序类似, λ演算中的简化过程也可能进入无限循环. 例如, 考虑以下λ表达式

$$
\lambda x.xx \; \lambda x.xx {{numeq}}{eq:lambdainfloopeq}
$$

若我们尝试通过将左侧函数作用于右侧函数来简化{{eqref:eq:lambdainfloopeq}}, 则会得到另一个{{eqref:eq:lambdainfloopeq}}的副本，因此该过程永不休止。在某些情况下，求值顺序会影响表达式是否可被简化，具体参见[习题8.9]()。

## 8.6 增强λ演算

### 8.6.1 增强λ演算中的函数计算

### 8.6.2 增强λ演算的图灵完备性

## 8.7 从增强λ演算到纯λ演算

### 8.7.1 列表处理

### 8.7.2 Y组合子: 不需要递归的递归

## 8.8 Church-Turing论题(讨论)

### 8.8.1 不同的计算模型

## 8.9 习题

## 8.10 参考文献