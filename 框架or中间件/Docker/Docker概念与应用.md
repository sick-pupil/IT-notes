## 1. 镜像、容器、仓库
- 镜像：镜像是一个只读模板，带有创建Docker容器的指令。通常，一个镜像是基于另一个镜像的，还需要进行一些额外的定制
- 容器：容器是用镜像创建的运行实例，每个容器都可以被启动，开始，停止，删除，同时容器之间相互隔离，保证应用运行期间的安全
- 仓库：集中存放镜像文件的场所
<img src="D:\Project\IT-notes\框架or中间件\Docker\img\docker运行与三大要素.png" style="width:700px;height:350px;" />

## 2. 安装docker
```shell
yum -y update
yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl start docker
docker run hello-world
```

**Linux环境安装Docker**
```sh
# 更新软件包索引
sudo apt-get update
 
# 安装需要的软件包以使apt能够通过HTTPS使用仓库
sudo apt-get install ca-certificates curl gnupg lsb-release

# 添加阿里云官方GPG密钥
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
 
# 写入阿里云Docker仓库地址
sudo sh -c 'echo "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list'

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 验证是否成功安装了docker
sudo systemctl status docker
docker --version

# 修改daemon.json文件，
vim /etc/docker/daemon.json

# daemon.json内容如下：
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://docker.m.daocloud.io",
        "https://cr.console.aliyun.com",
        "https://ccr.ccs.tencentyun.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://github.com/ustclug/mirrorrequest",
        "https://registry.docker-cn.com"
    ]
}

# 重载配置文件，并重启 docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 查看 Registry Mirrors 配置是否成功
sudo docker info
```
## 3. docker常用操作命令
<img src="D:\Project\IT-notes\框架or中间件\Docker\img\docker命令大全.jpg" style="width:700px;height:500px;" />

### 1. 帮助启动类命令
```shell
# 启动docker
systemctl start docker
# 停止docker
systemctl stop docker
# 重启docker
systemctl restart docker
# 查看docker状态
systemctl status docker
# 开机启动
systemctl enable docker
# 查看docker概要信息
docker info
# 查看docker总体帮助文档
docker --help
# 查看docker命令帮助文档
docker 具体命令 --help
```

### 2. 镜像命令
```shell
# 列出本地所有镜像，包含历史镜像
docker images -a
# 只显示镜像ID
docker images -q
# 通过镜像名称搜索某个镜像
docker search xxx
# 通过镜像名称拉取某个镜像
docker pull xxx
# 查看镜像/容器/数据卷所占的空间
docker system df
# 通过镜像ID删除镜像
docker rmi xxx
```

### 3. 容器命令
```shell
# 根据镜像运行容器
docker run --name=xxx -it ubuntu /bin/bash
# docker run 参数
--name=xxx 为容器指定一个新名字
-d 后台作为守护进程运行容器并返回容器ID
-i 以交互模式运行容器，通常与-t同时使用
-t 为容器重新分配一个伪输入终端，通常与-i配合使用
# 端口映射格式为 hostPort:containerPort，分别为宿主机端口与容器端口
-P(大写P) 随机端口暴露映射
-p(小写p) 指定端口暴露映射

# 列出正在运行的容器信息
docker ps
# docker ps 参数
-a 列出当前所有正在运行的容器+历史上运行过的
-l 显示最近创建的容器
-n 显示最近n个创建的容器
-q 静默模式，显示容器编号

# 容器新建、退出、重启
exit 退出后容器停止
ctrl+p+q 退出容器后容器并没有停止
# 启动已停止运行的容器
docker start 容器ID或容器名称
# 重启容器
docker restart 容器ID或容器名称
# 停止容器
docker stop 容器ID或容器名称
# 强制停止容器
docker kill 容器ID或容器名称
# 删除已停止的容器
docker rm 容器ID

# 查看容器日志
docker logs 容器ID

# 查看容器内运行的进程
docker top 容器ID

# 查看容器内部细节
docker inspect 容器ID

# 进入运行中的容器并以命令行进行交互
docker exec -it 容器ID /bin/bash 在容器中打开新的终端，并可以启动新的进程，用exit退出不会导致容器停止
docker attach 容器ID /bin/bash 直接进入容器启动命令的终端，不会启动新的进程，用exit退出会导致容器停止

# 从容器内拷贝文件到主机上
docker cp 容器ID:容器内路径 目的主机路径

# 将容器导入与导出为镜像
docker export 容器ID > 文件名.tar 导出容器内容作为一个tar归档文件
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号 从tar包中的内容创建一个新的文件系统再导入为镜像
```

