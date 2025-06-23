# 14-指针分析

* 别名 Alias：两个符号指向同一个内存地址
* 指针分析 Pointer Analysis：分析程序中指针的别名关系
* 目的：确定哪些指针可能指向同一内存位置，以便进行优化

## 指针别名分析

* May analysis：确定哪些指针可能指向同一内存位置，相当于 Must-not analysis（不满足 May analysis 的指针一定不别名）
  * Sound/Over-approximation：粗略判定，可能包含并非别名的别名关系
* Must analysis：确定哪些指针一定指向同一内存位置

### 流敏感的指针分析

* 开销过大

### 流不敏感的指针分析

```c
y = &x; // 取地址
y= x; // 赋值
*y = x; // 解引用，store
y = *x; // 取值，load
```

* `pts(x)`，`x`作为指针，可能指向的内存位置集合

#### Andersen 算法

| 语句形式     | 简写                                         | 约束                                                                     |
| -------- | ------------------------------------------ | ---------------------------------------------------------------------- |
| `y = &x` | $$\text{pts}(y) \supseteq \{x\}$$          | $$\text{pts}(y) \supseteq \{x\}$$                                      |
| `y = x`  | $$\text{pts}(y) \supseteq \text{pts}(x)$$  | $$\text{pts}(y) \supseteq \text{pts}(x)$$                              |
| `*y = x` | $$\text{pts}(*y) \supseteq \text{pts}(x)$$ | $$\forall v \in \text{pts}(y): \text{pts}(v) \supseteq \text{pts}(x)$$ |
| `y = *x` | $$\text{pts}(y) \supseteq \text{pts}(*x)$$ | $$\forall v \in \text{pts}(x): \text{pts}(y) \supseteq \text{pts}(v)$$ |

* 图
  * 节点：变量/内存位置
  * 边：$$y \to x \quad \text{代表} \quad \text{pts}(x) \supseteq \text{pts}(y)$$
* 通过动态维护传递闭包来添加新约束

#### Steensgaard 算法

| 语句形式     | 简写                                 | 约束                                                             |
| -------- | ---------------------------------- | -------------------------------------------------------------- |
| `y = &x` | $$\text{pts}(y) \supseteq \{x\}$$  | $$\text{pts}(y) \supseteq \{x\}$$                              |
| `y = x`  | $$\text{pts}(y) = \text{pts}(x)$$  | $$\text{pts}(y) = \text{pts}(x)$$                              |
| `*y = x` | $$\text{pts}(*y) = \text{pts}(x)$$ | $$\forall v \in \text{pts}(y): \text{pts}(v) = \text{pts}(x)$$ |
| `y = *x` | $$\text{pts}(y) = \text{pts}(*x)$$ | $$\forall v \in \text{pts}(x): \text{pts}(y) = \text{pts}(v)$$ |

* 图
  * 节点：变量/内存位置
  * 边：$$y \to x \quad \text{代表} \quad x \in \text{pts}(y)$$
  * 一个节点只有一条出边：转化为**并查集**，对于`=`直接合并父节点
  * 可能会遗漏一些别名关系

| 特性      | Steensgaard   | Anderson 分析  |
| ------- | ------------- | ------------ |
| **健全性** | ✅             | ✅            |
| **效率**  | 几乎线性          | 接近立方         |
| **精度**  | 合一式，较不精确（欠拟合） | 包含式，更精确（过拟合） |

* 两者都是 May analysis

## 基于 Datalog 的分析

### 逻辑范式编程语言

* 定义一系列 Facts 和 Rules
* 通过查询（Query）来推导新的 Facts

### Datalog

* Prolog 的子集
* 规则顺序无关
* 非图灵完备

```datalog
.decl rainy (c:symbol)
.decl cold(c:symbol)
.decl snowy(c:symbol)
.output snowy

rainy(“Nanjing”) % predicate
rainy(“Beijing”)
cold(“Beijing”)
snowy(c) :- rainy(c), cold(c)
```

* Predicate：谓词，n-元关系
* Rule：规则，`h :- b1, b2, ..., bn`，表示如果 $$b_1, b_2, ..., b_n$$ 都为真，则 $$h$$ 也为真
  * Rule 必须对所有取值都成立
  * 规则可以递归
  * 规则中可以出现 $$\lnot$$
* 所有未定义内容为假

### Datalog 分析 Reaching Definitions

* `def(B,N,X)`：表示在基本块 `B` 中，变量 `X` 在第 `N` 条语句被定义
* `succ(B,N,C)`：表示基本块 `C` 的后继是有 `N` 条语句的基本块 `B`
* `rd(B,N,C,M,X)`：变量 `X` 在基本块 `C` 的第 `M` 条语句被定义，并且可到达基本块 `B` 的第 `N` 条语句

```datalog
rd(B, N, B, N, X) :- def(B, N, X) % 变量定义

rd(B, N, C, M, X) :- rd(B, N-1, C, M, X), def(B, N, Y), X≠Y % 赋值无关变量

rd(B, 0, C, M, X) :- rd(D, N, C, M, X), succ(D, N, B) % 基本块的后继
```

* 基于控制流图，定义`def`和`succ`谓词
* 查询`rd`时，求解器会自动基于`def`和`succ`进行推导

### Datalog 分析 Pointer Analysis

原理类似，略

## Object Sensitivity

* 面向对象语言的 Context-Sensitivity
* 区分对象的方法调用
