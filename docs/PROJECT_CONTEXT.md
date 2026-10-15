# 项目背景与长期目标

> 维护说明：本文件记录相对稳定的研究背景与技术路线；频繁变动的进度放 docs/STATUS.md。
> 状态：初稿（2026-10-15）。内容基于官方仓库 README 与论文公开信息整理，尚未本地验证。

## 1. 论文信息

- 标题：Patch-Grid: An Efficient and Feature-Preserving Neural Implicit Surface Representation
- 作者：Lin, Guying; Yang, Lei; Zhang, Congyi; Pan, Hao; Ping, Yuhan; Wei, Guodong; Komura, Taku; Keyser, John; Wang, Wenping
- 发表：ACM Transactions on Graphics (TOG), Vol. 44, No. 2, Article 16, 2025 年 4 月
- DOI：https://doi.org/10.1145/3727142
- arXiv：https://arxiv.org/abs/2308.13934
- 官方代码：https://github.com/Carrie-lin/Patch-Grid
- 官方数据与结果：官方 README 提供的 OneDrive 链接（含 data/train、shape_space/checkpoints、data/edit、update_results 等）

## 2. 方法概要（据官方 README 与摘要，待精读论文后修订）

- 统一的神经隐式表面表示 Patch-Grid：将每个表面 patch 嵌入局部 latent volume，由共享 MLP 解码器解码；解码器在多种局部表面几何上预训练。
- 固定预训练解码器后，新形状拟合约 8 秒内完成，局部形状更新约 1 秒内完成。
- 采用 merge grid 在自适应 Octree 单元内对表面 patch 做局部 CSG 组合，保留锐利特征，比全局 CSG 构造更稳健。
- 适用于开放边界、薄结构等复杂几何；指标包括 Chamfer 距离、Hausdorff 距离、F-score、IoU、法向一致性，以及 field error 与锐利特征误差。

## 3. 长期研究目标

利用 LLM Agent 根据文字描述或 CAD 图片生成初始模型，再进行：

1. Patch 分割：将初始模型拆分为表面 patch；
2. 隐式表示拟合：每个 patch 用局部隐式表示（latent volume + 共享解码器）拟合；
3. 组合与编辑：基于 merge grid / 局部 CSG 组合，支持局部快速更新。

最终目标是探索一条可行的 CAD 生成技术路线。

## 4. 阶段划分与技术路线

| 阶段 | 内容 | 状态 |
|---|---|---|
| Phase 0 | 项目初始化、文档体系、仓库规范 | 已完成（2026-10-15） |
| Phase 1 | 复现官方实现：环境、数据、训练、抽取、指标、形状更新 | 未开始 |
| Phase 2 | 理解与消融：数据管线、超参敏感性、失败案例分析 | 未开始 |
| Phase 3 | LLM 初始模型生成探索（文字 / CAD 图片 → 初始几何） | 未开始 |
| Phase 4 | 分割—拟合—组合管线探索与评估 | 未开始 |

## 5. 相关资源

- NGLOD（官方 metrics 所用 CUDA 点面距离实现）：https://github.com/nv-tlabs/nglod
- 官方测试环境：Ubuntu 20.04、Python 3.9.7、CUDA 11.7（本机适配情况见 docs/ENVIRONMENT.md）
- 本机硬件约束：RTX 4050 Laptop，6 GB 显存（见 docs/ENVIRONMENT.md）
