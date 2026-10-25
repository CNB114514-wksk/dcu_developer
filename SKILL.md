---
name: dcu-developer-skills
description: 海光 DCU / ComfyUI / SCNet 平台开发技能合集（共 114 个 skill）的总索引。当任务涉及 DCU
  适配与量化推理、ComfyUI 运维与工作流、容器镜像与持久区、平台代理与网关、Skill 编写等场景时使用；本文件给出各领域子技能的位置与选用指引。
tasks: ""
---
# DCU 开发技能合集（114 个）

介绍：本skill包含了个人在平台里开发镜像时可能踩到的所有坑，可以帮agent一条龙配置好平台镜像所有基础环境

本仓库收集 DCU / ComfyUI 开发过程中沉淀下来的 skill。**每个子目录是一个独立 skill**，目录内的 `SKILL.md` 定义其适用场景与工作流，可选的 `scripts/`、`references/`、`templates/` 存放配套脚本与资料。本文件是总索引。

## 平台基础（★必读，先加载这段再干活）

> 跨任务的环境事实，踩错会浪费几小时；子 skill 的具体流程都以这里为前提。

### 网络与代理（按 zone 不同，绝不能写死）

- 平台**每个 zone 的 HTTP 代理地址不同**：本开发容器是 `10.1.100.10:3120`，无卡跳板机 / 部署机是 `10.15.100.43:3120`（均带凭据，并已注入 `http_proxy`/`https_proxy` 环境变量）。
- 取值必须 **env 优先 + 逐个探活**：`$SCNET_PROXY → $https_proxy → $http_proxy → 已知 zone 地址 → 直连`，3s TCP 探活选第一个可达的。**写死某个 IP 换 zone 必挂**（典型 `Errno 113 No route to host`；常表现为 git clone 能通、只有 LFS 对象下载挂）。
- git 也要走代理：`git -c http.proxy=$PROXY clone ...`；LFS 对象另走 CONNECT 隧道（见 `scnet-model-git-upload`）。
- 无外网时的兜底：在**有 egress 的容器**下好，再 `rsync` 到共享持久区（经 `ssh_connect_proxy.py` 实测 ~18 MB/s）。
- 排查与代码：`qwen38-flashnext-dcu-tuning/references/deploy-api.md` §「模型自动下载」。

### 硬件（海光 DCU，实测峰值——务必区分向量核 vs MMAC）

| 机型 | arch | CU | VRAM | MMAC FP16 | SIMT FP16 | 带宽 GB/s | 备注 |
|---|---|---|---|---|---|---|---|
| Z100SM | gfx906 | — | 16 GB | — | — | — | 入门 |
| Z100 | gfx906 | — | 32 GB | — | — | — | |
| K100_AI | gfx928 | 128 | 64 GB | **178** | 41 | 839 | 本项目主力 |
| K500SM_AI | gfx928 | 120 | 64 GB | **152** | 38 | 751 | |
| BW150 | gfx936 | 80 | 64 GB | **483** | 30.5 | 1339 | 矩阵核最强，FP64 真硬件 |

- **MMAC（矩阵核）比 SIMT（向量核）高 4~5×**；网上规格表里海光行多为**向量核**，别当矩阵峰值用。
- 远程执行/传文件统一用 `scripts/rmt.sh`；测 kernel 前先预热 GPU（空跑 ≥60 次）。

### 持久区（parastor）与镜像

- 持久区识别**只 source `persist_home.sh`**：notebook → `/root/private_data`；有卡容器 → `/work` 或 `/public/home/<user>`。**禁止自己 `findmnt` / 写死路径**。
- parastor 上 **mmap 缺页会间歇 hang**（像"慢"，实际读盘 0 MB/s）→ llama.cpp 用 `-lm none -lzm off`。
- **大文件只放持久区，别写镜像层**（撑爆镜像/盘满）；镜像层里删旧文件**不减体积**（只产生 whiteout）。

### 两台机器分工（SCNet 模型部署）

无卡跳板机（SSH，改镜像 / 看日志）→ 平台「保存镜像」→ 部署机（带卡，**只有 HTTP API，无 shell**）。部署机验收只能靠 HTTP 指纹，见 `qwen38-flashnext-dcu-tuning` §10 + `scripts/api_check.py`。

## 覆盖范围

| 领域 | 数量 | 内容 | 典型子技能 |
|---|---|---|---|
| ComfyUI / 模型推理与训练 | 29 | ComfyUI 运维排障、DCU 适配与量化推理、模型下载与工作流处理、MiniMax H3 视频提示词、评测与 vLLM/llama.cpp 部署 | `comfyui-attention-backends`、`comfyui-container-ram-debug`、`comfyui-core-upgrade` |
| 内容创作与可视化 | 19 | ComfyUI 工作流美化、信息图、架构图、ASCII/视频、Web 设计系统、音乐、手绘动画 | `architecture-diagram`、`ascii-art`、`ascii-video` |
| 软件开发与调试 | 12 | 系统化调试、TDD、Python/Node 调试、代码审查、精简重构、方案规划、Hermes 技能编写 | `comfyui-node-source-discovery`、`dogfood`、`hermes-agent-skill-authoring` |
| 办公与文档处理 | 11 | Word/Excel/PPT/PDF、OCR、Google 全家桶、Notion、Airtable、地图 | `airtable`、`google-workspace`、`hermes-docx` |
| 容器镜像与平台运维 | 12 | SCNet 容器环境、Docker 镜像手术/扁平化、持久区与 dsh/harness、网关转发、镜像打包 | `comfyui-image-packaging`、`docker-image-flatten`、`scnet-model-git-upload` |
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

合计 **114** 个 skill。

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
- 量化格式能不能用（FP8/FP4/int8/int4）、kernel 资源预算 → `dcu-quant-format-limits`
- DCU 上调 llama.cpp/llama-server（测法、NUMA、ub/ncmoe、MMAC）→ `dcu-llm-tuning`
- 推理性能诊断方法论（roofline/归因/A-B）→ `llm-inference-perf-playbook`
- ComfyUI 起不来 / 卡启动 / 插件报错 → `comfyui-ops`
- ComfyUI 内存或显存 OOM → `comfyui-memory-oom`
- 容器环境、持久区、代理与转发 → `scnet-container-env`、`scnet-gateway-forwarding`
- 镜像拉取/手术/扁平化 → `docker-image-ops`、`docker-image-layer-surgery`、`docker-image-flatten`

## 来源与适配说明

- **来源**：个人在 DCU 环境下开发适配各种大模型推理产生的skill，包含各种踩过的坑和解决方案
- **格式适配**：`SKILL.md` frontmatter 只保留 `name` / `description`；`triggers` 折进 `description`；正文相对路径改写为 `${CODEBUDDY_SKILL_DIR}/…`。
- **凭据清洗**：已清除写死的平台代理 URL（含真实口令），改为动态读取 `proxy_env.sh` / `$http_proxy`。
- **未收录**：`deepseek-unrestricted` 类别（越狱 prompt）。
- **改名**：`pdf`/`docx`/`xlsx` → `hermes-pdf`/`hermes-docx`/`hermes-xlsx`（避与官方 plugin 冲突）。

> 全部 114 个 skill 的逐条说明见 [`README.md`](README.md)。