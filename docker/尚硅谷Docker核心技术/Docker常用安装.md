### Docker 常用安装
1. 总体步骤
2. 安装 tomcat
3. 安装 mysql
4. 安装 redis

### 总体步骤
1. 搜索镜像
2. 拉取镜像
3. 查看镜像
4. 启动镜像
5. 停止容器
6. 移除容器

### 安装 tomcat
1. docker hub 上面查找 tomcat 镜像`docker search tomcat`
2. 从 docker hub 上拉取 tomcat 镜像到本地`docker pull tomcat`
3. docker images 查看是否有拉取到的 tomcat`docker images`
4. 使用 tomcat 镜像创建容器(也叫运行镜像)`docker run -it -p 8080:8080 tomcat`

### 安装 mysql
1. docker hub 上查找 mysql 镜像`docker search mysql`
2. 从 docker hub 上(阿里云加速器)拉取 mysql 镜像到本地标签为 5.6`docker pull mysql:5.6`
3. 使用 mysql5.6 镜像创建容器(也叫做运行镜像)
```
docker run -p 12345:3306 --name mysql
-v /Users/zhouying/Desktop/workspace/project/docker-demo/docker-mysql/conf:/etc/mysql/conf.d
-v /Users/zhouying/Desktop/workspace/project/docker-demo/docker-mysql/logs:/logs
-v /Users/zhouying/Desktop/workspace/project/docker-demo/docker-mysql/data:/var/lib/mysql
-e MYSQL_ROOT_PASSWORD=123456
-d mysql:5.6
```
  - 使用 mysql 镜像
  ```
  docker exec -it MySQL运行成功后的容器ID /bin/bash
  mysql -uroot -p123456
  ```
  - 外部 win10 也来连接运行在 docker 上的 mysql 服务
  - 数据备份小测试(可以不做)
  ```
  docker exec 容器ID sh -c 'exec mysqldump --all-databases -uroot -p"123456"' > /zy/all-databases.sql
  ```

### 安装 redis
1. 从 docker hub 上(阿里云加速器)拉取 redis 镜像到本地标签为3.2
2. 使用 redis3.2 镜像创建容器(也叫做运行镜像)
  - 使用镜像
  ```
  docker run -p 6379:6379
  -v /zy/docker-redis/data:/data
  -v /zy/docker-redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
  -d redis:3.2 redis-server /usr/local/etc/redis/redis.conf
  --appendonly yes
  ```
  - 在主机 `/zy/docker-redis/conf/redis.conf`目录下新建`redis.conf`文件`vim /zy/docker-redis/conf/redis.conf/redis.conf`(redis 的配置文件)
  - 测试 redis-cli 连接上来`docker exec -it 运行着Redis服务的容器ID redis-cli`
  - 测试持久化文件生成