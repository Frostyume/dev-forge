---
type: resume-project
lang: zh
updated: 2026-01-11
source_problems: [P-20260105-01, P-20260106-01]
---
# aurora-trainer — 高吞吐深度学习训练框架
**技术栈**：Python · PyTorch · CUDA/AMP · GitHub Actions
**角色 / 周期**：核心贡献者 · 2026.01–至今
## 项目简介
负责训练稳定性与可复现性，定位并系统性解决了混合精度训练崩溃与跨环境结果漂移两类问题，保障训练提速的同时恢复了回归测试可信度。
## 关键贡献（STAR）
- **根除混合精度训练崩溃**：开启 AMP 后训练在第 3 epoch loss 变 NaN，定位到 attention 在 fp16 下 softmax 前未做数值缩放导致前向溢出（GradScaler 无法拦截），通过 softmax 前减 row-wise max 并对敏感算子局部 fp32，使 AMP 训练稳定并保留约 1.8x 加速。
- **修复跨环境不可复现**：本地与 CI 同 seed 指标差 3 个点致回归测试失效，定位到未锁定 cudnn 确定性与依赖版本，通过开启 deterministic、固定 CI 依赖、收敛 seed 管理工具，恢复回归测试可信度。
## 亮点速记（可直接粘进简历的单行 bullet）
- 定位并修复 fp16 前向溢出导致的训练崩溃，在保留约 1.8x AMP 加速的同时消除 NaN。
- 通过锁定 cudnn 确定性与依赖版本，根除跨环境结果漂移，恢复 CI 回归测试可信度。
