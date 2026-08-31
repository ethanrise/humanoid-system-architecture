# 06 Skills & VLA

## 1. 目标与边界

Skill Layer 是高层认知与低层运动控制之间的接口层。

高层 Decision 不应该直接输出关节力矩；低层 Controller 也不应该理解“把桌上的杯子递给某人”这种语义任务。Skill Layer 将语义任务转换为稳定、可调用、可监控、可中断的机器人能力。

## 2. 技能分类

```text
Mobility Skills
  ├─ navigate
  ├─ walk_to
  ├─ follow
  ├─ turn
  └─ stop

Perception Skills
  ├─ observe
  ├─ search
  ├─ inspect
  └─ relocalize

Manipulation Skills
  ├─ reach
  ├─ grasp / pick
  ├─ place
  ├─ handover
  ├─ open / close
  └─ press / pull / push

Social Skills
  ├─ say
  ├─ face_person
  └─ wait
```

## 3. Skill API

推荐每个 skill 都有统一生命周期：

```yaml
skill_request:
  skill_id: pick_001
  skill_type: pick
  target_object: object_37
  parameters:
    arm: right
  timeout_s: 20
```

执行状态：

```text
PENDING → RUNNING → SUCCEEDED
                 ├→ FAILED
                 ├→ CANCELLED
                 └→ TIMEOUT
```

反馈：

```yaml
skill_feedback:
  progress: 0.65
  phase: approach
  target_visible: true
  error_code: 0
```

## 4. Preconditions / Postconditions

技能不是“调用模型就执行”，应显式声明前置条件。

例如 Pick：

```text
Preconditions
  ├─ target exists
  ├─ target pose confidence sufficient
  ├─ target reachable
  ├─ selected arm available
  └─ collision risk acceptable

Postconditions
  ├─ object attached/held
  ├─ grasp confidence sufficient
  └─ World State updated
```

这样高层 Decision 可以基于技能结果重规划。

## 5. VLA 的位置

VLA 更适合作为 Manipulation Skill 的实现之一，而不是整个机器人唯一大脑。

```text
Decision
   ↓
pick(object_37)
   ↓
Skill Manager
   ↓
VLA Policy
   ↓
EEF / joint action
   ↓
Controller
```

不同 VLA 输出粒度不同：

- EEF delta pose；
- absolute EEF pose；
- gripper command；
- joint target；
- action chunk。

输出越接近关节层，越需要更严格的安全与实时约束。

## 6. VLA 输入

典型输入：

```text
RGB / RGB-D
+ robot proprioception
+ language/task instruction
+ optional object/world-state context
```

需要避免让 VLA 重复承担已有系统可靠提供的所有信息。如果 World State 已明确目标 ID、3D pose、可达性，可把这些结构化信息作为条件，而不是全部依赖模型从图像重新猜。

## 7. Visual Servo

精细操作建议使用闭环：

```text
coarse target from World State
    ↓
reach / approach
    ↓
wrist camera observation
    ↓
visual servo / local policy
    ↓
EEF correction
    ↓
contact / grasp
```

头部/胸部感知更适合全局和粗定位；腕部传感器更适合近距离闭环。

## 8. Skill Composition

复杂技能可以由基础技能组合：

```text
pick_and_place
  ├─ observe(target)
  ├─ reachability(target)
  ├─ approach(target)
  ├─ pick(target)
  ├─ move_to(place_pose)
  └─ place(target)
```

复杂度较高时，由 Behavior Tree / Task Planner 编排，而不是无限扩大单一 skill。

## 9. 失败恢复

常见失败：

- target lost；
- pose invalid；
- reachability false；
- grasp slip；
- collision predicted；
- VLA timeout；
- navigation blocked。

恢复策略：

```text
reobserve
relocalize
change arm
change viewpoint
retry with limit
fallback to classical planner
ask user
abort safely
```

## 10. 推荐实现方式

ROS 2 中建议高层技能优先用 Action：

```text
/pick
/place
/navigate
/follow_person
/search_object
```

因为 Action 天然支持：goal、feedback、result、cancel。

## 11. 推荐参考

- NVIDIA Isaac GR00T
- OpenVLA
- LeRobot
- RT-1 / RT-2
- Diffusion Policy
- ACT
- MoveIt 2
- Nav2 Actions
- Ludi 0.1 tool abstraction
