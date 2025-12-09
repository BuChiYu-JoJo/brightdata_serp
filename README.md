# Bright Data SERP 性能测试工具

一个用于测试 Bright Data SERP API 性能的 Python 工具，支持多种搜索引擎（Google、Bing、Yandex、DuckDuckGo 等）的并发测试和性能分析。

**新版本特性：使用 asyncio 和 aiohttp 实现高性能异步并发请求。**

## 功能特性

- ✅ 支持 10 种搜索引擎的测试（7种Google引擎 + 3种其他搜索引擎）
- ✅ **异步并发请求**，使用 asyncio 和 aiohttp 提供更高性能
- ✅ 可自定义并发数
- ✅ 详细的性能统计（成功率、响应时间、延迟百分位数）
- ✅ 支持 JSON 和 HTML 两种响应格式
- ✅ 自动生成 CSV 统计报告
- ✅ 内置丰富的样本查询数据
- ✅ 灵活的参数配置

## 支持的引擎

| 引擎 | 说明 | 特殊参数 |
|------|------|----------|
| `search` | Google 搜索 | - |
| `maps` | Google 地图（POI/地点查询） | 查询参数在路径中 |
| `trends` | Google 趋势（关键词热度） | `brd_trends=timeseries,geo_map` |
| `reviews` | Google 本地评论 | `tbm=lcl` |
| `lens` | Google 图片搜索（反向图片搜索） | 不使用 `brd_json` |
| `hotels` | Google 酒店搜索 | 支持入住/退房日期 |
| `flights` | Google 航班搜索 | - |
| `bing` | Bing 搜索 | - |
| `yandex` | Yandex 搜索 | 使用 `text` 参数 |
| `duckduckgo` | DuckDuckGo 搜索 | - |

## 安装依赖

```bash
pip install aiohttp
```

**注意**：新版本使用 `aiohttp` 替代了 `requests`，以提供更好的异步并发性能。

## 使用方法

### 基本用法

```bash
# 测试单个引擎（使用随机关键词）
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search -n 10 -c 5

# 测试多个引擎
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search maps trends -n 10 -c 3

# 测试所有支持的引擎
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE --all-engines -n 5 -c 2
```

### 指定查询关键词

```bash
# 使用自定义查询
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search -q "pizza" -n 10 -c 5

# 测试多个引擎，使用相同的查询
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search maps -q "restaurant" -n 5 -c 3
```

### 使用 JSON 响应格式

```bash
# 获取 JSON 格式的响应（推荐用于数据解析）
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search --format json -n 10 -c 5

# 保存详细的请求记录到 CSV
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e search --format json --save-details -n 10 -c 5
```

### 列出支持的引擎

```bash
python brigtdata_serp.py --list-engines
```

## 命令行参数

| 参数 | 简写 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| `--api-token` | `-t` | ✅ | - | Bright Data API Token |
| `--zone` | `-z` | ❌ | `serp_api1` | Bright Data zone 名称 |
| `--engines` | `-e` | ❌* | - | 要测试的引擎列表（空格分隔） |
| `--all-engines` | - | ❌* | - | 测试所有支持的引擎 |
| `--num-requests` | `-n` | ❌ | `5` | 每个引擎的请求数 |
| `--concurrency` | `-c` | ❌ | `3` | 并发数 |
| `--query` | `-q` | ❌ | 随机 | 指定查询关键词 |
| `--format` | - | ❌ | `raw` | 响应格式（`raw` 或 `json`） |
| `--save-details` | - | ❌ | `false` | 保存每个请求的详细 CSV 记录 |
| `--output` | `-o` | ❌ | `brightdata_summary_statistics.csv` | 汇总统计输出文件名 |
| `--list-engines` | - | ❌ | - | 列出所有支持的引擎 |
| `--brd-json` | - | ❌ | `1` | 是否在 URL 中附加 brd_json 参数（`0` 或 `1`） |

*注：`--engines` 和 `--all-engines` 必须提供其中一个。

## 引擎特定说明

### Google Trends (`trends`)

Google Trends 引擎需要特殊参数以获取最高成功率的 widget 数据：

```python
# 自动添加的参数
brd_trends=timeseries,geo_map  # 获取时间序列和地理地图数据
brd_json=1                      # 返回解析后的 JSON 结果
```

