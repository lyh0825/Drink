## Q21 SSLError

### -Info

在使用`AsyncHTTPClient`调用`API`时, 出现错误

```python
AsyncHTTPClient
ssl.SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:852)
```

### -Analysis

```python
url = "https://txm.zidouchat.com/wx/get_access_token"
data = {"token": "doododisagreatcompany"}
body = json.dumps(data)
response = await httpclient.fetch(url, method="POST", body=body)
```

debug排查:

```python
print(ssl_options, host)
if ssl_options is not None:
	if timeout is not None:
		stream = yield gen.with_timeout(timeout, stream.start_tls(False, ssl_options=ssl_options, server_hostname=host))
	else:
		stream = yield stream.start_tls(False, ssl_options=ssl_options,server_hostname=host)
raise gen.Return(stream)
```

此网站的ssl证书验证失败

```python
<ssl.SSLContext object at 0x7f9f0d393ac8>   txm.zidouchat.com
```

### -Solution

方法一

> 在使用http链接建立请求时关闭ssl验证

```python
# 添加ssl跳过验证
import ssl
ssl_options = {"ssl_version": ssl.PROTOCOL_TLSv1}

# 添加ssl跳过验证
if ssl_options is not None:
	if timeout is not None:
		stream = yield gen.with_timeout(timeout, stream.start_tls(False, ssl_options=ssl_options, server_hostname=host))
    else:
        stream = yield stream.start_tls(False, ssl_options=ssl_options,
                                                server_hostname=host)
```

方法二

> 在实例化时赋予属性 validate_cert=False, 有一定风险仍然使用old_ssl

```python
import tornado.httpclient

request = tornado.httpclient.HTTPRequest(
    url="https://myhostname.com:9001",
    method="GET",
    validate_cert=False
)
print tornado.httpclient.HTTPClient().fetch(request)
```

### -Review

`curl http_address -k`

可以跳过ssl验证

```python
I'm trying to use tornado's http client to fetch a URL. I've done this before many times, but I'm getting a really odd SSL error this time. The endpoint I'm trying to consume does not have a valid cert, but a -k on a curl call still proves it works.
```

## Q20 Docker Compose

### -Info

```shell
# 拉取镜像报错
docker-compose up
```

### -Analysis

```shell
# 报错信息
ERROR: write /var/lib/docker/tmp/GetImageBlob264931462: no space left on device

# 指宿主机内部没有空间写入docker文件

# 查看内存使用
df -h
```

### -Solution

```shell
# 清除宿主机内不在使用的imgaes
docker rmi $ (docker images -aq)
```

## Q19GIt PUll失败

### -Info

```python
(venv) edy@bogon TXM % git pull

Your configuration specifies to merge with the ref 'refs/heads/watsons_uat'
from the remote, but no such ref was fetched.
```

### -Analysis

1. 远程仓库不存在要pull的分支

```shell
本地仓库当前分支是a，但是a的远程分支被删除掉了，pull的时候就会报错，这时候使用
git remote prune origin
同步下远程分支，并清除掉已删除的分支，再清除掉当前分支（如果当前分支在工作区的话）就可以了

建议：使用命令行拉取新分支代码前使用：git fetch -p 获取最新分支 然后获取远程分支代码
git fetch -p
```

### -Solution

```shell
# 切换到其他分支, 建议master分支
# 同步远程分支, 并清楚已经删除的分支
# 清除被删除的分支
git remote prune origin  

# 在获取之前，删除远程上不再存在的任何远程跟踪引用
git fetch -p  

# 切换到出现问题的分支
git checkout target_branch

# 关联分支
git branch --set-upstream-to=origin/master <target_branch>

git push --set-upstream origin <local_branch>
```

## Q18mogodb更新失败

### -Info

在更新ipdate_one()时出现update error

### -Analysis

查看所使用的是否是原生语法, 如果使用http连接更新, 则尝试使用原生语句进行更新

## Q17linux连接目标主机报错[ssh]

### -Info

**问题一**

```shell
通过linux主机，远程其他linux服务器时报错：Unable to negotiate with xx.xxx.xxx.x port 22:no matching cipher found. Their offer: aes128-cbc,aes256-cbc,3des-cbc,des-cbc
```

**问题二**

```shell
远程其他linux服务器时又报错：Unable to negotiate with xx.xx.xx.xx port 22: no matching key exchange method.found.Their.offer:diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```

### -Analysis

**问题一**

```shell
由于发起远程访问端的主机openssh版本过高（我的这台主机为最新8.1版本），导致不支持对低版本的远程连接主机的ssh解密，故报错。
```

**问题二**

```shell
根据提示，ssh不能远程登录的原因为：没有找到相关的主机密钥类型。
```

### -Soultion

**问题一**

