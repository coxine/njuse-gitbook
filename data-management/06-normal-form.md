# 06-关系范式

## 基本定义

* 五元组$$R(U,D,DOM,F)$$
  * $$R$$：关系模式名
  * $$U$$：属性集
  * $$D$$：属性的域
  * $$DOM$$：$$U$$到$$D$$的映射
  * $$F$$：函数依赖集
* 可省略$$D$$，$$DOM$$，变为三元组$$R<U,F>$$
  * 数据库中$$D$$和$$DOM$$是明确的
* 当且仅当$$U$$上的关系$$R$$满足$$F$$时，$$r$$称为关系模式$$R<U,F>$$ 的一个关系（合法的关系）

## 数据依赖

### 函数依赖

* $$(X)\to Y$$，$$X$$函数确定于$$Y$$，$$Y$$函数依赖于$$X$$
  * $$X$$可以唯一确定$$Y$$，$$Y=f(X)$$
  * 不存在$$X$$的值相同而$$Y$$的值不同的情况
  * $$X$$是属性集
* $$X \to Y, Y \to X \Rightarrow X \leftrightarrow Y$$
* $$X \nrightarrow Y$$：$$Y$$不函数依赖于$$X$$
* $$R$$中的所有关系都要满足$$F$$中的所有函数依赖
* 函数依赖是语义范畴的概念
* 平凡/非平凡：$$Y$$是否是$$X$$的子集
  * 平凡的函数依赖：$$X \to Y$$，$$Y \subseteq X$$
  * 非平凡的函数依赖：$$X \to Y$$，$$Y \not\subseteq X$$
* 完全/部分函数依赖：$$X$$的子集能否唯一确定$$Y$$
  * 完全函数依赖：$$X \to Y$$，$$\forall Y \subset X$$，$$Y \nrightarrow Y$$，记作$$X \xrightarrow{F} Y$$
  * 部分函数依赖：$$X \to Y$$，$$\exists Z \subset X$$，$$Z \to Y$$，记作$$X \xrightarrow{P} Y$$
* 传递依赖：$$X \to Y, Y \to Z, Y \nrightarrow X, Z \nsubseteq Y$$，记作$$X \xrightarrow{传递} Z$$
  * 若$$X \to Y , Y \to Z, Y \to X$$，则$$Z$$直接依赖于$$X$$，不算传递函数依赖

> - 若 $$X$$ 和 $$Y$$ 是 1:1 映射关系，则 $$X \to Y$$ , $$Y \to X$$
>   * 若 $$X$$ 和 $$Y$$ 是 1:N 映射关系，则 $$Y \to X$$
> - 若 $$X$$ 和 $$Y$$ 是 N:M 映射关系，则 不存在函数依赖

### 闭包

* $$X^+$$：$$X$$的闭包，$$X$$能唯一确定的属性集
* 生成方法：递归检查依赖，直到没有新的属性加入为止

### 多值依赖

* 某个属性同时决定两组不相关的信息，两组信息可以任意组合，产生冗余
* 设$$R(U)$$是属性集$$U$$上的一个关系模式。$$X,Y,Z$$是$$U$$的子集，并且$$Z=U-X-Y$$。关系模式$$R(U)$$中多值依赖 $$X\twoheadrightarrow Y$$ 成立，当且仅当对$$R(U)$$的任一关系$$r$$，给定的一对$$(x,z)$$值，有一组$$Y$$的值，这组值仅仅决定于$$x$$值而与$$z$$值无关。
* 平凡的多值依赖：$$Z=\emptyset$$
* 对称性：$$X \twoheadrightarrow Y \Rightarrow X \twoheadrightarrow Z,\quad \text{其中 } Z = U - X - Y$$
* 传递性：$$X \twoheadrightarrow Y,\ Y \twoheadrightarrow Z \Rightarrow X \twoheadrightarrow Z - Y$$
* 函数依赖是特例：$$X \rightarrow Y \Rightarrow X \twoheadrightarrow Y$$
* 并规则：$$X \twoheadrightarrow Y,\ X \twoheadrightarrow Z \Rightarrow X \twoheadrightarrow YZ$$
* 交规则：$$X \twoheadrightarrow Y,\ X \twoheadrightarrow Z \Rightarrow X \twoheadrightarrow Y \cap Z$$
* 差规则：$$X \twoheadrightarrow Y,\ X \twoheadrightarrow Z \Rightarrow X \twoheadrightarrow Y - Z,\ X \twoheadrightarrow Z - Y$$

