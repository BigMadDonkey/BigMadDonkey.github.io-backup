---
title: 编译系统笔记
author: MadDonkey
layout: post
categories: [note]
toc: true
---
这是编译系统的学习笔记。

# 紧急：讨论题1

编译的过程也可以看作是一种翻译的过程，只不过翻译的源语言是高级语言，目标语言是机器语言/汇编语言。

编译的过程需要进行词法分析、语法分析和语义分析，对应于人工翻译的对源语言句子的分析过程，都会得到语言无关的中间代码（对应着人工翻译中的语义），然后根据中间产物生成目标语言（对应于人工翻译中根据语义的造句）。



1. **<a href="#1">计算机程序设计语言</a>**
	<a href="#1.1">1.1 编译系统的结构</a>
	<a href="#1.2">1.2 词法分析概述</a>
	<a href="#1.3">1.3 语法分析概述</a>	
	<a href="#1.4">1.4 语义分析概述</a>
	<a href="#1.5">1.5 中间代码生成以及编译器后端</a>
	<a href="#1.6">1.6 T型图</a>
	
2. **<a href="#2">程序设计语言及文法</a>**
	<a href="#2.1">2.1 基础概念</a>
	<a href="#2.2">2.2 技巧：推导和规约</a>
	<a href="#2.3">2.3 文法分类</a>
	
3. **<a href="#3">词法分析</a>**
	<a href="#3.1">3.1 基础概念</a>
	<a href="#3.2">3.2 正则定义</a>
	<a href="#3.3">3.3 文法分类</a>

## <a id='1'>1. 计算机程序设计语言</a>

计算机程序设计语言的三个层次：

1. 机器语言：可以被计算机直接理解，但难以被人理解和记忆。
2. 汇编语言：在机器语言的基础上引入助记符，相对机器语言更直观，但依赖于特定机器。
3. 高级语言：以类似自然语言/数学定义的形式，更接近人类思维。

将汇编语言程序转变成机器语言的过程称为汇编，**将高级语言程序转变成汇编语言/机器语言的过程称为编译。**此处前者称为源语言，后者称为目标语言。

![image-20210307121123872](../assets/postResources/image-20210307121123872.png)

<center>    <img src="{{'assets/postResources/image-20210307121123872.png'|relative_url}}" alt="编译程序的流程" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.1 编译程序的过程</div> </center>

预处理的职能在于将不同源程序聚合在一起，并解析宏定义，而所谓可重定位（Relocatable）指的是程序在内存中存放的起始位置不固定。链接器负责将多个可重定位的机器代码文件和库文件连接到一起，解决了外部的内存地址问题。

### <a id = '1.1'>1.1 编译系统的结构</a>

编译系统做的，实际上是将高级语言翻译成抽象层度较低的语言这一任务。以语言翻译为例，把源语言翻译成一种目标语言有几个关键步骤：

1. 对源语言句子进行**词法分析(Lexical Analysis)**，确定词性（例如名词、动词、介词...）；
2. 在知道词性的基础上，对源语言句子进行**语法分析(Syntax Analysis)**，划分句子的成分（识别各类短语，比如名词短语、动宾短语...）；
3. 在划分成分的基础上，对源语言句子进行**语义分析(Semantic Analysis)**，获得句子的语义（分析出各个短语在句子中的成分，例如某个代词做主语，某个名词短语做宾语，某个介词短语做状语...）。

![image-20210307124153848](../assets/postResources/image-20210307124153848.png)

<center>    <img src="{{'assets/postResources/image-20210307124153848.png'|relative_url}}" alt="编译器的结构" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.2 编译器的结构</div> </center>

编译器的前4个阶段称为前端，只与源语言相关，后两个阶段则只与目标语言相关，称为后端。中间的机器无关代码生成过程与2者都不直接相关。具体的实现中，可在语法分析基础上直接进行语义分析，还可将分析结果直接表示成中间代码，因此语法分析、语义分析和生成中间代码的过程常常合而为一。

### <a id="1.2">1.2 词法分析概述</a>

