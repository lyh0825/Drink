## 服务器

### 详细对比

<img src="https://pic1.zhimg.com/v2-8dd72a44b6c9936bcf5e03ea21028eb3_r.jpg?source=1940ef5c" alt="preview" style="zoom:200%;" />

### 轻量应用服务器

**单机类应用**

- 个人网站
- 博客
- 云端测试等轻量级应用
  - 技术小白
  - 不想搭建云服务器环境
  - 没有云平台内网需求
  - 本身自带运行环境
- 特点
  - 轻运维、开箱即用

### 云服务器CVM

**专业级应用**

- 集群
- 高可用应用
- 特点
  - 高并发网站
  - 视频编解码
  - 大型游戏
  - 复杂分布式集群应用

## 终端连接CVM

1. **CVM生成hash密钥**

```shell
# 在云服务器配置中生成密钥rsa
# 需要注意生成的后只能绑定已经关机的实例
```

2. 修改rsa证书权限

```shell
# 将在CVM生成的pem证书文件移动到指定目录下

# 在次目录下运行
chomd 400 <target_rsa.pem>
```

3. **item远程链接服务器**

**配置信息如下**

| Profiles          | -              | order            |
| ----------------- | -------------- | ---------------- |
| name              | -              |                  |
| title             | job            |                  |
| command           | command        | **ssh -i order** |
| working directory | home directory |                  |

- ssh order`ssh [-i login_name] [-p port] [user@]hostname`

```shell
ssh -i ~/.ssh/drink_rsa.pem root@110.40.240.39
```

- ssh -i :ssh命令, 指定用户登录
  -  [-i identity_file]
  - <必须指定>
- ~/.ssh/drink_rsa.pem : cvm生成的rsa证书
- root : cvm主机用户
- 110.40.240.39 : cvm公网ip地址

### ssh操作

 ssh命令用于远程登陆上Linux主机

ssh默认port: 22

不指定用户：服务器

> ssh 192.168.0.11网络

指定用户：

> ssh -l root 192.168.0.11ssh

> ssh root@192.168.0.11tcp
> 若是修改过ssh登陆端口的能够：工具

> ssh -p 12333 192.168.0.11spa

> ssh -l root -p 12333 216.230.230.114操作系统

> ssh -p 12333 root@216.230.230.114

```shell
ssh [-b bind_address] [-c cipher_spec]
[-D [bind_address:]port] [-E log_file] 
[-e escape_char] [-F configfile] 
[-I pkcs11] [-i identity_file]
[-J [user@]host[:port]] [-L address]
[-l login_name] [-m mac_spec]
[-O ctl_cmd] [-o option] [-p port]
[-Q query_option] [-R address]
[-S ctl_path] [-W host:port] 
[-wlocal_tun[:remote_tun]][user@]hostname [command]
```

**默认配置文件和SSH端口**

/etc/ssh/sshd_config：OpenSSH服务器配置文件；

/etc/ssh/ssh_config：OpenSSH客户端配置文件；

~/.ssh/：用户SSH配置目录；

~/.ssh/authorized_keys：用户公钥（RSA或DSA）；

/etc/nologin：若是存在这个文件，sshd会拒绝除root用户外的其它用户登陆；

/etc/hosts.allow和/etc/hosts.deny：定义tcp-wrapper执行的访问控制列表；

3. **mac终端连接(需要CVM密码)**

- 打开mac终端
- 输入`chmod 400 <CVM-ssh-的绝对路径>`
- 输入`ssh root@CVM.ip`
- 输入`CVM密码`

----

- 打开mac终端
- 输入`chmod 400 <CVM-ssh-的绝对路径>`
- 输入`ssh -i ~/.ssh/id_rsa root@CVM.ip`
- **(无需密码即可)**

---

- 打开mac终端
- 输入`chmod 400 <CVM-ssh-的绝对路径>`
- 输入`ssh -l root -p 22 CVM.ip`
- **(VCM密码)**

## 0.Linux基础

**查看系统内核**

`uname -r`

**查看系统版本**

