# task-serve

一个简单的任务服务，包括API服务和算法服务，提供网页交互。

## 技术栈

- Golang 1.18
- VsCode
- Docker 
- Redis
- RabbitMQ

## Docker环境配置

- Docker 使用 Docker Desktop
- Redis 安装教程：[链接](https://hub.docker.com/_/redis/)
- RabbitMQ 安装教程：[链接](https://juejin.cn/post/7198430801850105916)

## 环境启动

- 创建Volume

```
docker volume create redis_volume
docker volume create rabbitmq_volume
```

- 启动Redis

```
docker run --name redis_serve -p 6379:6379 -v redis_volume:/data -d redis redis-server --save 60 1 --loglevel warning
```

- 启动RabbitMQ

```
docker run --name rabbitmq_serve -p 15672:15672 -p 5672:5672 -v rabbitmq_volume:/var/lib/rabbitmq -e RABBITMQ_DEFAULT_USER=ciaran -e RABBITMQ_DEFAULT_PASS=123456 -d rabbitmq:management
```

## 代码测试

- 均存放在*_use.go文件中,使用vscode 编辑器的run test通过

## 运行

- ~~运行算法服务 `go run .\main.go -s algo`~~
- ~~创建任务 `go run .\main.go -s api -op create -tname test111 -tcommand none`~~
- ~~查询状态 `go run .\main.go -s api -op select -tid task-24`~~
- 启动服务 `go run ./main.go`
- 创建任务 `post http://localhost:9001/create_task {task_name, task_command}`
- 查看任务 `get http://localhost:9001/view_task?task_id=`
- 查看所有任务 `get http://localhost:9001/get_task_list`
- 取消任务 `get http://localhost:9001/create_task?task_id=`

## 改进

- 提供了http api服务, **但还未编写前端页面**
- 将redis存储任务结构改为了hash表（后续可以使用mysql来存储任务信息，redis缓存）
- 使用连接池提高可用性 
    - 学习发现：redis 创建默认使用连接池，最大连接数为4倍CPU数
    - 编写了针对这个任务特定的简单的rabbitmq连接池，但是细节方面没有深入
- 添加取消任务操作

## 思考

- 当大并发请求时，如何提升服务可用性。我认为
    - 通过建立连接池来保持mq和redis链接，避免频繁链接的时间浪费
    - 通过协程和锁加快api请求的处理速度
    - 通过建立分布式系统来处理大量请求
- 耗时任务如何进行进度上报，取消任务等状态管理。我认为
    - 进度上报可以使用心跳机制，传输任务状态
    - 取消任务也可以通过心跳机制，传输任务操作



