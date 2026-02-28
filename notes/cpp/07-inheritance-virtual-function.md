# 07-继承 & 虚函数

## 继承

* 基于目标代码的复用
* 对事物进行分类
  * 派生类是基类的具体化
  * 把事物（概念）以层次结构表示出来，有利于描述和解决问题
* 增量开发

### 声明

```cpp
class DerivedClass;
// 不能写 class DerivedClass : [继承方式] BaseClass;
class DerivedClass : [继承方式] BaseClass {
// 类体
};
```

### 访问控制

| 权限   | `public` | `protected` | `private` |
| ---- | -------- | ----------- | --------- |
| 同一个类 | 可访问      | 可访问         | 可访问       |
| 派生类  | 可访问      | 可访问         | 不可访问      |
| 类外部  | 可访问      | 不可访问        | 不可访问      |

> Liskov 替换原则：子类对象必须能够替换掉所有父类对象出现的地方（is-a 关系）

### 继承方式

* 继承后的权限 = `min(继承方式, 基类中成员的权限)`
* 权限是对于**类外部的访问控制**，派生类本身还是可以访问基类的 `public` 和 `protected` 成员
* `public` 继承必须满足 is-a 关系：否则产生逻辑问题（`Square` 继承自 `Rectangle` 是错误的设计，不能单独改变正方形的高/宽）
* `private` 继承：在实现层面复用代码，不希望对用户暴露基类的接口

| 继承方式        | 基类的 `public` 成员 | 基类的 `protected` 成员 | 基类的 `private` 成员 |
| ----------- | --------------- | ------------------ | ---------------- |
| `public`    | `public`        | `protected`        | 不可访问             |
| `protected` | `protected`     | `protected`        | 不可访问             |
| `private`   | `private`       | `private`          | 不可访问             |

### 构造 & 析构

* 构造执行顺序
  * 虚基类的构造函数（见下文）
  * 基类构造函数：多继承时，按声明顺序调用（见下文）
  * 构造派生类的所有成员：若为对象成员类，执行构造函数
  * 执行派生类构造函数体
* 默认执行基类的默认构造函数 `BaseClass()`
* 若需执行非默认构造函数，需在 **派生类构造函数初始化列表** 中指定
  * 派生类的对象成员类的构造函数
  * 派生类构造函数
* 析构顺序：相反

```cpp
class A
{
  int x;

  public:
  A() { x = 0; }
  A(int i) { x = i; }
};
class B : public A
{
  int y;

  public:
  B() { y = 0; }
  B(int i) { y = i; }
  B(int i, int j) : A(i)
  { y = j; }
};
B b1;   // 执行A::A()和B::B()
B b2(1);// 执行A::A()和B::B(int)
B b3(0, 1); // 执行A::A(int)和B::B(int,int)
```

### 类型相容

* 派生类对象可以赋值给基类对象：赋值时切片，截取基类对象中的成员
  * 内存布局：基类对象，派生类对象
* 基类指针可以指向派生类对象
* 基类指针转换为派生类指针：无法直接转换，需使用 `dynamic_cast` 进行安全转换
* 不要使用原生数组存储派生类子对象：会**产生切片**，导致派生类的信息丢失
  * 正确做法：使用指针数组

```cpp
class Base {};
class Derived : public Base {};

Derived d;
Base b = d;  // OK
d = b;   // Error: cannot convert Base to Derived

Base *pb = &d; // OK
Derived *pd = pb; // Error: cannot convert Base* to Derived*

Base &rb = d; // OK
Derived &rd = rb; // Error: cannot convert Base& to Derived&
```

## 虚函数

### 静态绑定 & 动态绑定

* 无法确定使用哪个函数的情况
  * Overloading：函数名称相同，传入参数不同；编译时即可确定具体调用的函数
  * Overriding：父类子类都有同名函数；可在编译时/运行时确定具体调用的函数
* 前期绑定/静态绑定：在**编译期**确定函数调用
  * 根据变量**声明的类型**
  * C++ 默认非虚函数，静态绑定
  * 效率高，灵活性差
* 动态绑定：在运行期确定函数调用
  * 根据变量**实际的类型**
  * C++ 中如需实现动态绑定，需使用 `virtual` 关键字
  * Java 中的普通方法调用都是动态绑定：`invokevirtual`
  * 灵活性高，效率低

```cpp
class A
{
  int x, y;

  public:
  void f();
  virtual void g();
};
class B : public A
{
  int z;

  public:
  void f();
  void g();
};

func1(A &a)
{
  a.f();
  a.g();
}
func2(A *pa)
{
  pa->f();
  pa->g();
}
func1(b);   // 调用 A::f() 和 B::g()
func2(&b);  // 调用 A::f() 和 B::g()
```

### 虚函数的性质

* 如基类中被定义为虚成员函数，则派生类中对其重定义的成员函数均为虚函数
* 限制
  * 类的成员函数可以是虚函数
  * 构造函数、`static` 成员函数、`inline` 成员函数不能是虚函数
  * 析构函数往往是虚函数：通过基类指针删除派生类对象时，能正确调用派生类的析构函数

### 虚函数表

* 在类的内存布局最前方增加一个**指向虚函数表的指针** `vptr`
* 虚函数表 `vtable`：存放虚函数地址的数组
* 每个类有一个虚函数表，派生类的虚函数表会覆盖基类的虚函数表
* 对象创建时，`vptr` 指向对应类的虚函数表
* 通过基类指针调用虚函数时，先通过 `vptr` 找到虚函数表，再根据偏移找到具体的函数地址并调用

