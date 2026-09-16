---
name: dcu-developer-skills
description: 海光 DCU / ComfyUI / SCNet 平台开发技能合集（共 106 个 skill）的总索引。当任务涉及 DCU 适配与量化推理、ComfyUI 运维与工作流、容器镜像与持久区、平台代理与网关、Skill 编写等场景时使用；本文件给出各领域子技能的位置与选用指引。
---

# DCU 开发技能合集（106 个）

本仓库收集 DCU / ComfyUI 开发过程中沉淀下来的 skill。**每个子目录是一个独立 skill**，目录内的 `SKILL.md` 定义其适用场景与工作流，可选的 `scripts/`、`references/`、`templates/` 存放配套脚本与资料。本文件是总索引。

## 覆盖范围

| 领域 | 数量 | 内容 | 典型子技能 |
|---|---|---|---|
| ComfyUI / 模型推理与训练 | 25 | ComfyUI 运维排障、DCU 适配与量化推理、模型下载与工作流处理、MiniMax H3 视频提示词、评测与 vLLM/llama.cpp 部署 | `comfyui-attention-backends`、`comfyui-container-ram-debug`、`comfyui-core-upgrade` |
| 内容创作与可视化 | 17 | ComfyUI 工作流美化、信息图、架构图、ASCII/视频、Web 设计系统、音乐、手绘动画 | `architecture-diagram`、`ascii-art`、`ascii-video` |
| 软件开发与调试 | 12 | 系统化调试、TDD、Python/Node 调试、代码审查、精简重构、方案规划、Hermes 技能编写 | `comfyui-node-source-discovery`、`dogfood`、`hermes-agent-skill-authoring` |
| 办公与文档处理 | 11 | Word/Excel/PPT/PDF、OCR、Google 全家桶、Notion、Airtable、地图 | `airtable`、`google-workspace`、`hermes-docx` |
| 容器镜像与平台运维 | 10 | SCNet 容器环境、Docker 镜像手术/扁平化、持久区与 dsh/harness、网关转发、镜像打包 | `comfyui-image-packaging`、`docker-image-flatten`、`docker-image-layer-surgery` |
| 研究与信息检索 | 7 | arXiv 精读、论文写作、引用核验、事实核查、Polymarket、博客监控 | `arxiv`、`blogwatcher`、`grounded-citations` |
| AI Agent 协作 | 6 | Hermes 本体与自定义 provider、Claude Code/Codex/OpenCode 协同、桌面自动化 | `claude-code`、`codex`、`computer-use` |
| GitHub 工作流 | 6 | 认证、Issue、PR、代码审查、仓库管理、代码库勘察 | `codebase-inspection`、`github-auth`、`github-code-review` |
| Apple 生态 | 4 | 备忘录、提醒事项、iMessage、查找 | `apple-notes`、`apple-reminders`、`findmy` |
| 媒体处理 | 3 | YouTube 内容、GIF 检索、音频可视化 | `gif-search`、`songsee`、`youtube-content` |
| Hermes 自身 | 1 | 会话历史检索 | `hermes-session-history` |
| 邮件 | 1 | Himalaya 终端邮件 | `himalaya` |
| 笔记 | 1 | Obsidian 知识库 | `obsidian` |
| 智能家居 | 1 | Hue 灯光控制 | `openhue` |
| 社媒 | 1 | xurl / X 操作 | `xurl` |

合计 **106** 个 skill。

## 怎么用

1. 先按任务判断所属领域，在上表找到对应方向。
2. 进入该领域的子目录，读它的 `SKILL.md` —— 里面的「何时使用」会说明触发条件与执行步骤。
3. 需要脚本/模板时用子目录里的 `scripts/`、`templates/`；需要背景资料时看 `references/`。

```bash
git clone https://gitlab.scnet.cn:9002/skills/comfyui114514/dcu_developer.git
cp -rf dcu_developer/comfyui-ops ~/.codebuddy/skills/     # 按需拷单个 skill
```

## 几个入口推荐

- DCU 上模型跑得慢 / 量化选型 → `comfyui-dcu-performance`、`comfyui-dcu-quantized-inference`
- ComfyUI 起不来 / 卡启动 / 插件报错 → `comfyui-ops`
- ComfyUI 内存或显存 OOM → `comfyui-memory-oom`
- 容器环境、持久区、代理与转发 → `scnet-container-env`、`scnet-gateway-forwarding`
- 镜像拉取/手术/扁平化 → `docker-image-ops`、`docker-image-layer-surgery`、`docker-image-flatten`

## 来源与适配说明

- **来源**：个人在 DCU / ComfyUI 环境下使用 Hermes agent 期间自产的 skill 库。
- **格式适配**：`SKILL.md` frontmatter 只保留 `name` / `description`；`triggers` 折进 `description`；正文相对路径改写为 `${CODEBUDDY_SKILL_DIR}/…`。
- **凭据清洗**：已清除写死的平台代理 URL（含真实口令），改为动态读取 `proxy_env.sh` / `$http_proxy`。
- **未收录**：`deepseek-unrestricted` 类别（越狱 prompt）。
- **改名**：`pdf`/`docx`/`xlsx` → `hermes-pdf`/`hermes-docx`/`hermes-xlsx`（避与官方 plugin 冲突）。

> 全部 106 个 skill 的逐条说明见 [`README.md`](README.md)。
