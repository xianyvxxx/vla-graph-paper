# VLA + Graph 论文每日总结

**日期：** 2026-03-09  
**搜索来源：** arXiv  
**新增论文数量：** 3 篇

---

## 论文 1：GraSP-VLA

### 📋 文献信息
- **标题：** GraSP-VLA: Graph-based Symbolic Action Representation for Long-Horizon Planning with VLA Policies
- **arXiv ID：** arXiv:2511.04357
- **作者：** Maëlic Neau, Zoe Falomir, Paulo E. Santos, Anne-Gwenn Bosser, Cédric Buche
- **提交时间：** 2025-11-06
- **链接：** https://arxiv.org/abs/2511.04357

### 🎯 VLA 核心总结
提出了一种神经符号方法 GraSP-VLA，使用连续场景图（Continuous Scene Graph）表示来生成人类演示的符号表示，作为低级 VLA 策略的编排器，扩展了长视野任务中可连续执行的动作数量。

### 🔗 Graph 输入机制
- **图类型：** Continuous Scene Graph（连续场景图）
- **节点定义：** 场景中的物体和机器人组件
- **边类型：** 物体间的空间关系和交互关系
- **特征聚合：** 从人类演示中提取符号表示
- **融合位置：** 作为高级规划器与低级 VLA 策略之间的编排层

> **关键文段：**
> > "GraSP-VLA, a framework that uses a Continuous Scene Graph representation to generate a symbolic representation of human demonstrations. This representation is used to generate new planning domains during inference and serves as an orchestrator for low-level VLA policies, scaling up the number of actions that can be reproduced in a row."

---

## 论文 2：Graph-Fused VLA (GF-VLA)

### 📋 文献信息
- **标题：** Graph-Fused Vision-Language-Action for Policy Reasoning in Multi-Arm Robotic Manipulation
- **arXiv ID：** arXiv:2509.07957
- **作者：** Shunlei Li, Longsen Gao, Jiuwen Cao, Yingbai Hu
- **提交时间：** 2025-09-09
- **链接：** https://arxiv.org/abs/2509.07957

### 🎯 VLA 核心总结
提出了 GF-VLA 框架，使双臂机器人系统能够直接从 RGB-D 人类演示中执行任务级推理和执行，采用信息论方法提取任务相关线索，构建时序场景图并与语言条件 transformer 集成。

### 🔗 Graph 输入机制
- **图类型：** Temporally Ordered Scene Graph（时序场景图）
- **节点定义：** 手 - 物体交互、物体 - 物体交互
- **边类型：** 时间顺序关系、空间关系
- **特征聚合：** 信息论方法提取任务相关线索，选择性突出关键交互
- **融合位置：** 与语言条件 transformer 集成，生成层次化行为树和笛卡尔运动原语

> **关键文段：**
> > "GF-VLA employs an information-theoretic approach to extract task-relevant cues, selectively highlighting critical hand-object and object-object interactions. These cues are structured into temporally ordered scene graphs, which are subsequently integrated with a language-conditioned transformer to produce hierarchical behavior trees and interpretable Cartesian motion primitives."

---

## 论文 3：GraphCoT-VLA

### 📋 文献信息
- **标题：** GraphCoT-VLA: A 3D Spatial-Aware Reasoning Vision-Language-Action Model for Robotic Manipulation with Ambiguous Instructions
- **arXiv ID：** arXiv:2508.07650
- **作者：** Helong Huang, Min Cen, Kai Tan, Xingyue Quan, Guowei Huang, Hong Zhang
- **提交时间：** 2025-08-11 (v2: 2025-08-23)
- **链接：** https://arxiv.org/abs/2508.07650

### 🎯 VLA 核心总结
提出了 GraphCoT-VLA，一个高效的端到端模型，通过结构化 Chain-of-Thought 推理模块和实时可更新的 3D Pose-Object 图，增强模型解释模糊指令和理解 3D 空间交互的能力。

### 🔗 Graph 输入机制
- **图类型：** 3D Pose-Object Graph（3D 姿态 - 物体图）
- **节点定义：** 机器人关节、3D 空间中的物体
- **边类型：** 机器人关节的空间配置、物体间的拓扑关系
- **特征聚合：** 实时可更新，捕获 3D 空间配置和拓扑关系
- **融合位置：** 与 Chain-of-Thought 推理模块集成，支持高级任务理解和规划

> **关键文段：**
> > "Additionally, we construct a real-time updatable 3D Pose-Object graph, which captures the spatial configuration of robot joints and the topological relationships between objects in 3D space, enabling the model to better understand and manipulate their interactions."

---

## 📊 总结对比

| 论文 | Graph 类型 | 主要贡献 | 应用场景 |
|------|-----------|---------|---------|
| GraSP-VLA | Continuous Scene Graph | 符号表示编排 VLA 策略 | 长视野任务规划 |
| GF-VLA | Temporally Ordered Scene Graph | 信息论融合 + 行为树生成 | 双臂机器人操作 |
| GraphCoT-VLA | 3D Pose-Object Graph | 3D 空间感知 + 模糊指令处理 | 开放环境操作 |

---

*生成时间：2026-03-09 01:54:21 CST*
