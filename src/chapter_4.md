** 本章仍在翻译中 **

<!-- toc -->

# 4. 语法糖与通用函数计算 { #finiteuniversalchap }

## 学习目标{ .objectives }
* 习惯于语法糖或高级逻辑到低级门电路的自动转换. 
* 学习重要结论的证明: 任何有限函数都能通过布尔电路计算. 
* 开始从量化角度思考计算过程所需的代码行数. 

```admonish quote
[于1951年] 我曾有一个能运行的编译器, 但没人愿意碰它, 因为他们谨慎地告诉我, 计算机只能做算术, 不能执行程序.

*-Grace Murray Hopper, 1986.*
```

```admonish quote
语法糖会引起分号癌.

*-Alan Perlis, 1982.*
```

我们目前所考察的计算模型, 可谓极其精简.  
例如. 我们的 NAND-CIRC "编程语言" 仅包含单一操作 `foo = NAND(bar,blah)`.  
本章将揭示, 这些简单模型实际上与更复杂的模型完全等价.
关键发现在于: 我们可以用基础构件来实现复杂功能, 再将这些新功能作为构件去实现更高级的功能. 
这在编程语言设计领域被称为"语法糖"——因为我们并未改变底层编程模型本身, 而只是通过语法转换, 将使用了新特性的程序转译为不依赖这些特性的等效程序.

