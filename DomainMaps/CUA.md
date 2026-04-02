---
title: "Computer-Use Agents"
last_updated: "2026-04-02"
---
## Overview

让 AI agent 在数字设备（桌面、手机、浏览器）上根据自然语言指令自主完成复杂任务。核心问题：如何将 VLM 的视觉理解能力转化为精确的 GUI 操作？研究意义在于实现通用的数字自动化——任何人类能用电脑完成的任务，agent 都能代劳。

**发展现状**：2024–2025 年经历爆发式增长。OSWorld SOTA 从 12.24%（2024.04）飙升至 72.6%（2025.10，超人类 72.36%）。技术路线从 framework-based（prompt + 工具链包裹商用模型）转向 native agent（端到端 VLM + RL 训练）。产业界深度参与：Anthropic Computer Use、OpenAI Operator/CUA、ByteDance UI-TARS、Google Mariner。RL 已成为训练的 de facto 范式，data synthesis pipeline 正在解决数据瓶颈（成本降至 $0.55/trajectory）。但 safety/reversibility、跨应用 workflow、专业 icon grounding 仍是核心未解问题。

**与 VLA/Embodied AI 的桥梁**：CUA 和 VLA 共享核心技术栈（VLM backbone + action prediction + RL training），CUA 的 grounding/planning/error recovery 方法可能迁移到物理世界。Entropulse、data flywheel、reflective CoT 等训练范式具有跨域价值。

## Core Concepts

- **GUI Grounding**: 将自然语言指令映射到屏幕像素坐标的能力，是 agent 执行操作的前提。分为 text grounding（文字元素）和 icon grounding（图标/控件）
- **Native Agent**: 端到端训练的 VLM，仅以截图为输入直接输出动作，无需 prompt engineering 或外部工具链。代表：UI-TARS、ComputerRL
- **Agent Framework**: 模块化组合方案，将规划、grounding、执行分给不同模型/工具。代表：Agent S2/S3
- **Set-of-Mark (SoM)**: 在截图上叠加数字标签标注可交互区域，帮助 VLM 定位元素
- **Behavior Narrative**: 将多模态 agent 轨迹压缩为结构化文本摘要，用于 test-time 轨迹评估和选择
- **Execution-based Evaluation**: 不检查 agent 操作步骤，只验证任务完成后的系统状态，允许不同路径达到同一目标
- **Data Flywheel**: 模型生成的轨迹按质量分流（低质量→pretrain，高质量→SFT，可验证→RL），形成自增强循环
- **Entropulse**: RL/SFT 交替训练策略，解决 RL 训练中的 entropy collapse 问题
- **API-GUI Paradigm**: Agent 同时掌握 GUI 操作和程序化 API 调用，兼顾通用性和效率
- **Reflective CoT**: 三层结构化推理（观察→反思→动作），包含 error detection 和 recovery

## Established Knowledge

1. **GUI grounding 是 visual agent 的核心能力瓶颈**：SeeClick 首次明确这一发现，ScreenSpot-Pro 进一步揭示专业软件 icon grounding 仅 4%。Grounding 能力与下游 agent 任务性能呈强正相关。但瓶颈正在被快速攻克——GroundNext-3B 在 agentic 任务上已超越 72B 模型。
   - 来源：[[Papers/2401-SeeClick|SeeClick]]、[[Papers/2504-ScreenSpotPro|ScreenSpot-Pro]]、[[Papers/2511-GroundCUA|GroundCUA]]

2. **方法创新 > 模型规模**：ComputerRL 9B 超越 o3，GroundNext-3B 超越 72B agentic agent，ShowUI 2B 接近 7B 模型（256K 数据），OS-Atlas 7B 超越 GPT-4o。数据质量、训练策略和架构设计的杠杆效应远大于 scaling。
   - 来源：[[Papers/2508-ComputerRL|ComputerRL]]、[[Papers/2511-GroundCUA|GroundCUA]]、[[Papers/2411-ShowUI|ShowUI]]、[[Papers/2410-OSAtlas|OS-Atlas]]

