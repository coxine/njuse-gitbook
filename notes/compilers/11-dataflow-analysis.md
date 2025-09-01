# 11-数据流分析

* 静态分析，机器独立
* 优化源
  * 源码中的重复部分
    * 块内重复 Local
    * 块间重复 Global
  * 复制传播 Copy Propagation
  * 强度折减 Strength Reduction：使用成本更加低的操作
  * 消除循环变量：Induction-variable elimination

## Data Flow Fact

* 程序中状态的抽象描述
* 对于变量`x`
  * 若`x`的取值有限，则为`x`的取值集合`{1,2,3}`
  * 若`x`的取值无限，则为`NAC`
* $$\text{IN}[B]$$：块`B`执行前的 DFF
* $$\text{OUT}[B]$$：块`B`执行后的 DFF
* $$\text{OUT}[B]=f_B(IN[B])$$，$$f$$为转移函数，若`B`有多条语句，则以函数复合的方式计算
* $$\text{IN}[B]=\land_{P \in pred(B)}OUT[P]$$，$$\land$$为合并函数，合并多个路径的 DFF
* 进行数据流分析需要：$$f_B$$ $$\land$$ 每个块的初始 DFF

### Worklist Algorithm

* 对于以下的数据流分析，都可使用工作列表算法

```pseudocode
def worklist_algorithm:
    worklist = all blocks
    for block B in CFG:
        IN[B] = INIT
        OUT[B] = INIT

    while worklist is not empty:
        B = worklist.pop()
        IN[B] = ∧_{P \in pred(B)}OUT[P]
        OUT[B] = f_B(IN[B])
        if OUT[B] changed:
          for each successor S of B:
                worklist.add(S)
```

### 定义可达性分析 Reaching Definitions

* 是否存在一条路径，变量在路径上未被重新定义（kill）
* $$\text{gen}_B$$：块`B`中定义的变量
* $$\text{kill}_B$$：块`B`中被重新定义的变量
* $$\text{OUT}[B]=\text{gen}_B \cup (IN[B]-\text{kill}_B)$$
* $$\text{IN}[B]=\bigcup_{P \in pred(B)}OUT[P]$$（使用并，要包含所有路径的定义）

### 可用表达式分析 Available Expressions

* 应用：直接复用相关结果
* 定义：表达式 `x op y` 在 P 是可用的
  * 每一条到入口的路径上前都评估（计算）了该表达式
  * 从上一次评估到 P 之间，`x`和`y`没有被重新定义
* $$\text{e\_gen}_B$$：块`B`中产生的表达式
* $$\text{e\_kill}_B$$：块`B`中因为变量重新定义导致不可用的表达式
* $$\text{OUT}[B]= \text{e\_gen}_B \cup (IN[B]-\text{e\_kill}_B)$$
* $$\text{IN}[B]=\bigcap_{P \in pred(B)}OUT[P]$$（使用交，有一条路径改变了结果就无法复用）

### 活跃变量分析 Live Variable Analysis

* 应用：寄存器分配
* 后向分析：从程序末尾开始分析，$$\text{IN}$$ 还是块上面的 DFF，$$\text{OUT}$$ 还是块下面的 DFF
* $$\text{use}_B$$：块`B`中在赋值前使用的变量
  * 若变量同时出现在表达式两侧，视为 use
* $$\text{def}_B$$：块`B`中被赋值的变量/使用后赋值的变量
* $$\text{IN}[B]= \text{use}_B \cup (OUT[B]-\text{def}_B)$$
* $$\text{OUT}[B]= \bigcup_{P \in succ(B)}IN[P]$$

### 总结

|        | 定义可达性分析 Reaching Definitions              | 可用表达式分析 Available Expressions            | 活跃变量分析 Live Variable Analysis                   |
| ------ | ----------------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| 作用域    | 变量定义                                      | 表达式                                      | 变量                                              |
| 方向     | 前向分析                                      | 后向分析                                     | 前向分析                                            |
| 转移函数   | $$\text{gen}_B \cup (x - \text{kill}_B)$$ | $$\text{use}_B \cup (x - \text{def}_B)$$ | $$\text{e\_gen}_B \cup (x - \text{e\_kill}_B)$$ |
| 合并函数 ∧ | $$\cup$$                                  | $$\cup$$                                 | $$\cap$$                                        |
| 初始函数   | $$\text{OUT}[B] = \emptyset$$             | $$\text{IN}[B] = \emptyset$$             | $$\text{OUT}[B] = U$$                           |

### 块的最佳顺序

* 使用深度优先遍历，生成深度优先树
* 深度优先排序：深度优先树的后序遍历反转后的顺序
  * 有向无环图的拓扑排序
  * 有环图的类拓扑排序

### 迭代是否能终止

* 问题相当于$$\text{IN}[B]$$和$$\text{OUT}[B]$$能否收敛

#### 半格

* 集合$$V$$和偏序关系$$\land$$，使得
  * $$x \land x = x$$
  * $$x \land y = y \land x$$
  * $$x \land (y \land z) = (x \land y) \land z$$
  * $$x \land \top = x$$
  * $$x \land \perp = \perp$$
