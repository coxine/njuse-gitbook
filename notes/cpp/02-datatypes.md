# 02-类型

## 数值类型

### 查看类型范围

```cpp
#include <limits>

std::numeric_limits<T>::min(); // 最小值
std::numeric_limits<T>::max(); // 最大值
```

### 溢出

* `unsigned`：Wrap around，回到最大/最小值
* `signed`：未定义行为
  * 事前检查
  * `builtin_add_overflow(x,y,&result)`：检测加法溢出
  * `<stdckdint.h>`：C23，C++26，带溢出检测的运算

> UB
>
> * 程序行为未定义，导致不可预测的结果/程序崩溃
> * 编译器假设程序不会发生UB，从而进行优化

### 浮点数

```cpp
#include <limits>

numeric_limits<float>::epsilon(); // 最小精度
numeric_limits<float>::infinity(); // 无穷大
numeric_limits<float>::quiet_NaN(); // 非数
```

#### 使用 `bitset` 查看浮点数的二进制表示

```cpp
#include <iostream>
#include <bitset>

void printFloatBinary(float f) {
    unsigned int binary_rep;
    memcpy(&binary_rep, &f, sizeof(f));
    
    cout << "deci: " << f << endl;
    cout << "hex: 0x" << hex << binary_rep << dec << endl;
    cout << "binary: " << bitset<32>(binary_rep) << endl;
    
    unsigned int sign = (binary_rep >> 31) & 0x1;
    unsigned int exponent = (binary_rep >> 23) & 0xFF;
    unsigned int mantissa = binary_rep & 0x7FFFFF;
    
    cout << bitset<1>(sign) << endl;
    cout << bitset<8>(exponent) << " (" << 
                          (exponent - 127) << ")" << endl;
    cout << bitset<23>(mantissa) << endl;
    cout << endl;
}
```

#### NaN

* inf / inf = NaN
* inf - inf = NaN
* 0.0 / 0.0 = NaN
* inf \* 0.0 = NaN
* sqrt(-1.0) = NaN
* log(-1.0) = NaN

#### 规格化 & 非规格化

```cpp
#include <cfloat>
#include <cmath>
DBL_MIN; // 最小规格化正数
std::is_normal(x); // 是否为规格化数
std::fpclassify(x); // 分类
// FP_INFINITE, FP_NAN, FP_NORMAL, FP_SUBNORMAL, FP_ZERO
```

## 类型转换

### C 风格转换

* 暴力，可能存在 UB，在 C++ 中不推荐使用

### `static_cast`

* 算术类型转换：和 C 中规则相同
  * 整型长转短：截断高位
  * 整型短转长：有符号数补符号位/无符号数补0
  * 浮点转浮点：可能丢失精度
  * 浮点转整型：截断小数部分
  * 整型转浮点：可能丢失精度
* 枚举 <-> 整型
* `void*` <-> 对象指针
* 派生类 -> 基类（upcast）
* 基类 -> 派生类（downcast）：不安全，无运行时检查，使用 `dynamic_cast` 更安全
* 当类定义了`operator T()`时，调用转换方法

### `const_cast`

* 添加/移除 `const` 或 `volatile` 属性
* 仅应为兼容接口声明的类型转换
* 修改被 `const` 修饰的对象是 UB

```cpp
#include <iostream>

using namespace std;

int main()
{
  const int c = 128;
  int *q = const_cast<int *>(&c);
  *q = 111;

  cout << "c " << &c << c << endl;
  cout << "q " << &q << q << endl;
  cout << "*q " << q << *q << endl;
  return 0;
}
```

> c 0xbed15ff8f4 128
>
> q 0xbed15ff8f8 0xbed15ff8f4
>
> \*q 0xbed15ff8f4 111
>
> 查了下汇编，g++ 直接优化，把 128 放在寄存器里 可见 ub 的不确定性

### `reinterpret_cast`

* 低级别转换，通常用于指针类型之间的转换
* 不安全
* 用于序列化

```cpp
// 以二进制形式打印任意类型的值
template<typename T>
void printBinary(const T& value) {
  const unsigned char* bytes = reinterpret_cast<const unsigned char*>(&value);
  for (int i = sizeof(T) - 1; i >= 0; --i) {
    cout << bitset<8>(bytes[i]) << " ";
  }
cout << endl;
}
```

```cpp
// 序列化、
#pragma pack(push, 1) // 取消对齐
struct PacketHeader {
  std::uint32_t packetId;
  std::uint16_t payloadSize;
  std::uint8_t flags;
};
#pragma pack(pop)

bool writeHeaderToFile(const std::string& filename, const PacketHeader& header) {
  std::ofstream file(filename, std::ios::out|std::ios::binary);
  if (!file.is_open()) {...}

  file.write(reinterpret_cast<const char*>(&header), sizeof(header));

  if (!file.good()) {...}
  
  file.close();
  return true;
}

bool readHeaderFromFile(const std::string& filename, PacketHeader& header) {
  std::ifstream file(filename, std::ios::in|std::ios::binary);
  if (!file.is_open()) {...}

  file.read(reinterpret_cast<char*>(&header), sizeof(header));

  if (!file.good()) {...}

  file.close();
  return true;
```

### `any_cast`

* 用于 `std::any` 类型的转换
* 不匹配时返回
  * 指针类型：返回 `nullptr`
  * 引用类型：抛出 `std::bad_any_cast` 异常

```cpp
std::any a;
a = std::any_cast<int>(a) + 1;
cout << std::any_cast<int>(a) << endl;
a = 1.5;
cout << std::any_cast<double>(a) << endl;
a = std::string("hello");
cout << std::any_cast<std::string>(a) << endl;
```