3. **RL 训练一致提升 GUI agent 性能**：UI-TARS-2 (PPO)、ComputerRL (step-level GRPO + Entropulse)、DART-GUI (解耦异步 PPO)、GroundCUA (RLOO) 均验证 RL 在不同框架中带来一致提升。核心难点是 entropy collapse 和 multi-turn credit assignment。
   - 来源：[[Papers/2509-UITARS2|UI-TARS-2]]、[[Papers/2508-ComputerRL|ComputerRL]]、[[Papers/2509-DARTGUI|DART-GUI]]、[[Papers/2511-GroundCUA|GroundCUA]]

4. **数据质量 > 数据规模**：GroundCUA 用 700K 高质量样本胜过 9M 自动采集数据；OpenCUA 的 reflective CoT 比无反思版本提升 32%。数据的结构化程度和标注密度比规模更重要。
   - 来源：[[Papers/2511-GroundCUA|GroundCUA]]、[[Papers/2508-OpenCUA|OpenCUA]]

5. **Training-time 和 Test-time scaling 正交互补**：RL training scaling（UI-TARS-2、ComputerRL）和 test-time compute scaling（Agent S3 BJudge）可独立提升性能，组合后 OSWorld 达 72.6% 超人类。
   - 来源：[[Papers/2510-ScalingAgents|Agent S3]]、[[Papers/2509-UITARS2|UI-TARS-2]]、[[Papers/2508-ComputerRL|ComputerRL]]

6. **Execution-based evaluation 是正确范式**：WebArena 首创、OSWorld 确立的 functional correctness 评测，优于 action matching，已被全领域采纳。
   - 来源：[[Papers/2307-WebArena|WebArena]]、[[Papers/2404-OSWorld|OSWorld]]

7. **感知 pipeline 质量是被低估的性能因素**：WindowsAgentArena 发现 SoM 标注质量造成 15-57% 性能波动；OmniParser 证明 local semantics（icon 功能描述）提升 23.3%。感知质量对最终性能的影响可能大于 reasoning 能力。
   - 来源：[[Papers/2409-WindowsAgentArena|WindowsAgentArena]]、[[Papers/2408-OmniParser|OmniParser]]

8. **GUI agent RL 的瓶颈在系统效率而非算法**：DART-GUI 通过工程优化将环境利用率从 12.2% 提升至 67.7%（5.5×），训练吞吐 1.9×，7B 模型即超越 Claude-4-Sonnet。系统效率的杠杆可能大于算法改进。
   - 来源：[[Papers/2509-DARTGUI|DART-GUI]]

## Active Debates

1. **Native Agent vs Compositional Framework，哪个路线更优？**
   Native agent（UI-TARS-2 47.5%、ComputerRL 48.9%）在单次执行上更强；compositional framework（Agent S3 72.6%）通过 test-time scaling 超人类。但 Agent S3 依赖 GPT-5+Opus 4.5 做 judge，Native agent 成本更低。最优方案可能是 RL-trained native agent + test-time scaling。
   - 支持 Native: [[Papers/2501-UITARS|UI-TARS]]、[[Papers/2508-ComputerRL|ComputerRL]]
   - 支持 Compositional: [[Papers/2510-ScalingAgents|Agent S3]]、[[Papers/2504-AgentS2|Agent S2]]
   - **status**: unresolved — 两者正交可组合

2. **纯视觉 vs 结构化信息（HTML/DOM/a11y tree），哪种观察模式更优？**
   纯视觉（screenshot-only）更通用、跨平台，CogAgent 首次在 Mind2Web 上超越 HTML 方法。但 Agent S2 的 structural grounding expert 在 spreadsheet 场景仍有优势。OS Agents Survey 指出趋势从 textual 向 visual 迁移，但 dual approach 可能最优。
   - 支持纯视觉: [[Papers/2312-CogAgent|CogAgent]]、[[Papers/2501-UITARS|UI-TARS]]
   - 支持混合: [[Papers/2504-AgentS2|Agent S2]]、[[Papers/2508-OSAgentsSurvey|OS Agents Survey]]
   - **status**: leaning-visual — 趋势明确但特定场景仍需结构化信息

