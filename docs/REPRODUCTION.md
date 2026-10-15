# 官方论文复现步骤

> 状态：全部【未开始】。步骤与命令整理自官方仓库 README（2026-10-15 抓取），命令保留官方原文；实际执行时如有修改，把改动登记到 docs/DEVIATIONS.md 并记入当日日志。

## 验收标准（满足即认定复现成功）

1. 至少一个官方形状（如 00000006_0）完成 Patch-Grid（固定预训练解码器）训练与网格抽取，全流程无报错。
2. mesh_metrics 与 field_metrics 产出数值，且与论文报告值数量级一致（对照表待精读论文后补充，先不凭记忆填写）。
3. Patch-Grid-TS（从头训练）至少跑通一次。
4. 形状更新（shape updating）流程至少跑通一次。

## 执行计划（2026-10-15 制定，分阶段推进）

| 阶段 | 内容 | 完成标准 |
|---|---|---|
| S1 代码与环境检查 | 获取官方代码副本；分析数据读取/网络/训练/提取/评估流程；本机环境与兼容性分析；确定最小可行方案 | 分析结论写入本文件与 docs/ENVIRONMENT.md |
| S2 配置运行环境 | 独立 conda 环境（不碰系统 Python/驱动）；按依赖关系装包；验证 torch GPU 运算与模块导入 | GPU 张量运算测试通过 |
| S3 数据与最小实验 | 获取官方数据 + 预训练 Decoder；冒烟测试（数据加载/前向/反向/存 ckpt，监控显存耗时）；单模型拟合 → extract → OBJ；可行则算指标 | 生成可查看的重建 Mesh |
| S4 记录与整理 | 更新 REPRODUCTION / ENVIRONMENT / STATUS / logs；本地 Git 提交（不推送） | 四个文档与实际状态一致 |

约束：同一问题两轮有依据修复仍未解决 → 记录并暂停该分支；不伪造数据/权重/结果；实验记录写入 docs/logs/。

## 步骤清单

### 1. 获取官方代码【未开始】

- 克隆 https://github.com/Carrie-lin/Patch-Grid 到 patch-grid/，记录官方 commit 哈希到 docs/DEVIATIONS.md 基线信息。

### 2. 创建环境【未开始】

```bash
conda env create -f patchgrid.yaml
conda activate patchgrid
pip install https://download.pytorch.org/whl/cu111/torch-1.10.1%2Bcu111-cp39-cp39-linux_x86_64.whl
pip install https://download.pytorch.org/whl/cu111/torchaudio-0.10.1%2Bcu111-cp39-cp39-linux_x86_64.whl
pip install https://download.pytorch.org/whl/cu111/torchvision-0.11.2%2Bcu111-cp39-cp39-linux_x86_64.whl
```

### 3. 编译指标 CUDA 扩展（NGLOD）【未开始】

```bash
cd metrics/lib/extensions
chmod +x build_ext.sh && ./build_ext.sh
```

### 4. 下载官方数据【未开始】

- 从官方 README 的 OneDrive 链接下载：data/train 放入 data/train；预训练解码器（shape_space/checkpoints）放入 _results/shape_space。

### 5. 训练 Patch-Grid（固定预训练解码器）【未开始】

```bash
python train.py --dataset_path data/train/00000006_0 --exp_name 00000006_0 --pretrained _results/shape_space/decoder_1.pth --fix_decoder --use_shape_space
```

### 6. 训练 Patch-Grid-TS（从头训练）【未开始】

```bash
python train.py --dataset_path data/train/00000006_0 --exp_name 00000006_0 --epochs 500 --mile_stone 470 485
```

### 7. 网格抽取【未开始】

```bash
python extract.py --pretrained _results/models/00000006_0.pth --mesh_path_prefix _results/mesh --exp_name 00000006_0 --dataset_path data/train/00000006_0
```

### 8. 指标评估【未开始】

```bash
python metrics/mesh_metrics.py --gt_path data/train/00000006_0/00000006_0.obj --recon_path _results/mesh/00000006_0_composed_all.obj --metrics_file metrics.txt
python metrics/field_metrics.py --local_path _results/models/00000006_0.pth --dataset_path data/train/00000006_0 --field_metrics_file metrics_field.txt
```

