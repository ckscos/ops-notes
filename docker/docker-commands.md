# Docker 容器常用命令

三、Docker 容器运维
————————————————————————————————————————————————————————————————


## 3.1 镜像管理

使用场景：拉取基础镜像 / 构建应用镜像 / 推送私有仓库

  docker pull nginx:latest                    拉取镜像
  docker pull python:3.10-slim                拉取指定版本（slim 更小）
  docker images                               查看本地镜像列表
  docker rmi 镜像名:标签                        删除指定镜像
  docker rmi $(docker images -q)              删除所有镜像

  docker build -t 镜像名:标签 .                  根据 Dockerfile 构建镜像（注意后面有个点）
  docker build --no-cache -t 镜像名:标签 .       不利用缓存重新构建

  docker tag 原镜像:标签 新镜像:标签              给镜像打标签
  docker login 仓库地址                          登录远程仓库
  docker push 仓库地址/命名空间/镜像名:标签        推送镜像到仓库

  docker inspect 镜像名:标签                      查看镜像详细元数据
  docker history 镜像名:标签                      查看镜像构建历史


## 3.2 容器生命周期

使用场景：部署服务 / 启停管理 / 日志排查

  docker run -d 镜像名                          后台运行容器
  docker run -d -p 80:80 镜像名                  端口映射（宿主机:容器）
  docker run -d --name my-app 镜像名             指定容器名称
  docker run -d --restart always 镜像名          宕机自动重启（生产必备）

  docker ps                                     查看运行中的容器
  docker ps -a                                  查看所有容器（含已停止）
  docker ps -q                                  仅显示容器 ID（配合批量操作）

  docker stop  容器名                            优雅停止
  docker start 容器名                            启动已停止的容器
  docker restart 容器名                          重启容器

  docker rm 容器名                               删除已停止的容器
  docker rm -f 容器名                            强制删除（运行中也删）
  docker container prune                        删除所有已停止的容器

  docker exec -it 容器名 bash                    进入容器终端（调试用）
  docker exec -it 容器名 sh                      进入容器（镜像没有 bash 时）
  docker logs 容器名                             查看日志
  docker logs -f 容器名                          实时追踪日志（Ctrl+C 退出）
  docker logs --tail 100 容器名                  只看最后 100 行
  docker top 容器名                              查看容器内运行的进程

  docker cp 本地文件 容器名:容器路径               从宿主机拷贝到容器
  docker cp 容器名:容器路径 本地路径               从容器拷贝到宿主机


## 3.3 数据卷与持久化

使用场景：MySQL 数据不随容器删除丢失 / 配置文件持久化

  -- 使用数据卷（Docker 管理，推荐） --
  docker volume create 卷名                    创建数据卷
  docker volume ls                             查看数据卷列表
  docker volume inspect 卷名                   查看数据卷宿主机路径
  docker run -d -v 卷名:/容器内路径 镜像名        挂载数据卷

  实战示例（MySQL 持久化）：
  docker run -d --name mysql-db -v mysql_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root123 mysql:8.0
  删容器重建，数据还在

  -- 绑定挂载（宿主机指定路径） --
  docker run -d -v /宿主机路径:/容器内路径 镜像名
  示例：docker run -d -v /root/html:/usr/share/nginx/html nginx


## 3.4 监控与资源查看

使用场景：看容器吃了多少资源 / 获取容器 IP / 性能排查

  docker stats                                实时查看所有容器 CPU/内存/网络 IO
  docker stats 容器名                          查看指定容器
  docker inspect 容器名                        查看完整元数据（IP、挂载、环境变量）
  docker inspect 容器名 | grep IPAddress       查看容器 IP
  docker logs --since 30m 容器名                查看过去 30 分钟日志


## 3.5 Docker Compose

使用场景：多容器项目 / 一键启停全套服务

  docker-compose up -d                        启动所有服务（后台）
  docker-compose ps                           查看服务状态
  docker-compose logs -f                      查看所有服务实时日志
  docker-compose stop                         停止服务（保留数据卷和数据）
  docker-compose down                         停止并删除容器和网络
  ★★ docker-compose down -v ★★                停止 + 删容器 + 删数据卷（数据会丢，慎用）
  docker-compose up -d --build                重新构建镜像并启动


## 3.6 清理与安全

使用场景：Docker 占太多磁盘 / 清理无用资源

  docker system df                            查看 Docker 占了多少空间
  docker container prune                      删除所有已停止的容器（安全）
  docker image prune                          删除无标签 dangling 镜像（安全）
  docker volume prune                         删除未被使用的数据卷（确认数据不要了再用）
  docker system prune                         清理容器+镜像+网络

  ★★ docker system prune -a ★★  高危！删除所有未使用的镜像、容器、网络、构建缓存，不可逆！确认不需要再执行


————————————————————————————————————————————————————————————————
