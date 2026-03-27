# MindFlow Specification

> 本文件是 MindFlow 的 **single source of truth**，记录系统的当前设计和约定。
> CLAUDE.md 引用本文件提供结构信息，自身聚焦 AI 操作指令。

**Last updated**: 2026-03-27

---

## 1. What is MindFlow

MindFlow 是一个基于 Obsidian 的 **Human-AI 协作科研知识管理系统**。它将论文阅读、idea 孵化、实验追踪、记忆蒸馏等科研工作流编码为可执行的 Markdown skill，通过 Claude Code 在 vault 内直接执行，实现 Human 和 AI 在同一知识库上的无缝协作。

**设计哲学**：
- **Markdown-native**：所有数据以纯 Markdown 存储，Human 和 AI 共用同一格式
- **Skill-driven**：科研工作流标准化为 skill，自然语言触发，可组合、可复现
- **Memory as first-class**：AI 的经验不是一次性的，通过分层记忆系统（L0→L4）逐步蒸馏为持久知识

## 2. Architecture

### 双层设计

```
┌─────────────────────────────────────────────────┐
│  Layer 2: Orchestrator (optional) 🔮 Planned     │
│  Scheduler · Memory Index · Notifier ·           │
│  Agent Bridge                                    │
├─────────────────────────────────────────────────┤
│  Layer 1: Skill Protocol (core) ✅ Implemented   │
│  skills/*.md · Workbench/ · Templates/           │
│  Zero dependency, any agent can execute          │
├─────────────────────────────────────────────────┤
│  Obsidian Vault (Markdown)                       │
│  Papers/ Topics/ Ideas/ Domain-Map/              │
│  Workbench/ (AI working state)                   │
└─────────────────────────────────────────────────┘
```

**Layer 1（核心）**：纯 Markdown skill + vault 模板 + 协议文档。零依赖，任何支持文件读写的 AI agent 均可执行。当前已实现。

**Layer 2（可选）** 🔮：当需要 AI 完全自主运行时启用。包括：
- `scheduler/` — Cron / 事件驱动的任务调度
- `memory_index/` — 基于向量的记忆检索（从 Markdown 可重建）
- `notifier/` — 推送通知（Telegram / Email 等）
- `agent_bridge/` — 统一 agent 抽象（Claude Code / Codex / Gemini CLI）

**接口契约**：Layer 2 只读写 vault Markdown 文件，不引入 Layer 1 不知道的状态。同一个 skill 无论由 Human 手动触发还是 Layer 2 调度，行为完全一致。

### 四种角色模式

| 模式 | AI 角色 | Human 角色 | 触发条件 | 状态 |
|:-----|:--------|:-----------|:---------|:-----|
| **Copilot** | 执行具体任务 | 在线，给指令 | Human 发起 + 明确命令 | ✅ |
| **Autopilot** | 自主探索 | 离线，事后审阅 | Human 离线 + agenda 有活跃方向 | ✅ 部分 |
| **Sparring** | 辩论伙伴 | 在线，讨论 idea | Human 发起 + 开放性问题 | 🔮 |
| **Reporter** | 结构化汇报 | 离线，异步审阅 | 定期 / 重大发现 / 需要决策 | 🔮 |

模式切换是**隐式**的——由交互上下文决定，不需要显式配置。例外：Autopilot 的权限边界在 `identity.md` 中显式定义（涉及信任边界）。

**Report 格式** 🔮：

```markdown
# Reports/YYYY-MM-DD-{type}.md
---
type: weekly / discovery / decision-needed
period: YYYY-MM-DD ~ YYYY-MM-DD
---
## Highlights
[Top 1-3 findings with evidence links]

## Progress by Direction
### Direction A
- **Actions taken**: ...
- **Key findings**: ...
- **Needs Human decision**: [yes/no]

## New Discoveries
[Unexpected patterns / notable new papers]

## Questions for Human
1. [Questions requiring Human judgment]

## Resource Usage
- Papers read: N / Experiments run: N / API tokens: ~N
```

## 3. Directory Structure

