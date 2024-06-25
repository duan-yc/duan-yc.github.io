---

layout: post
title: "RocketMQ"
subtitle: "RocketMQ"
author: "DYC"
header-img: "img/post-bg-linux.jpg"
header-mask: 0.3
catalog: true
tags:
- java
---

#### RocketMQ使用场景
###### 异步耦合
最常见的一个场景是用户注册后，需要发送注册邮件和短信通知，以告知用户注册成功。
传统的做法有以下两种：
**串行方式**
耗时150ms
![image.png](https://cdn.nlark.com/yuque/0/2024/png/42839395/1719219244119-c865c49c-cc8f-4dc4-b8a4-afd4ebec48f1.png#averageHue=%23fecba7&clientId=ucc68065f-84c7-4&from=paste&height=158&id=u900a4034&originHeight=237&originWidth=1322&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=35219&status=done&style=none&taskId=u338395f1-0f0a-4e45-bc14-1314467f9be&title=&width=881.3333333333334)
**并行方式**
耗时100ms
![image.png](https://cdn.nlark.com/yuque/0/2024/png/42839395/1719219358364-fe386699-bde5-4f29-a0f1-20fe37e6033a.png#averageHue=%23fedec6&clientId=u12563593-f9fa-4&from=paste&height=378&id=u7303e80b&originHeight=567&originWidth=1029&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=57498&status=done&style=none&taskId=ud13cda91-b5dc-472a-857d-cf510cd75dd&title=&width=686)
**异步解耦**
发送信息不是自己关注的，所以将这两个任务加到消息队列中，进行异步完成，所以耗时为55ms
![image.png](https://cdn.nlark.com/yuque/0/2024/png/42839395/1719220191241-323b391b-55a8-4d1b-b180-f57309490316.png#averageHue=%23fee8d7&clientId=u8d9075f4-909d-4&from=paste&height=353&id=u0ec983b4&originHeight=530&originWidth=1308&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=66895&status=done&style=none&taskId=ue23c36a8-da49-497e-b2be-49f1bc1d864&title=&width=872)
###### 流量削峰
在秒杀或团队抢购活动中，由于用户请求量较大，导致流量暴增，秒杀的应用在处理如此大量的访问流量后，下游的通知系统无法承载海量的调用量，甚至会导致系统崩溃等问题而发生漏通知的情况。为解决这些问题，可在应用和下游通知系统之间加入 RocketMQ
秒杀处理流程如下所述：

1. 用户发起海量秒杀请求到秒杀业务处理系统。
2. 秒杀处理系统按照秒杀处理逻辑**将满足秒杀条件的请求发送 RocketMQ。**
3. 下游的通知系统订阅 RocketMQ 的秒杀相关消息，再将秒杀成功的消息发送到相应用户。
4. 用户收到秒杀成功的通知
###### 顺序信息
顺序消息是 RocketMQ 提供的一种对消息发送和消费顺序有严格要求的消息
顺序消息分为分区顺序消息和全局顺序消息。

- 分区顺序消息：对于指定的一个 Topic，所有消息根据 Sharding Key 进行区块分区，同一个分区内的消息按照严格的先进先出（FIFO）原则进行发布和消费。**同一分区内的消息保证顺序，不同分区之间的消息顺序不做要求**。
   - 适用场景：适用于性能要求高，以 Sharding Key 作为分区字段，在同一个区块中严格地按照先进先出（FIFO）原则进行消息发布和消费的场景。
   - 示例
      - 用户注册需要发送验证码，以用户 ID 作为 Sharding Key，那么同一个用户发送的消息都会按照发布的先后顺序来消费。
      - 电商的订单创建，以订单 ID 作为 Sharding Key，那么同一个订单相关的创建订单消息、订单支付消息、订单退款消息、订单物流消息都会按照发布的先后顺序来消费。
- 全局顺序消息：对于指定的一个Topic，**所有消息按照严格的先入先出（FIFO）的顺序来发布和消费**。
   - 适用场景：适用于性能要求不高，所有的消息严格按照 FIFO 原则来发布和消费的场景。
   - 示例：在证券处理中，以人民币兑换美元为 Topic，在价格相同的情况下，先出价者优先处理，则可以按照 FIFO 的方式发布和消费全局顺序消息
###### 分布式模式缓存同步
使用缓存技术也无法满足对商品价格的访问需求，缓存服务器网卡满载。访问较多次商品价格查询影响会场页面的打开速度。
此时需要提供一种广播机制，一条消息本来只可以被集群的一台机器消费，如果使用 **RocketMQ 的广播消费模式**，**那么这条消息会被所有节点消费一次，相当于把价格信息同步到需要的每台机器上，取代缓存的作用**。
#### 基本概念
###### 主题Topic
Topic是RocketMQ中**消息传输和存储的顶级容器**，用于标识同一类业务逻辑的消息
作用如下：

- 定义数据的分类隔离：将不同业务类型的数据拆分到不同主题中，通过主题实现存储的隔离性和订阅的隔离性
- 定义数据的身份和权限：RocketMQ的消息本身是匿名无身份的，同一分类的消息使用相同的主题来身份实现和权限管理
###### 队列
队列是消息存储和传输的实际容器，**是消息队列的最小存储单元**，RocketMQ的所有**主题都是由多个队列组成**，以此实现队列数量的水平拆分和队列内部的流失存储
###### 消息
消息是**最小的数据传输单元**，**生产者**将**业务数据的负载和拓展属性**包装成消息发送给**消息队列服务端**，服务端按照相关语义将消息投递到消费者端进行消费
###### 生产者
发布消息的角色，通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递，投递过程支持快速失败和重试
###### 消费者
消息消费的角色

- 支持以推（push），拉（pull）两种模式对消息进行消费。
- 同时也支持集群方式和广播方式的消费。
- 提供实时消息订阅机制，可以满足大多数用户的需求。
###### 代理服务器Broker
负责消息的存储、投递和查询以及服务高可用保证
在 Master-Slave 架构中，Broker 分为 Master 与 Slave。一个 Master 可以对应多个 Slave，但是一个 Slave 只能对应一个 Master。Master 与 Slave 的对应关系通过指定相同的 BrokerName，不同的 BrokerId 来定义，BrokerId 为 0 表示 Master，非 0 表示 Slave。Master 也可以部署多个
###### 名字服务器NameServer
是一个简单的Topic路由注册中心，支持Topic、Broker的动态注册与发现

- Broker管理，NameServer **接受 Broker 集群的注册信息并且保存下来作为路由信息的基本数据**。然后提供心跳检测机制，**检查 Broker **是否还存活；
- 路由信息管理，每个 NameServer **将保存关于 Broker 集群的整个路由信息和用于客户端查询的队列信息**。Producer 和 Consumer 通过 NameServer 就可以知道整个 Broker 集群的路由信息，从而进行消息的投递和消费。
#### RocketMQ工作原理
![image.png](https://cdn.nlark.com/yuque/0/2024/png/42839395/1719241690038-7656a9cc-2013-49c6-93c1-442bd81bb763.png#averageHue=%23d6d6d6&clientId=uc5e3ed2a-7761-4&from=paste&height=415&id=uca27d5f6&originHeight=830&originWidth=1370&originalType=binary&ratio=2&rotation=0&showTitle=false&size=206340&status=done&style=none&taskId=ua695b482-352c-4ff5-8993-2b38d6284b6&title=&width=685)

1. 启动NameServer

启动 NameServer。NameServer 启动后监听端口，等待 Broker、Producer、Consumer 连接，相当于一个**路由控制中心**

2. 启动Broker

启动 Broker。与所有 NameServer 保持长连接，定时发送心跳包。心跳包中包含当前 Broker 信息以及**存储所有 Topic 信息**。注册成功后，NameServer 集群中就有 Topic 跟 Broker 的映射关系

3. 创建Topic

创建Topic时需要指定要存储在哪些Broker上，也可以在发送消息时自动创建Broker

4. 生产者发送消息

生产者发送消息。启动时**先跟 NameServer 集群中的其中一台建立长连接**，并从 **NameServer 中获取当前发送的 Topic 存在于哪些 Broker **上，轮询从队列列表中选择一个队列，然后**与队列所在的 Broker 建立长连接从而向 Broker发消息**

5. 消费者接受消息

消费者根据订阅信息（订阅哪个Topic，订阅哪些Tag等）从Broker拉取消息。跟**其中一台 NameServer 建立长连接**，获取当前**订阅 Topic 存在哪些 Broker 上**，然后直接**跟 Broker 建立连接通道**，然后开始消费消息
#### 实操
###### 启动RocketMQ
安装NameServer
```java
docker run -d -p 9876:9876 --name rmqnamesrv foxiswho/rocketmq:server-4.5.1
```
安装Broker

1. 创建目录
```java
mkdir -p ${HOME}/docker/software/rocketmq/conf
```

2. 新建配置文件
```java
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# 此处为本地ip, 如果部署服务器, 需要填写服务器外网ip
brokerIP1 = xx.xx.xx.xx
```

3. 创建容器
```java
docker run -d \
-p 10911:10911 \
-p 10909:10909 \
--name rmqbroker \
--link rmqnamesrv:namesrv \
-v ${HOME}/docker/software/rocketmq/conf/broker.conf:/etc/rocketmq/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "JAVA_OPTS=-Duser.home=/opt" \
-e "JAVA_OPT_EXT=-server -Xms512m -Xmx512m" \
foxiswho/rocketmq:broker-4.5.1
```
安装RocketMQ控制台
```java
docker pull pangliang/rocketmq-console-ng
docker run -d \
--link rmqnamesrv:namesrv \
-e "JAVA_OPTS=-Drocketmq.config.namesrvAddr=namesrv:9876 -Drocketmq.config.isVIPChannel=false" \
--name rmqconsole \
-p 8088:8080 \
-t pangliang/rocketmq-console-ng
```
浏览器输入localhost:8088 查看控制台
###### 发送普通消息
**引入RocketMQ依赖**
```java
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.2.3</version>
</dependency>

```
**启动自动装配**
项目使用的是SpringBoot3，项目中使用到的RocketMQ是2.2.3，没有适配SpringBoot3，所以需要手动搞定自动装配（如果是SpringBoot2就不需要这一步）
resources 目录下创建 META-INF/spring 目录，并创建org.springframework.boot.autoconfigure.AutoConfiguration.imports 文件。
在文件中加入下面配置
```java
# RocketMQ 2.2.3 version does not adapt to SpringBoot3
org.apache.rocketmq.spring.autoconfigure.RocketMQAutoConfiguration

```
**配置消息生产者**
配置文件中引入 RocketMQ 相关配置定义，比如连接 NameServer 地址等
```java
server:
  port: 6060

rocketmq:
  name-server: 127.0.0.1:9876 # NameServer 地址
  producer:
    group: rocketmq-4x-service_common-message-execute_pg # 全局发送者组定义

```
定义**消息生产者**，通过** RocketMQTemplate** 向 RocketMQ 发送普通常规消息
```java
import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson.JSON;
import com.nageoffer.springbootladder.rocketmq4x.event.GeneralMessageEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.MessageConst;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

/**
 * 普通消息发送者
 *
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class GeneralMessageDemoProduce {

    private final RocketMQTemplate rocketMQTemplate;

    /**
     * 发送普通消息
     *
     * @param topic            消息发送主题，用于标识同一类业务逻辑的消息
     * @param tag              消息的过滤标签，消费者可通过Tag对消息进行过滤，仅接收指定标签的消息。
     * @param keys             消息索引键，可根据关键字精确查找某条消息
     * @param messageSendEvent 普通消息发送事件，自定义对象，最终都会序列化为字符串
     * @return 消息发送 RocketMQ 返回结果
     */
    public SendResult sendMessage(String topic, String tag, String keys, GeneralMessageEvent messageSendEvent) {
        SendResult sendResult;
        try {
            StringBuilder destinationBuilder = StrUtil.builder().append(topic);
            if (StrUtil.isNotBlank(tag)) {
                destinationBuilder.append(":").append(tag);
            }
            Message<?> message = MessageBuilder
                    .withPayload(messageSendEvent)
                    .setHeader(MessageConst.PROPERTY_KEYS, keys)
                    .setHeader(MessageConst.PROPERTY_TAGS, tag)
                    .build();
            sendResult = rocketMQTemplate.syncSend(
                    destinationBuilder.toString(),
                    message,
                    2000L
            );
            log.info("[普通消息] 消息发送结果：{}，消息ID：{}，消息Keys：{}", sendResult.getSendStatus(), sendResult.getMsgId(), keys);
        } catch (Throwable ex) {
            log.error("[普通消息] 消息发送失败，消息体：{}", JSON.toJSONString(messageSendEvent), ex);
            throw ex;
        }
        return sendResult;
    }
}

```
**配置消息消费者**
定义**消息消费者**，从** RocketMQ Broker** 拉取对应** Topic Tag 的消息列表**
```java
import com.alibaba.fastjson.JSON;
import com.nageoffer.springbootladder.rocketmq4x.event.GeneralMessageEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;

/**
 * 普通消息消费者
 *
 */
@Slf4j
@Component
@RequiredArgsConstructor
@RocketMQMessageListener(
        topic = "rocketmq-demo_common-message_topic",
        selectorExpression = "general",
        consumerGroup = "rocketmq-demo_general-message_cg"
)
public class GeneralMessageDemoConsume implements RocketMQListener<GeneralMessageEvent> {

    @Override
    public void onMessage(GeneralMessageEvent message) {
        log.info("接到到RocketMQ消息，消息体：{}", JSON.toJSONString(message));
    }
}

```
**发送消息**
发送普通消息的方法返回值就是发送 RocketMQ Broker 返回的状态码，成功的话就是 SEND_OK
```java
import com.nageoffer.springbootladder.rocketmq4x.event.GeneralMessageEvent;
import com.nageoffer.springbootladder.rocketmq4x.produce.GeneralMessageDemoProduce;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.apache.rocketmq.client.producer.SendResult;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@RequiredArgsConstructor
@SpringBootApplication
@Tag(name = "RocketMQ发送示例", description = "RocketMQ发送示例启动器")
public class RocketMQDemoApplication {

    private final GeneralMessageDemoProduce generalMessageDemoProduce;

    @PostMapping("/test/send/general-message")
    @Operation(summary = "发送RocketMQ普通消息")
    public String sendGeneralMessage() {
        String keys = UUID.randomUUID().toString();
        GeneralMessageEvent generalMessageEvent = GeneralMessageEvent.builder()
                .body("消息具体内容，可以是自定义对象，最终都会序列化为字符串。如果是取消订单，这里应该是订单ID或者相关联的信息")
                .keys(keys)
                .build();
        SendResult sendResult = generalMessageDemoProduce.sendMessage(
                "rocketmq-demo_common-message_topic",
                "general",
                keys,
                generalMessageEvent
        );
        return sendResult.getSendStatus().name();
    }

    public static void main(String[] args) {
        SpringApplication.run(RocketMQDemoApplication.class, args);
    }
}

```
项目中引入了 Swagger3，访问 http://127.0.0.1:6060/swagger-ui/index.html，调用定义的发送 RocketMQ 普通消息方法
#### RocketMQ部署架构
###### 本地部署
单组节点单副本模式
这种方式风险较大，因为 Broker 只有一个节点，一旦Broker重启或者宕机时，会导致整个服务不可用。不建议线上环境使用, 可以用于本地测试
多组节点单副本模式
一个集群内全部部署 Master 角色，不部署 Slave 副本，例如2个 Master 或者3个 Master，这种模式的优缺点如下：

- 优点：配置简单，单个 Master 宕机或重启维护对应用无影响，在磁盘配置为 RAID10 时，即使机器宕机不可恢复情况下，由于 RAID10 磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢），性能最高；
- 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响
###### 生产部署
多节点多副本模式--异步复制
每个 Master 配置一个 Slave，有多组 Master-Slave，HA 采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下：

- 优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，同时 Master 宕机后，消费者仍然可以从 Slave 消费，而且此过程对应用透明，不需要人工干预，性能同多 Master 模式几乎一样；
- 缺点：Master 宕机，磁盘损坏情况下会丢失少量消息

多节点多副本模式--同步双写
每个 Master 配置一个 Slave，有多对 Master-Slave，HA 采用同步双写方式，即只有主备都写成功，才向应用返回成功，这种模式的优缺点如下：

- 优点：数据与服务都无单点故障，Master 宕机情况下，消息无延迟，服务可用性与数据可用性都非常高；
- 缺点：性能比异步复制模式略低（大约低10%左右），发送单个消息的RT会略高，且目前版本在主节点宕机后，备机不能自动切换为主机。
