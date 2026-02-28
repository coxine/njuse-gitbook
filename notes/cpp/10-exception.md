# 10-异常

* 错误：语法错误、逻辑错误
* 异常：运行环境造成，可以预见，无法避免
  * 内存不足、文件操作失败……

## 语法

* `throw`：抛出异常
* `try`：包含可能抛出异常的代码块
* `catch(type var)`：捕获异常
  * 可以有多个`catch`，**按顺序匹配**
  * 匹配规则同函数重载
  * `var` 可省略
* 嵌套`try-catch`：内层`catch` 未捕获异常时，传递到外层`try-catch`
* 若异常对象在`catch`中未被捕获，系统的 `abort` 函数将被调用，程序终止
* 特例
  * `catch(type var) { throw; }`：重新抛出异常
  * `catch(...) { }`：捕获所有类型异常
* 特性
  * 不影响对象布局
  * 因异常退出时，RAII 机制保证作用域中的对象被正确析构

```cpp
void f()
{
  throw 1;
  throw 1.0;
  throw "abcd";
}

try {
  f();
} catch (int e) {
  std::cout << "catch int: " << e << std::endl;
  throw; // 重新抛出异常
} catch (double e) {
} catch (const char *e) {
} catch (...) {
  std::cout << "catch all" << std::endl;
}
```

```cpp
template<class T, class E>
inline void Assert(T exp, E e)
{
    if (DEBUG)
        if (!exp) throw e;
}

Assert(x != 0, "x 不能为零");
```
