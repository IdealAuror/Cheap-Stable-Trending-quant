# research/reports/

存放每轮 spec 的执行产出报告，由 Flash 与高级模型共同填充：

| 文件后缀 | 产出者 | 内容 |
|---------|--------|------|
| `-deviation.md` | Flash | 执行偏差记录（歧义处理、数据缺失、执行异常） |
| `-code-review.md` | 高级模型 | 代码审核结论（看结果前完成） |
| `-adjudication.md` | 高级模型 | 结果判定与诊断 |

## 命名规范

见 `research/_index.md` 第一节。文件名：`<spec_id>-<后缀>.md`。

## 顺序约束

`-code-review.md` 必须先于 `-adjudication.md` 产出（见 `docs/prompts/03` 门禁）。
