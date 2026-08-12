# 2026-复习提纲

## Linux 基础知识

### 概念与历史（简答送分题）

* Linux 系统
  * 根据 GPL 许可证开发的免费 Unix 类型操作系统
  * GNU/Linux 系统 = Linux Kernel + GNU 工具和库
  * 特点：开源、多平台支持、流行
* 发行版：Ubuntu, Debian, Fedora……

### 许可证（今年可能考）

* GPL (GNU General Public License)：copyleft
  * 可以复制、获取源码、修改并重新分发
  * 可以对服务收费，但不能限制他人同样的权利
  * 修改后的代码必须同样以 GPL 发布
* BSD 许可证：更宽松，允许闭源使用
* 区别：GPL 要求衍生作品也开源；BSD 不要求

### 文件系统（知道名称）

* VFS：虚拟文件系统，提供**统一接口**
* EXT2 / EXT3 / EXT4：Linux 原生文件系统
* FAT32：兼容性好，常用于 USB
* NTFS：Windows 文件系统

### GRUB（了解即可）

* GRand Unified Boot Loader
* 存储在 MBR（第一阶段）和 `/boot/grub`（1.5 和第二阶段）

### 虚拟终端（要考）

* 控制台模拟多个虚拟终端，每个终端可被视为独立控制台
* 典型设置：
  * VT 1-6：文本模式登录
  * VT 7：图形模式登录
* 切换：`Alt-Fn`（在 X 中为 `Ctrl-Alt-Fn`）

### 目录结构（要考）

| 目录           | 用途                              |
| ------------ | ------------------------------- |
| `/`          | 根目录                             |
| `/boot`      | 内核、bootloader 配置（vmlinuz, grub） |
| `/etc`       | 系统配置文件                          |
| `/bin`       | 基本用户命令（ls, cp 等）                |
| `/sbin`      | 系统管理命令（ifconfig, fdisk）         |
| `/usr`       | 用户程序、库、头文件                      |
| `/usr/local` | 从源码安装的程序                        |
| `/lib`       | `/bin` 和 `/sbin` 的支持库           |
| `/home`      | 普通用户主目录                         |
| `/root`      | root 用户主目录                      |
| `/dev`       | 设备文件                            |
| `/proc`      | 虚拟文件系统，进程/系统信息                  |
| `/var`       | 可变数据（日志 `/var/log`、邮件等）         |
| `/tmp`       | 临时文件，重启后删除                      |
| `/mnt`       | 临时挂载点                           |
| `/media`     | 可移动设备挂载（CD-ROM 等）               |
| `/opt`       | 可选附加应用                          |

### 【重点】七种文件类型

<table data-search="false"><thead><tr><th>类型</th><th align="center">ls -l 首字符</th><th>说明</th></tr></thead><tbody><tr><td>普通文件</td><td align="center"><code>-</code></td><td>文本或二进制数据，无特别内部结构</td></tr><tr><td>目录</td><td align="center"><code>d</code></td><td>文件列表</td></tr><tr><td>字符设备文件</td><td align="center"><code>c</code></td><td>代表字符型硬件设备，位于 <code>/dev</code></td></tr><tr><td>块设备文件</td><td align="center"><code>b</code></td><td>代表块型硬件设备，位于 <code>/dev</code></td></tr><tr><td>符号链接</td><td align="center"><code>l</code></td><td>软链接，类似快捷方式</td></tr><tr><td>管道（FIFO）</td><td align="center"><code>p</code></td><td>进程间通信</td></tr><tr><td>套接字（socket）</td><td align="center"><code>s</code></td><td>网络/进程间数据通信接口</td></tr></tbody></table>

补充：

* 硬链接：同一文件的另一个文件名，不占额外磁盘空间和 inode，不能跨文件系统，不能链接目录
* 软链接：独立文件，存储目标路径，可跨文件系统，可链接目录

### 【重点】权限

```
-rwxr-xr-- 1 user group 4096 Jan 1 12:00 filename
│└┬─┘└┬─┘└┬─┘│  │     │     │              │
│ │   │   │  │  │     │     │              └─ 文件名
│ │   │   │  │  │     │     └─ 修改时间
│ │   │   │  │  │     └─ 文件大小（字节）
│ │   │   │  │  └─ 所属组
│ │   │   │  └─ 所有者
│ │   │   │
│ │   │   └─ 硬链接数
│ │   └─ 其他用户权限 (other)
│ └─ 组权限 (group)
└─ 用户权限 (user) + 文件类型标识
```

#### 三个访问级别

* 用户（u）：文件所有者
* 组（g）：所属组的用户
* 其他（o）：其余所有用户

#### 三个权限

| 权限      | 对文件    | 对目录         |
| ------- | ------ | ----------- |
| 读（r=4）  | 读取内容   | 列出目录内容      |
| 写（w=2）  | 修改内容   | 创建/删除目录中的文件 |
| 执行（x=1） | 作为程序执行 | 进入该目录（cd）   |

