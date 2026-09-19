# Bonsai Qwen Local Provider — ENGINEERING REPORT v1
**Series:** Bonsai_Qwen_Local_Provider (local OpenAI-compatible 27B seat on RTX 4080 Laptop; the parent/base series is none — this is v1 of a new lineage. PufferKernel_Engineering_Report_v2/v3 exist in reports/ and are a different series.)
**Project:** llama-bonsai-qwen systemd user unit + PTQ1_0 + turbo3_tcq + OrcaRouter LoRA
**Date:** 2026-09-19T01:02:58Z (measured `date -u`)
**Author:** omp coding-agent (xai-oauth/grok-4.6)
**Baseline:** spiritbuun/buun-llama-cpp @ 1d0f493
**Commit:** `1d0f493c73817176f6953069b7c10211de9555a1` (`git -C /home/leviathan/buun-llama-cpp log -1`)
**Container:** none · **Image:** none (host-live CUDA, not omp-ct)

---

## ONE-PARAGRAPH SUMMARY

This lineage is a single host-live OpenAI Chat Completions server named `bonsai-qwen` that loads PrismML Ternary-Bonsai-2-27B PTQ1_0 (5,946,648,928 bytes) plus a rank-1 OrcaRouter abliterate LoRA (9,682,464 bytes, SHA256 `f1669534803d340a496015f5c45125f3437b4d13ec764f40e34488ce83967f42`) into one `llama-server` process built from `spiritbuun/buun-llama-cpp` commit `1d0f493`, CUDA 12.8 sm_89, with K and V caches pinned to `turbo3_tcq` (ggml type id 46), flash-attn on, no mmproj, n_ctx 262144, n_slots 1, bound `0.0.0.0:18081`, supervised by systemd `--user` unit `llama-bonsai-qwen.service` (enabled, active, MainPID 2063850). It produces `GET /health {"status":"ok"}`, `GET /v1/models` id `bonsai-qwen`, `GET /lora-adapters` scale 1.0, and `POST /v1/chat/completions` with thinking traces; VRAM at report time is 10465 MiB used of 12282 MiB on NVIDIA GeForce RTX 4080 Laptop GPU. It is not the Tailscale funnel MCP web-shell (`/` still proxies `127.0.0.1:4050`). It is not a second 27B. The deleted `~/beellama.cpp` tree is gone (`test ! -e` → GONE).

---

## THE LIFECYCLE MAP

```
┌──────────────────────────────────────────────────────────────────┐
│  OPERATOR pin  Bonsai Qwen local provider stays up               │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  systemd --user  llama-bonsai-qwen.service                       │
│  file: ~/.config/systemd/user/llama-bonsai-qwen.service:11-21    │
│  ExecStart llama-server  -a bonsai-qwen  port 18081              │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  llama-server  1d0f493  LD_LIBRARY_PATH=build/bin                │
│  binary 17888 bytes  libggml-cuda.so 166570104 bytes             │
└───────────────────────────────┬──────────────────────────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ PTQ1_0.gguf      │  │ LoRA scale 1.0   │  │ KV turbo3_tcq    │
│ 5946648928 B     │  │ 9682464 B        │  │ ggml.h:438 id 46 │
│ sha 53107f53...  │  │ sha f1669534...  │  │ vbr.enabled=false│
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
         │                     │                     │
         └──────────┬──────────┴──────────┬──────────┘
                    ▼                     ▼
┌──────────────────────────────────────────────────────────────────┐
│  HTTP  0.0.0.0:18081  OpenAI /v1  alias bonsai-qwen              │
│  health ok  models id=bonsai-qwen  lora-adapters scale=1.0       │
└──────────────────────────────────────────────────────────────────┘
```

shown=8 total=8 capped=0  box width=68 (ruler: 68 cols)

---

## THE ARCHITECTURE (per module)

### Module 1 — systemd user unit

Interfaces: Type=simple, Restart=always, RestartSec=10, WantedBy=default.target, SyslogIdentifier=llama-bonsai-qwen. Data flow: user session linger → default.target → ExecStart argv (unit lines 11-21) → PID 2063850. Failure modes: bind fail (port in use) → journal `couldn't bind HTTP server socket` then Restart=always retries; OOM/GPU → process exit, systemd restarts after 10s; TimeoutStopSec=30 on stop.

