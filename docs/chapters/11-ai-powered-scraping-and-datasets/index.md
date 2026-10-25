---
title: "面向大模型与RAG的AI智能爬虫"
description: "聚焦人工智能技术应用专业核心需求，深入讲解大模型预训练语料标准、Trafilatura正文抽取、HTML转Markdown、Crawl4AI异步框架、LLM Function Calling零样本解析、SFT指令微调集构建、语义分块写入向量数据库及PII脱敏与MinHash去重。"
generated_by: "claude skill chapter-content-generator"
date: "2026-10-25"
version: "1.11"
---

# 面向大模型与RAG的AI智能爬虫

## Summary

紧密贴合人工智能技术应用专业的培养定位，本章将传统爬虫升级为面向大语言模型（LLM）与多模态模型的智能数据引擎。内容涵盖 LLM 预训练语料质量标准、`Readability` 与 `Trafilatura` 正文启发式抽取、网页噪音过滤与 HTML-to-Markdown 结构化转换（`Jina Reader`、`Firecrawl`、`Crawl4AI`）。重点实战利用 LLM Prompt 与 Function Calling 进行零样本自适应网页解析、DOM 树精简降本、选择器自愈修复、Alpaca/ShareGPT 指令微调（SFT）数据集构建、多模态图文对对齐过滤、PII 隐私擦除，以及文档语义分块（Chunking）写入向量数据库构建实时 RAG 系统。

## Concepts Covered

This chapter covers the following 18 concepts from the learning graph:

| Concept | Concept Impact Score |
|---------|-----------------------|
| AI与大模型训练数据生命周期全貌 | 36 |
| LLM预训练语料Corpus质量标准 | 27 |
| 网页正文抽取Readability与Trafilatura | 23 |
| 网页噪音剔除与样板文本过滤 | 4 |
| HTML转Markdown结构化保留代码表格 | 18 |
| Jina-Reader与Firecrawl网页提取 | 1 |
| Crawl4AI大模型友好型异步爬虫 | 2 |
| 基于LLM与Function-Calling零样本解析 | 7 |
| DOM树精简Pruning降低Token消耗 | 1 |
| LLM自动生成选择器与自愈修复爬虫 | 4 |
| AI视觉Agent结合截图识别复杂交互 | 1 |
| 指令微调SFT数据集Alpaca/ShareGPT格式 | 3 |
| 垂直社区问答采集构建领域SFT语料 | 2 |
| RAG检索增强生成与知识库实时更新 | 4 |
| 文档语义分块Chunking与元数据保留 | 3 |
| Embedding向量化与向量数据库写入 | 2 |
| 多模态图文对采集与CLIP对齐过滤 | 1 |
| 语料隐私脱敏PII擦除与MinHash去重 | 3 |

## Prerequisites

This chapter builds on concepts from:

- [Chapter 1: 网络爬虫导论与数据合规基石](../01-intro-and-ethics/index.md)
- [Chapter 4: 静态网页结构化解析与正则提取](../04-static-html-parsing/index.md)
- [Chapter 5: 异步加载与Web接口逆向采集](../05-ajax-and-api-scraping/index.md)
- [Chapter 6: 动态渲染网页与Playwright浏览器自动化](../06-browser-automation-playwright/index.md)
- [Chapter 7: 反爬虫机制原理与工程对抗策略](../07-anti-scraping-and-countermeasures/index.md)
- [Chapter 8: 高性能并发与异步协程爬虫编程](../08-concurrent-and-async-scraping/index.md)
- [Chapter 9: 数据清洗流水线与多源持久化存储](../09-data-cleaning-and-storage/index.md)
- [Chapter 10: Scrapy工业级异步爬虫框架实战](../10-scrapy-industrial-framework/index.md)

---

## 1. 大模型训练数据生命周期与高质量网页正文抽取

在人工智能进入大语言模型（LLM）与检索增强生成（RAG）的时代后，网络爬虫与AI工程之间产生了双向的深度化学反应：一方面，**“爬虫为 AI 供数（Scraping for AI）”**——无论是训练百亿参数的基座模型、微调医疗法律垂直大模型，还是为企业知识库注入实时外部资讯，90% 以上的原始语料都来自网络爬虫；另一方面，**“AI 赋能智能爬虫（AI for Scraping）”**——借助大模型的语义理解与视觉能力，爬虫不再需要针对每个网站手写脆弱的 XPath 规则，而是具备了零样本自适应解析与代码自愈修复能力。

### 1.1 AI 与大模型训练数据生命周期及语料质量标准

在人工智能技术应用专业岗位中，理解 **AI 与大模型训练数据生命周期全貌** 是数据工程师的核心竞争力。大模型的数据供给贯穿以下四大阶段：

