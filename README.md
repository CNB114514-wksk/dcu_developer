# DCU 开发 Skill 集

个人在开发 DCU / ComfyUI 环境下沉淀下来的 skill 合集，共 **106 个**，
覆盖 ComfyUI 与模型推理、容器镜像与平台运维、内容创作、软件开发、办公文档、研究检索等方向。

> 每个 skill 是一个独立目录，目录内 `SKILL.md` 定义该技能的适用场景与工作流，
> 可选的 `scripts/`、`references/`、`templates/` 存放配套脚本、参考文档与模板。

## 覆盖范围

| 领域 | 数量 | 主要内容 |
|---|---|---|
| ComfyUI / 模型推理与训练 | 25 | ComfyUI 运维、DCU 适配、量化推理、模型与工作流处理、MiniMax H3 视频提示词 |
| 内容创作与可视化 | 17 | ComfyUI 工作流美化、信息图、图表、ASCII/视频、Web 设计、音乐、手绘动画 |
| 软件开发与调试 | 12 | 系统化调试、TDD、Python/Node 调试、代码审查、精简重构、方案规划 |
| 办公与文档处理 | 11 | Word/Excel/PPT/PDF、OCR、Google 全家桶、Notion、Airtable、地图 |
| 容器镜像与平台运维 | 10 | SCNet 平台环境、Docker 镜像手术、持久区、网关转发、镜像打包 |
| 研究与信息检索 | 7 | arXiv、论文写作、引用核验、事实核查、Polymarket、博客监控 |
| AI Agent 协作 | 6 | Hermes 本体与自定义 provider、Claude Code/Codex/OpenCode、桌面自动化 |
| GitHub 工作流 | 6 | 认证、Issue、PR、代码审查、仓库管理、代码库勘察 |
| Apple 生态 | 4 | 备忘录、提醒事项、iMessage、查找 |
| 媒体处理 | 3 | YouTube 内容、GIF 检索、音频可视化 |
| Hermes 自身 | 1 | 会话历史检索 |
| 邮件 | 1 | Himalaya 终端邮件 |
| 笔记 | 1 | Obsidian 知识库 |
| 智能家居 | 1 | Hue 灯光控制 |
| 社媒 | 1 | xurl / X 操作 |

共 106 个 skill。

## ComfyUI / 模型推理与训练（25）

