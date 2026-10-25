---
title: "Web前端基石与HTTP通信原理"
description: "深入拆解URI/URL语法与编码、DNS/TCP/TLS握手流程、HTTP请求与响应报文结构、状态码与核心请求头、Cookie/Session/JWT会话机制及Chrome DevTools抓包技术。"
generated_by: claude skill chapter-content-generator
date: "2026-10-25 18:18:00"
version: 1.11
---

# Web前端基石与HTTP通信原理

## Summary

网络爬虫的本质是模拟客户端与 Web 服务器进行标准协议通信。本章深入拆解从浏览器输入 URL 到页面呈现背后的完整网络链路，涵盖 URI/URL 编码规范、DNS 解析、TCP 三次握手、TLS 加密传输以及 HTTP 请求/响应报文格式。通过详解常见状态码、核心请求头（`User-Agent`、`Referer`、`Content-Type`）与 Cookie/Session/JWT 会话跟踪机制，并配合 Chrome DevTools 的 Network 抓包实操，帮助学生具备直接阅读和诊断底层网络流量的核心能力。

## Concepts Covered

This chapter covers the following 22 concepts from the learning graph:

| Concept | Concept Impact Score |
|---------|-----------------------|
| URI与URL语法结构剖析 | 3820 |
| URL百分号编码与解码 | 366 |
| DNS域名解析流程 | 3440 |
| TCP三次握手与连接建立 | 3439 |
| TLS加密握手与HTTPS原理 | 13 |
| HTTP请求与响应模型 | 3340 |
| HTTP请求方法GET与POST | 452 |
| HTTP状态码分类体系 | 325 |
| 常见成功与重定向状态码 | 2 |
| 客户端与服务端错误状态码 | 167 |
| HTTP请求头Headers结构 | 711 |
| User-Agent客户端标识 | 191 |
| Referer防盗链与来源校验 | 187 |
| Content-Type媒体类型声明 | 131 |
| Accept与内容协商机制 | 1 |
| HTTP响应头与Gzip压缩 | 3 |
| HTTP无状态特性与Cookie原理 | 174 |
| 服务端Session会话跟踪 | 58 |
| Token与JWT身份认证机制 | 1 |
| HTTP/1.1持久连接与HTTP/2多路复用 | 8 |
| Chrome开发者工具DevTools基础 | 308 |
| Network面板抓包与HAR导出 | 192 |

## Prerequisites

This chapter builds on concepts from:

- [Chapter 1: 网络爬虫导论与数据合规基石](../01-intro-and-ethics/index.md)

---

## 1. 寻址与连接：从输入网址到建立安全传输通道

在编写任何一行网络爬虫代码之前，我们首先要弄清楚一个最核心的底层问题：**当你在浏览器地址栏输入一个网址并按下回车键的那一刻，你的计算机究竟与远端服务器经历了怎样的对话？** 初学者在写爬虫时最容易陷入“只背代码模板、不懂通信协议”的误区，一旦遇到网址乱码、连接超时（Timeout）、SSL 证书报错或 `403 Forbidden` 拦截便束手无策。事实上，网络爬虫并非什么神秘的黑魔法，它只是一个严格遵循互联网标准协议的自动化客户端。理解从 URL 寻址到底层安全连接建立的完整链路，是精准构造爬虫请求的第一步。

### 1.1 URI与URL语法结构剖析

在万维网规范中，我们经常同时听到 **URI（Uniform Resource Identifier，统一资源标识符）** 和 **URL（Uniform Resource Locator，统一资源定位符）** 两个概念。对**URI与URL语法结构剖析**可以从“身份证号”与“家庭详细住址”的比喻入手：URI 是一个更宽泛的顶层概念，任何能够唯一标识互联网上某个资源的字符串都叫 URI（包含只标明名称但不说明如何找到它的 URN，如图书的 ISBN 编号 `urn:isbn:978-7-115-54608-1`）；而 **URL 是 URI 最核心的子集**，它不仅标识了资源是谁，还明确给出了“通过什么通信协议、去哪台主机、在哪个目录路径下获取该资源”的完整定位方法。在爬虫开发语境中，我们日常处理的网址几乎 100% 都是标准 URL。

一个完整的标准 URL 严格遵循 RFC 3986 语法规范，由六个核心组件按固定分隔符拼接而成：

$$\text{scheme} :// \text{host} : \text{port} / \text{path} ? \text{query} \# \text{fragment}$$

让我们以人工智能开源模型社区的一个真实搜索网址为例，逐一拆解这六个组成部分在爬虫工程中的具体作用：

`https://models.example.edu:8443/v1/datasets/search?category=人工智能&page=2&sort=downloads#top-results`

1. **协议方案（`scheme`）**：如 `https` 或 `http`（在实时流采集中还会遇到 `ws` 或 `wss` WebSocket 协议）。它决定了客户端与服务器之间采用明文传输还是 TLS 加密传输，同时决定了默认端口号。
2. **主机名（`host`）**：如 `models.example.edu`，也可以是直接的 IPv4 地址（如 `114.114.114.114`）。它指明了目标 Web 服务器在互联网上的主机位置。
3. **端口号（`port`）**：紧跟在主机名冒号后的整数，如 `:8443`。在实际网页URL中，端口号通常被省略——当省略时，**`http` 协议默认使用 `80` 端口，`https` 协议默认使用 `443` 端口**。
4. **资源路径（`path`）**：如 `/v1/datasets/search`。它采用类似 Unix 文件系统的斜杠分层结构，映射到服务器端的路由控制器（Router）或静态文件目录。
5. **查询参数（`query`）**：以问号 `?` 开头，多个参数之间用 `&` 符号连接，每个参数采用 `key=value` 键值对形式（如 `category=人工智能&page=2&sort=downloads`）。**查询参数是网络爬虫构造动态翻页（修改 `page=2` 为 `page=3`）与关键词检索的核心控制点。**
6. **锚点片段（`fragment`）**：以井号 `#` 开头（如 `#top-results`），用于在浏览器端将滚动条直接跳转到网页内部的某个指定 HTML 元素位置。**爬虫工程师必须牢记一条重要铁律：URL 中的 `#fragment` 锚点仅在浏览器本地生效，浏览器或爬虫在向服务器发送 HTTP 请求时，绝不会把 `#` 及其后面的内容发送给服务器！**（但部分早期单页应用 SPA 会利用 `#/ detail` 实现前端路由，需在客户端渲染章节特殊处理）。

### 1.2 URL百分号编码与解码

在上面的 URL 示例中，查询参数包含了中文字符 `category=人工智能`。然而，早期的互联网底层协议（RFC 3986）规定，标准 URL 中只允许直接出现 **ASCII 字母、数字以及少数安全保留字符（如 `-`、`_`、`.`、`~`）**。如果 URL 中直接包含中文字符、空格、或者具有特殊分隔语法含义的字符（如 `&`、`=`、`?`、`/`、`#`），就会导致服务器解析路由时发生歧义甚至截断报错。

为了解决这一问题，万维网引入了 **URL百分号编码与解码（Percent-Encoding，俗称 URL 编码）** 机制。其编码规则非常直观：

1. 将非 ASCII 字符（如中文汉字）首先按照目标字符集（现代 Web 标准统一为 **UTF-8**，每个常见中文字符占 3 个字节）转换为底层的十六进制字节序列；
2. 在每个字节的两个十六进制数字前加上百分号 `%`。例如，“人工智能”四个汉字的 UTF-8 十六进制字节分别为 `E4 B8 BA`（人）、`E5 B7 A5`（工）、`E6 99 BA`（智）、`E8 83 BD`（能），经过百分号编码后就会变成 `%E4%BA%BA%E5%B7%A5%E6%99%BA%E8%83%BD`。
3. 空格字符在 URL 路径中通常被编码为 `%20`，而在查询参数（`application/x-www-form-urlencoded` 格式）中则常被编码为加号 `+`。

在 Python 中，我们无需手动查表计算字节，标准库 `urllib.parse` 提供了极其强大的组件拆解函数 `urlsplit()`、参数字典编码函数 `urlencode()` 以及单字符串编码/解码函数 `quote()` 与 `unquote()`。下面的代码示例展示了如何将一条复杂的原始 URL 拆解为结构化组件，并安全地修改其翻页与中文搜索词参数：

```python
from urllib.parse import parse_qs, quote, unquote, urlencode, urlsplit, urlunsplit

# 1. 剖析包含中文查询参数与锚点的完整 URL
raw_url = "https://models.example.edu:8443/v1/datasets/search?category=人工智能&page=1#top-results"
parsed = urlsplit(raw_url)

print(f"协议 (scheme)  : {parsed.scheme}")
print(f"主机与端口     : {parsed.netloc} (主机={parsed.hostname}, 端口={parsed.port})")
print(f"资源路径 (path): {parsed.path}")
print(f"原始查询串     : {parsed.query}")
print(f"锚点 (fragment): {parsed.fragment}  <-- 注意：此部分不会发送给服务器")

# 2. 单字符串百分号编码与解码 (quote / unquote)
keyword = "大模型 语料&清洗"
encoded_kw = quote(keyword, encoding="utf-8")
print(f"\n百分号编码结果 : {encoded_kw}")
print(f"百分号解码还原 : {unquote(encoded_kw)}")

# 3. 批量解析查询参数并构造第 2 页的新 URL
query_dict = parse_qs(parsed.query)
print(f"\n解析出的参数字典: {query_dict}")

# 修改字典参数并重新编码为合法的 URL 查询字符串
new_params = {
    "category": "具身智能",
    "page": 2,
    "sort": "downloads",
}
new_query_string = urlencode(new_params, encoding="utf-8")

# 重新组装为完整无锚点的爬虫请求 URL
target_crawl_url = urlunsplit(
    (parsed.scheme, parsed.netloc, parsed.path, new_query_string, "")
)
print(f"构造的新抓取URL: {target_crawl_url}")
```