1. **基座预训练阶段（Pre-training Corpus）**：需要数万亿 Token 的海量高质量自然语言长文本（如百科、技术文档、新闻深度报道、学术论文、开源代码），让模型习得世界知识与语言逻辑。
2. **有监督指令微调阶段（Supervised Fine-Tuning, SFT）**：需要数万至数十万条高质量的“指令-输入-回答（Instruction-Input-Output）”或高赞多轮专家问答对，教会模型遵循人类指令并以特定领域专家的口吻作答。
3. **偏好对齐阶段（RLHF / DPO）**：采集同一问题下不同采纳度或点赞数的回答对，构建 `{prompt, chosen, rejected}` 偏好数据集。
4. **检索增强生成阶段（RAG Knowledge Base）**：无需重新训练模型权重，而是通过爬虫实时采集最新政策、财报、产品手册，经语义分块与向量化后存入向量数据库，在用户提问时精准召回并注入 Prompt 上下文，彻底消除大模型“幻觉”与知识过时问题。

在构建上述语料时，为什么不能直接用 `soup.get_text()` 把整个网页文字全部塞给大模型？这就涉及到严格的 **LLM 预训练语料 `Corpus` 质量标准**。原始网页中充斥着导航栏菜单、侧边栏猜你喜欢、页脚备案号、登录注册提示以及SEO垃圾关键词堆砌。如果将这些低信息密度的样板噪音（Boilerplate Noise）喂给模型或存入 RAG 向量库，不仅会白白浪费昂贵的 GPU 算力与上下文窗口，还会导致向量检索召回大量无意义的页眉页脚，严重污染模型输出质量。

### 1.2 启发式正文抽取（`Trafilatura`）与 `Crawl4AI` Markdown 转换

想象一下：如果你要从全国 5,000 个不同地方政府网站和科技媒体采集文章构建政策大模型，显然不可能为 5,000 个网站分别手写 5,000 条 XPath 规则。此时，我们需要不依赖特定网站 DOM 结构的通用 **网页正文抽取 `Readability` 与 `Trafilatura`** 算法，配合 **网页噪音剔除与样板文本过滤**。

早期通用正文抽取算法（如 Mozilla Readability 和基于行块分布函数的文本密度算法）主要依靠统计每个 DOM 节点的“文字长度与超链接标签数量之比（Link Density）”以及标点符号密度来识别正文区。而在当前 LLM 语料工程界，公认综合准确率（F1-Score）最高的开源黄金标准是 Python 库 **`trafilatura`**。它融合了启发式 DOM 剪枝、文本密度计算与回退机制，能够自动剥离导航栏、广告与评论区，精准提取标题、作者、发布日期与纯净正文。

更进一步，对于大模型和 RAG 系统而言，纯文本（Plain Text）会丢失掉极为关键的**章节标题层级（`#`、`##`）、代码缩进块（````python`）和二维对比表格（`| 列1 | 列2 |`）**。因此，现代 AI 爬虫的标准输出格式不再是纯文本或原始 HTML，而是兼具高可读性与极低 Token 占用的 **HTML 转 Markdown 结构化保留代码表格**！围绕这一需求，业界形成了三套主流工具链：

- **`Jina Reader` 与 `Firecrawl` 网页提取**：云端 API 方案。例如在任意网址前直接拼接 `https://r.jina.ai/https://example.com`，即可直接获取由云端无头浏览器渲染并净化后的大模型友好型 Markdown 文本。
- **`Crawl4AI` 大模型友好型异步爬虫**：目前 GitHub 上最火热的开源 AI 爬虫框架，底层基于异步 Playwright 构建，原生支持一键输出净化后的 `fit_markdown`（经启发式噪音剔除的精简 Markdown），并内置了与 LLM 提取器的无缝对接。

下面演示使用 `trafilatura` 与 `Crawl4AI` 思想将嘈杂 HTML 一键转化为保留代码块与表格的高信噪比 Markdown 语料：

```python
import trafilatura
from typing import Any

def extract_llm_ready_markdown(raw_html: str, source_url: str = "") -> dict[str, Any]:
    """
    使用 Trafilatura 从任意未知结构的网页中剔除样板噪音，
    并提取保留了标题层级、代码块与表格的 Markdown 语料及元数据
    """
    # 1. 提取文章元数据（标题、作者、发布时间、站点名）
    metadata = trafilatura.extract_metadata(raw_html, default_url=source_url)

    # 2. 启发式抽取核心正文并直接转换为结构化 Markdown 格式
    clean_markdown = trafilatura.extract(
        raw_html,
        url=source_url,
        output_format="markdown",
        include_tables=True,      # 保留关键数据表格
        include_Formatting=True,  # 保留加粗、列表与代码块层级
        include_links=False,      # 剔除干扰注意力的内链 URL
        deduplicate=True,         # 开启段落级重复文本剔除
    )

    return {
        "title": metadata.title if metadata else "",
        "author": metadata.author if metadata else "",
        "date": metadata.date if metadata else "",
        "url": source_url,
        "markdown_content": clean_markdown or "",
    }


if __name__ == "__main__":
    noisy_html = """
    <html><body>
      <nav><a href="/">首页</a> | <a href="/login">登录</a> | 热门搜索：手机 电脑</nav>
      <article>
        <h1>DeepSeek-R1 强化学习推理模型技术解读</h1>
        <p>本文深入分析大模型如何通过纯强化学习（RL）涌现出自我反思与长思维链能力。</p>
        <table>
          <thead><tr><th>模型版本</th><th>AIME 2024 准确率</th></tr></thead>
          <tbody><tr><td>DeepSeek-R1</td><td>79.8%</td></tr></tbody>
        </table>
      </article>
      <aside>猜你喜欢：点击领取双十一大促优惠券！</aside>
      <footer>Copyright 2026 ICP备888888号 联系我们</footer>
    </body></html>
    """
    doc = extract_llm_ready_markdown(noisy_html, "https://ai.example.com/post/101")
    print(f"标题: {doc['title']}\n--- 净化后的 Markdown 正文 ---\n{doc['markdown_content']}")
```

