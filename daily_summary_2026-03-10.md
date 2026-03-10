# VLA + Graph 论文每日总结

**日期：** 2026-03-10  
**搜索来源：** arXiv  
**新增论文数量：** 1 篇

---

## 论文 1：UniManip

### 📋 文献信息
- **标题：** UniManip: General-Purpose Zero-Shot Robotic Manipulation with Agentic Operational Graph
- **arXiv ID：** arXiv:2602.13086
- **作者：** Haichao Liu, Yuanjiang Xue, Yuheng Zhou, Haoyuan Deng, Yinan Liang, Lihua Xie, Ziwei Wang
- **提交时间：** 2026-02-13
- **链接：** https://arxiv.org/abs/2602.13086
- **项目主页：** https://henryhcliu.github.io/unimanip

### 🎯 VLA 核心总结
提出了 UniManip 框架，基于双层 Agentic Operational Graph (AOG) 统一语义推理和物理接地，通过高层 Agentic Layer 任务编排和低层 Scene Layer 动态状态表示的耦合，实现鲁棒的零样本长视野任务执行，相比 SOTA VLA 基线成功率提升 22.5%。

### 🔗 Graph 输入机制
- **图类型：** Bi-level Agentic Operational Graph (AOG，双层智能体操作图)
- **节点定义：** 
  - 高层 Agentic Layer：任务节点、语义意图
  - 低层 Scene Layer：物体中心场景图节点、几何约束
- **边类型：** 
  - 高层：任务依赖关系、语义关联
  - 低层：物体空间关系、碰撞约束
- **特征聚合：** 
  - 动态实例化物体中心场景图
  - 结构化记忆支持自主诊断和失败恢复
  - 安全感知局部规划器参数化表示为无碰撞轨迹
- **融合位置：** 
  - 双层图持续对齐抽象规划与几何约束
  - 作为动态智能体循环的核心表示

> **关键文段：**
> > "UniManip, a framework grounded in a Bi-level Agentic Operational Graph (AOG) that unifies semantic reasoning and physical grounding. By coupling a high-level Agentic Layer for task orchestration with a low-level Scene Layer for dynamic state representation, the system continuously aligns abstract planning with geometric constraints, enabling robust zero-shot execution."

> > "Unlike static pipelines, UniManip operates as a dynamic agentic loop: it actively instantiates object-centric scene graphs from unstructured perception, parameterizes these representations into collision-free trajectories via a safety-aware local planner, and exploits structured memory to autonomously diagnose and recover from execution failures."

---

## 📊 与已收录论文对比

| 论文 | Graph 类型 | 核心优势 | 应用场景 |
|------|-----------|---------|---------|
| UniManip (新增) | Bi-level Agentic Operational Graph | 零样本泛化 +22.5%，动态智能体循环 | 通用操作、移动操作 |
| GraSP-VLA | Continuous Scene Graph | 符号表示编排 VLA 策略 | 长视野任务规划 |
| GF-VLA | Temporally Ordered Scene Graph | 信息论融合 + 行为树生成 | 双臂机器人操作 |
| GraphCoT-VLA | 3D Pose-Object Graph | 3D 空间感知 + 模糊指令处理 | 开放环境操作 |

---

## 📝 备注

**其他搜索结果：**
- arXiv:2603.07647 (TempoFit) - 主要使用 KV 记忆机制，非 Graph 结构，未收录
- 已收录的 3 篇论文 (GraSP-VLA, GF-VLA, GraphCoT-VLA) 已去重跳过

---

*生成时间：2026-03-10 13:30:10 CST*
