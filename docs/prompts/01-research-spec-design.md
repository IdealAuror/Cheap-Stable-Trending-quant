# Prompt 01 — research-spec 设计（高级模型）

> 本文件是高级模型设计 `research-spec` 的工作模板。每个因子 / 变体 / 合成方案
> 在执行前都必须先有一份 spec，落盘到 `research/specs/`，命名规范见
> `research/_index.md`。spec 一经落盘，**预注册的通过标准不可事后调整**。

---

## spec 的角色

spec 是高级模型与 Flash 之间的**契约**。它必须足够精确，使 Flash 在不查阅任何
外部资料的情况下能完整执行；又必须足够克制，不规定实现细节（让 Flash 自由选择
pandas / numpy 的写法）。

## spec 必须包含的字段

```yaml
# ============ 元信息 ============
spec_id: <见 _index.md 命名规范，如 P1-F1-EV-2026Q2-v1>
phase: <Phase 1 | Phase 2 | Phase 3 | Phase 4>
type: <single-factor | multi-factor | stress-test | live-prep>
created_at: <ISO 日期>
created_by: <高级模型 / 人类>
status: <draft | locked | executing | reviewed | passed | abandoned>

# ============ 假设 ============
hypothesis: |
  一句话陈述待检验假设。例："EV<0 因子在 A 股具有显著的正向选股能力。"
economic_intuition: |
  为什么这个因子应产生超额收益？补偿何种风险溢价 / 行为偏差？
  必须回应 thinking-prompt.md 对该因子的全部深层追问（壳价值、受限资金、
  填权、回购无区分度、国企内生性、质量=下行保护……）。
failure_condition: |
  明确的失败条件。例："IC_IR < 0.2 或 Q1 多头组未能跑赢基准。"

# ============ 回测设置（口径对齐 PROJECT-PLAN.md §1.1）============
backtest:
  time_range: [<起>, <止>]           # 例 [2014-01, 2026-06]
  samples:                            # 跨样本一致性是 Gate 1 必检项
    - <沪深300 | 中证500 | 全A>
  benchmark_primary: 000300.XSHG       # 沪深 300 全收益
  benchmark_secondary: <可选，中证 800 全收益>
  rebalance: quarterly                 # 季度调仓，对齐披露截止日
  pit_mode: strict                     # 强制 PIT，见 data_layer.is_data_available_at
  data_layer: joinquant/data_layer.py  # 必须经此层，禁止直接 get_fundamentals
  parallel:                            # 并行执行协议（见 prompts/02 第六节）
    enabled: <true | false>            # 是否启用多代理并行
    slice_dimensions:                  # 切片维度，仅 enabled=true 时填
      - <sample | param_grid | regime_break | variant>
    datamining_guard:                  # 并行时的多重比较防御
      report_all_slices: true          # 全量上报，禁止择优
      dsr_required: <true | false>     # 是否必须计算并报告 DSR

# ============ 因子定义 ============
factors:
  - name: factor_ev_negative
    type: binary                       # binary | continuous
    formula: "EV = market_cap + total_liability - cash_equivalents; EV<0 → 1"
    fields:
      market_cap: valuation.market_cap
      total_liability: balance.total_liability
      cash_equivalents: balance.cash_equivalents
    notes: |
      现金质量待校验：cash_equivalents 是否含受限资金？
      壳价值污染：2019 注册制前的收益需拆分。

# ============ 数据处理 ============
data_processing:
  winsorize: continuous_only           # 仅连续因子，1%/99%
  zscore: continuous_only              # 二值因子不做 Z-score（修复 B2）
  missing: drop_if_any_critical_missing
  critical_fields: [market_cap, pe_ttm, net_profit]
  neutralize:
    - industry                         # 行业中性化 IC 必报
    - size                             # 市值中性化 IC 必报（C11）

# ============ 分析方法 ============
analysis:
  primary_metric: spearman_rank_ic     # 月度截面
  significance: newey_west_t           # 滞后 4 期
  ic_ir: nw_std                        # 用 NW 标准差
  grouping: Q1-Q5                      # 等权月收益
  report_q1_absolute: true             # 必须单独报告 Q1 多头绝对收益
  ic_decay: [lag0, lag1, lag3, lag6]   # 衰减曲线
  market_cap_weighted_ic: true         # 市值加权 IC 对照组
  cross_sample_consistency: true       # 三样本方向一致

# ============ 预注册通过标准（Gate，不可事后调整）============
pass_criteria:
  gate: <Gate 1 | Gate 2+3 | Gate 4 | Gate 5>
  rules:
    - "Rank IC_IR > 0.3（NW 标准误，滞后 4 期）"
    - "Q1 多头组年化收益 > 基准年化 + 2%"
    - "Q1-Q5 分组单调"
    - "三样本（沪深300/中证500/全A）表现方向一致"
    - "行业中性化 IC 与市值中性化 IC 均显著"

# ============ 输出要求 ============
outputs:
  tables:
    - "表1: 月度 Rank IC 序列描述统计（均值、标准差、NW-t、IC_IR、胜率、正负比例）"
    - "表2: Q1-Q5 分组年化收益、年化波动、最大回撤、夏普比率"
    - "表3: 分市值档（大/中/小）Rank IC"
    - "表4: 分行业 Rank IC（行业中性化后）"
    - "表5: 市值中性化后 Rank IC"
  series:
    - "Q1 多头组累计净值曲线（每月末净值，CSV）"
    - "月度 Rank IC 时间序列（CSV）"
  notes:
    - "执行偏差记录：每处歧义 spec 怎么写 vs 实际怎么处理"
    - "数据缺失记录：spec 要求字段中聚宽没有的、如何处理"

# ============ 防 data-mining ============
anti_datamining:
  pre_registered: true                 # 通过标准已预注册
  prior_index_read: true               # 已读 research/_index.md 历史结论
  abandonment_rule: "连续两轮 IC_IR<0.2 → 放弃，记录原因"
  parallel_harking_guard:              # 并行模式下的 HARKing 防御（若 parallel.enabled=true）
    report_all_slices: true            # 全量上报含失败切片，禁止择优
    dsr_multitest_correction: <true | false>  # 多重比较修正
```

## 设计 spec 时的自检清单

设计完 spec 后，高级模型在落盘前自问：

- [ ] 假设是否回应了 thinking-prompt.md 对该因子的全部深层追问？
- [ ] 通过标准是否可证伪（即存在明确的失败条件）？
- [ ] 通过标准是否在结果出来前就写定？
- [ ] 是否要求了行业中性化 IC 与市值中性化 IC？
- [ ] 是否要求了三样本跨样本一致性？
- [ ] 是否要求了 Q1 多头绝对收益（而非仅 Q1-Q5 多空）？
- [ ] 是否要求经 data_layer 撮合层（退市/涨跌停/停牌）？
- [ ] 是否要求 PIT 强校验？
- [ ] 是否已读 `research/_index.md` 该因子的历史结论？

任一项不满足，spec 不得标记为 `locked`。

## spec 修订流程

spec 一旦 `locked`，**预注册的通过标准不可调整**。其他字段（如时间区间、样本）
的修订必须：

1. 在 `research/_index.md` 记录修订原因；
2. 生成新版本号（v1 → v2），旧版本保留；
3. 若修订涉及通过标准 → 视为新一轮假设，重置迭代计数器。
