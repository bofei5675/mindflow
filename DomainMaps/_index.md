---
last_updated: "2026-04-01"
---

# Domain Map

> Human-AI 共同维护的核心认知地图。所有 Papers/Ideas/Experiments 的精华汇聚于此。
> 每个 domain 独立一个文件，便于精准维护和 skill 读写。

## Domains

| Domain | 文件 | 说明 | 最后更新 |
|:-------|:-----|:-----|:---------|
| VLA | [[DomainMaps/VLA\|VLA]] | Vision-Language-Action 模型 | 2026-04-01 |
| VLN | [[DomainMaps/VLN\|VLN]] | Vision-Language Navigation | 2026-04-01 |
| Spatial Representation | [[DomainMaps/SpatialRep\|SpatialRep]] | 语义 SLAM 与空间表示 | 2026-04-01 |
| World Models | [[DomainMaps/WorldModel\|WorldModel]] | World Models for Embodied AI | 2026-04-01 |

> 新增 domain：创建 `DomainMaps/{Name}.md`，使用 `Templates/DomainMap.md` 模板，在上表添加一行。

## Cross-Domain Insights

- **VLN-VLA 架构趋同**：两个领域都在向 hierarchical VLM reasoning + low-level policy execution 的架构收敛。NaVILA 直接将 VLN 重构为 navigation VLA；DM0 首次在同一框架中训练 navigation 和 manipulation。详见 [[VLN-VLA-Unification]]。
- **Spatial representation 是统一的关键瓶颈**：VLN 依赖显式空间表示（topological map、TSDF），VLA 通常无显式空间表示（end-to-end）。GTA 的 explicit world representation 和 PROSPECT 的 3D-2D fusion 证明显式 3D 信息对两者都有价值。3D scene graph 仍是最有潜力同时服务两者的表示形式。详见 [[DomainMaps/SpatialRep|SpatialRep]]。
- **Embodied Reasoning 是跨 domain 核心能力**：GRPO-based RL 已成为 embodied reasoning 的 de facto 训练范式（Robot-R1, Embodied-R, Embodied-R1, VLN-R1），in-domain reasoning data 远比 OOD 数据有效（VLASER），explicit spatial representation 大幅提升 navigation reasoning。详见 [[Topics/Embodied-Reasoning-Survey|Embodied Reasoning Survey]]。
- **World Model 正在成为 VLA 的 foundation**：DreamZero 和 Motus 证明 video generation 能力可直接转化为 motor control，zero-shot 泛化远超纯 VLA。World-VLA-Loop 验证 world model 可替代真实环境做 VLA RL training。World model 有望成为连接 VLN、VLA、Spatial Rep 三个 domain 的统一基础。详见 [[DomainMaps/WorldModel|WorldModel]]。
- **Data recipe 比 model scale 更重要**：跨所有 domain 的一致发现——SmolVLA（0.45B）> OpenVLA（7B），X-VLA（0.9B）6 benchmark SOTA，DM0（2B）> π0.5（3B）。训练策略（gradient decoupling、soft prompt、data pyramid）的杠杆效应远大于单纯 scaling。
