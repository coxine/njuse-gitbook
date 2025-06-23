# 00-基础知识

* 编译器：将 Source language 翻译为 Target language
  * 前端：源文件内容 -> IR
    * Lexical Analysis 词法分析：通过 Pattern 找到 Lexeme（词素），转换为 Token `Token = <class, attribute>`
    * Syntax Analysis 语法分析：生成 AST
    * Semantic Analysis 语义分析：检查类型、范围等语义问题，修饰 AST
    * IR Generation 生成中间表示：将 AST 转换为 IR
  * 中端：机器无关优化，IR -> IR
    * 相较于源文件，IR 更容易处理
    * 相较于汇编，IR 与机器无关
  * 后端：机器相关优化，生成目标，IR -> asm
    * 分配寄存器
    * 选择指令：机器相关
    * 安排指令：利用并行的硬件资源
* 解释器：直接解析 Source language 里面的声明
* 链接器：将多个编译单元链接为单一文件
* 函数式编程：没有副作用，不改变数据和状态，函数作为参数……
* 逻辑编程：声明 facts 和 rules，通过推理得到结果
* 静态/动态类型：编译时/运行时检查类型
* 强/弱类型：是否允许隐式类型转换
* 静态/动态作用域：编译时/运行时解析变量
* 值/引用传递：传递值/传递指针
* 虚函数：可被子类重写的函数
