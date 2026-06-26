# Phase 0 完成与缺口报告

> 对照 `docs/PROJECT-PLAN.md` §三 Phase 0 的 DoD 逐项核对。本报告是 Phase 0 的
> 收尾产出，也是 Phase 1 启动的前置确认。

---

## 一、Phase 0 DoD 逐项核对

| DoD 项（PROJECT-PLAN §五） | 状态 | 交付物 |
|---------------------------|------|--------|
| 目录齐全 | ✅ | 见下方"目录结构" |
| data_layer.py：PIT 查询 + 退市/停牌标记 + 限价单撮合 + 先卖后买 | ✅ | `joinquant/data_layer.py` |
| data_layer PIT 单测通过 | ✅ | `data_layer._local_smoke_test`（本地冒烟，含 PIT 截止日断言） |
| data_layer 三类撮合单测（涨停/跌停/停牌） | ⚠️ 部分 | `simulate_limit_order` 已实现三类逻辑；单测目前内嵌于冒烟测试，Phase 1 启动前补独立单测文件 |
| factor_lib.py：修复后因子计算 | ⏳ 推迟 | 推迟到 Phase 1 首个因子 spec 落地时一并实现（见"缺口说明"） |
| `_index.md` 命名规范就位 | ✅ | `research/_index.md` |
| spec/报告模板就位 | ✅ | `docs/prompts/01`（spec 模板）+ `research/_index.md`（命名规范） |
| `.gitignore` | ✅ | 根目录 `.gitignore` |
| README 补全 | ✅ | 根目录 `README.md` |

## 二、本批次交付物清单

### 计划文档层（本批次重点）

| 文件 | 用途 |
|------|------|
| `README.md`（重写） | 项目定位、目录结构、阶段路线图、关键口径、快速开始 |
| `.gitignore`（新建） | results 大文件、密钥、聚宽缓存、草稿 |
| `docs/prompts/00-system-role.md` | 高级模型恒定身份与行为基线 |
| `docs/prompts/01-research-spec-design.md` | spec 设计模板（含预注册通过标准） |
| `docs/prompts/02-flash-execution.md` | Flash 执行守则（歧义处理、澄清通道、偏差记录） |
| `docs/prompts/03-code-review-gate.md` | 代码审核门禁（结果分析之前，8 大类清单） |
| `docs/prompts/04-result-adjudication.md` | 结果判定流程（Gate 1–5、不通过诊断、六机制） |
| `research/_index.md` | 研究看板：命名规范、因子状态、待验证假设、迭代追溯 |
| `research/specs/README.md` | spec 目录说明与修订规则 |
| `research/reports/README.md` | 报告目录说明与顺序约束 |
| `research/decisions/README.md` | 决策目录说明与模板 |
| `results/README.md` | 结果目录说明、命名规范、复现路径 |
| `docs/phase-0-status.md` | 本文件 |

### 代码层（前置部分）

| 文件 | 用途 | 状态 |
|------|------|------|
| `joinquant/data_layer.py` | PIT 查询 / 退市停牌日历 / 限价单撮合 / 先卖后买 / 成本模型 | ✅ |
| `joinquant/factor_lib.py` | 五大因子计算（修复 B1/B2） | ⏳ 推迟 |

## 三、目录结构（当前）

```
epic-leek-quant/
├── README.md                            ✅ 重写
├── LICENSE
├── .gitignore                           ✅ 新建
├── epic-leek-value-investing.md         原文，不改
├── docs/
│   ├── agent-workflow.md                原文，不改
│   ├── plan-review.md                   原文，不改
│   ├── PROJECT-PLAN.md                  ★ 单一事实来源
│   ├── phase-0-status.md                ✅ 新建（本文件）
│   └── prompts/                         ✅ 新建
│       ├── 00-system-role.md
│       ├── 01-research-spec-design.md
│       ├── 02-flash-execution.md
│       ├── 03-code-review-gate.md
│       └── 04-result-adjudication.md
├── joinquant/
│   ├── data_layer.py                    ✅ 新建
│   └── strategies/                      ⏳ Phase 1 起填充
├── research/
│   ├── thinking-prompt.md               原文，不改
│   ├── _index.md                        ✅ 新建
│   ├── specs/      ✅ README 占位
│   ├── reports/    ✅ README 占位
│   └── decisions/  ✅ README 占位
└── results/                             ✅ README 占位
```

