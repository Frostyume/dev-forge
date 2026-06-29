---
type: weekly-report
period: 2026-W01
range: 2026-01-05 ~ 2026-01-11
level: standard
generated: 2026-01-11
projects: [aurora-trainer]
---
# 周报 2026-W01（01-05 ~ 01-11）
## 概览
本周新增 2 个问题，解决 2 个，仍 open 0 个，累计耗时 6h。

| 维度 | 分布 |
|---|---|
| 严重度 | critical 0 · high 1 · medium 1 · low 0 |
| 类别 | numerics:1 · env:1 |

## 本周解决
- **P-20260105-01 混合精度下 loss 变 NaN**（high，4h）根因：fp16 前向 softmax 未缩放致 inf→NaN → 方案：softmax 前减 row-max + 敏感算子局部 fp32，保留 1.8x 加速。
- **P-20260106-01 CI 与本地结果不一致**（medium，2h）根因：未锁 cudnn 确定性与版本 → 方案：开 deterministic、固定 CI 依赖、收敛 seed 工具。
## 仍未结案（carryover）
- 无
## 关键经验
- fp16 风险在前向溢出，exp/softmax 前必做数值缩放。
- 可复现要锁 cudnn 确定性与依赖版本，而不仅是 seed。
