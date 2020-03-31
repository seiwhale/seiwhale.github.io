---
title: Linux 神奇的 ! 命令
date: 2020-3-31 17:15:43
tags:
  - linux
categories:
  - 技术随笔
keywords: linux
description: ! 不起眼却非常神奇的 Linux 命令，让我们一起看看怎么使用吧！
cover: https://cdn.jsdelivr.net/gh/seiwhale/assets/img/blog/linux.jpg
---

`Linux` 中可以通过很多命令进行系统的管理，可以说 `Linux` 系统管理的正常运行就是指命，今天我们要说一个不起眼却非常神奇的 Linux 命令：`!`（感叹号）。

## !! 执行上一条命令

例如，在执行完上面一条命令后，可以使用下面的方式再次执行上一条命令：

```bash
$ whereis bash  #执行命令
bash: /bin/bash /etc/bash.bashrc /usr/share/man/man1/bash.1.gz

$ !!            #再次执行上一条命令
whereis bash
bash: /bin/bash /etc/bash.bashrc /usr/share/man/man1/bash.1.gz
```

`!!` 代表了上一条执行的命令。可以看到，当输入两个感叹号时，它显示上条命令的同时会执行上一条命令。当然了，通常我们还会想到使用 `UP` 键来完成这个事情。但是如果是基于上条命令扩充，`!!` 就来得更加方便了。

比如，你想查看某个文件，但是忘了输入 `more`：

```bash
$ /opt/user/test.txt  # 忘记输入more
$ more !!      #这样是不是快多了？
```

使用 `!!` 是不是方便多了？

## !\$ 使用上个命令最后一个参数

比如，你在使用 `ls` 列出目录内容时，没有带任何参数，但是想再次执行，带上 `-al` 参数，又不想输入长长的参数，可以使用下面的方式：

```bash
$ ls /proc/1/task/1/net/tcp
/proc/1/task/1/net/tc

$ ls -al !$
ls -al /proc/1/task/1/net/tcp
-r--r--r-- 1 root root 0 12月 22 17:30 /proc/1/task/1/net/tcp
```

这里的 `!$` 代表了上一条命令的最后一个参数。

## !^ 使用上个命令第一个参数

当然我们也可以通过 `!^` 使用上个命令第一个参数。

```bash
$ ls -al !^
```

## !:- 去掉最后一个参数执行上一个命令

如果想执行上条命令，但不想带上最后一个参数：

``` bash
$ ls -al dir  #假设dir是一个很长的字符串
$ !:-
ls -al
```

什么场景下可能会用呢？比如你上一条命令最后一个参数是一个长长的字符串，而你恰好不想不用它，并且退格键删除又慢的时候，可以使用上面的方法。

## !* 使用上条命令的所有参数

前面说了使用上条命令的最后一个参数，那如果不是最后一个参数，该如何使用呢？很简单，使用 `!*` 即可。例如我们在输入 `find` 命令输错了，想要纠正的时候：

``` bash
$ fin -name "test.zip"  #这里find输错了。
$ find !*
find ./ -name "test.zip"
./workspaces/shell/find/test.zip
./workspaces/shell/test.zip
```

## ![命令名]:[参数号] 使用其中某个参数

有的读者可能会问了，如果我只想用其中某个参数呢？按照 `![命令名]:[参数号]` 的规则即可。例如：

``` bash
$ cp -rf dira dirb/   #将dira拷贝到dirb
$ ls -l !cp:2        #查看dira的内容
ls -l dira
total 0
-rw-rw-r-- 1 hyb hyb 0 12月 22 17:45 testfile
```

当上条命令的参数很长，而你需要取用中间的某个参数时，效果就比较明显了。

## ![历史命令数值] 采用 history 中的命令

我们都知道可以通过 `history` 命令可以查看之前执行过的命令，但是如何再次执行 `history` 中的命令呢？我们可以通过 `UP` 键可以查看，但是历史命令很长的时候，并不是很方便，这个时候 `!` 便派上了用场：

``` bash
$ history
 (这里省略更多内容)
 2043  touch ./dira/testfile
 2044  cp -rf dira dirb/
 2045  ls -al dira
 2046  ls -l dira
 2047  ls -al dira
 2048  ls -l dira
 2049  ls -al dira
 2050  ls -l dira
 2051  history 
```
我们可以看到，`history` 命令出来可以看到之前执行过的命令，也会看到它前面带了一个数值。如果我们想执行前面的 `cp -rf dira dirb/` 命令，实际上只要用下面的方式即可：

``` bash
$ !2044   #2044是执行的第n条命令
cp -rf dira dirb/
```

即通过 `![历史命令数值]` 的方式执行历史命令。
当然了，如果我们想执行倒数第二条命令，也是有方法的：

``` bash
$ !-2   #感叹号后面跟着一个负数，负几代表倒数第几条
```

## ![关键字] 按照关键字执行

`!` 可以根据关键字执行命令。

执行上一条以关键字开头的命令
例如，执行上一条 `find` 命令：

``` bash
$ !find    #执行上条以find开头的命令
```

执行上一条包含关键字的命令
再例如，执行上一条包含 `name` 的命令：

``` bash
$ find ./ -name "test"
./test
./find/test
$ !?name?
find ./ -name "test"
./test
./find/test
```

## !!:gs/[old]/[new] 替换上条命令的参数

例如：

``` bash
$ find ./ -name "old*" -a -name "*.zip"
```

如果我们需要将这条命令中的 `old` 更换为 `new`：

``` bash
$ !!:gs/old/new
```

## 逻辑非

这个是它最为人所熟悉的作用，例如删除除了 `cfg` 结尾以外的所有文件：

``` bash
rm !(*.cfg)  #删除需谨慎
```

## 结尾

`!` 的神奇不仅仅止于上面我们所描述的，我们在这边不再展开，要是有更多好的发现，可以在下面进行留言哦~