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
分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案</br>
Dubbo 就是资源调度和治理中心的管理工具。</br>
Dubbo 就是类似于webservie的关于系统之间通信的框架</br>
并可以统计和管理服务之间的调用情况（包括服务被谁调用，调用的次数，以及服务的使用状况）</br>
**核心功能**
  1. Remoting:远程通讯，提供对多种NIO框架抽象封装，包括“同步转异步”和“请求-响应”模式的
  信息交换方式。</br>
  2. Cluster：服务框架，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载
  均衡，失败容错，地主路由，动态配置等集群支持。</br>
  3. Registry：服务注册，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地
  址透明，使服务提供方可以平滑增加或减少机器。

#### 3. Dubbo的架构
![架构图](https://github.com/liuyashuang/Dubbo-/blob/master/img/dubbo.png)</br>
**Provider:服务的提供者，将服务发布出去（Spring)，
发布的内容有：方法，IP地址，端口等等 交给注册中心**</br>
<br>**Consumer:服务的调用者。subscribe到注册中心订阅，注册中心通过notify提醒Consumer**</br>
<br>**Registry：注册中心**</br>
<br>**Monitor:监控中心，来统计服务被谁调用，调用次数 调用时间 等等**</br>
<br>**Container:服务运行容器，常见的容器有Spring容器</br>**
##### <br>调用关系说明:</br>
    1. 服务容器负责启动、加载、运行服务提供者
    2. 服务提供者在启动时，向注册中心注册自己提供的服务
    3. 服务消费者在启动时，想注册中心订阅自己所需的服务
    4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更
       数据给消费者
    5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调
       用失败，再选另一台调用
    6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到
       监控中心Monitor

##### Dubbo总体架构</br>
![总体架构](https://github.com/liuyashuang/Dubbo-/blob/master/img/dubbo1.png)</br>
Dubbo 框架设计一共划分了10个层，而最上面的Service层是留给实际想要使用Dubbo开发分布式服</br>
务的开发者实现业务逻辑的接口层。左边淡蓝色背景的为服务消费方使用的接口，右边淡绿色背景的</br>
为服务提供方使用的接口，位于中轴线上的为双方都用到的接口</br>

**各层设计要点：**</br>

    服务接口层(Service)：与实际业务逻辑相关的，根据服务提供方和服务消费方的业务设计对应
    的接口和实现

    配置层(Config)：对外配置接口，以ServiceConfig和ReferenceConfig为中心，可以直接new
    配置类，也可以通过Spring解析配置生成配置类

    服务代理层(Proxy)：服务接口透明代理，生成服务的客户端Stub和服务器端Skeleton,以Service
    Proxy为中心，扩展接口为ProxyFactory

    服务注册层(Registry)：封装服务地址的注册与发现，以服务URL为中心，扩展接口为Registryf
    actory、Registry和RegistryService。可能没有服务注册中心，此时服务提供方直接暴露服务。

    集群层(Cluster)：封装多个提供者的路由及负债均衡，并桥接注册中心，以Invoker为中心，扩
    展接口为Cluster、Directory、Router、和LoadBalance。将多个服务提供方组合为一个服务
    提供方，实现对服务消费方来透明，只需要与一个服务提供方进行交互

    监控层(Monitor)：RPC调用次数和调用时间监控，以Statistics为中心，扩展接口为Monitor
    Factory、Monitor和MonitorService。

    远程调用层(Protocol)：封装RPC调用，以Invocation和Result为中心，扩展接口为Protocol
    Invoker和Exporter。Protocols是服务域，它是Invoker暴露和引用的主功能入口，它负责
    Invoker的生命周期管理。Invoker是实体域，它是Dubbo的核心模型，其它模型都向它靠扰，
    或转换成它，它代表一个可执行体，可向它发起invoke调用，它有可能是一个本地的实现，
    也可能是一个远程的实现，也可能一个集群实现。

    信息交换层(Exchange)：封装请求响应模式，同步转异步，以Request和Response为中心，扩
    展接口为Exchanger、ExchangeChannel、ExchangeClient和ExchangeServer

    网络传输层(Transport):抽象mina和netty接口，以Message为中心，扩展接口为Channel、
    Transporter、Client、Server和Codec

    数据序列化层(Serialize)：可复用的一些工具，扩展接口为Seriaization、ObjectInput、
    ObjectOutput和ThreadPool


**服务调用流程**</br>
![调用流程](https://github.com/liuyashuang/Dubbo-/blob/master/img/dubbo2.png)</br>
**连接方式**</br>

    1. 直连：不通过注册中心，直接由消费者访问提供者
    2. 集群：提供者把服务注册到注册中心，然后消费者询问注册中心，请求对应的服务需要请求
            哪个提供者，注册中心返回结果，消费者根据结果向提供者请求服务，
    3. 超时和重连机制：
        (1)dubbo超时，不会返回结果，直接报异常
        (2)配置超时重试次数、超时时间
        (3)dubbo在调用服务不成功时，默认会重试2次。
          Dubbo的路由机制，会把超时的请求路由到其他机器上，而不是本机尝试，所以 dubbo
          的重试机器也能一定程度的保证服务的质量。但是如果不合理的配置重试次数，当失败
          时会进行重试多次，这样在某个时间点出现性能问题，调用方再连续重复调用，系统请
          求变为正常值的retries倍，系统压力会大增，容易引起服务雪崩，需要根据业务情况
          规划好如何进行异常处理，何时进行重试。
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
