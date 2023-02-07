---
title: DockerDesktop错误总结
date: 2023-02-07 16:44:02
tags:
---


##### 开发环境：
- windows11
- AMD Ryzen 7 
- 16G+512


#### 问题一
``Docker Desktop System.InvalidOperationException``

目前不知道该问题出现原因，在电脑重启后会发生该错误，解决方案如下

使用带有**管理员权限**的cmd执行、重启便可修复
``netsh winsock reset``


#### 问题二
问题出现原因
想在虚拟机里进行Docker环境学习开发，把本地Docker卸载时所发生的问题

首先我没有找到Docker desktop的卸载方法，使用windows自带卸载工具提示我无法完成
我试着将Docker desktop安装目录、容器目录删除 窃窃欢喜大功告成
过了几天后 想试着安装Docker desktop做一些简单的环境开发，却发现点击安装包时一直无响应

解决过程

1. ~~清理Docker安装目录（无效）~~ 
![在这里插入图片描述](https://img-blog.csdnimg.cn/84aa84bf02c54518b50d0d9477298451.png)
2. ~~安装进程监控器 找到Docker的UpgradeCodes(无效、没有执行UpgradeCodes进程)~~ 
![在这里插入图片描述](https://img-blog.csdnimg.cn/79e9fe2e9752496280cc7a3cf52a35f6.png)
最终解决方案
windows下win+r、打开regedit(注册表)，搜索docker，将相关的注册表全部删除，重启电脑后再次尝试安装

> 删除时需仔细查看  有的可能不是docker的不用删除

#### 问题三
安装docker desktop后，状态栏点击quit、restart等功能按钮无效、进入设置一直加载、docker命令提示
``Error response from daemon: open \\.\pipe\docker_engine_linux: The system cannot find the file specified``

解决方案：

1. 管理员运行cmd或powershell
```powershell
cd "C:\Program Files\Docker\Docker"

./DockerCli.exe -SwitchDaemon
```
2. DockerD desktop->TroubleShoot->Clean/purge data->选择WSL 2->delete->重启Docker desktop
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a42d2702a1d43aba345fd9b1d043eff.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/9ad27157bdb941f29199d96c79029809.png)
3. 进入docker desktop settings 取消WSL2，点击保存便可成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/aa5eec896027414aa07a0d77ab3658d3.png)

#### 问题四
WSL启动Ubuntu时报错“参考的对象类型不支持尝试的操作

方法一：
调用管理员终端输入
```shell
    netsh winsock reset
```
执行后无需重启，便可以顺利打开Ubuntu，报错消失。
但是不得不说确实是“临时方法”，每次开机都要输一遍指令太麻烦了

方法二：
下载NoLsp.exe

方法三：
关闭游戏加速器
