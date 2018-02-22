---
categories:
- Music
date: 2013-11-09T00:00:00Z
tags:
- Mac
- Cubase
title: 解决Cubase在Retina Mackbook Pro下运行无响应的办法
---

本方案在 
Cubase 6 AI
13寸Retina Macbook Pro 2013款
下成功解决无响应问题

升级到Retina MBP之后发现Cubase不能用，打开后鼠标点击么没有反应，好不容易进入程序也什么都做不了。Google一下发现R屏太清晰，Cubase还不支持。官方给出了一个[解决办法][solution_link]。

[solution_link]: https://www.steinberg.net/en/support/knowledgebase_new/show_details/kb_show/using-a-macbook-pro-retina-display-with-steinberg-software.html

翻译如下：

* Quit the application if it is currently open 退出当前打开的程序
* In the Finder, choose "Applications" from the "Go menu" 到Application文件夹里面找到Cubase
* In the applications folder, select the application's icon and press Command+I to open the information window 右键属性（或者Cmd+I)
* Check "Open in Low Resolution" to enable the low resolution mode. 选中 “在低分辨率下运行”
* Close the window and double click the application to reopen it. 重新启动Cubase


![Cubase]({{ site.url }}/images/cubase.png) 
{: .pull-right}

这样就大功告成了。
