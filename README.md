# brightdata_serp

Bright Data SERP 性能测试脚本会默认在生成的 URL 中附加 `brd_json=1`，确保返回 JSON 结构。如需跳过该参数，可在运行时添加 `--brd-json 0`。

## Google SERP 引擎请求示例
下表给出了 Bright Data SERP API 针对 Google 多个垂直场景的真实请求构造方式。使用脚本时，`url` 字段填入下述基准 URL；`q`/`url` 等查询参数会在脚本中按需自动拼接（并默认带上 `brd_json=1`）。

| 引擎 | 基准 URL | 关键参数 | 默认/示例参数 | 备注 |
| --- | --- | --- | --- | --- |
| search | `https://www.google.com/search` | `q` | 默认使用 `KEYWORD_POOL` 中的词，例如 `pizza` | 普通搜索。可额外追加 `hl`/`gl`/`uule` 等地区语言参数。 |
| maps | `https://www.google.com/maps/search/` | `q` | 随机使用 `coffee near me`、`pharmacy shanghai`、`gas station los angeles` 等 | 地图/POI 搜索，脚本默认附带 `hl=en&gl=us`。 |
| trends | `https://trends.google.com/trends/explore` | `q` | 随机使用 `ai news`、`bitcoin`、`nba`、`旅游`，并带上 `geo`/`date` 示例 | 趋势检索，脚本默认附带 `geo=US&hl=en`，可按需改地理范围或时间窗口。 |
| reviews | `https://www.google.com/search` | `q`，`tbm=lcl` | 随机使用 `best sushi in nyc`、`coffee shop san francisco`、`hotel shenzhen` 等 | 本地点评/门店列表，垂直参数 `tbm=lcl` 由脚本自动追加。指定门店 ID 时可使用 `lrd` 参数。 |
| lens | `https://lens.google.com/uploadbyurl` | `url` | 随机使用三张公开图片的 URL | Lens 以图搜图。必须提供图片 URL；脚本会追加 `hl=en`。 |
| hotels | `https://www.google.com/travel/hotels` | `q` | 随机使用城市/酒店名，含带日期的示例请求 | 酒店搜索，可在 `q` 中写城市或酒店名，脚本默认 `hl=en&gl=us`。 |
| flights | `https://www.google.com/travel/flights` | `q` | 随机使用 `SFO to JFK`、`LAX to NRT`、`PEK to PVG` 等航线 | 机票航班查询，`q` 可写“出发地 to 目的地”，同样默认 `hl=en&gl=us`。 |

运行示例（开启所有预置引擎并保存汇总）：

```bash
python brigtdata_serp.py -t YOUR_TOKEN -z serp_api1 --all-engines -n 5 -c 3 -o summary.csv
```