### 1.3 DNS域名解析、TCP三次握手与TLS加密握手

当爬虫拿到了编码规范的目标 URL（例如 `https://models.example.edu/v1/datasets/search`）并调用发送函数时，在真正的 HTTP 报文发出之前，操作系统底层网络栈必须依次完成三项关键的先导动作：**DNS域名解析流程**、**TCP三次握手与连接建立**以及**TLS加密握手与HTTPS原理**。

1. **DNS域名解析流程（Domain Name System Resolution）**  
   TCP/IP 网络层的路由器只认识形如 `93.184.216.34` 的 IP 地址，不认识 `models.example.edu` 这样的字符域名。因此客户端首先按“由近及远”的缓存优先级发起 DNS 查询：
   - **浏览器与操作系统缓存 $\rightarrow$ 本地 `hosts` 文件**：若近期刚访问过该域名且未过 TTL（生存时间）有效期，直接秒级返回 IP；
   - **本地递归 DNS 服务器（如运营商 DNS 或公共 DNS `223.5.5.5`/`8.8.8.8`）**：若本地未命中，递归服务器依次向根域名服务器（`.`）、顶级域服务器（`.edu`）和权威域名服务器查询，最终取回目标主机的真实 IP 地址。  
   *爬虫工程启示*：如果爬虫在短时间内向数千个不同域名发起请求，DNS 解析耗时就会成为瓶颈，此时在异步爬虫中启用 **DNS 缓存池（DNS Caching）** 能够显著降低请求延迟。
2. **TCP三次握手与连接建立（TCP Three-Way Handshake）**  
   拿到目标服务器 IP 地址与端口号（HTTPS 为 `443`）后，客户端操作系统内核与目标服务器之间必须通过 **TCP（传输控制协议）** 建立一条可靠的、面向连接的字节流通道，这就是著名的“三次握手”：
   - **第一次握手（`SYN`）**：客户端向服务器发送同步标志包 `SYN=1, seq=x`，询问：“你好，我想和你建立连接，我的初始序列号是 $x$，你能听到吗？”
   - **第二次握手（`SYN + ACK`）**：服务器收到后回复确认包 `SYN=1, ACK=1, ack=x+1, seq=y`，表示：“我听到了，你的序列号 $x$ 已确认；我也准备好向你发送数据，我的初始序列号是 $y$。”
   - **第三次握手（`ACK`）**：客户端收到后再次向服务器发送确认包 `ACK=1, ack=y+1`，表示：“收到你的序列号 $y$，连接正式建立！”  
   *爬虫工程启示*：每次新建 TCP 连接都需要耗费至少 1 个 **RTT（Round-Trip Time，网络往返时延，通常为几十到几百毫秒）**。如果爬虫每抓一个页面都新建并断开一次 TCP 连接，90% 的时间都会浪费在握手等待上！
3. **TLS加密握手与HTTPS原理（TLS Handshake & HTTPS）**  
   早期的 `http://` 协议直接在 TCP 通道上明文裸奔传输报文，中间的网络路由器、公共 Wi-Fi 甚至运营商都可以轻易窃听或篡改网页内容（例如插入弹窗广告或窃取登录密码）。现代互联网已全面普及 **HTTPS（HTTP Secure，即 HTTP over TLS）**。在 TCP 三次握手完成后，客户端与服务器立即进行 **TLS（传输层安全协议）加密握手**：
   - 客户端发送 `ClientHello`（声明自己支持的 TLS 版本如 TLS 1.3、加密套件列表及客户端随机数）；
   - 服务器返回 `ServerHello`、**服务器数字证书（含公钥）**及服务器随机数；
   - 客户端校验数字证书是否由受信任的 CA 机构签发且域名匹配，随后双方利用非对称加密（如 ECDHE 密钥交换算法）安全协商出仅本次会话有效的**对称会话密钥（Session Key）**；
   - 后续所有的 HTTP 请求头、Cookie 和网页 HTML 源码，全部使用该对称密钥进行高强度加密（如 AES-256-GCM）后再通过 TCP 发送。  
   *爬虫工程启示*：许多高级反爬虫系统会检测客户端在 `ClientHello` 阶段发送的加密套件排列顺序——即 **TLS/JA3 指纹**（将在第 7 章深入攻克）；同时，当我们在爬虫中使用抓包工具（如 Charles、Fiddler、Mitmproxy）或访问自签名证书的内网网站时，必须理解数字证书校验机制才能正确配置信任证书或豁免校验。

### 1.4 HTTP/1.1持久连接与HTTP/2多路复用

随着网页包含的资源越来越丰富，HTTP 协议自身的传输效率也经历了三次重大代际演进，理解 **HTTP/1.1持久连接与HTTP/2多路复用** 直接关系到我们如何选型高性能爬虫客户端：

- **HTTP/1.0（短连接时代）**：默认每发送一次请求、接收一次响应后，立即关闭 TCP 连接。抓取同一网站的 100 张图片需要重复执行 100 次 TCP 三次握手。
- **HTTP/1.1（持久连接 Keep-Alive 时代）**：默认启用 `Connection: keep-alive` 持久连接。客户端与服务器建立一次 TCP+TLS 连接后，该通道保持打开状态，后续对同一域名的数十次请求可以直接复用这条通道！这也是为什么在第 3 章中我们强烈要求使用 `requests.Session()` 而不是裸调 `requests.get()` 的底层原因。但 HTTP/1.1 仍存在**队头阻塞（Head-of-Line Blocking）**缺陷：同一条 TCP 连接上请求必须严格按先后顺序串行一问一答，前一个慢请求会堵死后面的所有请求。
- **HTTP/2（二进制分帧与多路复用时代）**：现代主流网站与强反爬站点普遍采用 HTTP/2。它彻底摒弃了纯文本按行传输的底层方式，引入**二进制分帧层（Binary Framing）**、**头部压缩（HPACK）**与**多路复用（Multiplexing）**——允许在**单条 TCP+TLS 连接上同时并发交错发送数十个请求和响应流（Streams）**，彻底消除了应用层队头阻塞。在 Python 爬虫中，传统的 `requests` 库仅支持 HTTP/1.1，而新一代客户端 `httpx` 与 `curl_cffi` 则原生支持 HTTP/2 协议通信。

下表总结了从 URL 解析到应用层协议演进的核心技术要点：

| 协议/阶段名称 | 所处网络层级 | 核心职责与产出 | 爬虫工程实战影响 |
|---|---|---|---|
| **URL 解析与百分号编码** | 应用层寻址 | 拆解协议/主机/路径/查询串，将非 ASCII 字符转为 `%XX` | 防止中文参数截断乱码；忽略 `#fragment` 锚点 |
| **DNS 域名解析** | 网络层寻址辅助 | 将域名（如 `example.com`）映射为服务器 IP 地址 | 大规模跨站抓取时需开启本地 DNS 缓存避免解析超时 |
| **TCP 三次握手** | 传输层（Port 80/443） | 通过 `SYN` $\rightarrow$ `SYN+ACK` $\rightarrow$ `ACK` 建立可靠双向连接 | 每次新建连接耗费 1 RTT，必须通过连接池复用 |
| **TLS 加密握手** | 安全传输层（HTTPS） | 校验 CA 数字证书，协商对称会话密钥加密后续报文 | 自签名证书会触发 `SSLError`；`ClientHello` 暴露 JA3 指纹 |
| **HTTP/1.1 Keep-Alive** | 应用层协议 | 默认开启长连接复用，但单连接内请求仍需串行排队 | 使用 `requests.Session` 复用底层 TCP 连接可提速 3~5 倍 |
| **HTTP/2 多路复用** | 应用层协议 | 二进制分帧、HPACK 头部压缩、单连接并发多流传输 | 部分高防网站强制校验 HTTP/2 协议特征，需用 `httpx` 应对 |

---

## 2. HTTP请求与响应报文深度解剖

当底层 TCP 与 TLS 安全通道搭建完毕后，客户端与服务器正式开始应用层对话。**HTTP请求与响应模型（HTTP Request-Response Model）** 采用严格的“客户端主动发问、服务器被动应答”机制。无论是复杂的网页浏览还是 AI 接口调用，在网络线路上传输的都只是结构极其规整的 **HTTP 请求报文（Request Message）** 与 **HTTP 响应报文（Response Message）**。学会像阅读普通中文句子一样直接阅读原始 HTTP 报文，是区分“初级代码搬运工”与“专业爬虫工程师”的分水岭。

### 2.1 HTTP请求报文结构与GET/POST方法对比

一个标准的 HTTP/1.1 请求报文由四个部分组成，各部分之间以回车换行符（`CRLF`，即 `\r\n`）分隔，其中**请求头与请求体之间必须有一行完全空白的空行（`\r\n\r\n`）**：

```http
POST /api/v1/corpus/query?lang=zh HTTP/1.1
Host: data.example.edu
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/130.0.0.0 Safari/537.36
Content-Type: application/json; charset=utf-8
Accept: application/json, text/plain, */*
Referer: https://data.example.edu/search
Cookie: session_id=xyz987654321

{"keyword": "多模态大模型", "page": 1, "page_size": 20}
```

