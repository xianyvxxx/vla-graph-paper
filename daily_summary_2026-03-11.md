# VLA+Graph 论文搜索每日总结 - 2026-03-11

## 执行概览

| 指标 | 数量 |
|------|------|
| 搜索总数 | 14 篇 |
| 已收录跳过 | 4 篇 |
| 非 Graph 相关跳过 | 7 篇 |
| **新增收录** | **3 篇** |
| GitHub 同步 | 待完成 |

---

## 新增论文详情

### 1. Differentiate-and-Inject (iSTAR)

- **arXiv ID**: arXiv:2602.07541
- **提交日期**: 2026-02-07
- **标题**: Differentiate-and-Inject: Enhancing VLAs via Functional Differentiation Induced by In-Parameter Structural Reasoning
- **作者**: Jingyi Hou, Leyu Zhou, Chenchen Jing, Jinghan Yang, Xinbo Yu, Wei He
- **领域**: cs.RO

**VLA 核心总结**:
提出 iSTAR 框架，通过在模型参数中嵌入任务级语义结构来增强 VLA 模型的功能分化。不同于将 VLA 视为单一策略，iSTAR 将任务级推理直接注入模型参数，无需外部规划器或手工提示。

**Graph 输入机制**:
- 使用 **implicit dynamic scene-graph knowledge**（隐式动态场景图知识）
- 场景图捕获对象关系、子任务语义和任务级依赖
- Graph 知识以参数空间结构推理的形式存在，而非显式输入

**关键文段**:
> "This injected structure takes the form of implicit dynamic scene-graph knowledge that captures object relations, subtask semantics, and task-level dependencies in parameter space."

**收录理由**: 明确使用 scene-graph 知识进行任务级推理，符合 Graph+VLA 研究方向。

---

### 2. FORGE-Tree

- **arXiv ID**: arXiv:2510.21744
- **提交日期**: 2025-10-07
- **标题**: FORGE-Tree: Diffusion-Forcing Tree Search for Long-Horizon Robot Manipulation
- **作者**: Yanjia Huang, Shuo Liu, Sheng Liu, Qingxiao Xu, Mingyang Wu, Xiangbo Gao, Zhengzhong Tu
- **领域**: cs.RO

**VLA 核心总结**:
提出 FORGE-Tree，一个插件式控制层，将阶段对齐的 Diffusion Forcing (DF) 头与测试时蒙特卡洛树扩散 (MCTD) 结合。通过部分去噪目标段而非整个轨迹，将轨迹优化转化为局部编辑序列。

**Graph 输入机制**:
- 使用 **scene graph** 提供扩展先验 (supplies priors for expansion)
- 使用几何关系感知评分进行 rollout (geometry relation-aware scoring for rollouts)
- 生成树结构化去噪，性能随搜索预算扩展

**关键文段**:
> "A scene graph supplies priors for expansion and geometry relation-aware scoring for rollouts, yielding tree-structured denoising whose performance scales with search budget while preserving the executed prefix."

**实验结果**: 在 LIBERO 基准上，相比原生 VLA 基线成功率提升 13.4-17.2 个百分点 (OpenVLA 和 Octo-Base)。

**收录理由**: 明确使用 scene graph 作为 MCTD 搜索的先验知识，符合 Graph+VLA 研究方向。

---

### 3. Mobility VLA

- **arXiv ID**: arXiv:2407.07775
- **提交日期**: 2024-07-10 (v2: 2024-07-12)
- **标题**: Mobility VLA: Multimodal Instruction Navigation with Long-Context VLMs and Topological Graphs
- **作者**: Hao-Tien Lewis Chiang, Zhuo Xu, Zipeng Fu, Mithun George Jacob, Tingnan Zhang, Tsang-Wei Edward Lee, Wenhao Yu, Connor Schenck, David Rendleman, Dhruv Shah, Fei Xia, Jasmine Hsu, Jonathan Hoech, Pete Florence, Sean Kirmani, Sumeet Singh, Vikas Sindhwani, Carolina Parada, Chelsea Finn, Peng Xu, Sergey Levine, Jie Tan
- **领域**: cs.RO, cs.AI

**VLA 核心总结**:
提出分层 VLA 导航策略，结合长上下文 VLM 的环境理解能力和基于拓扑图的鲁棒底层导航策略。高层策略使用长上下文 VLM 从演示视频中找到目标帧，底层策略使用目标帧和离线构建的拓扑图生成机器人动作。

**Graph 输入机制**:
- 使用 **offline constructed topological graph**（离线构建的拓扑图）
- 拓扑图作为底层导航策略的基础
- 支持多模态指令导航（语言 + 图像）

**关键文段**:
> "Mobility VLA, a hierarchical Vision-Language-Action (VLA) navigation policy that combines the environment understanding and common sense reasoning power of long-context VLMs and a robust low-level navigation policy based on topological graphs."

**实验结果**: 在 836m² 真实世界环境中评估，对之前未解决的多模态指令（如"我应该把这个放回哪里？"）实现高成功率。

**收录理由**: 明确使用 topological graphs 作为 VLA 底层策略的核心组件，符合 Graph+VLA 研究方向。

---

## 跳过论文说明

| arXiv ID | 标题 | 跳过原因 |
|----------|------|----------|
| arXiv:2603.07647 | TempoFit | 时序 KV 记忆机制，无 Graph 结构 |
| arXiv:2602.13086 | UniManip | 已收录 (2026-03-10) |
| arXiv:2508.05342 | Information-Theoretic Graph Fusion | GF-VLA 期刊版本，与 arXiv:2509.07957 重复 |
| arXiv:2512.03913 | Hierarchical VLA Using Success and Failure Demonstrations | 无 Graph 结构 |
| arXiv:2511.18082 | ActDistill | 知识蒸馏，无 Graph 结构 |
| arXiv:2511.01224 | Embodiment Transfer Learning | 迁移学习，无 Graph 结构 |
| arXiv:2511.04357 | GraSP-VLA | 已收录 (2026-03-09) |
| arXiv:2509.07957 | Graph-Fused VLA | 已收录 (2026-03-09) |
| arXiv:2508.07650 | GraphCoT-VLA | 已收录 (2026-03-09) |
| arXiv:2503.17406 | IRef-VLA | 交互指代基准，无 Graph 结构 |
| arXiv:2411.03540 | VLA-3D Dataset | 3D 数据集，无 Graph 结构 |

---

## 累计收录统计

| 日期 | 新增数量 | 累计数量 |
|------|----------|----------|
| 2026-03-09 | 3 篇 | 3 篇 |
| 2026-03-10 | 1 篇 | 4 篇 |
| 2026-03-11 | 3 篇 | 7 篇 |

---

## 下一步

1. ✅ 更新 Check.md 添加 3 篇新论文
2. ⏳ 通过 GitHub API 同步 daily_summary_2026-03-11.md 和 Check.md

---

*生成时间：2026-03-11*
