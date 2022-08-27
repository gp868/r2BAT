# Bash

# 什么是 Bash？

Bash（GNU Bourne-Again Shell）是一个为 GNU 计划编写的 Unix shell，它是许多 Linux 平台默认使用的 shell。

`shell` 是一个命令解释器，是介于操作系统内核与用户之间的一个绝缘层。准确地说，它也是能力很强的计算机语言，被称为`解释性语言`或`脚本语言`。

它可以通过将系统调用、公共程序、工具和编译过的二进制程序”粘合“在一起来建立应用，这是大多数脚本语言的共同特征，所以有时候脚本语言又叫做“`胶水语言`”。

事实上，所有的 UNIX 命令和工具再加上公共程序，对于 shell 脚本来说，都是可调用的。Shell 脚本对于管理系统任务和其它的重复工作的例程来说，表现的非常好，根本不需要那些华而不实的成熟紧凑的编译型程序语言。
## 一个小练习
- 编写 Hello World
```
vim hello.sh
```
使用 `vim` 编辑 `hello.sh`，输入如下代码并保存：

```
#!/bin/bash
# This is a comment
echo Hello World
```
vim 中插入按 `i`，保存并退出换行按 `esc` 然后输入 `:wq` 再按下 `enter`。

`#!` 是说明 hello 这个文件的类型，有点类似于 Windows 系统下用不同文件后缀来表示不同文件类型的意思（但不相同）。

Linux 系统根据 `#!` 及该字符串后面的信息确定该文件的类型，可以通过 man magic 命令 及 /usr/share/magic 文件来了解这方面的更多内容。

在 BASH 中 第一行的 #! 及后面的 /bin/bash 就表明该文件是一个 BASH 程序，需要由 /bin 目录下的 bash 程序来解释执行，BASH 这个程序一般是存放在 /bin 目录下。

如果你的 Linux 系统比较特别，bash 也有可能被存放在 /sbin 、/usr/local/bin 、/usr/bin 、/usr/sbin 或 /usr/local/sbin 这样的目录下。

如果还找不到，你可以用 `locate bash` ,`find / -name bash 2>/dev/null `或 `whereis bash` 这三个命令找出 bash 所在的位置；如果仍然找不到，那你可能需要自己动手安装一个 BASH 软件包了。

第二行的 `# This is a ...` 就是 BASH 程序的注释，在 BASH 程序中从 `#` 号（注意：后面紧接着是 ! 号的除外）开始到行尾的部分均被看作是程序的注释。

第三行的 `echo` 语句的功能是把 `echo` 后面的字符串输出到标准输出中去。由于 echo 后跟的是 "Hello World" 这个字符串，因此 "Hello World"这个字串就被显示在控制台终端的屏幕上了。

> 需要注意的是 `BASH` 中的绝大多数语句结尾处都**没有分号**。

- 运行 Bash 脚本的方式：

```
# 使用shell来执行
sh hello.sh

# 使用bash来执行
bash hello.sh

# 使用.来执行
. ./hello.sh

# 使用source来执行
source hello.sh

# 还可以赋予脚本所有者执行权限，允许该用户执行该脚本
chmod u+rx hello.sh
./hello.sh
```

- 使用重定向

比如我们想要保存刚刚的 hello world 为一个文本，那么该怎么办呢？

`>` 这个符号是重定向，执行以下代码，就会在当前目录下生成一个 my.txt。打开看看有没有 hello world

```
  #!/bin/bash
  echo "Hello World" > my.txt
```

- 使用脚本清除 /var/log 下的 log 文件

首先我们看一看 /var/log/dpkg.log 里面有啥东西。

```
cat /var/log/dpkg.log
```
这个文件中记录了我们使用 apt 安装的软件包的一些信息，现在我们需要写一个脚本把里面的东西清空，但是保留文件。

```
vim cleanlogs.sh
```

