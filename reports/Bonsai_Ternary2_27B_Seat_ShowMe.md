# Bonsai 2 27B Single-Seat Setup — ShowMe PLAN

Target: 1 llama-server seat, PTQ1_0 + turbo3_tcq + OrcaRouter LoRA
Generated: 2026-09-18T23:43Z
Mode: PLAN (does not exist yet; building on this host)
Ingredients: ascii-diagrams, ascii-graph, ascii-visualize, ascii-simulator, trident-deep-planning (DPL1-lite)

## Problem (verbatim)

SET THIS UP CORRECTLY NOW. 1 seat at max reasoning on turbo3. Best
quality I can possibly get on this 1 seat. Install the OrcaRouter
uncensored version.

## Core insight

One process, one model load, one KV type. Weights stay in VRAM once.
turbo3_tcq is the densest KV that stays usable for driving a real
session. Do not run a second 27B. Do not use VBR. Do not use Docker.

## Measured host (this turn)

```
nvidia-smi: RTX 4080 Laptop, 12282 MiB, 10975 MiB free, sm 8.9
nvcc:       /usr/local/cuda-12.8  V12.8.93
cmake:      3.28.3
disk:       36G free on /
RAM:        30Gi total, 1.3Gi available  (build uses -j4)
clone:      /home/leviathan/buun-llama-cpp  HEAD 1d0f493
model:      prism-ml/Ternary-Bonsai-2-27B-gguf
            Ternary-Bonsai-2-27B-PTQ1_0.gguf  (5.95 GB, public)
LoRA:       Continuum-AI-Corp/OrcaBonsai-27B-Uncensored  HTTP 401
            HF_TOKEN unset, tokenfile absent
```

## Scope (<=20)

1. Clone spiritbuun/buun-llama-cpp --depth 1
2. CMake CUDA 12.8, ARCH 89, FA on, TURBO_FA on (default)
3. Build llama-server -j4
4. hf download PTQ1_0 to /home/leviathan/models/bonsai
5. LoRA download after hf login (gated)
6. Launch 1 seat: -c 262144 -ctk turbo3_tcq -ctv turbo3_tcq
7. Prove /health + nvidia-smi used < 11000 MiB

## Success criteria (mechanical)

| Check | Command / eyes |
|---|---|
| Binary exists | test -x .../build/bin/llama-server |
| TURBO_FA linked | strings llama-server \| grep TURBO_FA |
| GGUF on disk | stat PTQ1_0.gguf size ~5.95G |
| Server up | curl -sf http://127.0.0.1:8080/health |
| Cache type | process argv contains turbo3_tcq |
| VRAM | nvidia-smi memory.used < 11000 MiB |
| LoRA | --lora applied OR documented 401 gate |

## Master graph (entry to terminal)

```
┌──────────────────────────────────────────────────────────────────┐
│  HOST  RTX 4080 Laptop  12282 MiB  CUDA 12.8  sm_89              │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  A  clone  /home/leviathan/buun-llama-cpp  1d0f493               │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  B  cmake  GGML_CUDA=ON  ARCH=89  TURBO_FA=ON (default)          │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  C  cmake --build --target llama-server -j4                      │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
┌────────────────────────────┐  ┌─────────────────────────────────┐
│  D  PTQ1_0.gguf  5.95 GB   │  │  E  LoRA  401 until hf login    │
│  models/bonsai/            │  │  skip on launch if missing      │
└─────────────┬──────────────┘  └────────────────┬────────────────┘
              │                                  │
              └──────────────┬───────────────────┘
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│  F  llama-server  :8080  np=1  c=262144  turbo3_tcq  fa on       │
│     ngl=99  kv-unified  no-mmproj  reasoning on  effort xhigh    │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  G  curl /health  +  nvidia-smi  +  one /v1/chat/completions     │
└──────────────────────────────────────────────────────────────────┘
```

shown=7 total=7 capped=0

## File tree (landing zone)

```
┌──────────────────────────────────────────────────────────────────┐
│ /home/leviathan/                                                 │
├──────────────────────────────────────────────────────────────────┤
│ ├─ buun-llama-cpp/          fork source + build                  │
│ │  └─ build/bin/llama-server                                     │
│ ├─ models/bonsai/           GGUF + LoRA                          │
│ │  ├─ Ternary-Bonsai-2-27B-PTQ1_0.gguf                           │
│ │  └─ bonsai-abliterate-lora.gguf   (after login)                │
│ └─ JARVIS_WORKSPACE/Shared_Workspace/reports/                    │
│    └─ Bonsai_Ternary2_27B_Seat_ShowMe.md                         │
└──────────────────────────────────────────────────────────────────┘
```