| skill | 说明 |
|---|---|
| `comfyui-attention-backends` | ComfyUI flash_attn/SolAttn import 失败或 attention 后端需修复时用。 |
| `comfyui-container-ram-debug` | ComfyUI 二次运行内存暴涨时用。容器 psutil 不感知 cgroup 的排查与修复。 |
| `comfyui-core-upgrade` | 升级 ComfyUI 核心版本且保留平台魔改时用：盘点、备份、逐文件重打补丁、依赖装 /usr/local。 |
| `comfyui-dcu-node-porting` | ComfyUI 自定义节点在海光 DCU/DTK 上移植适配（跑不了/悄悄 fallback 时用）。 |
| `comfyui-dcu-performance` | DCU 上 ComfyUI 推理慢/量化选型/算子后端诊断：int8 GEMM 不支持、fp16≈fp32。 |
| `comfyui-dcu-quantized-inference` | Use when choosing quantized model dtype on Hygon DCU. |
| `comfyui-memory-oom` | ComfyUI RAM/VRAM OOM 或二次运行内存暴涨时用：缓存诊断与 --cache 参数决策；闪退无 Traceback 查 dmesg OOM；flash_attn/diffusers 兼容坑与镜像保存前模板同步也归此。 |
| `comfyui-ops` | ComfyUI 起不来/卡启动/插件报错时用：诊断、禁用插件、批量清理、hf-mirror 下载。 |
| `comfyui-workflow-json-editing` | 美化/编辑 ComfyUI 工作流 JSON：布局分组、SetNode/GetNode 虚拟连线改造、一致性校验。 |
| `comfyui-workflow-model-audit` | 导入外部 ComfyUI 工作流时校对模型引用与本地文件，修正名字不匹配。 |
| `evaluating-llms-harness` | lm-eval-harness: benchmark LLMs (MMLU, GSM8K, etc.). |
| `huggingface-hub` | HuggingFace hf CLI: search/download/upload models, datasets. |
| `llama-cpp` | llama.cpp local GGUF inference + HF Hub model discovery. |
| `minimax-h3-3d-animation` | 将一句话创意转化为完整风格化3D动画短片的制作流程。含中文版SKILL.cn.md。 |
| `minimax-h3-brand-promo` | 将品牌信息转化为精美宣传短片的制作流程，含产品能力展示、使用场景和行动号召。 |
| `minimax-h3-coop-game` | 创建双人合作游戏菜单或开场动画的制作流程，含角色卡、UI排版和互动动效。 |
| `minimax-h3-handdrawn-live` | 创作融合手绘线条动画与实景空间的超现实短视频。 |
| `minimax-h3-music-video` | 为音乐人/视频创作者制作AI音乐MV或带歌词排版的情绪短片。 |
| `minimax-h3-paper-collage` | 用纸拼贴风格为叙述、知识观点或抽象主题制作视觉化解释视频。 |
| `minimax-h3-papercraft` | 用触感手工纸艺视觉解释科学/教育/通识知识，含纸角色、分层立体场景和道具。 |
| `minimax-h3-product-ad` | 将产品图片和广告需求转化为极简主义产品广告短片，含节拍同步排版和分镜。 |
| `minimax-h3-prompt-writing` | Write MiniMax H3 video prompts for all 5 generation modes. |
| `minimax-h3-wenxi-wuxi` | MiniMax H3 中文提示词：文戏对话模板或武戏战斗模板。 |
| `serving-llms-vllm` | vLLM: high-throughput LLM serving, OpenAI API, quantization. |
| `weights-and-biases` | W&B: log ML experiments, sweeps, model registry, dashboards. |

## 内容创作与可视化（17）

| skill | 说明 |
|---|---|
| `architecture-diagram` | Dark-themed SVG architecture/cloud/infra diagrams as HTML. |
| `ascii-art` | ASCII art: pyfiglet, cowsay, boxes, image-to-ascii. |
| `ascii-video` | ASCII video: convert video/audio to colored ASCII MP4/GIF. |
| `baoyu-infographic` | Infographics: 21 layouts x 21 styles (信息图, 可视化). |
| `claude-design` | Design one-off HTML artifacts (landing, deck, prototype). |
| `comfyui` | Generate images, video, and audio via diffusion workflows. |
| `comfyui-workflow-beautify` | 美化ComfyUI工作流JSON布局/分组/虚拟连线时用。改副本不动原文件。 |
| `design-md` | Author/validate/export Google's DESIGN.md token spec files. |
| `excalidraw` | Hand-drawn Excalidraw JSON diagrams (arch, flow, seq). |
| `humanizer` | Humanize text: strip AI-isms and add real voice. |
| `manim-video` | Manim CE animations: 3Blue1Brown math/algo videos. |
| `p5js` | p5.js sketches: gen art, shaders, interactive, 3D. |
| `popular-web-designs` | 54 real design systems (Stripe, Linear, Vercel) as HTML/CSS. |
| `pretext` | Build creative browser demos with DOM-free text layout. |
| `sketch` | Throwaway HTML mockups: 2-3 design variants to compare. |
| `songwriting-and-ai-music` | Songwriting craft and Suno AI music prompts. |
| `touchdesigner-mcp` | Control TouchDesigner via twozero MCP. |

## 软件开发与调试（12）

