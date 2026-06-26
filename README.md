# epic-leek-quant · 史诗级韭菜量化

> 把 B 站财经 UP 主"史诗级韭菜"的格系"捡烟蒂"深度价值投资理念，量化为可在聚宽（JoinQuant）平台上回测、归因、实盘校准的多因子选股体系。
>
> *Quantifying the deep-value "cigar-butt" investing philosophy of Bilibili finance creator "Epic Leek" into a backtestable, attributable, live-tradable multi-factor stock-selection framework on the JoinQuant platform.*
>
> **本仓库是研究仓库，不是投资建议。** 详见末尾免责声明。
> *This is a research repository, not investment advice. See disclaimer at the bottom.*

---

## 项目定位 · Project Positioning

核心理念量化表述：**利用市场系统性情绪偏差（neglect / pessimism），在控制破产风险的前提下，捕获跨周期的估值回归收益。**

*Quantified core idea: **Capture cross-cycle mean-reversion returns by exploiting systematic market sentiment biases (neglect / pessimism), while controlling for bankruptcy risk.***

五大投资原则 → 五大可计算因子 / *Five investing principles → five computable factors*：

| 原则 / Principle | 因子 / Factor | 关键字段 / Key Fields |
|------|------|---------|
| 财务稳健 + EV 为负 / Financial soundness + negative EV | `factor_ev_negative` | `market_cap` / `total_liability` / `cash_equivalents` |
| 高盈利收益率 / High earnings yield | `factor_low_pe` / `earnings_yield` | `pe_ttm` / `net_profit` / `market_cap` |
| 持续股东回报 / Sustained shareholder return | `div_yield_continuous` + 连续分红标志 / consecutive-dividend flag | `div_yield` / `dividend_payable` |
| 国企背景 / State-owned background | `factor_state_owned` | `actual_controller` |
| 财务质量（低杠杆 + 正经营现金流）/ Financial quality (low leverage + positive OCF) | `factor_low_leverage` / `factor_positive_cf` | `total_liability` / `total_assets` / `operating_cash_flow` |

> **重要基调**：原文档中的"年化 16–20%""超额 409%""显著且持续"等数字**一律视为待验证假设**，不作为结论或通过标准。所有结论由 IC 体系（NW-t、IC_IR、Q1 多头绝对收益、Q1–Q5 单调性、跨样本一致性）支撑。
>
> *Important tone: Optimistic figures from the original document ("annualized 16–20%", "excess 409%", "significant and persistent") are **treated as hypotheses to be validated**, not as conclusions or pass criteria. All conclusions are supported by the IC system (NW-t, IC_IR, Q1 long-only absolute return, Q1–Q5 monotonicity, cross-sample consistency).*

---

## 目录结构 · Directory Structure

```
epic-leek-quant/
├── README.md                            # 本文件 / this file
├── LICENSE
├── .gitignore
├── epic-leek-value-investing.md         # 原文（理论 + 聚宽框架，不改）/ original, unchanged
├── docs/
│   ├── agent-workflow.md                # Agent 工作流程（不改）/ unchanged
│   ├── plan-review.md                   # 计划审查报告（不改）/ unchanged
│   ├── PROJECT-PLAN.md                  # ★ 单一事实来源 / single source of truth
│   ├── phase-0-status.md                # Phase 0 完成与缺口报告 / completion & gap report
│   └── prompts/                         # DeepSeek 提示词模板 / prompt templates
│       ├── 00-system-role.md
│       ├── 01-research-spec-design.md
│       ├── 02-flash-execution.md
│       ├── 03-code-review-gate.md
│       └── 04-result-adjudication.md
├── joinquant/                           # 聚宽策略与数据接口封装层 / JoinQuant data interface layer
│   ├── data_layer.py                    # PIT 查询 / 退市停牌 / 限价单撮合 / 先卖后买
│   ├── factor_lib.py                    # 五大因子计算（修复 B1/B2）/ five-factor calc — Phase 1
│   └── strategies/                      # 每个因子/变体一个策略文件 / one per factor — Phase 1+
├── research/
│   ├── thinking-prompt.md               # 分析框架（不改）/ analytical framework, unchanged
│   ├── theory-framework.md              # 互联网检索整理的理论框架与 A 股适配 / web-researched
│   ├── _index.md                        # ★ 研究记录索引/看板 / research index & dashboard
│   ├── specs/                           # 每轮 research-spec / per-round specs
│   ├── reports/                         # 每轮分析报告 / per-round reports
│   └── decisions/                       # 因子去留/权重/参数决策 / factor decisions
└── results/                             # 回测原始输出（净值 CSV、IC 表），大文件 gitignore
```