```shell
# 修改发起远程访问的本机SSH客户端配置（/etc/ssh/ssh_config）文件，将默认注释的：
Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
# 使用即可。
```

**问题二**

```shell
# 修改发起远程访问的本机SSH客户端配置（/etc/ssh/ssh_config）文件，在最后加上一行：
KexAlgorithms +diffie-hellman-group1-sha1
```

## Q16Docker连接redis失败

### -Info

在docker-container中发现container重启

`docker logs -f --tail 100 container_id<container name>`

发现是连接redis报错

### -Analysis

分析发现是docker连接不到redis

### -Solution

```shell
# 查看docker状态
services docker status

# 重启docker
systemctl restart docker
```



## Q15Docker运行脚本

### -Info

在需要跑数据时, 可以 [根据场景选择] :

1. 写script
2. 写临时API

### -Solution

```python
import asyncio

from config.config import Table
from model.base_model import BaseModel

phone = ['16620132242', '16676748179']


async def export_excel():
    friend_count = []
    for sim in phone:
        print(f'---------sim:{sim}')
        bot_username = await BaseModel.fetch_one(
            Table.BotInfo,
            "*",
            BaseModel.where_dict({"sim": sim})
        )

        if not bot_username:
            friend_count.append(0)
            continue

        friend_count_info = await BaseModel.count_(
            Table.Friend,
            BaseModel.where_dict({"bot_username": bot_username.username, "is_friend": 1})
        )

        friend_count.append(friend_count_info)
    return friend_count


async def main():
    task = asyncio.Task(export_excel())
    done, pending = await asyncio.wait({task})
    if task in done:
        print(task)


if __name__ == "__main__":
  	# 需要初始化BaseModel的model_dict()
    BaseModel.init_model_dict()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
    loop.close()
```

### -Review

使用到的命令:

```python
# copy file from linux-server to another linux-server
scp source_file root@remote_ip:/target_directory/

# create file
vim target_file.py

# copy file from linux-server to linux-server-container
docker cp source_file container_id[name]:/target_directory/
```

需要注意的:

```python
# 需要初始化BaseModel的model_dict()
BaseModel.init_model_dict()

# 导出excel
from openpyxl import Workbook


people_count_list = [4190, 1415]
book = Workbook()
sheet = book.active

for index, value in enumerate(people_count_list):
    sheet[f"A{index+1}"] = value
    print(value)

book.save("/Users/edy/Desktop/sample.xlsx")
```

## Q14导出大数据Excel

### -Info

在导出群-机器人数据时, 涉及到群数据量很大导出excel失败

### -Solution

使用page pagesize进行分页导出

## Q13字典获取key

获取dict中key所对应的vaule, 使用. get ["key"]有什么区别?

## Q12unhashed type dict

### -Info

```python
ERROR: unhashed type dict
```

### -Analysis

字典不能存储在集合

## Q11生成器对象不可下标

### -Infolocal

```python
TypeError: ‘generator’ object is not subscriptable 
```

### -Analysis

通常是list.append(generator)

### -Solution

```python
a = []
b = []
a.append([for item in range(1, 5)])
b.extend(for item in range(1, 5))

[[1, 2, 3, 4]]
[1, 2, 3, 4]

# 列表生成器
[for item in range(1, 5)]
```

## Q10导入包含同名文件冲突

### -Info

```json
import resolves to its containing file

file> component
import time
```

### -Analysis

在`component`目录下有`time`文件,且import 了 time(python) 与python自带的第三方time库冲突

### -Solution



## Q9Mongodb保存|插入字段失败

- [ ] 1.查看Table定义==>fiedls = fiedls.**IntFields()**   ==>需要加()

## Q8虚拟环境配置

- [ ] 拉取项目后如何配置虚拟环境?
- [ ] 如何激活虚拟环境, 并每次进入pycharm的terminal都保证是venv?

1. venv的目录和interpreter的目录要配置相同

![image-20211018164551110](/Users/edy/Library/Application Support/typora-user-images/image-20211018164551110.png)

![image-20211018164714977](/Users/edy/Library/Application Support/typora-user-images/image-20211018164714977.png)

2. terminal的配置如下

![image-20211018164922890](/Users/edy/Library/Application Support/typora-user-images/image-20211018164922890.png)

## Q7Nginx Timeout 504

### -Info

在前端导出excel时, 导出失败并返回`GateWay TimeOut 504`

### -Analysis

- 重新导出查看log

```shell
url: /channel/get_excel?token=ab5c40fa9da40d74f5a2ba17d84dcf63
GET /channel/get_excel?token=ab5c40fa9da40d74f5a2ba17d84dcf63 (183.81.181.194) 67000.83ms
```

- 说明导出正常, 其中导出时间大于`60s`
- nginx做反向代理，默认请求是有一个60秒的超时，如果http请求超过了60秒，再返回，连接就会被nginx中断，前端就会得到504的错误：gateway time-out。

