#### 阿里云服务器

---

| 服务器      | 外网ip        | 内网ip         | 密码            |
| ----------- | ------------- | -------------- | --------------- |
| 华北1(青岛) | 47.105.54.177 | 172.31.112.177 | swallow@ying92? |
| 华北2(北京) | 47.95.112.95  | 172.17.213.187 | Wangyan@123     |



#### 服务

47.95.112.95

| 服务名称           | 服务端口   | 服务基础 |
| ------------------ | ---------- | -------- |
| nginx              | 80         | docker   |
| elasticsearch      | 9200、9300 | docker   |
| elasticsearch-head | 9100       | 非docker |



#### 注意

elasticsearch-head需要修改链接地址 不然请求有问题 

修改head目录下的Gruntfile.js文件 在93行添加hostname:"*"

修改head目录下_site/目录下的app.js文件,把下面红框中的地址换为ES的地址

修改ES目录下的config目录下elasticsearch.yml文件,增加

http.cors.enabled: true
http.cors.allow-origin: "*"

cd 到head目录下(必须是主目录需要Gruntfile.js 文件).把nodejs启动:

grunt server

然后访问：http://${hostname}:9100



jar包 启动 监听

nohup java 

 -Xmx5g -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005

-Dspring.profiles.active=preprodtest 

-Dcom.sun.management.jmxremote.ssl=false 

-Dcom.sun.management.jmxremote.authenticate=false 

 -Dcom.sun.management.jmxremote.port=10001 

-Djava.rmi.server.hostname=192.168.2.142 -jar  /home/production/config/s-1.0.jar  >./import.log 2>&1 &