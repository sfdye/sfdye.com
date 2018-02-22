---
categories:
- Tech
date: 2014-03-09T00:00:00Z
header:
  image: /assets/images/django-demo.png
last_modified_at: 2014-03-15 13:43:01 +08:00
tags:
- Django
- Best practice
- Nginx
- Gunicorn
- Supervisor
- Ubuntu
- Centos
title: Django最佳实践与部署：Nginx + Gunicorn + Supervisor（Ubuntu和CentOS）
---

## 题头

头图是我一个Django项目--[Santa](http://ntusanta.com)的代码片段，放出来给大家欣赏一下。

第一篇很技术的文章。Mrxu同学强烈要求我出个教程，基于Django的部署实在有太多坑，为后人铺条路。以下文字是我摸爬滚打无数小时换来的一个可行方案。希望对大家有所帮助。

## Django最佳实践

### 项目结构
相信很多朋友在使用Django的时候都会遇到这个问题，项目到底如何组织？只有一个app的时候不要紧，有两个、三个甚至多个app的时候，*模板*（templates）要放在那里，*静态文件*（static files）放在哪里？由于Django社区（不像Rails）一直没有统一说法，所以此处说的只不过是仁者见仁，智者见智。希望Django早日出台官方的最佳实践。这里仅给大家一个参考。

下面我用一个myproject项目为大家阐述。这个项目由两个app组成：

* 每个app有自己的*static*目录
* 每个app有自己的*templates*目录
* 根目录下与项目同名的文件夹（比如这里的*myproject*）为项目设置，包含*settings.py*
* 根目录下有*requirements.txt*

为什么要这么做？这个在后面生产境部署的时候优势就会显现出来。

{{< highlight bash >}}
$ django-admin.py startproject myproject
$ cd myproject
$ python manage.py startapp myapp1
$ python manage.py startapp myapp2  
$ touch requirements.txt                 # 新建requirement.txt
$ mkdir myapp1/{static,templates}        # 新建两个空文件夹
$ mkdir myapp2/{static,templates}
  $ tree .                               # 非常有用的一个命令, 图形化显示文件目录结构

.
├── manage.py
├── myapp1
│   ├── __init__.py
│   ├── admin.py
│   ├── models.py
│   ├── static
│   ├── templates
│   ├── tests.py
│   └── views.py
├── myapp2
│   ├── __init__.py
│   ├── admin.py
│   ├── models.py
│   ├── static
│   ├── templates
│   ├── tests.py
│   └── views.py
├── myproject
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── requirements.txt

7 directories, 16 files
{{< / highlight >}}

### 用requirements.txt来组织项目依赖包

如果你使用了大量第三方Django插件，如何方便管理？有没有办法一次pip全部安装？（试想想在生产环境的每一个服务器上手动安装一百个依赖包）这就是`requirements.txt`的作用。`requirements.xt`是一个txt文本，格式为`依赖包名称==依赖包版本`，一行一个。

{{< highlight bash >}}
# myproject/requirements.txt
Django==1.6.2
MySQL-python==1.2.4
BeautifulSoup==4.3.2
requests==1.2.3
pytz==2012d

{{< / highlight >}}

安装所有依赖包只需要

{{< highlight bash >}}
$ pip install -r requirements.txt
{{< / highlight >}}

### 创立虚拟环境Virtualenv

想像在一台服务器上你同时有10个Django项目，不同项目需要完全不一样的依赖包。比如A项目不需要gunicorn，B项目不需要用requests，虚拟环境就能帮你解决这个问题。简单的说，虚拟环境就一个大文件夹，里面放着这个项目所有东西，Python环境，pip包，静态文件，代码。这样的好处就是不同项目可以使用不同版本的Python，不同依赖包，相互独立不受其他项目影响。

安装Virtualenv

{{< highlight bash >}}
$ pip install virtualenv
{{< / highlight >}}

创建一个虚拟环境

{{< highlight bash >}}

$ mkdir webapps
$ cd webapps
$ virtualenv myproject   # 安装虚拟环境，这里myproject是虚拟环境的名字，可以改成别的以却分
$ New python executable in myproject/bin/python
$ Installing setuptools, pip...done.
$ cd myproject
$ source bin/activate    # 激活虚拟环境
(myproject) $            

{{< / highlight >}}

到这里看到括号里是项目名称就代表成功激活虚拟环境了，之后pip安装都会装到虚拟环境（bin）而不是装到全局系统里。

如果要使用其他版本Python（而不是系统默认）

{{< highlight bash >}}
$ virtualenv --python=path_to_your_python myproject
{{< / highlight >}}

可以用这个命令查看当然默认Python的位置

{{< highlight bash >}}
$ which python
{{< / highlight >}}

附： [Virtualenv的官方文档](http://www.virtualenv.org/en/latest/index.html)


### 如何做到更好（更模块化）

以上说的方法很简单，一个项目一个`settings.py`，本地使用时没有问题。到了生产环境，问题来了。Debug不能再设为True（这个选项是为了调试方便，如果在生产环境打开的话就有种"裸奔“的感觉，因为任何人都可以看到你的环境变量，文件目录等等），静态文件，模板文件的路径不能设为相对路径，等等。怎么处理开发环境和生产环境设置的不同呢？

#### 方法一（简单，不推荐）

把`settings.py`加入`.gitignore`，本地和服务器放两个不同版本的`settings.py`。

**[更新]**：只用一个settings.py这个方法有个弊端，每次增加app需要手动更新`INSTALLED_APPS`（或者修改其他地方）。谢谢MrXu指出。

#### 方法二（可以用，推荐）

这个方法是Django book上的推荐的。原理就是通过hostname来判断当前环境是本地还是服务器，从而做出不同配置。

{{< highlight python >}}
# settings.py

import socket

if socket.gethostname() == 'my-laptop':
    DEBUG = TEMPLATE_DEBUG = True
else:
    DEBUG = TEMPLATE_DEBUG = False

# ...
{{< / highlight >}}

#### 方法三（最模块化，强烈推荐）

我在网上看到的一个做法，原理跟Ruby on Rails很像，那就是不同环境用不同的配置文件。感觉这种方法不久会被Django官方采用。

`dev.py`对应开发环境（*development*），`prod.py`对应生产环境(*production*)，而`test.py`对应测试环境。（根据需要还可以加入`staging.py`）

{{< highlight bash >}}
$ rm myproject/settings.py              # 删除原来的settings.py
$ mkdir myproject/settings
$ touch myproject/settings/{__init.py,dev.py,prod.py,test.py}
{{< / highlight >}}

注：如果在生产环境使用了不同的`settings.py`，在syncdb或其他操作的时候需要指明它的位置。（这一点又跟rails一样）以`syncdb`为例：

{{< highlight bash >}}
$ python manage.py syncdb --settings=myproject.settings.prod
{{< / highlight >}}

另外一个方法是修改`manage.py`中的`DJANGO_SETTINGS_MODULE`，但这个方法可用性不高，故不推荐。特别是后文中使用一个bash script自动控制gunicorn，就完全没有修改`manage.py`的必要。

同理，对于`requirements.txt`也可以模块化：
`dev.txt`，`prod.txt`，`test.txt`，再加上一个`common.txt`用来放共享的依赖包。   

{{< highlight bash >}}
$ rm requirements.txt                   # 删除原来的requirements.txt
$ mkdir requirements
$ touch requirememts\{common.txt,dev.txt,prod.txt,test.txt}
{{< / highlight >}}

最后的效果

{{< highlight bash >}}
.
├── manage.py
├── myapp1
│   ├── __init__.py
│   ├── admin.py
│   ├── models.py
│   ├── static
│   ├── templates
│   ├── tests.py
│   └── views.py
├── myapp2
│   ├── __init__.py
│   ├── admin.py
│   ├── models.py
│   ├── static
│   ├── templates
│   ├── tests.py
│   └── views.py
├── myproject
│   ├── __init__.py
│   ├── settings
│   │   ├── __init__.py
│   │   ├── dev.py
│   │   ├── prod.py
│   │   └── test.py
│   ├── urls.py
│   └── wsgi.py
└── requirements
    ├── common.txt
    ├── dev.txt
    ├── prod.txt
    └── test.txt

9 directories, 22 files
{{< / highlight >}}

## Django部署：Nginx + Gunicorn + Supervisor

这部分假设你已经有一台虚拟主机（VPS），Digital Ocean也好，Amazon EC2也好，或者是阿里云。（Webfaction用户可以跳过VPS设置部分）操作系统方面我会尽量涵盖Ubuntu 12.04 LTS（Debian）和CentOS 6（RHEL）。

### 修改root密码
第一步，用ssh登陆服务器，修改root。

{{< highlight bash >}}
$ ssh root@12.34.56.78    # 这里换成你VPS的ip
$ passwd
{{< / highlight >}}

### 创建用户，给sudoer权限
我们不想用root用户进行日常操作，一不小心`rm -rf`后果不堪设想。所以可以创建一个普通拥有管理员权限的账号。

{{< highlight bash >}}
$ adduser demo             # 换成你自己的用户名
$ passwd demo              # 为demo用户设置密码
{{< / highlight >}}

现在我们来给demo用户管理员权限。运行下面命令会打开`/etc/sudoers`这个文件，如果在Ubuntu则会用默认的文本编辑器nano打开。

{{< highlight bash >}}
$ visudo
{{< / highlight >}}

如果不想用nano，可以改用vi

{{< highlight bash >}}
$ vi /etc/sudoers
{{< / highlight >}}

找到这一行

{{< highlight bash >}}
# User privilege specification
root    ALL=(ALL:ALL) ALL
{{< / highlight >}}

紧接着在下面加上

{{< highlight bash >}}
demo    ALL=(ALL:ALL) ALL   # 给demo用户分配sudoer权限
{{< / highlight >}}

保存退出，重新登陆VPS。这个时候用我们刚才创建的用户。

{{< highlight bash >}}
$ ssh demo@12.34.56.78
{{< / highlight >}}

### 系统更新
{{< highlight bash >}}
# Ubuntu 
$ sudo apt-get update
$ sudo apt-get -y upgrade
# CentOS
$ sudo yum -y update
{{< / highlight >}}

### 安装基础依赖包
{{< highlight bash >}}

# Ubuntu 
$ sudo apt-get install -y build-essential
$ sudo apt-get install python-dev

# CentOS
$ sudo groupinstall -y 'development tools'
$ sudo yum install -y zlib-dev openssl-devel sqlite-devel bzip2-devel
{{< / highlight >}}

### 安装pip, virtualenv
{{< highlight bash >}}
# Ubuntu
$ sudo apt-get install python-pip
$ sudo pip install virtualenv

# CentOS
$ sudo yum install python-pip
$ sudo pip install virtualenv

{{< / highlight >}}

### 安装Git（或者其他版本管理工具）
{{< highlight bash >}}

# Ubuntu
$ sudo apt-get install git-core

# CentOS
$ sudo yum install git-gore
{{< / highlight >}}

### Checkout项目源文件和安装项目依赖包
这里我推荐一个做法：在你的home根目录下创建一个文件夹`webapps`用来存放所有的项目。

{{< highlight bash >}}
$ cd ~                                    # 切换到home，这里是/home/demo
$ mkdir webapps
$ virtualenv myproject                    # 创建虚拟环境
$ cd myproject                           
$ source bin/activate                     # 激活虚拟环境
(myproject)$ git clone https://github.com/sfdye/myproject  # 这里换成你项目的地址
(myproject)$ pip install -r myproject/requirements.txt    

{{< / highlight >}}

### 安装及配置Database
这里以MySQL为例，如果使用其他数据库例如PostgreSQL，方法类似。

{{< highlight bash >}}
# Ubuntu
$ sudo apt-get install mysql-server
$ sudo service mysqld start

# CentOs
$ sudo yum install mysql-server
$ sudo service mysqld start

{{< / highlight >}}

设置MySQL root密码

{{< highlight bash >}}
$ sudo /usr/bin/mysql_secure_installation
$ Enter current password for root (enter for none): 
  OK, successfully used password, moving on...
{{< / highlight >}}

创建Database

{{< highlight bash >}}
$ mysql -u root -p			# 按提示输入root密码

{{< / highlight >}}

{{< highlight mysql >}}
mysql> create database myproject;			# 创建一个空的db，方便等会syncdb
Query OK, 1 row affected (0.00 sec)

mysql> quit
Bye

{{< / highlight >}}

### 项目设置 *settings.py*

根据情况修改不同的文件（例如：`production.py`）。这里说一下需要修改的地方。

{{< highlight python >}}
# settings.py

DEBUG = False 			# 一定要改为False，不然任何人都能看到你Django的出错页面
										
TEMPLATE_DEBUG = DEBUG

ADMINS = (
	('Your Name', 'your_email@example.com'),   # 这里填上你的名字和邮箱
)

MANAGERS = ADMINS

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'myproject',                      
        'USER': '',                 # 填上数据库的用户 
        'PASSWORD': '',             # 填上数据库的密码   
        'HOST': '',                 # 默认则不用填写
        'PORT': '',                 # 默认则不用填写    
    }
}

