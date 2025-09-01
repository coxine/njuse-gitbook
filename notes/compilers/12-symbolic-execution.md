# 12-符号执行

* 定义：将程序中的变量替换为符号值，执行程序以探索所有可能的路径。
  * 每个输入变量和一个符号关联
  * 每个表达式都是符号表达式
  * 程序的路径：以路径约束（一组条件表达式）描述分支处的路径选择
* 路径敏感 Path-Sensitive：程序中不同的执行路径会被分别分析、分别追踪。
* 目标：发现程序中的错误、漏洞或不符合预期的行为（如除零错误、越界访问等）。

## 检查断言：约束求解

* $$\sigma$$：记录符号值，如`𝝈 = { a -> 𝜶_a ; b -> 𝜶_b ; x -> 1 ; y -> 0 }`，在语句执行后改变
* $$\pi$$：路径约束，记录分支条件，如`𝜋 = { x > 0 ; y < 10 }`，初始为`true`，在每一次分支处改变
* 检查`assert(cond)`语句：构造违反条件：$$eg cond$$，如果$$\pi \land \neg cond$$为假（即在$$\pi$$条件下不可能发生违反条件），则不会失败

```c
int x = 1, y = 0;
if (a != 0) {
    y = 3 + x;
    if (b == 0) {
        x = 2 * (a + b);
    }
}
assert(x - y != 0);
```

* 下一步目标：检查诸如$$\pi \land \neg cond$$的条件判断式是否成立
* 使用**约束求解器** Constraint Solver 来判断是否满足条件
  * 可满足 Satisfiable：存在`assert`失败的可能性
  * 不可满足 Unsatisfiable：`assert`不会失败
  * 未知 Unknown：无法确定是否存在失败可能性（过于复杂，超时）

### 限制

* 可拓展性
  * Path Explosion：路径数量随着分支和循环的增加而指数级增长
  * External Calls：外部函数调用可能导致路径不确定
  * Constraint Solving：复杂的约束可能导致求解器无法处理
* 优化方法
  * Path/State Merging：合并相似路径或状态，减少路径数量
    * 引入`ite(cond, true_val, false_val)`函数，合并相似路径
  * Induction for Loops：对循环进行归纳，减少路径数量
    * 把循环内容转化为条件表达式（不一定始终可用）
  * Concolic Execution：结合符号执行和具体执行，运行时 生成符号值
  * Speculative Execution：预测路径，提前执行可能的分支

## 可满足性 Satisfiability

### 命题逻辑 Proposition Logic

* 只涉及命题变量和逻辑运算，不涉及具体变量的运算
* 真值符号（$$b$$）：$$\top$$ 和 $$\bot$$（真和假）
* 命题变量（$$v$$）：$$p$$、$$q$$、$$r$$ 等
* 文字（$$l$$）：原子或其否定
* 公式（$$F$$）：
  * $$l$$
  * $$\lnot F$$（否定）
  * $$F_1 \lor F_2$$（或）
  * $$F_1 \land F_2$$（与）
  * $$F_1 \Rightarrow F_2$$（蕴含）
  * $$F_1 \Leftrightarrow F_2$$（等价）
* 可满足 Satisfiable：**存在一种**真值赋值使得公式为真
* 有效 Valid：**对于所有**真值赋值，公式都为真
* 在命题逻辑中的所有公式$$F$$，$$F$$有效当且仅当$$\lnot F$$不可满足

## 范式 Normal Form

* 对公式语法的限制，把公式语法规范化为某种接口
* 有助于简化判定流程

### 否定范式 Negation Normal Form (NNF)

* 定义：不允许使用蕴含/等价（$$ightarrow \Leftrightarrow$$）运算符，只允许对变量取否（$$\lnot$$），不允许对更复杂的子公式取否
* 所有的公式都可以转换为 NNF

### 合取范式 Conjunctive Normal Form (CNF)

* 定义：合取范式（CNF）是一个公式，由若干“析取项（$$\lor$$）”之间通过合取（$$\land$$）连接组成。
  * 公式中只允许使用 $$\lor$$、$$\land$$ 和 $$\lnot$$，其中：
  * $$\lnot$$ 只能作用于原子命题变量
  * 最外层是合取（$$\land$$）。
  * 析取项中不应再包含 $$\land$$。
* 形式：$$(l_1 \lor l_2 \lor \ldots) \land (l_3 \lor \lnot l_4 \lor \ldots) \land \ldots$$
* 所有的公式都可以转换为 CNF

### 析取范式 Disjunctive Normal Form (DNF)

