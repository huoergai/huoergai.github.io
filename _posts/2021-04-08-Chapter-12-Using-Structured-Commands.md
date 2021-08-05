---
title: Chapter-12 Using Structured Commands
date: 2021-04-08 20:34 
tags:
    - Linux Command Line and Shell Scripting Bible, 3E
---

```shell
# chapter 12 structured command

# 1.if-then
# if command
# then
#     commands
# fi
if pwd
then
    echo "wa la ..."
fi

# 1.1 
# if command; then
# commands
# fi
if date; then
    echo "first command"
    echo "second command"
fi

# 1.2 if-then-else
# if command
# then
#     commands
# else
#     commands
# fi
if !data;then
    echo "got date"
    echo "let's go next"
else
    echo "failed to get date"
    echo "it's done"
fi

# 1.3 nested if
if date; then
    echo "step in 0"
    if pwd; then
        echo "step in 1"
	echo "step 1 done"
    fi
fi

# 1.3.1 elif
# if command1
# then
#     commands
# elif command2
# then
#     commands
# fi
if data;then
	echo "condition 1"
elif pwd;then
	echo "condition 2"
fi

# 1.4 test command
# test condition
test 4>2

# if test condition; then
#     commands
# elif test condition; then
#     commands
# fi
t1=12
t2=10
if test $t1>t2; then
    echo "more"
elif test $t1<$t2; then
    echo "less"
fi
 
# Note: pay attention to the empty space at both side of the "condition" 
# if [ condition ]; then
#     commands
# fi
if [ $t1>$t2 ]; then
    echo "t1 > t2"
fi

# test can operate three kinds of condition:
# a.numeric comparison
# b.string comparison
# c.file comparison

# 1.4.1 numeric comparison

# n1 -eq n2	n1 =  n2
# n1 -ge n2	n1 >= n2
# n1 -gt n2	n1 >  n2
# n1 -le n2	n1 <= n2
# n1 -lt n2	n1 <  n2
# n1 -ne n2	n1 /= n2

if [ 10 -eq 11 ]; then
    echo "10 = 11"
elif echo ""; then
    echo "10 /=11"
fi

# 1.4.2 string comparison

# str1  =   str2
# str1  !=  str2
# str1  <   str2
# str1  >   str2
# -n str1   check wether len(str1)!=0
# -z str1   check wether len(str1)=0

if [ -n "hello" ]; then
    echo "len("hello") != 0"
fi

# Note:
#      a. must transfer > and <, or shell will regard it as a redirector.
#      b. >,< and sort command usr different order.

if ["hello" \> "hi"]; then
    echo "hello > hi"
elif echo ""; then
    echo "hello <= hi"
fi

# 1.4.3 file comparison

# -d file		file exist and is a director
# -e file		file exist
# -f file		file exist and is a file
# -r file		file exist and readable
# -s file		file exist and not empty
# -w file		file exist and writable
# -x file 		file exist and excutable
# -o file		file exist and belong to curent user
# -G file		file exist and default group is the same with current user
# file1 -nt file2	file1 newer than file2
# file1 -ot file2	file1 older than file2

touch testfile1.txt
touch testfile2.txt

if [ -e testfile1.txt ]; then
    echo "exist"
fi

# 1.5 compound condition test

# a.[ condition1 ] && [ condition2 ]
# b.[ condition1 ] || [ condition2 ]

if [ "a" > "b" ] && [ "b" > "c" ]; then
    echo "a > b > c"
fi


# 1.6 advance feature of if-then
# - 用于数学表达式的双括号
# - 用于高级字符串处理功能的双方括号

# 1.6.1 double parentheses
# 双括号命令允许使用高级数学表达式进行比较
# (( expression ))

# val++		
# val--
# ++val
# --val
# !		逻辑求反
# ~		位求反
# **		幂运算
# <<
# >>
# &
# |
# &&
# ||

if (( 2 ** 3 > 7 )); then
    echo "2*2*2 > 7"
fi

# 1.6.2 double square brackets
# 双方括号中的 expression 使用 test 命令的标准字符串比较，同时提供模式匹配
# [[ expression ]]

if [[ $USER == x* ]]; then
    echo "has x in user"
fi
echo "user = $USER"

# 1.7 case command
# case variable in
# pattern1 | pattern2) command;;
# pattern3) command;;
# *) default commands;;
# esac

a=wen
case $a in
mon | tue | wen | thu | fri) echo "hello workday!";;
sat | sun) echo "hello weekend!";;
*) echo "out of week";;
esac

```