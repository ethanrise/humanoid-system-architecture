# Humanoid System Architecture

面向人形机器人的系统级架构研究与设计文档库。

目标不是围绕单一算法或单一模型整理资料，而是从完整机器人系统出发，梳理从机械本体、传感器、算力、感知、世界状态、记忆、认知决策、技能到实时控制的整体分层关系、数据流、接口与公开参考实现。

## 总体架构

```text
Physical World
    │
    ▼
Sensors & Compute
    │
    ▼
Perception
    │
    ▼
World State
    │
    ├──────────────► Memory
    │                  │
    │                  ▼
    └──────────────► Cognition / Decision
                       │
                       ▼
                    Skills / VLA
                       │
                       ▼
               Cerebellum / Control
                       │
                       ▼
                  Robot Body
                       │
                       └──────── feedback ────────┐
                                                │
                                                └──► Sensors
```

这里采用两条同时存在的划分轴：

1. **功能职责轴**：感知、世界状态、记忆、决策、技能、执行分别独立建模；
2. **控制时间尺度轴**：高层认知主要位于“大脑”侧，低延迟运动控制主要位于“小脑”侧。

因此，不简单把“感知”“VLA”“Memory”都塞进一个所谓大脑模型，而是明确每层的输入、输出、更新频率和系统边界。

## 文档结构

- [01 Robot Body](01_robot_body.md)：机械本体、关节、执行器、电气与身体拓扑
- [02 Sensors & Compute](02_sensors_compute.md)：传感器布置、算力、总线、网络与时间同步
- [03 Perception & World State](03_perception_world_state.md)：感知、融合、追踪、SLAM、Scene Graph、World State
- [04 Memory](04_memory.md)：Working / Object / Spatial / Episodic / Semantic / Interaction Memory
- [05 Cognition & Decision](05_cognition_decision.md)：VLM/LLM、任务规划、Tool Calling、Behavior Tree、异常恢复
- [06 Skills & VLA](06_skills_vla.md)：Navigate、Follow、Reach、Pick、Place、Visual Servo、VLA
- [07 Cerebellum & Control](07_cerebellum_control.md)：Locomotion、WBC、MPC、RL、IK、力控、关节控制
- [Papers](papers.md)：论文索引与阅读入口
- [References](references.md)：开源工程、平台和官方资料索引

## 设计原则

### 1. Observation 与 World State 分离

感知输出的是某一时刻的 Observation；World State 表示机器人当前对世界的持续估计。目标暂时被遮挡，不代表目标从世界中消失。

### 2. World State 与 Memory 分离

World State 解决“现在我认为世界是什么样”；Memory 解决“过去是什么样、发生过什么、哪些信息值得长期保留”。

### 3. Decision 与 Execution 分离

高层系统决定做什么以及调用哪个技能；低层系统负责如何稳定、安全、实时地完成动作。

### 4. Skill 是大脑与小脑之间的接口

`navigate(target)`、`pick(object)`、`place(object, target)`、`follow(person)` 等技能应该形成稳定接口。大脑不直接输出关节力矩，小脑也不负责理解自然语言任务。

### 5. 不以单一模型定义架构

VLM、VLA、World Model、RL Policy 都只是架构中的实现手段。先定义职责、状态和接口，再选择模型。

## 当前阶段

当前版本优先建立系统边界与参考架构。后续某一模块内容增大后，再从单文件升级为独立目录，例如：

```text
04_memory.md
    ↓
04_memory/
├── README.md
├── object_memory.md
├── episodic_memory.md
├── data_schema.md
└── references.md
```

这种方式可以避免项目早期目录过度复杂，同时保留后续扩展空间。