Calibration constants (named, from unit file):

| Name | Value | Anchor |
|---|---|---|
| port | 18081 | llama-bonsai-qwen.service:18 |
| n_ctx | 262144 | llama-bonsai-qwen.service:15 |
| n_slots | 1 | llama-bonsai-qwen.service:15 |
| alias | bonsai-qwen | llama-bonsai-qwen.service:14 |
| lora scale | 1.0 | llama-bonsai-qwen.service:13 |
| cache K/V | turbo3_tcq | llama-bonsai-qwen.service:16 |
| ngl | 99 | llama-bonsai-qwen.service:17 |
| host | 0.0.0.0 | llama-bonsai-qwen.service:18 |

```
┌──────────────────────────────────────┐
│ systemd --user                       │
│ llama-bonsai-qwen.service:1-29       │
│ wc -l = 29                           │
└──────────────────┬───────────────────┘
                   │ ExecStart argv
                   ▼
┌──────────────────────────────────────┐
│ llama-server pid 2063850             │
│ Restart=always on non-zero exit      │
└──────────────────────────────────────┘
```

Command:

```
systemctl --user is-enabled llama-bonsai-qwen.service
enabled
systemctl --user is-active llama-bonsai-qwen.service
active
systemctl --user show llama-bonsai-qwen.service -p MainPID --value
2063850
```

### Module 2 — llama-server binary + CUDA backend

Interfaces: OpenAI `/v1/chat/completions`, `/v1/models`, `/health`, `/lora-adapters`, `/props`. Data flow: GGUF mmap → CUDA offload ngl 99 → turbo3_tcq KV encode (set-rows) → FA decode (fattn-mma-turbo) → logits → HTTP JSON. Failure modes: unknown GGUF type if stock llama.cpp (not this fork); missing libggml-cuda.so if LD_LIBRARY_PATH wrong (unit:9); port bind (see bug B1).

```
┌──────────────────────────────────────┐
│ llama-server  17888 bytes stub       │
│ version 0.4.1-dev commit 1d0f493     │
└──────────────────┬───────────────────┘
                   │ dlopen
                   ▼
┌──────────────────────────────────────┐
│ libggml-cuda.so.0.24.0               │
│ 166570104 bytes  strings TURBO_FA    │
└──────────────────────────────────────┘
```

Command:

```
ls -l .../llama-server
-rwxr-x--x 17888 Sep 19 04:13
llama-server --version
version: 0.4.1-dev (build 1, commit 1d0f493)
built with GNU 13.3.0 for Linux x86_64
wc -c libggml-cuda.so.0.24.0
166570104
```

### Module 3 — PTQ1_0 weights

Interfaces: GGUF type PTQ1_0, ggml enum `GGML_TYPE_PTQ1_0 = 91` wire id 143 (`ggml.h:495`). Data flow: file 5946648928 bytes sha256 `53107f530aa52eb00912263ab1ee29bd199261c87cd7b4ad4ca1318c1fe33ee3` → load_model → GPU. Failure modes: stock llama.cpp rejects unknown type; second process doubles 5.95 GiB and OOMs the 12 GiB card.

```
┌──────────────────────────────────────┐
│ Ternary-Bonsai-2-27B-PTQ1_0.gguf     │
│ 5946648928 bytes                     │
│ sha256 53107f53...ee3                │
│ ggml.h:495  PTQ1_0 = 91              │
└──────────────────┬───────────────────┘
                   │ -m path
                   ▼
┌──────────────────────────────────────┐
│ load_model  props ftype              │
│ PTQ1_0 (group 128)                   │
└──────────────────────────────────────┘
```

### Module 4 — KV turbo3_tcq

Interfaces: `-ctk turbo3_tcq -ctv turbo3_tcq` parsed in `common/arg.cpp` (cache type table includes `turbo3_tcq`). Enum `GGML_TYPE_TURBO3_TCQ = 46` at `ggml.h:438`. Data flow: F16 activations → TCQ encode → 3.25 bpv blocks → FA kernels. Failure modes: VBR would auto-degrade (forbidden; props `vbr.enabled=false`); turbo2_tcq would drop quality (forbidden).