### -Solution

> 修改nginx的配置, 将超时设置延长至300s

```shell
docker ps 

docker exec -it nginx_container_id bash

whereis nginx.conf

cd /etc/config

vim nginx.conf

{
    ...
    keepalive_timeout 300;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;
    proxy_read_timeout 300;
    send_timeout 300;
    ...
}
```

> 使用挂载修复

```shell
# 进入服务器
ssh root@xxx

# 编辑volumne
vim docker-compose.yaml

# 添加挂载
 nginx-proxy:
    image: registry.cn-beijing.aliyuncs.com/doododmain/nginx-proxy:20mbody
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /certs:/etc/nginx/certs
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - /root/nginx/nginx.conf:/etx/nginx/nginx.conf

# 在外部创建对应的挂载文件[宿主机]
vim /root/nginx/nginx.conf

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout 300;

    proxy_connect_timeout 300;

    proxy_send_timeout 300;

    proxy_read_timeout 300;

    send_timeout 300;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
daemon off;
```

### -Review

- 如果程序正常, log也没有报错, 很可能是配置的原因

## Q6mongodb插入数据报错

### -Info

在mongodb中插入数据报错`{'code': -1, 'msg': 'exception'}`

### -Analysis

经过排查不是以下原因:

- 插入字段类型与定义不符
- 缺少必要字段

### -Solution

在mongodb表中定义字段如下:

```python
# 群发任务
class AssistantBatchSend(BaseModel):
    __table__ = 'assistant_batch_send'
    client_id = fields.IntField()     # 创建人
    task_id = fields.IntField()       # 任务标识
    message_list = fields.JsonField()  # 发送内容
    bot_username_list = fields.JsonField()  # 机器人id
```

在插入的数据中

```python
task_identification = int(str(self.user_info.client_id) + str(int(time.time()*1000000))) # 任务标识
immediate_info = CM(Table.AssistantBatchSend)
immediate_info.task_id = task_identification
```

其中task_id定义为`intfiled`长度最大

为:

![image-20211011134300296](/Users/edy/Library/Application Support/typora-user-images/image-20211011134300296.png)

但是传入的长度为`22`, 应该把此字段定义为`StringField`

## Q5SSH连接VCM断开

### -Question

**报错信息**

**info1**

client_loop: send disconnect: Broken pipe

**info2**

yum-config-manager client_loop: send disconnect: Broken pipe

### -Analysis

**info1**

终端连接服务器无响应且没用自动间隔连接配置**VCM的问题**

### -Solution

**info1**

修改`/etc/ssh/sshd_config`文件, 修改`ClientAliveInterval 120`

```bash
# 通过每 300 秒（每 5 分钟）进行响应确认来保持连接。 我认为这个部分的数字可以是60或120。
>/etc/ssh/sshd_config
#PermitUserEnvironment no
#Compression delayed
ClientAliveInterval 60
#ClientAliveCountMax 3
```

重新启动ssh服务

```bash
systemctl restart sshd
```

或者在客户端进行更改`/etc/ssh/ssh_config`

```shell
# 添加参数
TCPKeepAlive yes
ServerAliveInterval 60
```

**info2**

配置`/etc/ssh/ssh_config`文件，增加以下内容即可：

```bash
Host *
				IPQoS=throughput
        # 断开时重试连接的次数
        ServerAliveCountMax 5

        # 每隔5秒自动发送一个空的请求以保持连接
        ServerAliveInterval 5
```

重新启动ssh服务

```bash
systemctl restart sshd
```

### -Review

**1.在服务端进行修改**

在服务器端， 可以让服务器发送“心跳”信号测试提醒客户端进行保持连接

通过修改 sshd 的配置文件，能够让 SSH Server 发送“心跳”信号来维持持续连接，下面是设置的内容

打开服务器 `/etc/ssh/sshd_config`，我在最后增加一行

```
ClientAliveInterval 60
ClientAliveCountMax 1
```

这样，SSH Server 每 60 秒就会自动发送一个信号给 Client，而等待 Client 回应，（注意：是服务器发心跳信号，不是客户端，这个有别于一些 FTP Client 发送的 KeepAlives 信号），如果客户端没有回应，会记录下来直到记录数超过 ClientAliveCountMax 的值时，才会断开连接。

**2.在客户端进行修改**

只要在/etc/ssh/ssh_config文件里加两个参数

```
TCPKeepAlive yes
ServerAliveInterval 60
```

前一个参数是说要保持连接，后一个参数表示每过1分钟发一个数据包到服务器表示“我还活着”

如果你没有root权限，修改或者创建~/.ssh/ssh_config也是可以的

## Q4vim安装失败

### -Question

1. 在使用`vim`时发现报错`bash: vim: command not found`

### -Analysis

使用`cat /proc/version`查看系统版本

