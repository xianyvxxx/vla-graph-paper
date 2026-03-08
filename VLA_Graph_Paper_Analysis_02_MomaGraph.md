# MomaGraph 论文解析

**arXiv:2512.16909v1 [cs.CV] | 18 Dec 2025**

---

## 1. VLA 核心总结

**核心问题：** 家庭环境中的移动操作机器人需要同时具备导航和操作能力，这需要一个紧凑、语义丰富的场景表示，能够捕捉物体在哪里、如何工作、以及哪些部分是可操作的。

**解决方案：** MomaGraph 提出了一种**统一场景表示框架**，将空间关系和功能关系整合在一起，并引入部件级交互节点，为具身智能提供紧凑、动态、任务对齐的知识结构。

**关键创新点：**

1. **MomaGraph 表示法**：首个同时建模空间 + 功能关系并包含部件级交互节点的场景图表示
2. **MomaGraph-Scenes 数据集**：首个大规模任务驱动场景图数据集（1050 个子图，6278 张多视图图像，350+ 家庭场景，93 种任务指令）
3. **MomaGraph-R1 模型**：基于 Qwen2.5-VL-7B 的 7B VLM，使用 DAPO 强化学习算法训练，通过图对齐奖励函数优化空间 - 功能推理
4. **Graph-then-Plan 框架**：先生成任务特定场景图作为中间结构化表示，再基于图进行任务规划，显著提升推理效果和可解释性
5. **MomaGraph-Bench 基准**：包含 6 种推理能力的系统评估套件（动作序列、空间推理、物体功能、前提条件&效果、目标分解、视觉对应）

**性能表现：**
- MomaGraph-R1 在 MomaGraph-Bench 上达到 **71.6% 准确率**（开源模型 SOTA）
- 比最佳基线提升 **+11.4%**
- 与闭源模型（Claude-4.5-Sonnet、GPT-5）性能相当
- 在真实机器人实验（RobotEra Q5）上实现 **70% 整体任务成功率**

---

## 2. Graph 输入机制（重点详解）

### 2.1 场景图定义

给定室内房间，智能体接收：
- **输入**：多视图图像 `{ℐᵢ}ᵢ₌₁ⁿ` + 自然语言指令 `𝒯`
- **输出**：任务导向场景图 `𝒢𝒯 = (𝒩𝒯, ℰ𝒯ˢ, ℰ𝒯ᶠ)`

其中：
- `𝒩𝒯`：任务相关物体节点集合
- `ℰ𝒯ˢ`：空间关系边集合（9 种类型）
- `ℰ𝒯ᶠ`：功能关系边集合（6 种类型）

### 2.2 数据流向

```
多视图 RGB 图像 + 任务指令
         ↓
   MomaGraph-R1 (7B VLM)
         ↓
   任务特定场景图 𝒢𝒯
   (节点 + 空间边 + 功能边)
         ↓
   Graph-then-Plan 规划器
         ↓
   结构化动作序列
         ↓
   参数化原语技能库
         ↓
   低层轨迹执行
```

### 2.3 场景图结构详解

**节点类型 (`𝒩𝒯`)：**
- **物体级节点**：完成任务所需的主要物体（如冰箱、灯、电视）
- **部件级节点**：交互式部件元素（如把手、旋钮、按钮）
  - 示例："打开冰箱" → 节点包含 [冰箱，把手]
  - 示例："开灯" → 节点包含 [开关，顶灯]

**空间关系边 (`ℰ𝒯ˢ`) - 9 种类型：**
- **方向性**：[LEFT OF], [RIGHT OF], [IN FRONT OF], [BEHIND], [HIGHER THAN], [LOWER THAN]
- **距离性**：[CLOSE], [FAR], [TOUCHING]

**功能关系边 (`ℰ𝒯ᶠ`) - 6 种类型：**
- [OPEN OR CLOSE] - 开/关门或柜子
- [ADJUST] - 参数调整
- [CONTROL] - 设备控制
- [ACTIVATE] - 激活设备
- [POWER BY] - 电源供应
- [PAIR WITH] - 组装配对（不改变物体内部状态，但修改空间配置）

