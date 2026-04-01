---
title: "Vision-Language-Action Models"
last_updated: "2026-04-01"
---
## Overview

用 Vision-Language Model 驱动 robot action generation 的统一框架。核心问题：如何将 VLM 的语义理解能力转化为精确的物理操作？研究意义在于实现 language-conditioned 的通用机器人控制。

**发展现状**：领域正从 "VLM + action head" 的简单范式向多维度分化演进。π₀ 系列确立了 flow matching + hierarchical inference 的标准架构；开源生态（Octo → OpenVLA → X-VLA → SmolVLA）快速迭代，小模型（0.45B–0.9B）已能匹敌甚至超越大模型（3B–7B）。2025–2026 年涌现四大趋势：（1）Embodied-Native 训练范式（DM0）挑战传统 Pretrain-then-Adapt；（2）World Action Model（DreamZero、Motus）将 video generation 与 action prediction 联合建模，zero-shot 泛化提升 2×+；（3）Self-correction 机制（CycleVLA、RoboClaw）解决 long-horizon 错误累积；（4）Action decomposition（DAM-VLA）实现 arm/gripper 解耦。但 real-world 大规模部署仍在早期，safety guarantee 和 fully autonomous self-improvement 几乎空白。

## Core Concepts

- **VLA (Vision-Language-Action Model)**: 接受视觉和语言输入、直接输出 robot action 的端到端模型
- **Flow Matching**: 基于连续 flow 的 action generation 方法，支持高频（50 Hz）连续控制，替代离散 token prediction
- **Action Expert**: VLA 架构中专门负责 action generation 的模块，与 VLM backbone 解耦
- **Hierarchical Inference**: 高层 VLM 做语义推理（subtask planning），低层 action expert 做运动生成
- **Cross-embodiment**: 跨不同机器人形态的数据共享与迁移学习
- **World Action Model (WAM)**: 联合预测 video frames 和 robot actions 的统一模型，通过 video generation 的物理理解提升 action prediction 泛化
- **Embodied-Native Training**: 从训练起点融合 physical data 作为一等公民，区别于传统 Pretrain-then-Adapt 范式
- **Gradient Decoupling**: Action expert 梯度不回传 VLM backbone，防止 embodied training 侵蚀语义能力
- **Self-Correction**: VLA 在执行过程中主动检测失败并回退重试的能力

## Established Knowledge

1. **Flow matching + action expert 是当前最强 action generation 范式**：连续 action 生成（flow matching）在控制频率和灵巧操作上全面超越 autoregressive token prediction。π₀ 系列确立了 VLM backbone + flow matching action expert 的标准架构，X-VLA 进一步验证 0.9B 参数即可达 SOTA。
   - 来源：[[2410-Pi0]]、[[2504-Pi05]]、[[2412-RoboVLMs]]、[[2510-XVLA]]

2. **Hierarchical inference（高层语义推理 + 低层 action 生成）是 long-horizon 任务的有效架构**：π0.5、Hi Robot、NaVILA、DM0 均验证了分层设计的优越性。Hi Robot 的独立 VLM reasoning + VLA execution 超越 GPT-4o 40%+。DM0 的 Spatial Scaffolding（subtask → bbox → trajectory → action）提供了 coarse-to-fine 的课程学习路径。
   - 来源：[[2504-Pi05]]、[[2502-HiRobot]]、[[2412-NaVILA]]、[[2602-DM0]]

3. **Fine-tuning 设计和数据策略比模型规模更重要**：OpenVLA-OFT（7B）达 97.1%（LIBERO），超越 π₀（3.3B）的 94.2%；SmolVLA（0.45B）超越 OpenVLA（7B）；X-VLA（0.9B）在 6 个 benchmark 上达 SOTA；DM0（2B）超越 GigaBrain（3B）和 π0.5（3B）。VLA 领域存在显著的过参数化，data recipe 和训练策略可能是更大的杠杆。
   - 来源：[[2502-OpenVLA-OFT]]、[[2506-SmolVLA]]、[[2510-XVLA]]、[[2602-DM0]]

4. **Data diversity >> Data specificity**：π0.5 的 97.6% 训练数据不来自目标任务，但 co-training 对泛化至关重要。Post-training 策略（先 cross-embodiment 预训练，再 in-domain fine-tune）优于直接 co-training。DreamZero 进一步证明多样化数据（500 hrs）优于重复性 demonstration。
   - 来源：[[2504-Pi05]]、[[2412-RoboVLMs]]、[[2602-DreamZero]]

5. **Soft prompt 是参数高效的 cross-embodiment 方案**：X-VLA 用 per-source learnable embedding 吸收平台异构性，仅调 ~1% 参数即可适配新 embodiment，在 6 个 benchmark 和 3 个真实平台达 SOTA。
   - 来源：[[2510-XVLA]]

6. **Gradient decoupling 有效防止 embodied training 侵蚀 VLM 能力**：DM0 验证了 action expert 梯度不回传 VLM 可在保持 multimodal understanding 的同时提升 manipulation 性能。π₀ 用参数隔离（MoE-style）、DM0 用梯度隔离，解决的是同一问题。
   - 来源：[[2602-DM0]]、[[2410-Pi0]]

7. **Arm/gripper 解耦提升精细操作**：DAM-VLA 将 arm movement 和 gripper manipulation 分为两个专用 diffusion head，配合 action routing 和 dual-scale weighting，在 SIMPLER 83%、real-world 86.8%，显著优于统一 action head 方法。
   - 来源：[[2603-DAMVLA]]