词法分析的任务从左向右，逐行扫描源程序的内容，识别出各个单词，确定单词的类型，并将识别出的单元转化为统一的机内表示——词法单元(token)。

token是一个二元组，由**种别码**和**属性值**构成。

种别码表示token的类型。在程序设计语言中，词类大体上分为以下五类：

![image-20210307130141582](../assets/postResources/image-20210307130141582.png)

<center>    <img src="{{'assets/postResources/image-20210307130141582.png'|relative_url}}" alt="单词类型和种别码" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.3 单词类型和种别码</div> </center>

对于每种特定的程序设计语言，关键字集合是事先确定的，因此为每种关键字设定一个种别码；而标识符不能事先确定，所有的标识符都归为同一个种别码，通过token的第二个元素：属性值来区分具体的类型，因为属性值存放着token的具体字面值。

常量类似于标识符，也是开放的、无法枚举的集合，但**不同类型的常量的构成方式不同**，故而每种类型的常量分配一个种别码同样用token的属性值来存放字面值，以区分不同常量。

运算符和界限符与关键字类似，因此可以分配成一词一码。也可以把运算符划分成不同类型，不同类型下再根据属性值划分，不过都差不多就是了。

### <a id="1.3">1.3 语法分析概述</a>

语法分析的任务是：从词法分析器输出的token序列中识别出各类短语，构造语法分析树**（parse tree）**。

![image-20210307132016036](../assets/postResources/image-20210307132016036.png)

<center>    <img src="{{'assets/postResources/image-20210307132016036.png'|relative_url}}" alt="语法分析示例" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.4 语法分析示例</div> </center>

### <a id="1.4">1.4 语义分析概述</a>

程序设计语言中的语句主要分为声明语句/可执行语句。

语义分析的任务是：收集标识符的属性信息——种属Kind（简单变量or复杂变量or过程？）、类型Type（int？char？int *？）、存储位置和占用长度、值、作用域、参数和返回值信息（对于过程而言）等。

这些标识符的属性信息会被存放到**符号表（Symbol Table）**中。具体实现中，符号表往往带有一个字符串表来存放所有标识符的名称属性，因此Name部分往往实现为一个字符串表中的起始位置和一个长度值，这样就能在字符串表中索引到名称。

![image-20210307133343246](../assets/postResources/image-20210307133343246.png)

<center>    <img src="{{'assets/postResources/image-20210307133343246.png'|relative_url}}" alt="符号表" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.5 符号表</div> </center>

语义分析的另一个任务是语义检查，检查语义的错误并作出响应。例如重复的变量声明、未经声明就使用、运算分量不匹配等。

### <a id="1.5">1.5 中间代码生成以及编译器后端</a>

源程序的中间表示可以有多种形式，例如三地址码（类似于汇编语言的指令序列，每个指令至多由三个操作数）和语法结构树（Syntax Tree）（注：语法树和语法分析树不是一样的东西）。

![image-20210307134105113](../assets/postResources/image-20210307134105113.png)

<center>    <img src="{{'assets/postResources/image-20210307134105113.png'|relative_url}}" alt="三地址指令示例" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.6 三地址指令示例</div> </center>

值得注意的部分：

①之所以可以使用变量名/常量等（即源程序中的标识符）作为地址，是因为他们的地址都在符号表存好了。

②条件跳转指令只有x和y满足relop时才会跳转到对应地址。

③param x表示将x设为参数。

④过程调用的操作中，p为过程的地址，n为参数的个数。

⑤过程返回语句表示返回到对应地址。

⑥数组操作中，x表示数组的基地址，i表示**偏移地址**，而不是下标！

三地址指令的表示也有几种形式：

1. 四元式：(op, y, z, x)。注意，第一个元素为操作符，第二个、第三个元素分别为源操作数，第四个元素对应目标操作数。因此(op,y, z, x)相当于x = y op z。类似地，(op, y, _, x)表示 x = op y，(=, y,  _, x)表示x = y。
2. 三元式
3. 间接三元式

### <a id="1.6">1.6 T型图</a>