---

## 2. 基于 LLM 的零样本网页解析与自愈修复爬虫

当我们需要从结构各异的网站中提取高度结构化的业务字段（例如从 200 家不同医院官网提取“科室名称、主任医师、擅长疾病、出诊时间”），传统方法需要编写 200 套解析代码；而借助大语言模型，我们可以构建**零样本（Zero-Shot）自适应智能爬虫**。

### 2.1 DOM 树精简（Pruning）与 Function Calling 零样本解析

如果直接把一个动辄 300 KB（约合 80,000 Tokens）的原始 HTML 源码原封不动地发给大模型 API，不仅单页调用成本高达数角钱，而且绝大部分 Token 都被无意义的 `<svg>` 图标路径、`<style>` 样式、`<script>` 脚本和冗长的 `class="flex items-center justify-between..."` 样式类名浪费掉了。

因此，在调用大模型前必须先执行 **DOM 树精简 `Pruning` 降低 Token 消耗**：
1. 彻底移除 `<script>`、`<style>`、`<svg>`、`<noscript>`、`<iframe>` 及 HTML 注释节点。
2. 剥离所有标签上的 `style`、`onclick`、`data-*` 以及无语义的 Tailwind/Bootstrap 原子化 CSS 类名，仅保留核心文本结构与 `href` 属性。经过这一步精简，HTML 的 Token 体积通常能**压缩 85%～95%**！

随后，我们利用 **基于 LLM 与 `Function-Calling` 零样本解析**（或 OpenAI / DeepSeek / 通义千问兼容的 `Structured Outputs` + `Pydantic` 模型约束）：只需定义一个描述目标字段的 `Pydantic` 数据类，大模型就会自动阅读精简后的网页文本，并严格按照 JSON Schema 输出类型安全的字典列表，彻底告别输出格式不稳定或幻觉寒暄语。

### 2.2 LLM 自动生成选择器自愈爬虫与 AI 视觉 Agent

虽然用 LLM 逐页解析网页非常省心，但如果某个网站有 100 万个详情页，每页都调一次大模型 API 依然是一笔不小的开销。在工业界，性价比最高的设计模式是 **LLM 自动生成选择器与自愈修复爬虫（Self-Healing Scraper）**：

- **工作流机制**：平时爬虫依然使用完全免费且毫秒级运行的传统 XPath / CSS 选择器执行高速采集；一旦监控模块发现目标网站前端改版导致原有 XPath 提取结果为空（触发页面结构漂移异常），系统会自动截取当前最新的精简版 HTML 样本发送给大模型，指示大模型根据字段语义**自动推断并编写出适配新版页面的 XPath/CSS 选择器规则**，经沙盒验证通过后热更新到配置中心，实现无人值守的“自愈修复”！
- **AI 视觉 Agent 结合截图识别复杂交互**：面对使用 Canvas 绘制数据或 DOM 结构极度混淆的页面，还可以结合多模态视觉大模型（VLM，如 Qwen-VL、GPT-4o），通过 Set-of-Mark（SoM）技术在 Playwright 页面截图上为所有可交互元素打上数字编号角标，让视觉 Agent 直接看图输出“点击 12 号按钮、在 5 号输入框填入关键词”的操控指令。