---

## 阶段路线图 · Phase Roadmap

| Phase | 目标 / Goal | 状态 / Status |
|-------|------|------|
| **0** 脚手架 / Scaffolding | 目录 / 数据接口 / 因子库 / 模板 | ✅ 完成（见 `docs/phase-0-status.md`）|
| **1** 单因子验证 / Single-factor validation | 5 大因子逐一 IC 验证，从 EV<0 起步 | ⏳ 待启动 / Pending |
| **2** 多因子合成 / Multi-factor synthesis | Fama-MacBeth 筛选 + Risk Parity 合成 | ⏸ 阻塞于 Phase 1 |
| **3** 压力测试 / Stress testing | Purged WF-CV / 制度断点 / DSR | ⏸ 阻塞于 Phase 2 |
| **4** 实盘校准 / Live-trading prep | 成本净收益 / 容量 / 拥挤度 / 失效预案 | ⏸ 阻塞于 Phase 3 |

每阶段通过对应 Gate 才能进入下一阶段。Gate 定义见 `docs/PROJECT-PLAN.md` 第三节。

*Each phase must pass its Gate before advancing. Gate definitions: `docs/PROJECT-PLAN.md` §3.*

---

## 关键口径 · Key Calibration

> 与原文档冲突时以此为准。完整口径见 `docs/PROJECT-PLAN.md` 第一节。
>
> *Overrides original docs on conflict. Full calibration in `docs/PROJECT-PLAN.md` §1.*

| 议题 / Topic | 统一口径 / Unified Calibration |
|------|---------|
| 换仓频率 / Rebalance | **季度调仓**（每季报披露截止后首个交易日：4/30、8/31、10/31、次年 4/30）/ **Quarterly** |
| 佣金 / Commission | 万 2.5 双边 / 0.025% both sides |
| 印花税 / Stamp tax | 千 1 仅卖 / 0.1% sell-only |
| 最低佣金 / Min commission | 5 元每笔 / ¥5 per trade |
| 滑点 / Slippage | 0.3% |
| 冲击成本 / Market impact | 分级估算（Phase 4 升级 Almgren-Chriss）/ tiered (Almgren-Chriss in Phase 4) |
| 基准 / Benchmark | 主沪深 300 全收益 + 辅中证 800 全收益；强制 TE/IR / CSI 300 TR + CSI 800 TR; TE/IR mandatory |
| 退市/涨跌停/T+1/停牌 | 必须纳入；限价单排队撮合；先卖后买；停牌冻结至复牌首日 / must include; queue matching; sell-first; freeze until resumption |
| PIT 数据 / PIT data | `get_fundamentals(date=)` PIT 语义与财报修正覆盖，代码审核门禁必检 / PIT semantics & revision coverage; code-review gate mandatory |
| 中性化 / Neutralization | 单因子 IC 同时给行业+市值中性化；Phase 2 用 Fama-MacBeth 控制市值 / both industry & size; Fama-MacBeth in Phase 2 |
| 二值因子 / Binary factors | 直接等权求和，不做 Z-score；连续因子做 1%/99% winsorize + Z-score / equal-weight sum, no Z-score; continuous: winsorize + Z-score |
| `debt_to_assets` | 先算比率，再取中位数（修复 B1）/ ratio first, then median (fixes B1) |