**边的方向性：** 所有边均为有向边，从**触发物体**指向**受影响物体**

### 2.4 场景图 JSON 格式示例

```json
{
  "subgraph_id": "da21b9f9-f4fa-4a85-961b-2e2c2e182d3e",
  "scene_id": "466828",
  "action_type": "press",
  "function_type": "device_control",
  "task_instruction": "Turn on the television.",
  "nodes": [
    {"label": "remote control", "id": "f15474de-..."},
    {"label": "tv", "id": "91486017-..."}
  ],
  "edges": [
    {
      "relation_id": "ef3e72fe-...",
      "functional_relationship": "control",
      "object1": {"label": "remote control", ...},
      "object2": {"label": "tv", ...},
      "spatial_relations": ["lower_than", "in_front_of", "close"],
      "is_touching": false
    }
  ]
}
```

### 2.5 状态感知动态更新机制

场景图不是静态的，而是随环境状态变化动态更新：

**时刻 t 的场景图：**
`𝒢𝒯(t) = (𝒩𝒯(t), ℰ𝒯ˢ(t), ℰ𝒯ᶠ(t))`

**执行动作 aₜ 并观察新状态 sₜ₊₁ 后：**
`𝒢𝒯(t+1) = 𝒰(𝒢𝒯(t), aₜ, sₜ₊₁)`

**更新函数 𝒰(·) 的作用：**
- 移除不一致的假设
- 强化已确认的对应关系

**示例：** 厨房炉灶有多个旋钮，只有一个控制当前烹饪任务所需的燃烧器。通过旋转特定旋钮并观察燃烧器是否点燃，系统建立该旋钮与燃烧器之间的 [control] 功能边，同时剪枝其他旋钮的边。

### 2.6 强化学习奖励函数

MomaGraph-R1 使用 DAPO 算法训练，奖励函数设计为：

`ℛ(𝒢𝒯_pred, 𝒢𝒯_gt) = wₐ·(R_action + R_edges + R_nodes) + w_f·R_format + wₗ·R_length`

**奖励组件：**
1. **动作类型预测** `R_action`：确保正确预测所需动作类型
2. **空间 - 功能边整合** `R_edges`：联合评估每条边的空间和功能关系标签
3. **节点完整性** `R_nodes`：计算任务相关物体的 IoU 相似度
4. **格式验证** `R_format`：确保有效 JSON 结构
5. **长度控制** `R_length`：惩罚过于冗长的输出

**默认权重：** (wₐ=0.8, w_f=0.2, wₗ=0.5)

### 2.7 与 GraSP-VLA 的对比

| 特性 | GraSP-VLA | MomaGraph |
|------|-----------|-----------|
| Graph 角色 | VLA 的"编排器" | VLM 的直接输出 + 规划中间表示 |
| Graph 类型 | 多层连续场景图 (ML-CSG) | 统一空间 - 功能场景图 |
| 节点粒度 | 物体级 | 物体级 + 部件级 |
| 动态更新 | 未强调 | 状态感知动态更新机制 |
| 训练方式 | 监督学习 | 强化学习 (DAPO) |
| 基础模型 | 未指定 | Qwen2.5-VL-7B |
| 评估基准 | 自建 | MomaGraph-Bench (6 种推理能力) |

---

## 3. 文献信息

**标题：** MomaGraph: State-Aware Unified Scene Graphs with Vision–Language Model for Embodied Task Planning

**作者：**
- Yuanchen Ju*¹, Yongyuan Liang*², Yen-Jen Wang*¹
- Nandiraju Gireesh¹, Yuanliang Ju³, Seungjae Lee²
- Qiao Gu³, Elvis Hsieh¹, Furong Huang†², Koushil Sreenath†¹

**机构：**
- ¹ University of California, Berkeley
- ² University of Maryland, College Park
- ³ University of Toronto

