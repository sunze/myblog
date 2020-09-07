# 一夜技术博客
**拆解基数书籍**
## 云原生技术
### telepresence
### Kubernetes
Kubernetes核心是对Pod对控制
> kubernetes的网络访问机制
k8s尽开放了pod间的相互访问，pod的ip对外部是不可见的，外网访问k8s的机制有一下几种
* NodePort
* 
* 

> kubernetes的无状态控制器
* ReplicationController

## 分布式应用框架
###Dubbo
###SpringCloud

## 中间件技术
### 缓存
### 索引
### 消息队列
### 配置中心
### 分布式文件系统

## 接入层网关

## 网络协议
###HTTP/HTTP2/HTTP3

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