与 `PROJECT-PLAN.md` §二的目标目录结构**完全对齐**。

## 四、缺口说明

### 缺口 1：factor_lib.py 推迟到 Phase 1

**决策**：不在 Phase 0 空写 factor_lib.py，而是与 Phase 1 第一个因子（EV<0）的
spec 一并落地。

**理由**：
- 原文档第四章已有修复后的参考片段（PROJECT-PLAN.md §1.3），口径明确，Phase 1
  启动时可快速实现。
- 空写 factor_lib 而无 spec 校验，容易写出"看起来对但与首个 spec 不匹配"的代码，
  反而增加返工。
- Phase 0 的核心价值是**数据接口与撮合口径**（data_layer），因子计算是 Phase 1
  的主体工作。

**风险**：低。data_layer 已封装所有 PIT 与撮合逻辑，factor_lib 只是薄薄一层
因子计算，不阻塞 Phase 1 启动。

### 缺口 2：撮合层独立单测

**决策**：`simulate_limit_order` 的三类场景（涨停/跌停/停牌）单测目前内嵌于
`_local_smoke_test`，Phase 1 启动前补独立单测文件 `joinquant/tests/`。

**理由**：完整单测需要分钟级价格数据构造一字涨停/盘中打开等场景，本地无聚宽
数据无法跑通；Phase 1 在聚宽环境中补齐更高效。

### 缺口 3：严格版撮合（分钟级排队）

**决策**：当前 `simulate_limit_order` 为日级简化版（全成交或全不成交），严格版
（分钟级排队、部分成交、成交量加权）推迟到 Phase 4 实盘校准。

**理由**：见 `data_layer.py` 末尾注释。Phase 1–3 的因子验证用日级版即可保证
结论方向可信；严格版主要影响容量估算与净收益精度，是 Phase 4 的事。

## 五、Phase 1 启动前置确认

进入 Phase 1（单因子验证，从 EV<0 起步）前，确认以下条件已满足：

- [x] Phase 0 目录与模板齐全
- [x] data_layer.py 就位，PIT 与撮合口径统一
- [x] prompts 五件套就位，高级模型 / Flash 边界清晰
- [x] `_index.md` 命名规范就位
- [ ] factor_lib.py 实现（与首个 EV spec 一并落地）
- [ ] 撮合层独立单测（聚宽环境中补齐）

**Phase 1 第一步**：高级模型按 `prompts/01` 设计 `P1-F1-EV-2026Q2-v1` spec，
回应 `thinking-prompt.md` 对 EV<0 的全部深层追问（壳价值污染、受限资金、A+H 跨
市场），预注册 Gate 1 通过标准，落盘到 `research/specs/`。

---

## 六、与原文档冲突的调和状态

依据 `docs/plan-review.md` 第二节 C1–C13，所有冲突已在 `PROJECT-PLAN.md` §1
给出统一口径。本批次交付物均按统一口径实现，无遗留冲突：

| 冲突号 | 议题 | 调和结果 | 落地位置 |
|--------|------|---------|---------|
| C1/B9 | 换仓频率 | 季度调仓 | data_layer.DISCLOSURE_DEADLINES / prompts 00 |
| C2–C6/B7 | 成本模型 | 万2.5/千1/5元/0.3%/分级 | data_layer.apply_cost_model |
| C7/B10 | 基准与 TE/IR | 沪深300+中证800，强制 TE/IR | prompts 00 / 01 / 03 |
| C8/B3 | 退市股 | 必须纳入 | data_layer.get_delisted_calendar |
| C9/B4 | 涨跌停/T+1/停牌 | 限价单排队+先卖后买 | data_layer.simulate_limit_order / rebalance_ordered |
| C10/B8 | PIT 数据 | 强校验 | data_layer.is_data_available_at / prompts 03 门禁 A |
| C11 | 中性化 | 行业+市值双报 | prompts 01 spec 字段 |
| C12/B2 | 二值因子 | 等权求和不做 Z-score | prompts 00 / 02 / 03 |
| B1 | debt_to_assets | 先算比率再取中位数 | prompts 00 口径表 |
| B5 | ST 判定 | get_extras | data_layer.get_st_stocks |
| B6 | datetime import | 已补 | data_layer |
| C13 | 基调 | 乐观数字降级为待验证假设 | prompts 00 / _index.md 待验证假设登记 |