---

## 快速开始 · Quick Start

### 在聚宽研究环境中运行 / Run in JoinQuant

1. **数据接口层 / Data layer**：`joinquant/data_layer.py` 提供统一的 PIT 查询、退市/停牌标记、限价单撮合器。Phase 1 起的所有策略必须经此层访问数据，**禁止直接调用 `get_fundamentals`**。
   *Provides unified PIT query, delisting/suspension tagging, limit-order matching. All Phase 1+ strategies **must** access data through this layer — direct `get_fundamentals` is forbidden.*

2. **因子库 / Factor library**：`joinquant/factor_lib.py` 实现 `calculate_all_factors` 与 `composite_score`（Phase 1 随首个 spec 落地），已修复原文档第四章的 B1/B2 缺陷。
   *Implements factor calc (lands with first spec in Phase 1), with B1/B2 defects from original Ch.4 fixed.*

3. **执行流程 / Workflow**：见 `docs/agent-workflow.md` 与 `docs/prompts/` 五份提示词模板。每轮研究先由高级模型写 `research-spec`，Flash 执行，独立代码审核，再分析结果。
   *See `docs/agent-workflow.md` and five prompt templates. Each round: senior model writes spec → Flash executes → independent code review → result analysis.*

### 在本地仓库中协作 / Collaborate Locally

```bash
git clone https://github.com/IdealAuror/epic-leek-quant.git
cd epic-leek-quant
# 阅读顺序建议 / Suggested reading order:
#   1. README.md（本文件 / this file）
#   2. docs/PROJECT-PLAN.md（单一事实来源 / single source of truth）
#   3. docs/plan-review.md（口径冲突清单 / calibration conflict list）
#   4. research/thinking-prompt.md（学术标准 / academic standard）
#   5. research/theory-framework.md（互联网检索整理的理论框架 / web-researched framework）
#   6. epic-leek-value-investing.md（原文理论，乐观数字需标注"待验证" / original; optimistic figures need "to-be-validated" tag）
```

---

## 与原文档的关系 · Relationship to Original Documents

- 三份原文档（`epic-leek-value-investing.md`、`docs/agent-workflow.md`、`research/thinking-prompt.md`）作为**背景与理念来源**保留，**不修改**。
  *The three original docs are preserved as **background & idea sources**, **unchanged**.*

- 当原文档与 `docs/PROJECT-PLAN.md` 冲突时，**以 PROJECT-PLAN 为准**。冲突调和依据见 `docs/plan-review.md` 第二节。
  *On conflict with `docs/PROJECT-PLAN.md`, **PROJECT-PLAN prevails**. Reconciliation: `docs/plan-review.md` §2.*

- 调和原则：**以 `thinking-prompt.md` 的严格学术标准为准绳，以 theory 文档的乐观数字为"待验证假设"**。
  *Reconciliation principle: **`thinking-prompt.md`'s rigorous academic standard is the yardstick; the theory doc's optimistic figures are "hypotheses to be validated".***

- `research/theory-framework.md` 是对原文档的互联网检索交叉验证与 A 股适配，新增 H9–H15 待验证假设。
  *Cross-validates the original via web research and adapts to A-share specifics, adding hypotheses H9–H15.*

---

## 免责声明 · Disclaimer

本仓库所有内容（案例、数据、回测框架、因子定义）均基于公开信息整理，仅供投资研究与量化学习参考，**不构成任何形式的投资建议**。股市有风险，投资需谨慎。过往业绩不代表未来收益。

*All content in this repository (cases, data, backtesting frameworks, factor definitions) is compiled from public information and is for investment research and quantitative learning reference only. **It does not constitute any form of investment advice.** The stock market carries risk; invest with caution. Past performance does not guarantee future returns.*
