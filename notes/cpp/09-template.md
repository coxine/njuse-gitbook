# 09-模板

## 模板

* 代码复用
* 参数化模块：为模块（类/函数）加上类型参数，对不同类型的数据实施相同的操作
* 实例化：编译器根据模板定义和类型参数**生成具体的类/函数**
  * 按需实例化
    * 仅实例化用到的模板
    * **若类中的某些成员函数未被调用，则不实例化这些成员函数**：模板的声明和实现都要放在头文件中
* 多态的一种形式

## 类属函数 Template Function

```cpp
template <typename T>
bool equal(T a, T b)
{
  return a == b; // 需重载 ==
}

equal(1, 2);           // 正确，自动推导为 equal<int>(1, 2);
equal(1, 2.0);         // 错误，产生歧义
equal<double>(1, 2.0); // 正确，显式指定类型参数

template <typename T>
void sort(T A[], unsigned int num)
{
  for (int i = 1; i < num; i++) {
    for (int j = 0; j < num - i; j++) {
      if (A[j] > A[j + 1]) { // 需重载 >
        T t = A[j];          // 需重载拷贝构造
        A[j] = A[j + 1];
        A[j + 1] = t;
      }
    }
  }
}

// 带多个类型参数
template <class T1, class T2>
void f(T1 a, T2 b) {}
f(1, 2.0); // 自动推导为 f<int, double>(1, 2.0);

// 带普通参数
template <class T, int size>
void g(T a) { T temp[size]; }

g<int, 10>(1); // 非类型参数：必须显式指定
```

* 同一函数对不同类型的数据完成相同的操作
* 编译时实例化模板
* 自动推导类型：仅当类型参数能从函数实参推出，且无歧义时，可省略模板参数列表

### 不优雅的实现方式

* 宏实现：只能实现简单的功能，没有类型检查
* 函数重载：需要定义的函数太多，定义不全
* 函数指针：需要定义额外参数，大量指针运算，实现复杂，可读性差

### 万能引用

* `std::forward<T>(arg)`：完美转发
  * 若 `T` 推导出来是左值，转发为左值引用
  * 若 `T` 推导出来是右值，转发为右值引用
* **有名称的变量都是左值**，和类型无关

```cpp
void process(const std::string &s) {} // 1
void process(std::string &&s) {}      // 2

std::string a = "hello";
process(a);            // 调用 1
process(std::move(a)); // 调用 2

template <typename T>
void transfer(T &&arg)
{
  std::cout << "收到参数..." << std::endl;
  process(arg); // 调用 1，左值右值和类型无关，只要变量有名字就是左值
  process(std::forward<T>(arg)); // 完美转发，调用 transfer 时的实参是左值则调用 1，是右值则调用 2
}
int main()
{
  transfer(std::string("world"));
}
```

### 显式具体化

```cpp
template <typename T>
bool isEqual(T a, T b)
{
  return a == b;
}

template <>
bool isEqual<const char *>(const char *a, const char *b)
{
  return std::strcmp(a, b) == 0;
}
```

* 覆盖模板实例化的默认实现
* 必须先定义通用模板，再定义具体化版本：特化版本依赖于通用模板
* 和函数重载不同：显式具体化属于模板体系
* 例子：`std::vector<bool>`，优化为位数组

## 类属类

```cpp
template <class T, int size>
class Stack
{
  T buffer[size];

  public:
  void push(T x);
  T pop();
};
template <class T, int size>
void Stack<T, size>::push(T x) {}

template <class T, int size>
T Stack<T, size>::pop() {}

Stack<int, 100> st1;
Stack<double, 200> st2;
```

* 对于每一个不同的类型参数 `T`，编译器都会实例化出一个 `Stack<T>` 类
  * 静态变量在 `T` 相同的模板实例间共享
* 可带有多个参数 & 普通参数：`template <typename T, int N, typename Alloc>`

## 模板元编程

* 在编译期执行计算

```cpp
template <typename T>
std::string autoToString(T val)
{
  if constexpr (std::is_same_v<T, std::string>) {
    return val;
  } else {
    return std::to_string(val);
  }
}
// 若 `T==std::string`，编译器编译为 `return val;`
// 否则，代码变成 `return std::to_string(val);`
// 在编译期执行条件判断
```

```cpp
template <int N>
class Fib
{
  public:
  enum { value = Fib<N - 1>::value + Fib<N - 2>::value };
};

template <>
class Fib<0>
{
  public:
  enum { value = 1 };
};
template <>
class Fib<1>
{
  public:
  enum { value = 1 };
};

int main()
{
  std::cout << Fib<8>::value << std::endl;
}
```

## 编译期类型推导

* `auto x = expr;`：根据 `expr` 的类型推导出 `x` 的类型
  * `auto` 变量必须初始化
* `auto` 默认推导值拷贝，丢弃引用和 `const` 限定符
  * `const auto &x = expr;`：保留 `const` 和引用
  * `auto &x = expr;`：引用，可修改