`/dev/null` 这个东西可以理解为一个黑洞，里面是空的（可以用 cat 命令看一看）。

```
#!/bin/bash

# 初始化一个变量
LOG_DIR=/var/log

cd $LOG_DIR

cat /dev/null > dpkg.log

echo "Logs cleaned up."

exit
```

运行脚本前，先使用 `sudo chmod +x cleanlogs.sh` 授予脚本执行权限，然后再看看 /var/log/dpkg.log 文件内是否有内容。运行此脚本后，文件的内容将被清除。

- 执行：

由于脚本中含有对系统日志文件内容的清除操作，这要求要有管理员权限.不然会报 `permission denied` 错误。使用 `sudo` 命令调用管理员权限才能执行成功：

```
sudo ./cleanlogs.sh
```

`#!/bin/bash` 这一行是表示使用 `/bin/bash` 作为脚本的解释器，这行要放在脚本的行首并且不要省略。

脚本正文中以 `#` 号开头的行都是注释语句，这些行在脚本的实际执行过程中不会被执行。这些注释语句能方便我们在脚本中做一些注释或标记，让脚本更具可读性。

- 遇到权限不够的提示，为什么？如何解决？

权限不够加 `sudo` 啊，可是你会发现 sudo cat /dev/null > /var/log/dpkg.log 一样会提示权限不够，为什么呢？

因为 `sudo` 只能让 `cat` 命令以 `root` 的权限执行，而对于 `>` 这个符号并没有 `root` 的权限。

我们可以使用 `sudo sh -c "cat /dev/null > /var/log/dpkg.log"` 让整个命令都具有 `root` 的权限执行。

# bash特殊字符
## 注释（#）
行首以 # 开头(除#!之外)的是注释。#! 是用于指定当前脚本的解释器，我们这里为 bash，且应该指明完整路径，所以为 /bin/bash。

当然，在 echo 中转义的 # 是不能作为注释的：

vim test.sh
copy
输入如下代码，并保存。（中文为注释，不需要输入）

```
#!/bin/bash

echo "The # here does not begin a comment."
echo 'The # here does not begin a comment.'
echo The \# here does not begin a comment.
echo The # 这里开始一个注释
echo $(( 2#101011 ))     # 数制转换（使用二进制表示），不是一个注释，双括号表示对于数字的处理
```
执行脚本，查看输出：

```
bash test.sh
```


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/7d22e36e-4d4d-4a11-ab07-1db25e639c70.png)

上面的脚本说明了如何使用 `echo` 打印出一段字符串和变量内容，这里采用了几种不同的方式，希望你可以理解这几种不同方式的异同。

## 分号（;）
- 命令分隔符

使用分号 `;` 可以在同一行上写两个或两个以上的命令。

```
vim test2.sh
```
输入如下代码，并保存：

```
#!/bin/bash
echo hello; echo there
filename=ttt.sh
if [ -e "$filename" ]; then    # 注意: "if"和"then"需要分隔，-e用于判断文件是否存在
    echo "File $filename exists."; cp $filename $filename.bak
else
    echo "File $filename not found."; touch $filename
fi; echo "File test complete."
```

执行脚本：

```
bash test2.sh
```
查看结果：

```
ls
```

上面脚本使用了一个 `if` 分支判断一个文件是否存在，如果文件存在打印相关信息并将该文件备份；如果不存在打印相关信息并创建一个新的文件。最后将输出"测试完成"。

- 终止 case 选项（双分号）

使用双分号 `;;` 可以终止 `case` 选项。

```
vim test3.sh
```
输入如下代码，并保存。

```
#!/bin/bash

varname=b

case "$varname" in
    [a-z]) echo "abc";;
    [0-9]) echo "123";;
esac
```
执行脚本，查看输出

```
bash test3.sh
abc
```

上面脚本使用 `case` 语句，首先创建了一个变量初始化为 b,然后使用 `case` 语句判断该变量的范围，并打印相关信息。如果你有其它编程语言的经验，这将很容易理解。

