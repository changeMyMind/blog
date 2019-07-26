### 1.docker的安装以及启动

```shell
安装：yum install -y docker
启动：systemctl start docker
查看状态信息：systemctl status docker
加入开机启动：systemctl enable docker
简略版本信息：docker --version
详细版本信息：docker version
```

### 2.docker镜像管理
```shell
查询镜像：docker search centos:7
查看镜像：docker images
push镜像：docker pull centos:7
导入/导出镜像：
导出：docker save centos:7 >/opt/centos.tar.gz
导入：docker load </opt/centos.tar.gz
删除镜像：docker rmi $IMAGEID/$TAG
利用镜像创建容器：docker run -it centos:7/bin/bash
```

#### 3.docker容器管理
```shell
在本地执行/bin/echo "hehe" 一样：docker run centos:7 /bin/echo "hehe"
启动一个bash终端，允许用户进行交互
--name: 给容器定义名称
-i: 让容器的标准输入保持打开
-t: Docker分配一个伪终端并绑定到容器的标准输入上
docker run --name mydocker -it centos:7 /bin/bash
启动/停止容器
docker start/stop $NAMES/$CONTAINERID
列出已经启动你那个的容器：docker ps
列出所有容器，包括未启动的：docker ps -a
删除容器：docker rm $CONTAINERID
删除正在运行的容器：docker rm -f $CONTAINERID
```
### 4. 进入容器
```shell
attach 命令进入
docker attach $CONTAINERID
nsenter命令进入
1. 安葬 nsenter
yum install -y util-linux
2.找到容器进程ID
docker inspect --format "{{.State.Pid}}" test
3. 进入容器
nsenter -t 19245 -u -9 -p
-t, --target <pid>     target process to get namespaces from
指定容器的进程ID
-m, --mount[=<file>]   enter mount namespace
进入到mount namespace空间中
-u, --uts[=<file>]     enter UTS namespace (hostname etc)
进入到UTS namespace空间中
-i, --ipc[=<file>]     enter System V IPC namespace
进入到System V IPC namespace空间中
-n, --net[=<file>]     enter network namespace
进入到network namespace空间中
-p, --pid[=<file>]     enter pid namespace
进入到pid namespace空间

编写脚本快速进入
vim docker_id.sh

#/bin/bash
PID=$(docker inspect -f "{{.State.Pid}}" $1)
nsenter -t $PID -m -u -i -n -p
```
### 5. docker安装nginx

```shell
下载最新的nginx(自己查询对应的nginx)
docker pull nginx
启动
docker run -d --name "swallow_nginx" -p 80:80 nginx
关闭
docker stop swallow_nginx

以后启动直接
docker start swallow_nginx
如果是第三方的服务器 记得配置安全组 就是一个外部的防火墙
```

### 6. docker 推送到阿里云
```shell
docker login --username=15010918296 registry.cn-beijing.aliyuncs.com
密码：Wangyan@123

先pull
docker pull registry.cn-beijing.aliyuncs.com/swallow/swallow:[镜像版本号]

push
docker tag [ImageId] registry.cn-beijing.aliyuncs.com/swallow/swallow:[镜像版本号]
docker push registry.cn-beijing.aliyuncs.com/swallow/swallow:[镜像版本号]



sudo docker login --username=15010918296 registry.cn-qingdao.aliyuncs.com
sudo docker pull registry.cn-qingdao.aliyuncs.com/swallow/personal:[镜像版本号]
sudo docker tag [ImageId] registry.cn-qingdao.aliyuncs.com/swallow/personal:[镜像版本号]
sudo docker push registry.cn-qingdao.aliyuncs.com/swallow/personal:[镜像版本号]
```
文章查看地址：https://blog.51cto.com/lzhnb/2153225

#### 7.docker 小计

1. docker容器启动命令没有配置 —restart=always 导致docker 重启的时候容器没有启动

```powershell
docker container update --restart=always 容器名字
```

