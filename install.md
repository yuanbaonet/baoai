## 项目前端 BaoAIFront 安装步骤



需要安装 [Node.js](https://nodejs.org) 



```shell

# 安装 bower:

npm install -g bower



# 安装 gulp

npm install -g gulp



# npm 安装第三方js

bower install



# npm 安装依赖库:

npm install



# 运行前端代码方式一：自带数据模拟API，适合前端工程师

gulp server



# 运行前端代码方式二：Python全栈开发工程师

gulp serve



# 运行前端代码方式三：Python全栈开发工程师，反向代理(前后端共用相同地址和端口，仅目录不同)

gulp proxy



# 构建生产代码

gulp build



# 运行前端代码方式四：测试运行生产代码

gulp prod



```

生产代码保存在 `dist` 目录.



## 项目后端 BaoAIBack 安装步骤



需要 [Python 3.6](http://www.python.org) 



```shell

# 1. 创建虚拟环境

# windows, 假设项目根路径：d:/baoai/BaoaiBack/

cd d:/baoai/BaoaiBack

mkdir venv

cd venv

python -m venv .



# 运行虚拟环境

d:/baoai/BaoaiBack/venv/Scripts/activate.bat

cd d:/baoai/BaoaiBack



# linux, 假设项目根路径：/baoai/BaoaiBack/

cd /baoai/BaoaiBack

mkdir venv

cd venv

python -m venv .



# 运行虚拟环境

source /baoai/BaoaiBack/venv/bin/activate

cd /baoai/BaoaiBack



# 2. 安装依赖库(必须处于虚拟环境)

# windows 安装依赖库

python -m pip install --upgrade pip

pip install -r requirements.txt

# 如果下载速度慢可以采用国内镜像

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt



# linux 安装依赖库

python -m pip3 install --upgrade pip

pip3 install -r requirements.txt

# 如果下载速度慢可以采用国内镜像

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt



# 3. 运行 Restful 服务

# windows

run_baoai.bat



# linux

# 默认使用gunicorn做为wsgi

chmod +x run_baoai.sh

./run_baoai.sh



# 4. 运行 www 服务(Jinja模块)

# windows

run_www.bat



# linux

chmod +x run_www.sh

./run_www.sh



# 常用功能

# 清空缓存

python manage.py clean

```

## 项目后端数据库



本项目支持绝大部门流行的关系数据库，包括：SQLite、MySQL、Postgres、Oracle、MS-SQL、SQLServer 和 Firebird。



已提供Sqlite数据库，和MySQL数据脚本文件。MySQL支持5.5及以上版本。



数据库转换无需修改代码，仅修改config.py中的SQLALCHEMY_DATABASE_URI即可。



默认使用sqlite数据库，优点是无需安装专门数据库软件，方便测试开发，生产部署请使用mysql或其它数据库软件。



sqlite数据保存在 `db/baoai.db`，直接使用。



mysql数据库脚本保存在 `db/baoai.mysql.sql`，需要新建数据库如baoai，然后导入脚本。



如果使用其他数据库，可以使用`Navicat Premium`工具菜单中的`数据传输`，进行不同数据库之前的数据迁移。



数据库相关操作：

```

# 数据迁移服务

# 初始化

python manage.py db init



# 模型迁移

python manage.py db migrate



# 数据库脚本更新（操作数据）

python manage.py db upgrade

```



## 项目代码自动产生模块



使用自动代码产生模块，可以使字段、模型、生成数据库、前端代码、后端代码和权限配置一并可视化完成，一般项目可以零代码实现。

该部份主要包括三个扩展模块： 数据迁移模块、自动代码模型模块和自动代码产生模块