## 点号（.）
等价于 `source` 命令，bash 中的 `source` 命令用于在当前 bash 环境下读取并执行 FileName.sh 中的命令。

```
source test.sh

. test.sh
```

## 引号
 - 双引号（`"`)

"STRING" 将会阻止（解释）STRING 中`大部分`特殊的字符。

- 单引号（`'`）

'STRING' 将会阻止 STRING 中`所有`特殊字符的解释，这是一种比使用`"`更强烈的形式。

- 区别

这里举一个例子，能够更加生动的说明：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/d5efac9d-8d41-4cfc-ba13-261755a5b936.png)

同样是 `$HOME`，单引号会直接认为是字符，而双引号认为是一个变量。

## 斜线和反斜线
- 斜线（`/`）

文件名路径分隔符。分隔文件名不同的部分（如 `/home/bozo/projects/Makefile`），也可以用来作为除法算术操作符。

注意在 linux 中表示路径的时候，许多个 `/` 跟一个 `/` 是一样的。`/home/shiyanlou` 等同于 `////home///shiyanlou`。

- 反斜线（`\`）

一种对单字符的引用机制。

`\`通常用来转义双引号（`"`）和单引号（`'`），这样双引号和单引号就不会被解释成特殊含义了。

`\X` 将会“转义”字符 `X`。这等价于`"X"`，也等价于`'X'`。


| 符号 |                说明                 |
| :--: | :---------------------------------: |
|  \n  |            表示新的一行             |
|  \r  |              表示回车               |
|  \t  |           表示水平制表符            |
|  \v  |           表示垂直制表符            |
|  \b  |             表示后退符              |
|  \a  |      表示"alert"(蜂鸣或者闪烁)      |
| \0xx | 转换为八进制的 ASCII 码, 等价于 0xx |
| `\"` |         表示引号字面的意思          |

转义符也提供续行功能，也就是编写多行命令的功能。

每一个单独行都包含一个不同的命令，但是每行结尾的转义符都会转义换行符，这样下一行会与上一行一起形成一个命令序列。

## 反引号（`）
反引号中的命令会优先执行，如：

```
cp `mkdir back` test.sh back
ls
```

先创建了 back 目录，然后复制 test.sh 到 back 目录。

## 冒号（:）
- 空命令

等价于`“NOP”`（no op，一个什么也不干的命令）。也可以被认为与 shell 的内建命令 `true` 作用相同。`:`命令是一个 bash 的内建命令，它的退出码（exit status）是（0）。

如：

```
#!/bin/bash

while :
do
    echo "endless loop"
done
```
等价于

```
#!/bin/bash

while true
do
    echo "endless loop"
done
```

可以在 `if/then` 中作占位符：

```
#!/bin/bash

condition=5

if [ $condition -gt 0 ] #gt表示greater than，也就是大于，同样有-lt（小于），-eq（等于）
then :   # 什么都不做，退出分支
else
    echo "$condition"
fi
```

- 变量扩展/子串替换

在与 `>` 重定向操作符结合使用时，将会把一个文件清空，但是并不会修改这个文件的权限。如果之前这个文件并不存在，那么就创建这个文件。

```
: > test.sh   # 文件“test.sh”现在被清空了
# 与 cat /dev/null > test.sh 的作用相同
# 然而,这并不会产生一个新的进程, 因为“:”是一个内建命令
```

在与 `>>` 重定向操作符结合使用时，将不会对预先存在的目标文件 `: >> target_file` 产生任何影响。如果这个文件之前并不存在，那么就创建它。

也可能用来作为注释行，但不推荐这么做。使用 `#` 来注释的话，将关闭剩余行的错误检查，所以可以在注释行中写任何东西。然而，使用 : 的话将不会这样。

`:` 还用来在 `/etc/passwd` 和 `$PATH` 变量中做分隔符，如：