### 9. 形状更新（shape updating）【未开始】

- 下载 data/edit 数据；先训练原始形状（--num_layers 2），再用 --update 训练更新形状，最后 extract。具体命令见官方 README「Shape updating」一节，执行时原文记录到当日日志。

## 代码流程分析（S1，2026-10-15，静态分析结论，未运行验证）

### 数据目录期望格式（dataset_path/<shape>/）

`patches/{i}.obj`（分割好的 patch 网格）、`adaptive_resolution.json`（每 patch 特征体分辨率指数）、`merge_order_adapt.json`（CSG 合并顺序）、`redboxes_adapt.json` / `redboxes_extract.json`（merge 盒）、`bounds/box_p{pid}_r{2^res}_surface.bin` 与 `_extend.bin`（格点盒起点）、`connect_type.txt`（patch 凸凹连接）；`--open` 时另需 `bdr{i}.npy`，`--update` 时另需 `change_info.txt`。**采样在训练时现场进行**（trimesh 切盒采样 + scipy cKDTree 分配；不用 igl/pysdf/SDF 库），缓存到 `--dataset_save`（默认 data/samples）。预处理代码官方未发布（todolist.txt 证实），数据必须来自官方 OneDrive。

### 训练 / 提取 / 评估要点

- 训练：优化 latent codes（nn.Embedding，--fix_decoder 时 decoder 冻结）；损失 = surf(200)+normal(50)+eikonal(5)+jitter(50)+merge(400)；**1 epoch = 1 次迭代**，Patch-Grid 主路径默认 300 epochs；checkpoint 覆盖写 `_results/models/<exp>.pth`（save_every=50 触发落盘）。
- `--use_shape_space --pretrained`：按 `<pretrained 去后缀>_latent.txt` 的数量建 codebook 严格加载 decoder 后丢弃 latent 重建（本仓库自带 decoder_1_latent.txt=21415、decoder_2_latent.txt=12151）。decoder 结构参数（hidden_dim/emb_deg/num_layers/activation）必须与 ckpt 一致。
- 提取：`skimage.measure.marching_cubes_lewiner` 逐 merge 盒 MC + CSG 组合 → `<prefix>/<exp>_composed_all.obj`；`--extract_resolution` 默认 2^11、`--extract_batch_size` 默认 80 万。
- 评估：mesh_metrics 用 trimesh + mesh2sdf(CUDA)；field_metrics 有既存 import 缺陷（`from option import parse_options`，仓库无该模块）需最小修补。

### 官方代码既存缺陷（非本机适配引入）

1. `example.sh` 用 `--mile_stones`（复数），argparse 定义为 `--mile_stone`，直接跑会报 unrecognized argument（README 命令正确）。
2. `metrics/field_metrics.py:9` `from option import parse_options` 必然 ImportError。
3. `extract.py:217` `np.int` 仅在存在 `delete_info.txt` 时触发（numpy≥1.24 移除）；用 numpy 1.22.4 可回避。
4. trainer 最终保存条件 `epoch == args.epochs` 恒为假（epochs 已 +1），实际靠 `save_every` 落盘。

### 环境风险与对策（6GB 显存）

- **mesh2sdf CUDA 扩展必须可 import**（utils→torchgp 无条件导入）→ 需在本环境编译：nvcc（conda 安装，不动系统）+ gcc≤11（conda-forge 编译器）+ TORCH_CUDA_ARCH_LIST=8.9。
- sol_nglod 扩展不编译（不影响训练+提取+mesh_metrics）。
- 显存推断：默认配置单形状拟合峰值 <1GB，充足；采样阶段 CPU 内存约 500 万点数组；提取时浅层大盒可能逼近上限 → 必要时调小 `--extract_batch_size` / `--extract_resolution`。
- 冒烟测试计划：epochs=2 跑通数据加载/前向/反向/存 ckpt，同时监控 nvidia-smi。

## 与论文的数值对照

- 待精读论文后补充各表格参考值（避免凭记忆填错）。

## 已知问题

- （暂无）
