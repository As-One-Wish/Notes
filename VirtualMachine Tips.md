# 一、配置CentOS 8 虚拟机

1、创建虚拟机后，选择自定义

2、硬件兼容性，默认为`Workstation 17.x`，直接下一步

3、选择稍后安装操作系统，或者也可以直接选择CentOS 8 的光盘映像文件

<img style="width:40%;" src=".\Images\virtualMachine-1-2.png">

4、选择`Linux(L)`和`CentOS 8 64位`

5、虚拟机名称看个人喜好，主要是位置，自己要设置好

<img style="width:40%;" src=".\Images\virtualMachine-1-3.png">

6、处理器数量看自己电脑配置，默认就行，好像不要超过本机的一半，我是`2 1 2`

7、关于此虚拟机的内存，在不超过最大推荐内存的情况下，可以稍大一点，2或4或以上

8、网络类型涉及到以后虚拟机和主机的连接问题，每种类型连接配置不太一样，我这里是选了[NAT](#0.NAT配置)

<img style="width:40%;" src=".\Images\virtualMachine-1-4.png">

9、然后一路下一步，按照推荐的来，直到创建新磁盘，大小自己看着来，下面的单个文件和多个文件没什么影响，然后一路下一步直到完成。

<img style="width:40%;" src=".\Images\virtualMachine-1-5.png">

10、然后配置镜像文件（如果前面已经配置了直接跳过）

<img style="width:70%;" src=".\Images\virtualMachine-1-6.png">

11、开机虚拟机，选择安装CentOS 8，选择第一个，第二个是要先检查文件的完整性再安装，很慢，默认是第二个，记得改

<img style="width:40%;" src=".\Images\virtualMachine-1-7.png">

12、基础信息的配置

<img style="width:60%;" src=".\Images\virtualMachine-1-8.png">

- 对于1，自动分区就可以，点进去一下就行
- 对于2，点进去记得把以太网打开
- 对于3，如果选择带GUI的服务器，可以把右侧的附加软件都选上
- 对于4，亚洲-上海
- 对于下方的根密码和用户，自己设置成自己记得住的

设置完，点击开始安装，静待安装就OK了。

# 二、CentOS 8 下载谷歌浏览器

1、并且使用`wget`下载最新版的Chrome`.rpm`安装包(建议直接切换root用户)

```shell
[root@192 hwj]# wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
```

​	此命令会将安装包下载至当前目录，例如上命令会下载至`/hwj`

2、安装 Chrome 浏览器(有root权限)

```shell
sudo dnf localinstall google-chrome-stable_current_x86_64.rpm
```

​	此命令执行如果有错误，根据[解决方案](#1、下载元数据失败)解决后再次执行。

3、卸载自带火狐浏览器

```shell
[root@192 hwj]# yum remove firefox	----卸载火狐
[root@192 hwj]# whereis firefox		----查看残留
firefox: /usr/lib64/firefox
[root@192 hwj]# cd /usr/lib64/		
[root@192 lib64]# rm -rf firefox	----进入对应文件夹并删除
```

# 三、CentOS 8 安装Docker

1、Docker 要求 CentOS 的内核版本，至少高于 3.10 ，可以用命令`uname -r`查看，低于1.10用`yum update`升级

2、安装Docker对应的依赖

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3、添加仓库

```shell
# 官方仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 阿里云仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4、安装`docker engine`，包括`docker-ce`、`docker-ce-cli`和`containerd.io`

```shell
yum install docker-ce docker-ce-cli containerd.io
```

​	如果第一次安装，此处应该会报错：

```shell
 - 软件包 buildah-1.22.3-2.module_el8.5.0+911+f19012f9.x86_64 需要 runc >= 1.0.0-26，但没有提供者可以被安装
  - 软件包 docker-ce-3:24.0.5-1.el8.x86_64 需要 containerd.io >= 1.6.4，但没有提供者可以被安装
  - 软件包 containerd.io-1.6.10-3.1.el8.x86_64 与 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）冲突
  - 软件包 containerd.io-1.6.10-3.1.el8.x86_64 取代了 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）
```

​	根据提示，`runc`包冲突，删掉

```shell
yum remove runc
```

​	之后再执行安装命令，一路按y即可成功安装

5、启动Docker

```shell
systemctl start docker
```

6、查看Docker版本，验证是否安装成功

```shell
docker version
```

​	Docker命令似乎要在管理员权限下运行，否则会有警告：

```shell
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
```

# 四、Docker创建Redis容器

### 0.NAT配置

- 点击菜单栏`编辑`->`虚拟网络编辑器`，选择当前使用的NAT网关，一般是VMnet8，然后点击更改设置，进入NAT设置

<img style="width:40%;" src=".\Images\virtualMachine-4-1.png">

- 在端口转发处点击添加，在打开的窗口中，填写主机端口（主机闲置的端口就行）；类型任选；虚拟机IP地址可以使用`hostname -I`或`ifconfig`查看；虚拟机端口，SSH的话是22，HTTP的话是80（并没有察觉的不一样）；设置完后一路确定就好

<img style="width:60%;" src=".\Images\virtualMachine-4-2.png">

- 回到主机，打开主机的网络适配器选项，右键任务栏网络图标进入`网络和Internet设置`$\rightarrow$`高级网络设置`$\rightarrow$`更多网络适配器选项`，找到`VMnet8`，按以下顺序，进入IPv4的配置界面。

<img style="width:80%;" src=".\Images\virtualMachine-4-3.png">

​	然后其中的IP地址填入虚拟机中的网段中的IP地址（也就是前三部分要一样，最后一部分不要和虚拟机的重合）。两部分IP的设置并没有先后顺序，只要保证主机和虚拟的网段一致即可。

- 打开防火墙和网络保护，点击其中高级设置

<img style="width:60%;" src=".\Images\virtualMachine-4-4.png">

​	在进入界面的左侧菜单选择入站规则， 将其中`文件和打印机共享（····ICMPv4-In）`规则启用，同时将出站规则中相同规则也启用

<img style="width:80%;" src=".\Images\virtualMachine-4-5.png">

- 以上一番操作应该可以连接了，如果还是不行就把虚拟机防火墙关掉，在主机使用`telnet ip port`进行验证

### 1.下载Redis镜像

```shell
docker pull redis				# 拉取最新版Redis镜像
docker pull redis:xxx		# 下载指定版本Redis镜像
```

### 2.创建Redis配置文件

​	启动前需要先创建Redis外部挂载的配置文件，Redis本身容器只存在 /etc/redis 目录 ，本身就不创建 redis.conf 文件，当服务器和容器都不存在 redis.conf 文件时,，执行启动命令的时候 docker 会将 redis.conf 作为目录创建 , 这并不是我们想要的结果。

```shell
mkdir -p /docker/redis
vim /docker/redis/redis.conf

## 以下是redis.conf内容
appendonly yes # 启动Redis持久化功能（默认为no，所有信息都存储在内存[重启丢失]，设置为yes，将存储在硬盘[重启还在]）
protected-mode no # 关闭protected-mode模式，外部网络可以直接访问
# bind 0.0.0.0 # 设置所有IP都可以访问
requirepass redis # redis连接密码
# daemonize yes # 是否后台运行
port 6379
```

### 3.创建容器并启动

```shell
docker run --name redisContaine \   # 容器名字
	-p 6379:6379 \		# 指明容器的6379端口指向虚拟机的6379端口
  -v /docker/redis/data:/data \		# 容器的存储文件挂载到主机  主机：容器
  -v /docker/redis/redis.conf:/etc/redis/redis.conf \		# 容器的配置文件挂载到主机
  -d --restart=always redis redis-server /etc/redis/redis.conf  # restart:重启策略  redis-server:启动
```

​	正常执行完会返回一串容器ID

```shell
docker ps  # 查看容器是否执行
```

### 4.进入容器

```shell
docker exec -it [容器名字] bash  # 进入容器命令行交互模式
```

​	如果报错显示容器一直运行，见[解决方案](#2.容器一直运行)

# 五、Docker创建RabbitMQ容器

### 1.下载RabbitMQ镜像

```shell
docker pull rabbitmq
```

### 2.创建容器并启动

~~~shell
docker run -d --name RabbitMQ -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=rabbit -e RABBITMQ_DEFAULT_PASS=rabbit rabbitmq
~~~

> - `RABBITMQ_DEFAULT_USER`、`RABBITMQ_DEFAULT_PASS`是环境变量，分别是用户名和密码

```c#
docker start [容器名]
```

​	此时就可以访问`IP：15672`来进入RabbitMQ的客户端页面。[无法进入?](#3.RabbitMQ客户端无法连接)

​	客户端可以进入，打开`channel`页面时出现`Stats in management UI are disabled on this node`错误：[这里](#4.RabbitMQ客户端问题)

### 3.设置容器自启动

~~~shell
docker update [容器名] --restart=always
~~~

# 六、CentOS设置共享文件夹

### 1.开启共享文件夹

<img style="width:60%;" src=".\Images\virtualMachine-6-1.png">

​	在虚拟机的选项中开启共享文件夹，并选择本机的文件夹作为共享文件夹

### 2.挂载共享文件夹

​	然后进入虚拟机，在命令行中执行以下命令

```shell
vmhgfs-fuse .host:/ /mnt/hgfs
```

​	则共享文件夹就挂载在`/mnt/hgfs`中，但是此命令单次有效，下次开机后就要重新执行

### 3.设置开机自动挂载

​	执行`vim /etc/fstab`，在打开的文件的最后一行加入以下命令,其中`xxx`是共享文件夹名字

```shell
.host:/xxx /mnt/hgfs fuse.vmhgfs-fuse allow_other 0 0
```

# 七、Docker创建PostgreSQL/PostGis容器

### 1.单独安装PostgreSQL

##### 1.拉取镜像

```shell
docker pull postgres
```

##### 2.创建容器

```shell
docker run -it --name Postgres --restart=always # 设置容器名字及开机自启
  -e POSTGRES_PASSWORD='postgres'  # 设置连接密码
  -e ALLOW_IP_RANGE=0.0.0.0/0 	# 设定其他主机连接，不加则只允许本机连接
  -v /docker/postgres/data:/var/lib/postgresql  # 文件挂载
  -p 5432:5432 -d postgres	# 端口映射
```

### 2.安装PostGIS

​	PostGis中包含PostgreSQL，无需额外安装，如果之前创建过PostgreSQL容器，可以删除：`docker rm -f [容器名]`，`-f`表示强制删除，即容器开启时也可删除

##### 1.拉取镜像

```shell
docker pull postgis/postgis
```

##### 2.创建volumes

​	创建数据卷，可进行持久化存储，即创建的容器的数据可以一直存放在数据卷中，如果容器删除或重新创建，数据不会消失；如果只是单纯挂载文件夹，那么容器删除或者新建的容器没有以往的数据。

```shell
docker volume create postgis_data 
docker volume create pg_data
```

##### 3.创建容器

```shell
docker run --name Postgis \
	--restart=always \
	-e POSTGRES_USER=postgres \
	-e POSTGRES_PASSWORD=postgres -p 5432:5432 \ 
	-v postgis_data:/var/lib/postgis/data  \
	-v pg_data:/var/lib/postgresql/data -d postgis/postgis
```

# 八、Docker创建Minio容器

### 1.下载Minio镜像

```shell
docker pull minio/minio
```

### 2、虚拟机创建对应文件

```shell
mkdir -p /docker/minio/config
mkdir -p /docker/minio/data
```

### 3、.创建容器并启动

~~~shell
docker run --network=host --name minio -d --restart=always -e "MINIO_ROOT_USER=minioadmin" -e "MINIO_ROOT_PASSWORD=minioadmin" -v /docker/minio/config:/root/.minio -v /docker/minio/data:/data minio/minio server /data --address ":9001"
~~~

> - `--network=host`:Docker容器的网络会附属在主机上，两者是互通的
> - `--address ":9001"`:指定 Minio 服务在容器内部监听的地址和端口

最后出现以下命令表明正确部署：

```shell
WARNING: Detected default credentials 'minioadmin:minioadmin', we recommend that you change these values with 'MINIO_ROOT_USER' and 'MINIO_ROOT_PASSWORD' environment variables
MinIO Object Storage Server
Copyright: 2015-2023 MinIO, Inc.
License: GNU AGPLv3 <https://www.gnu.org/licenses/agpl-3.0.html>
Version: RELEASE.2023-11-20T22-40-07Z (go1.21.4 linux/amd64)

Status:         1 Online, 0 Offline. 
S3-API: http://192.168.149.128:9001  http://172.17.0.1:9001  http://192.168.122.1:9001  http://127.0.0.1:9001               
Console: http://192.168.149.128:46345 http://172.17.0.1:46345 http://192.168.122.1:46345 http://127.0.0.1:46345        

Documentation: https://min.io/docs/minio/linux/index.html
Warning: The standard parity is set to 0. This can lead to data loss.
```

#  解决方案

## 1.下载元数据失败

```shell
错误：为仓库 'appstream' 下载元数据失败 : Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

**问题原因**：

> `CentOS Linux 8`操作系统版本结束了生命周期，`Linux`社区已不再维护该操作系统版本。所以原来的`CentOS Linux 8`的`yum`源也都失效了！

**解决**：

```shell
[root@192 hwj]# cd /etc/yum.repos.d/		----1、切换至yum源文件夹
[root@192 yum.repos.d]# ls -l
总用量 48
-rw-r--r--. 1 root root  719 9月  15 2021 CentOS-Linux-AppStream.repo
-rw-r--r--. 1 root root  704 9月  15 2021 CentOS-Linux-BaseOS.repo
-rw-r--r--. 1 root root 1130 9月  15 2021 CentOS-Linux-ContinuousRelease.repo
-rw-r--r--. 1 root root  318 9月  15 2021 CentOS-Linux-Debuginfo.repo
-rw-r--r--. 1 root root  732 9月  15 2021 CentOS-Linux-Devel.repo
-rw-r--r--. 1 root root  704 9月  15 2021 CentOS-Linux-Extras.repo
-rw-r--r--. 1 root root  719 9月  15 2021 CentOS-Linux-FastTrack.repo
-rw-r--r--. 1 root root  740 9月  15 2021 CentOS-Linux-HighAvailability.repo
-rw-r--r--. 1 root root  693 9月  15 2021 CentOS-Linux-Media.repo
-rw-r--r--. 1 root root  706 9月  15 2021 CentOS-Linux-Plus.repo
-rw-r--r--. 1 root root  724 9月  15 2021 CentOS-Linux-PowerTools.repo
-rw-r--r--. 1 root root 1124 9月  15 2021 CentOS-Linux-Sources.repo
[root@192 yum.repos.d]# mkdir bak			
[root@192 yum.repos.d]# mv CentOS-Linux-* bak		----2、创建备份文件夹并备份（感觉不是很必要）
[root@192 yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
																									#----3、下载yum源文件
--2023-07-25 19:44:07--  https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
正在解析主机 mirrors.aliyun.com (mirrors.aliyun.com)... 36.150.239.241, 36.156.72.113, 36.156.72.115, ...
正在连接 mirrors.aliyun.com (mirrors.aliyun.com)|36.150.239.241|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：2495 (2.4K) [application/octet-stream]
正在保存至: “/etc/yum.repos.d/CentOS-Base.repo”
/etc/yum.repos.d/CentOS-Base 100%[==============================================>]   2.44K  --.-KB/s  用时 0s      
2023-07-25 19:44:07 (13.5 MB/s) - 已保存 “/etc/yum.repos.d/CentOS-Base.repo” [2495/2495])

[root@192 yum.repos.d]# yum makecache		----4、建立新的源数据缓存
CentOS-8.5.2111 - Base - mirrors.aliyun.com                                         3.0 MB/s | 4.6 MB     00:01    
CentOS-8.5.2111 - Extras - mirrors.aliyun.com                                       4.3 kB/s |  10 kB     00:02    
CentOS-8.5.2111 - AppStream - mirrors.aliyun.com                                    2.9 MB/s | 8.4 MB     00:02    
元数据缓存已建立。
[root@192 /]# cd /home/hwj			----5、记得切换回原安装包文件夹
[root@192 hwj]# sudo dnf localinstall google-chrome-stable_current_x86_64.rpm
```

## 2.容器一直运行

​	进入容器时报错

```shell
[root@192 /]# docker ps
CONTAINER ID   IMAGE     COMMAND                   CREATED          STATUS                          PORTS     NAMES
c829fdba1213   redis     "docker-entrypoint.s…"   12 minutes ago   Restarting (0) 13 seconds ago             redisContainer
[root@192 /]# docker exec -it redisContainer bash
Error response from daemon: Container c829fdba121393817510df690c262a2143d56fd5d7fbc40e032eedc9775a8f3f is restarting, 
wait until the container is running
```

​	容器一直在重启，同时`docker logs redisContainer`也没有输出

**问题原因：**

> 在我的`redis.conf`配置文件中，其中有一项`daemonize yes`，此项是控制redis后台运行，即以守护进程方式启动，而在创建容器时的`-d`就是以守护进程方式启动redis，两者冲突了

**解决：**将`daemonize yes`注释掉或者改为`daemonize no`即可

```shell
appendonly yes # 启动Redis持久化功能（默认为no，所有信息都存储在内存[重启丢失]，设置为yes，将存储在硬盘[重启还在]）
protected-mode no # 关闭protected-mode模式，外部网络可以直接访问
# bind 0.0.0.0 # 设置所有IP都可以访问
requirepass redis # redis连接密码
# daemonize yes # 是否后台运行
port 6379
```

## 3.RabbitMQ客户端无法连接

**问题原因：**

> 或许是客户端服务未打开或者是虚拟机的防火墙未关闭（本机访问不了可能是这个原因）

**解决：**

~~~shell
# 客户端服务未打开
docker exec -it [容器名] bash  # 进入RabbitMQ容器的交互命令行
rabbitmq-plugins enable rabbitmq_management  # 开启服务

# 防火墙未关闭
systemctl status firewalld.service			# 查看状态
systemctl start firewalld.service				# 打开防火墙（临时）
systemctl stop firewalld.service				# 关闭防火墙（临时，重启后会自动打开）
systemctl enable firewalld.service			# 开启防火墙（永久）
systemctl disable firewalld.service			# 禁用防火墙（永久，重启后不会打开）
~~~

## 4.RabbitMQ客户端问题

**解决**

```shell
# 进入rabbit容器并进入以下路径
docker exec -it [容器名] bash
cd /etc/rabbitmq/conf.d/

# 执行以下命令后退出并重启容器2
echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf
exit
docker restart [容器名]
```

