---
title: Linux常用操作命令
date: 2019-02-08 15:45:23
categories: JavaScript
tags: [Linux]
---

## 常用指令
|命令|修饰符|描述|
|---|---|---|
|ls||显示文件或目录|
||-l|列出文件详细信息l(list)|
||-a|列出当前目录下所有文件及目录，包括隐藏的a(all)|
|mkdir||创建目录|
||-p|创建目录，若无父目录，则创建p(parent)|
|cd||切换目录|
|touch||创建空文件|
|echo||创建带有内容的文件|
|cat||查看文件内容|
|cp||拷贝|
|mv||移动或重命名|
|rm||删除文件|
||-r|递归删除，可删除子目录及文件|
||-f|强制删除|
|find||在文件系统中搜索某文件|
|wc||统计文本中行数、字数、字符数|
|grep||在文本文件中查找某个字符串|
|rmdir||删除空目录|
|tree||树形结构显示目录，需要安装tree包|
|pwd||显示当前目录|
|ln||创建链接文件|
|more、less||分页显示文本文件内容|
|head、tail||显示文件头、尾内容|
|ctrl+alt+F1||命令行全屏模式|

## 系统管理命令
|命令|修饰符|描述|
|---|---|---|
|stat||显示指定文件的详细信息，比ls更详细|
|who||显示在线登陆用户|
|whoami||显示当前操作用户|
|hostname||显示主机名|
|uname||显示系统信息|
|top||动态显示当前耗费资源最多进程信息|
|ps||显示瞬间进程状态 ps -aux|
|du||查看目录大小|
||-h /home|带有单位显示目录信息|
|df||查看磁盘大小|
||-h|带有单位显示磁盘信息|
|ifconfig||查看网络情况|
|ping||测试网络连通|
|netstat||显示网络状态信息|
|clear||清屏|
|alias||对命令重命名 如：alias showmeit="ps -aux" ，另外解除使用unaliax showmeit|
|kill||杀死进程，可以先用ps 或 top命令查看进程的id，然后再用kill命令杀死进程|

## vim使用
vim三种模式：命令模式、插入模式、编辑模式。使用ESC或i或:来切换模式。
命令模式下：

|命令|修饰符|描述|
|---|---|---|
|:q||退出|
|:q!||强制退出|
|:wq||保存并退出|
|:set number||显示行号|
|:set nonumber||隐藏行号|
|/apache||在文档中查找apache 按n跳到下一个，shift+n上一个|
|yyp||复制光标所在行，并粘贴|
|h||左移一个字符←|
|j||下一行↓|
|k||上一行↑|
|l||右移一个字符→|

## 用户及用户组管理
|命令|修饰符|描述|
|---|---|---|
|/etc/passwd||存储用户账号|
|/etc/group||存储组账号|
|/etc/shadow||存储用户账号的密码|
|/etc/gshadow||存储用户组账号的密码|
|useradd、adduser||建立用户帐号和创建用户的起始目录|
|userdel||删除用户帐号|
|groupadd||建立组账号|
|groupdel||删除组账号|
|passwd root||给root设置密码|
|su root||切换root|
|/etc/profile||系统环境变量|
|bash_profile||用户环境变量|
|.bashrc||用户环境变量|
|su user||切换用户，加载配置文件.bashrc|
|su - user||切换用户，加载配置文件/etc/profile ，加载bash_profile|

## 文件权限管理
三种基本权限

|权限|描述|数值表示|
|---|---|---|
|R|读|4|
|W|写|2|
|X|可执行|1|

更改权限
sudo chmod [u所属用户  g所属组  o其他用户  a所有用户]  [+增加权限  -减少权限]  [r  w  x]   目录名 
