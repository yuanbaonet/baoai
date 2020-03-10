建议部署在Linux环境

## BaoAI Front前端部署
- nginx 做为 Web前端服务器

- 反向代理，用户无需访问后端API服务地址

如：原API服务地址：localhost:8000/api

反向代理：localhost:3000/api 代替 localhost:8000/api

- URL地址采用HTML5Mode模式

如正常模式： http://www.baoai.co/#/web/main 

HTML5Mode模式：http://www.baoai.co/web/main 

CentOS 7 例子：
```
# 显示中文

export LANG=zh_CN.GB2312

# 添加CentOS 7 EPEL仓库
yum install epel-release
```

1.构建生产代码：

```

# BaoAIFront项目根目录

gulp build

```

将dist目录下的文件复制到centos的/baoai/BaoAIFront目录

2.安装nginx

```

# 现在Nginx存储库已经安装在您的服务器上，使用以下yum命令安装Nginx

yun install -y nginx

# 启动Nginx

systemctl start nginx
# 如果想在系统启动时启用Nginx

systemctl enable nginx
```

3.修改配置文件nginx.conf

nginx.conf默认在 /etc/nginx/nginx.conf
```

  upstream node {
      server 127.0.0.1:8000;
  }
  server {
    listen 80;
    server_name _;
    access_log /data/wwwlogs/access_nginx.log combined;
    root /baoai/BaoAIFront;
    index index.html index.htm index.php;
        if ($request_uri ~ ^/(static|login|register|password|app|api)) {
                set $cookie_nocache 1;
        }
        location / {
            index  index.html index.htm index.php;
            try_files $uri $uri/ /index.html =404;
            # proxy_cache cache;
            # proxy_cache_valid   200 304 12h;
            # proxy_cache_valid   any 10m;
            # add_header  Nginx-Cache "$upstream_cache_status";
            # proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            # proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
            # proxy_no_cache $http_pargma $http_authorization;
        }

        location ^~ /static/ {
            proxy_pass http://node;
            proxy_set_header Host $http_host;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            # proxy_cache cache;
            # proxy_cache_valid   200 304 12h;
            # proxy_cache_valid   any 10m;
            # add_header  Nginx-Cache "$upstream_cache_status";
            # proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            # proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
            # proxy_no_cache $http_pargma $http_authorization;
        }

        location ^~ /api/ 
            proxy_pass http://node;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;          
            # proxy_cache cache;
            # proxy_cache_valid   200 304 12h;
            # proxy_cache_valid   any 10m;
            # add_header  Nginx-Cache "$upstream_cache_status";
            # proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            # proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
            # proxy_no_cache $http_pargma $http_authorization;
        }
    #error_page 404 /404.html;
    #error_page 502 /502.html;
    location /nginx_status {
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
    }

    # location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
    #   expires 30d;
    #   access_log off;
    # }
    # location ~ .*\.(js|css)?$ {
    #   expires 7d;
    #   access_log off;
    # }
    location ~ ^/(\.user.ini|\.ht|\.git|\.svn|\.project|LICENSE|README.md) {
      deny all;
    }
  }
  ```

4.重启nginx

systemctl restart nginx

## BaoAI Back后端部署

- 复制源码至服务器

- 安装mariadb【mysql】数据库

- 安装Python环境

   + 安装 Python3.6.5

   + 创建虚拟环境 venv

   + 虚拟环境中安装依赖包

- gunicorn 做为 wsgi服务器

- supervisor 做为管理和守护程序

CentOS 7 例子：

```
# 显示中文
export LANG=zh_CN.GB2312

# 添加CentOS 7 EPEL仓库
yum install epel-release
```

1.将BaoAIBack源码复制到/baoai/BaoAIBack

2.安装mariadb【mysql】

注：MariaDB 数据库管理系统是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可。开发这个分支的原因之一是：甲骨文公司收购了 MySQL 后，有将 MySQL 闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB完全兼容mysql，使用方法也是一样的

```
# 安装mariadb-server
yum install mariadb-server

# MariaDB服务开启，并设置为开机启动
systemctl start mariadb  # 开启服务
systemctl enable mariadb  # 设置为开机自启动服务

# 数据库的配置
mysql_secure_installation
Enter current password for root (enter for none):  # 输入数据库超级管理员root的密码(注意不是系统root的密码)，第一次进入还没有设置密码则直接回车

Set root password? [Y/n]  # 设置密码，y

New password:  # 新密码
Re-enter new password:  # 再次输入密码

Remove anonymous users? [Y/n]  # 移除匿名用户， y

Disallow root login remotely? [Y/n]  # 拒绝root远程登录，n，不管y/n，都会拒绝root远程登录

Remove test database and access to it? [Y/n]  # 删除test数据库，y：删除。n：不删除，数据库中会有一个test数据库，一般不需要

Reload privilege tables now? [Y/n]  # 重新加载权限表，y。或者重启服务也可

# 登录MariaDB [mysql]

[root@mini ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>

# 创建数据库
MariaDB [(none)]>create database baoai;

# 进入数据库
MariaDB [(none)]>use baoai;

# 导入脚本
MariaDB [(none)]>source /baoai/BaoAIBack/db/baoai.sql;

3.安装Python 3.6
yum install -y python36 python36-devel

# 查看版本
python3 -V
pip3 -V
```

4.创建虚拟环境

```

# 进入项目目录

cd /baoai/BaoAIBack/

mkdir venv

cd venv

python3 -m venv .

运行
source /baoai/BaoAIBack/venv/bin/activate

# 查看版本
python -V

# 升级pip
pip3 install --upgrade pip
```

5.依赖包导入

```
pip install  -r requirements.txt

# 如果速度慢，可以使用国内镜像
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt
```

6.测试运行
(venv) /baoai/BaoAIBack>python manage.py runserver --threaded

5.gunicorn运行项目
./run_baoai.sh

6.安装supervisor

```
# systemctl enable supervisord # 开机自启动
# systemctl start supervisord # 启动supervisord服务

# systemctl status supervisord # 查看supervisord服务状态
# ps -ef|grep supervisord # 查看是否存在supervisord进程
```

7.加入baoai程序

```
cp /baoai/BaoAIBack/deploy/baoai.ini /etc/supervisord.d/

重启supervisor
supervisorctl restart all 或 supervisorctl restart baoai

查看状态
supervisorctl status

停止应用
supervisorctl stop all 或 supervisorctl stop baoai
```

8.关闭所有python应用
```
./kill_python.sh
```



