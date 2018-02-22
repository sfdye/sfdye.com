---
categories:
- Tech
date: 2014-04-04T00:00:00Z
last_modified_at: 2014-04-04 17:23:25 +08:00
tags:
- Django
- Python
- Virtualenv
- Virtualenvwrapper
title: 管理多Python环境之Virtualenv与Virtualenvwrapper
---

在[《Django 最佳实践与部署》][article]一文中我曾提到过用 virtualenv 创建虚拟环境的好处。今天将结合 virtualenvwrapper---virtualenv 的*终极伴侣*，来具体谈一谈如何利用虚拟环境提高开发效率。

[article]: http://sfdye.com/articles/django-best-practice-and-deployment-with-nginx-gunicorn-and-supervisor/

### 为什么要使用 virtualenv 和 virtualenvwrapper

作为一个完美主义者，不喜欢看到系统`site-packages`放着各种各样 Python 包。很多包只是因为某个项目需要，而根本没有必要放在全局。我喜欢 virtualenv 这种相互独立环境的概念，就好像有很多房间，每个房间可以有不同装饰，拥有自己个性。

### virtualenv

#### 安装

{{< highlight bash >}}
$ pip install virtualenv
{{< / highlight >}}

#### 创建环境

{{< highlight bash >}}
$ virtualenv ENV # 创建名为‘ENV‘的虚拟环境
$ New python executable in ENV/bin/python
$ Installing setuptools, pip...done.
{{< / highlight >}}

这样就成功创建好一个 python 的虚拟环境，实际上他为你建立了三个目录：

* bin
* include
* lib

这里面包含一个 Python 环境（默认为系统 Python 的版本），一些基本工具（如 easy_install 和 pip）以及一些 symbolic link。而以后的 package 都会装到`lib/pythonX.X/site-packages`里面。

#### 激活环境

使用虚拟环境之前必须激活，不然 package 还是会装到系统的`site-packages`里面。

{{< highlight bash >}}
$ source bin/activate

{{< / highlight >}}

激活成功后，prompt 左边会出现一个括号，里面对应就虚拟环境的名字。这个时候就代表激活成功了，而此时用 pip 安装任何包都会装到虚拟环境的`site-packages`，对其他项目和系统 Python 环境都完全不影响。

#### 注销环境

如果想退出虚拟环境，也很简单。

{{< highlight bash >}}
$ deactivate
{{< / highlight >}}

#### 更多设置

请参考 virtualenv 的[官方文档](http://www.virtualenv.org/en/latest/index.html)

### virtualenvwrapper

当 virtualenv 上手之后，我觉得效率提高很多。可是随着虚拟环境数目增多又遇到一个问题。每次切换环境的过程显得非常繁琐：`deactivate`一个环境，cd 到另外一个目录，`source bin/activate`。virtualenvwrapper 正是为了解决这个问题出现：集中管理多个虚拟环境，快速切换。（支持**tab completion**!）

下面用一个动图来显示它的强大。（假设我现在有两个环境：`demo_env1`和`demo_env1` ）

![Demo](/assets/images/virtualenvwrapper_demo.gif)

#### 安装 & 设置

同样使用 pip 安装

{{< highlight bash >}}
$ pip install virtualenvwrapper
{{< / highlight >}}

环境变量设置。把下面几行加入`.bashrc`或`.bash_profile`

{{< highlight bash >}}
export WORKON_HOME=$HOME/.virtualenvs # 放所有虚拟环境的地方
export PROJECT_HOME=$HOME/Devel # 放所有项目的地方
source /usr/local/bin/virtualenvwrapper.sh
{{< / highlight >}}

#### 使用

`mkvirtualenv`命令创建一个新的虚拟环境

{{< highlight bash >}}
$ mkvirtualenv mynewenv
New python executable in mynewenv/bin/python
Installing distribute.............................................
..................................................................
..................................................................
done.
(mynewenv)$ workon
mynewenv
(mynewenv)$
{{< / highlight >}}

这样会在`WORKON_HOME`也就是默认的`~/.virtualenvs`下创建一个名为`mynewenv`的 virtualenv。我不喜欢这种工作模式，因为对于一个把所有项目都放在单独一个目录的人来说（比如`~/Projects`），我不想让`virtualenvwrapper`打破原有的习惯。还好`virtualenvwrapper`提供了不错的项目管理功能。

{{< highlight bash >}}
$ mkproject mynewnenv
New python executable in mynewnenv/bin/python
Installing setuptools, pip...done.
Creating ~/Devel/mynewnenv
Setting project for mynewnenv to ~/Devel/mynewnenv
(mynewnenv)~
{{< / highlight >}}

这样会在`WORKON_HOME`创建虚拟环境，在`PROJECT_HOME`创建项目目录。这正是我想要的效果：所有的虚拟环境在一处(`WORKON_HOME`)，所有的项目在另一处（`PROJECT_HOME`）。pip 安装的时候会将所有包装到当前激活的虚拟环境中，就实现了看似分离，实际又是统一的效果。

#### 为已有项目添加虚拟环境

为已存在项目添加虚拟环境也十分容易。(当然前提是已经创建好待添加的虚拟环境才可以绑定)

{{< highlight bash >}}
$ setvirtualenvproject ~/.virtualenvs/mynewenv ~/Devel/myproject

Setting project myproject to ~/Devel/myproject

$ workon myproject
(myproject)~
{{< / highlight >}}

#### 其他常用命令（全部支持 tab completion）

* `workon` 切换到环境
* `deactivate` 注销当前环境
* `lsvirtualenv` 列出所有环境
* `rmvirtualenv` 删除环境
* `cpvirtualenv` 复制环境
* `cdsitepackages` cd 到当前环境的`site-packages`目录
* `lssitepackages` 列出当前环境中`site-packages`内容
* `setvirtualenvproject` 绑定现存的项目和环境
* `wipeenv` 清除环境内所有第三方包

#### 扩展

`virtualenvwrapper`提供了几个`template`以供我们初始化新的项目：

* [virtualenvwrapper.bitbucket](https://bitbucket.org/dhellmann/virtualenvwrapper.bitbucket)
* [virtualenvwrapper.django](https://bitbucket.org/dhellmann/virtualenvwrapper.django)
* [virtualenvwrapper.github](https://github.com/sfdye/virtualenvwrapper.github)

这里详细介绍一下`virtualenvwrapper.github`，因为我对这个项目小有贡献。这个插件的作用是，当创建一个`virtualenvwrapper`的项目的时候，可以从指定的 Github 位置导入初始代码。

##### 安装

可以通过

{{< highlight bash >}}
$ pip install virtualenvwrapper.github
{{< / highlight >}}

来获取这个插件。但由于原作者没能继续维护这个项目，我建议直接用我的 fork，下载[`github.py`][github.py]，复制到`site-packages`下的`virtualenvwrapper`目录即可。

[github.py]: https://raw.github.com/sfdye/virtualenvwrapper.github/master/virtualenvwrapper/github.py

##### 使用方法

首先确保这个 repo 的链接存在：`git@github.com:your_github_username/my_repo.git`

{{< highlight bash >}}
$ export VIRTUALENVWRAPPER_GITHUB_USER=your_github_username # 设置环境变量
$ mkproject -t github my_repo
{{< / highlight >}}