#### 修改权限 `chmod`

* 符号模式：`chmod <who><op><what> file`
  * who: `u` / `g` / `o` / `a`（all）
  * op: `+` / `-` / `=`
  * what: `r` / `w` / `x`
  * 例：`chmod u+x file`，`chmod go-w file`
* 数字模式：`chmod 755 file`（rwxr-xr-x）

#### 默认权限

* 文件：`644`（`-rw-r--r--`）
* 目录：`755`（`drwxr-xr-x`）

### Linux 命令（要求能写出命令）

#### 目录操作

| 命令      | 功能             |
| ------- | -------------- |
| `pwd`   | 显示当前目录         |
| `cd`    | 切换目录           |
| `mkdir` | 创建目录           |
| `rmdir` | 删除空目录          |
| `ls`    | 列出目录内容         |
| `ls -l` | 长格式（详细信息）      |
| `ls -a` | 显示隐藏文件（`.` 开头） |
| `ls -R` | 递归列出子目录        |

#### 文件操作

| 命令            | 功能                |
| ------------- | ----------------- |
| `touch`       | 更新时间戳 / 创建空文件     |
| `cp`          | 复制                |
| `mv`          | 移动 / 重命名          |
| `rm`          | 删除                |
| `ln`          | 创建链接（`ln -s` 软链接） |
| `cat`         | 显示文件内容            |
| `more`/`less` | 分页显示（`less` 支持回退） |
| `head`/`tail` | 显示文件头/尾部          |

#### 文件属性

| 命令      | 功能    |
| ------- | ----- |
| `chmod` | 修改权限  |
| `chown` | 修改所有者 |
| `chgrp` | 修改所属组 |

#### 其他常用命令

| 命令               | 功能                            |
| ---------------- | ----------------------------- |
| `passwd`         | 修改密码                          |
| `mkpasswd`       | 创建随机密码                        |
| `who` / `whoami` | 查看登录用户 / 当前用户                 |
| `ps`             | 查看进程                          |
| `kill`           | 发送信号终止进程                      |
| `jobs`/`fg`/`bg` | 作业控制                          |
| `top`            | 实时进程监控                        |
| `su`             | 切换用户                          |
| `tar`            | 打包/解包（`tar zxvf file.tar.gz`） |

#### 命令提示符

* `$`：普通用户
* `#`：root 用户
* 格式：`用户名@主机名:当前目录$`

### 进程基本概念（选择题送分）

* 进程：正在运行的程序实例，包含执行程序、当前值、状态信息及系统资源
* 所有进程由其他进程启动（父子关系，树形结构）
* 唯一例外：`init`（PID 1）由内核启动
* **PID**：进程自身的标识号
* **PPID**：父进程的标识号（Parent PID）
* Shell：读取用户命令并启动对应进程的进程

### 重定向（会考）

标准 I/O 流：

| 名称   | 文件描述符 | C 变量     |
| ---- | :---: | -------- |
| 标准输入 |   0   | `stdin`  |
| 标准输出 |   1   | `stdout` |
| 标准错误 |   2   | `stderr` |

重定向符号：

| 符号     | 作用        | 示例                   |
| ------ | --------- | -------------------- |
| `<`    | 输入重定向     | `cmd < input.txt`    |
| `>`    | 输出覆盖写入文件  | `cmd > out.txt`      |
| `>>`   | 输出追加到文件   | `cmd >> out.txt`     |
| `2>`   | 错误输出重定向   | `cmd 2> err.txt`     |
| `2>&1` | 错误合并到标准输出 | `cmd > all.txt 2>&1` |

示例：

```bash
kill -HUP 1234 > killout.txt 2> killerr.txt
kill -HUP 1234 > killout.txt 2>&1
```

### 管道（会考）

* 将一个进程的标准输出连接到另一个进程的标准输入
* 语法：`cmd1 | cmd2 | cmd3`
*   示例：

    ```bash
    ls | wc -l           # 统计文件数
    ls -lF | grep ^d     # 只显示目录
    ```

### 环境变量（和 Shell 一起考）

* 定义：操作环境的参数
* 查看：`echo $VAR`、`env`、`set`
* 设置：`VAR=value`，`export VAR`
*   关键变量：`PATH`（命令搜索路径）

    ```bash
    echo $PATH
    PATH=$PATH:/new/path
    export PATH
    ```
* 永久生效需修改配置文件（如 `/etc/profile`）

### find / sed / grep（读懂即可）

#### find

* 查找文件：`find <路径> <条件> <动作>`
* 示例：`find . -name "*.c"`

### grep

* 字符串匹配：`grep <pattern> <file>`
* 常用选项：`-i`（忽略大小写）、`-r`（递归）、`-n`（显示行号）
* 示例：`grep "main" *.c`

#### sed

