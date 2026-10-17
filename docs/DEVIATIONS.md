# 与官方实现的差异记录

> 对 patch-grid/ 官方代码的每一处修改都必须登记一行；宁可多记，不可漏记。新条目追加在表格末尾。

## 官方代码基线

- 来源：https://github.com/Carrie-lin/Patch-Grid
- 克隆日期：2026-10-15（以 tarball 方式获取，等效于克隆后移除 .git）
- commit 哈希：cdd97d9ecf063ff58087f1ba1da38f21e4b14935（main 分支 HEAD）

## 差异清单

| # | 日期 | 文件 | 修改内容 | 原因 | 验证方式 |
|---|------|------|----------|------|----------|
| 1 | 2026-10-17 | metrics/field_metrics.py:9 | `from option import parse_options` → `from configs.option import parse_options` | 官方既存缺陷：仓库无 metrics/option.py，原 import 必然 ImportError；extract.py 对同一函数使用的就是 configs.option 路径 | field_metrics 实际运行通过（见 logs/2026-10-17） |

## 说明

- 环境层面的偏离（如更换 PyTorch 版本）记录到 docs/DECISIONS.md；本文件只记代码差异。
- 每次实验日志（docs/logs/）中如涉及修改过的代码，应引用本表编号。