## 4. docker镜像分层
`docker`镜像是由一系列层来构成的，每层代表`dockerfile`中的一条指令
```text
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
以上`dockerfile`包含四个命令，每个命令都会新创建一个层：`FROM`语句会从`ubuntu:18.04`镜像创建一个层；`COPY`指令会从`docker`客户端的当前目录下添加一些文件；`RUN`指令使用了`make`指令来构建；最后`CMD`是指在容器中运行什么命令

**可写容器层与镜像层**
- 容器而对于`docker`来说，创建新容器时，每一层都会彼此堆叠，可以在基础层的基础上添加新的**可写容器层。对容器的所做的所有更改都将写入到该可写容器层中**。在容器中添加数据或者修改现有数据的所有读写操作都会存储在此可写层中。删除容器后，可写层也会被删除
- 镜像层是只读的

<img src="D:\Project\IT-notes\框架or中间件\Docker\img\docker镜像层与容器层.png" style="width:700px;height:500px;" />

`UnionFS`联合文件系统：`UnionFS`支持对文件系统的修改看作一次提交来一层层叠加，同时可以把不同目录挂载到同一个虚拟文件系统下。`UnionFS`文件系统是`docker`镜像的基础，镜像可以通过分层来进行继承。`UnionFS`一次同时加载多个文件系统，把各层文件系统叠加起来

**docker镜像加载原理**：
- `bootfs`主要包含`bootloader`和`kernel`，`bootloader`主要是引导加载`kernel`，`Linux`刚启动时也会加载`bootfs`文件系统，在`docker`镜像的最底层是`bootfs`，包含`boot`加载器和内核。加载完成后内核就存在于内存中
- `rootfs`在`bootfs`之上，包括最基本的命令、工具和程序库就行，如`/dev /proc /bin /etc`，对应不同的操作系统发行版本

**docker支持从容器构造一个新的镜像：commit命令**
```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
docker commit -a="mrhelloworld" -m="jdk11 and tomcat9" 容器ID mycentos:7

-a：提交的镜像作者
-c：使用dockerfile指令创建镜像
-m：提交时的文字说明
-p：commit时将容器暂停
```

## 5. 容器数据卷
`docker`实现了将文件或者目录（数据卷）挂载到容器中的功能，卷存在于多个容器当中，解决了数据持久化与共享数据的需求
*在容器中创建新增的内容能在宿主机中看到；主机修改数据也会同步到容器中*

容器卷特点：
1. 数据卷可在容器之间共享或重用数据
2. 数据卷中的更改不会包含在镜像的更新中
3. 卷中的更改可以直接生效
4. 数据卷的生命周期一直持续到没有容器使用它为止

```shell
# 卷创建命令
docker volume create myvolume # 默认卷为local模式，仅允许本主机的容器访问
docker run -it -v 宿主机绝对路径目录:容器内目录 镜像名 # 使用-v的方式指定容器内需要被持久化的路径，Docker会自动为我们创建卷，并且绑定到容器中
docker run -it -v 宿主机绝对路径目录:容器内目录:ro 镜像名 # 挂载的数据卷内部只能由宿主机写
docker run -it -v 宿主机绝对路径目录:容器内目录:rw 镜像名 # 宿主机与容器双方都有读写的权限
docker run -d -P --name web -v my-vol:/wepapp --mount source=my-vol,target=/webapp training/webapp python app.py #创建一个容器，同时创建一个卷，并将卷挂载入容器中

# 卷删除命令
docker volume rm myvolume # 容器删除的同时并不会一带删除容器创建的数据卷，因此数据卷需要手动删除

# 查看宿主机目录相关的卷信息
docker volume inspect 宿主机目录

# 查看宿主机的卷目录
docker volume ls
```

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用**数据卷容器**。数据卷容器
也是一个容器，但是它的目的是专门用来提供数据卷供其他容器挂载
```shell
docker run -d --name data-volume -v /data/nginx:/usr/share/nginx/html -v /data/mysql:/var/lib/mysql centos:7.8.2003

