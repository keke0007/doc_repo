# Docker Compose `.env` 与 Docker 常用命令速查

> 本文档用于说明 `.env` 如何被 Docker Compose 使用，并整理 Docker Compose、Docker 镜像、容器、卷、网络、资源清理的常用命令。

## 1. `.env` 是什么

`.env` 是 Docker Compose 默认读取的环境变量文件。它通常和 `docker-compose.yaml` 放在同一个目录，用于把密码、端口、镜像版本、宿主机 IP 等可变配置从 compose 文件中抽离出来。

示例目录：

```text
my-project/
├── .env
└── docker-compose.yaml
```

示例 `.env`：

```dotenv
COMPOSE_PROJECT_NAME=my-project
MYSQL_VERSION=8.0.42
MYSQL_ROOT_PASSWORD=root
REDIS_PASSWORD=123456
HOST_IP=192.168.1.100
```

示例 `docker-compose.yaml`：

```yaml
services:
  mysql:
    image: mysql:${MYSQL_VERSION}
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

  redis:
    image: redis:7.0
    command: redis-server --requirepass ${REDIS_PASSWORD}
```

执行：

```bash
docker compose up -d
```

Compose 会自动读取当前 compose 项目目录下的 `.env`，把 `${MYSQL_VERSION}`、`${MYSQL_ROOT_PASSWORD}`、`${REDIS_PASSWORD}` 替换成实际值。

## 2. `.env` 与 `environment` 的区别

这两个概念容易混：

| 名称 | 作用位置 | 作用 |
| --- | --- | --- |
| `.env` | Docker Compose 解析配置时使用 | 给 `docker-compose.yaml` 做变量替换 |
| `environment` | 容器运行时使用 | 把环境变量传进容器内部 |

示例：

```dotenv
MYSQL_ROOT_PASSWORD=root
```

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

这里发生了两件事：

1. Compose 先读取 `.env`，把 `${MYSQL_ROOT_PASSWORD}` 替换成 `root`。
2. Docker 再把 `MYSQL_ROOT_PASSWORD=root` 注入到 MySQL 容器内部。

最终效果等价于：

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
```

## 3. `.env` 的常见写法

### 3.1 基础格式

```dotenv
KEY=value
```

注意：

- 等号两边不要加空格。
- 一行一个变量。
- `#` 开头是注释。
- 密码里如果有特殊字符，建议用引号包起来。

```dotenv
REDIS_PASSWORD="abc@123!?"
```

### 3.2 Compose 中引用变量

```yaml
image: mysql:${MYSQL_VERSION}
ports:
  - "${MYSQL_PORT}:3306"
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

对应 `.env`：

```dotenv
MYSQL_VERSION=8.0.42
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=root
```

### 3.3 默认值

如果变量不存在，可以给默认值：

```yaml
image: mysql:${MYSQL_VERSION:-8.0}
ports:
  - "${MYSQL_PORT:-3306}:3306"
