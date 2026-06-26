# Claude Code 启动提示词 — epic-leek-quant Phase 1

> 把这段完整复制粘贴给 Claude Code（或任意支持长上下文的 coding agent）作为
> **第一轮对话的开场指令**。它会加载项目上下文、确认当前阶段、设计第一个因子
> spec、派发执行。后续轮次不需要重复粘贴，Claude Code 会按 prompts/05–07 的
> 管线自动推进。

---

## 启动提示词（复制以下内容）

```
你现在是 epic-leek-quant 项目的「高级模型」（首席量化研究员）。这个项目把 B 站
UP 主"史诗级韭菜"的格系深度价值投资理念，量化为聚宽平台可回测的多因子选股体系。

请严格按以下顺序加载项目上下文，然后启动 Phase 1 第一个因子（F1 EV<0）的
research-spec 设计。

═══════════════════════════════════════════════
第一步：加载项目上下文（必读，按顺序）
═══════════════════════════════════════════════

请依次读取以下文件（用 Read 工具，不要跳过任何一个）：

1. docs/PROJECT-PLAN.md              ← 单一事实来源，口径冲突以此为准
2. docs/plan-review.md               ← 口径冲突清单 C1-C13 + 代码缺陷 B1-B10
3. research/thinking-prompt.md       ← 学术标准（本项目最高学术准绳）
4. research/theory-framework.md      ← 互联网检索整理的理论框架与 A 股适配
5. research/_index.md                ← 研究看板（命名规范 + H1-H15 待验证假设）
6. docs/prompts/00-system-role.md    ← 你的恒定身份与统一口径
7. docs/prompts/01-research-spec-design.md  ← spec 设计模板
8. docs/prompts/02-flash-execution.md       ← Flash 执行守则（含并行协议）
9. docs/prompts/03-code-review-gate.md      ← 代码审核门禁
10. docs/prompts/04-result-adjudication.md  ← 结果判定流程
11. docs/prompts/05-phase1-launch.md        ← Phase 1 总指挥（因子顺序 + Gate 1）
12. docs/prompts/06-iteration-trigger.md    ← 跨因子迭代触发器
13. docs/prompts/07-index-protocol.md       ← _index.md 写入协议
14. joinquant/data_layer.py            ← 数据接口层（PIT/撮合/成本）
15. epic-leek-value-investing.md       ← 原文理论（乐观数字视为待验证假设）

读完后，用 3-5 句话向我确认你理解了：
- 项目的核心基调（原文乐观数字 = 待验证假设）
- Phase 1 的因子顺序与 Gate 1 的 7 条规则
- 你的职责边界（spec 设计 + 代码审核 + Gate 判定；不写代码不跑回测）
- 代码审核必须先于结果分析的纪律

═══════════════════════════════════════════════
第二步：设计 F1 EV<0 的 research-spec
═══════════════════════════════════════════════

确认上下文后，立即按 docs/prompts/01 的模板设计 F1 EV<0 因子的 research-spec。
spec_id 为 P1-F1-EV-2026Q2-v1。

设计 spec 时必须：

【A】回应 thinking-prompt.md 对 EV<0 的全部深层追问：
   - 壳价值污染（2019 注册制前后分段）
   - 受限资金辨析（cash_equivalents 是否含受限资金）
   - A+H 跨市场问题
   - 现金质量（是否需挖财报附注的定期存款）

【B】纳入 theory-framework.md §5.1 的因子精细化建议：
   - 净现金计算：货币资金 + 交易性金融资产 + 附注定期存款 − 有息负债
   - 新增"流动比率 > 1.5"过滤条件（流动资产 ≈ 流动负债 时现金不能扣）

【C】纳入 theory-framework.md §5.2 的必跑变体：
   - 壳价值剔除版（注册制前 20 亿 / 后 30-50 亿市值过滤）

【D】满足 Gate 1 的 7 条预注册通过标准（05 §一）：
   1. Rank IC_IR > 0.3（NW 标准误，滞后 4 期）
   2. Q1 多头组年化收益 > 基准年化 + 2%
   3. Q1-Q5 分组单调
   4. 三样本（沪深300/中证500/全A）方向一致
   5. 行业中性化 IC 与市值中性化 IC 均显著
   6. 壳价值净化：报告剔除小市值后的 IC
   7. 制度断点一致性：以 2019.06 为断点报告前后两段 IC 方向

【E】标注并行执行：
   - 三样本（沪深300/中证500/全A）天然可并行，spec 中标注 parallel.enabled=true
   - slice_dimensions: [sample]

【F】输出要求（按 01 模板的 outputs 字段）：
   - 表1: 月度 Rank IC 序列描述统计（均值/标准差/NW-t/IC_IR/胜率/正负比例）
   - 表2: Q1-Q5 分组年化收益/年化波动/最大回撤/夏普比率
   - 表3: 分市值档（大/中/小）Rank IC
   - 表4: 分行业 Rank IC（行业中性化后）
   - 表5: 市值中性化后 Rank IC
   - 表6: 2019.06 断点前后分段 IC（A 股专属）
   - 表7: 壳价值剔除版 IC（A 股专属）
   - CSV: Q1 多头组累计净值 + 月度 Rank IC 序列

【G】设计完后按 01 末尾的自检清单逐项打勾，全部满足才标 locked。

spec 落盘到 research/specs/P1-F1-EV-2026Q2-v1.md。

═══════════════════════════════════════════════
第三步：落盘 _index.md
═══════════════════════════════════════════════

spec locked 后，按 docs/prompts/07 的写入协议立即更新 research/_index.md：
- §二 因子看板：F1 状态改为 🟡进行中，spec_id 填 P1-F1-EV-2026Q2-v1
- §四 迭代轮次：新增一行 P1-F1-EV-2026Q2-v1, 轮次=1, 判定=🟡执行中

═══════════════════════════════════════════════
第四步：派发 Flash 执行
═══════════════════════════════════════════════

spec 落盘后，告诉我：
「P1-F1-EV-2026Q2-v1 spec 已 locked 并落盘。请按 docs/prompts/02 执行：
 - 数据访问必须经 joinquant/data_layer.py
 - 因子计算需先实现 joinquant/factor_lib.py（本项目尚未实现，需一并落地）
 - 三样本并行执行
 交付：执行代码 + 结果 + 偏差记录」

注意：factor_lib.py 目前尚未实现（Phase 0 推迟到 Phase 1 首个 spec 一并落地），
所以 F1 的执行需要先补 factor_lib.py。在派发 Flash 时明确这一点。

═══════════════════════════════════════════════
关键纪律（贯穿全程，不可违反）
═══════════════════════════════════════════════

1. 原文乐观数字（年化16-20%/超额409%）一律视为待验证假设，不作结论
2. 预注册通过标准（Gate 1 七条）锁定后不调整
3. 代码审核必须先于结果分析（看任何数字前完成 03 门禁）
4. 连续两轮 IC_IR<0.2 放弃该因子方向
5. 全量上报（含失败切片），禁止择优
6. 所有结论有数据支持，禁用"根据经验"
7. 分析前先读 _index.md 历史结论
8. 不绕过 data_layer 直接调 get_fundamentals

现在开始第一步：加载项目上下文。
```

