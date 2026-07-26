# 自托管 GitHub 项目知识问答 MVP 设计

## 目标

在现有 `rag-knowledge-graph` 工作之外，基于 `neo4j-labs/create-context-graph` 建立一个自托管、只读的研发项目知识问答 MVP。首批用户为研发负责人/项目经理。

系统必须把 GitHub 的研发事实与受控文档中的解释性证据连接到一个图谱中，回答项目问题时提供可验证来源，而非生成无法核验的总结。

## 选型与边界

### 选型

- 基线：fork `neo4j-labs/create-context-graph`。
- 图数据库：自托管 Neo4j；不使用 NAMS 或其他托管记忆后端。
- 数据访问：试点 GitHub 组织/仓库的只读 GitHub App；一个受控文档目录或知识库。
- 交互：问答页面、证据抽屉、局部关系图、管理员同步健康页。

### 包含

- GitHub 仓库、Issue、Pull Request、Commit、Review 的只读导入。
- 可选的 Workflow Run 导入，用于回答已知的构建/发布状态。
- 文档解析、分块、版本和证据锚点保存。
- 项目、需求、决策与风险的少量人工维护或固定目录导入。
- 受权限约束的项目知识问答与局部图谱探索。
- 新鲜度、来源、关系可信度与同步错误可见性。

### 不包含

- 向 GitHub、Jira 或其他外部系统写回数据。
- 自动分派、审批、风险自动处置或自动修改项目状态。
- 首期接入 Jira、飞书/Slack IM、全组织或全仓库。
- 自动风险评分与自动告警。
- 以未验证 LLM 抽取结果替代 GitHub API 返回的事实关系。

## 本体模型

```text
Project ─ owns ──────> Repository
Project ─ contains ──> Requirement
Project ─ contains ──> Risk
Project ─ records ───> Decision

Requirement ─ tracked_by ─> Issue
Issue ─ implemented_by ──> PullRequest
PullRequest ─ includes ───> Commit
PullRequest ─ reviewed_by ─> Review
PullRequest ─ verified_by ─> WorkflowRun

Document ─ supports ───────> Requirement | Decision | Issue | PullRequest
Document ─ has_chunk ──────> DocumentChunk
DocumentChunk ─ evidences ─> 以上任一对象或关系
```

所有节点保留来源系统、稳定主键、来源 URL、源更新时间、平台抓取时间与数据新鲜度。关系必须标记为 `explicit` 或 `inferred`：

- `explicit`：来自 GitHub API 的关联，例如 PR 关联/关闭 Issue、Commit 属于 PR、Review 属于 PR。
- `inferred`：从正文中解析出的 Issue 编号、URL、需求编号或文档引用；必须保存原文证据锚点，不能作为与 API 同等的事实。

## 身份与稳定键

| 对象 | 稳定键 |
|---|---|
| `Project` | 平台内部稳定 ID |
| `Repository` | GitHub `node_id` |
| `Issue` | 仓库 ID + Issue 编号 |
| `PullRequest` | 仓库 ID + PR 编号 |
| `Commit` | 完整 commit SHA |
| `WorkflowRun` | GitHub workflow run ID |
| `Document` | 来源系统 ID + 版本或内容哈希 |
| `DocumentChunk` | Document ID + 版本 + 片段锚点 |

标题、显示名称和文档路径不得作为关联主键。

## 数据同步

### GitHub App 权限

仅安装到试点仓库，且仅申请读取：

- Metadata
- Contents
- Issues
- Pull requests
- Commit statuses / Checks
- Actions（仅在启用构建/发布问答时）

### 行为

1. 首次对试点仓库执行历史回填。
2. 后续执行定时增量同步；实现应可迁移至 webhook 触发，但 MVP 不依赖 webhook。
3. 同步采用幂等 upsert。重复执行同一输入不得产生重复节点、边或证据片段。
4. 源对象删除时先标记 `deleted`，保留追溯记录，不立即物理删除。
5. GitHub API 或文档解析失败时保留最后有效快照，标记相关对象为 `stale` 或 `unavailable`，记录失败来源、时间和原因。

## 问答与证据

### 查询流程

```text
用户问题
  → 访问范围过滤
  → 项目与对象识别
  → 图谱事实检索
  → 关联文档片段检索
  → 受证据约束的答案生成
  → 结论、来源、新鲜度与不确定项输出
```

每个关键结论必须包含：

- 事实来源：对象名称、来源 URL、源更新时间。
- 文档证据：片段、位置、版本与来源 URL。
- 关系可信度：`explicit` 或 `inferred`。
- 数据状态：`fresh`、`stale` 或 `unavailable`。

当事实层与文档层冲突时，系统必须并列展示来源和时间，不得由模型静默决定哪一方正确。未找到足够证据时，系统必须明示该限制，禁止补全猜测。

## 界面

### 项目问答页

- 选择试点项目或仓库范围。
- 输入自然语言问题。
- 显示简短结论、事实依据、文档依据、不确定项/冲突和来源链接。

### 证据抽屉

- 展开单条结论的 GitHub Issue、PR、Commit、Review、文档片段及版本。
- 跳转到原始来源。

### 局部关系图

- 从项目、需求、Issue、PR 或文档展开与当前问题相关的局部子图。
- 通过边样式区分 `explicit` 与 `inferred`。
- 禁止默认渲染整个图数据库。

### 同步健康页

- 只向管理员显示。
- 显示每个数据源的最后成功时间、状态、失败原因和受影响对象数量。

## 权限与数据安全

- GitHub App 仅有读取权限，且仅面向试点仓库。
- 平台用户的仓库/文档访问范围必须在检索前过滤。
- 未授权数据不得通过答案、关系图、聚合计数或错误信息泄露其存在性。
- 将私有代码或文档交给模型前，必须确认模型供应商、数据保留与组织合规策略；不获准时使用已批准的私有部署或本地模型。

## 异常处理

| 场景 | 行为 |
|---|---|
| 没有匹配资料 | 返回未找到足够证据，不生成猜测性结论。 |
| 数据过期 | 显示最后成功同步时间和受影响来源。 |
| GitHub 同步失败 | 保留快照，标记 `stale`，记录重试信息。 |
| 文档解析失败 | 保存文档元数据和失败原因，不写入不完整证据。 |
| 文档与事实冲突 | 并列显示冲突来源及各自时间。 |
| 权限不足 | 返回通用无结果/无权限结果，不泄露对象存在性。 |
| 模型服务失败 | 返回已检索到的原始事实和证据链接，不生成摘要。 |

## 验收标准

1. 对一个试点需求，可从文档或 Issue 追溯到关联 PR，并打开每个原始来源。
2. 每个“已完成”“已合并”“被阻塞”或“采用某方案”的关键结论均带可点击来源和时间。
3. 无权用户不能通过问答、关系图、计数或错误信息获取受限仓库内容。
4. GitHub 同步失败后，答案显示数据过期，且不将快照表述为当前状态。
5. 当文档状态与 GitHub 状态冲突时，系统显示冲突与两侧证据，而非编造统一结论。
6. 同一同步任务重复运行不会生成重复节点、边或证据片段。
7. 试点研发负责人可用自然语言回答项目状态、关联需求/Issue/PR 和决策依据，并可验证每条关键结论。