```
┌──────────────────────────────────────┐
│ -ctk turbo3_tcq                      │
│ -ctv turbo3_tcq                      │
│ ggml.h:438  TYPE = 46                │
└──────────────────┬───────────────────┘
                   │ KV tensors
                   ▼
┌──────────────────────────────────────┐
│ /props vbr.entry_type_k=turbo3_tcq   │
│ vbr.enabled=false                    │
│ chat timings kv_bpv=3.25             │
└──────────────────────────────────────┘
```

Command (verbatim `/props` vbr object):

```
{"enabled": false, "dynamic": false, "type_k": false, "type_v": false,
 "codec": "turbo", "entry_type_k": "turbo3_tcq",
 "entry_type_v": "turbo3_tcq", "floor_bpv": 0.0, ...}
```

### Module 5 — OrcaRouter LoRA

Interfaces: `--lora-scaled FNAME:SCALE` at `common/arg.cpp:4088-4097` pushes `{path, scale}` into `params.lora_adapters`. Runtime `GET /lora-adapters`. Data flow: 9682464-byte GGUF adapter, architecture qwen35, scale 1.0, applied at load. Failure modes: HF 401 if fetched from huggingface.co (bug B2); missing file → server start fail; scale 0 = original refusals.

```
┌──────────────────────────────────────┐
│ bonsai-abliterate-lora.gguf          │
│ 9682464 bytes                        │
│ sha256 f1669534...7f42               │
│ arg.cpp:4088 --lora-scaled           │
└──────────────────┬───────────────────┘
                   │ scale 1.0
                   ▼
┌──────────────────────────────────────┐
│ GET /lora-adapters                   │
│ id=0 path=...lora.gguf scale=1.0     │
└──────────────────────────────────────┘
```

### Module 6 — OpenAI HTTP + alias

Interfaces: `-a/--alias` at `common/arg.cpp:4134-4143` inserts into `params.model_alias`. `/v1/models` data[0].id. Data flow: client `{model:"bonsai-qwen"}` → completions. Failure modes: wrong model id 404; pointing Grok connector here (forbidden — that is web-shell :8787).

```
┌──────────────────────────────────────┐
│ -a bonsai-qwen                       │
│ arg.cpp:4134                         │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ GET /v1/models  id=bonsai-qwen       │
│ n_ctx=262144 n_vocab=248320          │
│ total_slots=1                        │
└──────────────────────────────────────┘
```

### Module 7 — Tailnet vs funnel (orthogonal)

Funnel `/` remains web-shell. Bonsai is a raw TCP port on the tailnet IP, not a funnel path.

```
┌──────────────────────────────────────┐
│ funnel 443                           │
│ / proxy 127.0.0.1:4050               │
│ web-shell MCP  NOT bonsai            │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ tailnet IP 100.120.45.9:18081        │
│ GET /health {"status":"ok"}          │
└──────────────────────────────────────┘
```

`tailscale serve status` this turn:

```
https://leviathanlocal.tail73d29c.ts.net (Funnel on)
|-- / proxy http://127.0.0.1:4050
```

---

## THE BUG LEDGER

| # | bug | root cause | fix | evidence |
|---|---|---|---|---|
| B1 | first hub start exit 1 | port 8080 already LISTEN 0.0.0.0 | bind 18081 | journal `couldn't bind HTTP server socket, hostname: 127.0.0.1, port: 8080` |
| B2 | LoRA fetch HTTP 401 | wrong host (HF gated) | GitHub raw | `sha256sum` = f1669534... matches Continuum README |
| B3 | VRAM estimate 9.70 GB vs live 10.22 GB | omitted CUDA context + FA workspace | report live nvidia-smi | 10465 MiB used |
| B4 | GGML_CUDA_FA_ALL_QUANTS warning | deprecated cmake flag | still maps to FA_QUANTS=all | cmake log `deprecated, use GGML_CUDA_FA_QUANTS=all` |
| B5 | hub persist vs systemd | two supervisors | systemd --user only | unit enabled+active; hub bonsai-seat stopped |

### B1 detail

