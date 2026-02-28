# 90-看代码写结果

## 指针 & 数组

```cpp
char* GetString1()
{
  char p[] = "Hello World";
  return p;
}
 
char* GetString2()
{
  char *p = "Hello World";
  return p;
}
 
int main(){
  cout<<"GetString1 returns:"<< GetString1()<<endl; // 悬空指针，函数内将 "Hello World" 拷贝在栈上，函数返回后该内存不可用
  cout<<"GetString2 returns:"<< GetString2()<<endl; // 正确，指针指向字符串常量区
  
  char str1[] = "abc";
  char str2[] = "abc";

  const char str3[] = "abc";
  const char str4[] = "abc";

  const char *str5 = "abc";
  const char *str6 = "abc";

  char *str7 = "abc";
  char *str8 = "abc";

  cout << (str1 == str2) << endl;  
  cout << (str3 == str4) << endl;  
  cout << (str5 == str6) << endl;  
  cout << (str7 == str8) << endl;  
}
```

* `str1` 和 `str2`：不同数组，比较地址，结果为 `0`
* `str3` 和 `str4`：不同数组，比较地址，结果为 `0`
* `str5` 和 `str6`：指向同一字符串常量，比较地址，结果为 `1`
* `str7` 和 `str8`：指向同一字符串常量，比较地址，结果为 `1`

```cpp
int main()  
{ 
  int m[3][3]={ {1}, {2}, {3} }, n[3][3]={ 1, 2, 3 };
  cout << m[1][0]+n[0][0] << " " << m[0][1]+n[1][0] << endl;  
}
```

* `3 0`
* `n` 相当于 `{{1,2,3},{0,0,0},{0,0,0}}`

```cpp
int main()
{
  int a[5] = {1, 2, 3, 4, 5};
  int *ptr = (int *)(&a + 1);
  cout << *(a + 1) << ", " << *(ptr - 1) << endl;
}
```

* `2, 5`
* 增加的是一整个数组的大小

## 函数

> 函数匹配规则

* 重载
  1. 精确匹配优先
  2. 类型提升：仅限 `char/bool/short` 提升为 `int`
  3. 类型转换
     * 算术类型转换：`int` -> `float`，`float` -> `double`
     * 指针类型转换：`Derived*` -> `Base*`
     * `const` 修饰符转换
  4. 用户自定义类型转换（如类的转换构造函数或转换运算符）
  5. 可变参数
* 若派生类有同名函数，**即使参数列表不同**，也会隐藏基类同名函数，优先从派生类查找
  * 解决方法：`using Base::func;` 将基类函数引入派生类作用域
* 若默认参数的函数调用存在歧义：编译错误

```cpp
void func(long a, long b)
{
  cout << "long";
}
void func(double a, double b)
{
  cout << "double";
}

int main(int argc, const char *argv[])
{
  int a = 1, b = 1;
  func(a, b);
  return 0;
}
```

* 编译错误

```cpp
template <class T>
T max(T a, T b)
{
  return a > b ? a : b;
}

double max(int a, double b)
{
  return a > b ? a : b;
}

int x;
double m;
double result = max(x, m); // 调用 double max(int, double)
```

> Lambda 表达式的捕获方式

* 不捕获任何变量：`[]`
* 按值捕获所有变量：`[=]`
* 按引用捕获所有变量：`[&]`
* 按值捕获特定变量：`[a]`
* 按引用捕获特定变量：`[&a]`
* 按值捕获所有变量，按引用捕获特定变量：`[=, &b]`
* 混合捕获：`[a, &b]`

## 模板

### 编译

```cpp
// file1.cpp
#include "file1.h"
template <class T>
void S<T>::f() {}

template <class T>
T max(T x, T y)
{
  return x > y ? x : y;
}

void main()
{
  int a, b;
  max(a, b);
  S<int> x;
  x.f();
}
// file1.h
template <class T>
class S
{
  T a;

  public:
  void f();
};
// file2.cpp
#include "file1.h"
extern double max(double, double);
void sub()
{
  max(1.1, 2.2);
  S<float> x;
  x.f();
}
```

