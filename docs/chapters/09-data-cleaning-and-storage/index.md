---
title: "数据清洗流水线与多源持久化存储"
description: "本章系统讲解面向人工智能训练语料的数据治理与存储工程：涵盖HTML残留剥离、空白与全半角归一化、日期数值标准化、Pydantic强类型校验、MD5与SimHash近似去重，以及CSV、JSONL、Parquet、SQLite/MySQL、MongoDB、Motor/aiomysql异步入库和Redis布隆过滤器亿级去重。"
generated_by: claude skill chapter-content-generator
date: "2026-10-20 14:32:00"
version: 1.11
---

# 数据清洗流水线与多源持久化存储

## Summary

“垃圾进，垃圾出（Garbage In, Garbage Out）”是人工智能数据工程的铁律。本章聚焦采集后的数据治理与持久化落地，系统讲解 HTML 残留剥离、空白字符与全半角归一化、日期数值标准化，以及使用 `Pydantic` 进行数据结构强校验与 MD5/SimHash 指纹去重。在存储层，全面覆盖 UTF-8-SIG 防乱码 CSV、大模型标准 `JSON Lines (.jsonl)`、`Parquet` 列式存储、`SQLite`/`MySQL` 参数化防注入与幂等 `UPSERT`、`MongoDB` 文档存储、`Motor`/`aiomysql` 异步入库，以及基于 `Redis` Set 与布隆过滤器（Bloom Filter）的亿级 URL 去重。

## Concepts Covered

This chapter covers the following 20 concepts from the learning graph:

| Concept | Concept Impact Score |
|---------|-----------------------|
| 原始网页脏数据与ETL清洗流水线 | 198 |
| HTML残留标签剥离与实体字符还原 | 45 |
| 空白字符归一化与全半角转换 | 21 |
| 日期时间标准化与数值单位转换 | 38 |
| Pydantic数据模型与字段类型强校验 | 37 |
| 内容指纹MD5/SHA256与SimHash去重 | 20 |
| CSV文件读写与UTF-8-SIG防乱码 | 1 |
| JSON与JSON-Lines流式行存储格式 | 14 |
| Parquet列式存储与Pandas批量导出 | 3 |
| SQLite轻量级嵌入式数据库存储 | 8 |
| MySQL表结构设计与utf8mb4字符集 | 7 |
| PyMySQL参数化防注入与批量写入 | 6 |
| 唯一索引约束与UPSERT幂等更新 | 4 |
| SQLAlchemy与数据库连接池高并发 | 1 |
| MongoDB文档数据库半结构化存储 | 6 |
| PyMongo集合操作与upsert幂等去重 | 3 |
| Motor与aiomysql异步数据库写入 | 2 |
| Redis内存数据库核心数据结构 | 10 |
| Redis集合Set与布隆过滤器URL去重 | 5 |
| 多模态文件哈希命名与对象存储归档 | 4 |

## Prerequisites

This chapter builds on concepts from:

- [Chapter 1: 网络爬虫导论与数据合规基石](../01-intro-and-ethics/index.md)
- [Chapter 3: Python网络请求核心编程](../03-python-http-requests/index.md)
- [Chapter 4: 静态网页结构化解析与正则提取](../04-static-html-parsing/index.md)
- [Chapter 5: 异步加载与Web接口逆向采集](../05-ajax-and-api-scraping/index.md)
- [Chapter 7: 反爬虫机制原理与工程对抗策略](../07-anti-scraping-and-countermeasures/index.md)
- [Chapter 8: 高性能并发与异步协程爬虫编程](../08-concurrent-and-async-scraping/index.md)

---

## 1. 网页脏数据治理与 ETL 清洗流水线

在人工智能与大语言模型（LLM）训练领域，有一条颠扑不破的公理：**“垃圾进，垃圾出（Garbage In, Garbage Out）”**。很多初学者认为，只要爬虫把网页上的文字提取出来放进字典里，采集任务就大功告成了。然而，如果你直接把刚从网页 DOM 树或 JSON 接口里抓取出来的原始数据喂给大模型做微调（SFT）或存入业务数据库，往往会引发灾难性后果：模型输出夹杂着奇怪的 `<div>` 标签和 `&nbsp;` 乱码、价格统计因为混入了 `"￥1,299.00元/台"` 字符串而直接报错崩溃、或者因为同一篇通稿被转载了 500 次而导致模型产生严重的过拟合复读机现象。

### 1.1 原始网页脏数据与ETL清洗流水线

直接从互联网页面采集回来的未经处理的原始数据被称为**原始网页脏数据（Dirty Web Data）**。在真实工程环境中，网页脏数据主要表现为六大典型形态：

1. **HTML 噪音残留**：正文中混有 `<p>`、`<br/>`、`<script>` 内联脚本、`<style>` 样式代码块，以及 `&amp;`、`&quot;`、`&#39;` 等 HTML 实体转义符。
2. **空白与不可见控制字符污染**：包含不间断空格 `\u00a0`（即 `&nbsp;`）、零宽空格 `\u200b`（常被网站用来做反爬暗桩水印）、连续多个换行符 `\r\n\t` 等。
3. **编码与全半角混杂**：中文语料中混有全角英文字母与全角数字（如 `ＡＩ大模型２０２６`），导致分词器（Tokenizer）将其切分为生僻词 Token。
4. **异构单位与非标时间格式**：同一字段在不同页面表现为 `"3小时前"`、`"昨天 14:30"`、`"2026年10月20日"`，或阅读量显示为 `"3.5万"`、`"12.8k"`、`"850"`。
5. **字段缺失与类型越界**：必填的正文字段为空字符串 `""`，或商品价格出现负数 `-99.0`。
6. **高度冗余与近似重复**：大量完全相同的镜像网页，或仅改动了标题前缀和底部广告尾巴的“洗稿/转载”文章。

为了系统化消灭这六类脏数据，工业界普遍采用 **ETL 清洗流水线（Extract-Transform-Load Pipeline）** 架构：

- **E（Extract，提取）**：由第 4~8 章编写的爬虫解析器从 HTML/API 中抽取原始字典 `Raw Item`；
- **T（Transform，转换与清洗）**：依次经过**标签剥离 $\rightarrow$ 字符归一化 $\rightarrow$ 日期数值标准化 $\rightarrow$ Pydantic 模式强校验 $\rightarrow$ 内容指纹去重**五道标准工序；
- **L（Load，加载入库）**：将 100% 干净、合规、唯一的标准化记录写入文件或数据库。

### 1.2 HTML残留标签剥离与实体字符还原

在 ETL 转换层的第一站，我们需要执行 **HTML残留标签剥离与实体字符还原（HTML Tag Stripping & Entity Unescaping）**。很多同学喜欢直接用简单的正则表达式 `re.sub(r"<[^>]+>", "", html_str)` 去标签，但这存在两个严重缺陷：一是如果直接删掉 `<script>var a=1;</script>` 的尖括号，里面的 JS 代码文本 `var a=1;` 会残留混入正文中；二是标签删掉后，原本靠 `<p>` 或 `<br>` 分隔的句子会粘连在一起。

正确的工业级做法分为两步：

1. **先彻底移除 `<script>` 与 `<style>` 整块节点**，并将 `<br>`、`</p>`、`</div>` 等块级闭合标签替换为换行符 `\n`，再剥离剩余标签；
2. **调用 Python 标准库 `html.unescape(text)` 还原 HTML 实体字符**：将 `&nbsp;` 还原为标准不间断空格 `\xa0`，将 `&lt;`、`&gt;`、`&amp;`、`&quot;` 还原为 `<`、`>`、`&`、`"`。

### 1.3 空白字符归一化与全半角转换

完成 HTML 还原后，文本中往往还残留着 `\u00a0`（不间断空格）、`\u200b`（零宽空格）和全角字符。此时进入第二站：**空白字符归一化与全半角转换（Whitespace Normalization & Unicode NFKC Full-width/Half-width Conversion）**。

这里向大家介绍一个 Python 标准库 `unicodedata` 中堪称“文本清洗神器”的函数：**`unicodedata.normalize("NFKC", text)`**。Unicode 的 **NFKC（Normalization Form Compatibility Composition，兼容性分解后再标准合成）** 能够自动完成两件原本需要写几十行映射表的繁琐工作：

- **自动全角转半角**：将全角字母 `ＡＢＣ`、全角数字 `１２３`、全角百分号 `％` 自动转换为标准 ASCII 半角 `ABC`、`123`、`%`（同时保留正常的中文标点如 `，。！？` 不受破坏）；
- **自动转换特殊兼容空白与上下标**：将不间断空格 `\u00a0`、全角空格 `\u3000` 自动转换为普通半角空格 `" "`，将连体字 `ﬁ` 拆分为 `fi`。

随后，再配合正则表达式剔除零宽字符 `[\u200b-\u200d\ufeff]` 并合并多余空白，就能得到极其干净的纯文本。

### 1.4 日期时间标准化与数值单位转换

在第三站 **日期时间标准化与数值单位转换（Datetime Standardization & Numeric Unit Conversion）** 中，我们的目标是把人类阅读的自然语言字符串转化为机器可排序、可聚合计算的**标准 ISO-8601 时间字符串（`YYYY-MM-DD HH:MM:SS`）**或 **UTC 时间戳**，以及纯粹的 **`float` / `int` 数值**：