docker run -itd --name nginx01 -p 80:80 --volumes-from data-volume nginx:1.19.3-alpine
```

## 6. dockerfile
`docker`执行`dockerfile`大致流程
1. 从基础镜像中运行一个容器
2. 按照顺序执行`dockerfile`中的每一条指令，对容器作出修改
3. 每执行一条指令，则进行一次`commit`提交一个新的镜像层
4. 继续执行指令直至执行完成

<img src="D:\Project\IT-notes\框架or中间件\Docker\img\dockerfile构建docker镜像过程.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\框架or中间件\Docker\img\执行dockerfile.png" style="width:700px;height:300px;" />

**dockerfile保留字指令**
- `FROM`：指定基础镜像
- `MAINTAINER`：镜像维护者姓名及邮箱地址
- `RUN`：容器构建时需要运行的命令
- `EXPOSE`：当前容器对外暴露的端口号
- `WORKDIR`：指定在创建容器后，终端默认登录进来的工作目录
- `ENV`：用来在构建镜像过程中设置环境变量
- `ADD`：将宿主机目录下的文件拷贝进镜像，`ADD`命令会自动处理`URL`和解压`tar`压缩包
- `COPY`：拷贝文件、目录到镜像中。具体是将从构建上下文目录中<src原路径>的文件或目录复制到新一层镜像的<目标路径>位置 ，有两种写法：`COPY src dest` 或者 `COPY ["src", "dest"]`
- `VOLUME`：容器数据卷，用于数据保存和持久化工作
- `CMD`：指定一个容器启动时要运行的命令，`CMD`与`RUN`相似，存在`shell`与`exec`格式
- `ENTRYPOINT`：指定一个容器启动时要运行的命令，与`CMD`一样都是在指定容器启动程序及参数
- `ONBUILD`：当构建一个被继承的`DockerFile`时运行命令， 父镜像在被子镜像继承后，父镜像的ONBUILD被触发

<img src="D:\Project\IT-notes\框架or中间件\Docker\img\dockerfile指令.png" style="width:700px;height:400px;" />

```shell
FROM centos
MAINTAINER lihongcheng<xxx@163.com>
ENV ETCPATH /etc
WORKDIR $ETCPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 5000
CMD echo "-----successful------"
CMD /bin/bash

FROM centos
MAINTAINER lihongcheng<xxx@163.com>
RUN yum -y install curl
CMD ["curl", "-s", "http://ip.cn"]
```

`RUN`、`CMD`、`ENTRYPOINT`的区别：
- `RUN`命令执行命令并创建新的镜像层，通常用于安装软件包
- `CMD`命令设置容器启动后默认执行的命令及其参数，但`CMD`设置的命令能够被`docker run`命令后面的命令行参数替换
- `ENTRYPOINT`配置容器启动时的执行命令（不会被忽略，一定会被执行，即使运行`docker run`时指定了其他命令），且`ENTRYPOINT`可使用`docker run`以及`CMD`的命令作为参数使用

## 7. docker network
`docker`默认内置三个网络

| network id | name | driver |
| ----- | ----- | ----- |
| 7fca4eb8c647 | bridge | bridge |
| 9f904ee27bf5 | none | null |
| cf03ee007fb4 | host | host |

`docker run -network=<NETWORK>`可以指定容器的网络模式，还存在以下四种网络模式
- `host`模式：`--net=host`，与宿主机在同一个网络中，但没有独立IP地址
- `none`模式：`--net=none`，关闭了容器的网络功能
- `bridge`模式：`--net=bridge`，连接到`docker0`虚拟网卡，通过`docker0`网桥以及`Iptables nat`表配置与宿主机通信
- `container`模式：`--net=container:NAME_or_ID`，不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围

## 8. docker compose
`docker compose`：用于定义和运行多容器`docker`应用程序的工具。`compose`可以使用`yaml`文件配置应用程序所需要的服务，使用命令就可以通过`yaml`配置文件创建并启动所有服务

`docker-compose.yml`文件定义了一组相关联的容器为一个工程，一个工程包含多个服务，每个服务中又定义了创建容器所需的镜像、参数、依赖

使用`docker compose`的三个步骤：
1. 使用`dockerfile`文件定义应用程序的环境
2. 使用`docker-compose.yml`文件定义构成应用程序的服务，这样它们可以在隔离环境中一起运行
3. 最后执行`docker-compose up`命令创建并启动所有服务

### 示例docker-compose.yml
```yaml
# 描述 Compose 文件的版本信息
version: "3.8"
# 定义服务，可以多个
services:
  nginx: # 服务名称
    image: nginx # 创建容器时所需的镜像
    container_name: mynginx # 容器名称，默认为"工程名称_服务条目名称_序号"
    ports: # 宿主机与容器的端口映射关系
      - "80:80" # 左边宿主机端口:右边容器端口
    networks: # 配置容器连接的网络，引用顶级 networks 下的条目
      - nginx-net

