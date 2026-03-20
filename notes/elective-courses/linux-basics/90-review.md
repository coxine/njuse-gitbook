# 90-期末复习

* 以理解为主，少部分实操
* PPT上为主

## 课堂练习&提问&作业

### Day1 作业

> 1. 在`/tmp`下新建一个名为`test`的目录。
> 2. 用命令`man`查看命令`touch`的使用手册。
> 3. 用命令`touch`在`test`目录中新建一个名为`test`的文件。
> 4. 用命令`echo`将以下内容一行一行地写入`test`文件。
> 5. 尝试执行这个文件，即将该脚本的路径（./`test`）输入到您的shell中并回车。如果程序无法执行，请使用`ls`命令来获取信息并给出其不能执行的原因。
> 6. 查看命令`chmod`的手册，使用命令`chmod`改变`test`文件的权限，使 ./`test` 能够成功执行，不要使用`sh test`来执行该程序。
> 7. 请问你的shell是如何知道这个文件需要使用`sh`来解析的。请通过网络搜索“unix shebang”来了解更多信息。
> 8. 请使用 `|` 和 `>` ，将test文件输出的最后5行内容写入自己主目录下的`last-5-lines.txt`文件中。

```bash
 !/bin/sh
curl --head --silent <https://www.nju.edu.cn>
```

```bash
yuxian@Coxine:~$ cd /tmp
yuxian@Coxine:/tmp$ mkdir test
yuxian@Coxine:/tmp$ man touch
yuxian@Coxine:/tmp$ cd ./test
yuxian@Coxine:/tmp/test$ touch test

yuxian@Coxine:/tmp/test$ echo '#!/bin/sh' >test
yuxian@Coxine:/tmp/test$ echo curl --head --silent https://www.nju.edu.cn >>test

yuxian@Coxine:/tmp/test$ cat test
#!/bin/sh
curl --head --silent https://www.nju.edu.cn

yuxian@Coxine:/tmp/test$ ./test
bash: ./test: 权限不够

yuxian@Coxine:/tmp/test$ ls -l
总计 4
-rw-rw-r-- 1 yuxian yuxian 54  7月 24 20:17 test
 原因：没有执行权限 (x)，所以无法执行该文件。

yuxian@Coxine:/tmp/test$ chmod +x ./test
 在 Unix 和类 Unix 系统中，`shebang`（也称为 `sha-bang` 或 `hashbang`）是脚本文件的第一行，以 `#!` 开头，后跟解释器的路径。这个行告诉操作系统使用哪个解释器来运行脚本

yuxian@Coxine:/tmp/test$ ./test | tail -n5 >last-5-lines.txt
yuxian@Coxine:/tmp/test$ cat ./last-5-lines.txt 
Cache-Control: private, max-age=600
Expires: Wed, 24 Jul 2024 12:34:28 GMT
ETag: "4460e-61dfb3bde2920-gzip"
Content-Language: zh-CN
```

### Day2 课堂练习

> 请编写两个`bash`函数`savedir`和`cddir`。执行函数`savedir`时，当前的工作目录应当以某种形式保存。执行函数`cddir`时，无论现在处在什么目录中，都`cd`进入当时执行函数`savedir`时所处的目录。
>
> 为了方便调试，你可以把代码写在单独的文件`test.sh`中，并通过 `source test.sh`命令加载函数。

```bash
savedir(){
 savedDir=$(pwd)
}
cddir(){
 cd $savedDir
}
```

### Day2 作业

> 请编写一段bash脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误输出记录到文件，报告脚本在失败前共运行了多少次，并通过记录的文件验证脚本输出结果的正确性。

```bash
#!/usr/bin/env bash
n=$(( RANDOM % 100 ))

if [[ n -eq 50 ]]; then   
  echo "Something went wrong"
  >&2 echo "The error was using a magic number"
  exit 1
fi

echo "Everything went according to plan"
```

```bash
#!/usr/bin/env bash
count=0
while true
do
     ./loop.sh &>> output.txt
     if [[ $? -ne 0 ]]; then
         cat output.txt
         echo "--------------------------"
         echo "failed after $count times"
         break
     fi
     ((count++))
