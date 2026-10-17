# 周报 · 2026 年第 42 周（10-12 ~ 10-18）

> 项目：PatchGrid 复现与 CAD 生成研究（详见 docs/PROJECT_CONTEXT.md）
> 撰写：2026-10-17（本周实际工作日：10-15、10-17）
> 状态：Phase 1 官方复现已完成；自本周起暂停本机复现类任务（硬件限制，见「风险与限制」）

## 一、本周概述

本周完成 PatchGrid 项目的全部基建与官方论文复现：从零建立跨工具（Kilo Code/Codex）的 Agent 工作规范与科研记录体系，在 RTX 4050 Laptop（6GB 显存 / 15GB 内存 / Ubuntu 24.04）上完成 Patch-Grid 官方实现的**全部核心流程复现**——3 个官方测试形状拟合、从头训练（TS）路径、field 指标、shape updating 局部更新，并将复现指标与论文 Table 1 对照，**判定复现与论文一致**。过程中经历 4 次系统内存耗尽（OOM）崩溃，已全部定位根因、记录并沉淀为本机资源约束清单。

## 二、主要进展

### 1. 项目基建（Phase 0，10-15）

- 建立全局 Agent 规范（~/.config/kilo/AGENTS.md，7 节 23 条）与项目级规范（AGENTS.md），Kilo/Codex 双工具共用。
- 建立 docs/ 科研记录体系：PROJECT_CONTEXT / STATUS / DECISIONS / ENVIRONMENT / REPRODUCTION / DEVIATIONS / logs；GitHub 私有仓库同步（LyhIvyLee/Patch-Grid），跨对话、跨模型可无缝衔接上下文。

### 2. 环境搭建与兼容性攻坚（Phase 1 S1/S2，10-15）

- 官方代码纳入版本管理（patch-grid/，基线 commit cdd97d9），完成全流程静态分析（数据格式、训练循环、提取与评估管线、官方代码 4 处既存缺陷定位）。
- 在 Ubuntu 24.04 上复原官方 2021 年环境栈：Python 3.9 + PyTorch 1.10.1+cu111，实测 cu111 构建（最高 sm_86）在 RTX 4050（sm_89）上经驱动兼容正常运行。
- 编译 mesh2sdf CUDA 扩展（训练导入链硬依赖，README 未完整说明）：conda 内 nvcc 11.8 + gcc 11 + TORCH_CUDA_ARCH_LIST=8.6 配方成功，kernel 数值验证正确。

### 3. 官方复现核心实验（Phase 1 S3/S4，10-15 ~ 10-17）

| 实验 | 结果 |
|---|---|
| 3 个官方形状 ss 路径（固定预训练 decoder） | 全部成功，训练 13.4s/形状 |
| Patch-Grid-TS 从头训练（500 epochs）+ 提取 + 指标 | 成功 |
| field_metrics（修补官方 import 缺陷后） | 成功，field error 0.00294 |
| shape updating（243_ori → 243_round 局部更新 80 epochs → 349MB 重建） | 流程成功，指标计算受本机内存限制终止（见四） |

### 4. 论文数值对照（10-17）

| 指标 | 论文（100 形状均值） | 本机（3 形状均值） |
|---|---|---|
| CD（×10⁻⁴） | 1.00 | 0.75 |
| IoU / NC / F-score | 99.61 / 99.83 / 99.78 | 99.66 / 99.76 / ≈100 |
| sharp feature error（×10⁻⁵） | 9.89 | 9.02（单形状） |

结论：全部指标同量级且接近，差异方向符合「3 形状样本 vs 100 形状均值」「笔记本 GPU vs 4090」的预期。**复现质量与论文一致。**

## 三、问题与解决（按类别）

1. **老环境兼容（已全部解决）**：torch 1.10 不识别 sm_89 → 用 8.6 编译（二进制兼容实测通过）；CUDA 扩展缺 cusparse/cusolver 头 → conda 内装完整 cuda-toolkit 11.8；setuptools 80 与 distutils 不兼容 → 按官方钉子降 58.2.0；protobuf/tensorboard 版本冲突 → 钉 3.19.0/2.7.0；igl 无 cp39 wheel → 分析确认仅死代码引用，不装。
2. **系统内存 OOM（4 次，根因相同，已沉淀约束）**：15GB 总内存，Chrome+VSCode 常驻 ~9GB，重型 Python 网格处理（MC 分块拼接、rtree 索引）为纯 CPU 实现，余量不足。缓解：提取分辨率降 2^10、批次 5 万（成功）；update 指标对 833 万面网格三轮适配（简化 50 万面/降采样/持久化进程）仍失败，按纪律终止并如实记录。
3. **官方代码缺陷（定位 4 处，仅修 1 处）**：field_metrics.py 的 import 错误（必然 ImportError，已最小修补并登记 DEVIATIONS #1，全库唯一代码修改）；example.sh 参数拼写、np.int 弃用、final-save 条件恒假（均规避，不改）。
4. **工具层**：后台批量命令的 shell 变量被传递层吞掉导致静默失败 → 改显式命令；会话宿主被 OOM 连坐 → 持久化进程缓解。

## 四、数据与结论

- 全部实验数据与命令原文见 docs/logs/2026-10-15.md、2026-10-17.md；指标文件在 patch-grid/_results/metrics/（不入库）。
- 已验证结论：复现指标与论文一致（见二.4 表）；本机可稳定支撑训练与提取（资源适配参数已固化到 ENVIRONMENT.md）。
- 未能完成：shape updating 的官方口径 mesh 指标（本机内存不足，补齐命令已备好于 REPRODUCTION.md，需关闭大内存应用或换大内存机器）。
- 遗留待归因：243 更新重建与 GT 包围盒偏差 0.069（train 形状 ≤0.001，疑与 scale_trans.json 后处理相关，记于 STATUS.md 研究假设）。

## 五、风险与限制

- **本机硬件是当前最大瓶颈**：6GB 显存充裕但 15GB 内存不足（复现类重计算已按用户决定暂停）；后续 Phase 2 消融与长期目标的 LLM 生成管线评估需更大内存（建议 ≥32GB）或云端资源。
- 官方预处理代码未发布（todolist.txt 证实），自造数据不可行（纪律禁止），长期目标所需的「初始模型 → patch 分割」管线必须逆向或重写（Phase 2 立项）。

## 六、下周计划

1. 不跑新的复现实验（用户决定）；如条件允许，在关闭大内存应用后补齐 update 指标（单条命令，非必须）。
2. Phase 2 立项准备（纯文档/阅读工作）：精读论文方法节与预处理管线逆向分析（基于已下载官方数据的结构反推 patches/redboxes 生成规则）。
3. 归因 243 更新重建的 0.069 包围盒偏差（scale_trans.json 后处理路径分析，不跑训练）。
4. 长期目标（LLM 文字/CAD 图 → 初始模型 → patch 分割 → 拟合组合）技术调研与路线立项。