```
┌──────────────────────────────────────────────────────────┐
│ B1 port 8080 bind                                        │
│ mechanism: llama_server start() bind() EADDRINUSE        │
│ first hub start: failed exit=1 uptime=136ms              │
│ log: couldn't bind HTTP server socket ... port: 8080     │
│ fix: --port 18081 --host 0.0.0.0  (unit:18)              │
│ verify: journal listening on http://0.0.0.0:18081        │
│         curl /health {"status":"ok"}                     │
└──────────────────────────────────────────────────────────┘
```

### B2 detail

```
┌──────────────────────────────────────────────────────────┐
│ B2 HF 401 vs GitHub LoRA                                 │
│ mechanism: huggingface.co/Continuum-AI-Corp/... = 401    │
│            HF_TOKEN unset, tokenfile absent              │
│            GitHub raw is public 9.3 MiB GGUF             │
│ fix: curl GitHub raw → models/bonsai/...lora.gguf        │
│ verify: sha256 f1669534803d340a496015f5c45125f3437b4d13  │
│         ec764f40e34488ce83967f42                         │
│         /lora-adapters scale=1.0                         │
└──────────────────────────────────────────────────────────┘
```

### B3 detail

```
┌──────────────────────────────────────────────────────────┐
│ B3 VRAM budget vs nvidia-smi                             │
│ plan math: 5.95 + 3.25 + 0.50 = 9.70 GB                  │
│ live: nvidia-smi memory.used = 10465 MiB (10.22 GiB)     │
│ delta: +0.52 GiB CUDA context / FA / meansub tables      │
│ still under 12282 MiB; free 1412 MiB                     │
│ no host-RAM spill observed                               │
└──────────────────────────────────────────────────────────┘
```

### B4 detail

```
┌──────────────────────────────────────────────────────────┐
│ B4 cmake deprecation                                     │
│ GGML_CUDA_FA_ALL_QUANTS is deprecated                    │
│ ggml/cmake/common.cmake:59 warning                       │
│ still sets FA_QUANTS=all so turbo FA instances built     │
│ strings libggml-cuda.so | grep TURBO_FA → TURBO_FA       │
│ not a runtime fail; leftover cmake flag                  │
└──────────────────────────────────────────────────────────┘
```

### B5 detail

```
┌──────────────────────────────────────────────────────────┐
│ B5 two supervisors                                       │
│ hub persist bonsai-seat AND systemd unit would race      │
│ hub stop bonsai-seat then systemd enable --now           │
│ verify: one PTQ1_0 argv in pgrep (pid 2063850 only)      │
└──────────────────────────────────────────────────────────┘
```

---

## THE TESTING LEDGER

No `bun test` / tsc battery exists for this seat (C++/CUDA host service). Tests are host-live HTTP + systemd + nvidia-smi. Container SHA: none. ocr scan: not run (honest gap).

| test | scope | how | result |
|---|---|---|---|
| systemd active | unit | `systemctl --user is-active` | `active` |
| systemd enabled | unit | `systemctl --user is-enabled` | `enabled` |
| health | HTTP | `curl -sf http://127.0.0.1:18081/health` | `{"status":"ok"}` |
| models id | OpenAI | `curl /v1/models` python print id | `bonsai-qwen` |
| lora scale | adapter | `curl /lora-adapters` | scale 1.0 |
| tailnet health | 100.120.45.9:18081 | curl --max-time 2 | `{"status":"ok"}` |
| single 27B | process | `pgrep -af PTQ1_0` | 1 pid 2063850 |
| beellama gone | disk | `test ! -e ~/beellama.cpp` | GONE |
| funnel intact | tailscale | `tailscale serve status` | `/ proxy 127.0.0.1:4050` |
| binary commit | --version | llama-server --version | 1d0f493 |
| chat smoke | POST completions | model=bonsai-qwen max_tokens=8 | HTTP 200, kv_bpv 3.25 |
| LoRA sha | file | sha256sum | f1669534...7f42 |
| GGUF sha | file | sha256sum | 53107f53...ee3 |

Pasted health:

```
curl -sf http://127.0.0.1:18081/health
{"status":"ok"}
```

Pasted lora-adapters:

