# 多人 MegaHunter 优化的稳健数据处理

此模块提供稳健的数据加载与验证函数，用于处理多人场景中的缺失帧和数据不一致问题。

## 功能

### 1. 稳健加载 SMPL 数据（`get_smpl_init_data_robust`）
- 遇到缺失的 SMPL 参数文件时能够妥善处理。
- 检测 SMPL 参数中的 NaN 值并报告。
- 使用最近邻插值补齐缺失帧。
- 详细记录缺失数据。

### 2. 稳健加载二维姿态数据（`get_pose2d_init_data_robust`）
- 处理缺失的二维姿态 JSON 文件。
- 过滤低置信度关键点。
- 报告有效关键点不足的帧。
- 支持为不同人员设置置信度阈值。

### 3. 多人数据验证（`validate_multi_human_data`）
- 验证 SMPL、二维姿态和边界框数据之间的一致性。
- 过滤有效帧不足的人员。
- 可配置每个人的最小有效帧比例阈值。
- 仅返回数据充足、可用于优化的人员。

### 4. 创建人物-帧掩码（`create_person_frame_mask`）
- 创建二值掩码，标记每个人出现在哪些帧中。
- 对多人优化至关重要，可处理某些帧中人员缺失的情况。
- 兼容 JAX 优化流程。

### 5. 插值缺失姿态（`interpolate_missing_poses`）
- 使用线性或最近邻方法插值缺失姿态。
- 处理序列开头或结尾缺失等边界情况。
- 支持针对不同数据类型使用不同插值方法。

## 用法

### 在 MegaHunter 优化中使用

使用 `--multihuman` 参数运行时，会自动调用这些稳健处理函数：

```bash
python stage2_optimization/01_megahunter_optimization.py \
    --world-env-path ./path/to/world.h5 \
    --bbox-dir ./path/to/bbox/json_data \
    --pose2d-dir ./path/to/pose2d \
    --smpl-dir ./path/to/smpl \
    --out-dir ./output \
    --multihuman \
    --top-k 3
```

### 调试 NaN 问题

使用调试脚本检查数据一致性：

```bash
# 基本检查
python debug_multihuman_nan.py \
    --bbox-dir ./path/to/bbox/json_data \
    --pose2d-dir ./path/to/pose2d \
    --smpl-dir ./path/to/smpl \
    --top-k 3

# 测试稳健加载
python debug_multihuman_nan.py \
    --bbox-dir ./path/to/bbox/json_data \
    --pose2d-dir ./path/to/pose2d \
    --smpl-dir ./path/to/smpl \
    --top-k 3 \
    --test-robust
```

## 主要优点

1. **防止 NaN 错误：** 在 NaN 值传播到优化流程前检测并处理。
2. **处理缺失帧：** 自动通过插值补齐缺失帧。
3. **过滤无效数据：** 移除有效数据不足的人员。
4. **详细日志：** 清晰报告数据问题。
5. **向后兼容：** 可与现有单人处理流程无缝配合。

## 实现细节

### 缺失帧处理
- 当文件不存在或包含 NaN 值时，判定为缺失帧。
- 使用最近的有效帧数据进行插值。
- 默认过滤缺失帧超过 50% 的人员。

### NaN 检测
- 检查所有 SMPL 参数（`betas`、`body_pose`、`global_orient`、`trans`）。
- 检查二维姿态关键点及其置信度。
- 报告 NaN 值的具体位置，便于调试。

### 数据验证
- 确保每个人物-帧组合都具备所需数据（SMPL、二维姿态、边界框）。
- 创建一致的人物-帧掩码供优化使用。
- 在优化前验证数据形状和类型。

## 故障排查

### 常见问题及解决办法

1. **“No valid SMPL data found for person X”（未找到人物 X 的有效 SMPL 数据）**
   - 检查 VIMO 是否成功处理了该人员。
   - 确认各处理阶段中的人员 ID 一致。
   - 考虑调低 `min_valid_frames_ratio`。

2. **“NaN found in frame X, person Y, param Z”（第 X 帧、人物 Y、参数 Z 中发现 NaN）**
   - 重新运行 VIMO 处理受影响的帧。
   - 检查该人员是否部分被遮挡。
   - 启用插值：`fill_missing=True`。

3. **“Only X out of Y persons have sufficient valid data”（Y 人中只有 X 人的数据有效）**
   - 若部分人员仅短暂出现在视频中，这属于正常情况。
   - 调整 `--top-k` 以处理较少人员。
   - 检查 SAM2 的检测质量。

## 参数

### 稳健处理的关键参数
- `min_confidence`：二维姿态关键点的最低置信度（默认值：`0.3`）。
- `min_valid_frames_ratio`：每个人所需的最低有效帧比例（默认值：`0.5`）。
- `fill_missing`：是否插值补齐缺失帧（默认值：`True`）。
- `frame_missing_thr`：允许插值的最大连续缺失帧数（默认值：`3`）。

### 多人优化参数
- `--multihuman`：启用多人处理。
- `--top-k`：要处理的人数，按人数较多者优先选择。
- `--joint2d-conf-threshold`：二维关节置信度阈值。
- `--joint3d-conf-threshold`：三维关节置信度阈值。
- `--frame-missing-thr`：缺失帧插值阈值。
