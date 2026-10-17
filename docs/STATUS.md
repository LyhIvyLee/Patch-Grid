# 项目状态

> 每次任务结束时更新本文件。最近更新：2026-10-17

## 当前阶段

Phase 1 复现：**核心流程全部完成**——3 个官方形状 ss 路径、TS 从头训练路径、field_metrics、shape updating 均在本机跑通，且复现指标与论文 Table 1 对照一致（见 docs/reports/weekly-2026-W42.md）。
**自 2026-10-17 起暂停本机复现/训练类任务（用户决定：本机 15GB 内存不足，4 次 OOM 事故记录见 ENVIRONMENT.md），后续以文档/阅读/分析类工作为主。**

## 已完成

- 2026-10-15：S1–S4（见 docs/logs/2026-10-15.md），00000006_0 ss 路径首次复现成功。
- 2026-10-17：00000008_0 / 00000089_1 ss 路径全流程；TS 路径全流程；field_metrics（官方 import 缺陷修补，DEVIATIONS #1）；shape updating 三步流程（243_ori → 243_round 更新 → 349MB 重建）；论文数值对照（Table 1 一致，Table 3 因本机内存终止并如实记录）；周报体系建立（docs/reports/）。

## 已验证的结论

- **三形状 ss 路径指标**（cd 均值 ~7.5e-5 / iou 0.993–0.999 / f_score ≈100，见 logs/2026-10-17 表格）与 **TS 路径**（cd 6.09e-05）复现稳定。
- field metrics：field error 0.002938 / sharp feature error 9.022e-05（00000006_0）；sol_nglod 扩展确实不需要。
- shape updating 全流程可跑通；243 更新重建与 GT 包围盒偏差 0.069（train 形状 ≤0.001），原因待确认（疑与 scale_trans.json 后处理相关）。
- 本机资源约束（已固化到 ENVIRONMENT.md）：提取分辨率 ≤2^10、批次 5 万；系统 RAM 先于 GPU 成为瓶颈。
- 官方代码累计仅 1 处修改（field_metrics import 修补，DEVIATIONS #1）。

## 进行中

- （无）

## 研究假设

- 243_round 更新重建的 0.069 包围盒偏差可能来自 scale_trans.json 后处理缺省——Phase 2 验证。

## 阻塞问题

- （无）

## 下一步计划

（约束：不跑新的复现/训练实验，仅文档、阅读与分析类工作）

1. Phase 2 立项准备：精读论文方法节；基于已下载官方数据逆向分析预处理管线（patches/redboxes/bounds 的生成规则），为长期目标的自建管线打基础。
2. 归因 243 更新重建的 0.069 包围盒偏差（分析 scale_trans.json 与提取后处理代码路径，不跑训练）。
3. 长期目标（LLM 文字/CAD 图 → 初始模型 → patch 分割 → 拟合组合）技术调研与路线立项，写入 docs/PROJECT_CONTEXT.md 与 DECISIONS.md。
4. 可选（需用户配合）：关闭 Chrome 等大内存应用后补齐 update 指标（单条命令，见 REPRODUCTION.md）。
