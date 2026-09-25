---
name: dcu-llm-tuning
description: 海光 DCU（K100AI/gfx928、BW150/gfx936）上用 llama.cpp 调 LLM 的实测知识：测法规范、NUMA/ub/ncmoe 杠杆、MMAC、已证伪路线、已踩坑。
---

# 海光 DCU 上调 LLM（llama.cpp）——实测知识

适用：在 SCNet 容器的海光 DCU 上跑/调 llama.cpp（llama-server）推理。所有结论 2026-09-24/25 于 K100AI(gfx928) 与 BW150(gfx936) 实测得出。

## 硬件事实（先记住，别再搞错）

| | K100AI | BW150 |
|---|---|---|
| 架构 | gfx928:sramecc+:xnack-（CDNA2） | gfx936（CDNA2） |
| CU / VRAM | 128 / 64 GiB | 80 / — |
| HBM 实测带宽 | **823 GB/s**（规格 896；「1.6TB/s」是错的） | **1338 GB/s** |
| dp4a 峰值 | 76.3 TOPS | 60.9 TOPS |
| **MMAC** INT8 / FP16 | **361.6 TOPS / 178 TFLOPS**（92-94% 规格） | INT8 822 TOPS |

- **矩阵核是 MMAC（`v_mmac_*`），不是 AMD MFMA（`v_mfma_*`）**。MFMA 确实不可用（`HYGON_DCU_NO_MFMA`），但 MMAC 默认可用 —— 探测用 `__has_builtin(__builtin_amdgcn_mmac_f32_16x16x16f16)`（host pass 为假是正常的，device pass 才算数）。gfx906/926 无此 builtin。
- 两台取舍：**要首字/prefill 用 K100AI（CU 多），要 decode 吞吐用 BW150（带宽 1.63×→decode 恰好快 1.62×）**。不要试图把一台调成另一台。

## 测法规范（不守这些，数字全是假的）

1. **测 kernel/任何基准前先预热 GPU**：首测比稳态低 ~20%（升频需要持续负载）。空跑 ≥60 次再计时。这个坑让同一内核两次被误判"性能回归"。
2. **llama-bench 不可靠**：它没有 `-c`（用自己的小 ctx，ub 结论会假阳性）；且 argsort 等 kernel 静默 launch 失败后照常计时再 SIGSEGV。**验收一律用 llama-server 的 `prompt eval time`**。
3. **合成 prompt 低估真实首字 ~25%**（PLE n-gram/专家命中行太少）。验收用真实流量。
4. **A/B 纪律**：每配置独立重启 server、只发 1 个请求、`cache_prompt=false`、**`-b` 和 `-ub` 必须同时显式设**（只设一个会得到 `-b 2048 -ub 4096` 这种错配）、交错多轮。K100AI 上 prefill 绝对值会随"同 server 多请求的 KV 状态"漂移 1.85×，等长请求则完全稳定 —— 所以统一用「新 server + 单请求」。
5. **kernel 归因必须 graphs 开**（默认）：关 graphs 会把 host launch/sync 开销放大几倍，得出 "~70% 时间不在 GPU" 的假象（实际 90% 在 kernel）。
6. **ggml INFO 日志要 `-lv 4`**：`-lv 3`（llama-server 默认）看不到 `MMAC F16 GEMM in use` 这类行，会误判内核没生效。
7. 杀服务用 `pkill -x llama-server`（`pkill -f` 会匹配到自己命令行自杀）。起服务 `setsid nohup ... & disown`，`(cmd &)` 会随 tool shell 退出被杀（症状：日志停在 loading model、无报错）。
8. **测前先跑 `detect_env.sh`**（探测 GPU/cpuset/NUMA/权重分层 → 推断 CPU_MASK/ncmoe/ub/t 等参数）。

## 杠杆（按实测性价比排序）