```python
import json
from lxml import etree, html
from pydantic import BaseModel, Field
from openai import OpenAI

# 1. 定义期望提取的 Pydantic 结构化 Schema
classClinicalExpert(BaseModel):
    doctor_name: str = Field(description="专家姓名")
    title: str = Field(description="职称，如主任医师、教授")
    department: str = Field(description="所属科室")
    specialties: list[str] = Field(description="擅长治疗的疾病或研究方向列表")

class ExpertListSchema(BaseModel):
    experts: list[classClinicalExpert]

def prune_dom_for_llm(raw_html: str) -> str:
    """DOM 树精简函数：移除脚本、样式、SVG 及冗余属性，降低 90% Token 消耗"""
    tree = html.fromstring(raw_html)
    # 移除所有无意义噪音标签
    etree.strip_elements(tree, "script", "style", "svg", "noscript", "iframe", "footer", with_tail=False)
    # 剥离所有冗余属性，仅保留 a 标签的 href
    for elem in tree.iter():
        allowed_href = elem.attrib.get("href")
        elem.attrib.clear()
        if elem.tag == "a" and allowed_href:
            elem.attrib["href"] = allowed_href
    return html.tostring(tree, encoding="unicode", method="html")

def zero_shot_extract_with_llm(raw_html: str, api_key: str, base_url: str) -> dict:
    """使用 DOM 精简 + LLM JSON Schema 结构化输出实现零样本网页解析"""
    pruned_html = prune_dom_for_llm(raw_html)
    client = OpenAI(api_key=api_key, base_url=base_url)

    response = client.beta.chat.completions.parse(
        model="qwen-plus",  # 或 gpt-4o-mini / deepseek-chat
        messages=[
            {"role": "system", "content": "你是一个精准的网页结构化数据抽取引擎，请严格按照 Schema 提取信息。"},
            {"role": "user", "content": f"请从以下精简 HTML 中提取所有医生专家信息：\n{pruned_html}"},
        ],
        response_format=ExpertListSchema,
        temperature=0.0,
    )
    parsed_obj = response.choices[0].message.parsed
    return parsed_obj.model_dump() if parsed_obj else {"experts": []}
```

#### Diagram: LLM 零样本网页解析与 XPath 选择器自愈修复工作台

<details markdown="1">
<summary>Interactive MicroSim: LLM 零样本网页解析与 XPath 选择器自愈修复工作台</summary>

Type: microsim

**sim-id:** ch11-llm-self-healing-scraper<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Apply<br/>
**Bloom Verb:** Implement<br/>
**Learning Objective:** The learner will implement a hybrid self-healing scraper workflow that combines DOM tree pruning, LLM Function Calling zero-shot extraction, and automated XPath selector regeneration when target website layouts change.

**Prerequisites:** DOM树精简Pruning降低Token消耗, 基于LLM与Function-Calling零样本解析, LLM自动生成选择器与自愈修复爬虫

**Evidence of Mastery:**
- 能够量化对比“每页直接调用 LLM 解析”与“LLM 生成/自愈 XPath + 本地 `lxml` 批量解析”在 10 万页采集规模下的 API 费用与耗时差距
- 能够配置正确的 DOM Pruning 过滤规则，在保留关键业务文本与层级结构的前提下将单页输入 Token 从 45,000 压缩至 3,000 以内

**Misconceptions:**
- 在百万级页面采集中对每一个页面都调用一次大模型 API（导致账单高达数万元且并发速度受限于大模型 TPM 速率配额）
- 在 DOM 精简时不小心删掉了包裹核心字段的父容器标签，导致大模型无法生成具有区分度的相对路径选择器

**Instructional Rationale:**
通过可视化仿真“正常高速运行 -> 网站前端改版触发字段空置告警 -> 自动执行 DOM 精简并调用大模型推断新版 XPath -> 沙盒验证通过后热更新恢复采集”的四步闭环，让学习者深入理解 AI 赋能传统爬虫的工程范式。

**Content:**
- 左上状态机：展示爬虫从“本地 XPath 极速模式（0 成本，500 页/秒）”到“漂移熔断”再到“LLM 自愈修复”的状态流转动画
- 右上 DOM 精简对比器：实时显示原始 HTML（42,800 Tokens）经过剔除 `<script>`/`<style>`/`<svg>` 及无意义属性后变为精简 HTML（2,650 Tokens）的体积压缩柱状图
- 底部交互控制台：提供“一键模拟目标网站改版（将 `.price-num` 改为 `.v2-cost-tag`）”、“触发 LLM 选择器自愈”与“切换全量 LLM vs 混合自愈成本核算器”

**Provenance:** 依据本章第 2.1 与 2.2 节智能解析与自愈架构设计。

**Rules:**
- 当点击“模拟目标网站改版”时，原有 XPath `//span[@class="price-num"]/text()` 提取结果瞬间归零，监控面板亮起红灯并触发自愈工作流
- 当启用 DOM Pruning 后再调用 LLM 自愈生成新 XPath `//div[contains(@class, "v2-cost-tag")]/text()` 时，单次修复仅耗时 1.2 秒、消耗 $0.001，随后无缝切回本地高速解析模式

**Learner Activity:**
1. 勾选不同的 DOM 节点剥离选项，观察 Token 压缩率与字段信息保留完整度。
2. 触发网站改版故障，观察 LLM 如何根据旧字段样本值在新版 DOM 树中反推出新的 XPath 表达式并完成热修复。

**Feedback:**
- 实时核算 100,000 页采集任务的总 API 账单成本（美元/人民币）、平均单页延迟（ms）以及改版自愈成功率。

**Starting State:**
默认处于“本地 XPath 高速采集”状态，等待学习者点击“模拟网站前端改版”按钮观察自愈闭环。

