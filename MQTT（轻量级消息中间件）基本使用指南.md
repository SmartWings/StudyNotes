# MQTT（轻量级消息中间件）基本使用指南
[原文地址](https://blog.csdn.net/footless_bird/article/details/150491442)
备注：怕csdn犯病，将原文转录为md文件，防止后续查找失败。推荐大家使用csdn访问原文链接！

#### 文章目录

  * MQTT 介绍 
  * MQTT 核心特性
    * MQTT 核心组件
    * QoS（服务质量）等级
    * 连接与会话管理
    * 消息结构
    * 安全机制
    * MQTT 在空气净化器小程序中的应用示例
    * 为什么 MQTT 适合物联网？
    * MQTT 服务端选型
    * 开源 MQTT 服务端（适合自建部署）
      * 云服务商托管 MQTT 服务（适合无需自建运维）
      * 轻量级 / 嵌入式 MQTT 服务端（适合边缘设备）
      * 选型建议
  * Eclipse Mosquitto 介绍
  * 核心特性
    * 核心组件与工具
    * 典型配置示例
    * 部署
    * Docker run 部署
      * Docker 编排文件部署
    * 优缺点与适用场景
  * MQTT 可视化客户端（MQTTX）的安装与使用



## MQTT 介绍

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是一种轻量级的**发布 / 订阅模式 （Pub/Sub）** 消息传输协议，专为低带宽、不稳定网络环境设计，广泛应用于物联网（IoT）、移动设备、传感器网络等场景（如空气净化器、智能家居设备的远程控制）。

  


### MQTT 核心特性

  1. **轻量级** ：协议头部精简（最小仅 2 字节），数据传输效率高，适合硬件资源有限的设备（如传感器、嵌入式设备）。
  2. **发布 / 订阅模式** ：消息发送者（发布者）与接收者（订阅者）解耦，无需直接建立连接，通过 “主题（Topic）” 传递消息。
  3. **支持 QoS （服务质量）**：提供三级消息可靠性保障，适应不同网络环境。
  4. **异步通信** ：设备可间歇性联网，重连后仍能接收离线期间的关键消息。
  5. **双向通信** ：支持设备向服务器上报数据（如传感器数据）和服务器向设备下发指令（如控制命令）。

  


### MQTT 核心组件

  1. **发布者 （Publisher）**
     * 发送消息的设备 / 应用（如空气净化器上报 PM2.5 数据、小程序发送控制指令）。
     * 无需知道谁会接收消息，只需指定消息的 “主题（Topic）”。
  2. **订阅者 （Subscriber）**
     * 接收消息的设备 / 应用（如服务器接收设备数据、小程序接收设备状态更新）。
     * 通过订阅 “主题（Topic）” 声明自己感兴趣的消息。
  3. **broker （服务器 / 代理）**
     * 核心组件，负责接收发布者的消息，根据主题分发给所有订阅者。
     * 功能：消息路由、QoS 管理、会话保持、客户端认证等。
     * 常见实现：Eclipse Mosquitto、EMQX、AWS IoT Core、阿里云 IoT 平台等。
  4. **主题 （Topic）**
     * 消息的分类标识，采用层级结构（类似文件路径），用斜杠（`/`）分隔，例如：
       * `device/airpurifier/123/status`（设备 123 的状态上报）
       * `device/airpurifier/123/command`（向设备 123 下发指令）
     * 支持通配符订阅：
       * `+`：匹配单个层级，如`device/+/123/status`匹配所有类型设备的 123 号设备状态。
       * `#`：匹配多个层级（必须在末尾），如`device/airpurifier/#`匹配该设备的所有主题。

  


### QoS（服务质量）等级

MQTT 定义了 3 种 QoS 等级，用于平衡消息可靠性与传输效率：

QoS 等级| 含义| 适用场景  
---|---|---  
QoS 0| **最多一次** ：消息发送后不确认，可能丢失或重复。| 非关键数据（如实时温度周期性上报）。  
QoS 1| **至少一次** ：消息确保到达，但可能重复（发送方收到接收方确认前会重发）。| 重要数据（如设备状态变更）。  
QoS 2| **刚好一次** ：消息确保仅到达一次（通过两次握手确认）。| 关键指令（如支付、设备控制指令）。  
  
**示例** ：空气净化器的开关机指令需用 QoS 2，确保指令仅执行一次；而 PM2.5 实时数据可用 QoS 0，偶尔丢失不影响整体监控。

  


### 连接与会话管理

  * **连接建立**
    * 客户端（设备 / 小程序后端）通过 TCP 连接到 broker 的指定端口（默认 1883，加密端口 8883）。
    * 连接时需发送 **CONNECT** 报文，包含：
      * `clientId`：客户端唯一标识（如设备 SN）。
      * `cleanSession`：是否清除会话（`false`表示 broker 保存离线消息）。
      * 认证信息（`username/password`，或更安全的 TLS 双向认证）。
  * **会话保持**
    * 若 `cleanSession=false`，broker 会为客户端保存：
      * 未确认的 QoS 1/2 消息。
      * 客户端订阅的主题列表。
    * 客户端重连后，broker 会推送离线期间积累的消息（按 QoS 等级处理）。
  * **断开连接**
    * 客户端发送 **DISCONNECT** 报文优雅断开，或网络异常时自动断开。
    * broker 检测到连接中断后，会根据会话设置决定是否保留消息。

  


### 消息结构

MQTT 消息由**固定头部** 、**可变头部** 和**有效载荷 （Payload）** 组成：

  * **固定头部** ：包含消息类型（如 CONNECT、PUBLISH）、QoS 等级、剩余长度等（最小 2 字节）。
  * **可变头部** ：按需存在，如 PUBLISH 消息的可变头部包含主题名和消息 ID。
  * **有效载荷** ：实际传输的数据（如 JSON 格式的设备状态：`{"pm25": 35, "power": "on"}`）。

  


### 安全机制

  * **传输加密** ：通过 TLS/SSL（即 MQTTS）加密通信，防止数据被窃听或篡改（端口 8883）。
  * 身份认证：
    * 基础认证：客户端连接时提供`username/password`，broker 验证后允许接入。
    * 证书认证：客户端与 broker 双向验证 X.509 证书，适合高安全性场景（如工业设备）。
  * **权限控制** ：broker 可配置主题访问权限（如设备只能发布自己的状态主题，不能订阅其他设备的指令主题）。

  


### MQTT 在空气净化器小程序中的应用示例

  1. **设备上报数据** ：

     * 空气净化器（发布者）定期向主题 `device/airpurifier/{sn}/status` 发布状态消息（QoS 1）：
           
           {"pm25": 28, "formaldehyde": 0.03, "power": "on", "filter_remaining": 120}
           

     * 服务器（订阅者）订阅该主题，接收数据并存储到数据库。

  2. **小程序下发指令** ：

     * 小程序后端（发布者）向主题 `device/airpurifier/{sn}/command`

发布控制指令（QoS 2）：
           
           {"action": "set_speed", "speed": "high"}
           

     * 空气净化器（订阅者）订阅该主题，接收指令后执行并返回结果。

  3. **状态同步** ：

     * 服务器订阅设备状态主题后，可将消息转发给小程序（通过 WebSocket 或推送），实现实时状态展示。

  


### 为什么 MQTT 适合物联网？

相比 HTTP 等协议，MQTT 的优势在于：

  * **低带宽消耗** ：适合物联网设备的有限网络（如 GPRS、NB-IoT）。
  * **长连接与异步通信** ：设备无需频繁建立连接，节省功耗（对电池供电设备至关重要）。
  * **灵活的消息路由** ：通过主题和通配符，轻松实现 “一对多”“多对多” 通信（如一个服务器管理上千台设备）。



因此，MQTT 成为物联网设备与云端通信的首选协议之一，尤其适合空气净化器这类需要远程控制和数据监测的场景。

  


### MQTT 服务端选型

选择合适的 MQTT 服务端（Broker）需根据项目规模、部署方式、功能需求和预算综合考虑。以下是针对不同场景的主流 MQTT 服务端推荐，涵盖开源方案、商业服务和轻量级选项：

  


#### 开源 MQTT 服务端（适合自建部署）

**Eclipse Mosquitto**

  * **特点** ：轻量级、易部署、支持 MQTT 3.1.1/5.0 协议，占用资源少（适合边缘设备或小型服务器）。
  * **优势 ：**
    * 单文件部署，配置简单（通过`mosquitto.conf`文件设置端口、认证、权限等）。
    * 支持 TLS 加密、用户名 / 密码认证、访问控制列表（ACL）。
    * 社区活跃，文档丰富，适合新手入门。
  * **适用场景** ：小型物联网项目、开发测试环境、边缘计算设备（如树莓派）。
  * **部署方式** ：支持 Linux、Windows、macOS，可通过包管理器（`apt/yum`）快速安装。

  


**EMQX**

  * **特点** ：高性能、分布式架构，专为大规模物联网场景设计，支持百万级并发连接。
  * **优势 ：**
    * 支持 MQTT 3.1.1/5.0、CoAP、LwM2M 等多协议，兼容主流物联网设备。
    * 内置集群、负载均衡、数据持久化（对接 MySQL/Redis/Kafka）功能。
    * 提供可视化管理控制台（Dashboard），方便监控设备连接、消息吞吐量等指标。
    * 支持规则引擎（如消息转发到数据库、触发 HTTP 回调），简化业务集成。
  * **适用场景** ：中大型物联网平台（如智能家居、工业设备管理）、需要高可用性的生产环境。
  * **版本** ：提供开源社区版（免费）和企业版（付费，含高级功能和技术支持）。

  


**VerneMQ**

  * **特点** ：基于 Erlang 开发，高并发、低延迟，支持水平扩展。
  * **优势 ：**
    * 内置集群功能，可通过简单配置实现节点扩容。
    * 支持 MQTT 5.0 特性（如共享订阅、消息过期时间）。
    * 适合处理高吞吐量的消息场景（如传感器数据采集）。
  * **适用场景** ：对并发和稳定性要求高的分布式系统。

  


#### 云服务商托管 MQTT 服务（适合无需自建运维）

**阿里云 IoT 平台**

  * **特点** ：集成 MQTT Broker、设备管理、数据存储等一站式功能，与阿里云生态（如 ECS、OSS）无缝对接。
  * **优势 ：**
    * 支持设备身份认证（基于设备证书或三元组）、消息加密传输。
    * 提供规则引擎，可将设备数据转发到 RDS、时序数据库（TSDB）等服务。
    * 内置设备影子（Device Shadow），解决设备离线时的指令缓存问题。
  * **适用场景** ：使用阿里云生态的企业级项目，减少自建服务器的运维成本。

  


**腾讯云 IoT Explorer**

  * **特点** ：腾讯云旗下的物联网平台，支持 MQTT 协议，深度整合微信生态（如小程序对接）。
  * **优势 ：**
    * 提供设备调试工具、数据可视化面板，适合快速开发。
    * 支持设备固件升级（OTA）、消息推送（如微信模板消息）。
  * **适用场景** ：面向国内用户的小程序 + 物联网项目（如空气净化器控制小程序）。



**AWS IoT Core**

  * **特点** ：亚马逊云提供的托管 MQTT 服务，全球节点覆盖，适合跨国部署。
  * **优势 ：**
    * 支持 MQTT 3.1.1/5.0，集成 AWS Lambda、DynamoDB 等服务，便于构建 Serverless 架构。
    * 提供细粒度的权限控制（基于 IAM）和设备证书管理。
  * **适用场景** ：需要全球化部署的物联网项目，或已使用 AWS 云服务的用户。

  


#### 轻量级 / 嵌入式 MQTT 服务端（适合边缘设备）

  * **Mosquitto （见上文）**

体积小巧（二进制文件约 1MB），可嵌入到嵌入式系统（如 ARM 架构设备）。

  * **Paho MQTT C/C ++ Broker**

    * **特点** ：轻量级 C 语言实现，适合资源受限的嵌入式设备（如单片机、传感器节点）。
    * **优势** ：可裁剪功能模块，降低内存占用，支持基本的发布 / 订阅功能。

  


#### 选型建议

  * **开发测试 / 小型项目** ：优先选择 **Eclipse Mosquitto** ，部署简单、学习成本低。
  * **中大型生产环境** ：推荐 **EMQX （开源版）** 或 **阿里云 IoT 平台** ，兼顾性能和可扩展性。
  * **小程序 + 物联网场景**：优先考虑 **腾讯云 IoT Explorer** ，便于与微信生态对接，减少跨平台开发成本。
  * **边缘计算 / 嵌入式设备** ：选择 **Mosquitto** 或 **Paho Broker** ，满足低资源消耗需求。



实际选型时，建议先通过 Docker 快速部署试用，测试其在高并发、网络不稳定等场景下的表现，再结合项目预算和运维能力做最终决策。

  


## Eclipse Mosquitto 介绍

Eclipse Mosquitto 是一款轻量级、开源的 MQTT 消息服务器（Broker），由 Eclipse 基金会维护，专为低资源消耗和易用性设计。它支持 MQTT 协议的所有核心特性，广泛应用于物联网（IoT）、传感器网络、智能家居等场景，尤其适合小型项目、边缘设备或开发测试环境。

  


### 核心特性

  * **轻量级设计**
    * 体积小巧（二进制文件仅几 MB），内存占用低（运行时通常仅需几十 MB 内存），适合嵌入式设备（如树莓派、单片机）和资源受限的环境。
    * 单进程架构，部署和维护简单，无需复杂的依赖管理。
  * **完整的 MQTT 协议支持**
    * 兼容 MQTT 3.1、3.1.1 和 MQTT 5.0 协议，支持 QoS 0/1/2 消息等级，满足不同可靠性需求。
    * 支持主题通配符（`+` 匹配单层级、`#` 匹配多层级）、消息留存（Retained Message）、遗嘱消息（Last Will and Testament）等核心功能。
  * **安全机制**
    * 传输加密：支持 TLS/SSL 加密（MQTTS），可配置单向 / 双向证书认证，防止数据窃听和篡改。
    * 身份认证：支持用户名 / 密码认证、基于文件的访问控制列表（ACL），限制客户端对主题的发布 / 订阅权限。
    * 权限细粒度控制：通过配置文件指定特定客户端或用户可访问的主题（如 `user1` 只能订阅 `sensors/temp`）。
  * **多平台支持**
    * 可运行于 Linux、Windows、macOS、FreeBSD 等系统，支持 ARM、x86 等架构，适配嵌入式设备。
  * **扩展功能**
    * 支持 WebSocket 接入，允许浏览器、小程序等通过 WebSocket 协议连接 MQTT 服务器。
    * 提供命令行工具（`mosquitto_pub`/`mosquitto_sub`），方便测试和调试。
    * 支持插件机制，可通过 C 语言编写插件扩展功能（如对接外部认证系统、消息持久化到数据库）。

  


### 核心组件与工具

  * **mosquitto** ：MQTT 服务器主程序，负责接收客户端连接、路由消息。

  * **mosquitto_pub** ：命令行发布工具，用于向指定主题发布消息

示例：
        
        mosquitto_pub -h localhost -t "device/temp" -m '{"value": 25}'
        

  * **mosquitto_sub** ：命令行订阅工具，用于订阅主题并接收消息

示例：
        
        mosquitto_sub -h localhost -t "device/temp"
        

  * **mosquitto_passwd** ：用于管理用户名 / 密码文件（如创建或修改认证用户）

示例：
        
        # 创建用户 user1
        mosquitto_passwd -c /etc/mosquitto/passwd user1
        

  * **配置文件**

    * 默认路径：`/etc/mosquitto/mosquitto.conf`（Linux）或 `C:\Program Files\mosquitto\mosquitto.conf`（Windows）。
    * 用于配置端口、认证方式、权限控制、日志等参数。

  


### 典型配置示例

以下是常见场景的配置示例（修改 `mosquitto.conf`）：

  * `mosquitto.conf`
        
        # 监听 1883 端口（MQTT 未加密）
        listener 1883
        
        # 许匿名连接（测试环境用） true/false
        #allow_anonymous true  
        
        ### 启用用户名/密码认证
        allow_anonymous false  
        
        # 指向密码文件（需用 mosquitto_passwd 命令创建：mosquitto_passwd /etc/mosquitto/passwd user1）
        password_file /mosquitto/config/passwd.conf
        
        # 访问控制列表（ACL），指向 ACL 配置文件
        acl_file /mosquitto/config/acl.conf
        
        
        ### 启用 TLS 加密（MQTTS）
        # 加密端口
        #listener 8883
        
        # CA 证书
        #cafile /etc/mosquitto/ca.crt
        
        # 服务器证书
        #certfile /mosquitto/config/server.crt
        
        # 服务器私钥
        #keyfile /mosquitto/config/server.key
        
        # 不强制客户端提供证书（单向认证）
        #require_certificate false
        

  * `acl.conf`
        
        # 针对特定用户的权限设置
        #user <用户名>
        #topic [read|write|readwrite] <主题>		# 若不指定权限，默认无任何权限
        
        # 针对匿名用户的权限设置（可选）
        #pattern [read|write|readwrite] <主题模式>
        
        
        
        # 用户 admin 可读写所有主题
        user admin
        topic #
        
        # 用户 user1 只能发布 topic/temp1，不能订阅
        user user1
        topic write topic/temp1
        
        # 用户 user2 只能订阅 topic/temp1，不能发布
        user user2
        topic read topic/temp2
        


  


### 部署

#### Docker run 部署

通过 Docker 快速部署 Mosquitto，适合开发测试：

  1. **拉取镜像**
         
         docker pull eclipse-mosquitto:latest
         

  2. **创建持久化目录**
         
         mkdir -p /opt/mosquitto/config /opt/mosquitto/data /opt/mosquitto/log
         chmod -R 777 /opt/mosquitto  # 简化权限（生产环境需细化）
         

  3. **启动容器**
         
         docker run -d \
           --name mosquitto \
           -p 1883:1883 \       # MQTT 未加密端口
           -p 8883:8883 \       # MQTT 加密端口
           -p 9001:9001 \       # WebSocket 端口（默认 9001）
           -v /opt/mosquitto/config:/mosquitto/config \
           -v /opt/mosquitto/data:/mosquitto/data \
           -v /opt/mosquitto/log:/mosquitto/log \
           eclipse-mosquitto:latest
         

  4. **自定义配置**  
在 `/opt/mosquitto/config/mosquitto.conf` 中添加配置，重启容器生效：
         
         docker restart mosquitto
         


  


#### Docker 编排文件部署
    
    
    version: "3.5"
    
    networks:
      host:
        external: true
    
    services:
      mosquitto:
        image: eclipse-mosquitto:2.0.22
        #ports:
          # MQTT 协议默认端口（未加密），设备通过 MQTT 协议连接时需映射
          #- 1883:1883
          # MQTT 协议加密端口（MQTTS，基于 TLS/SSL），需加密传输时映射（推荐生产环境）
          #- 8883:8883
          # MQTT over WebSocket 端口（未加密），网页 / 小程序通过 WebSocket 连接 MQTT 时映射
          #- 9001:9001
        volumes:
          - /data/file/mosquitto/data:/mosquitto/data
          - /data/file/mosquitto/logs:/mosquitto/log
          - /data/file/mosquitto/config:/mosquitto/config
        networks:
          - host
        deploy:
          replicas: 1
          placement:
            constraints:
              - node.labels.mosquitto == mosquitto
          resources:
            limits:
              cpus: '1'
              memory: 2048M
        logging:
          driver: "json-file"
          options:
            max-size: "20M"
    

  


### 优缺点与适用场景

**优点 ：**

  * **轻量级** ：资源占用低，适合嵌入式设备和小型项目。
  * **易用性** ：配置简单，文档完善，上手门槛低。
  * **兼容性** ：支持全版本 MQTT 协议，兼容各种客户端工具和设备。



**缺点 ：**

  * **性能上限较低** ：单节点并发连接数通常在 1 万 - 10 万级，不适合百万级设备的大规模场景。
  * **集群能力弱** ：原生不支持分布式集群（需通过第三方工具或定制开发实现）。
  * **高级功能少** ：无内置可视化控制台、规则引擎等，需依赖外部工具扩展。



**适用场景 ：**

  * 开发测试环境（快速搭建 MQTT 服务）。
  * 小型物联网项目（如家庭智能家居、小型传感器网络）。
  * 边缘设备（如树莓派、工业网关）本地部署。

  


## MQTT 可视化客户端（MQTTX）的安装与使用

MQTTX 是一个强大的跨平台 MQTT 客户端，支持MQTT 5.0 和 3.x。有桌面版，CLI 版和在线版。支持 Windows、Mac、Linux、Docker。

下载地址： [传送门](https://mqttx.app/zh/downloads)

  


**1、安装后打开客户端**

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9b0b0fc3fd8949a69d7847dbb79d77f7.png#pic_center)

**2、创建连接**

EMQX提供了一个免费开源的服务器可供测试，地址：mqtt://broker.emqx.io:1883，也可以使用自建部署的 MQTT 服务进行测试。

填入服务器信息，可选择MQTT版本，MQTT 5.0 支持遗嘱消息，填写完成后点击右上角的连接。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f8ae49e02331490c9708a8e11c90a4b3.png#pic_center)

**3、添加订阅**

填写 topic，可使用通配符，如：`testtopic/#` 表示多层匹配，可匹配如 `testtopic/1/test` 和 `testtopic/1`

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ffbbf5d585a940e0b0b9e36144af9615.png#pic_center)

**4、查看订阅的消息**

如果想自己发送消息，则在消息框上方填写对应的 topic，输入相应的消息发送即可。

通过订阅此 topic，也能接收到自己发送的消息。如图：

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0c3f37df7de6429aa78faa4e6443280c.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/302b8b6646df4172a1ba810c094f1cd7.png#pic_center)

![在这里插入图片描述](https://csdnimg.cn/release/blogv2/dist/pc/img/vip-limited-close-newWhite.png)
