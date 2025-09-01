# 03-上下文无关文法

## Context-Free Grammar (CFG) 上下文无关文法

* 四元组$$G=(N,T,S,P)$$
  * $$N$$ 非终结符的有限集合：还能继续推导的符号
  * $$T$$ 终结符的有限集合：不能继续推导的符号
  * $$S\in N$$ 开始符号
  * $$P$$ 产生式的有限集合，格式为$$A\to\alpha$$，其中$$A\in N$$，$$\alpha\in(N\cup T)^*$$
* 推导：$$A\to\alpha$$，$$uAv\Rightarrow u\alpha v$$
  * 连续推导：$$\rightarrow^*$$
* 上下文无关语言：$$L(G)=\{w\in T^*|S\Rightarrow^*w\}$$
* 最左/右推导：每次替换最左/右边的非终结符
* 最左/右句型：最左/右推导直到没有非终结符为止
* 语法树：根据推导过程建立
  * 叶子节点是终结符
  * 非叶子节点是非终结符
* 句柄：在推导时被替换的部分
  * 最左/右句型的句柄：上一次推导中被替换掉的，最左/右部分只有终结符

### 歧义问题

* Ambiguity 歧义：一个字符串对应多个语法树
* 无法抑制歧义
* 没有算法可以判断一个文法是否有歧义
* 解决方法：根据运算符优先级设置产生式，将优先级高的产生式用其他产生式替换

{% hint style="info" %}
以下产生式：

* $$E\to EOE$$
* $$E\to (E)$$
* $$E\to v|d$$
* $$O\to +|*$$
* 最左推导 $$v * (v+d)$$：$$E\Rightarrow EOE\Rightarrow vOE\Rightarrow v*E\Rightarrow v*(E)\Rightarrow v*(EOE)\Rightarrow v*(vOE)\Rightarrow v*(v+E)\Rightarrow v*(v+d)$$
* 最右推导 $$v * (v+d)$$：$$E\Rightarrow EOE\Rightarrow EO(E)\Rightarrow EO(EOE)\Rightarrow EO(EOd)\Rightarrow EO(E+d)\Rightarrow EO(v+d)\Rightarrow E* (v+d)\Rightarrow v*(v+d)$$
{% endhint %}

## Pushdown Automata (PDA) 下推自动机

* 上下文无关语言对应 PDA = NFA + 栈
* 状态转移格式：$$a, b\to c$$
  * $$a$$ 是输入符号
  * $$b$$ 是`pop`进栈的串
  * $$c$$ 是`push`出栈的串
* 定义：七元组$$M=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$$
  * $$Q$$ 状态集合
  * $$\Sigma$$ 输入符号集合
  * $$\Gamma$$ 栈上的符号
  * $$\delta：Q \times (\Sigma \cup \{\epsilon\}) \times \Gamma \to 2^{Q \times \Gamma^*}$$ 状态转移函数
  * $$q_0$$ 初始状态
  * $$z_0$$ 初始栈符号
  * $$F \in Q$$ 终止状态集合
* 下推自动机在某一时刻的状态：三元组$$(q, w, \gamma)$$
  * $$q$$ 是状态
  * $$w$$ 是暂未读取的输入串
  * $$\gamma$$ 是栈上的符号
* $$(q', \gamma')\in\delta(q, a, b)$$：在$$q$$状态，读取输入$$a$$，栈顶是$$b$$，可以转移到$$q'$$状态，弹出$$b$$，栈顶压入$$\gamma'$$
* 状态转移：$$(q, aw, \gamma)\vdash_M(q', w, \gamma')$$

### PDA 的语言

* 字符被 PDA 接受的条件
  * Acceptance by final state：到达终止状态，输入串消耗完毕，不关心栈的内容 $$L(M)=\{w\in\Sigma^*|(q_0, w, z_0)\vdash^*_M(q, \epsilon, \gamma), q\in F\,\gamma \in \Gamma^*\}$$
  * 或 Acceptance by empty stack：栈为空，输入串消耗完毕，不关心是否到达终止状态 $$N(M)=\{w\in\Sigma^*|(q_0, w, z_0)\vdash^*_M(q, \epsilon, \epsilon), q\in F\}$$
* 二者等价
  * 给定 L(M) 可构造出 N(M)
    * 在初始栈底增加$$x_0$$，防止清空栈
    * 在终止状态后可增加不断`pop`栈的状态转移，清空栈
  * 给定 N(M) 可构造出 L(M)
    * 在初始栈底增加$$x_0$$
    * 在栈内只剩$$x_0$$后增加状态转移到终止状态，同时`pop`掉$$x_0$$

## $$CFG = PDA$$

### $$CFG \subseteq PDA$$