* 编译错误
  * `max(1.1, 2.2)`：找不到 `max<double>(double, double)` 的定义
  * `x.f()`：找不到 `S<float>::f()` 的定义
* 解决方法：将声明和实现放在同一个头文件中

## 异常

```cpp
void f()
{
  try {
    g();
  } catch (int) {

  } catch (char *) {
  }
}
void g()
{
  try {
    h();
  } catch (int) {
  }
}
void h()
{
  if (some_condition)
    throw 1; // 由g捕获并处理
  else
    throw "abcd"; // 由f捕获并处理
}
```

```cpp
class BaseException { };
class DerivedException : public BaseException { };

void f()
{
  try {
    throw DerivedException();
  } catch (BaseException &e) {
    cout << "Caught BaseException" << endl;
  } catch (DerivedException &e) {
    cout << "Caught DerivedException" << endl;
  }
}
```

* 输出 `Caught BaseException`
* 派生类异常对象被基类引用捕获，匹配第一个 `catch`

## 左值 & 右值

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

```cpp
template <typename T>
void process(T &x) { cout << "Lvalue processed: " << x << endl; }
template <typename T>
void process(T &&x) { cout << "Rvalue processed: " << x << endl; }
template <typename T>
void forward(T &&x) { process<T>(x); }
template <typename T>
void forwardCorrect(T &&x) { process(std::forward<T>(x)); }

int main()
{
  int a = 5;
  forward(a);
  forward(20);
  forwardCorrect(a);
  forwardCorrect(20);
  return 0;
}
```

```plaintxt
Lvalue processed: 5
Lvalue processed: 20
Lvalue processed: 5
Rvalue processed: 20
```

* 无论是左值还是右值传入，作为函数参数都是左值表达式
* `std::forward<T>(x)` 保留了传入参数的值类别，实现完美转发

## 面向对象

### 构造 / 析构

```cpp
class A
{
  int x;
  const int y;
  int &z;

  public:
  A() : y(1), z(x), x(0)
  {
    cout << "在构造函数体内：" << "x = " << x << ", y = " << y << ", z = " << z << endl;
    x = 100;
    cout << "赋值后 x = " << x << ", y = " << y << ", z = " << z << endl;
  }
};
```

* `在构造函数体内：x = 0, y = 1, z = 0`
* `赋值后 x = 100, y = 1, z = 100`

```cpp
class A {
  public:
  explicit A(int x) {}
};

void f() {
  A a1(10);
  A a2 = 10; 
}
```

* `A a2 = 10;` 编译错误
* `explicit` 防止隐式类型转换

```cpp
struct B {
  int x;
};

struct C {
  int x;
  C() : x(42) {}
};

B b{};
C c{};

int i{};
double d{};

int main()
{
  cout << "B.x: " << b.x << ", C.x: " << c.x << endl;
  cout << "i: " << i << ", d: " << d << endl;
  return 0;
}
```

* `B.x: 0, C.x: 42`
* `i: 0, d: 0`

```cpp
class base
{
  private:
  int m_i;
  int m_j;

  public:
  base(int i) : m_j(i), m_i(m_j) {}
  base() : m_j(0), m_i(m_j) {}
  int get_i() { return m_i; }
  int get_j() { return m_j; }
};

int main()
{
  base obj(98);
  cout << obj.get_i() << endl  
       << obj.get_j() << endl;
}
```

* `随机数 98`
* 变量初始化的顺序是**按成员声明顺序**，而非初始化列表顺序

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

* 异常安全性：把可能失败的操作放在前面，成功后再修改对象状态