`cat /etc/os-release`

**用户与主机名**

```shell
# [root@localhost]
root: 当前登陆的用户
localhost: 当前的主机名

# 修改主机名
1.临时修改
sudo hostname <new-hostname>

2.永久修改 命令修改|配置修改
> sudo hostnamectl set-hostname <newhostname>
> vim手动修改/etc/hostname文件

# 查看当前主机名
hostnamectl

# 修改之后重启生效
reboot
```

**退出|关机|重启linux**

```bash
退出:
exit

关机命令：
1、halt 立刻关机
2、poweroff 立刻关机
3、shutdown -h now 立刻关机(root用户使用)
4、shutdown -h 10 10分钟后自动关机 如果是通过shutdown命令设置关机的话，可以用shutdown -c命令取消重启
5、init0 停机或者关机

重启命令：
1、reboot
2、shutdown -r now 立刻重启(root用户使用)
3、shutdown -r 10 过10分钟自动重启(root用户使用)
4、init6 重启

init一共分为7个级别，这7个级别的所代表的含义如下
0：停机或者关机（千万不能将initdefault设置为0）
1：单用户模式，只root用户进行维护
2：多用户模式，不能使用NFS(Net File System)
3：完全多用户模式（标准的运行级别）
4：安全模式
5：图形化（即图形界面）
6：重启（千万不要把initdefault设置为6）
```

### yum命令

> yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

### SSH登陆

- 从CVM中获取`rsa_pub.pem`
- 将`rsa_pub.pem`保存在client的`./ssh`目录下
- 从client`./ssh`中获取`rsa_pub`
- 将`rsa_pub`中的内容放在CVM的`/root/.ssh/authorized_keys`中

```shell
# client 
ssh root@ip

# ori
ssh -i ~/.ssh/rsa_CVM.pub root@CVM.ip
```

#### SSH

`/root/.ssh/authorized_keys`

**远程主机将用户的公钥，保存在此**

- 存放访问的client的pub_rsa

----

`/etc/ssh/sshd_config`

`/etc/ssh/ssh_config`

- ssh_config是针对客户端的配置文件

- sshd_config是针对服务器的配置文件

**ssh服务配置文件**

**文件修改后使用:**

`service sshd restart `保存

## 1.安装docker

1⃣️卸载旧版本docker

```bash
$  sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2⃣️安装yum

```bash
$ sudo yum install -y yum-utils
```

3⃣️设置镜像仓库

```bash
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4⃣️更新yum索引和cache

```bash
yum makecache fast
```

5⃣️安装docker

```bash
yum install docker-ce docker-ce-cli container.io
```

6⃣️启动docker

```shell
systemctl start docker
```

7⃣️查看docker配置|版本

```bash
docker version
```

8⃣️启动hello world

```bash
docker run hell-world
```

9⃣️查看docker的images

```bash
docker images
```

## 2.yum 命令

### 简介

yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令

### 使用方法

### yum 语法

```
yum [options] [command] [package ...]
```

- **options：**可选，选项包括-h（帮助），-y（当安装过程提示选择全部为 "yes"），-q（不显示安装的过程）等等。
- **command：**要进行的操作。
- **package：**安装的包名。

### 常用命令

## yum常用命令

- \1. 列出所有可更新的软件清单命令：**yum check-update**
- \2. 更新所有软件命令：**yum update**
- \3. 仅安装指定的软件命令：**yum install <package_name>**
- \4. 仅更新指定的软件命令：**yum update <package_name>**
- \5. 列出所有可安裝的软件清单命令：**yum list**
- \6. 删除软件包命令：**yum remove <package_name>**
- \7. 查找软件包命令：**yum search <keyword>**
- \8. 清除缓存命令:
  - **yum clean packages**: 清除缓存目录下的软件包
  - **yum clean headers**: 清除缓存目录下的 headers
  - **yum clean oldheaders**: 清除缓存目录下旧的 headers
  - **yum clean, yum clean all (= yum clean packages; yum clean oldheaders)** :清除缓存目录下的软件包及旧的 headers
