---
title: Linux课堂练习1
date: 2018-04-16 00:08:35
tags:
	- "Linux实验"

categories:
	- "Linux"
	
---

环境：win10 + redhat 5

<!--more-->

##### 1、创建组testgroup。

```bash
groupadd testgroup
```

##### 2、创建用户a2012，先采用默认设置创建，然后使该用户加入testgroup组。

```bash
useradd a2012
usermod -g testgroup
```

##### 3、创建用户a2013，其用户主目录为/tmp/a2013，其主组为testgroup，附加组为users。

```bash
useradd a2013
usermod -g testgroup
usermod -G users
```

##### 4、用id命令显示a2012和a2013用户信息，并且把这些信息记录到日志文件/tmp/test.log中。

```bash
id a2012 > /tmp/test.log
id a2013 >> /tmp/test.log
```

##### 5、参考书本98-99页crontab命令内容，使用root执行crontab -e，编写时程表，完成每隔5分钟把当前时间追加进/tmp/test.log中。

```bash
crontab -e
*/5 * * * * date >> /tmp/test.log
```

##### 6、执行crontab -l，把输出内容追加进/tmp/test.log。

```bash
crontab -l >> /tmp/test.log
```

##### 7、待完成2次时间记录追加后，执行crontab -r删除当前的时程表。

```bash
crontab -r
```

##### 8、把/tmp/test.log拷贝到windows中（注意文本格式的转换），采用记事本打开，看是否看到完整内容。

```bash
unix2dos test.log   #将unix中的换行符转换为dos环境下能识别的类型
```

最后将文件复制到win环境下得到test.log