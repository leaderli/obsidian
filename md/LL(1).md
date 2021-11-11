## 预测分析法

从[[语言及文法#文法|文法]] [[语言及文法#开始符号（start symbol）|开始符号]]出发，在每一步 [[语言及文法#推导|推导]] 过程中根据当前[[语言及文法#句型|句型]]的最左[[语言及文法#非终结符|非终结符]]A和当前输入符号$\alpha$ ，选择正确的A-[[语言及文法#产生式|产生式]]。为保证分析的确定性，选出的[[语言及文法#候选式|候选式]]必须是唯一的。

## S_文法
简单的确定性[[语言及文法#文法|文法]]
- 每个[[语言及文法#产生式|产生式]]的右部都以[[语言及文法#终结符|终结符]]开始
- 同一个[[语言及文法#非终结符|非终结符]]的各个[[语言及文法#候选式|候选式]]的[[#首终结符]]都不相同
- S_文法不包含 $\varepsilon$[[语言及文法#产生式|产生式]]，当某[[语言及文法#非终结符|非终结符]]A与档期输入符 $\alpha$ 不匹配时，若存在 $A\to\varepsilon$，可以通过检测 $\alpha$ 是否可以出现在A的后面，来决定是否可以使用[[语言及文法#产生式|产生式]] $A\to\varepsilon$ （若[[语言及文法#文法|文法]]中无$A\to\varepsilon$ 则应报错

例如：

![[Pasted image 20211031212452.png]]

## 非终结符的后续符号集

可能在某个[[语言及文法#句型|句型]]中紧跟在A后边的[[语言及文法#终结符|终结符]] $\alpha$ 的集合，记作FOLLOW(A)

$$
FOLLOW(A) = \lbrace\alpha\mid S \Rightarrow^*\alpha A \alpha \beta,\alpha \in V_T,\alpha,\beta \in (V_T \cup V_N)^*\rbrace
$$

例如

![[Pasted image 20211031213621.png]]


## 产生式的可选集

[[语言及文法#产生式|产生式]]$A\to\beta$ 的可选集是指可以选用该[[语言及文法#产生式|产生式]]进行[[语言及文法#推导|推导]]时对应的输入符号的集合，记作$SELECT(A\to\beta)$
- $SELECT(A\to\beta) = \lbrace\alpha\rbrace$
-  $SELECT(A\to\varepsilon) = FOLLOW(A)$

## q_文法

每个[[语言及文法#产生式|产生式]]的右部或为 $\varepsilon$ 或以[[语言及文法#终结符|终结符]]开始
具有相同左部的[[语言及文法#产生式|产生式]]有不相交的可选集
不含右部以[[语言及文法#非终结符|非终结符]]打头的[[语言及文法#产生式|产生式]]

## 首终结符

串首第一个符号，并且是[[语言及文法#终结符|终结符]]，也叫做串首终结符集

## 串首终结符集

给定一个文法符号串 $\alpha$ ，$\alpha$ 的串首终结符集 $FIRST(\alpha)$ 被定义为可以从 $\alpha$ 推导出的所有串首终结符构成的集合。如果 $\alpha \Rightarrow^* \varepsilon$  那么 $\varepsilon$ 也在 $FIRST(\alpha)$ 中

$$
\forall_\alpha \in (V_T \cup V_N)^+ , FIRST(\alpha) = \lbrace \alpha\mid\alpha \Rightarrow^* \alpha\beta,\alpha \in V_T,\beta \in (V_T \cup V_N)^* \rbrace;

$$

$$
\alpha \Rightarrow ^* \varepsilon , \varepsilon \in FIRST(\alpha)
$$


[[语言及文法#产生式|产生式]] $A \to \alpha$ 的可选集 $SELECT$

如果 $\varepsilon \notin FIRST(\alpha)$ ，那么 $SELECT(A\to\alpha)=FIRST(\alpha)$
如果 $\varepsilon \in FIRST(\alpha)$ ，那么 $SELECT(A\to\alpha)=(FIRST(\alpha)-\varepsilon) \cup FOLLOW(A)$


## LL(1)文法

[[语言及文法#文法|文法]] G 是 LL(1)的，当且仅当  G 的任意两个具有相同左部的产生式 $A\to\alpha\mid\beta$ 满足下面的条件：

- 如果 $\alpha$ 和 $\beta$ 均不能推导出 $\varepsilon$ ，则 $FIRST(\alpha) \cap FIRST(\beta)= \emptyset$
-  $\alpha$ 和 $\beta$ 至多有一个能推导出 $\varepsilon$ 
-  如果$\beta \Rightarrow^*\varepsilon$ ，则$FIRST(\alpha) \cap FOLLOW(A)= \emptyset$
-  如果$\alpha \Rightarrow^*\varepsilon$ ，则$FIRST(\beta) \cap FOLLOW(A)= \emptyset$

1. 同一[[语言及文法#非终结符|非终结符]]的各个[[语言及文法#产生式|产生式]]的[[#产生式的可选集|可选集]]互不交互
2. 可以为 LL(1) [[语言及文法#文法|文法]]构建预测分析器
3. 第一个 `L` 表示从左向右扫描输入
3. 第二个 `L` 表示产生最左[[语言及文法#推导|推导]]
4. `1`表示在每一步只需要向前看一个输入符号来决定语法分析动作