---
title: "Cheatsheet"
date: 2021-02-14T15:59:01+08:00
lastmod: 2021-02-14T15:59:01+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: "lttzz"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

<!--more-->

# Environment

``` plain text
Deskmini310/Hackintosh/macOS Big Sur
Thinkpad x1 carbon 2016/Linux/Manjaro/i3
```

## Hackintosh

### missing xcrun

``` bash
$ xcode-select --install
```

### 关闭sip

使用OC打开config.plist，选择NVRAM，找到并选中 7C436110AB2A-4BBB-A880-FE41995C9F82，在右边部分空白处，新建csr-active-config ，DATA类型，10.15 的系统值写入 E7030000，11的系统值写 77000000。

原文：macOS黑苹果系统关闭SIP服务教程

https://www.mfpud.com/topics/2821/

### 修改分辨率

Github：[one-key-hidpi](https://github.com/xzhih/one-key-hidpi)

## Manjaro/i3



# Editor

## vim

dotfile: vimrc

``` bash
ggvG=		# 代码格式化

# 跳转
%		# 跳转到匹配的(){}[]
(		# 跳转到下一个句子（句号分隔）
)		# 跳转回上一个句子（句号分隔）
{		# 跳转到下一个段落（空行分隔）
}		# 跳转回上一个段落（空行分隔）
+		# 跳转到下一行首个非空字符
-		# 跳转回上一行首个非空字符
H		# 跳转到屏幕上部
M		# 跳转到屏幕中部
L		# 跳转到屏幕下部
gd		# 跳转到局部定义
gD		# 跳转到全局定义
zz		# 调整光标所在行到屏幕中央
ctrl-o		# 跳转到上一个位置
ctrl-i		# 跳转到下一个位置
ctrl-f		# 下滚屏
ctrl-b		# 上滚屏

# 自动补全
ctrl-x ctrl-f		# 文件名补全
ctrl-x ctrl-n		# 文本内文字补全

# 剪切板
"+y	复制到系统剪切板
"+p	从系统剪切板粘贴

# 多文件
:e {filename}		# 打开文件并编辑，filename为空时，重载当前文件
:e .		# 打开文件管理器
:ls		# 列出缓存文件列表
:bn		# 切换到下一个缓存文件
:bp		# 切换到上一个缓存文件
:b {id}		# 切换到标号为 id 的缓存文件
:b {abc}		# 切换到文件名 abc 开头的缓存文件

# 多窗口
:sp {filename}		# 以横向分屏打开文件
:vs {filename}		# 以纵向分屏打开文件
:sp		# 横向分屏当前文件
:vs		# 纵向分屏当前文件
ctrl-w w		# 跳到下一个窗口
ctrl-w h		# 跳到左边的窗口
ctrl-w j		# 跳到下面的窗口
ctrl-w k		# 跳到上面的窗口
ctrl-w l		# 跳到右面的窗口

# 其他
gu/gU	# 大小写转换
:nohl		# 清除搜索高亮
:earlier 15m		# 回滚到15m前到文本
:later 15m
```

## jupyter

写笔记不要太爽，建议配上 **conda**。

``` bash
$ pip install jupyter

$ conda install jupyter
```



# Tool

## tmux

也不晓得为啥使用`<C-b>`作为默认的prefix key，`<C-space>`不香吗？香，我选择`<C-a>`。

``` bash
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

复制粘贴的轮子有人造好了：[tmux-yank](https://github.com/tmux-plugins/tmux-yank)

看到yank就晓得支持vim快捷键，好评。

vim为何使用yank/put，而不是通常的copy/paste，请看此处：[知乎@Dictionaryphile](https://www.zhihu.com/question/68713673/answer/273169409)

## gdb

## ctags

``` bash
ctags -R --c++-kinds=+p+l+x+c+d+e+f+g+m+n+s+t+u+v --fields=+liaS --extra=+q

:ta <keyword>
```

## cppman

``` bash
C++ manual pages for Linux/MacOS,with source from cplusplus.com and cppreference.com.
  
# cache all available man apges from cplusplus.com to enable off‐line browsing
$ cppman -c
# clear all cached files
$ cppman -C
# force cppman to update  existing  cache  when  '--cache-all'  or browsing man pages that were already cached
$ cppman -o
# rebuild index database from cplusplus.com
$ cppman -r
```

## cppcheck

``` bash
$ cppcheck filenam.cc

# 启用所有检查，并启用多线程
$ cppcheck --enable=all -j 4 filename.cc
```

## git

``` bash
# 设置账户信息
$ git config --global user.name “username”
$ git config --global user.email “email@example.com ”

# 显示用户设置信息
$ git config --global --list 

# 测试
$ ssh -T git@github.com

$ ssh-agent -s
$ ssh-add ~/.ssh/id_rsa

# 慎用 force（除非永远只有你一个人使用此repo，否则永远不要使用force
$ git push --force origin main

# 2021.08.13之日起将不再支持用户名密码的方式push代码

$ vim ~/.ssh/config
Host Github
    Hostname github.com
    IdentityFile ~/.ssh/xxx
    IdentitiesOnly yes
    
# 回退到 commitid 版本，适用于未 push 的情况，直接删除指定的 commit
$ git reset {commitid}
$ git reset --soft {commitid}

# 回退到 commitid 版本，适用于 push 的情况，和 reset 不同，是新增一次 commit
$ git revert {commitid}

# 修改上一次的 commit message
$ git commit --amend

$ git remote -v
$ git remote add upstream https://github.com/xxx
```

### 改端口

``` bash
$ ssh -T git@github.com
ssh: connect to host github.com port 22: Operation timed out

$ ssh -T -p 443 git@ssh.github.com
Hi lttzz! You've successfully authenticated, but GitHub does not provide shell access.

# 虽修改端口号
Host github.com
    Hostname ssh.github.com
    IdentityFile ~/.ssh/xxxx
    IdentitiesOnly yes
    Port 443
```



## Commitizen

``` bash
# commitizen 用于格式化 git commit message
# conventional-changelog 用于生成 change log

# 安装 commitizen
$ npm install -g commitizen conventional-changelog
# 项目目录下 init
$ commitizen init cz-conventional-changelog --save --save-exact

# 使用 git cz 替换 git commit
$ git cz 

# 生成 change log
$ conventional-changelog -p angular -i CHANGELOG.md -w
```

## ssh

``` bash
# 解决ssh连接超时，
# 方法一：server端添加以下两行
$ vim /etc/ssh/sshd_config
ClientAliveInterval 60 				//server每隔60秒发送一次请求给client
ClientAliveCountMax 3

# 方法二：client端添加以下两行
$ vim /etc/ssh/ssh_config
ServerAliveInterval 60 				//client每隔60秒发送一次请求给server
ServerAliveCountMax 3

# 方法三：ssh命令添加参数
$ ssh -o ServerAliveInterval=60
```

## tldr

## docker

``` bash
# fedora 俺来啦
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/fedora/docker-ce.repo
$ sudo dnf install docker-ce
# 设置自启
$ sudo systemctl start docker
$ sudo systemctl enable docker
$ sudo systemctl status docker

$ sudo dnf install docker-compose


## 复制文件
$ docker cp 
```

## MySQL

``` bash
# ubuntu 18.04

## 安装
$ apt install mysql-server
# 初始化安全配置
$ mysql_secure_installatio

## 卸载
# 查看依赖项
$ dpkg --list|grep mysql
# 卸载
$ sudo apt-get remove mysql-common
$ sudo apt-get autoremove --purge mysql-server-5.7
# 清楚残留数据
$ dpkg -l|grep ^rc|awk '{print$2}'|sudo xargs dpkg -P
# 再次查看依赖项，检查是否卸载干净
$ dpkg --list|grep mysql


## 远程访问
$ mysql -uroot -p

mysql> use mysql

mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)

