# 07 Cerebellum & Control

## 1. 目标与边界

本层负责把技能目标转成稳定、实时、安全的身体运动。它解决的是“动作如何做出来”，而不是“为什么做这个动作”。

主要职责：

- locomotion；
- balance；
- whole-body coordination；
- IK / trajectory tracking；
- contact / force control；
- joint-level command generation；
- safety override。

## 2. 分层结构

```text
Skill Target
   │
   ▼
Motion / Task Controller
   │
   ├─ IK / trajectory optimization
   ├─ RL policy
   ├─ WBC
   └─ MPC
   │
   ▼
Joint Targets
   │
   ▼
Joint Servo
   │
   ▼
Motor Driver
   │
   ▼
Actuator
```

## 3. Body State 输入

典型输入：

- joint position / velocity / torque；
- IMU orientation / angular velocity / acceleration；
- foot contact / force；
- hand force / tactile；
- base pose / velocity；
- target trajectory / EEF target。

## 4. Locomotion

可采用：

- model-based MPC；
- whole-body control；
- RL locomotion policy；
- hybrid RL + model-based control。

目标包括：

- velocity tracking；
- disturbance rejection；
- terrain adaptation；
- foot placement；
- balance recovery。

## 5. Whole Body Control

当抓取与行走同时发生时，需要协调全身：

```text
EEF task
+ CoM task
+ foot contact constraints
+ joint limits
+ collision constraints
   ↓
WBC / QP
   ↓
joint acceleration / torque
```

## 6. Manipulation Control

Skill/VLA 可输出较高层动作目标，小脑负责：

- trajectory smoothing；
- IK；
- velocity / acceleration limit；
- collision constraint；
- impedance control；
- force tracking；
- contact transition。

## 7. RL Policy

RL policy 更适合高频 sensorimotor mapping，例如：

```text
proprioception
+ command
    ↓
RL policy
    ↓
joint target / torque
```

需要通过 sim2sim / sim2real 验证，并显式处理 observation normalization、latency、actuator dynamics 和 safety limits。

## 8. 频率分层

典型量级：

```text
Task / Skill command       1~50 Hz
Motion policy / WBC       50~1000 Hz
Joint servo              ~1 kHz
Motor current loop        several kHz+
```

实际平台不同，但原则是越靠近物理执行器，越要求低延迟与确定性。

## 9. 安全优先级

建议控制优先级：

```text
Emergency Stop
    >
Safety Supervisor
    >
Balance / Fall Protection
    >
Joint Limits / Collision Constraints
    >
Skill Command
    >
High-level Decision
```

高层 Agent 不应拥有绕过底层安全约束的权限。

## 10. Skill 与小脑接口

例如 `reach`：

```yaml
motion_target:
  frame: base_link
  end_effector: right_hand
  pose: ...
  max_velocity: ...
  stiffness: ...
  contact_mode: free_space
```

小脑返回：

```yaml
motion_feedback:
  tracking_error: ...
  contact_state: ...
  stability_margin: ...
  status: running
```

## 11. Robot State Estimation

严格来说 State Estimation 是感知与控制之间的基础模块，但对小脑尤其关键：

```text
IMU
+ encoders
+ contact
+ optional odometry
   ↓
state estimator / EKF
   ↓
base pose / velocity / contact state
```

高层语义 World State 与低层 Body State 应分开维护。

## 12. 推荐参考

- Isaac Lab humanoid locomotion
- AgiBot X1 RL control
- Berkeley Humanoid Lite locomotion stack
- Unitree humanoid low-level SDK
- whole-body QP control
- MPC humanoid locomotion
- ros2_control