```
echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/sbin:/usr/sbin:/usr/games
```

## 问号（?）
在一个双括号结构中，`?` 就是 C 语言的三元操作符，如：

```
vim test.sh
```
输入如下代码，并保存：

```
#!/bin/bash

 a=10
 (( t=a<50?8:9 ))
 echo $t
```
运行测试

```
bash test.sh
8
```

# bash特殊字符
## 小括号（( )）
- 命令组

在括号中的命令列表，将会作为一个`子 shell` 来运行。

在括号中的变量，由于是在子 shell 中，所以对于脚本剩下的部分是不可用的。

父进程，也就是脚本本身，将不能够读取在子进程中创建的变量，也就是在子 shell 中创建的变量。如：

```
vim test20.sh
```
输入代码：

```
#!/bin/bash

a=123
( a=321; )

echo "$a" #a的值为123而不是321，因为括号将判断为局部变量
```
运行代码：

```
bash test20.sh
a = 123
```
在圆括号中 `a` 变量，更像是一个局部变量。

- 初始化数组

创建数组

```
vim test21.sh
```
输入代码：

```
#!/bin/bash

arr=(1 4 5 7 9 21)
echo ${arr[3]} # get a value of arr
```
运行代码：

```
bash test21.sh
7
```

## 大括号（{ }）
- 文件名扩展

复制 `t.txt` 的内容到 `t.back` 中

```
vim test22.sh
```
输入代码：

```
#!/bin/bash

if [ ! -w 't.txt' ];
then
    touch t.txt
fi
echo 'test text' >> t.txt
cp t.{txt,back}
```
运行代码：

```
bash test22.sh
```
查看运行结果：

```
ls
cat t.txt
cat t.back
```
> 注意： 在大括号中，**不允许有空白**，除非这个空白被引用或转义。

- 代码块

代码块，又被称为`内部组`，这个结构事实上创建了一个匿名函数（一个没有名字的函数）。

然而，与“标准”函数不同的是，在其中声明的变量，对于脚本其他部分的代码来说还是`可见`的。

```
vim test23.sh
```
输入代码：

```
#!/bin/bash

a=123
{ a=321; }
echo "a = $a"
```
运行代码：

bash test23.sh
a = 321

变量 `a` 的值被更改了。

## 中括号（[ ]）
- 条件测试

条件测试表达式放在 `[]` 中。下列练习中的 `-lt` (less than)表示小于号。

```
vim test24.sh
```
输入代码：

```
#!/bin/bash

a=5
if [ $a -lt 10 ]
then
    echo "a: $a"
else
    echo 'a>=10'
fi
```
运行代码：

```
bash test24.sh
a: 5
```
双中括号`[[ ]]`也可用作条件测试（判断）。

- 数组元素

在一个 array 结构的上下文中，中括号用来引用数组中每个元素的编号。

```
vim test25.sh
```
输入代码：

```
#!/bin/bash

arr=(12 22 32)
arr[0]=10
echo ${arr[0]}
```
运行代码：

```
bash test25.sh
10
```

## 尖括号（< 和 >）
- 重定向

`test.sh > filename`：
重定向 test.sh 的输出到文件 filename 中。如果 filename 存在的话，那么将会被覆盖。

`test.sh &> filename`：
重定向 test.sh 的 stdout（标准输出）和 stderr（标准错误）到 filename 中。

`test.sh >&2`：
重定向 test.sh 的 stdout 到 stderr 中。

`test.sh >> filename`：
把 test.sh 的输出`追加`到文件 filename 中。如果 filename 不存在的话，将会被创建。

## 竖线（|）
- 管道

分析前边命令的输出，并将输出作为后边命令的输入。这是一种产生命令链的好方法。

```
vim test26.sh
```
输入代码：

```
#!/bin/bash

tr 'a-z' 'A-Z'
exit 0
```
现在让我们输送 `ls -l` 的输出到一个脚本中：

