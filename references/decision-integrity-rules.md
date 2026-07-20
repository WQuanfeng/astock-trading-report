# 决策完整性规则

所有工作流在形成结论前应用本文件。目标是确保报告只使用当时可获得的信息、
区分研究条件与实际可交易性、披露筛选覆盖，并用预先确定的基准评价 T+1 结果。

## 信息可用时点

1. 将 context 的 `as_of` 作为本报告的信息截止时间，而不是市场收盘时间或文件
   生成时间；它可以覆盖收盘后已披露的信息，但不得晚于 `GENERATED_AT`。
2. 只有 `available_at <= as_of` 且 `point_in_time_safe=true` 的证据可以进入筛选、
   排名、行为判断、介入条件或方向结论。
3. 公告、新闻、研报和财务数据同时记录实际发布时间 `published_at` 与数据源可
   取得时间 `available_at`。财务报告期不能代替披露时间。
4. 行情和 K 线只使用目标交易日收盘前已经形成的数据；技术指标不得包含目标日
   之后的 K 线。收盘后抓取不等于前视，只要数据本身的可用时点不晚于 `as_of`。
5. 无法核验可用时点时，将 `point_in_time_safe` 设为 `false`，记录质量警告，
   并从会影响结论的证据中移除；不得用抓取时间猜测发布时间。
6. 复盘旧报告时仍以旧报告记录的 `as_of` 为边界，不得用后来修订或补录的数据
   重新解释当日计划。

每项 provenance 都写入 `published_at`、`available_at` 和
`point_in_time_safe`；不适用的 `published_at` 使用 `null`。

## A 股可交易性

对每只请求股票和最终候选收集：停复牌/特别处理状态、数据源给出的涨跌停锁定
状态、流动性、已知下一交易日限制、临近公司行动或其他交易限制。不要自行使用
通用涨跌幅百分比推断锁定状态；优先使用数据源返回的证券状态和适用涨跌停价。

| HTML 显示 | JSON `tradability.status` | 含义 |
| --- | --- | --- |
| 当前可正常观察 | `tradable` | 核心状态可核验，报告时点无已知交易限制。 |
| 当前交易受限，次日需复核 | `restricted` | 当前停牌、价格锁定、流动性不足或存在已知限制。 |
| 可交易性未知 | `unknown` | 缺少判断所需的核心状态。 |

- 用户指定股票即使受限也可继续研究，但不得生成介入条件。
- 筛选继续执行既有 ST、停牌和严重流动性排除规则。当前价格锁定不自动证明次日
  仍不可交易；可以保留已通过固定门槛的候选，但状态只能为“观察”，并将次日
  可交易性复核写入不介入原子条件。
- `tradability.status=unknown` 时，状态只能为“数据不足”或“观察”，不得显示
  “条件满足后可介入”。
- `liquidity_status` 必须引用明确的成交额或换手率事实及比较基准，不得只写主观
  标签；必要数据缺失时使用 `unknown`。
- 不得为计划在 T+1 新买入的普通 A 股股票安排同一交易日卖出；已有持仓的退出
  判断必须与新开仓计划分开。

## 筛选覆盖审计

两类候选筛选分别在 `screening_audits[]` 中按下列口径记录，且每只股票只计入
一个首要排除或失败原因：

| 字段 | 口径 |
| --- | --- |
| `universe_total` | 目标板块/市场范围内枚举到的股票数，尚未执行规则排除。 |
| `excluded_by_reason` | ST、停牌、历史不足、流动性不足、重大事件等规则排除计数。 |
| `universe_eligible` | `universe_total` 减去全部规则排除后的数量。 |
| `data_failures_by_reason` | 对资格池取数或计算失败的计数，不得混入规则排除。 |
| `universe_evaluated` | 取得全部必要门槛和排序字段并完成评价的数量。 |
| `coverage_rate` | `universe_evaluated / universe_eligible`；分母为零时使用 `null`。 |

必须满足：`universe_total = universe_eligible + 各排除数之和`，且
`universe_eligible = universe_evaluated + 各数据失败数之和`。无法对账时记录
质量警告，不得输出最终两只。

只有证据已确认股票确实不满足规则时才计入排除；无法取得足够历史或状态数据时
计入数据失败，不得伪装成规则排除。

HTML 同时显示资格池、已评价数、覆盖率、主要排除原因和数据失败原因。覆盖率低于
100% 时，只能称结果为“已评价子集中的最终候选”，不得暗示遗漏股票不可能入选。

## T+1 同期基准评价

在 T-1 计划中预先记录 `evaluation_benchmark`，不得在看到 T+1 结果后更换：

1. `stock`：用户指定基准优先；否则使用可唯一映射的主行业指数；再否则使用
   沪深 300。
2. `low-active-leader`：用户指定板块对应指数优先；不可得时使用沪深 300。
3. `early-trend`：可唯一映射的主行业指数优先；不可得时使用沪深 300。

无法取得稳定代码或同周期行情时将 `evaluation_benchmark` 设为 `null`，相对结果
为 `not_evaluable`。不得使用事后表现更好的指数替换原基准。

触发后，以 `triggered_at` 同一最小可靠周期的基准收盘值作为参考，计算到 T+1
收盘的基准收益和个股超额收益：

`benchmark_return_pct = (benchmark_t1_close / benchmark_reference_level - 1) * 100`

`excess_return_pct = t1_return_pct - benchmark_return_pct`

超额收益大于、等于或小于 0 时，`relative_outcome` 分别为 `outperformed`、
`matched` 或 `underperformed`。数据不足时为 `not_evaluable`。相对结果不替代
`selection_outcome`，也不能推翻已经发生的失效条件；二者必须并列展示。

本 Skill 不在此加入手续费、滑点、多周期持有或参数优化；这些属于另一套策略假设。
