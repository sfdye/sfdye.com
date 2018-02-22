---
categories:
- Tech
date: 2014-03-28T00:00:00Z
last_modified_at: 2014-03-28 20:53:39 +08:00
tags:
- Sublime text
- Ssh
title: 如何使用Sublime Text编辑远程文件
---

对于我这个Vim使用起来还不太顺手的人来说，能用Sublime Text来编辑远程服务器上的文件简直就是福音。

很惊奇还有很多小伙伴不知道这个技巧。今天来分享一下:

* 用Sublime Text的Package Control安装一个叫**rsub**的插件
* 修改本地的ssh config（如果没有就创建这个文件）

{{< highlight bash >}}
$ vim ~/.ssh/config 	
{{< / highlight >}}
* 加入下面两行 

{{< highlight bash >}}
Host your_remote_server.com     # 这里填服务器的IP地址
    RemoteForward 52698 127.0.0.1:52698  
{{< / highlight >}}
	
* SSH到服务器

{{< highlight bash >}}
$ sudo wget -O /usr/local/bin/rsub https://raw.github.com/aurora/rmate/master/rmate
$ sudo chmod +x /usr/local/bin/rsub  
{{< / highlight >}}
	
* 大功告成。试试编辑任何文件，是不是在Sublime Text打开了？

{{< highlight bash >}}
$ rsub ~/webapps/myproject/some_file
{{< / highlight >}}

* 如果有权限问题导致不能编辑可以加`sudo`或者`-f`


*注：Textmate也用这个功能，命令是`rmate`，其实准确说`rsub`是从`rmate`fork来的*



### 参考文章

* [Sublime Tunnel of Love: How to Edit Remote Files With Sublime Text via an SSH Tunnel](http://log.liminastudio.com/writing/tutorials/sublime-tunnel-of-love-how-to-edit-remote-files-with-sublime-text-via-an-ssh-tunnel)