```cpp
#include <iostream>
using namespace std;
class B
{
  public:
  B() { cout << "+B" << endl; }
  virtual ~B() { cout << "-B" << endl; }
};

class C
{
  public:
  C() { cout << "+C" << endl; }
  ~C() { cout << "-C" << endl; }
};

class D : public B
{
  C c;

  public:
  D() { cout << "+D" << endl; }
  ~D() { cout << "-D" << endl; }
};

int main()
{
  B *p = new D;
  delete p;
}
```

* `+B +C +D -D -C -B`

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
B b2(b1);
cout << b2.x << "," << b2.y << "," << b2.z << endl;
```

* `0,0,1`
* 若自定义拷贝构造函数，而没有在拷贝构造函数的初始化列表中提及成员：和默认构造函数机制相同
  * 内置类型成员：不初始化，可能为垃圾值
  * 类类型成员：调用对应类的**默认构造函数**（不是拷贝构造函数）

```cpp
class C
{
  public:
  C(int i, int j)
  {
    c1 = i;
    c2 = j;
  }
  void Sum(C a, C b)
  {
    c1 = a.c1 + b.c1;
    c2 = a.c2 + b.c2;
  }
  void Print() { cout << "c1=" << c1 << ',' << "c2=" << c2 << endl; }

  private:
  int c1, c2;
};
int main()
{
  C a(6, 9);
  C b(a);
  C c(b);
  c.Sum(a, b);
  c.Print(); 
}
```

* `c1=12,c2=18`
* 调用默认的拷贝构造函数

```cpp
class A
{
  int i, j;

  public:
  A() { i = j = 0; }
};
class B
{
  A *p;

  public:
  B() { p = new A; }
  ~B() { delete p; }
};
void f(B x)
{
}
int main()
{
  B b;
  f(b);
}
```

* 运行时发生错误
* `f(b)` 中的参数是浅拷贝
* `b` 和 `x` 的成员指针 `p` 指向同一块内存
* `f(b)` 结束销毁 `x`，`main` 结束销毁 `b`，导致重复释放内存，产生未定义行为

```cpp
class A
{
  public:
  A(int i)
  {
    a = i;
  }
  A()
  {
    a = 0;
    cout << "Default constructor called." << a << endl;
  }
  ~A()
  {
    cout << "Destructor called." << a << endl;
  }
  void Print()
  {
    cout << a << endl;
  }

  private:
  int a;
};
int main()
{
  A a[4], *p;
  int n = 1;
  p = a;
  for (int i = 0; i < 4; i++)
    a[i] = A(++n);
  for (int i = 0; i < 4; i++)
    (p + i)->Print();
}
```

```plaintxt
Default constructor called.0
Default constructor called.0
Default constructor called.0
Default constructor called.0
Destructor called.2 // 赋值时临时产生的右值对象被析构
Destructor called.3
Destructor called.4
Destructor called.5
2
3
4
5
Destructor called.5
Destructor called.4
Destructor called.3
Destructor called.2
```

* 构造和析构的顺序是相反的，早构造的晚析构

```cpp

class C
{
  public:
  C(int i)
  {
    c = i;
  }
  C()
  {
    c = 0;
    cout << "Default constructor called." << c << endl;
  }
  ~C() { cout << "Destructor called." << c << endl; }
  void Print() { cout << c << endl; }

  private:
  int c;
};
int main()
{
  C *p;
  p = new C[4];
  int n = 1;
  for (int i = 0; i < 4; i++)
    p[i] = C(n++);
  for (int i = 0; i < 4; i++)
    p[i].Print();
  delete[] p;
}
```

```plaintxt
Default constructor called.0
Default constructor called.0
Default constructor called.0
Default constructor called.0
Destructor called.1 # 赋值时临时产生的右值对象被析构
Destructor called.2
Destructor called.3
Destructor called.4
1
2
3
4
Destructor called.4 # `delete[]` 从后向前析构数组元素
Destructor called.3
Destructor called.2
Destructor called.1
```

```cpp
class A
{
  public:
  A() { cout << "in A 's constructor\n"; };
  ~A() { cout << "in A 's destructor\n"; };
};
class B
{
  public:
  B() { cout << "in B 's constructor\n"; };
  ~B() { cout << "in B 's destructor\n"; };
};
class C : public B
{
  private:
  A a;

