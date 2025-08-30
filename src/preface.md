# 前言

## 前言1 致学生们

这一本书可能相当具有挑战性，主要原因是这本书中将各种计算理论的观点和技术融合到了一起。其中一些技巧是比较难以掌握的，不论是通过对角线论证证明停机问题的不可判定性，还是在NP完全问题归约中使用的组合工具，抑或是分析概率算法，再或者是通过讨论对抗过程从而证明密码学基本构件的安全性。

阅读这本书的最佳方法是积极地去阅读笔记，所以说我建议你准备好你的笔。当你阅读这本书的时候，我鼓励你不时地停下来并思考如下地问题：

* 当我陈述一个定理的时候，停下来并用一点时间尝试在你阅读证明之前自己证明这个定理。在这短短五分钟的尝试以后，你将会为能更好地理解标准证明而感到惊喜。
* 当你在阅读定义的时候，一定要保证你完全理解了定义的含义，以及哪些自然的例子能够成为定义所描述的对象。此外，一定要去思考定义背后的动机，以及是否有其它自然的方式来形式化这些概念。
* 积极地注意你阅读过程中脑中浮现出的问题，并且考虑他们是否在阅读文本的过程中得到了解答。

一个普遍的规则是，理解定义比理解定理更加重要，理解定理比理解证明更加重要。不论如何，在你证明定理之前，你一定要理解它到底陈述的是什么，一定要了解定理中对象所涉及的定义。不论证明如何复杂，我会提供证明定理的“证明想法”。你可以在第一次阅读的时候自由地跳过正式的证明，而单独把注意力放在“证明想法”上面。

本书包含了一部分代码片段，但这并不代表他们是编程文本。实际上，在阅读本书的时候，你不必知道如何进行编程。我们使用代码的原因，仅仅是为了更加精确地描述计算过程。实际的实现细节对于我们而言是不重要的，因此我们将会强调以更多考量换取代码可读性的重要性，例如错误处理、代码封装，等等，这些技巧对于我们实际的编程是非常重要的。

### 但是这些努力是否值得？

这并不是一本简单的书，你有理由思考自己为什么自己要花费这么多精力在学习这本书之上。一个对于计算理论课程的经典辩护是，你可能会在你未来的工作中遇到这些概念。你可能会遇到一个非常困难的问题，而后续你会意识到它是一个NP完全问题；又或者，你会在未来找到应用你所学到的正则表达式的地方。这可能是实话，但是这本书的主要功用并不是教给你任何实用的工具或者技能，而是给予你一种全新的思考方式：一种在计算问题出现时、穿破重重看似无关的设定识别它们的思想，一种建模计算任务和问题的思想，以及通过以上两个思想进行推理的能力。

无论你如何运用这本书，我都相信学习这本书是非常重要的。这是因为，它包含了许多非常优美且基本的概念。在本世纪，计算与信息扮演了能量和物质在上世纪的角色——作为我们理解世界的基石，而并不仅仅是作为我们科技和经济的工具。这本书将会让你简单了解这些理论背后的内容，并且希望能够激发各位读者进一步学习和了解更多知识的动力。


## 前言 2 致未来的授课教师们

这本书虽然是我为哈佛大学课程所作，但是我希望其它授课教师也认为它大有裨益。从某种意义上来讲，它与卡内基梅隆大学和麻省理工学院开设的“计算理论导论”和“伟大的想法”等课程的内容是类似的。

这本书所使用的教学方法与其它传统书籍（例如Hopcroft和Ullman于1969年出版的教材以及Sipser在1997年出版的教材）最显著的不同，在于本书将不会从有限自动机开始建立计算模型。相反，我们将会从布尔电路出发来建立这一切。我们相信，布尔电路才是计算理论中比自动机更加基本的内容。（甚至也是更加实用的！）更重要的是，布尔电路是许多只会在介绍现代理论计算机理论的课程中才会被提及的诸概念的前置概念。这些理论理论包括现代密码学、量子计算、去随机化理论、一些对于证明$\bf{P}\ne \bf{NP}$的尝试，等等等等。甚至，在某些并不必须使用布尔电路的情况下，布尔电路能够极大地简化这些问题（例如在证明Cook-Levin定理时）。

