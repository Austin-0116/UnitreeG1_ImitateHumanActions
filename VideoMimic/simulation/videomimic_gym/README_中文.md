<div align="center">
  <h1 align="center">📹 <em>VideoMimic Gym</em></h1>
</div>

<p align="center">
  <strong>本仓库用于从人类视频中学习在不同地形上的运动，包含普通行走、DeepMimic 和策略蒸馏的实现，也支持深度渲染、高程图等功能。</strong>
</p>

## 📦 安装与配置

安装和配置步骤请参阅 [setup.md](/doc/setup.md)。

## 🔁 流程概览

使用强化学习实现运动控制的基本流程如下：

`训练` → `播放` → `仿真到仿真` → `仿真到实机`

- **训练：** 在 Gym 仿真环境中让机器人与环境交互，学习最大化所设计奖励的策略。为避免降低效率，不建议训练时实时可视化。
- **播放：** 使用播放命令验证训练结果，确认策略符合预期。
- **仿真到仿真：** 将 Gym 中训练的策略部署到其他仿真器，确认策略并非只适用于 Gym 的仿真特性。
- **仿真到实机：** 将策略部署到实体机器人上，实现机器人运动控制。

## 🛠️ 使用指南

### 1. 训练

运行以下命令开始训练：

```bash
python legged_gym/scripts/train.py --task=xxx
```

多 GPU 训练可使用 `torchrun`：

```bash
torchrun --nproc-per-node <num_gpus> legged_gym/scripts/train.py --multi_gpu --task=xxx
```

#### ⚙️ 参数说明

##### 基本参数

- `--task`：必填参数，可选任务包括：
  - `g1_deepmimic`：普通强化学习训练。
  - `g1_deepmimic_dagger`：策略克隆/蒸馏。
- `--headless`：默认启动图形界面；设置后以无界面模式运行，效率更高。
- `--resume`：从检查点恢复训练（从 `load_run` 指定的运行记录恢复）。
- `--experiment_name`：要运行或加载的实验名称。
- `--run_name`：本次运行的名称。
- `--load_run`：要加载的运行记录名称，默认加载最新记录。
- `--checkpoint`：检查点编号，默认加载最新文件。
- `--num_envs`：并行仿真环境数量。
- `--seed`：随机种子。
- `--max_iterations`：最大训练迭代次数。
- `--sim_device`：仿真计算设备，例如指定 CPU：`--sim_device=cpu`。
- `--rl_device`：强化学习计算设备，例如指定 CPU：`--rl_device=cpu`。
- `--multi_gpu`：启用多 GPU 训练。
- `--wandb_note`：添加 Weights & Biases 备注；含空格的字符串需用引号括起，例如 `"说明文字"`。

还可以通过 `--env.x=y` 覆盖环境配置，通过 `--train.x=y` 覆盖训练配置。下面列出一些常用参数。

##### 环境参数（`--env.*`）

