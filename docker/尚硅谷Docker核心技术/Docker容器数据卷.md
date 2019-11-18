### Docker 容器数据卷
1. 是什么
  - 将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
  - 容器之间希望有可能共享数据
  - Docker 容器产生的数据，如果不通过 docker commit 生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了
  - 为了能保存数据在 docker 中我们使用卷
2. 能干嘛
  - 容器的持久化
  - 容器间继承 + 共享数据
  - 卷就是目录或文件，存在于一个或多个容器中，由 docker 挂载到容器，但不属于联合文件系统，因此能够绕过 Union File System 提供一些用于持续存储或共享数据的特性：
  - 卷的设计目的就是数据的持久化，完全独立于容器的生命周期，因此 Docker 不会在容器删除时删除其挂载的数据卷
  - 特点：
    - 数据卷可在容器之间共享或重用数据
    - 卷中的更改可以直接生效
    - 数据卷中的更改不会包含在镜像的更新中
    - 数据卷的生命周期一直持续到没有容器使用它为止
3. 数据卷
  - 直接命令添加
    - 命令`docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名`
    ```
    docker run -it -v ~/Desktop/myDataVolume:/dataVolumeContainer centos
    ```
    - 查看数据卷是否挂载成功
    ```
    # 通过使用 docker inspect 容器ID
    # 里面如果有一个 Volumes 里面有绑定关系就成功
    ```
    - 容器和宿主机之间数据共享
    - 容器停止退出后，主机修改后数据是否同步
      - 容器先停止退出
      - 主机修改a.log
      - 容器重新进入
      - 查看主机修改过的a.log，内容同步
    - 命令(带权限)
      - `docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名`
      - 容器内没有写的权限
  - DockerFile 添加
    - 根目录下新建 mydocker 文件夹并进入
    - 可在 Dockerfile 中使用 **VOLUME 指令** 来给镜像添加一个或多个数据卷
    ```
    VOLUME["/dataVolumeContainer","/dataVolumeContainer2","dataVolumeContainer3"]
    # 说明：
    # 由于可移植和分享的考虑，用 -v 主机目录:容器目录这种方法不能够直接在 Dockerfile 中实现
    # 由于宿主机目录是依赖于特定宿主机的，并不能保证在所有的宿主机上都存在这样的特定目录
    ```
    - File 构建
    ```
    # volume test
    FROM centos
    VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
    CMD echo "finished,----------success1"
    CMD /bin/bash
    ```
    - build 后生成镜像，获得一个新镜像 zzyy/centos
    ```
    docker build -f /mydocker/Dockerfile -t zzyy/centos .
    ```
    - run 容器
    - 通过上述步骤，容器内的卷目录地址已经知道对应的主机目录地址哪？？
    - 主机对应默认地址
    ```
    docker inspect 容器ID，里面可以查找到存放的默认位置
    ```
  - 备注
    - Docker 挂载主机目录 Docker 访问出现 cannot open directory .: Permission denied
    - 解决办法：在挂载目录后多加一个 `--provoleged=true` 参数即可
4. 数据卷容器
  - 是什么：命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器
  - 总体介绍
    - 以上一步新建的镜像`zzyy/centos`为模版并运行容器 dc01/dc02/dc03
    - 他们已经具有容器卷
      - /dataVolumeContainer1
      - /dataVolumeContainer2
  - 容器间传递共享(--volumes-from)
    - 先启动一个父容器`dc01`：在`dataVolumeContainer2`新增内容
    ```
    docker run -it --name dc01 zzyy/centos
    ```
    - `dc02/dc03`继承自`dc01`，`--volumes-from`，命令：`dc02/dc03`分别在`dataVolumeContainer2`各自新增内容
    ```
    docker run -it --name dc02 --volumes-from dc01 zzyy/centos
    ```
    - 回到`dc01`可以看到`dc02/dc03`各自添加的都能共享了
    - 删除`dc01`，`dc02`修改后`dc03`可否访问
    - 删除`dc02`后`dc03`可否访问
    - 新建`dc04`继承`dc03`后再删除`dc03`
    - 结论：容器之间配置信息的传递，数据卷的生命周期移植持续到没有容器使用它为止