1. **NUMA 钉核（decode 最大杠杆，K100AI 上 3.5-4×）**：8 NUMA 节点容器里，把线程钉到权重所在节点（`-C <mask> --cpu-strict 1`）：decode 187.5→53.0 ms/tok。原理：**收益来自"钉得一致"而非钉到特定节点** —— 线程钉哪，权重 first-touch 就落哪。`CPU_MASK=auto` 已实现「cpuset 跨多节点→收窄到第一个节点」。⚠️ **机器相关**：BW150 新容器实测钉核无收益（页全在 node0、带宽太高掩盖延迟），别互相套用。K100AI 上页是粘的（历史运行落在 node1/2），页探测（扫 `/proc/*/numa_maps`，须 server 运行中）可再挤 10%。
2. **ub=4096 + ncmoe=14（通用，两台都采纳）**：K100AI prefill +6.8%（3.00→2.81 ms/tok，干净 A/B 修正过，旧报的 1.20× 是错配 -b 造成的假象）；BW150 1.14×（TTFT 19.08→16.50s）。ncmoe 12→14 本身只贵 2%，是为了给 ub=4096 腾 4.43 GiB 计算缓冲。**ub 是比 MMAC 更前置的杠杆**：ub 翻倍 → MoE 算术强度翻倍 → MMAC 天花板 K100AI 1.0→1.35×、BW150 1.38→2.76×。
3. **`-fa on` 必须**：qwen4exp（lightning indexer 稀疏注意力）的稀疏路径只在 FA 内核里实现，`-fa off` → 输出退化成复读 `'/'`（服务活着、日志全 200、无报错）。小 prompt 恰好能蒙对，别被"长度阈值"假象骗。
4. **手写 MMAC f16 GEMM（已接入，1.32×）**：DTK 的 rocBLAS 只有 gfx906 的 Tensile 库，gfx928 f16 GEMM 掉到非矩阵核 kernel（4.3-10.8 TFLOPS）；手写 MMAC GEMM 同形状 74-97 TFLOPS。接入点 = `ggml_cuda_should_use_mmf()` 对 CDNA2 F16 返回 false → dense F16 必走 cublas，在那里先试 MMAC（`mmac-gemm.{cu,cuh}`，env `GGML_CUDA_MMAC_GEMM` 开关，tokens≥256 门槛）。任意形状靠「行钳制+写出守卫」，**千万别用逐元素标量 mask**（97.7→14.6 TFLOPS 的实锤）；K 必须是 32 的倍数。
5. **MoE 设备侧 MMQ 补丁（M1，~2.2×）**：gfx928 被归类 CDNA2 → `should_use_mmq` 拒绝量化 MoE → 落 host fallback（512 次 per-expert GEMM + 2 次 streamSync + 不可入图）。补丁：`mmq.cu` 加 `if (GGML_CUDA_CC_IS_HYGON_DCU(cc)) return n_experts > 0;`。连带修掉了 hipGraphExecUpdate 段错误（MoE op 稳定入图后不再反复 update）。
6. **前缀缓存（服务层最大杠杆）**：混合 SSM 模型 warm TTFT 地板 ~2s（前缀缓存恢复开销），但冷启动 15k prompt ≈ 40-64s。**预热一次固定前缀后，所有请求（含全新 session）首字 2-3s**。`PREWARM=1` 已内置。

## 已证伪清单（别再试）

- ❌ **MPOL_INTERLEAVE**：被 numa_balancing=1（容器内只读）中和；钉节点时无差别，不钉时反而慢 8-10%。
- ❌ **改 ggml MoE 循环结构**：disable_chunking 使每线程恰好领一块、零原子争用、负载天然均衡；n_as 扫描开销只占 0.05%。
- ❌ **hipBLASLt grouped GEMM**：DTK 26.04 头文件里没有 grouped API；也不认 Q4_K 块格式，dequant 成 f16 流量反增 4.5×。
- ❌ **`--cache-reuse`**：SSM hybrid 模型被 llama-server 自动禁用（`cache_reuse is not supported by this context`），设多少都无差别。
- ❌ **强制 dense Q4_K 走 MMQ**：比 dequant+cublas 慢 7.6%，上游 dense 启发式是对的。
- ❌ **调参救 prefill**：加线程更慢（t16 6.84 → t128 8.35 ms/tok）、ubatch 无影响、`-tb` 明显有害 → **没有任何 server flag 能改善 prefill**，只能 kernel 级。
- ❌ **MFMA 路线**（`+mai-insts` 等）：gfx928 汇编器直接 invalid instruction。矩阵核出路是 MMAC。
- ❌ **graphs 提速**：补丁后 graphs 稳定无段错误，但对吞吐零收益（3.73 vs 3.73 ms/tok）。

