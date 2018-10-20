---
title: Linux shell程序设计实验1
date: 2018-04-21 00:08:35
tags:
    - "Linux实验"

categories:
    - "Linux"
    
---

实验1-1

设计如下一个菜单驱动程序

Use one of the following options:
P:To display current directory
S:To display the name of running file
D:To display today's date and present time (要求显示为：2017-04-26 05:45:12) 
L:To see the list of files in your present working directory
W:To see who is logged in
Q:To quit this program
Enter your option and hit:
菜单程序将根据用户输入的选择项给出相应信息。要求对用户的输入忽略大小写，对于无效选项的输入给出相应提示。要求使用case语句实现以上功能，输入相应的字母后应该执行相应的命令完成每项功能，如输入P或p，就执行pwd命令。

<!--more-->
实验1-2
编写一段bash shell程序，
根据键盘输入的学生成绩，显示相应的成绩登等级，
其中
60分以下为"Failed!",
60～69分为"Passed!",
70~79分为"Medium!",
80~89分为"Good!"，
90～100为"Excellent!"。
如果输入超过100的分数，则显示错误分数提示。

如果输入负数，则退出程序，否则一直提示用户输入成绩

1-1

```bash
#!/bin/bash
clear
echo "Use one of the following options:
P:To display current directory
S:To display the name of running file
D:To display today's date and present time
L:To see the list of files in your present working directory
W:To see who is logged in
Q:To quit this program
Enter your option and hit:"
read ans
case "$ans" in
P|p)
    pwd
    ;;
S|s)
    ls
    ;;
D|d)
    date -d '2017-04-26 05:45:12' '+%Y-%m-%d %H:%M:%S' 
    ;;
L|l)
    ls -l
    ;;
W|w)
    who
    ;;
Q|q)
    exit 1
    ;;
*)
    exit 1
esac
exit 0
```

1-2

```bash
#!/bin/bash
while true
echo "Please input your score: (0 - 100)"
do
read score
if [ $score -lt 0 ];
then
    break
elif [ $score -lt 60 ];
then
    echo "Failed!"
elif [ $score -lt 70 ];
then
    echo "Passed!"
elif [ $score -lt 80 ];
then
    echo "Medium!"
elif [ $score -lt 90 ];
then
    echo "Good!"
elif [ $score -le 100 ];
then
    echo "Excellent!"
else
    echo "the score must be smaller than 100"
fi
done
exit 0
```