- **相对时间与中文日期归一化**：针对新闻社交页面常见的 `"15分钟前"`、`"3小时前"`、`"昨天"`，以当前采集时间 `datetime.now()` 为基准减去对应的 `timedelta`；针对 `"2026年10月20日 14时05分"`，通过正则提取年月日时分秒后构造标准 `datetime` 对象。
- **中文与英文数量级单位换算**：建立单位乘数映射表（如 `"亿": 1e8`、`"万": 1e4`、`"w": 1e4`、`"k": 1e3`），利用正则分离数值主体与单位后缀并相乘，将 `"3.5万"` 精准转化为整数 `35000`。

下面这段完整可运行的代码，将上述 **1.2~1.4 节的全部清洗算子**封装为一套标准函数库，并演示如何把一条面目全非的原始爬虫脏数据清洗为干净整洁的结构化字典：

```python
import html
import re
import unicodedata
from datetime import datetime, timedelta


def clean_html_and_text(raw_html: str) -> str:
    """步骤1 & 2：剥离 HTML 残留标签、还原实体字符、执行 NFKC 全半角与空白归一化"""
    if not raw_html:
        return ""
    # 1. 彻底移除 <script> 和 <style> 标签及其内部代码内容
    text = re.sub(r"(?is)<(script|style).*?>.*?</\1>", "", raw_html)
    # 2. 将换行类块级标签替换为换行符，防止段落粘连
    text = re.sub(r"(?i)<(br\s*/?|/p|/div|/li|/tr)>", "\n", text)
    # 3. 剥离所有剩余的 HTML 标签
    text = re.sub(r"<[^>]+>", "", text)
    # 4. 还原 HTML 实体字符 (如 &amp; -> &, &quot; -> ", &nbsp; -> \xa0)
    text = html.unescape(text)
    # 5. Unicode NFKC 归一化：全角字母数字转半角，\xa0 与 \u3000 自动转普通空格
    text = unicodedata.normalize("NFKC", text)
    # 6. 剔除零宽空格等隐形反爬控制字符
    text = re.sub(r"[\u200b-\u200d\ufeff]", "", text)
    # 7. 将连续水平空白合并为单个空格，合并多余空行
    text = re.sub(r"[ \t]+", " ", text)
    lines = [line.strip() for line in text.splitlines() if line.strip()]
    return "\n".join(lines)


def parse_chinese_number(raw_val: str | int | float) -> float:
    """步骤3a：将带有 '万/亿/w/k/元/人' 及千分位逗号的字符串转换为标准 float"""
    if isinstance(raw_val, (int, float)):
        return float(raw_val)
    cleaned = unicodedata.normalize("NFKC", str(raw_val)).replace(",", "").strip()
    unit_map = {"亿": 1e8, "万": 1e4, "w": 1e4, "W": 1e4, "k": 1e3, "K": 1e3}
    multiplier = 1.0
    for unit_char, factor in unit_map.items():
        if unit_char in cleaned:
            multiplier = factor
            break
    match = re.search(r"[-+]?\d+(?:\.\d+)?", cleaned)
    if not match:
        raise ValueError(f"无法从字符串中提取有效数值: {raw_val!r}")
    return round(float(match.group(0)) * multiplier, 2)


def normalize_datetime(raw_dt: str, base_time: datetime | None = None) -> str:
    """步骤3b：将相对时间 ('3小时前') 或中文日期统一转换为 'YYYY-MM-DD HH:MM:SS'"""
    now = base_time or datetime.now()
    s = unicodedata.normalize("NFKC", raw_dt).strip()

    # 匹配相对时间：X分钟前 / X小时前 / X天前
    if m := re.search(r"(\d+)\s*分钟前", s):
        dt = now - timedelta(minutes=int(m.group(1)))
        return dt.strftime("%Y-%m-%d %H:%M:%S")
    if m := re.search(r"(\d+)\s*小时前", s):
        dt = now - timedelta(hours=int(m.group(1)))
        return dt.strftime("%Y-%m-%d %H:%M:%S")
    if m := re.search(r"(\d+)\s*天前", s):
        dt = now - timedelta(days=int(m.group(1)))
        return dt.strftime("%Y-%m-%d %H:%M:%S")

    # 匹配年月日格式：2026年10月20日 14:30 或 2026-10-20 14:30:00
    s_std = re.sub(r"[年月/]", "-", s).replace("日", " ").replace("时", ":").replace("分", "")
    s_std = re.sub(r"\s+", " ", s_std).strip()
    for fmt in ("%Y-%m-%d %H:%M:%S", "%Y-%m-%d %H:%M", "%Y-%m-%d"):
        try:
            dt = datetime.strptime(s_std, fmt)
            return dt.strftime("%Y-%m-%d %H:%M:%S")
        except ValueError:
            continue
    raise ValueError(f"不支持的日期时间格式: {raw_dt!r}")


if __name__ == "__main__":
    dirty_sample = {
        "raw_title": "  <h1>【重磅】ＡＩ大模型\u200b２０２６训练指南 &amp; 实战！</h1><script>alert(1)</script> ",
        "raw_views": "阅读量：3.5万+",
        "raw_price": "￥1,299.50元",
        "raw_pub_time": "3小时前",
    }
    anchor_now = datetime(2026, 10, 20, 15, 0, 0)
    print("清洗后标题:", clean_html_and_text(dirty_sample["raw_title"]))
    print("清洗后阅读量:", int(parse_chinese_number(dirty_sample["raw_views"])))
    print("清洗后价格:", parse_chinese_number(dirty_sample["raw_price"]))
    print("标准化时间:", normalize_datetime(dirty_sample["raw_pub_time"], base_time=anchor_now))
```

---

## 2. 数据结构强校验与内容指纹去重

### 2.1 Pydantic数据模型与字段类型强校验

即使我们编写了清洗函数，面对目标网站随时可能改版的页面结构，仍然可能出现“标题抓成了空字符串”、“网址拼写残缺”或“价格转换失败”等异常情况。如果用几十个 `if...else...` 去逐一检查字典里的键值，代码会变得极其臃肿且难以维护。

在现代 Python 数据工程与大模型开发中，工业界统一使用 **`Pydantic`数据模型与字段类型强校验**（安装命令：`pip install pydantic`，当前主流为基于 Rust 内核加速的 Pydantic V2）作为 ETL 管道的“质检守门员”。通过继承 `pydantic.BaseModel`：

1. **声明式类型约束与范围检查**：利用 Python 类型注解（Type Hints）配合 `Field(min_length=..., ge=0)`，声明每个字段的合法边界；
2. **自定义前置/后置校验器（`@field_validator`）**：在数据入模时自动触发上述 HTML 剥离、数值换算与业务规则校验；
3. **快速失败与结构化报错（`ValidationError`）**：任何不符合契约的脏数据都会被精准拦截并给出清晰的字段级错误报告，绝不让一条残次品混入下游数据库！

### 2.2 内容指纹MD5/SHA256与SimHash去重

通过了 Pydantic 质检的数据，在入库前还要过最后一关：**内容指纹MD5/SHA256与SimHash去重**。在爬虫工程中，去重分为两个层次：

1. **精确去重（Exact Deduplication —— 基于 MD5 / SHA-256 密码学哈希）**  
   针对 URL 地址或经过归一化清洗后的正文全文，使用 `hashlib.md5(text.encode("utf-8")).hexdigest()` 或 `hashlib.sha256` 计算出固定长度（MD5 为 128 位 / 32 个十六进制字符，SHA-256 为 256 位 / 64 个十六进制字符）的唯一摘要指纹。
   - **特点**：具备极强的“雪崩效应”——哪怕两篇文章只相差一个标点符号，算出的 MD5 值也会截然不同。因此它非常适合用来对**规范化后的 URL** 或 **一字不差的完全相同文本** 进行精确判重。
2. **近似去重（Near-Duplicate Detection —— 基于局部敏感哈希 SimHash 与汉明距离）**  
   在构建大模型预训练语料库时，互联网上存在大量“改头换面”的转载文章：两篇 2,000 字的新闻报道，正文 98% 完全一样，仅仅是末尾多了一句“责任编辑：张三”或替换了几个同义词。此时它们的 MD5 完全不同，精确去重彻底失效！为此，Google 提出了著名的 **SimHash 算法**：
   - **算法四步曲**：
     1. **分词与权重提取（Tokenize & Weight）**：将文档切分为特征词序列（或字符 $n\text{-gram}$ 切片），并为每个词赋予权重 $w_i$（如词频 TF 或 TF-IDF）；
     2. **计算词哈希（Hash）**：将每个特征词映射为一个固定 $b$ 位（通常 $b=64$）的传统二进制哈希串；
     3. **加权累加向量（Weighted Vector）**：初始化一个长度为 $b$ 的全零数组 $V$。遍历每个词的 $b$ 位二进制哈希：若第 $k$ 位为 `1`，则 $V[k] += w_i$；若第 $k$ 位为 `0`，则 $V[k] -= w_i$；
     4. **降维生成签名（Sign Compression）**：遍历累加完成后的数组 $V$，若 $V[k] > 0$ 则最终 SimHash 签名的第 $k$ 位置为 `1`，否则置为 `0`。
   - **汉明距离判定（Hamming Distance）**：两个 64 位整数签名 $S_1$ 与 $S_2$ 之间二进制位不同的个数称为**汉明距离**，可通过位异或操作极速计算：
     \[
     d_{\text{Hamming}}(S_1, S_2) = \text{popcount}(S_1 \oplus S_2)
     \]
     在 64 位 SimHash 标准工业实践中，**当两篇长文档的汉明距离 $d_{\text{Hamming}}(S_1, S_2) \le 3$ 时，即可判定二者为近似重复文档（Near-Duplicates）并予以过滤！**