* 流编辑器，逐行处理文本
* 替换：`sed 's/old/new/g' file`

## Shell 编程

### Shell 概述

### 执行脚本的三种方法（重点）

| 方法  | 命令                                       |  是否新开进程 |     环境变量影响    |
| --- | ---------------------------------------- | :-----: | :-----------: |
| 方法1 | `sh script_file`                         |    是    |   不影响当前shell  |
| 方法2 | `chmod +x script_file` → `./script_file` |    是    |   不影响当前shell  |
| 方法3 | `source script_file` 或 `. script_file`   | 否（当前进程） | **影响当前shell** |

### 脚本基本结构

```bash
#!/bin/bash
# 注释以#开头
# 第一行指定解释器

commands...

exit 0  # 退出码，0表示成功
```

### 变量（重点语法规则）

#### 赋值与使用

```bash
var=value        # 赋值号前后【不能有空格】！
echo $var        # 使用变量加$
echo ${var}_suffix  # 大括号消除歧义
```

#### read 命令

```bash
read var         # 从键盘读入
read -p "提示:" var  # 带提示
read -t 5 var    # 5秒超时
read -s var      # 静默输入（密码）
```

### 引号规则（重点）

| 引号        | 规则                                            |
| --------- | --------------------------------------------- |
| 单引号 `' '` | 所有字符保持原义，**不做任何解析**（`$`就是`$`）                 |
| 双引号 `" "` | **解析** `$`（变量）、`` ` ` ``（命令替换）、`\`（转义），其余保持原义 |
| 无引号       | 全部解析，且受词分割和通配符扩展影响                            |

### 环境变量

| 变量      | 含义                   |
| ------- | -------------------- |
| `$HOME` | 当前用户主目录              |
| `$PATH` | 命令搜索路径（冒号分隔）         |
| `$PS1`  | 主提示符（通常`$`）          |
| `$PS2`  | 辅助提示符（通常`>`）         |
| `$IFS`  | 输入字段分隔符（默认空格/Tab/换行） |

### 参数变量

| 变量         | 含义                        |
| ---------- | ------------------------- |
| `$#`       | 参数个数                      |
| `$0`       | 脚本名                       |
| `$1`\~`$9` | 位置参数                      |
| `$*`       | 所有参数（作为一个字符串，用IFS第一个字符分隔） |
| `$@`       | 所有参数（每个参数独立）              |

注意：**函数内部的参数也是 `$1`, `$2`...**，会覆盖脚本的参数。

### 条件测试（重点）

语法：`test expression` 或 `[ expression ]`

**注意**：`[` 是一个命令，`expression` 两边**必须有空格**！

#### 算术比较（不能用 `<` `>`，被重定向占用）

| 运算符   | 含义   |
| ----- | ---- |
| `-eq` | 等于   |
| `-ne` | 不等于  |
| `-gt` | 大于   |
| `-ge` | 大于等于 |
| `-lt` | 小于   |
| `-le` | 小于等于 |

#### 字符串比较

| 运算符            | 含义 |
| -------------- | -- |
| `str1 = str2`  | 相同 |
| `str1 != str2` | 不同 |
| `-z str`       | 为空 |
| `-n str`       | 非空 |

#### 文件测试

| 运算符        | 含义        |
| ---------- | --------- |
| `-e file`  | 存在        |
| `-f file`  | 普通文件      |
| `-d file`  | 目录        |
| `-r/-w/-x` | 可读/可写/可执行 |
| `-s file`  | 长度非零      |

#### 逻辑操作

| 运算符              | 含义 |
| ---------------- | -- |
| `! expr`         | 取反 |
| `expr1 -a expr2` | 与  |
| `expr1 -o expr2` | 或  |

### 分支语句

#### if 语句（结束用 `fi`）

```bash
if [ expression ]; then
    statements
elif [ expression ]; then
    statements
else
    statements
fi
```

#### case 语句（结束用 `esac`）

```bash
case "$var" in
    pattern1 | pattern2) statements;;
    pattern3) statements;;
    *) statements;;  # 默认匹配
esac
```

### 循环语句（全部 `do` 开头 `done` 结束）

#### for

```bash
for var in list
do
    statements
done
```

示例：

```bash
for file in $(ls *.sh); do
    echo $file
done
```

#### while

```bash
while [ condition ]
do
    statements
done
```

示例：

```bash
x=0
while [ "$x" -lt 10 ]; do
    echo $x
    x=$(($x+1))
done
```

#### until（条件为假时循环，不推荐使用）

```bash
until condition
do
    statements
done
```

### 命令组合（AND/OR 短路）

```bash
# AND：前面成功才执行后面
statement1 && statement2

# OR：前面失败才执行后面
statement1 || statement2

# 分号：顺序执行，不关心成功与否
cmd1 ; cmd2
```

### 函数

