---
title: "World Models for Embodied AI"
last_updated: "2026-04-01"
---
## Overview

用 learned model 预测世界动态（visual observations、physical states），为 embodied agent 提供 planning、RL training 和 action generation 的基础。核心问题：如何将 video generation model 对物理世界的理解转化为可用的 robot control signal？

**发展现状**：领域正经历从 "world model for RL" 到 "world model as policy" 的范式转变。传统路线将 world model 作为 RL 训练环境（DIAMOND、World-VLA-Loop）；新兴的 World Action Model（WAM）范式（DreamZero、Motus、UWM）将 video prediction 与 action prediction 联合建模，直接产出 policy。DreamZero（14B）在 zero-shot 泛化上比纯 VLA 提升 2×+，验证了 video generation 能力到 motor control 的迁移。Motus 用 optical flow 作为 latent action 统一了 video generation、VL understanding 和 action prediction。UWM 通过 decoupled diffusion timesteps 实现了一个架构四种推理模式。但计算成本高（DreamZero 需 2× GB200 GPU for 7Hz control）、long-horizon 推理不足（~6.6 秒视觉记忆）、sub-centimeter 精度受限仍是核心挑战。

## Core Concepts

- **World Model**: 学习环境动态的 neural network，给定当前状态和动作预测未来状态
- **World Action Model (WAM)**: 联合预测 video frames 和 robot actions 的统一模型，video generation 和 action prediction 共享 denoising objective
- **Latent Action**: 从视频中提取的动作表示（如 optical flow），无需 ground-truth action labels 即可学习
- **Video World Model**: 以 video generation 为基础的 world model，隐式编码物理规律
- **Model-based RL**: 用 learned world model 替代真实环境进行 RL 训练，降低物理交互成本
- **Unified World Model**: 在单一架构中同时支持 policy inference、forward dynamics、inverse dynamics 和 video prediction

## Established Knowledge

1. **Video diffusion model 天然具备物理世界理解能力**：DreamZero 证明 video generation model 的 spatiotemporal dynamics 理解可直接迁移到 motor control，zero-shot 泛化到新任务比纯 VLA 提升 2×+。Cosmos 在 2M 小时视频数据上训练的 foundation model 为下游任务提供了物理 prior。
   - 来源：[[2602-DreamZero]]、[[2501-Cosmos]]

2. **Joint video-action prediction 优于分离训练**：DreamZero 端到端联合训练（共享 denoising objective）确保 video 和 action 紧密对齐；Motus 的 Tri-model Joint Attention 连接 video generation、VL understanding 和 action prediction 三个 expert。分离训练的 world model + inverse dynamics model 在 action following 精度上不如联合方案。
   - 来源：[[2602-DreamZero]]、[[2512-Motus]]、[[2504-UWM]]

3. **Latent action（optical flow）支持大规模无标注预训练**：Motus 用 optical flow 压缩为 14 维 latent action，桥接 visual dynamics 与 control signal，使模型可在无 action label 的 internet video 上预训练。Genie 通过 unsupervised latent action discovery 也验证了类似思路。
   - 来源：[[2512-Motus]]、[[2402-Genie]]

4. **Decoupled diffusion timesteps 自然统一多种推理模式**：UWM 为 action 和 observation 各自分配独立 noise schedule，单一架构即可实现 policy、forward dynamics、inverse dynamics、video prediction 四种推理模式，cotraining with action-free video 一致性提升性能。
   - 来源：[[2504-UWM]]

5. **World model 可替代真实环境进行 RL post-training**：World-VLA-Loop 用 state-aware video world model（joint reward prediction）替代物理环境做 VLA 的 RL 训练，LIBERO +12.7%，real-world +23.4%。关键创新是 near-success data 迫使 world model 关注 spatial dynamics 细粒度差异，以及 co-evolving 闭环机制。
   - 来源：[[2602-WorldVLALoop]]

6. **Diffusion 优于 autoregressive 做 model-based RL 的 world model**：DIAMOND 证明 diffusion world model 在 Atari 100k 上达到 1.46 HNS SOTA，visual details 对 model-based RL 至关重要，而 autoregressive model 容易丢失精细视觉信息。
   - 来源：[[2405-DIAMOND]]

## Active Debates

1. **WAM vs 纯 VLA，哪个是最终范式？** DreamZero（14B WAM）zero-shot 泛化 2×+ 于纯 VLA，但需 2× GB200 GPU for 7Hz。纯 VLA（X-VLA 0.9B）在 seen tasks 上更高效，DreamZero-Flash 通过解耦 video/action noise 实现 38× 加速但仍计算密集。WAM 在泛化上优势明显，纯 VLA 在效率上胜出。
   - 来源：[[2602-DreamZero]]、[[2510-XVLA]]

2. **Joint training vs modular composition**：DreamZero 和 Motus 主张端到端联合训练，DreamGen 走 world model + IDM modular 路线（先用 video model 生成轨迹，再用 IDM 提取 action）。DreamGen 更灵活但 action 精度受 IDM 限制。
   - 来源：[[2602-DreamZero]]、[[2512-Motus]]、[[2505-DreamGen]]

3. **World model 的最优 architecture**：Autoregressive diffusion transformer（DreamZero 14B）、Mixture-of-Transformers（Motus）、Unified diffusion transformer（UWM）三种架构各有优劣。DreamZero 最强但最大；Motus 最统一但依赖 optical flow；UWM 最优雅但 scale 未验证。
   - 来源：[[2602-DreamZero]]、[[2512-Motus]]、[[2504-UWM]]

4. **Data：internet video vs robot demonstration**：DreamZero 的 cross-embodiment transfer（12 min human video → 54.3% task progress）暗示 internet video 的巨大潜力，但仅在实验室规模验证。Motus 的 Data Pyramid 系统性整合多层数据源。能否 scale 到 internet-scale 尚未验证。
   - 来源：[[2602-DreamZero]]、[[2512-Motus]]

## Open Questions

1. **Long-horizon reasoning**：当前 WAM 本质是 "System 1" reactive model（DreamZero 最大 6.6 秒视觉记忆），复杂多步任务需要外部 planning system。如何让 world model 具备 long-horizon planning 能力？RAP 用 LLM as world model + MCTS 是一种思路。
2. **Sub-centimeter precision**：WAM 因优先训练广度（多样化任务）而非密度（重复 demonstration），在高精度装配等任务上受限。如何在保持泛化的同时提升精度？
3. **World model 的 scaling law**：DreamZero 14B >> 5B 暗示 scaling 有效，但缺乏系统性的 WAM-specific scaling law 研究（不同于 LLM scaling law）。
4. **Autoregressive video quality drift**：World-VLA-Loop 指出超过 200 frames 后 video quality 下降，限制了 world model 在 long-horizon 场景的应用。如何维持长程生成质量？
5. **World model 与 spatial representation 的融合**：当前 world model 生成的是 2D video，缺乏 explicit 3D spatial structure。OccSora（occupancy prediction）和结构化 3D/4D 模型是探索方向。

## Known Dead Ends

1. **外部 VLM 做 world model 的 reward signal**：World-VLA-Loop 证明外部 VLM（Qwen3-VL）的 reward alignment 仅 50-55%，远低于 joint reward prediction 的 75-95%。World model 必须内化 reward prediction 而非依赖外部评估。
   - 来源：[[2602-WorldVLALoop]]

2. **Pixel-space supervision 做 predictive learning**：PROSPECT 指出在 explicit pixel space 做 video prediction 容易过拟合到与 navigation 无关的视觉细节。在 frozen teacher 的 latent space 做 prediction 更有效。
   - 来源：[[2603-PROSPECT]]
