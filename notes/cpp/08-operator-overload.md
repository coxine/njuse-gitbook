# 08-操作符重载

## 多态

* 同一个域中，一个元素有多种解释：提高语言灵活性
  * 函数重载：静态
    * 匹配规则：参考 04，编译器确定
  * Template：静态
  * 虚函数：动态
  * 操作符重载
    * 内置类型：操作符有固定含义，编译器确定
    * 自定义类型：程序员定义
      * 提高可读性：`a.add(b)` vs `a + b`
      * 提高可扩充性：便于集成到模板中

## 重载规则

* 至少有一个操作数是自定义类型（`new` `delete` 除外）：防止修改内置类型行为
* 重载不改变操作符优先级
* 重载不改变操作符结合性
* 不可重载的运算符
  * `::` 作用域解析运算符
  * `.` 成员访问运算符
  * `.*` `->*` 成员指针访问运算符
  * `?:` 条件运算符
  * `sizeof` 大小运算符
  * `typeid` 类型信息运算符
  * `#` 预处理运算符

## 双目操作符重载

* 成员函数：`retType operator Op (const ClassName &rhs);`
  * **无法隐式转换**
  * 使用：`obj1 0p obj2` 等价于 `obj1.operator Op(obj2)`
* 友元函数：`friend retType operator Op (const ClassName &lhs, const ClassName &rhs);`
  * 无法重载 `=` `[]` `()` `->` 操作符
    * `=` 编译器会默认生成，类外定义无法覆盖默认生成的版本
    * `[]` 防止外部代码注入索引的逻辑
    * `()` 防止函数调用存在歧义
    * `->` C++ 递归地解析成员访问直至找到实际对象，类外定义无法实现递归解析
* 全局函数：`retType operator Op (const ClassName &lhs, const ClassName &rhs);`
  * 无法重载 `=` `[]` `()` `->` 操作符

```cpp
class A {
private:
    int value;
public:
    A(int v) : value(v) {}

    // --- 方案 A: 成员函数版本 ---
    A operator+(const A& rhs) const { 
        return A(this->value + rhs.value); 
    }

    // --- 方案 B: 友元函数版本 ---
    friend A operator-(const A& lhs, const A& rhs);
};

A operator-(const A& lhs, const A& rhs) {
    return A(lhs.value - rhs.value);
}

int main() {
    A a(10), b(20);

    A c = a + b; // 使用重载的 + 操作符，等效于 a.operator+(b)
    
    A c1 = a - 5; // 正确：5 被隐式转换为 A(5)
    A c2 = 5 - a; // 只有【友元/全局版本】才支持，成员函数版本会报错
    
    return 0;
}
```

### 重载的建议

* 不应重载： `&&` `||`，破坏短路求值
* 类内实现
  * 赋值符号 `=` `+=` `-=` 等，修改类内状态
  * `=` `[]` `()` `->`
* 类外实现
  * 对称操作符 `+` `-` `*` `/` 等
  * 流输入输出 `<<` `>>`

### 重载流操作符

```cpp
enum Day { SUN, MON, TUE, WED, THU, FRI, SAT };

Day &operator++(Day &d)
{
  return d = (d == SAT) ? SUN : Day(d + 1);
};

ostream& operator << (ostream& o, const Day& d)
{ switch (d)
 { case SUN: o << "SUN" << endl;break;
  case MON: o << "MON" << endl;break;
  case TUE: o << "TUE" << endl;break;
  case WED: o << "WED" << endl;break;
  case THU: o << "THU" << endl;break;
  case FRI: o << "FRI" << endl;break;
  case SAT: o << "SAT" << endl;break;
 }
 return o; // 返回流对象以支持链式操作
}
```

```cpp
class Rational
{
  public:
  Rational(int, int);
  const Rational &operator*(const Rational &r) const;

  private:
  int n, d;
};

// * 返回什么？
// return Rational(n * r.n, d * r.d); 错误：临时对象，调用结束后被销毁
// return new Rational(n * r.n, d *r.d); 错误：在连续计算时 (x*y*z) 中间步骤会内存泄漏
// static Rational result;
// result.n = n * r.n;
// result.d = d * r.d;
// return result;
// 错误： ((a*b)==(c*d)) 时，result 被覆盖
// 正确：返回栈上对象，同时优化拷贝/赋值构造函数
// const Rational operator*(const Rational &r) const;
```

