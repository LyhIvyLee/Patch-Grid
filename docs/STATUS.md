# 项目状态

> 每次任务结束时更新本文件。最近更新：2026-10-15

## 当前阶段

Phase 1 复现：**首要目标已达成**——00000006_0 官方测试模型在本机完成拟合并生成重建 Mesh，官方 mesh 指标质量优秀。

## 已完成

- 2026-10-15 S1：官方代码纳入 patch-grid/（基线 cdd97d9）；全流程静态分析；兼容性风险清单。
- 2026-10-15 S2：conda env `patchgrid` 搭建完成；torch 1.10.1+cu111 GPU 验证；mesh2sdf 扩展编译并数值验证；主链路 import 全通过。
- 2026-10-15 S3：数据就位（用户手动下载）→ 冒烟测试通过 → 正式拟合（301 epochs / 13.4s）→ 提取重建 OBJ（142MB / 347 万面 / 包围盒对齐 ≤0.001）→ GT 对比渲染图 → mesh_metrics 全指标产出。
- 2026-10-15 S4：文档更新 + 提交推送。

## 已验证的结论

- **00000006_0 拟合指标**（`patch-grid/_results/metrics_00000006_0_ss.txt`）：CD 7.20e-05、HD 8.50e-04、IoU 0.9990、NC 0.9992、F-score 100.0（τ=0.001）。数量级合理（与论文中 Patch-Grid 报告的水平一致；精确表格对照待精读论文后补充到 REPRODUCTION.md）。
- 训练默认配置在 6GB GPU 上余量充足（显存 ~2.7GB 峰值在提取阶段；训练阶段 <1GB）。
- 提取阶段两次资源适配均为官方参数旋钮、零代码修改：`--extract_batch_size 50000`（GPU OOM）与 `--extract_resolution 1024`（系统 RAM OOM，15GB 全局内存限制，2048 时拼接阶段 734MB 分块载入膨胀导致 VSCode 被 OOM killer 杀死——即 17:31 会话闪退原因）。
- RTX 4050（sm_89）可运行 cu111 构建；mesh2sdf sm_86 编译配方有效。
- OneDrive 官方数据无法编程下载（已由用户手动完成）。

## 进行中

- （无）

## 研究假设

- （暂无）

## 阻塞问题

- （无）

## 下一步计划

1. 其余两个官方形状（00000008_0、00000089_1）同流程拟合+提取+指标，验证稳定性。
2. Patch-Grid-TS 从头训练路径（--epochs 500）至少跑通一次。
3. field_metrics：需先修补官方 `from option import parse_options` 缺陷（登记 DEVIATIONS）并评估是否编译 sol_nglod。
4. 精读论文补充数值对照表（当前仅数量级合理性判断）。
5. shape updating 流程（需下载 data/edit 数据）。