1. **请求行（Request Line — 第 1 行）**：包含三个以空格分隔的字段：**请求方法（Method）**、**请求目标路径与查询参数（Request-URI）**、**HTTP 协议版本（Version）**。
2. **请求头（Request Headers — 第 2~7 行）**：由若干行 `Key: Value` 键值对组成，用于向服务器声明客户端的环境信息、期望的数据格式、来源页面及身份凭证。
3. **空行（Blank Line — 第 8 行）**：仅包含 `\r\n`，作为请求头结束、请求体开始的标志边界。
4. **请求体 / 载荷（Request Body / Payload — 第 9 行）**：客户端实际提交给服务器的业务数据（如登录账号密码、搜索过滤 JSON 条件或上传的文件字节流）。

在 HTTP 协议定义的多种请求方法（`GET`、`POST`、`PUT`、`DELETE`、`HEAD`、`OPTIONS`、`PATCH`）中，网络爬虫 99% 的场景都在与 **HTTP请求方法GET与POST** 打交道。二者在语义、参数位置与幂等性上存在本质区别：

- **`GET` 方法（获取资源）**：语义是从服务器**只读检索**指定资源（如打开新闻列表页、加载图片、按关键词搜索）。`GET` 请求的所有参数都直接拼接在 URL 的 `?` 查询字符串后面，**通常不包含请求体（Request Body 为空）**。因为 URL 长度受浏览器和 Web 服务器（如 Nginx）配置限制（通常为 2KB~8KB），`GET` 不适合传输大段文本或二进制文件。此外，`GET` 请求具有**幂等性（Idempotent）**——重复请求 1 次和请求 100 次对服务器资源状态不产生任何副作用。
- **`POST` 方法（提交载荷与执行动作）**：语义是向服务器**提交数据以触发状态变更或复杂计算**（如提交用户登录表单、调用复杂的多条件筛选异步接口、向大模型 API 发送长 Prompt）。`POST` 请求将核心业务参数放在 **请求体（Request Body）** 中传输，在 URL 地址栏中不可见，且数据体积理论上不受限制。在爬虫逆向中，一旦看到目标接口是 `POST` 请求，我们不仅要关注 URL，更要重点檢查它的**请求体载荷内容**以及声明载荷格式的 **`Content-Type` 请求头**。

### 2.2 HTTP响应报文结构与状态码分类体系

服务器处理完客户端的请求后，会原路发回一份 **HTTP 响应报文**。其结构同样由四部分组成：**状态行（Status Line）**、**响应头（Response Headers）**、**空行（`\r\n`）** 和 **响应体（Response Body / Payload）**：

```http
HTTP/1.1 200 OK
Date: Sun, 25 Oct 2026 10:15:30 GMT
Server: nginx/1.26.0
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Set-Cookie: visited_flag=true; Path=/; HttpOnly

<!DOCTYPE html>
<html lang="zh-CN">
<head><title>AI语料中心</title></head>
<body><h1>欢迎访问开源人工智能数据集仓库</h1></body>
</html>
```

状态行中的三位数字（如 `200`）被称为 **HTTP状态码分类体系（HTTP Status Codes）**。它是服务器给爬虫返回的“第一眼体检报告”——爬虫无需解析庞大的响应体，仅凭这三位数字就能立刻判断本次请求是成功拿到数据、被重定向跳转、还是触发了反爬虫封禁。HTTP 状态码按首位数字严格划分为五大类（`1xx`~`5xx`），其中爬虫开发必须熟记于心的是 **常见成功与重定向状态码** 与 **客户端与服务端错误状态码**：

1. **`2xx` 成功状态码（Success）**
   - **`200 OK`**：爬虫最期待看到的状态码，表示请求完全成功，响应体中包含了目标 HTML 源码或 JSON 数据。
   - **`204 No Content`**：服务器成功处理了请求，但响应体故意为空（常见于埋点上报或 `OPTIONS` 预检请求）。
   - **`206 Partial Content`**：范围请求成功，常见于爬虫使用 `Range: bytes=0-1024` 请求头进行大文件或音视频的分块断点续传下载。
2. **`3xx` 重定向状态码（Redirection）**
   - **`301 Moved Permanently`（永久重定向）** 与 **`302 Found` / `307 Temporary Redirect`（临时重定向）**：表示你请求的原网址已经搬家，服务器会在响应头的 `Location: https://new.example.edu/login` 字段中给出新网址。**爬虫工程极其隐蔽的踩坑点**：许多网站在发现你未登录或触发反爬时，并不是直接返回 `403`，而是返回 `302` 将你重定向到登录页或验证码页！如果爬虫默认开启了自动跟随重定向，最后拿到的状态码虽然也是 `200 OK`，但响应体其实是登录页的 HTML，导致解析出空数据。
   - **`304 Not Modified`（未修改缓存）**：当增量爬虫在请求头中附带了上次抓取时的 `If-Modified-Since` 或 `If-None-Match (ETag)` 时，若服务器发现网页内容自上次以来未发生任何变化，就会返回 `304` 且不返回响应体，从而为爬虫节省 100% 的正文下载流量。
3. **`4xx` 客户端错误状态码（Client Error — 反爬对抗主战场）**
   - **`400 Bad Request`**：请求报文语法错误，通常是爬虫拼接的 URL 未正确进行百分号编码，或者 `POST` 提交的 JSON 格式缺少引号/括号。
   - **`401 Unauthorized`**：未授权访问，说明目标接口需要携带有效的登录凭证（如 `Cookie` 或 `Authorization: Bearer <Token>`）。
   - **`403 Forbidden`**：服务器理解你的请求，但**严词拒绝执行**！这是爬虫工程师最常遭遇的反爬虫拦截码，通常意味着你的 `User-Agent` 暴露了 Python 脚本身份、缺少合法的 `Referer` 防盗链头、或者当前 IP 被防火墙拉黑。
   - **`404 Not Found`**：请求的 URL 资源在服务器上不存在（如拼接了错误的详情页 ID 或文章已被删除），遇到 `404` 应直接记录日志并跳过，**绝不要对 `404` 执行重试**。
   - **`429 Too Many Requests`**：请求频率过高触发限流（Rate Limiting）！服务器明确警告你在单位时间内发送了过多请求，此时爬虫必须立即降速、休眠退避或切换代理 IP。
4. **`5xx` 服务端错误状态码（Server Error — 需退避重试）**
   - **`500 Internal Server Error`**：目标服务器内部代码抛出未捕获异常（有时是因为爬虫传入了越界的分页参数如 `page=-1` 导致后端数据库报错）。
   - **`502 Bad Gateway` / `503 Service Unavailable` / `504 Gateway Timeout`**：目标网站的网关（如 Nginx/Cloudflare）无法从后端应用服务器拿到响应，或服务器当前过载维护（注意：Cloudflare 五秒盾反爬验证页面也经常返回 `503` 状态码）。对于真实的 `502/503/504` 瞬时过载，爬虫应当通过**指数退避策略（Exponential Backoff）**等待几秒后自动重试。

### 2.3 HTTP请求头Headers结构与响应头解析

在 HTTP 报文中，真正决定爬虫“像不像一个真实人类浏览器”的核心区域就是 **HTTP请求头Headers结构**。当我们使用 Python 的 `requests` 库且不传任何自定义配置时，它默认发出的 `User-Agent` 请求头是 `python-requests/2.32.3`——这无异于在脑门上贴着“我是爬虫，快来封我”的标签去敲服务器大门！下面深入剖析爬虫开发中最关键的五大请求头与响应头机制：

1. **`User-Agent` 客户端标识（User-Agent Client String）**  
   `User-Agent`（简称 UA）用于向服务器报告当前客户端的操作系统版本、CPU 架构、渲染引擎及浏览器版本号。例如一段标准的 Windows 11 Chrome 浏览器 UA 为：  
   `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36`  
   在爬虫工程中，将 `User-Agent` 替换为真实主流浏览器的字符串是最基础的“伪装第一步”。此外，部分网站针对手机端（移动端 UA，包含 `iPhone` 或 `Android`）返回的 HTML 结构比电脑端更简洁、反爬更弱，巧妙切换移动端 UA 往往能事半功倍。
2. **`Referer` 防盗链与来源校验（Referer Anti-Hotlinking）**  
   `Referer`（注：HTTP 早期规范拼写漏了一个 `r`，沿用至今）告诉服务器：“我是从哪个上一个网页点击链接跳转到当前这个网址的？”  
   **核心实战场景**：在采集图片网站、音视频流或异步 JSON 接口时，服务端普遍开启 **Referer 防盗链校验**。例如，图片地址位于 `https://img.example.com/photo/101.jpg`，如果你直接在爬虫里裸请求这个地址，服务器发现请求头中没有 `Referer` 或者 `Referer` 不是自家主站域名 `https://www.example.com/`，就会直接返回 `403 Forbidden` 或一张“禁止盗链”的假图片。只需在请求头中补上 `Referer: https://www.example.com/` 即可瞬间破局。
3. **`Content-Type` 媒体类型声明（MIME Type Declaration）**  
   在 `POST` 请求头和所有响应头中，`Content-Type` 用于明确声明当前报文体（Body）的数据编码格式。爬虫必须严格区分三种最常见的 `POST` 请求 `Content-Type`：
   - `application/x-www-form-urlencoded`：传统 HTML 表单格式，请求体形如 `username=admin&role=student`（在 `requests` 中对应 `data=dict` 参数）；
   - `application/json; charset=utf-8`：现代前后端分离 API 标准格式，请求体是序列化的 JSON 字符串 `{"username": "admin"}`（在 `requests` 中对应 `json=dict` 参数）；
   - `multipart/form-data; boundary=----WebKitFormBoundary...`：用于文件上传或包含二进制流的多部分表单（在 `requests` 中对应 `files=...` 参数）。  
   **常见报错根源**：如果后端接口要求 `application/json`，而爬虫却以默认的表单格式发送，服务器会立即返回 `400 Bad Request` 或 `415 Unsupported Media Type`。
