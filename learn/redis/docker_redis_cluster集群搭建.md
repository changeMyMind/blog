> 本人写这玩意的时候学shell编程时间不长 如果感觉不足的地方希望能好心提醒下 一起进步哈 不要为难菜鸟呀

#### 1. docker pull redis镜像

> 注意 本教程是redis5.0+的哦 因为5.0一下的需要自行配置ruby 和 ruby-redis 的api 具体查看 [redis集群搭建](learn/redis/redis集群搭建.md)

```powershell
# 后期修改成自己维护的镜像仓库 如 阿里云镜像仓库
docker pull redis
```

#### 2. 单机启动redis 容器

> 最后实验还是带上redis.conf 比较好 因为容器重启的时候你总不能修改相应的docker 启动命令重新来一遍吧 虽然说线上基本不改，但是不太灵动的玩意在集群环境中如果删除重新添加节点还是比较麻烦的

```powershell
# 启动
docker run --name redis_6379 -p 6379:6379 --rm \
	-v ~/redis/data:/data \
	-d redis:latest --appendonly yes
# 查看 或者docker ps -a 此命令列出所有的容器包括未启动的
docker ps 
# 进入容器 
docker exec -it redis_6379 /bin/bash
```

#### 3.创建network 为了关联redis集群机 如果是多台机子记得 直接关联 host 哦

```powershell
# 创建network
docker network create redis-net
# 查看
docker network ls
```

#### 4.copy 相应的redis.conf文件 重命名为redis-cluster.tmpl  脚本生成配置文件

```powershell
#!/bin/bash
clusterPasswd=""
redisPasswd=""
passwd=$1
if [ "$passwd" != "" ]
then
    clusterPasswd=masterauth" "$passwd
    redisPasswd=requirepass" "$passwd
fi
dname=$(dirname "$PWD")  #上级目录
backDir=$(date "+%Y%m%d%H%M%S")
for port in `seq 7000 7005`
do
    if [ -d $dname/${port} ]
    then
        # echo "存在目录"
        echo "备份数据中"
        mkdir -p $dname/back/$backDir \
        && zip -r ${port}.zip $dname/${port} && mv ${port}.zip $dname/back/$backDir
        echo "数据移动到：$dname/back/$backDir"
        rm -rf $dname/${port}
    fi
    mkdir -p $dname/${port}/conf \
    && redisPasswd=$redisPasswd clusterPasswd=$clusterPasswd envsubst < $dname/redis-cluster.tmpl > $dname/${port}/conf/redis.conf \
    && mkdir -p $dname/${port}/data
done
```

#### 5.创建多个redis的docker容器

```powershell
#拼接数据
dname=$(dirname "$PWD")  #上级目录
# 自定以网关或者创建
read -p "输入network：(如redis-net)" nets
netsExist=$(docker network ls | grep "redis-net")
if [ "$netsExist" = "" ]
then
    docker network create $nets
fi
mst="";
for port in `seq 7000 7005`
do
    docker run --name redis_cluster_${port} -p ${port}:6379 -p 1${port}:16379 --restart always \
        -v $dname/${port}/conf/redis.conf:/usr/local/redis/redis.conf \
        -v $dname/${port}/data:/data \
        --net $nets \
        -d redis:latest redis-server /usr/local/redis/redis.conf
    mst=$mst$(docker inspect --format '{{(index .NetworkSettings.Networks "redis-net").IPAddress}}' redis_cluster_$port):6379" "
done
```

#### 6.进入容器启动集群

```powershell
# 进入某个容器后执行
passwd=$1
if [ "$passwd" != "" ]
then 
    docker exec -it redis_cluster_7000 redis-cli -a $passwd --cluster create --cluster-replicas 1 $mst 
else
    docker exec -it redis_cluster_7000 redis-cli --cluster create --cluster-replicas 1 $mst 
fi
```

> 是不是感觉很奇怪是的 就是不让你运行起来 嘻嘻嘻 因为我有一个分装好的启动脚本哦

#### 7. 分装的启动脚本

```powershell
#!/bin/bash
read -p "初始化会删除所有数据文件，请做好备份：(yes 执行)" pot
if [ "$pot" == "yes" ]
then
    read -p "请输入redis集群密码：" passwd
    sh docker_redis_remove.sh
    sh docker_redis_predata.sh $passwd
    sh docker_redis_create.sh $passwd
fi
```

> 是的没错就是这样 你还是不知道其他脚本在哪 嘻嘻嘻 下载吧 慢慢看

#### 8.脚本地址

```powershell
https://gitee.com/swallowying/docker_env/tree/master
```

#### 9. 脚本启动docker集群

```powershell
# 网络地址 https://gitee.com/swallowying/docker_env/tree/master
# 解压 没unzip 就自己想办法 如果目录不对 自己看看 
unzip redis_cluster.zip && cd cluster/shell
# 初始化集群脚本 其他脚本自己看了
sh docker_redis_init.sh
```

#### 10.小作业

```powershell
有些地方没有做的哦 就是节点的管理啥的 以后慢慢更新点小玩意
```





