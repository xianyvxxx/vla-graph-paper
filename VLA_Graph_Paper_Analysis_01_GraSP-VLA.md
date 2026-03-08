# VLA + Graph 论文深度解析

## 论文一：GraSP-VLA (arXiv 2511.04357v1)

---

### 📋 一、VLA 核心总结

**论文标题**: GraSP-VLA: Graph-based Symbolic Action Representation for Long-Horizon Planning with VLA Policies

**核心问题**: 当前 VLA 模型在长视野任务 (long-horizon tasks) 中性能显著下降，缺乏高层符号规划能力。

**解决方案**: 
- 提出**神经符号方法** GraSP-VLA，将 Continuous Scene Graph 作为 VLA 策略的"编排器"(orchestrator)
- 不训练单个复杂策略，而是训练多个**原子行为策略** (atomic behaviors)，在推理时用图结构组合
- 支持从单次演示中学习新任务 (single-shot demonstration learning)

**关键创新**:
1. 多层连续场景图 (Multi-Layer Continuous Scene Graph, ML-CSG) - 作为智能体内部记忆
2. 自动 PDDL 规划域生成算法 - 从观察中自动提取动作描述
3. 基于 CSG 的动作编排器 - 将任务分解为原子行为序列
4. 客户端 - 服务器架构的策略银行 (Policy Bank) - 预训练 VLA 策略库

**实验结果**:
- 在 6 个连续技能任务中，GraSP-VLA 成功率 0.4，而端到端微调仅 0.0
- 在 2 个技能任务中，GraSP-VLA 成功率 0.6，端到端微调仅 0.2
- PDDL 动作生成准确率平均 96%

---

### 🔗 二、Graph 输入机制 (重点详解)

#### 1. 图的数据结构

**多层场景图 (Multi-Layer Scene Graph)**:
```
G' = {V, E, L}
```
- **V**: 顶点集合，每个节点 v = (b, c, w)
  - b: 边界框坐标
  - c: 类别标签
  - w: 置信度值
- **E**: 边集合，每条边 e = (u, v, l, c, w)
  - (u, v) ∈ V: 连接的两个节点
  - l ∈ L: 所属层
- **L**: 层集合 (4 种关系类型)

**4 种关系层**:
| 层类型 | 示例 | 用途 |
|--------|------|------|
| Functional (功能) | ⟨hand, holding, cup⟩ | 检测人类动作/物体功能 |
| Topological (拓扑) | ⟨cup, on top of, table⟩ | 检测物体空间状态 |
| Physical Part-Whole (部分 - 整体) | ⟨hand, part of, person⟩ | 关联物体部件与动作 |
| Attributive (属性) | ⟨person, wearing, shirt⟩ | 补充物体状态 |

---

#### 2. 连续场景图 (Continuous Scene Graph) - 核心记忆结构

**定义**:
```
G_k^+ = {V_k, E_k, f_k, l_k}
```
- 在每个离散时间 k 更新
- f_k: 边到标签的映射函数
- l_k: 边到对应层的映射函数

**时间连续性机制**:
- 使用**多目标跟踪 (MOT)** 算法 (OC-SORT) 为每个物体分配跟踪 ID
- 节点变为 v = (b, c, w, i)，其中 i 是跟踪 ID
- 边通过 n×m 矩阵表示时间连续性 (n=时间戳数，m=4 层)

**状态精炼机制** (State Refinement):
- 滑动窗口 θ=3 时间戳
- 新关系需连续多次检测相同标签才确认
- 过滤误检，避免状态跳变 (如 above→below→above 的不合理变化)

**置信度更新公式**:
```
ω_r = ω_(r-1) + σ(τ_c - τ_r)
```
- σ=0.5 常数
- τ_c: 当前时间戳
- τ_r: 关系最后出现时间戳
- 低置信度关系自动移除

---

