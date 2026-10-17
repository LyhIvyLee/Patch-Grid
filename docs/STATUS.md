# 项目状态

> 每次任务结束时更新本文件。最近更新：2026-10-17

## 当前阶段

Phase 1 复现：**核心流程全部完成**——3 个官方形状 ss 路径、TS 从头训练路径、field_metrics、shape updating 均在本机跑通且指标质量优秀。

## 已完成

- 2026-10-15：S1–S4（见 docs/logs/2026-10-15.md），00000006_0 ss 路径首次复现成功。
- 2026-10-17：00000008_0 / 00000089_1 ss 路径全流程；TS 路径全流程；field_metrics（官方 import 缺陷修补，DEVIATIONS #1）；shape updating 三步流程（243_ori → 243_round 更新 → 349MB 重建）；规范修订（日志时段词、RAM 事故约束固化）。

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

1. 精读论文，把复现指标与论文表格数值做精确对照（当前仅数量级合理性判断）。
2. Phase 2 理解与消融：数据管线（patches/redboxes 如何生成——预处理代码未发布，可逆向分析）、scale_trans 后处理、update 空间偏差归因。
3. 为长期目标（LLM 初始模型 → patch 分割 → 拟合组合）做技术调研与可行性立项（超出复现阶段，先立项再实施）。
