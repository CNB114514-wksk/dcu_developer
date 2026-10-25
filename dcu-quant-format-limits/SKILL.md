---
name: dcu-quant-format-limits
description: 海光 DCU 量化格式能力边界（2026-10-25 实测）：不支持 FP8/FP4，int4 仅 gfx936、gfx928 最高 int8；矩阵核只有 MMAC（MFMA 是死路）。含 gfx928 线程/VGPR/LDS/带宽硬约束、int8 grouped MoE 配方、MXFP4 直读反例、端到端选型教训。选量化格式或写 GEMM/MoE kernel 前先读。
tasks: ""
---

# 海光 DCU 量化格式边界（实测）

适用：在 DCU 上选模型量化格式、判断某格式有没有矩阵核、写 GEMM/grouped MoE kernel 前定线程与资源预算。
结论来自 2026-10-25 在 K100_AI（gfx928）上的手写 MMAC kernel 与 grouped MoE 实测。

## 何时使用

- 要决定模型用 FP8 / FP4 / int8 / int4 / GGUF K-quant 哪一种
- 评估「把权重再压一半能不能更快」（MXFP4/FP4 路线）
- 手写或调 GEMM / grouped MoE kernel，需要线程数、VGPR、LDS、带宽预算
- 判断 AMD/ROCm 上游 kernel 能否直接搬（先看它是不是 MFMA）

## 一、格式支持矩阵（硬事实）

| 格式 | gfx928（K100_AI / K500SM_AI） | gfx936（BW150） | 说明 |
|---|---|---|---|
| FP8（E4M3/E5M2） | ✗ | ✗ | 无 fp8 矩阵单元，只能软解 |
| FP4 / MXFP4 | ✗ | ✗ | 无 fp4 单元；直读不加速，见 §五 |
| int8（W8A8） | ✓ `I8I` | ✓ | **gfx928 上唯一有矩阵核的量化格式** |
| int4（W4A*） | ✗ | ✓ `4xi8I` | gfx928 只有 `v_dot8_i32_i4` 这类点积，没有 int4 矩阵核 |
| fp16 / bf16 | ✓ | ✓ | 基线 |

- **DCU 整体不支持 FP8 / FP4**（用户确认，2026-10-25）。
- **int4 只在 gfx936 有矩阵核**；gfx928 天花板是 int8。
- 选型一句话：**gfx928 → int8；gfx936 → int4**。4bit 让 DeepSeek-V4-Flash-0731 有机会压进 137 GB → 2 卡可行。

## 二、矩阵核只有 MMAC，没有 MFMA

- 唯一可用的矩阵指令是 **MMAC**（`__builtin_amdgcn_mmac_*`）。
- **MFMA 是死路**：加 `-mllvm -amdgpu-enable-mai-insts` 之类能编过，但硬件执行直接 `ILLEGAL_INSTRUCTION`。看到上游 kernel 用 `v_mfma_*` / `__builtin_amdgcn_mfma_*` / rocWMMA，直接判定不可用，只能按算法结构重写。
- 探测：`__has_builtin(__builtin_amdgcn_mmac_f32_16x16x16f16)` 在 host pass 为假是正常的，**以 device pass 为准**。

## 三、gfx928 硬约束（写 kernel 前先算）

| 约束 | 值 | 说明 |
|---|---|---|
| `maxThreadsPerBlock` | **1024** | 硬件真实值 |
| `hipFuncGetAttributes` 报 256 | 假象 | 没写 `__launch_bounds__` 时 clang 的默认值，**不是硬件墙** |
| VGPR 预算 | **131072 / CU** | 必须满足 `numRegs × threads ≤ 131072`（例：255 regs 装不下 768 线程） |
| LDS | **64 KiB / block**（= 64 KiB / MP，无 opt-in 可用） | 决定能放几个 stage / 能否存反量化后的 bf16 |
| HBM 读带宽 | **722.8 GB/s**（kernel 级 microbench） | 与 llama.cpp 侧 823 GB/s 的差别是测法口径不同，见 `dcu-llm-tuning` |
| L2 | 8 MiB | |
| CU / 频率 | 128 CU @ 1.4 GHz | |