| skill | 说明 |
|---|---|
| `comfyui-node-source-discovery` | Find missing ComfyUI node sources and fix workflow bugs. |
| `dogfood` | Exploratory QA of web apps: find bugs, evidence, reports. |
| `hermes-agent-skill-authoring` | Author in-repo SKILL.md files: frontmatter and structure. |
| `inspecting-hermes-desktop-dom` | Read the live Hermes desktop DOM/CSS over CDP. |
| `node-inspect-debugger` | Debug Node.js via --inspect + Chrome DevTools Protocol CLI. |
| `plan` | Write a markdown plan to .hermes/plans/; no execution. |
| `python-debugpy` | Debug Python: pdb REPL + debugpy remote (DAP). |
| `requesting-code-review` | Pre-commit review: security scan, quality gates, auto-fix. |
| `simplify-code` | Parallel 4-agent cleanup of recent code changes. |
| `spike` | Throwaway experiments to validate an idea before build. |
| `systematic-debugging` | 4-phase root cause debugging: understand bugs before fixing. |
| `test-driven-development` | TDD: enforce RED-GREEN-REFACTOR, tests before code. |

## 办公与文档处理（11）

| skill | 说明 |
|---|---|
| `airtable` | Airtable REST API via curl. Records CRUD, filters, upserts. |
| `google-workspace` | Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python. |
| `hermes-docx` | Create, read, edit Word .docx documents and templates. |
| `hermes-pdf` | Create, merge, split, fill, and secure PDF files. |
| `hermes-xlsx` | Create, read, edit Excel .xlsx spreadsheets and CSVs. |
| `maps` | Geocode, POIs, routes, timezones via OpenStreetMap/OSRM. |
| `nano-pdf` | Edit text in existing PDFs via natural-language prompts. |
| `notion` | Notion API + ntn CLI: pages, databases, markdown, Workers. |
| `ocr-and-documents` | Extract text from PDFs/scans (pymupdf, marker-pdf). |
| `powerpoint` | Create, read, edit .pptx decks, slides, notes, templates. |
| `teams-meeting-pipeline` | Teams meeting summaries, job replay, Graph subscriptions. |

## 容器镜像与平台运维（10）

| skill | 说明 |
|---|---|
| `comfyui-image-packaging` | ComfyUI 模板区/持久区同步与镜像打包分享：rsync 方案、user 目录取舍、工作流密钥扫描。 |
| `docker-image-flatten` | Flatten docker images; sync final state to live container. |
| `docker-image-layer-surgery` | Diff/extract Docker images via registry API without docker. |
| `docker-image-ops` | Use for Docker/OCI image ops without docker CLI. |
| `dsh-ops` | dsh harness 运行期问题——provider API 报错/超时诊断、provider 配置与切换时用. |
| `scnet-comfyui-deploy` | Use when deploying or troubleshooting ComfyUI on SCNet. |
| `scnet-container-env` | SCNet 容器通用环境装配/排障：node/npm、hermes dashboard、代理网络。 |
| `scnet-fasttrans-transfer` | Use for SCNet zzefile 快传/rayfile-c file transfers. |
| `scnet-gateway-forwarding` | SCNet ai-forward网关暴露服务(dsh/ComfyUI)白屏404排障与aiproxy改写方案时用. |
| `scnet-git-pull` | SCNet 容器从 GitHub 拉取/更新代码时用：ghfast.top 镜像 503 时直连 github。 |

## 研究与信息检索（7）

| skill | 说明 |
|---|---|
| `arxiv` | Search arXiv papers by keyword, author, category, or ID. |
| `blogwatcher` | Monitor blogs and RSS/Atom feeds via blogwatcher-cli tool. |
| `grounded-citations` | Ground answers and documents in cited, verifiable sources. |
| `llm-wiki` | Karpathy's LLM Wiki: build/query interlinked markdown KB. |
| `polymarket` | Query Polymarket: markets, prices, orderbooks, history. |
| `research-paper-writing` | Write ML papers for NeurIPS/ICML/ICLR: design→submit. |
| `web-fact-finding` | Use when search fails: fetch official sites and JS bundles. |

## AI Agent 协作（6）

| skill | 说明 |
|---|---|
| `claude-code` | Delegate coding to Claude Code CLI (features, PRs). |
| `codex` | Delegate coding to OpenAI Codex CLI (features, PRs). |
| `computer-use` | Drive the user's desktop in the background — clicking, typing, scrolling, dragging — without stealing the cursor, keyboard focus, or switching virtual desktops / Spaces. Cross-platform: macOS, Windows, Linux. Works with any tool-capable model. Load this skill whenever the `computer_use` tool is available. |
| `hermes-agent` | Use, configure, theme, extend, and orchestrate Hermes Agent. |
| `hermes-custom-providers` | Set Hermes custom provider context_length caps correctly. |
| `opencode` | Delegate coding to OpenCode CLI (features, PR review). |