# 域名白名单
ALLOWED_HOSTS = ["example.com"]     # 当DEBUG是False的时候，只有这里
                                    # 设置过的域名才允许访问我们的项目
                                    # 为的是防止CSRF攻击

# 静态文件设置
STATIC_ROOT = '/home/demo/webapps/myproject/static'
STATIC_URL  = 'http://example.com/static/'       # example.com换成你的域名

STATICFILES_DIRS = (
    '/home/demo/webapps/myproject/myproject/app1/static/',   # 这里对应我们上面创建的2个app
    '/home/demo/webapps/myproject/myproject/app2/static/'    
)

# 用户上传文件设置
MEDIA_ROOT = '/home/demo/webapps/myproject/media'
MEDIA_URL  = 'http://example.com/media'	 # example.com换成你的域名


{{< / highlight >}}

需要注意的几点：

* `DEBUG`和`TEMPLATE_DEBUG`一定要改成`False`
* 一定要把你的域名加入`ALLOWED_HOSTS`
* `STATIC_ROOT`和`MEDIA_ROOT`一定要用绝对路径
* `STATIC_URL`和`MEDIA_URL`要改成完整地址

#### 创建Database Schema

这一步在刚才创建的数据库`myproject`里面加入Schema。

superuser对应的是Django的admin功能，如果你需要后台管理界面（类似phpmyadmin）请创建一个。