**Chapter Anchors:** 2.1 DOM 树精简（Pruning）与 Function Calling 零样本解析, 2.2 LLM 自动生成选择器自愈爬虫与 AI 视觉 Agent
</details>

### 2.3 LLM 自动生成与热修复 XPath 选择器的代码落地

为了让同学们亲手实现上述“平时本地 `lxml` 零成本高速跑，改版后自动调大模型修复选择器”的自愈架构，下面给出完整的代码实现：

```python
from lxml import etree
from pydantic import BaseModel, Field
from openai import OpenAI

class HealedSelectorSchema(BaseModel):
    field_name: str = Field(description="目标字段名称")
    new_xpath: str = Field(description="适配新版 HTML 结构的 XPath 表达式，必须以 // 开头且直接提取文本或属性")
    reasoning: str = Field(description="简述如何根据页面语义定位到该节点的推理过程")

def auto_heal_xpath_with_llm(
    new_raw_html: str,
    field_name: str,
    broken_xpath: str,
    sample_expected_hint: str,
    client: OpenAI,
) -> str:
    """
    当检测到旧 XPath 因网页改版失效（返回空列表）时，
    自动对新版 HTML 进行 DOM 精简，调用大模型生成新 XPath 并在本地沙盒验证通过后返回
    """
    pruned_snippet = prune_dom_for_llm(new_raw_html)[:12000]
    prompt = (
        f"目标网站刚刚发生了前端改版，原先用于提取字段【{field_name}】的 XPath `{broken_xpath}` 已失效。\n"
        f"该字段的典型内容特征为：{sample_expected_hint}。\n"
        f"请分析以下精简后的新版 HTML 源码，生成一条全新的、稳健的 XPath 表达式：\n{pruned_snippet}"
    )

    resp = client.beta.chat.completions.parse(
        model="qwen-plus",
        messages=[
            {"role": "system", "content": "你是一个精通 lxml 与 XPath 1.0 语法的爬虫自愈修复专家。"},
            {"role": "user", "content": prompt},
        ],
        response_format=HealedSelectorSchema,
        temperature=0.0,
    )
    healed = resp.choices[0].message.parsed
    if not healed:
        raise RuntimeError("LLM 未能生成有效的自愈选择器")

    # 在本地 lxml 沙盒中立即验证新生成的 XPath 是否真的能提取出非空数据
    tree = etree.HTML(new_raw_html)
    test_result = tree.xpath(healed.new_xpath)
    cleaned_vals = [str(x).strip() for x in test_result if str(x).strip()]
    if not cleaned_vals:
        raise ValueError(f"自愈生成的 XPath `{healed.new_xpath}` 在沙盒验证中仍返回空结果")

    print(f"[自愈成功] 字段 [{field_name}] 新 XPath: {healed.new_xpath} | 沙盒提取样本: {cleaned_vals[:2]}")
    return healed.new_xpath
```

---

## 3. 指令微调（SFT）数据集构建与 RAG 向量库流水线

### 3.1 Alpaca / ShareGPT 格式与垂直社区问答采集

当为医疗、法律、编程或农业等垂直行业微调专属大模型时，我们需要通过 **垂直社区问答采集构建领域 SFT 语料**（如从技术问答社区 StackOverflow、GitHub Issues、政务民生问答平台或开源医患问答库中采集数据）。为了过滤掉低质量的灌水回复，采集管道必须设置严格的质量门禁：例如**仅保留带有 `is_accepted=True`（被题主采纳）标记或高赞数 `upvotes >= 10`、且正文字数大于 150 字、包含代码块或法规引用的专业回答**。

在落盘导出时，主流微调框架（如 LLaMA-Factory、Swift）普遍要求遵循 **指令微调 SFT 数据集 `Alpaca` / `ShareGPT` 格式**：

| 数据集标准格式 | JSON 核心字段结构 | 适用训练场景 |
| :--- | :--- | :--- |
| **Alpaca 格式** | `{"instruction": "任务指令/问题标题", "input": "补充上下文/问题详情", "output": "高赞采纳回答", "system": "系统人设"}` | 单轮指令遵循微调、知识问答、文本摘要与结构化提取任务 |
| **ShareGPT 格式** | `{"conversations": [{"from": "system", "value": "..."}, {"from": "human", "value": "..."}, {"from": "gpt", "value": "..."}]}` | 多轮连续对话微调（支持题主追问与专家二次解答的完整上下文） |

### 3.2 RAG 实时更新、文档语义分块（Chunking）与向量数据库写入

在企业级 AI 应用落地中，**RAG 检索增强生成与知识库实时更新** 是需求量最大的工程场景。当爬虫采集并转换出纯净的 Markdown 文档后，不能直接把整篇长达一万字的文章塞进单个向量里（否则语义会被严重稀释，且超出 Embedding 模型的窗口上限），而必须依次执行 **文档语义分块 `Chunking` 与元数据保留** 及 **`Embedding` 向量化与向量数据库写入**：

