# 项目状态

> 每次任务结束时更新本文件。最近更新：2026-10-15

## 当前阶段

Phase 1 复现：S1（代码与环境检查）与 S2（环境搭建）已完成，S3（数据与最小实验）阻塞于官方数据下载。

## 已完成

- 2026-10-15 S1：官方代码纳入 patch-grid/（基线 cdd97d9）；数据格式/训练循环/提取/评估流程静态分析（见 REPRODUCTION.md）；兼容性风险清单。
- 2026-10-15 S2：conda env `patchgrid` 搭建完成；torch 1.10.1+cu111 GPU 验证通过；mesh2sdf CUDA 扩展本地编译并数值验证；官方主链路 import 全通过。
- 2026-10-15 本地 Git 提交 4 个（docs×2 / repro×1 / env×1），未推送（无远程）。

## 已验证的结论

- RTX 4050（sm_89）可运行 cu111 构建：torch matmul/autograd 与 mesh2sdf kernel（sm_86 cubin）均实测正确。
- mesh2sdf 扩展编译配方：conda cuda-toolkit 11.8 + gcc 11 + TORCH_CUDA_ARCH_LIST=8.6（torch 1.10 架构表不识别 8.9）。
- 官方 import 链（utils/trainer/dataset/torchgp）与 extract 依赖（open3d/skimage lewiner MC）在本环境全部通过。
- OneDrive 官方数据无法编程下载（匿名 API unauthenticated，实测）。
- 本机硬件基线见 docs/ENVIRONMENT.md（6GB 显存为训练规模硬约束）。

## 进行中

- 等待用户从官方 OneDrive 手动下载数据（见「阻塞问题」）。

## 研究假设

- （暂无）

## 阻塞问题

- 【阻塞 S3】官方 OneDrive 数据需浏览器手动下载：`data/train`（3 形状）→ 放 `patch-grid/data/train/`；`shape_space/checkpoints`（decoder_1.pth 等）→ 放 `patch-grid/_results/shape_space/`。下载链接见官方 README。

## 下一步计划

1. 用户下载数据并放置到位（唯一阻塞）。
2. 冒烟测试（命令见 REPRODUCTION.md 4a）：--epochs 2 验证采样/前向/反向/存盘 + 显存监控。
3. 显存与耗时允许 → 正式拟合 00000006_0（默认 300 epochs）→ extract → `_results/mesh/00000006_0_ss_composed_all.obj`。
4. 可行则跑 mesh_metrics（field_metrics 需先修补官方 import 缺陷并编译 sol_nglod，登记 DEVIATIONS 后再做）。
