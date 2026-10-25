# Chapters

本教材共分为 **12 章**，系统覆盖知识图谱中的 **240 个核心概念**。

## Chapter Overview

1. [网络爬虫导论与数据合规基石](01-intro-and-ethics/index.md) - 本章系统介绍网络爬虫的基本概念、分类体系、标准工作流，以及面向人工智能数据采集的 Robots 协议与法律法规红线。
2. [Web前端基石与HTTP通信原理](02-web-and-http-fundamentals/index.md) - 本章深入剖析 URL 结构、DNS/TCP/TLS 握手流程、HTTP 请求与响应报文结构、Cookie/Session/Token 会话机制及 Chrome DevTools 抓包技术。
3. [Python网络请求核心编程](03-python-http-requests/index.md) - 本章讲解使用 Python 的 urllib、Requests 与新一代 HTTPX 库构造各类网络请求、维持会话、配置代理及实现指数退避重试的工程方法。
4. [静态网页结构化解析与正则提取](04-static-html-parsing/index.md) - 本章系统讲授 HTML5 DOM 树拓扑结构、字符集乱码治理，以及正则表达式、BeautifulSoup4、CSS 选择器、lxml/XPath 与极速解析器的数据提取方法。
5. [异步加载与Web接口逆向采集](05-ajax-and-api-scraping/index.md) - 本章剖析前后端分离（CSR）架构下的 AJAX/XHR/Fetch 通信机制，讲授利用 DevTools 定位隐藏 JSON 接口、解构分页参数及采集 GraphQL 与 WebSocket 数据的方法。
6. [动态渲染网页与Playwright浏览器自动化](06-browser-automation-playwright/index.md) - 本章讲解浏览器渲染引擎原理，并使用现代自动化框架 Playwright 实现智能元素定位、自动等待、复杂交互模拟、登录态持久化与底层网络拦截。
7. [反爬虫机制原理与工程对抗策略](07-anti-scraping-and-countermeasures/index.md) - 本章全面剖析服务端反爬检测维度，系统讲授 UA 轮换、高可用代理 IP 池、WebDriver 与 TLS 指纹伪装、CSS/字体反爬还原、验证码识别及 JS 签名参数逆向。
8. [高性能并发与异步协程爬虫编程](08-concurrent-and-async-scraping/index.md) - 本章对比网络 I/O 密集型与 CPU 密集型任务特征，深入讲解多线程池、多进程解析以及基于 asyncio、aiohttp 与 httpx 的高并发异步协程爬虫与平滑限流。
9. [数据清洗流水线与多源持久化存储](09-data-cleaning-and-storage/index.md) - 本章讲授原始网页脏数据的 ETL 清洗、Pydantic 类型强校验、哈希/SimHash 去重，以及向 CSV、JSONL、Parquet、SQLite、MySQL、MongoDB 与 Redis 的持久化存储。
10. [Scrapy工业级异步爬虫框架实战](10-scrapy-industrial-framework/index.md) - 本章深入剖析 Scrapy 框架六大组件与数据流转拓扑，系统讲授 Spider/CrawlSpider 开发、Item Pipeline 入库、Downloader Middleware 反爬集成与 Scrapy-Redis 分布式架构。
11. [面向大模型与RAG的AI智能爬虫](11-ai-powered-scraping-and-datasets/index.md) - 本章聚焦人工智能前沿，讲授大模型预训练与微调语料标准、网页正文智能抽取、HTML 转 Markdown、Crawl4AI 与 LLM 零样本结构化解析及 RAG 向量知识库构建。
12. [爬虫工程化运维部署与综合实训项目](12-engineering-deployment-and-capstone/index.md) - 本章总结企业级爬虫工程的模块化架构、Docker 容器化部署、定时调度、日志监控告警与数据集质量审计，并提供四个端到端综合实训项目。

## How to Use This Textbook

本教材严格遵循知识依赖图谱（DAG）的拓扑顺序编排章节。每一章所依赖的先修概念均已在前面的章节中完成讲解，建议读者按章节顺序循序渐进地学习，并结合每章配套的代码示例与交互式微仿真（MicroSims）动手实践。

---

**Note:** Each chapter includes a list of concepts covered and their Concept Impact Scores (CIS). Make sure to complete prerequisites before moving to advanced chapters.
