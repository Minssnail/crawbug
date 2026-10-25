# 交互式微仿真实验目录（List of MicroSims）

本页面汇总了《网络爬虫技术》全书 12 章设计的 **25 个交互式微仿真（MicroSim）实验规格**。每个微仿真均严格对齐 2001 修订版布鲁姆教育目标分类学（Bloom's Taxonomy），帮助学生通过可视化交互攻克核心原理与工程重难点。

| 所属章节 | MicroSim ID (`sim-id`) | 微仿真实验名称 | 布鲁姆认知层级 | 推荐实现库 |
| :---: | :--- | :--- | :---: | :---: |
| [第 1 章](../chapters/01-intro-and-ethics/index.md) | `standard-crawler-workflow` | 标准网络爬虫四阶段闭环工作流探索器 | Understand (理解) | `p5.js` |
| [第 1 章](../chapters/01-intro-and-ethics/index.md) | `crawler-bfs-dfs-predictor` | 爬虫遍历策略预测器（BFS 广度优先 vs DFS 深度优先） | Apply (应用) | `vis-network` |
| [第 1 章](../chapters/01-intro-and-ethics/index.md) | `robots-compliance-auditor` | `robots.txt` 协议解析与合规风险沙盒 | Evaluate (评价) | `p5.js` |
| [第 2 章](../chapters/02-web-and-http-fundamentals/index.md) | `http-status-header-triage` | HTTP 状态码与请求/响应头故障诊断台 | Analyze (分析) | `p5.js` |
| [第 2 章](../chapters/02-web-and-http-fundamentals/index.md) | `http-lifecycle-session-inspector` | HTTP 全链路报文与 Cookie/Token 会话追踪仪 | Understand (理解) | `p5.js` |
| [第 3 章](../chapters/03-python-http-requests/index.md) | `python-http-request-builder` | Python HTTP 请求参数与载荷组装实验室 | Apply (应用) | `p5.js` |
| [第 3 章](../chapters/03-python-http-requests/index.md) | `session-backoff-simulator` | TCP 连接复用与指数退避重试模拟器 | Evaluate (评价) | `Chart.js` |
| [第 4 章](../chapters/04-static-html-parsing/index.md) | `ch04-css-xpath-sandbox` | CSS 选择器与 XPath 双向定位沙盒 | Apply (应用) | `p5.js` |
| [第 4 章](../chapters/04-static-html-parsing/index.md) | `ch04-parser-benchmark-comparator` | 静态网页解析器选型决策与性能基准仪 | Evaluate (评价) | `Chart.js` |
| [第 5 章](../chapters/05-ajax-and-api-scraping/index.md) | `api-pagination-simulator` | 页码偏移 vs 游标（Cursor/MaxID）分页稳定性模拟器 | Analyze (分析) | `p5.js` |
| [第 5 章](../chapters/05-ajax-and-api-scraping/index.md) | `api-vs-dom-decision-matrix` | 异步 API 逆向与静态 DOM 采集工程权衡评估器 | Evaluate (评价) | `Chart.js` |
| [第 6 章](../chapters/06-browser-automation-playwright/index.md) | `ch06-selenium-vs-playwright-arch` | Selenium 与 Playwright 通信架构及渲染生命周期对比器 | Understand (理解) | `p5.js` |
| [第 6 章](../chapters/06-browser-automation-playwright/index.md) | `ch06-network-route-interceptor` | Playwright 网络路由拦截与 API 响应截获流水线 | Analyze (分析) | `p5.js` |
| [第 7 章](../chapters/07-anti-scraping-and-countermeasures/index.md) | `ch07-slider-captcha-trajectory-sim` | 滑块验证码生物动力学轨迹与风控评分模拟器 | Apply (应用) | `p5.js` |
| [第 7 章](../chapters/07-anti-scraping-and-countermeasures/index.md) | `ch07-js-signature-reverse-lab` | DevTools XHR 断点回溯与接口签名逆向沙盒 | Analyze (分析) | `p5.js` |
| [第 8 章](../chapters/08-concurrent-and-async-scraping/index.md) | `concurrency-model-timeline-simulator` | 同步、多线程与 `asyncio` 协程事件循环时序模拟器 | Analyze (分析) | `p5.js` |
| [第 8 章](../chapters/08-concurrent-and-async-scraping/index.md) | `semaphore-token-bucket-limiter` | `Semaphore` 并发阀门与令牌桶（Token Bucket）平滑限流沙盒 | Apply (应用) | `Chart.js` |
| [第 9 章](../chapters/09-data-cleaning-and-storage/index.md) | `etl-simhash-cleaning-workbench` | 脏数据 ETL 清洗流水线与 SimHash 海明距离去重实验台 | Apply (应用) | `p5.js` |
| [第 9 章](../chapters/09-data-cleaning-and-storage/index.md) | `storage-bloom-filter-calculator` | 多源存储选型与布隆过滤器（Bloom Filter）空间误判率计算器 | Evaluate (评价) | `Chart.js` |
| [第 10 章](../chapters/10-scrapy-industrial-framework/index.md) | `scrapy-architecture-dataflow` | Scrapy 六大核心组件与异步数据流转沙盒 | Understand (理解) | `p5.js` |
| [第 10 章](../chapters/10-scrapy-industrial-framework/index.md) | `scrapy-production-cluster-configurator` | `Scrapy-Redis` 分布式集群与 `AutoThrottle` 调优模拟器 | Evaluate (评价) | `Chart.js` |
| [第 11 章](../chapters/11-ai-powered-scraping-and-datasets/index.md) | `ch11-llm-self-healing-scraper` | LLM 零样本网页解析与 XPath 选择器自愈修复工作台 | Apply (应用) | `p5.js` |
| [第 11 章](../chapters/11-ai-powered-scraping-and-datasets/index.md) | `ch11-rag-chunking-pipeline-sim` | 面向 RAG 与 SFT 的智能网页语料清洗及分块沙盒 | Evaluate (评价) | `p5.js` |
| [第 12 章](../chapters/12-engineering-deployment-and-capstone/index.md) | `ch12-drift-monitor-and-snr-auditor` | 页面结构漂移巡检与数据集信噪比质量审计仪 | Evaluate (评价) | `Chart.js` |
| [第 12 章](../chapters/12-engineering-deployment-and-capstone/index.md) | `ch12-capstone-architecture-designer` | 企业级爬虫技术栈选型与故障巡检架构沙盒 | Create (创造) | `p5.js` |
