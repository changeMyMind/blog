# docsify安装使用

> just for fun 别认真哈 具体查看官方文档哈 [docsify](https://docsify.js.org/#/zh-cn/)  （要是你喜欢看英文文档就换语言呗）

### 1. 安装对应的依赖

- 需要安装npm （个人跟喜欢cnpm）

```shell
# linux 安装 选择安装的目录
mkdir /usr/local/node && cd /usr/local/node
# 在对应的地址中找到对应的tar.gz包
wget https://npm.taobao.org/mirrors/node/v6.9.0/node-v6.9.0-linux-x64.tar.gz
# 解压
tar -zxvf node-v6.9.0-linux-x64.tar.gz
# 建立软连接 这个很重要很重要 不然全局使用npm不能使用（当然有别的办法 可惜我不知道）
# 还有就是 记得两个都是绝对路径 不要搞事情 知道吧 会出问题的 你可以试一试相对路径
# 可能需要权限 sudo 一下 输入密码 就ok了
ln -s /usr/local/node/node-v6.9.0-linux-x64/bin/npm /usr/local/bin/npm
ln -s /usr/local/node/node-v6.9.0-linux-x64/bin/node /usr/local/bin/node
# 查看下版本 看看有木有问题(有问题 就是你倒霉了 自己研究去吧)
npm -v
```

- cnpm 安装（不喜欢可以不用呀 你网快 你牛逼）

```shell
# 安装
npm install -g cnpm --registry=https://registry.npm.taobao.org
# 建立软连接
ln -s /usr/local/node/node-v6.9.0-linux-x86/bin/cnpm /usr/local/bin/cnpm
# 查看版本 
cnpm -v
```

### 2. 依赖安装完后执行命令

```shell
# 安装docsify
cnpm i docsify-cli -g
# 创建目录 也就是项目目录 mkdir [project-path]
mkdir docs
# 初始化
docsify init docs
# 运行 docsify server [project-path] 
#  如果不知道命令的 一个一个 的 --help 就行了 这个很牛逼 要 会自学哦 不如自定义端口号
docsify serve docs
```