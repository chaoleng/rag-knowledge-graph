# Palantir Ontology 详情扩充 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为 14 个 Palantir Ontology 子节点增加结构化详情，同时保持现有 74 节点知识图谱和 RAG 页面行为不变。

**Architecture:** 在两个现有静态 HTML 文件的节点数据区增加 `ontologyDetails` 索引，并在节点创建后把 `tags` 与 `sections` 合并到对应节点。详情渲染只对拥有 `sections` 的节点使用带标题区块；旧 RAG 节点继续使用 `body`/`summary` 回退。两份 HTML 必须保持字节一致。

**Tech Stack:** 静态 HTML、原生 JavaScript、现有 CSS、Python 静态契约脚本、Browser smoke test。

---

## 文件边界

- Modify: `index.html:76-96,129` — 根节点、Ontology 详情数据、节点合并和详情面板的主页面副本。
- Modify: `rag-concepts-preview.html:76-96,129` — 与 `index.html` 完全相同的公开预览副本。
- Modify: `docs/superpowers/specs/2026-07-23-palantir-ontology-detail-expansion-design.md` — 已审核的设计规格，不再改动，除非实现发现规范矛盾。
- Test: 不新增持久测试文件；使用确定性的 Python 静态契约和本地/公开浏览器验证。

### Task 1: 添加结构化 Ontology 详情数据

**Files:**
- Modify: `index.html:76-96`
- Modify: `rag-concepts-preview.html:76-96`

- [ ] **Step 1: 在 `ontologyChildren` 后增加详情索引**

在两个 HTML 文件的 `ontologyChildren` 数组结束后、`const nodes=` 之前增加 `ontologyDetails`。必须包含以下 14 个键，每个键必须有 `tags` 和四个 `sections`：

