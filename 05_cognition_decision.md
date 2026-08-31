# 05 Cognition & Decision

## 1. 目标与边界

本层负责理解任务、结合当前 World State 与 Memory 做推理，并把目标分解成可执行技能。

核心问题：

- 用户要什么；
- 当前世界是否允许执行；
- 下一步应该做什么；
- 任务如何分解；
- 技能失败后如何恢复；
- 用户中途修改任务怎么办；
- 什么信息需要向 Memory 查询。

## 2. 基本架构

```text
User / Task Input
      │
      ▼
Intent Understanding
      │
      ├──────────► Memory Query
      │               │
      ▼               ▼
World State ─────► Reasoning Core
                    │
                    ▼
                Task Planner
                    │
                    ▼
               Skill Selector
                    │
                    ▼
                  Tools
```

## 3. 输入

认知层输入不应只是一张图和一句话，至少包括：

- user instruction；
- current World State；
- robot status；
- active task state；
- relevant Memory retrieval；
- skill availability；
- previous tool result；
- safety constraints。

## 4. 输出

推荐高层输出保持结构化，而不是自然语言直接驱动底层：

```yaml
next_action:
  skill: pick
  target: object_37
  parameters:
    preferred_arm: right
  preconditions:
    reachable: true
  timeout_s: 20
  fallback: reobserve
```

## 5. Task Planning

典型任务：

```text
“把桌上的 Pepsi 给 Chloe”
    ↓
1. locate Pepsi
2. approach object
3. verify reachability
4. pick Pepsi
5. locate Chloe
6. navigate to Chloe
7. handover / place
8. confirm completion
```

高层 Planner 不应该关心每个关节如何运动。

## 6. VLM / LLM 的位置

VLM/LLM 更适合处理：

- 模糊语言理解；
- 开放世界语义；
- 多模态推理；
- task decomposition；
- tool selection；
- 失败解释；
- 用户交互。

不适合直接承担：

- 1 kHz 关节控制；
- 平衡闭环；
- 强实时安全逻辑。

## 7. Tool Calling

技能应以工具接口暴露，例如：

```text
say(text)
lookup(object/person/place)
observe(region)
navigate(target)
reachability(object, arm)
pick(object)
place(object, target)
follow(person)
wait(duration)
stop()
```

这种设计类似 Ludi 0.1：VLM 负责决策，Harness/Skill 层负责可靠执行。

## 8. Behavior Tree / State Machine

对于稳定流程，可将大模型与传统任务编排结合：

```text
VLM/LLM
  │
  ▼
Task Plan
  │
  ▼
Behavior Tree / State Machine
  │
  ▼
Skills
```

大模型解决开放性，BT/状态机保证确定性和恢复逻辑。

## 9. 中断与重规划

系统应原生支持：

- 用户说“停”；
- 用户修改目标；
- 目标丢失；
- skill timeout；
- navigation blocked；
- grasp failed；
- safety supervisor 抢占控制。

任务状态至少要支持：

```text
PENDING
RUNNING
PAUSED
CANCELLED
FAILED
SUCCEEDED
```

## 10. 决策频率

大脑层通常是低频、事件驱动：

- interaction：按用户事件；
- VLM reasoning：约 0.5~5 Hz 或按需；
- task planner：任务状态变化时触发；
- Behavior Tree：可 10~50 Hz tick，但不意味着每次都调用大模型。

## 11. 安全边界

高层模型输出必须经过约束层：

- skill whitelist；
- workspace constraints；
- collision / reachability check；
- task permission；
- emergency stop override；
- physical safety rules。

## 12. 推荐参考

- Ludi 0.1：VLM Agent + tools + interruption
- BehaviorTree.CPP / Nav2 BT
- SayCan / PaLM-E / RT-2 等 embodied planning 工作
- OpenAI / function-calling 风格的 tool-use pattern（仅作为软件架构思想）