可以使用T型图来表示编译器。

![image-20210310205821005](../assets/postResources/image-20210310205821005.png)

<center>    <img src="{{'assets/postResources/image-20210307134105113.png'|relative_url}}" alt="T型图" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图1.7 T型图</div> </center>

如图所示，P表示该编译器，T型图的顶部表示编译器的功能，其中S为源语言程序，T为目标语言程序。I表示编译器的底层实现语言。

## <a id="2">2. 程序设计语言及文法</a>

### <a id="2.1">2.1 基本概念</a>

字母表$\Sigma$表示一个**有穷**符号集合。这里的符号可以是字母、数字、标点符号...等等。以下给出一些字母表上的运算：

1. 乘积：$\Sigma_1\Sigma_2 = \{ab|a\in\Sigma_1, b\in\Sigma_2\}$
2. 幂：$\begin{aligned}\Sigma^0 &= \{\epsilon\}\\\Sigma^n &= \Sigma^{n-1}\Sigma, n\ge1\end{aligned}$也即长度为n的符号串的集合。
3. 正闭包：$\Sigma^+ = \Sigma \and \Sigma^2 \and \Sigma^3 \cdots$也即长度为正数的符号串之集合。
4. 克林闭包：$\Sigma^* = \Sigma^0 \and \Sigma \and \Sigma^2 \and \Sigma^3 \cdots$

假设$\Sigma$是一个字母表，对于$\forall x \in \Sigma^*$，x称为$\Sigma$上的一个串。串的长度记作$|x|$，表示s中符号的个数。串上支持的运算如下（以下示例中x，y，z，s均为串）：

1. 连接：$xy$表示将y附加到x之后形成的新串。注：空串$\epsilon$是任意串对于连接运算的单位元。如果x = yz，则称y是x的前缀，z是x的后缀。
2. 幂：把串上的连接运算看作一种乘积，则幂运算可以看作累积的连接。$\begin{aligned}s^0 &= \epsilon \\ s^n &= s^{n-1}s, n\ge1, \end{aligned}$

语言是一系列特定**句子**的集合，而**文法**表示一种语言的句子的构成规则。文法的形式化定义如下：

$G = (V_T,V_N,P,S)$表示一个文法G，其中$V_T$表示终结符集合，是文法所定义的基本符号，有时也称为token。顾名思义，终结符是不能再衍生的，是句型拓展的终结点。$V_N$表示文法的非终结符集合，也称为语法成分/语法变量。例如$V_N$可能包含<句子>、<名词短语>...非终结符是文法推导的关键。

$P$表示产生式集合，描述了将终结符/非终结符组合成串的方法。一般形式为$\alpha\rightarrow \beta$，读作“alpha定义为beta”。其中**$\alpha \in (V_T \and V_N)^+$且$\alpha$中至少包含一个非终结符**，策划归纳为产生式的左部/头。$\beta \in (V_T \and V_N)^*$称为产生式的右部/体。如果一系列产生式有相同的左部，则可以将他们的右部写在一起，用“|”分割，这时这些右部又叫做候选式。

$S$表示开始符号。开始符号是文法推导的起点，是非终结符集合的元素之一，是涵盖范围最大的语法成分。

**注意，一般场合下采用如下的符号规定：**

1. 使用排在开头的小写英文字母表示**终结符**，例如a, b, c，运算符、标点、数字和加粗的字符串（如**id**）也视为终结符。
2. 使用字母表排在前面的大写英文字母表示**非终结符**，例如A, B, C。除此之外，通常用大写字母S表示开始符号，使用小谢斜体的名词表示特定的非终结符，如*expr*，以及代表程序构造的大写字母（如E、T、F）
3. 使用字母表排在后面的大写字母表示**文法符号**（既可以表示终结符，又可以表示非终结符），例如X, Y, Z。
4. 使用字母表排在后面的小写字母表示**终结符号串**，例如x, y, z。
5. 使用小写希腊字母表示**文法符号串**，例如$\alpha$。
6. 除非特别说明，否则第一个产生式的头即为开始符号。