```
chmod 755 test26.sh
ls -l | ./test26.sh
```
输出的内容均变为了大写字母。

## 破折号（-）
- 选项，前缀

在所有的命令内如果想使用选项参数的话,前边都要加上`-`。

```
vim test27.sh
```
输入代码：

```
#!/bin/bash

a=5
b=5
if [ "$a" -eq "$b" ]
then
    echo "a is equal to b."
fi
```
运行代码：

```
bash test27.sh

a is equal to b.
```

- 用于重定向 stdin 或 stdout

下面脚本用于备份最后 24 小时当前目录下所有修改的文件：

```
vim test28.sh
```
输入代码：

```
#!/bin/bash

BACKUPFILE=backup-$(date +%m-%d-%Y)
# 在备份文件中嵌入时间.
archive=${1:-$BACKUPFILE}
#  如果在命令行中没有指定备份文件的文件名,
#  那么将默认使用"backup-MM-DD-YYYY.tar.gz".

tar cvf - `find . -mtime -1 -type f -print` > $archive.tar
gzip $archive.tar
echo "Directory $PWD backed up in archive file \"$archive.tar.gz\"."

exit 0
```
运行代码：

```
bash test28.sh
ls
```

# 变量和参数
## 变量定义
- 概念

变量的名字就是变量保存值的地方。引用变量的值就叫做变量替换。

如果 variable 是一个变量的名字，那么 `$variable` 就是引用这个变量的值，即这变量所包含的数据。

`$variable` 事实上只是 `${variable}` 的简写形式。在某些上下文中 `$variable` 可能会引起错误，这时候你就需要用 `${variable}` 了。

- 定义变量

定义变量时，变量名不加美元符号（$，PHP 语言中变量需要），如：

```
myname="shiyanlou"
```
> 注意：变量名和等号之间不能有空格。

同时，变量名的命名须遵循如下规则：

1. 首个字符必须为字母（a-z，A-Z）。
2. 中间不能有空格，可以使用下划线（_）。
3. 不能使用标点符号。
4. 不能使用 bash 里的关键字（可用 help 命令查看保留关键字）。

除了直接赋值，还可以用语句给变量赋值，如：

```
for file in `ls /etc`
```
## 使用变量
变量名前加美元符号，如：

```
myname="shiyanlou"
echo $myname
echo ${myname}
echo ${myname}Good
echo $mynameGood

myname="miao"
echo ${myname}
```

加花括号帮助解释器识别变量的边界，若不加，解释器会把 mynameGood 当成一个变量（值为空）。

推荐给所有变量加花括号，已定义的变量可以重新被定义。

## 只读变量
使用 `readonly` 命令可以将变量定义为只读变量，只读变量的值不能被改变。 下面的例子尝试更改只读变量，结果报错：

```
#!/bin/bash
myUrl="http://www.shiyanlou.com"
readonly myUrl
myUrl="http://www.shiyanlou.com"
```

