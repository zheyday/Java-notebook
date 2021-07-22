### MAC

chsh -s /bin/zsh 使用的shell是zsh

chsh -s /bin/bash 使用的shell是bash

## centos忘记登陆密码

在这个界面，键盘按下`e`

![1614588037777](Linux%E5%AD%A6%E4%B9%A0/1614588037777.png)

![1614588209429](Linux%E5%AD%A6%E4%B9%A0/1614588209429.png)

按下Ctrl+x

![1614588299336](Linux%E5%AD%A6%E4%B9%A0/1614588299336.png)

继续输入`exit`，`reboot`两个命令

## 静态ip

```cmd
vim /etc/sysconfig/network-scripts/ifcfg-ens33
#重启网络服务
service network restart
```

## 脚本

```cmd
vim hello.sh
-----
#!/bin/zsh
#注释
echo -e "\e[30;31m hello world \e[0m"
-----
#两种调用方法
#1、设置权限
chmod u+x hello.sh  chmod 755 hello.sh
./hello.sh
# 2、通过bash命令调用
bash hello.sh

netstat -lnp|grep 6379
```



```cmd
nohup java -jar XXX.jar &
nohup java -jar some.jar > std.out 2>&1 &
#查看进程
ps -ef|grep java
#杀死进程
kill -9 pid
```

![image-20210422190214049](/Users/zcs/Java-notebook/basic/Linux学习/008i3skNly1gpsr10nqjmj31700p4diz.jpg)

![image-20210423162302996](/Users/zcs/Java-notebook/basic/Linux学习/008i3skNly1gptr6wg34jj311y0hywn1.jpg)



![image-20210423164240556](/Users/zcs/Java-notebook/basic/Linux学习/008i3skNly1gptrr8qp1rj30uo0m2jyp.jpg)



## 命令帮助

| 命令     | 说明           |
| -------- | -------------- |
| cd/cd ~  | 切换到home     |
| grep     | 文本搜索       |
| df -h    | 磁盘空间状态   |
| du -sh   | 当前文件夹大小 |
| touch -a | 修改“读取时间” |
| touch -m | 修改“修改时间” |
| touch -d | 同时修改       |
| mkdir -p | 递归创建文件夹 |
|          |                |

## cmd

```xml
Ctrl+左右：单词之间跳转

ctrl+a:光标移到行首。

ctrl+e:光标移到行尾。 

ctrl+c:杀死当前进程。 

ctrl+k:清除光标后至行尾的内容。 

ctrl+u: 清除光标前至行首间的所有内容。 

ctrl+l:清屏，相当于clear。 

ctrl+r:搜索之前打过的命令。会有一个提示，根据你输入的关键字进行搜索bash的history 

ctrl+w: 移除光标前的一个单词 

ctrl+t: 交换光标位置前的两个字符 

ctrl+d: 删除光标所在字母;注意和backspace以及ctrl+h的区别，这2个是删除光标前的字符 
```



## chmod

![image-20210425141029992](/Users/zcs/Java-notebook/basic/Linux学习/image-20210425141029992.png)



![image-20210425141818322](/Users/zcs/Java-notebook/basic/Linux学习/image-20210425141818322.png)

## dd

按照指定大小和个数的数据块来复制文件或转换文件

```xml
dd if=/dev/zero of=560_file count=1 bs=560M
```

![1613359250789](Linux%E5%AD%A6%E4%B9%A0/1613359250789.png)

## cp

```yml
cp [options] source target
```



![1613366153366](Linux%E5%AD%A6%E4%B9%A0/1613366153366.png)

## grep



## vi

![1612874883253](Linux%E5%AD%A6%E4%B9%A0/1612874883253.png)

### 普通模式

```yml
#跳到末尾
G
#保存并退出
ZZ
#不保存退出
ZQ
#查找字符串
/和?
#继续查找下一个
n
#复制一行
yy
#粘贴到下一行
p
#粘贴到上一行
P
#删除光标所在字符
x
#撤销
u
```

### 命令行模式

普通模式下，按`:`进入

|    命令    |    说明    |
| :--------: | :--------: |
|     :w     |    保存    |
| :w newfile |   另存为   |
|    :wq     |  保存退出  |
|    :q!     | 不保存退出 |
|     :q     |  直接退出  |

## 正则表达式

![1612923864841](Linux%E5%AD%A6%E4%B9%A0/1612923864841.png)

![1612923883464](Linux%E5%AD%A6%E4%B9%A0/1612923883464.png)



## 打包压缩

```yml
#打包 只会打包成.tar
tar -cvf zcs.tar zcs
tar -zcvf zcs.tar.gz zcs
#解包
tar -xvf zcs.tar
#压缩
gzip t1.txt  #生成t1.txt.gz
#解压
gunzip t1.txt.gz
```



|      |                      |
| ---- | -------------------- |
| -c   | 创建压缩文件         |
| -x   | 解压文件             |
| -z   | 用Gzip压缩或解压     |
| -j   | 用bzip2压缩或解压    |
| -v   | 显示压缩或解压过程   |
| -f   | 目标文件名=          |
| -p   | 保留原始的权限和属性 |
| -P   | 使用绝对路径         |
| -C   | 解压到的目录         |











# windows

```cmd
netstat -ano | findstr "8080"
tasklist|findstr "pid"
```











