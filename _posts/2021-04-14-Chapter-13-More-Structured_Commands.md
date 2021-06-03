---
title: Chapter-13 More Structured Commands
date: 2021-04-14 21:52
tags: 
    - Linux Command Line and Shell Scripting Bible, 3E
---

# Chapter-13 More Structured Commands

```shell
#!/bin/bash
# chapter-13 more structural command

# for loop
# until and while 
# loop
# output of redirect loop

# 1. for command
# for var in list
# do
#     commands
# done

# for var in list; do
#     commands
# done

for today in mon tue wen thu fri sat sun; do
    echo "today is $today"
done
echo "the last day is $today"

# 1.1 read complicate value from list:
# a.使用转义字符(反斜线)转义单引号
# b.使用双引号包裹包含单引号的值

# I don't know if this'ill work
for w in I don\'t know if "this'll" work; do
    echo $w
done

# note: for 循环使用空格进行分割；如果值中包含空格，需将该值用双引号圈起来。
for city in Boston "New York" LA; do
   echo "city=$city"
done

# 1.2 read list from variable
citys="Toyoko Shanghai London"
for city in $citys; do
    echo $city
done 

# 1.3 read list form command
file="states"
for city in $(cat $file); do
    echo "state=$city"
done

# 1.4 修改字段分隔符(IFS)
# a.IFS 环境变量定义了 bash shell 用作字段分割的一系列字符；
# b.默认情况下 bash shell 会将“空格/制表符/换行符”当做字段分隔符；
# 可通过临时修改 IFS 值来解决含有已被当做字段分隔符的数据；

ifs_tmp=$IFS
IFS=$'\n'
for city in $(cat $file); do
    echo "new state=$city"
done
IFS=$ifs_tmp

# 1.5 使用通配符读取目录
# 在文件名或路径名中使用通配符，以使用 for 命令自动遍历目录。
road=$pwd
echo "road=$road"
for file in $road/tmp/*; do
    if [ -d $file ]; then
        echo "$file is a directory"
    elif [ -f $file ]; then
        echo "$file is a file"
    else
        echo "$file doesn't exist"
    fi
done

# 2.C-style for command
# for (i = 0; i < 10; i++)
# {
#     printf("The next number is %d\n", i);
# }

# 2.1 C-style for loop in bash
# for (( vaiable assignment ; condition ; iteration process ))
for (( i = 0; i < 10; i++ )); do
    echo "i=$i"
done

#2.2 multiple variables
for (( a = 0, b = 9; a < 10; a++, b-- )); do
    echo "$a - $b"
done

# 3. while command
# while test command; do
#     other commands
# done
w=10
while [ $w -gt 0 ]; do
    echo "w=$w"
    w=$[ $w - 1 ]
done

# 3.1 apply multiple test commands
multi_w=10
while echo "test $multi_w"
      [ $multi_w -ge 0 ]
do
    echo "$multi_w >= 0"
    multi_w=$[ $multi_w - 1 ]
done

# 4.until command
# until test command; do
#     other commands
# done
u_val=10
until [ $u_val -eq 0 ]; do
    echo "u_val = $u_val"
    u_val=$[ $u_val - 2 ]
done

# 5. nested loop
for (( a = 0; a < 10; a++ )); do
    echo "a = $a"
    for (( b = 10; b > 0; b-- )); do
        echo "b = $b"
    done
done

# 6. deal file data with loop
# a. use nested loop
# b. change IFS environment param

tmp_ifs=$IFS
IFS=$'\n'
for entry in $(cat /etc/passwd); do
    echo "values in $entry -"
    IFS=:
    for item in $entry; do
        echo "  $item"
    done
done
IFS=$tmp_ifs

# 7. control loop
# a. break
# b. continue

# 7.1 breadk command
# 7.1.1 break out of single loop
#       when deal with nested loop,"break" will break out of the innermost loop.
for i in 1 2 3 4 5 6 7 8 9 10; do
    if [ $i -eq 5 ]; then
        echo "break 5"
        break
    fi
    echo "i = $i"
done
echo "The for loop is completed."

# 7.1.2 break out of the outer loop
# a. break n
for (( i = 0; i < 10; i++)); do
    echo "outer loop: $i"
    for (( j = 0; j < 10; j++)); do
        if [ $j -gt 4 ]; then
            echo "break from outer loop"
            break 2
        fi
        echo "inner loop: $j"
    done
done

# 7.2 continue command
for (( i = 0; i < 20; i++ )); do
    if [ $i -gt 5 ] && [ $i -lt 15 ]; then
        echo "continue"
        continue
    fi
    echo "i=$i"
done

# 8. deal output of loop
# 在 shell 脚本中，可以对循环的输出使用管道和重定向; 通过在 done 命令后添加命令实现.
for (( i = 0; i < 10; i++ )); do
    echo "item $i"
done > ./tmp/loop_output.txt

# 9. examples
# 循环是迭代系统数据的常用方法，包括目录中的文件和文件中的数据。
# 9.1 look for executable files

ifs_tmp=$IFS
IFS=:
for td in $PATH; do
    echo "folder: $td"
    for tf in $td; do
        if [ -x $tf ]; then
            echo "  $tf"
        fi
    done
done
IFS=$ifs_tmp

# 9.2 create accounts
ifs_tmp=$IFS
input="./tmp/users.csv"
while IFS=',' read -r user_id user_name; do
    echo "id=$user_id name=$user_name"
done < "$input"
# Note: 最后一行数据录入完成后要跳入到下一行再保存，否则最后一行有效数据会读取失败。
IFS=$ifs_tmp
```