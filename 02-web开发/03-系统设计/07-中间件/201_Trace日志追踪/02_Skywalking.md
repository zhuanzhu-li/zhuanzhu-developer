# Skywalking
## 概述

Apache SkyWalking 是一个开源的分布式追踪和应用性能监控（APM）平台，旨在帮助开发者和运维人员理解和优化复杂的微服务架构。它提供了全面的服务监控能力，包括但不限于分布式追踪、性能指标收集、日志分析以及服务拓扑图展示等功能。SkyWalking 最早由中国开发者吴晟发起并捐赠给 Apache 基金会，现已成为 Apache 的顶级项目。

### 核心功能

* **分布式追踪**：能够跟踪跨多个服务的请求链路，帮助用户理解请求在不同服务之间的流转情况，识别性能瓶颈、异常调用和依赖关系。
* **指标监控**：提供丰富的指标监控功能，包括 CPU 使用率、内存使用率、响应时间、吞吐量、错误率等。
* **服务拓扑图**：自动生成服务之间的调用关系图，展示服务间的依赖关系和调用链路。
* **日志关联**：可以将日志与追踪数据关联起来，帮助用户在出现问题时快速定位相关的日志记录。
* **告警和通知**：提供基于规则的告警功能，支持多种通知渠道，如电子邮件、Slack、PagerDuty、Webhook 等。

### 架构概述

SkyWalking 的架构主要由以下几个组件组成：

* **Agent（探针）**：负责收集应用的性能数据，包括追踪信息、指标和日志。以字节码注入的方式工作，能够在不修改代码的情况下自动收集数据。
* **Collector（收集器）**：负责接收来自 Agent 的数据，并进行聚合、存储和分析。支持高可用性和水平扩展，可以处理大规模集群中的海量数据。
* **Storage（存储）**：支持多种存储后端，包括 Elasticsearch、H2、MySQL、TiDB、InfluxDB 等。
* **UI（用户界面）**：提供了直观的 Web 界面，用户可以通过浏览器访问和操作，展示各种监控数据、追踪链路和服务拓扑图。
* **Alarm（告警）**：负责根据预设的规则生成告警，并通过指定的通知渠道发送告警信息。

### 使用场景

SkyWalking 广泛应用于以下领域：

* **微服务架构**：监控和追踪多个服务之间的调用链路，识别性能瓶颈和依赖关系。
* **容器化应用**：结合 Kubernetes 和 Docker，监控容器化应用的性能，提供详细的追踪和诊断信息。
* **云原生环境**：支持云原生架构，能够与 Prometheus、Grafana 等工具集成，形成完整的可观测性解决方案。

### 优势

* **全面的可观测性**：提供追踪、指标、日志和告警等全方位的可观测性功能。
* **易于集成**：支持多种编程语言和框架，用户可以轻松将其集成到现有的应用中，无需大量修改代码。
* **高性能和可扩展性**：具有高效的采集和处理能力，能够处理大规模集群中的海量数据，支持水平扩展。
* **活跃的社区支持**：拥有一个活跃的开源社区，用户可以获得大量的文档、教程和技术支持。
* **云原生兼容**：完全支持云原生架构，特别是 Kubernetes 和微服务架构，能够与主流的云原生工具无缝集成。
## Spring Boot接入Skywalking
在 Spring Boot 项目中接入 SkyWalking，可以通过以下步骤完成：

### 1. 安装 SkyWalking

首先，需要下载并安装 SkyWalking。可以从 SkyWalking 的官方 GitHub 仓库下载最新版本。解压后，进入 `bin` 目录，运行 `startup.bat`（Windows）或 `startup.sh`（Linux）启动 SkyWalking 服务。

### 2. 配置 SkyWalking Agent

从 SkyWalking 8.x 版本开始，Agent 需要单独下载。下载完成后，解压至指定目录，例如 `D:/skywalking-agent`。

### 3. 配置 Agent 参数

编辑 `agent.config` 文件，配置以下参数：

* `collector.backend_service`: 指定 SkyWalking OAP 服务器的地址，默认为 `127.0.0.1:11800`。
* `agent.service_name`: 设置服务名称，以便在 SkyWalking Web 界面上区分不同的应用。
* `agent.trace_sampling`: 配置采样率，默认为 1，表示采集所有请求；可以根据需要调整为更小的值以减少性能开销。

### 4. 配置 Spring Boot 项目

#### 添加依赖

在 `pom.xml` 文件中添加 SkyWalking 的依赖库：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-apm-skywalking</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

#### 配置文件

在 `application.yml` 或 `application.properties` 文件中进行必要的配置：

```yaml
spring:
  cloud:
    apm:
      skywalking:
        collector-backend-service: 127.0.0.1:11800
        service-name: my-springboot-app
```

### 5. 启动 Spring Boot 项目

#### IDEA 部署探针

修改启动类的 VM options（虚拟机选项）配置，添加以下 JVM 参数：

```
-javaagent:D:/skywalking-agent/skywalking-agent.jar
-Dskywalking.agent.service_name=my-springboot-app
-Dskywalking.collector.backend_service=127.0.0.1:11800
```

#### Java 命令行启动

使用以下命令启动 Spring Boot 应用：

