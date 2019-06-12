### Dubbo笔记
#### 1.系统间的通信
如何实现远程通信？
1. 使用webservice
效率不高，基于soap协议（http+xml)
2. 使用restful形式的服务
http+json，如果服务越来越多，服务于服务之间的调用关系复杂，
3. 使用dubbo
使用rpc协议进行远程调用，直接使用socket通信，传输效率高，并且可以统计出系统之间的调用关系，调用次数，管理服务

#### 2. Dubbo的介绍
分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案
Dubbo 就是资源调度和治理中心的管理工具。
Dubbo 就是类似于webservie的关于系统之间通信的框架，
并可以统计和管理服务之间的调用情况（包括服务被谁调用，调用的次数，以及服务的使用状况）

#### 3. Dubbo的架构
![架构图](https://github.com/liuyashuang/Dubbo-/blob/master/img/dubbo.png)</br>
**Provider:服务的提供者，将服务发布出去（Spring)，
发布的内容有：方法，IP地址，端口等等 交给注册中心**</br>
<br>**Consumer:服务的调用者。subscribe到注册中心订阅，注册中心通过notify提醒Consumer**</br>
<br>**Registry：注册中心**</br>
<br>**Monitor:监控中心，来统计服务被谁调用，调用次数 调用时间 等等**</br>

#### 3.1 使用方法
##### 3.1.1 Spring配置
<p>将服务定义部分放在服务提供方，remote-provider.xml,将服务引用部分放在服务消费方remote-consumer.xml</p>
并在提供方增加暴露服务配置 < dubbo:service> ,在消费方增加引用服务配置< dubbo:reference></br>

![配置](https://github.com/liuyashuang/Dubbo-/blob/master/img/111.png)

#### 3.2 注册中心
##### 3.2.1zookeeper
注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心
不转发请求，压力较小。</br>
<br>Zookeeper是Apache Hadoop的子项目，是一个树形的目录服务，支持变更推送，适合作为Dubbo服务的注册中心，

#### 3.3 添加Dubbo依赖（服务层、表现层都需要添加）
#### 3.4 框架整合
##### 3.4.1 整合思路
###### 1.Dao层
<br>mybatis整合spring,通过Spring管理SqlSessionFactory、mapper代理对象。需要mybatis和Spring的整合包，由spring创建数据库连接池。

整合内容|对应工程
---|:---:|
POJO|Taotao-manger-POJO
Mapper映射文件|Taotao-manger-dao
Mapper接口|Taotao-manger-dao
SqlMapConfig.xml(全局配置)|Taotao-manger.service
applicationContext-dao.xml(数据库连接池)|Taotao-manger-service

###### 2. Service层
所有的实现类都放到spring容器中管理。并有spring管理事务，发布dubbo

整合内容|对应工程
---|:---:|
Service接口|Taotao-manger-interface
Service实现类|Taotao-manger-service
applicationContext-service.xml|Taotao-manger-service
applicationContext-trans.xml|Taotao-manger.service
<br>web.xml配置：配置加载spring容器
###### 3.表现层
Springmvc整合spring框架，由springmvc管理controller，引入dubbo服务

整合内容|对应工程
---|:---:|
springmvc.xml|Taotao-manger-web
Controller|Taotao-manger-web
<br>web.xml的配置：前端控制器的配置，配置URL拦截形式