- `deepmimic.use_amass`：是否使用 AMASS 动作捕捉数据（True/False）。
- `deepmimic.amass_terrain_difficulty`：AMASS 地形难度。AMASS 动作可配上随机崎岖地形；设为 1 表示没有崎岖地形，最大为 5，表示从无崎岖地形到最难地形中采样。
- `deepmimic.use_human_videos`：是否使用人类视频数据（True/False）。
- `deepmimic.human_video_oversample_factor`：人类视频过采样倍数。混合少量人类视频与较多 AMASS 数据时，可用它增加人类视频地形的采样频率。
- `deepmimic.amass_replay_data_path`：AMASS 数据文件路径，可包含通配符，例如 `ACCAD_export_retargeted_vnp6/*.pkl`。
- `deepmimic.human_video_folders`：人类视频数据目录列表。
- `deepmimic.init_velocities`：重置时是否使用参考动作的初始速度。
- `deepmimic.randomize_start_offset`：是否随机化动作参考的起始偏移；关闭时每次都从动作开头开始。
- `deepmimic.n_append`：在动作末尾追加的冻结帧数，可用于迫使模型在动作结束时保持稳定。
- `deepmimic.link_pos_error_threshold`：连杆位置误差阈值。若任意关节的笛卡尔位置与参考动作的误差超过该值，则终止回合。
- `deepmimic.is_csv_joint_only`：是否只使用 CSV 关节数据（仅用于将 Unitree LaFan 数据重新导出为 pkl 格式）。
- `deepmimic.cut_off_import_length`：导入动作的最大长度，可用于截断过长动作。
- `deepmimic.respawn_z_offset`：重生时的垂直偏移量；当动作中的脚与地面相交时，可通过抬高机器人避免该问题。
- `deepmimic.weighting_strategy`：采样起始位置的权重策略，可选 `uniform`（均匀）或 `linear`（线性）。
- `terrain.n_rows`：地形行数，用于提高数据加载效率，详见下文。
- `asset.terminate_after_large_feet_contact_forces`：是否在脚部接触力过大时终止回合。
- `asset.large_feet_contact_force_threshold`：脚部接触力过大的判定阈值。
- `asset.use_alt_files`：是否使用替代机器人模型文件，以便在不同 GPU 上略微随机化机器人几何形状（例如使用球形碰撞几何体）。
- `rewards.scales.xx`：奖励项权重，可能的配置见 [`g1_deepmimic_config.py`](/legged_gym/envs/g1/g1_deepmimic_config.py)。
- `rewards.only_positive_rewards`：是否只保留正奖励。普通任务建议设为 `True`；DeepMimic 任务建议设为 `False`，尽管训练初期表现可能变差，否则模型会忽略惩罚项。
- `rewards.joint_pos_tracking_k`：关节位置跟踪奖励的系数。奖励大致为 `exp(-关节位置误差之和 × k)`；`k` 越大，只有跟踪更精确时才能获得较高奖励，但过大可能导致策略放弃优化该奖励。
- `rewards.joint_vel_tracking_k`：关节速度跟踪奖励系数，含义同上。
- `rewards.link_pos_tracking_k`：连杆位置跟踪奖励系数，含义同上。
- `rewards.collision`：碰撞惩罚权重。
- `rewards.feet_contact_matching`：脚部接触匹配权重。
- `normalization.clip_actions`：动作裁剪上限。G1 推荐值约为 10。
- `normalization.clip_observations`：观测值裁剪上限。G1 推荐值约为 100。
- `control.beta`：动作输出的指数移动平均（EMA）系数。数值越低，平滑效果越强；1.0 表示不做 EMA 平滑。
- `domain_rand.randomize_base_mass`：是否随机化基座质量。
- `domain_rand.push_robots`：是否对机器人施加推力扰动。
- `domain_rand.max_push_vel_xy`：推力扰动的最大水平速度。
- `domain_rand.max_push_vel_interval`：施加推力扰动的最大时间间隔。
- `domain_rand.torque_rfi_rand`：是否随机化扭矩 RFI。
- `domain_rand.p_gain_rand`：是否随机化 P 增益。
- `domain_rand.p_gain_rand_scale`：P 增益随机化幅度。
- `domain_rand.d_gain_rand`：是否随机化 D 增益。
- `domain_rand.d_gain_rand_scale`：D 增益随机化幅度。
- `domain_rand.control_delays`：是否加入控制延迟。
- `domain_rand.control_delay_min`：最小控制延迟。
- `domain_rand.control_delay_max`：最大控制延迟。

##### 训练参数（`--train.*`）

- `policy.re_init_std`：是否使用噪声重新初始化策略。
- `policy.init_noise_std`：策略初始化噪声的标准差。
- `algorithm.learning_rate`：学习率。
- `algorithm.bc_loss_coef`：行为克隆损失系数（用于 DAgger）。
- `algorithm.policy_to_clone`：待克隆策略的路径（用于 DAgger）。
- `algorithm.bounds_loss_coef`：边界损失系数，用于防止策略动作均值超出 `clip_actions` 指定的范围，推荐值约为 0.0005。
- `algorithm.entropy_coef`：熵正则化系数。较高的值会鼓励策略标准差保持较大，从而在回合后期继续探索。
- `algorithm.schedule`：学习率调度方式；`fixed` 表示固定学习率，`adaptive` 表示根据 KL 散度自适应调整。
- `algorithm.desired_kl`：目标 KL 散度。
- `runner.save_interval`：模型保存间隔。

