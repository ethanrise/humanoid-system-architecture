# References

本文件整理人形机器人系统架构各层可公开获取的工程、平台与官方资料。

## 1. Robot Body / Hardware

### AgiBot X1

适合研究：

- 机械本体；
- BOM；
- 执行器；
- 电气；
- IMU；
- 整机装配；
- RL 控制；
- 实机部署。

入口：

- https://agibot.com/DOCS/OS
- https://github.com/AgibotTech/agibot_x1_infer

对应：`01_robot_body.md`、`07_cerebellum_control.md`

### Berkeley Humanoid Lite

适合研究：

- 低成本 humanoid 本体；
- 3D 打印机械结构；
- 执行器；
- BOM；
- URDF/MJCF/USD；
- Isaac Lab；
- sim2sim / sim2real；
- low-level control。

入口：

- https://github.com/HybridRobotics/Berkeley-Humanoid-Lite
- https://berkeley-humanoid-lite.gitbook.io/docs

对应：`01_robot_body.md`、`07_cerebellum_control.md`

## 2. Sensors / Compute / Middleware

### NVIDIA Isaac ROS

适合研究：

- GPU accelerated perception；
- ROS 2 graph；
- image pipeline；
- detection / DNN inference；
- zero-copy / NITROS；
- Jetson 部署。

入口：

- https://github.com/NVIDIA-ISAAC-ROS

对应：`02_sensors_compute.md`、`03_perception_world_state.md`

### ROS 2 / ros2_control

适合研究：

- middleware；
- DDS；
- hardware abstraction；
- controllers；
- lifecycle；
- Action / Service / Topic 接口。

入口：

- https://docs.ros.org/
- https://control.ros.org/

## 3. Perception / World State

### NVIDIA Isaac ROS perception stack

用于构建硬件加速的视觉感知链路。

### Hydra / Dynamic Scene Graph

适合研究：

- metric-semantic mapping；
- object / place / room 层次表达；
- dynamic scene graph；
- robot world representation。

### ConceptGraphs

适合研究：

- open-vocabulary 3D scene graph；
- visual-language features 与 3D object graph。

## 4. Memory / Cognitive Architecture

### Embodied VideoAgent

适合研究：

- persistent object memory；
- temporal history；
- dynamic scene understanding。

入口：

- https://arxiv.org/abs/2501.00358

### ARMAR / ArmarX

适合研究：

- cognitive robotics architecture；
- working / episodic / long-term memory；
- memory 与 perception/action 的关系。

建议入口：

- https://armarx.humanoids.kit.edu/

## 5. Cognition / Decision

### Ludi 0.1

适合研究：

- VLM Agent；
- tool calling；
- interruption / task correction；
- interaction memory；
- VLA 作为 manipulation skill。

入口：

- https://arxiv.org/abs/2608.22035

### BehaviorTree.CPP / Nav2 Behavior Trees

适合研究：

- deterministic task orchestration；
- failure recovery；
- action composition。

入口：

- https://www.behaviortree.dev/
- https://navigation.ros.org/

## 6. Skills / VLA

### NVIDIA Isaac GR00T

适合研究：

- humanoid foundation model；
- VLA；
- robot data pipeline；
- simulation / training / deployment；
- Jetson inference。

入口：

- https://developer.nvidia.com/isaac/gr00t
- https://github.com/NVIDIA/Isaac-GR00T

### LeRobot

适合研究：

- robot dataset；
- imitation learning；
- ACT / Diffusion / VLA workflows；
- training/evaluation pipeline。

入口：

- https://github.com/huggingface/lerobot

### MoveIt 2

适合研究：

- manipulation planning；
- IK；
- collision checking；
- planning scene。

入口：

- https://moveit.picknik.ai/

## 7. Cerebellum / Control

### Isaac Lab

适合研究：

- RL locomotion；
- manipulation；
- simulation；
- domain randomization；
- policy training。

入口：

- https://github.com/isaac-sim/IsaacLab

### Unitree SDK / ROS2 ecosystem

适合研究：

- humanoid low-level state；
- command interface；
- real robot integration。

## 8. Navigation

### Nav2

适合研究：

- navigation stack；
- planner/controller/recovery；
- Behavior Tree orchestration；
- ROS 2 Action interface。

入口：

- https://navigation.ros.org/

### KISS-ICP

适合研究：

- LiDAR odometry；
- lightweight real-time localization。

## 9. 推荐研究顺序

如果以完整系统架构为目标，建议按以下顺序阅读和验证：

```text
AgiBot X1 / Berkeley Humanoid Lite
    ↓
理解 Body + Control
    ↓
Isaac ROS / ROS2 / Sensors
    ↓
Perception + World State
    ↓
Embodied VideoAgent / ARMAR
    ↓
Memory
    ↓
Ludi
    ↓
Cognition / Decision
    ↓
GR00T / LeRobot / MoveIt / Nav2
    ↓
Skills + VLA + Execution
```

最终目的不是复刻某一个项目，而是抽象出一套平台无关的人形机器人 Reference Architecture。
