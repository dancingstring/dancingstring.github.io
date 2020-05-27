---
title: ssh别名免密登录
date: 2020-05-27
---

ssh登录远程服务器是开发和运维经常做的事，但是每次输入ip、用户名密码费心费力，所以此处记录下使用密钥并设置别名实现免密登录远程服务器的步骤。

#### 使用私钥实现免密登录
1、在终端下执行以下命令生成私钥文件：
```
ssh-keygen -t rsa
```
> 如果文件"~/.ssh/id_rsa"存在，会提示是否覆盖该文件，此时可选择"n"不覆盖该文件而使用已有的id_rsa文件。

2、将生成的id_rsa.pub文件拷贝到远程服务器的 ~/.ssh 目录下，然后在终端下执行以下命令将公钥追加到授权KEY里面。
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
> 也可手动将id_rsa.pub内容追加到authorized_keys中，效果一样。

此时在客户端终端执行`ssh root@xx.xx.xx.xx`即可实现免密登录。

#### 配置远程主机别名
为避免每次ssh登录都要输入`ssh -p port user@host`，可以将常用的远程服务器设置别名，然后ssh指定别名连接。


在~/.ssh文件夹下创建一个名为config的文件。然后添加如下内容：
```
Host myserver #myserver 可以替换为想设置的别名
HostName ip #远程主机的IP地址
User user #远程主机的用户名
Port port#远程主机的端口号
```

此时在终端执行`ssh myserver`即可登录远程服务器。