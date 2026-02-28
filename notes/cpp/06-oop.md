# 06-OOP

### OOP 目的

* 面向过程：数据暴露，不安全
* 面向对象：封装
* 早期 C++：使用 CFront，将面向对象的特性转换为 C 代码
  * 在编译阶段检查 private/public

### OPP 概念

* 程序 = 对象1 + 对象2 + ...
* 对象 = 数据 + 方法
* 传递信息：调用对象的方法
* 类：对象的类型，定义对象的数据和方法

> Object-based 和 Object-oriented 的差别
>
> * Object-based：支持对象、封装，不支持继承和多态
> * Object-oriented：支持对象、封装、继承和多态

### OOP 的优点

* 提升开发效率
* 对外：提升正确性、效率、鲁棒性、可靠性、可用性、可复用性
* 对内：提升可读性、可维护性、可移植性

## 类的基本语法

```cpp
// Tdate.h
class Tdate
{
  public:
  void SetDate(int y, int m, int d); // 成员函数
  int isLeapYear();  // 成员函数
  private:
  int year, month, day; // 成员变量
};

// Tdate.cpp
void TDate::SetDate(int y, int m, int d)
{
  year = y;
  month = m;
  day = d;
}
int TDate::isLeapYear() { return (year % 4 == 0 && year % 100 != 0)||(year % 100 == 0); }
```

### 成员函数定义位置

* 声明：头文件（分离声明和实现）
* 实现：类内或类外均可

|            | 类内定义    | 类外定义     |
| ---------- | ------- | -------- |
| 默认 inline？ | 是       | 否        |
| 适合的函数      | 小函数、访问器 | 复杂逻辑、大函数 |

### 模块

* 头文件的缺点：文本复制、重复编译、效率低
* 模块：使用编译后的接口文件，提升编译效率

#### 模块的文件类型

| 文件类型                                               | 作用                                                        | 典型拓展名 / 注释                              |
| -------------------------------------------------- | --------------------------------------------------------- | --------------------------------------- |
| **模块接口单元（Module Interface Unit）**                  | 定义模块接口，包含导出的函数、类、变量等，是模块的“公共 API”。其他代码通过 `import` 来使用该模块。 | `.cppm`（标准推荐，也有 `.ixx`、`.mpp` 等历史/厂商拓展） |
| **模块分区接口单元（Module Partition Interface Unit）**      | **将模块接口拆分为多个分区**，方便大型模块分块开发和编译。                           | `.cppm` / `.ixx`（可与主接口单元拓展相同）           |
| **模块实现单元（Module Implementation Unit）**             | 定义模块内部实现，可以访问模块接口，但不一定导出给外部使用。                            | `.cpp` / `.mpp`（取决于编译器）                 |
| **模块分区实现单元（Module Partition Implementation Unit）** | **将模块实现拆分为多个分区**，但不对外导出。                                  | `.cpp` / `.mpp`                         |
| **模块接口头文件（Header Unit，非标准头）**                      | 可将传统头文件也导入为模块接口单元，供 `import` 使用。                          | `.h` / `.hpp` / `.hxx`                  |
| **普通源文件**                                          | 可以包含普通函数或类实现，不属于任何模块，但可以被模块实现单元调用。                        | `.cpp`                                  |

#### 模块代码示例

```cpp
// math.cppm
export module Math;

export int add(int a, int b);
export double pi = 3.14159;
```

```cpp
// math_impl.cpp
module Math; // 实现单元

int add(int a, int b) { return a + b; }
```

```cpp
// main.cpp
import Math;

#include <iostream>
int main() {
    std::cout << add(2, 3) << "\n";
}
```

#### 模块编译

* 编译后输出
  * 可导入的`pcm`文件（接口单元）
  * 可链接的对象文件
* 编译流程（需要手动在`Makefile`或构建脚本中指定）
  1. 编译模块接口单元
  2. 编译模块实现单元
  3. 若模块间存在依赖，按照依赖顺序编译
  4. 模块编译完成后，编译普通的源文件，链接模块对象文件