运行脚本，结果如下：
![](https://files.mdnice.com/user/24021/f3186194-8808-4365-9462-b2535b9a3b7e.png)

## 特殊变量
- 局部变量

这种变量只有在代码块或者函数中才可见。

- 环境变量

这种变量将影响用户接口和 shell 的行为。

在通常情况下，每个进程都有自己的“环境”，这个环境是由一组变量组成的，这些变量中存有进程可能需要引用的信息。在这种情况下，shell 与一个一般的进程没什么区别。

- 位置参数

从命令行传递到脚本的参数：$0，$1，$2，$3...

`$0` 就是脚本文件自身的名字，`$1` 是第一个参数，`$2` 是第二个参数，`$3` 是第三个参数，然后是第四个。`$9` 之后的位置参数就必须用大括号括起来了，比如，${10}，${11}，${12}。
| 命令 |                             解释                             |
| :--: | :----------------------------------------------------------: |
|  $#  |                     传递到脚本的参数个数                     |
|  $*  | 以一个单字符串显示所有向脚本传递的参数。与位置变量不同,此选项参数可超过 9 个 |
|  $$  |                   脚本运行的当前进程 ID 号                   |
|  $!  |              后台运行的最后一个进程的进程 ID 号              |
|  $@  |      与 $* 相同,但是使用时加引号,并在引号中返回每个参数      |
|  $   |        显示 shell 使用的当前选项,与 set 命令功能相同         |
|  $?  | 显示最后命令的退出状态。 0 表示没有错误,其他任何值表明有错误。 |

- 位置参数实例
这个十分重要，在我们运行一套脚本的时候，有时候是需要参数的，这里我们教大家如何获取参数。

```
vim test30.sh
```
输入代码（中文皆为注释，不用输入）：

```
#!/bin/bash

# 作为用例, 调用这个脚本至少需要10个参数, 比如：
# bash test.sh 1 2 3 4 5 6 7 8 9 10
MINPARAMS=10

echo

echo "The name of this script is \"$0\"."

echo "The name of this script is \"`basename $0`\"."


echo

if [ -n "$1" ]              # 测试变量被引用.
then
echo "Parameter #1 is $1"  # 需要引用才能够转义"#"
fi

if [ -n "$2" ]
then
echo "Parameter #2 is $2"
fi

if [ -n "${10}" ]  # 大于$9的参数必须用{}括起来.
then
echo "Parameter #10 is ${10}"
fi

echo "-----------------------------------"
echo "All the command-line parameters are: "$*""

if [ $# -lt "$MINPARAMS" ]
then
 echo
 echo "This script needs at least $MINPARAMS command-line arguments!"
fi

echo

exit 0
```
运行代码：

```
bash test30.sh 1 2 10


The name of this script is "test.sh".
The name of this script is "test.sh".

Parameter #1 is 1
Parameter #2 is 2
-----------------------------------
All the command-line parameters are: 1 2 10

This script needs at least 10 command-line arguments!
```

# 基本运算符
## 算数运算符

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/26d2cecc-4ca0-46c3-945a-33314a851edf.png)

```
vim test.sh
```

```
#!/bin/bash

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a == b"
fi
if [ $a != $b ]
then
   echo "a != b"
fi
```

运行

```
bash test.sh

a + b : 30
a - b : -10
a * b : 200
b / a : 2
b % a : 0
a != b
```

**TIPS**

原生 bash 不支持简单的数学运算，但是可以通过其他命令来实现，例如 `awk` 和 `expr`，`expr` 最常用。

`expr` 是一款表达式计算工具，使用它能完成表达式的求值操作，注意使用的`反引号`（esc 键下边）

表达式和运算符之间要有空格， `$a + $b` 写成 `$a+$b` 不行

条件表达式要放在方括号之间，并且要有空格 `[ $a == $b ]` 写成 `[$a==$b]` 不行

乘号（`*`）前边必须加反斜杠（`\`)才能实现乘法运算

## 关系运算符
> 关系运算符`只支持数字`，不支持字符串，除非字符串的值是数字。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a788cd04-a021-4905-9cf1-6c82c7fbd5a4.png)


实例

```
vim test2.sh
```

```
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a == b"
else
   echo "$a -eq $b: a != b"
fi
```

运行

```
bash test2.sh

10 -eq 20: a != b
```

## 逻辑运算符

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/8c574548-642e-4082-a9d5-5c4448025813.png)

实例

```
#!/bin/bash
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "return true"
else
   echo "return false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "return true"
else
   echo "return false"
fi
```

结果

```
return false
return true
```

## 字符串运算符

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/046f51b9-1c41-4668-b4e6-82cd47cdc6be.png)

```
#!/bin/bash

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a == b"
else
   echo "$a = $b: a != b"
fi
if [ -n $a ]
then
   echo "-n $a : The string length is not 0"
