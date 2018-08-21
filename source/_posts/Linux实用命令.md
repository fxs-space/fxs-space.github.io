---
title: Linux实用命令
date: 2017-10-23 20:19:00
categories: Linux
tags: Linux
---
## Linux实用命令

#### Linux实用命令

##### 查看当前特定的所有进程

```
ps -ef | grep *

eg: ps -ef | grep VirtualBox
```
##### 杀死特定的进程

```
kill -9 PID

eg: kill -9 63586
```

<!--more-->


##### 查看命令行的历史记录

```
history

(后面可加number,查最近n条记录)
eg: history 10
```

##### 查看特定命令的历史记录

```
history | grep *

eg: history | grep code-push
```