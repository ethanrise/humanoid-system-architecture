# 01 Robot Body

## 1. 目标与边界

本层描述机器人“身体本身”：机械结构、自由度、关节、执行器、电气连接和身体拓扑。它不负责感知、任务理解或高层规划。

核心问题：

- 机器人有多少自由度，如何分布；
- 每个关节能提供多大扭矩、速度和运动范围；
- 执行器、减速器、编码器、驱动器如何组合；
- 机械结构如何为感知、抓取、行走和散热服务；
- 本体如何暴露统一的状态与控制接口给上层。

## 2. 典型身体拓扑

```text
Head
 ├─ neck yaw / pitch
 └─ sensor mount

Torso
 ├─ waist yaw / pitch / roll
 ├─ compute / power
 └─ chest sensor mount

Left Arm / Right Arm
 ├─ shoulder
 ├─ elbow
 ├─ wrist
 └─ hand / gripper

Left Leg / Right Leg
 ├─ hip
 ├─ knee
 └─ ankle
```

典型全尺寸 humanoid 常见 20~40+ DoF；是否需要灵巧手会显著增加自由度与控制复杂度。

## 3. 关节与执行器

每个关节至少需要维护：

```yaml
joint_id: 12
name: left_elbow_pitch
type: revolute
position: 0.43
velocity: 0.02
torque: 3.1
temperature: 48.2
voltage: 48.0
current: 2.6
status: normal
limits:
  position_min: -1.5
  position_max: 1.5
  velocity_max: 8.0
  torque_max: 45.0
```

执行器通常由以下组件构成：

- 电机：PMSM / BLDC；
- 减速器：谐波、行星、摆线等；
- 编码器：电机侧、输出侧或双编码器；
- 驱动器：电流环 / 速度环 / 位置环；
- 温度、电流、电压等健康状态传感器。

## 4. 本体状态 Robot Body State

本体层应该向上提供统一的身体状态，而不是让上层直接理解每种驱动器协议。

建议至少包含：

```yaml
robot_state:
  timestamp: ...
  joint_positions: [...]
  joint_velocities: [...]
  joint_torques: [...]
  base_pose: ...
  base_twist: ...
  imu: ...
  contact_state:
    left_foot: true
    right_foot: true
  actuator_health: ...
```

## 5. 控制接口

根据执行层能力，可暴露不同级别接口：

- position target；
- velocity target；
- torque target；
- hybrid position + stiffness + damping；
- whole-body target。

原则：越靠近电机，实时性越强、抽象越低；越靠近技能层，抽象越高。

## 6. 硬实时要求

典型时间尺度：

- 电流环：10 kHz 级；
- 关节伺服：1 kHz 左右；
- WBC / RL locomotion：50~1000 Hz；
- 高层技能：1~50 Hz。

具体数字依平台而定，架构重点是确保高层大模型延迟不会进入低层稳定控制闭环。

## 7. 安全与故障

需要考虑：

- 过流 / 过温 / 过压；
- 编码器异常；
- 关节通信失联；
- 机械限位；
- 急停；
- 摔倒检测；
- 安全姿态和断电顺序；
- 单关节故障后的降级策略。

## 8. 推荐参考

- AgiBot X1：整机 BOM、机械、电气、执行器、部署链路；
- Berkeley Humanoid Lite：机械设计、执行器和低层控制；
- Unitree G1/H1：工业化 humanoid 本体与 SDK；
- ROS 2 `ros2_control`：统一硬件抽象与控制接口。

## 9. 与其他层的接口

上行：

```text
Robot Body
   ↓
Joint State / IMU / Contact / Health
   ↓
02 Sensors & Compute / 07 Control
```

下行：

```text
07 Cerebellum & Control
   ↓
Joint / Torque / Hybrid Commands
   ↓
Robot Body
```