```js
const ontologyDetails={
  'ontology-core-concepts':{tags:['语义层','对象','属性','关系','动作'],sections:[
    {label:'定义',text:'说明对象、属性、关系和动作如何共同形成面向运营的语义层，并明确它不是单纯的数据库表目录。'},
    {label:'核心要素',text:'对象类型表达实体或事件，属性描述状态，关系连接业务上下文，动作和函数把决策转化为可审计的操作。'},
    {label:'典型例子',text:'以设备为对象，关联位置、维护记录和告警，再通过动作创建维护工单。'},
    {label:'落地提示',text:'先定义稳定的业务语义和责任边界，再映射数据源；不要把一次性报表字段直接当成长期对象模型。'}
  ]},
  'ontology-object-types':{tags:['Object Types','对象建模','实体','事件','生命周期'],sections:[
    {label:'定义',text:'把真实实体或事件建模为可识别、可查询、可授权的业务对象类型。'},
    {label:'核心要素',text:'稳定标识、业务属性、关系、生命周期、访问策略和数据来源共同决定对象类型的可用性。'},
    {label:'典型例子',text:'将设备建模为对象类型，连接位置、维护记录、告警和负责人。'},
    {label:'落地提示',text:'先确定稳定的业务标识，再设计属性和关系；不要把临时分析字段直接固化为核心对象模型。'}
  ]},
  'ontology-properties':{tags:['Properties','属性','时间序列','地理信息','派生属性'],sections:[
    {label:'定义',text:'用属性描述对象的特征、状态、时间序列、地理信息和元数据，并保留与来源数据的映射关系。'},
    {label:'核心要素',text:'属性类型、来源字段、更新时间、质量状态、是否派生以及读取和修改权限。'},
    {label:'典型例子',text:'设备对象同时拥有型号、当前位置、温度时间序列和最近维护时间。'},
    {label:'落地提示',text:'区分源数据、计算属性和人工修正值；对时间敏感属性明确新鲜度和缺失值语义。'}
  ]},
  'ontology-link-types':{tags:['Link Types','关系类型','基数','遍历','依赖'],sections:[
    {label:'定义',text:'用具有业务含义的关系连接对象类型，表达归属、上下游、依赖和事件关联。'},
    {label:'核心要素',text:'关系方向、基数、目标类型、有效期、遍历方式、来源和关系级权限。'},
    {label:'典型例子',text:'设备属于工厂、设备产生告警、维护工单服务设备，这些关系共同表达运营上下文。'},
    {label:'落地提示',text:'关系应回答明确的业务问题；控制高扇出遍历和重复关系，必要时记录生效时间与来源。'}
  ]},
  'ontology-interfaces':{tags:['Interfaces','接口','共享能力','复用','兼容性'],sections:[
    {label:'定义',text:'抽取多个对象类型共享的属性、关系或动作，形成可复用的能力形状。'},
    {label:'核心要素',text:'共享字段、共享关系、共享动作、实现对象类型和接口变更兼容规则。'},
    {label:'典型例子',text:'设备和车辆都实现可维护接口，因此都能提供维护状态、维护记录关系和提交维护动作。'},
    {label:'落地提示',text:'接口描述稳定能力而不是临时字段集合；变更共享接口前要评估所有实现对象和应用。'}
  ]},
  'ontology-actions':{tags:['Actions','动作','写回','授权','审计','幂等'],sections:[
    {label:'定义',text:'定义如何修改对象或触发业务流程，把用户或系统决策连接到可审计的业务操作。'},
    {label:'核心要素',text:'输入参数、前置校验、授权、状态变化、副作用、错误处理、幂等键和审计记录。'},
    {label:'典型例子',text:'用户批准维护工单后，动作更新工单状态、安排负责人并记录批准人和时间。'},
    {label:'落地提示',text:'把动作设计成明确的业务意图；对重复提交、部分失败和越权调用定义可观察结果。'}
  ]},
  'ontology-functions':{tags:['Functions','函数','派生计算','异步','外部 API'],sections:[
    {label:'定义',text:'承接派生计算、对象更新、通知发送或外部系统调用的执行逻辑。'},
    {label:'核心要素',text:'输入输出、执行时机、纯计算或副作用、超时重试、权限、日志和失败补偿。'},
    {label:'典型例子',text:'函数根据设备温度和历史趋势计算风险等级，再供告警动作和运营应用使用。'},
    {label:'落地提示',text:'区分可重复的纯计算与有副作用的外部调用；为异步函数设计状态、超时和重试边界。'}
  ]},
  'ontology-object-sets':{tags:['对象集合','查询','过滤','关系遍历','权限边界'],sections:[
    {label:'定义',text:'按类型、关系、属性和权限组合对象集合，为应用和分析提供稳定的查询边界。'},
    {label:'核心要素',text:'筛选条件、关系遍历、排序分页、权限裁剪、快照时点和查询性能。'},
    {label:'典型例子',text:'查询当前工厂中处于高风险状态且过去七天未维护的设备集合。'},
    {label:'落地提示',text:'先限定对象范围再扩展关系；避免无界遍历，并确保查询结果始终服从对象和属性权限。'}
  ]},
  'ontology-data-integration':{tags:['数据接入','对象映射','数据集','虚拟表','血缘','数据质量'],sections:[
    {label:'定义',text:'把数据集、虚拟表和外部系统映射到对象、属性与关系，并追踪数据来源和同步状态。'},
    {label:'核心要素',text:'主键匹配、字段映射、增量同步、冲突处理、血缘、质量校验和失败重跑。'},
    {label:'典型例子',text:'把 ERP 设备主数据、IoT 温度流和工单系统映射到同一设备对象及其关系。'},
    {label:'落地提示',text:'先解决统一标识和主数据责任，再处理字段映射；同步延迟和脏数据必须在应用中可见。'}
  ]},
  'ontology-semantic-layer':{tags:['语义层','数字孪生','现实实体','状态','事件','新鲜度'],sections:[
    {label:'定义',text:'用统一语义描述现实世界实体、状态和事件，把数据资产连接到运营上下文。'},
    {label:'核心要素',text:'现实实体标识、当前状态、历史事件、时间语义、数据来源和新鲜度。'},
    {label:'典型例子',text:'设备对象同时呈现实时状态、历史告警、维护计划和所属工厂，形成可操作的数字表示。'},
    {label:'落地提示',text:'明确当前状态与历史事件的区别；为延迟、过期和来源冲突定义可解释的处理规则。'}
  ]},
  'ontology-security-governance':{tags:['安全','治理','权限','标记','审计','最小权限'],sections:[
    {label:'定义',text:'对对象、属性、关系、动作和日志实施细粒度权限、标记、审计与变更治理。'},
    {label:'核心要素',text:'身份、角色、对象范围、属性限制、动作授权、数据标记、审计事件和审批流程。'},
    {label:'典型例子',text:'维护团队可以查看负责区域设备并提交维护动作，但不能读取其他区域的敏感成本属性。'},
    {label:'落地提示',text:'默认采用最小权限；同时验证读权限和写动作权限，定期检查权限漂移与审计完整性。'}
  ]},
  'ontology-operational-workflows':{tags:['运营工作流','状态机','触发器','审批','通知','可观测性'],sections:[
    {label:'定义',text:'把对象状态、动作、函数、通知和审批组合成可观测的业务闭环。'},
    {label:'核心要素',text:'状态模型、触发条件、人工审批、自动动作、超时、异常路径和流程指标。'},
    {label:'典型例子',text:'告警触发检查函数，确认风险后创建工单，经主管批准后安排维护并关闭告警。'},
    {label:'落地提示',text:'显式建模等待、拒绝和失败状态；为每个自动动作保留负责人、原因和可追踪日志。'}
  ]},
  'ontology-driven-apps':{tags:['本体驱动应用','对象视图','动作表单','仪表盘','写回'],sections:[
    {label:'定义',text:'围绕对象和动作构建面向业务用户的应用、仪表盘和决策界面。'},
    {label:'核心要素',text:'对象详情、关系导航、筛选视图、动作表单、权限裁剪、写回反馈和任务上下文。'},
    {label:'典型例子',text:'运营人员从工厂视图进入设备详情，查看告警和维护历史后直接提交延期维护动作。'},
    {label:'落地提示',text:'围绕业务任务而不是数据表组织页面；动作成功、失败和权限不足都要给出明确反馈。'}
  ]},
  'ontology-ai-agents':{tags:['AI','智能体','grounding','工具调用','权限','审计','评估'],sections:[
    {label:'定义',text:'让 AI 和智能体在本体语义、权限、动作和审计约束下读取信息并执行受控操作。'},
    {label:'核心要素',text:'语义 grounding、对象查询、工具或动作调用、用户授权、人工确认、审计和结果评估。'},
    {label:'典型例子',text:'智能体根据设备对象、告警和维护规则总结风险，并在用户确认后提交维护动作。'},
    {label:'落地提示',text:'先限制可读对象和可执行动作；高影响写操作需要确认、幂等和完整审计，不能只依赖模型文本。'}
  ]}
};
```

