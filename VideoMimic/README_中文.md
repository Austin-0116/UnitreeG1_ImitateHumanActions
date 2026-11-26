# VideoMimic

**CoRL 2025 最佳学生论文奖。**

[[项目主页]](https://www.videomimic.net/) [[arXiv 论文]](https://arxiv.org/pdf/2505.03729) [[会议论文集]](https://proceedings.mlr.press/v305/allshire25a.html)

**论文题目：Visual Imitation Enables Contextual Humanoid Control（视觉模仿实现具备情境感知能力的人形机器人控制）。**

<div style="background-color: #333; padding: 16px 20px; border-radius: 8px; color: #eee; font-family: sans-serif; line-height: 1.6;">
  <div style="font-size: 14px; margin-bottom: 12px;">
    Arthur Allshire<sup>*</sup>, Hongsuk Choi<sup>*</sup>, Junyi Zhang<sup>*</sup>, David McAllister<sup>*</sup>,
    Anthony Zhang, Chung Min Kim, Trevor Darrell, Pieter Abbeel, Jitendra Malik, Angjoo Kanazawa（* 表示共同第一作者）
  </div>
  <div style="font-size: 14px;">
    <i>加州大学伯克利分校</i>
  </div>
</div>

## 更新

- **2025 年 9 月 30 日：** VideoMimic 获得最佳学生论文奖。
- **2025 年 9 月 15 日：** 发布仿真代码和初步的仿真到实机代码。
- **2025 年 7 月 6 日：** 首次发布从真实视频到仿真的处理流程。

## VideoMimic Real-to-Sim

VideoMimic 的[真实到仿真处理流程](real2sim/README.md)从单目视频重建三维环境和人体动作，并将动作重定向到人形机器人，用于模仿学习。该流程提取世界坐标系下的人体姿态，将其映射为机器人构型；同时将环境重建为点云，再转换为网格。

## VideoMimic Simulation

提供仿真训练流程。详情见[说明文档](simulation/README.md)。流程分为四个阶段：动作捕捉预训练、场景条件化跟踪、策略蒸馏和强化学习微调。

## VideoMimic Sim-to-Real

提供实机部署流程。详情见[说明文档](sim2real/README.md)。仓库提供 C++ 文件，可将其编译为二进制程序，在真实机器人上运行 TorchScript 导出的检查点。

## 视频数据集

数据集[下载地址](https://drive.google.com/file/d/1lQWmxebQX9Yu_KBX_tpyg3CuP61hZrvP/view?usp=sharing)。请注意，视频以 JPEG 图像序列的形式提供，而不是编码后的 MP4 文件。

**[BibTeX 引用]**
```
@inproceedings{allshire2025visual,
  title={Visual Imitation Enables Contextual Humanoid Control},
  author={Allshire, Arthur and Choi, Hongsuk and Zhang, Junyi and McAllister, David and Zhang, Anthony and Kim, Chung Min and Darrell, Trevor and Abbeel, Pieter and Malik, Jitendra and Kanazawa, Angjoo},
  booktitle={Proceedings of The Conference on Robot Learning},
  series={Proceedings of Machine Learning Research},
  year={2025}
}
```
