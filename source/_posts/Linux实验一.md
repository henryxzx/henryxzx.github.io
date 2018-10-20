---
title: Linux实验一
date: 2018-04-12 00:08:35
tags:
	- "Linux实验"

categories:
	- "Linux"
	
---
#### Linux实验一
<!--more-->
#### 环境：win10 + redhat 5
####  1、root帐号登录，查看/tmp目录，如果/tmp目录下没有子目录myshare，则建立该目录。
```bash
ls /tmp | grep myshare
```
```bash
mkdir -p /tmp/myshare
```
#### 2、创建帐号testuser。
```bash
useradd testuser
```
```bash
passwd testuser
```
#### 3、把myshare目录及其目录下的所有文件和子目录的拥有者该为testuser，工作组改为users。
```bash
chown -R testuser:users /tmp/myshare
```
#### 4、切换至testuser帐号。进入/tmp/myshare目录，采用vim编辑器编写以下程序,程序名称为hello.sh：
> \#!/bin/bash
echo "app start"
echo -e
func (){
  echo "hello world!"
}
func
echo -e
echo "app end"
```bash
su testuser
```
```bash
vim /tmp/myshare/hello.sh
```
#### 5、保存hello.sh后，给予hello.sh拥有者可读、可写和可执行的权限，同组可读可执行，其他人可执行权限。
```bash
chmod 751 hello.sh
```
#### 6、输入./hello.sh，观察程序输出的效果。
```bash
./hello.sh
```
#### 7、进入testuser的用户主目录，在这个目录下创建hello.sh的软链接，同时拷贝hello.sh到该目录下并改名为hello.sh.bak，要求copy时保留文件属性值。
```bash
ln -s /tmp/myshare/hello.sh 
```
```bash
cp -p /tmp/myshare/hello.sh /home/testuser/hello.sh.bak
```
#### 8、退出testuser帐号，回到root帐号，从/开始查找后缀名为.conf的所有文件，把输出结果重定向到testuser帐号的主目录下的output.txt文件。
```bash
exit
```
```bash
find / -type f -name ".conf" >> /home/testuser/output.txt
```

#### 9、在上一步操作的.conf文件中找出文件容量最大的和最小那个，并把这两个文件的容量大小输出到output.txt文件中。>>
```bash
find / -type f -name ".conf" -ls | awk '{print $7}'|sort -n > /home/testuser/tmp.out
```
```bash
head -1 /home/testuser/tmp.out >> /home/testuser/output.txt
```
```bash
tail -1 /home/testuser/tmp.out >> /home/testuser/output.txt
```
#### 10、统计出系统中有多少个用户帐号，把数量输出到output.txt文件中。
```bash
cat /etc/passwd | wc -l >> /home/testuser/output.txt
```
#### 11、把output.txt文件转换为windows记事本可正规打开的格式。
```bash
unix2doc output.txt
```
#### 12、tar打包压缩testuser帐号主目录下的所有文件。
```bash
tar zcvf testuser.tar.gz /home/testuser
```
#### 13、把tar文件另存在window系统下。
U盘或者ftp

#### 14、执行userdel -r testuser，执行rm -fr myshare
