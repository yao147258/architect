# architect Skill 设计文档

- 日期：2026-09-16
- 状态：待用户审核
- 目标仓库：`github.com/yao14728/architect`（同时作为插件仓库与自带 marketplace）

## 1. 背景与目标

开发一个「架构师」Claude Code 技能（skill），使其具备：

- 精通多种编程语言、中间件、架构风格的判断力
- 能从用户对话或需求文档中拆解出真实的架构决策点
- 能为对应场景设计出合适的系统架构，并产出结构化的架构设计文档

核心能力范围：

1. 理解用户需求与使用场景（含模糊需求的澄清）
2. 输出架构设计文档，重点是系统模块划分与架构图

该技能最终以 **Claude Code 插件** 的形式发布到 GitHub，供他人通过
`/plugin marketplace add yao14728/architect` 安装使用。

## 2. 使用场景（scope）

覆盖全栈（后端 / 前端 / 全栈 / 移动端），四类场景：

| 场景 | 说明 |
|---|---|
| 全新系统从 0 设计 | 给定需求描述或 PRD，输出完整架构设计 |
| 已有系统新增模块/子系统 | 先扫描现有代码库的分层、技术栈、命名约定，再设计兼容的新模块 |
| 现有架构评审/重构建议 | 分析现状问题（耦合、性能瓶颈、扩展性），给出演进路线 |
| 技术选型对比决策 | 单独回答"选型 A 还是 B"，输出对比表和推荐，不产出完整 9 章文档 |

## 3. 不做什么（YAGNI）

- 不做多 agent 并行编排（需求强耦合，拆分收益低于同步成本）
- 不自动生成代码脚手架（这是实现阶段的事，不在本技能范围）
- 不维护每种语言/框架的详尽 API 文档（交给 context7 等专用工具）
- 轻量选型问题不strong-force 走完整 9 章文档流程

## 4. 整体方案：分层知识库（SKILL.md 精简 + references 按需加载）

原因：技能要覆盖全栈 × 4 类场景 × 多种中间件，知识总量大；若全部内联进
SKILL.md，技能被匹配时会整篇加载进上下文，代价高且模型难以在长文档中
保持精度。故将 SKILL.md 收窄为"流程 + 判断逻辑 + 硬性门禁"，具体领域
知识拆到 `references/*.md`，按阶段需要才读取。

## 5. 核心工作流

### 5.1 场景路由（第一步）

| 场景 | 起点动作 |
|---|---|
| 全新系统从 0 设计 | 直接进入阶段 1 |
| 已有系统新增模块 | 先用 Glob/Grep/Read 扫描代码库：分层结构、技术栈、命名/包约定、现有模块边界 → 再进阶段 1，新设计必须与现状兼容 |
| 现有架构评审/重构 | 先扫描代码库 + 询问痛点（哪里疼、何时疼、持续多久）→ 跳过方案选型，走"现状 → 问题 → 演进路线" |
| 技术选型对比 | 轻量路径：只走"约束量化 → 对比矩阵 → 推荐"，不产出完整 9 章文档 |

### 5.2 五阶段主流程

**阶段 1 · 需求理解**

- 解析输入（对话描述 / PRD 文档 / 现有代码库）
- 提取：业务目标、核心用例、用户角色、关键流程
- 缺失信息从 `references/clarifying-questions.md` 中挑选**真正影响架构决策**
  的问题，使用 AskUserQuestion 一次问 3-4 个，提供推荐项
- 产出：一段「需求理解确认」——理解的目标 / 核心用例清单 / 已确认约束 / 假设列表

**阶段 2 · 约束与质量属性量化**

将模糊描述转成数字：DAU、峰值 QPS、数据量与增速、响应时间要求、可用性目标、
一致性要求（强/最终）、团队规模与技术栈熟悉度、工期、预算、合规要求。
没有数字时给出数量级估算并标注为"假设"。这些数字是阶段 3-4 一切决策的依据。

**阶段 3 · 架构方案选型**

给出 2-3 个候选方案，每个方案写明：适用前提、优点、代价（运维复杂度/团队要求/
成本）、翻车场景。给出带理由的推荐。

**阶段 4 · 模块划分与详细设计**（见第 6 节）

**阶段 5 · 出文档**（见第 7 节模板）

按模板写单份 Markdown，落到 `docs/arch/YYYY-MM-DD-<系统名>-架构设计.md`
（用户项目目录下，非本插件仓库）。

### 5.3 三道硬性门禁