4. **`Accept` 与内容协商机制（Content Negotiation）**  
   客户端通过 `Accept` 系列请求头向服务器声明自己能够处理的响应格式：
   - `Accept`：期望接收的媒体类型（如 `application/json, text/javascript, */*; q=0.01`，其中 `q` 代表权重优先级）；
   - `Accept-Language`：期望的自然语言（如 `zh-CN,zh;q=0.9,en;q=0.8`，采集海外多语言站点时修改此头可直接获取中文版页面）；
   - `Accept-Encoding`：客户端支持的压缩算法（如 `gzip, deflate, br, zstd`）。
5. **HTTP响应头与Gzip压缩（Response Headers & Compression）**  
   为了节省网络传输带宽，Web 服务器在发送 HTML 或 JSON 文本前，通常会使用 `gzip` 或 Google 的 `br`（Brotli）算法将原本 500KB 的文本压缩为 60KB，并在响应头中标注 `Content-Encoding: gzip`。主流 Python HTTP 客户端（`requests`/`httpx`）会自动解压 `gzip` 和 `deflate`；但**如果我们在自定义请求头时手动写死了 `Accept-Encoding: gzip, deflate, br`，而当前 Python 环境未安装 `brotli` 第三方库，当服务器返回 `br` 压缩流时，爬虫打印出的 `response.text` 就会是一堆无法阅读的二进制乱码！** 遇到此类乱码时，要么执行 `pip install brotli`，要么从请求头的 `Accept-Encoding` 中删掉 `br`。

下面我们通过一段无需依赖外部网站、完全在本地利用 Python 内置库解析原始 HTTP 请求与响应报文的实战脚本，直观演示爬虫底层是如何拆解请求行、状态码、Headers 字典与 JSON/Gzip 载荷的：

```python
import gzip
import json
from email.parser import BytesParser

# 模拟抓包截获的原始 HTTP POST 请求字节流
RAW_HTTP_REQUEST = (
    b"POST /api/v1/datasets/query?source=open HTTP/1.1\r\n"
    b"Host: ai-hub.example.edu\r\n"
    b"User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/130.0.0.0\r\n"
    b"Referer: https://ai-hub.example.edu/explore\r\n"
    b"Content-Type: application/json; charset=utf-8\r\n"
    b"\r\n"
    b'{"task": "text-classification", "min_samples": 10000}'
)


def parse_raw_http_request(raw_bytes: bytes) -> dict:
    """拆解原始 HTTP 请求报文的请求行、Headers 字典与 Body 载荷"""
    # 1. 按第一个空行 \r\n\r\n 切分头部区域与请求体区域
    header_part, body_bytes = raw_bytes.split(b"\r\n\r\n", maxsplit=1)
    request_line_bytes, *header_lines = header_part.split(b"\r\n")

    # 2. 解析第一行：Method、Path、HTTP Version
    method, target_uri, http_version = request_line_bytes.decode("ascii").split(" ")

    # 3. 解析请求头键值对
    headers_block = b"\r\n".join(header_lines)
    headers = dict(BytesParser().parsebytes(headers_block).items())

    # 4. 根据 Content-Type 解析请求体
    payload = None
    if "application/json" in headers.get("Content-Type", ""):
        payload = json.loads(body_bytes.decode("utf-8"))

    return {
        "method": method,
        "uri": target_uri,
        "version": http_version,
        "headers": headers,
        "json_payload": payload,
    }


if __name__ == "__main__":
    req_info = parse_raw_http_request(RAW_HTTP_REQUEST)
    print(f"请求方法: {req_info['method']} | 目标路径: {req_info['uri']}")
    print(f"防盗链源: {req_info['headers']['Referer']}")
    print(f"JSON载荷: {req_info['json_payload']}")

    # 演示 Gzip 响应体压缩与解压对比
    raw_html = "<html><body><h1>人工智能开源数据集列表</h1></body></html>" * 20
    compressed_bytes = gzip.compress(raw_html.encode("utf-8"))
    print(
        f"\n原始HTML字节数: {len(raw_html.encode('utf-8'))} B -> Gzip压缩后: {len(compressed_bytes)} B"
    )
    decompressed_text = gzip.decompress(compressed_bytes).decode("utf-8")
    assert decompressed_text == raw_html
```

下表汇总了爬虫工程中最常见的 HTTP 状态码与核心请求头的故障诊断指南：

| 状态码 / Header 名称 | 典型表象与返回结果 | 底层触发原因 | 爬虫工程标准解决对策 |
|---|---|---|---|
| **`200 OK`（但内容是登录页）** | 状态码正常，但提取不到目标业务字段 | 触发了 `302` 重定向跳转至登录页或安全验证页 | 设置 `allow_redirects=False` 检查 `Location` 响应头，补充登录态 Cookie |
| **`400 Bad Request`** | 服务器直接报错拒绝解析请求 | URL 含未编码特殊字符，或 `POST` 的 JSON 格式不合法 | 使用 `urlencode`/`quote` 编码参数，用 `json=` 自动序列化并设置正确 `Content-Type` |
| **`401 Unauthorized`** | 接口返回 `{"code": 401, "msg": "未登录"}` | 缺失身份认证凭证，或 Token/Session 已过期 | 在请求头中携带有效的 `Cookie` 或 `Authorization: Bearer <JWT>` |
| **`403 Forbidden`** | 返回权限拒绝页或防盗链替图 | 默认 `python-requests` UA 被识破、缺失 `Referer` 或 IP 被封 | 伪装浏览器 `User-Agent`、补齐来源页 `Referer`、切换高匿代理 IP |
| **`429 Too Many Requests`** | 抓取几百页后突然大批报错，响应头含 `Retry-After` | 单位时间内单 IP 并发或请求频次超过服务端阈值 | 引入随机延迟、使用 `Semaphore` 限制并发数、接入指数退避重试与代理池 |
| **`Content-Encoding: br`** | 状态码 `200`，但 `response.text` 打印出乱码符号 | 请求头声明支持 `br`（Brotli），但本地未安装解压库 | 安装 `pip install brotli` 或将 `Accept-Encoding` 改为 `gzip, deflate` |

在掌握了各类状态码与请求头的工程含义后，请通过下方的交互式故障诊断器，对 8 个真实的爬虫报错抓包现场进行根因诊断与修复方案匹配。

#### Diagram: HTTP Status Code and Header Triage Simulator

```mermaid
flowchart TD
    Req["发送 HTTP 请求"] --> Status{"检查响应状态码 Status Code"}
    Status -->|200 OK / 206 Partial| CheckBody["校验响应体编码与格式后送入解析器"]
    Status -->|301 / 302 / 307 重定向| CheckLoc["追踪 Location 头 (警惕重定向至登录/验证码页)"]
    Status -->|401 / 403 权限拒绝| FixAuth["补齐 User-Agent / Referer / Cookie / 签名 Token"]
    Status -->|429 Too Many Requests| FixRate["解析 Retry-After 头 + 切换高匿代理 + 指数退避限流"]
    Status -->|500 / 502 / 503 / 504 服务端异常| Retry["触发幂等重试机制 (最多 3 次)"]
```

<details markdown="1">
<summary>HTTP Status Code and Header Triage Simulator</summary>
Type: microsim
**sim-id:** http-status-header-triage<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Analyze<br/>
**Bloom Verb:** deconstruct<br/>
**Learning Objective:** The learner will deconstruct 8 captured HTTP request/response failure logs to identify the exact root cause and select the correct header or request fix.

**Prerequisites:** HTTP methods (`GET`/`POST`), status codes (`200`, `302`, `400`, `401`, `403`, `429`, `503`), request headers (`User-Agent`, `Referer`, `Content-Type`, `Accept-Encoding`, `Authorization`), and response headers (`Location`, `Content-Encoding`, `Retry-After`) (all defined in Sections 2.1–2.3 above).

**Evidence of Mastery:** For each of the 8 captured HTTP packet logs, the learner inspects the request headers, status line, and response headers, then selects both the Root Cause and the Engineering Fix from 4 options. Mastery is correctly diagnosing at least 7 of the 8 packet logs on the first attempt.

**Misconceptions:** (1) Every `200 OK` response means the target business data was successfully fetched (ignoring silent `302 -> 200` redirects to a login page). (2) Garbled characters in `response.text` are always caused by `GBK` vs `UTF-8` charset issues rather than `Content-Encoding: br` (Brotli) compression when `brotli` is not installed. (3) Passing a Python dictionary via `data=` automatically sends `Content-Type: application/json`.

**Instructional Rationale:** Analyze-level deconstruction requires learners to cross-reference specific lines in raw HTTP request and response headers (such as `User-Agent: python-requests/2.32.3` paired with `403 Forbidden`, or `Content-Type: application/x-www-form-urlencoded` paired with `400 Bad Request` on a JSON endpoint) rather than memorizing isolated status code definitions.

**Content:**

The 8 HTTP packet diagnostic cases (presented in fixed order):

