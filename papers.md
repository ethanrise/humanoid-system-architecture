# Papers

本文件作为人形机器人系统架构相关论文索引。重点不是追求论文数量，而是标注每篇论文主要解决架构中的哪一层问题。

## 1. Memory / World State

### Embodied VideoAgent

**Embodied VideoAgent: Persistent Memory from Egocentric Videos and Embodied Sensors Enables Dynamic Scene Understanding**  
arXiv: 2501.00358

关注：

- egocentric video；
- object-centric persistent memory；
- 2D perception 到 3D object memory；
- static / dynamic object update；
- temporal history；
- VLM-based state/action understanding；
- spatial / temporal reasoning。

对应本仓库：

```text
03 Perception & World State
        +
04 Memory
```

论文入口：

- https://arxiv.org/abs/2501.00358
- https://arxiv.org/pdf/2501.00358

---

### ARMAR / ArmarX Robot Memory

关注：

- cognitive architecture 中的机器人 Memory；
- working / episodic / long-term memory；
- semantic 与 sensorimotor representation；
- distributed / active / associative memory。

对应：

```text
04 Memory
        +
05 Cognition & Decision
```

建议进一步检索：

- A memory system of a robot cognitive architecture and its implementation in ArmarX

---

## 2. Cognition / Agent

### Ludi 0.1

**Ludi₀.₁: An Agentic System for Socially Intelligent Robots**  
arXiv: 2608.22035

关注：

- VLM 作为 reasoning core；
- tool calling；
- navigation / reachability / VLA skill；
- interaction history；
- mid-task interruption；
- task correction；
- context compaction；
- Agent Harness。

对应：

```text
04 Memory
05 Cognition & Decision
06 Skills & VLA
```

论文入口：

- https://arxiv.org/abs/2608.22035
- https://arxiv.org/pdf/2608.22035

---

## 3. VLA / Manipulation

### OpenVLA

关注：

- vision-language-action policy；
- language-conditioned manipulation；
- robot action generation。

对应：

```text
06 Skills & VLA
```

### RT-1 / RT-2

关注：

- large-scale robot policy；
- vision-language-action representation；
- semantic generalization。

### Diffusion Policy

关注：

- action trajectory generation；
- manipulation policy；
- multi-modal action distribution。

### ACT

关注：

- action chunking；
- imitation learning；
- low-cost robot manipulation。

---

## 4. Scene Graph / World Representation

建议持续补充：

- Dynamic Scene Graph；
- Hydra；
- ConceptGraphs；
- open-vocabulary 3D scene graph；
- persistent object-centric mapping；
- embodied spatial memory。

这些工作主要对应：

```text
03 Perception & World State
04 Memory
```

## 5. Locomotion / Control

建议持续补充：

- humanoid RL locomotion；
- whole-body control；
- model predictive control；
- sim2real locomotion；
- whole-body manipulation。

对应：

```text
07 Cerebellum & Control
```

## 6. 阅读记录建议模板

后续新增论文时建议记录：

```yaml
title: ...
year: ...
link: ...
layer:
  - perception
  - memory
problem: ...
inputs: ...
outputs: ...
models: ...
state_representation: ...
memory_representation: ...
robot_platform: ...
real_robot: true/false
open_source: true/false
key_takeaways: ...
limitations: ...
```

这样论文阅读最终可以回到“系统架构中解决了哪一个问题”，而不是只形成零散摘要。