## GitHub 工作流（6）

| skill | 说明 |
|---|---|
| `codebase-inspection` | Inspect codebases w/ pygount: LOC, languages, ratios. |
| `github-auth` | GitHub auth setup: HTTPS tokens, SSH keys, gh CLI login. |
| `github-code-review` | Review PRs: diffs, inline comments via gh or REST. |
| `github-issues` | Create, triage, label, assign GitHub issues via gh or REST. |
| `github-pr-workflow` | GitHub PR lifecycle: branch, commit, open, CI, merge. |
| `github-repo-management` | Clone/create/fork repos; manage remotes, releases. |

## Apple 生态（4）

| skill | 说明 |
|---|---|
| `apple-notes` | Manage Apple Notes via memo CLI: create, search, edit. |
| `apple-reminders` | Apple Reminders via remindctl: add, list, complete. |
| `findmy` | Track Apple devices/AirTags via FindMy.app on macOS. |
| `imessage` | Send and receive iMessages/SMS via the imsg CLI on macOS. |

## 媒体处理（3）

| skill | 说明 |
|---|---|
| `gif-search` | Search/download GIFs from Tenor via curl + jq. |
| `songsee` | Audio spectrograms/features (mel, chroma, MFCC) via CLI. |
| `youtube-content` | YouTube transcripts to summaries, threads, blogs. |

## Hermes 自身（1）

| skill | 说明 |
|---|---|
| `hermes-session-history` | When session_search fails, inspect state.db for past chats. |

## 邮件（1）

| skill | 说明 |
|---|---|
| `himalaya` | Himalaya CLI: IMAP/SMTP email from terminal. |

## 笔记（1）

| skill | 说明 |
|---|---|
| `obsidian` | Read, search, create, and edit notes in the Obsidian vault. |

## 智能家居（1）

| skill | 说明 |
|---|---|
| `openhue` | Control Philips Hue lights, scenes, rooms via OpenHue CLI. |

## 社媒（1）

| skill | 说明 |
|---|---|
| `xurl` | X/Twitter via xurl CLI: raw post search, posting, DM, media. |

## 使用方式

把需要的 skill 目录拷进 CodeBuddy 的用户级或项目级 skill 目录即可：

```bash
git clone https://gitlab.scnet.cn:9002/skills/comfyui114514/dcu_developer.git
cp -rf dcu_developer/<skill-name> ~/.codebuddy/skills/     # 用户级
# 或 .codebuddy/skills/                                    # 项目级
```

细节见每个目录下的 `SKILL.md`。

## 来源与适配说明

- **来源**：Hermes agent skill 库（`~/.hermes/skills`）。

- **格式适配**：`SKILL.md` 的 frontmatter 只保留 CodeBuddy 认识的 `name` / `description`，
  Hermes 专有字段（`version`/`author`/`platforms`/`prerequisites`/`metadata` 等）已丢弃；
  `triggers` 折进 `description`；正文里的 `scripts/ references/ templates/ …` 相对路径
  已改写为 `${CODEBUDDY_SKILL_DIR}/…`（因为 CodeBuddy 里 cwd 是项目目录而非 skill 目录）。

- **凭据清洗**：已清除正文与脚本中写死的平台代理 URL（含真实口令），
  改为动态读取 `proxy_env.sh` / `$http_proxy`。

- **未收录**：`deepseek-unrestricted` 类别（绕过模型安全闸门的越狱 prompt）。

- **改名**：`pdf` / `docx` / `xlsx` → `hermes-pdf` / `hermes-docx` / `hermes-xlsx`，
  避免与 CodeBuddy 官方 plugin 同名冲突。

- **已知遗留**：部分 skill 引用了上游源库里本就不存在的 `references/case-*.md`，属上游遗留。
