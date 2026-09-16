# 架构图规范（Mermaid）

## 必画四张图

1. **系统上下文图** —— 本系统 + 用户角色 + 外部系统，看清系统边界在哪里
2. **容器图** —— 进程/服务/存储/中间件，标注技术栈和通信协议
3. **模块依赖图** —— 逻辑模块 + 依赖方向，用来验证模块划分是否无环
4. **关键流程时序图** —— 最核心的 1-3 条链路，其中**必须有一条是失败/异常
   路径**（如下单超时、支付回调丢失）——这是最能暴露架构缺陷的图，不能只画
   Happy Path

按需补充：ER 图、状态机图、部署拓扑图。

## 通用绘制规范

- 节点必须标注技术栈，不写光秃秃的"订单服务"，要写"订单服务（Spring Boot）"
- 边必须标注协议 + 同步/异步：实线代表同步调用，虚线代表异步
- 存储类节点用 `[()]`（圆柱形），外部系统用 `{{}}`（六边形），视觉上一眼区分
- **单张图不超过 15 个节点**，超出就按子系统分层拆成多张图——看不懂的图
  等于没画

## 模板 1：系统上下文图

```mermaid
flowchart TB
  U([终端用户]) --> SYS["本系统<br/>电商交易平台"]
  ADMIN([运营人员]) --> SYS
  SYS -->|"支付回调"| PAY{{"支付宝/微信支付<br/>外部系统"}}
  SYS -->|"短信/推送"| NOTIFY{{"短信网关<br/>外部系统"}}
  SYS -->|"物流查询"| LOGISTICS{{"物流服务商<br/>外部系统"}}
```

## 模板 2：容器图

```mermaid
flowchart TB
  U([用户]) --> GW["API 网关<br/>Nginx + Spring Cloud Gateway"]
  GW --> ORD["订单模块<br/>Spring Boot"]
  GW --> INV["库存模块<br/>Spring Boot"]
  ORD -->|"同步 HTTP"| INV
  ORD -.->|"异步 Kafka"| SETL["结算模块<br/>Spring Boot"]
  ORD --> DB[("订单库<br/>MySQL")]
  ORD --> C[("缓存<br/>Redis")]
  PAY{{"支付宝<br/>外部系统"}} -.回调.-> ORD
```

## 模板 3：模块依赖图

```mermaid
flowchart LR
  ORD["订单模块"] --> INV["库存模块"]
  ORD --> USER["用户模块"]
  ORD -.事件.-> SETL["结算模块"]
  SETL -.事件.-> NOTIFY["通知模块"]
  INV --> USER
```

绘制后检查：图中不能出现环（如 A→B→C→A），出现环即说明模块边界需要重新
划分。

## 模板 4：关键流程时序图（含异常路径）

正常路径：

```mermaid
sequenceDiagram
    participant U as 用户
    participant ORD as 订单模块
    participant INV as 库存模块
    participant PAY as 支付宝(外部)
    U->>ORD: 提交订单
    ORD->>INV: 扣减库存(同步)
    INV-->>ORD: 扣减成功
    ORD->>PAY: 发起支付
    PAY-->>ORD: 支付成功回调
    ORD-->>U: 订单完成
```

异常路径（必须画）：

```mermaid
sequenceDiagram
    participant U as 用户
    participant ORD as 订单模块
    participant INV as 库存模块
    participant PAY as 支付宝(外部)
    U->>ORD: 提交订单
    ORD->>INV: 扣减库存(同步)
    INV-->>ORD: 扣减成功
    ORD->>PAY: 发起支付
    Note over ORD,PAY: 支付回调超时未到达
    ORD->>PAY: 主动查询支付状态(定时补偿)
    alt 支付实际成功
        PAY-->>ORD: 返回已支付
        ORD-->>U: 订单完成
    else 支付实际失败/超时
        ORD->>INV: 释放已扣减库存
        ORD-->>U: 订单取消，请重新下单
    end
```

## 模板 5：ER 图（按需）

```mermaid
erDiagram
    ORDER ||--|{ ORDER_ITEM : contains
    ORDER {
        bigint id PK
        bigint user_id FK
        varchar status
        datetime created_at
    }
    ORDER_ITEM {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        int quantity
    }
    USER ||--o{ ORDER : places
```

## 常见错误示范（反例）

```mermaid
flowchart TB
  A[订单服务] --> B[库存服务]
  B --> C[用户服务]
```

错在哪：节点没有技术栈标注、边没有协议和同步/异步标注、看不出这是内部
调用还是跨系统调用。对照模板 2 重写。
