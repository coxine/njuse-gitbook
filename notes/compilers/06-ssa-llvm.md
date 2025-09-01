# 06-SSA LLVM

## 静态单赋值 Static Single Assignment (SSA)

* 变量只能被定义（赋值）一次
* 多次赋值时，使用后缀来表示：`a1`, `a2`, `a3` 等
* 便于优化：def-use chains 是明确的
* 使用 $$\phi$$ 函数在分支合并点合并不同路径的变量

### Gated SSA

* 变量只能被定义（赋值）一次
* 使用递归的 $$\gamma$$ 函数在分支合并点合并不同路径的变量（$$\gamma$$函数的参数可以是$$\gamma$$函数的返回值）
* 使用 $$\mu$$ $$\eta$$ 表示循环变量

```c
if (flag) 
  x = 1;
else
  x = 2;
```

```ssa
if (flag) 
  x1 = 1;
else
  x2 = 2;
x3 = φ(x1, x2);
```

```gsa
if (flag) 
  x1 = 1;
else
  x2 = 2;
x3 = γ(flag, x1, x2);
y = x3 + 1;
```

### 时空复杂度

* 从三地址码到 SSA：时间 $$O(n)$$ 空间 $$O(n)$$

## LLVM IR

* Module
  * Types
  * Globals
  * Functions
    * Function
      * BasicBlock
        * Instruction
  * Metadata

```bash
sudo apt install clang llvm
clang -c -o hello hello.c
clang -c -S -emit-llvm hello.c -o hello.ll
clang -c -emit-llvm hello.c -o hello.bc
llvm-dis hello.bc -o hello.ll
llvm-as hello.ll -o hello.bc
```

* SSA 风格的 IR

```llvm
define i32 @factorial(i32 %0) {
  1:
  %2 = icmp eq i32 %0, 0 ## %2 为 bool(i1)，未显式声明
  br i1 %2, label %7, label %3
  3:
  %4 = add i32 %0, -1
  %5 = call i32 @factorial(i32 %4)
  %6 = mul i32 %5, %0
  br label %7
  7:
  %8 = phi i32 [ %6, %3 ], [ 1, %1 ] ## 如果从 %1 来，则 %8 = 1；如果从 %3 来，则 %8 = %6
  ret i32 %8
}
```

* LLVM 类型
  * `void`
  * `i1` `i8` `i16` `i32` `i64`
  * `float` `double`
  * 函数：`return type(param types)`
  * 指针：`i32*` `i8**` `ptr`(void\*) `[40 x i32]*`(指向数组的指针)
  * 数组：
    * `[40 x i32]`
    * `[40 x i32*]` ：包含40个指针的数组
  * 结构体：`{ i32, i8 }`
* 内置函数
  * 作为指令的拓展，可直接在 IR 中使用
  * `@llvm.memcpy` `@llvm.memset` `@llvm.memmove`
* LLVM 全局变量
  * 所有的全局变量都是指针
  * `@`开头：`@var = global i32 0`

例：

```c
struct RT {
  char A;
  int B[10][20];
  char C;
};
struct ST {
  int X;
  double Y;
  struct RT Z;
};
int foo(struct ST *s) {
  return s[1].Z.B[5][13];
}
```

```llvm
%struct.RT = type { i8, [10 x [20 x i32]], i8 }
%struct.ST = type { i32, double, %struct.RT }
define i32 @foo(ptr %s) {
  %0 = getelementptr %struct.ST, ptr %s, i64 1, i32 2, i32 1, i64 5, i64 13
  %1 = load i32, ptr %0
  ret i32 %1
}
```

## 翻译成 SSA

* 将三地址码风格为基本块
  * Leader
    * 函数的第一条指令
    * 跳转的目标指令
    * 紧接着跳转指令的下一条指令
  * 基本块：连续指令的序列，从 Leader 开始，到下一个 Leader 前结束

### 控制流图

