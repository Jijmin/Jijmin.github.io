### DockerFile 解析
1. 是什么
  - DockerFile 是用来构建 Docker 镜像的构建文件，是由一些列命令和参数构成的脚本
  - 构建三步骤
    - 编写DockerFile文件
    - docker build
    - docker run
  - 文件什么样？？？
  ```DockerFile
  FROM centos:7
  ENV container docker
  RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
  systemd-tmpfiles-setup.service ] || rm -f $i; done); \
  rm -f /lib/systemd/system/multi-user.target.wants/*;\
  rm -f /etc/systemd/system/*.wants/*;\
  rm -f /lib/systemd/system/local-fs.target.wants/*; \
  rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
  rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
  rm -f /lib/systemd/system/basic.target.wants/*;\
  rm -f /lib/systemd/system/anaconda.target.wants/*;
  VOLUME [ "/sys/fs/cgroup" ]
  CMD ["/usr/sbin/init"]
  ```
2. DockerFile 构建过程解析
  - DockerFile 内容基础知识
    - 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
    - 指令按照从上到下顺序执行
    - `#` 表示注释
    - 每条指令都会创建一个新的镜像层，并对镜像进行提交 
  - Docker 执行 DockerFile 的大致流程
    - docker 从基础镜像运行一个容器
    - 执行一条指令并对容器作出修改
    - 执行类似`docker commit`的操作提交一个新的镜像层
    - docker 再基于刚提交的镜像运行一个新的容器
    - 执行 dockerfile 中的下一条指令直到所欲指令都执行完成
  - 小总结
    - 从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段
      - Dockerfile 是软件的原材料
      - Docker 镜像是软件的交付品
      - Docker 容器则可以认为是软件的运行态
    - Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石
    - Dockerfile，需要定义一个 Dockerfile，Dockerfile 定义了进程需要的一切东西。Dockerfile 涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发型版、服务进程和内核进程(当应用进程需要和服务系统和内核进程打交道，这时需要考虑如何涉及namespace的权限控制)等等
    - Docker 镜像，在用 Dockerfile 定义一个文件之后，docker build 时会产生一个 Docker 镜像，当运行 Docker 镜像时，会真正开始提供服务
    - Docker 容器，容器是直接提供服务的
3. DockerFile 体系结构(保留字指令)
4. 案例
  - Base 镜像(scratch)：Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的
  - 自定义镜像 mycentos
  - CMD/ENTRYPOINT 镜像案例
  - 自定义镜像 Tomcat9
5. 小总结

### DockerFile 体系结构(保留字指令)
- `FROM`：基础镜像，当前新镜像是基于哪个镜像的
- `MAINTAINER`：镜像维护者的姓名和邮箱地址
- `RUN`：容器构建时需要运行的命令
- `EXPOSE`：当前容器对外暴露出的端口
- `WORKDIR`：指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点
- `ENV`：用来在构建镜像过程中设置环境变量
  - `EVN MY_PATH /usr/mytest`：这个环境变量可以在后续的任何 RUN 指令中使用，就是如同在命令前面指定了环境变量前缀一样；也可以在其他指令中直接使用这些环境变量，比如：`WORKDIR $MY_PATH`
- `ADD`：将宿主机目录下的文件拷贝进镜像且 ADD 命令会自动处理 URL 和解压 tar 压缩包
- `COPY`：类似 ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
  - `COPY src dest`
  - `COPY ["src", "dest"]`
- `VOLUME`：容器数据卷，用于数据保存和持久化工作
- `CMD`：指定一个容器启动时要运行的命令
  - Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
- `ENTRYPOINT`：指定一个容器启动时要运行的命令
  - ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数
- `ONBUILD`：当构建一个被继承的 Dockerfile 时运行命令，父镜像在被子继承后父镜像的 onbuild 被触发
```
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl", "-s", "http://ip.cn"]
ONBUILD RUN echo "father images onbuild --------- 886"
```
```
docker build -f /xxxx/DockerFile-onbulid -t myip_father .
cp DockerFile-ip2 DockerFile-ip3
```
```
FROM myip_father
RUN yum install -y curl
ONBUILD ["curl", "-s", "http://ip.cn"]
```
```
docker build -f /xxxx/DockerFile-ip3 -t myip_son .
```
- 小总结
![dockerfile命令.png](./images/dockerfile命令.png)

### 自定义镜像 mycentos
1. 编写
  - Hub 默认 CentOS 镜像什么情况
    - 初始 centos 运行该镜像进时默认路径是/
    - 默认不支持 vim
    - 默认不支持 ifconfig
    - 自定义 mycentos 目的使我们自己的镜像具备如下
      - 登陆后的默认路径
      - vim 编辑器
      - 查看网络配置 ifconfig 支持
  - 准备编写 DockerFile 文件
  - myCentOS 内容 DockerFile
  ```dockerfile
  FROM centos
  MAINTAINER zy<zy@163.com>

  ENV MYPATH /usr/local
  WORKDIR $MYPATH

  RUN yum -y install vim
  RUN yum -y install net-tools

  EXPOSE 80

  CMD echo $MYPATH
  CMD echo "success------------ok"
  CMD /bin/bash
  ```