```

含义：

- `.env` 中有 `MYSQL_VERSION`，使用 `.env` 的值。
- 没有 `MYSQL_VERSION`，使用默认值 `8.0`。

### 3.4 必填变量

如果变量必须显式配置，可以这样写：

```yaml
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:?MYSQL_ROOT_PASSWORD is required}
```

如果 `.env` 没有配置该变量，执行 `docker compose up` 会直接报错。

## 4. `.env` 的读取规则

常用情况下：

```bash
cd my-project
docker compose up -d
```

Compose 会读取当前项目目录下的 `.env`。

也可以手动指定 env 文件：

```bash
docker compose --env-file .env.dev up -d
docker compose --env-file .env.prod up -d
```

适合区分开发、测试、生产：

```text
my-project/
├── .env.dev
├── .env.test
├── .env.prod
└── docker-compose.yaml
```

## 5. 检查变量替换结果

这是排查 `.env` 是否生效最常用的命令：

```bash
docker compose config
```

它会输出变量替换后的最终 compose 配置。

只检查配置是否合法：

```bash
docker compose config --quiet
```

把最终配置保存下来：

```bash
docker compose config > docker-compose.rendered.yaml
```

如果发现 `${XXX}` 没有被替换，通常是：

- `.env` 不在当前 compose 项目目录。
- 变量名拼错。
- 执行命令时所在目录不对。
- 使用了 `-f` 指定其他 compose 文件，但没有同步指定正确工作目录或 env 文件。

## 6. Docker Compose 常用命令

### 6.1 启动与停止

```bash
docker compose up -d
```

后台启动所有服务。

```bash
docker compose up
```

前台启动，日志直接输出到当前终端。

```bash
docker compose down
```

停止并删除当前 compose 项目的容器和网络，不删除 bind mount 数据目录。

```bash
docker compose stop
```

只停止容器，不删除容器。

```bash
docker compose start
```

启动已存在但停止的容器。

```bash
docker compose restart
```

重启所有服务。

```bash
docker compose restart redis
```

只重启指定服务。

### 6.2 查看状态与日志

```bash
docker compose ps
```

查看当前 compose 项目的容器状态。

```bash
docker compose logs
```

查看所有服务日志。

```bash
docker compose logs -f
```

持续跟踪日志。

```bash
docker compose logs -f nacos
```

只跟踪指定服务日志。

```bash
docker compose logs --tail=200 mysql
```

查看 MySQL 最近 200 行日志。

### 6.3 构建与拉取镜像

```bash
docker compose pull
```

拉取 compose 中声明的镜像。

```bash
docker compose build
```

构建 compose 中带 `build` 配置的服务镜像。

```bash
docker compose up -d --build
```

启动前先构建镜像。

```bash
docker compose up -d --pull always
```

启动前尝试拉取最新镜像。

### 6.4 进入容器与执行命令

```bash
docker compose exec mysql bash
```

进入 MySQL 服务容器。

```bash
docker compose exec redis redis-cli -a 123456
```

进入 Redis CLI。

```bash
docker compose exec mysql mysql -uroot -proot
```

进入 MySQL CLI。

```bash
docker compose run --rm app sh
```

临时启动一个服务容器执行命令，执行完自动删除。

### 6.5 指定文件和项目名

```bash
docker compose -f docker-compose.yaml up -d
```

指定 compose 文件。

```bash
docker compose -f docker-compose.yaml -f docker-compose.override.yaml up -d
```

合并多个 compose 文件，后面的文件覆盖前面的同名配置。

```bash
docker compose -p my-project up -d
```

指定项目名。项目名会影响默认网络名、容器名前缀、卷名前缀。

也可以在 `.env` 中指定：

```dotenv
COMPOSE_PROJECT_NAME=my-project
```

### 6.6 删除与重建

```bash
docker compose down
```

删除容器和网络。

```bash
docker compose down --remove-orphans
```

删除 compose 文件中已经不存在的旧服务容器。

```bash
docker compose down -v
```

删除容器、网络和 compose 管理的 named volumes。谨慎使用。

```bash
docker compose up -d --force-recreate
```

强制重建容器。

```bash
docker compose up -d --no-deps app
```

只重建指定服务，不启动它依赖的服务。

## 7. Docker 镜像管理命令

### 7.1 查看镜像

```bash
docker images
```

或：

```bash
docker image ls
```

查看本地镜像列表。

```bash
docker image ls mysql
```

只看 MySQL 镜像。

### 7.2 拉取镜像

```bash
docker pull mysql:8.0.42
docker pull redis:7.0
docker pull nacos/nacos-server:v2.2.2
```

### 7.3 删除镜像

```bash
docker rmi mysql:8.0.42
```

或：

```bash
docker image rm mysql:8.0.42
```

如果镜像正在被容器使用，需要先删除相关容器。

强制删除：

```bash
docker image rm -f mysql:8.0.42
```

谨慎使用。

### 7.4 查看镜像详情

```bash
docker image inspect mysql:8.0.42
```

查看镜像层、环境变量、入口命令等元信息。

### 7.5 镜像打标签

```bash
docker tag my-app:latest registry.example.com/my-app:1.0.0
```

### 7.6 推送镜像

```bash
docker push registry.example.com/my-app:1.0.0
```

## 8. Docker 容器管理命令

### 8.1 查看容器

```bash
docker ps
```

查看运行中的容器。

```bash
docker ps -a
```

查看全部容器，包括已停止容器。

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

按表格格式查看关键信息。

### 8.2 启动、停止、重启

```bash
docker start mysql
docker stop mysql
docker restart mysql
```

### 8.3 删除容器

```bash
docker rm mysql
```

删除已停止容器。

```bash
docker rm -f mysql
```

强制停止并删除容器。谨慎使用。

### 8.4 查看日志

```bash
docker logs mysql
docker logs -f mysql
docker logs --tail=200 mysql
```

### 8.5 进入容器

```bash
docker exec -it mysql bash
```

如果容器没有 bash：

```bash
docker exec -it redis sh
```

### 8.6 查看容器详情

```bash
docker inspect mysql
```

查看容器 IP、挂载目录、环境变量、网络等详情。

只看容器 IP：

```bash
docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" mysql
```

### 8.7 查看容器资源占用

```bash
docker stats
```

实时查看所有容器 CPU、内存、网络、磁盘 IO。

```bash
docker stats mysql redis
```

只看指定容器。

### 8.8 查看容器进程

```bash
docker top mysql
```

### 8.9 容器与宿主机复制文件

从容器复制到宿主机：

```bash
docker cp mysql:/var/log/mysql ./mysql-log
```

从宿主机复制到容器：

```bash
docker cp ./backup.sql mysql:/tmp/backup.sql
```

## 9. Docker 卷管理命令

查看卷：

```bash
docker volume ls
```

查看卷详情：

```bash
docker volume inspect volume_name
```

创建卷：

```bash
docker volume create mysql_data
```

删除卷：

```bash
docker volume rm mysql_data
```

删除未使用的卷：

```bash
docker volume prune
```

注意：本文 golden template 更多使用 bind mount，例如 `./mysql/data:/var/lib/mysql`。这种数据目录不在 `docker volume ls` 中显示，而是在项目目录下。

## 10. Docker 网络管理命令

查看网络：

```bash
docker network ls
```

查看网络详情：

```bash
docker network inspect my-project_middleware_net
```

创建网络：

```bash
docker network create middleware_net
```

删除网络：

```bash
docker network rm middleware_net
```

删除未使用网络：

```bash
docker network prune
```

查看某个容器加入了哪些网络：

```bash
docker inspect mysql --format "{{json .NetworkSettings.Networks}}"
```

## 11. Docker 磁盘与资源清理命令

### 11.1 查看 Docker 占用空间

```bash
docker system df
```

查看镜像、容器、卷、构建缓存占用。

```bash
docker system df -v
```

查看更详细的占用。

### 11.2 清理停止的容器

```bash
docker container prune
```

### 11.3 清理悬空镜像

```bash
docker image prune
```

清理 dangling images，也就是 `<none>` 镜像。

### 11.4 清理未使用镜像

```bash
docker image prune -a
```

会删除当前没有被任何容器使用的镜像。谨慎使用。

### 11.5 清理构建缓存

```bash
docker builder prune
```

清理未使用构建缓存。

```bash
docker builder prune -a
```

清理更多构建缓存。谨慎使用。

### 11.6 一键清理未使用资源

```bash
docker system prune
```

清理：

- 停止的容器
- 未使用网络
- dangling 镜像
- 构建缓存

更彻底清理：

```bash
docker system prune -a
```

会删除未被容器使用的镜像。谨慎使用。

连未使用卷也一起删：

```bash
docker system prune -a --volumes
```

高风险命令，会删除未使用卷中的数据。执行前确认没有重要数据。

## 12. 日常排错命令组合

### 12.1 Compose 配置是否正确

```bash
docker compose config
docker compose config --quiet
```

### 12.2 服务为什么没起来

```bash
docker compose ps
docker compose logs --tail=200 服务名
docker inspect 容器名
```

示例：

```bash
docker compose ps
docker compose logs --tail=200 nacos
docker inspect nacos
```

### 12.3 端口是否被占用

Linux/macOS：

```bash
netstat -tunlp | grep 8848
lsof -i :8848
```

Windows PowerShell：

```powershell
netstat -ano | findstr :8848
Get-Process -Id <PID>
```

### 12.4 容器之间网络是否能通

进入某个容器：

```bash
docker compose exec app sh
```

在容器内测试：

```bash
ping mysql
nc -vz mysql 3306
nc -vz redis 6379
```

有些镜像没有 `ping`、`nc`，可以临时启动工具容器：

```bash
docker run --rm -it --network my-project_middleware_net nicolaka/netshoot
```

### 12.5 数据库初始化脚本为什么没执行

检查 MySQL 数据目录是否已经有数据：

```bash
ls mysql/data
```

MySQL 官方镜像只会在 `/var/lib/mysql` 为空时执行 `/docker-entrypoint-initdb.d/` 下的初始化脚本。如果 `mysql/data` 已经初始化过，后续新增 SQL 不会自动执行。

处理方式：

```bash
docker compose down
```

确认数据可删除后，再删除 `mysql/data` 并重新启动。

## 13. 推荐日常工作流

新项目首次启动：

```bash
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
docker compose logs -f
```

修改 `.env` 后：

```bash
docker compose config
docker compose up -d
```

修改 compose 服务配置后：

```bash
docker compose config --quiet
docker compose up -d
```

修改镜像版本后：

```bash
docker compose pull
docker compose up -d
```

排查某个服务：

```bash
docker compose ps
docker compose logs --tail=200 服务名
docker compose restart 服务名
```