```cpp
class A
{
  public:
  A() { f(); }
  virtual void f();
  void g();
  void h()
  {
f();
g();
  }
  virtual void m();
};

// vtable_A:
// [0] &A::f
// [1] &A::m

class B : public A
{
  public:
  void f() { g(); }
  void g();
};

// vtable_B:
// [0] &B::f
// [1] &A::m

B b;
A *p = &b; // A::A()，A::f, B::B(),
p->f();// B::f, B::g
p->g();// A::g
p->h();// A::h, B::f, A::g

// g() 不是虚函数，通过基类指针调用时，仍然是静态绑定，取决于 `this` 指针的类型
```

### `override` `final`

* `override`：**显式指定**重写基类的虚函数
  * 若基类无完全匹配的虚函数：编译错误
  * 作用：编译时检查，避免由于类型撰写错误，导致重载而非重写虚函数
* `final`：指定虚函数不能被重写，或者类不能被继承

### 虚函数 & 访问控制

* 访问控制不影响虚函数的动态绑定机制
* 通过基类指针调用虚函数时，**仍然遵循访问控制规则**
* 原因：编译时检查访问权限/默认参数，运行时决定调用哪个函数

```cpp
struct B {
  protected:
  virtual void f() {}
};
struct D : B {
  public:
  void f() override {} // 放宽为 public
};
int main()
{
  D d;
  d.f(); // 正确，D::f 是 public
  B *pb = &d;
  pb->f(); // 错误，B::f 是 protected
}
```

### 纯虚函数 & 抽象类

* 纯虚函数：只有生命，没有实现，`virtual int f() = 0;`
* 抽象类：包含纯虚函数的类
  * **不能实例化对象**
  * 只能作为基类使用
* 派生类必须**重写所有纯虚函数**，才能实例化对象，否则仍然是抽象类

> 使用场景：抽象工厂模式

### 虚析构函数

* 基类的析构函数应声明为虚函数：`virtual ~BaseClass() = default;`
* 通过基类指针删除派生类对象时，能正确调用派生类的析构函数
* 否则只会调用基类的析构函数，导致资源泄漏

## 继承 & 虚函数

* 不要定义与继承而来的**非虚成员函数同名**的新成员函数
  * 如需体现多态性，必须使用虚函数
  * 否则会静态绑定，当指针类型为基类时，调用的仍然是基类的函数，破坏预期的多态性

| 函数类型           | 是否继承接口 | 是否继承实现  | 子类是否必须 override | 多态支持        |
| -------------- | ------ | ------- | --------------- | ----------- |
| 纯虚函数（`=0`）     | 是      | 否       | 是               | 是           |
| 虚函数（`virtual`） | 是      | 是（默认实现） | 否，若有额外需求可覆盖     | 是           |
| 非虚函数           | 是      | 是（固定实现） | 否，不应覆盖基类实现      | 否（静态绑定，非多态） |

* 不要重新定义继承来的默认参数
  * 虚函数运行时动态绑定
  * **默认参数编译时静态绑定**
  * 若重新定义默认参数，可能导致调用时使用了错误的默认参数值

```cpp
class A
{
  public:
  virtual void f(int x = 0) = 0;
};

class B : public A
{
  public:
  virtual void f(int x = 1) { cout << x; }
};

class C : public A
{
  public:
  virtual void f(int x) { cout << x; }
};

A *p_a;
B b;
p_a = &b;
p_a->f(); // 输出 0

A *p_a1;
C c;
p_a1 = &c;
p_a1->f(); // 输出 0
```

* 无法将构造函数声明为虚函数，但是可以声明一个 `clone` 纯虚函数，用于复制对象

## 多继承

* `class Derived : [继承方式] Base1, [继承方式] Base2 { ... };`
* 继承方式同单继承
* 若存在同名成员，需使用作用域限定符进行区分 `Base1::member` `Base2::member`
* 基类的声明次序决定
  * 内存布局：按声明顺序排列基类子对象，最后为派生类成员
  * 构造顺序：按声明顺序调用基类构造函数
  * 析构顺序：与构造顺序相反

### 菱形继承 & 虚基类

```
  A
 / \
B   C
 \ /
  D
```

* 带来的问题：`B` 和 `C` 都继承自 `A`，`D` 会有两个 `A` 的子对象
  * 内存浪费
  * 二义性：通过 `D` 访问 `A` 的成员时，编译器无法确定使用哪个 `A` 子对象
* 解决方案：使用虚基类 `virtual` 关键字，使最终的派生类只包含一个基类子对象
* 虚基类的构造函数由最新派生出的类的构造函数调用
  * `D` 直接调用 `A` 的构造函数，`B` 和 `C` 不再调用 `A` 的构造函数
  * 若 `A` 有参数化构造函数，需在 `D` 的构造函数初始化列表中指定参数
* 构造顺序：虚基类最先构造，然后按声明顺序构造非虚基类，最后构造派生类

```cpp
class A { public: A(){cout<<"A ";} };
class B : virtual public A { public: B(){cout<<"B ";} };
class C : virtual public A { public: C(){cout<<"C ";} };
class D : public B, public C { public: D(){cout<<"D ";} };

int main() { D d; }
// 输出： A B C D
```