* 定义：析取范式（DNF）是一个公式，由若干“合取项（$$\land$$）”之间通过析取（$$\lor$$）连接组成。
  * 公式中只允许使用 $$\lor$$、$$\land$$ 和 $$\lnot$$，其中：
  * $$\lnot$$ 只能作用于原子命题变量
  * 最外层是析取（$$\lor$$）。
  * 合取项中不应再包含 $$\lor$$。
* 形式：$$(l_1 \land l_2 \land \ldots) \lor (l_3 \land \lnot l_4 \land \ldots) \lor \ldots$$
* 所有的公式都可以转换为 DNF

## Tseitin Transformation

### 同时可满足性 Equisatisfiability

* 定义：两个公式$$F_1$$ 和 $$F_2$$ 是同时可满足的，当且仅当 $$F_1$$ 可满足等价于 $$F_2$$ 可满足
  * 说人话：两个公式要么都是可满足的，要么都是不可满足的
* 对于任意公式$$F$$，可以构造一个同时可满足的 CNF 范式公式$$F'$$，且 $$F'$$ 的大小倍数是常数级别的

### 具体实现

1. 建立语法树，把每一个叶子节点对应的表达式$$F_i$$替换为一个新的变量$$x_i$$
2. 根节点 $$F_1$$ 转换为 $$x_1 \land (x_1 \Leftrightarrow F_1)$$
3. 随后在式子后面加上$$\land (x_i \Leftrightarrow F_i)$$，直到所有的叶子节点都被替换
4. 通过下面大一离散课的知识，把所有的$$\Leftrightarrow$$ 和 $$ightarrow$$ 替换为 CNF 形式的合取范式

#### 离散课回忆

| 原始逻辑形式                  | 用与/或/非表示                                    |
| ----------------------- | ------------------------------------------- |
| $$A \Rightarrow B$$     | $$\lnot A \lor B$$                          |
| $$A \Leftrightarrow B$$ | $$(A \lor \lnot B) \land (\lnot A \lor B)$$ |

## SAT 问题

* 找到是否存在能让$$F$$ 为真的变量赋值
* 解决方法
  * 通过 Tseitin Transformation 转换为 CNF
  * 使用 DPLL 算法（Davis-Putnam-Logemann-Loveland）

### DPLL 算法

* 选择一个变量 $$v$$，赋值为真/假
* 传播这个变量的值
* 优化
  * $$(a_0 \lor a_1 \lor \ldots \lor a_n \lor c) \land (b_0 \lor b_1 \lor \ldots \lor b_m \lor \lnot c)$$ 可转化为 $$(a_0 \lor a_1 \lor \ldots \lor a_n \lor b_0 \lor b_1 \lor \ldots \lor b_m)$$

```c
bool DPLL (F, I) {
  if (F == false) return UNSAT;
  if (F == true) return SAT;
  
  p = choose(F);
  bool ret = DPLL(F[pàtrue], I[pàtrue])
  if (ret == SAT) return SAT;
  
  return DPLL(F[pàfalse], I[pà false]);
}
```

### CDCL 算法

* Conflict-Driven Clause Learning
* 从矛盾中学习，优化算法速度
* 略，来不及复习了

## 一阶逻辑 First-Order Logic (FOL)

* 定义：由对象（objects）和它们之间的关系（relations）构成的逻辑系统，可以表达“某个对象”、“对象之间的关系”和“对象上的函数”等更复杂结构。
  * 对象符号：$$a, b, c, \ldots$$，表示具体的对象或个体。
  * $$n$$-元谓词：$$p(t_1, t_2, \ldots, t_n)$$，输入 $$n$$ 个对象，输出一个真假值（True/False）。
  * $$n$$-元函数：$$f(t_1, t_2, \ldots, t_n)$$，输入 $$n$$ 个对象，输出一个关系。
  * 公式：通过连接词和量词构造的逻辑表达式。
    * $$\top$$ $$\bot$$
    * 原子公式 $$t$$
    * $$eg, \lor, \land, \Rightarrow, \Leftrightarrow$$
    * $$\forall x. F$$ $$\exists x. F$$
* Theory of Bit Vectors：所有数据都是有长度的位向量，把运算看作位向量上的逻辑运算，可以将 FOL 转化为 PL

### Satisfiability Modulo Theories

1. 词级别处理，在词层面化简逻辑公式
2. Bit blasting：将公式转换为位向量的形式，把 FOL 转化为 PL
3. 问题转化为 SAT 问题，使用 DPLL 或 CDCL 算法求解