| 行号 | 指令                   | 块号 | 备注         |
| -- | -------------------- | -- | ---------- |
| 1  | `i = 1`              | B1 | Pre-Header |
| 2  | `j = 1`              | B2 | Header     |
| 3  | `t1 = 10 * i`        | B3 |            |
| 4  | `t2 = t1 + j`        | B3 |            |
| 5  | `t3 = 8 * t2`        | B3 |            |
| 6  | `t4 = t3 - 88`       | B3 |            |
| 7  | `a[t4] = 0.0`        | B3 |            |
| 8  | `j = j + 1`          | B3 |            |
| 9  | `if j <= 10 goto B3` | B3 |            |
| 10 | `i = i + 1`          | B4 | Latch      |
| 11 | `if i <= 10 goto B2` | B4 | Exit       |
| 12 | `i = 1`              | B5 |            |
| 13 | `t5 = i - 1`         | B6 |            |
| 14 | `t6 = 88 * t5`       | B6 |            |
| 15 | `a[t6] = 1.0`        | B6 |            |
| 16 | `i = i + 1`          | B6 |            |
| 17 | `if i <= 10 goto B6` | B6 |            |

* 把基本块间 goto 关系用有向边连接起来
* 组件
  * Pre-Header：循环前的基本块
  * Header：循环入口，Back Edge 的目标
  * 前进边：从一个基本块到后代的边
  * 后退边：从一个基本块到它的祖先的边，比如说上面代码中从 `11` 到 `2` 的边
  * 回边：a->b 且 b dom a
    * 回边都是后退边，后退边不一定是回边
    * 如果所有的后退边都是回边，则流图是可规约的
  * 交叉边：边的两端不存在祖先/后代关系
  * 循环由回边定义
  * Latch：循环前的最后一个基本块
  * Exit：判断循环是否结束的基本块

### 支配关系

* 支配节点：所有从入口到 B 的路径都经过 A => A dom B
* 后支配节点：所有从 B 到出口的路径都经过 A => A postdom B
* 严格支配/后支配节点：A dom/postdom B，且 A != B
* 直接支配 A idom B：A strictdom B，且不存在 C，使得 A strictdom C 且 C strictdom B
  * 除入口节点以外的所有节点都有唯一的直接支配节点

#### 支配节点性质

* 自反性：A dom A
* 传递性：A dom B 且 B dom C => A dom C
* 反对称性：A dom B 且 B dom A => A = B

#### 支配树

* 可在接近 O(n) 时间内从控制流图构建
* 节点：块
* 边：直接支配的关系
* Common Dominator：给定块 a b，共同支配 a b 的块
  * O(n) 预处理后可在 O(1) 时间内查询

#### 控制依赖

* 当且仅当 A 的执行结果影响 B 的执行时，B 控制依赖于 A
  * A 有多个出口
  * 不是所有的出口都到达 B
* 判断是否控制依赖
  * **B 后支配于 A 的某些后继**
  * **B 不后支配于 A 的所有后继**
* Dominance Frontier：若 A dom B，且 A 不 strictdom C，C 是 B 的直接后继，则 C 在 A 的 Dominance Frontier 中
  * 拓展定义：$$DF(\mathbb{B}) = \bigcup _{B \in \mathbb{B}} DF(B)$$
* 判断是否是 DF(A)
  * 所有被 A 支配的块（**包括 A 本身**）的**直接后继**
  * 且 不被 A 严格支配
* Iterated dominance frontier
  * $$DF_1=DF(\mathbb{B_0})$$
  * $$\mathbb{B_1}=DF_1 \cup \mathbb{B_0}$$
  * $$DF_2=DF(\mathbb{B_1})$$
  * ……，直到不动点即为$$IDF(\mathbb{B_0})$$

## IDF vs. SSA

* SSA：在合并点使用 $$\phi$$ 函数，复杂
* IDF：以块为单位，假设`x`在$$A B C$$中定义，只需在$$IDF({A,B,C})$$插入$$\phi$$函数即可