3. **Hierarchical planning vs Flat planning？**
   Agent S2 用 Manager-Worker 层级 + proactive planning 获得一致提升。但 Agent S3 发现 flat planning 替代 hierarchical 可减少 52.3% LLM 调用和 62.4% 延迟，且性能更好。可能层级规划在中等复杂度有效，超长 horizon 反而引入噪声。
   - 支持 Hierarchical: [[Papers/2504-AgentS2|Agent S2]]
   - 支持 Flat: [[Papers/2510-ScalingAgents|Agent S3]]
   - **status**: leaning-flat — 但需更多 benchmark 验证

4. **人工标注 vs 自动合成数据，哪种更 cost-effective？**
   OpenCUA 人工标注 22.6K trajectories → 45.0%；AgentTrek 自动合成 10.4K → 性能提升但未达同等水平。GroundCUA 证明高质量人工标注（700K）胜过 9M 自动数据。但 TongUI 143K 自动合成在 ScreenSpot 达 83.4%。Cost-quality trade-off 取决于目标任务的复杂度。
   - 支持人工: [[Papers/2508-OpenCUA|OpenCUA]]、[[Papers/2511-GroundCUA|GroundCUA]]
   - 支持自动: [[Papers/2412-AgentTrek|AgentTrek]]、[[Papers/2504-TongUI|TongUI]]、[[Papers/2412-OSGenesis|OS-Genesis]]
   - **status**: unresolved — 取决于任务复杂度和目标

## Open Questions

- **Safety & Reversibility**：如何在 agent 自主操作环境时防止不可逆损害？需要 undo 机制、sandbox isolation、action 确认的系统级方案。几乎所有论文都未充分讨论 — related: [[Topics/ComputerUseAgents-Survey]]
- **跨应用 Workflow**：Agent S2 在 Workflow 类任务仅 18.21%，需要跨应用状态追踪和长程 context 维护，可能需要 explicit memory / world model — related: [[Papers/2504-AgentS2|Agent S2]]
- **专业 Icon Grounding**：ScreenSpot-Pro 揭示 icon 识别仅 4%，需要 domain-specific 预训练数据或检索增强 — related: [[Papers/2504-ScreenSpotPro|ScreenSpot-Pro]]
- **CUA→Embodied 迁移**：Entropulse、data flywheel、reflective CoT 是否适用于 VLA 训练？GUI grounding 技术能否迁移到 embodied scene grounding？ — related: [[DomainMaps/VLA|VLA]]
- **Training-time vs Test-time Scaling 的 Pareto Frontier**：两种 scaling 路径的最优组合？ — related: [[Papers/2510-ScalingAgents|Agent S3]]
- **Agent 自主终止判断**：WebArena 发现 54.9% 可行任务被错误判为不可能，agent 缺乏 self-monitoring 能力 — related: [[Papers/2307-WebArena|WebArena]]

## Known Dead Ends

- **纯 prompt engineering + 商用模型**：依赖手工设计的 prompt 和 workflow，泛化性差、无法自我改进。UI-TARS 系统性证明 native agent 全面优于 framework-based 方案 — why: 被 native agent 路线全面超越，OSWorld 从 ~12% 提升到 48%+
- **Action matching 评测**：仅比较 agent 执行路径与参考路径的相似度，无法处理多路径达到同一目标的情况 — why: WebArena/OSWorld 的 execution-based evaluation 已成为共识标准
- **低分辨率输入（<1080p）**：ScreenSpot-Pro 证明高分辨率对专业场景至关重要，低分辨率截图丢失大量细粒度 UI 信息 — why: 专业软件中目标元素相对尺寸极小，低分辨率无法定位
- **Token merging for GUI**：ShowUI 发现 token merging（42.3%）远差于 token selection（70.4%），因为 GUI grounding 强依赖 positional encoding，merging 破坏了位置信息 — why: GUI 元素定位本质上是空间任务，位置信息不可压缩