```bash
func_name()
{
    local var=value   # 局部变量用 local
    echo $1           # 函数参数也是 $1, $2...
    return 0          # 返回值
}

# 调用
func_name arg1 arg2
```

### 捕获命令输出

```bash
result=$(command)    # 推荐
```

### 算术扩展

```bash
x=$(($x + 1))
y=$((3 * 5))
```

### 即时文档 Here Document（肯定会考）

将多行输入传递给命令：

```bash
command << DELIMITER
内容行1
内容行2
DELIMITER
```

示例：

```bash
cat >> file.txt << EOF
Hello, this is a here document.
Line 2.
EOF
```

#### 重定向符号总结（配合即时文档一起考）

| 符号   | 作用                  |
| ---- | ------------------- |
| `<`  | 输入重定向（从文件读入）        |
| `>`  | 输出重定向（覆盖写入）         |
| `>>` | 输出追加                |
| `<<` | 即时文档（Here Document） |

### 杂项命令

| 命令             | 功能           |
| -------------- | ------------ |
| `break`        | 退出循环         |
| `continue`     | 跳到下一次循环      |
| `exit n`       | 退出脚本，返回码n    |
| `return`       | 函数返回         |
| `export`       | 导出变量为环境变量    |
| `set`          | 设置shell参数变量  |
| `unset`        | 删除变量或函数      |
| `trap`         | 捕获信号并执行指定动作  |
| `:`            | 空命令（什么都不做）   |
| `.` / `source` | 在当前shell执行脚本 |

## 编译与链接

### ELF 格式（了解定义）

* ELF：Executable and Linkable Format（可执行与可链接格式）
* Linux 下可执行文件的封装格式

### 编译与链接的职责

| 阶段  | 职责                           | 输入 → 输出                   |
| --- | ---------------------------- | ------------------------- |
| 预处理 | 展开 `#include`、`#define`、条件编译 | `.c` → `.i`               |
| 编译  | 将源码翻译为汇编代码                   | `.i` → `.s`               |
| 汇编  | 将汇编代码翻译为目标文件                 | `.s` → `.o`               |
| 链接  | 将多个 `.o` 和库合并为可执行文件          | `.o` + `.a`/`.so` → 可执行文件 |

关键理解：

* 每个 `.c` 源文件独立编译为一个 `.o` 目标文件
* 链接器将所有 `.o` 文件和库文件合并，解析符号引用

### `#include` 与预处理（重点）

* `#include` 在**预处理阶段**（编译之前）执行
* 作用：找到头文件，将其文本内容替换到该位置
* 头文件中通常包含：函数声明、宏定义、类型定义

#### `#if` 和 `if` 的区别

|      | `#if`            | `if`     |
| ---- | ---------------- | -------- |
| 时机   | **预处理阶段**（编译前）   | **运行时**  |
| 作用   | 条件编译，决定代码是否参与编译  | 控制程序执行流程 |
| 不满足时 | 代码**不会出现在**目标文件中 | 代码存在但不执行 |

```c
#if DEBUG
    printf("debug info\n");  // 若DEBUG未定义或为0，此行不编译
#endif

if (debug) {
    printf("debug info\n");  // 无论如何都会编译，运行时判断
}
```

### 静态库与动态库（重点）

|      | 静态库 (.a)              | 动态库 (.so)      |
| ---- | --------------------- | -------------- |
| 链接时机 | 编译时链接，代码**复制**到可执行文件中 | 运行时加载          |
| 文件大小 | 可执行文件较大               | 可执行文件较小        |
| 更新   | 需重新编译                 | 替换 .so 文件即可    |
| 内存   | 每个进程各有一份副本            | 多个进程**共享**同一份  |
| 制作工具 | `ar`（打包 .o 文件）        | `gcc -shared`  |
| 依赖   | 无运行时依赖                | 运行时需要 .so 文件存在 |
| 问题   | 磁盘/内存浪费、更新不便          | 版本冲突（DLL Hell） |

### GCC 命令与参数（要掌握）

用法：`gcc [options] [filename]`

#### 编译控制选项

| 选项           | 作用                             |
| ------------ | ------------------------------ |
| `-E`         | 只做预处理（输出 `.i`）                 |
| `-S`         | 预处理 + 编译（输出 `.s`）              |
| `-c`         | 预处理 + 编译 + 汇编，**不链接**（输出 `.o`） |
| `-o file`    | 指定输出文件名                        |
| `-g`         | 生成调试信息（给 gdb 用）                |
| `-O` / `-O2` | 优化等级                           |
| `-Wall`      | 显示所有警告                         |

#### 搜索路径与链接选项

| 选项              | 作用                              |
| --------------- | ------------------------------- |
| `-Idir`         | 指定额外的**头文件**搜索路径                |
| `-Ldir`         | 指定额外的**库文件**搜索路径                |
| `-lname`        | 链接库 `libname.a` 或 `libname.so`  |
| `-DMACRO[=VAL]` | 定义宏（等效于源码中 `#define MACRO VAL`） |

