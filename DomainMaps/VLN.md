---
title: "Vision-Language Navigation"
last_updated: "2026-04-01"
---
## Overview

通过自然语言指令引导 agent 在真实或模拟环境中导航。核心问题：如何将语言指令 grounding 到可执行的导航策略？与 VLA 趋同的关键在于 hierarchical planning 和 spatial representation。

**发展现状**：R2R-CE SOTA 从 2022 年 ~50% SR 提升至 2026 年 65%+（supervised）/ 60%+（streaming VLA）。领域沿三条路线分化：（1）Graph-based 方法持续优化（ETPNav → Efficient-VLN，64.2% R2R-CE SR）；（2）Streaming VLA 路线快速崛起（StreamVLN → PROSPECT，60.3% SR），将 VLN 重构为连续控制问题；（3）Zero-shot MLLM-based 方法（GTA，48.8% SR）通过 explicit world representation 大幅缩小与 supervised 的差距。GRPO-based RL（VLN-R1）成为新的训练范式。真实机器人部署案例增多（NaVILA 88% SR、PROSPECT 多场景验证、GTA 跨 embodiment transfer），但大多限于短距室内导航。

## Core Concepts

- **VLN (Vision-Language Navigation)**: 给定自然语言指令，agent 在视觉环境中导航到目标位置
- **Topological Map**: 以节点（viewpoint）和边（connectivity）表示环境的图结构，是 VLN 的主流空间表示
- **Nav-graph vs Continuous**: 离散导航图（teleport between viewpoints）与连续动作空间（low-level velocity control）两种评测设定
- **Mid-level Action**: 介于高层语义规划和低层运动控制之间的动作抽象（如 "turn left and go to the door"）
- **Streaming VLA**: 将 VLN 重构为连续视觉流处理 + 实时动作输出的端到端框架，无需预建图
- **Explicit World Representation (EWR)**: 为 MLLM 提供 geometric grounding 的显式空间表示（TSDF、semantic map 等），解耦 spatial estimation 和 semantic planning

## Established Knowledge

1. **Topological map 是 VLN 的核心空间表示**：从 DUET 的 dual-scale graph transformer 到 ETPNav 的 online topological mapping，topological map 作为 intermediate spatial representation 被广泛验证有效。GTA 进一步将 topological graph 与 TSDF volumetric map 结合，实现更精确的 geometric grounding。
   - 来源：[[2202-DUET]]、[[2304-ETPNav]]、[[2602-GTA]]

2. **Hierarchical planning（high-level waypoint + low-level control）是 continuous VLN 的有效范式**：ETPNav 在 R2R-CE 上大幅超越 prior SOTA，这一架构与 VLA 的 hierarchical inference 高度平行。GTA 的 decoupled architecture（spatial estimation + semantic planning）将此推向极致。
   - 来源：[[2304-ETPNav]]、[[2412-NaVILA]]、[[2602-GTA]]

3. **VLN 可以重构为 navigation-focused VLA**：NaVILA 将 VLM 微调为 navigation VLA，用语言化 mid-level action 桥接高层规划和低层运动控制，R2R-CE 54% SR，real-world 88% SR。PROSPECT 进一步验证 streaming VLA 路线在 continuous VLN 中的竞争力（60.3% SR）。
   - 来源：[[2412-NaVILA]]、[[2603-PROSPECT]]

4. **YouTube 视频是 scalable 的 real-world navigation 数据源**：NaVILA 用 2000 个 YouTube egocentric touring videos 生成 20000 条 training trajectories，有效缓解了 robot navigation 数据稀缺问题。AirNav 用 143K UAV 导航样本验证了大规模数据在 UAV VLN 中的价值。
   - 来源：[[2412-NaVILA]]、[[2601-AirNav]]

5. **3D spatial feature 融合 2D semantic feature 显著提升 continuous VLN**：PROSPECT 用 CUT3R（3D spatial）+ SigLIP（2D semantic）cross-attention fusion，在 R2R-CE 达 60.3% SR；GTA 用 TSDF + topological graph 在 zero-shot 达 48.8% SR、SPL +16.4。显式 3D 建模对 long-horizon 任务尤其有效。
   - 来源：[[2603-PROSPECT]]、[[2602-GTA]]

