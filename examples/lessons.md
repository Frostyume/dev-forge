# dev-forge 避坑清单（pre-flight checklist）
由 `/dev-forge lessons` 从 `problems/` 中已解决问题自动蒸馏而成。开工或改动前，按 category 扫一遍对应分组，把过去踩过的坑挡在前面；每条附来源 id，可追溯回原始记录。
## numerics（数值稳定 · 写代码前）
- [ ] 任何 `exp` / `softmax` 前先做数值缩放（如减 row-wise max），不要假设输入幅度安全 — 来源 P-20260105-01
- [ ] 对数值敏感的小算子在 AMP 下局部 `autocast(enabled=False)` 强制 fp32；记住 fp16 风险主要在**前向溢出**而非梯度，GradScaler 挡不住前向的 inf — 来源 P-20260105-01
## env（环境 / 可复现 · 提交或上 CI 前）
- [ ] 「可复现」不止固定 python/numpy/torch 的 seed：还要锁 `cudnn.deterministic=True` / `benchmark=False`，并固定 cudnn 与依赖版本 — 来源 P-20260106-01
- [ ] 把所有可复现设置收进**一个**集中的工具函数，避免散落各处导致遗漏 — 来源 P-20260106-01
