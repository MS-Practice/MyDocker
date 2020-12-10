# Docker 常用命令

```
docker images	//查看所有镜像列表
docker rmi image-id	//删除指定镜像
docker rmi $(docker images -q)

docker ps	// 查看正在运行的容器
docker ps -a // 查看所有容器
docker stop container-id	// 停止指定容器
docker stop $(docker ps -a -q)	// 停止所有容器
docker rm container-id	// 删除指定容器
docker rm $(docker ps -a -q)	// 删除所有容器

docker inspect container-id	// 查看指定容器详细信息
```



## Docker 连接 MSSQL

```
docker run -e ACCEPT_EULA=Y -e SA_PASSWORD=YourPassword -p 1433:1433 --name sqlserver2017_efcore_sample -d microsoft/mssql-server-linux:2017-latest	// 运行容器，如果没有 mssql 镜像会自动 pull 对应的镜像，这个时候就可以用 ssms 连接 docker 中的数据库了


docker exec -it container-id bash	// 进入具体容器
```

## Docker 继承 Jenkins

```dockerfile
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins_home jenkins/jenkins
// 上面在运行 jenkins 时要注意，一定要先在docker内部绑定到8080端口，然后映射到外部其他端口，否则启动之后会发生能访问 localhost:50000 而无法访问 8080 端口
// 执行 command
docker exec -it <container_name/ID>

```

注意，在 dockerhub 你查到的 jenkins 很有可能是一个已经废弃的仓库（废弃仓库名：jenkins）。你运行jenkins之后初始化插件发现会有很多装不上。这个时候你就要重新拉取新的仓库

```
docker pull jenkins/jenkins
```



https://github.com/jenkinsci/docker/issues/787



# Docker 安装 CentOS

docker hub 拉去镜像

```shell
docker pull centos:centos7
```

查看本地的镜像，然后找到拉取对应的 centos 版本运行

```shell
docker images
docker run -itd --name centos-ms centos:centos7
docker exec -it centos-ms /bin/bash
```

## 检查是否安装 ifconfig 和 ssh

直接输入 ifconfig 和 ssh，如果显示不识别命令，则说明没有安装

```
yum search ifconfig
yum install net-tools.x86_64
```

## 查看 ssh 服务

```
rpm -qa |grep sshd
rpm -qa |grep ssh
```

如果没有列表信息弹出，说明要安装 ssh 服务

```
yum install -y openssh-server
```

## 启动 sshd

```
/usr/sbin/sshd -D &
```

如果发现报错，提示 ssh_host\_***_key 不存在，则需要添加对应的文件

```
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ssh-keygen -t rsa -f /etc/ssh/ssh_host_ecdsa_key
ssh-keygen -t rsa -f /etc/ssh/ssh_host_ed25519_key
```

## 修改 sshd_config 

```shell
vi /etc/ssh/sshd_config
# UsePAM yes 改为 UsePAM no 
# UsePrivilegeSeparation sandbox 改为 UsePrivilegeSeparation no
```

然后重新启动sshd

```
/usr/sbin/sshd -D &
```

## 验证 sshd

```
ps -ef | grep sshd
```

## 修改ssh连接密码

```
passwd root
```



## 安装 lsof

```
yum -y install lsof
lsof -i:22
```

## 添加启动程序

```
vi run.sh
```

然后添加如下脚本内容

```
#/bin/bash
/usr/sbin/sshd -D &
```

修改脚本执行权限

```
chmod 755 run.sh
exit #退出
```

## 提交前面作出的修改

```
docker commit 容器id sshd_centos
docker run -p 10022:22 -d sshd_centos /usr/sbin/sshd –D
```

最后用 ssh 工具连接本机ip+端口号连接即可

## Mac 电脑下连接 Docker 中的 CentOS

在做好上面的步骤之后，可以用 Mac 自带的终端来连接 CentOS

"终端" -> "新建命令"，然后输入如下命令

```shell
ssh -p 10022 root@192.168.0.103
```

注意！mac 下的终端在连接服务器时，每次都会循环密码。

## 推送自己的镜像

1. 首先将镜像名 tag 为 dockerhub 的账户名，假设我现在的镜像名为 ms_docker_demo，那么

   ```shell
   docker tag ms_docker_demo marsonshine/ms_docker_demo
   ```

2. 登录

   ```
   docker login
   ```

3. 推送

   ```
   docker push marsonshine/ms_docker_demo
   ```

# Docker 安装 Mysql

```cmd
docker pull mysql	# 下载 mysql 镜像
docker run -itd --name ms-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql	# 运行容器，设置 mysql 密码
docker exec -it mysql bash	# 运行 mysql bash 进入 mysql 容器内部
mysql -u root -p	# 以 root 身份进入 mysql cli
```
# Docker 中的容器启用代理

如果是通过 docker 运行的服务器，如 CentOS，那么我们在安装某些软件的时候，下载速度非常慢，这个时候我们可以使用代理，具体方法在 `/etc/yum.conf` 添加如下结点

```shell
proxy=http://127.0.0.1:7777
```

但是这里有个问题，因为 docker 内部是无法访问 `http://127.0.0.1` 这个ip，它会报 `Connection refuse` ，解决方案也是有的，将 `http://127.0.0.1` 改成 `host.docker.internal` 即可

```shell
proxy=host.docker.internal:7777
```

# 关于 Docker 端口映射问题

先将我目前的需求：

