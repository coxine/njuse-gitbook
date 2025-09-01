# 04-语法分析

## 自顶向下

* 从文法的开始符号出发，通过推导和归纳，逐步推导出句子的结构
* 对应**先序遍历**：从根节点出发，先访问根节点，然后递归访问左右子树
* 递归下降
* 可能出现的问题
  * 回溯：当一种匹配方式失败时，需要回溯到之前的状态，用其他方式解析
  * 左递归：无限循环 `A -> Aa | b`

### 左递归的解决方案

* 变换公式：$$A \to A\alpha_1 | A\alpha_2 | \cdots | A\alpha_n | \beta_1 | \beta_2 | \cdots | \beta_m \Rightarrow$$
* $$A \to \beta_1A' | \beta_2A' | \cdots | \beta_mA'$$
* $$A' \to \alpha_1A' | \alpha_2A' | \cdots | \alpha_nA' | \epsilon$$

{% hint style="info" %}
$$A$$实际上是$$\beta_i | (\alpha_i)^*$$
{% endhint %}

* **任何语法都能转换为不包含环和**$$\epsilon$$ **production的等价语法**
* 解决环：代入消除环
* 解决$$\epsilon$$ production
  * 枚举$$\epsilon$$的时候，$$S$$的格式
  * 增加一个新的开始符号$$S'$$，$$S' \to S | \epsilon$$

> 例：$$S \to aSbS | \epsilon$$
>
> * $$S \to aSbS | aSb | abS | ab$$
> * $$S' \to S | \epsilon$$
>
> 例：$$A \to B | a, B \to A | b$$
>
> * $$A \to A | b | a$$
> * $$ightarrow A \to b | a$$ （最左边无法匹配，可直接消除）
> * $$ightarrow B \to a | b$$

### 预测分析法

* 无需回溯
* 适用于：LL(1) 文法
  * L：从左到右扫描
  * L：从最左推导
  * 1：使用 1 个符号的前瞻，决定使用哪个产生式
* 特点：没有左递归，没有歧义
  * 可以对每一个输入字符，构建预测表

#### 构建预测表

* 构建预测表需要 $$\text{FIRST}$$ 和 $$\text{FOLLOW}$$ 集合
* $$\text{FIRST}(\alpha)$$：一系列能作为$$\alpha$$开头的终结符
  * 若$$X$$是终结符，$$\text{FIRST}(X) = \{X\}$$
  * 若$$X \to \epsilon$$，$$\{\epsilon\} \in \text{FIRST}(X)$$
  * 若$$X \to Y_1Y_2\cdots Y_k$$，$$\epsilon \in \text{FIRST} \bigcap_{j=1}^{i-1}(Y_j) \land a \in \text{FIRST}(Y_i) \Rightarrow a \in \text{FIRST}(X)$$
* $$\text{FOLLOW}(\alpha)$$：一系列能跟在$$\alpha$$后面的终结符
  * 若$$S$$为开始符号，$$\$ \in \text{FOLLOW}(S)$$
  * $$A \to \alpha B \beta \Rightarrow \text{FIRST}(\beta) - \{\epsilon\} \subseteq \text{FOLLOW}(B)$$
  * $$A \to \alpha B$$或$$A \to \alpha B\beta, \epsilon \in \text{FIRST}(\beta) \Rightarrow \text{FOLLOW}(A) \subseteq \text{FOLLOW}(B)$$
* 构造规则：$$\forall A \to \alpha$$
  * 若$$a \in \text{FIRST}(\alpha)$$，填充$$M[A, a] = A \to \alpha$$
  * 若$$\epsilon \in \text{FIRST}(\alpha) \land b \in \text{FOLLOW}(A)$$，填充$$M[A, b] = A \to \alpha$$
* 根据表格匹配唯一的规则，若为空则匹配错误

#### LL(1) 文法的 正式定义

* 对于文法 $$A \to \alpha | \beta$$
  * 不能存在相同的首终结符：不存在$$a$$，使得$$\alpha, \beta$$能同时推导出$$a$$开头的串（$$\text{FIRST}(\alpha) \bigcap \text{FIRST}(\beta) = \emptyset$$）
  * 不能同时推导出空串
  * 无二义性：如果$$\beta$$能推出空串，则$$\alpha$$不能推导出$$\text{FOLLOW}(\alpha)$$ 里的终结符

#### 递归/非递归预测分析

* 递归：递归下降
* 非递归：显式地维护一个栈，用于存储当前的状态，相当于 PDA

## 自底向上 非重点

* 使用后序遍历建立语法树
* 不断将输入符号入栈，直到产生产生式右侧的符号
* 一旦栈顶符号序列和某个产生式右侧相同时，就不断规约，直至无法规约，随后继续入栈
* 问题
  * 应该规约还是继续入栈
  * 如果有多条匹配的规则，如何选择

### LR(0) 文法

* 在语法分析时不用做选择
* 定义
  * L：从左到右扫描符号流
  * R：从右到左规约
  * 0：不需要向前看任何符号，即可完成规约
* 表达式右边：可以使用 DFA 表示
  * 表达式左侧：初始状态
  * 状态转移：右侧的每一个符号
  * 可以用$$\epsilon$$变换，连接和状态转移的非终结符相同的初始状态
