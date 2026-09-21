# 🤖 Embodied-Future-GOAI2026

> GOAI 世界人工智能开源大赛 ·「具身未来」赛道参赛项目

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10%2B-blue" />
  <img src="https://img.shields.io/badge/License-Apache_2.0-yellow" />
  <img src="https://img.shields.io/badge/Demo-Runnable-brightgreen" />
</p>

---

## 📌 项目简介

本项目面向 **GOAI 世界人工智能开源大赛「具身未来」赛道**，聚焦具身智能（Embodied Intelligence）在真实物理世界中的落地应用。项目围绕两大赛题展开：

| 赛题 | 核心能力 | 技术栈 |
|------|---------|--------|
| 🦾 **双臂协作操作** | VLA 模型驱动的灵巧操作 | VLA · Isaac Sim · Sim-to-Real |
| 🐕 **全地形巡逻** | 多模态感知 + 自主导航 | A* · SLAM · Q-learning Locomotion |

---

## 🚀 快速开始

### 轻量 Demo 模式（推荐，无需 GPU / Isaac Sim）

仅需 Python 3.10+ 和 numpy 即可运行：

```bash
# 克隆仓库
git clone https://github.com/lilexi-bot/robot.git
cd robot

# 安装依赖（仅需 numpy）
pip install numpy

# 赛题一：双臂协作 Demo
python track1_dual_arm/scripts/demo.py --task pick_and_place

# 赛题一：训练策略（进化策略）
python track1_dual_arm/scripts/train.py --epochs 50

# 赛题一：评估策略
python track1_dual_arm/scripts/eval.py

# 赛题二：校园巡逻 Demo
python track2_patrol/scripts/run_patrol.py --mission campus_patrol

# 赛题二：训练运动控制（Q-learning）
python track2_patrol/scripts/train_locomotion.py --episodes 200

# 赛题二：可视化巡逻轨迹
python track2_patrol/scripts/visualize.py --mode ascii
```

### 完整仿真模式（需 Isaac Sim + GPU）

```bash
# 完整环境要求
# - NVIDIA GPU（RTX 3090 / A100 及以上）
# - Isaac Sim 4.2+
# - ROS2 Humble
# - CUDA 12.x
pip install -e ".[dev]"
```

---

## 🎮 Demo 演示

### Track 1: 双臂协作操作

**演示内容：** 双臂机器人执行 pick-and-place 任务，使用规则式 VLA 模型（基于 IK 启发式）生成关节目标，在轻量仿真环境中闭环运行。

```bash
python track1_dual_arm/scripts/demo.py --task pick_and_place --max_steps 200
```

**预期输出：**
- 3 个 episode 的实时进度日志（步数、奖励、抓取状态、任务阶段）
- 成功率、平均奖励、平均步数汇总
- 结果保存至 `outputs/track1_demo/demo_summary.json`

### Track 2: 全地形校园巡逻

**演示内容：** M20 四足机器人在 80×60m 模拟校园中完成 13 个检查点巡逻。集成 A* 路径规划、10 种地形分类、地形自适应步态切换、LiDAR 障碍检测、GPS/IMU 传感器融合。

```bash
python track2_patrol/scripts/run_patrol.py
```

**预期输出：**
- ASCII 校园地图（含建筑、草地、 gravel、沙地等地形分区）
- 巡逻路径可视化
- 地形统计（各地形步数/距离）
- 步态切换日志（如 trot→walk on gravel）
- 13/13 检查点到达确认
- 结果保存至 `outputs/track2_patrol/patrol_result.json`

详细输出示例见 [docs/demo_output_example.md](docs/demo_output_example.md)

---

## 🏗️ 技术架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Embodied-Future-GOAI2026                      │
├─────────────────────────┬────────────────────────────────────────┤
│  Track 1: 双臂协作操作   │  Track 2: 全地形巡逻                    │
│  ┌───────────────────┐  │  ┌──────────────────────────────────┐  │
│  │  VLA Model Layer  │  │  │    Navigation Stack              │  │
│  │  ├─ mock_vla (IK) │  │  │    ├─ A*/Dijkstra Planner        │  │
│  │  └─ action_head   │  │  │    ├─ SLAM Module                │  │
│  ├───────────────────┤  │  │    └─ NavStack                    │  │
│  │  Control Layer    │  │  ├──────────────────────────────────┤  │
│  │  ├─ dual_arm_ctrl │  │  │    Perception Module              │  │
│  │  └─ sim2real      │  │  │    ├─ Terrain Classifier (10类)   │  │
│  ├───────────────────┤  │  │    ├─ Obstacle Detector           │  │
│  │  Mock Environment │  │  │    └─ Multi-Sensor Fusion         │  │
│  │  ├─ mock_env      │  │  ├──────────────────────────────────┤  │
│  │  └─ Isaac Sim opt │  │  │    Locomotion Module              │  │
│  └───────────────────┘  │  │    ├─ Q-learning Controller       │  │
│                         │  │    ├─ Gait Generator              │  │
│                         │  │    └─ Terrain Adaptive            │  │
│                         │  └──────────────────────────────────┘  │
├─────────────────────────┴────────────────────────────────────────┤
│                    Common Layer (公共模块)                         │
│   Data Utils | Visualization | ROS2 Bridge (optional)            │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📂 项目结构

```
Embodied-Future-GOAI2026/
├── track1_dual_arm/        # 赛题一：双臂协作操作
│   ├── models/             # VLA 模型（mock + 接口）
│   ├── simulation/         # 仿真环境（mock + Isaac Sim 接口）
│   ├── control/            # 双臂控制算法
│   ├── evaluation/         # 评测指标
│   └── scripts/            # demo / train / eval
├── track2_patrol/          # 赛题二：全地形巡逻
│   ├── navigation/         # A*/Dijkstra 规划器、SLAM、导航栈
│   ├── perception/         # 地形分类、障碍检测、传感器融合
│   ├── locomotion/         # Q-learning 控制器、步态生成、地形自适应
│   ├── simulation/         # 校园环境、M20 机器人封装
│   ├── task_scheduler/     # 巡逻任务调度
│   └── scripts/            # run_patrol / train / visualize
├── common/                 # 公共模块
├── web/                    # Web 展示页
├── docs/                   # 文档
└── README.md
```

---

## 📖 文档

| 文档 | 说明 |
|------|------|
| [系统架构](docs/architecture.md) | 整体架构设计与模块说明 |
| [环境搭建](docs/setup_guide.md) | 环境搭建（含轻量 Demo 说明） |
| [Demo 输出示例](docs/demo_output_example.md) | 运行 demo 的预期输出 |

---

## 👥 团队

| 角色 | 成员 | 职责 |
|------|------|------|
| 队长 | TBD | 项目管理 & 架构设计 |
| 双臂方向 | TBD | VLA 模型 & 控制算法 |
| 巡逻方向 | TBD | 导航 & 感知 & 运动控制 |

---

## 📄 License

本项目基于 [Apache License 2.0](LICENSE) 开源。

---

> Built with ❤️ for GOAI 2026 - Embodied Future Track
> 
> 仓库地址：https://github.com/lilexi-bot/robot