2. 构建：`docker build -t 新镜像名字:TAG .`
```
docker build -f /Users/zhouying/Desktop/workspace/project/docker-demo/zy-dockerfile/DockerFile-vim -t mycentos:1.3 .
```
3. 运行：`docker run -it 新镜像名字:TAG`
4. 列出镜像的变更历史：`docker history 镜像名`

### CMD/ENTRYPOINT 镜像案例
1. 都是指定一个容器启动时要运行的命令
2. CMD
  - Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
  - Case：tomcat 的讲解演示`docker run -it -p 8888:8080 tomcat ls -l`
3. ENTRYPOINT
  - `docker run` 之后的参数会被当作参数传递给 ENTRYPOINT，之后形成新的命令组合
  - Case：
    - 制作 CMD 版可以查询 IP 信息的容器
    ```
    FROM centos
    RUN yum install -y curl
    CMD ["curl", "-s", "http://ip.cn"]
    ```
      - curl 命令解释
        - curl 命令可以用来执行下载、发送各种 HTTP 请求，指定 HTTP 头部等操作
        - 如果系统没有 curl 可以使用 `yum install curl`安装，也可以下载安装
        - curl 是将下载文件输出到 stdout
        - 使用命令：`curl http://www.baidu.com`
        - 执行后，www.baidu.com 的 html 就会显示在屏幕上
        - 这是最简单的使用方法。用这个命令获得了 http://curl.haxx.se 指向的页面，同样，如果这里的 URL指向的是一个文件或者一幅图都可以直接下载到本地。如果下载的是 HTML 文档，那么雀圣的将只显示文件头部，即 HTML 文档的 header。要全部显示，请加参数 -i
    - 问题`docker run myip -i`
    ```
    zhouyingdeMacBook-Pro:zy-dockerfile zhouying$ docker run myip -i
    docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "exec: \"-i\": executable file not found in $PATH": unknown.
    ERRO[0000] error waiting for container: context canceled
    ```
    - WHY
      - 我们可以看到可执行文件找不到的报错，executable file not found
      - 之前我们说过，**跟在镜像后面的是 command，运行时会替换 CMD 的默认值**
      - 因此这里的 -i 替换了原来的 CMD，而不是添加在原来的 `curl -s http://ip.cn` 后面。而 -i 根本不是命令，所以自然找不到
      - 那么如果我们希望加入 -i 这个参数，我们就必须重新完整的输入这个命令`docker run myip curl -s http://ip.cn -i`
    - 制作 ENTRYPOINT 版查询 IP 信息的容器
    ```
    FROM centos
    RUN yum install -y curl
    ENTRYPOINT ["curl", "-s", "http://ip.cn"]
    ```

### 自定义镜像 Tomcat9
1. `mkdir -p /zy/mydockerfile/tomcat9`
2. 在上述目录下 touch c.txt
3. 将 jdk 和 tomcat 安装的压缩包拷贝进上一步目录
  - apache-tomcat-9.0.8.tar.gz
  - jdk-8u171-linux-x64.tar.gz
4. 在 `/zy/mydockerfile/tomcat9` 目录下新建 Dockerfile 文件
  - 目录内容
  ```
  FROM centos
  MAINTAINER zy<zy@163.com>
  # 把宿主机当前上下文的 c.txt 拷贝到容器 /usr/local/ 路径下
  COPY c.txt /usr/local/cincontainer.txt
  # 把 java 与 tomcat 添加到容器中
  ADD jdk-8u171-linux-x64.tar.gz /usr/local
  ADD apache-tomcat-9.0.8.tar.gz /usr/local
  # 安装 vim 编辑器
  RUN yun -y install vim
  # 设置工作访问时候的 WORKDIR 路径，登陆落脚点
  ENV MYPATH /usr/local
  WORKDIR $MYPATH
  # 配置 java 与 tomcat 环境变量
  ENV JAVA_HOME /usr/local/jdk1.8.0_171
  ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
  ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
  ENV PATH $PATH:$JAVA_HOME/bin"$CATALINA_HOME/lob:$CATALINA_BASE/bin
  # 容器运行时监听的端口
  EXPOSE 8080
  # 启动时运行 tomcat
  # ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh"]
  # CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh", "run"]
  CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs/catalina.out
  ```
5. 构建
  - 构建完成`docker build -t zytomcat9 .`
6. run
```
docker run -d -p 9080:8080 --name myt9
-v /zy/mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test
-v /zy/mydockerfile/tomcat9/tomcat9logs/:usr/local/apache-tomcat-9.0.8/logs
--privileged=true
zytomcat9
```
  - 备注
7. 验证
8. 结合前述的容器卷将测试的 web 服务 test 发布
  - 总体概述
  ```
  ll
  a.jsp
  WEB-INF
  cd WEB-INF && ll
  web.xml
  ```
  - web.xml
  ![web-inf内容.png](./images/web-inf内容.png)
  - a.jsp
  ![a-jsp内容.png](./images/a-jsp内容.png)
  - 测试