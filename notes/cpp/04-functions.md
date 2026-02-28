# 04-函数

* 原则
  * 定义不允许嵌套
  * 先定义后使用

## 函数的执行机制

1. 建立被调用函数的栈空间
   * 上一函数的`esp`（栈顶）作为新函数的`ebp`（栈底）
   * 逐个压入函数参数
   * 压入`eip`（返回地址）
   * 压入调用函数的`ebp`
   * 此时的`esp`为新函数的`esp`
2. 执行被调用函数体
3. 函数返回
   * 将返回值放入`eax`
   * 恢复调用函数的`ebp`和`esp`
   * 跳转到`eip`处继续执行

## 参数传递

* 值传递：把值的副本传递给被调函数，函数内改动不影响原值
* 引用传递：把变量的引用传递给被调函数，函数内改动影响原值，相较于指针更安全
  * 引用必须非空
  * 不可改变引用指向

## 函数调用约定

| 调用约定               | 参数压栈顺序                             | 谁负责清栈                        | 常见用途                      |
| ------------------ | ---------------------------------- | ---------------------------- | ------------------------- |
| cdecl              | 从右到左                               | 调用者 caller                   | C/C++ 默认；支持可变参数（`printf`） |
| stdcall            | 从右到左                               | 被调函数 callee                  | Windows API 常用；不支持可变参数    |
| fastcall           | 前两个参数进寄存器（通常 `ECX`, `EDX`），剩下右到左入栈 | callee                       | 编译器优化、某些系统函数              |
| thiscall（C++ 成员函数） | `this` 放 `ECX`，其它参数右到左             | Visual C++ 下 callee（gcc 有变化） | C++ 成员函数默认约定              |

```cpp
// 指定调用约定
__attribute__((cdecl)) void func_cdecl(int a);
__attribute__((stdcall)) void func_stdcall(int a);
__attribute__((fastcall)) void func_fastcall(int a, int b);
```

## 重载

* 名称相同，参数的个数/类型/顺序不同
* 返回值类型不作为重载依据

### 重载匹配原则

1. 精确匹配优先
2. 类型提升：仅限 `char/bool/short` 提升为 `int`
3. 类型转换（如 `int` 转为 `float`，`float` 转为 `double`）
4. 用户自定义类型转换（如类的转换构造函数或转换运算符）
5. 可变参数

### Name Mangling

* 编译器在符号表中为重载函数生成唯一名称

|函数|MSVC 编译后|GCC 编译后| |--||-| |`f(int)`|`?f@@YAXH@Z`|`_Z1fi`| |`f(double)`|`?f@@YAXN@Z`|`_Z1fd`|

* 不同编译器的 Name Mangling 规则不同，导致跨编译器链接困难
* `extern "C"`：关闭 C++ 的重载和 Name Mangling，使函数以 C 方式导出，便于跨语言调用

```cpp
#ifdef __cplusplus
extern "C" {
#endif

void foo();
int bar(int);

#ifdef __cplusplus
}
#endif
```

## 默认参数

```cpp
void f(int a, int b = 2, int c = 3); // ✅
void f(int a = 1, int b, int c = 3); // ❌ 不连续
```

* 必须在声明时指定默认参数
* 默认参数必须从右向左连续提供
* 可能会由于重载产生歧义

```cpp
void f(int);
void f(int, int = 2);

f(10);
```

## inline

* 将函数直接展开成代码插入调用点，减少上下文切换开销
* 目的
  * 提高可读性
  * 提高效率
* 无法内联的情况
  * 递归
  * 函数体较大，涉及 `switch`、循环等复杂结构
  * 涉及函数指针
* `inline` 只是建议，编译器可选择忽略
* 适用场景：函数简单，使用频繁
  * 构造函数
* `inline` 函数体必须在头文件中：以便每个使用该函数的编译单元都能看到其定义，否则无法内联
* 缺点：增加代码大小，导致频繁换页，指令缓存命中率下降

## 函数式编程

* 无副作用

### 迭代器

* `std::back_inserter(container)`：插入迭代器，自动在容器末尾添加元素，对应`container.push_back(value)`，会动态扩容
* `std::front_inserter(container)`：插入迭代器，自动在**容器前端**添加元素，仅适用于支持`push_front`的容器（如`std::deque`、`std::list`），会动态扩容
* `std::inserter(container, it)`：插入迭代器，在指定位置`it`插入元素，对应`container.insert(it, value)`
* `container.begin()`：返回指向**容器第一个元素**的迭代器
* `container.end()`：返回指向**容器最后一个元素后面**的迭代器
* `container.rbegin()`：返回指向**容器最后一个元素**的反向迭代器
* `container.rend()`：返回指向容器**第一个元素前面**的反向迭代器

```cpp
deque<int> d = {3, 4, 5};
vector<int> a = {1, 2};
copy(a.begin(), a.end(), front_inserter(d)); // d => 2, 1, 3, 4, 5
```

### 判定类（返回 `bool`）

* `all_of(first, last, pred)`：是否全部满足
* `any_of(first, last, pred)`：是否存在满足
* `none_of(first, last, pred)`：是否全部不满足

### 查找类

* `find(first, last, value)`：找等于某值
* `find_if(first, last, pred)`：找第一个满足条件的
* `find_if_not(first, last, pred)`：找第一个不满足条件的

