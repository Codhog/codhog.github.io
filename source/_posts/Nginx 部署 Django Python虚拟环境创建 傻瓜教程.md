---
title: Nginx 部署 Django Python虚拟环境创建 傻瓜教程
date: 2021-03-21 08:29:03
tags:
    - my CSDN blog
    - tutorial
---
#

这里不讨论uwsgi 与 nginx之间的关系，但是建议学习

通俗说，Nginx就是可以让你的网页支持更多请求时保证负载均衡，

简单的网页用uwsgi部署配合django runserver也可以达到要求，所以从负载能力是

**Nginx > uwsgi > 本地 django runserver**

（你的服务器需要先安装好Python 和 pip 和 nginx 和 开启环境后安装uwsgi）- 这个步骤先于第一条

项目进入部署状态 ALLOWED_HOST = ['*'] 

DEBUG=False 之类的

## 1. 首先你需要在服务器配置虚拟环境 virtualenv 

注意(只有**python**2.7及更高版本才支持virtualenv)

如果是Python 2

导航到你的项目文件夹后

```
virtualenv -p /path/to/new/virtual/environment venv
```

如果是Python 3

```
python3 -m venv /path/to/new/virtual/environment
```

运行这个指令会建造一个新文件夹 environment

并且该虚拟环境被取名为 venv

被创建的这个新文件夹里面会有三个文件夹，其中bin是以后我们启动虚拟环境的起点目录

如果需要删除该虚拟环境，直接**删除该虚拟环境**的根目录就行

如下： 使用source 指令开启 使用deactivate [venv名字] 关闭  如果要关闭初始[base]环境 使用conda deactivate

```
source /path/to/new/virtual/environment/bin/activate
```

注意 一定要记得给虚拟环境安装uwsgi

```
pip install uwsgi
```

接下来 同样重要的是给虚拟环境安装 配置文件 (就是pycharm里面虚拟环境安装的给你项目用的东西)

cd 到**本地**项目文件夹 或者 pycharm terminal里面输入

```
pip freeze > requirements.txt
```

如果有问题 (我出现了 -ip 24 的包出现的问题 百度上搜索 删除即可) 你需要对每一个出现问题的包进行查询

然后检查requirements.txt的格式

大概是这样  包==版本号 必须严格相同 pip freezelist 那个不行 

```
asgi-redis==1.4.3
asgiref==3.3.4
asttokens==2.0.4
async-timeout==2.0.1
```

然后 scp -r 上传本地项目 至 服务器

然后在服务器上  启动虚拟环境 检查pip版本 python 版本

```
source /path/to/new/virtual/environment/bin/activate
```

然后 安装依赖  (这个步骤可能会有坑 主要都是网域问题 所以Pycharm用的时候不能偷懒 必须一个项目一个环境

不然太多包总有网域不支持的包版本  尝试对出问题的包单独使用镜像 设置http为safe source等

csdn上别人写的太详细了我不好意思抄)

```
pip install -r requirements.txt
```

如果出现

```
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, '[SSL: UNKNOWN_PROTOCOL] unknown protocol 
```

是因为pip version==20.3.3(最新版本)已经完全使用SSL认证, 如果服务器是http, 考虑降级pip至20.2.2

## 2. 配置uwsgi.ini

在django项目的根目录下 (与manage.py同级目录)

测试下uwsgi连接

先建立个py测试文件

```
# /path/to/django/project/test.py  
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

cd到 /path/to/django/project/ 命令行中输入以下

```
uwsgi --http :8001 --wsgi-file test.py
```

浏览器输入你服务器网址加上端口8001 如果在页面上看到 Hello World 说明uwsgi配置成功

如果提示Internal Server Error

1. 检查uwsgi的版本 看是否安装到了虚拟环境中
2. 检查test.py是否运行的是正确的那一个

接下来正式配置Django项目的uwsgi.ini 文件, 启动这个文件要修改一些之前的命令 因为用的入口是module

```
uwsgi --ini uwsgi.ini
```

新建uwsgi.ini文件

```
[uwsgi]

socket = 127.0.0.1:6667
chdir = /path/to/django/project/[你的django项目名]
module = [你的django项目名].wsgi
master = true         
processes = 10
threads = 2
max-requests = 2000
chmod-socket = 664
pidfile= /path/to/django/project/[你的django项目名]/[你的django项目名].pid
vacuum = true
py-autoreload = 1
daemonize = /path/to/django/project/[你的django项目名]/log/uwsgi.log

