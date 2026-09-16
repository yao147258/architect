# architect

A Claude Code plugin that acts as a software architect: it understands your
requirements and usage scenarios, then produces a system architecture design
document — module breakdown, Mermaid diagrams, technology selection, and an
evolution roadmap.

## Install

```
/plugin marketplace add yao147258/architect
```

Then enable the `architect` plugin. The skill is invoked automatically when
your request matches architecture design, module decomposition, technology
selection, or architecture review scenarios — or explicitly via
`architect:architect`.

## What it does

1. **Understands requirements** — parses your conversation or a requirements
   doc, asks clarifying questions only when the answer would change an
   architecture decision.
2. **Quantifies constraints** — turns vague words ("high concurrency",
   "lots of users") into numbers: QPS, data volume, SLA, consistency
   requirements.
3. **Compares 2-3 architecture approaches** with explicit trade-offs, and
   recommends one with reasoning.
4. **Breaks down modules** following DDD-style bounded contexts, not
   technical layers — each module gets a one-line responsibility, its
   interface, dependencies, and data ownership.
5. **Produces a full design document** with four required Mermaid diagrams
   (context, container, module dependency, sequence-with-failure-path) and
   a 9-section architecture document.

## Scenarios covered

- Designing a brand-new system from scratch
- Adding a module/subsystem to an existing codebase (scans the repo first)
- Reviewing/refactoring an existing architecture
- Comparing technology options (lightweight path, no full document)

## Design principle

Complexity must be justified by a quantified metric. The skill defaults to
the simplest architecture that satisfies your constraints — modular
monolith over microservices, single database over sharding — and only
recommends more complexity when a specific number from your requirements
demands it.

## Local development

```
/plugin marketplace add /path/to/your/local/clone/of/architect
```

## License

MIT

---

# architect（中文说明）

一个扮演软件架构师角色的 Claude Code 插件：理解你的需求与使用场景，产出
包含模块划分、Mermaid 架构图、技术选型与演进路线的系统架构设计文档。

## 安装

```
/plugin marketplace add yao147258/architect
```

安装后启用 `architect` 插件。当你的请求匹配架构设计、模块划分、技术选型、
架构评审等场景时会自动触发，也可以显式调用 `architect:architect`。

## 能做什么

1. **理解需求**——解析对话描述或需求文档，只在缺失信息会影响架构决策时才
   反问澄清问题。
2. **量化约束**——把"高并发""用户量很大"这类模糊描述换算成具体数字：
   QPS、数据量、SLA、一致性要求。
3. **对比 2-3 个架构方案**，给出明确的权衡分析和带理由的推荐。
4. **划分模块**——按业务能力/限界上下文而非技术分层拆分，每个模块给出
   一句话职责、对外接口、依赖关系、数据所有权。
5. **产出完整设计文档**——含四张必画 Mermaid 图（上下文图、容器图、模块
   依赖图、含异常路径的时序图）和 9 章架构文档。

## 覆盖场景

- 全新系统从零设计
- 已有系统新增模块/子系统（先扫描现有代码库）
- 现有架构评审/重构建议
- 技术选型对比决策（轻量路径，不产出完整文档）

## 设计原则

复杂度必须被量化指标证明。默认从满足约束的最简架构起步——模块化单体优先
于微服务、单库优先于分库分表——只有当需求里的具体数字要求更复杂的方案时，
才会推荐引入更高的复杂度。

## 本地开发调试

```
/plugin marketplace add /path/to/your/local/clone/of/architect
```

## 许可证

MIT