### `dynamic_cast`

* 用于多态类型的安全转换
* 失败返回
  * 指针类型：返回 `nullptr`
  * 引用类型：抛出 `std::bad_cast` 异常

## `union`

* 联合体，所有成员共享同一块内存
* 通过不同的别名访问内存

```cpp
union Matrix{
  struct { float m00, m01, m10, m11; };
  float m[2][2];
}
```

## `std::variant`

* 类型安全的`union`
* 编译时确定、检查类型
* 运行时仅检查当前存放数据的激活索引是否匹配
* 所有的 `get` 都是模板函数，编译时生成代码，不会有运行时开销

### 实现

* 内存分布
  * `buffer`：存储实际数据，大小为最大成员大小
  * `index`：存储当前活动类型的索引

```cpp
#include <variant>

std::variant<int, float, std::string> v;
v = 42;
v = 3.14f;
v = "hello";

std::cout << std::get<std::string>(v) << std::endl; // "hello"，通过类型访问
std::cout << std::get<2>(v) << std::endl; // "hello"，通过索引访问
std::cout << std::get<int>(v) << std::endl; // 获取当前值，类型错误会抛出异常

bool isInt = std::holds_alternative<int>(v); // 检查当前类型是否为 int，false
std::string* str = std::getif<std::string>(&v); // 获取当前值的指针，类型错误返回 nullptr

visit([](auto&& value) {
    cout << value;
}, v); // 类型安全地访问当前值，编译器会自动生成适当的代码
```

### 回顾：访问者模式

```cpp
#include <iostream>
using namespace std;

struct Circle;
struct Square;

struct Visitor {
    virtual void visit(Circle&) = 0;
    virtual void visit(Square&) = 0;
};

struct Shape {
    virtual void accept(Visitor&) = 0;
};

struct Circle : Shape {
    void accept(Visitor& v) override { v.visit(*this); }
};

struct Square : Shape {
    void accept(Visitor& v) override { v.visit(*this); }
};

struct DrawVisitor : Visitor {
    void visit(Circle&) override { cout << "Draw Circle\n"; }
    void visit(Square&) override { cout << "Draw Square\n"; }
};

int main() {
    Circle c; Square s;
    DrawVisitor d;
    c.accept(d);
    s.accept(d);
}
```

* 双分派：
  * `Shape::accept` -> `Circle::accept`
  * `Circle::accept` -> `DrawVisitor::visit(Circle&)`
* 解耦组件和行为：用于添加新行为而不修改组件

## `std::any`

* 类型安全
* 运行时多态：类型在运行时确定，存在局限
* 需要通过 `any_cast` 获取实际类型，含 `Handler` 函数运行开销和类型比较开销
* 语义与限制
  * 支持空值状态（默认构造时为空）
  * 值必须可拷贝构造
  * 支持拷贝、移动拷贝、移动赋值

```cpp
#include <any>

std::any a;
a = 42; // int
a.has_value(); // true
a.reset(); // 清空
a.has_value(); // false
```

### 实现

```cpp
enum class _Action
{ _Destroy, _Copy, _Move, _Get, _TypeInfo };

Handler _HandleFuncPtr = void* (*) (Args...);

using _HandleFuncPtr =
    void *(*)(_Action, const any *, any *, const std::type_info *, const void *);
switch (__act) {
case _Action::_Destroy:
  __destroy(const_cast<any &>(*__this));
  return nullptr;
case _Action::_Copy:
  __copy(*__this, *__other);
  return nullptr;
case _Action::_Move:
  __move(const_cast<any &>(*__this), *__other);
  return nullptr;
case _Action::_Get:
  return __get(const_cast<any &>(*__this), __info, __fallback_info);
case _Action::_TypeInfo:
  return __type_info();
}

union Storage {
    void* ptr;
    __any_imp::_Buffer __buf;
};
```

* 32 字节
* `Handler`：处理类型擦除的函数指针，8 字节
  * 根据类型对应不同的`_Action`，产生不同的`Handler`
* `Storage`：存储实际数据，24 字节
  * `ptr`：存储大对象的指针
  * `__buf`：小对象优化（SOO），若对象大小 <= 24 字节，直接存储在缓冲区

## `struct`

* 对齐
  * `char`：1 字节
  * `short`：2 字节
  * `int`：4 字节
  * `long`：8 字节
  * 若字节不足，补 padding

## `tuple`

* 用于函数返回多个值
  * 不优雅方法：传入指针/引用参数、返回结构体

```cpp
#include <tuple>
#include <iostream>
using namespace std;

tuple<int, float> foo() {
    return std::make_tuple(1, "Alice", 92.5);
    return std::tuple<int, std::string, double>(2, "Bob", 88.0);
    return {1, 3.0f};  // 返回一个 tuple，自动类型推导
}

int main() {
    auto r = foo();
    cout << get<0>(r) << " " << get<1>(r) << "\n"; // 访问元素

    // C++11 解包
    int i;
    float f;
    tie(i, f) = foo();  // tuple 解包到 i 和 f
    cout << i << f;

    // C++17 结构化绑定
    auto [i, f] = foo();  // 自动生成 i 和 f
    cout << i << f;
}
```

## `optional`

* 用于返回值可能为空的场景
  * 不优雅方法：使用状态码/状态布尔变量，返回空指针

```cpp
#include <optional>

std::optional<int> find(const std::vector<int>& vec, int target) {
    for (int num : vec) {
        if (num == target) {
            return num; // 找到，返回值
        }
    }
    return std::nullopt; // 未找到，返回空
}
```