支持的查询格式：
```python
# 简单查询
"pizza"

# 带地理位置的查询
{"q": "bitcoin", "geo": "US"}

# 带时间范围的查询
{"q": "ai news", "geo": "US", "date": "now 7-d"}
```

生成的 URL 示例：
```
https://trends.google.com/trends/explore?q=pizza&geo=us&brd_trends=timeseries,geo_map&brd_json=1
```

### Google Maps (`maps`)

Maps 引擎使用路径参数格式：
```
https://www.google.com/maps/search/pizza/
```

### Google Lens (`lens`)

Lens 引擎需要图片 URL 作为查询参数：
```bash
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e lens -q "https://example.com/image.jpg"
```

### Google Hotels (`hotels`)

Hotels 引擎支持丰富的参数：
```python
{
    "q": "Tokyo luxury hotels",
    "checkin": "2025-12-15",
    "checkout": "2025-12-20",
    "adults": 2
}
```

### Google Flights (`flights`)

Flights 引擎支持航线查询：
```python
{"q": "SFO to JFK", "src": "searchbox"}
```

### Bing (`bing`)

Bing 搜索引擎使用标准的 `q` 参数：
```bash
# 使用 KEYWORD_POOL 中的随机关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e bing -n 10 -c 5

# 指定查询关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e bing -q "artificial intelligence" -n 10
```

生成的 URL 示例：
```
https://www.bing.com/search?q=pizza&brd_json=1
```

### Yandex (`yandex`)

Yandex 搜索引擎使用 `text` 参数（而非 `q`）：
```bash
# 使用 KEYWORD_POOL 中的随机关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e yandex -n 10 -c 5

# 指定查询关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e yandex -q "cryptocurrency" -n 10
```

生成的 URL 示例：
```
https://www.yandex.com/search/?text=pizza&brd_json=1
```

### DuckDuckGo (`duckduckgo`)

DuckDuckGo 搜索引擎使用标准的 `q` 参数：
```bash
# 使用 KEYWORD_POOL 中的随机关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e duckduckgo -n 10 -c 5

# 指定查询关键词
python brigtdata_serp.py -t YOUR_API_TOKEN -z YOUR_ZONE -e duckduckgo -q "privacy search" -n 10
```

生成的 URL 示例：
```
https://duckduckgo.com/?q=pizza&brd_json=1
```

**注意**：Bing、Yandex 和 DuckDuckGo 引擎默认使用内置的 KEYWORD_POOL 随机选择查询关键词，包含 60+ 个常见搜索词。

## 输出结果

### 控制台输出

测试完成后，会在控制台显示统计表格：

```
汇总统计表:
------------------------------------------------------------------------------------------------------------------------------------------------------
引擎           请求数    并发  速率(req/s)      成功  成功率  平均响应(s)      P50      P75      P90  完成时间(s)    响应大小(KB)
------------------------------------------------------------------------------------------------------------------------------------------------------
search              10      5       2.456        10    100%        1.234    1.200    1.350    1.450        4.068         85.234
maps                10      5       2.123         9     90%        1.456    1.400    1.550    1.650        4.712         92.156
trends              10      5       1.987        10    100%        1.678    1.600    1.750    1.850        5.033         45.678
------------------------------------------------------------------------------------------------------------------------------------------------------
```

### CSV 输出

#### 汇总统计文件（默认：`brightdata_summary_statistics.csv`）

包含所有引擎的性能统计数据：
- 引擎名称
- 请求总数
- 并发数
- 请求速率
- 成功次数和成功率
- 平均响应时间
- 延迟百分位数（P50、P75、P90）
- 并发完成时间
- 平均响应大小

#### 详细记录文件（使用 `--save-details` 时生成）

格式：`brightdata_{engine}_details_{timestamp}.csv`

包含每个请求的详细信息：
- 时间戳
- 引擎
- 查询内容
- HTTP 状态码
- 响应时间
- 响应大小
- 成功/失败状态
- 错误信息
- 响应摘要

## 样本查询数据

工具内置了丰富的样本查询数据，当不指定 `-q` 参数时会随机选择：

- **Maps**: pizza, coffee, restaurant, hotel, gym, theater, museums, transit, pharmacy
- **Trends**: ai news, bitcoin, nba, 旅游, iphone, switch（带地理位置和时间范围）
- **Reviews**: best sushi in nyc, coffee shop san francisco, bakery london 等
- **Lens**: 多个图片 URL（来自 Imgur, Picsum, Unsplash 等）
- **Hotels**: 带入住日期的酒店查询
- **Flights**: 航线查询（SFO to JFK, LAX to NRT 等）
- **Bing/Yandex/DuckDuckGo**: 使用通用关键词池
- **通用关键词池 (KEYWORD_POOL)**: 69 个常见搜索关键词（pizza, coffee, weather, news, hotel, flight 等）