## 构造函数

* 与类同名、无返回类型
* 自动调用，不可直接调用
* 可重载

### 默认构造函数

* 无参数，编译系统自动生成
  * 内置类型成员：不初始化，可能为垃圾值
  * 类类型成员：调用对应类的默认构造函数
* 不会生成默认构造函数的情况
  * 已定义其他构造函数
  * 有`const`或引用类型成员变量，没有对应的默认构造函数
* `delete`构造函数：无法调用默认构造函数，编译时报错
* `default`：显式请求编译器生成默认构造函数
* 构造函数默认 `public`
  * `private` 构造函数：类接管对象创建职责，如单例模式

```cpp
class C1 {
public:
    C1(int x); // 有参构造函数，默认构造函数不会自动生成
};

class C2 {
public:
    C2() = default; // 显式默认构造
};

class C3 {
public:
    C3() = delete; // 禁用默认构造函数
};
```

### 成员初始化列表

* `const`、引用、无默认构造函数的成员变量必须使用
* 初始化顺序
  * **按成员变量声明顺序**，而非初始化列表顺序
  * 先按成员变量顺序初始化，再调用构造函数体
* 提升效率：直接调用构造函数，避免默认构造+赋值（C++17 起编译器对构造函数优化，是否使用初始化列表影响不大）
* 当数据成员过多时，不推荐使用

```cpp
class A
{
  int x;
  const int y;
  int &z;

  public:
  A() : y(1), z(x), x(0)
  {
    cout << "在构造函数体内：" << "x = " << x << ", y = " << y << ", z = " << z << endl; // 0,1,0
    x = 100;
    cout << "赋值后 x = " << x << ", y = " << y << ", z = " << z << endl; // 100,1,100
  }
};
```

### 委托构造

* 一个构造函数调用同类的另一个构造函数
* 减少代码重复
* 编译器确保只调用一次最终的构造函数
* 仅限在成员初始化列表中使用

```cpp
class C {
public:
    C() : C(1) { }
    C(int x) : C(x, 2) { }
    C(int x, int y) { /* 真正初始化 */ }
    C() { C(1, 2); } // 错误，不能在函数体内委托
};
```

### 各种初始化方式

* `A a = A(1);`：显式初始化
  * C++ 17 前先在右侧创建临时对象，再拷贝给`a`，开销较大
  * C++ 17 起编译器优化，和直接初始化无差别
* `A a(1);`：直接初始化，直接调用构造函数，效率高
* `A a = 1;`：隐式类型转换
  * C++ 17 前先在右侧创建临时对象，再拷贝给`a`，开销较大
  * C++ 17 起编译器优化，和直接初始化无差别
  * 当构造函数为`explicit`时不可用
* `A a{1};`：列表初始化
  * C++11 起支持
  * 统一的初始化语法
  * 避免窄化转换

```cpp
double d = 3.14;
int a(d);  // 允许
int b = d; // 允许
int c{d};  // 错误，列表初始化不允许窄化转换
```

```cpp
class A
{
  public:
  A();
  A(int i);
  A(char *p);
};

A a1 = A(1);      // 调用 A(int i)
                  // 也可 'A a1(1);' or 'A a1 = 1;'
A a2 = A();       // 调用 A()
                  // 也可 'A a2;'
                  // 注意：'A a2();' 不是初始化！这是一个函数的声明.
A a3 = A("abcd"); // 调用 A(char *p);
                  // 也可 'A a3("abcd");' or 'A a3="abcd";'
A a[4];           // 调用四次 A()
A b[5] = {A(), A(1), A("abcd"), 2, "xjy"};
// 感谢 xjy 学长的笔记，PPT 上的代码太糟糕了
```

## 析构函数

* `~ClassName()`，无返回值，无参数
* 调用时机
  * 栈对象：RAII，离开作用域时自动调用，释放资源
  * 堆对象：`delete` 时调用

### 将析构函数设为 `private`

