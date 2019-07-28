---
title: python3+nginx+uwsgi+django环境搭建（Centos）
date: 2019-07-27 06:49:10
tags: python, nginx, uwsgi, django
---

# 1、python3环境安装

默认的云服务器上安装python版本现在是2.7的版本。如果新开发的应用，使用了python3，由于语法上的不向后兼容，必须得在服务器上安装python3。

## (1)、安装编译环境包

```
yum install gcc-c++ gcc make cmake zlib-devel bzip2-devel openssl-devel ncurse-devel -y
```

## (2)、下载3.7源代码

```shell
cd ~
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
```

如果wget命令不存在，使用yum安装：
```
yum -y install wget
```

## (3)、解压、配置、安装

```shell
# 解压
tar Jxvf Python-3.7.0.tar.xz
cd Python-3.7.0
# 创建安装目录
mkdir -p /usr/local/python3
#配置（指定安装目录）
./configure --prefix=/usr/local/python3 --enable-optimizations
make && make install
```

在make && make install这一步，有可能出现“ModuleNotFoundError: No module named '_ctypes'”的错误。
此时我们把libffi-devel安装上，然后再执行make && make install即可。

```
yum install libffi-devel -y
```

## (4)、创建python3软链接

```
ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3
```

## (5)、检查

```
python3 -V
```

如果出现3.7.0版本，说明安装成功。

![Python3.7-success](python3-success.png)

# 2、uwsgi安装

```shell
pip3 install uwsgi
#创建uwsgi3的软链接，和python2的区分开来
ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi3
#查看 uwsgi 版本
uwsgi3 --version
```

检测uwsgi是否正常：
新建 /www/test.py 文件，输入如下内容：
```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello uwsgi"]
```

然后在终端运行：
```
uwsgi3 --http :8001 --wsgi-file test.py
```
在浏览器中输入 http://127.0.0.1:8001 , 看看是否有”Hello uwsgi“的字样输出。如果没有看看报错信息，据此来查找解决方案。

# 3、django安装

```shell
pip3 install django
```

测试 django 是否正常，运行：

```shell
mkdir /www
cd /www
django-admin.py startproject demosite
cd demosite
python3 manage.py runserver 0.0.0.0:8002
```
在浏览器内输入：http://127.0.0.1:8002 ,检查django是否运行正常。

# 4、nginx安装

```shell
cd ~
wget http://nginx.org/download/nginx-1.9.9.tar.gz 
tar xzvf nginx-1.9.9.tar.gz 
cd nginx-1.9.9
# 配置（制定安装目录）
./configure --prefix=/usr/local/nginx-1.9.9 --with-http_stub_status_module  --with-http_gzip_static_module
make && make install
```

##（1）uwsgi配置

```shell
[uwsgi]
socket = 127.0.0.1:9090
master = true         #主进程
vhost = true          #多站模式
no-site = true        #多站模式时不设置入口模块和文件
workers = 2           #子进程数
reload-mercy = 10     
vacuum = true         #退出、重启时清理文件
max-requests = 1000   
limit-as = 512
buffer-size = 30000
pidfile = /var/run/uwsgi9090.pid    #pid文件，用于下面的脚本启动、停止该进程
daemonize = /www/uwsgi9090.log
```
## (2) Nginx配置

找到nginx的安装目录（如：/usr/local/nginx/），打开conf/nginx.conf文件，修改server配置：

```shell
server {
        listen       80;
        server_name  localhost;
        
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9090;              #必须和uwsgi中的设置一致
            uwsgi_param UWSGI_SCRIPT demosite.wsgi;  #入口文件，即wsgi.py相对于项目根目录的位置，“.”相当于一层目录
            uwsgi_param UWSGI_CHDIR /www/demosite;       #项目根目录
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }
```

设置完成后，在终端运行：
```shell
uwsgi --ini /etc/uwsgi9090.ini & /usr/local/nginx/sbin/nginx
```
浏览器输入：http://127.0.0.1，你就可以看到 django 的 "It work" 了。