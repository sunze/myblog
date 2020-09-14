# 一夜技术博客
**拆解技术书籍**
## 云原生技术
### telepresence
### Kubernetes
Kubernetes核心是对Pod对控制
> kubernetes的健康检查
* HTTP GET获取POD状态
* TCP Socket状态检查
* Exec 命令方式 

> kubernetes的services
k8s的pod的ip是可变不固定的，多pod的ip也不同，需要service作为固定唯一的访问入口
集群内部service访问:添加selector, 环境变量和FQDN
集群外部service访问:不指定selector, 创建endpoints资源关联外部集群外部服务ip:port

> kubernetes的网络访问机制
k8s尽开放了pod间的相互访问，pod的ip对外部是不可见的，外网访问k8s的机制有一下几种
* NodePort(重定向外部包到service)
* LoadBalancer(NodePort的扩展类型) 
* Ingress(HTTP网络层) 

> kubernetes的无状态控制器
* ReplicationController[已废弃]
* ReplicaSets(Pod模版, 标签选择，Pod数)
* DaemonSet(一个Node一个Pod)
* Job/CronJob
> kubernetes的有状态控制器
> kubernetes的自定义控制器

## 分布式应用框架
### Dubbo
### SpringCloud
### Spring,SpringMvc,SpringBoot
作为知名的MVC框架，随着前后端分离的持续推进，越来越多应用不再使用MVC的模式, 主要原因是模版渲染不再被使用

## 数据库
数据库作为持久方式，是提供有状态的服务，数据库的底层还是用文件的形式实现

## 中间件技术
### 缓存
####分布式缓存Redis
####应用本地缓存
### 索引
### 消息队列
### 配置中心
### 数据库中间件
### 分布式任务框架
### 分布式存储方案
### 唯一ID生成服务
### 告警系统
Minlo

## 接入层网关

## 网络协议
### HTTP/HTTP2/HTTP3

## 设计模式
### 代理模式
代理模式被使用来增加对象能力和统一工作的能力

## 技术理念
### 分布式和高并发
分布式和高并发没有必然关系， 高并发在任何场景都会涉及，只要用户端同时批量请求，服务端会多线程或进程处理用户请求，在多核服务端处理下，必然会有一致性问题，
而分布式是用来面对大流量请求的一种架构设计，是建立在网络通信基础上的，带来伸缩性上的方便的同时增加了系统的复杂程度,分布式也是通常用来应对大流量的一种基数方案，
大流量下必然会有高并发的问题，就导致了分布式和高并发通常是同时出现


## 开发注意事项
* 数字的更新操作不能用set操作，应该用原子性的incr操作，mysql和redis都有使用场景 


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


### 个人情况

互联网开发人员，专注代码三十年，一心一意过人生

