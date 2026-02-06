# brightdata_serp

Bright Data SERP 性能测试脚本，用于并发压测多个搜索引擎入口并输出汇总统计/明细 CSV。

## 运行要求

- Python 3.9+
- 依赖：`requests`

```bash
pip install requests
```

## 基本用法

```bash
python brigtdata_serp.py -t <API_TOKEN> -z <ZONE> -e search -n 10 -c 5
```

常见参数：

- `-t, --api-token`：Bright Data API Token（必填）
- `-z, --zone`：Bright Data zone，默认 `serp_api1`
- `-e, --engines`：要测试的引擎列表（如 `search maps trends bing`）
- `--all-engines`：测试所有支持引擎
- `-n, --num-requests`：每个引擎请求数
- `-c, --concurrency`：并发数
- `-q, --query`：固定查询词（不传则随机）
- `--format {raw,json}`：Bright Data 响应格式
- `--data-format {screenshot,parsed_light}`：附加 data_format
- `--save-details`：保存每次请求明细 CSV
- `--save-responses-dir <DIR>`：保存原始响应内容到指定目录
- `--brd-json {0,1}`：URL 是否附加 `brd_json`（默认 1）
- `-o, --output`：汇总统计 CSV 文件名

## Google Search 语言/地区参数（hl/gl）

`search` 引擎支持附加：

- `--hl`：语言参数（如 `en`, `zh-CN`）
- `--gl`：国家/地区参数（如 `us`, `cn`）

示例：

```bash
python brigtdata_serp.py \
  -t <API_TOKEN> -z serp_api1 -e search -q pizza \
  --hl en --gl us
```

会构造等价于以下查询参数形式的 Google URL：

`https://www.google.com/search?q=pizza&hl=en&gl=us`

> 说明：脚本内部使用 Python 的 `dict -> urlencode` 组装 URL，不需要也不应做 shell 风格的 `&` 转义。

## parsed_light 使用说明

当 `--data-format parsed_light` 时：

- 返回值期望为结构化 JSON（payload 本身即结果）
- 不依赖 `Content-Type` 判断 JSON 解析
- 结果成功判定基于内容：
  - payload 是 `dict`
  - 不包含 `error`
  - `organic` 字段为非空 `list`
- `response_excerpt` 会优先展示 `organic` 的 JSON 片段，便于在明细 CSV 快速查看结构化结果

示例：

```bash
python brigtdata_serp.py \
  -t <API_TOKEN> -z serp_api1 -e search -q pizza \
  --data-format parsed_light --save-details
```

## 其他示例

测试多个引擎：

```bash
python brigtdata_serp.py -t <API_TOKEN> -z serp_api1 -e search maps trends -n 5 -c 3
```

测试所有引擎并输出汇总：

```bash
python brigtdata_serp.py -t <API_TOKEN> -z serp_api1 --all-engines -n 10 -c 5 -o brightdata_summary_statistics.csv
```
