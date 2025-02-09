# 项目说明

这是一个根据《黑马商城》为基础，逐步修改和优化的一个商城项目。

前端：hmall-nginx

后端：hmall



# hmall

## 服务模块

cart-service

item-service

pay-service

trade-service

user-service

## 网关模块

hm-gateway

## 通用模块

hm-common

hm-api



# hmall-nginx

在目录下,打开cmd执行：

```
start nginx.exe
```

前端项目启动完成，默认端口为18080









# CentOS

## 配置信息

本次使用的是：`CentOS-7-x86_64-Minimal-2009.iso`

网络资源：https://ftp.iij.ad.jp/pub/linux/centos-vault/centos/7.9.2009/isos/x86_64/

网络配置：使用`静态ip`配置，方便。



- 用户名：root

- 密码：123456

## 操作

### 1.yum换源

```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

这段命令的意思是使用 `curl` 工具下载一个 CentOS 7 的软件源配置文件，并将其保存到系统的 `/etc/yum.repos.d/` 目录下，文件名为 `CentOS-Base.repo`。

具体分析：

- `curl`：是一个命令行工具，用于从网络上下载或上传数据。
- `-o /etc/yum.repos.d/CentOS-Base.repo`：`-o` 选项指定下载的文件保存的路径和文件名，这里是 `/etc/yum.repos.d/CentOS-Base.repo`。
- `https://mirrors.aliyun.com/repo/Centos-7.repo`：这是一个 CentOS 7 的阿里云镜像源配置文件的 URL。

这通常用于替换系统默认的 yum 源配置，改用阿里云的镜像源加速软件包的下载速度。

### 2.yum安装基本库

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

这个命令用于在基于 **RPM** 的 Linux 系统（如 CentOS、Red Hat、Fedora）上安装一些工具和库。让我们逐部分分析：

- **`yum install -y`**: 这部分使用 **`yum`** 包管理器来安装软件包。`-y` 表示自动回答“是”来确认所有的安装，不需要用户手动确认。
- **`yum-utils`**: 这是一个工具集，提供了一些扩展的 `yum` 命令，例如查询、清理缓存、解决依赖关系等，方便管理员管理软件包。
- **`device-mapper-persistent-data`**: 这是用于 **Device Mapper** 的持久性数据支持。Device Mapper 是 Linux 内核的一个模块，主要用于创建、管理和映射设备。此包提供一些持久性存储支持，常用于 **LVM**（逻辑卷管理）和 **Docker** 等工具。
- **`lvm2`**: 这是 **LVM (Logical Volume Manager)** 的工具集，它允许你在物理存储设备上创建逻辑卷，以便更灵活地管理存储资源。LVM 提供了动态分配和扩展存储的功能。

总体来说，这个命令安装了一些用于存储管理和工具的包，通常在设置磁盘、管理分区、或使用 Docker 等容器化技术时会需要这些工具。

3.

```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

4.

```
sudo yum makecache fast
```

### 5.安装docker

```
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6.docker基本操作

```
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```

### 7.创建docker网络

```
docker network create hm-net
```



# Docker



## docker pull代理

在执行docker pull时，是由守护进程dockerd来执行。因此，代理需要配在dockerd的环境中。而这个环境，则是受systemd所管控，因此实际是systemd的配置。

```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

在这个proxy.conf文件（可以是任意*.conf的形式）中，添加以下内容：

```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

然后需要重启服务

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```



参考文献

https://cloud.tencent.com/developer/article/1806455



# MobarXterm

全局密码忘记了



# 依赖

## mysql

### 1.pull

```
docker pull mysql:8.0.36
```

### 2.run

```shell
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  -v /root/mysql/data:/var/lib/mysql \
  -v /root/mysql/conf:/etc/mysql/conf.d \
  -v /root/mysql/init:/docker-entrypoint-initdb.d \
  --network hm-net \
  --privileged=true \
  mysql:8.0.36