- [ ] **Step 2: 在两个文件的节点创建循环后合并详情**

保留现有节点创建循环，并在其后追加：

```js
for(const node of nodes){
  const detail=ontologyDetails[node.id];
  if(detail){
    node.tags=[...new Set([...node.tags,...detail.tags])];
    node.sections=detail.sections;
  }
}
```

这保证旧节点不受影响，Ontology 节点的 `summary` 仍保持现有短文本。

- [ ] **Step 3: 运行静态数据检查**

从 `E:/omp/rag-knowledge-graph` 执行：

```powershell
python -c 'from pathlib import Path; ids=["ontology-core-concepts","ontology-object-types","ontology-properties","ontology-link-types","ontology-interfaces","ontology-actions","ontology-functions","ontology-object-sets","ontology-data-integration","ontology-semantic-layer","ontology-security-governance","ontology-operational-workflows","ontology-driven-apps","ontology-ai-agents"]; ts=[Path("index.html").read_text(encoding="utf-8"),Path("rag-concepts-preview.html").read_text(encoding="utf-8")]; assert all("const ontologyDetails" in t and all(i in t for i in ids) for t in ts); assert ts[0]==ts[1]; print("ontology detail data: PASS")'
```

Expected output: `ontology detail data: PASS`。

- [ ] **Step 4: Commit data model changes**

```powershell
git add index.html rag-concepts-preview.html
git commit -m "feat: add Ontology detail sections"
```

### Task 2: 渲染带标题的详情区块

**Files:**
- Modify: `index.html:129`
- Modify: `rag-concepts-preview.html:129`

- [ ] **Step 1: 替换 `renderInspector` 中的正文计算**