## 码

* $$K$$ 为 $$R<U,F>$$ 的一个属性集
* 候选码：可唯一确定元组的属性集
  * $$K \xrightarrow{F} U$$，$$K$$是$$R$$的一个候选码（最小超码）
* 超码：在候选码的基础上加其他属性
  * $$K \xrightarrow{P} U$$，$$K$$是$$R$$的一个超码（候选码的超集）
* 若$$R$$有多个候选码，选一个作为主码
* 主属性：候选码中出现的属性
* 非主属性/非码属性：不包含在任意候选码中的属性
* 全码：整个属性组都是码，缺一不可
* 外码：关系模式中的属性集$$K$$并非候选码，但$$K$$在另一个关系模式中是候选码

## 范式

* 满足某一级别的关系模式的集合
* $$1NF \supset 2NF \supset 3NF \supset BCNF \supset 4NF \supset 5NF$$
* $$R \in nNF$$：$$R$$满足$$nNF$$的所有条件
* 规范化：低一级范式通过模式分解转换成高一级范式模式的集合

### 1NF

* 每个分量都是不可分的原子值，不能出现多个项$$Y=\{a_1,a_2\}$$的情况
* 关系数据库的基本要求
* 缺点
  * 数据冗余 & 更新异常：必须要更改所有的，以免数据不一致
  * 插入异常：插入一个元组时，必须要插入所有的属性值，无法仅插入部分属性值
  * 删除异常：删除某个元组后，可能会丢失仅依赖于该元组的其他信息
* 解决方法：分解成多个关系模式，用规范化理论消除不合适的数据依赖

### 2NF

* 非主属性必须**完全依赖**于候选码
* 消除部分依赖
* 表里属性都和候选码相关
* 改善：数据冗余、更新异常（但是仍然存在）

```
# 满足 1NF 不满足 2NF 的例子
表格 (A, B, C, D)
A -> C
(A, B) -> D
# 解决方案
表格1 (A, C)
A -> C
表格2 (A, B, D)
(A, B) -> D
```

### 3NF

* 消除传递依赖
* 关系模式中没有传递依赖，所有非主属性都直接依赖于候选码
* 表里属性都和候选码直接相关
* 改善：插入异常、删除异常（但是仍然存在）
* 若所有属性都是主属性，则满足 3NF

```
# 满足 2NF 不满足 3NF 的例子
表格 (A, B, C)
A -> B
B -> C
# 解决方案
表格1 (A, B)
A -> B
表格2 (B, C)
B -> C
```

### BCNF

* 除了候选码本身，不允许任何非候选码的属性集参与决定关系中的其他属性
* 所有非主属性都完全依赖于每个候选码
* 所有主属性的完全依赖于每个不包含他的候选码
* 没有任何属性完全函数依赖于非码的属性
* 彻底消除插入异常、删除异常

```
# 满足 3NF 不满足 BCNF 的例子
C -> B  
(A, B) -> C
```

### 4NF 5NF

略。

### 判断流程

> 感谢 [https://www.bilibili.com/video/av741922043/](https://www.bilibili.com/video/av741922043/)

1. 求候选码、主属性、非主属性
2. 所有函数依赖的左边都是超码 -> BCNF
3. 左边非超码的函数依赖的右边是主属性 -> 3NF
4. 任意候选码的真子集都无法推出非主属性 -> 2NF
5. 所有属性都是原子值 -> 1NF