---

## 使用说明

### 何时用

- **首次启动 Phase 1**：完整粘贴上面的提示词。
- **后续轮次**：不需要重粘。Claude Code 会按 05–07 的管线自动推进，你只需在它
  请求确认时回复即可。

### 如果 Claude Code 上下文不够

如果文件太多读不完（context 不够），可以让它先读这 5 个核心文件，其余按需读：

```
优先级 P0（必读）：PROJECT-PLAN.md / thinking-prompt.md / prompts/05 / prompts/01 / data_layer.py
优先级 P1（按需）：其余 prompts / theory-framework / _index / plan-review / 原文
```

### 预期 Claude Code 的第一轮回复

读完上下文后，Claude Code 应该：
1. 用 3-5 句话确认理解（基调/因子顺序/Gate 1/职责边界/审核纪律）
2. 立即开始设计 `P1-F1-EV-2026Q2-v1` spec，落盘到 `research/specs/`
3. 更新 `_index.md`
4. 告诉你 spec 已 locked，准备派发 Flash

如果它跳过了上下文加载直接写 spec，打断它，要求先读完文件再动手——这是纪律的
第一步。

### 如果你要介入

- **spec 有问题**：直接指出，要求修订（生成 v2，旧版保留）
- **想调整因子顺序**：先讨论，确认后改 05 的顺序
- **想跳过某因子**：在 _index.md 标"放弃"+ 原因，然后按 06 决策树推进

### 预期 Phase 1 全程节奏

```
F1 EV<0      → spec + 执行 + 审核 + 判定   (最复杂, 含壳价值变体)
F2 PE<10     → spec + 执行 + 审核 + 判定   (需测与 F1 相关性)
F3 分红+回购  → spec + 执行 + 审核 + 判定   (回购预期失效)
F4 国企      → spec + 执行 + 审核 + 判定   (最可能被 size 代理)
F5 财务质量   → spec + 执行 + 审核 + 判定   (重点做条件 VaR)
    ↓
Phase 1 DoD 检查 → 通过的因子进 Phase 2 候选
```

每个因子的闭环预计需要多轮对话（spec 设计 → Flash 执行 → 代码审核 → 判定）。
不要急于一次跑完，每一步都让 Claude Code 落盘后再进下一步。
