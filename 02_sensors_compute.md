# 02 Sensors & Compute

## 1. 目标与边界

本层负责机器人如何“获得数据”和“在哪里计算”。它连接物理世界与软件系统，是感知、定位、控制和认知的基础设施层。

核心问题：

- 哪些传感器放在哪里；
- 每种传感器解决什么问题；
- 数据带宽、时间同步、标定如何保证；
- MCU、实时 CPU、GPU/SoC、边缘计算分别承担什么任务；
- 控制总线与高带宽感知网络如何分离。

## 2. 推荐传感器分区

```text
Head
 ├─ RGB-D / stereo camera
 ├─ optional fisheye / panoramic camera
 └─ IMU (optional, depending on system design)

Chest
 └─ wide-FOV RGB-D

Wrist L/R
 └─ close-range RGB-D / RGB camera

Neck / Torso
 └─ LiDAR

Hands
 └─ tactile / force sensors

Body
 ├─ joint encoders
 ├─ motor current / torque estimate
 ├─ base IMU
 └─ foot force / contact sensors
```

## 3. 传感器职责

| 传感器 | 主要职责 | 典型更新率 |
|---|---|---:|
| RGB | 语义、检测、识别、VLM | 15~60 Hz |
| RGB-D / stereo | 近中距离 3D 几何 | 15~60 Hz |
| LiDAR | SLAM、全局几何、障碍物 | 10~20 Hz |
| IMU | 姿态、状态估计、稳定控制 | 100~1000+ Hz |
| Joint Encoder | 本体状态 | 500~2000+ Hz |
| Force/Torque | 接触与力控 | 100~1000 Hz |
| Tactile | 抓取接触、滑移 | 50~1000 Hz |

具体频率取决于硬件，以上仅用于架构量级判断。

## 4. 算力分层

推荐至少划分三个计算域：

```text
AI Compute Domain
  GPU / Jetson / x86 + NVIDIA GPU
  ├─ detection
  ├─ segmentation
  ├─ depth
  ├─ VLM / VLA
  └─ world state / memory

Real-time Compute Domain
  Real-time CPU / controller
  ├─ state estimation
  ├─ WBC / MPC / RL inference
  └─ safety supervisor

Motor Control Domain
  MCU / motor driver
  ├─ current loop
  ├─ velocity loop
  └─ position / torque servo
```

原则：大模型和图像推理不能阻塞实时控制域。

## 5. 通信网络

典型架构：

```text
Cameras ─ USB3 / GMSL / Ethernet ┐
LiDAR ───────── Ethernet ────────┼─► AI Computer
                                 │
AI Computer ─ DDS / ROS2 ────────┼─► Real-time Computer
                                 │
Real-time Computer ─ EtherCAT/CAN┼─► Joint Drivers
                                 │
Joint Drivers ────────────────► Motors
```

控制域优先使用确定性总线，例如 EtherCAT；高带宽视觉数据使用 Ethernet / GMSL / USB 等。

## 6. 时间同步

多传感器系统必须显式管理时间：

- hardware timestamp 优先；
- PTP / IEEE 1588；
- ROS time 只作为统一时间表达，不等价于硬件同步；
- camera trigger / sync pulse；
- IMU-camera、LiDAR-camera 时间偏移标定；
- 记录 `sensor_timestamp` 与 `receive_timestamp`。

感知融合不应该只依赖“消息到了就融合”。

## 7. 标定体系

至少维护：

```text
camera intrinsics
camera distortion
camera ↔ robot extrinsics
LiDAR ↔ robot extrinsics
wrist camera ↔ wrist_link extrinsics
IMU ↔ base_link extrinsics
```

所有空间输出最终应能转换到统一 TF 树：

```text
map
 └─ odom
    └─ base_link
       ├─ head_link
       │  └─ head_camera
       ├─ chest_camera
       ├─ left_wrist
       │  └─ left_wrist_camera
       └─ right_wrist
          └─ right_wrist_camera
```

## 8. 带宽与数据管理

高分辨率 RGB-D 很容易成为系统瓶颈，因此需要明确：

- 原始数据是否跨进程传输；
- intra-process / zero-copy；
- compressed image 是否用于远程监控而非算法链路；
- GPU buffer 是否能够复用；
- 点云是否按需生成；
- 是否仅传输目标级结果而非全量点云。

## 9. 健康状态

每个传感器建议提供：

```yaml
sensor_health:
  online: true
  fps: 30.0
  latency_ms: 18.4
  timestamp_age_ms: 12
  dropped_frames: 2
  temperature: 51.0
  calibration_valid: true
```

上层 World State 应知道数据是否可信，而不是把所有输入视为同质量。

## 10. 推荐参考

- NVIDIA Jetson Thor / Orin
- Isaac ROS
- ROS 2 DDS / intra-process communication
- EtherCAT
- PTP / IEEE 1588
- AgiBot X1 electrical architecture
- Berkeley Humanoid Lite hardware stack
