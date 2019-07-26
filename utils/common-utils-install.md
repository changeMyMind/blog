# 常用工具安装

### jdk下载以及环境搭建

1. jdk下载

   访问地址 https://github.com/frekele/oracle-java/releases 找到对应的版本下载即可

2. 解压、配置（windows自行安装不想bb）

```shell
# 创建目录
mkdir /usr/local/jdk && cd /usr/local/jdk
# wget获取对应的资源 忽略了 可以上传到码云里面哦（嘻嘻嘻）
# 解压 忽略哈
# 配置环境变量
vi /etc/profile
# 在最后添加 java_home 自己改改哦
export JAVA_HOME=/usr/local/jdk/jdk1.8.0_111
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

### idea下载安装以及配置

1. 官网直接下载安装 下载老版本的 最新的破解不了
2. 破解 [链接](http://idea.lanyus.com/)

3. 常用插件推荐
   1. Lombok
   2. Maven Helper
   3. MybatisX
   4. GenerateAllSetter
   5. RestfulToolkit
   6. Translate
   7. GsonFormat
   8. CamelCase

### git安装使用

1. git 下载安装
   - 从某个地方下载哈 https://mirrors.edge.kernel.org/pub/software/scm/git/ 可以自己获取对应的版本哦

```shell
# 下载依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
# 移除旧版本
yum remove git # 如果没有删除 可以使用 rpm -qa| grep git 查看 然后rpm -e [filename] 删除
# 进入对应的目录
mkdir /usr/local/git && cd /usr/local/git
# 下载
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
# 解压
tar -zxvf git-2.9.5.tar.gz
# 进入对应目录
cd git-2.9.5
# 配置安装路径
./configure prefix=/usr/local/git/
# 编译安装
make && make install
# 将git指令添加到bashrc中
vi ~/.bashrc
# 在最后一行加入 保存退出 并使其生效
export PATH=$PATH:/usr/local/git/bin
source ~/.bashrc
# 如果git pull出现一下错误 fatal: unable to access 'https://github.com/google/glog.git/': SSL connect error  直接更新对应的包
yum update -y nss curl libcurl
```

### maven下载安装配置

1. 官方下载并放入 `/usr/local/maven`中
2. 解压配置/etc/profile

```shell
# vi /etc/profile 最后一行加入
export MAVEN_HOME=/usr/local/maven/对应的路径
export PATH=${PATH}:${MAVEN_HOME}/bin
# 保存退出并source 使其生效
source /etc/profile
```

3. setting.xml 配置镜像

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

### 其他工具官方下载然后安装即可

#### postman 

#### typroa 

#### vscode

#### [iterm2/os](utils/iterm2.md)、xshell/windows

#### FileZilla

#### [navicat需破解](utils/navicat.md) 