#### 3. Graph → VLA 的数据流向

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase I: 任务建模                          │
├─────────────────────────────────────────────────────────────┤
│  人类演示视频 → SGG 模型 → 多层场景图 → 时间聚合 → CSG         │
│                                              ↓               │
│                              PDDL 动作生成器 (Algorithm 1)    │
│                                              ↓               │
│                              PDDL 动作描述 + 调度顺序          │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                   Phase II: 任务执行                          │
├─────────────────────────────────────────────────────────────┤
│  Action Orchestrator (动作编排器)                            │
│      ↓ 读取当前 CSG 状态                                      │
│      ↓ 验证前置条件 (Preconditions)                          │
│      ↓ 匹配 PDDL 动作 → VLA 策略                              │
│      ↓ 客户端 - 服务器通信                                    │
│  Policy Bank (策略银行)                                      │
│      - pick_knife, pick_fork, pick_spoon                    │
│      - place_left, place_right, place_inside                │
│      ↓ 执行 VLA 策略                                          │
│      ↓ 返回完成信号                                           │
│      ↓ 更新 CSG 状态                                          │
│      ↓ 调用下一个策略                                         │
└─────────────────────────────────────────────────────────────┘
```

---

#### 4. 从 Graph 提取 PDDL 动作 (Algorithm 1)

**核心思想**: 动作 = 关系集合的前后变化

**算法流程**:
```
输入：滑动窗口 ζ=10, 功能层 F, 拓扑层 T
输出：动作列表 A

对每个时间 k=n 检测到的功能层关系 r_f = ⟨s, p, o⟩:
  1. 前置条件 P(r_f) = 拓扑层中 k=n-ζ 到 k=n 的关系
     (与对象 o 相关的状态)
  
  2. 效果 E(r_f) = 拓扑层中 k=n 到 k=n+ζ 的关系
     (与对象 o 相关的状态变化)
  
  3. 如果 P(r_f) ≠ ∅ 且 E(r_f) ≠ ∅ 且 E(r_f) ≠ P(r_f):
     - 计算负效果 NE(r_f) = {not(r) | r ∈ P(r_f) 且 r ∉ E(r_f)}
     - E(r_f) ← E(r_f) ∪ NE(r_f)
     - A ← A ∪ {⟨r_f, P(r_f), E(r_f)⟩}
```

**示例** (移动玻璃杯到架子):
- 初始状态 (t=0): ⟨glass_1, on, table_1⟩ (拓扑层)
- 动作中 (t=n): ⟨person_1, holding, glass_1⟩ (功能层) + ⟨glass_1, on, table_1⟩ 消失
- 最终状态 (t=n+ζ): ⟨glass_1, on, shelf_1⟩ (拓扑层) + ⟨person_1, holding, glass_1⟩ 消失

生成的 PDDL 动作:
```
(:action move_object
 :parameters (?obj ?from ?to)
 :precondition (and (on ?obj ?from))
 :effect (and (on ?obj ?to) (not (on ?obj ?from))))
