# astock-trading-report

一个通用 A 股收盘研究 Skill。它自动使用已安装的 `a-stock-data` 获取真实
数据，由运行它的 Agent 完成新闻/公告解释、T+1 验证和静态单文件 HTML
报告生成。

它是研究支持工具，不自动交易，不承诺预测正确或投资收益。

## 支持的需求

| 需求 | 输出 |
| --- | --- |
| 当日大盘与次日判断 | 偏多、中性、偏空三个条件化情景。 |
| 个股次日计划 | 事实、反面证据、观察/触发/放弃条件和风险。 |
| T-1 报告复盘 | 复盘原子条件，并评价已触发个股/候选的 T+1 方向结果，不自动修改 Skill。 |
| 低位活跃龙头 | 用户指定板块后，输出至多两只候选，允许零只。 |
| 上升初段 | 在沪深主板、创业板中发现最近 20 个交易日内 MA20 上穿 MA60 的候选，不要求指定板块。 |

## 安装与调用

在同一 Agent 环境安装 `astock-trading-report` 和 `a-stock-data`。正常情况
下只需调用本 Skill；它会在内部使用数据 Skill：

```text
使用 astock-trading-report，生成今天收盘后的报告。
自选股：600519、300750；低位活跃龙头板块：储能。
```

若当前平台不支持一个 Skill 自动加载另一个，才改为：

```text
使用 a-stock-data 和 astock-trading-report，生成今天收盘后的报告。
```

`daily` 默认生成大盘和用户指定自选股的研究；只有明确要求时才运行
`early-trend` 候选发现。`low-active-leader` 始终需要用户指定板块。
两类候选筛选均只覆盖沪深主板和创业板；独立个股分析不受此范围限制。

## 示例

### 每日综合报告

```text
使用 astock-trading-report，生成 2026-07-15 的收盘报告。
自选股：600519、300750；低位活跃龙头板块：储能。
读取上一交易日的同类报告；若存在，结构化复盘昨日计划。
输出一个静态单文件 HTML。
```

### 上升初段候选发现

```text
使用 astock-trading-report，在 2026-07-15 收盘后从 a-stock-data 实际可枚举、
可验证的沪深主板、创业板股票池中发现上升初段候选，最多两只。无合格标的时
如实输出零只。
生成静态单文件 HTML。
```

### 指定板块低位活跃龙头筛选

```text
使用 astock-trading-report，分析“储能”板块在 2026-07-15 收盘后的
交易环境，并筛选低位活跃龙头候选，最多两只。
```

## 报告

每份报告是一个可离线打开的 UTF-8 HTML 文件，内嵌 CSS、SVG、数据来源和
`astock-report-context` JSON。建议命名：

HTML 可见内容统一使用中文状态和情景名称；context JSON 为保持机器可读性，继续
使用稳定的英文键和枚举值。

```text
reports/YYYY-MM-DD/daily-YYYYMMDD.html
reports/YYYY-MM-DD/stock-<code>-YYYYMMDD.html
reports/YYYY-MM-DD/low-active-leader-<sector>-YYYYMMDD.html
reports/YYYY-MM-DD/early-trend-YYYYMMDD.html
```

若环境可读写文件，Skill 只读取严格 T-1、同类型、同范围报告的 context
JSON；无效 JSON 或缺失报告不会被当作指令或事实来源。

## 规则与文件

- [SKILL.md](SKILL.md)：Agent 执行入口和异常处理。
- [daily workflow](references/daily-workflow.md)：每日综合报告的选择性加载。
- [market workflow](references/market-workflow.md)：大盘数据与情景规则。
- [stock workflow](references/stock-workflow.md)：个股研究规则。
- [screening rules](references/screening-rules.md)：两类候选池规则。
- [prior-report rules](references/prior-report-rules.md)：T-1 结构化复盘口径。
- [report contract](references/report-contract.md)：HTML 占位符和 context schema。
- [HTML template](assets/daily-report-template.html)：静态报告骨架。