下面这段代码完整演示了如何使用 **Pydantic V2 `BaseModel`** 构建自动化清洗校验模型，并手写一个零第三方依赖的 **64 位 SimHash 近似去重器**：

```python
import hashlib
import re
from pydantic import BaseModel, Field, ValidationError, field_validator


# 1. 使用 Pydantic V2 定义强类型语料清洗校验模型
class ArticleCorpusItem(BaseModel):
    url: str = Field(..., pattern=r"^https?://[^\s/$.?#].[^\s]*$")
    title: str = Field(..., min_length=3, max_length=200)
    content: str = Field(..., min_length=20)
    views: int = Field(default=0, ge=0)
    publish_time: str

    @field_validator("title", "content", mode="before")
    @classmethod
    def auto_strip_html(cls, v: str) -> str:
        """入模前自动触发 HTML 剥离与 NFKC 归一化"""
        return clean_html_and_text(str(v))

    @field_validator("views", mode="before")
    @classmethod
    def auto_convert_views(cls, v: str | int | float) -> int:
        return int(parse_chinese_number(v))


# 2. 实现 64 位 SimHash 算法与汉明距离近似去重
def compute_simhash_64(text: str) -> int:
    """基于字符 2-gram (Bigram) 特征计算文本的 64 位 SimHash 整数签名"""
    cleaned = re.sub(r"\s+", "", text)
    # 提取字符 2-gram 特征并统计词频权重
    features: dict[str, int] = {}
    for i in range(max(1, len(cleaned) - 1)):
        gram = cleaned[i : i + 2]
        features[gram] = features.get(gram, 0) + 1

    v = [0] * 64
    for token, weight in features.items():
        # 取 MD5 的前 8 个字节 (64 bits) 作为该特征词的基础哈希
        digest = hashlib.md5(token.encode("utf-8")).digest()[:8]
        token_hash = int.from_bytes(digest, byteorder="big")
        for bit_idx in range(64):
            if (token_hash >> bit_idx) & 1:
                v[bit_idx] += weight
            else:
                v[bit_idx] -= weight

    # 降维压缩：大于 0 置 1，否则置 0
    fingerprint = 0
    for bit_idx in range(64):
        if v[bit_idx] > 0:
            fingerprint |= 1 << bit_idx
    return fingerprint


def hamming_distance(hash1: int, hash2: int) -> int:
    """利用按位异或 (^) 与 bit_count() 计算两个 64 位签名的汉明距离"""
    return (hash1 ^ hash2).bit_count()


if __name__ == "__main__":
    doc_a = "人工智能技术应用专业聚焦网络爬虫、数据清洗与大语言模型微调全链路工程实践，培养高素质AI工程师。"
    doc_b = "人工智能技术应用专业聚焦网络爬虫、数据清洗与大语言模型微调全链路工程实践，培养高素质AI工程师！（转载自某网）"
    doc_c = "今天天气晴朗，周末适合去公园散步和品尝美食，完全是一篇与人工智能无关的生活随笔文章。"

    sh_a, sh_b, sh_c = compute_simhash_64(doc_a), compute_simhash_64(doc_b), compute_simhash_64(doc_c)
    dist_ab = hamming_distance(sh_a, sh_b)
    dist_ac = hamming_distance(sh_a, sh_c)
    print(f"🔍 [转载近似文档 A vs B] 汉明距离 = {dist_ab} (<=3 判定为近似重复: {dist_ab <= 3})")
    print(f"🔍 [无关文档 A vs C]     汉明距离 = {dist_ac} (<=3 判定为近似重复: {dist_ac <= 3})")
```

在进入文件与数据库存储学习之前，请通过下方的交互式 ETL 数据清洗与 SimHash 去重实验室，亲自配置清洗算子并检验近似重复文档的汉明距离阈值。

#### Diagram: Web Data ETL Cleaning and SimHash Deduplication Workbench

<details markdown="1">
<summary>Web Data ETL Cleaning and SimHash Deduplication Workbench</summary>
Type: microsim
**sim-id:** etl-simhash-cleaning-workbench<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Apply<br/>
**Bloom Verb:** execute<br/>
**Learning Objective:** The learner will execute the 5-stage ETL cleaning and SimHash near-duplicate filtering pipeline on 6 raw scraped web records to produce a validated, deduplicated AI training dataset.

**Prerequisites:** HTML tag stripping & entity unescaping (`html.unescape`), Unicode `NFKC` normalization, numeric unit conversion (`万`/`k`), Pydantic field validation, MD5 exact hashing, 64-bit SimHash & Hamming distance ($d_{\text{Hamming}} \le 3$) (all defined in Sections 1.1–2.2 above).

**Evidence of Mastery:** For each of the 6 raw scraped records in the batch, the learner predicts the final pipeline disposition (`✅ 入库保留 KEEP` / `🚫 Pydantic校验拦截 REJECT_SCHEMA` / `🔁 MD5精确重复丢弃 DROP_MD5` / `🔁 SimHash近似重复丢弃 DROP_SIMHASH`) and identifies the exact cleaning transformation or Hamming distance value. Mastery is classifying at least 5 of the 6 records correctly on the first attempt.

**Misconceptions:** (1) MD5 hashing can detect near-duplicate reposted articles that differ only by a trailing source attribution line. (2) Removing `<script>` tags with a naive `<[^>]+>` regex removes the JavaScript code inside the tag body. (3) Full-width digits `２０２６` and half-width digits `2026` have identical UTF-8 byte sequences without `NFKC` normalization.

**Instructional Rationale:** Apply-level execution requires learners to trace how raw dirty strings transform across each ETL stage and apply the quantitative Hamming distance threshold ($d_{\text{Hamming}} \le 3$) and Pydantic schema constraints to classify concrete records.

**Content:**

Pipeline validation and deduplication rules displayed in the reference bar:
- **Pydantic Schema Rules**: `url` must start with `http://` or `https://`; cleaned `title` length $\ge 3$; cleaned `content` length $\ge 20$; converted `views` $\ge 0$.
- **Deduplication Rules**: Compare each valid record against already-accepted records in the corpus store:
  - If `MD5(cleaned_content)` matches an existing record exactly ($d = 0$ bytes diff) $\rightarrow$ `DROP_MD5`.
  - Else if `hamming_distance(SimHash64(cleaned_content), existing_SimHash64) <= 3` $\rightarrow$ `DROP_SIMHASH`.
  - Otherwise $\rightarrow$ `KEEP` and insert into the corpus store.