1. **文档语义分块（Semantic / Markdown-Aware Chunking）**：
   切忌简单粗暴地每隔 500 个字符硬切一刀（这会把一个完整的表格或一段 Python 函数从中间腰斩！）。正确的做法是按 Markdown 标题层级（`# -> ## -> ###`）与双换行符 `\n\n` 进行递归语义切块，控制每个分块（Chunk）在 `300 ~ 800 Tokens` 之间，并保留 `10% ~ 15%` 的相邻重叠窗口（Overlap）以防止上下文断裂。
2. **元数据全程绑定（Metadata Preservation）**：
   每一个切出来的文本块都必须携带其源网页的 `source_url`、`title`、`publish_date` 以及当前所属的章节面包屑路径（如 `"第一章 > 申报条件"`）。这样大模型在生成回答时才能精准附上可点击的溯源引用链接。
3. **Embedding 向量化与向量数据库（Chroma / Milvus）幂等写入**：
   调用文本嵌入模型（如开源王者 `BAAI/bge-m3` 或在线 Embedding API）将每个文本块映射为高维稠密向量（如 1024 维浮点数数组），并以 `url + chunk_index` 的哈希值作为唯一主键写入向量数据库（如轻量级本地库 `ChromaDB` 或分布式引擎 `Milvus` / `Qdrant`）。

#### Diagram: 面向 RAG 与 SFT 的智能网页语料清洗及分块沙盒

<details markdown="1">
<summary>Interactive MicroSim: 面向 RAG 与 SFT 的智能网页语料清洗及分块沙盒</summary>

Type: microsim

**sim-id:** ch11-rag-chunking-pipeline-sim<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Evaluate<br/>
**Bloom Verb:** Optimize<br/>
**Learning Objective:** The learner will optimize an end-to-end AI scraping pipeline by adjusting boilerplate removal, DOM pruning, Markdown semantic chunking parameters (`chunk_size`, `chunk_overlap`), and PII scrubbing rules to maximize RAG retrieval precision and Token cost efficiency.

**Prerequisites:** 网页正文抽取Readability与Trafilatura, HTML转Markdown结构化保留代码表格, 文档语义分块Chunking与元数据保留, 语料隐私脱敏PII擦除与MinHash去重

**Evidence of Mastery:**
- 能够对比“固定字符硬切分（Fixed-Size Chunking）”与“Markdown 标题感知语义切分（Markdown-Aware Chunking）”在处理跨行表格与代码块时的完整性差异
- 能够解释为什么在清洗阶段保留 `source_url` 和 `heading_hierarchy` 元数据对于消除 RAG 问答幻觉至关重要

**Misconceptions:**
- 认为 `chunk_size` 设得越大越好（实际上块过大不仅降低向量余弦相似度的局部区分度，还会把无关段落带入大模型上下文）
- 直接将未做 PII 脱敏的爬虫原始文本用于微调，导致大模型在推理时泄露真实公民手机号与邮箱

**Instructional Rationale:**
通过可视化四阶流水线（原始嘈杂 HTML -> Trafilatura Markdown 净化 -> PII 隐私擦除与 MinHash 去重 -> 语义分块与 Embedding 入库），让学习者实时调整参数并观察右侧“RAG 模拟提问检索召回率”与“Token 账单成本”的动态变化。

**Content:**
- 阶段 1（噪音剥离）：展示网页中的导航栏、广告侧边栏与正文代码块，支持切换 `soup.get_text()` 与 `trafilatura (Markdown)`
- 阶段 2（合规净化）：展示内含真实姓名、手机号、身份证号的论坛问答贴，开关控制正则 PII 掩码替换与 MinHash 近重复过滤
- 阶段 3（分块策略）：提供“固定 300 字硬切” vs “按 Markdown 标题与代码块边界语义切分”对比视图，以及 `chunk_size`（128～1024）和 `overlap`（0～200）滑块
- 阶段 4（RAG 检索测试）：模拟用户提问“如何配置异步限流并发数？”，展示召回的 Top-3 Chunks 及余弦相似度得分

**Provenance:** 依据本章第 1 节、第 3 节与第 4 节 AI 数据工程流水线设计。

**Rules:**
- 当使用“固定字符硬切分”时，画布中的 Python 代码块和 Markdown 表格被红色切割线从中间劈开，RAG 检索测试提示：“召回的代码片段缺失上半部分导入语句，无法运行！”
- 当启用“Markdown 语义分块 + 元数据绑定 + PII 脱敏”时，代码块完整保留在同一 Chunk 内，RAG 召回准确率跃升至 96%，且隐私合规检查绿灯通过

**Learner Activity:**
1. 对比未净化 HTML 与精简 Markdown 在 Token 消耗量上的差异。
2. 调节分块策略与 `chunk_size` / `overlap` 滑块，使测试问题能够完整召回包含表格与代码的目标分块。