### 文件后缀名

| 后缀    | 含义                 |
| ----- | ------------------ |
| `.c`  | C 源码（需预处理）         |
| `.i`  | 预处理后的 C 源码         |
| `.h`  | 头文件                |
| `.s`  | 汇编代码               |
| `.S`  | 需预处理的汇编代码          |
| `.o`  | 目标文件（Object file）  |
| `.a`  | 静态库（Archive）       |
| `.so` | 动态库（Shared Object） |

### Makefile（要求能读，不要求写）

#### 基本规则结构

```makefile
target: prerequisites
 command        # 必须用 Tab 缩进
```

* **target**：目标文件（可以是 `.o`、可执行文件、或伪目标如 `clean`）
* **prerequisites**：依赖的文件
* **command**：生成目标的 shell 命令

#### 执行逻辑

1. make 查找当前目录的 `Makefile` 或 `makefile`
2. 找到第一个 target 作为默认目标
3. 如果 target 不存在，或依赖文件比 target 新，则执行 command
4. 递归处理依赖

#### 示例

```makefile
hello: main.o kbd.o
 gcc -o hello main.o kbd.o

main.o: main.c defs.h
 gcc -c main.c

kbd.o: kbd.c defs.h command.h
 gcc -c kbd.c

clean:
 rm hello main.o kbd.o
```

#### 变量与自动变量

```makefile
OBJECTS = main.o kbd.o
CC = gcc

hello: $(OBJECTS)
 $(CC) -o hello $(OBJECTS)
```

| 自动变量 | 含义         |
| ---- | ---------- |
| `$@` | 目标文件名      |
| `$<` | 第一个依赖文件    |
| `$^` | 所有依赖文件（去重） |
| `$?` | 比目标新的依赖文件  |

#### 模式规则

```makefile
%.o: %.c
 $(CC) -c $(CFLAGS) $< -o $@
```

#### 伪目标

```makefile
.PHONY: clean
clean:
 rm -f *.o hello
```

* 伪目标不是文件，只是标签
* 用 `.PHONY` 声明避免与同名文件冲突
* 必须显式指定才执行：`make clean`

#### make 命令

```bash
make                  # 执行默认目标
make target           # 执行指定目标
make -f MyMakefile    # 指定makefile文件
make clean            # 执行伪目标
```

## 系统调用

### VFS 四种对象（重点）

VFS（Virtual File System）：虚拟文件系统，仅存在于内存中，为不同的底层文件系统提供统一接口。

| 对象                | 作用                                              |
| ----------------- | ----------------------------------------------- |
| 超级块（super block）  | 描述一个文件系统的整体信息（类型、参数）；对应一个磁盘分区                   |
| i 节点对象（i-node）    | 描述磁盘上的一个真实文件，按索引号访问                             |
| 文件对象（file object） | 描述进程打开的文件（文件描述符）；`open` 创建，`close` 释放；不对应磁盘上的实体 |
| 目录项对象（dentry）     | 描述路径中的一个组成部分，缓存目录结构                             |

### 硬链接 vs 软链接（重点，配合命令考）

|        | 硬链接 (Hard Link)      | 软链接 (Symbolic Link)     |
| ------ | -------------------- | ----------------------- |
| 本质     | 不同文件名指向**同一个 inode** | 独立文件，存储目标文件的**路径名**     |
| 跨文件系统  | 不能                   | 可以                      |
| 链接目录   | 不能                   | 可以                      |
| 删除原文件后 | 仍可访问（inode 引用计数>0）   | 失效（悬空链接）                |
| inode  | 共享同一个                | 各自独立                    |
| 系统调用   | `link()`             | `symlink()`             |
| 命令     | `ln oldfile newfile` | `ln -s oldfile newfile` |

### 系统调用 vs 库函数

|    | 系统调用                | 库函数               |
| -- | ------------------- | ----------------- |
| 层次 | 内核对外接口              | 依赖于系统调用           |
| 功能 | 提供最小接口              | 提供较复杂功能           |
| 参数 | 文件描述符 `int fd`      | 文件流指针 `FILE *fp`  |
| 缓冲 | 无缓冲（Unbuffered I/O） | 有缓冲（Buffered I/O） |

### 文件描述符

* 小的非负整数：`int fd`
* 预定义（`<unistd.h>`）：
  * `STDIN_FILENO` = 0
  * `STDOUT_FILENO` = 1
  * `STDERR_FILENO` = 2
* 文件操作一般步骤：`open` → `read`/`write` → \[`lseek`] → `close`

### 核心系统调用

#### open / creat

```c
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
int creat(const char *pathname, mode_t mode);
// 返回：成功返回新fd，失败返回-1
```

**flags 参数**（可用 `|` 组合）：