```
MindFlow/
│
├── Papers/              # 论文笔记（YYMM-ShortTitle.md）
├── Ideas/               # 研究 idea（status: raw → developing → validated → archived）
├── Projects/            # 项目追踪（status: planning → active → paused → completed）
├── Topics/              # 文献调研 / 跨论文分析报告
├── Experiments/         # 实验记录
├── Reports/             # AI 生成的报告
├── Meetings/            # 会议记录（YYYY-MM-DD-Description.md）
├── Daily/               # 每日研究日志（YYYY-MM-DD.md）
│
├── Domain-Map/          # 核心认知地图（按 domain 拆分）
│   ├── _index.md        #   索引页：domain 列表 + cross-domain insights
│   ├── VLA.md           #   各 domain 的四象限认知地图
│   ├── VLN.md
│   └── SpatialRep.md
│
├── Templates/           # Obsidian 模板（Paper.md, Idea.md, ...）
├── Attachments/         # 文件附件
│
├── skills/              # Skill 定义
│   ├── 1-literature/    #   文献类（paper-digest, cross-paper-analysis, literature-survey）
│   └── 5-evolution/     #   进化类（memory-distill）
│
├── references/          # 协议文档
│   ├── skill-protocol.md
│   ├── memory-protocol.md
│   ├── agenda-protocol.md
│   └── tag-taxonomy.md
│
├── Workbench/           # AI 工作状态（Human 可随时查看和编辑）
│   ├── agenda.md        #   研究议程
│   ├── identity.md      #   AI 身份与权限配置
│   ├── memory/          #   蒸馏后的记忆（patterns, insights, ...）
│   ├── queue/           #   待办队列（reading, review, questions）
│   ├── logs/            #   每日操作日志（YYYY-MM-DD.md）
│   └── evolution/       #   演化记录（changelog.md）
│
├── SPEC.md              # ★ 本文件：系统设计 single source of truth
├── CLAUDE.md            # AI 操作指令（引用 SPEC.md）
└── .obsidian/           # Obsidian 配置
```

## 4. Key Concepts

### 4.1 Notes

所有笔记遵循 `Templates/` 中对应的模板。共通约定：
- YAML frontmatter 存储结构化元数据
- 正文用中文撰写，英文技术术语保持英文不翻译
- 笔记之间通过 `[[wikilinks]]` 建立连接

| 类型 | 目录 | 命名规则 | 模板 |
|:-----|:-----|:---------|:-----|
| 论文笔记 | `Papers/` | `YYMM-ShortTitle.md`（YYMM 取自 date_publish） | `Templates/Paper.md` |
| 研究 Idea | `Ideas/` | 描述性名称 | `Templates/Idea.md` |
| 项目 | `Projects/` | 描述性名称 | `Templates/Project.md` |
| 文献调研 | `Topics/` | `{Topic}-Survey.md` | `Templates/Topic.md` |
| 实验 | `Experiments/` | 描述性名称 | `Templates/Experiment.md` |
| 会议 | `Meetings/` | `YYYY-MM-DD-Description.md` | `Templates/Meeting.md` |
| 每日日志 | `Daily/` | `YYYY-MM-DD.md` | `Templates/Daily.md` |

### 4.2 Domain Map

Domain Map 是 vault 中**层级最高的知识**——从所有 Papers/Topics/Ideas/Experiments 中蒸馏而来的核心认知。

**结构**：`Domain-Map/` 目录，每个研究 domain 一个文件，包含四个象限：

| 象限 | 含义 |
|:-----|:-----|
| **Established Knowledge** | 高置信度的领域共识，附来源论文 |
| **Active Debates** | 存在矛盾或未定论的观点 |
| **Open Questions** | 尚未回答的问题 |
| **Known Dead Ends** | 已证伪或不推荐的方向 |

**治理规则**：
- AI skill（literature-survey, cross-paper-analysis）不直接修改 Domain Map，只在产出文件中标注"建议加入 Domain-Map"
- memory-distill 可通过 `Workbench/queue/review.md` 提出晋升建议
- Human 有最终审批权

**新增 domain**：创建 `Domain-Map/{Name}.md`，复制四象限结构，在 `_index.md` 表格中添加一行。

### 4.3 Skills

Skill 是 MindFlow 的自动化核心——定义在 `skills/<category>/<name>/SKILL.md` 中的可执行能力单元。

**格式**：YAML frontmatter（元数据）+ Purpose + Steps + Guard + Examples。详见 → `references/skill-protocol.md`

**当前 skills**：

