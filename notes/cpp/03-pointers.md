# 03-指针

## 数组

* debug
  * VS：/RTC1
  * C：mallopt
  * Linux：`valgrind`
* 数组名不能`++`/`--`，但可以进行指针运算

### 左值 & 右值

| 特性    | 左值       | 右值        |
| ----- | -------- | --------- |
| 可否赋值  | 可以       | 不可以（普通右值） |
| 可否取地址 | 可以       | 不行（普通右值）  |
| 生命周期  | 至少到作用域结束 | 临时对象，通常很短 |
| 举例    | 变量名、数组名  | 字面量、表达式结果 |

### 数组的退化

* 数组作为函数参数/右值时，数组会退化为指针，丢失大小信息
* 数值作为左值时，如`int arr[10];`，不会退化
* `sizeof` 计算的是指针大小，`typeid.name()` 输出的是指针类型

### 指针定义

* C：`#define NULL ((void *)0)`
* C++：`#define NULL 0`
* 应当使用 `nullptr`：避免歧义

```cpp
void func(int);
void func(char*);
func(NULL); // 调用 func(int)，NULL 被视为整数 0
func(nullptr); // 调用 func(char*), nullptr 明确表示空指针
```

## 多维数组

```cpp
int a[M][N];

using T = int[N]; 
T a[M];    // a 是一个包含 M 个元素的数组，每个元素是一个包含 N 个整数的数组（M 行 N 列）
```

* 升维/降维：重新解释同一段内存

```cpp
#include <iostream.h>

int maximum(int a[], int n)
{
  int max = 0;
  for (int k = 0; k < n; k++)
    if (a[k] > max)
      max = a[k];
  return max;
}

int main()
{
  int A[2][4] = {{68, 69, 70, 71}, {85, 86, 87, 89}};
  cout << "the max grade is " << maximum(A[0], sizeof(A) / sizeof(A[0][0])) << endl;
  cout << typeid(A[0]).name();
}
// the max grade is 89
// A4_i
```

```cpp
int b[12];
void show(int a[], int n);
void show(int a[][2], int n);
void show(int a[][2][3], int n);

void main(){
  using T = int[2];
  using T1 = int[3];
  using T2 = T1[2];
  show(b, 12);
  show((T *)b, 6);          // 用第2个show
  show((T2 *)b, 2);         // 用第3个show
}
```

## 智能指针

* 传统指针：异常/退出时易导致内存泄漏
* 智能指针：自动管理内存，防止泄漏
* RAII：Resource Acquisition Is Initialization
  * 资源获取即初始化
  * 对象生命周期内管理资源
  * 在构造函数中获取资源，在析构函数中释放资源
  * 特点
    * 谁申请，谁释放
    * 利用栈展开
    * 异常安全
    * 零开销抽象

| 指针类型         | 引用计数 | 是否拥有对象 | 用途                        |
| ------------ | ---- | ------ | ------------------------- |
| `unique_ptr` | 无    | 独占     | 单所有者，作用域结束自动释放            |
| `shared_ptr` | 有    | 共享     | 多个所有者，引用计数归零时释放           |
| `weak_ptr`   | 无    | 不拥有    | 观察 `shared_ptr` 对象，防止循环引用 |

> C++98 引入 `auto_ptr`，但存在语义问题，C++11 引入 `unique_ptr` 取代

### `unique_ptr`

* 用途：独占所有权，作用域结束自动释放

```cpp
void newer_use(Args a) {
 auto p = unique_ptr<Blob>(new Blob(a));
 // ...
 if (foo) throw Bad(); // 不会泄漏
 if (bar) return; // 不会泄漏
 // ...
}
```

### `shared_ptr`

* 用途：多个所有者共享所有权
* 维护引用计数，引用计数归零时释放对象
* `make_shared<T>(args...)`：构造 + 创建 `shared_ptr`，推荐使用
* `use_count()`：获取当前引用计数
* `get()`：获取原始指针
* `reset()`：重置指针，减少引用计数，指向新对象/`nullptr`

```cpp
shared_ptr<int> shared_p{new int{1}}; // = make_shared<int>(int{ 1 });
cout << "shared_p: " << *shared_p << " count: " << shared_p.use_count() << endl; // 1,1
shared_ptr<int> shared_p_ref = shared_p;
*shared_p_ref = 11;
cout << "add a ref : " << endl
     << "shared_p : " << *shared_p << " count:" << shared_p.use_count() << endl; // 11,2
weak_ptr<int> weak_p{shared_p};
cout << "add a weak :" << endl
     << " shared_p: " << *shared_p << " count:" << shared_p.use_count() << endl; // 11,2
cout << "weak_p : " << weak_p.use_count() << endl; // 2

auto temp = weak_p.lock(); // 增加一次引用计数
cout << "after lock :" << endl;
if (nullptr == temp) cout << "shared resource expired" << endl;
else { *temp = 111;
 cout << "shared_p : " << *shared_p << " count:" << shared_p.use_count() << endl; } // 111,3
shared_p.reset(new int{2}); // shared_p 指向新对象，引用计数减 1，原来的两个引用仍然有效
cout << "shared_p moved " << endl
     << "shared_p " << *shared_p << " count:" << shared_p.use_count() << endl; // 2,1
cout << "shared_p_ref: " << *shared_p_ref << " count:" << shared_p_ref.use_count() << endl; // 111,2
```

### `weak_ptr`

* 用途：**观察** `shared_ptr` 对象，防止循环引用
* 不影响引用计数
* 无法直接解引用，需通过 `lock()` 获取
  * 若对象已被释放，返回 `nullptr`
  * 若对象未被释放，返回有效的 `shared_ptr`，引用计数加 1，在作用域结束时自动减 1

```cpp
// 观察者模式
struct Subject;

struct Observer {
    std::weak_ptr<Subject> subject;
};

struct Subject {
    std::vector<std::shared_ptr<Observer>> observers;
};
```

```cpp
// 双向链表节点
struct Node {
    int v;
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;
};
```

### 实现

* `shared_ptr` 分为两部分
  * `Data Pointer`：指向实际对象
  * `Control Block`：管理引用计数等信息
    * `strong count`：`shared_ptr` 数量，即`use_count()`
    * `weak count`：`weak_ptr` 数量

## 函数指针

* `ret (*func_ptr)(arg_types...)`：定义函数指针
* `func_ptr = &func_name`：指向函数
* `(*func_ptr)(args...)`：调用函数
* 可以将函数作为参数传递给其他函数