不仅如此，我认为以布尔电路而不是有限自动机作为起始，还有许多教学上的理由。布尔电路是更加自然的计算模型，其与硅基电路联系紧密，能够与学生们的实践直接产生关联。按理来说，有限值函数往往比无限值函数更容易掌握，因为我们完全可以将它的真值表直接写出。“任何一个有限值函数都可以被布尔函数计算”，这样的简单但是重要的定理可以作为课程的一个极好的起点。更进一步，许多计算理论中的观点，例如编码和数据之间对偶关系的观点、“普遍性”的观点等等，我们都可以从这一理论中体悟出来。

紧随布尔电路，我们将会进入图灵机的学习，并且证明一些重要的结果，例如通用图灵机的存在、停机问题的不可计算性以及Rice's Theorem。我们将会在了解图灵机和不可判定性之后讨论自动机，并将其作为限制型计算模型的例子（这一类机器的停机问题可以被高效解决）。

尽管按照电路——图灵机——自动机的顺序来介绍并不是我们的初衷，这个顺序与这些模型的发现的时间顺序是恰好吻合的。布尔代数可以追溯到Boole和DeMorgan在19世纪40年代的工作（尽管布尔电路的严格定义由Shannon在90年之后才给出）。Alan Turing在20世纪30年代定义了我们现在所称呼的“图灵机”，而有限自动机在1943年才在McCulloch和Pitts的工作中被正式提出。并且，直到1959年Rabin和Scott发表了他们重要的工作以后，自动机才逐渐被人们所了解。

更重要的是，尽管诸如有限自动机、正则表达式以及上下文无关语法在工程中非常重要，这些模型能够得到重用（不论是用于语法解析、分析生命周期和安全性还是用于软件定义路由表）绝大部分都要归因于他们是可控制的模型，我们可以轻易地通过它们来回答一些语义上的问题。在学生了解了通用计算模型的语义性质的不可判定性，它们可能会对这些实际应用上的想法感到叹为观止。

从电路入门使得我们证明Cook-Levin Theorem非常方便。事实上，我们的证明可以被一些Python程序完成。通过将这个证明与标准的归约结合，学生们能够直观地欣赏计算理论中的问题是如何被转化为图中独立集的存在性问题的。

这里，我们列举出一些与过往文献的不同：

1. 为了衡量时间复杂度，我们使用在算法课中使用过的标准的RAM机器模型（隐式的）而不是图灵机。尽管这两个模型毫无疑问是多项式等价的，且两个模型上复杂度类$\bf{P}$和$\bf{NP}$以及$\bf{EXP}$没有任何区别，我们的选择使得记号$O(n)$和$O(n^2)$之间的区别更加有意义。这样的选择使得这些更加细致的复杂度类型对应上学生们在算法课上学到的关于线性和二次时间的非正式定义（或者是对于需要他们手写代码的面试环节有所好处）（译者注：面试环节通常需要面试者在白板上手写代码，并给出时间复杂度分析）。

2. 我们使用“函数”而不是“语言”。这就是说，与其说“图灵机$M$判定语言$L\subseteq \{0,1\}^*$”，我们说它“计算了一个布尔函数$F:\{0,1\}^* \to \{0,1\}$”（译者注：$\{0,1\}^*$表示任意长度的二进制串，或者说0-1串）。“语言”这一术语兴起于Chomsky的1956年的工作，但是往往令人迷惑。“语言”相关的术语同时也使得讨论有关计算有多比特输出的函数的算法相关的概念非常的低效（包含一些非常基本的任务，例如讨论加法、乘法，等等）。但是，使用函数而不是语言意味着我们必须格外警惕学生可能会把“计算任务的规范”（函数）和“该任务的实现”（程序）搞混。另一方面，我们必须重复向学生强调和并训练他们要牢记这一点，无论使用何种记号。但是与此同时，本书同样会时不时提及“语言”相关的术语，以便于学生在课外查找相关资料。

上面教学大纲对于有限自动机和上下文无关语言的减免使得授课教师们能够讲授更多在现代理论课程之前所需要了解的知识。它们包括：随机性和计算，程序和证明之间的交互（包含哥德尔不完备定理、交互式证明系统、甚至包含一些Lambda演算、Curry-Howard同构）、密码学以及量子计算。

这本书提供了足以进行自学的细节。为了达到这个目的，每一个章节的开头都会列举这个章节的学习目标，末尾则会进行总结和回顾，行文之间穿插着“停顿框”以鼓励学生们停下来并求解一个问题或者检查他们是否在继续学习之前完全明白了前文所述的定义。

“第0章”的第五节提供了本书的一个“地图”，概括性地描述了不同章节的大概内容，同时还阐述了他们之间的依赖关系。这对于课程的规划是非常有益的。