| 标志         | 含义                   |
| ---------- | -------------------- |
| `O_RDONLY` | 只读                   |
| `O_WRONLY` | 只写                   |
| `O_RDWR`   | 读写                   |
| `O_APPEND` | 追加模式                 |
| `O_CREAT`  | 不存在则创建               |
| `O_TRUNC`  | 存在则截断为0              |
| `O_EXCL`   | 与O\_CREAT一起，文件已存在则报错 |

`creat` 等价于 `open(path, O_CREAT|O_WRONLY|O_TRUNC, mode)`

**mode 参数**（权限，4位八进制）：`S_IRUSR`(0400) / `S_IWUSR`(0200) / `S_IXUSR`(0100) 等，用 `|` 组合。

#### close / read / write

```c
#include <unistd.h>
int close(int fd);

ssize_t read(int fd, void *buf, size_t count);
// 返回：读到的字节数，到文件尾返回0，出错返回-1

ssize_t write(int fd, const void *buf, size_t count);
// 返回：成功为已写字节数，出错返回-1
```

#### lseek

```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
// 返回：成功为新的偏移量，失败返回-1
```

| whence     | 含义                |
| ---------- | ----------------- |
| `SEEK_SET` | 从文件头偏移 offset 字节  |
| `SEEK_CUR` | 从当前位置偏移 offset 字节 |
| `SEEK_END` | 从文件尾偏移 offset 字节  |

#### dup / dup2（重定向实现）

```c
#include <unistd.h>
int dup(int oldfd);          // 复制fd，返回最小可用fd
int dup2(int oldfd, int newfd);  // 将oldfd复制到newfd
```

用途：实现重定向。例如将标准输出重定向到文件：

```c
int fd = open("out.txt", O_WRONLY|O_CREAT, 0644);
dup2(fd, STDOUT_FILENO);  // 1号fd现在指向out.txt
```

#### fcntl

```c
#include <fcntl.h>
int fcntl(int fd, int cmd, ...);
```

cmd 取值：

* `F_DUPFD`：复制文件描述符
* `F_GETFD`/`F_SETFD`：获取/设置 close-on-exec 标志
* `F_GETFL`/`F_SETFL`：获取/设置文件状态标志
* `F_GETLK`/`F_SETLK`/`F_SETLKW`：文件锁操作

#### link / unlink / symlink

```c
#include <unistd.h>
int link(const char *oldpath, const char *newpath);    // 创建硬链接
int unlink(const char *pathname);                       // 删除链接
int symlink(const char *oldpath, const char *newpath); // 创建软链接
int readlink(const char *path, char *buf, size_t bufsiz); // 读软链接内容
```

### stat 系列（获取文件属性）

```c
#include <sys/stat.h>
int stat(const char *path, struct stat *buf);   // 跟随符号链接
int lstat(const char *path, struct stat *buf);  // 不跟随符号链接
int fstat(int fd, struct stat *buf);
```

#### struct stat 主要字段

```c
struct stat {
    mode_t  st_mode;    // 文件类型 + 权限
    ino_t   st_ino;     // inode号
    nlink_t st_nlink;   // 硬链接计数
    uid_t   st_uid;     // 所有者UID
    gid_t   st_gid;     // 所属组GID
    off_t   st_size;    // 文件大小（字节）
    time_t  st_mtime;   // 最后修改时间
};
```

#### 文件类型测试宏

| 宏            | 文件类型    |
| ------------ | ------- |
| `S_ISREG()`  | 普通文件    |
| `S_ISDIR()`  | 目录      |
| `S_ISCHR()`  | 字符设备    |
| `S_ISBLK()`  | 块设备     |
| `S_ISFIFO()` | FIFO/管道 |
| `S_ISLNK()`  | 符号链接    |
| `S_ISSOCK()` | 套接字     |

### 目录操作（了解）

```c
#include <dirent.h>
DIR *opendir(const char *name);
struct dirent *readdir(DIR *dir);
int closedir(DIR *dir);

#include <unistd.h>
int mkdir(const char *path, mode_t mode);
int rmdir(const char *path);
int chdir(const char *path);
char *getcwd(char *buf, size_t size);
```

目录扫描示例模式：

```c
DIR *dp = opendir(dir);
struct dirent *entry;
while ((entry = readdir(dp)) != NULL) {
    // entry->d_name 是文件名
    lstat(entry->d_name, &statbuf);
    if (S_ISDIR(statbuf.st_mode))
        // 是目录
}
closedir(dp);
```

## 进程/信号

### 进程的入口与退出

* C 程序入口：`main(int argc, char *argv[])`
* 由 `exec` 系统调用启动，运行时启动代码 `crt0.o` 调用 `main`

#### 进程终止方式

* 正常终止：
  * `main` 中 `return`
  * 调用 `exit()`（库函数，会执行清理）
  * 调用 `_exit()`（系统调用，立即终止）
* 异常终止：
  * 调用 `abort()`
  * 被信号终止