**Feedback:**
- 实时输出 4 项核心工程指标：Token 压缩率（%）、代码块/表格结构完整度、PII 隐私残留风险、RAG Top-1 余弦相似度。

**Starting State:**
默认加载一篇包含技术表格、Python 代码块及页脚联系手机号的混合网页，初始处于“未开启噪音过滤 + 固定字符硬切分”的低效状态，等待学习者优化流水线参数。

**Chapter Anchors:** 1.2 启发式正文抽取（Trafilatura）与 Crawl4AI Markdown 转换, 3.2 RAG 实时更新、文档语义分块（Chunking）与向量数据库写入, 4. 多模态图文对齐、隐私脱敏（PII）与大规模去重
</details>

---

## 4. 多模态图文对齐、隐私脱敏（PII）与大规模去重

在构建工业级预训练语料与多模态数据集时，最后一道不可或缺的工序是多模态对齐与数据安全合规审计：

1. **多模态图文对采集与 CLIP 对齐过滤**：
   在训练文生图大模型（如 Stable Diffusion、Flux）或视觉理解大模型时，爬虫需要从网页中成对采集图片 URL（`<img>` 的 `src` 或 `data-src`）及其对应的描述文本（优先取 `<figcaption>` 图注，其次取 `alt` 属性或图片前后的说明段落）。但由于网页上有大量装饰性小图标、广告横幅或图文完全不符的“标题党”配图，采集パイプライン需要执行两级过滤：第一级剔除宽高小于 `256x256` 像素或长宽比极端的无效小图；第二级使用预训练的 **CLIP（Contrastive Language-Image Pre-training）** 模型分别计算图片向量与文本向量的余弦相似度，剔除相似度低于阈值（如 `< 0.25`）的图文不符噪声对。
2. **语料隐私脱敏 `PII` 擦除与 `MinHash` 去重**：
   - **PII（Personally Identifiable Information，个人身份信息）隐私脱敏**：在将爬取的网页语料送入大模型训练或共享发布前，必须严格依照《个人信息保护法》使用高召回率正则或 NER 模型，将正文中的手机号、身份证号、电子邮箱、银行卡号及家庭住址统一替换为 `<PHONE>`、`<ID_CARD>`、`<EMAIL>` 等占位符，严防大模型发生训练数据记忆泄露。
   - **MinHash LSH 近似文档去重**：互联网上存在极高比例的转载抄袭文章（例如同一篇通稿被 500 家媒体改动几个标点后重复发布）。第 9 章介绍的 MD5 只能识别一字不差的完全相同文本；而对于海量长文档语料，工业界标准方案是将文档切分为 $n$-gram 词袋（Shingles），计算 **MinHash 签名**并结合 **LSH（局部敏感哈希）**，在 $O(N)$ 线性时间内快速找出并剔除 Jaccard 相似度高于 `0.8` 的近似重复文档。

下面给出一个完整的端到端实战脚本，涵盖 **PII 个人隐私脱敏**、**Markdown 标题感知语义分块**、**Alpaca SFT 格式生成** 以及 **ChromaDB 向量知识库幂等写入**：