done
```

### Day3 课堂练习

> 统计words文件中包含至少四个a且不以s 结尾的单词个数。这些单词中，出现频率前三的末尾三个字母是什么

```bash
cat words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){4}.*$" | grep -v "s$" | sed -E "s/.*([a-z]{3})$/\1/" | sort | uniq -c | sort -nk1,1 | tail -n3
```

1. `cat words`：读取文件 `words` 的内容并输出。
2. `tr "[:upper:]" "[:lower:]"`：将所有大写字母转换为小写字母。
3. `grep -E "^([^a]*a){4}.*$"`：使用正则表达式筛选出包含至少四个字母 ‘a’ 的行。
4. `grep -v "s$"`：排除以字母 ‘s’ 结尾的行。
5. `sed -E "s/.*([a-z]{3})$/\1/"`：使用正则表达式提取每行最后三个字母。
6. `sort`：对提取出的三字母字符串进行排序。
7. `uniq -c`：统计每个三字母字符串出现的次数。
8. `sort -nk1,1`：按出现次数对统计结果进行排序。
9. `tail -n3`：输出出现次数最少的三个三字母字符串。

### Day3 作业

> 请通过bash脚本和shell命令组合找出你最近五次启动Linux系统的最长启动时间、最短启动时间，并计算出平均启动时间。 注1：你可以执行shell命令`sudo reboot`重启Linux系统。 注2：你需要查询`journalctl`命令的手册，了解如何查看系统的启动时间。 注3：`journalctl`命令输出的有关系统启动时间的日志信息类似下面这样： `Jul 06 09:49:32 ubuntu systemd[1]: Startup finished in 3.634s (kernel) + 4.283s (userspace) = 7.918s.`

```bash
#!/usr/bin/env bash
count=0
rm ./out.log
while [[ $count -ne 5 ]]; do
    journalctl -b -$count | grep Startup | grep = | sed -E 's/.*= ([0-9\.]+)(s\.)$/\1/' >>out.log
    ((count++))
done

min=$(cat out.log | sort | head -n1)
max=$(cat out.log | sort | tail -n1)
average=$(cat out.log | paste -sd+ | bc -l | awk '{print $1/5}')

echo min:${min} max:${max} average:${average}
```

### Day4 课堂练习

> 找出最常用的10条命令

```bash
history | awk '{$1=""; print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10
```

## 如何查找命令的操作方法

* `man command`
* `command -h`
* `command --help`
* `tldr command`

## 权限

* 查看目录的权限信息：`ls -l`
* 目录、普通文件的权限不同
  * 目录 `d` 开头
  * 文件 `-` 开头
* 修改文件权限的方法：`chmod 权限转成的8进制三位数 文件名`
* `suid`：让普通用户临时拥有该文件的属主的执行权限，权限号码 `4xxx`
* `sticky`：允该目录下的文件的创建者删除自己的创建的文件，不允许其他人删除文件，权限号码`1xxx`

## shell 命令

* 管道前后的命令不知道彼此的存在，都只接受参数/输入，做本职工作
* `>` 重定向输出-覆盖
* `>>` 重定向输出-追加
* `<` 重定向输入
* shell 负责解析：管道、各个命令
* 命令负责解析：自己的参数
* 给只接受参数而不接受标准输入的命令传参：`command1 | xargs command2`

## shell 脚本

### 特殊变量

| 变量   | 描述                 |
| ---- | ------------------ |
| `$0` | 当前脚本的文件名           |
| `$n` | 传递给脚本或函数的第 n 个参数   |
| `$_` | 最后一个参数             |
| `$?` | 上一个命令的退出状态，或函数的返回值 |
| `!!` | 完整的上一个命令，包括参数      |

### 命令替换&进程替换

* 命令替换：结果是字符串、可用于赋值变量
* 进程替换：结果是管道输入、可用于命令的输入

```bash
 命令替换
output=$(ls dir1)
echo "$output"

 进程替换
diff <(ls dir1) <(ls dir2)
```

### 循环

```bash
#!/bin/bash
count=1
until [[ $count -gt 5 ]]; do
  echo "Count: $count"
  ((count++))
done
 循环的结尾是done

 ---

#!/bin/bash
count=1
while [[ $count -le 5 ]]; do
  echo "Count: $count"
  ((count++))
done

 ---

#!/bin/bash
for file in $(ls); do
  echo "File: $file"
done

for ((i = 1; i <= 5; i++)); do
  echo "Number: $i"
done

```

### 判断

```bash
#!/bin/bash
num=10

 多分支 if-elif-else 语句
if [[ $num -gt 10 ]]; then
  echo "$num is greater than 10"
elif [[ $num -eq 10 ]]; then
  echo "$num is equal to 10"
else
  echo "$num is less than 10"
fi
 判断的结尾是fi