* 作用
  * 禁止显式调用`delete` 销毁对象
  * 强制对象只能在堆上创建：对于栈对象，编译器无法在离开作用域时调用析构函数，导致编译错误
* 更好的方法，使用`static void free(ClassName* p)` 静态成员函数销毁对象

## 拷贝构造函数

* 作用：创建新对象，将原有对象的数据拷贝给新对象
* 声明格式：`ClassName(const ClassName &);`
* 使用场景
  * 赋值操作：`ClassName obj2 = obj1;`
  * 将对象**按值传参**给函数
  * 对象作为函数的返回值
* 默认拷贝构造函数：浅拷贝
  * 内置类型成员：值拷贝
    * 若出现指针成员，会复制地址，**可能导致多个对象指向同一内存，出现悬空指针等问题**
  * 类类型成员：调用对应类的拷贝构造函数
* 若自定义拷贝构造函数，而没有在拷贝构造函数的初始化列表中提及成员：和默认构造函数机制相同
  * 内置类型成员：不初始化，可能为垃圾值
  * 类类型成员：调用对应类的**默认构造函数**（不是拷贝构造函数）

```cpp
class A
{
  int x, y;

  public:
  A() { x = y = 0; }
  void inc() { x++; y++; }
};

class B

{
  int z;
  A a;

  public:
  B() { z = 0; }
  B(const B &b) { z = b.z; }
  void inc() { z++; a.inc(); }
};

B b1;
b1.inc();
B b2(b1); // 此时 b2.a 的成员 x 和 y 都为 0，因为没有在拷贝构造函数中初始化 a
```

### 拷贝赋值运算符

* `obj2 = obj1;`
* 需要额外声明`ClassName & operator=(const ClassName &);`
* 拷贝构造和拷贝赋值的区别
  * 拷贝构造：创建新对象时调用
  * 拷贝赋值：已有对象赋值时调用
* 异常安全性：把可能失败的操作放在前面，成功后再修改对象状态

```cpp
ResourceBox &operator=(const ResourceBox &rhs)
  {
    if (this != &rhs) {
      // 1. 先申请新内存并拷贝数据（危险操作放前面）
      int *new_content = new int[rhs.size];
      std::memcpy(new_content, rhs.content, rhs.size * sizeof(int));

      // 2. 只有上面成功了，才开始修改本对象的状态（安全操作放后面）
      delete[] content; // 现在可以安全释放旧资源了
      content = new_content;
      size = rhs.size;
    }
    return *this;
  }
```

### 禁用拷贝构造

* 适用于不允许拷贝的类，如互斥锁
* `ClassName(const ClassName&) = delete;`
* `ClassName& operator=(const ClassName&) = delete;`

## 右值引用

### 左值 & 右值

| 特性    | 左值       | 右值        |
| ----- | -------- | --------- |
| 可否赋值  | 可以       | 不可以（普通右值） |
| 可否取地址 | 可以       | 不行（普通右值）  |
| 生命周期  | 至少到作用域结束 | 临时对象，通常很短 |
| 举例    | 变量名、数组名  | 字面量、表达式结果 |

* 左值 lvalue：有名称，可寻址的对象，生命周期至少到作用域结束
* 纯右值 prvalue：无名称，临时对象，生命周期通常很短
* 将亡值 xvalue：即将被销毁的对象，通常是右值引用类型

### 左值引用，常量引用，右值引用

* 非常量引用`Type &`：只能绑定到左值，避免悬空引用
* 常量引用`const Type &`：可以绑定到左值和右值，延长右值的生命周期至引用的作用域结束
  * 限制：不能修改引用对应的对象
* 右值引用：`Type &&`：只能绑定到右值
  * 用于实现移动语义，提升性能（所指的对象不再有用，可以被安全地消费/破坏，无需管对象死活）
  * 若想绑定到左值，需使用`std::move`将左值转换为右值