# 修改为任意主机访问
mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | debian-sys-maint |
| localhost | mysql.session    |
| localhost | mysql.sys        |
+-----------+------------------+
4 rows in set (0.00 sec)

# 只时候无法使用公网ip访问，但是127.0.0.1是可以正常登录的
# 检查监听地址
$ netstat -antp | grep mysql
tcp   0   0  127.0.0.1:3306   0.0.0.0:*   LISTEN   23909/mysqld

# 修改配置文件
$ vim /etc/mysql/mysql.conf.d/mysqld.cn
bind-address          = 127.0.0.1		# 把这一行注释掉

$ systemctl restart mysql
# 再次检查监听地址
$ netstat -antp | grep mysql
tcp6       0      0 :::3306                 :::*                    LISTEN      24068/mysqld

# 至此，可以远程登录了
```

## rlwrap

解决MySQL等命令无法上下翻的问题，使用 `rlwrap MySQL`

## synergy

## transmission

``` bash
# error:permission denied 解决方法
[root@iZ6bp8hwl2ngmsZ ~]# ps -ef | grep transmission
transmi+  989295       1  0 19:18 ?        00:00:00 /usr/bin/transmission-daemon -f --log-error

# 方法一就是把 User 改为 root
$ systemctl edit transmission-daemon
### Editing /etc/systemd/system/transmission-daemon.service.d/override.conf
### Anything between here and the comment below will become the new contents of the file