| Case # | Request Snippet Shown | Response Snippet Shown | Correct Root Cause & Fix | Feedback Explanation (Why) |
|---|---|---|---|---|
| 1 | `GET /articles/105 HTTP/1.1` / `User-Agent: python-requests/2.32.3` | `HTTP/1.1 403 Forbidden` / `Server: waf-guard` / Body: `Access Denied: Automated tool detected` | `默认 User-Agent 暴露爬虫身份 -> 在 Headers 中替换为真实 Chrome 浏览器 User-Agent` | 服务端 WAF 防火墙直接拦截了包含 `python-requests` 特征字符串的默认客户端标识。 |
| 2 | `GET /media/dataset-chart.png HTTP/1.1` / `User-Agent: Mozilla/5.0...` (无 `Referer` 头) | `HTTP/1.1 403 Forbidden` / `X-Error-Reason: Hotlinking Not Allowed` | `缺失防盗链来源校验头 -> 在 Headers 中添加 Referer: https://主站域名/` | 静态图片服务器开启了防盗链保护，要求请求头必须携带合法的站内来源页面地址 `Referer`。 |
| 3 | `POST /api/search HTTP/1.1` / `Content-Type: application/x-www-form-urlencoded` / Body: `page=1&kw=AI` | `HTTP/1.1 400 Bad Request` / Body: `{"error": "Expected application/json payload"}` | `POST 载荷媒体类型不匹配 -> 改用 json=payload 发送并声明 Content-Type: application/json` | 后端 RESTful 接口要求标准 JSON 格式请求体，而爬虫发送了传统表单键值对格式。 |
| 4 | `GET /user/export-jsonl HTTP/1.1` (经过自动重定向后) | 历史链路: `302 Found (Location: /login)` -> 最终响应: `HTTP/1.1 200 OK` (Body 为 `<form id="login-form">`) | `未携带登录态触发 302 静默跳转至登录页 -> 注入登录 Cookie/Session 并校验响应 URL 是否发生偏离` | 虽然最终状态码是 `200 OK`，但中间发生了 `302` 重定向，实际抓回来的是登录页而非数据页。 |
| 5 | `GET /api/v2/items?page=85 HTTP/1.1` (过去 5 秒内已连续发送 300 次请求) | `HTTP/1.1 429 Too Many Requests` / `Retry-After: 30` | `单 IP 请求频率超限触发限流 -> 读取 Retry-After 休眠退避，降低并发并接入高匿代理 IP 池` | 状态码 `429` 明确表示触发了服务端速率限制（Rate Limiting），继续硬闯会导致 IP 被永久封禁。 |
| 6 | `GET /news/list HTTP/1.1` / `Accept-Encoding: gzip, deflate, br` | `HTTP/1.1 200 OK` / `Content-Encoding: br` / Body 显示为 `\x1b\x9f\x01\x88...` 不可读乱码 | `服务端返回 Brotli (br) 压缩流而本地缺少解压库 -> 安装 brotli 库或将 Accept-Encoding 改为 gzip, deflate` | 手动复制浏览器请求头带上了 `br`，但 Python 环境未安装 `brotli` 解码器导致字节流未解压。 |
| 7 | `GET /api/private/corpus HTTP/1.1` / `User-Agent: Mozilla/5.0...` (无 `Authorization` 头) | `HTTP/1.1 401 Unauthorized` / `WWW-Authenticate: Bearer realm="api"` | `缺失 API 令牌凭证 -> 在 Headers 中添加 Authorization: Bearer <JWT_Token>` | `401` 与 `WWW-Authenticate: Bearer` 响应头表明该接口采用 Token/JWT 身份认证机制。 |
| 8 | `GET /archive/2026/detail_99.html HTTP/1.1` | `HTTP/1.1 503 Service Unavailable` / `Server: nginx` / Body: `Upstream backend temporarily overloaded` | `目标服务端瞬时过载不可用 -> 使用 tenacity 实现指数退避（Exponential Backoff）延迟自动重试` | `502/503/504` 属于服务端临时负载过高或网关超时，适合等待数秒后自动重试（切忌对 404 重试）。 |

**Provenance:** Derived from the status code and HTTP header diagnostic rules and summary table in Chapter 2 Sections 2.1–2.3.

**Rules:** Each case displays the captured Request packet on the left and Response packet on the right, with 4 candidate Root Cause & Fix options below. Exactly 1 option matches the `Correct Root Cause & Fix` column in Content.

**Learner Activity:**

1. The learner examines the HTTP Request and HTTP Response packet panels for the active case (`Case 1` through `Case 8`).
2. The learner clicks specific header lines (such as `User-Agent`, `Content-Type`, `Location`, `Content-Encoding`, `Retry-After`) to highlight clues.
3. The learner selects one of the four Root Cause & Fix options and clicks Submit.
4. The sim reveals whether the diagnosis is correct, explains the protocol mechanism, and unlocks the next packet case.

**Feedback:** 8 cases in fixed order, 1 attempt per case. Correct: "诊断精准！" followed by the `Feedback Explanation (Why)` text. Incorrect: "诊断有误：请重点观察报文中的关键状态码与高亮 Header 字段——" followed by the `Feedback Explanation (Why)` text and the correct option. Score counter displays `已修复抓包故障：n / 8`.

**Starting State:** Case 1 is loaded showing the `GET /articles/105` request with `User-Agent: python-requests/2.32.3` and the `403 Forbidden` WAF response, asking: "请分析上方抓包报文，找出导致本次爬虫请求失败的根本原因及修复方案。"

**Chapter Anchors:** All 8 status codes (`200`, `302`, `400`, `401`, `403`, `429`, `503`), the `python-requests/2.32.3` default UA string, the `Content-Encoding: br` pitfall, and the `Referer` anti-hotlinking mechanism are explicitly explained in Sections 2.2 and 2.3.
</details>

---

## 3. 会话跟踪与身份认证：突破无状态协议限制

### 3.1 HTTP无状态特性与Cookie原理

在设计之初，HTTP 被定义为一门纯粹的 **无状态协议（Stateless Protocol）**。什么叫“无状态”？意思是 Web 服务器天生患有所谓的“脸盲症”与“健忘症”：哪怕同一个客户端在 第 1 秒刚刚发送了账号密码完成了登录，到了第 2 秒该客户端再次发送 `GET /my-orders` 请求查看个人订单时，仅凭 HTTP 协议本身，服务器根本不知道第 2 秒的这个请求和第 1 秒那个登录请求来自同一个人！

为了在无状态的 HTTP 协议之上实现“记住用户身份、保持登录状态、记录购物车商品”，网景公司（Netscape）工程师于 1994 年发明了 **Cookie（小型文本甜饼）** 机制。**HTTP无状态特性与Cookie原理** 的完整闭环分为三步：

1. **服务端种植 Cookie（`Set-Cookie` 响应头）**：当你在浏览器中输入账号密码向 `POST /login` 发起请求并验证通过后，服务器在返回 `200 OK`（或 `302`）响应时，会在响应头中附带一行或多行 `Set-Cookie` 指令，例如：  
   `Set-Cookie: session_id=a9f8e7d6c5; Domain=.example.edu; Path=/; Max-Age=86400; Secure; HttpOnly`
2. **浏览器本地存储 Cookie**：浏览器收到响应后，解析 `Set-Cookie` 指令，将键值对 `session_id=a9f8e7d6c5` 保存在本地磁盘或内存的专属 Cookie 数据库中，并严格记录其作用域属性：
   - `Domain` 与 `Path`：决定访问哪些子域名和路径时才允许携带该 Cookie；
   - `Expires` / `Max-Age`：决定 Cookie 的过期销毁时间（若不设置，则为会话级 Cookie，关闭浏览器即销毁）；
   - `Secure`：规定仅在加密的 `HTTPS` 请求中才允许发送该 Cookie；
   - **`HttpOnly`**：一项极其重要的安全属性！带有 `HttpOnly` 标记的 Cookie **禁止前端通过 JavaScript 的 `document.cookie` 读取**（防止 XSS 跨站脚本窃取），只能在浏览器发起底层 HTTP 请求时由浏览器内核自动附带。
3. **客户端后续请求自动回传（`Cookie` 请求头）**：此后，只要浏览器再次向匹配 `Domain` 和 `Path` 的网址发起任何请求，都会自动在 HTTP 请求头中加上一行：  
   `Cookie: session_id=a9f8e7d6c5; theme=dark`  
   服务器收到该请求头后，读取 `session_id` 的值，立刻认出：“原来是刚才登录过的张三同学！”，从而返回专属的授权页面。

### 3.2 服务端Session会话跟踪与Token/JWT认证

理解了客户端的 Cookie 之后，我们再来看服务器端的配合机制，即 **服务端Session会话跟踪** 与现代前后端分离架构广泛采用的 **Token与JWT身份认证机制**。

1. **Cookie + 服务端 Session 模式（有状态会话）**  
   如果服务器直接把 `Cookie: user_id=1001; role=admin` 明文发给客户端，恶意用户只要手动把请求头里的 `role=user` 改成 `role=admin` 就能越权篡改身份！因此，传统 Web 框架（如 Django、Spring、PHP）采用 **服务端 Session（会话）**：
   - 用户的真实身份信息（如 `user_id`、权限等级、登录时间）全部保存在**服务器端的内存、数据库或 Redis 缓存**中，称为一个 `Session` 对象；
   - 服务器只生成一个毫无规律、不可猜测的随机长字符串——即 **`Session ID`**（如 `a9f8e7d6c5...`），通过 `Set-Cookie` 发给客户端保存；
   - 爬虫在采集此类网站时，只需要在登录后保存好这个包含 `Session ID` 的 Cookie（或直接使用第 3 章的 `requests.Session` 自动管理 CookieJar），即可在有效期内畅通无阻地采集所有需登录页面。