```cpp
class A
{
  int val;
  public:
  void setVal(int v) { val = v; }
};

A getA() { return A(); }

int main()
{
  int a = 1;
  int &ra = a;           // OK
  const A &cra = getA(); // OK
  A &rA = getA();        // Error: binding non-const lvalue reference to rvalue
  A &&aa = getA();       // OK
  aa.setVal(2);          // OK
  A &&ba = a;            // Error: cannot bind rvalue reference to lvalue
  ba = std::move(a);     // OK
}
```

* 在函数内部，右值引用的参数被视为左值

```cpp
void process(int &&r) {}
void handle(int &&rvalue) { process(rvalue); }            // Error: 不能将左值传递给需要右值引用的参数
void handle(int &&rvalue) { process(std::move(rvalue)); } // OK
```

## 移动构造函数

* 拷贝：右侧的对象是左值，需要保留右侧对象不变
  * `ClassName(const ClassName &);`
  * `ClassName& operator=(const ClassName &);`
* 移动：右侧的对象是右值，需要将资源“移走”，无需管右侧对象的死活
  * `ClassName(ClassName &&) noexcept;`
  * `ClassName& operator=(ClassName &&) noexcept;`
* 作用：避免不必要的深拷贝，提升性能
* 移动构造和拷贝构造都要检查是否自我赋值

```cpp
class MyArray {
public:
    // 拷贝赋值
    MyArray& operator=(const MyArray& other) {
        if (this == &other)
            return *this;

        if (arr) {
            delete[] arr;
            arr = nullptr;
        }

        size = other.size;
        arr = new int[size];        // 原代码缺这行，否则 memcpy 写野指针
        memcpy(arr, other.arr, size * sizeof(int));

        return *this;
    }

    // 移动赋值
  MyArray& operator=(MyArray&& other) {
      if (this == &other)
          return *this;

      delete[] arr;      // 释放旧资源

      size = other.size;
      arr = other.arr;

      other.arr = nullptr;
      other.size = 0;

      return *this;
  }
};
```

## 拷贝构造和移动构造的规则

### Rule of Three

* 如果类定义了以下任意一个，则通常需要定义全部三个
  1. 析构函数
  2. 拷贝构造函数
  3. 拷贝赋值运算符

### Rule of Five

* 如果类定义了以下任意一个，则通常需要定义全部五个
  1. 析构函数
  2. 拷贝构造函数
  3. 拷贝赋值运算符
  4. 移动构造函数
  5. 移动赋值运算符

### Rule of Zero

* 如果类无需管理资源（如动态内存、文件句柄等），则不需要定义上述五个函数

## 动态对象

* `new`：`Type *p = new Type(args);`
  * 分配内存，调用构造函数，返回指向新对象的指针
  * 内置类型：也可使用 `new`，若不初始化可能为垃圾值
  * 若分配失败，抛出`std::bad_alloc`异常
* `delete`：`delete p;`
  * 调用析构函数，释放内存
  * `delete` 后建议将指针设为`nullptr`，避免悬空指针
* `new[]`：`Type *p = new Type[n];`
  * 分配内存，调用默认构造函数，返回指向数组首元素的指针
* `delete[]`：`delete[] p;`
  * 调用每个元素的析构函数，释放内存
  * 对于 `new[]` 创建的数组，必须使用`delete[]` 销毁，否则只能调用第一个对象的析构函数，导致资源泄漏
* 和 `malloc` / `free` 的区别
  * `malloc` / `free` 仅分配和释放内存，不调用构造和析构函数
  * `new` 可重载

```cpp
char **chArray2;

// allocate the rows
chArray2 = new char *[ROWS];

// allocate the (pointer) elements for each row
for (int row = 0; row < ROWS; row++)
  chArray2[row] = new char[COLUMNS];

// Reverse the creation algorithm to delete
for (int row = 0; row < ROWS; row++) {
  delete[] chArray2[row];
  chArray2[row] = NULL;
}

delete[] chArray2;
chArray2 = nullptr;
```

## `const`

### `const` 成员变量

* 不可修改
* 初始化必须在成员初始化列表中完成

### `const` 成员函数

