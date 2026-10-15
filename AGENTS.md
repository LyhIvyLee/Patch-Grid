# PatchGrid 科研项目 — Agent 工作规范

本文件是跨工具（Kilo Code、Codex 等）共用的项目级指令，保持简洁稳定；详细的科研内容放在 docs/ 中，随研究推进单独维护。

## 项目身份（详见 docs/PROJECT_CONTEXT.md）

- 论文：Patch-Grid: An Efficient and Feature-Preserving Neural Implicit Surface Representation（ACM TOG 2025, Vol. 44, No. 2, Article 16）
- DOI：https://doi.org/10.1145/3727142（arXiv: 2308.13934）
- 官方仓库：https://github.com/Carrie-lin/Patch-Grid
- 当前阶段：复现官方实现
- 长期目标：利用 LLM Agent 根据文字或 CAD 图片生成初始模型，再进行 Patch 分割、隐式表示拟合与组合，探索 CAD 生成方法

## 仓库结构

| 路径 | 用途 |
|---|---|
| AGENTS.md | 项目级 Agent 规则（本文件） |
| docs/ | 科研记录：背景、状态、决策、环境、复现、差异、日志 |
| docs/logs/ | 按日期维护的科研日志（YYYY-MM-DD.md） |
| patch-grid/ | 官方代码副本（Phase 1 克隆，纳入版本管理） |
| data/ | 训练/测试数据（不入库） |
| _results/ | 实验输出与权重（不入库） |

## 任务开始时

1. 先读 docs/STATUS.md 了解当前进度、阻塞与下一步，再按需读取与当前任务相关的文档（docs/PROJECT_CONTEXT.md、docs/REPRODUCTION.md、docs/DECISIONS.md、docs/ENVIRONMENT.md、docs/DEVIATIONS.md）。
2. 确认任务属于当前阶段范围（当前为复现）；超出范围的研究想法记入 docs/STATUS.md 的「研究假设」，不直接实施。

## 代码修改原则

3. 初期优先保证官方实现可以复现，不提前添加未验证的新算法、新依赖或重构。
4. 尽量保留官方代码结构与命名；兼容性修改（如 CUDA/PyTorch 版本适配）应尽可能小。
5. 对 patch-grid/ 的每一处修改都必须说明原因，并登记到 docs/DEVIATIONS.md（位置、内容、原因、验证方式）。
6. 不得用虚构的实验数据、自造权重或来源不明的资源代替官方数据与官方预训练权重；官方资源一律从官方渠道获取。

## 实验纪律

7. 启动训练或评估前，先评估 GPU 显存（本机 RTX 4050 Laptop，6 GB，见 docs/ENVIRONMENT.md）与内存需求；先用小规模配置试运行并监控资源，确认无误后再放大规模。
8. 每次实验必须记录：输入数据、配置、依赖环境、模型参数、完整运行命令（保留原文）、输出位置、评估结果；写入 docs/logs/ 当日日志。
9. 严格区分四类内容：已完成的工作、已验证的结论、研究假设、下一步计划；docs/STATUS.md 按此语义维护，不得把未验证内容写成已验证。
10. 不编造实验结果；失败的实验同样如实记录（命令、报错、初步归因）。

## 任务结束时

11. 更新 docs/STATUS.md 的进度与下一步，并在 docs/logs/ 当日日志追加本次工作记录。
12. 环境或技术路线发生变化时，同步更新 docs/ENVIRONMENT.md 或 docs/DECISIONS.md。

## Git 纪律（项目级补充）

13. 提交聚焦当前任务；不混入无关文件、数据集、权重或大型实验输出（.gitignore 已排除）。
14. 提交信息格式：`<类型>: <摘要>`，类型取 docs / repro / env / fix / exp 之一。
15. 本仓库是个人研究仓库，永不向官方仓库推送；远程操作遵守全局规则第 14 条。

## 环境安全

16. 官方参考环境（未验证）：Ubuntu 20.04、Python 3.9.7、CUDA 11.7、PyTorch 1.10.1+cu111，conda 环境名 patchgrid；以 docs/ENVIRONMENT.md 记录的实际确认为准。
17. 不修改系统 Python；所有实验在项目专属 conda 环境中进行。