6. **GRPO-based RL 是 VLN 的有效训练范式**：VLN-R1 将 DeepSeek-R1 的 GRPO 方法引入 VLN，配合 time-decayed reward，在 continuous action space 中取得改进，与 embodied reasoning 领域（Robot-R1、Embodied-R1）的趋势一致。
   - 来源：[[2506-VLNR1]]、[[Topics/Embodied-Reasoning-Survey]]

7. **Explicit world representation 大幅提升 zero-shot MLLM 导航**：GTA 证明为 MLLM 提供 metric world representation（TSDF + BEV visual prompting + coordinate grid）可将 zero-shot VLN 从 41.0% 提升到 48.8% SR，且 SPL 提升更为显著（+16.4）。EWR 作为 plug-in 可提升多种 existing methods。
   - 来源：[[2602-GTA]]

## Active Debates

1. **Discrete nav-graph vs continuous action space**：传统 VLN 在 discrete nav-graph 上性能更高（DUET R2R 72% SR），但 continuous 环境更接近真实场景。Streaming VLA（PROSPECT 60.3%）正在缩小差距，但 graph-based 方法（Efficient-VLN 64.2%）仍有优势。两种范式是否会收敛？
   - 来源：[[2202-DUET]]、[[2304-ETPNav]]、[[2603-PROSPECT]]、[[2512-EfficientVLN]]

2. **显式空间表示 vs end-to-end**：VLN 传统依赖 hand-designed spatial representation（topological map），streaming VLA 趋向 end-to-end。GTA 和 PROSPECT 代表两种折中路线：GTA 用 TSDF 做 explicit geometric grounding，PROSPECT 用 learned 3D features 做 latent spatial encoding。哪种更 scalable？
   - 来源：[[2602-GTA]]、[[2603-PROSPECT]]、[[2507-MTU3D]]

3. **Zero-shot MLLM vs supervised specialist**：GTA 的 zero-shot 48.8% SR 已接近部分 supervised 方法，且随 MLLM 能力提升自然 scaling（37.2% → 47.2%）。但与 Efficient-VLN 的 64.2% 仍有 15%+ 差距。长期看 MLLM 路线是否会胜出？
   - 来源：[[2602-GTA]]、[[2512-EfficientVLN]]

## Open Questions

1. **Building-scale long-horizon navigation**：现有评测多为单层室内短距导航，跨楼层、跨建筑的长距导航缺乏研究。LHVLN 的 3,260 条长程任务是初步探索，PROSPECT 在 >=100 步任务上 +4.14% SR 表明 predictive representation 有价值。
2. **Sim-to-real gap**：GTA 和 PROSPECT 的 real-world 实验增加了信心，但评估规模仍小（30–50 trials）。GTA 的跨 embodiment（wheeled + aerial）transfer 是亮点，但极端条件（夜间、户外）仍有挑战。
3. **VLN 与 VLA 的 shared spatial representation**：导航过程中构建的空间表示能否直接服务到达目的地后的 manipulation？详见 [[DomainMaps/SpatialRep|SpatialRep]]。
4. **Physical embodiment grounding**：VLN-PE 发现不同机器人形态对导航策略有显著影响，multi-robot co-training 比单一形态更有效。如何在 VLN 中显式建模 physical embodiment？

## Known Dead Ends

1. **GPT-4 zero-shot navigation reasoning**：NavGPT 证明纯文本 LLM（无视觉 grounding）的 zero-shot navigation 性能远低于 trained models。但 GTA 证明给 MLLM 提供 explicit geometric grounding 后性能大幅提升，问题不在 LLM 推理能力而在缺乏 spatial information。
   - 来源：[[2305-NavGPT]]、[[2602-GTA]]

2. **Oversimplified text memory 做空间表示**：GTA 指出 linearized text memory 无法提供 geometrically grounded 的决策基础，是 "confident but incorrect" 决策的根源。需要 metric/geometric 表示。
   - 来源：[[2602-GTA]]