## Active Debates

1. **Embodied-Native vs Pretrain-then-Adapt，哪个范式更优？** DM0 主张从训练起点融合 physical data，2B 参数超越 π0.5（3B）。但两者对比存在 data recipe、数据量等 confounding factors，需要更多 controlled ablation。π₀ 系列的成功也说明 Pretrain-then-Adapt 并非不可行。
   - 来源：[[2602-DM0]]、[[2410-Pi0]]、[[2504-Pi05]]

2. **World Action Model vs 纯 VLA，哪个更 cost-effective？** DreamZero（14B WAM）在 zero-shot 上提升 2×+，但需 2× GB200 GPU；Motus（optical flow latent action）达 87% RoboTwin SR。而纯 VLA（X-VLA 0.9B）在 seen tasks 上也很强。WAM 的优势集中在 zero-shot 泛化，纯 VLA 在 seen tasks 上更高效。
   - 来源：[[2602-DreamZero]]、[[2512-Motus]]、[[2510-XVLA]]

3. **RL self-improvement vs 更多 demonstration，哪个更 cost-effective？** π\*₀.₆ 的 Recap 实现 >2× throughput 提升；World-VLA-Loop 用 world model 做 RL 环境提升 +12.7%（LIBERO）。但 VLASER 发现 in-domain reasoning data 远比 OOD 数据有效，更多高质量 demonstration 是否能达到同样效果未定。
   - 来源：[[2511-PiStar06]]、[[2602-WorldVLALoop]]、[[2510-VLASER]]

4. **Cross-embodiment 数据的最优使用方式**：RoboVLMs 发现 in-domain 数据比 cross-embodiment 数据更有效；但 π0.5 和 DreamZero 表明 cross-embodiment 对 open-world generalization 至关重要。X-VLA 的 soft prompt 方案提供了一种折中——共享 backbone 知识但保持 embodiment-specific adaptation。
   - 来源：[[2412-RoboVLMs]]、[[2504-Pi05]]、[[2510-XVLA]]、[[2602-DreamZero]]

5. **最优 action representation**：Flow matching 对 multimodal action distribution 建模更好但计算成本高；L1 regression 在 unimodal 场景下更简洁高效；optical flow 作为 latent action（Motus）支持无 action label 预训练。最优选择可能是 task-dependent。
   - 来源：[[2502-OpenVLA-OFT]]、[[2506-SmolVLA]]、[[2512-Motus]]

## Open Questions

1. **Navigation + Manipulation 统一架构**：如何在单一 VLA 中同时支持 building-scale navigation 和灵巧操作？DM0 是首次在同一框架中训练两者的尝试，但 navigation 仅在 simulation 中验证。详见 [[VLN-VLA-Unification]]。
2. **Fully autonomous self-improvement**：如何去除人工 reward labeling，实现 intrinsic motivation / self-supervised reward？World-VLA-Loop 的 joint reward prediction 是初步尝试，RoboClaw 的 EAP self-resetting 减少了人工干预但未消除。
3. **长期空间记忆**：MEM 解决了 15 分钟级记忆（video + language dual memory），但缺乏 explicit spatial memory。如何维护 persistent 的空间表示支持跨房间任务？详见 [[DomainMaps/SpatialRep|SpatialRep]]。
4. **Safety 与 failure recovery**：VLA 走向真实世界部署后，系统性的 safety guarantee 和 failure recovery 机制几乎空白。CycleVLA 的 proactive self-correction 是初步探索，但依赖可逆性假设且仅在 simulation 验证。
5. **VLA 的 scaling law**：模型规模、数据规模、任务多样性之间的 scaling 关系尚不清楚。SmolVLA/X-VLA 的成功暗示 VLA 的 scaling law 可能与 LLM 不同。DreamZero 的 14B >> 5B 又暗示在 WAM 范式下 scaling 依然重要。
6. **Self-correction 的通用化**：CycleVLA 的 subtask backtracking 在 LIBERO long-horizon 上 +39.9%，但依赖可逆操作和外部 VLM。如何为不可逆操作设计 self-correction？MBR decoding 作为 test-time scaling 策略值得更多探索。

## Known Dead Ends

1. **纯 autoregressive token prediction 做灵巧操作**：RT-2 的 ~3 Hz 控制频率已被 flow matching 的 50 Hz 全面超越。离散化 action tokenization 在精细操作中精度不足。但 autoregressive 仍可用于高层语义推理（如 π0.5 的 subtask prediction、DM0 的 Spatial Scaffolding）。
   - 来源：[[2307-RT2]]、[[2410-Pi0]]、[[2602-DM0]]

2. **GPT-4 zero-shot 做机器人控制**：π0.5 对比 GPT-4 zero-shot 高层规划仅 ~20% 成功率，纯 LLM 规划不足以驱动真实 robot。需要 fine-tuned VLM + 视觉 grounding。
   - 来源：[[2504-Pi05]]、[[2502-HiRobot]]

3. **统一 action head 处理异构动作**：DAM-VLA 证明 arm movement 和 gripper manipulation 的本质差异（path constraint、visual attention、数据分布）使得统一 action head 成为性能瓶颈。解耦设计（双头 diffusion）在 real-world 上提升 24%+。
   - 来源：[[2603-DAMVLA]]