* 不修改对象状态
* 在成员函数声明后加 `const`：`void func() const;`
* 相当于`func(const ClassName * this)`
* `const` 函数可以修改 `mutable` 成员变量
* `const` 成员函数与非 `const` 成员函数可以重载，是两个不同的函数：`void func() const;` 和 `void func();`
* `const` 对象只能调用 `const` 成员函数（编译时检查）
* 非 `const` 对象可以调用 `const` 和非`const` 成员函数

```cpp
struct Fib {
  Fib(int n) : n_(n) {}
  int value() const
  {
    if (!cached) {
      cache = fib(n);
      cached = true;
    }
    return cache;
  }
  int n_;
  mutable bool cached = false;
  mutable int cache = 0;
}
```

### `constexpr` 和 `consteval`

| 特性        | `constexpr` (常量表达式)   | `consteval` (即时函数)  |
| --------- | --------------------- | ------------------- |
| **引入标准**  | C++11                 | C++20               |
| **核心含义**  | 表示一个值或函数**可以在编译时计算**。 | 表示一个函数**必须在编译时计算**。 |
| **计算时机**  | 编译时 或 运行时 (取决于上下文)    | 只能是编译时              |
| **函数返回值** | 编译时或运行时可用             | 一定是编译时常量            |
| **关键字位置** | 可用于变量、函数、构造函数等        | **仅用于函数**           |

* 被 `constexpr` 和 `consteval` 修饰的函数不得包含任何有运行时行为的代码：I/O 操作、动态内存分配、异常处理等

```cpp
constexpr int sqr(int x) { return x * x; }
constexpr int A = sqr(10);
int y = 3;
int B = sqr(y);
consteval int pow2(int n) { return 1 << n; }
constexpr int M = pow2(8); // OK，编译期计算
int r = pow2(y);           // ERROR，运行期调用不允许
```

## `static`

### `static` 成员变量

* 所有对象共享同一份数据
* `static` 变量需要在类外初始化
  * `inline static` 变量可在类内初始化（C++17 起支持）
  * `const static` 整型或枚举类型变量可在类内初始化
  * `const static` 非整型变量需在类外初始化

### `static` 成员函数

* **只能访问静态成员变量和静态成员函数**
* 无 `this` 指针

```cpp
// 1. 通过对象使用（不推荐）
A a;
a.f();

// 2. 通过类名使用（推荐）
A::f();
```

```cpp
class A
{
  static int obj_count; // 也可写为 inline static int obj_count = 0; (C++17 起)
  const static int SIZE = 100;     // ✅ 整型或枚举类型，可以在类内初始化
  const static double PI;          // ❌ 非整型，必须在类外定义

  public:
  A() { obj_count++; }
  ~A() { obj_count--; }
  static int get_num_of_obj();
};

int A::obj_count = 0; // 在类外初始化
int A::get_num_of_obj() { return obj_count; } 
const double A::PI = 3.14159; // 在类外初始化
```

## 友元

* 作用：在类外开放 `private` 和 `protected` 成员的访问权限
  * 提高程序设计灵活性
  * 数据保护和数据存取效率的折中
* 分类
  * 友元函数
  * 友元类
  * 友元类成员函数
* 符合迪米特法则
* 友元的关系是严格且单向的，不能传递
  * 友元的友元不是友元
  * 类的派生类不是友元

```cpp
void func();
class B;
class C
{
  void f();
};
class A
{
  friend void func(); // 友元函数
  friend class B;     // 友元类
  friend void C::f(); // 友元类成员函数
};
```

```cpp
class Base {
protected:
    int prot_mem;
};

class Sneaky : public Base {
    friend void clobber(Sneaky&);
    friend void clobber(Base&);
    int j;
};

void clobber(Sneaky &s) {
    s.j = 0;          // OK，友元访问 Sneaky 的 private
    s.prot_mem = 0;   // OK，友元访问 Sneaky 基类继承来的 protected
}

void clobber(Base &b) {
    b.prot_mem = 0;   // 错误！clobber(Base&) 不是 Base 的友元
}
```