**项目网站：** https://HybridRobotics.github.io/MomaGraph/

**arXiv：** 2512.16909v1 [cs.CV]

**发布日期：** 2025 年 12 月 18 日

**训练配置：**
- 基础模型：Qwen2.5-VL-7B-Instruct
- 训练框架：EasyR1
- 硬件：8×80GB A100 GPU
- 训练时间：约 13 小时
- 总步数：175 步
- 学习率：1e-6

---

## 4. 关键文段

### 4.1 动机阐述

> "When mobile manipulators enter household environments, they face the fundamental challenge of understanding how the environment works, which objects are interactive, and how they can be used. In other words, such robots must not only be capable of navigating through the home, but also manipulate objects within it."

> "However, existing scene graphs suffer from notable limitations: (1) Their edges typically encode only a single type of relationship, either spatial or functional. (2) Most existing methods are limited to static scenes and struggle to adapt to dynamic environments. (3) They lack task relevance, as they fail to emphasize information directly tied to task execution."

### 4.2 Graph-then-Plan 框架

> "We propose the Graph-then-Plan strategy, which first generates task-specific scene graphs as an intermediate structured representation before high-level planning. By explicitly modeling objects and their relations, this approach significantly improves the accuracy and robustness of task planning."

> "Unlike prior graph‐then‐plan methods that either assume reliable scene graphs or treat graph construction and planning as separate modules, our approach enables a single VLM to jointly generate structured, task-oriented scene graphs and perform high-level planning."

### 4.3 实验发现

**直接规划 vs Graph-then-Plan：**
> "Despite their scale, closed-source VLMs like GPT-5 produce incorrect or incomplete plans, missing prerequisite steps, or misidentifying interaction types. In contrast, our Graph-then-Plan approach consistently produces correct and complete action sequences aligned with ground-truth logic."

**单一关系 vs 统一关系：**
> "Graph representations that rely solely on spatial relationships or solely on functional relationships are insufficient. For embodied agents, a unified representation that jointly models both spatial and functional relationships provides a more complete and effective foundation for perception and action."

### 4.4 性能总结

> "MomaGraph-R1 achieves performance on par with closed-source giants like Claude-4.5-Sonnet and GPT-5, while clearly surpassing all leading open-source VLMs. Notably, MomaGraph-R1 delivers a +11.4% relative improvement over its base model (Qwen2.5-VL-7B) under w/ Graph setting."

> "As task complexity increases from Tier 1 to Tier 4, the performance of most open-source baselines drops sharply, reflecting their limited ability to generalize to multi-step reasoning. In contrast, MomaGraph-R1 exhibits a much smaller degradation, preserving strong performance in Tier 3 and Tier 4."

### 4.5 真实机器人部署

> "Our real-world evaluations show that MomaGraph-R1 delivers robust scene understanding and task planning even in unseen scenarios, while remaining directly compatible with standard mobile humanoid systems. This combination underscores the strength of our model and its practicality for real-world deployment."

**复杂多步任务评估（10 次试验）：**
- 场景图生成成功率：80%
- 规划成功率（基于正确图）：87.5%
- 整体任务成功率：70%

**主要失败模式：**
1. 场景图生成阶段的空间关系错误或节点缺失
2. 规划阶段的动作排序错误

---

## 附录：评估基准能力分类

**MomaGraph-Bench 六维评估：**

1. **动作序列推理 (T1)**：理解动作顺序和依赖关系
2. **空间推理 (T2)**：推理空间关系（如 left_of、in_front_of），判断可达性
3. **物体功能推理 (T3)**：推断物体功能（如旋钮可旋转、柜子可打开）
4. **前提条件&效果推理 (T4)**：理解动作前提和效果（如门需先关才能开）
5. **目标分解**：将复杂任务分解为子目标，确定并行/顺序执行策略
6. **视觉对应**：跨多视图保持物体一致性，整合视角变化信息

---

**解析完成时间：** 2025-12-18
**论文来源：** arXiv 2512.16909v1
