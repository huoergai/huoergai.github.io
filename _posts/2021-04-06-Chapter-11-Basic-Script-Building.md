---
title: Chapter-11 Basic Script Building
date: 2021-04-06 23:16
tags: 
     - Linux Command Line and Shell Scripting Bible, 3E
---
# Chapter-11 Basic Script Building #
```shell
#!bin/bash

# 1.第一个脚本文件
date
pwd

# 2.使用 echo 命令显示信息
echo "--> something important!"i
echo ""
echo "what time is it now?"

# 3.连续的显示
echo -n "It's:"
date

# 4.变量 - 环境变量
echo UID:$UID
echo HOME:$HOME

# 5.变量 - 用户变量
PI=3.14
echo "PI=${PI}"
# note: 赋值符号两端没有空格
pi=$PI
echo "pi=$pi"

# 6.命令替换: `和$()
echo date=`date`
echo where am I:$(pwd)
today=$(date +%y%m%d)

# 7.重定向-输入和输出
#   a.重定向输出：command > outputfile
echo ls -al > log.$today
#   b.追加数据
ls -il >> file_list.txt
#   c.重定向输入：command < inputfile
wc < file_list.txt
#   d.内联输入重定向: command << flag ... data .. flag

# 8.管道: command | command
#   a.管道两端的命令是同时执行的
# ls -il | sort -r
# apt list --installed | sort | more

# 9.shell 脚本中进行数学运算的两种途径: expr 和 [].
#    a.expr 命令能识别少数的数字和字符串操作符
expr 1+3
# 使用转义字符(反斜杠)标识容易被错误解释的字符.
expr 2 \* 3
# 使用命令替换来获取 expr 命令的输出.
tt=$(expr 12 / 3)
echo $tt
#    b.用美元符号和方括号将数学表达式围起来：$[ operation ]
result=$[49%5]
echo result=$result

# Note: bash shell 数学运算符支支持整数运算, zsh 提供了完整的浮点算数操作

# 10.内建的浮点解决方案: bc
#    a.浮点运算是由内建变量 scale 控制的, 将这个值设置为计算结果中期望保留的小数位数.
#    b.输入 bc 进入计算器; 要退出 bc 计算器, 输入 quit.
#    c.在脚本中运行 bc 命令, 并将结果赋值给变量: variable=$(echo "options; expression" | bc)
bct=$(echo "scale=2; 3.14 / 2" | bc)
echo the answer is $bct

# 使用内联输入重定向
# variable=$(bc << EOF
# options
# statements
# expressions
# EOF
#)

vbcr=$(bc << EOF
scale=3
1.234/0.789
EOF
)
echo $vbcr

# 11 退出脚本
#    a.Linux 提供了专门的变量 $? 来保存上个已命令执行命令的退出状态码.
#    b.成功的退出码是 0.
#    c.使用 exit 命令为脚本结束时指定退出状态码: exit 0 (0,255).

date
echo $?

exit 0
```