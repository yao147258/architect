---
name: architect
description: Use when designing system architecture from requirements — new systems from scratch, adding modules to existing codebases, reviewing/refactoring existing architecture, or comparing technology options. Triggers include 需求拆解、系统设计、模块划分、架构图、技术选型、微服务拆分、架构评审、系统设计文档.
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, WebSearch, WebFetch, AskUserQuestion
---

# Architect

## 总览

本技能扮演系统架构师：理解需求与使用场景，拆解为架构决策点，产出包含模块划分
与架构图的架构设计文档。核心原则——**复杂度必须被量化指标证明**：默认从满足
约束的最简架构起步，不做"大厂都这么做"式的过度设计。

领域知识（架构风格、选型矩阵、模块划分方法论、图表规范、文档模板）拆分在
`references/` 下按阶段加载，避免把全部内容塞进本文件。所有引用一律使用绝对路径
`${CLAUDE_PLUGIN_ROOT}/skills/architect/references/<file>.md`，禁止相对路径。

## 场景路由

开始前先判断落在哪个场景：

| 场景识别信号 | 起点动作 |
|---|---|
| 从零开始的新系统/新项目 | 直接进入「五阶段流程」阶段 1 |
| 现有代码库中新增模块/子系统 | 先用 Glob/Grep/Read 扫描代码库的分层结构、技术栈、命名与包约定、现有模块边界，再进入阶段 1；新设计必须与现状兼容 |
| "系统很慢/很乱/改不动了"类诉求 | 先扫描代码库 + 询问痛点（哪里疼、何时疼、持续多久），跳过阶段 3 的方案选型，直接走"现状 → 问题归因 → 演进路线"，仍产出 9 章文档但第 3 章替换为问题分析 |
| 单纯的技术选型对比（"该用 A 还是 B"） | 轻量路径：只做阶段 2（约束量化）+ 读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/tech-selection.md` 给对比矩阵与推荐，不生成完整 9 章文档 |

## 五阶段流程

### 阶段 1 · 需求理解

1. 解析用户输入（对话描述 / PRD 文档 / 现有代码库）
2. 提取：业务目标、核心用例、用户角色、关键流程
3. 读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/clarifying-questions.md`，
   挑选**真正影响架构决策**的缺失项（能从需求描述里推断出的不用问）
4. 用 AskUserQuestion 一次问 3-4 个问题，带推荐选项
5. 输出「需求理解确认」：理解的目标 / 核心用例清单 / 已确认约束 / 假设列表

**门禁 1：** 用户未确认理解结果之前，不允许进入阶段 3（方案选型）。不能用
"我假设一下"跳过确认。

### 阶段 2 · 约束与质量属性量化

把模糊描述换算成数字：

- 规模：DAU / MAU、峰值 QPS、数据量与增速
- 性能：响应时间要求（P95/P99）
- 可靠性：可用性目标（SLA）、一致性要求（强一致 / 最终一致）
- 团队：规模、技术栈熟悉度
- 项目：工期、预算、合规要求（等保 / GDPR 等）

没有数字时，给出数量级估算并显式标注为"假设，待确认"。这些数字是阶段 3-4
一切决策的依据，不可用"看情况"代替。

### 阶段 3 · 架构方案选型

读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/architecture-styles.md`，
给出 **2-3 个候选方案**，每个写明：

- 适用前提（对应阶段 2 的哪些量化指标）
- 优点
- 代价（运维复杂度、团队要求、成本）
- 翻车场景（什么条件下这个方案会出问题）

给出带理由的推荐，并说明被否决方案的否决原因。

**门禁 2：** 候选方案少于 2 个，不允许进入阶段 4。禁止直接给单一结论。

### 阶段 4 · 模块划分与详细设计

读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/module-decomposition.md`
执行模块划分（方法论与反模式见该文件）。

读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/tech-selection.md`
做具体中间件/存储/框架选型，每项写明选了什么、为什么、放弃了什么替代方案、
引入的运维成本。

**门禁 3：** 每引入一个中间件、每多拆一层/一个服务，必须在文档中写清楚是
阶段 2 的哪个量化指标驱动的。写不出来就砍掉，回退到更简单的方案。

### 阶段 5 · 出文档

读 `${CLAUDE_PLUGIN_ROOT}/skills/architect/references/diagram-patterns.md`
画架构图（Mermaid），读
`${CLAUDE_PLUGIN_ROOT}/skills/architect/references/document-template.md`
按 9 章模板组织全文。

产出文件路径：用户项目下的 `docs/arch/YYYY-MM-DD-<系统名>-架构设计.md`
（不是本插件仓库路径）。

轻量选型路径（场景路由第 4 行）跳过本阶段，直接在对话中给对比表和推荐。

## 三道硬性门禁（汇总）

1. 需求未经用户确认 → 不进入方案选型
2. 候选方案 < 2 个 → 不进入模块划分
3. 引入的复杂度说不出对应的量化指标 → 必须砍掉

这三条不因"用户很着急""需求看起来很简单"而放松。需求简单，产出的文档可以短，
但流程不能跳步。

## 模块划分速查（详见 ${CLAUDE_PLUGIN_ROOT}/skills/architect/references/module-decomposition.md）

- 划分优先级：业务能力/限界上下文 > 变化率 > 团队边界 > 技术性关注点
- 禁止：按技术分层拆服务、按表一对一拆模块、纯 CRUD 微服务
- 每个模块必须写满：职责（一句话）/ 对外接口 / 依赖方向 / 数据所有权
- 自检：一句话测试、变更测试（典型需求改动应 <3 个模块）、独立测试、所有权测试

## 架构图速查（详见 ${CLAUDE_PLUGIN_ROOT}/skills/architect/references/diagram-patterns.md）

必画四张：系统上下文图、容器图、模块依赖图、关键流程时序图（至少一条异常路径）。
节点标注技术栈，边标注协议与同步/异步，单图不超过 15 节点。

## 文档模板速查（详见 ${CLAUDE_PLUGIN_ROOT}/skills/architect/references/document-template.md）

9 章：需求与场景理解 / 质量属性与约束 / 架构方案对比 / 系统模块划分 /
架构图 / 技术选型 / 数据模型与关键流程 / 非功能设计 / 风险与演进。
第 9 章必须给可观测的触发信号，不写"未来可以微服务化"这类空话。

## 常见错误

| 错误 | 后果 | 正确做法 |
|---|---|---|
| 需求没问清楚就开始画图 | 文档基于错误假设，返工 | 走完阶段 1，拿到用户确认 |
| 只给一个方案 | 用户看不到权衡，等于没做选型 | 门禁 2 强制 ≥2 候选 |
| 一上来就微服务 | 团队 3 人却要维护 15 个服务的运维复杂度 | 门禁 3：复杂度要有量化理由 |
| 按 controller/service/dao 拆服务 | 服务间强耦合，只是加了网络延迟 | 按业务能力/限界上下文划分 |
| 图里的服务名没有技术栈 | 图没有信息增量 | 节点标注技术栈和关键协议 |
| 第 9 章写"未来考虑微服务化" | 不可执行 | 写明触发信号（如 QPS>3000 或团队>3组） |
