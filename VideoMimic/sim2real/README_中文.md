# VideoMimic 实机部署

我们提供了完整的部署代码供参考。请注意，这些代码尚未经过充分测试和整理，主要作为参考实现。

可以在 Unitree 机器人的 Jetson 上部署，也可以部署在与 G1 相连的另一台计算机上。目前大多数代码都假设目标是 G1；若在机器人外部计算机运行，通常只需少量修改。

在 Unitree 机器人自带的 Jetson 上安装 ROS 1 Noetic。

### 安装高程地图代码

代码位于[此处](https://github.com/ArthurAllshire/elevation_mapping_humanoid)，它是在原始项目[此处](https://github.com/smoggy-P/elevation_mapping_humanoid)的基础上稍作修改得到的。

### 获取检查点

你可以从仿真中导出自己的检查点。

从[此处](https://drive.google.com/drive/u/0/folders/1VzfDTnC4KbeNbzWW0l3LiZHu11ajDFhr)下载 JIT 检查点。需要修改 [videomimic_inference_real](videomimic_real/videomimic_inference_real.cpp) 脚本中的 `default_model_path` 和 `heightmap_model_path`，使其指向你在系统中解压文件后的实际位置。默认路径为 `/home/unitree/Desktop/20250414_170842_g1_deepmimic_dict.pt` 和 `/home/unitree/Desktop/20250502_124756_g1_deepmimic_dict.pt`。

### 构建示例程序

要构建本仓库中的示例程序，先创建构建目录：

```bash
mkdir build
cd build
```

然后运行 CMake。为了让节点更灵活，我希望避免在节点中显式使用 Unitree 的 ROS 组件。因此需要手动针对 ROS 构建，并将 ROS 安装目录及工作空间的 `devel` 目录添加到 `CMAKE_PREFIX_PATH`。TorchScript 还需要针对 Torch 和 CUDA 进行构建。

以下命令应适用于 G1 默认搭载的 Jetson（需要将 ROS 工作空间路径改成你自己的路径）：

```bash
export CMAKE_PREFIX_PATH=/home/unitree/.local/lib/python3.8/site-packages/torch:/home/unitree/noetic_ws/devel:/opt/ros/noetic
cmake .. -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_CUDA_ARCHITECTURES="87" -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc
```

```bash
make
```

接下来启动高程地图等组件。示例见 `scripts/start_robot.sh`。该脚本也会运行推理程序（同样需要把路径改成你的 ROS 工作空间路径）。此时可执行文件应位于 `build/bin/videomimic_inference_real`。注意需要传入网络接口；如果程序在机器人本机运行，通常是 `eth0`。
