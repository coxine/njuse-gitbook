# 01-有限自动机&正则文法

## 基础概念

* Alphabet: $$\Sigma$$，有限的字符集合
* String: 由$$\Sigma$$中的字符组成的有限序列
  * 空串: $$\epsilon$$，长度为0的字符串
  * 子串
  * 前缀/后缀
* 字符串运算
  * Concatenation: $$w_1 w_2$$，$$w_1$$和$$w_2$$的连接
  * Reverse: $$w^R$$，字符串的逆序
  * Length: $$|w|$$，字符串的长度
  * Power: $$w^n$$，字符串的重复，$$w^0 = \epsilon$$
* Language: 字符串的集合
  * Kleene star: $$\Sigma^*$$，字母表$$\Sigma$$上的所有字符串的集合
  * Plus: $$\Sigma^+$$，字母表$$\Sigma$$上的所有非空字符串的集合
  * Language 是 $$\Sigma^*$$ 的非空子集（可以为$$\{\epsilon\}$$）
* 语言运算
  * 交并减补
  * Reverse：$$L^R = \{w^R | w \in L\}$$
  * Concatenation：$$L_1 L_2 = \{w_1 w_2 | w_1 \in L_1, w_2 \in L_2\}$$
  * Power：$$L^n = \{w_1 w_2 \cdots w_n | w_i \in L\}$$
  * Star-closure：$$L^* = \bigcup_{i=0}^{\infty} L^i$$
  * Positive-closure：$$L^+ = \bigcup_{i=1}^{\infty} L^i= L^* - \{\epsilon\}$$

{% hint style="info" %}
例题

$$L_= \{a, b\}, L^3 = \{aaa, aab, aba, abb, baa, bab, bba, bbb\}$$$$L_= \{a^nb^n:n \ge 0\}, L^2 = \{a^nb^n a^mb^m: n, m \ge 0\}$$
{% endhint %}

## Deterministic Finite Automata 有限自动机

* 有限自动机：输入字符串，输出接受/拒绝
  * 两个圆圈的状态表示接受，一个圆圈的状态表示拒绝
  * 箭头为状态转移
* DFA：确定性有限自动机，对于每个状态和输入字符，只有一个状态转移
  * $$M = (Q, \Sigma, \delta, q_0, F)$$
  * $$Q$$：状态集合
  * $$\Sigma$$：输入字母表
  * $$\delta$$：状态转移函数，$$\delta: Q \times \Sigma \mapsto Q$$
  * $$q_0 \in Q$$：初始状态
  * $$F \subseteq Q$$：接受状态集合
  * $$\delta^*: Q \times \Sigma^* \mapsto Q$$：状态转移函数的扩展，接受多个输入字符
* 使用 DFA 定义语言：$$L(M) = \{w \in \Sigma^* | \delta^*(q_0, w) \in F\}$$

### 【必考】 最小化 DFA

* 核心思想：将状态划分为集合，如果集合内两个状态在任何输入下都是等价的（输出状态所在集合相同），则保持不变，如果不等价，则分开
* 步骤
  1. 初始化：在去除不可达的状态后，将状态划分为接受和拒绝两个集合
  2. 找到不等价的状态，划分为新的集合
  3. 重复2，直到没有新的划分
* 时间复杂度：$$O(n \log \log n)$$

### DFA Bi-Simulation

* Bi-Simulation：两个状态的输入输出行为相同，两个 DFA 是等价的
* 判断步骤
  1. 找到两个 DFA 的初始状态作为待检查的状态
  2. 对于每一个输入，找到待检查状态转移后的状态，将其加入待检查状态
  3. 重复 2 ，直到没有新的状态被加入
  4. 所有状态对中，**如果两个状态同时为终止状态或同时为非终止状态，则两个 DFA 是等价的**

## Non-Deterministic Finite Automata 非确定性有限自动机

* 对于一个输入字符，可以有多个状态转移，只要有一个状态转移成功，就可以接受
* 允许 $$\epsilon$$ 转移
* 定义
  * $$M = (Q, \Sigma \cup \{\epsilon\}, \delta, q_0, F)$$
  * $$\delta: Q \times (\Sigma \cup \{\epsilon\}) \mapsto 2^Q$$ ($$2^Q$$ 表示 $$Q$$ 的幂集，所有的子集)
* $$\epsilon \text{-closure} (q)$$：从状态$$q$$开始，所有通过 $$\epsilon$$ 转移，不消耗其他字符可以到达的状态的集合
  * $$q \in \epsilon \text{-closure} (q)$$
  * 状态转移的拓展：匹配完成后，如果还能通过 $$\epsilon$$ 转移到达其他状态，则将这些状态也加入
    * $$\delta^*: Q \times \Sigma^* \mapsto 2^Q$$
    * $$\delta^*(q, w) = \bigcup_{q' \in \delta^*(q, a)} \epsilon \text{-closure} (q')$$
  * 使用 NFA 定义语言：$$L(M) = \{w \in \Sigma^* | \delta^*(q_0, w) \cap F \neq \emptyset\}$$

## 【必考】 DFA 和 NFA 转换

* DFA 显然是 NFA 的特例
* NFA 可以转换为 DFA
* 转换步骤：子集构造法
  1. 初始化 DFA 的初始状态集合为 $$S_0=\epsilon \text{-closure} (q_0)$$
  2. 对于每一个输入字符$$a$$，找到所有能到达的转移状态的集合$$S' = \bigcup_{q \in S} \delta(q, a)$$
  3. 将$$S''=\epsilon \text{-closure} (S')$$加入 DFA 的状态集合，如果转移状态中包含 NFA 的 Final 状态，则加入 DFA 的 Final 状态集合
  4. 重复，直到所有状态都被加入
* 最差时间复杂度：$$\Theta(2^n)$$

{% hint style="warning" %}
当 NFA 转换为 DFA 时，规模不一定会减小
{% endhint %}