## 单目操作符重载

* 成员函数：`retType operator Op ();`
  * 使用：`obj 0p` 等价于 `obj.operator Op()`
* 友元函数：`friend retType operator Op (const ClassName &obj);`
* 全局函数：`retType operator Op (const ClassName &obj);`

### `a++` 与 `++a` 区别

* `++a`：`ClassName &operator++();`
  * 返回增加后的对象
  * 返回引用：支持链式调用 `++(++a)`，和 C++ 语义一致
* `a++`：`ClassName operator++(int);`
  * 返回增加前的对象
  * 返回值：返回旧值的拷贝，和 C++ 语义一致
  * `int` 参数：dummy 参数，用于区分前置和后置版本

```cpp
class Counter
{
  int value;

  public:
  Counter() { value = 0; }
  Counter &operator++() //  ++a
  {
    value++;
    return *this;
  }
  Counter operator++(int) // a++
  {
    Counter temp = *this;
    value++;
    return temp;
  }
}
```

## 特殊操作符重载

### `=` 操作符重载

* 默认赋值函数：编译器自动生成
  * 成员逐一赋值（浅拷贝，成员为指针时仅复制地址）
  * 对象成员：递归调用成员的赋值函数
  * 返回类型：`ClassName &`，解决 `a = b = c`链式赋值问题
  * **不可继承**：无法处理派生类新增的成员
* 重载 `=`：重载拷贝/移动赋值函数，见 06
  * 需要判断自赋值 `if (this == &rhs) return *this;`

### `[]` 操作符重载

* 可以实现类似数组的下标访问功能，同时增加边界检查等功能
* 需要重载两个版本来应对常量和非常量对象的访问
  * `char &operator[](int i);`：非常量版本，返回左值引用，可赋值
  * `const char operator[](int i) const;`：常量版本，返回右值，不可赋值

```cpp
class string
{
  char *p;
  string(char *p1)
  {
    p = new char[strlen(p1) + 1];
    strcpy(p, p1);
  }
  char &operator[](int i) { return p[i]; }
  const char operator[](int i) const { return p[i]; }

  virtual ~string() { delete[] p; }
};

const string s((char *)"hello");
s[2] = 'b';  // 错误：常量只能调用 const 版本，返回右值，无法赋值
string s2((char *)"hello");
s2[2] = 'b'; // 正确：调用非常量版本，返回 char&，可赋值
// 若给 char& operator[](int i) 加上 const，则编译错误，因为两个函数签名完全相同
```

* 代理类：原生重载无法实现 `a[i][j]` 二维数组访问，需要返回一个代理类
  * 相当于 `a.operator[](i).operator[](j)`

```cpp
class Array2D
{
  int *p;
  int num1, num2;

  public:
  class Array1D
  {
    int *p;

public:
    Array1D(int *p) { this->p = p; }
    int &operator[](int index) { return p[index]; }
    const int operator[](int index) const { return p[index]; }
  };
  Array2D(int n1, int n2)
  {
    p = new int[n1 * n2];
    num1 = n1;
    num2 = n2;
  }
  virtual ~Array2D() { delete[] p; }
  Array1D operator[](int index) { return p + index * num1; }
  const Array1D operator[](int index) const { return p + index * num1; }
};
```

### `()` 操作符重载

#### 函数对象

* 函数对象：重载了 `()` 操作符的类对象，可以像函数一样调用
* 拥有状态：通过成员变量保存
* 效率高：内联调用，避免函数调用开销
  * 函数指针由于地址指向内容的不确定性，无法内联
* Lambda 表达式：匿名函数对象，编译器自动生成类名
  * 创建函数对象的语法糖

```cpp
// 相较于 operator[] 能接受多个参数，无需代理类
int& operator()(int i, int j) { return (p + i * num2)[j]; }

Array2D arr(3, 4);
arr(1, 2) = 10; // 访问第 1 行第 2 列元素
```

```cpp
bool cmpInt(int a, int b) { return a < b; }
class CmpInt
{
  bool operator()(const int a, const int b) const
  {
    return a < b;
  }
};

int main()
{
  std::vector<int> items{4, 3, 1, 2};
  std::sort(items.begin(), items.end(), cmpInt); // 函数指针
  std::sort(items.begin(), items.end(), CmpInt()); // 函数对象
  std::sort(items.begin(), items.end(), [](int a, int b) { return a < b; }); // Lambda 表达式
  return 0;
}
```

