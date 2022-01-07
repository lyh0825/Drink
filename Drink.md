# Drink

## 0.安装准备

### 创建目录

```bash
# 创建目录
mkdir Drink
```

### **了解工具**

- apt-get

Advanced Packaging Tool（apt）是[Linux]下的一款安装包管理工具，是一个客户/服务器系统。

- homebrew

brew 又叫Homebrew，是Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件， 只需要一个命令， 非常方便brew类似ubuntu系统下的apt-get的功能

### **安装mysql**

**查看文件位置**

```bash
which mysql
```

**安装mysql**

```bash
brew install mysql
```

**查看是否安装成功**

```bash
```

**启动mysql**

```bash
# 软连接启动
mysql.server start

# 软连接重启
mysql.server restart

# 软连接终止
mysql.server stop

# 软连接查看
mysql.server status

# root用户登陆, 免密码
mysql -u root -p

# 文件下启动
sudo /usr/local/mysql/support-files/mysql.server start
```

#### 遇到的问题

**1.输入`$ mysql`**

`ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)`

**此问题是mysql服务器没有启动, 需要执行一下操作启动服务器**

```bash
# 1.软连接启动
mysql.server start

# 2.文件启动
which mysql
xxx/xxx/mysql.server start
```

**2.输入`$ mysql`**

`ERROR 1045 (28000): Access denied for user 'edy'@'localhost' (using password: NO)`

**MAC上安装完MYSQL后，MYSQL默认给分配了一个默认密码，在终端上使用默认密码登录时，总会提示一个授权失败的错误**

**解决方案**

现在没法登录到数据库中，改密码和添加用户等操作也无从谈起。好在MySQL中还提供了一种免去密码校验进入数据库的方法，先使用这种方法登入数据库。然后将默认密码替换掉

```bash
# 执行完毕后输入密码
mysql -u root -p 
>password:  # 回车
```

**`-u`** 表示选择登陆的用户名， **`-p`** 表示登陆的用户密码

#### 导入数据

```sql
# 创建数据库zerd
create database zerd

# 进入数据库zerd
use zerd

# 倒入下载的数据,在指定的database中导入
source /xxx/xxx.sql

# 查看所有的tables查看倒入的数据
show tables
```

## 1.配置环境

#### Python3.6

查询本地的python3的版本 `$ python3 --version` 如果不是python3.6的版本

```bash
sudo apt-get update 
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:jonathonf/python-3.6 
sudo apt-get install python3.6
```

#### pipenv

> [Pipenv]是Kenneth Reitz在2017年1月发布的Python依赖管理工具，现在由PyPA维护。你可以把它看做是pip和virtualenv的组合体，而它基于的[Pipfile]则用来替代旧的依赖记录方式（requirements.txt）。

- pipenv==>安装虚拟环境

**首先安装pip3包管理工具**

pip3是python3用来管理包的工具，可以用来安装、升级、卸载第三方库，当然也可以查看包信息，更加可以发布自己所写的包到Pypi（一般会借助另一个工具twine）

```bash
sudo apt install python3-pip
```

**安装pipenv**

`pip3 install pipenv`

> pipenv操作

```python
# 安装指定模块, 并写入Pipfile(requirements)中
pipenv install falsk

# 安装指定模块的指定版本
pipenv install flask==1.0.2

# 卸载指定模块
pipenv uninstall flask

# 更新指定模块
pipenv update flask

# 查看安装列表
pip list

# 查看安装列表, 及其相应的依赖
pipenv graph

# 虚拟环境信息
pipenv --venv

# python解释器信息
pipenv --py 

# 卸载当前虚拟环境
pipenv --rm

# 退出当前虚拟环境
exit
```

## 2.本地启动

### clone项目

```bash
# 进入文件夹, 克隆项目
cd Drink

git clone https://github.com/Allen7D/mini-shop-server.git

cd mini-shop-server
```

### 生成虚拟环境

````bash
# 创建.venv文件夹, 用于存放该项目的python解释器包括所有的依赖包
mkdir .venv

# 指定某python版本创建环境
pipenv --python 3.6

# 激活虚拟环境
pipenv shell

# 安装包依赖
pipenv install

# 启动 port:5000
python server.py run 

# 启动 port 8080
python server.py run -p 8080

# 启动 本地ip port:8080
python server.py run -h 0.0.0.0 -p 8080
````

## 3.生成临时管理员信息

```bash
python fake.py
```

## 4.Pycharm配置

Pycharm中 配置 Pipenv生成的虚拟环境，并使用 **`指定端口`** 开启「Debug模式」

1. 获取该虚拟环境下 Python的解释器的路径