#### <a id="2.2">2.2 技巧：推导和规约</a>

推导Derivations是利用给定文法的产生式，用产生式的右部替换左部。例如如果$\alpha \rightarrow \beta \in P$，$\gamma\alpha\delta$就可以写成$\gamma\beta\delta$，也称作前者可以直接推导出后者。也可记作$\gamma\alpha\beta\implies\gamma\beta\delta$。

如果$\alpha_0$可以经过n步推导得到$\alpha_n$，则可记为$\alpha_0\implies^n\alpha_n$。注意$\alpha\implies^0\alpha$，$\implies^+$表示经过正数步推导，$\implies^*$表示经过若干步推导。如果一个符号串可以由文法的开始符号推导而得到，则证明该符号串满足此文法。

根据推导替换的非终结符不同，推导还可分为最左推导（每一步替换最左边的非终结符）和最右推导（与之对应）。

而规约Redution则与推导相反，使用产生式的左部来尝试替换右部，如果最后可以归约到文法的开始符号，则证明该符号串满足此文法。

推导和规约从生成和识别两个角度来判别一个句子是否属于某个文法的范畴。

如果$S\implies^*\alpha$，且$\alpha \in (V_T \and V_N)^*$，则认为$\alpha$是G的一个**句型**。注意句型可以包括非终结字符，也可以包括终结字符，甚至可以是空串。

如果$S\implies^*w$，且$w \in {V_T}^*$，则称w是G的一个**句子**。句子是不包含非终结符的句型。

由文法G的开始符号推导出的所有句子的集合称为文法G生成的**语言**。记作L(G)。

