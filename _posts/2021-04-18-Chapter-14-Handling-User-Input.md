---
title: Chapter-14 Handling User Input
date: 2021-04-18 22:33
tags:
    - Linux Command Line and Shell Scripting Bible, 3E
---

# Chapter-14 Handling User Input

```shell
#!/bin/bash

# chapter-14 Handling User Input

# 1. 命令行参数
# 使用命令行参数是向 shell 脚本传递参数最基本的方法，以在脚本运行时向命令行添加数据。
# $ ./xxx_.sh param1 param2

# 1.1 read parameters
# bash shell 默认会将一些成为『位置参数』的特殊变量分配给输入到命令行的参数。
# 『位置参数』变量是标准的数字：$0 是程序名 $1是第一个 $2是第二个参数  ... $9 第九个参数
# 当参数个数大于9后，必须在变量数字周围加上花括号，${10}。
echo "app name = $0"

if [ -n "$1" ]; then
    echo "param1=$1"
else
    echo "param1 is empty!"
fi
# note: a.$1 要用双引号转为字符串, -n 比较才有效；
#       b.使用命令行参数前一定要检查;

# 1.2 使用 basename 命令返回不包含路径的脚本名。
clean_name=$(basename $0)
echo "pure name = $clean_name"

# 2. 特殊参数变量

# 2.1 参数统计
# a. $# 命令行参数个数
# b. ${!#} 最后一个命令行参数
echo "we got $# command parameters"
echo "the last one is ${!#}"
# note: 当没有任何命令行参数时，$# 值0，但 ${!#} 返回脚本名。

# 2.2 抓取所有的数据
# a. $* 会将所有命令行参数作为一个整体保存。
# b. $@ 会将命令行提供的所有参数当作同一字符串中的多个独立的单词，可通过 for 遍历所有的参数值。
echo "all cmd parameters in one:『$*』"
for item in $@; do
    echo "param=$item"
done

# 3. 移动变量
# shift 命令会根据命令行参数的相对位置来对其进行移动，默认左移一个位置。
# 通过给 shift 添加参数可指明要移动的位数： shift n
# 是在不知道有命令行参数个数时进行遍历的另一种方法。

# pos=1
# while [ -n "$1" ]; do
#     echo "number #$pos = $1"
#     pos=$[ $pos + 1 ]
#     shift 
# done
# Note: 当参数被移出后，即丢失并不可恢复。

# 4.处理选项
# 『选项』是跟在单破折线后面的单个字母。
# 4.1 处理简单选项

# while [ -n $1 ]; do
#     case "$1" in
#       -a) echo "option -a";;
#       -b) echo "option -b";;
#       -c) echo "option -c";;
#        *) echo "$1 is not an option"
#           break
#     esac
#     shift
# done

# 4.2 分离参数和选项
# 在脚本中同时使用选项和参数时，使用特殊字符『单破折线』将二者区分开来。
# 同时使用『双破折线』表明选项列表的结束。

# read options
# while [ -n "$1" ]; do
#     case "$1" in
#       -a) echo "option -a";;
#       -b) echo "option -b";;
#       -c) echo "option -c";;
#       -d) echo "option -d";;
#       --) shift
#           break;;
#        *) echo "$1 is not an option";;
#     esac
#     shift
# done
# 
# # read cmd parameters right after options
# cnt=1
# for param in $@; do
#     echo "param #$cnt $param"
#     cnt=$[ $cnt + 1 ]
# done
# 
# ./chapter-14.sh -a -b -- t1 t2 t3

# 4.3 处理带值的选项
# 实例： ./xxx.sh -a v1 -b -c -d v2
# 遇到带值的选项时，使用 $2 取出对应的值，再使用 shift 移动一位跳过值进入下一选项。
# while [ -n $1 ]; do
#     case "$1" in
#       -a) echo "option -a";;
#       -b) echo "option -b"
#           echo "param of -b is $2"
#           shift;;
#       -c) echo "option -c";;
#       -d) echo "option -d";;
#       --) shift
#           break;;
#        *) echo "invalide option";;
#     esac
#     shift
# done
# ./ xxx.sh -a -b hello -c -d -- t1 t2

# 5. 使用 getopt 命令
# 『getopt』 命令可以接收任意形式的命令行选项和参数，并将其转换成适当的格式
# getopt optstring parameters
# a.在 optstring 中列出脚本中要用到的所有选项。
# b.并在需要"参数值"的选项字母后面加一个冒号。
# c.在命令后加 -q 选项可忽略错误信息。
# getopt ab:cd -a -b hello -cd t1 t2
# ==> -a -b hello -c -d -- t1 t2

# 5.1 在脚本中使用 getopt
# # set --$(getopt -q ab:cd "$@")
# 首先获取所有命令行参数传递给 getopt 进行格式化；再使用 set 命令用格式化后的命令行参数替换旧的命令行参数。

# set --$(getopt -q a:bc:d "$@")
# echo "$@"
# 
# while [ -n "$1" ]; do
#     case "$1" in 
#       -a) echo "option -a";;
#       -b) echo "option -b"
#           echo "parameter of -b is $2"
#           shift;;
#       -c) echo "option -c"
#           echo "parameter of -c is $2"
#           shift;;
#       --) shift
#           break;;
#        *) echo ""
#     esac
#     shift
# done
# for pp in "$@"; do
#     echo "pp=$pp"
# done
# note: getopt 命令不擅长处理带空格和引号的参数值

# 6.使用更高级的 getopts
# getopts optstring variable
# a.在 optstring 中，选项字母要带参数值，就在后面加一个冒号。
# b.在 optstring 之前加一个冒号可去掉错误信息。
# c.当前选项的参数值保存在 OPTARG 环境变量中。
# d.正在处理的参数位置保存在 OPTIND 环境变量中。 

while getopts :ab:cd opt; do
    case "$opt" in
        a) echo "option -a";;
        b) echo "option -b"
           echo "parameter of option -b is $OPTARG";;
        c) echo "option -c";;
        *) echo "invalide option: $opt";;
    esac
done

echo "opt=$@"
shift $[ $OPTIND - 1 ]
cnt=1
for pp in "$@"; do
    echo "p#$cnt=$pp"
    cnt=$[ $cnt +1 ]
done

# 7.获取用户输入
# 7.1基本的读取
# read 命令会从标准输入(键盘)或者另一个文件描述符中接收输入，并将数据放入一个变量。
# read 命令会将提示符后的输入依次分配给变量，如果变量数不够，则把剩余数据全部分配给最后一个变量。
# 如果未指定变量，read 命令则将数据放入环境变量 REPLY。
# -p 选项，可直接在 read 命令后跟上提示符。
echo -n "enter your name: "
read name1 name2
echo "hello $name1 and $name2"
echo -n "enter you score: "
read 
echo "congratulation you got $REPLY!"

#7.2 超时
# a.read 命令加上 -t 选项可指定等待输入的秒数，超时后 read 命令会返回非 0 状态码。
# b.read 命令加上 -nx 选项，x 代表要统计输入的字符数，当输入达到预设的 x 个字符时，read 命令会自动返回。
# c.-s 选项可避免输入数据显示在显示器上。
# d.
if read -t 10 -p "how long can you wait: " wt; then
    echo "it's $wt seconds"
else 
    echo
    echo "you're out of time"
fi

read -n10 -p "read another 10 words:" tw
echo
echo "you typed『$tw』"

# 7.3 读取文件
# 每次调用 read 命令都从文件读取一行文本；当没有内容时，read 命令会以非零状态码退出。

cnt=1
cat ./tmp/test.txt | while read line; do
    echo "Line #$cnt  -$line"
    cnt=$[ $cnt + 1 ]
done
echo "Finished reading the file."

```