  public:
  C() { cout << "in C 's constructor\n"; };
  ~C() { cout << "in C 's destructor\n"; };
};
int main()
{
  A a;
  B *p = new C;
  delete p;
}
```

```plaintxt
in A 's constructor
in B 's constructor
in A 's constructor
in C 's constructor
in B 's destructor
in A 's destructor
```

* 基类的析构函数没设为虚函数的下场🤣

### `static`

```cpp
class B
{
  public:
  B() { cout << ++b << ' '; }
  ~B() { cout << b-- << ' '; }
  static int Getb() { return b; }

  private:
  static int b;
};
int B::b = 10;
int main()
{
  B b1, b2, b3; 
  cout << B::Getb() << ' '; 
} 
```

* `11 12 13 13 13 12 11`

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

### 友元

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

* 友元可访问所处类的 `private` 和 `protected` 成员
* 友元的基类/派生类不是友元
* 友元的友元不是友元

### 虚函数

```cpp
class A
{
public:
virtual void Fun(int number = 10)
  {
      cout << "A::Fun with number " << number;
  }
};

class B: public A
{
public:
virtual void Fun(int number = 20)
  {
    cout << "B::Fun with number " << number;
  }
};
 
int main()
{
  B b;
  A &a = b;
  a.Fun(); 
  return 0;
}
```

* `B::Fun with number 10`
* 默认参数在**编译时静态绑定**
* 虚函数选择在**运行时动态绑定**

```cpp
class A
{
  public:
  virtual void fun1()
  {
    cout << "调用A::fun1()\n";
  }
  void fun2()
  {
    fun1();
  }
};

class B : public A
{
  public:
  void fun1()
  {
    cout << "调用B::fun1()\n";
  }
};

void print(A &a)
{
  a.fun1();
}

void main()
{
  A a, *pa;
  B b;

  b.fun1(); // B::fun1()

  pa = &a;
  pa->fun1(); // A::fun1()

  pa = &b;
  pa->fun1(); // B::fun1()

  print(a); // A::fun1()
  print(b); // B::fun1()
}
```

```cpp
class A
{
  public:
  A() { cout << "In A cons.\n"; }
  virtual ~A() { cout << "In A des.\n"; }
  virtual void f1() { cout << "In A f1().\n"; }
  void f2() { f1(); }
};
class B : public A
{
  public:
  B()
  {
    f1();
    cout << "In B cons.\n";
  }
  ~B() { cout << "In B des.\n"; }
};
class C : public B
{
  public:
  C() { cout << "In C cons.\n"; }
  ~C() { cout << "In C des.\n"; }
  void f1() { cout << "In C f1().\n"; }
};

int main()
{
  A *pa = new C;
  pa->f2();
  delete pa;
}
```

```plaintxt
In A cons.
In A f1().
In B cons.
In C cons.
In C f1().
In C des.
In B des.
In A des.
```

* 在构造函数过程中，调用虚函数会调用当前构造的类的版本，而非派生类的版本
* 由于 `B::f1()` 未定义，调用 `A::f1()`

### 权限

```cpp
class Base
{
  protected:
  int value;

  public:
  Base(int v) : value(v) {}
};

class Derived : public Base
{
  private:
  int privateValue;

  public:
  Derived(int v) : Base(v), privateValue(v + 10) {}

  void access(Base &b, Derived &d)
  {
    cout << "Base value: " << b.value << endl;                   // 访问基类的protected成员
    cout << "Derived value: " << d.value << endl;                // 访问派生类的protected成员
    cout << "Derived private value: " << d.privateValue << endl; // 访问派生类的private成员
  }
};
```

* `Base value` 行编译失败：不能访问别人的基类的 `protected` 成员，瞎认爹是不行的
