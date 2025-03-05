# 在 Python 中使用 HTTPX 进行网页抓取

[![Promo](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn) 

本指南将为你介绍如何使用 HTTPX（一个功能强大的 Python HTTP 客户端）来进行网页抓取，包括：

- [HTTPX 是什么？](#httpx-是什么)
- [使用 HTTPX 进行抓取：分步指南](#使用-httpx-进行抓取分步指南)
  - [步骤 #1：项目设置](#步骤-1项目设置)  
  - [步骤 #2：安装抓取相关库](#步骤-2安装抓取相关库)  
  - [步骤 #3：获取目标页面 HTML](#步骤-3获取目标页面-html)  
  - [步骤 #4：解析 HTML](#步骤-4解析-html)  
  - [步骤 #5：从中提取数据](#步骤-5从中提取数据)  
  - [步骤 #6：导出抓取到的数据](#步骤-6导出抓取到的数据)  
  - [步骤 #7：将所有步骤整合在一起](#步骤-7将所有步骤整合在一起)
- [HTTPX 网页抓取的高级功能与技巧](#httpx-网页抓取的高级功能与技巧)
  - [设置自定义请求头](#设置自定义请求头)
  - [自定义 User-Agent](#自定义-user-agent)
  - [设置 Cookies](#设置-cookies)
  - [代理集成](#代理集成)
  - [错误处理](#错误处理)
  - [会话管理](#会话管理)
  - [异步 API](#异步-api)
  - [重试失败的请求](#重试失败的请求)
- [HTTPX 与 Requests 在网页抓取中的比较](#httpx-与-requests-在网页抓取中的比较)
- [总结](#总结)

## HTTPX 是什么？

[HTTPX](https://github.com/projectdiscovery/httpx) 是一个功能完善的 Python 3 HTTP 客户端，基于 [`httpcore`](https://pypi.org/project/httpcore/) 库构建，能够在高并发多线程环境下提供稳定可靠的性能。HTTPX 同时支持同步与异步 API，并且支持 HTTP/1.1 和 HTTP/2 协议。

**功能亮点：**

- **简单且模块化的代码**：易于开发者参与和扩展。  
- **快速且可配置**：提供可完全自定义的标志来探测各种元素。  
- **多样化的探测方法**：支持多种基于 HTTP 的探测方式。  
- **自动协议回退**：可以从 HTTPS 智能回退到 HTTP。  
- **灵活的输入方式**：输入可为主机、URL 或者 CIDR。  
- **高级特性**：支持代理、自定义 HTTP 请求头、可配置超时、基本身份验证等更多功能。

**优点：**

- **命令行可用**：通过 [`httpx[cli]`](https://pypi.org/project/httpx/) 在命令行中使用。  
- **功能丰富**：支持 HTTP/2 与异步 API。  
- **活跃的开发**：定期更新，不断改进。

**缺点：**

- **更新频繁**：新版本有时可能引入不兼容更改。  
- **受欢迎度较低**：相比 [`requests`](https://requests.readthedocs.io/en/latest/) 库，使用人数相对较少。

## 使用 HTTPX 进行抓取：分步指南

HTTPX 作为一个 HTTP 客户端，只负责获取目标网页的 HTML；若想从中解析并提取数据，需要额外使用 HTML 解析工具，比如 [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)。

> **注意**：  
> 虽然 HTTPX 只在流程的早期阶段（获取 HTML）用到，但我们将演示完整的抓取流程。如果你对更高级的 HTTPX 抓取技巧感兴趣，可以在第 3 步结束后跳到下一章节。

### 步骤 #1：项目设置

首先，确保你的计算机上安装了 Python 3+，然后创建一个用于 HTTPX 抓取的项目文件夹：

```bash
mkdir httpx-scraper
```

进入该文件夹后，初始化一个 [虚拟环境](https://docs.python.org/3/library/venv.html)：

```bash
cd httpx-scraper
python -m venv env
```

使用 Python IDE 打开该项目文件夹，在其中创建一个名为 `scraper.py` 的文件，然后激活虚拟环境。  
Linux 或 macOS 系统上，运行：

```bash
./env/bin/activate
```

Windows 系统上，运行：

```bash
env/Scripts/activate
```

### 步骤 #2：安装抓取相关库

安装 HTTPX 与 BeautifulSoup：

```bash
pip install httpx beautifulsoup4
```

然后在 `scraper.py` 文件中导入这些依赖：

```python
import httpx
from bs4 import BeautifulSoup
```

### 步骤 #3：获取目标页面 HTML

这里以 “[Quotes to Scrape](https://quotes.toscrape.com/)” 为示例目标页面：

![Quotes to Scrape 主页](https://github.com/bright-cn/httpx-web-scraping/blob/main/Images/image-70.png)

使用 HTTPX 的 `get()` 方法获取首页的 HTML：

```python
# 向目标页面发起 HTTP GET 请求
response = httpx.get("http://quotes.toscrape.com")
```

在幕后，HTTPX 会向服务器发送 HTTP GET 请求，服务器返回页面的 HTML 内容。可通过 `response.text` 属性访问 HTML：

```python
html = response.text
print(html)
```

这将打印原始 HTML 内容，类似如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- 省略部分内容... -->
</body>
</html>
```

### 步骤 #4：解析 HTML

将获取到的 HTML 字符串传给 BeautifulSoup 来完成解析：

```python
# 使用 BeautifulSoup 解析 HTML
soup = BeautifulSoup(html, "html.parser")
```

变量 `soup` 现在存储了已解析的 HTML，并提供了各种方法来提取感兴趣的数据。

### 步骤 #5：从中提取数据

接下来从页面中抓取名言（quotes）相关数据：

```python
# 用于存储抓取到的数据
quotes = []

# 找到页面中所有的 "quote" 元素
quote_elements = soup.find_all("div", class_="quote")

# 遍历每个 quote，提取文本、作者和标签
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # 将抓取的数据存储为字典
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

上面的代码定义了一个名为 `quotes` 的列表来存储抓取数据。随后，定位所有带有 “quote” 类名的 HTML 元素并逐个遍历，提取出文本、作者和标签信息。每条记录以字典形式存入 `quotes` 列表，方便后续使用或导出。

### 步骤 #6：导出抓取到的数据

将抓取到的数据导出到 CSV 文件中：

```python
# 指定要导出的文件名称
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # 写入表头
    writer.writeheader()

    # 写入抓取到的名言数据
    writer.writerows(quotes)
```

上述代码会打开一个名为 `quotes.csv` 的文件，指定列名为 `text`、`author`、`tags`，然后先写入表头，再将 `quotes` 列表中每个字典的数据写入 CSV 文件。`csv.DictWriter` 帮助我们自动处理格式化的问题。

记得在脚本开头导入 `csv`：

```python
import csv
```

### 步骤 #7：将所有步骤整合在一起

最终的 HTTPX 网页抓取脚本如下所示：

```python
import httpx
from bs4 import BeautifulSoup
import csv

# 发起 HTTP GET 请求
response = httpx.get("http://quotes.toscrape.com")

# 获取目标页面的 HTML
html = response.text

# 使用 BeautifulSoup 解析 HTML
soup = BeautifulSoup(html, "html.parser")

# 用于存储抓取到的数据
quotes = []

# 找到页面中所有的 "quote" 元素
quote_elements = soup.find_all("div", class_="quote")

# 遍历每个 quote，提取文本、作者和标签
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author").get_text()
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # 将抓取数据存储为字典
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })

# 指定要导出的文件名称
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # 写入表头
    writer.writeheader()

    # 写入抓取到的名言数据
    writer.writerows(quotes)
```

执行：

```bash
python scraper.py
```

或在 Linux/macOS 上：

```bash
python3 scraper.py
```

这会在项目根目录下生成一个名为 `quotes.csv` 的文件，内容如下所示：

![包含抓取数据的 CSV](https://github.com/bright-cn/httpx-web-scraping/blob/main/Images/image-71.png)

## HTTPX 网页抓取的高级功能与技巧

现在让我们看一个更复杂的示例。目标网站为 [HTTPBin.io 的 `/anything` 接口](https://httpbin.io/anything)，这个特殊的 API 会返回请求方发送的 IP 地址、请求头以及其他相关信息。

### 设置自定义请求头

可以使用 [`headers` 参数](https://www.python-httpx.org/quickstart/#custom-headers) 来指定自定义的请求头：

```python
import httpx

# 自定义请求头
headers = {
    "accept": "application/json",
    "accept-language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
}

# 携带自定义请求头发起 GET 请求
response = httpx.get("https://httpbin.io/anything", headers=headers)
# 处理响应...
```

### 自定义 User-Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) 是 [网页抓取中最重要的请求头之一](https://www.bright.cn/blog/web-data/http-headers-for-web-scraping)。HTTPX 默认的 `User-Agent` 为：

```
python-httpx/<VERSION>
```

此标识很可能会让目标网站识别到你的请求是自动化脚本，从而导致封禁。可以将 `User-Agent` 替换为模拟真实浏览器，例如：

```python
import httpx

# 定义一个自定义的 User-Agent
headers = {
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
}

# 携带自定义 User-Agent 发起请求
response = httpx.get("https://httpbin.io/anything", headers=headers)
# 处理响应...
```

### 设置 Cookies

要在 HTTPX 中携带 Cookies，可以通过 [`cookies` 参数](https://www.python-httpx.org/quickstart/#cookies)：

```python
import httpx

# 将 cookies 定义成字典
cookies = {
    "session_id": "3126hdsab161hdabg47adgb",
    "user_preferences": "dark_mode=true"
}

# 携带自定义 cookies 发起请求
response = httpx.get("https://httpbin.io/anything", cookies=cookies)
# 处理响应...
```

这样就能在请求中包含会话信息，对于需要登录或保存用户偏好信息的场景非常有用。

### 代理集成

在进行网页抓取时，通过代理来 [隐藏真实 IP、规避目标网站封禁](https://www.python-httpx.org/advanced/proxies/) 十分常见。只需在 `get()` 中添加 `proxy` 参数：

```python
import httpx

# 使用你的代理服务器 URL
proxy = "<YOUR_PROXY_URL>"

# 通过代理服务器发起请求
response = httpx.get("https://httpbin.io/anything", proxy=proxy)
# 处理响应...
```

### 错误处理

HTTPX 默认只会在 [连接或网络问题](https://www.python-httpx.org/exceptions/) 时抛出错误。若要对 [`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) 或 [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) 状态码的响应抛出异常，可使用 `raise_for_status()` 方法：

```python
import httpx

try:
    response = httpx.get("https://httpbin.io/anything")
    # 对 4xx 和 5xx 响应抛出异常
    response.raise_for_status()
    # 处理正常响应...
except httpx.HTTPStatusError as e:
    # 处理 HTTP 状态码错误
    print(f"发生 HTTP 错误: {e}")
except httpx.RequestError as e:
    # 处理连接或网络错误
    print(f"发生请求错误: {e}")
```

### 会话管理

直接使用 HTTPX 的顶层 API（`httpx.get()` 等）时，每次请求都会新建一次连接，不能复用 TCP 连接。对于频繁访问同一主机的情况，这样做效率会变低。

而使用 `httpx.Client` 对象则支持 [HTTP 连接池](https://devblogs.microsoft.com/premier-developer/the-art-of-http-connection-pooling-how-to-optimize-your-connections-for-peak-performance/) 功能，可在同一主机多次请求时重用已有的连接，从而减少连接建立的次数，降低时延与网络负载。

使用 `Client` 相较于顶层 API 的好处包括：

- 多次请求之间减少握手带来的延迟  
- 降低 CPU 占用和网络往返次数  
- 节省带宽和提高速度

此外，`Client` 中还支持在顶层 API 中无法实现的会话管理功能，比如：

- 请求之间的 Cookie 持久化  
- 为所有请求统一设置配置信息  
- 通过 HTTP 代理发送请求

通常建议通过上下文管理器（`with` 语句）来使用 `Client`：

```python
import httpx

with httpx.Client() as client:
    # 使用 client 发起任意 HTTP 请求
    response = client.get("https://httpbin.io/anything")

    # 获取 JSON 并打印
    response_data = response.json()
    print(response_data)
```

也可以手动管理客户端，并在使用完成后调用 `client.close()` 来释放连接池资源：

```python
import httpx

client = httpx.Client()
try:
    # 使用 client 发起请求
    response = client.get("https://httpbin.io/anything")

    # 获取 JSON 并打印
    response_data = response.json()
    print(response_data)
except:
    # 处理错误...
    pass
finally:
    # 关闭 client，释放资源
    client.close()
```

> **提示**：  
> 若熟悉 `requests` 库，可以理解 `httpx.Client()` 与 [`requests.Session()`](https://requests.readthedocs.io/en/latest/user/advanced/#session-objects) 类似。

### 异步 API

HTTPX 默认提供同步 API，同时也有一个 [异步客户端](https://www.python-httpx.org/async/) 供需要时使用。如果你在使用 [`asyncio`](https://docs.python.org/3/library/asyncio.html)，就可以通过异步客户端设计更高效的 HTTP 请求流程。

要在 HTTPX 中发送异步请求，需要初始化 `AsyncClient`，并用 `await` 调用其方法：

```python
import httpx
import asyncio

async def fetch(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text

async def main():
    urls = ["https://httpbin.io/anything"] * 5
    responses = await asyncio.gather(*(fetch(url) for url in urls))
    for response in responses:
        print(response)

asyncio.run(main())
```

通过 `with` 上下文，异步客户端会在代码块结束时自动关闭；也可手动调用 `await client.close()` 来显式关闭连接。

所有在异步客户端中的请求方法（`get()`、`post()` 等）都是异步的，因此必须在它们前面加上 `await` 才能获得返回值。

### 重试失败的请求

在网络不稳定或网页抓取时，难免：

- 连接失败  
- 超时发生  

HTTPX 可以使用 [`HTTPTransport`](https://www.python-httpx.org/advanced/transports/) 机制在出现 `httpx.ConnectError` 或 `httpx.ConnectTimeout` 时进行重试。以下示例配置了最多重试 3 次：

```python
import httpx

# 配置 transport，使得连接出错或超时时自动重试至多 3 次
transport = httpx.HTTPTransport(retries=3)

# 将 transport 与 HTTPX 客户端绑定
with httpx.Client(transport=transport) as client:
    response = client.get("https://httpbin.io/anything")
    # 处理响应...
```

需要注意的是，只有连接相关错误会触发重试。如果想在读写错误或特定 HTTP 状态码发生时进行重试，需要借助类似 [`tenacity`](https://tenacity.readthedocs.io/en/latest/) 这样的库实现自定义逻辑。

## HTTPX 与 Requests 在网页抓取中的比较

下表对比了 HTTPX 与 [Requests](https://www.bright.cn/blog/web-data/python-requests-guide) 在网页抓取场景下的主要功能：

| **功能**                         | **HTTPX**               | **Requests**               |
|---------------------------------|-------------------------|----------------------------|
| **GitHub 星标数**               | 8k                      | 52.4k                      |
| **异步支持**                     | ✔️                      | ❌                         |
| **连接池**                       | ✔️                      | ✔️                         |
| **HTTP/2 支持**                 | ✔️                      | ❌                         |
| **自定义 User-Agent**           | ✔️                      | ✔️                         |
| **代理支持**                     | ✔️                      | ✔️                         |
| **Cookies 处理**                | ✔️                      | ✔️                         |
| **超时设置**                     | 可针对连接与读取分别配置 | 可针对连接与读取分别配置    |
| **重试机制**                     | 可通过 transport 配置     | 可通过 `HTTPAdapter` 配置   |
| **性能表现**                     | 高                       | 中                         |
| **社区支持与流行度**             | 正在增长                 | 用户基数庞大               |

## 总结

当你自动化发送 HTTP 请求时，你的公共 IP 地址会暴露你所在的地理位置，甚至可能暴露个人身份，存在隐私与安全风险。为此，建议在抓取过程中使用代理隐藏你的真实 IP。

Bright Data 拥有全球顶尖代理服务器，为世界 500 强企业及超过 2 万名客户服务，提供多种代理类型：

- [数据中心代理](https://www.bright.cn/proxy-types/datacenter-proxies) – 超过 770,000 个数据中心 IP。  
- [住宅代理](https://www.bright.cn/proxy-types/residential-proxies) – 超过 72M 个来自 195+ 个国家/地区的住宅 IP。  
- [ISP 代理](https://www.bright.cn/proxy-types/isp-proxies) – 超过 700,000 个 ISP IP。  
- [移动代理](https://www.bright.cn/proxy-types/mobile-proxies) – 超过 7M 个移动 IP。

立即注册免费的 Bright Data 账号，尝试我们的抓取解决方案与代理服务吧！