else
   echo "-n $a : The string length is  0"
fi
if [ $a ]
then
   echo "$a : The string is not empty"
else
   echo "$a : The string is empty"
fi
```

结果

```
abc = efg: a != b
-n abc : The string length is not 0
abc : The string is not empty
```
## 文件测试运算符

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/7af21446-8bdd-4488-a896-c7c40572d63d.png)

实例：

```
#!/bin/bash

file="/home/shiyanlou/test.sh"
if [ -r $file ]
then
   echo "The file is readable"
else
   echo "The file is not readable"
fi
if [ -e $file ]
then
   echo "File exists"
else
   echo "File not exists"
fi
```
结果

```
The file is readable
File exists
```

## 浮点运算

浮点运算，比如实现求圆的面积和周长。

`expr` 只能用于整数计算，可以使用 `bc` 或者 `awk` 进行浮点数运算。

```
#!/bin/bash

radius=2.4

pi=3.14159

girth=$(echo "scale=4; 3.14 * 2 * $radius" | bc)

area=$(echo "scale=4; 3.14 * $radius * $radius" | bc)

echo "girth=$girth"

echo "area=$area"
```

以上代码如果想在环境中运行，需要先安装 `bc`。

```
sudo apt-get update
sudo apt-get install bc
```

# 流程控制
## if else
和 Java、PHP 等语言不一样，sh 的流程控制不可为空

在 sh/bash 里可不能这么写，如果 else 分支没有语句执行，就不要写这个 else。

- if

if 语句语法格式：

```
if condition
then
    command1
    command2
    ...
    commandN
fi
```
- if else

if else 语法格式：

```
if condition
then
    command1
    command2
    ...
    commandN
else
    command
fi
```
if-elif-else 语法格式：

```
if condition1
then
    command1
elif condition2
then
    command2
else
    commandN
fi
```
以下实例判断两个变量是否相等：

```
a=10
b=20
if [ $a == $b ]
then
   echo "a == b"
elif [ $a -gt $b ]
then
   echo "a > b"
elif [ $a -lt $b ]
then
   echo "a < b"
else
   echo "Ineligible"
fi
```
输出结果：

```
a < b
```
if else 语句经常与 `test` 命令结合使用

```
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo 'Two numbers are equal!'
else
    echo 'The two numbers are not equal!'
fi
```
输出结果：

```
Two numbers are equal!
```

## for 循环
for 循环一般格式为：

```
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```
例如：

- 顺序输出当前列表中的数字：

```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
输出结果：

```
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```
- 顺序输出字符串中的字符：

```
for str in This is a string
do
    echo $str
done
```
输出结果：

```
This
is
a
string
```

## while 语句
while 循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。

其格式为：

```
while condition
do
    command
done
```
```
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```
运行脚本，输出：

```
1
2
3
4
5
```

如果 int 小于等于 5，那么条件返回真。int 从 1 开始，每次循环处理时，int 加 1。运行上述脚本，返回数字 1 到 5，然后终止。

使用了 Bash `let` 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量，与`((...))`等价。

while 循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量 MAN，按 `<Ctrl+D>` 结束循环。

```
echo 'press <CTRL-D> exit'
echo -n 'Who do you think is the most handsome: '
while read MAN
do
    echo "Yes! $MAN is really handsome"
done
```

## 无限循环
无限循环语法格式：

```
while :
do
    command
done
```
或者
```
while true
do
    command
done
```
或者

```
for (( ; ; ))
```

## until 循环
until 循环执行一系列命令直至条件为真时停止。 until 循环与 while 循环在处理方式上刚好相反。 

一般 while 循环优于 until 循环，但在某些时候，也只是极少数情况下，until 循环更加有用。 

until 语法格式:

```
until condition
do
    command
done
```
> 注意：条件可为任意测试条件，`测试发生在循环末尾，因此循环至少执行一次`。

