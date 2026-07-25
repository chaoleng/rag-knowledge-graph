# Palantir Ontology 独立根模块设计

## 目标

在 RAG 知识图谱中增加一个独立根模块 `Palantir Ontology（帕兰提尔本体论）`，用于组织 Palantir Ontology 的核心概念、数据语义、业务动作、治理和 AI 集成知识。

## 数据范围

新增一个无父节点的根节点：

- ID：`palantir-ontology`
- 标题：`Palantir Ontology（帕兰提尔本体论）`
- 类型：`root`
- 子节点：14 个

14 个子节点：

1. Ontology 概念与边界
2. Object Types 对象类型
3. Properties 属性
4. Link Types 关系类型
5. Interfaces 接口
6. Actions 动作
7. Functions 函数
8. 对象集合与查询
9. 数据接入与对象映射
10. 语义层与数字孪生
11. 安全与治理
12. 运营工作流
13. 本体驱动应用
14. AI 与智能体集成

`安全与治理` 标记为 `risk`，`AI 与智能体集成` 标记为 `research`，其余子节点使用普通细节类型。内容依据 Palantir 官方 Ontology 文档整理，不把产品能力描述成通用 RAG 能力。

## 多根节点支持

当前页面只渲染固定根节点 `rag`。改造为从节点数据动态收集所有 `parentId === null` 的根节点：

- 知识树逐个渲染 RAG 与 Palantir Ontology 根节点。
- 当前默认选中与重置行为仍保持 RAG，避免改变现有用户路径。
- 选中 Palantir Ontology 后，展开其子树并在详情面板显示独立路径。
- `pathOf`、局部关系图和节点筛选继续复用现有逻辑。

## 关系

新增关系：

- RAG ↔ Palantir Ontology：两个根模块互相关联。
- Palantir Ontology → LLM、Prompt、Grounding：表示本体语义、生成、提示词和证据落地的交叉关系。

现有 RAG、Prompt、幻觉和评估关系不删除、不改语义。

## 计数与同步

当前 59 个节点加上 1 个根节点和 14 个子节点后，总数为 74。必须同步修改：

- `index.html`
- `rag-concepts-preview.html`

两份文件的静态应用内容必须保持一致，顶部显示 `74 个概念节点`。

## 验证标准

1. 页面显示 `74 节点`，知识树同时显示 RAG 与 Palantir Ontology 两个根节点。
2. 展开 Palantir Ontology 后显示全部 14 个子节点。
3. 搜索 `Object Types`、`Actions`、`AI 与智能体集成` 能命中节点。
4. 选择新根节点后，面包屑以 `Palantir Ontology（帕兰提尔本体论）` 开头，详情面板显示 14 个子节点。
5. `risk` 和 `research` 筛选能分别找到安全与治理、AI 与智能体集成。
6. RAG 默认视图、原有 Prompt 12 个子节点和旧关系保持正常。
7. GitHub Pages 工作流成功后，公开 URL 显示同样的 74 节点和独立根模块。