#### exit vs \_exit

|    | `exit()`              | `_exit()`    |
| -- | --------------------- | ------------ |
| 类型 | 库函数                   | 系统调用         |
| 清理 | 执行 atexit 注册的函数、刷新缓冲区 | **不执行**，立即终止 |

#### atexit（选择题可能出现）

```c
#include <stdlib.h>
int atexit(void (*func)(void));
// 注册在 exit() 时自动调用的函数
```

### fork（重点，编程题可能考）

```c
#include <unistd.h>
pid_t fork(void);
```

* 创建子进程：**复制**父进程
* 返回值：
  * 父进程中：返回**子进程的 PID**（>0）
  * 子进程中：返回 **0**
  * 失败：返回 -1

典型用法：

```c
pid_t pid = fork();
if (pid == 0) {
    // 子进程代码
} else if (pid > 0) {
    // 父进程代码
} else {
    // fork失败
}
```

关键理解：

* fork 后父子进程各自独立执行
* 子进程从 fork() 返回处继续执行（不会重新执行之前的代码）
* 子进程是父进程的副本（内存、文件描述符等都复制）

### exec 系列（重点）

```c
#include <unistd.h>
int execl(const char *path, const char *arg0, ..., (char *)0);
int execlp(const char *file, const char *arg0, ..., (char *)0);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
```

* 用**新程序替换当前进程**的内存映像
* **不创建新进程**，PID 不变
* 成功不返回；失败返回 -1

命名规则：

* `l`：参数以列表方式传递
* `v`：参数以数组方式传递
* `p`：在 PATH 中搜索程序
* `e`：可传递环境变量

fork + exec 组合：

```c
if (fork() == 0) {
    execl("/bin/ls", "ls", "-l", NULL);  // 子进程执行ls
    // execl成功后不会到这里
}
```

### wait / waitpid（选择题）

```c
#include <sys/wait.h>
pid_t wait(int *status);              // 等待任一子进程终止（阻塞）
pid_t waitpid(pid_t pid, int *status, int options);
```

wait 的几种结果：

* 有子进程终止 → 立即返回
* 无子进程终止 → **阻塞等待**
* 无子进程 → 出错返回

waitpid 的 pid 参数：

* `pid > 0`：等待指定 PID 的子进程
* `pid == 0`：等待同组的任一子进程
* `pid == -1`：等待任一子进程（同 wait）
* `pid < -1`：等待组ID为|pid|的任一子进程

waitpid 优势：可指定 pid、可非阻塞（`WNOHANG`）

### signal 信号（选择题）

#### 概念

* 信号：软件中断，用于异步事件通知
* 信号名以 `SIG` 开头，定义为正整数（`<signal.h>`）

#### 常见信号

| 信号          | 说明              | 触发        |
| ----------- | --------------- | --------- |
| `SIGINT`    | 终端中断            | Ctrl+C    |
| `SIGQUIT`   | 终端退出            | Ctrl+\\   |
| `SIGKILL`   | 终止（**不可捕捉/忽略**） | `kill -9` |
| `SIGSTOP`   | 停止（**不可捕捉/忽略**） |           |
| `SIGTSTP`   | 终端挂起            | Ctrl+Z    |
| `SIGCHLD`   | 子进程停止或退出        |           |
| `SIGALRM`   | 定时器超时           | `alarm()` |
| `SIGSEGV`   | 段错误（无效内存访问）     |           |
| `SIGTERM`   | 终止（默认 kill 信号）  | `kill`    |
| `SIGUSR1/2` | 用户自定义信号         |           |

#### 信号处理方式

1. 忽略信号（`SIGKILL`/`SIGSTOP` 除外）
2. 执行系统默认动作
3. 捕捉信号（自定义处理函数）

#### signal 函数

```c
#include <signal.h>
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
// 返回：之前的处理函数；失败返回SIG_ERR
```

handler 可以是：

* 用户自定义函数
* `SIG_DFL`（默认动作）
* `SIG_IGN`（忽略）

示例：

```c
void sig_handler(int signo) {
    printf("Caught signal %d\n", signo);
}

signal(SIGUSR1, sig_handler);  // 注册处理函数
```

#### kill / raise

```c
#include <signal.h>
int kill(pid_t pid, int sig);   // 向指定进程发送信号
int raise(int sig);             // 向自己发送信号
```

#### alarm / pause

```c
unsigned int alarm(unsigned int seconds); // 设置定时器，超时发SIGALRM
int pause(void);                          // 挂起直到收到信号
```

### mmap（了解）

```c
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```

* 将文件或设备映射到内存
* prot：`PROT_READ` / `PROT_WRITE` / `PROT_EXEC` / `PROT_NONE`
* flags：
  * `MAP_SHARED`：修改对其他进程可见（可用于进程间通信）
  * `MAP_PRIVATE`：写时复制，修改不影响原文件
  * `MAP_ANONYMOUS`：不关联文件，用于分配内存