{{< highlight bash >}}
$ python manange.py sycndb

Creating tables ...
Creating table django_admin_log
Creating table auth_permission
Creating table auth_group_permissions
Creating table auth_group
Creating table auth_user_groups
Creating table auth_user_user_permissions
Creating table auth_user
Creating table django_content_type
Creating table django_session

You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): yes
Username (leave blank to use 'demo'):
Email address:
Password:
Superuser created successfully.
Installing custom SQL ...
Installing indexes ...
Installed 0 object(s) from 0 fixture(s)

{{< / highlight >}}

到这里数据库的Schema同步好了，如果你使用了South（Django 1.7之前）或者是migration。那么还需要migrate

{{< highlight bash >}}

$ python manange.py migrate

Synchronizing apps without migrations:
  Creating tables...
  Installing custom SQL...
  Installing indexes...
Installed 0 object(s) from 0 fixture(s)
Running migrations:
  Applying xxx... OK

{{< / highlight >}}


#### 收集静态文件
{{< highlight bash >}}
$ python manage.py collectstatic

You have requested to collect static files at the destination
location as specified in your settings:

    /home/demo/webapps/myproject/static

This will overwrite existing files!
Are you sure you want to do this?

Type 'yes' to continue, or 'no' to cancel: yes
...
...
...
xxx static files copied to '/home/demo/webapps/myproject/static'.
{{< / highlight >}}

