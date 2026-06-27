---
name: P1-F5-LowVol-IC验证
overview: 创建低波动率因子（F5-LowVol）IC 验证 standalone 脚本，复用 F2-EP 框架，信号改为历史价格已实现波动率（取负），目标是通过市值中性化 t≥2 门槛后与 F2-EP 组合降回撤。同时记录 F4-ROE 放弃决策并清理 F4 相关回测脚本。
todos:
  - id: create-f5-ic-script
    content: 创建 F5-LowVol IC 验证 standalone 脚本，含 _calc_realized_volatility 核心函数、简化数据链、Gate 1 七条完整框架
    status: completed
  - id: cleanup-f4-files
    content: 删除 F4-ROE 和 F2F4-EP-ROE 回测脚本，保留 F4 IC 脚本作记录
    status: completed
  - id: update-decision-doc
    content: 更新决策文档，追加 F4-ROE 放弃决策和 F5-LowVol 启动记录
    status: completed
    dependencies:
      - cleanup-f4-files
---

## 产品概述

F2-EP 单因子回测年化 13.1% 但回撤 51.6%。F4-ROE 因子 IC 市值中性化 t=1.81 未过门槛，单因子回测负 alpha（-0.01），双因子 EP+ROE 全方位劣化。ROE 方向放弃，转向低波动率因子（F5-LowVol）作为降回撤新方向。

## 核心功能

- 创建 F5-LowVol 低波动率因子 IC 验证脚本（Gate 1 七条，决定性门槛：市值中性化后 NW-t >= 2）
- 信号 = -std(过去60交易日日收益率)，低波动 = 高信号 = IC>0 有效
- 对照信号 = 120交易日波动率，验证窗口敏感性
- 清理 F4-ROE 失败的回测脚本，更新决策文档记录 F4 放弃与 F5 启动

## 技术栈

- Python 3.6/3.7 兼容（聚宽研究环境，不用 f-string `=` 调试语法）
- standalone 脚本模式：内联所有依赖，聚宽研究环境直接粘贴运行
- scipy.stats（Spearman IC）、statsmodels（Newey-West t）、pandas/numpy
- 聚宽 API：get_price（历史价格）、get_fundamentals（valuation.market_cap）、get_industry（行业中性化）

## 实现方案

### 整体策略

基于 F2-EP IC 脚本（`research/scripts/P1-F2-EP-2026Q2-v1-ic_analysis-standalone.py`）的完整框架，替换因子计算层。低波动率因子只需价格数据 + market_cap（中性化用），数据链比 F2-EP/F4-ROE 大幅简化，不踩财务表字段名坑。

### 关键技术决策

**1. 波动率计算**

- 主信号：60交易日（约3个月）日收益率标准差
- 对照信号：120交易日（约6个月）日收益率标准差
- 信号 = -vol（取负，使低波动 = 高信号 = IC>0 有效）
- 过滤：至少30个有效日收益观测值（半窗口下限），避免新股/停牌导致样本不足
- 数据获取：`get_price(stocks, end_date=date_str, count=lookback_days+1, fields=['close'], skip_paused=False, panel=False, fq='post')`，count+1 因 pct_change 丢首行

**2. 数据链简化**

- 仅需 valuation 表（market_cap 列，单位亿元），无需 balance/income/cash_flow
- 无财务字段名探测陷阱（`_get_col` 仍保留兜底）
- 无 ROE 历史查询（F4 的 `_fetch_roe_from_finance` 返回0条问题不复现）

**3. Size 共线风险（决定性测试）**

- 低波动率因子天然与市值共线：小盘股波动大，LowVol 信号与 ln(market_cap) 强相关
- 市值中性化后 IC NW-t >= 2 是决定性门槛，若未过则 LowVol 也是 size proxy
- 银行股天然低波动：保留金融股剔除（sw_l1 严格相等匹配），避免信号被银行主导，同时与 F2-EP 口径一致便于后续组合

**4. 框架复用清单**

| 函数 | 来源 | 改动 |
| --- | --- | --- |
| `_get_stock_pool` | F2-EP | 原样复用（剔ST/次新/金融股） |
| `_exclude_finance_stocks` | F2-EP | 原样复用 |
| `_fetch_fundamentals_pit` | F2-EP | 简化：只查 valuation 表 |
| `_calc_realized_volatility` | 新建 | 核心新函数 |
| `_calculate_all_factors` | F2-EP | 替换：计算 vol_60d/vol_120d/signal |
| `build_cross_section` | F2-EP | 修改：调用波动率计算，过滤改为有效观测数 |
| `analyze_sample` | F2-EP | 修改：更新标签，对照改为 vol_120d |
| 其余工具函数 | F2-EP | 原样复用 |


### 性能考虑

- AllA 约2000+只股票，60交易日日频价格约12万行，单次 get_price 可处理
- 波动率按股票逐列算 std，O(N×T) 复杂度，N=2000, T=60，毫秒级
- 无跨期财报查询（F2-EP 的 `_fetch_roe_history` 要循环3年），单截面速度大幅提升

## 目录结构

```
project-root/
├── research/
│   ├── scripts/
│   │   └── P1-F5-LowVol-2026Q2-v1-ic_analysis-standalone.py  # [NEW] F5低波动率因子IC验证脚本。内联data_layer+factor_lib，聚宽研究环境直接粘贴运行。主信号vol_60d=-std(60日日收益率)，对照vol_120d。复用F2-EP的Gate 1七条框架（IC方向/Q单调性/NW-t/分市值档/行业中性化/断点/多空夏普），决定性门槛市值中性化t>=2。
│   └── decisions/
│       └── P1-F1-EV-2026Q2-v1-decision.md                     # [MODIFY] 追加Phase 1.3 F4-ROE验证结果+放弃决策，追加Phase 1.4 F5-LowVol启动记录。
├── joinquant/
│   └── strategies/
│       ├── P1-F4-ROE-2026Q2-v1-AllA-standalone.py             # [DELETE] F4-ROE单因子回测，负alpha(-0.01)，收益86%跑输基准121%。
│       └── P1-F2F4-EP-ROE-2026Q2-v1-AllA-standalone.py        # [DELETE] 双因子EP+ROE回测，全方位劣化(187%/0.18/57.1%)。
└── research/
    └── scripts/
        └── P1-F4-ROE-2026Q2-v1-ic_analysis-standalone.py       # [KEEP] F4-ROE IC脚本保留作记录，不删除。
```

## 关键代码结构

```python
def _calc_realized_volatility(date_str, stocks, lookback_days=60):
    """计算实现波动率：过去 lookback_days 交易日日收益率标准差。
    
    使用 get_price(count=lookback_days+1) 取 trailing 日收盘价，
    pct_change 算日收益率后取 std。count+1 因 pct_change 丢首行。
    
    返回 {code: vol}，vol = std(daily returns)。
    信号 = -vol（低波动 = 高信号 = IC>0 有效）。
    有效观测 < lookback_days/2 的股票返回 NaN（新股/长期停牌）。
    """
```