### Lines below this comment will be discarded

### /usr/lib/systemd/system/transmission-daemon.service
# [Unit]
# Description=Transmission BitTorrent Daemon
# After=network.target
#
# [Service]
# User=transmission
# Type=notify
# ExecStart=/usr/bin/transmission-daemon -f --log-error
# ExecReload=/bin/kill -s HUP $MAINPID
# NoNewPrivileges=true
#
# [Install]
# WantedBy=multi-user.target


# 方法二就是改文件夹所有者，这里 tr 是下载目录
$ chmod 0777 -R
$ chown -R transmission:users tr
```

## pdftk

[pdftk](https://www.pdflabs.com/tools/pdftk-server/)

``` bash
# 合并
$ pdftk a.pdf b.pdf output out.pdf

# 截取指定页码范围
$ pdftk in.pdf cat 1-3 7-end output out.pdf
# 转换成单页面PDF
$ pdftk file.pdf burst
```

## adb

``` bash
adb connect {ip}
adb devices

adb install /path/to/application.apk
adb install -r /path/to/application.apk # 保留数据安装
adb uninstall {packagename}

adb shell pm list packages	# 列出所有包名
adb shell pm list packages -3	# 列出第三方包名

adb logcat -c
adb logcat -f | grep {keyword}
adb logcat -s {TAG}:I	# 输出标签为TAG的I级别之上的log
```

## you-get

mp4格式下载后未自动合并的话可能是没装ffmpeg

## youtube-dl

## gcc

``` bash
# 写 c++ primer 习题，用着 macos 的 llvm/clang 真不舒服

# http://hpc.sourceforge.net/

$ brew install gcc
```

## sicp

``` bash
# 1、下载 
http://www.gnu.org/software/mit-scheme/

# 2、根据安装文档
http://www.gnu.org/software/mit-scheme/documentation/stable/mit-scheme-user/Unix-Installation.html
```



# Linux

## find

``` bash
# case insensitive
find PATH -iname <pattern>
```

## other

``` bash
# 断开其他用户会话
$ w		# Show who is logged on and what they are doing.
$ tty		# print the file name of the terminal connected to standard input
$ pkill -9 -t pts/[id]
```

## RTFS

``` bash
# 以ubuntu为例
$ which cat
/bin/cat
$ dpkg -S /bin/cat
coreutils: /bin/cat
$ apt-get source coreutils

# 或者通过git将源码pull下来
http://www.gnu.org/software/
```