> 线程数先按 `numRegs × threads ≤ 131072` 反推上限，再看 LDS 够放几个 block —— 这两个数决定 occupancy，比调 tile 更前置。

## 四、性能基准（2048³，T=512，tile 128×128，cos 1.000000）

| 数据类型 | TFLOPS |
|---|---|
| f16 | 92 ~ 96 |
| bf16 | 95.9 |
| **int8** | **161.7** |

- **decode 交叉点 M≈12**：M=1 时纯 GEMV 0.170 ms（603 GB/s）比 MMAC 0.371 ms **快 2.2×**；M≥16 起 MMAC 反超。**小 batch decode 别上矩阵核**，GEMV 是带宽问题不是算力问题。
- **swizzle 只在大 M 有用**：2048×7168² 50.9 → 74.5 TFLOPS（**+46%**）；M=256 时无收益——每字节权重只读一次，L2 复用无从谈起。

## 五、int8 grouped MoE（DSV4 形状：E=256 / top-6 / 4096→2048）

- **768 线程成立**：8 个 consumer wave + 4 个 producer wave，`numRegs=47`（预算 170），LDS 20.3 KiB。
- **砍 producer 降到 512 线程会慢 1.4×**。当年 megamoe「512 线程零 producer」是错解，**正解是保住 producer、压 VGPR**（768 线程装不下时先降寄存器，不是砍流水线）。
- 相对 torch 逐 expert `bmm`（bf16 常驻）：**4.6× ~ 10.4×**；相对每次 forward 反量化：**10× ~ 22.6×**。
- **瓶颈是纯带宽墙**：M=256 时权重读取占 100%（4.27 GB @ 557 GB/s），int8 算力只占 4.2%。此时再优化算力=白做。

## 六、★反例：MXFP4 直读 + kernel 内反量化，比 int8 没有提速

- 实测 **0.99 ~ 1.06×**（等于没提速），尽管权重字节减半（2.40 GB vs 4.26 GB）。
- 原因：反量化后的 bf16 要过 LDS，**LDS 占用翻倍 → 每 CU 只能放 1 个 block（int8 是 3 个）→ 有效带宽 320 vs 557 GB/s**，把字节减半的收益全吃掉。
- 反量化本身占 23%；即使把它降到 0 也只到 5.76 ms，离 3.32 ms 理论下限仍远。
- **结论：硬件没有 fp4 单元，这条路没有加速可拿。** 别再试 FP4/MXFP4 直读路线。

## 七、策略与端到端教训

- 格式：**gfx928 → int8（`I8I`）**；**gfx936 → int4（`4xi8I`）**。
- **优化前先确认瓶颈在哪一层**：MoE 单层从 16.24 ms 降到 0.282 ms（57×），端到端 decode 只从 1.27 → 1.30 tok/s（**+2.1%**），因为 MoE 只占 787 ms/token 里的 16 ms。**局部 50× 不等于端到端有感知**，先做时间归因再动手。
- 量化收益的判断顺序：① 该格式在本卡有没有矩阵核 → ② 反量化/布局转换是否引入额外 LDS 或带宽 → ③ 端到端该层占比。

## 八、测法（不守这些数字全是假的）

1. **测前预热 GPU**：空跑 ≥60 次；首测比稳态低约 20%（升频需要持续负载）。
2. **必须报 cos**：量化 kernel 只看 TFLOPS 会漏「静默返全零 / NaN」，用 `cos ≈ 1.000000` 作为数值正确性门槛。
3. **区分算力与带宽**：先算 `bytes / 实测带宽`，若已接近实测时间，就是带宽墙，别再调计算部分。
4. 与 torch 对比要写清基线是「bf16 常驻」还是「每次 forward 反量化」——两者差 2~5×。