* 对于集合内各元素的运算，形成了半格，有“上界”
* 所以只要状态转移函数是单调增/减到一个极限，就能收敛
* $$\text{IN}[B]=\bigcup_{P \in pred(B)}OUT[P]$$显然是单调的
* $$\text{OUT}[B]=f_B(IN[B])$$是单调的
  * $$x \subseteq y \Rightarrow \text{gen} \cup (x - \text{kill}) \subseteq \text{gen} \cup (y - \text{kill}) \Rightarrow f_B(x) \subseteq f_B(y)$$

## 部分冗余消除

### 冗余

* 公共子表达式：从先前的计算到现在，表达式的值未改变
* 循环不变代码
* 部分冗余：一条路径上的表达式是冗余的，但另一条路径上不是

### Lazy Code Motion Problem

* 消除冗余时创建了新的代码块/重复代码
* Lazy Code Motion Problem：尽量减小消除冗余带来的额外开销

#### 要求

* 在消除重复计算的同时，不创建新的代码块：不能破坏原有的控制流
* 不能进行原程序所有计算以外的计算
* 表达式尽可能推迟计算，直到需要前再计算：减少寄存器占用

#### 操作方法

* 详见龙书P415

| 类型   | Anticipated Expr 预期表达式                                                | Available Expr 可用表达式                                            | Postponable Expr 可推迟表达式                                         | Used Expr 已使用表达式                                                |
| ---- | --------------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------- |
| 定义   | 当从 `p` 出发的所有路径上最终都计算了表达式 `b+c` 的值，且 `b,c` 的值已被定义，则 `b+c` 在 `p` 是预期表达式 | 当表达式 `b+c` 在到达 `p` 的所有路径上都是预期表达式，则 `b+c` 在 `p` 是可用表达式           | 表达式和使用结果的地方存在距离                                                 | 只用过一次的“中间变量”                                                    |
| 方向   | 后向分析                                                                  | 前向分析                                                            | 前向分析                                                            | 后向分析                                                            |
| 操作   | 把预期表达式替换成变量                                                           | 把可用表达式替换成变量                                                     | 移动表达式                                                           | 消除中间变量                                                          |
| 初始函数 | $$\text{IN}[B] = U$$                                                  | $$\text{OUT}[B] = U$$                                           | $$\text{OUT}[B] = U$$                                           | $$\text{IN}[B] = \emptyset$$                                    |
| 转移函数 | $$e.\text{use}_B \cup (x - e.\text{kill}_B)$$                         | $$(\text{anticipated}[B].\text{in} \cup x) - e.\text{kill}_B$$  | $$(\text{earliest}[B] \cup x) - e.\text{use}_B$$                | $$(e.\text{use}_B \cup x) - \text{latest}[B]$$                  |
| 合并函数 | $$\text{OUT}[B] = \bigcap_{S \in \text{succ}(B)} \text{IN}[S]$$       | $$\text{IN}[B] = \bigcap_{P \in \text{pred}(B)} \text{OUT}[P]$$ | $$\text{IN}[B] = \bigcap_{P \in \text{pred}(B)} \text{OUT}[P]$$ | $$\text{OUT}[B] = \bigcup_{S \in \text{succ}(B)} \text{IN}[S]$$ |

* $$\text{earliest}[B] = \text{anticipated}[B].\text{in} - \text{available}[B].\text{in}$$
* $$\text{latest}[B] = (\text{earliest}[B] \cup \text{postponable}[B].\text{in}) \cap \left( e.\text{use}_B \cup \neg \left( \bigcap_{S \in \text{succ}(B)} (\text{earliest}[S] \cup \text{postponable}[S].\text{in}) \right) \right)$$

## 常量传播

* 特点
  * 有无限多的取值集合：`UNDEF`、`NAC`、所有可能的值
  * 不能分配：$$f(a \land b) \neq f(a) \land f(b)$$
* `x = y`
  * `y`是常量，则`x`也是常量
  * `y`未定义，则`x`是未定义
  * 否则`x`是`NAC`
* `x = y op z`
  * `y`和`z`都是常量，则`x`也是常量
  * `y`或`z`未定义，则`x`是未定义
  * 否则`x`是`NAC`
* 合并函数
  * 半格，交是下界
  * `UNDEF` ^ `CONST` = `CONST`
  * `CONST` ^ `NAC` = `NAC`
  * `CONST` ^ `CONST` = `CONST`
  * `CONST` ^ `CONST‘` = `NAC`
* 转移策略
  * Meet-Then-Transfer：对于表达式，先合并各个路径的结果，然后再进行转移
  * Transfer-Then-Meet：对于表达式，先进行计算，然后再合并各个路径的结果（可以传播两个路径上可能`x` `y` 的值不同，但是`x op y`的值相同的情况）

## 稀疏 DFA

* 思想：只在必要的节点作分析
* Temporal sparcity：跳过那些不会改变分析结果的路径或节点
* Space sparcity：只在“需要”的程序点保留数据流信息，比如定义点、使用点，而不是每一行代码都维护一份完整的数据流状态。
* 和 SSA 紧密相关