### Nginx设置

[Nginx](http://nginx.org/)是一个轻量级的反向代理Web Server。配置容易，性能高，特别是在并发性上处理，远超Apache。谢谢@Danyang指出。

#### 安装Nginx

{{< highlight bash >}}
# Ubuntu
$ sudo apt-get install nginx

# CentOs
$ sudo yum install nginx

$ sudo service nginx start  # 启动Nginx服务
nginx: the configuration file /opt/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /opt/nginx/conf/nginx.conf test is successful
Stopping nginx:                                            [  OK  ]
Starting nginx:                                            [  OK  ]
{{< / highlight >}}    

这个时候在浏览器访问12.34.56.78（换成你的static ip），如果看到`Welcome to nginx`就说明成功了。我们继续来配置nginx。

注：nginx之所以可以在安装完后不用配置的情况下显示欢迎页面是因为它自带一个default的配置。这个对新手非常方便，但自己配置的时候，一定要把这个default配置删掉，或者换成其他端口。因为这个默认配置监听的端口80是默认端口，也就是12.34.56.78 -> 12.34.56.78:80。不处理的结果就是，无论你怎么刷新，访问12.34.56.78都只能看到nginx的欢迎界面。当然你也可以为你的app使用其他的端口，比如81。

找到`nginx.conf`的位置。不同安装方法的位置可能不一样。

{{< highlight bash >}}
$ sudo vim /opt/ngxin/conf/nginx.conf
{{< / highlight >}}

加上下面这一行。`sites-enabled`就是我们以后为每个app放配置的地方。

{{< highlight bash >}}
http {

    include /opt/nginx/conf/sites-enabled/*;

}

{{< / highlight >}}

下面的gist是我们myproject的nginx配置。

{% gist 9563322 myproject.nginxconf %}

### Gunicorn配置

[Gunicorn](http://gunicorn.org/)是一个Python的WSGI HTTP Server，与nginx一起使用效果非常好。

{{< highlight bash >}}
$ pip install gunicorn    # 安装gunicorn
{{< / highlight >}}

下面这个gist是一个自动加载虚拟环境并启动gunicorn的一个脚本文件。把它放在虚拟环境的`bin`下。

{% gist 9563322 gunicorn_start.bash %}

### Supervisor配置

到这个时候基本全部配置都完成了。可是不足的是，每次都要手动启动gunicorn，如果退出ssh，或者服务器出错，我们都需要去维护。这里介绍的小工具[Supervisord](http://supervisord.org/)就可以完美的解决这些问题。

安装Supervisord

{{< highlight bash >}}
$ sudo pip install supervisor
$ sudo vim /etc/supervisord.conf    # 修改supervisor配置
{{< / highlight >}}

加入以下gist内容

{% gist 9563322 myproject.conf %}

重启supervisor让配置生效

{{< highlight bash >}}
$ sudo service supervisord restart  
{{< / highlight >}}



### DNS设置

经常听到朋友问

> 我买了一个域名，XXX.com，怎么让访问它的时候自动链接到我的网站呢？
  我现在可以通过12.34.56.78看到网站，怎么样能换成XXX.com呢？

这都属于DNS配置的问题。DNS全称是Domain Name System，它的作用就是把XXX.com这样人可以记住的域名翻译成12.34.56.78这样的IP地址。想了解更多，可以参考知乎上的这个[问题](http://www.zhihu.com/question/22572025)。

下面我们来配置DNS。通常有两种做法:

1. 交给Domain name registar做，比如你的域名在Godaddy买的，那么就在Godaddy上设置
2. 交给VPS来处理（比如Digital Ocean）来做。如果你的不想在Godaddy上设置，或者在一个VPS上有多个应用需要映射到同一个域名

第一种情况，在你购买域名的网站（比如Godaddy），找到DNS Manager。

添加一条A record。第一栏填上子域名，比如(`myproject.your_domain.com`)就写`myproject`，第二栏填VPS的IP地址。
如果想直接使用naked domain（就是`your_domain.com`，前面没有加任何子域名），那么就写`@`。当然这也是需要时间时间生效的，通常是24小时到48小时。（实际可能只需要2到3个小时）

{{< highlight bash >}}
A    myproject   12.34.56.78
A    @           12.34.56.78
  
{{< / highlight >}}

第二种情况，首先把DNS的任务转交给VPS。

同样需要购买域名的网站（比如Godaddy）找到DNS Manager。修改Nameserver。此举一劳永逸，也就是说一次修改，以后就跟Godaddy无关了。全部域名解析都交给Digital Ocean的Nameserver来处理。

改好后效果如下

![Godaddy setup](/assets//images/godaddy.png)

一般需要24小时到48小时来生效。（因为需要在整个互联网传播，实际上通常几个小时就好了）

验证DNS是否修改成功

{{< highlight bash >}}
$ whois your_domain.com
{{< / highlight >}}

如果你看到Nameserver已经改为Digital Ocean的，就说明成功了。

今后修改DNS只需到Digital Ocean的管理页面设置即可。对于每一个app，同样是添加一条A record。

![Digital Ocean DNS配置](/assets//images/do_dns.png) 

<!-- ### VPS配置（虚拟内存, 自动运行mysql） -->

## 工具推荐

### VPS: Digital Ocean
卖点是便宜。每个月最低配只要5美元即可拥有属于自己的VPS。
如果你觉得这篇文章有用，请使用我的[referral](https://www.digitalocean.com/?refcode=8a95d662fb6f)来购买DO。

### [Transmit](https://panic.com/transmit/)
Mac上的FTP/SFTP的工具，支持Amazon S3。配合Sublime Text用适合新手。

### Terminal
ssh直接用原生Terminal即可。一个不错的替代品：[iTerm2](http://www.iterm2.com/)。


## To-Do

* Django的安全
* 更多最佳实践

## 参考文档

* [Django Project Structure](http://www.deploydjango.com/django_project_structure/)
* [Deploying Django](http://www.djangobook.com/en/2.0/chapter12.html)
* [Deployment checklist](https://docs.djangoproject.com/en/1.6/howto/deployment/checklist/)
* [How To Install and Configure Django with Postgres, Nginx, and Gunicorn](https://www.digitalocean.com/community/articles/how-to-install-and-configure-django-with-postgres-nginx-and-gunicorn)
* [How To Deploy a Local Django App to a VPS](https://www.digitalocean.com/community/articles/how-to-deploy-a-local-django-app-to-a-vps)
* [Setting up Django with Nginx, Gunicorn, virtualenv, supervisor and PostgreSQL](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/)
* [Django最佳实践](http://lenciel.cn/django-notes/docs/coding-style/)
