

# shell学习笔记

### 1. shell介绍

我们常说的shell是shell脚本(shell script)的意思，是一种用shell语言编写的脚本程序

##### 1.1 shell环境

shell编程和php、javascript一样，只要一个用来编辑代码的问题本编辑器和脚本解释器就可以了

linux的shell解释器有很多：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）
- ……

这里我们使用`bash`，bash在日常工作中被广泛使用，而且也是大多数linux系统默认使用的shell解释器,

##### 1.2 编写第一个shell脚本

```bash
#告诉系统使用哪个解释器解释这个脚本,也可以使用php，python，就是告诉内核使用什么解释器解释下面的命令
#!/bin/bash
# echo 是用于窗口输出
echo 'hello world!'
```

##### 1.3 执行shell脚本

执行shell有两种方法：

- **做为可执行程序**：修改脚本执行权限，通过`./test.sh`执行，如果不加`./`，linux会去PATH中寻找test.sh，通常情况下个人目录并不在PATH中，所以要告诉系统在当前目录下找test.sh
- **作为解释器的参数执行**：`/bin/sh test.sh`或者`/bin/bash test.sh`

### 2. shell变量

##### 2.1 变量的分类

shell分为三种变量:

- **局部变量**：局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
- **环境变量**:所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量
- **shell变量**：是由shell程序设置的特殊变量，shell变量中有一部分是环境变量，一部分是局部变量

##### 2.2 变量的定义和使用

- **定义变量时，不需要加其他任何符号，但是在使用的时候需要用`$`,而且变量和`=`之间不能有空格**

  ```bash
  name='yangguang'
  echo $name
  ```

- 变量名最好加上`{}`花括号，使解释器区分变量和其他值，也可以提高代码的可读性，

  ```bash
  #不加花括号也是可以的，但如果需要在引号中解析变量，就需要用到花括号，同php一样
  name=java
  echo ${name}script
  ```

- **只读变量**，需要用`readonly`关键字，只读变量不能被修改

  ```bash
  name=yang
  #注意！readonly的时候不能添加$
  readonly name
  #当修改变量的时候，运行时会提醒该变量只读，不能被手动删除
  #readonly的变量怎么删除呢，其实readonly的变量只存在于当前shell开辟的内存中，关闭当前shell，内存就会被自动释放，就像新打开一个命令行，就是一个新的shell，原来的命令行中的只读变量并不能在这个命令行中输出，关闭原来的命令行，内存就会释放
  ```

- **删除变量**，使用`unset`，

  ```bash
  name=yangguang
  #注意！unset的时候不能添加$
  unset name
  ```

##### 2.3 字符串变量

字符串变量的定义可以使用单引号，可以使用双引号，也可以不使用引号，单引号和双引号的区别同php

字符串变量常用用法:

- **获取字符串长度**

  ```bash
  string='baidu'
  echo ${#string} #输出5
  ```

- **截取字符串**

  ```bash
  string='baidu'
  echo ${string:1:4} #截取字符串从第一位到第四位（从0位开始），同php的substr
  ```

- 查找子字符串，查找字符 **i** 或 **o** 的位置(哪个字母先出现就计算哪个)

  ```bash
  string="runoob is a great site"
  #注意！这里是反引号，而不是单引号 , expr是linux的手工命令行计数工具
  echo `expr index "$string" io`  # 输出 4
  ```

##### 2.4 数组变量

**shell支持一维数组，不支持多维数组，且不限制数组大小**

- **定义数组**

  ```bash
  #数组名=(值1 值2 值3 ...)
  #或者可以这样定义
  #数组名[0]=值1
  #数组名[1]=值2
  #数组名[2]=值3
  #也可以不使用连续的下标
  ```

- **获取数组**

  ```bash
  #${数组名[下标]}
  #@符可以代表所有的元素
  echo ${test_array[@]}
  ```

- **获取数组元素个数或者某个元素长度**

  ```bash
  # 取得数组元素的个数
  length=${#array_name[@]}
  # 或者
  length=${#array_name[*]}
  # 取得数组单个元素的长度
  lengthn=${#array_name[n]}
  ```

### 3. 向脚本内传递参数

我们可以向shell脚本中传入参数，脚本中获取参数的格式是`$n`，n为数字，0表示执行的文件名,1表示第一个参数，2表示第二个参数

```bash
echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

另外，还有几个常用的特殊字符来处理参数

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| $#   | 获取所有传入参数个数                                         |
| $*   | 以一个字符串的格式显示所有传入的参数，                       |
| $$   | 获取脚本运行的当前进程ID                                     |
| $!   | 后台运行的最后一个进程ID                                     |
| $@   | 也是显示所有传入的参数，但是和$*不同的是，有多少个参数显示多少个 |

### 4. shell基本运算符

shell支持的运算符：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

##### 4.1 算术运算符

原生的 bash不支持算数运算，可以通过awk或者expr工具实现，常用的是expr工具

```bash
value1=1
value2=2
#注意！完整的表达式是被反引号包含，而不是单引号，而且运算符两边都需要有空格
echo `expr ${value1} + ${value2}` #输出3
```

> 基本的算数运算也可以使用`[]`中括号，如：
>
> a=1
>
> b=2
>
> echo $[a+b]

常用的算数运算符

| 运算符 | 说明                                                    | 举例                                       |
| :----- | :------------------------------------------------------ | :----------------------------------------- |
| +      | 加法                                                    | `expr $a + $b` 结果为 30。                 |
| -      | 减法                                                    | `expr $a - $b` 结果为 -10。                |
| *      | 乘法（*前必须加反斜杠转义）                             | `expr $a \* $b` 结果为  200。*需要被转义   |
| /      | 除法                                                    | `expr $b / $a` 结果为 2。                  |
| %      | 取余                                                    | `expr $b % $a` 结果为 0。                  |
| =      | 赋值                                                    | a=$b 将把变量 b 的值赋给 a。不需要用到expr |
| ==     | 相等。用于比较两个数字，相同则返回 true。（注意空格！） | [ $a == $b ] 返回 false。不需要用到expr    |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。           | [ $a != $b ] 返回 true。不需要用到expr     |

> 注意：在MAC上，可以用$(($a * $b))代替 反引号加expr的写法，而且*也不用加反斜杠

##### 4.2 关系运算符

关系运算符只支持数字或者字符串格式的数字，不支持字符串

| 运算符              | 说明                                                  | 举例                       |
| :------------------ | :---------------------------------------------------- | :------------------------- |
| -eq（equal）        | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne（not equal）    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt (greater than)  | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt（less than）    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge (greater equal) | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le (less equal)    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

##### 4.3 布尔运算符

| 运算符 | 说明                                                | 举例                                     |
| :----- | :-------------------------------------------------- | :--------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

```sh
a=10
b=20
# ！！注意！！ 布尔运算符需要使用单个的中括号[] 
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a == $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