## 性能指标说明

- **成功率 (%)**: 返回 HTTP 200 且无错误的请求百分比
- **平均响应时间 (s)**: 成功请求的平均响应时间
- **P50/P75/P90 延迟 (s)**: 响应时间的百分位数（中位数、75分位、90分位）
- **请求速率 (req/s)**: 每秒处理的请求数
- **并发完成时间 (s)**: 完成所有并发请求的总时间
- **响应大小 (KB)**: 成功响应的平均大小

## 示例场景

### 1. 快速性能测试

```bash
# 测试搜索引擎的基本性能（10个请求，5并发）
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e search -n 10 -c 5
```

### 2. 多引擎对比测试

```bash
# 对比不同引擎的性能
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e search maps trends -n 20 -c 5 --save-details

# 对比多个搜索引擎（Google、Bing、Yandex、DuckDuckGo）
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e search bing yandex duckduckgo -n 20 -c 5
```

### 3. 压力测试

```bash
# 高并发测试（50个请求，10并发）
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e search -n 50 -c 10 --format json
```

### 4. 特定关键词测试

```bash
# 测试特定查询的稳定性
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e search trends -q "cryptocurrency" -n 30 -c 5

# 测试多个搜索引擎使用相同关键词
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE -e bing yandex duckduckgo -q "artificial intelligence" -n 20 -c 5
```

### 5. 完整测试报告

```bash
# 测试所有引擎并保存详细报告（包括10个引擎）
python brigtdata_serp.py -t YOUR_TOKEN -z YOUR_ZONE --all-engines -n 20 -c 5 --save-details -o full_report.csv
```

## 错误处理

工具会自动检测和记录以下错误：

- HTTP 错误状态码（非 200）
- 空响应
- Bright Data 代理错误
- 请求超时（30秒）
- JSON 解析错误

所有错误都会记录在结果中，并影响成功率统计。

## 注意事项

1. **API Token**: 确保使用有效的 Bright Data API Token
2. **Zone 配置**: Zone 名称需要与 Bright Data 账户中的配置一致
3. **并发限制**: 根据 Zone 的并发限制设置合理的 `-c` 参数
4. **超时设置**: 默认请求超时为 30 秒
5. **Trends 引擎**: 仅支持解析后的 JSON 结果（自动添加 `brd_json=1`）
6. **Lens 引擎**: 需要提供有效的图片 URL

## 技术细节

### URL 构建

工具使用 `EngineConfig` 类构建符合 Bright Data 规范的 URL：

- 自动处理查询参数编码
- 支持路径参数和查询参数两种格式
- 确保 URL 格式正确（无尾随 & 等问题）
- 支持额外参数注入

### 并发执行

**异步并发实现（新版本）：**

使用 `asyncio` 和 `aiohttp` 实现高性能异步并发：

- **异步 I/O**：使用 `aiohttp.ClientSession` 进行非阻塞 HTTP 请求
- **协程并发**：使用 `asyncio.gather()` 并发执行多个请求任务
- **连接池管理**：通过 `TCPConnector` 控制并发连接数
- **性能优势**：相比线程池，异步 I/O 在高并发场景下内存占用更少，性能更高
- **可配置并发数**：通过 `-c` 参数控制同时发起的请求数量
- **自动结果聚合**：所有异步请求完成后自动收集结果

技术栈：
- `asyncio`：Python 标准库异步编程框架
- `aiohttp`：高性能异步 HTTP 客户端库
- `asyncio.gather()`：并发执行多个协程
- `TCPConnector`：管理 TCP 连接池

### 统计计算

- 成功率：成功请求 / 总请求
- 百分位数：使用排序后的精确计算
- 响应时间：使用 `time.perf_counter()` 高精度计时

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

本项目使用的许可证请参考项目根目录的 LICENSE 文件。

## 相关链接

- [Bright Data 官方文档](https://docs.brightdata.com/)
- [Bright Data SERP API 文档](https://docs.brightdata.com/scraping-automation/serp-api/)
- [Google Trends 参数说明](https://trends.google.com/)