# VideoMimic 仿真

## 安装

如何安装 `videomimic_gym` 和 `videomimic_rl`，请参阅[安装指南](setup.md)。

### 下载数据

可以用以下命令下载检查点和视频数据集：

```
cd data
bash download_videomimic_data.sh
```

### 运行推理

运行以下任一推理脚本后，可通过 `localhost:8080` 打开 Viser 界面查看机器人动作。如果服务器是远程机器，可以使用 SSH 隧道、VS Code 端口转发或持久化 Cloudflare 隧道转发该端口。

先激活 `videomimic` Conda 环境，再运行：

*地形策略：*

```bash
bash videomimic_gym/legged_gym/scripts/play_terrain_policy.sh
```

（该命令只加载一个动作。也可以传入 `resources/data_config/human_motion_list_123_motions.yaml` 以运行全部动作，但加载所有网格可能需要较长时间。）

*蒸馏后的平地策略（用于比较）：接收根方向，但不接收动作参考。*

```bash
bash videomimic_gym/legged_gym/scripts/play_flat_policy.sh
```

*MCPT（第 1 阶段）策略推理：接收参考关节和根方向。*

```bash
bash videomimic_gym/legged_gym/scripts/play_mcpt_policy.sh
```

以上所有检查点都已在 Unitree G1 实机上测试。

### 运行训练

以下命令也都需要先激活 `videomimic` Conda 环境。

#### MoCap 预训练

```bash
bash videomimic_gym/legged_gym/scripts/train_stage_1_mcpt.sh
```

#### 第 2 阶段（在地形上跟踪动作）

```bash
bash videomimic_gym/legged_gym/scripts/train_stage_2_terrain_rl.sh
```

#### 第 3 阶段（策略蒸馏）

```bash
LOAD_RUN=stage_2_run_name # 替换为第 2 阶段的输出名称
bash videomimic_gym/legged_gym/scripts/train_stage_3_distillation.sh ${LOAD_RUN}
```

#### 第 4 阶段（强化学习微调）

```bash
LOAD_RUN=stage_3_run_name # 替换为第 3 阶段的输出名称
# 原文此处命令存在拼写错误；以下根据前面阶段脚本路径整理为正确形式
bash videomimic_gym/legged_gym/scripts/train_stage_4_rl_finetune.sh ${LOAD_RUN}
```

#### 其他说明

请根据可用 GPU 数量调整 `nproc-per-node`。本项目也原生支持多节点训练。

发布前并非所有阶段都经过端到端测试，结果也可能因数据而异。如遇到具体问题，可联系作者 Arthur。

关于如何修改训练脚本参数，详见 [`videomimic_gym` 的对应说明文档](videomimic_gym/README.md)。