## 已踩坑（现象 → 根因）

- **服务退化复读 `'/'`** → `-fa off`（见杠杆 3）。
- **llama-bench 崩 / 数字不可信** → `k_argsort_f32_i32` 无 `__launch_bounds__`，DTK 编出 max 256，ncols_pad∈(256,512] 时静默 launch 失败后 SIGSEGV。修法：加 `__launch_bounds__(1024)`。见到 launch-bounds 报错先确认二进制对应当前源码，别急着改 kernel。
- **首测慢 20% 误判回归** → 没预热（见测法 1）。
- **以为内核没生效** → 日志级别 `-lv 3` 看不到（见测法 6）。
- **MMAC fragment 布局两套约定**：K 连续套（`A[l,i]=Am[h][4g+i]`，B 要 bitswap，f16x4 向量加载，快）与 K 步长 4 套（BW150 在用，只能标量加载，慢）。两套各自正确但**参数必须成套**，混用错 192/256。别再误报"gfx928/936 布局不同"。
- **MoE MMQ 同形状不同量化差 30×**（q6_K 53 TFLOPS vs q4_K 2.5）：q6_K 用 `SRAM_LAYOUT_Q6_K`，其余用 Q8_0/Q8_1 布局 → 疑 LDS bank conflict，未确证。诊断工具：`GGML_MMQ_DEBUG=1`。
- **上报 `prompt eval time` ≈ 内部 prefill 的 2 倍**：每请求有 ~3.4s（K100AI）/6.2s（bw150）固定开销（疑似尾批触发 graph 重建），与长度无关。对比时明确口径。
- **估显存对账**：MemTotal 与 buffer 换算要 ×1.048576；PLE 表（28.8 GB）本来就在 host 不占显存，别当杠杆动（get_rows 仅 0.2% kernel 时间）。

## roofline 速算法

MoE 每 chunk 专家权重字节与 M 无关、MACs ∝ M → `AI = M × 0.0306 MACs/byte`（Qwen3.8-Flash-Next 几何：48 层/512 专家/top-10/2560/640）。脊点 = 峰值算力 ÷ 带宽：K100AI dp4a 脊点 92.7（M≥3026 才算力受限）、MMAC 439；BW150 dp4a 45.5（M≥1487）。工具 `scripts/roofline.py`。**给另一台机器的建议要按它的带宽/算力重算，别照抄。**

## 成绩基线（2026-09-25，交付默认配置 ncmoe=14/ub=4096/ctx=65536/t=8/MMAC 开，新 server+单请求）

| prompt | K100AI prefill/首字/decode | BW150 prefill/首字/decode |
|---:|---|---|
| 10247 | 377 t/s / 27.2s / 13.7 t/s | 321 t/s / 32.0s / 22.2 t/s |

规律：prefill t/s 随长度升并趋平（短 prompt 摊不薄固定开销，别用短 prompt 代表整体）；首字 ≈ prompt÷prefill 速率；decode 当常数。

## 模型/存储注意

- **模型放持久区（`$PERSIST_HOME`，用 scnet-container-env skill 的 `persist_home.sh` 识别）**，别拷进 `/root` —— 撑爆镜像层。持久区冷读只影响**头几条请求的 prefill**（页缓存热了后无差别），decode 完全不受影响（权重已进 VRAM）。
- 拷 build 目录必须 `--exclude='core.*'`（曾有 2.5 GB core dump 混进去）。
- 放置判定：有卡容器 GPU MoE 主导；`-ncmoe N` 把 N 层专家留 CPU —— ncmoe 是显存/速度关键旋钮，锚点外推：64 GiB→14，48G→25，32G→35，24G→41。
