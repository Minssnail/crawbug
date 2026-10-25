# Learning Graph for 网络爬虫技术

本节包含《网络爬虫技术》智能教材的知识依赖图谱（Learning Graph）。知识图谱将整本教材的 **240 个核心概念** 表示为有向无环图（DAG）中的节点，有向边代表概念之间的先修依赖关系（Learning Dependency）。

在图谱的最左侧是**先修基石概念（Foundational Concepts）**，它们没有出边（不依赖其他课内概念），只有指向它们的入边；在图谱的最右侧是**高阶与综合实训概念（Terminal / Capstone Concepts）**，掌握它们需要先修完其路径上的所有前置概念。

## Course Description

我们使用 [课程大纲（Course Description）](../course-description.md) 作为提取核心概念的种子文档。课程大纲遵循 2001 布鲁姆教育目标分类学（Bloom's Taxonomy）组织六个层级的学习成果。

## List of Concepts

我们从课程大纲中提炼出 [240 个核心概念列表](./concept-list.md)，每个概念均采用不超过 32 个字符的精炼命名。

## Concept Dependency List

我们构建了严格验证的有向无环图（DAG），并提供两种工程格式：

1. [CSV 格式（learning-graph.csv）](learning-graph.csv)：包含 `ConceptID,ConceptLabel,Dependencies,TaxonomyID` 四列。
2. [JSON 格式（learning-graph.json）](learning-graph.json)：遵循 `vis-network` 标准格式，包含 `metadata`、`groups`、`nodes`（含递归概念影响力分值 `cis`）与 `edges`。

## Analysis & Documentation

### Course Description Quality Assessment

评估课程大纲的完整性、布鲁姆分类覆盖度与概念派生潜力（得分：**98/100**）。

- [查看课程大纲质量评估报告](course-description-assessment.md)

### Learning Graph Quality Validation

利用图算法对 240 个概念节点和 523 条依赖边进行拓扑验证：

- **有效 DAG 验证**：✅ 无环（0 Cycles）
- **连通分量**：✅ 单一连通图（1 Connected Component，0 孤立节点）
- **基石概念**：7 个入口节点
- **终端节点比例**：18.8%（45 个节点，处于 5%~40% 健康区间）
- **最长依赖链深度**：22 层

- [查看知识图谱质量验证报告](quality-metrics.md)

### Concept Taxonomy

为了在可视化图谱中直观呈现知识模块结构，我们将 240 个概念划分为 **12 个分类体系（Taxonomies）**，涵盖从导论合规、HTTP 协议、请求编程、静态解析、API 逆向、Playwright 动态渲染、反爬对抗、异步并发、多源存储、Scrapy 框架到 AI 智能爬虫与综合工程实训。

- [查看概念分类体系定义](concept-taxonomy.md)

### Taxonomy Distribution

统计并检验 12 个分类体系的概念分布均衡度（最大类别占比 10.0%，最小 5.0%，极差仅 5.0%，无任何类别超过 30% 阈值）。

- [查看分类体系分布报告](./taxonomy-distribution.md)
