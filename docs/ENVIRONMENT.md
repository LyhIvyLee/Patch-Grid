# 运行环境记录

> 只记录实际确认过的信息。「官方参考」指官方仓库 README 声称的环境，本机是否可行需在 Phase 1 验证；验证结果更新到本文件。

## 本机已确认（2026-10-15 检测，命令：nvidia-smi / python3 --version / conda --version / df -h / cat /etc/os-release）

| 项目 | 实际值 |
|---|---|
| OS | Ubuntu 24.04.5 LTS |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU（6141 MiB 显存） |
| 驱动版本 | 580.178.04 |
| 系统 Python | 3.12.3（不用于本项目） |
| conda | 25.5.1 |
| 磁盘可用 | 约 416 GB（/ 分区） |

## 官方参考环境（未验证）

- Ubuntu 20.04、Python 3.9.7、CUDA 11.7
- PyTorch 1.10.1+cu111 / torchaudio 0.10.1 / torchvision 0.11.2（官方 README 指定 wheel）
- conda 环境名 patchgrid（由官方 patchgrid.yaml 创建）
- 指标依赖 NGLOD 的 CUDA 扩展（需编译）

## 本项目实际环境

- 项目 conda 环境：未创建（Phase 1 创建）。
- CUDA 编译工具链（用于 NGLOD 扩展）：未验证。
- 官方数据 / 预训练权重：未下载。

## 已识别风险（推断，待 Phase 1 验证）

1. 显存：6 GB 相对官方实验规模可能偏小，训练前先用小形状 / 低配置试运行并监控 `nvidia-smi`。
2. 兼容性：cu111 wheel（2021 年）与 Ubuntu 24.04 的 glibc / GCC 13 默认工具链可能不兼容，尤其影响 CUDA 扩展编译；如失败，考虑降低 GCC 版本或评估新版 PyTorch（偏离需登记 docs/DEVIATIONS.md 与 docs/DECISIONS.md）。
3. Python 3.9：conda 仍可创建，但需确认依赖解析可用。
