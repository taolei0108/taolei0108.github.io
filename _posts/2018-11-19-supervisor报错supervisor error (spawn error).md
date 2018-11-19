---
layout:     post
title:      supervisor报错supervisor error (spawn error)
subtitle:   工作中一次报错的记录
date:       2018-11-19
author:     TL
header-img: img/supervisor_head.jpg
catalog: true
tags:
    - 报错
    - supervisor
---

这几天部门里同事做一个镜像的迭代测试，需要我收集镜像里生成的日志。所以我在镜像里安装filebeat去收集日志写入kafka。

轻车熟路的在Dockerfile里配好了supervisor的配置，就自信满满的打了镜像，去测试了。

结果过后去看时，执行命令supervisorctl，结果发现如下图报错：

![image](https://note.youdao.com/yws/public/resource/d84350ab4ba80db7297e1136abb91b01/xmlnote/C471C22845F747969D0A516329F7E307/10945)

```
ps -ef |grep filebeat
```
发现filebeat服务进程存在。

这时候检查supervisor的日志，发现没有报错。检查filebeat的服务日志也没有报错，这下就只能求助google了。google了会发现别人的博客里，日志里都有报错，所以和
我不一样。所以就去查了supervisor的官网。发现了如下：

![image](https://note.youdao.com/yws/public/resource/d84350ab4ba80db7297e1136abb91b01/xmlnote/6967C5224C0F4DFFA3898A0DFAD8054A/10946)

意思是supervisor 的sub_programname需要放在前台运行，supervisor是父进程，它创建的每一个子进程监听管理的进程，通过SIGCHLD信号得知管理的进程死亡。所以管理的
进程不能使用daemon模式。所以你在配置文件里的common必须是运行在前台的命令。

最后我编了个脚本，死循环的每秒监听服务，用supervisor去监听脚本运行。

希望对大家能有帮助！
