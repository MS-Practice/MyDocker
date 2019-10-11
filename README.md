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

