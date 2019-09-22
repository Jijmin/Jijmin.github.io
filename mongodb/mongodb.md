### 使用docker容器运行mongodb
```shell
# 下载mongodb的官方镜像
$ docker pull mongo:4

# 查看下载的镜像
$ docker images

# 暂停mongo容器
$ docker ps
$ docker stop <CONTAINER ID>
$ docker rm <CONTAINER ID> # 直接删除掉这个ID

# 启动一个mongodb服务容器
$ docker run --name mymongo -v /Users/zhouying/dockerCacheFile/mongo/data:/data/db -d mongo:4
--name mymongo --> 容器名字
-v /mymongo/data:/data/db --> 挂载数据目录
-d --> 后台运行容器

# 查看docker容器状态
$ docker ps

# 查看数据库服务器日志
$ docker logs mymongo

# mongo express 是一个基于网络的mongodb数据库管理界面
# 下载mongo-express镜像
$ docker pull mongo-express

# 运行mongo-express
$ docker run --link mymongo:mongo -p 8081:8081 mongo-express

# mongo shell 是用来操作mongodb的javascript客户端界面
# 运行mongo shell
$ docker exec -it mymongo mongo

# 使用test数据库
> use test

# 查看test数据库中的集合
> show collections

# 开始创建第一个文档
db.accounts.insertOne(
  {
    _id: "account1",
    name: "alice",
    balance: 100
  }
)

## ......中间视频缺失【读取文档】关于读取文档 你需要知道这些事【读取文档】动手实战 - 最直接的匹配查询和比较操作符【读取文档】动手实战 - 逻辑操作符和字段操作符

# 筛选文档
# 数组操作符
# $all 匹配数组字段中包含所有查询值的文档
# $elemMatch 匹配数组字段中至少存在一个满足筛选条件的文档
# $regex 匹配满足正则表达式的文档

error pulling image configuration: Get https://production.cloudflare.docker.com/registry-v2/docker/registry/v2/blobs/sha256/cd/cdc6740b66a71617b310e476e9b08d68a3be7af75beee27139dc8bc3d35ef4c5/data?verify=1567272108-cArzwTWJNm3uEiFxdG7qY6u1c6o%3D: Service Unavailable
```