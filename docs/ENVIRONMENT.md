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

## S1 兼容性分析结论（2026-10-15，代码分析 + 本机检测；详见 docs/REPRODUCTION.md「代码流程分析」）

| 项 | 结论 | 依据 |
|---|---|---|
| glibc | 2.39，满足 cu111 wheel（manylinux2014，需 ≥2.17） | `ldd --version` |
| conda 环境 | `patchgrid`（Python 3.9.23，官方 3.9.7 的补丁版本差异，推断可接受） | `conda create -n patchgrid python=3.9` |
| GPU 架构 | RTX 4050 为 sm_89；cu111 官方构建最高 sm_86 + PTX。**假设**：驱动 580 可 JIT PTX 到 sm_89（首次运行有编译延迟），装好后实测 | CUDA 二进制兼容规则（推断） |
| mesh2sdf CUDA 扩展 | **训练/提取/评估均需可 import**（utils.py→torchgp→mesh2sdf 无条件导入；数值上仅 metrics 调用）；需 nvcc + 匹配 torch 的编译环境，是环境搭建最大风险点 | 代码分析（子代理报告 5.3） |
| sol_nglod 扩展 | 仅 field/基线代码可达，训练+提取+mesh_metrics 不依赖，可不编译 | 代码分析（子代理报告 5.3） |
| numpy 钉子 | 必须用 1.22.4（官方 yaml 值）：`np.int`（extract.py:217）与 `np.core.records`（utils.py:186）在 numpy≥1.24/2.0 会崩（前者仅 delete_info 分支） | 代码分析（子代理报告 8） |
| scikit-image 钉子 | 必须用 0.18.x：`marching_cubes_lewiner` 在新版已移除 | 代码分析（子代理报告 8） |
| 数据获取 | OneDrive 新式分享链接匿名 API 返回 unauthenticated，无法编程下载，需用户手动下载 data/train 与 shape_space/checkpoints | curl 实测 |

## 本项目实际环境（S2 搭建中，完成后更新为已验证版本清单）

- conda：patchgrid（Python 3.9.23）
- torch/torchvision/torchaudio：安装中（官方 README 指定 cu111 wheel 版本）
- 其余依赖：待 torch 完成后按官方 yaml 版本钉子安装（见 docs/DECISIONS.md D-0005）
