# 一夜技术博客


## I/O


**拆解技术书籍**
## 云原生技术
云原生让分布式服务端系统成为可能
### telepresence
### Kubernetes

Kubernetes组成
> Master节点组件控制节点，主节点，负责管理集群状态
* etcd配置存储
* API服务器
* 调度器
* 控制器管理器
> Node节点的组件, 运行在工作node上
* Kubelete 管理pod
* kube-proxy 网络通信和负载
* 运行容器 
> 附加组件
* Kubernetes的DNS服务器
* 控制台
* Ingress控制器
* 监控工具
* 容器网络插件
Kubernetes架构图
![Image](src)

Kubernetes核心是对Pod对控制
> kubernetes的pod健康检查(liveness probees)
* HTTP GET获取POD状态
* TCP Socket状态检查
* Exec 命令方式 
> kubernetes的pod准备检查(readiness probe)
* HTTP GET获取POD状态
* TCP Socket状态检查
* Exec 命令方式

> kubernetes的services
k8s的pod的ip是可变不固定的，多pod的ip也不同，需要service作为固定唯一的访问入口
集群内部service访问:添加selector, 环境变量和FQDN
集群外部service访问:不指定selector, 创建endpoints资源关联外部集群外部服务ip:port

> kubernetes的外网访问机制
k8s尽开放了pod间的相互访问，pod的ip对外部是不可见的，外网访问需要固定的ip入口，外网访问k8s的机制有一下几种
* NodePort(重定向外部请求包到service,可以通过node的ip访问service)
* LoadBalancer(NodePort的扩展类型) 
* Ingress(HTTP网络层) 

> kubernetes的服务访问异常排查
1. 集群内部访问服务集群IP
2. 确保pod的健康检查是OK的
3. 确认pod是服务的一部分，检查服务的endpoint
4. 通过FQDN访问确保是正确的
5. 确认你连接的是服务的port而部署目标port

> kubernetes的无状态控制器
* ReplicationController[已废弃]
* ReplicaSets(Pod模版, 标签选择，Pod数)
* DaemonSet(一个Node一个Pod)
* Job/CronJob
* Deployment
> kubernetes的有状态控制器
> kubernetes的自定义控制器

> kubernetes的volumn
用来挂载持久化的数据
* emptyDir在pod的容器中共享，pod删除后一起消失
* hostpath在node节点上的目录
* gitRepo获取git内容
* PVC屏蔽了PV的多样性，提供了一致的接口
动态数据传递, 命令行和环境变量无法避免去修改k8s的pod资源文件, 配置和密码将动态的数据从镜像抽离出来，减少不必要的镜像重构和维护
* 命令行command和args
* 环境变量env
* ConfigMap
ConfigMap用键值的方式保存配置, 可以通过字面量设置，可以读取文件设置, 也可以读取文件夹设置
ConfigMap可以作为容器环境变量读取,也可以作为文件挂载到容器 
ConfigMap的修改会更新到容器里面，避免了容器的重新启动
* Secrets
用来存储敏感信息，不会落磁盘文件

> kubernetes的应用
kubernetes用四种资源组成一个应用（workloads, loadbalance, service, volumes)

### Docker
####ENTRYPOINT和CMD的区别
ENTRYPOINT会在容器启动时执行，不会被覆盖
CMD在启动运行容器时执行，可以在docker run时覆盖执行命令
####ADD和COPY的区别
功能都是将文件添加到镜像，ADD会多其他功能，针对tar的压缩文件会解压, 从url拷贝文件到镜像中

## 分布式应用框架
### Dubbo
### SpringCloud
### Spring,SpringMvc,SpringBoot
作为知名的MVC框架，随着前后端分离的持续推进，越来越多应用不再使用MVC的模式, 主要原因是模版渲染不再被使用

## 数据库
数据库作为持久方式，是提供有状态的服务，数据库的底层还是用文件的形式实现

## 中间件技术
对任何中间件的引入都要谨慎，一定程度上破坏来应用的无状态，同时造成了应用复杂度的上升，中间件的维护成本。
中间件技术本质是用来减小代码复杂度和解耦的作用
### 缓存
#### 分布式缓存Redis
#### 应用本地缓存
### 索引
### 消息队列
### 配置中心
配置中心将常变的数据从代码中抽离出来， 不再依赖于应用的重新发布，但也会造成配置的更新不再和应用发布同步, 
可以通过配置新的键值来同步发布配置
### 数据库中间件
### 分布式任务框架
### 分布式存储方案
### 唯一ID生成服务
### 告警系统
Minlo

## 代码库
### JSON和XML解析

## 接入层网关

## 网络协议
### HTTP/HTTP2/HTTP3

## 设计模式
### 代理模式
代理模式被使用来增加对象能力和统一工作的能力

## 性能优化
性能优化要从监控，分析，调优三个方面入手

#### 监控指标
> CPU使用率
因为我们强调的是应用性能，所以系统态CPU使用率越低越好
> 网络IO
> 磁盘IO,空间
空间的使用情况
> 内存使用率
> 锁竞争

## 持续集成

## 技术理念
### 分布式和高并发
分布式和高并发没有必然关系， 高并发在任何场景都会涉及，只要用户端同时批量请求，服务端会多线程或进程处理用户请求，在多核服务端处理下，必然会有一致性问题，
而分布式是用来面对大流量请求的一种架构设计，是建立在网络通信基础上的，带来伸缩性上的方便的同时增加了系统的复杂程度,分布式也是通常用来应对大流量的一种基数方案，
大流量下必然会有高并发的问题，就导致了分布式和高并发通常是同时出现

#### 并发问题
> 文件锁
解决进程间对同一资源的占用访问问题

## 开发注意事项
* 数字的更新操作不能用set操作，应该用原子性的incr操作，mysql和redis都有使用场景 
## Java开发规范
> 空指针异常
* 字符串常量作为equal调用方，能避免nullpoint
## Java开发工具
> IDEA开发插件
- 阿里巴巴编码规范插件安装
- lombok插件配置
- mybatis插件Free-idea-mybatis安装
- Rainbow Brackets （括号高亮）插件安装




### 个人情况

没什么想说的




### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).