```bash
java -javaagent:D:/skywalking-agent/skywalking-agent.jar -Dskywalking.agent.service_name=my-springboot-app -Dskywalking.collector.backend_service=127.0.0.1:11800 -jar my-springboot-app.jar
```

#### 编写 sh 脚本启动（Linux 环境）

创建一个 sh 脚本文件，内容如下：

```bash
#!/bin/bash

# 设置 SkyWalking Agent 的路径
AGENT_PATH="/path/to/skywalking-agent"

# 设置 Java 应用的 JAR 文件路径
JAR_PATH="/path/to/my-springboot-app.jar"

# 设置 SkyWalking 服务名称和 Collector 后端服务地址
SERVICE_NAME="my-springboot-app"
COLLECTOR_BACKEND_SERVICE="127.0.0.1:11800"

# 构造 Java Agent 参数
JAVA_AGENT="-javaagent:$AGENT_PATH/skywalking-agent.jar -Dskywalking.agent.service_name=$SERVICE_NAME -Dskywalking.collector.backend_service=$COLLECTOR_BACKEND_SERVICE"

# 启动 Java 应用
java $JAVA_AGENT -jar $JAR_PATH
```

### 6. 验证集成

启动 SkyWalking 服务和 Spring Boot 项目后，访问 SkyWalking 的 Web 界面（默认地址为 `http://localhost:8080/`），查看是否能够看到应用的链路跟踪数据。

通过以上步骤，即可完成 Spring Boot 项目与 SkyWalking 的集成，实现应用的链路跟踪和性能监控。

## 中间件支撑
### Kafka

* **插件名称**：kafka-plugin
* **配置方法**：使用 Spring-Kafka 进行 Kafka 的操作，通过配置 `@SpringBootApplication` 注解启动 Spring Boot 应用，并在 IDEA 的「Run/Debug Configurations」配置使用 SkyWalking Agent。

### RabbitMQ

* **插件名称**：rabbitmq-5.x-plugin
* **配置方法**：使用 Spring-AMQP 进行 RabbitMQ 的操作，通过配置 `@SpringBootApplication` 注解启动 Spring Boot 应用，并在 IDEA 的「Run/Debug Configurations」配置使用 SkyWalking Agent。

### RocketMQ

* **插件名称**：rocketMQ-4.x-plugin
* **配置方法**：使用 `ons-client` 库进行 RocketMQ 消息的发送和消费，可以通过修改 SkyWalking `rocketMQ-4.x-plugin` 插件的源码，支持 `ons-client` 库的链路追踪。或者直接使用 SkyWalking `rocketMQ-4.x-plugin` 插件进行链路追踪。

### ActiveMQ

* **插件名称**：activemq-plugin
* **配置方法**：使用 ActiveMQ 的客户端库进行消息的发送和消费，通过配置 SkyWalking Agent 的 `agent.config` 文件，设置 `plugin.activemq.enabled=true` 来启用 ActiveMQ 插件。

### Pulsar

* **插件名称**：pulsar-plugin
* **配置方法**：使用 Pulsar 的客户端库进行消息的发送和消费，通过配置 SkyWalking Agent 的 `agent.config` 文件，设置 `plugin.pulsar.enabled=true` 来启用 Pulsar 插件。

### Kafka Tracing

* **工具包**：apm-toolkit-kafka
* **配置方法**：添加 `apm-toolkit-kafka` 依赖，并使用 KafkaTemplate 发送消息时，无需做任何改动。如果没有使用 KafkaTemplate 的场景，可以使用注解 `@KafkaPollAndInvoke` 来支持 tracing。

### 异步线程 Tracing

* **工具包**：apm-toolkit-trace
* **配置方法**：添加 `apm-toolkit-trace` 依赖，并使用 `RunnableWrapper` 或注解 `@TraceCrossThread` 来支持异步线程的 tracing。

### 关系型数据库

* **MySQL**：SkyWalking 支持使用 MySQL 作为后端存储，适用于存储和检索性能数据、跟踪数据和日志数据。
* **PostgreSQL**：SkyWalking 也支持 PostgreSQL，可以将性能数据、跟踪数据和日志数据存储在 PostgreSQL 数据库中。
* **TiDB**：TiDB 是一种开源的分布式 SQL 数据库，SkyWalking 支持使用 TiDB 作为后端存储，适用于需要水平扩展和高可用性的场景。
* **Oracle**：SkyWalking 支持 Oracle 数据库，可以将性能数据、跟踪数据和日志数据存储在 Oracle 数据库中。

### 非关系型数据库

* **Elasticsearch**：Elasticsearch 是一个分布式搜索和分析引擎，SkyWalking 使用 Elasticsearch 作为其主要的后端存储，适用于存储和查询大量的监控数据。
* **H2**：H2 是一个嵌入式的 Java 关系型数据库，SkyWalking 支持使用 H2 数据库作为后端存储，通常用于开发和测试环境。

### 其他数据库

* **ClickHouse**：SkyWalking 社区有二开支持了 ClickHouse，可以用于存储和查询监控数据。
* **ShardingSphere**：SkyWalking 支持使用 ShardingSphere 进行分库分表，以解决关系型数据库在数据量较大时出现的瓶颈问题。