```

---

#### 5. 推理时的图使用

**Action Orchestrator 工作流程**:
1. 从 CSG 提取当前状态 Π(l,k)
2. 对每个 PDDL 动作，检查前置条件是否满足
3. 如果满足，通过客户端 - 服务器调用对应 VLA 策略
4. 等待策略执行完成 (同步通信)
5. 更新 CSG 状态，验证效果是否达成
6. 如果效果未达成，可重新调用策略 (fallback 机制)
7. 调用下一个策略

**关键设计**:
- **不保留技能持续时间**: 每个技能执行到完成为止
- **开放式模仿学习**: 无预定义目标，动作在线组合
- **实时监控**: 用 CSG 监控每个策略执行结果

---

### 📚 三、文献信息

**作者**:
- Maëlic Neau¹, Zoe Falomir¹, Paulo E. Santos², Anne-Gwenn Bosser³, Cédric Buche⁴,⁵

**机构**:
1. 于默奥大学计算机科学系，瑞典
2. PrioriAnalytica, 澳大利亚
3. Bretagne INP - ENIB, 法国
4. IMT Atlantique, 法国
5. CNRS IRL 2010 CROSSING, 澳大利亚

**发表信息**:
- arXiv:2511.04357v1 [cs.RO]
- 日期：2025 年 11 月 6 日
- 领域：机器人学 (Robotics)

**许可证**: CC BY 4.0

**相关项目**:
- SGG 骨干：REACT 模型 (基于 YOLOV8m)
- VLA 基础：SmolVLA (shukor2025smolvla)
- 多目标跟踪：OC-SORT
- 数据集：IndoorVG (SGG 训练), DAHLIA (评估)

---

### ✨ 四、关键文段摘录

#### 关于 VLA 局限性的论述:
> "A major concern with current VLA models is the significant drop in performance in long-horizon tasks. Instead of training a single policy on a long and complex task, we propose to train a collection of individual policies on a set of simple, atomic behaviors which can be composed at inference time with our ML-CSG to reproduce the entire task."

#### 关于 Scene Graph 优势的论述:
> "In contrast to VLMs, SGG models are light-weight, do not hallucinate, and directly ground the symbolic representation to the visual content, which makes them good candidates for the autonomous extraction of action descriptions in AML."

#### 关于 Continuous Scene Graph 的定义:
> "We introduce the concept of a Continuous Scene Graph where nodes and edges are persistent through time. This representation then serves as the internal memory of a robotic agent."

#### 关于编排器机制的论述:
> "The preconditions of each action are validated using the current state of the G_k^+ before execution of the given policy. If the preconditions are not satisfied, the next policy is called."

#### 关于实验结果的论述:
> "By decoupling the learning in sets of unitary behavior and using our Continuous Scene Graph to decompose the task into a sequence of actions, we are able to largely improve the success rate. For instance, our approach improves the accuracy from 0.2 to 0.6 when combining 2 skills in a row."

#### 关于未来工作的论述:
> "As future work, we intend to extend our approach by infusing a latent representation of the Continuous Graph as an additional signal for the Action Expert of the VLA model. We believe that the spatial information contained in key graph relations can increase the success rate of VLA policies in real-world settings."

---

### 📊 补充：实验数据汇总

**表 I: SGG 模型性能 (IndoorVG 数据集)**
| 模型 | R@50/100 | mR@50/100 | mAP50 | 延迟 (ms) |
|------|----------|-----------|-------|-----------|
| REACT | 31.4 / 34.5 | 17.5 / 19.7 | 37.9 | 26.6 |

**表 II: DAHLIA 数据集规划域生成结果**
| 方法 | TP | FP | Recall |
|------|-----|-----|--------|
| Baseline | 平均 2.6 动作/视频 | 少 | 0.69 |
| w/ Informative Selection | 平均 32.2 动作/视频 | 多 | 0.51 |

**表 III: VLA 策略准确率**
| 策略 | Pick 准确率 | Place 准确率 |
|------|------------|-------------|
| knife | 0.8 | - |
| fork | 0.6 | - |
| spoon | 0.7 | - |
| left | - | 1.0 |
| right | - | 1.0 |
| inside | - | 1.0 |

**表 IV: GraSP-VLA 整体成功率**
| 技能数 | 动作生成准确率 | GraSP-VLA 成功率 | 端到端微调成功率 |
|--------|--------------|----------------|----------------|
| 2 | 1.0 | 0.6 | 0.2 |
| 4 | 1.0 | 0.4 | 0.1 |
| 6 | 0.9 | 0.4 | 0.0 |

---

### 💡 核心洞见

1. **Graph 不是 VLA 的输入，而是 VLA 的"大脑"**: CSG 不直接输入到 VLA 模型中，而是作为高层编排器决定何时调用哪个 VLA 策略

2. **四层图结构是关键创新**: 功能/拓扑/部分 - 整体/属性四层分离，使系统能区分动作 (功能层) 和状态 (拓扑层)

3. **时间连续性解决帧间不一致**: MOT+ 状态精炼机制确保图状态在时间上稳定，避免误检导致的规划错误

4. **开放式模仿学习范式**: 无需预定义目标，从单次演示中在线提取动作序列并立即执行

5. **可解释性强**: PDDL 动作描述提供符号级可解释性，便于调试和扩展

---

*解析完成时间：2025-11-06*
*论文来源：arXiv 2511.04357v1*