```

说明：

- **`docker run -d`**:
  - **`docker run`**: 这是启动 Docker 容器的命令。
  - **`-d`**: 容器将在后台运行（即分离模式）。
- **`--name mysql`**:
  - 给启动的容器命名为 `mysql`，这样可以更方便地引用该容器。
- **`-p 3306:3306`**:
  - 将主机的 **3306** 端口映射到容器的 **3306** 端口，这是 MySQL 默认的端口。这意味着你可以通过宿主机的 3306 端口访问 MySQL 数据库。
- **`-e TZ=Asia/Shanghai`**:
  - 设置环境变量 **`TZ`** 为 **Asia/Shanghai**，即容器的时区设置为上海时区。这个设置是为了确保容器内的时间与本地时区一致。
- **`-e MYSQL_ROOT_PASSWORD=123`**:
  - 设置环境变量 **`MYSQL_ROOT_PASSWORD`** 为 **123**，这是 MySQL 容器的根用户（`root`）的密码。在实际使用中，建议设置一个更强的密码。
- **`-v /root/mysql/data:/var/lib/mysql`**:
  - 将宿主机上的 **`/root/mysql/data`** 目录挂载到容器内的 **`/var/lib/mysql`** 目录，这样 MySQL 的数据将保存在宿主机的指定路径下，确保数据持久化，不会因为容器删除而丢失。
- **`-v /root/mysql/conf:/etc/mysql/conf.d`**:
  - 将宿主机上的 **`/root/mysql/conf`** 目录挂载到容器内的 **`/etc/mysql/conf.d`** 目录。这个目录用于 MySQL 的配置文件，挂载后可以让你定制 MySQL 配置（比如调整性能参数、启用插件等）。
- **`-v /root/mysql/init:/docker-entrypoint-initdb.d`**:
  - 将宿主机上的 **`/root/mysql/init`** 目录挂载到容器内的 **`/docker-entrypoint-initdb.d`** 目录。容器启动时，MySQL 会从这个目录执行初始化脚本，比如创建数据库、导入数据等。
- **`--network hm-net`**:
  - 将容器连接到 Docker 网络 **`hm-net`**。这样，其他容器或服务可以通过网络连接到这个 MySQL 容器。
- **`--privileged=true`**:
  - 允许容器拥有更高的权限，基本上给容器更多的特权，类似于宿主机的 root 权限。这个选项通常用于需要访问硬件或进行低级别操作的容器。在大多数情况下不推荐使用，除非有特别需求。
- **`mysql:8.0.36`**:
  - 指定使用的 Docker 镜像是 **MySQL 8.0.36** 版本。如果本地没有该镜像，Docker 会自动从 Docker Hub 拉取该镜像。

这条命令的作用是启动一个配置好的 MySQL 容器，主要包括：

1. 运行 **MySQL 8.0.36** 镜像。
2. 设置时区为上海。
3. 设置 MySQL 根密码。
4. 挂载宿主机目录以持久化数据和配置。（映射这三个目录，防止容器重启之后，数据就随之不见。）
5. 使用指定的网络进行容器间通信。
6. 初始化脚本自动执行。
7. 容器使用特权模式。

通过这个命令，你可以在 Docker 中快速运行并配置一个 MySQL 服务。

## nacos

### 1.pull

```
docker pull nacos/nacos-server:v2.1.0-slim
```

### 2.run

```
docker run -d \
--name nacos \
--env-file /root/nacos/custom.env \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--restart=always \
nacos/nacos-server:v2.1.0-slim
```

`./nacos/custom.env`

（nacos只有这一个文件）

```
PREFER_HOST_MODE=hostname
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_SERVICE_HOST=192.168.32.137
MYSQL_SERVICE_DB_NAME=nacos
MYSQL_SERVICE_PORT=3306
MYSQL_SERVICE_USER=root
MYSQL_SERVICE_PASSWORD=123
MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
```



## rabbitmq

### 1.pull

```
docker pull rabbitmq:3.8-management
```

### 2.run

```
docker run \
 -e RABBITMQ_DEFAULT_USER=itheima \
 -e RABBITMQ_DEFAULT_PASS=123456 \
 --name rabbitmq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 --network hm-net \
 -d \
 rabbitmq:3.8-management
```



## sentinel

### run

```
java -Dserver.port=8090 -Dcsp.sentinel.dashboard.server=localhost:8090 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.6.jar
```



## seata

### 1.pull

```
docker pull seataio/seata-server:1.5.2
```

### 2.run

```
docker run --name seata \
-p 8099:8099 \
-p 7099:7099 \
-e SEATA_IP=192.168.32.137 \
-v /root/seata:/seata-server/resources \
--privileged=true \
--network hm-net \
-d \
seataio/seata-server:1.5.2
```



## elasticsearch

https://www.elastic.co/downloads/past-releases/elasticsearch-7-17-2

### 1.pull

```
docker pull elasticsearch:7.17.2
```

### 2.run

```
docker run -d \
  --name es \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -e "discovery.type=single-node" \
  -v /root/elasticsearch/config/analysis-ik/:/usr/share/elasticsearch/config/analysis-ik/ \
  --privileged \
  --network hm-net \
  -p 9200:9200 \
  -p 9300:9300 \
  elasticsearch:7.17.2

```

要注意：

```
/root/elasticsearch/config/analysis-ik/
一定要给足权限，防止run container失败
```



## ik

https://release.infinilabs.com/analysis-ik/stable/

### install

```
docker exec -it es ./bin/elasticsearch-plugin  install https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.17.2.zip
```

会收到：

```
[root@localhost ~]# docker exec -it es ./bin/elasticsearch-plugin  install https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.12.1.zip
-> Installing https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.12.1.zip
-> Downloading https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.12.1.zip
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.net.SocketPermission * connect,resolve
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed analysis-ik
-> Please restart Elasticsearch to activate any plugins installed
[root@localhost ~]#

```

2.重启es容器

```
docker restart es
```

## kibana

### 1.pull

```
docker pull kibana:7.17.2
```

### 2.run

```
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=hm-net \
-p 5601:5601  \
kibana:7.17.2
```