#### 自定义类型转换

* 重载 `operator Type()` 实现自定义类型转换

```cpp
class Rational
{
  public:
  Rational(int n1, int n2)
  {
    n = n1;
    d = n2;
  }
  operator double() { return (double)n / d; }

  private:
  int n, d;
};
Rational r(1, 2);
double x = r;
x = x + r;
```

```cpp
class A {
public:
    operator int() { return 1; }
    operator double() { return 2.0; }
};

A a;
cout << a + 3 << endl;      // 存在二义性，报错
```

* 避免二义性：使用 `explicit` 关键字，禁止隐式转换，只允许显式转换

```cpp
class A {
public:
    explicit operator int() { return 1; }
    explicit operator double() { return 2.0; }
};

A a;
cout << a + 3 << endl;      // 错误：禁止隐式转换
cout << static_cast<int>(a) + 3 << endl; // 正确：显式转换
cout << static_cast<double>(a) + 3 << endl; // 正确：显式转换
```

### `->` 操作符重载

* `->` 本身是二元运算符
* 重载时视作单目操作符，不接受除了对象本身以外的参数
* 返回：指针 / 重载了 `operator->()` 的对象
* 重载仅对对象生效，对指针无效，指针正常解引用

```cpp
class CPen
{
  int m_color;
  int m_width;

  public:
  void setColor(int c) { m_color = c; }
  int getWidth() { return m_width; }
};

class CPanel
{
  CPen m_pen;
  int m_bkColor;

  public:
  CPen *operator->() { return &m_pen; }
  void setBkColor(int c) { m_bkColor = c; }
};

CPanel c;
c->setColor(16); // c.operator->()->setColor(16) / c.m_pen.setColor(16)
c->getWidth();   // c.operator->()->getWidth() / c.m_pen.getWidth()
CPanel *p = &c;
p->setBkColor(10); // 为指针，正常解引用
```

* 可以实现智能指针功能：封装指针，自动管理内存

```cpp
template <class T>
class auto_ptr
{
  public:
  auto_ptr(T *p = 0) : ptr(p) {}
  ~auto_ptr() { delete ptr; }
  T *operator->() const { return ptr; }
  T &operator*() const { return *ptr; }

  private:
  T *ptr;
};
```

### `new` 和 `delete` 操作符重载

* 作用：自定义内存分配和释放行为，提高效率
  * 调用系统存储分配，申请较大的内存
  * 针对该内存自行管理
* 静态成员函数
* 遵循类的访问控制
* 可被子类继承

```cpp
class A {
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void operator delete(void* p);
};
```

#### 重载 `new`

* `new ClassName(args);`
  * 调用 `operator new(sizeof(ClassName));` 分配内存
  * 调用 `ClassName::ClassName(args);` 构造对象
* `size`：根据构造函数所属的类类型，自动计算并传入
* 特殊版本
  * `void* operator new(std::size_t size, int x, double y);`：传入额外参数（是传给 `new` 的参数，不是传给构造函数的参数）
  * `void* operator new(std::size_t size, void* ptr) throw();`：Placement new，不重新分配内存，在 `ptr` 指向的内存上构造对象
  * `void* operator new(std::size_t size, const std::nothrow_t&) throw();`：失败时返回 `nullptr`，而不是抛出异常

#### 重载 `delete`

* `delete p;`
  * 调用 `ClassName::~ClassName();` 析构对象
  * 调用 `operator delete(p);` 释放内存
* 默认版本：`void operator delete(void* p);`
* 特殊版本：`void operator delete(void* p, size_t size);`：编译器自动计算并传入内存大小
* `delete` 的重载只有一个，重载后默认的版本将不可用

## 编译器默认生成的函数

* `ClassName()`：默认构造函数
* `ClassName(const ClassName &)`：拷贝构造函数
* `ClassName(ClassName &&)`：移动构造函数（C++11 起）
* `~ClassName()`：析构函数
* `ClassName &operator=(const ClassName &)`：拷贝赋值函数
* `ClassName &operator=(ClassName &&)`：移动赋值函数（C++11 起）
* `ClassName* operator&()`：取地址操作符
* `const ClassName* operator&() const`：常量取地址操作符