## Mechanism (launch)

```
┌────────────────────────────────────────┐
│ load PTQ1_0 weights onto GPU           │
│   if LoRA file exists                  │
│     apply --lora-scaled path:1.0       │
│ pin K and V to turbo3_tcq              │
│ force flash-attn on                    │
│ skip mmproj                            │
│ bind :8080  np=1  ctx=262144           │
│ thinking: on  effort: xhigh            │
│ sample: temp=1.0 top_p=0.95 top_k=20   │
└────────────────────────────────────────┘
```

## Exact launch command (after binary + GGUF exist)

```
/home/leviathan/buun-llama-cpp/build/bin/llama-server \
  -m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf \
  -c 262144 -np 1 \
  -ctk turbo3_tcq -ctv turbo3_tcq \
  -ngl 99 -fa on --kv-unified --no-mmproj \
  --port 8080 --jinja \
  --reasoning on --reasoning-effort xhigh \
  --temp 1.0 --top-p 0.95 --top-k 20 \
  --host 127.0.0.1
```

If LoRA is on disk, append:

```
  --lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0
```

Do not pass both --lora and --lora-scaled for the same file.

## VRAM budget (1 seat, turbo3_tcq)

```
PTQ1_0 weights     5.95 GB
turbo3_tcq KV      3.25 GB   (262144 ctx, 16 full-attn layers)
overhead           0.50 GB
--------------------------
total              9.70 GB
card               12.00 GB
headroom           2.30 GB
```

## Runtime trace (happy path)

TIMELINE
═══════════════════════════════════════════════════════════
T-0: clone 1d0f493, cmake sm_89, hf download PTQ1_0

STEP 1: START SERVER
┌─────────────────────────────────────────────────────────┐
│ exec: llama-server                                      │
│ args: -m PTQ1_0.gguf -c 262144                          │
│       -ctk turbo3_tcq -ctv turbo3_tcq                   │
│       -ngl 99 -fa on --port 8080                        │
│ → listen 127.0.0.1:8080                                 │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
STEP 2: HEALTH
┌─────────────────────────────────────────────────────────┐
│ curl -sf http://127.0.0.1:8080/health                   │
│ → HTTP 200                                              │
└─────────────────────────────────────────────────────────┘
                    │
                    ▼
STEP 3: VRAM
┌─────────────────────────────────────────────────────────┐
│ nvidia-smi --query-gpu=memory.used --format=csv,noheader│
│ pass: used < 11000 MiB                                  │
│ fail: used >= 11000 MiB or process gone                 │
└─────────────────────────────────────────────────────────┘

ROUTING
┌─────────────────────────────────────────────────────────┐
│ Q: LoRA file present?                                   │
│   YES → add --lora-scaled path:1.0                      │
│   NO  → start without LoRA, print HF 401 gate           │
│ Q: second 27B wanted? → NO, refuse, one process         │
└─────────────────────────────────────────────────────────┘

DELIVER
┌─────────────────────────────────────────────────────────┐
│ OpenAI API  POST http://127.0.0.1:8080/v1/chat/completions│
│ Web UI      http://127.0.0.1:8080                       │
└─────────────────────────────────────────────────────────┘

OBSERVER
═══════════════════════════════════════════════════════════
  nvidia-smi watch: spill to host RAM = 10x slowdown, stop
  Quiet case: used ~9.7 GB, no host fallback, health 200

## Test angles (>=3)

| Angle | Pass token | Fail token |
|---|---|---|
| TOOLS | llama-server exists, exit 0 health | no binary |
| CONFIG | argv has turbo3_tcq | cache type f16 |
| BOUNDARY | -c 262144 accepted | OOM / host spill |
| INTEGRATION | /v1/chat/completions 200 | connection refused |
| STATE | nvidia-smi used < 11000 | used >= 11000 |

## Open decisions

| Fork | Evidence | Default |
|---|---|---|
| LoRA now vs later | HF 401, no token | later: hf login then --lora-scaled |
| temp 1.0 vs 0.7 | model card thinking=1.0 | 1.0 for max reasoning |
| -j4 vs -j32 | 1.3Gi RAM free | -j4 |

## Anti-patterns (do not)

- Second llama-server with same GGUF (double 5.95 GB = OOM)
- -ct vbr on this seat
- turbo2_tcq / turbo1_tcq on the drive seat
- Loading mmproj (0.63 GB wasted)
- PATH nvcc 12.0 instead of 12.8

## Handoff

This is DPL1-lite. L2 (wave desks) not needed: one binary, one model.
Execute remaining: wait build + download, then launch.
