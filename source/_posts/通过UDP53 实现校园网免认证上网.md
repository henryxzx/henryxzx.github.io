---
title: 通过UDP53实现校园网免认证上网
date: 2018-09-17 20:02:47
tags:
	- "校园网"
	- "Linux"
	- "服务器"

categories:
	- "计算机网络"	

---

### 一、 前言

阿里云的学生服务器买了有一段时间了，本来想拿来部署一个网站的，但是因为国内坑爹的备案制度，导致我又买了一个国外的服务器来部署（国外的学生服务器比国内便宜多了），所以目前阿里云这个服务器就空闲了下来，为了不浪费这个服务器资源，于是就去知乎找了一些有关于学生服务器的用处，就发现有人提了这个用处，而且我们学校 12 点断网后连手机信号也会断，只能通过这种方法来上网，顿时觉得我们学校太 tm 坑了，但这个方法并不适用于所有校园网认证，如需**锐捷认证**的**有线连接**，正巧我们学校的还有无线 web 认证，<S> 可能是学校自己弄的 </S>，总之这个认证系统就很水，这个方法用了一段时间了并没有出任何问题，至于以后就不知道了，<S> 也许网管某天会发现 53 端口走了一堆 malformed packet 然后警告我一波。</S>
<!--more-->
### 二、环境搭建

你需要：

- 一台服务器

  > 阿里云，腾讯云的学生机都可以，甚至 DigitalOcean，Vultr 的国外节点也可以，但由于国外的 vps 使用这个方法时会受到 GFW 的干扰，所以很不稳定，不推荐使用国外节点，但是国外服务器不限带宽还是很爽的。

- 一台能连接网络的设备

  > 经过试验，pc 端 windows，macbook，linux 都可以，移动端 ios，Android 也没问题，但是管理工具移动端没有。

#### 测试网关

一般来说如果是使用的无线网络，无线网关会放行 UDP67/68，也就是 DHCP 端口来获取 ip，和 UDP53（DNS 端口）来获取 DNS，使用如下工具来测试 53 端口是否通行：

[项目地址 ](https://github.com/BennyThink/UDP53-Filter-Type)

下载下来之后解压并运行 dist 内的 UDP53.exe。

如果出现的是 congratulations，那就代表 53 端口完全放行，可以使用下面的方法，如果不是的话，就只能通过 DNS 通道来搭建环境，下面不做介绍。

#### 购买服务器

> 下面都以阿里云服务器为例子

如果你没有服务器，先在阿里云服务器上购买一个学生机，这是我的[推广链接 ](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=hcbpp5n6)，注册一个阿里云账号登陆并找到云翼计划，购买一个学生机，系统选择 centOS7，设置登陆密码，然后使用类似 putty 或 secureCRT 的 ssh 工具，远程连接到服务器上。

#### 服务器环境搭建

- ##### 安装 softether VPN server

使用 ssh 工具连接到 vps 上，先更新 yum 源。

![](https://i.loli.net/2018/09/17/5b9fbc122385f.png)

```bash
yum update
```

输入 y 确认，然后安装依赖库

```bash
yum -y install gcc zlib-devel openssl-devel readline-devel ncurses-devel
```

> 如果是 ubuntu 的话，使用的是 apt-get 来作为包管理器。

在当前目录 /root 下载 softether VPN server 端

```bash
wget http://oks2t4o68.bkt.clouddn.com/softether-vpnserver-v4.22-9634-beta-2016.11.27-linux-x64-64bit.tar.gz
```

解压缩

```bash
tar -zvxf softether-vpnserver-v4.22-9634-beta-2016.11.27-linux-x64-64bit.tar.gz
```

进入目录并安装

```
cd vpnserver
make
```

选项全部选 1

安装完后再当前目录继续输入

```bash
./vpnserver start
```

开启服务

- ##### 设置 softether VPN server 服务端

```bash
./vpncmd
```

依次输入 1，回车，回车，再输入

```bash
ServerPasswordSet
```

设置服务端密码，到此服务器端已经部署完毕

#### 管理 softether VPN server

下面回到 pc 端，以 win10 为例，先从官网下载最新版 https://www.softether-download.com/cn.aspx?product=softether（需翻墙），然后安装管理工具，打开 slmgr.exe

![](https://i.loli.net/2018/09/17/5b9fbb93ed112.png)

点击新设置

![](https://i.loli.net/2018/09/17/5b9fbd1161937.jpg)

主机名填写你的**vps 服务器 ip**，输入你刚才设置的密码

将弹出的设置关掉，进入主界面

![](https://i.loli.net/2018/09/17/5b9fbe50be650.jpg)

点击管理虚拟 HUB

![](https://i.loli.net/2018/09/17/5b9fbe76ef3e0.png)

再点击管理用户

![](https://i.loli.net/2018/09/17/5b9fbed0793d1.png)

点击新建

![](https://i.loli.net/2018/09/17/5b9fbefc6f997.png)

如图设置，用户名和密码自己填

回到管理虚拟 HUB 界面，点击虚拟 NAT 的 DHCP 服务器，启用 SecureNAT

![](https://i.loli.net/2018/09/17/5b9fbf1c907bf.png)

将端口改为 53，并点击为 OpenVPN Client 生成配置样本文件，将其保存到一个地方。

![](https://i.loli.net/2018/09/17/5b9fbf956a7d5.png)

#### 配置 OpenVPN

到官网下载 OpenVPN，https://openvpn.net/index.php/open-source/downloads.html（需翻墙，跟 VPN 有关的东西在国内墙得厉害），下载 windows 版并安装，然后打开，点击 Import file 将刚才的样本文件中的以 l3 结尾的文件导入到 OpenVPN 的配置目录中

![](https://i.loli.net/2018/09/17/5b9fc2a1346f4.png)

![](https://i.loli.net/2018/09/17/5b9fc2cd88e75.png)

导入完之后点击 connect

![](https://i.loli.net/2018/09/17/5b9fc30bdf586.png)

将你刚才设置的虚拟 HUB 中的用户名和密码输入，勾选 Save password，如果出现这个提示，就表示设置成功了

![](https://i.loli.net/2018/09/17/5b9fc3a8cce0b.png)

这时你可以注销认证，再试试连接，发现依然可以联网。

### 三、实际使用

阿里云学生服务器自带的带宽是 1M，网速很不理想，最快也就 150K/s 的速度，但是阿里云可以通过暂时提升带宽来提升网速，总的来说就是服务器的带宽决定了我们的网速，下面是 10M 的速度作为参考。。。

![](https://i.loli.net/2018/09/17/5b9fc5f489c7c.png)

如果没有特殊的需求的话，看看普通网页的话 1M 的速度够了，主要是可以解决 12 点后完全没网的尴尬。

#### 参考文章

- [UDP 53 免费上网、DNS 隧道经验谈 ](https://www.bennythink.com/udp53.html)
- [SoftEther VPN Server 安装笔记 + 福利 ](https://www.bennythink.com/softether-vpnserver.html)
- [Centos7.4 使用 SoftEther 搭建 VPN](https://blog.csdn.net/qq_39591494/article/details/78625394)