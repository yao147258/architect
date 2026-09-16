# 技术选型判据矩阵

原则：不给"哪个最好"，给"什么条件下选哪个"。每类先列选项，再给判据表。

## 关系型数据库

选项：MySQL、PostgreSQL、分布式 NewSQL（TiDB/CockroachDB）

| 条件 | 选择 |
|---|---|
| 团队最熟悉、生态最成熟、简单读写为主 | MySQL |
| 需要复杂查询（窗口函数/JSON/地理空间/全文检索）、数据完整性约束严格 | PostgreSQL |
| 单机容量/写入瓶颈已被量化指标证实（而非预测），且团队能承受运维复杂度 | 分库分表（ShardingSphere 等）或 NewSQL |
| 需要金融级强一致跨地域部署 | TiDB/CockroachDB 等分布式 NewSQL |

## NoSQL

选项：MongoDB（文档型）、Cassandra/HBase（宽列，海量写）、DynamoDB/Redis（KV）

| 条件 | 选择 |
|---|---|
| 数据结构多变、嵌套文档、快速迭代的业务模型 | MongoDB |
| 海量写入、跨机房、最终一致可接受（如日志、埋点、IM 消息） | Cassandra/HBase |
| 简单键值访问、超高读写吞吐、云托管优先 | DynamoDB（云）或自建 Redis |
| 业务本质是强关系（多表 join、事务）却选了 NoSQL | 反例——先看关系型能否满足 |

## 时序数据库

选项：InfluxDB、TimescaleDB（PostgreSQL 插件）、Prometheus（监控专用）

| 条件 | 选择 |
|---|---|
| 监控指标采集与告警 | Prometheus |
| 通用时序业务数据（IoT 传感器、金融行情） 且团队已用 PostgreSQL | TimescaleDB |
| 独立时序场景、高写入吞吐、原生时序查询语言 | InfluxDB |

## 缓存

选项：Redis（分布式）、Caffeine（本地/进程内）、Memcached

| 条件 | 选择 |
|---|---|
| 多实例间需要共享缓存状态 | Redis |
| 单机极致低延迟、数据量小、可接受多实例不一致 | Caffeine（本地缓存） |
| 只需简单 KV 缓存、无需持久化和丰富数据结构 | Memcached（多为历史系统延续，新系统优先 Redis） |
| 高并发下拿不准，先加本地缓存还是分布式缓存 | 先本地兜底热点 key，再分布式缓存兜底一致性 |

## 消息队列

选项：Kafka、RocketMQ、RabbitMQ、Pulsar

| 条件 | 选择 |
|---|---|
| 高吞吐日志/事件流、需要长期保留和多消费者重复消费 | Kafka |
| 电商/金融场景，需要事务消息、延迟消息、消息轨迹（国内生态友好） | RocketMQ |
| 复杂路由规则（topic/queue/exchange）、中小吞吐、须成熟的 AMQP 协议支持 | RabbitMQ |
| 多租户、存算分离、云原生场景 | Pulsar |
| 只是需要简单的异步解耦，QPS 不高 | 先考虑数据库表+轮询或进程内事件，不一定要上 MQ |

## API 网关

选项：Nginx（反向代理）、Kong、Spring Cloud Gateway、APISIX

| 条件 | 选择 |
|---|---|
| 只需要基础反向代理/负载均衡，无需插件生态 | Nginx |
| Java/Spring 技术栈、需要与服务发现/配置中心深度集成 | Spring Cloud Gateway |
| 多语言技术栈、需要丰富插件生态（限流/鉴权/日志） | Kong 或 APISIX |
| 云原生、追求高性能和动态配置 | APISIX |

## 注册与配置中心

选项：Nacos、Consul、Zookeeper、Eureka

| 条件 | 选择 |
|---|---|
| 需要注册发现 + 配置管理二合一，国内生态 | Nacos |
| 多数据中心、需要健康检查和 KV 存储 | Consul |
| 已有 Kafka/Hadoop 生态依赖 ZK，或需要强一致协调服务 | Zookeeper |
| 遗留 Spring Cloud Netflix 系统 | Eureka（新系统不建议，社区维护趋缓） |

## 搜索引擎

选项：Elasticsearch、Meilisearch、数据库自带全文索引

| 条件 | 选择 |
|---|---|
| 复杂聚合分析、日志检索、大数据量全文搜索 | Elasticsearch |
| 简单站内搜索、轻量部署、追求开箱即用体验 | Meilisearch |
| 搜索需求简单（模糊匹配几个字段）、数据量不大 | 数据库自带全文索引（避免引入新中间件） |

## 可观测性

选项：Prometheus+Grafana（指标）、ELK/Loki（日志）、SkyWalking/Jaeger（链路追踪）、OpenTelemetry（统一采集标准）

| 条件 | 选择 |
|---|---|
| 单体或模块化单体，运维资源有限 | 结构化日志 + 基础告警即可，暂不需要链路追踪 |
| 微服务架构 | 必须有链路追踪（SkyWalking/Jaeger），否则故障定位会灾难化 |
| 多套监控工具希望统一采集标准，避免厂商锁定 | OpenTelemetry 作为采集层，后端可换 |

## 任务调度

选项：XXL-JOB、ElasticJob、Quartz、云函数定时触发

| 条件 | 选择 |
|---|---|
| 单机定时任务，无需分布式协调 | Quartz（内嵌） |
| 需要可视化管理、失败重试、分片执行，Java 技术栈 | XXL-JOB |
| 已用当当 ElasticJob 生态或需要弹性分片 | ElasticJob |
| Serverless 架构、任务轻量 | 云函数定时触发器 |

## 前端框架与状态管理

选项：React（Redux/Zustand）、Vue（Pinia）、Svelte

| 条件 | 选择 |
|---|---|
| 团队已有 React 经验、生态要求高、大型复杂应用 | React + Zustand（简单场景）或 Redux（强规范/时间旅行调试需求） |
| 团队偏好渐进式、模板语法友好、中大型应用 | Vue + Pinia |
| 追求极致包体积和运行时性能、团队愿意接受较小生态 | Svelte/SvelteKit |
| 全局状态很少、多为局部/服务端状态 | 优先 React Query/SWR 这类服务端状态库，而非引入 Redux |

## 部署形态

选项：物理机/云主机、容器+Kubernetes、Serverless

| 条件 | 选择 |
|---|---|
| 团队规模小、服务数量少（1-5个）、无需精细弹性伸缩 | 云主机 + Docker Compose |
| 服务数量多、需要精细化资源调度和自愈能力、团队有运维能力 | Kubernetes |
| 流量波动剧烈、团队小、不想管基础设施 | Serverless（FaaS） |
| 团队没有 K8s 运维经验却选择自建 K8s 集群 | 反例——先用托管 K8s（云厂商）或先不用 K8s |
