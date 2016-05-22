---
title: Fabric入门
date: 2016-05-22 12:20:38
category: [技术总结]
tags: [技术总结]
---

## 前言
Fabric是一个用Python写的SSH连接工具，可用于应用开发、部署和系统管理。

我的项目需要使用SSH远程登录到服务器中执行某个命令，可以使用Fabric实现整个流程。但我只需要在一台服务器上执行命令，使用Fabric有些浪费，完全可以使用更轻的框架。好在Fabric很容易上手，就暂定用它执行命令。

以下内容来自Fabric的官方文档[《Overview and Tutorial》](http://docs.fabfile.org/en/latest/tutorial.html)

## 什么是Fabric
Fabric提供一套SSH组件，包括Python库和命令行工具，用于应用开发和系统管理。简单说，Fabric有下面两种工具：

通过命令行远程执行Python函数
使用Python函数通过ssh远程执行shell命令

## Hello, fab
创建一个fabfile.py文件

```python
def hello():
    print("Hello world!")
```
在命令行中使用fab命令运行

```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello
Hello world!
Done.
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello
Hello world!

Done.
```

## 任务参数
为上面函数添加参数

```python
def hello(name="world"):
    print("Hello %s!" % name
```
运行，参数格式 :,=,….
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello:name=windroc
Hello windroc!
Done.
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello:name=windroc
Hello windroc!
Done.
```
下面方式具有同样效果
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello:windroc
Hello windroc!

Done.

vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab hello:windroc
Hello windroc!

Done.
```

本地命令
使用 ```fabric.api.local```函数在本机运行shell命令，从这里我们将开始看到Fabric的强大功能。

fabfile.py
```python
from fabric.api import local

def prepare_deploy():
    local("date")
    local("who")
    local("uname -a")
```

运行结果如下
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab prepare_deploy
[localhost] local: date
2015年 02月 15日 星期日 07:03:46 UTC
[localhost] local: who
vagrant  pts/0        2015-02-15 01:15 (10.0.2.2)
vagrant  pts/1        2015-02-11 12:03 (10.0.2.2)
vagrant  pts/2        2015-02-14 13:11 (10.0.2.2)
vagrant  pts/4        2015-02-14 13:16 (10.0.2.2)
vagrant  pts/5        2015-02-15 02:56 (10.0.2.2)
[localhost] local: uname -a
Linux precise64 3.8.0-44-generic #66~precise1-Ubuntu SMP Tue Jul 15 04:01:04 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
 
Done.
```

## 组织fabfile
比如分成多个子任务
```python
from fabric.api import local

def print_date():
    local("date")

def print_who():
    local("who")


def print_uname():
    local("uname -a")

def prepare_deploy():
    print_date()
    print_who()
    print_uname()

```
输出与之前一样

## 故障
Fabirc检查命令操作的返回值，如果没有正常退出则终止。

在fabfile.py中加入一个肯定会出错的命令。
```python
from fabric.api import local

def print_date():
    local("date")

def print_who():
    local("who")

def print_uname():
    local("uname -a")

def error_task():
    local("asdagasdg")

def prepare_deploy():
    print_date()
    print_who()
    error_task()
    print_uname()
```

运行出错
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab prepare_deploy
[localhost] local: date
2015年 02月 15日 星期日 07:06:55 UTC
[localhost] local: who
vagrant  pts/0        2015-02-15 01:15 (10.0.2.2)
vagrant  pts/1        2015-02-11 12:03 (10.0.2.2)
vagrant  pts/2        2015-02-14 13:11 (10.0.2.2)
vagrant  pts/4        2015-02-14 13:16 (10.0.2.2)
vagrant  pts/5        2015-02-15 02:56 (10.0.2.2)
[localhost] local: asdagasdg
/bin/sh: 1: asdagasdg: not found
 
Fatal error: local() encountered an error (return code 127) while executing 'asdagasdg'
 
Aborting.
local() encountered an error (return code 127) while executing 'asdagasdg'
```
可以看到，运行出错后，不再运行后续任务

## 故障处理
环境变量warn_only，可以将abort转为warning，并提供灵活的处理方式

fabfile

```python
from fabric.api import local, settings, abort
from fabric.contrib.console import confirm
 
 
def print_date():
    local("date")
 
def print_who():
    local("who")
 
def print_uname():
    local("uname -a")
 
def error_task():
    with settings(warn_only=True):
        result = local("asdagasdg")
    if result.failed and not confirm("Test failed. Continue anyway?"):
        abort("Aborting at user request")
 
def prepare_deploy():
    print_date()
    print_who()
    error_task()
    print_uname()
```

出错时，会根据用户输入决定是否运行后面的程序

运行
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab prepare_deploy
[localhost] local: date
2015年 02月 15日 星期日 07:10:39 UTC
[localhost] local: who
vagrant pts/0 2015-02-15 01:15 (10.0.2.2)
vagrant pts/1 2015-02-11 12:03 (10.0.2.2)
vagrant pts/2 2015-02-14 13:11 (10.0.2.2)
vagrant pts/4 2015-02-14 13:16 (10.0.2.2)
vagrant pts/5 2015-02-15 02:56 (10.0.2.2)
[localhost] local: asdagasdg
/bin/sh: 1: asdagasdg: not found

Warning: local() encountered an error (return code 127) while executing ‘asdagasdg’

Test failed. Continue anyway? [Y/n] Y
[localhost] local: uname -a
Linux precise64 3.8.0-44-generic #66~precise1-Ubuntu SMP Tue Jul 15 04:01:04 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

Done.

## 建立连接
Fabric最强大的功能，通过SSH远程执行任务

修改任务

```python
def print_ls():
    with cd('/var/log'):
        run("ls")
```
执行任务，会提示输入host，使用ssh连接形式user@host:port
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab prepare_deploy
No hosts found. Please specify (single) host string for connection: wangdp@uranus.hpc.nmic.cn
[wangdp@uranus.hpc.nmic.cn] run: date
[wangdp@uranus.hpc.nmic.cn] Passphrase for private key: 
[wangdp@uranus.hpc.nmic.cn] Login password for 'wangdp': 
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: Sun Feb 15 07:19:01 GMT 2015
[wangdp@uranus.hpc.nmic.cn] out: 
 
[wangdp@uranus.hpc.nmic.cn] run: who
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
…
[wangdp@uranus.hpc.nmic.cn] out: nwp_bj      pts/65      Feb 15 03:20     (localhost:24.0)
[wangdp@uranus.hpc.nmic.cn] out: 
 
[wangdp@uranus.hpc.nmic.cn] run: ls
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: aso       sudo.log  xcat
[wangdp@uranus.hpc.nmic.cn] out: 


Done.
Disconnecting from uranus.hpc.nmic.cn... done.
```
也可以在命令行中给出host。
```python
from fabric.api import run
 
def host_type():
    run("uname -s")
```
运行结果
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab -H localhost host_type
[localhost] Executing task 'host_type'
[localhost] run: uname -s
[localhost] Passphrase for private key: 
Sorry, you can't enter an empty password. Please try again.
[localhost] Passphrase for private key: 
[localhost] Login password for 'vagrant': 
[localhost] out: Linux
[localhost] out: 
 
 
Done.
Disconnecting from localhost... done.
```

## 远程交互
Fabric可以与远端进行对话，看下面的例子

fabfile.py
```python
def rm_file():
    tmp_dir = "/cma/g3/wangdp/tmp"
 
    with cd(tmp_dir):
        run("touch 1.txt")
        run("rm -i 1.txt")
 
def deploy():
    rm_file()
```
运行
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab deploy
No hosts found. Please specify (single) host string for connection: wangdp@uranus.hpc.nmic.cn
[wangdp@uranus.hpc.nmic.cn] run: touch 1.txt
[wangdp@uranus.hpc.nmic.cn] Passphrase for private key: 
[wangdp@uranus.hpc.nmic.cn] Login password for 'wangdp': 
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: 
 
[wangdp@uranus.hpc.nmic.cn] run: rm -i 1.txt
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: rm: remove 1.txt? y
[wangdp@uranus.hpc.nmic.cn] out: 
 
 
Done.
Disconnecting from uranus.hpc.nmic.cn... done.
```

提前定义连接
最常见的方法，设置env.hosts列表

在fabfile模块头部设置env.hosts

```python
env.hosts = ['wangdp@uranus.hpc.nmic.cn']
```

运行
```
vagrant@precise64:/vagrant/sms_log_agent_monitor$ fab deploy
[wangdp@uranus.hpc.nmic.cn] Executing task 'deploy'
[wangdp@uranus.hpc.nmic.cn] run: touch 1.txt
[wangdp@uranus.hpc.nmic.cn] Passphrase for private key: 
[wangdp@uranus.hpc.nmic.cn] Login password for 'wangdp': 
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: 
 
[wangdp@uranus.hpc.nmic.cn] run: rm -i 1.txt
[wangdp@uranus.hpc.nmic.cn] out: welcome login
[wangdp@uranus.hpc.nmic.cn] out: [YOU HAVE NEW MAIL]
[wangdp@uranus.hpc.nmic.cn] out: rm: remove 1.txt? y
[wangdp@uranus.hpc.nmic.cn] out: 
 
 
Done.
Disconnecting from wangdp@uranus.hpc.nmic.cn... done.
```

## 总结
本文介绍了Fabric的基本功能：

定义任务，并用fab命令执行
使用local运行本地命令
使用settings修改环境变量
处理命令出错，提醒用户，手动中止
定义host列表，使用run远程执行命令。