1. 我在 docker 拉取了一个centos镜像并启动 docker run -dit sshd_centos /bin/bash
2. 我的想法是在把 1 启动的centos作为一个虚拟服务器，我要布署多个 app，比如我在centos中安装并运行了consul，在容器内部运行为 localhost:8500，那么问题是我该如何做到在我本机访问这个docker中的centos中的8500端口？

当我直接启动一个 ip 时

```shell
docker run -p 10022:22 -d sshd_centos /usr/sbin/sshd –D
```

如果我通过 ssh 远程连接 centos，安装并运行

```shell
ssh -p 10022 root@192.168.3.67
consul agent -dev -config-dir=./consul.d
```

consul 运行之后会服务器内部暴露一个 8500 端口的 ui 界面，但是这个容器由于已经在运行的时候进行了一个端口映射：本地 10022 端口映射容器内部 22 端口。所以在本机是无法以 `192.168.3.67:8500` 访问 ui 界面的。

后经同事提醒，docker 本就推荐一次成型，在运行之后就不要做改动。所以我就没有尝试继续下去，转而直接 commit 正在运行的容器，然后再次运行并指定 8500 端口。

```shell
docker commit 95527e8116d6 marsonshine/consul_demo
docker run -d -p 8500:8500 marsonshine/consul_demo
```

运行之后点击浏览器访问发现还是无法访问，这是因为运行的容器还没有启动 consul，之前启动 consul 是在第一个容器中启动的，这是一个新容器，需要进去容器运行，有两种方法：

```shell
# 方法1：直接进入 consul 宿主环境，也就是 centos
# 然后启动 consul
docker exec -it clever_poitras /bin/bash
consul agent -dev -config-dir=./etc/consul.d 

# 方法2：直接运行 consul 启动命令脚本
docker exec -it clever_poitras /bin/consul agent -dev -config-dir=./etc/consul.d
```

# Docker 安装运行 Rabbitmq

docker 镜像地址：https://hub.docker.com/_/rabbitmq

```cmd
docker run -d --hostname my-rabbit --name some-rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

## RabbitMQ 集群搭建

```bash
docker run -d --hostname rabbit1 --name my-rabbit1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management

# 将rabbit2节点加入到 rabbit1 集群中
docker run -d --hostname rabbit2 --name my-rabbit2 -p 15673:15672 -p 5673:5672 --link my-rabbit1:rabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management

# 将rabbit3节点加入到 rabbit1 集群中
docker run -d --hostname rabbit3 --name my-rabbit3 -p 15674:15672 -p 5674:5672 --link my-rabbit1:rabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management

## 注意，在最新版 3.8.9 中，`-e RABBITMQ_ERLANG_COOKIE`已经过时，请用 `--erlang-cookie` 替换

# 上面的设置在我的本地有异常，两个节点可以连接成功，但是最后一个连接不上，报 rabbit2 节点与 rabbit3 节点无法连接
# 所以我将第三个节点启动节点改成如下形式
docker run -d --hostname rabbit3 --name my-rabbit3 -p 15674:15672 -p 5674:5672 --link my-rabbit1:rabbit1 --link my-rabbit2:rabbit2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management;
```

注意，设置集群时，命令行中的 `--link` 是必不可少的，而集群中必须要使用相同的 cookie，所以命令行中的 cookie 设置也是必须的。

```bash
docker exec -it my-rabbit1 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app

docker exec -it my-rabbit2 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbit1
rabbitmqctl start_app

docker exec -it my-rabbit3 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbit1
rabbitmqctl start_app
```

注意，rabbitmq 集群分为两种，一种是普通集群，只有两个节点。

还有一种高可用的镜像集群，只要开启一下命令即可快速进入镜像集群模式：

```bash
docker exec my-rabbit1 rabbitmqctl set_policy ha "." '{"ha-mode":"all"}'
```



# Docker 安装运行 Kafka

运行 Kafka 之前先要安装 Zookeeper

```shell
docker pull wurstmeister/zookeeper
```

拉取 kafka

```
docker pull wurstmeister/kafka
```

运行 Zookeeper

```
docker run -d zookeeper -p 2181:2181 -t wurstmeister/zookeeper
```

启动 Kafka

```
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.3.67:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.3.67:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
# 其中的 192.168.3.67 是你的本地局域网 IP
```

进入 kafka 容器内部操作命令

```
docker exec -it kafka /bin/bash
```

进入 kafka 的命令所在目录

```
cd opt/kafka_version_id	# 我的版本号是 kafka_2.13-2.6.0
```

创建 kafka topic

```
./bin/kafka-topics.sh --create --zookeeper 192.168.3.67:2181 --replication-factor 1 --partition 1 --topic mykafka
```

查看服务器所有的 topic 集合

```
./bin/kafka-topics.sh --list --bootstrap-server 192.168.3.67:9092
```

查看指定 topic 详情

```
./bin/kafka-topics.sh --bootstrap-server 192.168.3.67:9092 --describe --topic mykafka
```

创建生产者

```
./bin/kafka-console-producer.sh --broker-list 192.168.3.67:9092 --topic mykafka
```

另开一个 cmd 窗口创建消费者

```
./bin/kafka-console-consumer.sh --zookeeper 192.168.3.67:2181 --topic mykafka
```

在生产者输入消息之后会在消费者窗口输出。

# Docker 安装 ELK

```powershell
docker pull sebp/elk

docker run -dit --name elk -p 5601:5601 -p 9200:9200 -p 5044:5044 sebp/elk
```



## 参考连接

https://www.pianshen.com/article/827378623/

https://www.cnblogs.com/whutxldwhj/p/6427530.html

https://www.runoob.com/docker/docker-install-centos.html