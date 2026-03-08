# VLA + Graph 论文搜索与解析进度

## 已完成解析的论文

### 1. GraSP-VLA (arXiv:2511.04357)
- **标题**: Graph-based Symbolic Action Representation for Long-Horizon Planning with VLA Policies
- **解析文件**: `VLA_Graph_Paper_Analysis_01_GraSP-VLA.md`
- **核心**: Graph 作为 VLA 的"编排器"，使用多层连续场景图 (ML-CSG)

### 2. MomaGraph (arXiv:2512.16909v1)
- **标题**: State-Aware Unified Scene Graphs with Vision–Language Model for Embodied Task Planning
- **解析文件**: `VLA_Graph_Paper_Analysis_02_MomaGraph.md`
- **核心**: 统一空间 - 功能场景图 + Graph-then-Plan 框架 + 强化学习训练

## 新发现的候选论文（待解析）

### 3. UniManip (arXiv:2602.13086) - 最新！
- **提交日期**: 2026 年 2 月 13 日
- **标题**: General-Purpose Zero-Shot Robotic Manipulation with Agentic Operational Graph
- **机构**: 未明确（作者来自多个机构）
- **核心**: Bi-level Agentic Operational Graph (AOG)
  - 高层 Agentic Layer：任务编排
  - 低层 Scene Layer：动态状态表示
  - 主动实例化物体中心场景图
  - 结构化记忆用于自主诊断和恢复
- **性能**: 比 SOTA VLA 和分层基线分别高 22.5% 和 25.0% 成功率
- **项目页面**: https://henryhcliu.github.io/unimanip

### 4. Differentiate-and-Inject (arXiv:2602.07541)
- **提交日期**: 2026 年 2 月 7 日
- **标题**: Enhancing VLAs via Functional Differentiation Induced by In-Parameter Structural Reasoning
- **核心**: 通过参数内结构推理增强 VLA

### 5. FORGE-Tree (arXiv:2510.21744)
- **提交日期**: 2025 年 10 月 7 日
- **标题**: Diffusion-Forcing Tree Search for Long-Horizon Robot Manipulation
- **核心**: 扩散 - 强制树搜索用于长视界操作

### 6. Graph-Fused VLA (arXiv:2509.07957)
- **提交日期**: 2025 年 9 月 9 日
- **标题**: Graph-Fused Vision-Language-Action for Policy Reasoning in Multi-Arm Robotic Manipulation
- **核心**: 图融合 VLA 用于多臂机器人操作

### 7. Information-Theoretic Graph Fusion VLA (arXiv:2508.05342)
- **提交日期**: 2025 年 8 月 7 日（2026 年 2 月 6 日更新）
- **标题**: Information-Theoretic Graph Fusion with Vision-Language-Action Model for Policy Reasoning and Dual Robotic Control
- **期刊**: Information Fusion (已接受)
- **核心**: 信息论图融合 + VLA 用于双机器人控制

### 8. IRef-VLA (arXiv:2503.17406)
- **提交日期**: 2025 年 3 月 20 日
- **标题**: A Benchmark for Interactive Referential Grounding with Imperfect Language in 3D Scenes
- **会议**: ICRA 2025
- **核心**: 3D 场景中不完美语言的交互式指称接地基准

## GitHub 上传状态

### ✅ 上传成功！

**已上传文件**:
1. `VLA_Graph_Paper_Analysis_01_GraSP-VLA.md` (12,032 bytes)
2. `VLA_Graph_Paper_Analysis_02_MomaGraph.md` (10,909 bytes)

**仓库地址**: https://github.com/xianyvxxx/vla-graph-paper

---

## 下一步计划

1. **优先解析 UniManip** (最新，2026 年 2 月提交)
2. 解析 Differentiate-and-Inject (2026 年 2 月)
3. 解析 Graph-Fused VLA 系列 (2025 年 9 月/8 月)
4. 根据需要解析其他论文
5. 继续上传新解析的论文

## 搜索策略

- ✅ arxiv.org 直接搜索：`VLA scene graph robot`
- ✅ 使用 Bing 搜索（需验证）
- 🔄 可尝试搜索词：
  - `scene graph VLA robot manipulation`
  - `knowledge graph vision language action`
  - `GNN VLA policy`
  - `graph reasoning robot learning`

## 搜索时间线

- **第一轮**: 百度搜索 → 找到 ReconVLA、RynnVLA-002、FastDriveVLA 等
- **第二轮**: arxiv 直接搜索 → 找到 GraSP-VLA、MomaGraph、UniManip 等 7 篇相关论文
- **进行中**: 深度解析每篇论文

---

**最后更新**: 2026-02-13 (根据论文提交日期推断当前时间)
