# 05-杂项

## 头文件

* 存放声明，编译时不会生成代码/占用存储空间
  * 函数声明
  * 全局变量声明
* 常量定义：`inline constexpr` 或 `const`，避免重复定义
* 类型定义：`struct`、`class`、`enum`、`typedef`、`using`
* **模板的声明 + 实现**
* `inline` 函数定义

## 关键字

* `extern`：声明变量或函数在其他文件中定义
* `static`
  * 全局作用域下：限制变量或函数的可见性为当前文件
  * 类内：声明静态成员变量或函数，属于类而非对象实例
  * 函数内：变量只初始化一次，保持其值跨函数调用

## `namespace`

* 作用域：防止命名冲突

```cpp
// 定义命名空间
namespace MyNamespace {
    void myFunction();
    class MyClass {};
}

MyNamespace::myFunction(); // 使用命名空间成员

// using-declaration
using MyNamespace::myFunction;
myFunction(); // 直接使用

// using-directive，同一作用域内最好只使用一次
using namespace MyNamespace;
MyClass obj; // 直接使用

// 别名
namespace MN = MyNamespace;

// 开放，可以在任何地方添加新成员
namespace MyNamespace {
    void anotherFunction();
}

// 嵌套命名空间
namespace Outer {
    namespace Inner {
        void innerFunction();
    }
}

Outer::Inner::innerFunction();

// 重载
namespace B{
  myFunction(); // 与 MyNamespace::myFunction 不冲突
}
```

## 编译预处理

* `ifdef` / `ifndef` / `endif` / `if` / `else` / `elif`：条件编译
* `define` / `undef`：定义和取消宏
* `include`：包含头文件
* `pragma`：编译器指令

### 特点

* 编译前替换
* 与作用域/类型/接口等概念格格不入
* 潜伏与环境：命令行/头文件/Makefile 定义宏……
* 穿透作用域：粗暴替换
* 用途广泛，难以替代

## I/O 流

```cpp
ifstream in("in.txt");
streambuf *cinbuf = cin.rdbuf(); // save old buf
cin.rdbuf(in.rdbuf());           // redirect cin to in.txt!
ofstream out("out.txt");
streambuf *coutbuf = cout.rdbuf(); // save old buf
cout.rdbuf(out.rdbuf());           // redirect cout to out.txt!
string word;
cin >> word;         // input from the file in.txt
cout << word << " "; // output to the file out.txt
cin.rdbuf(cinbuf);   // reset to standard input again
cout.rdbuf(coutbuf); // reset to standard output again
cin >> word;         // input from the standard input
cout << word;        // output to the standard input
```