```

常用选项：

http ： 协议类型和端口号

processes ： 开启的进程数量

workers ： 开启的进程数量，等同于processes（官网的说法是spawn the specified number ofworkers / processes）

chdir ： 指定运行目录（chdir to specified directory before apps loading）

wsgi-file ： 载入wsgi-file（load .wsgi file）

stats ： 在指定的地址上，开启状态服务（enable the stats server on the specified address）

threads ： 运行线程。由于GIL的存在，我觉得这个真心没啥用。（run each worker in prethreaded mode with the specified number of threads）

master ： 允许主进程存在（enable master process）

daemonize ： 使进程在后台运行，并将日志打到指定的日志文件或者udp服务器（daemonize uWSGI）。实际上最常用的，还是把运行记录输出到一个本地文件上。 (**需要自己手动建立log文件夹)** 否则报错

pidfile ： 指定pid文件的位置，记录主进程的pid号。

vacuum ： 当服务器退出的时候自动清理环境，删除unix socket文件和pid文件（try to remove all of the generated file/sockets）

```
uwsgi --ini uwsgi.ini
```

## 3.创建nginx.conf配置文件

同样是在django项目的根目录下 (与manage.py同级目录)

这是我的配置

```nginx
events {
    worker_connections  1024;
}

http {
    include       /usr/local/nginx/conf/mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #gzip  on;
    server {
        listen 10110;
        server_name localhost;
        charset     utf-8;
        access_log      /path/to/django/project/[你的django项目名]/log/nginx_access.log;
        error_log       /path/to/django/project/[你的django项目名]/log/nginx_error.log;
        large_client_header_buffers 4 16k;     # 读取大型客户端请求头的缓冲区的最大数量和大小
        client_max_body_size 10000m;             #设置nginx能处理的最大请求主体大小。
        client_body_buffer_size 128k;          #请求主体的缓冲区大小。
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        proxy_buffer_size 64k;
        proxy_buffers   4 32k;
        proxy_busy_buffers_size 64k;
        proxy_temp_file_write_size 64k;


        location /static {
                alias /path/to/django/project/[你的django项目名]/static; # 新静态文件收集地址
        }

        location / {
                uwsgi_send_timeout 600;        # 指定向uWSGI传送请求的超时时间，完成握手后向uWSGI传送请求的超时时间。
                uwsgi_connect_timeout 600;
                uwsgi_read_timeout 600;
                include     /usr/local/nginx/conf/uwsgi_params;
                uwsgi_pass  127.0.0.1:8208;
        }
    }
}
```

简单一点

```nginx
listen 88;　　　　　　　　　　　　　　　 # 区别于uwsgi设置的端口
server_name www.wcwnina.com;　　　　　# 记得在系统的/etc/hosts文件中添加IP与域名的映射！

location / {
    include uwsgi_params;　　　　　　　# 与nginx.conf同目录
    uwsgi_pass 192.168.1.2:8000;　　 # 与uwsgi配置中的socket一致
}

location /static {
    alias /path/to/django/project/[你的django项目名]/static;
}
```

创建(修改)好后 重启nginx

```
nginx -s reload
```



## 4. 收集静态文件

同样是在django项目的根目录下 (与manage.py同级目录)

```
python manage.py collectstatic
```



## 5. 再次运行uwsgi

同样是在django项目的根目录下 (与manage.py同级目录)

```
uwsgi --ini uwsgi.ini
```

如果碰到了问题，需要到项目目录的log文件夹里查看nginx_error.log

可以用 以下 看最后10行 和 新增的信息 然后根据 具体问题 具体百度

```
tail -f nginx_error.log
```

本文对pip 安装 requirements 以及安装其他包 会碰到的问题省略了 

可以从 python nginx pip 版本   网域代理 pip代理 方面下手 (再难的我也不会了...)

希望 你的过程能一帆风顺 否则 具体问题 具体百度

如果遇到问题需要重启 查找该进程的pid  用kill-9 关闭  然后再次尝试

ps -ef | grep [项目名]或[:端口号]    ps: 看到端口号前的 **:** 了么

lsof -i :端口号

kill -9 [pid]
