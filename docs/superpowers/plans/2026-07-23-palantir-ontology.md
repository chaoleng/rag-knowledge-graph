# Palantir Ontology Module Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an independent `Palantir Ontology（帕兰提尔本体论）` root module with 14 children and make the static graph render multiple roots.

**Architecture:** Keep the self-contained HTML application and add a second root data branch. Replace the tree's hard-coded `renderTreeNode('rag')` call with a `rootIds` list derived from nodes with no parent; retain RAG as the default selected/reset root. Keep graph layout, search, filters, inspector, and source-link generation unchanged.

**Tech Stack:** Static HTML/JavaScript, Node.js 20+, GitHub Pages.

---

### Task 1: Add the independent Ontology data branch

**Files:**
- Modify: `index.html:49,75-90`
- Modify: `rag-concepts-preview.html:49,75-90`

- [ ] **Step 1: Add the root and 14 children after the existing module loop data**

Add this data before `const byId` and attach all children to `palantir-ontology`:

```js
const ontologyRoot={id:'palantir-ontology',title:'Palantir Ontology（帕兰提尔本体论）',type:'root',parentId:null,summary:'把真实世界对象、关系、属性和业务动作组织成可操作的语义层。',tags:['Palantir','Ontology','语义层'],body:'Ontology 连接数据资产、业务语义、权限和可执行动作，使分析结果能够进入运营工作流。'};
const ontologyChildren=[
  ['ontology-core-concepts','Ontology 概念与边界','定义对象、关系、属性和动作如何共同构成面向运营的语义层。'],
  ['ontology-object-types','Object Types 对象类型','把真实实体或事件建模为可识别、可查询、可授权的对象类型。'],
  ['ontology-properties','Properties 属性','描述对象特征、时间序列、地理信息和元数据，并映射到底层数据。'],
  ['ontology-link-types','Link Types 关系类型','用具有业务含义的关系连接对象类型，表达上下游、归属和依赖。'],
  ['ontology-interfaces','Interfaces 接口','抽取多个对象类型共享的属性、关系或动作，形成可复用的能力形状。'],
  ['ontology-actions','Actions 动作','定义如何修改对象并把用户或系统决策连接到可审计的业务操作。'],
  ['ontology-functions','Functions 函数','计算派生结果、更新对象、发送通知或调用外部 API，承接动作执行逻辑。'],
  ['ontology-object-sets','对象集合与查询','按类型、关系、属性和权限组合对象集合，为应用和分析提供稳定查询边界。'],
  ['ontology-data-integration','数据接入与对象映射','把数据集、虚拟表和外部系统映射到对象、属性与关系，并追踪来源。'],
  ['ontology-semantic-layer','语义层与数字孪生','用统一语义描述现实世界实体、状态和事件，连接数据资产与运营上下文。'],
  ['ontology-security-governance','安全与治理','对对象、属性、关系、动作和日志实施细粒度权限、标记、审计与变更治理。','risk'],
  ['ontology-operational-workflows','运营工作流','把对象状态、动作、函数、通知和审批组合成可观测的业务闭环。'],
  ['ontology-driven-apps','本体驱动应用','围绕对象和动作构建面向业务用户的应用、仪表盘和决策界面。'],
  ['ontology-ai-agents','AI 与智能体集成','让 AI 和智能体在本体语义、权限、动作和审计约束下读取与执行。','research']
];
const nodes=[{id:'rag',title:'RAG：检索增强生成',type:'root',parentId:null,summary:'外部知识检索增强大语言模型生成的完整系统方法。',tags:['RAG','根节点'],body:'RAG 由知识加工、检索、上下文编排、生成和评估组成。'},ontologyRoot];
```

Replace the old single-root initialization and module loop with:

```js
function addNode(parentId,item){const [id,title,summary,type='detail',nested=[]]=item;nodes.push({id,title,type,parentId,summary,tags:[title],body:summary});for(const child of nested)addNode(id,child)}
for(const module of modules){nodes.push({...module,parentId:'rag',children:undefined});for(const item of module.children)addNode(module.id,item)}
for(const item of ontologyChildren)addNode(ontologyRoot.id,item)
```