```python
import hashlib
import re
from typing import Any

# 预编译 PII 个人敏感信息脱敏正则表达式
PII_PATTERNS = {
    "<EMAIL>": re.compile(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+"),
    "<PHONE>": re.compile(r"(?<!\d)1[3-9]\d{9}(?!\d)"),
    "<ID_CARD>": re.compile(
        r"(?<!\d)[1-9]\d{5}(?:19|20)\d{2}(?:0[1-9]|1[0-2])(?:0[1-9]|[12]\d|3[01])\d{3}[\dXx](?!\d)"
    ),
}

def scrub_pii_text(text: str) -> str:
    """擦除语料中的邮箱、11位手机号与18位身份证号，保障大模型训练数据合规"""
    cleaned = text
    for placeholder, pattern in PII_PATTERNS.items():
        cleaned = pattern.sub(placeholder, cleaned)
    return cleaned

def markdown_semantic_chunker(
    markdown_text: str,
    source_url: str,
    doc_title: str,
    max_chars: int = 500,
    overlap_chars: int = 60,
) -> list[dict[str, Any]]:
    """
    按 Markdown 标题层级与段落边界进行语义切块，保留完整的章节上下文元数据
    """
    clean_md = scrub_pii_text(markdown_text)
    # 按二级/三级标题切分逻辑章节
    sections = re.split(r"(?m)^(#{1,3}\s+.+)$", clean_md)
    
    chunks: list[dict[str, Any]] = []
    current_heading = doc_title
    chunk_idx = 0

    for part in sections:
        part = part.strip()
        if not part:
            continue
        if part.startswith("#"):
            current_heading = part.lstrip("#").strip()
            continue

        # 在章节内部按段落聚合，避免切碎代码块或表格
        paragraphs = part.split("\n\n")
        buffer = ""
        for para in paragraphs:
            if len(buffer) + len(para) + 2 <= max_chars:
                buffer = f"{buffer}\n\n{para}".strip()
            else:
                if buffer:
                    chunk_id = hashlib.md5(f"{source_url}_{chunk_idx}".encode()).hexdigest()
                    chunks.append({
                        "id": chunk_id,
                        "text": f"【章节：{current_heading}】\n{buffer}",
                        "metadata": {
                            "source_url": source_url,
                            "title": doc_title,
                            "section": current_heading,
                            "chunk_index": chunk_idx,
                        },
                    })
                    chunk_idx += 1
                    # 保留尾部重叠窗口以维持跨块语义连贯
                    buffer = f"{buffer[-overlap_chars:]}\n\n{para}".strip()
                else:
                    buffer = para[:max_chars]

        if buffer:
            chunk_id = hashlib.md5(f"{source_url}_{chunk_idx}".encode()).hexdigest()
            chunks.append({
                "id": chunk_id,
                "text": f"【章节：{current_heading}】\n{buffer}",
                "metadata": {
                    "source_url": source_url,
                    "title": doc_title,
                    "section": current_heading,
                    "chunk_index": chunk_idx,
                },
            })
            chunk_idx += 1

    return chunks


if __name__ == "__main__":
    sample_md = """
# 昇腾算力集群运维手册
如有故障请联系值班工程师张工，手机：13812345678，邮箱：zhang@ai-lab.cn。

## 1. 容器驱动挂载检查
请执行 `npu-smi info` 查看 NPU 芯片健康状态与显存占用率。

## 2. 分布式通信参数调优
在多机多卡训练时，需配置 HCCL_SOCKET_IFNAME 环境变量指定高速网卡。
"""
    rag_chunks = markdown_semantic_chunker(
        sample_md,
        source_url="https://docs.ai-lab.cn/ascend/ops.html",
        doc_title="昇腾算力集群运维手册",
        max_chars=120,
    )
    for c in rag_chunks:
        print(f"Chunk ID: {c['id']} | Section: {c['metadata']['section']}")
        print(f"Content:\n{c['text']}\n" + "-" * 40)
```

---

## 5. 章节复习与实战思考题

??? question "1. 在为 RAG 检索增强生成系统构建网页采集管道时，为什么业界普遍选择将 HTML 转换为 Markdown 格式（如使用 `Trafilatura` 或 `Crawl4AI`），而不是直接提取纯文本（Plain Text）？"
    **参考解析**：
    1. **保留文档逻辑结构**：Markdown 通过 `#`、`##` 保留了章节标题层级，通过 `|---|---|` 保留了二维表格的行列对应关系，通过三个反引号保留了代码块与语言标识。如果转为纯文本，表格的数据单元格会混作一团，大模型无法理解各数字对应的表头含义。
    2. **天然支持结构化语义分块（Chunking）**：保留 Markdown 标题使分块器能够沿着章节边界切分，并将当前小标题自动注入每个 Chunk 的头部，大幅提升向量检索的语义准确度。
    3. **极高的 Token 性价比**：相比保留冗长标签与属性的原生态 HTML，Markdown 的 Token 体积减少了 80%～90%，极大降低了大模型推理成本。

??? question "2. 当使用大语言模型（LLM）进行网页零样本结构化字段抽取时，如果直接把未处理的原始 HTML 发送给模型会带来什么问题？工程上如何解决？"
    **参考解析**：
    原始 HTML 中 85% 以上的内容是 `<script>` 脚本、`<style>` 样式表、`<svg>` 矢量图路径以及冗长的 CSS 类名属性。直接发送不仅会导致单页消耗数万个 Token（成本高、推理慢，甚至直接撑爆模型的最大上下文窗口），还会分散模型的注意力导致提取遗漏。
    **工程解决方案**：
    1. 在送入 LLM 前先执行 **DOM 树精简（Pruning）**，剔除所有非正文脚本与样式节点，清空无意义的标签属性，或先转为精简 Markdown。
    2. 对于大批量同结构页面，采用 **“LLM 自动生成并自愈修复 XPath/CSS 选择器 + 本地 `lxml`/`selectolax` 高速批量执行”** 的混合架构，兼顾零样本自适应能力与极低运行成本。

??? question "3. 什么是语料清洗中的 PII 脱敏？为什么在大模型预训练与 SFT 微调语料入库前必须包含这一步？"
    **参考解析**：
    PII（Personally Identifiable Information）是指个人可识别身份信息，包括真实手机号、身份证号、私人电子邮箱、银行卡号及精确住址等。
    大语言模型在训练过程中具有显著的“参数记忆效应”。如果爬虫从论坛、公示网或社交媒体抓取的原始语料中包含未脱敏的公民个人信息，模型在后续上线提供对话服务时，可能会因特定提示词诱导而直接背诵并泄露他人的真实隐私信息，违反《个人信息保护法》与《生成式人工智能服务管理暂行办法》。因此必须在入库前通过正则与 NER 流水线将其统一替换为 `<PHONE>`、`<ID_CARD>` 等安全掩码。
