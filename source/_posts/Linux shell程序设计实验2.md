---
title: Linux shell程序设计实验2
date: 2018-04-22 00:08:35
tags:
    - "Linux实验"

categories:
    - "Linux"
    
---

编写一个shell过程完成如下功能（必须在脚本中使用函数）

<!--more-->

1.程序接收3个参数:$1/$2和$3,合并两个文件$1/$2为$3,并显示，三个文件均为文本文件。
2.如果文件$3不存在，那么先报告缺少$3，然后将合并后的内容输出到mydoc.txt。如果有$3，就合并到$3。
3.如果文件$2或文件$3不存在，那么先报告缺少$2/$3，只显示$1的内容。

4.如果文件$1不存在，则提示缺少$1，要求重新运行程序。

```bash
#!/bin/sh
 
function f1(){
    echo "missing \$2 and \$3"
    echo "output \$1:"
    cat $1
}
 
function f2(){
    echo "missing \$3, output mydoc.txt:"
    cat $1 $2 >mydoc.txt
    cat mydoc.txt
}
 
function error(){
    echo "missing \$1 \$2 \$3, error!"
}
 
if [ $# -eq 0 ]; then
    error
elif [ $# -eq 1 ]; then
    f1 $1
elif [ $# -eq 2 ]; then
    f2 $1 $2
elif [ $# -eq 3 ]; then
    cat $1 $2 >$3
    echo "output \$3:"    
    cat $3    
fi
 
exit 0
```