```
[{"id":0,"path":"/home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf","scale":1.0,"task_name":"","prompt_prefix":""}]
```

Pasted nvidia-smi:

```
name, memory.used [MiB], memory.free [MiB], memory.total [MiB]
NVIDIA GeForce RTX 4080 Laptop GPU, 10465 MiB, 1412 MiB, 12282 MiB
```

Pasted pgrep (one process):

```
2063850 /home/leviathan/buun-llama-cpp/build/bin/llama-server -m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf --lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0 -a bonsai-qwen -c 262144 -np 1 -ctk turbo3_tcq -ctv turbo3_tcq -ngl 99 -fa on --kv-unified --no-mmproj --host 0.0.0.0 --port 18081 --jinja --reasoning on --reasoning-effort xhigh --temp 1.0 --top-p 0.95 --top-k 20
```

Script tests: none (N/A). Container tests: none (N/A). Regression sweep: none (first series report). Token/lint/purity: none (N/A).

Fails named: B1 (8080 bind, fixed), B2 (HF 401, fixed via GitHub), B3 (VRAM estimate short), B4 (cmake deprecation, non-fatal), B5 (dual supervisor, resolved). No claim that "everything works" without those five.

---

## THE SPEC MANDATE → ENGINEERING MAP

| the operator said | what was built | evidence |
|---|---|---|
| 1 seat max quality turbo3 | `-ctk turbo3_tcq -ctv turbo3_tcq` `-np 1` | unit:15-16; props vbr.enabled=false |
| OrcaRouter uncensored | `--lora-scaled ...:1.0` | /lora-adapters scale=1.0 |
| named server not localhost-whatever | `-a bonsai-qwen` systemd SyslogIdentifier | /v1/models id=bonsai-qwen |
| plugin OpenAI | 0.0.0.0:18081 /v1 | health + models + completions |
| Tailnet not funnel MCP | port 18081; funnel / untouched | tailscale serve `/ proxy :4050` |
| no HF login for LoRA | GitHub raw download | sha256 f1669534... |
| delete beellama | `rm -rf ~/beellama.cpp` | test ! -e → GONE |
| never second 27B | one PTQ1_0 argv | pgrep count 1 |
| no VBR turbo2 Docker mmproj | flags `--no-mmproj` no -ct vbr | props + argv |
| n_ctx 262144 | `-c 262144` | props n_ctx 262144 |
| reasoning max | `--reasoning on --reasoning-effort xhigh` | argv; smoke reasoning_content |

---

## THE NUMBERS

```
┌────────────────────────┬──────────────────────────────┐
│ metric                 │ measured                     │
├────────────────────────┼──────────────────────────────┤
│ tsc exit               │ N/A (no TS tree)             │
│ bun test / battery     │ N/A 0 tests                  │
│ expect count           │ N/A                          │
│ test count host-live   │ 12 checks this turn          │
│ file count unit        │ 1 (29 lines)                 │
│ ggml.h lines           │ 3243  wc -l                  │
│ arg.cpp lines          │ 6041  wc -l                  │
│ llama-server bytes     │ 17888                        │
│ libggml-cuda bytes     │ 166570104                    │
│ GGUF bytes             │ 5946648928                   │
│ LoRA bytes             │ 9682464                      │
│ GGUF sha256            │ 53107f53...ee3               │
│ LoRA sha256            │ f1669534...7f42              │
│ git commit             │ 1d0f493c73817176f6953069...  │
│ n_ctx                  │ 262144                       │
│ n_vocab                │ 248320                       │
│ n_slots                │ 1                            │
│ kv_bpv (chat timings)  │ 3.25                         │
│ VRAM used/total MiB    │ 10465 / 12282                │
│ VRAM free MiB          │ 1412                         │
│ MainPID                │ 2063850                      │
│ PTQ1_0 process count   │ 1                            │
│ container SHA          │ none                         │
│ live receipt count     │ 12 HTTP/sysctl/smi this turn │
│ listening journal hits │ 1                            │
│ turbo3_tcq type id     │ 46  ggml.h:438               │
│ PTQ1_0 type id         │ 91  ggml.h:495               │
└────────────────────────┴──────────────────────────────┘
```