2. **Token 与 JWT 身份认证机制（无状态分布式令牌）**  
   在现代微服务架构、移动端 App 接口以及前后端分离（Vue/React）网站中，如果服务器要在 Redis 里为几千万活跃用户维护 Session 状态，会带来高昂的集群同步开销。于是工业界转向了基于密码学签名的 **JWT（JSON Web Token）** 无状态认证机制：
   - 用户登录成功后，服务器不再通过 `Set-Cookie` 种入 Session ID，而是在响应的 JSON 正文中直接返回一个长字符串 `access_token`。
   - 一个标准的 JWT 字符串由三段 Base64URL 编码的字符串通过两个英文句点 `.` 拼接而成，形如 **`Header.Payload.Signature`**（例如 `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMDAxLCJleHAiOjE3OTM1MjAwMDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`）：
     1. **Header（头部）**：声明签名算法（如 `HS256` 或 `RS256`）与令牌类型 `JWT`，经 Base64URL 编码；
     2. **Payload（载荷）**：存放非敏感的用户声明信息，如用户 ID（`sub`/`user_id`）、签发时间（`iat`）和**过期时间戳（`exp`）**。**注意：JWT 的 Header 和 Payload 仅仅是经过了 Base64URL 编码，并未加密！任何拿到 JWT 的人（包括我们的爬虫脚本）都可以直接用 Base64 解码看到里面的用户 ID 和准确过期时间 `exp`！**
     3. **Signature（数字签名）**：服务器使用保存在后端的绝密私钥（Secret Key），对前两段 `Header.Payload` 计算 HMAC-SHA256 或 RSA 数字签名。
   - 前端拿到 JWT 后，通常将其存储在浏览器的 `localStorage` 或 `sessionStorage` 中，并在后续发起 AJAX 请求时，通过 JavaScript 手动将其塞入请求头：  
     **`Authorization: Bearer <JWT字符串>`**  
   - 服务器收到请求后，无需查询数据库，直接用自己的私钥重新算一遍签名并与第三段 `Signature` 比对——只要客户端篡改了 Payload 里的任何一个字符，签名比对就会立即失败并返回 `401 Unauthorized`！

下面这段 Python 代码演示了爬虫工程师在逆向分析一个采用 JWT 认证的网站接口时，如何不依赖任何第三方密钥，直接解码 JWT 的 `Header` 与 `Payload` 来检查令牌的剩余有效期（`exp`），从而实现“在 Token 过期前 5 分钟自动触发重新登录刷新令牌”的工程逻辑：

```python
import base64
import json
import time


def decode_jwt_without_secret(jwt_token: str) -> dict:
    """解码 JWT 令牌的 Header 与 Payload（用于爬虫检查 Token 过期时间 exp）"""
    parts = jwt_token.split(".")
    if len(parts) != 3:
        raise ValueError("非法的 JWT 格式，必须包含 Header.Payload.Signature 三部分")

    def _b64url_decode(segment: str) -> dict:
        # Base64URL 编码在传输时省略了末尾的 '=' 填充符，解码前需按 4 的倍数补齐
        padding = "=" * ((4 - len(segment) % 4) % 4)
        raw_bytes = base64.urlsafe_b64decode(segment + padding)
        return json.loads(raw_bytes.decode("utf-8"))

    header = _b64url_decode(parts[0])
    payload = _b64url_decode(parts[1])
    return {"header": header, "payload": payload, "signature_preview": parts[2][:12] + "..."}


if __name__ == "__main__":
    # 构造一个模拟的 JWT 令牌 (Header + Payload + Signature)
    sample_jwt = (
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."
        "eyJ1c2VyX2lkIjoiYWlfc3R1ZGVudF8wMSIsInJvbGUiOiJ2aXAiLCJleHAiOjE4OTM0NTYwMDB9."
        "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
    )

    decoded = decode_jwt_without_secret(sample_jwt)
    print("JWT 头部 (Header) :", decoded["header"])
    print("JWT 载荷 (Payload):", decoded["payload"])

    exp_timestamp = decoded["payload"].get("exp", 0)
    remaining_seconds = exp_timestamp - int(time.time())
    print(f"令牌过期时间戳    : {exp_timestamp} (剩余有效秒数: {remaining_seconds} s)")
```

下表全面对比了 Cookie、服务端 Session 与 JWT Token 三大会话认证机制在爬虫工程中的差异：

| 对比维度 | Cookie（客户端凭证载体） | 服务端 Session（有状态会话） | Token / JWT（无状态令牌） |
|---|---|---|---|
| **数据存储位置** | 浏览器本地（Cookie 数据库/内存） | 服务器内存、数据库或 Redis 缓存 | 浏览器 `localStorage`/`sessionStorage` 或 Cookie |
| **HTTP 响应下发方式** | 响应头 `Set-Cookie: key=val` | 借用 `Set-Cookie` 下发 `session_id` | 登录接口响应 JSON 正文返回 `access_token` |
| **HTTP 请求携带方式** | 请求头 `Cookie: key=val`（浏览器自动附带） | 请求头 `Cookie: session_id=xxx` | 请求头 `Authorization: Bearer <JWT>` |
| **载荷内容可读性** | 明文键值或加密串 | 仅为一个无意义的随机索引 ID | `Header` 与 `Payload` 可直接用 Base64URL 解码查看 `exp` |
| **防篡改机制** | 依赖 `Secure`/`HttpOnly` 或签名 Cookie | 数据在服务端，客户端无法篡改内容 | 依赖第三段 `Signature` 服务端密钥签名校验 |
| **爬虫维持登录态策略** | 使用 `requests.Session` 自动管理或字典注入 | 保持 `session_id` Cookie 有效，定期心跳保活 | 解析登录 JSON 提取 Token 注入 `Authorization` 头，按 `exp` 定时刷新 |

---

## 4. 爬虫工程师的透视眼：Chrome DevTools 抓包实战

### 4.1 Chrome开发者工具DevTools基础

如果说 Python 是爬虫工程师手中的手术刀，那么现代浏览器内置的 **Chrome开发者工具DevTools基础（Chrome Developer Tools）** 就是不可或缺的“X 光透视仪”。无论目标网站的前端界面设计得多么炫酷复杂，只要它需要把数据展示给人类用户看，就必然要通过浏览器渲染 DOM 并通过网络发送 HTTP 请求。按下键盘上的 **`F12` 键**（或使用快捷键 **`Ctrl + Shift + I`**，或在网页任意位置右键点击选择 **“检查 / Inspect”**），即可唤出 Chrome DevTools。

在爬虫开发与逆向分析中，我们最常协同使用的四大核心面板是：

1. **`Elements`（元素面板）**：实时展示经过浏览器 JavaScript 渲染挂载后的**当前内存 DOM 树**以及每个元素的 CSS 样式。
   - **核心提醒**：在网页右键点击“查看网页源代码（View Page Source，`Ctrl + U`）”看到的只是服务器最初返回的**原始静态 HTML 文本**；而在 `Elements` 面板中看到的是执行完所有异步 AJAX 请求和 JS 动态插入后的**最终 DOM 树**！如果某个商品价格在 `Elements` 面板里看得到，但在 `Ctrl + U` 源码里搜不到，说明该数据是异步动态加载的（将在第 5、6 章专门攻克）。
   - 使用面板左上角的 **“选择元素箭头图标（`Ctrl + Shift + C`）”** 点击网页上的任意文字或图片，即可在 DOM 树中瞬间高亮定位到对应的 HTML 标签，辅助编写 CSS 选择器与 XPath。
2. **`Network`（网络抓包面板）**：按时间顺序完整记录浏览器发起的**每一笔 HTTP/HTTPS/WebSocket 网络请求**，是分析请求 URL、方法、状态码、请求头、响应头与 JSON 载荷的核心主战场。
3. **`Application`（应用存储面板）**：位于 Storage 栏下，支持可视化查看、编辑、删除当前网站保存在本地的所有 **`Cookies`**（包括 `HttpOnly` 标记列）、**`Local Storage`**、**`Session Storage`** 以及 **`IndexedDB`**，是排查登录态凭证存放在哪里的首选面板。
4. **`Sources`（源代码与断点调试面板）** 与 **`Console`（控制台面板）**：用于查看前端加载的 `.js` 脚本文件、打断点追踪加密签名参数生成逻辑（第 7 章 JS 逆向核心工具），以及在 `Console` 中直接输入 JavaScript 表达式或 `$x("//div[@class='title']")` 实时测试 XPath 语法。

### 4.2 Network面板抓包与HAR导出

掌握 **Network面板抓包与HAR导出** 的标准操作流，能够让你在 30 秒内摸清任何目标网站的通信底牌。以下是专业爬虫工程师进行抓包勘察的“黄金六步法”：

1. **第一步：勾选 `Preserve log`（保留日志）与 `Disable cache`（禁用缓存）**  
   - **为什么要勾选 `Preserve log`？** 默认情况下，一旦网页发生跳转（例如点击登录按钮后页面刷新跳转到首页），`Network` 面板会自动清空跳转前的所有抓包记录，导致你根本抓不到刚才那笔最关键的 `POST /login` 请求！勾选 `Preserve log` 后，跨页面跳转的所有历史请求都会完整保留。
   - **为什么要勾选 `Disable cache`？** 如果不禁用浏览器缓存，刷新页面时浏览器可能直接从本地磁盘（`from disk cache`）或内存（`from memory cache`）读取上次缓存的数据，或者返回 `304 Not Modified`，导致你看不到完整的 `200 OK` 请求与响应报文。
2. **第二步：按资源类型精准过滤（Filter Bar）**  
   一个大型门户网页加载一次往往会产生 100~300 个并发请求（包含大量的 CSS、字体、图标和统计埋点），直接在大海里捞针效率极低。善用类型过滤按钮：
   - **`Doc`（Document）**：只看主 HTML 文档请求（对应静态网页爬虫的目标）；
   - **`Fetch/XHR`**：只看通过 `fetch()` 或 `XMLHttpRequest` 发起的异步数据接口请求（对应动态 JSON API 爬虫的核心目标）；
   - **`JS` / `CSS` / `Img` / `Media` / `Font`**：分别过滤脚本、样式表、图片、音视频与自定义字体文件；
   - **`WS`（WebSocket）**：过滤全双工实时长连接消息流（如股票实时行情、直播间弹幕）。