### 计数类

* `count(first, last, value)`：统计等于某值的个数
* `count_if(first, last, pred)`：统计满足条件的个数

### 转换 / 应用类

* `for_each(first, last, func)`：对每个元素应用函数
* `transform(first, last, out, func)`：转换生成新序列（相当于 map）

### `filter`

```cpp
vector<int> vec = {1,2,3,4,5,6};
vector<int> new_vec;
bool cond(int x) { return x % 2 == 0; } // 条件函数
auto it = remove_if(vec.begin(), vec.end(), cond); // vec={1,3,5,4,5,6} 返回的迭代器指向新结尾
vec.erase(it, vec.end()); // 删除满足 cond 的元素 vec={1,3,5}

// 或者
std::copy_if(vec.begin(), vec.end(), std::back_inserter(new_vec), cond); // 复制满足 cond 的到新容器
// 此处使用 back_inserter，在 new_vec 末尾插入元素
```

### 聚合 / 归约

* `accumulate(first, last, init, op)`（在 `<numeric>`）
  * `op` 默认为 `+`
* `reduce(first, last, init, op)`（C++17，支持并行）

```cpp
vector<vector<int>> vec = {{1,2,3}, {4,5}, {6}};
vector<int> flattened = accumulate(vec.begin(), vec.end(), vector<int>{},
    [](vector<int> a, const vector<int>& b) {
        a.insert(a.end(), b.begin(), b.end());
        return a;
    });
```

### 排序 / 排列

* `sort`, `stable_sort`, `nth_element`
* `unique`（配合 erase）去重
* `next_permutation`, `prev_permutation`

### 集合操作（要求有序序列）

* `set_union`
* `set_intersection`
* `set_difference`
* `set_symmetric_difference`

### C++20 ranges

* 惰性求值：只有在遍历时才会计算结果

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};

    auto r =
        v
|std::views::filter([](int x) { return x % 2 == 0; })
|std::views::transform([](int x) { return x * x; })
|std::views::take(2)
|std::views::reverse;

    for (int x : r)
        std::cout << x << " "; // 输出：16 4
}
```

```cpp
// 按照某一指标分类
auto processStudents(const std::vector<Student>& students) {
     auto select= students 
|std::views::transform([](const Student& s) { 
             return std::make_pair(s.major, s); 
        });

    return std::accumulate(
        select.begin(), select.end(),
        std::map<int, std::vector<Student>>{},
        [](auto acc, const auto& pair) {
            auto& [major, student] = pair;
            acc[major].push_back(student);
            return acc;
        }
    );
}
```

## Lambda表达式

```cpp
[a, &b](int x) mutable noexcept -> double {
    a += x;     // 修改值捕获的 a，必须 mutable 才能编译
    b += 1;     // 引用捕获的 b 本来就能改
    return a + b;
}
```

* `[]`：捕获列表
  * 不捕获任何变量：`[]`
  * 按值捕获所有变量：`[=]`
  * 按引用捕获所有变量：`[&]`
  * 按值捕获特定变量：`[a]`
  * 按引用捕获特定变量：`[&a]`
  * 按值捕获所有变量，按引用捕获特定变量：`[=, &b]`
  * 混合捕获：`[a, &b]`
* `(int x)`：参数列表
* `mutable`：允许修改按值捕获的变量
* `noexcept`：表示该 lambda 不会抛出异常
* `-> double`：返回类型，可省略，编译器会自动推导
* 大括号内的代码：函数体

### 函数对象

* 编译器会将 lambda 表达式转换为一个匿名类，该类重载了 `operator()`

```cpp
class __Lambda_123 {
private:
    double a;   // 值捕获的 a 作为成员变量
    double& b;  // 引用捕获的 b 作为引用成员

public:
    __Lambda_123(double a_, double& b_) : a(a_), b(b_) {}

    double operator()(int x) noexcept {
        a += x;  
        b += 1;  
        return a + b;
    }
};
```

## `bind`

```cpp
auto f = std::bind(func, std::placeholders::_1, 42); // 绑定第二个参数为42
f(10); // 相当于调用 func(10, 42)
```

## `function` 包装器

* 可以存储任意可调用对象
  * 函数
  * 函数对象
  * Lambda 表达式
  * 函数指针
  * `bind` 结果

```cpp
#include <functional>
#include <iostream>
#include <map>
using namespace std;

int add(int a, int b)
{
  return a + b;
}
struct ADD {
  int operator()(int a, int b)
  {
    return a + b;
  }
};

int Add3(int a, int b, int c)
{
  return a + b + c;
}

int main()
{
  auto newAdd = std::bind(Add3, std::placeholders::_1, 100, std::placeholders::_2);
  map<string, function<int(int, int)>> opmap =
      {{"fun_pointer", add},
       {"operator()", ADD()},
       {"lambda", [](int a, int b) -> int { return a + b; }},
       {"bind", newAdd}
      };

  cout << opmap["fun_pointer"](1, 2) << endl;
  cout << opmap["operator()"](3, 4) << endl;
  cout << opmap["lambda"](5, 6) << endl;
  cout << opmap["bind"](7, 8) << endl;
  return 0;
}
```