1. **澄清未确认 → 不许进入方案选型。** 不能用"假设一下"跳过阶段 1-2。
2. **候选方案少于 2 个 → 不许进入模块划分。** 防止直接给单一结论。
3. **复杂度必须被量化指标证明。** 默认从"能满足约束的最简架构"起步
   （模块化单体优先于微服务、单库优先于分库分表、进程内调用优先于引入 MQ）。
   每引入一个中间件或拆出一层，文档中必须写清是哪个量化指标驱动的；
   写不出来就砍掉。

## 6. 模块划分规则

先划逻辑模块，再定部署边界——不可合并成一步。

**划分依据优先级**：

1. 业务能力 / 限界上下文（首选）
2. 变化率（高频变化的与稳定的分离）
3. 团队边界（康威定律）
4. 技术性关注点（网关/认证/通知，最后考虑）

**禁止的反模式**（写入 skill 明确禁止）：

- 按技术分层拆服务（controller 服务 / service 服务 / dao 服务）
- 按数据库表一对一拆模块
- 纯 CRUD 微服务

**每个模块必须写满四项**：

| 项 | 要求 |
|---|---|
| 职责 | 一句话说清；说不清即需重划 |
| 对外接口 | 提供的能力（接口名/事件名），不暴露内部结构 |
| 依赖 | 依赖谁、被谁依赖、依赖方式（调用/事件/无） |
| 数据所有权 | 拥有哪些数据；一张表只能有一个模块写 |

**依赖方向规则**：单向、无环、跨模块只走接口或事件。

**四个自检测试**：一句话测试、变更测试（典型需求变更动 ≥3 模块则边界错误）、
独立测试、所有权测试（是否有两个模块写同一张表）。

**前端/全栈场景**：模块维度换成路由与页面、业务组件、基础组件、状态管理边界、
API 服务层、前后端契约；需额外画前后端职责分界图，明确校验和权限逻辑归属
（前端校验只是体验，不是安全边界）。

## 7. 架构图规范

**必画四张**：

1. 系统上下文图——系统 + 用户角色 + 外部系统
2. 容器图——进程/服务/存储/中间件，标注技术栈和通信协议
3. 模块依赖图——逻辑模块 + 依赖方向（验证无环）
4. 关键流程时序图——1-3 条核心链路，至少一条为失败/异常路径

按需补充：ER 图、状态机图、部署拓扑图。

**Mermaid 绘制规范**：

- 节点标注技术栈，不写光秃秃的服务名
- 边标注协议 + 同步（实线）/异步（虚线）
- 存储用 `[()]`，外部系统用 `{{}}`
- 单张图不超过 15 个节点，超出则分层拆图

## 8. 架构设计文档模板（9 章）

产出到用户项目 `docs/arch/YYYY-MM-DD-<系统名>-架构设计.md`：

| 章节 | 硬性要求 |
|---|---|
| 1 需求与场景理解 | 含"本文档基于的假设"列表 |
| 2 质量属性与约束 | 全部量化；无数字则给数量级估算并标注（假设） |
| 3 架构方案对比 | ≥2 候选 + 权衡表 + 推荐理由 + 被否决方案的否决原因 |
| 4 系统模块划分 | 每模块四项齐全 + 逻辑模块到部署单元的映射表 |
| 5 架构图 | 必画四张，符合第 7 节规范 |
| 6 技术选型 | 选了什么/为什么/放弃了什么替代方案/引入的运维成本 |
| 7 数据模型与关键流程 | 核心实体 ER + 1-3 条关键链路说明 |
| 8 非功能设计 | 性能/可用性/安全/可观测 |
| 9 风险与演进 | 已知风险 + 应对 + 分阶段演进路线（含可观测的触发信号，而非"未来可以微服务化"式空话） |

## 9. 知识库内容纲要（references/）

| 文件 | 内容 | 何时加载 |
|---|---|---|
| `clarifying-questions.md` | 分领域澄清清单：通用/后端/前端/数据/移动端，每题标注影响哪个架构决策 | 阶段 1 |
| `architecture-styles.md` | 分层单体、模块化单体、微服务、SOA、事件驱动、CQRS+ES、Serverless、微内核插件、中台。统一五段式：定义/适用前提（量化门槛）/优点/真实代价/何时不用+翻车案例 | 阶段 3 |
| `tech-selection.md` | 关系库/NoSQL/时序库、缓存、MQ（Kafka/RocketMQ/RabbitMQ/Pulsar）、网关、注册配置中心、搜索、可观测、任务调度、前端框架与状态管理、部署形态的选型判据 | 阶段 3-4 |
| `module-decomposition.md` | 第 6 节方法论全文 + 电商/SaaS/内容/IoT 等典型业务域参考划分 | 阶段 4 |
| `diagram-patterns.md` | 五类 Mermaid 图模板 + 绘制规范 + 常见错误示范 | 阶段 5 |
| `document-template.md` | 9 章完整模板，含每章填写要求与示例片段 | 阶段 5 |