- [ ] **Step 2: Update the visible count and related edges**

Change the brand subtitle from `59 个概念节点` to `74 个概念节点`. Add the following entries to `related` without removing existing entries:

```js
'rag':['palantir-ontology'],
'palantir-ontology':['rag','rag-concept-llm','rag-concept-prompt','rag-concept-grounding'],
```

- [ ] **Step 3: Keep the two HTML copies identical**

Apply the same final source content to `index.html` and `rag-concepts-preview.html`. Verify with:

```bash
python -c "from pathlib import Path; a=Path('index.html').read_bytes(); b=Path('rag-concepts-preview.html').read_bytes(); assert a == b; print('HTML copies identical')"
```

Expected: `HTML copies identical`.

### Task 2: Make tree rendering multi-root aware

**Files:**
- Modify: `index.html:79,101,103,114`
- Modify: `rag-concepts-preview.html:79,101,103,114`

- [ ] **Step 1: Derive root IDs after building the child map**

Immediately after the existing `children` map construction, add:

```js
const rootIds=nodes.filter(node=>!node.parentId).map(node=>node.id);
```

- [ ] **Step 2: Render every root while preserving RAG defaults**

Keep `autoExpand` adding `rag`. In both HTML files, replace only this exact assignment:

```js
treeScroll.innerHTML=renderTreeNode('rag');
```

with:

```js
treeScroll.innerHTML=rootIds.map(id=>renderTreeNode(id)).join('');
```

Leave the following `treeCount` update and both event-listener expressions unchanged. The reset handler must remain `state.selectedId='rag'` and `state.expandedIds=new Set(['rag'])`.

- [ ] **Step 3: Apply the runtime changes identically to both HTML files**

Run the copy comparison from Task 1 again. No other UI or layout behavior should change.

### Task 3: Validate, deploy, and smoke-test

**Files:**
- Verify: `index.html`
- Verify: `rag-concepts-preview.html`
- Verify: `https://chaoleng.github.io/rag-knowledge-graph/rag-concepts-preview.html`

- [ ] **Step 1: Run static assertions**

```bash
python -c "from pathlib import Path; import re; s=Path('index.html').read_text(encoding='utf-8'); ids=['palantir-ontology','ontology-core-concepts','ontology-object-types','ontology-properties','ontology-link-types','ontology-interfaces','ontology-actions','ontology-functions','ontology-object-sets','ontology-data-integration','ontology-semantic-layer','ontology-security-governance','ontology-operational-workflows','ontology-driven-apps','ontology-ai-agents']; assert '74 个概念节点' in s; assert all(i in s for i in ids); assert 'rootIds=nodes.filter(node=>!node.parentId).map(node=>node.id)' in s; assert 'rootIds.map(id=>renderTreeNode(id)).join(\'\')' in s; print('ontology static contract: PASS')"
git diff --check
```

Expected: `ontology static contract: PASS` and no diff-check output.

- [ ] **Step 2: Commit and push the target repository**

```bash
git add index.html rag-concepts-preview.html docs/superpowers/specs/2026-07-23-palantir-ontology-design.md docs/superpowers/plans/2026-07-23-palantir-ontology.md
git commit -m "feat: add Palantir Ontology root module"
git push origin main
```

- [ ] **Step 3: Confirm Pages deployment and public behavior**

Check the latest `Deploy RAG knowledge graph` workflow run for the pushed commit. Then open the public URL and verify:

1. Header and tree count show `74`.
2. Both `RAG：检索增强生成` and `Palantir Ontology（帕兰提尔本体论）` appear as root rows.
3. Clicking the Ontology root shows all 14 children in the tree and inspector.
4. Searching `Object Types` and `AI 与智能体集成` returns the new nodes.
5. Existing Prompt children and RAG default view remain available.