* 用途：高效文件I/O、进程间共享内存

## 内核与驱动

### 内核基础概念

* 操作系统最重要的部分构成内核
* Linux 采用**单内核**架构：一个大进程，内部分模块，模块间通过函数调用通信
* 微内核：模块作为独立进程，通过消息传递通信
* Linux 内核能力：内存管理、文件系统、进程管理、多线程、抢占式调度、多处理器支持

### 编译内核的步骤（要掌握）

1. **清除旧目标文件**：`make clean`
2. **配置内核**：`make menuconfig`（图形界面选择编译选项）
3.  **编译**

    ```bash
    make              # 编译内核
    make zImage       # 生成内核映像
    make bzImage      # 生成压缩内核映像
    make modules      # 编译模块
    ```
4. **安装**：`make install`（复制内核到 /boot，更新引导菜单）

### 模块命令（要掌握）

| 命令                            | 功能                              |
| ----------------------------- | ------------------------------- |
| `insmod <module.ko> [params]` | 加载模块（需 root 权限）                 |
| `rmmod <module>`              | 卸载模块                            |
| `lsmod`                       | 列出已加载模块（等价 `cat /proc/modules`） |
| `modprobe <module>`           | 加载模块 + 自动处理依赖                   |
| `modprobe -r <module>`        | 卸载模块 + 处理依赖                     |
| `modinfo <module>`            | 显示模块信息                          |
| `depmod`                      | 生成模块依赖关系                        |

**insmod vs modprobe**：

* `insmod`：底层命令，只加载指定模块，不处理依赖
* `modprobe`：高层命令，自动解决模块依赖关系

### 模块依赖

* 模块A引用模块B导出的符号 → B被A引用
* 加载A之前必须先加载B
* 导出符号：`EXPORT_SYMBOL(name)` / `EXPORT_SYMBOL_GPL(name)`

### 内核模块 vs 应用程序

|      | 应用程序      | 内核模块               |
| ---- | --------- | ------------------ |
| 运行空间 | 用户空间      | 内核空间               |
| 入口   | `main()`  | `module_init()` 指定 |
| 出口   | 无（return） | `module_exit()` 指定 |
| 运行方式 | 直接执行      | `insmod` 加载        |
| 调试工具 | gdb       | kdb, kgdb 等        |
| C库   | 可用        | **不可用**            |
| 内存保护 | 有         | **无**              |

### 读懂简单内核模块（重点）

```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

static int __init hello_init(void)   // __init: 仅初始化时使用，之后释放内存
{
    printk(KERN_INFO "Hello world\n");
    return 0;
}

static void __exit hello_exit(void)  // __exit: 仅卸载时使用
{
    printk(KERN_INFO "Goodbye world\n");
}

module_init(hello_init);    // 声明初始化函数
module_exit(hello_exit);    // 声明卸载函数
```

要点：

* `static`：函数仅文件内可见
* `__init` / `__exit`：内存优化标记
* `module_init()` / `module_exit()`：宏，让内核找到入口/出口
* `printk`：内核态打印（不能用 printf）

### 字符设备驱动初始化加载过程（重点）

#### 三种设备类型

| 类型     | 特点                  |
| ------ | ------------------- |
| 字符设备   | 按字节流访问，简单，应用广泛      |
| 块设备    | 按块随机访问，结构复杂（**不考**） |
| 网络接口设备 | 网络数据收发              |

#### 字符设备驱动加载步骤

1.  **申请设备号**

    ```c
    int register_chrdev_region(dev_t first, unsigned count, char *name);
    int alloc_chrdev_region(dev_t *dev, unsigned firstminor, unsigned count, char *name);
    ```

    * 主设备号：标识驱动程序
    * 次设备号：标识使用该驱动的各设备
2. **定义 file\_operations 结构体**
   * 包含 `read`、`write`、`open`、`release` 等函数指针
   * 是字符设备驱动对上层的接口
3.  **创建并初始化 cdev 结构体**

    ```c
    struct cdev *my_cdev = cdev_alloc();
    my_cdev->ops = &my_fops;
    // 或
    cdev_init(&my_cdev, &my_fops);
    ```
4.  **将 cdev 注册到系统，绑定设备号**

    ```c
    int cdev_add(struct cdev *dev, dev_t num, unsigned count);
    ```
5.  **在 /dev 中创建设备文件，绑定设备号**

    ```bash
    mknod /dev/mydev c <major> <minor>
    ```

#### 释放流程

```c
cdev_del(&my_cdev);                              // 注销设备
unregister_chrdev_region(dev_num, count);         // 释放设备号
```

### 和硬件打交道（读程序即可）

* `ioremap(phys_addr, size)`：将物理地址映射到虚拟地址空间
* `iounmap(virt_addr)`：撤销映射
* 通过映射后的虚拟地址操作硬件寄存器
