# 核心术语中英对照表（Glossary）

本术语表汇总了《网络爬虫技术》全书 12 章核心专业词汇的中英文对照与简明工程释义，按英文字母顺序排列。

---

## A

- **AJAX (Asynchronous JavaScript and XML，异步 JavaScript 和 XML)**：浏览器无需重新加载整个网页即可通过后台 `XMLHttpRequest` 或 `Fetch API` 与服务器交换 JSON 数据的前端通信技术。
- **Alpaca Format (Alpaca 指令微调格式)**：大模型有监督微调（SFT）的标准 JSON 数据格式，包含 `instruction`、`input`、`output` 和可选的 `system` 字段。
- **AST (Abstract Syntax Tree，抽象语法树)**：源代码语法结构的树状表示形式，在反爬虫对抗中常用于对混淆后的前端 JavaScript 代码进行自动化还原。
- **Auto-Waiting (自动等待机制)**：Playwright 内置的智能等待机制，在执行点击或填写操作前自动校验目标元素是否已挂载、可见、稳定且未被遮挡。

## B

- **Bloom Filter (布隆过滤器)**：一种基于位数组与多个哈希函数的极高空间效率概率型数据结构，广泛用于亿级 URL 分布式去重。
- **Boilerplate Removal (样板噪音剔除)**：在网页正文抽取时，自动剥离页眉导航栏、侧边栏广告、推荐链接和页脚版权声明等非核心文本的过程。
- **BrowserContext (浏览器隔离上下文)**：Playwright 在单个浏览器进程内创建的轻量级无痕沙盒会话，不同上下文之间的 Cookie、本地存储与代理 IP 完全隔离。

## C

- **CDP (Chrome DevTools Protocol，Chrome 开发者工具协议)**：Chromium 内核暴露的底层 WebSocket 调试协议，Playwright 与 DrissionPage 均通过该协议实现高性能自动化控制。
- **Chunking (文档语义分块)**：在构建 RAG 向量知识库时，将长篇 Markdown 文档按章节标题与段落边界切分为适中长度（如 300～800 Tokens）并保留重叠窗口与元数据的过程。
- **CORS (Cross-Origin Resource Sharing，跨域资源共享)**：浏览器基于同源策略实施的安全机制，通过 `Access-Control-Allow-Origin` 等响应头控制跨域异步请求权限。
- **CSR (Client-Side Rendering，客户端渲染)**：服务端仅返回空 HTML 骨架与 JS 脚本，由浏览器执行 JS 请求接口并在本地动态生成 DOM 节点的渲染模式。

## D

- **DOM (Document Object Model，文档对象模型)**：将 HTML 或 XML 文档在内存中表示为父、子、兄弟节点相连的树形拓扑结构。

## E

- **Event Loop (事件循环)**：异步编程（如 Python `asyncio` 与浏览器 JS 引擎）的核心调度中枢，在单线程内通过非阻塞 I/O 事件通知高效并发切换成千上万个协程任务。
- **Exponential Backoff (指数退避重试)**：网络请求失败后，每次重试的等待间隔呈指数级递增（如 2s、4s、8s）并叠加随机抖动（Jitter），以保护目标服务器并提高恢复率。

## G

- **GIL (Global Interpreter Lock，全局解释器锁)**：CPython 解释器中的互斥锁，限制同一时刻单个进程内仅有一个线程执行 Python 字节码，适合 I/O 密集型并发但限制了多线程 CPU 密集型计算。

## J

- **JA3 Fingerprint (TLS JA3 指纹)**：根据客户端在 TLS 握手 `Client Hello` 阶段发送的加密套件、扩展与椭圆曲线列表计算出的 32 位 MD5 特征值，常被 WAF 用于识别非浏览器请求。
- **JSON Lines (JSONL，行分隔 JSON 格式)**：每行存储一个独立合法 JSON 对象的流式文本格式（`.jsonl`），是爬虫流式追加落盘与大模型训练语料交付的工业标准格式。

## M

- **MinHash LSH (最小哈希与局部敏感哈希)**：用于在大规模预训练语料清洗中快速检测并剔除近似重复长文档（如改动少量字词的转载文章）的集合相似度算法。

## P

- **PII (Personally Identifiable Information，个人可识别身份信息)**：包括手机号、身份证号、电子邮箱、银行卡号等公民隐私信息，在语料入库前必须执行脱敏擦除。
- **Proxy Pool (高可用代理 IP 池)**：集中采集、异步健康拨测评分并按需为爬虫分配高匿出口 IP 的分布式中间件系统。

## R

- **RAG (Retrieval-Augmented Generation，检索增强生成)**：在向大语言模型提问时，先从外部向量数据库中检索相关最新文档片段注入上下文，以消除模型幻觉并实现知识实时更新的架构。
- **Robots Exclusion Protocol (`robots.txt`，君子协定)**：存放于网站根目录下的文本文件，声明允许或禁止特定网络爬虫抓取的路径范围与速率限制。

## S

- **Scrapy-Redis (分布式爬虫组件)**：将 Scrapy 默认的本地内存调度队列与去重集合迁移至共享 Redis 实例，实现多机多进程协同采集的分布式扩展库。
- **SSR (Server-Side Rendering，服务端渲染)**：由服务器直接将业务数据填充进 HTML 模板并返回完整网页源码的渲染模式。

## T

- **Token Bucket (令牌桶算法)**：以恒定速率向桶内放入令牌、请求前需先获取令牌的平滑限流算法，用于在异步高并发采集中严格控制 QPS 请求速率。
- **Trafilatura (启发式正文抽取引擎)**：Python 生态中准确率领先的网页正文抽取与噪音过滤库，支持将杂乱 HTML 直接转换为保留表格与代码块的 Markdown 语料。

## W

- **WebFont Encryption (自定义字体加密反爬)**：网页通过 `@font-face` 引入动态 `.woff2` 字体文件并将核心数字替换为私有 Unicode 编码的反爬手段，可通过 `fontTools (TTFont)` 解析字形映射还原。

## X

- **XPath (XML Path Language，XML 路径语言)**：支持按层级路径、属性谓语、文本模糊包含（`contains`）以及父节点/兄弟节点轴向反查 DOM 树节点的强表达力查询语言。
