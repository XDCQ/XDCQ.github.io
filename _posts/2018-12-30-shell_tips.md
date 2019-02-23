---
layout: post
title: some shell tips
description:
image: assets/images/shell.jpeg
---

# Contents  
- [基本语法](#basic)
- [字符串](#String)
-[数组](#Array)
- [here文档](#heredoc) 
- [关键字](#keyword) 
- [脚本预定义参数](#params) 
- [echo](#echo)
- [shell选项](#set)

## 基本语法(basic)

- 控制语句
```
# if控制语句
if [ 表达式 ];then
// 逻辑分支
elif
// 逻辑分支
else
// 逻辑分支
fi

# for循环
for file in `ls .`;
do
// 循环体逻辑
done

# while循环
while true;
do
// while循环体
done

# case控制语句
way="b"
case $way in
a)
        echo "aa";
;;
b)
        echo "bb";
;;
c)
        echo "cc";
;;
*)
        echo $1;
;;
esac
```

- 函数定义执行
```
#定义函数fn
fn()
{
// 函数体
}

#调用fn
fn

#获取函数输出内容
result=`fn`
```

- 变量取赋值
```
#定义变量
foo="bar"
#获取变量
echo $foo
```

## 字符串(String)

### 单引号和双引号

**单引号字符串的限制：**

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）；

**双引号的优点：**

- 双引号里可以有变量；
- 双引号里可以出现转义字符；

### 字符串截取

```
#定义一个变量
xdcq@XDCQ-ubuntu:~$ var=http://www.aaa.com/123.htm
xdcq@XDCQ-ubuntu:~$ echo $var
http://www.aaa.com/123.htm
```

1  '#'号截取，删除左边字符，保留右边字符。
```
#*// 表示从左边开始删除第一个 // 号及左边的所有字符
xdcq@XDCQ-ubuntu:~$ echo ${var#*//}
www.aaa.com/123.htm
```
2  '##'号截取，删除左边字符，保留右边字符。
```
##*/ 表示从左边开始删除最后（最右边）一个 / 号及左边的所有字符
xdcq@XDCQ-ubuntu:~$ echo ${var##*/}
123.htm
```
3  %号截取，删除右边字符，保留左边字符
```
#%/* 表示从右边开始，删除第一个 / 号及右边的字符
xdcq@XDCQ-ubuntu:~$ echo ${var%/*}
http://www.aaa.com
```
4  %% 号截取，删除右边字符，保留左边字符
```
#%%/* 表示从右边开始，删除最后（最左边）一个 / 号及右边的字符
xdcq@XDCQ-ubuntu:~$ echo ${var%%/*}
http:
```
5  从左边第几个字符开始，及字符的个数
```
#其中的 0 表示左边第一个字符开始，5 表示字符的总个数。
xdcq@XDCQ-ubuntu:~$ echo ${var:0:5}
http:
```
6  从左边第几个字符开始，一直到结束。
```
#其中的 7 表示左边第8个字符开始，一直到结束。
xdcq@XDCQ-ubuntu:~$ echo ${var:7}
www.aaa.com/123.htm
```
7  从右边第几个字符开始，及字符的个数
```
#其中的 0-7 表示右边算起第七个字符开始，3 表示字符的个数。
xdcq@XDCQ-ubuntu:~$ echo ${var:0-7:3}
123
```
8  从右边第几个字符开始，一直到结束。
```
#表示从右边第七个字符开始，一直到结束。
xdcq@XDCQ-ubuntu:~$ echo ${var:0-7}
123.htm
```
>注：（左边的第一个字符是用 0 表示，右边的第一个字符用 0-1 表示)
> '#、##' 表示从左边开始删除。一个 # 表示从左边删除到第一个指定的字符；两个 # 表示从左边删除到最后一个指定的字符。
>'%、%%' 表示从右边开始删除。一个 % 表示从右边删除到第一个指定的字符；两个 % 表示从左边删除到最后一个指定的字符。

删除包括了指定的字符本身.

## 数组(Array)



## here文档(heredoc)
操作数据库的场景
```
count()
{
sqlplus -S user/password@sid <<EOF
select count(1) user;
EOF
}
echo "`count1`"
```

## 关键字(keyword)

### local局部变量
```
local局部变量
[root@vm3 ~]# function defval(){
> a=66
> }
[root@vm3 ~]# echo $a
[root@vm3 ~]# defval
[root@vm3 ~]# echo $a
66
[root@vm3 ~]# function defval(){
> local a=3
> }
[root@vm3 ~]# defval
[root@vm3 ~]# echo $a
66﻿​
```

## 脚本预定义参数(params)
|变量|含义|
|-|-|
|$0|当前脚本的文件名|
|$n|传递给脚本或函数的参数|
|$#|传递给脚本或函数的参数个数|
|$*|传递给脚本或函数的所有参数|
|$@|传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同|
|$?|上个命令的退出状态，或函数的返回值|
|$$|当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID,如果是子shell，则为父进程id|
|$-|当前的选项集|

**$* 和 $@ 的区别**

- 不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数

- 被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体

## 退出状态
所谓退出状态，就是上一个命令执行后的返回结果。
一般情况下，大部分命令执行
>成功0，失败 1。

不过，也有一些命令返回其他值，表示不同类型的错误。

## echo(echo)

echo -n 不换行输出

echo -e 处理特殊字符

若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出： 
\a 发出警告声； 
\b 删除前一个字符； 
\c 最后不加上换行符号； 
\f 换行但光标仍旧停留在原来的位置； 
\n 换行且光标移至行首； 
\r 光标移至行首，但不换行； 
\t 插入tab； 
\v 与\f相同； 
\ 插入\字符； 
\nnn 插入nnn（八进制）所代表的ASCII字符；

## shell选项(set)

https://www.cnblogs.com/tychyg/p/5069011.html

## 初始化

shell（这里指bash）的初始化过程是这样的：

1. bash 检查文件/etc/profile 是否存在;
2. bash 检查主目录下的文件.bash_profile 是否存在。
3. bash 检查主目录下的.bash_login 是否存在。
4. bash 检查主目录下的文件.profile 是否存在