# Research Workspace

AI 研究领域的知识管理工作区，基于 Obsidian。

## Structure

| Folder | Purpose | Naming |
|--------|---------|--------|
| `Papers/` | 论文笔记 | `AuthorYear-ShortTitle.md` |
| `Ideas/` | 研究灵感 | 自由命名 |
| `Projects/` | 项目追踪 | 项目名称 |
| `Topics/` | 主题综述 | 主题名称 |
| `Meetings/` | 会议记录 | `YYYY-MM-DD-Description.md` |
| `Daily/` | 每日日志 | `YYYY-MM-DD.md` |
| `Templates/` | Obsidian 模板 | — |
| `Attachments/` | 附件 | — |
| `Resources/` | AI Prompts 等参考资料 | — |

## Templates

使用 Obsidian 核心 Templates 插件（已配置好，模板文件夹为 `Templates/`）。

新建笔记后，`Ctrl/Cmd + P` → "Templates: Insert template" → 选择对应模板。

| Template | 用途 |
|----------|------|
| Paper | 论文笔记（含 Mind Map） |
| Idea | 研究灵感 |
| Project | 项目追踪 |
| Topic | 主题综述/文献对比 |
| Meeting | 会议记录 |
| Daily | 每日研究日志 |

## Tags

使用术语的通用写法，保持扁平：

- **领域**: `LLM`, `CV`, `RL`, `multimodal`, `diffusion`
- **方法**: `transformer`, `RLHF`, `distillation`, `RAG`
- **会议**（可选）: `NeurIPS`, `ICML`, `ICLR`, `ACL`, `CVPR`
- **任务**: `text-generation`, `image-classification`, `alignment`

## AI Workflow

1. 把论文关键词/链接发给 AI（Claude），附上 [AI-Prompts](Resources/AI-Prompts.md) 里的 prompt
2. AI 按 Paper 模板格式输出笔记
3. 粘贴到 Obsidian，阅读修改
4. 补充 frontmatter 元数据、双向链接和 tags

可通过 Claudian 插件在 Obsidian 内直接使用，或在外部 Claude 对话中使用。
