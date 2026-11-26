# SLOPER4D 评估脚本

此目录包含用于 SLOPER4D 数据集的评估脚本。按照以下步骤处理数据集并运行评估。

## 前置条件

开始前请确认所需环境和依赖已安装。从[此处](http://www.lidarhumanmotion.net/data-sloper4d/)下载 Sloper4D 数据集，并将其放入 `demo_data` 目录。

## 第 1 步：复制数据集

先复制 SLOPER4D 序列并重命名文件：

```bash
# 在仓库根目录运行
bash sloper4d_eval_script/copy_sloper4d_sequences.sh
```

```bash
# 在仓库根目录运行
python sloper4d_eval_script/rename_files.py
```

## 第 2 步：预处理人体数据

使用以下脚本预处理全部人体数据：

```bash
bash sloper4d_eval_script/preprocess_human_all.sh
```

## 第 3 步：预处理 MegaSAM

运行分布式 MegaSAM 预处理：

```bash
conda activate mega_sam
bash sloper4d_eval_script/preprocess_megasam_distributed.sh
```

## 第 4 步：运行我们的方法

### Hunt3r 处理

基于分块数据运行 Hunt3r：

```bash
bash sloper4d_eval_script/process_hunt3r.sh
```

### 合并结果

合并结果以获取最终输出：

```bash
python sloper4d_eval_script/cat_our_results.py --seq_num 007
python sloper4d_eval_script/cat_our_results.py --seq_num 008
```

## 第 5 步：运行评估

进入评估脚本目录并运行评估：

```bash
cd sloper4d_eval_script/
```

### 评估我们的方法

```bash
python run_eval_sloper4d.py --pred_smpl_path ../results/sloper4d_seq008/hps_combined_track_0.npy --output_dir ../results/sloper4d_seq008 --gt_pkl_path ../demo_data/sloper4d/seq008_running_001/seq008_running_001_labels.pkl

python run_eval_sloper4d.py --pred_smpl_path ../results/sloper4d_seq007/hps_combined_track_0.npy --output_dir ../results/sloper4d_seq007 --gt_pkl_path ../demo_data/sloper4d/seq007_garden_001/seq007_garden_001_labels.pkl
```

### 运行 CD 评估

```bash
python cd_eval.py
```

## 说明

- 根据实际配置调整序列编号（`--seq_num`）和文件路径。
- 评估结果会生成在指定的输出目录中。