# 定义网络，可以多个。如果不声明，默认会创建一个网络名称为"工程名称_default"的 bridge 网络
networks:
  nginx-net: # 一个具体网络的条目名称
    name: nginx-net # 网络名称，默认为"工程名称_网络条目名称"
    driver: bridge # 网络模式，默认为 bridge
```

```shell
# 前台启动
docker-compose up
# 后台启动
docker-compose up -d
# 停止并删除容器、网络
docker-compose down
```

### `compose`文件配置
1. `version`：`compose`版本
2. `services`：
	- 服务名称，自定义
		- `image`，镜像名称与标签
		- `container_name`，容器名称
		- `ports`，宿主机端口:容器端口
		- `environment`：创建容器时的环境变量
		- `volumes`，容器卷
		- `build`，根据所给路径执行`dockerfile`
			- `context`，`dockerfile`所在目录
			- `dockerfile`，文件名称
		- `depends_on`：该容器在哪些依赖容器后才启动执行
		- `restart`：有四个选项，`no on-failure always unless-stopped`决定容器是否跟随`docker`服务同时启动
		- `networks`：连接网络
3. `networks`：
	- 网络条目名称，自定义
		- `name`：网络名称
		- `driver`：网络模式，`bridge host none`

### `compose`命令
`docker-compose`命令基本使用格式：`docker-compose [-f or --file -p or --project-name --x-networking --x-network-driver --verbose -v] [command] [args]`
1. `-f or --file`：指定使用的`compose`模板文件
2. `-p or --project-name`：指定项目名称
3. `--x-networking`：使用`docker`可插拔网络特性
4. `--x-network-driver`：指定后端驱动，默认`bridge`
5. `--verbose`：输出更多调试信息
6. `-v or --version`：打印版本并退出

`docker-compose`具体命令：
- `config`：验证`docker-compose`文件配置正确与否
- `pull`：拉去服务依赖的镜像
- `up`：创建并启动所有服务的容器
- `logs`：查看服务容器的输出日志，可以使不同服务输出不同颜色的日志
- `ps`：列出工程中的所有服务容器
- `run`：在执行服务中的某个容器上执行一个命令
- `exec`：进入服务容器
- `pause`：暂停容器
- `unpause`：恢复被暂停容器
- `restart`：重启容器
- `start`：启动容器
- `stop`：停止容器
- `kill`：杀死容器
- `rm`：删除容器
- `down`：停止被`up`命令启动的容器，并移除网络
- `images`：列出`compose`文件包含的镜像
- `ports`：设置端口映射

## 9. EntryPoint.sh
许多`dockerfile`内的`entrypoint`如下：`ENTRYPOINT ["docker-entrypoint.sh"]`

举例：
`MySQL`容器启动时，`docker-entrypoint-initdb.d/docker-entrypoint.sh`会初始化自定义数据库
```
When a container is started for the first time, a new database with the specified name will be created and initialized with the provided configuration variables. Furthermore, it will execute files with extensions `.sh`, `.sql` and `.sql.gz` that are found in `/docker-entrypoint-initdb.d`. Files will be executed in alphabetical order. You can easily populate your `mysql` services by [mounting a SQL dump into that directory⁠](https://docs.docker.com/storage/bind-mounts/) and provide [custom images⁠](https://docs.docker.com/reference/dockerfile/) with contributed data. SQL files will be imported by default to the database specified by the `MYSQL_DATABASE` variable.
```

再如`redis`：
```sh
#!/bin/bash
if [[ $redis_ip ]]; then
	sed -i 's/redis_ip="[0-9.]*"/redis_ip="'$redis_ip'"/' config.ini
fi
if [[ $redis_port ]]; then
	sed -i 's/redis_port="[0-9]*"/redis_port="'$redis_port'"/' config.ini
fi
echo "1" > /proc/sys/kernel/core_uses_pid
echo $CORE_PATH"/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
exec "$@"
```

```sh
docker run -d --restart=always \
  --ulimit core=-1 --privileged=true\
  -e redis_ip=$REDIS_IP \
  -e redis_port=$REDIS_PORT \
  xxx
```