语言支持的运算如下(L、M都表示语言）：

1. 并：$L\and M = \{s|s\in L 或s\in M\}$
2. 连接：$LM = \{st|s\in L 且 t \in M\}$
3. 幂：$\begin{aligned}L^0 &= \{\epsilon\}\\ L^n &= L^{n-1}L, n\ge1\end{aligned}$
4. 正闭包：$L^+ = \bigcup_{i=1}^\infin L^i$
5. Kleene闭包：$L^* = \bigcup_{i=0}^\infin L^i$

#### <a id="2.3">2.3 文法分类</a>

Chomsky将文法分为四类：

1. 0型文法：无限制文法。此文法只要求生成式的左部至少有一个非终结符即可。非常宽泛。
2. 1型文法：上下文相关文法。此文法要求生成式左部的长度小于等于右部的长度。也可以认为1型文法的所有产生式都满足$\alpha A \beta \rightarrow \alpha\gamma\beta$，其中$\alpha, \beta \in (V_N \and V_T)^*$而$\gamma \in (V_N \and V_T)^+$。注意1型文法不包含ε产生式。
3. 2型文法：上下文无关文法（CFG）。**上下文无关文法**要求产生式的左部必须是非终结符集合中的元素。
4. <a id = "正则文法"></a>3型文法：正则文法。正则文法分为左线性文法和右线性文法。其中左线性文法的产生式规则为$A\rightarrow B$或$A\rightarrow Bw$，右线性文法与之相反。由3型文法生成的语言称为**正则语言**。

四种文法关系是随着文法等级提升，逐级限制，底层文法涵盖上层文法，即$0型\subset 1型 \subset 2型 \subset 3型$。

#### 语法分析树

对于CFG，可以根据推导，展开其分析树。根节点为文法的开始符号，每个内部节点都代表着存在一个产生式的左部为该节点，右部从左到右构成了该节点的子节点。

分析树呃叶节点既可以是非终结符，也可以是终结符。从左到右排列叶节点得到的符号串称为该树的yield（产出）或者frontier（边缘）。

分析树的推导的图形化表示，每一步推导得到的句型都可以构造出以之为产出的分析子树。给定一个句型，其分析树的**每一颗子树的frontier**称为该句型的一个短语。如果这颗子树还恰好高度为2，则称为直接短语。

另外，如果一个文法可以为某个句型生成多个分析树，则该文法是二义的。

## <a id="3">3. 词法分析</a>

### <a id="3.1">3.1 正则表达式</a>

正则表达式Regular Expression是一种用于**描述正则语言的更紧凑的表示方法**。正则变道时可以由较小的正则表达式按照规则递归地构建。每个正则表达式r定义了一个语言，记为L(r)。正则表达式的要点如下：

空串ε也是一个RE。L(ε) = {ε}。

如果a是字母表中的一个字符，a也是一个RE，L(a) = {a}。

如果r和s都是RE，则r|s 也是一个RE，L(r|s) = L(r)∪L(s)。类似地，rs也是一个RE，L(rs) = L(r)L(s)。

$r^*$也是一个RE，$L(r^*) = (L(r))^*$。

(r)也是一个RE，$L((r)) = L(r)$。

**注意，上述运算的优先级顺序为* > 连接运算 > |。**

**可以用RE定义的语言叫做正则语言/正则集合**。<a href="#正则文法">由此也可知</a>，对于任何正则文法G，存在定义同一语言的正则表达式r，反之亦然。

RE满足的一些代数定律：

1. 或运算的交换律：$r|s = s | r$
2. 或运算的结合律：$r|(s|t) =  (r|s)|t$
3. 连接运算的结合律：$r(st) = (rs)t$
4. 连接运算对或运算满足分配律：$r(s|t) = rs|rt$
5. ε是连接运算的单位元：$\epsilon r = r\epsilon = r$
6. Kleene闭包中包含ε：$r^* = (r|\epsilon)^*$
7. 闭包运算的幂等性：$(r^*)^* = r^*$

### <a id="3.2">3.2 正则定义</a>

为方便起见可以为已有的正则表达式命名，方便后续引用。正则定义是具有如下形式的定义序列：
$$
\begin{aligned}
d_1 &\rightarrow r_1 \\
d_2 &\rightarrow r_2 \\
d_3 &\rightarrow r_3 \\
&\cdots\\
d_n &\rightarrow r_n
\end{aligned}
$$
其中每个$d_i$都是新符号，不是字母表$\Sigma$中的，而且各不相同。而每个$r_i$是以$\Sigma\cup\{d_1,d_2\cdots d_{i-1}\}$为新字母表的正则表达式。此所谓后续的正则定义可以使用先前已有的定义。例如C语言中使用正则定义表示标识符合法范围的正则表达式：
$$
\begin{aligned}
&digit \rightarrow 0|1|2|\cdots|9\\
&letter\_ \rightarrow A|B|\cdots|Z|a|b|\cdots|z|\_ \\
&id \rightarrow letter\_(letter\_|digit)^*
\end{aligned}
$$


### <a id="3.3">3.3 有穷自动机</a>

有穷自动机Finite Automata，具有一系列离散的输入输出和有穷数目的内部状态。能够根据当前所处的状态和面临的输入信息决定系统的后继行为，并且系统处理了输入后内部状态也会发生改变。

![image-20210311152702865](../assets/postResources/image-20210311152702865.png)

<center>    <img src="{{'assets/postResources/image-20210311152702865.png'|relative_url}}" alt="FA模型" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图3.1 FA模型</div> </center>

一个经典的FA结构如图所示。输入带上存放着输入符号串，读头从左向右移动，逐个读取输入符号，不能往返移动。控制器根据当前状态和当前输入符号做出响应操作，转入下一状态。

可以使用转换图Transition Graph表示FA的结构。转换图中的结点表示状态，边表示在结点之间存在可行的状态转移，边上的字符表示接收的输入。一个转换图只能有一个初始状态（用start箭头标记），但可以有多个终止状态（终止状态外部额外有一层圈），转换图的示例可见后文的DFA和NFA。

一个FA也对应着一种语言。称给定输入串x，如果该FA存在一个对应于串x输入，从初始状态到某个终止状态的**转换序列**，则称串x被该FA**接收**；而一个FA所接收的所有串构成的集合就称为该FA定义/接收的语言，记作L(M)。

**FA和RE是等价的**。这是说，给定FA，可以构造出一个RE，它们表示相同的语言，反之亦然。前文中有提到过RE和正则文法在语言表达上是等价的，因此
$$
FA \iff RE \iff 正则文法
$$


##### 最长字串匹配原则

如果出现了输入串的多个前缀与FA的一个或多个模式匹配的情况（实际上，这很常见），优先选择最长的前缀的匹配方案。也正是因此，到达某个终止状态后，只要输入带还有符号，FA就会继续前进，以寻找尽可能长的匹配。

#### DFA、NFA和ε-NFA

确定的有穷自动机DFA，形式化定义为$M = (S, \Sigma, \delta, s_0, F)$，其中S表示DFA的有穷状态集合，$\Sigma$表示输入字符集（ε不算在内），$\delta$是一个从$S\times\Sigma$映射到$S$的转换函数（注：既然身为函数，就代表无论 自变量如何取，函数值总是存在的，这意味着对于DFA的任意一个状态，不存在接收输入而不知道下一状态是什么的情况）。$\delta(s,a)$表示从s状态出发，沿着标记为a的边所能达到的下一个状态。$s_0$表示开始状态，F表示终止状态集合。

![image-20210311160146800](../assets/postResources/image-20210311160146800.png)

<center>    <img src="{{'assets/postResources/image-20210311160146800.png'|relative_url}}" alt="DFA模型" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图3.2 DFA模型</div> </center>

可以将$\delta$的所有可能取值化成转换表。转换表和转换图一样可以表示DFA。

非确定的有穷自动机NFA，形式化定义为$M = (S, \Sigma, \delta, s_0, F)$，与普通DFA唯一的区别在于$\delta$。在NFA中，$\delta$是从$S\times\Sigma$映射到$2^S$的转换函数，表示的含义是在状态s下接收输入a所能到达的状态集合。这也是NFA”非确定性“的所在：即使当前状态和输入确定了，下一状态却仍是非确定的，可能是该集合中的某一个状态。

![image-20210311160047953](../assets/postResources/image-20210311160047953.png)

<center>    <img src="{{'assets/postResources/image-20210311160047953.png'|relative_url}}" alt="NFA模型" />    <br>    <div style="color:orange; border-bottom: 1px solid #d9d9d9;    display: inline-block;    color: #999;    padding: 2px;">图3.3 NFA模型</div> </center>

注意，NFA照比图3.2少了很多边。观察转换表不难发现，很多情况下转换函数的结果是空集。这意味着NFA可能存在无法前进的情况。

ε-NFA是一种特殊的NFA。在NFA的基础上，修改$\delta$的定义，$\delta$变为从$S\times(\Sigma\cup\{\epsilon\})$映射到$2^S$的函数，换言之，允许增加一些ε边，使得ε-NFA可能做到不读取有效输入字符串自发地在某些状态之间调整。

##### DFA和NFA和ε-NFA的等价性

对于任何DFA，存在一个特定的NFA，它们接收/定义了相同的语言；反之亦然。因此DFA和NFA在定义语言的层面上讲具有等价性。类似地NFA和ε-NFA也具有等价性。

#### 从正则表达式到DFA的转换

从给定的RE直接转换乘DFA略有困难，一般先将RE转换成NFA，再构造DFA。

从RE构造NFA的技巧：

1. 字母表中每个字符也是一个RE，可以画出一个简单的只有起始和终止状态的NFA。
2. 出现两个RE连接的情况，只需将两个RE各自的NFA首尾相连即可。
3. 出现两个RE或运算的情况，将两个RE各自的NFA”并联“起来即可。
4. 出现Kleene闭包的情况，将RE首尾循环连接起来即可。

从NFA构造DFA的技巧：

若原NFA形式化表示为：$M = (S, \Sigma, \delta, s_0, F)$构造一个新的DFA，这个DFA的状态集合为$2^S$，字符集合不变，初始状态为$\{s_0\}$，终结状态集合为$\{P|P \subseteq 2^S 且 P \cap F \neq \emptyset\}$，而
