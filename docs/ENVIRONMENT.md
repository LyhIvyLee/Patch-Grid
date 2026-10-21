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

## 本项目实际环境（2026-10-15 S2 完成，全部已验证）

### 核心组件

| 组件 | 版本 | 验证方式 |
|---|---|---|
| conda env | `patchgrid`（Python 3.9.23） | conda create |
| torch | 1.10.1+cu111 | `torch.cuda.is_available()=True`；4096² matmul + autograd 实测通过（RTX 4050 sm_89 运行 sm_86 构建，驱动 JIT/兼容） |
| torchvision / torchaudio | 0.11.2+cu111 / 0.10.1+cu111 | pip 安装成功 |
| mesh2sdf 扩展 | 0.1.0（官方源码本地编译，TORCH_CUDA_ARCH_LIST=8.6） | 单位球 SDF 数值正确（球心 -0.4977、0.7 处 +0.2） |

> **服务器迁移提醒**：`TORCH_CUDA_ARCH_LIST=8.6` 是针对本机 RTX 4050（sm_89，兼容运行 sm_86 cubin）的选择。在服务器上重编译前，先用 `python -c "import torch; print(torch.cuda.get_device_capability(0))"` 查目标 GPU 架构并取不超过 torch 1.10 支持上限（8.6）的对应值：V100=7.0、A100=8.0、30 系=8.6；40/50 系（sm_89+）沿用 8.6（二进制兼容）。
| nvcc / CUDA toolkit | 11.8（conda `nvidia/label/cuda-11.8.0`，装在 env 内，不动系统） | 编译扩展成功 |
| gcc / g++ | 11（conda-forge，env 内） | 编译扩展成功 |

### 关键钉子包（避免升版：与官方 yaml 一致或有兼容依据）

numpy==1.22.4、scipy==1.7.1、matplotlib==3.4.3、opencv-python==4.5.4.58、scikit-image==0.18.3（`marching_cubes_lewiner` 依赖）、trimesh==3.9.34、open3d==0.18.0、plyfile==0.7.4、PyYAML==6.0、protobuf==3.19.0（tensorboard 2.7 必需）、tensorboard==2.7.0、tqdm、ninja

未安装（主链路不需要，见 REPRODUCTION.md 分析）：igl/pysdf（无 cp39 linux wheel，且仅预处理死代码引用）、polysdf→polyscope、sk-video、h5py、networkx、pandas、dash 系

### 使用方式（训练/提取前设置）

```bash
conda activate patchgrid
export CUDA_HOME=$CONDA_PREFIX        # 编译扩展时需要；纯运行非必需
```

### 踩坑记录（详见当日 logs）

1. torchvision 依赖解析会把 numpy 拉到 2.0.2 → 必须显式钉 numpy==1.22.4
2. torch 1.10 cpp_extension 架构表最高 8.6：`TORCH_CUDA_ARCH_LIST=8.9` 报 Unknown arch；用 8.6（sm_86 cubin 兼容运行于 sm_89）
3. 编译扩展需完整 cuda-toolkit（CUDAContext.h 引用 cusparse/cusolver 头，单装 cuda-nvcc 不够）
4. tensorboard 2.7.0 必须配 protobuf==3.19.0（新版 protobuf 报 Descriptors 错误）
5. **系统 RAM 是比 GPU 更先触及的约束（三次事故记录，均经 journalctl `oom-kill` 确认）**：
   - 10-15 17:31：extract 默认 2^11 分辨率，CPU 拼接载入 3127 个分块（734MB 文本，Python 膨胀 5–10×）→ 15GB 耗尽，VSCode 被杀。
   - 10-17 14:22：mesh_metrics 对 833 万面重建网格建 rtree 索引（数 GB）+ 系统常驻 → VSCode 被杀。
   - 10-17 14:29：VSCode 重启恢复期内存峰值 + metrics 载入 50 万面网格 → 再次被杀（余量被连续大进程叠加耗尽）。
   - **根因**：15GB 总内存，Chrome+VSCode 常驻 ~9GB，重型 Python 任务真实余量 ~5GB；官方实现的网格载入/邻近查询是纯 CPU 实现（不受益于 GPU）。
   - **本机约束与缓解**：提取分辨率 ≤2^10、批次 5 万；超 50 万面网格先 simplify_quadric_decimation 再跑指标（简化误差 ≪ 指标阈值）；指标采样点可降至 15 万；重型 metrics 用持久化后台进程跑（避免宿主崩溃连坐）；跑重任务时避免同时开新的大型应用。
