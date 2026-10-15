# 技术决策记录

> 格式：编号 / 日期 / 决策 / 理由 / 状态。凡影响仓库结构、技术路线或与官方实现偏离的决策都在此登记；对官方代码的逐处修改记录在 docs/DEVIATIONS.md。

## D-0001 仓库组织：研究工作区 + 官方代码副本（2026-10-15）

- 决策：本仓库作为研究工作区，官方代码克隆到 patch-grid/ 子目录并纳入本仓库版本管理，克隆时记录官方 commit 哈希作为基线。
- 理由：对官方代码的全部兼容性修改可由本仓库 git 历史追溯（相对基线 commit 的 diff 即「与原版的差异」）；docs/ 与代码同库，便于 GitHub 同步与 Kilo / Codex 双工具使用。
- 备选方案：fork 官方仓库再叠加 docs（缺点：个人科研记录与官方历史耦合，提交混杂，且与「永不向官方仓库推送」的纪律冲突）。
- 状态：已采纳（克隆动作在 Phase 1 执行）。

## D-0002 数据与大型产物不入库（2026-10-15）

- 决策：data/、_results/、*.pth 权重、日志等通过 .gitignore 排除。
- 理由：数据集与权重体积大，GitHub 不适合承载；需要同步的核心状态是科研文档与代码。
- 状态：已采纳。

## D-0003 docs/ 作为跨工具共享状态载体（2026-10-15）

- 决策：项目进度、决策、环境、复现步骤与日志全部放在 docs/；AGENTS.md 只放稳定的规则，不放易变内容。
- 理由：Kilo Code 与 Codex 都会读取仓库根目录的 AGENTS.md（开放标准）；docs/ 与工具无关，人与 Agent 均可读写，通过 GitHub 同步后在任一工具中都可恢复上下文。
- 状态：已采纳。

## D-0004 全局规则按工具各自维护（2026-10-15）

- 决策：Kilo 全局规则位于 ~/.config/kilo/AGENTS.md；Codex 全局规则保留在 ~/.codex/AGENTS.md（既有 4 条，未改动）。两者语义兼容但不强制逐字同步；项目级规则统一由仓库根目录 AGENTS.md 承担。
- 理由：两个工具的全局指令加载路径不同；跨项目通用偏好各有侧重，项目内行为靠项目 AGENTS.md 保证一致。
- 状态：已采纳。

## D-0005 环境搭建方式：手动最小集安装，不直接用官方 patchgrid.yaml（2026-10-15）

- 决策：`conda create -n patchgrid python=3.9` 后用 pip 安装实际代码分析所得的最小依赖集 + 官方 README 指定的三个 cu111 wheel（torch 1.10.1 / torchvision 0.11.2 / torchaudio 0.10.1）。
- 理由：官方 yaml 混用 `conda-forge/label/cf201901`、`defaults` 等旧 channel 与大量带 build string 的老包（如 openssl=3.0.0、sqlite=3.36），在现代 conda 25.5 上解析大概率失败；且其中 dash/flask/jupyter 等为可视化杂项，不属于训练主链路。按代码真实 import（torch/numpy/scipy/matplotlib/cv2/igl/open3d/skimage/trimesh/pysdf/yaml/tqdm 等）安装更可控。
- 影响：属于环境层偏离（不修改官方代码），版本清单记录于 docs/ENVIRONMENT.md；若后续 import 报缺包再增量补装。
- 状态：已采纳。