### 数据加载说明

目前支持两类数据：

* 带有地形的人类视频数据：[此处](../data/videomimic_captures)
* 不带地形的其他动作捕捉数据：[此处](../data/unitree_lafan)

数据由 [`replay_data.py`](legged_gym/tensor_utils/replay_data.py) 加载。该类接收一组 pickle 文件，并通过成员方法从中采样。动作片段以 pkl 文件形式导入，数据位于 [`data`](../data) 目录。

Isaac Gym（以及其他仿真器）通常会让多个环境共用地形网格，以批量处理地形，从而提升效率。但如果每个环境要使用不同地形，这种做法就不方便。此处的解决办法是将不同地形的网格拼接成一个网格，并维护全局环境偏移量 `env_offsets`（见 [`robot_deepmimic.py`](/legged_gym/envs/base/robot_deepmimic.py)）；根据动作片段的起始位置增加对应偏移，使机器人与地形对齐。

另一个需要注意的问题是：如果机器人在仿真器中彼此重叠，Isaac Gym 会检测到机器人之间的碰撞。虽然物理效果不会受到影响，但仿真速度会变得非常慢。如果很多机器人同时在同一块地形上执行相同动作，问题尤其明显，也会大幅增加内存占用。因此提供了 `n_rows` 参数，用于创建多行地形；这会有效增加地形数量，减少机器人彼此重叠的情况。

地形网格的拼接由 [`DeepMimicTerrain`](/legged_gym/utils/deepmimic_terrain.py) 完成，并根据动作片段索引计算偏移量。

本仓库支持加载两类动作片段，相关逻辑位于 [G1 DeepMimic 类](legged_gym/envs/g1/g1_deepmimic.py)中。第一类是普通 AMASS 动作片段，通过 `amass_replay_data_path` 指定目录，并通过 `use_amass` 开启或关闭；这些动作会与随机地形配对。第二类是人类视频数据，因为需要地形信息，所以会从同时包含 pkl 和网格信息的目录中读取。通过 `use_human_videos` 参数开关，并使用 `human_video_folders=[ /list/of/paths/of/videos/within/the/retargeted/data/folder/ ]` 指定重定向数据目录中的视频路径列表（见上文参数说明）。

**默认训练结果目录：** `logs/<experiment_name>/<date_time>_<run_name>/model_<iteration>.pt`

#### 示例命令

1. 多 GPU 强化学习训练：

```bash
torchrun --nproc-per-node 2 legged_gym/scripts/train.py --multi_gpu --task=g1_deepmimic \
  --headless --wandb_note "new_ft_old_terrains" \
  --env.deepmimic.use_amass=False \
  --load_run 20250225_132031_g1_deepmimic --resume \
  --env.deepmimic.use_human_videos=True \
  --env.deepmimic.human_video_oversample_factor=10 \
  --env.terrain.n_rows=6 \
  --train.policy.re_init_std=True \
  --train.policy.init_noise_std=0.5 \
  --train.algorithm.learning_rate=2e-5 \
  --env.deepmimic.n_append=50 \
  --env.deepmimic.link_pos_error_threshold=0.5 \
  --env.deepmimic.init_velocities=True \
  --env.deepmimic.randomize_start_offset=True \
  --env.asset.terminate_after_large_feet_contact_forces=False \
  --env.asset.use_alt_files=False
```

2. 策略克隆（DAgger）：

```bash
torchrun --nproc_per_node 2 legged_gym/scripts/train.py --task=g1_deepmimic_dagger \
  --multi_gpu --headless --wandb_note "distill" \
  --env.deepmimic.use_amass=False \
  --env.terrain.n_rows=10 \
  --env.deepmimic.amass_terrain_difficulty=1 \
  --env.deepmimic.use_human_videos=True \
  --env.deepmimic.init_velocities=True \
  --env.deepmimic.randomize_start_offset=True \
  --env.rewards.scales.feet_orientation=0.0 \
  --env.control.beta=1.0 \
  --train.runner.save_interval=50 \
  --train.algorithm.policy_to_clone_jitted=False \
  --train.algorithm.policy_to_clone=logs/g1_deepmimic/20250317_152046_g1_deepmimic \
  --train.algorithm.bc_loss_coef=1.0 \
  --train.algorithm.learning_rate=1e-4 \
  --env.deepmimic.n_append=50 \
  --env.asset.terminate_after_large_feet_contact_forces=False \
  --num_envs 2048
```

