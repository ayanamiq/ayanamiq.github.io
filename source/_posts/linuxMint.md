---
title: linuxMint
tags: 其他
date: 2019-03-16 12:37:30
---

以后办公的笔记本就用linux了。一直想用deepin，奈何挣扎了一天都没装上(笔记本的问题)，耗尽了我的耐心，然后安装了linuxMint。下面记录踩坑之路。

# 安装必要指令
## gedit
`sudo apt-get install gedit`

# 软件安装
## jdk
首先卸载系统自带的jdk，查看安装的软件包
sudo dpkg --list | grep -i jdk

删除jdk
sudo apt-get purge openjdk*

删除其他的包
sudo apt-get purge icedtea-* openjdk-*
cd /usr/lib/jvm （如果没有jvm这个目录，就不用管这步了）
sudo rm -rf jdk<version>

再次确认是否被删除
sudo dpkg --list | grep -i jdk

https://www.oracle.com/technetwork/java/javase/downloads/index.html
下载想要的jdk，下载完成后，解压到/opt目录
`sudo tar zxvf jdk.tar.gz -C /opt/`
注意将jdk改成自己下载的jdk名字

下面配置环境变量，需要提前安装gedit，如上。
gedit安装好后，`sudo gedit /etc/profile`
在 profile 文件的后面加上以下语句
```
# JAVA Config
export JAVA_HOME=/opt/jdk1.8.0_101
export JRE_HOME=/opt/jdk1.8.0_101/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
注意将jdk名字改成自己的。

使文件立即生效
`source /etc/profile`
在终端输入java -version，看到版本号说明成功。


## node
https://nodejs.org/zh-cn/download/
官网下载linux版本的二进制文件
`xz -d  node-v8.11.1-linux-x64.tar.xz`
上边指令执行完后，文件末尾没有.xz了，然后将文件解压到想要的目录
`sudo tar zxvf node_10.15.tar -C /home/wjy/install`

添加链接，可以全局使用命令
`ln -s /root/node-10.15/bin/node /usr/local/bin/node`
`ln -s /root/node-10.15/bin/npm /usr/local/bin/npm`

执行完在终端任意目录 输入 node -v，看到版本号说明成功。

## snap
```
sudo apt-get install snapd 
sudo apt-get install snapcraft 
```

- 列出已经安装的snap包
`sudo snap list`

- 搜索要安装的snap包
`sudo snap find <text to search>`

- 安装一个snap包
`sudo snap install <snap name>`

- 更新一个snap包，如果你后面不加包的名字的话那就是更新所有的snap包
`sudo snap refresh <snap name>`

- 把一个包还原到以前安装的版本
`sudo snap revert <snap name>`

- 删除一个snap包
`sudo snap remove <snap name>` 

- 安装网易云音乐
`sudo snap install netease-music --devmode --beta`

- 安装微信
`sudo snap install electronic-wechat`
需要重启才能在左下角管理中看到微信

## git
更新linux mint 内核及软件包
```
sudo apt-get update 
sudo apt-get upgrade
```

使用dpkg包管理器查询是否已安装git
`sudo dpkg -l git`


通过apt-get 安装软件库中最新版本的git
```
sudo apt-get install git
```

检查是否安装成功
`git --version`

### idea中配置git
使用apt-get install 方式安装的软件，
1.下载的软件存放位置 
/var/cache/apt/archives

2.安装后软件默认位置 
/usr/share

3.可执行文件位置 
/usr/bin

4.配置文件位置 
/etc

5.lib文件位置 
/usr/lib

所以在idea中配置git的可执行文件，选择位置use/bin/git
点击test，显示git版本号，说明成功，就可以从github或coding上拉取代码了。

## zookeeper
下载地址：
http://mirror.bit.edu.cn/apache/zookeeper/
下载stable版本。

将下载好的.gz解压并复制到想要的位置。/home/wjy/install

到zk的bin目录下启动：
`./zkServer.sh statr`

查看状态
```
ZooKeeper JMX enabled by default
Using config: /home/wjy/install/zookeeper-3.4.13/bin/../conf/zoo.cfg
Mode: standalone
```
出现上边的状态说明启动成功。

## redis
https://redis.io/
官网下载，解压并复制到想要的位置 /home/wjy/install

cd到目录下，执行
`make`
报错：
```
net.c:36:23: fatal error: sys/types.h: No such file or directory(没有那个文件或目录)
...
adlist.c:32:20: fatal error: stdlib.h: No such file or directory
```
原因是缺少文件，执行以下命令后再次make：
`sudo apt-get install libc6-dev`

还是报错:
`zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory`

原因是libc不是默认的分配器，在make后添加参数解决：
`sudo make MALLOC=libc`

等待执行完成，到redis/src目录下，执行
`./redis-server`
出现端口号和进程PID，证明启动成功。

## postman
https://www.getpostman.com/downloads/
下载对应版本.gz,并解压到理想位置。

创建全局变量，也就是在任何地方都可以执行postman，不用去到安装目录，执行
`sudo ln -s /home/wjy/install/Postman/Postman  /usr/bin/postman`

添加启动器应用图标，也就是可以从启动器快速启动，执行
`vim /usr/share/applications/postman.desktop`
填写如下内容：注意修改路径
```
[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=postman
Icon=/home/wjy/install/Postman/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
```
然后就可以在启动器搜索到postman，然后可以将该应用添加到桌面。

## navicat
官网下载：https://www.navicat.com.cn/download/navicat-premium
直接下载试用版，解压到理想位置。
进入到解压好的目录,启动
`./start_navicat`
启动成功。

将navicat快捷方式添加到桌面
`vim /usr/share/applications/navicat.desktop`
填写如下内容：注意修改路径
```
[Desktop Entry]
Encoding=UTF-8
Name=Navicat Premium
Comment=The Smarter Way to manage dadabase
Exec=/bin/sh "/home/wjy/install/navicat121_premium_cs_x64/start_navicat"
Icon=/home/wjy/install/navicat121_premium_cs_x64/navicat.png
Categories=Application;Database;MySQL;navicat
Version=1.0
Type=Application
Terminal=0
```
然后就可以在启动器搜索到navicat，然后可以将该应用添加到桌面。


# 快捷键冲突问题
- ctrl+alt+s
这个快捷键是idea的setting的快捷键，但是在mint中被fcitx输入法占用了。
右键右下角的输入法，选择配置fcitx->全局配置->显示高级选项,找到想修改的快捷键，建议直接置空(选中后，点击esc)。

# VI模式中上下左右键和回退键出现字母
`sudo vi /etc/vim/vimrc.tiny`
上下左右字母：
这个文件里面的倒数第二句话是“set compatible”改为“set nocompatible”


回退健：
在“set nocompatible”后面(下一行)加上
set backspace=2
