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

   



## 参考连接

https://www.pianshen.com/article/827378623/

https://www.cnblogs.com/whutxldwhj/p/6427530.html

https://www.runoob.com/docker/docker-install-centos.html