| Skill | 目录 | 功能 |
|:------|:-----|:-----|
| `paper-digest` | `skills/1-literature/` | 消化单篇论文 → Paper 笔记 |
| `cross-paper-analysis` | `skills/1-literature/` | 跨论文对比分析 → 分析报告 |
| `literature-survey` | `skills/1-literature/` | 主题级文献调研（搜索 + 批量 digest + 综合） |
| `memory-distill` | `skills/5-evolution/` | 从日志蒸馏记忆（patterns → insights） |

**触发方式**：自然语言触发，AI 识别意图后读取对应 SKILL.md 并严格执行。

### 4.4 Memory System

AI 的经验通过五级层级逐步蒸馏：

```
L0: Raw Log        Workbench/logs/YYYY-MM-DD.md     每次操作自动记录
     ↓ memory-distill 提取
L1: Pattern         Workbench/memory/patterns.md     跨日期重复出现的观察
     ↓ ≥2 独立来源
L2: Provisional     Workbench/memory/insights.md     初步洞察（待验证）
     ↓ 实验/文献验证
L3: Validated       Workbench/memory/insights.md     已验证洞察
     ↓ Human 审批 或 auto-promote
L4: Domain Map      Domain-Map/{Name}.md             持久领域知识
```

**核心规则**：
- **Append-only**：记忆文件只追加不修改
- **来源引用必须明确**：每条 pattern/insight 必须包含指向具体日志的 wikilink
- **Domain-Map logging**：每次更新 Domain Map 必须在 `Workbench/evolution/changelog.md` 中记录

详见 → `references/memory-protocol.md`

### 4.5 Evolution Mechanisms 🔮 Planned

Memory System 的 L0→L4 晋升由三种进化机制驱动（改编自 EvoScientist）：

| 机制 | 全称 | 触发时机 | 输入 | 输出 |
|:-----|:-----|:---------|:-----|:-----|
| **IDE** | Insight Direction Evolution | cross-paper-analysis 或 knowledge-synthesis 完成后 | Topics/ 分析 + Papers/ | 新方向 → `memory/insights.md`（provisional） |
| **IVE** | Insight Validation Evolution | 研究方向被放弃时 | agenda.md 废弃方向 + 关联实验/论文 | 失败教训 → `memory/failed-directions.md` |
| **ESE** | Experiment Strategy Evolution | 实验分析完成后 | `Experiments/<id>/` 全套文件 | 有效方法 → `memory/effective-methods.md` |

### 4.6 Autopilot Core Loop (insight-loop) 🔮 Planned

Autopilot 模式下 AI 的核心执行循环，每次触发执行一个完整 cycle：

```
insight-loop (one cycle)
    │
    ├── Phase 1: Orient — 读取 agenda + queue + memory + Domain Map，决定下一步
    │   优先级：queue/review 紧急项 > Human 新增任务 > 待验证 insight
    │         > agenda 最高优先级方向 > Domain Map Open Questions > paper-discovery
    │
    ├── Phase 2: Act — 查询 stage-skill-map.json，调用对应 skill（每 cycle 一个 action）
    │   执行前检查 identity.md 权限边界；需审批的写入 queue/review.md 并跳过
    │
    ├── Phase 3: Learn — 记录日志 + 条件触发进化（IDE/IVE/ESE）+ 检查 insight 晋升
    │
    └── Phase 4: Report — 满足条件时生成 Report
        条件：高置信度 insight 晋升 / 重大矛盾 / 实验结果异常 / 需要 Human 决策
```

**触发方式**：
- Layer 1：Human 手动执行 `/insight-loop`
- Layer 2：Daemon scheduler 自动触发（可配置频率）

**错误处理**：
- Skill 失败 → 记录日志，跳过本 cycle，不重试相同输入
- API 超时 → 重试一次（60s 后），仍失败则跳过并写入 queue/review
- 部分状态 → 利用 git commit/revert 实现原子性
- 资源耗尽 → 进入 COMPACT mode（读摘要而非全文），仍失败则暂停 Autopilot 并触发 Reporter

**API 预算**（定义在 `Workbench/identity.md`）：

| 参数 | 默认值 | 说明 |
|:-----|:-------|:-----|
| `daily_token_limit` | 500k | 日 token 上限 |
| `per_cycle_limit` | 50k | 单 cycle 上限 |
| `expensive_action_threshold` | 100k | 超过此值需审批 |