## 致谢

这段文字正在持续更新，我收到了许多人的反馈，对此我深怀感激。Salil Vadhan 与我共同教授了这门课程的最初版本，在此过程中给予了我大量宝贵的反馈与洞见。Michele Amoretti 和 Marika Swanberg 仔细审阅了本书的若干章节，并提供了极其详尽且有益的评论。Dave Evans 和 Richard Xu 提交了许多 pull request，修正错误并改进措辞。感谢 Anil Ada、Venkat Guruswami 和 Ryan O’Donnell 分享他们在教授 CMU 15-251 时的经验与建议。感谢 Adam Hesterberg 和 Madhu Sudan 就使用本书教授 CS 121 的经验提出意见。Kunal Marwaha 提供了诸多评论，并在本书的技术制作方面给予了极大帮助。

感谢所有通过 GitHub 仓库 https://github.com/boazbk/tcs 发送评论、报告错别字或提交 issue 与 pull request 的人。特别感谢以下人士的宝贵反馈：Scott Aaronson、Michele Amoretti、Aadi Bajpai、Marguerite Basta、Anindya Basu、Sam Benkelman、Jarosław Błasiok、Emily Chan、Christy Cheng、Michelle Chiang、Daniel Chiu、Chi-Ning Chou、Michael Colavita、Brenna Courtney、Rodrigo Daboin Sanchez、Robert Darley Waddilove、Anlan Du、Juan Esteller、David Evans、Michael Fine、Simon Fischer、Leor Fishman、Zaymon Foulds-Cook、William Fu、Kent Furuie、Piotr Galuszka、Carolyn Ge、Jason Giroux、Mark Goldstein、Alexander Golovnev、Sayan Goswami、Maxwell Grozovsky、Michael Haak、Rebecca Hao、Lucia Hoerr、Joosep Hook、Austin Houck、Thomas Huet、Emily Jia、Serdar Kaçka、Chan Kang、Nina Katz-Christy、Vidak Kazic、Joe Kerrigan、Eddie Kohler、Estefania Lahera、Allison Lee、Benjamin Lee、Ondřej Lengál、Raymond Lin、Emma Ling、Alex Lombardi、Lisa Lu、Kai Ma、Aditya Mahadevan、Kunal Marwaha、Christian May、Josh Mehr、Jacob Meyerson、Leon Mlodzian、George Moe、Todd Morrill、Glenn Moss、Haley Mulligan、Hamish Nicholson、Owen Niles、Sandip Nirmel、Sebastian Oberhoff、Thomas Orton、Joshua Pan、Pablo Parrilo、Juan Perdomo、Banks Pickett、Aaron Sachs、Abdelrhman Saleh、Brian Sapozhnikov、Anthony Scemama、Peter Schäfer、Josh Seides、Alaisha Sharma、Nathan Sheely、Haneul Shin、Noah Singer、Matthew Smedberg、Miguel Solano、Hikari Sorensen、David Steurer、Alec Sun、Amol Surati、Everett Sussman、Marika Swanberg、Garrett Tanzer、Eric Thomas、Sarah Turnill、Salil Vadhan、Patrick Watts、Jonah Weissman、Ryan Williams、Licheng Xu、Richard Xu、Wanqian Yang、Elizabeth Yeoh-Wang、Josh Zelinsky、Fred Zhang、Grace Zhang、Alex Zhao 与 Jessica Zhu。
在本书的排版与制作过程中，我使用了许多开源软件包，对此我满怀感激。特别感谢 Donald Knuth 与 Leslie Lamport 创造了 LaTeX，以及 John MacFarlane 开发了 Pandoc。David Steurer 编写了最初用于生成此文本的脚本。当前版本使用了 Sergio Correia 的 panflute。LaTeX 与 HTML 模板源自 Tufte LaTeX、Gitbook 和 Bookdown。感谢 Amy Hendrickson 提供的 LaTeX 咨询。Juan Esteller 与 Gabe Montague 最初用 OCaml 与 JavaScript 实现了 NAND* 编程语言。我使用 Jupyter 项目编写了补充代码片段。

最后，我要感谢我的家人：我的妻子 Ravit，以及我的孩子 Alma 与 Goren。撰写本书（以及相应的课程）占用了我大量时间，以至于 Alma 在她的五年级作文中写道：“大学不应当逼迫教授过度工作。”遗憾的是，我所能展示的成果，似乎只是 600 页极度枯燥的数学文字。


