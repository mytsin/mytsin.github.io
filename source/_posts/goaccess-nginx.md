---
title: Goaccess - Nginx日志分析/图表生成工具
copyright: true
tags:
  - Goaccess
  - Nginx
categories: 教程
abbrlink: 32034
date: 2018-10-28 12:47:00
---
![Goaccess官网界面](https://raw.githubusercontent.com/mytsin/Image/master/Blog/10-27/Goaccess_01.png)

`GoAccess`**是一款开源的且具有交互视图界面的实时`Web`日志分析工具**，可分析`Apache/Nginx`等WEB日志，同时还支持生成`HTML`、`JSON`、`CSV`等数据报告。`GoAccess`既可以在终端中展示结果，也可以生成`HTML`报表在浏览器中查看，且生成的报表足够酷炫，这也是我为什么一直在折腾，想要实现的原因。我花了几天时间，查阅了大量的资料，终于实现了在浏览器中查看HTML报表
<!-- more-->
先看一下最终效果：[mytsin.com/analysis.html已失效](mytsin.com/analysis.html)

![Goaccess](https://raw.githubusercontent.com/mytsin/Image/master/Blog/10-27/Goaccess_02.png)


# 准备工作

系统：`CentOS 6.10`

web服务：`Nginx`

面板：[宝塔面板](https://www.bt.cn/download/linux.html)

日志文件：`access.log`

工具：`Goaccess-1.2`

中文网站：[goaccess.cc](goaccess.cc)

官方网站：[goaccess.io](goaccess.io)

- 安装前请先备份好相关数据（例如网站数据库等）
- 确定日志文件的所在目录
- 以下日志文件、网站目录，根据自己的实际情况进行修改
- 演示的日志文件及目录为`/home/wwwlogs/access.log`
- 演示的网站目录为`/www/wwwroot/mytsin.com/`

# 安装Goaccess

虽然可以通过`apt-get`或`yum+epel`进行安装，但是版本不同且功能上有些差异，建议下载源码进行编译安装。

## 首先安装依赖包

**CentOS/Fedora/RHEL：**`yum install ncurses-devel geoip-devel`

**Ubuntu/Debian：**`apt-get install libncursesw5-dev libgeoip-dev`

```
#安装依赖
$ yum -y install libmaxminddb-devel #下载源码
$ wget https://tar.goaccess.io/goaccess-1.2.tar.gz

#解压
$ tar -xzvf goaccess-1.2.tar.gz

#进入目录
$ cd goaccess-1.2/

#编译安装
$ ./configure --enable-utf8 --enable-geoip=mmdb --with-openssl --with-libmaxminddb-devel
$ make && make install
```

如有需要，也可以安装`GeoIP`
```
$ cd /usr/local/src
$ wget http://geolite.maxmind.com/downloadgeoip/api/c/GeoIP-1.4.6.tar.gz
$ wget http://geolite.maxmind.com/downloadgeoip/database/GeoLiteCountry/GeoIP.dat.gz
$ tar xzvf GeoIP-1.4.6.tar.gz
$ cd GeoIP-1.4.6
$ ./configure && make && make install
$ cd ..
$ mv GeoIP.dat.gz /usr/local/share/GeoIP/
```

## 使用方法
### 终端下查看

分析日志：`goaccess -f /home/wwwlogs/access.log --log-format=COMBINED`

**常用参数说明：**

-f 指定要分析的日志`/path/to/log`

--log-format 日志的格式，`LNMP`默认格式为：`COMBINED`

-a 在Host模块是否启用点开`IP`显示`user-agents`

终端下效果如下图：
![终端显示结果](https://raw.githubusercontent.com/mytsin/Image/master/Blog/10-27/Goaccess_03.jpg)

**操作快捷键**

q 退出当前小窗口、模块视图或退出`Goaccess`

o 打开当前激活模块的详细视图，当前激活模块会以黄色显示

0-9 数字0-9可以控制切换各个模块

c 改变当前配色

/ 搜索

F1 帮助

F5 窗口重新绘图

2、生成图表网页
```
$ goaccess -f /home/wwwlogs/access.log --log-format=COMBINED -a > www/wwwroot/mytsin.com/analysis.html
```
`access.log`：日志文件路径（请填写绝对路径）

`/mytsin.com/`：为您站点根目录，根据自身情况修改

运行后，就自动生成一个酷炫的网页图表，可以直接在浏览器里打开查看

### 定时生成HTML报告到站点目录

每次都去手动生成报告太麻烦啦，有没有更简便的方法？写个简单的shell脚本自动生成就搞定啦~

假设文件要放在`/root/`目录下
```
$ cd /root
$ vim goaccess.sh
```

将下面的内容另存为`goaccess.sh`
```
#!/bin/bash

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin

export PATH

goaccess /home/wwwlogs/access.log -a -o www/wwwroot/mytsin.com/analysis.html --log-format=COMBINED
```
`/access.log/`：日志文件路径（请自行替换，填写绝对路径）

`/mytsin.com/`：为您站点根目录，根据自身情况修改

别忘记加上可执行权限：`chmod u+x goaccess.sh`，再使用`crontab`每小时生成一次HTML报告
```
#添加计划任务
$ crontab -e

#添加以下内容，/root/goaccess.sh为上面脚本的绝对路径
$ */15 * * * * /root/goaccess.sh > /dev/null

#重新加载cron配置和重启cron服务
$ service crond reload
$ service crond restart
```
大功告成，这样`Goaccess`每隔15分钟就会为我们生成一次HTML日志报告，访问网站[https://mytsin.com/analysis.html已失效](https://mytsin.com/analysis.html)就可以看到直观的HTML报告啦

# 参考资料：

1. [Nginx日志分析/图表生成工具 - goaccess](https://www.vpser.net/manage/goaccess.html)

2. [CentOS安装GoAccess快速方便的分析网站日志](https://www.centos.bz/2018/07/centos%E5%AE%89%E8%A3%85goaccess%E5%BF%AB%E9%80%9F%E6%96%B9%E4%BE%BF%E7%9A%84%E5%88%86%E6%9E%90%E7%BD%91%E7%AB%99%E6%97%A5%E5%BF%97/)