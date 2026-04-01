---
title: Spatial Representation
last_updated: "2026-04-01"
---
## Overview

为 embodied agent 构建语义丰富的空间表示，支撑导航与操作。核心问题：如何设计一种 shared spatial representation 同时服务 VLN 的路径规划和 VLA 的物体操作？

**发展现状**：三类方案并行发展——dense feature maps（VLMaps）、3D scene graphs（ConceptGraphs）、neural/Gaussian fields（SplaTAM）。2025–2026 年的关键进展：（1）3D spatial features 与 2D semantic features 的 cross-attention fusion 成为 VLN 新标准（PROSPECT: CUT3R+SigLIP，GTA: TSDF+topological graph）；（2）Multi-scale memory 架构（MEM: video 短期 + language 长期）为 VLA 提供了 15 分钟级记忆；（3）Explicit world representation 作为 plug-in 可普遍提升 MLLM-based agent 的性能。但 real-time incremental update 和 Nav+Manip 统一表示仍未解决。

## Core Concepts

- **Semantic SLAM**: 在 SLAM 基础上融合语义信息（object class、language description）的空间建图方法
- **3D Scene Graph**: 以 object 为节点、spatial/semantic relation 为边的层次化场景表示
- **Dense Feature Map**: 每像素存储 VLM feature 的稠密表示（如 VLMaps），支持 open-vocabulary 空间查询
- **Neural/Gaussian Field**: 用 neural implicit 或 Gaussian Splatting 表示的连续 3D 场景（如 SplaTAM）
- **Online Spatial Memory**: 从 RGB-D 流实时增量构建的空间记忆，无需离线重建
- **Explicit World Representation (EWR)**: TSDF volumetric map + topological graph 组合，为 MLLM 提供 metric geometric grounding
- **Multi-Scale Embodied Memory**: 不同时间尺度采用不同表征——近期事件需密集视觉细节，远期事件只需语义抽象

## Established Knowledge

1. **三类主流语义空间表示**：（1）dense feature maps（VLMaps：per-pixel VLM features）；（2）3D scene graphs（ConceptGraphs：object nodes + semantic relations）；（3）neural/Gaussian fields（SplaTAM：Gaussian Splatting SLAM）。各有适用场景。
   - 来源：[[2210-VLMaps]]、[[2309-ConceptGraphs]]、[[2312-SplaTAM]]

2. **3D scene graph 是最有潜力同时服务 VLN 和 VLA 的表示**：object nodes 提供 navigation waypoints（VLN）和 manipulation targets（VLA），semantic relations 支持 task planning。VL-Nav 用 3D scene graph + image memory 达 83.4% 室内 SR。
   - 来源：[[2309-ConceptGraphs]]、[[2502-VLNav]]、[[VLN-VLA-Unification]]

3. **Online spatial memory 优于 offline 3D 重建**：MTU3D 的 online query merging 直接从 RGB-D 流构建 dynamic spatial memory，无需离线重建，更适合实时 agent。
   - 来源：[[2507-MTU3D]]

4. **3D spatial + 2D semantic cross-attention fusion 是 VLN 的新标准**：PROSPECT（CUT3R 3D + SigLIP 2D）在 R2R-CE 达 60.3% SR；GTA（TSDF + topological graph + BEV visual prompting）在 zero-shot 达 48.8% SR。两者从不同角度验证了显式 3D 建模的价值。
   - 来源：[[2603-PROSPECT]]、[[2602-GTA]]

5. **Multi-scale memory 解决 long-horizon VLA 的记忆需求**：MEM 的双记忆架构——video 短期记忆（54 秒，时空可分离注意力）+ language 长期记忆（分钟级，语义摘要）——使 VLA 在 15 分钟级任务中达 70-80% success rate，而无记忆 baseline 仅 ~10%。
   - 来源：[[2603-MEM]]

6. **Explicit world representation 作为 plug-in 可普遍提升 MLLM agent**：GTA 的 EWR 不仅在自身框架中有效，作为 plug-in 加入 NavGPT（+1.5% SR）和 SmartWay（+6.7% SR）也有提升，验证了 metric representation 的通用价值。
   - 来源：[[2602-GTA]]

## Active Debates

1. **Explicit spatial representation vs end-to-end**：VLA 趋向 end-to-end（无显式空间表示），但 VLN 和 long-horizon 任务仍需 explicit spatial memory。MEM 用 video + language memory 部分替代了 spatial representation，但缺乏 3D geometric 信息。PROSPECT 和 GTA 从不同角度证明 explicit 3D 信息的价值。最优方案可能是二者结合。
   - 来源：[[2603-MEM]]、[[2507-MTU3D]]、[[2603-PROSPECT]]、[[2602-GTA]]

2. **Dense vs sparse 表示**：Dense feature maps 提供细粒度信息但计算成本高；sparse scene graphs 更高效但丢失几何细节。GTA 的 TSDF + topological graph 是一种层次化折中方案（dense geometry + sparse semantic graph）。
   - 来源：[[2210-VLMaps]]、[[2309-ConceptGraphs]]、[[2602-GTA]]

3. **Learned 3D features vs hand-crafted 3D representation**：PROSPECT 用 CUT3R 的 learned absolute-scale 3D features，GTA 用 TSDF 的 hand-crafted geometric representation。PROSPECT 在 supervised 上更强（60.3% vs 48.8%），但 GTA 的 geometric 严谨性保证了 action 的 physical validity（ray-casting on TSDF）。两条路线适用场景不同。
   - 来源：[[2603-PROSPECT]]、[[2602-GTA]]

## Open Questions

1. **Shared spatial representation for Nav+Manip**：如何设计一种空间表示，既能支持 building-scale navigation planning，又能提供 manipulation 所需的 object-level 3D geometry？详见 [[Ideas/SpatialToken-VLA]] 和 [[Ideas/Unified-GEM-Framework]]。
2. **Real-time incremental update**：真实部署中 spatial representation 需要实时增量更新。ConceptGraphs 目前是 offline batch processing，如何改造为 online incremental？PROSPECT 的 streaming 3D encoder（CUT3R ~4Hz）是初步探索。
3. **Language-grounded spatial querying**：如何让 VLA 的高层推理直接通过自然语言查询 spatial representation？GTA 的 BEV + coordinate grid visual prompting 提供了一种方案，但需要 360° 旋转扫描。
4. **Spatial memory 与 VLA memory 的融合**：MEM 的 video/language memory 和 spatial representation 如何统一？一个 agent 不应维护两套独立的记忆系统。MEM 缺乏 explicit 3D geometry，spatial representation 缺乏 temporal event memory。

## Known Dead Ends

1. **Linearized text memory 做空间表示**：GTA 证明 oversimplified text memory 无法提供 geometrically grounded 决策基础，是 MLLM "confident but incorrect" 决策的根源。需要 metric/geometric 表示。
   - 来源：[[2602-GTA]]