##### 4.4 逻辑运算符

| 运算符 | 说明                     | 举例                                       |
| :----- | :----------------------- | :----------------------------------------- |
| &&     | 逻辑的 AND(同php中的&&)  | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\|   | 逻辑的 OR(同php中的\|\|) | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

> 布尔运算符和逻辑运算符的区别在于：
>
> 逻辑运算符有一个短路的功能，如果在逻辑与中，如果一个是false，后面的就不执行，在逻辑或中，第一个是true，第二个就不执行
>
> 布尔运算符没有这个功能
>
> 而且逻辑运算符需要用的双中括号[[]]，也可以[xxx]&&[xxx]
>
> 布尔运算符用到单中括号[]

```sh
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

##### 4.5 字符串运算符

| 运算符 | 说明                                         | 举例                     |
| :----- | :------------------------------------------- | :----------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。    | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否为空，不为空返回 true。        | [ $a ] 返回 true。       |

```bash
#!/bin/bash
a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n "$a" ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

##### 4.6 文件测试运算符

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

```sh
#!/bin/bash

file="/var/www/runoob/test.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

### 5. echo命令

shell中的echo命令和php中的echo命令类似，常用的用法就不介绍了，这些写一些有意思的用法

```bash
#!/bin/sh
#read命令：是从标准输入中读取一行，并把
read name 
echo "$name It is a test"
```

### 6. printf命令

printf 命令模仿 C 程序库（library）里的 printf() 程序。

printf 由 POSIX 标准所定义，因此使用 printf 的脚本比使用 echo 移植性好。

printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认 printf 不会像 echo 自动添加换行符，我们可以手动添加 \n。

printf 命令的语法：

```bash
printf  format-string  [arguments...]
```

格式说明：

- Format-string: 为格式控制字符串
- Arguments: 为参数列表

接下来显示printf的一些强大功能

```bash
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 
```

显示结果：

```bash
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```

%s %c %d %f都是格式替代符

%-10s 指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。

%-4.2f 指格式化为小数，其中.2指保留2位小数。

### 7. 流程控制

在shell中的流程控制不能为空，如在shell中的if else，如果else中没有需要执行的语句，就不要写这个else

##### 7.1 if...else...

语法：

```bash
#第一种，只有if的情况
if condition
then
    command1 
    command2
    ...
    commandN 
fi
#第二种：if else
if condition
then
    command1 
    command2
    ...
    commandN 
else
		command
fi
#第三种： if else-if else
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

##### 7.2 for循环

for一般格式：

```bash
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

也可以使用类C的格式

```bash
#注意！需要加两个括号
for ((a=1; a<11; a++))
do
  echo $[a+100]
done
```

##### 7.3 while 语句

语法：

```bash
while condition
do
    command
done
```

例子：

```bash
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

以上实例使用了 Bash let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<Ctrl-D>结束循环。

```bash
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done
```

##### 7.4 case

case的用法和php的switch...case相同, 在php中使用else捕获其他，在shell中使用`*`捕获其他

语法：

```bash
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

示例：

```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

##### 7.5 跳出循环

Break、continue 和php中用法相同

### 8. 函数

说明：

- 1、可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。
- 2、参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值

示例：

```bash
#/bin/zsh
test(){
		echo '第一个shell函数';
}
test
```

下面定义一个带返回的函数：

```bash
funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

函数的返回值通过`$?`获取

**函数传参:**

在shell中，向函数内传入参数是在函数后跟参数，以空格分隔，在函数内接受参数使用$1,$2,$3.... **需要注意的是，从第十个参数开始，需要用`${10},${11}..`来表示参数**

示例：

```bash
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    #输出的10
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本或函数的参数个数                                   |
| $*       | 以一个单字符串显示所有向脚本传递的参数                       |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

### 9. 文件包含

包含文件的语法如下：

```bash
. filename   # 注意点号(.)和文件名中间有一空格

或

source filename
```

### 实例

创建两个 shell 脚本文件。

test1.sh 代码如下：

```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

url="http://www.runoob.com"
```

test2.sh 代码如下：

```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "菜鸟教程官网地址：$url"
```

接下来，我们为 test2.sh 添加可执行权限并执行：

```
$ chmod +x test2.sh 
$ ./test2.sh 
菜鸟教程官网地址：http://www.runoob.com
```

> *被包含的文件 test1.sh 不需要可执行权限。*

