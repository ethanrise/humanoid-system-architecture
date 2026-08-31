# 04 Memory

## 1. 目标与边界

Memory 负责把短时 Observation 与 World State 转化为可持续使用的历史、实体、空间、事件与知识表示。

它回答的不是“这一帧看到了什么”，而是：

- 这个对象是谁？
- 它之前在哪里？
- 什么时候发生了什么？
- 谁和谁发生了交互？
- 哪些状态变化值得保留？
- 哪些信息应该遗忘、压缩或长期保存？

Memory 属于高层认知系统的一部分，服务于决策、规划、问答和长期任务，但它不是实时控制环。

## 2. 建议的 Memory 分层

```text
Working Memory
  ├─ 当前任务
  ├─ 当前关注对象
  ├─ 当前局部上下文
  └─ 秒级状态

Object Memory
  ├─ identity
  ├─ category
  ├─ appearance embedding
  ├─ geometry
  └─ long-lived attributes

Spatial Memory
  ├─ room / zone
  ├─ object location
  ├─ topological relation
  └─ map-linked semantic state

Episodic Memory
  ├─ event
  ├─ actor
  ├─ target
  ├─ time
  ├─ location
  └─ outcome

Semantic Memory
  ├─ persistent facts
  ├─ object affordance
  ├─ place knowledge
  └─ learned concepts

Interaction Memory
  ├─ dialogue
  ├─ user correction
  ├─ task history
  └─ tool/action outcomes
```

## 3. Object Memory

建议字段：

```yaml
object_memory:
  object_id: cup_37
  category: cup
  first_seen: ...
  last_seen: ...
  appearance_embedding: ...
  geometry:
    size: ...
    canonical_shape: ...
  attributes:
    color: red
    movable: true
  current_ref: world_state/object_37
```

长期 identity 与当前状态分离。位置、可见性等动态信息应由 World State 维护，Memory 只保存必要的历史与长期属性。

## 4. Episodic Memory

事件应结构化表达：

```yaml
event:
  event_id: evt_20260831_001
  start_time: ...
  end_time: ...
  actor: person_12
  action: pick
  target: cup_37
  location: table_02
  before_state: on(table_02)
  after_state: holding(person_12, cup_37)
  confidence: 0.86
  evidence: [...]
```

事件是把连续 perception history 压缩成高价值语义单元的关键。

## 5. Working Memory

Working Memory 更接近认知系统当前上下文：

```yaml
working_memory:
  active_task: deliver_drink
  current_step: pick_object
  target_object: cup_37
  target_person: person_12
  focus_region: kitchen_table
  unresolved_questions: []
  recent_failures: []
```

它应该体积小、访问快、频繁更新。

## 6. Memory 写入流程

```text
Perception
   ↓
World State
   ↓
Change Detection
   ↓
Event / Entity Update
   ↓
Memory Write Policy
   ├─ ignore transient noise
   ├─ update object identity
   ├─ append event
   ├─ update semantic fact
   └─ compress old context
```

不是每一帧都写长期记忆。

## 7. 记忆读取

读取可以分为：

- exact ID lookup；
- temporal query；
- spatial query；
- semantic retrieval；
- graph traversal；
- vector similarity；
- hybrid structured + vector retrieval。

例如：

```text
“最后一次看到红杯子是什么时候？”
    ↓
Object Memory → cup_37
    ↓
Episodic / observation history
    ↓
last_seen event
```

## 8. 遗忘与压缩

需要明确 retention policy：

- 高频重复观测：聚合；
- 无变化状态：只保留关键帧；
- 低置信度短暂目标：快速淘汰；
- 重要任务事件：长期保存；
- 对话历史：阶段性摘要；
- 空间知识：持续修正而非无限追加。

## 9. 不确定性

Memory 不应把推断结果当成确定事实。

建议记录：

```yaml
fact:
  value: cup_37_on_table_02
  confidence: 0.72
  source: inferred
  evidence_ids: [...]
  last_verified: ...
```

## 10. Memory 与 World State 的关系

```text
World State = 当前 belief
Memory      = 历史 + 长期实体 + 事件 + 知识
```

Memory 可以反向帮助 World State：例如目标被遮挡时，用 last-known location 和历史运动模式维持 belief。

## 11. Memory 与 Decision 的关系

Decision 读取：

- 当前任务历史；
- 目标对象历史；
- 人物偏好/关系；
- 已失败动作；
- 过去位置；
- 环境语义知识。

Decision 也产生新的 interaction/action/result 写回 Memory。

## 12. 推荐参考

- Embodied VideoAgent：Object / temporal memory
- ARMAR / ArmarX Memory System：机器人认知架构中的主动 Memory
- Ludi 0.1：interaction history 与 context compaction
- Dynamic Scene Graph 相关工作
- Long-term embodied agent memory 研究