## case
Shell case 语句为多选择语句。可以用 case 语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。

case 语句格式如下：

```
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac
```
取值后面必须为单词 `in`，每一模式必须以`右括号`结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 `;;`。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 `*` 捕获该值，再执行后面的命令。

下面的脚本提示输入 1 到 4，与每一种模式进行匹配：

```
echo 'Enter a number between 1 and 4:'
echo 'The number you entered is:'
read aNum
case $aNum in
    1)  echo 'You have chosen 1'
    ;;
    2)  echo 'You have chosen 2'
    ;;
    3)  echo 'You have chosen 3'
    ;;
    4)  echo 'You have chosen 4'
    ;;
    *)  echo 'You did not enter a number between 1 and 4'
    ;;
esac
```
输入不同的内容，会有不同的结果，例如：

```
Enter a number between 1 and 4:
The number you entered is:
3
You have chosen 3
```

# 函数
## 函数定义
shell 中函数的定义格式如下：

```
[ function ] funname [()]

{

    action;

    [return int;]

}
```

- 说明：

可以带 function fun() 定义，也可以直接 fun() 定义，不带任何参数。

参数返回，可以加return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return 后跟数值 n(0-255)

下面的例子定义了一个函数并进行调用：

```
#!/bin/bash

demoFun(){
    echo "This is my first shell function!"
}
echo "-----Execution-----"
demoFun
echo "-----Finished-----"


Output the result：
-----Execution-----
This is my first shell function!
-----Finished-----
```
下面定义一个带有 return 语句的函数：

```
#!/bin/bash
funWithReturn(){
    echo "This function will add the two numbers of the input..."
    echo "Enter the first number: "
    read aNum
    echo "Enter the second number: "
    read anotherNum
    echo "The two numbers are $aNum and $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "The sum of the two numbers entered is $? !"
```
输出类似下面：

```
This function will add the two numbers of the input...
Enter the first number:
1
Enter the second number:
2
The two numbers are 1 and  2 !
The sum of the two numbers entered is 3 !
```
> 函数返回值在调用该函数后通过 `$?` 来获得，所有函数在使用前必须定义。

## 函数参数
在 Shell 中，调用函数时可以向其传递参数。在函数体内部，通过 `$n` 的形式来获取参数的值，例如：`$1` 表示第一个参数，`$2` 表示第二个参数...

带参数的函数示例：

```
#!/bin/bash
funWithParam(){
    echo "The first parameter is $1 !"
    echo "The second parameter is $2 !"
    echo "The tenth parameter is $10 !"
    echo "The tenth parameter is ${10} !"
    echo "The eleventh parameter is ${11} !"
    echo "The total number of parameters is $# !"
    echo "Outputs all parameters as a string $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```
输出结果：

```
The first parameter is 1 !
The second parameter is 2 !
The tenth parameter is 10 !
The tenth parameter is 34 !
The eleventh parameter is 73 !
The total number of parameters is 11 !
Outputs all parameters as a string 1 2 3 4 5 6 7 8 9 34 73 !
```
> 注意：`$10` 不能获取第十个参数，获取第十个参数需要 `${10}`。当 n>=10 时，需要使用 `${n}` 来获取参数。

| 命令 |                             解释                             |
| :--: | :----------------------------------------------------------: |
|  $#  |                     传递到脚本的参数个数                     |
|  $*  | 以一个单字符串显示所有向脚本传递的参数。与位置变量不同,此选项参数可超过 9 个 |
|  $$  |                   脚本运行的当前进程 ID 号                   |
|  $!  |              后台运行的最后一个进程的进程 ID 号              |
|  $@  |      与 $* 相同,但是使用时加引号,并在引号中返回每个参数      |
|  $   |        显示 shell 使用的当前选项,与 set 命令功能相同         |
|  $?  | 显示最后命令的退出状态。 0 表示没有错误,其他任何值表明有错误。 |