Commands for the table: `wc -l`, `wc -c`, `sha256sum`, `git log -1`, `nvidia-smi --query-gpu=...`, `curl -sf`, `systemctl --user show -p MainPID`, `pgrep -af PTQ1_0`.

---

## THE FILE MANIFEST

```
┌──────────────────────────────────────────────────────────────────┐
│ landing zone                                                     │
├──────────────────────────────────────────────────────────────────┤
│ ├─ ~/.config/systemd/user/llama-bonsai-qwen.service  NEW 29 ln   │
│ ├─ ~/buun-llama-cpp/                         EXISTING clone      │
│ │  └─ build/bin/llama-server                 NEW build 17888 B   │
│ │  └─ build/bin/libggml-cuda.so.0.24.0       NEW 166570104 B     │
│ ├─ ~/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf  NEW 5.95G   │
│ ├─ ~/models/bonsai/bonsai-abliterate-lora.gguf       NEW 9.3M    │
│ ├─ reports/Bonsai_Ternary2_27B_Seat_ShowMe.md        EXISTING    │
│ └─ reports/Bonsai_Qwen_Local_Provider_Engineering_   NEW this    │
│    Report_v1.md                                                  │
│ DELETED: ~/beellama.cpp                          GONE            │
│ UNTOUCHED: tailscale funnel / → :4050                            │
└──────────────────────────────────────────────────────────────────┘
```

shown=8 total=8 capped=0

---

## RUNTIME TRACE (5 parts)

TIMELINE (context sources converging)
═══════════════════════════════════════════════════════════

T-hours: PTQ1_0 downloaded (hf download prism-ml/...PTQ1_0.gguf)
         LoRA from GitHub raw (not HF)
         cmake CUDA 12.8 ARCH=89 built llama-server 1d0f493
T-now:   systemd unit enabled, pid 2063850 listening :18081

STEP 1: CLIENT CALLS MODELS
┌─────────────────────────────────────────────────────┐
│ curl -sf http://127.0.0.1:18081/v1/models           │
│ → data[0].id = "bonsai-qwen"                        │
│ → meta.n_ctx = 262144                               │
│ → meta.vbr.enabled = false                          │
│ → meta.vbr.entry_type_k = turbo3_tcq                │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
STEP 2: CLIENT CALLS COMPLETIONS
┌─────────────────────────────────────────────────────┐
│ POST /v1/chat/completions                           │
│ args: { model: "bonsai-qwen",                       │
│         messages: [{role:user, content:"Say READY"}]│
│         max_tokens: 8 }                             │
│ → HTTP 200  model field "bonsai-qwen"               │
│ → timings.kv_bpv = 3.25                             │
│ → reasoning_content non-empty (thinking on)         │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
STEP 3: LORA PATH
┌─────────────────────────────────────────────────────┐
│ GET /lora-adapters                                  │
│ → [{id:0, path:.../bonsai-abliterate-lora.gguf,     │
│     scale:1.0}]                                     │
└─────────────────────────────────────────────────────┘

STEP 4: ROUTING CHECK
┌─────────────────────────────────────────────────────┐
│ Q1: Is this funnel MCP? → NO (port 18081, not 443)  │
│ Q2: Second PTQ1_0? → NO (pgrep count 1)             │
│ Q3: VBR degrade? → NO (enabled=false)               │
│ Q4: Port 8080? → NO (B1, using 18081)               │
│ → llama-server handles Chat Completions directly    │
│ IF Grok connector: would be :8787 /mcp NOT here     │
└─────────────────────────────────────────────────────┘

STEP 5: DELIVER
┌─────────────────────────────────────────────────────┐
│ JSON choices[0].message                             │
│ OpenAI clients use baseURL ...:18081/v1             │
│ model id bonsai-qwen  apiKey local                  │
│ Tailnet same path on 100.120.45.9:18081             │
└─────────────────────────────────────────────────────┘

OBSERVER ACTIVITY
═════════════════════════════════════════════════════════
  nvidia-smi 10465 MiB used / 1412 MiB free
  systemd Restart=always watching pid 2063850
  funnel / still :4050 — ZERO funnel rewrite
  quiet case: no second 27B, no VBR degrade log,
  no host-RAM spill warning in journal this turn
  Result: ZERO extra PTQ1_0 processes. ZERO funnel change.