```

### 比较运算符

| 运算符   | 描述   |
| ----- | ---- |
| `-eq` | 等于   |
| `-ne` | 不等于  |
| `-gt` | 大于   |
| `-lt` | 小于   |
| `-ge` | 大于等于 |
| `-le` | 小于等于 |

## vim

### 模式

| 模式名称 | 进入方法                    | 描述              |
| ---- | ----------------------- | --------------- |
| 正常模式 | 启动 Vim 后默认进入            | 移动光标、删除文本、复制粘贴  |
| 插入模式 | `i`、`I`、`a`、`A`、`o`、`O` | 插入文本            |
| 命令模式 | `:`                     | 执行保存、退出、查找替换等命令 |
| 可视模式 | `v`、`V`、`Ctrl+v`        | 选择文本块           |
| 替换模式 | `R`                     | 替换文本            |

### 常见的移动命令

* 移动命令前可加数字，代表重复次数

| 指令         | 描述                     |
| ---------- | ---------------------- |
| `h`        | 向左移动一个字符               |
| `j`        | 向下移动一行                 |
| `k`        | 向上移动一行                 |
| `l`        | 向右移动一个字符               |
| `w` `W`    | 移动到下一个单词的开头 （大写忽略标点符号） |
| `e` `E`    | 移动到当前单词的结尾 （大写忽略标点符号）  |
| `b` `B`    | 移动到上一个单词的开头（大写忽略标点符号）  |
| `0`        | 移动到行首                  |
| `^`        | 移动到行首的第一个非空白字符         |
| `$`        | 移动到行尾                  |
| `gg`       | 移动到文件的第一行              |
| `G`        | 移动到文件的最后一行             |
| `Ctrl + f` | 向前翻整页（forward）         |
| `Ctrl + b` | 向后翻整页（backward）        |
| `Ctrl + d` | 向下翻半页（down）            |
| `Ctrl + u` | 向上翻半页（up）              |

### 常见的编辑命令

* `y/c/d` + 移动命令为按移动方向进行复制/修改/删除

| 指令   | 描述         |
| ---- | ---------- |
| `i`  | 进入插入模式     |
| `a`  | 在光标后插入     |
| `o`  | 在当前行下方插入新行 |
| `O`  | 在当前行上方插入新行 |
| `x`  | 删除光标所在字符   |
| `dd` | 删除当前行      |
| `yy` | 复制当前行      |
| `p`  | 粘贴         |

## regex

### 特殊字符的含义

| 特殊字符       | 含义              |
| ---------- | --------------- |
| .          | 除换行符之外的任意单个字符   |
| \*         | 匹配前面字符零次或多次     |
| +          | 匹配前面字符一次或多次     |
| \[abc]     | 匹配 a、b、c 中的任意一个 |
| (RX1\|RX2) | 或               |
| ^          | 行首              |
| $          | 行尾              |

* `sed` 开启了 `-E` 才能使用`(ab)*` 进行匹配
* `/g`表示替换所有的匹配
* `cat ssh.log | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/’`:`\2`表示保留第二个括号内的内容 `s`表示替换

## 数据处理

| 命令        | 常见用法                                                        |
| --------- | ----------------------------------------------------------- |
| `sort`    | 对文件内容进行排序。例如：`sort file.txt` 按行排序。                          |
| `uniq`    | 去除有序文本的重复行，`uniq -c`在前面添加次数                                 |
| `awk`     | 强大的文本处理工具，用于模式匹配和数据提取。例如：`awk '{print $1}' file.txt` 打印第一列。 |
| `grep`    | 搜索文本中的模式。例如：`grep 'pattern' file.txt` 搜索包含 `pattern` 的行。    |
| `grep -v` | 反向匹配，显示不包含指定模式的行。例如：`grep -v 'pattern' file.txt`。           |
| `find`    | 查找文件和目录。例如：`find /path -name '*.txt'` 查找指定路径下的所有 `.txt` 文件。 |

## 信号

| 信号      | 作用                       | 键盘操作      |
| ------- | ------------------------ | --------- |
| SIGINT  | 中断程序执行                   | Ctrl + C  |
| SIGQUIT | 终止程序执行并生成一个核心转储          | Ctrl + \\ |
| SIGTERM | 通过kill命令发送，终止程序执行        | 无键盘操作     |
| SIGHUP  | 在终端连接被挂断或关闭时发送给进程，终止程序执行 | 无键盘操作     |
| SIGSTOP | 暂停程序执行                   | Ctrl + Z  |
| SIGCONT | 恢复程序执行                   | 无键盘操作     |