The 6 raw scraped input records processed in fixed order (Record #1 is the baseline first record):

| # | Raw Input Record Shown to Learner | Cleaned Transformation Preview | MD5 / SimHash Distance vs. Accepted Store | Correct Disposition | Feedback & Explanation (Why) |
|---|---|---|---|---|---|
| 1 | `url`: `"https://ai.edu/news/101"`<br/>`title`: `"  <h1>ＡＩ大模型\u200b２０２６实训指南 &amp; 规范</h1> "`<br/>`content`: `"<p>人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践。</p><script>track()</script>"`<br/>`views`: `"3.5万"` | `title`: `"AI大模型2026实训指南 & 规范"`<br/>`content`: `"人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践。"` (length 31)<br/>`views`: `35000` | 语料库首条记录（无历史碰撞） | `KEEP` | `<script>` 块被彻底剥离，`&amp;` 还原为 `&`，全角 `ＡＩ２０２６` 经 NFKC 转为半角，`"3.5万"` 转为整数 `35000`，通过 Pydantic 校验并作为首条基准记录入库。 |
| 2 | `url`: `"https://mirror.edu/repost/88"`<br/>`title`: `"AI大模型2026实训指南"`<br/>`content`: `"<div>人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践。</div>"`<br/>`views`: `"1.2k"` | `title`: `"AI大模型2026实训指南"`<br/>`content`: `"人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践。"`<br/>`views`: `1200` | 与记录 #1 清洗后正文一字不差，MD5 完全相同，汉明距离 $d = 0$ | `DROP_MD5` | 虽然 URL 和外层 HTML 标签（`<div>` vs `<p>`）不同，但剥离标签后正文内容与记录 #1 完全一致，被 MD5 内容指纹精确拦截去重。 |
| 3 | `url`: `"https://aggregator.com/a/99"`<br/>`title`: `"【转载】AI大模型2026实训指南"`<br/>`content`: `"<p>人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践！（来源：综合网）</p>"`<br/>`views`: `"850"` | `title`: `"【转载】AI大模型2026实训指南"`<br/>`content`: `"人工智能技术应用专业聚焦网络爬虫、数据清洗与大模型微调工程实践！（来源：综合网）"`<br/>`views`: `850` | MD5 与记录 #1 不同；但 64 位 SimHash 与记录 #1 的汉明距离 $d_{\text{Hamming}} = 2 \le 3$ | `DROP_SIMHASH` | 末尾多出的 `"！（来源：综合网）"` 导致 MD5 完全改变，精确去重失效；但 64 位 SimHash 计算出与记录 #1 的汉明距离仅为 $2$（满足 $\le 3$ 阈值），成功识别为近似洗稿转载并丢弃！ |
| 4 | `url`: `"https://ai.edu/news/102"`<br/>`title`: `"<b>简讯</b>"`<br/>`content`: `"<p>本院今日召开教研会。</p>"`<br/>`views`: `"120"` | `title`: `"简讯"` (length 2 < 3)<br/>`content`: `"本院今日召开教研会。"` (length 10 < 20)<br/>`views`: `120` | 未进入指纹比对阶段 | `REJECT_SCHEMA` | 剥离 HTML 标签后，`title` 长度仅为 2（不满足 Pydantic `min_length=3`）且 `content` 长度仅为 10（不满足 `min_length=20`），触发 `ValidationError` 被质检拦截。 |
| 5 | `url`: `"https://ai.edu/news/105"`<br/>`title`: `"<h3>Python 异步协程与 Redis 布隆过滤器深度解析</h3>"`<br/>`content`: `"<p>在亿级 URL 采集场景下，使用 Redis 位图与布隆过滤器可将内存消耗降低 90% 以上。</p>"`<br/>`views`: `"2.4w"` | `title`: `"Python 异步协程与 Redis 布隆过滤器深度解析"`<br/>`content`: `"在亿级 URL 采集场景下，使用 Redis 位图与布隆过滤器可将内存消耗降低 90% 以上。"` (length 44)<br/>`views`: `24000` | 与记录 #1 的 SimHash 汉明距离 $d_{\text{Hamming}} = 27 > 3$ | `KEEP` | 字段全部符合 Pydantic 强类型契约，`"2.4w"` 归一化为 `24000`，且与已入库记录的汉明距离高达 $27$（远大于阈值 $3$），属于全新高质量语料，准予入库。 |
| 6 | `url`: `"javascript:void(0)"`<br/>`title`: `"无效链接测试文章"`<br/>`content`: `"<p>这是一篇正文长度超过二十个汉字但源链接是前端伪协议的脏数据样本记录。</p>"`<br/>`views`: `"50"` | `url`: `"javascript:void(0)"` (不匹配 `^https?://` 正则) | 未进入指纹比对阶段 | `REJECT_SCHEMA` | `url` 字段值为前端伪协议 `"javascript:void(0)"`，无法通过 Pydantic 的 `pattern=r"^https?://..."` 正则校验，被直接拦截丢弃。 |

**Provenance:** Directly mirrors the cleaning functions in Chapter 9 Section 1.4 and the `ArticleCorpusItem` Pydantic model and `compute_simhash_64` implementation in Section 2.2.

**Rules:**
- Stage 1–3 (`clean_html_and_text` & `parse_chinese_number`) always run before Stage 4 (`Pydantic` validation).
- If Stage 4 fails any schema rule (`url` regex, `title` length $\ge 3$, `content` length $\ge 20$), the record immediately terminates as `REJECT_SCHEMA`.
- If Stage 4 passes, Stage 5 checks `MD5(cleaned_content)` first (`DROP_MD5` if identical), and then checks `hamming_distance(SimHash64, existing) <= 3` (`DROP_SIMHASH` if $\le 3$, else `KEEP`).

**Learner Activity:**

1. For each of the 6 raw records, the learner inspects the Raw Input card and clicks "执行清洗预览 (Run Stage 1–3)" to see the stripped HTML, NFKC normalization, converted `views`, and computed MD5/SimHash distance.
2. Before the final stage executes, the learner selects one of the four disposition buttons (`KEEP`, `REJECT_SCHEMA`, `DROP_MD5`, `DROP_SIMHASH`).
3. The workbench checks the learner's classification, updates the Accepted Corpus Store table, and advances to the next record.

**Feedback:** 6 records in fixed order, 1 attempt per record. Correct: "判定准确！" followed by the `Feedback & Explanation (Why)` text. Incorrect: "判定有误（正确处置应为 <Correct Disposition>）：" followed by the `Feedback & Explanation (Why)` text. Score display: `ETL 质检与去重正确率：n / 6`.

**Starting State:** Record #1 is loaded in the Raw Input inspector showing its raw HTML tags, full-width characters `ＡＩ大模型\u200b２０２６`, and `"3.5万"` views, with the prompt: "请查看第 1 条原始脏数据经过清洗转换后的字段与指纹距离，并选择最终入库处置结果。"

**Chapter Anchors:** Matches the exact functions (`clean_html_and_text`, `parse_chinese_number`, `ArticleCorpusItem`, `compute_simhash_64`) and the Hamming distance threshold $d_{\text{Hamming}} \le 3$ in Sections 1.2–2.2.
</details>

---

## 3. 结构化文件存储：CSV、JSON Lines 与 Parquet 列式存储

当脏数据经过 ETL 管道变成干净的 Python 字典或 Pydantic 模型后，最轻量、最直接的持久化方式就是写入磁盘文件。针对不同的下游消费场景（业务人员用 Excel 查看、大模型流式训练读取、或数据科学家用 Pandas 分析），我们必须精准掌握三种标准文件格式。

### 3.1 CSV文件读写与UTF-8-SIG防乱码

**CSV（Comma-Separated Values，逗号分隔值）** 是最通用的二维表格交换格式。在 Python 中编写爬虫导出 CSV 文件时，新手最常遇到的两个工程痛点及其标准解法如下：

1. **Windows Excel 打开 UTF-8 CSV 中文乱码问题 $\rightarrow$ 使用 `encoding="utf-8-sig"`**  
   如果你用标准的 `encoding="utf-8"` 保存包含中文的 CSV 文件，直接双击用 Windows 版 Microsoft Excel 打开时，中文会全部变成天书乱码！这是因为 Excel 在打开 CSV 时依靠文件开头的 **BOM（Byte Order Mark，字节序标记 `0xEF 0xBB 0xBF`）** 来识别 UTF-8 编码；若无 BOM，Excel 会退化使用本地系统的 GBK（ANSI）编码去强行解码 UTF-8 字节流。因此，**凡是需要交付给业务人员用 Excel 查看的 CSV，必须指定 `encoding="utf-8-sig"`**（Python 会自动在文件头部写入 3 字节 UTF-8 BOM）。
2. **Windows 下 CSV 每隔一行多出一个空行问题 $\rightarrow$ 必须指定 `newline=""`**  
   在调用 `open(..., mode="w", encoding="utf-8-sig", newline="")` 时，务必加上 `newline=""`，防止 Python 的文本流换行符转换与 `csv.DictWriter` 自带的 `\r\n` 叠加产生双倍空行。

### 3.2 JSON与JSON-Lines流式行存储格式

在处理包含嵌套列表或字典的半结构化语料（例如一篇文章包含多个评论列表）时，CSV 的二维扁平列结构就捉襟见肘了。此时需要使用 JSON 家族格式，但必须严格区分**传统标准 JSON 数组（`.json`）**与**大模型工业界绝对标准——`JSON Lines (.jsonl / .ndjson)`流式行存储格式**：

- **传统 `.json` 文件的致命缺陷**：整个文件是一个巨大的列表 `[{...}, {...}]`。第一，你无法方便地以追加模式（`"a"`）往文件末尾直接写新记录（因为末尾有一个闭合方括号 `]`）；第二，当文件达到 $50\text{ GB}$ 时，`json.load(f)` 必须一次性把整个 $50\text{ GB}$ 文件全部读入内存构建语法树，瞬间引发内存溢出（OOM）崩溃！
- **`JSON Lines (.jsonl)` 的三大核心优势**：
  1. **每行一个独立完整的 JSON 对象**，行与行之间仅用换行符 `\n` 分隔；
  2. **天然支持流式追加写入（Streaming Append）与断点续爬**：爬虫每抓完一条数据，直接 `f.write(json.dumps(item, ensure_ascii=False) + "\n")` 即可追加落盘，即便程序意外断电，前面已写入的所有行依然 100% 合法有效；
  3. **$O(1)$ 内存流式读取**：下游大模型训练框架（如 HuggingFace `datasets`）可以一行一行流式迭代读取 TB 级的 `.jsonl` 语料，内存占用始终保持在几 MB。

### 3.3 Parquet列式存储与Pandas批量导出

当采集的数据量达到千万行级别，且下游需要进行高频的数据统计分析、特征工程或上传至 HuggingFace Hub 时，纯文本格式的 CSV 和 JSONL 存在两个明显短板：**磁盘体积过大**、**按列聚合查询极慢（因为行式存储必须扫描整行文本并反复解析数据类型）**。

为此，现代大数据与 AI 工程广泛采用 **Apache Parquet——`Parquet`列式存储与`Pandas`批量导出**（安装命令：`pip install pandas pyarrow`）。Parquet 的核心特征是：

- **列式物理存储（Columnar Storage）**：将同一列的数据（如全部 100 万个 `price` 浮点数或全部 `publish_time`）连续存放在一起，对同类数据采用游程编码（RLE）、字典编码与 Snappy/Zstandard 压缩，**文件体积通常只有同名 CSV/JSONL 的 $15\%\sim 25\%$**；
- **自带严格 Schema 元数据**：内嵌每一列的精确数据类型（`int64`、`float64`、`timestamp`），读取时无需重新做文本类型推断，Pandas 读取速度比 CSV 快 $10\sim 30$ 倍。

下面这段代码一次性演示了如何将同一批清洗好的 AI 课程语料分别导出为 **Excel 零乱码 CSV（`utf-8-sig`）**、**大模型流式 `.jsonl`** 以及 **高压缩比 `.parquet` 列式文件**：

```python
import csv
import json
import pandas as pd


def export_to_three_file_formats(records: list[dict], base_name: str = "ai_dataset") -> None:
    """将结构化字典列表同时导出为 CSV (utf-8-sig)、JSONL 与 Parquet 三种工业标准格式"""
    if not records:
        return

    # 1. 导出为兼容 Windows Excel 的 CSV 文件 (必须指定 encoding='utf-8-sig' 与 newline='')
    csv_path = f"{base_name}.csv"
    fieldnames = list(records[0].keys())
    with open(csv_path, mode="w", encoding="utf-8-sig", newline="") as f_csv:
        writer = csv.DictWriter(f_csv, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(records)

    # 2. 导出为大模型训练标准 JSON Lines (.jsonl) 格式 (每行一个 JSON，ensure_ascii=False)
    jsonl_path = f"{base_name}.jsonl"
    with open(jsonl_path, mode="a", encoding="utf-8") as f_jsonl:
        for item in records:
            f_jsonl.write(json.dumps(item, ensure_ascii=False) + "\n")

    # 3. 使用 Pandas + PyArrow 批量导出为 Parquet 列式压缩格式
    parquet_path = f"{base_name}.parquet"
    df = pd.DataFrame(records)
    df.to_parquet(parquet_path, engine="pyarrow", compression="snappy", index=False)

    print(f"📁 成功导出三种格式: {csv_path} | {jsonl_path} | {parquet_path}")
```

---

## 4. 关系型数据库持久化：SQLite、MySQL、UPSERT 幂等与连接池

虽然文件存储简单高效，但当我们需要支持**多爬虫进程并发写入、按条件毫秒级检索、多表关联分析或线上实时业务更新**时，就必须将数据存入关系型数据库（RDBMS）中。

### 4.1 SQLite轻量级嵌入式数据库存储

**`SQLite`轻量级嵌入式数据库存储** 是直接内置于 Python 标准库（`import sqlite3`）中的无服务器（Serverless）关系型数据库。它的整个数据库（包含所有的表、索引与数据）就保存在磁盘上的**单个 `.db` 文件**中，无需安装和启动任何后台数据库服务进程，非常适合单机中小型爬虫项目（百万级记录以内）的快速结构化落地与断点状态管理。

### 4.2 MySQL表结构设计与utf8mb4字符集

在企业生产环境中，最主流的开源关系型数据库当属 **MySQL**。在进行爬虫存储的 **MySQL表结构设计与`utf8mb4`字符集** 规划时，有一个所有爬虫工程师都吃过亏的经典深坑：

!!! warning "MySQL 历史大坑：永远使用 `utf8mb4`，绝不使用残血的 `utf8`！"
    在 MySQL 的早期实现中，名为 `utf8`（别名 `utf8mb3`）的字符集存在严重设计缺陷——它**每个字符最多只支持 3 个字节**！然而，现代互联网网页（尤其是微博、小红书、公众号文章及评论区）中充斥着大量的 **Emoji 表情符号（如 `🚀`、`🔥`、`👍`）以及生僻汉字/罕见符号**，这些字符在标准 UTF-8 编码下需要占用 **4 个字节**。
    如果你的 MySQL 表或数据库连接使用了 `utf8`，一旦爬虫抓到包含 Emoji 的标题或评论，写入时就会瞬间抛出 `OperationalError: (1366, "Incorrect string value: '\\xF0\\x9F\\x9A\\x80...'")` 报错导致入库中断！
    **工业界铁律**：建库、建表以及 Python 建立数据库连接时，字符集必须统一显式声明为 **`utf8mb4`**（排序规则推荐 `utf8mb4_unicode_ci` 或 `utf8mb4_0900_ai_ci`）！

### 4.3 PyMySQL参数化防注入与批量写入

在 Python 中操作 MySQL 最常用的纯 Python 驱动是 **`PyMySQL`参数化防注入与批量写入**（安装命令：`pip install pymysql`）。在编写 SQL 插入语句时，有两条不可逾越的准则：

1. **严禁使用 `f-string` 或 `%` 拼接 SQL 字符串，必须使用 `%s` 占位符参数化查询！**  
   网页上的文章标题和正文完全不受我们控制，极可能包含单引号 `'`、双引号 `"` 甚至恶意构造的 SQL 片段（如 `Title'); DROP TABLE articles; --`）。如果用 `f"INSERT INTO articles VALUES ('{title}')"` 直接拼接字符串，一旦标题里出现一个单引号就会直接引发 SQL 语法报错，更会造成致命的 **SQL 注入漏洞（SQL Injection）**！正确做法是把 SQL 模板写为 `INSERT INTO articles (url, title) VALUES (%s, %s)`，并将数据作为元组第二个参数传给 `cursor.execute(sql, (url, title))`，由驱动底层安全转义。
2. **使用 `cursor.executemany(sql, tuple_list)` 实现批量事务提交**  
   如果每抓一条数据就执行一次 `execute()` + `conn.commit()`，大量的网络往返与磁盘事务日志刷盘（fsync）会让写入速度慢如蜗牛。将每 $500\sim 1,000$ 条记录攒成一个元组列表，调用一次 `cursor.executemany()` 并统一 `commit()`，写入吞吐量可提升 **$20\sim 50$ 倍**！

### 4.4 唯一索引约束与UPSERT幂等更新

在增量爬虫或断点重跑场景下，同一个网页 URL 很可能会被重复抓取，或者我们需要更新某件已入库商品的最新价格。如何保证数据库里绝不产生重复记录，同时还能自动更新最新字段？这就需要结合数据库层的 **唯一索引约束（`UNIQUE KEY`）与 `UPSERT` 幂等更新（`INSERT` + `UPDATE`）**：

- 首先，在建表时对 `url_hash`（URL 的 MD5 值，固定 `CHAR(32)`，比对长 URL 字符串直接建索引更快更省空间）建立唯一索引 `UNIQUE KEY uk_url_hash (url_hash)`；
- **策略一（忽略重复）：`INSERT IGNORE INTO ...`**（SQLite 中对应 `INSERT OR IGNORE INTO ...`）：当遇到已存在的 `url_hash` 时，静默跳过不报错；
- **策略二（存在则更新）：`INSERT INTO ... ON DUPLICATE KEY UPDATE price = VALUES(price), updated_at = NOW()`**（SQLite 3.24+ 中对应 `ON CONFLICT(url_hash) DO UPDATE SET ...`）：当首次抓取时执行插入；当再次抓到同一商品时，自动就地更新其最新价格与采集时间戳，从而实现完美的**幂等写入（Idempotency，无论重复运行多少次，结果状态一致且无脏副本）**！

### 4.5 SQLAlchemy与数据库连接池高并发

当我们在第 8 章的 `ThreadPoolExecutor` 多线程爬虫中并发写入数据库时，注意：**同一个 `pymysql.Connection` 连接对象绝对不是线程安全的**，多个线程同时共用一个连接会直接导致数据包错乱崩溃；但如果每个线程每写一条数据都重新建立一次 TCP + MySQL 认证握手连接，数据库服务器又会被瞬间拖垮。

工业级解法是使用 **`SQLAlchemy`与数据库连接池高并发**（`from sqlalchemy import create_engine, text`）。`create_engine` 内部自带线程安全的 **QueuePool 连接池**（可通过 `pool_size=10, max_overflow=20, pool_recycle=3600` 配置常驻连接数、突发溢出连接数与自动重连周期），多线程只需从池中借用连接并在 `with engine.begin() as conn:` 块结束时自动归还即可。

下面这段代码首先是可直接运行的 **SQLite 嵌入式幂等 `UPSERT` 完整实战**，随后给出了生产环境 **MySQL `utf8mb4` 建表与 `PyMySQL` 批量 `ON DUPLICATE KEY UPDATE`** 的标准模板：

```python
import hashlib
import sqlite3


def run_sqlite_idempotent_upsert() -> None:
    """使用内置 SQLite3 演示 UNIQUE 唯一索引 + ON CONFLICT DO UPDATE 幂等批量入库"""
    # 使用内存数据库 (生产中替换为文件路径如 'crawler_data.db')
    conn = sqlite3.connect(":memory:")
    cursor = conn.cursor()

    # 1. 建表：对 url_hash 设置 UNIQUE 唯一约束
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS ai_products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            url_hash TEXT NOT NULL UNIQUE,
            url TEXT NOT NULL,
            name TEXT NOT NULL,
            price REAL NOT NULL
        )
    """)

    # 2. 编写支持幂等更新 (UPSERT) 的参数化 SQL 语句
    upsert_sql = """
        INSERT INTO ai_products (url_hash, url, name, price)
        VALUES (?, ?, ?, ?)
        ON CONFLICT(url_hash) DO UPDATE SET
            name = excluded.name,
            price = excluded.price
    """

    # 模拟两批采集数据：其中 item/101 在第二批中价格发生了降价更新
    raw_batches = [
        ("https://shop.edu/item/101", "RTX 4090 算力卡", 15999.0),
        ("https://shop.edu/item/102", "AI 边缘计算盒", 2499.0),
        ("https://shop.edu/item/101", "RTX 4090 算力卡(促销价)", 14599.0),  # 同一 URL 重复采集
    ]

    params = [
        (hashlib.md5(u.encode("utf-8")).hexdigest(), u, name, price)
        for u, name, price in raw_batches
    ]

    # 3. 使用 executemany 批量参数化写入并提交事务
    cursor.executemany(upsert_sql, params)
    conn.commit()

    cursor.execute("SELECT url, name, price FROM ai_products")
    for row in cursor.fetchall():
        print(f"📦 [SQLite 幂等表记录] {row}")
    conn.close()


# 生产环境 MySQL utf8mb4 + PyMySQL 批量 UPSERT 标准模板 (供连接真实 MySQL 实例使用)
MYSQL_UPSERT_TEMPLATE_SQL = """
CREATE TABLE IF NOT EXISTS crawled_articles (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    url_hash CHAR(32) NOT NULL COMMENT 'URL的MD5定长唯一指纹',
    url VARCHAR(768) NOT NULL,
    title VARCHAR(500) NOT NULL,
    views INT UNSIGNED NOT NULL DEFAULT 0,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_url_hash (url_hash)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- PyMySQL executemany 幂等更新语句模板 (使用 %s 参数化防 SQL 注入):
INSERT INTO crawled_articles (url_hash, url, title, views)
VALUES (%s, %s, %s, %s)
ON DUPLICATE KEY UPDATE
    title = VALUES(title),
    views = VALUES(views);
"""

if __name__ == "__main__":
    run_sqlite_idempotent_upsert()
```

---

## 5. NoSQL、异步数据库与亿级去重：MongoDB、Redis 与对象存储

### 5.1 MongoDB文档数据库半结构化存储与PyMongo幂等去重

在很多复杂采集任务中（例如抓取全网不同电商平台的商品详情页），手机品类有“屏幕尺寸、电池容量”参数，服装品类有“尺码、面料”参数，而图书品类有“ISBN、作者、出版社”参数。如果用传统 MySQL 存储这种**字段高度动态变化、带有多层嵌套列表与字典的半结构化数据（Semi-Structured Data）**，要么需要建几十张关联表，要么只能把多余字段塞进一个大 JSON 字段里。

这正是 **`MongoDB`文档数据库半结构化存储** 大显身手的舞台。MongoDB 将每一条记录存储为一个 **BSON（Binary JSON）文档**，天然映射 Python 的 `dict` 字典，无需预先定义死板的表结构（Schema-flexible）。

在 Python 中，我们通过官方驱动 **`PyMongo`集合操作与`upsert`幂等去重**（`pip install pymongo`）操作 MongoDB。为了防止重复插入，只需两步：

1. 在集合初始化时对 `url`（或 `url_hash`）创建唯一索引：`collection.create_index("url", unique=True)`；
2. 写入时摒弃简单的 `insert_one`，改用 **`collection.update_one({"url": item["url"]}, {"$set": item}, upsert=True)`**（批量写入时使用 `bulk_write([UpdateOne(..., upsert=True)])`）。当 `url` 不存在时自动插入新文档，已存在时自动用 `$set` 更新最新字段！

### 5.2 Motor与aiomysql异步数据库写入

回顾第 8 章的铁律：**在 `asyncio` 异步协程爬虫中，绝不能直接调用同步阻塞的 `pymysql` 或 `pymongo`！** 否则在等待数据库网络往返的几毫秒内，整个事件循环又会被卡住。为了打通全异步采集与入库的最后一公里，我们必须使用 **`Motor`与`aiomysql`异步数据库写入**：

- **`motor`（异步 MongoDB 官方驱动）**：用法与 PyMongo 几乎一模一样，只需把客户端替换为 `AsyncIOMotorClient`，并在 `await collection.update_one(..., upsert=True)` 前加上 `await` 即可。
- **`aiomysql`（异步 MySQL 驱动）**：通过 `await aiomysql.create_pool(..., charset="utf8mb4")` 创建异步连接池，在协程内通过 `async with pool.acquire() as conn:` 非阻塞执行 SQL。

```python
# Motor (异步 MongoDB) 与 aiomysql (异步 MySQL) 核心协程入库范式示例
from motor.motor_asyncio import AsyncIOMotorClient


async def save_to_mongo_async(items: list[dict], mongo_uri: str = "mongodb://localhost:27017") -> None:
    """在 asyncio 协程中使用 Motor 异步非阻塞写入 MongoDB 并实现 upsert 幂等去重"""
    client = AsyncIOMotorClient(mongo_uri)
    db = client["ai_crawler_db"]
    col = db["raw_documents"]

    # 确保在 url 字段上建立唯一索引
    await col.create_index("url", unique=True)

    for item in items:
        # 使用 await 非阻塞执行幂等更新：存在则 $set 覆盖，不存在则自动插入 (upsert=True)
        await col.update_one(
            {"url": item["url"]},
            {"$set": item},
            upsert=True,
        )
    client.close()
```

### 5.3 Redis内存数据库核心数据结构与布隆过滤器URL去重

当我们的爬虫从单机迈向多机分布式集群（详见第 10 章），或者累计待抓取的 URL 数量突破**千万乃至数亿级别**时，单机 Python 内存里的 `set()` 集合将彻底失效：一方面多台机器无法共享同一个 Python 进程内存，另一方面在 Python `set` 中存储 **1 亿条（$10^8$）** 长度为 64 字节的 URL 字符串，光哈希表指针与字符串对象开销就会吃掉约 **$8\text{ GB}\sim 10\text{ GB}$ 内存**！

为此，工业界统一引入基于内存的高性能键值数据库——**`Redis`内存数据库核心数据结构**。Redis 提供了五大在爬虫工程中各司其职的核心数据结构：

1. **`List`（双向链表）**：通过 `LPUSH` / `RPOP`（或阻塞版 `BRPOP`）充当多机共享的 FIFO/LIFO 待抓取 URL 队列；
2. **`Set`（无序哈希集合）**：通过 `SADD key member` 实现 $O(1)$ 跨机器精确去重（返回 `1` 代表新 URL，返回 `0` 代表已存在）；
3. **`Sorted Set / ZSet`（有序集合）**：为每个 URL 绑定一个浮点权重分数 `score`（如优先级或下次抓取时间戳），实现带优先级的定时调度队列；
4. **`Hash`（哈希字典）**：存储爬虫节点的心跳状态、Cookie 池账号映射或配置元数据；
5. **`Bitmap`（位图）/ `BloomFilter`（布隆过滤器）**：专为**亿级乃至十亿级海量 URL 去重**打造的空间压缩神器！

让我们深入剖析 **`Redis`集合Set与布隆过滤器URL去重（Redis Set vs. Bloom Filter for 100M-Scale URL Deduplication）**：

- **Redis `Set` 的内存瓶颈**：在 Redis `Set` 中存入 1 亿个 32 字节的 MD5 指纹，依然需要消耗约 **$4.5\text{ GB}\sim 6.4\text{ GB}$ 内存**。
- **布隆过滤器（Bloom Filter）的空间魔法**：由 Burton Howard Bloom 于 1970 年提出。它底层仅使用一个长度为 $m$ 个二进制比特位（bits，初值全为 `0`）的超大**位数组（Bit Array）**，配合 $k$ 个相互独立的哈希函数 $h_1, h_2, \dots, h_k$：
  - **添加 URL（Add）**：对 URL 计算出 $k$ 个哈希值并对 $m$ 取模，得到位数组上的 $k$ 个位置索引，将这 $k$ 个比特位全部置为 `1`；
  - **查询 URL 是否已抓取（Query）**：同样计算该 URL 对应的 $k$ 个比特位位置：
    - **只要其中有任意 1 个比特位为 `0`** $\rightarrow$ **该 URL $100\%$ 绝对没有被抓取过（零漏报率 Zero False Negatives）！**
    - **如果 $k$ 个比特位全部为 `1`** $\rightarrow$ 判定为“已抓取过”（注意：由于不同 URL 的哈希位可能碰巧重叠，存在极低概率把未抓过的 URL 误判为已抓过，称为**误判率 False Positive Rate $p$**）。

给定预期去重 URL 总数 $n$ 与可接受的误判率 $p$（例如万分之一 $p = 0.0001 = 0.01\%$），布隆过滤器所需的**最优比特位数 $m$** 与**最优哈希函数个数 $k$** 由以下经典数学公式严格给出：

\[
m = -\frac{n \ln p}{(\ln 2)^2} \approx -2.081 \times n \ln p, \qquad k = \frac{m}{n} \ln 2 \approx 0.693 \times \frac{m}{n}
\]

让我们算一笔震撼的工程账：同样是对 **$n = 1\text{ 亿}$（$10^8$）个 URL** 进行去重，要求误判率不超过万分之一（$p = 0.0001$）：

- 代入公式得每条 URL 仅需约 $m / n \approx 19.17\text{ bits}$，最优哈希函数个数 $k \approx 13$；
- 1 亿个 URL 总共仅需 $1.917 \times 10^9\text{ bits} \approx \mathbf{228.5\text{ MB}}$ 内存！相比于 Redis `Set` 的 $6.4\text{ GB}$，**布隆过滤器节省了超过 $96\%$ 的内存空间！**

### 5.4 多模态文件哈希命名与对象存储归档

在采集图文多模态数据集（如商品主图、学术论文 PDF、短视频音频）时，**切忌将几百萬张图片的二进制字节流（BLOB）直接塞进 MySQL 或 MongoDB 表中**——这会让数据库体积瞬间膨胀至 TB 级并彻底拖垮查询性能。

正确的工业级实践是 **多模态文件哈希命名与对象存储归档（Content-Addressable Hash Naming & S3/MinIO Object Storage）**：

1. **基于内容 SHA-256 哈希的分级目录命名**：对下载回来的图片字节流 `img_bytes` 计算 `sha256_hex = hashlib.sha256(img_bytes).hexdigest()`。取前两位作为一级子目录、第 3~4 位作为二级子目录：例如保存路径为 `images/a3/f8/a3f89c...d2.jpg`。这样做既避免了单文件夹下存放百万文件导致 Linux 文件系统 `inode` 查找变慢，又天然实现了**相同图片文件物理级自动去重（Content-Addressable Storage）**，还彻底杜绝了原网址中包含特殊字符 `?=&` 引发的非法文件名报错。
2. **元数据与二进制分离归档（对象存储 S3 / MinIO / 阿里云 OSS）**：将图片/PDF 上传至分布式对象存储服务（如开源自建的 MinIO 或云端 S3），而在 MySQL/MongoDB 或 `.jsonl` 元数据表中**仅存储该文件的对象存储路径 URI 与 SHA-256 校验和**。

下表全面对比了本章所涵盖的 7 种主流数据存储方案的工程特性与选型边界：

| 存储方案 | 存储模型类别 | Schema 灵活性 | 1亿URL去重/亿级存储表现 | 最佳 AI 数据工程适用场景 |
|---|---|---|---|---|
| **CSV (`utf-8-sig`)** | 扁平二维文本表格 | 低（固定表头列） | 不适合超大文件或嵌套结构 | 小规模（10万行内）表格导出、直接交付业务人员用 Excel 审阅 |
| **JSON Lines (`.jsonl`)** | 流式行级半结构化文本 | 高（每行独立 JSON） | 支持 TB 级 $O(1)$ 内存流式追加与读取 | **大语言模型（LLM）预训练语料、SFT 指令微调集标准交换格式** |
| **Apache Parquet** | 二进制列式压缩文件 | 中（强类型列元数据） | 极高压缩比（省 80% 空间），列分析极快 | 千万级结构化特征分析、Pandas/Polars 数据科学处理、HuggingFace 数据集发布 |
| **SQLite** | 嵌入式关系型单文件库 | 低（严格 SQL 表结构） | 适合单机百万级以内，不支持多机高并发写 | 单机中小型爬虫持久化、本地轻量级任务状态与断点管理 |
| **MySQL (`utf8mb4`)** | 企业级关系型数据库 | 低（严格表结构 + 索引） | 支持高并发事务与 `ON DUPLICATE KEY UPDATE` | 金融/电商结构化业务数据、需要复杂多表 JOIN 与强一致性事务的生产系统 |
| **MongoDB** | 分布式文档型 NoSQL | 极高（动态 BSON 文档） | 支持海量半结构化文档与 `upsert=True` 幂等 | 页面结构多变的异构商品详情、多层嵌套评论树、原始 API 响应归档 |
| **Redis (`Set` / `BloomFilter`)** | 内存键值与概率型结构 | 键值/位数组原语 | **布隆过滤器仅需 ~229 MB 即可完成 1 亿 URL 去重** | 分布式爬虫共享 URL 调度队列、亿级 URL 极速判重、分布式代理与 Cookie 池 |

下面请通过交互式存储架构选型与布隆过滤器容量计算器，亲自对比不同规模下的内存消耗并为典型 AI 采集场景匹配最佳存储引擎。

#### Diagram: Storage Architecture Selector and Bloom Filter Sizing Calculator

<details markdown="1">
<summary>Storage Architecture Selector and Bloom Filter Sizing Calculator</summary>
Type: microsim
**sim-id:** storage-bloom-filter-calculator<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Evaluate<br/>
**Bloom Verb:** select<br/>
**Learning Objective:** The learner will select the optimal persistence and deduplication architecture for 5 real-world web scraping scenarios by evaluating data structure shape, concurrency scale, and Bloom Filter memory requirements ($m$ and $k$).

**Prerequisites:** CSV (`utf-8-sig`), JSON Lines (`.jsonl`), Parquet columnar storage, MySQL (`utf8mb4` + `ON DUPLICATE KEY UPDATE`), MongoDB (`upsert=True`), Redis `Set` vs. Bloom Filter optimal sizing formulas ($m = -n \ln p / (\ln 2)^2$, $k = (m/n) \ln 2$), SHA-256 content-addressable object storage (all defined in Sections 3.1–5.4 above).

**Evidence of Mastery:** Across 5 architecture design scenarios, the learner uses the interactive Bloom Filter calculator to verify memory constraints and selects the single best storage/deduplication solution for each scenario. Mastery is selecting the correct architecture on at least 4 of the 5 scenarios on the first attempt.

**Misconceptions:** (1) Storing 100 million URL strings in a Python `set` or Redis `Set` takes less than $500\text{ MB}$ of RAM. (2) A Bloom Filter can produce false negatives (telling the crawler a previously visited URL was never visited). (3) Storing millions of raw `.jpg` image byte arrays directly inside MySQL `LONGBLOB` columns is better than saving SHA-256-named files in object storage (S3/MinIO) and storing only URI paths in the database.

**Instructional Rationale:** Evaluate-level selection requires learners to combine quantitative memory trade-offs from the Bloom Filter calculator with qualitative architectural criteria (schema variability, streaming append capability, Excel compatibility, and binary media separation) to justify engineering storage choices.

**Content:**

Interactive Bloom Filter vs. Redis Set Calculator controls:

| Quantity | Min | Max | Step | Default | Unit |
|---|---|---|---|---|---|
| 预期去重 URL 总规模 $n$ | 1,000,000 | 100,000,000 | 1,000,000 | 100,000,000 | 条 URL (URLs) |
| 允许误判率 $p$ (可选档位) | 0.0001 | 0.01 | discrete (`0.01`, `0.001`, `0.0001`) | 0.0001 | 概率 (ratio) |

Calculator outputs displayed dynamically:
- Python `set` / Redis `Set` 精确存储内存开销（按每条 URL 哈希及哈希表槽位平均 $64\text{ Bytes}$ 计）：$\text{Memory}_{\text{Set}} = n \times 64 / 1024^2\text{ MB}$（当 $n = 10^8$ 时为 $6,103.5\text{ MB} \approx 6.0\text{ GB}$）。
- 布隆过滤器最优比特数 $m = \lceil -n \ln p / (\ln 2)^2 \rceil$ 与内存开销 $\text{Memory}_{\text{Bloom}} = m / (8 \times 1024^2)\text{ MB}$（当 $n = 10^8, p = 0.0001$ 时为 $\mathbf{228.5\text{ MB}}$，最优哈希函数个数 $k = 13$，内存节省率 $96.3\%$）。

Five engineering architecture selection scenarios (fixed order):

| # | Scenario Description & Constraints | Candidate Options | Correct Selection | Feedback & Explanation (Why) |
|---|---|---|---|---|
| 1 | **亿级分布式爬虫 URL 去重受限内存挑战**：你需要部署一个抓取全网 1 亿（$10^8$）个网页的分布式爬虫，但云端 Redis 实例仅剩余 **$512\text{ MB}$ 可用内存**，允许万分之一（$p=0.0001$）的极低漏抓率，但绝不允许重复抓取已访问页面。应选择哪种去重方案？ | A. `Python 内置 set()`<br/>B. `Redis Set (SADD)`<br/>C. `Redis BloomFilter (m≈228.5MB, k=13)`<br/>D. `MySQL UNIQUE 索引表` | `C. Redis BloomFilter (m≈228.5MB, k=13)` | 1 亿条 URL 在 Redis Set 中需要约 $6.0\text{ GB}$ 内存，远超 $512\text{ MB}$ 限制；而布隆过滤器在 $p=0.0001$ 下仅需 $228.5\text{ MB}$ 内存，且对已访问过的 URL 漏报率为 0%（绝不重复抓取）。 |
| 2 | **大模型 SFT 微调语料流式追加与断点续爬**：单机异步爬虫持续采集 200 万条包含多轮对话嵌套列表 `messages: [{"role": "user", ...}]` 的语料，要求支持边抓边追加落盘、断电不损坏已有记录，且下游 HuggingFace `datasets` 可按行流式读取。应选择哪种文件格式？ | A. `标准整块 JSON 数组 (.json)`<br/>B. `JSON Lines (.jsonl) + UTF-8`<br/>C. `CSV 表格 (.csv)`<br/>D. `SQLite 二进制库` | `B. JSON Lines (.jsonl) + UTF-8` | `.jsonl` 每行是一个独立完整的 JSON 对象，既天然支持多层嵌套列表结构，又支持 `"a"` 模式流式追加写入和 $O(1)$ 内存逐行读取，是大模型训练集的标准格式。 |
| 3 | **非技术运营团队 Windows Excel 报表交付**：爬虫采集了 5,000 条竞品门店的扁平价格监控数据（含中文店名与地址），要求导出为单个表格文件发给市场部同事，确保对方在 Windows 电脑上双击用 Excel 打开时**中文绝对不乱码**且无多余空行。应如何配置？ | A. `csv.DictWriter + encoding="utf-8"`<br/>B. `csv.DictWriter + encoding="utf-8-sig", newline=""`<br/>C. `json.dump(..., ensure_ascii=True)`<br/>D. `Parquet 列式文件` | `B. csv.DictWriter + encoding="utf-8-sig", newline=""` | Windows Excel 依赖文件头部的 3 字节 UTF-8 BOM 签名来识别 UTF-8 编码；使用 `utf-8-sig` 写入 BOM 并配合 `newline=""` 可完美解决 Excel 中文乱码与隔行空行问题。 |
| 4 | **多品类异构电商详情页幂等入库**：爬虫需要采集全网 50 个不同品类的商品详情（手机含芯片参数、服装含尺码列表、图书含作者目录，字段结构差异极大且随时新增字段），并在重复抓取时按 `url` 自动更新最新字段。应选择哪种数据库方案？ | A. `MongoDB + update_one({"url": u}, {"$set": doc}, upsert=True)`<br/>B. `固定 100 列的宽表 CSV`<br/>C. `Redis String 键值存储`<br/>D. `MySQL 严格固定字段表（不加 JSON 列）` | `A. MongoDB + update_one({"url": u}, {"$set": doc}, upsert=True)` | MongoDB 采用无固定 Schema 的 BSON 文档模型，天然适配各品类字段差异巨大且含嵌套层级的半结构化数据，配合唯一索引与 `upsert=True` 即可实现幂等更新。 |
| 5 | **百万级多模态图文对（图片 + 文本描述）归档**：为训练视觉大模型采集了 500 万张高清商品图片（总计 $2\text{ TB}$）及其对应的标题与属性标签。应如何设计存储架构？ | A. `将图片二进制 bytes 直接存入 MySQL LONGBLOB 字段`<br/>B. `将图片转成 Base64 字符串塞进单个巨大的 .json 文件`<br/>C. `图片以内容 SHA-256 哈希分级命名存入 MinIO/S3 对象存储，数据库/Parquet 仅存 URI 路径与文本元数据`<br/>D. `将所有图片原文件名直接保存在同一个本地文件夹下` | `C. 图片以内容 SHA-256 哈希分级命名存入 MinIO/S3 对象存储，数据库/Parquet 仅存 URI 路径与文本元数据` | 遵循“二进制媒体流与结构化元数据分离”原则：SHA-256 哈希分级目录（如 `a3/f8/a3f8...jpg`）天然实现相同图片物理去重并避免单目录百万文件性能退化，数据库保持轻量高效。 |

**Provenance:** Formulas and memory figures come directly from Chapter 9 Section 5.3; storage trade-offs come from the Section 5.4 comparison table.

**Rules:**
- Calculator formulas: $m = \lceil -n \ln p / (\ln 2)^2 \rceil$, $k = \text{round}((m / n) \ln 2)$, $\text{Memory}_{\text{Bloom}} = m / (8 \times 1048576)\text{ MB}$, $\text{Memory}_{\text{Set}} = n \times 64 / 1048576\text{ MB}$.
- Each scenario allows 1 committed attempt for mastery scoring, with immediate explanation feedback.

**Learner Activity:**

1. The learner uses the top calculator panel to set $n = 100,000,000$ and $p = 0.0001$, observing the contrast between $6,103.5\text{ MB}$ (Set) and $228.5\text{ MB}$ (Bloom Filter, $k=13$).
2. The learner reads Scenario 1 through Scenario 5 in sequence and selects the optimal storage architecture option.
3. After each selection, the sim displays the engineering rationale and updates the mastery counter.

**Feedback:** 5 scenarios in fixed order, 1 attempt per scenario. Correct: "✅ 架构选型正确！" followed by `Feedback & Explanation (Why)`. Incorrect: "❌ 选型欠佳（最佳方案应为 <Correct Selection>）：" followed by `Feedback & Explanation (Why)`. Score counter: `存储架构评估得分：n / 5`.

**Starting State:** The calculator is initialized to $n = 100,000,000$ URLs and $p = 0.0001$ showing $\text{Memory}_{\text{Set}} = 6,103.5\text{ MB}$ vs. $\text{Memory}_{\text{Bloom}} = 228.5\text{ MB}$ ($k=13$), and Scenario 1 is displayed below with the prompt: "请结合上方布隆过滤器内存计算器，为以下 5 个真实 AI 数据工程场景选择最优持久化与去重架构。"

**Chapter Anchors:** Matches the exact Bloom Filter equations and the $n = 10^8, p = 0.0001 \Rightarrow 228.5\text{ MB}, k = 13$ calculation in Section 5.3, as well as the 7-engine comparison table in Section 5.4.
</details>

---

## 6. 本章小结与思考练习

### 6.1 核心要点回顾

- **五阶 ETL 数据清洗流水线**：针对原始网页脏数据，依次执行 **HTML 标签剥离与 `html.unescape` 实体还原** $\rightarrow$ **`unicodedata.normalize("NFKC")` 全半角与特殊空白归一化** $\rightarrow$ **相对日期与中文数值单位（万/亿/k/w）标准化** $\rightarrow$ **`Pydantic` 强类型数据模型校验** $\rightarrow$ **MD5 精确去重与 64 位 SimHash（汉明距离 $d_{\text{Hamming}} \le 3$）近似去重**。
- **三大结构化文件存储选型**：交付给 Windows Excel 用户审阅的二维表格必用 **`CSV (encoding="utf-8-sig", newline="")`**；大语言模型流式训练语料与断点续爬必用 **`JSON Lines (.jsonl)`**；千万级特征分析与高压缩比归档优先选用 **`Apache Parquet`**。
- **关系型与文档型数据库幂等入库**：MySQL 建表与连接必须强制使用 **`utf8mb4`** 字符集以兼容 4 字节 Emoji 表情，使用 `%s` 参数化查询杜绝 SQL 注入，并配合 `executemany` 与 `UNIQUE KEY` + `ON DUPLICATE KEY UPDATE` 实现高性能幂等更新；针对字段多变的半结构化嵌套数据，使用 MongoDB 的 `update_one(..., upsert=True)`；在 `asyncio` 爬虫中则对应替换为异步驱动 **`aiomysql`** 与 **`motor`**。
- **亿级 URL 去重与多模态分离存储**：利用 **Redis 布隆过滤器（Bloom Filter）** 仅需约 $228.5\text{ MB}$ 内存即可完成 1 亿条 URL 的极速判重（节省 $96\%$ 内存）；图片、音视频与 PDF 等多模态文件应采用 **内容 SHA-256 哈希分级目录命名** 存入 S3/MinIO 对象存储，与数据库结构化元数据解耦。

### 6.2 思考与自测练习

??? question "思考题 1：某新闻聚合爬虫每天采集 10 万篇科技资讯用于构建大模型 RAG 知识库。工程师发现，虽然已经对每篇文章的正文计算了 MD5 哈希值进行去重，但知识库里依然出现了几十篇内容几乎一模一样、仅末尾多了一句『本文转自XX公众号，侵删』的重复文章。请问为什么 MD5 去重失效了？应该如何升级去重方案？"
    因为 MD5 属于**密码学哈希函数**，具有极强的“雪崩效应”——哪怕两篇 3,000 字的文章只相差末尾几个字或一个标点符号，计算出的 128 位 MD5 值也会完全不同，因此 MD5 只能用于“一字不差”的**精确去重**。针对这种改动了个别字词或首尾附加了版权声明的转载文章，应该升级为局部敏感哈希 **64 位 SimHash 算法**：提取正文分词或 $n\text{-gram}$ 特征加权生成 64 位签名，计算新文章与已有文章签名之间的**汉明距离（Hamming Distance）**，当汉明距离 $d_{\text{Hamming}} \le 3$ 时即可精准识别为近似重复文档并予以过滤。

??? question "思考题 2：在编写 `PyMySQL` 入库代码时，以下两种写法有什么本质区别？为什么第一种写法在爬虫工程中属于严禁出现的重大安全隐患？<br/>写法 A：`cursor.execute(f\"INSERT INTO news (title) VALUES ('{title}')\")`<br/>写法 B：`cursor.execute(\"INSERT INTO news (title) VALUES (%s)\", (title,))`"
    写法 A 使用了 Python 的 `f-string` 直接将网页上抓来的字符串拼接到 SQL 命令中：一旦网页标题里包含单引号（例如英文标题 `"Python's GIL Explained"`），就会直接破坏 SQL 引号配对引发语法报错崩溃；更致命的是，如果网页内容被恶意植入了 `'); DROP TABLE news; --` 等字符串，数据库会将其当作合法 SQL 指令执行，造成严重的 **SQL 注入攻击（SQL Injection）**！而写法 B 采用了**参数化查询（Parameterized Query）**，`%s` 仅作为参数占位符，PyMySQL 驱动会在底层对 `title` 中的单引号和特殊字符进行严格的安全转义，既彻底根除了 SQL 注入风险，又避免了特殊标点导致的写入报错。

??? question "思考题 3：布隆过滤器（Bloom Filter）在判断一个 URL 是否已经抓取过时，『误判（False Positive）』和『漏判（False Negative）』分别指什么？为什么说布隆过滤器的这一特性刚好契合网络爬虫 URL 去重的需求？"
    在布隆过滤器中：
    - **漏判（False Negative，概率永远为 $0\%$）**：指一个**已经抓取过**的 URL 被错误判断为“没抓过”。布隆过滤器绝对不会发生漏判，因为已抓取 URL 对应的 $k$ 个比特位一定已经被置为了 `1`。这意味着爬虫**绝对不会重复抓取任何一个已经抓过的网页**，彻底杜绝了死循环。
    - **误判（False Positive，概率极低且可控，如 $p=0.0001$）**：指一个**从未抓取过**的新 URL，因为其对应的 $k$ 个比特位恰好被之前其他多个 URL 全部置为了 `1`，从而被误判为“已经抓过了”。在万分之一的误判率下，每 10,000 个新网页中最多只有 1 个会被略过不抓，而换来的是将 1 亿 URL 的去重内存从 $6\text{ GB}$ 压缩到 $228\text{ MB}$，这在海量网页采集中是性价比极高的工程折中。