接近日上限时进入"节约模式"（仅处理 Human 队列，跳过自主探索）。

**并发与冲突**（Layer 2 场景）：
- AI 写共享文件前先读取当前版本，写后检查 git diff
- 若文件在读写间被他人修改，AI 的版本写入 `.conflict` 文件并通知 Human
- Append-only 文件（logs、queue、memory）冲突极少
- Domain Map：AI 只追加新条目，Human 可自由修改/重排，结构冲突概率极低

### 4.7 Workbench

`Workbench/` 是 AI 的工作状态目录，Human 可随时查看和编辑：

| 文件/目录 | 职责 |
|:----------|:-----|
| `agenda.md` | 研究议程（active/paused/abandoned directions） |
| `identity.md` | AI 身份、权限、预算配置 |
| `memory/` | 蒸馏后的记忆文件 |
| `queue/` | 待办队列（reading、review、questions） |
| `logs/` | 每日操作日志 |
| `evolution/` | 系统演化记录 |

详见 → `references/agenda-protocol.md`

## 5. Conventions

### 语言
- 正文用**中文**撰写
- 英文技术术语（模型名、方法名、benchmark 名）保持英文不翻译
- Frontmatter 字段名用英文

### 文件命名
- Papers: `YYMM-ShortTitle.md`（CamelCase，2-4 关键词）
- Domain Map: `Domain-Map/{Name}.md`（CamelCase）
- Meetings: `YYYY-MM-DD-Description.md`
- Logs: `YYYY-MM-DD.md`
- Skills: `skills/{N}-{category}/{kebab-case-name}/SKILL.md`

### Wikilinks
- 笔记间引用使用 `[[wikilinks]]`
- 带 alias：`[[2410-Pi0|π₀]]`
- 在 Markdown `*` 可能被误解析时转义：`π\*₀.₆`

### Tags
- 每篇笔记 1-3 个 tag
- 从 `references/tag-taxonomy.md` 中选取
- 需要新 tag 时遵循设计原则（粒度适中、正交、稳定）并更新 taxonomy

## 6. Protocols

| 协议 | 文件 | 管辖范围 |
|:-----|:-----|:---------|
| Skill Protocol | `references/skill-protocol.md` | SKILL.md 格式、frontmatter 字段、skill levels |
| Memory Protocol | `references/memory-protocol.md` | 记忆文件格式、L0-L4 晋升规则、更新规则 |
| Agenda Protocol | `references/agenda-protocol.md` | agenda.md 格式、AI 权限矩阵、Human override |
| Tag Taxonomy | `references/tag-taxonomy.md` | Tag 列表、选择原则、更新记录 |

## 7. AI Behavior

AI（Claude Code）通过 `CLAUDE.md` 接收操作指令。关键约束：

**权限边界**：
- **CAN**：读论文、更新记忆、生成报告、发现新论文、按 agenda 探索新方向
- **CAN**（有条件）：auto-promote validated insight 到 Domain Map（需符合 memory-protocol 规则）
- **NEED APPROVAL**：启动 >2h 实验、放弃研究方向、修改 Domain Map Established Knowledge
- **CANNOT**：删除已有笔记、修改 Human 撰写的内容、对外发布

**Skill 执行规则**：
- 每次执行 skill 必须先 Read 对应 SKILL.md，不凭记忆执行
- 严格遵守 SKILL.md 中的 Steps 和 Guard
- 所有操作记录到 `Workbench/logs/`

## 8. Changelog

| 日期 | 变更 | 影响范围 |
|:-----|:-----|:---------|
| 2026-03-27 | Domain Map 从 `Topics/` 迁移到 vault 根目录 `Domain-Map/`，按 domain 拆分为独立文件 | CLAUDE.md, skills, references |
| 2026-03-27 | `Templates/Paper.md` 扩充为 single source of truth（含 `%%` 注释指导），`paper-digest` SKILL.md Step 4 精简为引用模板 | Templates/, skills/ |
| 2026-03-27 | 新增 SPEC.md 作为系统设计的 single source of truth；补充 Layer 2、Role Fluidity、insight-loop、Evolution Mechanisms 等未实现设计 | vault root |
| 2026-03-26 | 初始 vault 结构搭建，skill 系统、记忆协议、议程协议就位 | 全局 |