* 任意的 CFG，$$G=(N,T,S,P)$$，可以构造一个以空栈为接受条件的 PDA $$M=(\{q\}, T, N\cup T, \delta, q, S, \emptyset)$$，其中
  * 对于任意非终结符$$A\in N$$，$$\delta(q, \epsilon, A)=\{(q, \epsilon)|A\to\epsilon\in P\}$$：如果栈顶值是非终结符，不读入任何符号，将栈顶替换来进一步推导
  * 对于任意终结符$$a \in T$$，$$\delta(q, a, a)=\{(q, \epsilon)\}$$：如果栈顶值是终结符，读入终结符，将栈顶弹出

### $$PDA \subseteq CFG$$

* 对于任何（以空栈作为匹配成功标准的）PDA $$M=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,\emptyset)$$，可以构造一个 CFG $$G=(N,T,S,P)$$，其中
  * $$N=\{S\}\cup\{N_{pXq}|p, q\in Q, X\in\Gamma\}$$
    * $$N_{pXq}$$ 表示从状态$$p$$到状态$$q$$，栈顶是$$X$$的所有可能
  * $$T=\Sigma$$
  * $$P$$
    1. 起始规则：$$S \rightarrow N_{q_0 z_0 p}, \quad \forall p \in Q$$
    2. 只出栈不压栈的情况：$$(q, \epsilon) \in \delta(p, a, X) \Rightarrow N_{pXq} \rightarrow a$$
    3. 既出栈又压栈的情况：$$(q, X_1X_2 \dots X_k) \in \delta(p, a, X) \Rightarrow N_{pXq} \rightarrow a N_{qX_1 p_1} N_{p_1X_2 p_2} \dots N_{p_{k-1}X_k p_k}$$

## DPDA

* Deterministic PDA：对于任意输入符号和栈顶符号，只有一个状态转移
* $$\text{DFA/NFA} \subseteq \text{DPDA} \subseteq \text{NPDA}$$
* 无歧义的 CFL 一定有 DPDA
* 有歧义的 CFL 只有 NPDA

## CFL 的性质

### 封闭性

* 若 $$L_1, L_2$$ 是 CFL，则下列一定是 CFL：
  * $$L_1 \cup L_2$$
  * $$L_1 L_2$$
  * $$L_1^*$$
  * $$L_1^R$$：匹配规则里面每一条都反过来
* 若 $$L_1, L_2$$ 是 CFL，则下列不一定是 CFL：（假设$$\{a^nb^nc^n\}$$不是 CFL，下证）
  * $$L_1 \cap L_2$$：$$L_1 = \{a^nb^nc^m\}, L_2 = \{a^mb^nc^n\}, L_1 \cap L_2 = \{a^nb^nc^n\}$$
  * $$\overline{L_1}$$：若$$\overline{L_1}$$是 CFL，则通过德摩根定理，$$L_1 \cap L_2$$ 也是 CFL
  * $$L_1 - L_2$$：若$$L_1 - L_2$$是 CFL，则$$\overline{L_1}=\Sigma^*-L_1$$也是 CFL
* CFL 和 RL 的交是 CFL

### 判定

* 给定 CFG，判断 CFL 是否为空：查看起始符号$$S$$是否能推导出终结符，$$S \to S$$ 是否在推导过程中出现
* 给定 CFG，判断 CFL 是否有限
  * 去除无用非结束符
  * 去除 $$\epsilon$$-产生式
  * 去除单一产生式
  * 画依赖图，判断是否有环
* 给定 CFG，判断自字符串是否在 CFL 中：
  1. 构造 NPDA，判断是否接受
  2. CYK 算法：$$O(n^3)$$，后续章节详述
* 无法判定的问题
  * 是否有歧义
  * 是否继承了歧义
  * 交集是否为空
  * 两个 CFL 是否相等
  * 是否等于$$\Sigma^*$$

## CFL 的泵引理

### Chomsky 范式

* 可以将 CFG 转换为 Chomsky 范式
  * $$A\to BC$$
  * $$A\to a$$
  * $$S\to\epsilon$$
* 解析树是二叉树
* 对于 CFG $$G=(N,T,S,P)$$
  * 假设 $$|N|=m, n=2^m$$
  * 对于任意字符串 $$z (|z|\geq n)$$，解析树的叶子数量是 $$|z|$$
  * 设最长的路径为$$A_0A_1\dots A_ka, k\leq \log_2|z| = |N|$$
  * 路径上面有重复的非终止符$$A_i=A_j,k-m \leq i < j \leq k$$
  * $$z=uvwxy$$，其中$$vwx$$由$$A_i$$产生
  * 对于任意的$$i \leq 0$$, $$uv^iwx^iy$$也是 CFG 的产生式

### 泵引理

* 对于任意 CFL $$L$$，存在一个常数$$n$$，对于任意长的字符串$$z\in L$$，$$|z|\geq n$$，$$z$$可以分解为$$z=uvwxy$$，满足
  * $$|vwx|\leq n$$
  * $$vx \ne \epsilon$$
  * $$\forall i\geq 0$$，$$uv^iwx^iy\in L$$
* 用泵引理证明某个语言不是 CFL:找到$$uv^iwx^iy$$不在语言中
