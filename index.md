# 一夜技术博客

## I/O:经典I/O、NIO、NIO2
#### 经典I/O(BIO)
经典I/O的场景是基于文件流的I/O
File对象针对文件做相关操作和RandomAccessFile对文件进行读写操作
InputStream和OuputStream是面向字节流的输入输出
Writer和Reader是面向字符集的输入输出
面向字符和字节的I/O的类基本是类似的
#### NIO
NIO是基于通道和缓冲区的形式进行数据处理的，NIO是双向的
NIO和组成Buffer和Channel



##事务、锁、死锁



## Java标准库:集合类
## Java标准库:网络net
## Java标准库:并发current
## Java标准库:工具util

## Java Spi, Spring Spi, Dubbo Spi
SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件
只要按照SPI的约定编写接口和对应实现类和配置文件，就能自动加载对应的服务提供实现
1. 在META-INF/services/ 目录中创建以接口全限定名命名的文件，该文件内容为API具体实现类的全限定名
2. 使用ServiceLoader类动态加载 META-INF 中的实现类
3. 如 SPI 的实现类为 Jar 则需要放在主程序 ClassPath 中
4. API 具体实现类必须有一个不带参数的构造方法

### 服务接口类
```java
package com.sz.spi;
/**
 * @author sunze
 * @date 2020/10/9
 */
public interface Log {

    /**
     * 打印日志信息
     * @param str
     */
    void info(String str);
}
```
### 实现类
```java
package com.sz.impl;
import com.sz.spi.Log;
/**
 * @author sunze
 * @date 2020/10/9
 */
public class TestLog implements Log {
    @Override
    public void info(String s) {
        System.out.println("Test:" + s);
    }
}
```
### 实现类
```java
package com.sz.impl;
import com.sz.spi.Log;

/**
 * @author sunze
 * @date 2020/10/9
 */
public class DevLog implements Log {
    @Override
    public void info(String s) {
        System.out.println("Dev:" + s);
    }
}
```
### 测试类
```java
package com.sz;
import com.sz.spi.Log;
import java.util.Iterator;
import java.util.ServiceLoader;

/**
 * @author sunze
 * @date 2020/10/9
 */
public class SpiTest {
    public static void main(String[] args) {
        ServiceLoader<Log> peoples = ServiceLoader.load(Log.class);
        Iterator<Log> iterator = peoples.iterator();
        while (iterator.hasNext()) {
            Log log = iterator.next();
            log.info("hellow wolrd");
        }
    }
}
```
运行结果：
Test:hellow wolrd
Dev:hellow wolrd


## Java原生调用

**拆解技术书籍**
## 云原生技术
云原生让分布式服务端系统成为可能

| 单机系统         | k8s分布式系统      |
| ---------------- | ------------------ |
| 进程             | 应用系统           |
| 单应用           | 分布式             |
| 进程间通信       | 网络间通信         |
| 资源管控能力有限 | 统一的资源管控     |
| 文件系统         | PV/PVC             |
| 计算能力有限     | 计算能力理论上无限 |
| 单机硬件         | 集群机器           |

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
### 消息中间件

> 常见的消息中间件

> 如何防止消息丢失？

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
设计模式更多的是对过去编程经验的总结，形成的一套抽象的语言系统描述背后的设计思想，设计模式提高了维护效率但不会提高运行效率,本质上是对代码的组织维护的提升,加强整个工程的稳定性,是一种潜在的编程约定让其他人不会破坏这种约定
### 开放封闭原则OCP（对扩展开放，对修改封闭）
是所有面向对象原则的核心, 软件设计本身所追求的目标就是封装变化、降低耦合，而开放封闭原则正是对这一目标的最直接体现.
### 代理模式
代理模式被使用来实现对象的访问控制，对调用方隐藏真正的对象

> 代理模式的实现方式

### 装饰者模式
装饰者模式实现了对象的增强, 和基础对象拥有一样的方法，持有基础对象，同名方法在调用基础对象后会做额外的行为，Java I/O包就使用了这个设计模式。

### 门面模式



## 编码技巧

####Builder构建

在类构建参数很多的情况下，传参会成为潜在的问题，调用set方法的构建无法确保一致性，build模式能避免这种情况，通过编写类的Builder内部类来实现。这种技巧在许多工程项目中被用到

```java
public class People {

    private String name;
    private String state;
    private int age;
    private double height;

    public static class Builder {

        private String name = "";
        private String state = "";
        private int age = 0;
        private double height = 0.0;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder state(String state) {
            this.state = state;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder height(double height) {
            this.height = height;
            return this;
        }

        public People build() {
            return new People(this);
        }

    }

    private People (Builder builder) {
        this.name = builder.name;
        this.state = builder.state;
        this.age = builder.age;
        this.height = builder.height;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", state='" + state + '\'' +
                ", age=" + age +
                ", height=" + height +
                '}';
    }

    public static Builder builder() {
        return new Builder();
    }

    public static void main(String[] args) {
        People people = People.builder()
                .age(19)
                .state("china")
                .name("王飞")
                .height(180.5)
                .build();
        System.out.println(people);
    }
}
```