![image-20210928202753365](/Users/edy/Library/Application Support/typora-user-images/image-20210928202753365.png)

查看系统镜像中是否有`vim`, 发现`red hat 5`才有`vim`

### -Solution

- 使用`apt install vim `安装`vim`
- 发现`Unable to locate package vim`

![image-20210928203059115](/Users/edy/Library/Application Support/typora-user-images/image-20210928203059115.png)

- 需要输入：`apt-get update`（作用是：同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包）
- 之后再输入`apt-get install vim`安装（非root用户登录，`root apt-get intall vim`）

### -Review

**docker常用命令**

```shell
# 修改了image内的文件后重启image
docker restart image_id

# 启动container中的所有image
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)

# 停止container中的所有image
docker stop $(docker ps -a | awk '{ print $1}' | tail -n +2)

# 删除container中的所有image
docker rm $(docker ps -a | awk '{ print $1}' | tail -n +2)
```

## Q3镜像部署失败

### -Question

1. 部署对应的tag的时候出现`error`

![image-20210928155639667](/Users/edy/Library/Application Support/typora-user-images/image-20210928155639667.png)

2. 有正在运行的container, 无法部署

### -Analysis

- 在部署的过程中存在正在运行的container
- 需要将所有的正在运行的container停下来或者删除所有正在运行的container中的image

### -Solution

```bash
# 启动所有image
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)

# 停止运行所有image
docker stop $(docker ps -a | awk '{ print $1}' | tail -n +2)

# 删除container中的所有image
docker rm $(docker ps -a | awk '{ print $1}' | tail -n +2)

# 重新部署
sh deploy-debug.sh 1.0.0.65
```

### -Review

- 使用`docker ps -a`container中的所有image包括正在运行和未运行的
- `docker rmi -f 'docker iamges' `删除的已经存在但未运行的image
- `docker rm image_id`删除正在运行的image

## Q2镜像重启

### -Question

1. Docker ps 查看镜像状态显示`restart`

![image-20210928095953991](/Users/edy/Library/Application Support/typora-user-images/image-20210928095953991.png)

2. RabbitMQ只有生产者没有消费者

### -Analysis

1. 部分服务器没起来

### -Solution

1. `docker logs -f image_name `查看对应的image日志
2. 找到报错信息

![image-20210928095931103](/Users/edy/Library/Application Support/typora-user-images/image-20210928095931103.png)

`cerely can't find module name 'component.task.member_change_task.component'`

**cerely配置错误**

### -Review

- docker logs -f 
- command r 搜索已经使用的order

## Q1磁盘溢出

### -**Question**

1. mongodb连接失败报错
   - `failed to read 4 bytes from socket within 300000 milliseconds`
2. 紫豆超管后台登陆失败
   - `login 无响应`
3. 机器人正常但不可用
   - `机器人上报heartbeat给服务器但服务器不callback`

### -Analysis

1. 登陆紫豆服务器`ssh root@ip`
   - 报错显示disk overflower
2. `df -h`查看disk存储情况

![image-20210926170428181](file:///Users/edy/Library/Application%20Support/typora-user-images/image-20210926170428181.png?lastModify=1632625187)

### **-Solution**

1. 删除不用的container|容器修剪

   - PRO环境具有一定的风险
   - `docker system prune`  `docker container prune`

   ```dockerfile
   docker container prune
   WARNING! This will remove all stopped containers.
   Are you sure you want to continue? [y/N] y
   Deleted Containers:
   4a7f7eebae0f63178aff7eb0aa39cd3f0627a203ab2df258c1a00b456cf20063
   f98f9c2aa1eaf727e4ec9c0283bc7d4aa4762fbdba7f26191f26c97f64090360
   
   Total reclaimed space: 212 B
   ```

2. **删除镜像[优先选择]**

   - ```bahs
     docker rmi -f `docker images`
     ```

   - docker image需`

   - **删除已经存在的镜像「不包括正在运行的」**

### -Review

**docker rmi :** 删除本地一个或多个镜像。

```docker
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

**OPTIONS说明：**

- **-f :**强制删除；
- **--no-prune :**不移除该镜像的过程镜像，默认移除；

----

**docker prune 命令**

prune 命令用来删除不再使用的 docker 对象。

删除所有未被 tag 标记和未被容器使用的镜像:

```bash
$ docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
```

删除所有未被容器使用的镜像:

```bash
$ docker image prune -a
```

删除所有停止运行的容器:

```bash
$ docker container prune
```

删除所有未被挂载的卷:

```bash
$ docker volume prune
```

删除所有网络:

```bash
$ docker network prune
```

删除 docker 所有资源:

```bash
$ docker system prune
```

**docker system prune** 命令：

```
This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all dangling build cache
```

删除停止的容器、删除所有未被容器使用的网络、删除所有none的镜像。

