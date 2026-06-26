# epic-leek-quant · 史诗级韭菜量化

> 把 B 站财经 UP 主"史诗级韭菜"的格系"捡烟蒂"深度价值投资理念，量化为可在聚宽（JoinQuant）平台上回测、归因、实盘校准的多因子选股体系。
>
> *Quantifying the deep-value "cigar-butt" investing philosophy of Bilibili finance creator "Epic Leek" into a backtestable, attributable, live-tradable multi-factor stock-selection framework on the JoinQuant platform.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Phase](https://img.shields.io/badge/Phase-0%20Scaffolding%20%E2%9C%85-brightgreen)](./docs/phase-0-status.md)
[![Platform](https://img.shields.io/badge/Platform-JoinQuant-blue)](https://www.joinquant.com)
[![Language](https://img.shields.io/badge/Language-Python%203.13-blue)](https://www.python.org)
[![Status](https://img.shields.io/badge/Status-Research%20Only-lightgrey)](#免责声明--disclaimer)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-ff69b4)](https://github.com/IdealAuror/epic-leek-quant/pulls)

> **Suggested GitHub topics** (set in repo About → settings): `quantitative-finance` · `value-investing` · `multi-factor` · `a-share` · `joinquant` · `backtesting` · `deep-value` · `graham-net-net` · `china-stock-market` · `factor-investing`

---

> **Language toggle** — GitHub does not run `<script>`, so the toggle below uses native `<details>` tags. Click the triangle to expand/collapse each language. **English is shown by default.**
>
> **语言切换** — GitHub 不执行 `<script>`，下方切换使用原生 `<details>` 标签。点击三角展开/折叠对应语言。**默认显示英文。**

<details open>
<summary><b>🌍 English</b> &nbsp;·&nbsp; <a href="#中文">Switch to Chinese →</a></summary>

---

## Overview

**epic-leek-quant** is a research repository that converts the deep-value "cigar-butt" investing philosophy of Bilibili finance creator **"Epic Leek" (史诗级韭菜)** into a backtestable, attributable, live-tradable multi-factor stock-selection framework on the [JoinQuant](https://www.joinquant.com) platform.

**Quantified core idea**: *Capture cross-cycle mean-reversion returns by exploiting systematic market sentiment biases (neglect / pessimism), while controlling for bankruptcy risk.*

### Five Investing Principles → Five Computable Factors

| Principle | Factor | Key Fields |
|-----------|--------|-----------|
| Financial soundness + negative EV | `factor_ev_negative` | `market_cap` / `total_liability` / `cash_equivalents` |
| High earnings yield (PE < 10) | `factor_low_pe` / `earnings_yield` | `pe_ttm` / `net_profit` / `market_cap` |
| Sustained shareholder return | `div_yield_continuous` + consecutive-dividend flag | `div_yield` / `dividend_payable` |
| State-owned background | `factor_state_owned` | `actual_controller` |
| Financial quality (low leverage + positive OCF) | `factor_low_leverage` / `factor_positive_cf` | `total_liability` / `total_assets` / `operating_cash_flow` |

> **Important tone**: Optimistic figures from the original document ("annualized 16–20%", "excess 409%", "significant and persistent") are **treated as hypotheses to be validated**, not as conclusions or pass criteria. All conclusions are supported by the IC system (NW-t, IC_IR, Q1 long-only absolute return, Q1–Q5 monotonicity, cross-sample consistency).

### Directory Structure

```
epic-leek-quant/
├── README.md                            # This file
├── LICENSE
├── .gitignore
├── epic-leek-value-investing.md         # Original (theory + JoinQuant framework, unchanged)
├── docs/
│   ├── agent-workflow.md                # Agent workflow (unchanged)
│   ├── plan-review.md                   # Plan review report (unchanged)
│   ├── PROJECT-PLAN.md                  # ★ Single source of truth
│   ├── phase-0-status.md                # Phase 0 completion & gap report
│   └── prompts/                         # DeepSeek prompt templates
│       ├── 00-system-role.md
│       ├── 01-research-spec-design.md
│       ├── 02-flash-execution.md
│       ├── 03-code-review-gate.md
│       └── 04-result-adjudication.md
├── joinquant/                           # JoinQuant strategy & data interface layer
│   ├── data_layer.py                    # PIT query / delisting & suspension / limit-order matching / sell-first
│   ├── factor_lib.py                    # Five-factor calc (B1/B2 fixed) — Phase 1
│   └── strategies/                      # One file per factor/variant (Phase 1+)
├── research/
│   ├── thinking-prompt.md               # Analytical framework (unchanged)
│   ├── theory-framework.md              # Theory framework from web research + A-share adaptation
│   ├── _index.md                        # ★ Research index / dashboard / iteration log
│   ├── specs/                           # Per-round research-spec
│   ├── reports/                         # Per-round analysis reports
│   └── decisions/                       # Factor keep/drop / weight / param decisions
└── results/                             # Backtest raw outputs (NAV CSV, IC tables), large files gitignored
```

### Phase Roadmap

| Phase | Goal | Status |
|-------|------|--------|
| **0** Scaffolding | Directory / data interface / factor lib / templates | ✅ Done (see `docs/phase-0-status.md`) |
| **1** Single-factor validation | IC validation for each of 5 factors, starting with EV<0 | ⏳ Pending |
| **2** Multi-factor synthesis | Fama-MacBeth screening + Risk Parity synthesis | ⏸ Blocked by Phase 1 |
| **3** Stress testing | Purged WF-CV / regime breaks / DSR | ⏸ Blocked by Phase 2 |
| **4** Live-trading prep | Net-of-cost return / capacity / crowding / failure plan | ⏸ Blocked by Phase 3 |

Each phase must pass its Gate before advancing. Gate definitions: `docs/PROJECT-PLAN.md` §3.

### Key Calibration (overrides original docs on conflict)

Full calibration in `docs/PROJECT-PLAN.md` §1. Summary:

| Topic | Unified Calibration |
|-------|--------------------|
| Rebalance frequency | **Quarterly** (first trading day after each disclosure deadline: 4/30, 8/31, 10/31, next-year 4/30) |
| Commission / stamp tax / min commission | 0.025% both sides / 0.1% sell-only / ¥5 per trade |
| Slippage / market impact | 0.3% / tiered estimation (Almgren-Chriss in Phase 4) |
| Benchmark | Primary: CSI 300 Total Return; Secondary: CSI 800 Total Return. TE/IR mandatory |
| Delisting / price limits / T+1 / suspension | Must include; limit-order queue matching; sell-first; freeze until resumption |
| PIT data | `get_fundamentals(date=)` PIT semantics & financial-report revision coverage; code-review gate mandatory |
| Neutralization | Single-factor IC reports both industry- and size-neutralized; Phase 2 uses Fama-MacBeth to control size |
| Binary factors | Direct equal-weight sum, no Z-score; continuous factors: 1%/99% winsorize + Z-score |
| `debt_to_assets` | Compute ratio first, then median (fixes B1) |

### Quick Start

#### Run in JoinQuant Research Environment

1. **Data layer**: `joinquant/data_layer.py` provides unified PIT query, delisting/suspension tagging, and limit-order matching. All Phase 1+ strategies **must** access data through this layer — direct `get_fundamentals` is forbidden.
2. **Factor library**: `joinquant/factor_lib.py` implements `calculate_all_factors` and `composite_score` (lands with the first spec in Phase 1), with B1/B2 defects from the original Ch.4 fixed.
3. **Workflow**: See `docs/agent-workflow.md` and the five prompt templates in `docs/prompts/`. Each research round: senior model writes `research-spec` → Flash executes → independent code review → result analysis.

#### Collaborate Locally

```bash
git clone https://github.com/IdealAuror/epic-leek-quant.git
cd epic-leek-quant
# Suggested reading order:
#   1. README.md (this file)
#   2. docs/PROJECT-PLAN.md (single source of truth)
#   3. docs/plan-review.md (calibration conflict list)
#   4. research/thinking-prompt.md (academic standard)
#   5. research/theory-framework.md (web-researched theory framework)
#   6. epic-leek-value-investing.md (original theory; optimistic figures need "to-be-validated" tag)
```

### Relationship to Original Documents

- The three original docs (`epic-leek-value-investing.md`, `docs/agent-workflow.md`, `research/thinking-prompt.md`) are preserved as **background & idea sources**, **unchanged**.
- On conflict with `docs/PROJECT-PLAN.md`, **PROJECT-PLAN prevails**. Conflict reconciliation: `docs/plan-review.md` §2.
- Reconciliation principle: **`thinking-prompt.md`'s rigorous academic standard is the yardstick; the theory doc's optimistic figures are "hypotheses to be validated".**
- `research/theory-framework.md` cross-validates the original via web research and adapts to A-share specifics, adding hypotheses H9–H15.

### Theory Foundation

The "EV<0" factor is an extreme subset of **Benjamin Graham's Net-Net (NCAV) strategy**. Mohanty & Oxman (2026) empirically tested NCAV on US stocks over 50 years (1969–2019): value-weighted NCAV portfolio earned ~1.94%/month (≈25.6% annualized), with alpha of 1.09%/month (13.9%/year) surviving controls for Fama-French five factors, liquidity, and January effect — and the premium persists after size matching. This provides **academic legitimacy at the school-of-thought level**, not data-mining. See `research/theory-framework.md` for full literature and A-share adaptation.

### Disclaimer

All content in this repository (cases, data, backtesting frameworks, factor definitions) is compiled from public information and is for investment research and quantitative learning reference only. **It does not constitute any form of investment advice.** The stock market carries risk; invest with caution. Past performance does not guarantee future returns.

---

</details>

<a name="中文"></a>
<details>
<summary><b>🌍 中文</b> &nbsp;·&nbsp; <a href="#english">Switch to English →</a></summary>

---

## 项目概览

**epic-leek-quant** 是一个研究仓库，把 B 站财经 UP 主**"史诗级韭菜"**的格系"捡烟蒂"深度价值投资理念，转化为可在[聚宽](https://www.joinquant.com)平台上回测、归因、实盘校准的多因子选股框架。

**核心理念量化表述**：*利用市场系统性情绪偏差（neglect / pessimism），在控制破产风险的前提下，捕获跨周期的估值回归收益。*

### 五大投资原则 → 五大可计算因子

| 原则 | 因子 | 关键字段 |
|------|------|---------|
| 财务稳健 + EV 为负 | `factor_ev_negative` | `market_cap` / `total_liability` / `cash_equivalents` |
| 高盈利收益率（PE < 10） | `factor_low_pe` / `earnings_yield` | `pe_ttm` / `net_profit` / `market_cap` |
| 持续股东回报 | `div_yield_continuous` + 连续分红标志 | `div_yield` / `dividend_payable` |
| 国企背景 | `factor_state_owned` | `actual_controller` |
| 财务质量（低杠杆 + 正经营现金流） | `factor_low_leverage` / `factor_positive_cf` | `total_liability` / `total_assets` / `operating_cash_flow` |

> **重要基调**：原文档中的"年化 16–20%""超额 409%""显著且持续"等数字**一律视为待验证假设**，不作为结论或通过标准。所有结论由 IC 体系（NW-t、IC_IR、Q1 多头绝对收益、Q1–Q5 单调性、跨样本一致性）支撑。

### 目录结构

```
epic-leek-quant/
├── README.md                            # 本文件
├── LICENSE
├── .gitignore
├── epic-leek-value-investing.md         # 原文（理论 + 聚宽框架，不改）
├── docs/
│   ├── agent-workflow.md                # Agent 工作流程（不改）
│   ├── plan-review.md                   # 计划审查报告（不改）
│   ├── PROJECT-PLAN.md                  # ★ 单一事实来源（冲突时以此为准）
│   ├── phase-0-status.md                # Phase 0 完成与缺口报告
│   └── prompts/                         # DeepSeek 提示词模板
│       ├── 00-system-role.md
│       ├── 01-research-spec-design.md
│       ├── 02-flash-execution.md
│       ├── 03-code-review-gate.md
│       └── 04-result-adjudication.md
├── joinquant/                           # 聚宽策略与数据接口封装层
│   ├── data_layer.py                    # PIT 查询 / 退市停牌 / 限价单撮合 / 先卖后买
│   ├── factor_lib.py                    # 五大因子计算（修复 B1/B2 后口径）— Phase 1 实现
│   └── strategies/                      # 每个因子/变体一个策略文件（Phase 1 起填充）
├── research/
│   ├── thinking-prompt.md               # 分析框架（不改）
│   ├── theory-framework.md              # 互联网检索整理的理论框架与 A 股适配
│   ├── _index.md                        # ★ 研究记录索引/看板/迭代追溯
│   ├── specs/                           # 每轮 research-spec
│   ├── reports/                         # 每轮分析报告
│   └── decisions/                       # 因子去留/权重/参数决策记录
└── results/                             # 回测原始输出（净值 CSV、IC 表），大文件 gitignore
```

### 阶段路线图

| Phase | 目标 | 状态 |
|-------|------|------|
| **0** 脚手架 | 目录 / 数据接口 / 因子库 / 模板 | ✅ 完成（见 `docs/phase-0-status.md`）|
| **1** 单因子验证 | 5 大因子逐一 IC 验证，从 EV<0 起步 | ⏳ 待启动 |
| **2** 多因子合成 | Fama-MacBeth 筛选 + Risk Parity 合成 | ⏸ 阻塞于 Phase 1 |
| **3** 压力测试 | Purged WF-CV / 制度断点 / DSR | ⏸ 阻塞于 Phase 2 |
| **4** 实盘校准 | 成本净收益 / 容量 / 拥挤度 / 失效预案 | ⏸ 阻塞于 Phase 3 |

每阶段通过对应 Gate 才能进入下一阶段。Gate 定义见 `docs/PROJECT-PLAN.md` 第三节。

### 关键口径（与原文档冲突时以此为准）

完整口径见 `docs/PROJECT-PLAN.md` 第一节，摘要：

| 议题 | 统一口径 |
|------|---------|
| 换仓频率 | **季度调仓**（每季报披露截止后首个交易日：4/30、8/31、10/31、次年 4/30） |
| 佣金 / 印花税 / 最低佣金 | 万 2.5 双边 / 千 1 仅卖 / 5 元每笔 |
| 滑点 / 冲击成本 | 0.3% / 分级估算（Phase 4 升级 Almgren-Chriss） |
| 基准 | 主：沪深 300 全收益；辅：中证 800 全收益。强制报告 TE/IR |
| 退市股 / 涨跌停 / T+1 / 停牌 | 必须纳入；限价单排队撮合；先卖后买；停牌冻结至复牌首日 |
| PIT 数据 | `get_fundamentals(date=)` 的 PIT 语义与财报修正覆盖，代码审核门禁必检 |
| 中性化 | 单因子 IC 同时给行业中性化与市值中性化；Phase 2 用 Fama-MacBeth 控制市值 |
| 二值因子 | 直接等权求和，不做 Z-score；连续因子做 1%/99% winsorize + Z-score |
| `debt_to_assets` 计算 | 先算比率，再取中位数（修复 B1） |

### 快速开始

#### 在聚宽研究环境中运行

1. **数据接口层**：`joinquant/data_layer.py` 提供统一的 PIT 查询、退市/停牌标记、限价单撮合器。Phase 1 起的所有策略必须经此层访问数据，**禁止直接调用 `get_fundamentals`**。
2. **因子库**：`joinquant/factor_lib.py` 实现 `calculate_all_factors` 与 `composite_score`（Phase 1 随首个 spec 落地），已修复原文档第四章的 B1/B2 缺陷。
3. **执行流程**：见 `docs/agent-workflow.md` 与 `docs/prompts/` 五份提示词模板。每轮研究先由高级模型写 `research-spec`，Flash 执行，独立代码审核，再分析结果。

#### 在本地仓库中协作

```bash
git clone https://github.com/IdealAuror/epic-leek-quant.git
cd epic-leek-quant
# 阅读顺序建议：
#   1. README.md（本文件）
#   2. docs/PROJECT-PLAN.md（单一事实来源）
#   3. docs/plan-review.md（口径冲突清单）
#   4. research/thinking-prompt.md（学术标准）
#   5. research/theory-framework.md（互联网检索整理的理论框架）
#   6. epic-leek-value-investing.md（原文理论，乐观数字需标注"待验证"）
```

### 与原文档的关系

- 三份原文档（`epic-leek-value-investing.md`、`docs/agent-workflow.md`、`research/thinking-prompt.md`）作为**背景与理念来源**保留，**不修改**。
- 当原文档与 `docs/PROJECT-PLAN.md` 冲突时，**以 PROJECT-PLAN 为准**。冲突调和依据见 `docs/plan-review.md` 第二节。
- 调和原则：**以 `thinking-prompt.md` 的严格学术标准为准绳，以 theory 文档的乐观数字为"待验证假设"**。
- `research/theory-framework.md` 是对原文档的互联网检索交叉验证与 A 股适配，新增 H9–H15 待验证假设。

### 理论根基

"EV 为负"因子是**本杰明·格雷厄姆 Net-Net（NCAV）策略**的极端子集。Mohanty & Oxman (2026) 对美股 50 年（1969–2019）NCAV 策略的实证：价值加权组合月均收益约 1.94%（年化约 25.6%），控制 Fama-French 五因子、流动性、一月效应后 alpha 仍达 1.09%/月（13.9%/年），且规模匹配后溢价持续——这从**流派层面**提供了学术正当性，而非数据挖掘。完整文献与 A 股适配见 `research/theory-framework.md`。

### 免责声明

本仓库所有内容（案例、数据、回测框架、因子定义）均基于公开信息整理，仅供投资研究与量化学习参考，**不构成任何形式的投资建议**。股市有风险，投资需谨慎。过往业绩不代表未来收益。

---

</details>

<a name="english"></a>

---

> **License**: MIT — see [LICENSE](./LICENSE).
> **Repo**: https://github.com/IdealAuror/epic-leek-quant
> **Phase 0 status**: ✅ Complete — see [`docs/phase-0-status.md`](./docs/phase-0-status.md) for DoD checklist and Phase 1 entry conditions.
