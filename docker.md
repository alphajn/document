# Docker

[文档连接](https://yeasy.gitbook.io/docker_practice/)

基本

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

## 命令预览

命令 | 描述
-- | --
docker search | 查找官方仓库中的镜像 `docker search nginx`
docker push | 镜像推送到 Docker Hub `docker push username/nginx:12.04`
docker port | 查看端口映射 `docker port 容器 [端口]`
docker system df | 查看镜像、容器、数据卷所占用的空间

## 镜像-Image

### 获取镜像

命令：`docker pull [选项] [Docker Registry 地址[:端口号]/] 仓库名[:标签]`

```js
docker pull ubuntu:18.04
```

### 查看已下载镜像

命令：`docker image ls 或 docker images`

```js
docker image ls 或 docker images // 查看顶层镜像
docker image ls -a 或 docker images -a // 查看所有镜像
docker image ls --digests // 查看带有摘要的镜像

// 虚悬镜像
docker image ls -f dangling=true // 查看虚悬镜像(无用的镜像)
docker image prune // 删除虚悬镜像(无用的镜像)

// 根据仓库查看镜像
docker image ls ubuntu // 根据仓库名列出镜像
docker image ls ubuntu:18.04 // 根据仓库名和标签列出镜像

// 过滤器查看 --filter 或 -f
docker image ls -f since=ubuntu:18.04 // 查看ubuntu:18.04之后创建的镜像
docker image ls -f before=ubuntu:18.04 // 查看ubuntu:18.04之前创建的镜像
docker image ls -f label=ubuntu.example // 查看label是ubuntu.example的镜像

// 格式化查看
docker image ls -q // 只显示镜像id
docker image ls --format "{{.ID}}: {{.Repository}}" // 显示镜像id和仓库名
docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}" // 以表格等距显示
```

### 删除镜像

命令：`docker image rm [选项] <镜像1> [<镜像2> ...] 或 docker rmi 镜像`

`<镜像>` 可以是 `镜像短 ID`、`镜像长 ID`、`镜像名` 或者 `镜像摘要`

```js
docker image rm [ID] 或 docker rmi [id] // 使用ID删除，也可以取id中前三个以上的字符
docker image rm nginx:latest // 使用仓库名:标签删除
docker image prune // 删除虚悬镜像(无用的镜像)

// 配合 docker image ls -q 批量删除
docker image rm $(docker image ls -q nginx) // 删除所有的nginx镜像
```

## 使用Dockerfile定制镜像

### FROM 指定基础镜像

命令：`FROM 镜像`

`Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

```js
FROM scratch // 空白镜像
FROM nginx // 指定nginx镜像
```

### RUN 执行命令

1. shell格式：`RUN <命令>`
2. exec 格式：`RUN ["可执行文件", "参数1", "参数2"]`

```js
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

### 构建镜像

命令：`docker build [选项] <上下文路径/URL/tar/->`

选项：

  1. `-t`: 名称和可选的“名称：标签”格式的标签

上下文路径:

  1. `.`: 当前路径
  2. `repo`: 仓库地址，如：`https://github.com/master`
  3. `tar压缩包`: Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建

```js
// Dockerfile 文件构建
    FROM nginx
    RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html

docker build -t nginx:v1 .

// 仓库地址构建
docker build -t hello-world https://github.com/docker-library/hello-world.git

// 压缩包构建
docker build http://server/context.tar.gz
```

### Dockerfile指令

#### COPY 复制文件

命令：

  1. shell格式：`COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
  2. exec格式： `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

```js
COPY package.json /usr/src/app/
COPY home* /mydir/ // 通配符

// 改变文件的所属用户和所属组
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

#### ADD 更高级的复制文件,需要自动解压缩的场合才使用

命令：

  1. shell格式： `ADD [--chown=<user>:<group>] <源路径>... <目标路径>`
  2. exec格式：`ADD [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

```shell
ADD nginx.tar.gz /
ADD https://nginx/nginx.tar.gz / // 愿路径可以是路径
```

#### CMD 执行命令

命令：`CMD ["可执行文件", "参数1", "参数2"...]`

```js
CMD [ "sh", "-c", "echo $HOME" ]
```

#### ENTRYPOIN` 执行命令，跟`CMD`类似

```js
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

#### ENV 设置环境变量

命令：

  1. `ENV <key> <value>`
  2. `ENV <key1>=<value1> <key2>=<value2>...`

```js
ENV VERSION=1.0 DEBUG=on
```

#### EXPOSE 暴露端口

命令：`EXPOSE <端口1> [<端口2>...]`

```js
EXPOSE 80 443
```

#### WORKDIR 指定工作目录

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

命令：`WORKDIR <工作目录路径>`

```js
WORKDIR /app
```

#### USER 指定当前用户

`USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` `是改变工作目录，USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。

命令：`USER <用户名>[:<用户组>]`

```js
USER user:groupuser
```

#### HEALTHCHECK 健康检查

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。

命令：

  1. `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
  2. `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

选项

  1. `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
  2. `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
  3. `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。

```shell
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

## 容器-Container

### 启动容器

命令：`docker run [选项] 镜像 [COMMAND] [ARG...]`

选项：

  1. `-t`: 终端
  2. `-i`: 交互式操作
  3. `-d`: 后台运行
  4. `-P`: 随机映射端口到内部容器开放的网络端口
  5. `-p`: 映射本地端口到容器端口
  6. `--name`: 为容器指定一个名称
  7. `--rm`: 退出容器是自动删除, 不能与 `-d` 同时使用
  8. `-v`: 映射文件

```js
docker run nginx echo 'hello' // 启动nginx 输出hello后关闭
docker run -it nginx bash // 启动一个 bash 终端，允许用户进行交互 退出 exit 或 Ctrl+d

docker run -d nginx // 后台运行
docker run -d -P --name test-nginx-1 nginx // 随机端口映射
docker run -d -p 80:80 --name test-nginx-1 nginx // 本地的 80 端口映射到容器的 80 端口
// 容器映射主机文件
docker run -d --name nginx-v1 -p 80:80 -v /nginx/html:/usr/share/nginx/html/ nginx

docker run -it --rm nginx bash // 退出容器自动删除
```

### 查看容器

命令：`docker container ls [选项] 或 docker ps`

选项：运行`docker container ls -h` 查看

```js
docker container ls -a 或 docker ps -a // 查看所有的容器
docker container ls 或 docker ps // 查看运行的容器
docker container ls -f status=exited 或 docker ps -f status=exited // 查看停止的容器

// 查看容器输出内容
docker container logs [container ID or NAMES]
// 或
docker logs -f [container ID or NAMES] // docker logs nginx 跟踪输出日志
```

### 终止容器

命令：`docker stop [选项] 容器 [容器...] 或 docker container [选项] 容器 [容器...]`

```js
docker container stop 292ae607285c // 根据容器id终止
docker stop nginx-v1 // 根据容器name终止
docker stop nginx-v1 nginx-v2 // 终止多个
```

### 重启容器

命令：

  1. `docker start [选项] 容器 [容器...] 或 docker container start [选项] 容器 [容器...]`
  2. `docker restart [选项] 容器 [容器...] 或 docker container restart [选项] 容器 [容器...]`

```js
// 重启已停止容器
docker start 292ae607285c // 根据容器id启动
docker start nginx-v1 // 根据容器name启动

// 重启运行中容器
docker restart 292ae607285c // 根据容器id启动
docker restart nginx-v1 // 根据容器name启动
```

### 进入容器

命令：

  1. `docker attach container`: 退出会导致容器停止（不建议使用）
  2. `docker exec [选项] container COMMAND [ARG...]`

```js
docker attach 69d1
docker exec -it 69d1 bash
```

### 倒入导出

命令：

  1. `docker export [选项] container`: 导出
  2. `docker import [选项] file|URL|- [repository[:TAG]]`: 导入

```bash
docker container ls -a // 查看所有容器

docker export b0389fbc70e1 > export-nginx-v3.tar // 导出

// 导入
docker import export-nginx-v3.tar nginx-import-v3 // 导入镜像
cat export-nginx-v3.tar | docker import - nginx-import-v3:tag1 // 导入镜像
docker import http://example.com/exampleimage.tgz example/imagerepo // url导入镜像

```

**运行导入的镜像需要带command 否则报错**

### 删除容器

命令：`docker container rm [选项] container [container...] 或 docker rm [选项] container`

选项：

  1. `-f`: 强制删除（运行中的）
  2. `-l`: 删除指定链接

```js
docker container rm nginx-v1 或 docker rm nginx-v1 // 删除容器
docker container rm -f nginx-v1 或 docker rm -f nginx-v1 // 删除运行中的容器
docker container prune // 删除所有停止的容器
```

## 其他命令

### 拷贝

命令：

  1. `docker cp [选项] container:src_path dest_path`: 容器复制到主机
  2. `docker cp [选项] src_path container:dest_path`: 主机到容器

```js
docker cp b3366f7308e7:/usr/share/nginx/html/index.html ./html // 拷贝容器里的html到本地
docker cp ./nginx.conf b3366f7308e7:/etc/nginx/
```