#### 依赖注入

在组合对象时， 不建议直接构建所依赖的对象， 这会使依赖对象难以测试，通常通过构建函数注入的方式，将所依赖对象作为参数注入，依赖注入在spring等框架都已经实现，并广泛使用





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

持续集成的应用构成

> Git等代码管理工具

> 开发IDE静态规范检查

> CICD流水线

> 制品管理系统


## 编程语言

### Java应用启动过程
1. 解析命令行选项
2. 设置堆的大小和JIT编译器
3. 设置环境变量D_LIBRARY_PATH和CLASSPATH
4. JAR的manifest查找Main-Class或者从命令行读取Main-Class
5. 创建HotSpot VM线程
6. 加载JAVA Main-Class
7. HotSpot VM调用Java main和命令行参数传递
8. 

###java程序的调优和优化-JVM启动参数
java应启动，是通过启动JRE环境，加载指定的类，调用类的main函数，args的第一个参数通常是制定加载的类，或者如果-jar参数存在，就是JAR包的名称：
    public static void main(String[] args)
java命令参数支持许多种参数，可以分为一下几种类别：

- 标准参数
- 非标准参数
- 高级运行时参数
- 高级编译参数
- 高级服务参数
- 高级垃圾回收参数

标准参数(是最经常使用的参数，所有的JVM实现都会支持)
-verbose:jni 打印原声方法调用信息学
-verbose:gc 打印gc信息
-verbose:class 打印类信息
-jar filename 执行jar包程序
-help 显示帮助信息
-Dproperty=value 设置属性值
-server 服务端的模式启动
-client 客户端的模式启动

非标准参数(非标准参数是给HotSpot虚拟机使用的)

-X 显示所有非标准参数

-Xmnsize 设置年轻代大小

-Xmssize 设置堆初始化大小

-Xmxsize 设置分配内存最大值

-Xsssize 设置线程栈大小

-Xnoclassgc 不进行垃圾回收

-Xprof 打印运行记录，在开发环境有用

-XshowSettings:category 显示配置项*

高级运行参数

-XX:ThreadStackSize=size 设置线程栈大小

高级编译参数

高级垃圾回收参数




## 技术理念
### 分布式和高并发
分布式和高并发没有必然关系， 高并发在任何场景都会涉及，只要用户端同时批量请求，服务端会多线程或进程处理用户请求，在多核服务端处理下，必然会有一致性问题，
而分布式是用来面对大流量请求的一种架构设计，是建立在网络通信基础上的，带来伸缩性上的方便的同时增加了系统的复杂程度,分布式也是通常用来应对大流量的一种基数方案，
大流量下必然会有高并发的问题，就导致了分布式和高并发通常是同时出现

## web架构
web架构演化至今已经形成了多层架构的模式，网络代理在其中扮演了非常重要的角色，客户端进入到服务端的网络，到请求到达真正的服务器处理之间可能经过多层的网络代理

#### 并发问题
> 文件锁
解决进程间对同一资源的占用访问问题



## 开发注意事项
* 数字的更新操作不能用set操作，应该用原子性的incr操作，mysql和redis都有使用场景 
* lombok的使用是对代码有很强的入侵，隐藏代码的语法糖，而且必须添加IDE插件和相关依赖, 目前IDE都提供相关函数自动生成的工具，编写工作没我们想的那么繁重，
  对lombok的使用其实应该慎重
* mybatis-plus这个插件不建议使用，不是事实的规范，增加维护成本，造成编写的代码及其随意
* 对不熟悉的库谨慎使用



## Java开发注意事项
* （避免空指针异常）字符串常量作为equal调用方，能避免nullpoint
* （私有化构造函数）像Arrays, Collections， 对不可实例化的类，通常作为工具类，需要将构造函数设置为私有函数
* （避免不必要的自动装箱拆箱）不必要的自动装箱拆箱会频繁构造对象，函数内部能使用原始类型计算就使用，能有效减少原始类型的装箱操作
* （避免内存泄漏）java虽然会自动回收对象但是对不再使用的对象如果不消除，也会造成内存泄漏
* （尽量使用不可变对象）不可变对象能有效保持程序的健壮性



### Java开发问题和解决

> JVM的OOM



## Java开发工具
> IDEA开发插件
- 阿里巴巴编码规范插件安装
- lombok插件配置
- mybatis插件Free-idea-mybatis安装
- Rainbow Brackets （括号高亮）插件安装






### 个人情况
- 系统：linux、kubernetes
- 语言：Java、PHP、Goland、Python
- 数据库: mysql
- 框架: Dubbo

