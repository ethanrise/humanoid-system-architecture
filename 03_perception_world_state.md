# 03 Perception & World State

## 1. 目标与边界

本层解决两个不同但紧密关联的问题：

1. Perception：机器人“这一时刻观测到了什么”；
2. World State：机器人“当前认为世界是什么样”。

二者必须分离。短时遮挡、传感器丢帧或检测失败，不应直接导致对象从世界状态中消失。

## 2. 总体流程

```text
Sensors
   │
   ▼
Raw Observation
   │
   ├─ detection
   ├─ segmentation
   ├─ depth
   ├─ pose
   ├─ tracking
   ├─ SLAM
   └─ human perception
   │
   ▼
Multi-sensor Association / Fusion
   │
   ▼
World State
   ├─ Robot State
   ├─ Object State
   ├─ Person State
   ├─ Spatial Relations
   ├─ Scene Geometry
   └─ Confidence / Uncertainty
```

## 3. Perception 输出

建议把单时刻感知结果视为 Observation：

```yaml
observation:
  timestamp: ...
  sensor_id: head_rgbd
  frame_id: head_camera_optical_frame
  detections:
    - class_name: cup
      score: 0.94
      bbox_2d: ...
      mask: ...
      position_3d: ...
      covariance: ...
      embedding: ...
```

它是证据，不是世界事实。

## 4. World State 数据模型

建议至少维护：

```yaml
world_state:
  timestamp: ...
  robot: ...
  objects: [...]
  persons: [...]
  relations: [...]
  scene: ...
```

### Object State

```yaml
object:
  object_id: object_37
  category: cup
  pose: ...
  velocity: ...
  size: ...
  visible: false
  last_seen: ...
  confidence: 0.91
  motion_type: movable
  source_list: [head_rgbd, chest_rgbd]
  uncertainty: ...
```

### Person State

除普通 Object 字段外，还可增加：

- person identity / re-id embedding；
- body pose；
- gaze / orientation；
- interaction state；
- tracked duration；
- last known location。

### Relation

```yaml
relation:
  subject: object_37
  predicate: on
  object: table_02
  confidence: 0.88
  timestamp: ...
```

关系可包括：`on`、`inside`、`near`、`holding`、`attached_to`、`in_front_of` 等。

## 5. 数据关联

多时刻、多视角感知要回答：新 Observation 是否对应已有实体？

可组合：

- 2D/3D 空间距离；
- 3D IoU / overlap；
- 类别一致性；
- CLIP / DINO / ReID embedding；
- 运动模型；
- 传感器视场约束；
- 可见性与遮挡关系。

不要依赖单一 tracker ID 作为长期 identity。

## 6. 静态与动态实体

World State 中可区分：

- static：墙、固定柜体、门框等；
- semi-static：桌椅等通常不动但可搬移实体；
- movable：杯子、箱子、工具；
- articulated：门、抽屉；
- dynamic agent：人、机器人、移动设备。

不同类型采用不同状态更新和遗忘策略。

## 7. 遮挡与存在性

示例：

```text
t0: cup visible
    ↓
World State: cup_37 exists, visible=true

t1: person blocks camera
    ↓
Perception: no cup observation
    ↓
World State: cup_37 exists, visible=false,
             last_known_pose=..., confidence slowly decays
```

这是 Perception 与 World State 分层的核心价值。

## 8. Scene Graph

World State 可以进一步形成动态图：

```text
person_12 ─ holding ─► cup_37
cup_37    ─ near ─────► table_02
box_08    ─ inside ───► shelf_01
robot     ─ facing ───► person_12
```

Scene Graph 是认知决策与 Memory 之间很自然的结构化接口。

## 9. 感知算法候选

- Detection：YOLO 系列、YOLO-World、Grounding DINO；
- Segmentation：SAM 2 / FastSAM；
- Tracking：ByteTrack、BoT-SORT、3D tracker；
- Feature：DINOv2、CLIP、person ReID；
- Depth：RGB-D、stereo、Depth Anything 系列；
- SLAM：LiDAR / visual-inertial / multi-sensor SLAM；
- Pose：FoundationPose、MegaPose、category-specific pose；
- Human：body pose、face/person identification（按隐私和场景需要）。

## 10. 与 Memory 的关系

```text
Observation(t)
    ↓
World State(t)
    ↓
State Change / Event Extraction
    ↓
Memory
```

Memory 不应保存每一帧完整感知结果作为唯一表示；应按需求保留关键观测、状态变化、事件和长期实体属性。

## 11. 更新频率

- 底层 detection/tracking：10~60 Hz；
- World State 更新：10~30 Hz 或事件驱动；
- Scene Relation：1~10 Hz 或事件驱动；
- VLM 语义理解：通常更低频，按需触发。

## 12. 推荐参考

- Embodied VideoAgent：persistent 3D object memory 与动态 scene understanding；
- ConceptGraphs / Open-vocabulary 3D scene graph；
- Hydra / Dynamic Scene Graph；
- Isaac ROS perception stack；
- Nav2 / SLAM Toolbox / KISS-ICP 等定位导航组件。