将现有：

```js
const paragraphs=(node.body||node.summary).split('；').map(text=>`<p>${esc(text)}。</p>`).join('');
```

替换为：

```js
const sections=node.sections||[{label:'说明',text:node.body||node.summary}];
const paragraphs=sections.map(section=>`<div class="detail-section"><h4>${esc(section.label)}</h4><p>${esc(section.text)}</p></div>`).join('');
```

- [ ] **Step 2: 保持现有容器和回退逻辑**

不要改变以下容器结构：

```js
<div class="info-block"><h3>节点内容</h3><div class="content-copy">${paragraphs}</div></div>
```

这样旧 RAG 节点会显示一个“说明”区块，Ontology 节点会显示四个结构化区块；父节点、子节点、相关节点和来源区域保持原样。

- [ ] **Step 3: 确认 HTML 转义边界**

详情区块中的 `section.label` 和 `section.text` 必须分别经过 `esc`。不得使用 `innerHTML` 直接拼接未转义的详情文本；不得把详情正文加入 `matches()`，搜索关键字通过 `tags` 和 `summary` 提供。

- [ ] **Step 4: 在浏览器中验证详情区块**

打开本地页面 `file:///E:/omp/rag-knowledge-graph/rag-concepts-preview.html`，点击 `Palantir Ontology（帕兰提尔本体论）` 下的 `Object Types 对象类型`，确认详情面板同时出现：

```text
定义
核心要素
典型例子
落地提示
```

再点击原有 `RAG：检索增强生成` 和 `RAG 评估`，确认旧节点仍有“说明”内容、父子节点和相关节点区域。

- [ ] **Step 5: Commit renderer changes**

```powershell
git add index.html rag-concepts-preview.html
git commit -m "feat: render structured Ontology details"
```

### Task 3: 全量静态与交互验证

**Files:**
- Test: `index.html`
- Test: `rag-concepts-preview.html`

- [ ] **Step 1: 验证双副本一致和节点数量**

```powershell
python -c 'from pathlib import Path; a=Path("index.html").read_text(encoding="utf-8"); b=Path("rag-concepts-preview.html").read_text(encoding="utf-8"); ids=["ontology-core-concepts","ontology-object-types","ontology-properties","ontology-link-types","ontology-interfaces","ontology-actions","ontology-functions","ontology-object-sets","ontology-data-integration","ontology-semantic-layer","ontology-security-governance","ontology-operational-workflows","ontology-driven-apps","ontology-ai-agents"]; assert a==b; assert "74 个概念节点" in a and all(a.count(i)>=2 for i in ids); assert a.count("label:'定义'")>=14 and a.count("label:'核心要素'")>=14 and a.count("label:'典型例子'")>=14 and a.count("label:'落地提示'")>=14; print("ontology detail contract: PASS")'
git diff --check
```

Expected output: `ontology detail contract: PASS` and no `git diff --check` output。

- [ ] **Step 2: 本地浏览器 smoke test**

通过浏览器打开本地页面并验证：

1. 页面标题仍为 `RAG 知识图谱`。
2. 顶部显示 `局部关系探索 · 74 个概念节点`。
3. 树计数显示 `74 节点`。
4. Ontology 根节点下仍有 14 个子节点。
5. 点击 `Object Types 对象类型` 后，详情面板显示四个区块。
6. 搜索 `对象建模` 命中 `Object Types 对象类型`。
7. 搜索 `AI 与智能体` 命中 `AI 与智能体集成`。
8. 点击 `安全与治理` 后仍显示风险类型。
9. 点击原有 RAG 节点、重置视图、展开邻居均无 JavaScript 错误。

- [ ] **Step 3: 提交最终变更**

```powershell
git add index.html rag-concepts-preview.html
git commit -m "test: verify Ontology detail expansion"
```

- [ ] **Step 4: 推送并验证公开页面**

```powershell
git push origin main
```

确认 GitHub Actions `Deploy RAG knowledge graph` 对最新提交结论为 `success`。打开：

https://chaoleng.github.io/rag-knowledge-graph/rag-concepts-preview.html

重复 Task 3 Step 2 的公开页面验证，并确认页面仍显示 `74` 个节点、Ontology 根节点和四段详情。