训练舞蹈动作（假设你已按[安装说明](./doc/setup_en.md)克隆 `retargeted_data`）：

```bash
torchrun --nproc-per-node 2 legged_gym/scripts/train.py \
  --multi_gpu \
  --task=g1_deepmimic_mocap \
  --headless \
  --env.terrain.n_rows=4096 \
  --env.deepmimic.amass_replay_data_path=lafan_replay_data/env_11_dance1_subject2.pkl \
  --env.deepmimic.cut_off_import_length=1600
```

（如果没有多张 GPU，请删除 `multi_gpu` 参数，改用 `python legged_gym/scripts/train.py`。）

检查点会保存在 `logs/g1_deepmimic/TAG` 中，`TAG` 会根据日期和时间生成。如果配置了 WandB，也会看到带有该标签的运行记录。

---

### 2. 播放

运行以下命令，在 Gym 中可视化训练结果：

```bash
python legged_gym/scripts/play.py --task=xxx
```

#### 基本播放参数

- `--num_envs`：要可视化的环境数量（默认值：1）。
- `--load_run`：要加载的运行记录名称，默认加载最新记录。
- `--checkpoint`：检查点编号，默认加载最新文件。
- `--headless`：无界面运行，适用于录制等场景。

#### 可视化选项

##### 1. Isaac Gym 标准可视化

默认使用 Isaac Gym 内置查看器。它提供基础可视化功能，但交互性可能较弱。

##### 2. Viser 可视化（推荐）

Viser 提供交互性更好的可视化界面，也支持通过网络访问。使用方法：

```bash
python legged_gym/scripts/play.py --task=xxx --env.viser.enable=True
```

Viser 相关参数：

- `env.viser.enable`：启用 Viser 可视化。
- `env.control.decimation`：控制更新频率；数值越大，可视化播放越慢。
- `env.control.beta`：动作平滑系数；数值越低，动作越平滑。

#### 示例命令

1. 使用最新模型进行基础可视化：

```bash
python legged_gym/scripts/play.py --task=g1_deepmimic --num_envs 1
```

2. 使用指定模型和 DeepMimic 配置通过 Viser 可视化（例如回放舞蹈动作）：

```bash
python legged_gym/scripts/play.py \\
  --task=g1_deepmimic_mocap \\
  --env.viser.enable=True \\
  --load_run TAG \\
  --num_envs 1 \\
  --env.deepmimic.amass_replay_data_path=lafan_replay_data/env_11_dance1_subject2.pkl \\
  --headless
```

#### 💾 导出网络

可以直接通过 Viser 界面导出网络。

---

### 3. 仿真到实机（实体部署）

相关代码目前尚未发布。作者在 Python 初步测试中使用了 [Unitree RL Gym](https://github.com/unitreerobotics/unitree_rl_gym)，在 Jetson 上部署实体机器人时使用了 [Unitree SDK2](https://github.com/unitreerobotics/unitree_sdk2) 和 [Humanoid Elevation Mapping](https://github.com/smoggy-P/elevation_mapping_humanoid) 软件包。

---

## 🎉 致谢

本仓库建立在以下开源项目的支持与贡献之上，谨致谢意：

- [legged_gym](https://github.com/leggedrobotics/legged_gym)：训练和运行代码的基础。
- [rsl_rl](https://github.com/leggedrobotics/rsl_rl.git)：强化学习算法实现。
- [unitree_sdk2_python](https://github.com/unitreerobotics/unitree_sdk2_python.git)：实机部署的硬件通信接口。
- [Unitree rl gym](https://github.com/unitreerobotics/unitree_rl_gym)：Unitree 机器人的 Gym 仿真环境。

---