本章将提供一个"工具箱", 以用于证明许多函数都能通过NAND-CIRC程序(进而也能通过布尔电路)进行计算. 我们还将借助这个工具箱证明一个基本定理: 任意有限函数 $f:\{0,1\}^n \rightarrow \{0,1\}^m$ 都能由布尔电路实现(详见下文{{ref:thm:circuit-univ}}).  
虽然语法糖工具箱本身具有重要意义, 但{{ref:thm:circuit-univ}}也可以在不使用该工具箱的情况下直接证明. 我们将在[第4.5节](chapter_4.md#seccomputalternative)呈现这种替代证明方法. 图{{ref:fig:computefuncoverview}}概括了本章的核心结论脉络.

<!--图4.1-->
```admonish pic id="computefuncoverviewfig"
![computefuncoverviewfig](./images/chapter4/computefuncoverview.png)

{{pic}}{fig:computefuncoverview} 本章内容概要如下: 在[第4.1节](chapter_4.md#secsyntacticsugar)中, 我们将提供一套"语法糖"功能模块, 展示如何在NAND-CIRC中实现程序员自定义函数和条件语句等特性. 在[第4.3节](chapter_4.md#seclookupfunc)中, 我们将运用这些工具构建计算$LOOKUP$函数的NAND-CIRC程序(或等效的布尔电路). 由此出发, 我们将在[第4.4节](chapter_4.md#seccomputeallfunctions)中证明: NAND-CIRC程序(即布尔电路)能够计算 **所有** 有限函数. 该结论的另一种直接证明方法将在[第4.5节](chapter_4.md#seccomputalternative)中呈现. 
```

```admonish info title = "简要概述"
阅读本章, 我们希望读者能够有以下收获: 
 - 本章中, 我们将会得出第一个主要结果: **每个** 有限函数都可以被一些布尔电路计算(参见{{ref:thm:circuit-univ}} 和 {{ref:ide:finitecomputation}}).
 其有时也被称为$AND$, $OR$ 与 $NOT$ 函数的"通用性" (利用[第3章](chapter_3.md)中的等价, 这也是$NAND$的"通用性")
 - 尽管{{ref:thm:circuit-univ}}是一项重要结论, 但其证明过程实际上并不复杂. [第4.5节](chapter_4.md#seccomputalternative)将给出该结论的一个相对简洁的直接证明. 
 不过在[第4.1节](chapter_4.md#secsyntacticsugar)和[第4.3节](chapter_4.md#seclookupfunc)中, 我们采用了"语法糖"(参见{{ref:ide:synsugar}})这一概念来推导该结论. 对于编程语言的理论与实践而言, 这都是一个至关重要的概念. 
 "语法糖"的核心思想在于: 我们可以通过基础组件实现高级功能, 从而扩展编程语言的表现力. 例如, 基于[第3章](chapter_3.md)介绍的AON-CIRC和NAND-CIRC编程语言, 我们可以通过扩展实现用户自定义函数(如`def Foo(...)`)、条件语句(如`if blah ...`)等高级特性. 
 一旦掌握了这些扩展功能, 我们就不难证明: 通过获取任意函数的真值表(即所有输入输出对应表), 可以据此创建出能将每个输入映射至对应输出的AON-CIRC或NAND-CIRC程序. 
 - 本章中我们还将首次接触 **定量分析** 的概念. 虽然{{ref:thm:circuit-univ}}定理指出每个函数都能通过某个电路实现, 但该电路所需逻辑门的数量可能呈指数级增长. 
 (此处使用的"指数级"并非口语中泛指的"非常巨大", 而是精确的数学概念——当然这个数学概念恰好也意味着规模极其庞大. )
 我们发现, **某些函数** (例如, 整数加法和乘法) 事实上可以用更少的门电路计算. 我们将在[第5章](./chapter_5.md)与接下来的章节中更加深入探讨这种"门电路复杂度".
```

## 4.1 语法糖的一些例子  { #secsyntacticsugar }

现在我们将展示若干"语法糖"转换的实例, 这些转换可用于构建直线式程序或电路. 我们主要从计算模型的**直线式编程语言**视角出发, 并具体以NAND-CIRC编程语言为例进行说明(以便更清晰地阐述概念). 
这种视角的便利之处在于, 我们介绍的多数语法糖转换最容易理解的方式, 就是将其视为对程序源代码进行"查找替换"操作. 根据{{ref:thm:equivalencemodels}}定理, 我们得到的所有结论同样适用于电路模型——无论是使用NAND门的电路, 还是使用AND、OR及NOT门构成的布尔电路. 
虽然详细列举这类语法糖转换的实例可能略显枯燥, 但我们之所以这样做, 主要基于两个原因: 

1. 这可以让你确信, 尽管布尔电路或NAND-CIRC编程语言等简单模型看似基础且存在局限性, 但它们实际上具有强大的表达能力. 

2. 于是你就可以意识到, 选择学习计算理论课程而非编译原理课程是多么幸运... `:)`

### 4.1.1 用户定义过程

几乎所有编程语言都具备一个核心功能: 定义并执行**过程**或**子程序**的能力(在某些语言中常称为 **函数** , 但为避免与程序计算的函数混淆, 我们更倾向于使用*过程*这一名称). NAND-CIRC编程语言本身并未内置这种机制, 但我们可以通过沿用已久的"复制粘贴"技巧实现相同效果. 具体来说, 我们可以将定义过程的代码: 

```python
def Proc(a,b):
    proc_code
    return c
some_code
f = Proc(d,e)
some_more_code
```

替换为以下形式, 其中直接"粘贴"`Proc`过程的代码: 

```python
some_code
proc_code'
some_more_code
```

其中`proc_code'`是通过将`Proc`代码中所有`a`替换为`d`、`b`替换为`e`、`c`替换为`f`而得到的. 在执行此操作时, 我们需要确保`proc_code'`中出现的所有其他变量不会与其他变量产生冲突——这总是可以通过将变量重命名为之前未使用过的新名称来实现. 
由上述推理, 我们可以得到以下定理:

```admonish quote title=""
{{thmc}}{thm:functionsynsugar}[语法糖: 过程定义]

令 NAND-CIRC-PROC 为 NAND_CIRC 编程语言的一个拓展, 其具有定义过程的语法.
则对于每个 NAND-CIRC-PROC 程序 $P$, 存在一个标准的 (即"无糖") NAND-CIRC 程序 $P'$ 与 $P$ 计算相同的函数.
```

```admonish info
{{remc}}{rem:norecursion}[无递归过程]

NAND-CIRC-PROC只允许 **无递归** 过程. 事实上, 过程`Proc`的代码无法调用`Proc`, 而只能使用在其之前定义的过程.
如果没有这样的限制, 上述的"搜索并替换"的过程可能永远无法结束, 而{{ref:thm:functionsynsugar}}随之不成立.
```

{{ref:thm:functionsynsugar}} 可通过上述转换方法证明, 但由于形式化证明过程较为冗长繁琐, 此处予以省略. 

~~~admonish example
{{exac}}{exa:majcircnand}[使用语法糖通过NAND计算多数函数]
过程机制让我们能够更清晰简洁地表达NAND-CIRC程序. 例如, 由于我们可以通过NAND实现AND、OR和NOT运算, 因此可以通过以下方式计算 **多数** 函数: 

```python
def NOT(a):
    return NAND(a,a)
def AND(a,b):
    temp = NAND(a,b)
    return NOT(temp)
def OR(a,b):
    temp1 = NOT(a)
    temp2 = NOT(b)
    return NAND(temp1,temp2)

def MAJ(a,b,c):
    and1 = AND(a,b)
    and2 = AND(a,c)
    and3 = AND(b,c)
    or1 = OR(and1,and2)
    return OR(or1,and3)

print(MAJ(0,1,1))
# 1
```

{{ref:fig:progcircmaj}} 展示了通过"展开"此程序(将其中的过程调用替换为具体定义)后得到的"无糖"版NAND-CIRC程序及其对应电路. 
~~~

```admonish bigidea
{{idec}}{ide:synsugar}
一旦我们证明某个计算模型 $X$ 与具有特性 $Y$ 的模型等价, 那么在论证函数 $f$ 可由 $X$ 计算时, 即可直接假定我们拥有特性 $Y$. 
```

```admonish pic id="progcircmajfig"
![progcircmajfig](./images/chapter4/progcircmaj.png)

{{pic}}{fig:progcircmaj} 通过展开多数函数程序({{ref:exa:majcircnand}})中的过程定义后得到的标准(即"无糖")NAND-CIRC程序, 右侧为其对应电路. 需注意, 这并非实现多数函数最高效的NAND电路/程序: 通过简化某些步骤(例如当门电路 $u$ 计算 $NAND(v,v)$ 后, 门电路 $w$ 又计算 $NAND(u,u)$ 的情况, 图中绿色虚线箭头标示处), 我们可以减少逻辑门的使用数量. 
```

```admonish info
{{remc}}{rem:countinglines}[计算行数]

尽管我们可以通过使用语法糖来以一种更易读的方式 **表示** NAND-CIRC程序, 我们并没有改变语言本身的定义.
因此, 不管什么时候, 当我们说某个函数 $f$ 有一个 $s$ 行的NAND-CIRC程序时, 我们指的总是一个标准"无糖"NAND-CIRC程序, 其中所有的语法糖都已经被展开了.
例如, {{ref:exa:majcircnand}}的程序是计算 $MAJ$ 的一个 $12$ 行程序, 尽管使用NAND-CIRC-PROC时其可以用更少的代码行数写出.
```

### 4.1.2 由Python证明 (选读) { #functionsynsugarthmpython }

我们可以编写一个Python程序来实现{{ref:thm:functionsynsugar}}的证明. 该程序将接受包含过程定义的NAND-CIRC-PROC程序$P$, 通过简单的"查找替换"操作将其转换为标准的(即"无糖")NAND-CIRC程序$P'$, 使得$P'$在不使用任何过程的情况下计算与$P$相同的函数. 

核心思路很简单: 如果程序$P$包含一个带有两个参数`x`和`y`的过程`Proc`的定义, 那么每当遇到形如`foo = Proc(bar,blah)`的语句时, 我们可以用以下内容替换该行: 

1. 过程`Proc`的主体代码(将所有出现的`x`和`y`分别替换为`bar`和`blah`)

2. 一行`foo = exp`, 其中`exp`是过程`Proc`定义中`return`语句后面的表达式

为使转换更加健壮, 我们可以为`Proc`使用的内部变量添加前缀, 以确保它们不会与$P$中的变量冲突; 为简化起见, 我们在下面的代码中暂不考虑这个问题, 但实际实现时可以轻松添加此功能. 

以下Python函数`desugar`的代码实现了这样的转换: 

```admonish example
{{exac}}{exa:desugarcode}[将NAND-CIRC-PROC程序转化为标准无糖NAND-CIRC程序的Python代码]
~~~python
def desugar(code, func_name, func_args,func_body):
    """
    将所有具有形式
       foo = func_name(func_args) 
    用以下代码替换
       func_body[x->a,y->b]
       foo = [result returned in func_body]    
    """
    # 使用Python的正则表达式来简化代码
    # 参见 https://docs.python.org/3/library/re.html 和本书第九章

    # 捕获由逗号分割的参数列表的正则表达式
    arglist = ",".join([r"([a-zA-Z0-9\_\[\]]+)" for i in range(len(func_args))])
    # 捕获具有下列形式的正在表达式
    # "variable = func_name(arguments)"
    regexp = fr'([a-zA-Z0-9\_\[\]]+)\s*=\s*{func_name}\({arglist}\)\s*$'#$
    while True:
        m = re.search(regexp, code, re.MULTILINE)
        if not m: break
        newcode = func_body 
        # 将函数的参数用函数调用时传入的变量替换
        for i in range(len(func_args)): 
            newcode = newcode.replace(func_args[i], m.group(i+2))
        # 将新代码插入
        newcode = newcode.replace('return', m.group(1) + " = ")
        code = code[:m.start()] + newcode + code[m.end()+1:]
    return code
~~~
{{ref:fig:progcircmaj}} 展示了, 对{{ref:exa:majcircnand}}中使用语法糖计算的多数函数程序, 将`desugar`函数应用于其上得到的结果. 
具体来说, 我们首先应用`desugar`移除OR函数的使用, 然后再次应用以移除AND函数的使用, 最后第三次应用以移除NOT函数的使用. 
```

```admonish info
{{remc}}{rem:parsingdeg}[解析函数定义 (选读)]

{{ref:exa:desugarcode}}中的`desugar`函数假定过程定义已被拆分为名称、参数和主体部分. 
虽然精确描述如何扫描定义, 并将其拆分为这些组件, 对我们的目的并不关键. 但如果感兴趣, 可以通过以下Python代码实现这一拆分过程: 

~~~python
def parse_func(code):
    """将一个函数定义解析为名称, 参数列表与函数体"""
    lines = [l.strip() for l in code.split('\n')]
    regexp = r'def\s+([a-zA-Z\_0-9]+)\(([\sa-zA-Z0-9\_,]+)\)\s*:\s*'
    m = re.match(regexp,lines[0])
    return m.group(1), m.group(2).split(','), '\n'.join(lines[1:])
~~~
```



### 4.1.3 条件语句 {#ifstatementsec }

NAND-CIRC语言中另一个严重缺失的特性是条件语句(例如许多编程语言中常见的`if`/`then`结构). 
不过, 通过运用过程机制, 我们可以实现一种替代的条件判断结构. 
首先我们需要计算函数 $IF:\{0,1\}^3 \rightarrow \{0,1\}$, 该函数满足: 当 $a=1$ 时输出 $b$, 当 $a=0$ 时输出 $c$. 

```admonish pause title="思考时刻"
在继续阅读前, 请尝试思考如何用$NAND$门实现$IF$函数. 完成这一步后, 再思考如何利用它来模拟`if`/`then`类型的结构. 
```

如{{ref:pro:mux}}所示, $IF$函数可以通过NAND门按如下方式实现: 

```python
def IF(cond,a,b):
    notcond = NAND(cond,cond)
    temp = NAND(b,notcond)
    temp1 = NAND(a,cond)
    return NAND(temp,temp1)
```

$IF$又被称为 **多路** 函数, 因为$cond$可以被视作一个控制输出与$a$还是$b$相连的开关.
只要我们由计算$IF$函数的过程, 就可以在NAND中实现条件语句.
其思路为将具有以下形式的代码

```python
if (condition):  assign blah to variable foo
```

替换为具有以下形式的代码

```python
foo   = IF(condition, blah, foo)
```

其在`condition`等于$0$时将`foo`赋值为旧值, 否则将`foo`赋值为`blah`的值.
更一般地, 我们将如下形式的代码

```python
if (cond):
    a = ...
    b = ...
    c = ...
```

替换为如下形式的代码

```python
temp_a = ...
temp_b = ...
temp_c = ...
a = IF(cond,temp_a,a)
b = IF(cond,temp_b,b)
c = IF(cond,temp_c,c)
```

通过运用此类转换方法, 我们可以证明以下定理. 尽管其完整形式化证明(启发性有限)在此从略, 但读者可参阅[第4.1.2节](chapter_4.md#functionsynsugarthmpython)获取相关证明思路的提示. 

```admonish quote title=""
{{thmc}}{thm:conditionalsugar}[语法糖: 条件语句]
设NAND-CIRC-IF为在NAND-CIRC编程语言基础上扩展了`if`/`then`/`else`语句的语言版本, 允许代码根据变量取值是否为$0$或$1$来条件执行.   
则对于任意NAND-CIRC-IF程序$P$, 都存在一个标准的(即"无糖")NAND-CIRC程序$P'$能计算与$P$完全相同的函数. 
```

## 4.2 拓展样例: 加法与乘法(选读) { #addexample }

使用"语法糖", 我们能够写出以下的整数加法函数:

```python
# 将两个n为整数相加
# 为了简便, 使用最低有效位优先表示法
def ADD(A,B):
    Result = [0]*(n+1)
    Carry  = [0]*(n+1)
    Carry[0] = zero(A[0])
    for i in range(n):
        Result[i] = XOR(Carry[i],XOR(A[i],B[i]))
        Carry[i+1] = MAJ(Carry[i],A[i],B[i])
    Result[n] = Carry[n]
    return Result

ADD([1,1,1,0,0],[1,0,0,0,0]);;
# [0, 0, 0, 1, 0, 0]
```

其中`zero`是常数零函数, `MAJ`和`XOR`分别对应多数函数与异或函数. 虽然我们为方便起见使用了Python语法, 但此例中$n$是某个 **固定整数** , 因此对每个这样的$n$而言, `ADD`都是一个接收$2n$位输入并输出$n+1$位的有限函数. 特别地, 对于每个$n$, 我们只需将代码重复$n$次(将`i`的值依次替换为$0,1,2,\ldots,n-1$)即可消除`for i in range(n)`循环结构. 通过展开所有特性, 对每个$n$的取值, 我们都能将上述程序转换为标准的(无糖)NAND-CIRC程序. {{ref:fig:add2bitnumbers}}展示了$n=2$时的转换结果. 

```admonish pic id="add2bitnumbersfig"
![add2bitnumbersfig](./images/chapter4/add2bitnumbers.png)

{{pic}}{fig:add2bitnumbers} 通过"展开"所有语法糖功能得到的用于两个二进制数相加的NAND-CIRC程序及对应NAND电路. 该程序/电路包含43行代码/逻辑门, 但这远非最优实现. 实际上只需使用$9n$个NAND门即可完成$n$位二进制数的加法运算, 具体实现方法参见{{ref:pro:halffulladder}}. 
```

通过仔细分析上述程序并统计逻辑门数量, 我们可以证明以下定理(另见{{ref:fig:addnumoflines}}): 

```admonish quote title=""
{{thmc}}{thm:addition}[使用NAND-CIRC程序实现加法运算]
对于任意$n\in \N$, 令$ADD_n:\{0,1\}^{2n}\rightarrow \{0,1\}^{n+1}$为如下函数: 给定$x,x'\in \{0,1\}^n$, 计算$x$和$x'$所表示数值之和的二进制表示. 则存在常数$c \leq 30$, 使得对每个$n$, 都存在一个最多包含$cn$行代码的NAND-CIRC程序可计算$ADD_n$. {{footnote: $c$的值可优化至$9$, 具体参见{{ref:pro:halffulladder}}. }}
```

```admonish pic id="addnumoflinesfig"
![addnumoflinesfig](./images/chapter4/addnumberoflines.png)

{{pic}}{fig:addnumoflines} 我们实现的两位$n$比特二进制数相加的NAND-CIRC程序行数随$n$的变化关系($n$取值1到100). 虽然这不是该任务的最优实现, 但关键之处在于其复杂度呈现$O(n)$的线性特征. 
```

只要有了加法, 我们就可以使用小学乘法算法来获得乘法, 从而得到以下定义:

```admonish quote title=""
{{thmc}}{thm:theorem}[使用NAND-CIRC程序实现乘法运算]
对于任意$n$, 设$MULT_n:\{0,1\}^{2n}\rightarrow \{0,1\}^{2n}$为这样的函数: 给定$x,x'\in \{0,1\}^n$, 计算$x$和$x'$所表示数值之积的二进制表示. 则存在常数$c$, 使得对每个$n$, 都存在一个最多包含$cn^2$行代码的NAND-CIRC程序可计算函数$MULT_n$. 
```

我们在此省略证明过程, 不过在{{ref:pro:multiplication}}中, 我们将要求您以(用您熟悉的编程语言编写的)程序形式提供一份"构造性证明": 该程序以数字$n$作为输入, 输出一个最多包含$1000n^2$行代码的NAND-CIRC程序, 用于计算$MULT_n$函数. 
实际上, 利用Karatsuba算法可以证明: 存在一个包含$O(n^{\log_2 3})$行代码的NAND-CIRC程序能够计算$MULT_n$函数(若采用更优算法, 还能实现更进一步的渐进性优化). 

## 4.3 LOOKUP函数 { #seclookupfunc }

$LOOKUP$ 函数将在本章及后续章节中扮演重要角色.
其定义如下:

```admonish quote title="" 
{{defc}}{def:lookup}[查找函数]
对于每个 $k$, $k$阶 **查找** 函数 $LOOKUP_k: \{0,1\}^{2^k+k}\rightarrow \{0,1\}$ 定义如下:
对于每个 $x\in\{0,1\}^{2^k}$ 和 $i\in \{0,1\}^k$,
$$
LOOKUP_k(x,i)=x_i
$$
其中 $x_i$ 表示 $x$ 的第 $i^{th}$ 个条目, 使用二进制表示将 $i$ 识别为 $\{0,\ldots,2^k - 1 \}$ 中的一个数字.
```

```admonish pic id="lookupfig"
![lookupfig](./images/chapter4/lookupfunc.png)

{{pic}}{fig:lookup} $LOOKUP_k$ 函数接受一个输入在 $\{0,1\}^{2^k+k}$ 中, 我们将其表示为 $x,i$ (其中 $x\in \{0,1\}^{2^k}$ 和 $i \in \{0,1\}^k$). 输出是 $x_i$: $x$ 的第 $i$ 个坐标, 其中我们使用二进制表示将 $i$ 识别为 $[k]$ 中的一个数字. 在上面的例子中 $x\in \{0,1\}^{16}$ 和 $i\in \{0,1\}^4$. 由于 $i=0110$ 是数字 $6$ 的二进制表示, 在这种情况下 $LOOKUP_4(x,i)$ 的输出是 $x_6 = 1$.
```

对于 LOOKUP 函数的图示参见 {{ref:fig:lookup}}.
事实证明, 对于每个 $k$, 我们可以使用 NAND-CIRC 程序计算 $LOOKUP_k$:

```admonish quote title=""
{{thmc}}{thm:lookup}[查找函数]
对于每个 $k>0$, 存在一个 NAND-CIRC 程序计算函数 $LOOKUP_k: \{0,1\}^{2^k+k}\rightarrow \{0,1\}$. 此外, 该程序的行数最多为 $4\cdot 2^k$.
```

{{ref:thm:lookup}} 的一个直接推论是, 对于每个 $k>0$, $LOOKUP_k$ 可以由一个布尔电路(使用 AND,OR 和 NOT 门)计算, 其门数最多为 $8 \cdot 2^k$.

### 4.3.1 为$LOOKUP$构造一个NAND-CIRC程序

我们通过归纳法证明{{ref:thm:lookup}}.

对于情况 $k=1$, $LOOKUP_1$ 将 $(x_0,x_1,i) \in \{0,1\}^3$ 映射到 $x_i$. 
换句话说, 如果 $i=0$ 则它输出 $x_0$ , 否则它输出 $x_1$ , 
(在变量重新排序后)这与 [第4.1.3节](chapter_4.md#ifstatementsec) 中提出的 $IF$ 函数相同, 
该函数可以用一个4行 NAND-CIRC 程序计算.

作为一般情况的热身, 让我们考虑 $k=2$ 的情况. 给定 $LOOKUP_2$ 的输入 $x=(x_0,x_1,x_2,x_3)$ 和索引 $i=(i_0,i_1)$, 如果索引的最高有效位 $i_0$ 是 $0$ , 那么 $LOOKUP_2(x,i)$ 将等于 $x_0$ 如果 $i_1=0$ , 并等于 $x_1$ 如果 $i_1=1$ . 类似地, 如果最高有效位 $i_0$ 是 $1$ , 那么 $LOOKUP_2(x,i)$ 将等于 $x_2$ 如果 $i_1=0$ , 并将等于 $x_3$ 如果 $i_1=1$ . 另一种说法是, 我们可以将 $LOOKUP_2$ 写成如下形式:

```python
def LOOKUP2(X[0],X[1],X[2],X[3],i[0],i[1]):
    if i[0]==1:
        return LOOKUP1(X[2],X[3],i[1])
    else:
        return LOOKUP1(X[0],X[1],i[1])
```

换言之

```python
def LOOKUP2(X[0],X[1],X[2],X[3],i[0],i[1]):
    a = LOOKUP1(X[2],X[3],i[1])
    b = LOOKUP1(X[0],X[1],i[1])
    return IF( i[0],a,b)
```

更一般地, 如以下引理所示, 我们可以使用两次 $LOOKUP_{k-1}$ 调用和一次 $IF$ 调用来计算 $LOOKUP_k$ :

```admonish quote title=""
{{lemc}}{lem:lookup-rec}[查找递归]
对于每个 $k \geq 2$, $LOOKUP_k(x_0,\ldots,x_{2^k-1},i_0,\ldots,i_{k-1})$ 等于

$$
\begin{aligned}
IF (&i_0, LOOKUP_{k-1}(x_{2^{k-1}},\ldots,x_{2^k-1},i_1,\ldots,i_{k-1}), \\ 
    &LOOKUP_{k-1}(x_0,\ldots,x_{2^{k-1}-1},i_1,\ldots,i_{k-1}))
\end{aligned}
$$
```

```admonish proof collapsible=true, title = "对{{ref:lem:lookup-rec}}的证明"
如果 $i$ 的最高有效位 $i_{0}$ 为零, 那么索引 $i$ 在 $\{0,\ldots,2^{k-1}-1\}$ 中, 因此我们可以在 $x$ 的"前半部分"执行查找, 并且 $LOOKUP_k(x,i)$ 的结果将与 $a=LOOKUP_{k-1}(x_0,\ldots,x_{2^{k-1}-1},i_1,\ldots,i_{k-1})$ 相同. 另一方面, 如果这个最高有效位 $i_{0}$ 等于 $1$ , 那么索引在 $\{2^{k-1},\ldots,2^k-1\}$ 中, 在这种情况下, $LOOKUP_k(x,i)$ 的结果与 $b=LOOKUP_{k-1}(x_{2^{k-1}},\ldots,x_{2^k-1},i_1,\ldots,i_{k-1})$ 相同. 因此, 我们可以通过首先计算 $a$ 和 $b$ , 然后输出 $IF(i_0,b,a)$ 来计算 $LOOKUP_k(x,i)$ .
```

__基于 {{ref:lem:lookup-rec}} 的 {{ref:thm:lookup}} 证明.__ 既然我们已经证明 {{ref:lem:lookup-rec}}, 我们就可以完成 {{ref:thm:lookup}} 的证明.
我们将通过对$k$归纳证明, 存在一个最多 $4\cdot (2^k-1)$ 行的 NAND-CIRC 程序用于计算 $LOOKUP_k$.
对于 $k=1$, 这由我们之前见过的用于 $IF$ 的四行程序得出.
对于 $k>1$, 我们使用以下伪代码来计算:

```python
a = LOOKUP_(k-1)(X[0],...,X[2^(k-1)-1],i[1],...,i[k-1])
b = LOOKUP_(k-1)(X[2^(k-1)],...,X[2^(k-1)],i[1],...,i[k-1])
return IF(i[0],b,a)
```

如果我们令 $L(k)$ 表示 $LOOKUP_k$ 所需的行数, 那么上述伪代码表明
$$
L(k) \leq 2L(k-1)+4 \;. {{numeq}}{eq:induction-lookup}
$$

由归纳假设, $L(k-1) \leq 4(2^{k-1}-1)$, 我们有
$L(k) \leq 2\cdot 4 (2^{k-1}-1) + 4 = 4(2^k - 1)$, 这正是我们想要证明的.

对于我们实现的 $LOOKUP_k$ 的实际行数图, 参见 {{ref:fig:lookuplines}}.

```admonish pic id="lookuplinesfig"
![lookuplinesfig](./images/chapter4/lookup_numlines.png)

{{pic}}{fig:lookuplines} 我们实现的 `LOOKUP_k` 函数的行数关于 $k$ (即索引的长度) 的函数. 我们实现中的行数大约为 $3 \cdot 2^k$.
```

## 4.4 **通用** 函数计算 { #seccomputeallfunctions }

At this point we know the following facts about NAND-CIRC programs (and so equivalently about Boolean circuits and our other equivalent models):

1. They can compute at least some non-trivial functions.

2. Coming up with NAND-CIRC programs for various functions is a very tedious task.

Thus I would not blame the reader if they were not particularly looking forward to a long sequence of examples of functions that can be computed by NAND-CIRC programs.
However, it turns out we are not going to need this, as we can show in one fell swoop that NAND-CIRC programs can compute _every_ finite function:

```admonish quote title=""
{{thmc}}{thm:NAND-univ}[Universality of NAND]
There exists some constant $c>0$ such that for every $n,m>0$ and function $f: \{0,1\}^n\rightarrow \{0,1\}^m$, there is a NAND-CIRC program  with at most $c \cdot m 2^n$ lines that computes the function $f$ .
```

By {{ref:thm:equivalencemodels}},  the models of NAND circuits, NAND-CIRC programs, AON-CIRC programs, and Boolean circuits, are all equivalent to one another, and hence {{ref:thm:NAND-univ}} holds for all these models.
In particular, the following theorem is equivalent to {{ref:thm:NAND-univ}}:


```admonish quote title=""
{{thmc}}{thm:circuit-univ}[Universality of Boolean circuits]
There exists some constant $c>0$ such that for every $n,m>0$ and function $f: \{0,1\}^n\rightarrow \{0,1\}^m$, there is a Boolean circuit with at most $c \cdot m 2^n$ gates that computes the function $f$ .
```

```admonish bigidea
{{idec}}{ide:finitecomputation}
_Every_ finite function can be computed by a large enough Boolean circuit.
```






_Improved bounds._ Though it will not be of great importance to us, it is possible to improve on the proof of
{{ref:thm:NAND-univ}}  and shave an extra factor of $n$, as well as optimize the constant $c$, and so prove that
for every $\epsilon>0$, $m\in \N$ and sufficiently large $n$, if $f:\{0,1\}^n \rightarrow \{0,1\}^m$ then $f$ can be computed by a NAND circuit of at most
$(1+\epsilon)\tfrac{m\cdot 2^n}{n}$ gates.
The proof of this result is beyond the scope of this book, but we do discuss how to obtain a bound of the form $O(\tfrac{m \cdot 2^n}{n})$ in [第4.4.2节](chapter_4.md#tight-upper-bound); see also the biographical notes.





### 4.4.1 NAND通用性的证明

To prove {{ref:thm:NAND-univ}}, we need to give a NAND circuit, or equivalently a NAND-CIRC program,  for _every_ possible function.
We will restrict our attention to the case of Boolean functions (i.e., $m=1$).
{{ref:pro:mult-bit}} asks you  to extend the proof for all values of $m$.
A function $F: \{0,1\}^n\rightarrow \{0,1\}$ can be specified by a table of its values for each one of the $2^n$ inputs.
For example, the table below describes one particular function $G: \{0,1\}^4 \rightarrow \{0,1\}$:{{footnote:In case you are curious, this is the function on input $i\in \{0,1\}^4$ (which we interpret as a number in $[16]$), that outputs the $i$-th digit of $\pi$ in the binary basis.}}


| Input ($x$) | Output ($G(x)$) |
|:------------|:----------------|
| $0000$      | 1               |
| $0001$      | 1               |
| $0010$      | 0               |
| $0011$      | 0               |
| $0100$      | 1               |
| $0101$      | 0               |
| $0110$      | 0               |
| $0111$      | 1               |
| $1000$      | 0               |
| $1001$      | 0               |
| $1010$      | 0               |
| $1011$      | 0               |
| $1100$      | 1               |
| $1101$      | 1               |
| $1110$      | 1               |
| $1111$      | 1               |


Table: An example of a function $G:\{0,1\}^4 \rightarrow \{0,1\}$. 




For every $x\in \{0,1\}^4$, $G(x)=LOOKUP_4(1100100100001111,x)$, and so the following is NAND-CIRC "pseudocode"  to compute $G$ using syntactic sugar for the `LOOKUP_4` procedure.


```python
G0000 = 1
G1000 = 1
G0100 = 0
...
G0111 = 1
G1111 = 1
Y[0] = LOOKUP_4(G0000,G1000,...,G1111,
                X[0],X[1],X[2],X[3])
```


We can translate this pseudocode into an actual NAND-CIRC program by adding three lines to define variables `zero` and `one` that are initialized to $0$ and $1$ respectively,
and then replacing a statement such as `Gxxx = 0` with `Gxxx = NAND(one,one)` and a statement such as `Gxxx = 1` with `Gxxx = NAND(zero,zero)`.
The call to `LOOKUP_4` will be replaced by the NAND-CIRC program that computes $LOOKUP_4$, plugging in the appropriate inputs.

There was nothing about the above reasoning that was particular to the function $G$ above.
Given _every_ function $F: \{0,1\}^n \rightarrow \{0,1\}$, we can write a NAND-CIRC program that does the following:

1. Initialize $2^n$ variables of the form `F00...0` till `F11...1` so that for every $z\in\{0,1\}^n$,  the variable corresponding to $z$ is assigned the value $F(z)$.

2. Compute $LOOKUP_n$ on the $2^n$ variables initialized in the previous step, with the index variable being the input variables `X[`$0$ `]`,...,`X[`$n-1$ `]`. That is, just like in the pseudocode for `G` above, we use `Y[0] = LOOKUP(F00..00,...,F11..1,X[0],..,X[`$n-1$`])`

The total number of lines in the resulting program is $3+2^n$ lines for initializing the variables plus the $4\cdot 2^n$ lines that we pay for computing $LOOKUP_n$.
This completes the proof of {{ref:thm:NAND-univ}}.



```admonish info
{{remc}}{rem:discusscomputation}[Result in perspective]
While {{ref:thm:NAND-univ}} seems striking at first, in retrospect, it is perhaps not that surprising that every finite function can be computed with a NAND-CIRC program. After all, a finite function $F: \{0,1\}^n \rightarrow \{0,1\}^m$ can be represented by simply the list of its outputs for each one of the $2^n$ input values.
So it makes sense that we could write a NAND-CIRC program of similar size to compute it.
What is more interesting is that _some_ functions, such as addition and multiplication,  have a much more efficient representation: one that only requires $O(n^2)$ or even fewer lines.
```


### 4.4.2 Improving by a factor of $n$ (optional) {#tight-upper-bound}

By being a little more careful, we can improve the bound of {{ref:thm:NAND-univ}} and show that every function $F:\{0,1\}^n \rightarrow \{0,1\}^m$ can be computed by a NAND-CIRC program of at most $O(m 2^n/n)$ lines.
In other words, we can prove  the following improved version:

```admonish quote title=""
{{thmc}}{thm:NAND-univ-improved}[Universality of NAND circuits, improved bound]
There exists a constant $c>0$ such that for every $n,m>0$ and function $f: \{0,1\}^n\rightarrow \{0,1\}^m$, there is a NAND-CIRC program  with at most $c \cdot m 2^n / n$ lines that computes the function $f$.{{footnote:The constant $c$ in this theorem is at most $10$ and in fact can be arbitrarily close to $1$, see [杂记](#computeeveryfunctionbibnotes).}}
```


```admonish proof collapsible=true, title = "对{{ref:thm:NAND-univ-improved}}的证明"
As before, it is enough to prove the case that $m=1$.
Hence we let $f:\{0,1\}^n \rightarrow \{0,1\}$, and our goal is to prove that there exists a NAND-CIRC program of $O(2^n/n)$ lines (or equivalently a Boolean circuit of $O(2^n/n)$ gates) that computes $f$.

We let $k= \log(n-2\log n)$ (the reasoning behind this choice will become clear later on).
We define the function $g:\{0,1\}^k \rightarrow \{0,1\}^{2^{n-k}}$ as follows:
$$
g(a) = f(a0^{n-k})f(a0^{n-k-1}1) \cdots f(a1^{n-k}) \;.
$$
In other words, if we use the usual binary representation to identify the numbers $\{0,\ldots, 2^{n-k}-1 \}$ with the strings $\{0,1\}^{n-k}$, then for every $a\in \{0,1\}^k$ and $b\in \{0,1\}^{n-k}$
$$
g(a)_b = f(ab) \;. \label{eqcomputefusinggeffcircuit}
$$

[eqcomputefusinggeffcircuit](){.eqref} means that for every $x\in \{0,1\}^n$, if we write
$x=ab$ with $a\in \{0,1\}^k$ and $b\in \{0,1\}^{n-k}$ then we can compute $f(x)$ by
first computing the string  $T=g(a)$  of length $2^{n-k}$, and then computing $LOOKUP_{n-k}(T\;,\; b)$ to retrieve the
element of $T$ at the position corresponding to $b$ (see {{ref:fig:efficient_circuit_allfunc}}).
The cost to compute the $LOOKUP_{n-k}$ is $O(2^{n-k})$ lines/gates and the cost in NAND-CIRC lines (or Boolean gates) to compute  $f$ is at most
$$
cost(g) + O(2^{n-k}) \;, \label{eqcostcomputefusingg}
$$
where $cost(g)$ is the number of operations (i.e., lines of NAND-CIRC programs or gates in a circuit) needed to compute $g$.

To complete the proof we need to give a  bound on  $cost(g)$.
Since $g$ is a function mapping $\{0,1\}^k$ to $\{0,1\}^{2^{n-k}}$, we can also think of it as a
collection of $2^{n-k}$ functions $g_0,\ldots, g_{2^{n-k}-1}: \{0,1\}^k \rightarrow \{0,1\}$, where
$g_i(x) = g(a)_i$ for every $a\in \{0,1\}^k$ and $i\in [2^{n-k}]$. (That is, $g_i(a)$ is the $i$-th bit of $g(a)$.)
Naively, we could use  {{ref:thm:NAND-univ}}  to compute each $g_i$ in $O(2^k)$ lines, but then
the total cost is $O(2^{n-k} \cdot 2^k) = O(2^n)$ which does not save us anything.
However, the crucial observation is that there are only $2^{2^k}$ _distinct functions_ mapping
$\{0,1\}^k$ to $\{0,1\}$.
For example, if $g_{17}$ is an identical function to $g_{67}$ that means that if we already computed $g_{17}(a)$ then we can compute $g_{67}(a)$ using only a constant number of operations: simply copy the same value!
In general, if you have a collection of $N$ functions $g_0,\ldots,g_{N-1}$ mapping $\{0,1\}^k$ to $\{0,1\}$, of which at most $S$ are distinct then for every value $a\in \{0,1\}^k$ we can compute the $N$ values $g_0(a),\ldots,g_{N-1}(a)$ using at most $O(S\cdot 2^k + N)$ operations (see {{ref:fig:computemanyfunctions}}).

In our case, because there are at most $2^{2^k}$ distinct functions mapping $\{0,1\}^k$ to $\{0,1\}$, we can compute the function $g$ (and hence by [eqcomputefusinggeffcircuit](){.eqref}  also $f$) using at most  
$$O(2^{2^k} \cdot 2^k + 2^{n-k}) \label{eqboundoncostg}$$
operations.
Now all that is left is to plug  into [eqboundoncostg](){.eqref} our choice of $k = \log (n-2\log n)$.
By definition, $2^k = n-2\log n$, which means that   [eqboundoncostg](){.eqref} can be bounded
$$
O\left(2^{n-2\log n} \cdot (n-2\log n) +  2^{n-\log(n-2\log n)}\right) \leq
$$

$$
O\left(\tfrac{2^n}{n^2} \cdot n + \tfrac{2^n}{n-2\log n} \right)
\leq
O\left(\tfrac{2^n}{n}  + \tfrac{2^n}{0.5n} \right)  = O\left( \tfrac{2^n}{n} \right)
$$
which is what we wanted to prove. (We used above the fact that $n - 2\log n \geq 0.5 \log n$ for sufficiently large $n$.)
```

```admonish pic id="computemanyfunctionsfig"
![computemanyfunctionsfig](./images/chapter4/computemanyfunctions.png)

{{pic}}{fig:computemanyfunctions} If $g_0,\ldots, g_{N-1}$ is a collection of functions each mapping $\{0,1\}^k$ to $\{0,1\}$ such that at most $S$ of them are distinct then for every $a\in \{0,1\}^k$, we can compute all the values $g_0(a),\ldots,g_{N-1}(a)$ using at most $O(S \cdot 2^k + N)$ operations by first computing the distinct functions and then copying the resulting values.
```

```admonish pic id="efficient_circuit_allfuncfig"
![efficient_circuit_allfuncfig](./images/chapter4/efficient_circuit_allfunc.png)

{{pic}}{fig:efficient_circuit_allfunc} We can compute $f:\{0,1\}^n \rightarrow \{0,1\}$ on input $x=ab$ where $a\in \{0,1\}^k$ and $b\in \{0,1\}^{n-k}$ by first computing the $2^{n-k}$ long string $g(a)$  that corresponds to all $f$'s values on inputs that begin with $a$, and then outputting the $b$-th coordinate of this string.
```

Using the connection between NAND-CIRC programs and Boolean circuits, an immediate corollary of  {{ref:thm:NAND-univ-improved}} is the following improvement to  {{ref:thm:circuit-univ}}:

```admonish quote title=""
{{thmc}}{thm:circuit-univ-improved}[Universality of Boolean circuits,  improved bound]
There exists some constant $c>0$ such that for every $n,m>0$ and function $f: \{0,1\}^n\rightarrow \{0,1\}^m$, there is a Boolean circuit with at most $c \cdot m 2^n / n$ gates that computes the function $f$ .
```


## 4.5 **通用** 函数计算: 一个替代的证明 {#seccomputalternative }

{{ref:thm:circuit-univ}} is a fundamental result in the theory (and practice!) of computation.
In this section, we present an alternative proof of this basic fact that Boolean circuits can compute every finite function.
This alternative proof gives a somewhat worse quantitative bound on the number of gates but it has the advantage of being simpler, working directly with circuits and avoiding the usage of all the syntactic sugar machinery.
(However, that machinery is useful in its own right, and will find other applications later on.)


```admonish quote title=""
{{thmc}}{thm:circuit-univ-alt}[Universality of Boolean circuits (alternative phrasing)]
There exists some constant $c>0$ such that for every $n,m>0$ and function $f: \{0,1\}^n\rightarrow \{0,1\}^m$, there is a Boolean circuit with at most $c \cdot m\cdot n 2^n$ gates that computes the function $f$ .
```

```admonish pic id="computeallfuncaltfig"
![computeallfuncaltfig](./images/chapter4/computeallfunctionalt.png)

{{pic}}{fig:computeallfuncalt} Given a function $f:\{0,1\}^n \rightarrow \{0,1\}$, we let $\{ x_0, x_1, \ldots, x_{N-1} \} \subseteq \{0,1\}^n$ be the set of inputs such that $f(x_i)=1$, and note that $N \leq 2^n$. We can express $f$ as the OR of $\delta_{x_i}$ for $i\in [N]$ where the function $\delta_\alpha:\{0,1\}^n \rightarrow \{0,1\}$ (for $\alpha \in \{0,1\}^n$) is defined as follows:  $\delta_\alpha(x)=1$ iff $x=\alpha$. We can compute the OR of $N$ values using $N$ two-input OR gates. Therefore if we have a circuit of size $O(n)$ to compute $\delta_\alpha$ for every $\alpha \in \{0,1\}^n$, we can compute $f$ using a circuit of size $O(n \cdot N) = O(n \cdot 2^n)$.
```



```admonish proof collapsible=true, title = "对{{ref:thm:circuit-univ-alt}}的证明思路"
The idea of the proof is illustrated in {{ref:fig:computeallfuncalt}}. As before, it is enough to focus on the case that $m=1$ (the function $f$ has a single output), since we can always extend this to the case of $m>1$ by looking at the composition of $m$ circuits each computing a different output bit of the function $f$.
We start by showing that for every $\alpha \in \{0,1\}^n$, there is an $O(n)$-sized circuit that computes the function $\delta_\alpha:\{0,1\}^n \rightarrow \{0,1\}$ defined as follows: $\delta_\alpha(x)=1$ iff $x=\alpha$ (that is, $\delta_\alpha$ outputs $0$ on all inputs except the input $\alpha$). We can then write any function $f:\{0,1\}^n \rightarrow \{0,1\}$ as the OR of at most $2^n$ functions $\delta_\alpha$ for the $\alpha$'s on which $f(\alpha)=1$.
```

```admonish proof collapsible=true, title = "对{{ref:thm:circuit-univ-alt}}的证明"
We prove the theorem for the case $m=1$. The result can be extended for $m>1$ as before (see also {{ref:pro:mult-bit}}).
Let $f:\{0,1\}^n \rightarrow \{0,1\}$.
We will prove that there is an $O(n\cdot 2^n)$-sized Boolean circuit to compute $f$ in the following steps:

1. We show that for every $\alpha\in \{0,1\}^n$, there is an $O(n)$-sized circuit that computes the function $\delta_\alpha:\{0,1\}^n \rightarrow \{0,1\}$, where $\delta_\alpha(x)=1$ iff $x=\alpha$.

2. We then show that this implies the existence of an $O(n\cdot 2^n)$-sized circuit that computes $f$, by writing $f(x)$ as the OR of $\delta_\alpha(x)$ for all  $\alpha\in \{0,1\}^n$ such that $f(\alpha)=1$. (If  $f$ is the constant zero function and hence there is no such $\alpha$, then we can use the circuit $f(x) = x_0 \wedge \overline{x}_0$.)

We start with Step 1:

__CLAIM:__ For $\alpha \in \{0,1\}^n$, define $\delta_\alpha:\{0,1\}^n$ as follows:
$$
\delta_\alpha(x) = \begin{cases}1 & x=\alpha \\ 0 & \text{otherwise} \end{cases} \;.
$$
then there is a Boolean circuit using at most $2n$ gates that computes $\delta_\alpha$.

__PROOF OF CLAIM:__ The proof is illustrated in {{ref:fig:deltafunc}}.
As an example, consider the function $\delta_{011}:\{0,1\}^3 \rightarrow \{0,1\}$.
This function outputs $1$ on $x$ if and only if $x_0=0$, $x_1=1$ and $x_2=1$, and so we can write $\delta_{011}(x) = \overline{x_0} \wedge x_1 \wedge x_2$, which translates into a Boolean circuit with one NOT gate and two AND gates.
More generally, for every $\alpha \in \{0,1\}^n$, we can express $\delta_{\alpha}(x)$  as $(x_0 = \alpha_0) \wedge (x_1 = \alpha_1) \wedge \cdots \wedge (x_{n-1} = \alpha_{n-1})$, where if $\alpha_i=0$ we replace $x_i = \alpha_i$ with $\overline{x_i}$ and if $\alpha_i=1$ we replace $x_i=\alpha_i$ by simply $x_i$.
This yields a circuit that computes $\delta_\alpha$ using $n$ AND gates and at most $n$ NOT gates, so a total of at most $2n$ gates.

Now for every function $f:\{0,1\}^n \rightarrow \{0,1\}$, we can write

$$
f(x) = \delta_{x_0}(x) \vee \delta_{x_1}(x) \vee \cdots \vee \delta_{x_{N-1}}(x) \label{eqorofdeltafunc}
$$

where $S=\{ x_0 ,\ldots, x_{N-1}\}$ is the set of inputs on which $f$ outputs $1$.
(To see this, you can verify that the right-hand side of [eqorofdeltafunc](){.eqref} evaluates to $1$ on $x\in \{0,1\}^n$ if and only if $x$ is in the set $S$.)

Therefore we can compute $f$ using a Boolean circuit of at most $2n$ gates for each of the $N$ functions $\delta_{x_i}$ and combine that with at most $N$ OR gates, thus obtaining a circuit of at most $2n\cdot N + N$ gates.
Since $S \subseteq \{0,1\}^n$, its size $N$ is at most $2^n$ and hence the total number of gates in this circuit is $O(n\cdot 2^n)$.
```



```admonish pic id="deltafuncfig"
![deltafuncfig](./images/chapter4/deltafunc.png)

{{pic}}{fig:deltafunc} For every string $\alpha\in \{0,1\}^n$, there is a Boolean circuit of $O(n)$ gates to compute the function $\delta_\alpha:\{0,1\}^n \rightarrow \{0,1\}$ such that $\delta_\alpha(x)=1$ if and only if $x=\alpha$. The circuit is very simple. Given input $x_0,\ldots,x_{n-1}$ we compute the  AND of $z_0,\ldots,z_{n-1}$ where $z_i=x_i$ if $\alpha_i=1$ and $z_i = NOT(x_i)$ if $\alpha_i=0$. While formally Boolean circuits only have a gate for computing the AND of two inputs, we can implement an AND of $n$ inputs by composing $n$ two-input ANDs.
```


## 4.6 $SIZE_{n,m}(s)$类 {#secdefinesizeclasses }


We have seen that _every_ function $f:\{0,1\}^n \rightarrow \{0,1\}^m$ can be computed by a circuit of size $O(m\cdot 2^n)$, and _some_ functions (such as addition and multiplication) can be computed by much smaller circuits.
We define $SIZE_{n,m}(s)$ to be the set of functions mapping $n$ bits to $m$ bits that can be computed by NAND circuits of at most $s$ gates (or equivalently, by NAND-CIRC programs of at most $s$ lines).
Formally, the definition is as follows:

```admonish quote title=""
{{defc}}{def:size}[Size class of functions]
For all natural numbers $n,m,s$, let $SIZE_{n,m}(s)$ denote the set of all functions $f:\{0,1\}^n \rightarrow \{0,1\}^m$ such that there exists a NAND circuit of at most $s$ gates computing $f$.
We denote by $SIZE_n(s)$ the set $SIZE_{n,1}(s)$.
For every integer $s \geq 1$, we let $SIZE(s) = \cup_{n,m} SIZE_{n,m}(s)$ be the set of all functions $f$ for which there exists a NAND circuit of at most $s$ gates that compute $f$.
```


{{ref:fig:funcvscirc}} depicts the set $SIZE_{n,1}(s)$.
Note that $SIZE_{n,m}(s)$ is a set of _functions_, not of _programs!_ Asking if a program or a circuit is a member of $SIZE_{n,m}(s)$ is a _category error_ as in the sense of  {{ref:fig:cucumber}}.
As we discussed in [3.7.2节](./chapter_3.md#specvsimplrem) (and  [第2.6.1节](chapter_2.md#secimplvsspec)), the distinction between _programs_ and _functions_ is absolutely crucial.
You should always remember that while a program _computes_ a function, it is not _equal_ to a function.
In particular, as we've seen, there can be more than one program to compute the same function.



```admonish pic id="funcvscircfig"
![funcvscircfig](./images/chapter4/funcvscircs.png)

{{pic}}{fig:funcvscirc} There are $2^{2^n}$ functions mapping $\{0,1\}^n$ to $\{0,1\}$, and an infinite number of circuits with $n$ bit inputs and a single bit of output. Every circuit computes one function, but every function can be computed by many circuits. We say that $f \in SIZE_{n,1}(s)$ if the smallest circuit that computes $f$ has $s$ or fewer gates. For example $XOR_n \in SIZE_{n,1}(4n)$. {{ref:thm:NAND-univ}} shows that _every_ function $g$ is computable by some circuit of at most $c\cdot 2^n/n$ gates, and hence $SIZE_{n,1}(c\cdot 2^n/n)$ corresponds to the set of _all_ functions from $\{0,1\}^n$ to $\{0,1\}$.
```


While we defined $SIZE_n(s)$ with respect to NAND gates, we would get essentially the same class if we defined it with respect to AND/OR/NOT gates:

```admonish quote title=""
{{lemc}}{lem:nandaonsize}
Let $SIZE^{AON}_{n,m}(s)$ denote the set of all functions $f:\{0,1\}^n \rightarrow \{0,1\}^m$ that can be computed by an AND/OR/NOT Boolean circuit of at most $s$ gates.
Then,
$$
SIZE_{n,m}(s/2) \subseteq SIZE^{AON}_{n,m}(s) \subseteq SIZE_{n,m}(3s)
$$
```

```admonish proof collapsible=true, title = "对{{ref:lem:nandaonsize}}的证明"
If $f$ can be computed by a NAND circuit of at most $s/2$ gates, then by replacing each NAND with the two gates NOT and AND, we can obtain an AND/OR/NOT Boolean circuit of at most $s$ gates that computes $f$.
On the other hand, if $f$ can be computed by a Boolean AND/OR/NOT circuit of at most $s$ gates, then by {{ref:thm:NANDuniversam}} it can be computed by a NAND circuit of at most $3s$ gates.
```




```admonish pic id="cucumberfig"
![cucumberfig](./images/chapter4/cucumber.png)

{{pic}}{fig:cucumber} A "category error" is a question such as "is a cucumber even or odd?" which does not even make sense. In this book one type of category error you should watch out for is confusing _functions_ and _programs_ (i.e., confusing _specifications_ and _implementations_). If $C$ is a circuit or program, then asking if $C \in SIZE_{n,1}(s)$ is a category error, since $SIZE_{n,1}(s)$ is a set of _functions_ and not programs or circuits.
```


The results we have seen in this chapter can be phrased as showing that $ADD_n \in SIZE_{2n,n+1}(100 n)$
and $MULT_n \in SIZE_{2n,2n}(10000 n^{\log_2 3})$.
{{ref:thm:NAND-univ}} shows that  for some constant $c$, $SIZE_{n,m}(c m 2^n)$ is equal to the set of all functions from $\{0,1\}^n$ to $\{0,1\}^m$.




```admonish info
{{remc}}{rem:infinitefunc}[Finite vs infinite functions]
Unlike programming languages such as _Python_, _C_ or _JavaScript_, the NAND-CIRC and AON-CIRC programming language do not have _arrays_. 
A NAND-CIRC program $P$ has some fixed number $n$ and $m$ of inputs and output variable. Hence, for example, there is no single NAND-CIRC program that can compute the increment function $INC:\{0,1\}^* \rightarrow \{0,1\}^*$ that maps a string $x$ (which we identify with a number via the binary representation) to the string that represents $x+1$. Rather for every $n>0$, there is a NAND-CIRC program $P_n$ that computes the restriction $INC_n$ of the function $INC$ to inputs of length $n$. Since it can be shown that for every $n>0$ such a program $P_n$ exists of length at most $10n$, $INC_n \in SIZE_{n,n+1}(10n)$ for every $n>0$.

For the time being, our focus will be on _finite_ functions, but we will discuss how to extend the definition of size complexity to functions with unbounded input lengths later on in [第13.6节](chapter_13.md#nonuniformcompsec).
```


```admonish question
{{exec}}{exe:sizeclosundercomp}[$SIZE$ closed under complement.]

In this exercise we prove a certain "closure property" of the class $SIZE_n(s)$.
That is, we show that if $f$ is in this class then (up to some small additive term) so is the complement of $f$, which is the function $g(x)=1-f(x)$.

Prove that there is a constant $c$ such that for every $f:\{0,1\}^n \rightarrow \{0,1\}$ and $s\in \N$, if $f \in SIZE_n(s)$  then $1-f \in SIZE_n(s+c)$.
```

```admonish solution collapsible=true title="对{{ref:exe:sizeclosundercomp}}的解答"
If $f\in SIZE_n(s)$ then there is an $s$-line NAND-CIRC program $P$ that computes $f$.
We can rename the variable `Y[0]` in $P$ to a variable `temp` and add the line

~~~python
Y[0] = NAND(temp,temp)
~~~

at the very end to obtain a program $P'$ that computes $1-f$.
```



```admonish hint title="本章回顾"
* We can define the notion of computing a function via a simplified "programming language", where computing a function $F$ in $T$ steps would correspond to having a $T$-line NAND-CIRC program that computes $F$.
* While the NAND-CIRC programming only has one operation, other operations such as functions and conditional execution can be implemented using it.
* Every function $f:\{0,1\}^n \rightarrow \{0,1\}^m$ can be computed by a circuit of at most $O(m 2^n)$ gates (and in fact at most $O(m 2^n/n)$ gates).
* Sometimes (or maybe always?) we can translate an _efficient_ algorithm to compute $f$ into a circuit that computes $f$  with a number of gates comparable to the number of steps in this algorithm.
```

## 4.7 习题



```admonish question title=""
{{proc}}{pro:embedtuples}[Pairing]
This exercise asks you to give a one-to-one map from $\N^2$ to $\N$. This can be useful to implement two-dimensional arrays as "syntactic sugar" in programming languages that only have one-dimensional arrays.

1. Prove that the map $F(x,y)=2^x3^y$ is a one-to-one map from $\N^2$ to $\N$.

2. Show that there is a one-to-one map $F:\N^2 \rightarrow \N$ such that for every $x,y$, $F(x,y) \leq 100\cdot \max\{x,y\}^2+100$.

3. For every $k$, show that there is a one-to-one map $F:\N^k \rightarrow \N$ such that for every $x_0,\ldots,x_{k-1} \in \N$, $F(x_0,\ldots,x_{k-1}) \leq 100 \cdot (x_0+x_1+\ldots+x_{k-1}+100k)^k$.
```

```admonish question title=""
{{proc}}{pro:mux}[Computing MUX]
Prove that the NAND-CIRC program below computes the function $MUX$ (or $LOOKUP_1$) where $MUX(a,b,c)$ equals $a$ if $c=0$ and equals $b$ if $c=1$:

~~~python
t = NAND(X[2],X[2])
u = NAND(X[0],t)
v = NAND(X[1],X[2])
Y[0] = NAND(u,v)
~~~
```


```admonish question title=""
{{proc}}{pro:atleasttwo}[At least two / Majority]
Give a NAND-CIRC program of at most 6 lines to compute the function  $MAJ:\{0,1\}^3 \rightarrow \{0,1\}$
where $MAJ(a,b,c) = 1$ iff $a+b+c \geq 2$.
```

```admonish question title=""
{{proc}}{pro:conditionalsugar}[Conditional statements]
In this exercise we will explore {{ref:thm:conditionalsugar}}: transforming NAND-CIRC-IF programs that use code such as `if .. then .. else ..` to standard NAND-CIRC programs.

1. Give a "proof by code" of {{ref:thm:conditionalsugar}}: a program in a programming language of your choice that transforms a NAND-CIRC-IF program $P$ into a "sugar-free" NAND-CIRC program $P'$ that computes the same function. See footnote for hint.{{footnote:You can start by transforming $P$ into a NAND-CIRC-PROC program that uses procedure statements, and then use the code of {{ref:exa:desugarcode}} to transform the latter into a "sugar-free" NAND-CIRC program.}}

2. Prove the following statement, which is the heart of  {{ref:thm:conditionalsugar}}: suppose that there exists an $s$-line NAND-CIRC program to compute $f:\{0,1\}^n \rightarrow \{0,1\}$ and an $s'$-line NAND-CIRC program to compute $g:\{0,1\}^n \rightarrow \{0,1\}$.
Prove that there exist a NAND-CIRC program of at most $s+s'+10$ lines to compute the function $h:\{0,1\}^{n+1} \rightarrow \{0,1\}$ where $h(x_0,\ldots,x_{n-1},x_n)$ equals $f(x_0,\ldots,x_{n-1})$ if $x_n=0$ and equals $g(x_0,\ldots,x_{n-1})$ otherwise. (All programs in this item are standard "sugar-free" NAND-CIRC programs.)
```



```admonish question title=""
{{proc}}{pro:halffulladder}[Half and full adders]
1. A _half adder_ is the function $HA:\{0,1\}^2 :\rightarrow \{0,1\}^2$ that corresponds to adding two binary bits. That is, for every $a,b \in \{0,1\}$, $HA(a,b)= (e,f)$ where $2e+f = a+b$. Prove that there is a NAND circuit of at most five NAND gates that computes $HA$.

2. A _full adder_ is the function $FA:\{0,1\}^3 \rightarrow \{0,1\}^{2}$ that takes in two bits and a "carry" bit and outputs their sum. That is, for every $a,b,c \in \{0,1\}$, $FA(a,b,c) = (e,f)$ such that $2e+f = a+b+c$. Prove that there is a NAND circuit of at most nine NAND gates that computes $FA$.

3. Prove that if there is a NAND circuit of $c$ gates that computes $FA$, then there is a circuit of $cn$ gates that computes $ADD_n$ where (as in {{ref:thm:addition}}) $ADD_n:\{0,1\}^{2n} \rightarrow \{0,1\}^{n+1}$ is the function that outputs the addition of two input $n$-bit numbers. See footnote for hint.{{footnote:Use a "cascade" of adding the bits one after the other, starting with the least significant digit, just like in the elementary-school algorithm.}}

4. Show that for every $n$ there is a NAND-CIRC program to compute $ADD_n$ with at most $9n$ lines.
```


```admonish question title=""
{{proc}}{pro:addition}[Addition]
Write a program using your favorite programming language that on input of an integer $n$, outputs a NAND-CIRC program that computes $ADD_n$. Can you ensure that the program it outputs for $ADD_n$ has fewer than $10n$ lines?
```

```admonish question title=""
{{proc}}{pro:multiplication}[Multiplication]
Write a program using your favorite programming language that on input of an integer $n$, outputs a NAND-CIRC program that computes $MULT_n$. Can you ensure that the program it outputs for $MULT_n$ has fewer than $1000\cdot n^2$ lines?
```

```admonish question title=""
{{proc}}{pro:eff-multiplication}[Efficient multiplication (challenge)]
Write a program using your favorite programming language that on input of an integer $n$, outputs a NAND-CIRC program that computes $MULT_n$ and has at most $10000 n^{1.9}$ lines.{{footnote:__Hint:__ Use Karatsuba's algorithm.}} What is the smallest number of lines you can use to multiply two 2048 bit numbers?
```


```admonish question title=""
{{proc}}{pro:mult-bit}[Multibit function]
In the text {{ref:thm:NAND-univ}} is only proven for the case $m=1$.
In this exercise you will extend the proof for every $m$.

Prove that

1. If there is an $s$-line NAND-CIRC program to compute $f:\{0,1\}^n \rightarrow \{0,1\}$ and an $s'$-line NAND-CIRC program to compute $f':\{0,1\}^n \rightarrow \{0,1\}$ then there is an $s+s'$-line program to compute the function $g:\{0,1\}^n \rightarrow \{0,1\}^2$ such that $g(x)=(f(x),f'(x))$.

2. For every function $f:\{0,1\}^n \rightarrow \{0,1\}^m$, there is a NAND-CIRC program of at most $10m\cdot 2^n$ lines that computes $f$. (You can use the $m=1$ case of {{ref:thm:NAND-univ}}, as well as Item 1.)
```


```admonish question title=""
{{proc}}{pro:usesugar}[Simplifying using syntactic sugar]
Let $P$ be the following NAND-CIRC program:

~~~python
Temp[0] = NAND(X[0],X[0])
Temp[1] = NAND(X[1],X[1])
Temp[2] = NAND(Temp[0],Temp[1])
Temp[3] = NAND(X[2],X[2])
Temp[4] = NAND(X[3],X[3])
Temp[5] = NAND(Temp[3],Temp[4])
Temp[6] = NAND(Temp[2],Temp[2])
Temp[7] = NAND(Temp[5],Temp[5])
Y[0] = NAND(Temp[6],Temp[7])
~~~

1. Write a program $P'$ with at most three lines of code that uses both `NAND` as well as the syntactic sugar `OR` that computes the same function as $P$.

2. Draw a circuit that computes the same function as $P$ and uses only $AND$ and $NOT$ gates.
```



In the following exercises you are asked to compare the _power_ of pairs of programming languages.
By "comparing the power" of two programming languages $X$ and $Y$ we mean determining the relation between the set of functions that are computable using programs in  $X$ and $Y$ respectively. That is, to answer such a question you need to do both of the following:

1. Either prove that for every program $P$ in $X$ there is a program $P'$ in $Y$ that computes the same function as $P$, _or_ give an example for a function that is computable by an $X$-program but not computable by a $Y$-program.

_and_

2. Either prove that for every program $P$ in $Y$ there is a program $P'$ in $X$ that computes the same function as $P$, _or_ give an example for a function that is computable by a $Y$-program but not computable by an $X$-program.

When you give an example as above of a function that is computable in one programming language but not the other, you need to _prove_ that the function you showed is _(1)_ computable in the first programming language and _(2)_ _not computable_ in the second programming language.

```admonish question title=""
{{proc}}{pro:compareif}[Compare IF and NAND]
Let IF-CIRC be the programming language where we have the following operations `foo = 0`, `foo = 1`, `foo = IF(cond,yes,no)`  (that is, we can use the constants $0$ and $1$, and the $IF:\{0,1\}^3 \rightarrow \{0,1\}$ function such that $IF(a,b,c)$ equals $b$ if $a=1$ and equals $c$ if $a=0$). Compare the power of the NAND-CIRC programming language and the IF-CIRC programming language.
```

```admonish question title=""
{{proc}}{pro:comparexor}[Compare XOR and NAND]
Let XOR-CIRC be the programming language where we have the following operations `foo = XOR(bar,blah)`, `foo = 1` and `bar = 0` (that is, we can use the constants $0$, $1$ and the $XOR$ function that maps $a,b \in \{0,1\}^2$ to $a+b \mod 2$). Compare the power of the NAND-CIRC programming language and the XOR-CIRC programming language. See footnote for hint.{{footnote:You can use the fact that $(a+b)+c \mod 2 = a+b+c \mod 2$. In particular it means that if you have the lines `d = XOR(a,b)` and `e = XOR(d,c)` then `e` gets the sum modulo $2$ of the variable `a`, `b` and `c`.}}
```

```admonish question title=""
{{proc}}{pro:majasymp}[Circuits for majority]
Prove that there is some constant $c$ such that for every $n>1$, $MAJ_n \in SIZE_n(cn)$ where $MAJ_n:\{0,1\}^n \rightarrow \{0,1\}$ is the majority function on $n$ input bits. That is $MAJ_n(x)=1$ iff $\sum_{i=0}^{n-1}x_i > n/2$. See footnote for hint.{{footnote:One approach to solve this is using recursion and the  so-called [Master Theorem](https://en.wikipedia.org/wiki/Master%5Ftheorem%5F(analysis%5Fof%5Falgorithms)).}}
```


```admonish question title=""
{{proc}}{pro:thresholdcirc}[Circuits for threshold]
Prove that there is some constant $c$ such that for every $n>1$, and integers $a_0,\ldots,a_{n-1},b \in \{-2^n,-2^n+1,\ldots,-1,0,+1,\ldots,2^n\}$, there is a NAND circuit with at most $n^c$ gates that computes the _threshold_ function $f_{a_0,\ldots,a_{n-1},b}:\{0,1\}^n \rightarrow \{0,1\}$ that on input $x\in \{0,1\}^n$ outputs $1$ if and only if $\sum_{i=0}^{n-1} a_i x_i > b$.
```



## 4.8 杂记 { #computeeveryfunctionbibnotes  }


See Jukna's and Wegener's books [@Jukna12, @wegener1987complexity] for much more extensive discussion on circuits.
Shannon showed that every Boolean function can be computed by a circuit of exponential size [@Shannon1938]. The improved bound of $c \cdot 2^n/n$ (with the optimal value of $c$ for many bases) is due to Lupanov [@Lupanov1958]. An exposition of this for the case of NAND (where $c=1$) is given in Chapter 4 of his book [@lupanov1984].
(Thanks to Sasha Golovnev for tracking down this reference!)

The concept of "syntactic sugar" is also known as "macros" or "meta-programming" and is sometimes implemented via a preprocessor or macro language in a programming language or a text editor. One modern example is the [Babel](https://babeljs.io/) JavaScript syntax transformer, that converts JavaScript programs written using the latest features into a format that older Browsers can accept. It even has a [plug-in](https://babeljs.io/docs/plugins/) architecture, that allows users to add their own syntactic sugar to the language.