3. **第三步：使用全局搜索（`Ctrl + F`）从结果反推接口**  
   当页面上有 50 个 `Fetch/XHR` 请求，不知道究竟哪一个返回了网页上的“¥299.00”价格或文章标题时，在 `Network` 面板内点击放大镜搜索图标（或按 `Ctrl + F` 打开左侧全局网络搜索栏），直接输入页面上可见的一小段独特文字（如商品名称的三个字或数字），DevTools 会瞬间跨所有响应报文体进行全文检索，一秒锁定目标数据包！
4. **第四步：深度检视单条请求的四大子标签（`Headers` / `Payload` / `Preview` / `Response`）**  
   点击锁定的请求行，右侧会展开详情面板：
   - **`Headers`**：查看 `General`（完整 URL、请求方法、状态码）、`Response Headers` 和 `Request Headers`；
   - **`Payload`**：格式化展示 URL 查询参数（`Query String Parameters`）与 POST 请求体（`Request Payload` / `Form Data`），并可点击 `view source` 查看未编码的原始字符串；
   - **`Preview`** 与 **`Response`**：`Preview` 将返回的 JSON 字符串自动渲染为可折叠展开的交互式对象树，方便观察层级关系；`Response` 则展示原始响应文本。
5. **第五步：一键复制为 cURL（`Copy as cURL (bash)`）**  
   在目标请求行上点击鼠标右键 $\rightarrow$ `Copy` $\rightarrow$ **`Copy as cURL (bash)`**，即可将该请求包含的所有 URL、Method、Headers、Cookie 和 POST Body 完整复制为一条标准的命令行字符串，直接导入 Postman 或转换为 Python `requests` 代码（详见第 3 章）。
6. **第六步：导出与解析 HAR 抓包归档文件（`Export HAR with content`）**  
   当我们需要将整套复杂的翻页抓包流量保存下来供团队离线分析，或者直接用 Python 脚本批量提取刚刚在浏览器里滑动加载出的几十页 JSON 接口数据时，可以点击 `Network` 面板顶部工具栏的 **下载图标（`Export HAR (with content)...`）**，将所有抓包记录保存为一个标准的 `.har`（HTTP Archive）文件。**HAR 文件本质上就是一个结构化的标准 JSON 文件**，其顶层 `log.entries` 数组完整记录了每一笔请求的 `request`、`response`（含正文 `content.text`）和耗时 `time`。

下面的代码示例展示了如何编写一个实用的 Python 脚本，直接读取并解析从 Chrome DevTools 导出的 `.har` 抓包归档文件，自动筛选出所有返回 JSON 数据的 API 接口，并统计其状态码、响应耗时与载荷预览：

```python
import json

# 模拟从 Chrome DevTools Network 面板导出的标准 HAR (HTTP Archive) JSON 数据结构
SAMPLE_HAR_CONTENT = {
    "log": {
        "version": "1.2",
        "creator": {"name": "WebInspector", "version": "537.36"},
        "entries": [
            {
                "time": 45.2,
                "request": {
                    "method": "GET",
                    "url": "https://portal.example.edu/static/app.css",
                    "headers": [{"name": "User-Agent", "value": "Mozilla/5.0"}],
                },
                "response": {
                    "status": 200,
                    "content": {"mimeType": "text/css", "size": 12040, "text": "body{margin:0}"},
                },
            },
            {
                "time": 182.6,
                "request": {
                    "method": "POST",
                    "url": "https://portal.example.edu/api/v1/ai-news/list",
                    "headers": [
                        {"name": "User-Agent", "value": "Mozilla/5.0"},
                        {"name": "Content-Type", "value": "application/json"},
                        {"name": "Authorization", "value": "Bearer eyJhbGci..."},
                    ],
                    "postData": {"text": '{"category": "LLM", "page": 1}'},
                },
                "response": {
                    "status": 200,
                    "content": {
                        "mimeType": "application/json",
                        "size": 845,
                        "text": '{"code": 0, "total": 128, "items": [{"id": 101, "title": "开源大模型评测基准发布"}]}',
                    },
                },
            },
        ],
    }
}


def analyze_har_api_entries(har_data: dict) -> list[dict]:
    """从 DevTools 导出的 HAR 数据中自动筛选并提取所有 JSON API 接口请求"""
    entries = har_data.get("log", {}).get("entries", [])
    api_endpoints: list[dict] = []

    for entry in entries:
        req = entry["request"]
        resp = entry["response"]
        mime_type = resp.get("content", {}).get("mimeType", "")

        # 仅过滤响应类型为 application/json 的异步数据接口
        if "application/json" not in mime_type:
            continue

        req_headers = {h["name"]: h["value"] for h in req.get("headers", [])}
        raw_resp_text = resp.get("content", {}).get("text", "{}")

        api_endpoints.append({
            "method": req["method"],
            "url": req["url"],
            "status": resp["status"],
            "time_ms": entry["time"],
            "has_auth": "Authorization" in req_headers or "Cookie" in req_headers,
            "post_payload": req.get("postData", {}).get("text"),
            "response_preview": json.loads(raw_resp_text),
        })

    return api_endpoints


if __name__ == "__main__":
    discovered_apis = analyze_har_api_entries(SAMPLE_HAR_CONTENT)
    print(f"🔍 从 HAR 抓包日志中成功识别出 {len(discovered_apis)} 个核心 JSON 数据接口：")
    for api in discovered_apis:
        print(f"  [{api['method']}] {api['url']} (状态码: {api['status']}, 耗时: {api['time_ms']}ms)")
        print(f"  请求载荷: {api['post_payload']}")
        print(f"  响应数据: {api['response_preview']}")
```

在完整学习了从 URL 拆解、DNS/TCP/TLS 握手、HTTP 请求/响应报文到 Chrome DevTools 抓包与 Cookie/JWT 会话跟踪的全链路知识后，请通过下方的交互式微仿真，亲自动手完成“从 URL 输入到建立认证会话”的端到端协议栈生命周期推演。

#### Diagram: End-to-End HTTP Lifecycle and Session Inspector

```mermaid
sequenceDiagram
    participant C as Python爬虫客户端_Session
    participant S as 目标Web服务器
    C->>S: 1. POST /login 提交账号密码表单与User-Agent
    S-->>C: 2. 302 Found 并下发 Set-Cookie: session_id=xyz987
    Note over C: Session 自动解析并持久化保存 CookieJar
    C->>S: 3. GET /dashboard/data 自动携带 Cookie: session_id=xyz987
    S-->>C: 4. 200 OK 返回已鉴权专属 HTML 与 JSON 数据
```

<details markdown="1">
<summary>End-to-End HTTP Lifecycle and Session Inspector</summary>
Type: microsim
**sim-id:** http-lifecycle-session-inspector<br/>
**Library:** p5.js<br/>
**Status:** Specified<br/>
**Bloom Level:** Apply<br/>
**Bloom Verb:** implement<br/>
**Learning Objective:** The learner will implement a complete 6-stage HTTPS authenticated scraping sequence by ordering the protocol layers and configuring the correct URL encoding, headers, and session credentials at each stage.

**Prerequisites:** URL syntax & percent-encoding, DNS resolution, TCP 3-way handshake, TLS handshake, HTTP `GET`/`POST` headers, and `Set-Cookie`/`Cookie`/`Authorization` session tracking (all defined in Sections 1–4 above).

**Evidence of Mastery:** Across the 6 sequential stages of fetching a protected AI dataset endpoint (`https://data.example.edu/api/corpus?tag=大模型#section2`), the learner selects the correct protocol action or packet configuration for each stage. Mastery is completing all 6 stages with at most 1 error (at least 5 of 6 stages correct on the first attempt).

**Misconceptions:** (1) The `#section2` URL fragment is transmitted to the server in the HTTP Request Line. (2) TLS encryption negotiation happens before the TCP 3-way handshake completes. (3) After a server returns `Set-Cookie: sid=9981; HttpOnly`, the crawler does not need to send `Cookie: sid=9981` on subsequent requests to protected endpoints.

**Instructional Rationale:** Apply-level implementation requires learners to step through a realistic single-session packet trace from initial URL parsing to authenticated follow-up request, actively choosing what bytes go on the wire at each protocol stage and observing the resulting client/server state transitions.

**Content:**

Target URL to scrape: `https://data.example.edu/api/corpus?tag=大模型#section2`

The 6 sequential stages, the choices presented to the learner at each stage, and the resulting state:

| Stage # | Protocol Phase | Question / Configuration Task | Correct Choice | Why (Feedback on Selection) |
|---|---|---|---|---|
| 1 | `Stage 1: URL 解析与百分号编码` | 爬虫准备向服务器发起请求，调用 `urlsplit` 与 `quote` 处理目标 URL 后，实际写入 HTTP 请求行（Request Line）的目标路径与查询串应该是什么？ | `/api/corpus?tag=%E5%A4%A7%E6%A8%A1%E5%9E%8B` | 中文字符“大模型”必须按 UTF-8 转为百分号编码 `%E5%A4%A7%E6%A8%A1%E5%9E%8B`，且末尾的 `#section2` 锚点仅用于浏览器本地滚动，绝不会发送给服务器。 |
| 2 | `Stage 2: DNS 域名解析` | 客户端尚不知道 `data.example.edu` 的 IP 地址，此时操作系统网络栈首先执行什么动作？ | `按『本地缓存 -> hosts文件 -> 递归DNS服务器』顺序查询域名对应的 IP 地址 93.184.216.34` | 在建立任何传输层 TCP 连接之前，必须先通过 DNS 将主机域名解析为可路由的 IP 地址。 |
| 3 | `Stage 3: TCP 三次握手 (Port 443)` | 拿到目标 IP `93.184.216.34` 与 HTTPS 默认端口 `443` 后，客户端与服务器建立可靠传输通道的正确报文交互序列是？ | `Client 发送 SYN(seq=x) -> Server 回复 SYN+ACK(ack=x+1, seq=y) -> Client 发送 ACK(ack=y+1)` | 标准 TCP 三次握手通过双向同步与确认序列号，耗时 1 个 RTT 建立全双工字节流连接。 |
| 4 | `Stage 4: TLS 1.3 加密握手` | TCP 连接建立后，由于协议是 `https://`，双方如何建立加密传输通道？ | `发送 ClientHello 与 ServerHello，校验服务器 CA 数字证书并协商出本次会话专属的对称加密密钥` | TLS 握手在 TCP 连接之上完成身份验真与非对称密钥交换，后续所有 HTTP 报文均使用协商出的对称密钥加密传输。 |
| 5 | `Stage 5: POST 登录与获取会话凭证` | 目标接口要求先登录。爬虫向 `POST /api/login` 发送 JSON 账号密码后，服务器返回响应头 `Set-Cookie: sid=ai_8899; Path=/; HttpOnly`。此时客户端会话管理器（`Session`）应执行什么操作？ | `解析响应头中的 Set-Cookie 指令，将键值对 sid=ai_8899 存入本地 CookieJar 容器中` | `Set-Cookie` 是服务端下发会话标识的标准响应头，`requests.Session` 会自动将其提取并保存在本地 CookieJar 中。 |
| 6 | `Stage 6: 携带凭证请求目标数据接口` | 登录完成后，爬虫在同一持久连接（Keep-Alive）上正式发起 `GET /api/corpus?tag=%E5%A4%A7%E6%A8%A1%E5%9E%8B` 请求，请求头中必须包含哪些字段才能成功拿到 `200 OK` JSON 数据？ | `复用已有 TCP+TLS 连接，并在 Request Headers 中附带伪装的 User-Agent 与 Cookie: sid=ai_8899` | HTTP/1.1 Keep-Alive 允许直接省去 Stage 2~4 的握手开销复用连接，且自动在请求头回传 `Cookie: sid=ai_8899` 完成身份认证。 |

**Provenance:** Synthesizes the complete protocol stack covered in Chapter 2 Sections 1.1–1.4, 2.1–2.3, and 3.1–3.2.

**Rules:** The learner progresses strictly from Stage 1 to Stage 6. At each stage, 3 options are presented (1 correct, 2 distractors based on the Misconceptions). Selecting the correct option updates the live "Client State & Wire Packet Inspector" panel showing the resolved IP, TCP state (`ESTABLISHED`), TLS cipher (`TLS_AES_256_GCM_SHA384`), CookieJar (`sid=ai_8899`), and final JSON response.

**Learner Activity:**

1. The learner inspects the target URL `https://data.example.edu/api/corpus?tag=大模型#section2` at the top of the simulator.
2. For each of the 6 stages, the learner reads the protocol prompt and selects one of the three candidate actions/packets.
3. Upon each selection, the simulator updates the packet inspector panel and explains why that protocol action succeeded or failed.
4. At Stage 6 completion, the full timeline from DNS to `200 OK` JSON payload is displayed showing how `Keep-Alive` skips Stages 2–4 on subsequent requests.

**Feedback:** 6 sequential stages, 2 attempts per stage. Correct on 1st attempt: "阶段推进成功！" + `Why` explanation. Incorrect: "协议动作错误：" + explains the specific misconception (e.g., sending `#fragment` to server or attempting TLS before TCP `SYN`) and allows a second attempt. Score counter shows `协议栈通关进度：k / 6 | 首次正确：n / 6`.

**Starting State:** Stage 1 (`URL 解析与百分号编码`) is active, the Client State panel shows `DNS: Unresolved | TCP: CLOSED | TLS: None | CookieJar: Empty`, and the prompt asks the learner to construct the exact Request-URI for `https://data.example.edu/api/corpus?tag=大模型#section2`.

**Chapter Anchors:** Matches the RFC 3986 URL structure and `#fragment` rule in Section 1.1, UTF-8 percent-encoding in Section 1.2, DNS/TCP/TLS sequence in Section 1.3, Keep-Alive connection reuse in Section 1.4, and `Set-Cookie`/`Cookie` lifecycle in Section 3.1.
</details>

---

## 5. 本章小结与思考练习

### 5.1 核心要点回顾

- **URL 结构与百分号编码**：标准 URL 由 `scheme://host:port/path?query#fragment` 组成；中文与特殊字符必须经 UTF-8 百分号编码（`%XX`）后传输；末尾的 `#fragment` 锚点仅在浏览器本地生效，绝不发送给服务器。
- **底层连接三部曲与协议演进**：每次跨域首次请求需依次经历 **DNS 域名解析**（域名 $\rightarrow$ IP）、**TCP 三次握手**（建立可靠传输连接）与 **TLS 加密握手**（校验数字证书并协商对称会话密钥）。HTTP/1.1 默认通过 `Keep-Alive` 复用连接，而 HTTP/2 则通过二进制分帧与多路复用实现了单连接并发传输。
- **HTTP 请求与响应报文**：`GET` 请求参数位于 URL 查询串、无请求体且幂等；`POST` 请求参数位于 Request Body，且必须通过 `Content-Type` 明确声明是表单格式（`x-www-form-urlencoded`）还是 JSON 格式（`application/json`）。
- **状态码与请求头攻防**：熟记 `200`（注意排查 `302` 静默跳登录页）、`304`（未修改缓存）、`400`（格式错误）、`401`（未认证）、`403`（UA/Referer/IP 被拦截）、`404`（资源不存在，不重试）、`429`（频率超限）与 `502/503/504`（服务端临时过载，需退避重试）。
- **无状态会话跟踪与 DevTools 抓包**：通过 `Set-Cookie`/`Cookie` 配合服务端 `Session`，或在请求头携带 `Authorization: Bearer <JWT>` 突破 HTTP 无状态限制；熟练运用 Chrome DevTools 的 `Network` 面板（开启 `Preserve log` 与 `Disable cache`、过滤 `Fetch/XHR`、全局搜索关键字、导出 HAR 归档）是一切爬虫逆向的起点。

### 5.2 思考与自测练习

??? question "思考题 1：某同学用爬虫抓取分页网址 `https://bbs.example.com/list#page=2`、`https://bbs.example.com/list#page=3`，发现无论把 `#page=` 后面的数字改成多少，抓回来的永远都是第 1 页的内容。请从 URL 协议规范角度解释原因，并说明应当如何处理？"
    **原因**：URL 中井号 `#` 后面的 `#page=2` 属于 **锚点片段（Fragment）**。根据 HTTP 协议规范，客户端在向服务器发起 HTTP 请求时，会自动剥离 `#` 及其后面的所有字符，因此服务器收到的请求行永远都是 `GET /list HTTP/1.1`，自然每次都返回第 1 页的静态外壳。  
    **处理方法**：这种 `#page=2` 属于前端 JavaScript 监听 `hashchange` 事件触发异步加载的设计。应当打开 Chrome DevTools 的 `Network` 面板，切换到 `Fetch/XHR` 过滤器，在网页上点击“第 2 页”按钮，观察浏览器实际发出的后台异步 API 请求地址（例如 `GET /api/posts?page=2`），直接针对该底层 API 地址编写爬虫（或在第 6 章使用 Playwright 驱动浏览器渲染）。

??? question "思考题 2：在 Chrome DevTools 的 `Network` 面板中，把某条请求的 Request Headers 原封不动复制到 Python `requests` 脚本中发送，状态码返回 `200 OK`，但打印 `response.text` 却是一堆乱码（已确认网页编码是标准的 UTF-8）。最可能的原因是什么？"
    **最可能的原因**：从 Chrome 复制的请求头中包含 `Accept-Encoding: gzip, deflate, br, zstd`。当服务器选择了 **`br`（Brotli）** 或 **`zstd`** 压缩算法返回响应体（响应头带有 `Content-Encoding: br`），而当前 Python 虚拟环境中并未安装 `brotli` 或 `zstandard` 第三方解压库时，`requests` 无法自动解压字节流，直接将压缩后的二进制字节按文本解码就会产生乱码。只需从自定义 `headers` 字典中删掉 `Accept-Encoding` 键（让 `requests` 自动声明自己支持的压缩格式），或执行 `pip install brotli` 即可解决。

??? question "思考题 3：Cookie 带有 `HttpOnly` 属性后，前端 JavaScript 的 `document.cookie` 无法读取它。请问这是否意味着 Python 网络爬虫也无法获取和使用带有 `HttpOnly` 标记的 Cookie？"
    **完全不是！** `HttpOnly` 仅仅是浏览器为了防范 XSS（跨站脚本攻击）而在**浏览器 JavaScript 引擎层**设置的读取限制。而在底层的 HTTP 网络通信层，带有 `HttpOnly` 的 Cookie 依然是通过明文响应头 `Set-Cookie` 由服务器发送给客户端的。Python 网络爬虫直接工作在 HTTP 协议层，无论是读取 `response.cookies` 还是使用 `requests.Session()`，都能 100% 正常接收、保存并自动回传带有 `HttpOnly` 属性的 Cookie。