## 10. 插件化与发布

### 10.1 仓库结构

```
architect/                                 (GitHub: yao14728/architect)
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json                   # source: "./"，单仓库自带市场
├── skills/
│   └── architect/
│       ├── SKILL.md                       # 主流程，目标 <300 行
│       └── references/
│           ├── clarifying-questions.md
│           ├── architecture-styles.md
│           ├── tech-selection.md
│           ├── module-decomposition.md
│           ├── diagram-patterns.md
│           └── document-template.md
├── docs/superpowers/specs/                # 本设计文档
├── README.md
├── LICENSE                                # MIT
└── .gitignore
```

### 10.2 plugin.json

```json
{
  "name": "architect",
  "version": "1.0.0",
  "description": "Architect skill: understands requirements and scenarios, then designs system architecture with module breakdown and diagrams",
  "author": {
    "name": "yaofengqiao",
    "email": "1947846708@qq.com"
  },
  "homepage": "https://github.com/yao14728/architect",
  "repository": {
    "type": "git",
    "url": "https://github.com/yao14728/architect"
  },
  "license": "MIT",
  "keywords": ["architecture", "system-design", "skills", "架构设计", "系统架构"],
  "skills": "./skills"
}
```

### 10.3 marketplace.json（单仓库自带市场）

```json
{
  "name": "architect-marketplace",
  "owner": {
    "name": "yaofengqiao",
    "email": "1947846708@qq.com"
  },
  "description": "Marketplace for the architect skill plugin",
  "plugins": [
    {
      "name": "architect",
      "description": "Architect skill: understands requirements and scenarios, then designs system architecture with module breakdown and diagrams",
      "version": "1.0.0",
      "source": "./",
      "author": {
        "name": "yaofengqiao",
        "email": "1947846708@qq.com"
      }
    }
  ]
}
```

### 10.4 SKILL.md frontmatter

```yaml
---
name: architect
description: Use when designing system architecture from requirements — new systems from scratch, adding modules to existing codebases, reviewing/refactoring existing architecture, or comparing technology options. Triggers include 需求拆解、系统设计、模块划分、架构图、技术选型、微服务拆分、架构评审.
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, WebSearch, WebFetch, AskUserQuestion, mcp__context7__*
---
```

安装后引用名为 `architect:architect`。

### 10.5 路径引用规则

SKILL.md 内引用 references 文件必须使用 `${CLAUDE_PLUGIN_ROOT}` 绝对路径变量，
禁止相对路径（相对路径在插件安装后的实际目录下会失效）：

```
${CLAUDE_PLUGIN_ROOT}/skills/architect/references/architecture-styles.md
```

### 10.6 语言策略

- `plugin.json` / `marketplace.json` 的 description：英文
- SKILL.md frontmatter description：英文为主 + 嵌入中文关键词（保证中文需求描述下也能被检索匹配）
- SKILL.md 与 references 正文：中文为主，架构术语保留英文原词（Bounded Context、CQRS、Saga 等）
- README.md：英文在前、中文在后
- 产出的架构设计文档：跟随用户对话使用的语言

### 10.7 安装与调试命令

- 用户安装：`/plugin marketplace add yao14728/architect`
- 本地开发调试：`/plugin marketplace add C:\Users\Administrator\architect`（本地路径）
- 发布前校验：`claude plugin validate`

## 11. 验收标准

- [ ] `claude plugin validate` 通过
- [ ] 本地安装后 `architect:architect` 技能可被正确触发（中文需求描述能命中）
- [ ] 对一个模糊需求（如"做个商城"）能先反问关键问题，而非直接出文档
- [ ] 对一个量化清晰的需求，能产出符合 9 章模板的完整 Markdown 文档
- [ ] 文档中的架构图均为合法 Mermaid 语法，可正常渲染
- [ ] 触发"已有系统新增模块"场景时，能先执行代码库扫描
- [ ] 三道硬性门禁在测试对话中确实生效（不会被绕过）