Escalation path (not taken): if used >= 12000 MiB → stop
unit, do not start a second process, do not enable VBR.

---

## CALL TREE (HTTP hot path)

```
┌────────────────────────┐
│ POST /v1/chat/completions│
│ tools/server :18081    │
└───────────┬────────────┘
            │
            ▼
  alias match bonsai-qwen
  common/arg.cpp:4134
    apply LoRA scale 1.0
    arg.cpp:4097
      encode KV turbo3_tcq
      ggml.h:438
        fattn-mma-turbo decode
        libggml-cuda.so TURBO_FA
          JSON choices + timings
```

shown=6 total=6 capped=0

---

## MECHANISM (load)

```
┌────────────────────────────────────────┐
│ on(systemd start)                      │
│   exec llama-server argv               │
│   mmap PTQ1_0                          │
│   if LoRA file missing                 │
│     exit non-zero  systemd restart     │
│   apply lora scale 1.0                 │
│   pin K and V turbo3_tcq               │
│   skip mmproj                          │
│   bind 0.0.0.0:18081                   │
│   if bind fail                         │
│     log EADDRINUSE  exit 1             │
│   listen  emit model loaded            │
└────────────────────────────────────────┘
```

---

## WHAT'S NEEDED FROM THE OPERATOR

1. Plugin any OpenAI client: `baseURL http://127.0.0.1:18081/v1` model `bonsai-qwen` apiKey `local`. This report does not edit `~/.omp/agent/config.yml` (providers block has no custom OpenAI stanza; changing default model was not in the pin).
2. Tailnet clients: `http://100.120.45.9:18081/v1` — not `https://leviathanlocal.tail73d29c.ts.net/mcp`.
3. P0 none for the pin VERIFY set (all four VERIFY commands green this turn).
4. Ruling: whether to register `provider.bonsai` in OMP/MiMo global config (not done).
5. Ruling: whether to add `--api-key` (CORS currently allow-all, no key; journal warns security risk).
6. Do not restore `~/beellama.cpp`. Do not steal funnel `/`.

---

## HONEST GAPS

| gap | status |
|---|---|
| context_management/CURRENT_STATE.md | missing (ls: No such file) |
| .trident/container-test-results.json | missing; no container SHA |
| ocr / qwen-code-audit on fork src | not run |
| bun test / tsc | N/A |
| long 262144-token fill | not executed (would take long; VRAM 1412 MiB free) |
| LoRA behavioral refusal A/B | not measured (only load+scale receipt) |
| MagicDNS hostname curl | only 100.120.45.9 and 127.0.0.1 proven |
| OMP provider JSONC write | not applied |
| cmake FA_ALL_QUANTS deprecation | leftover, non-fatal |
| VRAM vs 9.70 GB napkin | live 10465 MiB is source of truth |

---

## TRIED AND REJECTED

| try | reject reason | evidence |
|---|---|---|
| HF download LoRA | HTTP 401 | curl -sI huggingface.co/Continuum-AI-Corp/... |
| hub persist on :8080 | EADDRINUSE | journal bind 8080 |
| two concurrent 27B | weights 2x 5.95 GiB > 12 GiB | math + pin DO NOT |
| VBR / turbo2 for drive seat | quality for coding | pin DO NOT |
| funnel / → 18081 | would kill web-shell MCP | tailscale serve still :4050 |
| Docker mahald image | operator: no Docker | pin DO NOT |

---

## SELF-CHECK

Mode: EXPLAIN (the seat exists). Ingredients loaded: show-me, ascii-diagrams, ascii-graph, ascii-visualize, ascii-simulator, engineering-report-full-density, engineering-report-slop-detectors. Box widths 68 and 54 counted. Every metric has a command. Five bugs have mechanism boxes. Runtime trace has 5 parts. Gaps named. Chat content = this file.

Slop detectors: no bullet-only architecture; bugs have mechanisms; tests paste curl/smi; modules have diagrams; metrics have commands; fails B1-B5 named; trace 5 parts; this file is well over 200 lines (`wc -l` after write).
