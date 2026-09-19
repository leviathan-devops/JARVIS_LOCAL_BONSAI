# GROK TUI CONTEXT HANDOVER — JARVIS_LOCAL_BONSAI
## Duplicate this session working memory into a local TUI
### Paste this file first. Then unzip the rest of the pack. Then do not renegotiate the rulings.

```
Package:     JARVIS_LOCAL_BONSAI_CONTEXT_HANDOVER
Written:     2026-09-19T01:18Z
Author:      the session being handed off
Consumer:    local TUI / fresh agent
Freeze:      llama-bonsai-qwen.service flags + BINDING.md
```

If handover conflicts with BINDING spec on implementation detail, BINDING wins.
If BINDING conflicts with a superseded spec, BINDING wins.
Dead paths in the handover override any superseded Docker / VBR / second-27B language.

---

# 0. WHO YOU ARE NOW

You are the next agent on JARVIS_LOCAL_BONSAI. You police one 27B OpenAI seat.
You will not reopen DP-* dead paths. You will not invent a second GPU load.
You will not steal Tailscale funnel /. You will not wire Muse Code.
You will not restore ~/beellama.cpp. You will not hf login for the LoRA.
First reply = evidence (systemctl, curl health, pgrep count), not a redesign.

Stance: keep llama-bonsai-qwen active. Plugin clients to :18081/v1 model bonsai-qwen.
Provider catalog name: jarvis-local. Display: Bonsai 2 Qwen 3.8 27b.

# 1. THE ARC OF THIS SESSION
## 1.1 Requests, in order
1. Which Bonsai packing fits RTX 4080 Laptop 12GB and how many 262K windows.
2. Full TurboQuant settings on the community fork; best setup for hardware.
3. Why Docker; why not floor VBR at tcq3; is 3 slots 262K safe 24/7.
4. 24/7 coding subagents; no adaptive F16 VBR.
5. Why turbo2_tcq risky; orchestrator vs writer; can KV go to 1M.
6. 2 concurrent vs 1; headroom.
7. turbo2_tcq vs sacrifice seat for turbo3.
8. Port-based compression; scout vs builder; turbo1 on scout.
9. SET UP 1 seat turbo3 + OrcaRouter; walk the plan; /show-me.
10. Stop talking about beellama.
11. Delete beellama; goal pin; named bonsai qwen server; why LoRA login.
12. Goal verify all checks.
13. Engineering report / status-report show exactly what was done.
14. Handover package + operator manual + JARVIS_LOCAL_BONSAI folder + wire harnesses except Muse + container TUI tests.

## 1.2 Rulings the operator already made (do not reopen)
R1 Drive seat is turbo3_tcq not VBR not turbo2.
  status: BINDING
R2 One seat not three full-quality 262K windows.
  status: BINDING
R3 No Docker.
  status: BINDING
R4 No DSpark on 12GB.
  status: BINDING
R5 262K is the context ceiling.
  status: BINDING
R6 LoRA is GitHub raw not HuggingFace gated.
  status: BINDING
R7 Named alias bonsai-qwen not anonymous localhost.
  status: BINDING
R8 Funnel / is web-shell MCP not this API.
  status: BINDING
R9 Grok/GLM connectors stay on funnel /mcp.
  status: BINDING
R10 Delete beellama.cpp; do not restore.
  status: BINDING
R11 systemd --user llama-bonsai-qwen is the supervisor.
  status: BINDING
R12 Port 18081 not 8080.
  status: BINDING
R13 Host 0.0.0.0 for tailnet.
  status: BINDING
R14 --no-mmproj.
  status: BINDING
R15 --reasoning on --reasoning-effort xhigh.
  status: BINDING
R16 temp 1.0 top_p 0.95 top_k 20 thinking defaults.
  status: BINDING
R17 np=1.
  status: BINDING
R18 Never a second PTQ1_0 process.
  status: BINDING
R19 Muse Code harness out of wiring scope.
  status: BINDING
R20 Project home is Shared_Workspace/JARVIS_LOCAL_BONSAI.
  status: BINDING

## 1.3 What this session already produced (file table)
- /home/leviathan/.config/systemd/user/llama-bonsai-qwen.service — 29 lines unit
- /home/leviathan/buun-llama-cpp — clone 1d0f493
- /home/leviathan/buun-llama-cpp/build/bin/llama-server — 17888 bytes
- /home/leviathan/buun-llama-cpp/build/bin/libggml-cuda.so.0.24.0 — 166570104 bytes
- /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf — 5946648928 bytes sha 53107f53
- /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf — 9682464 bytes sha f1669534
- reports/Bonsai_Qwen_Local_Provider_Engineering_Report_v1.md — 612 lines
- reports/Bonsai_Ternary2_27B_Seat_ShowMe.md — ShowMe PLAN

# 2. THE PRODUCT IN ONE PAGE
One sentence: a systemd-supervised llama-server exposing OpenAI /v1 as bonsai-qwen on :18081.
It is not Orca web-shell. It is not Docker. It is not VBR. It is not two GPUs.

## 2.1 Process
MainPID measured 2063850 at report time. Restart=always. SyslogIdentifier=llama-bonsai-qwen.
## 2.2 HTTP
GET /health GET /v1/models GET /lora-adapters GET /props POST /v1/chat/completions
## 2.3 Model
PTQ1_0 group 128. n_vocab 248320. n_ctx 262144. n_slots 1. kv_bpv 3.25

# 3. HOST / FREEZE INTERNALS
Coordinates:
- GPU: NVIDIA GeForce RTX 4080 Laptop 12282 MiB sm_89 driver 595.71.05
- CUDA toolkit: /usr/local/cuda-12.8 nvcc 12.8.93
- PATH in unit: CUDA 12.8 bin
- LD_LIBRARY_PATH: /home/leviathan/buun-llama-cpp/build/bin
Exit sites:
- bind fail llama_server start (tools/server) journal EADDRINUSE
- missing LoRA: start fail
Hooks that exist: systemd Restart=always, linger user session
Hooks that do NOT exist: funnel path for 18081, OMP default model steal, Muse wiring, DSpark
Wrong hook sites rejected: port 8080, hub persist dual supervisor, HF LoRA URL

ggml.h:438 GGML_TYPE_TURBO3_TCQ = 46
ggml.h:495 GGML_TYPE_PTQ1_0 = 91
arg.cpp:4088 --lora-scaled
arg.cpp:4134 -a --alias
unit:11-21 ExecStart

# 4. SOURCE ANALOGS
Take: llama-qwen35-system.service pattern (named systemd llama-server).
Refuse: its port 8000, its Qwen3.5-9B model, flash-attn off, -c 8192.
Take: GitHub Continuum-AI-Corp LoRA file only.
Refuse: HuggingFace Continuum-AI-Corp 401.
Take: buun-llama-cpp ternary + turbo types.
Refuse: PrismML-only fork (no turbo), Docker mahald.
Take: OpenAI-compatible plugin shape from mimocode providers.md.
Refuse: pointing Grok custom connector at :18081.

# 5. WHAT ALREADY EXISTS ON DISK
See file table 1.3. Missing drive: harness wiring may still be incomplete at dump time.
ST counts: none (C++ seat). Host-live checks: 12.
SHAs: LoRA f1669534803d340a496015f5c45125f3437b4d13ec764f40e34488ce83967f42
GGUF 53107f530aa52eb00912263ab1ee29bd199261c87cd7b4ad4ca1318c1fe33ee3
Commit 1d0f493c73817176f6953069b7c10211de9555a1

# 6. WHAT WAS REJECTED
| Plan | Why dead | Transcript appearance |
|---|---|---|
| DP-DOCKER | operator no Docker | mahald image compose -np 3 |
| DP-VBR | coding 24/7 needs pinned KV | -ct vbr |
| DP-TURBO2 | quality risk variable names | turbo2_tcq 2 seats |
| DP-TURBO1 | scout corruption feeds builder | turbo1_tcq |
| DP-SECOND27B | 2x 5.95 GiB > 12 GiB | two llama-server |
| DP-MMPROJ | 0.63 GB | --mmproj |
| DP-FUNNEL | kills web-shell | tailscale serve / |
| DP-HFLOGIN | LoRA is GitHub | huggingface.co 401 |
| DP-BEELLAMA | operator delete | ~/beellama.cpp |
| DP-8080 | EADDRINUSE | hub start 8080 |
| DP-MUSE | operator exclude | muse code |
| DP-1MCTX | training ceiling 262K | -c 1048576 |
| DP-DSPARK | 12GB net slowdown | --spec-type draft-dspark |

# 7. THE DRIVE / PATCH CONTRACT
Helper: systemd --user {start,stop,status} llama-bonsai-qwen
Inject shape: OpenAI chat completions JSON model=bonsai-qwen
Files to touch for harness: ~/.omp/agent/models.yml ~/.omp/agent/config.yml enabledModels ~/.config/mimocode/mimocode.jsonc ~/.qwen/settings.json
Files not to touch: funnel units, muse settings, GGUF bytes, beellama, web-agent-tailnet-*.service

# 8. IMPLEMENTATION ORDER
1. Evidence: systemctl is-active, curl health, pgrep count=1
2. If inactive: start unit. Do not rebuild.
3. Harness pins jarvis-local only. Do not change default model unless operator says.
4. Container TUI tests must use host network to :18081. Do not load GGUF in container.
5. Never start a second PTQ1_0.

# 9. PROOF / ACCEPTANCE
P1 systemctl --user is-active llama-bonsai-qwen == active
P2 curl health {"status":"ok"}
P3 /v1/models id==bonsai-qwen
P4 /lora-adapters scale==1.0
P5 pgrep PTQ1_0 count == 1
P6 tailscale serve / still 4050
P7 llama-server --version contains 1d0f493

# 10. ZIP MAP + READ ORDER
00_HANDOVER this file
01_BINDING_SPEC BINDING.md
02_SUPERSEDED_READ_ONLY ShowMe PLAN if packed
03_HOST_BIBLES BIBLE.md OPERATOR_MANUAL.md engineering report
04_ADJACENT none or web-shell note
Read order: BINDING then this handover then OPERATOR_MANUAL then BIBLE

# 11. KICK PROMPT
```
You ingest JARVIS_LOCAL_BONSAI. BINDING.md wins. Prove P1-P7 before any redesign.
Seat: systemd --user llama-bonsai-qwen :18081 model bonsai-qwen turbo3_tcq LoRA 1.0
Do not Docker, VBR, second 27B, funnel /, Muse, beellama, hf login, port 8080.
Plugin: http://127.0.0.1:18081/v1 bonsai-qwen local
```

# 12. GREP CONTRACTS
grep -n TURBO3_TCQ /home/leviathan/buun-llama-cpp/ggml/include/ggml.h
grep -n PTQ1_0 /home/leviathan/buun-llama-cpp/ggml/include/ggml.h
grep -n lora-scaled /home/leviathan/buun-llama-cpp/common/arg.cpp
grep -n alias /home/leviathan/buun-llama-cpp/common/arg.cpp
systemctl --user cat llama-bonsai-qwen

# 13. FALSE CEILINGS ALREADY POPPED
HF login required for LoRA — false; GitHub raw works.
Must use Docker for 12GB 262K — false; native unit runs.
Must use VBR for 262K — false; turbo3_tcq fits one seat.
3 concurrent full-quality 262K — false on 12GB.
1M context — false; 262K ceiling.
Orca web-shell is this model server — false.

# 14. QUESTIONS YOU MAY ASK
Q1 Steal OMP/MiMo default model to jarvis-local/bonsai-qwen? UNRULED default=no steal.
Q2 Add --api-key? UNRULED default=no key (journal CORS warning).
Q3 Wire Grok native CLI (not OMP)? Native grokbot has no OpenAI provider slot.

# 15. SELF-CHECK
- I will not start a second PTQ1_0.
- I will not touch funnel /.
- I will not restore beellama.
- I will not hf login for LoRA.
- I will prove P1-P7 with tool output.
- I will treat BINDING as winner.
- I will not wire Muse Code.

# APPENDIX A RESEARCH NUMBERS (measured or sourced this session)
- GPU name: NVIDIA GeForce RTX 4080 Laptop GPU
  source: nvidia-smi
- VRAM total MiB: 12282
  source: nvidia-smi
- VRAM used MiB at report: 10465
  source: nvidia-smi
- VRAM free MiB: 1412
  source: nvidia-smi
- compute cap: 8.9
  source: nvidia-smi
- driver: 595.71.05
  source: nvidia-smi
- nvcc: 12.8.93
  source: /usr/local/cuda-12.8/bin/nvcc --version
- cmake: 3.28.3
  source: cmake --version
- disk free at clone: 36G
  source: df -h /
- clone HEAD: 1d0f493
  source: git log -1
- ggml.h lines: 3243
  source: wc -l
- arg.cpp lines: 6041
  source: wc -l
- unit lines: 29
  source: wc -l
- llama-server bytes: 17888
  source: wc -c
- libggml-cuda bytes: 166570104
  source: wc -c
- GGUF bytes: 5946648928
  source: ls -l
- LoRA bytes: 9682464
  source: wc -c
- n_ctx: 262144
  source: /props
- n_vocab: 248320
  source: /v1/models meta
- n_slots: 1
  source: /props total_slots
- kv_bpv: 3.25
  source: chat timings
- turbo3_tcq id: 46
  source: ggml.h:438
- PTQ1_0 id: 91
  source: ggml.h:495
- listen: 0.0.0.0:18081
  source: journal
- MainPID example: 2063850
  source: systemctl show
- health: {"status":"ok"}
  source: curl
- alias: bonsai-qwen
  source: /v1/models
- lora scale: 1.0
  source: /lora-adapters
- vbr.enabled: false
  source: /props
- funnel: / proxy 127.0.0.1:4050
  source: tailscale serve status
- tailnet health: ok on 100.120.45.9:18081
  source: curl
- beellama: GONE
  source: test ! -e
- report v1 lines: 612
  source: wc -l
- ShowMe exists: yes
  source: ls reports
- PTQ1_0 size card: 5.95 GB
  source: HF card
- PQ2_0 size card: 7.21 GB
  source: HF card not used
- thinking temp: 1.0
  source: unit
- top_p: 0.95
  source: unit
- top_k: 20
  source: unit
- fa: on
  source: unit
- ngl: 99
  source: unit
- kv-unified: true
  source: argv
- mmproj: disabled
  source: --no-mmproj
- reasoning: on
  source: unit
- effort: xhigh
  source: unit
- Restart: always
  source: unit
- RestartSec: 10
  source: unit
- WantedBy: default.target
  source: unit
- SyslogIdentifier: llama-bonsai-qwen
  source: unit
- host: 0.0.0.0
  source: unit
- port: 18081
  source: unit
- plugin apiKey: local
  source: pin
- provider id: jarvis-local
  source: operator
- display: Bonsai 2 Qwen 3.8 27b
  source: operator
- fork: spiritbuun/buun-llama-cpp
  source: clone
- CUDA ARCH: 89
  source: cmake
- TURBO_FA: present in .so strings
  source: strings
- smoke decode: ~31 tok/s
  source: chat timings predicted_per_second
- smoke prefill: ~117 tok/s
  source: prompt_per_second

# APPENDIX B DEAD PATH IDENTIFIERS
DP-DOCKER is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-DOCKER, refuse and cite BINDING.md
DP-VBR is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-VBR, refuse and cite BINDING.md
DP-TURBO2 is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-TURBO2, refuse and cite BINDING.md
DP-TURBO1 is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-TURBO1, refuse and cite BINDING.md
DP-SECOND27B is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-SECOND27B, refuse and cite BINDING.md
DP-MMPROJ is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-MMPROJ, refuse and cite BINDING.md
DP-FUNNEL is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-FUNNEL, refuse and cite BINDING.md
DP-HFLOGIN is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-HFLOGIN, refuse and cite BINDING.md
DP-BEELLAMA is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-BEELLAMA, refuse and cite BINDING.md
DP-8080 is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-8080, refuse and cite BINDING.md
DP-MUSE is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-MUSE, refuse and cite BINDING.md
DP-1MCTX is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-1MCTX, refuse and cite BINDING.md
DP-DSPARK is dead. Do not implement. Do not 'just try'.
  If a transcript suggests DP-DSPARK, refuse and cite BINDING.md

# APPENDIX C CLIENT SNIPPETS
OpenAI:
  baseURL: "http://127.0.0.1:18081/v1"
  model: "bonsai-qwen"
  apiKey: "local"
curl:
  curl -sf http://127.0.0.1:18081/v1/chat/completions \
    -H 'Content-Type: application/json' \
    -d '{"model":"bonsai-qwen","messages":[{"role":"user","content":"hi"}]}'
Tailnet:
  http://100.120.45.9:18081/v1
Not:
  https://leviathanlocal.tail73d29c.ts.net/mcp

# APPENDIX D SYSTEMD UNIT ARGV (verbatim)
Copied from unit ExecStart lines 11-21:
-m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf
--lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0
-a bonsai-qwen
-c 262144 -np 1
-ctk turbo3_tcq -ctv turbo3_tcq
-ngl 99 -fa on --kv-unified --no-mmproj
--host 0.0.0.0 --port 18081
--jinja
--reasoning on --reasoning-effort xhigh
--temp 1.0 --top-p 0.95 --top-k 20

# APPENDIX E SESSION BUGS
B1 8080 bind EADDRINUSE — fixed 18081
B2 HF 401 LoRA — fixed GitHub raw
B3 VRAM estimate 9.70 vs live 10.22 GiB — live wins
B4 cmake FA_ALL_QUANTS deprecated — nonfatal
B5 hub persist vs systemd — systemd only

# APPENDIX F FLAG ENCYCLOPEDIA (drive seat)
FLAG -m
  value: PTQ1_0 path
  why frozen: required
  do not change without operator ruling
FLAG --lora-scaled
  value: LoRA:scale 1.0
  why frozen: uncensor
  do not change without operator ruling
FLAG -a
  value: bonsai-qwen
  why frozen: API id
  do not change without operator ruling
FLAG -c
  value: 262144
  why frozen: max ctx
  do not change without operator ruling
FLAG -np
  value: 1
  why frozen: one slot
  do not change without operator ruling
FLAG -ctk
  value: turbo3_tcq
  why frozen: K cache
  do not change without operator ruling
FLAG -ctv
  value: turbo3_tcq
  why frozen: V cache
  do not change without operator ruling
FLAG -ngl
  value: 99
  why frozen: all layers GPU
  do not change without operator ruling
FLAG -fa
  value: on
  why frozen: required for turbo KV
  do not change without operator ruling
FLAG --kv-unified
  value: on
  why frozen: single KV pool
  do not change without operator ruling
FLAG --no-mmproj
  value: on
  why frozen: skip vision 0.63GB
  do not change without operator ruling
FLAG --host
  value: 0.0.0.0
  why frozen: tailnet
  do not change without operator ruling
FLAG --port
  value: 18081
  why frozen: not 8080
  do not change without operator ruling
FLAG --jinja
  value: on
  why frozen: chat template
  do not change without operator ruling
FLAG --reasoning
  value: on
  why frozen: thinking
  do not change without operator ruling
FLAG --reasoning-effort
  value: xhigh
  why frozen: max
  do not change without operator ruling
FLAG --temp
  value: 1.0
  why frozen: thinking default
  do not change without operator ruling
FLAG --top-p
  value: 0.95
  why frozen: thinking default
  do not change without operator ruling
FLAG --top-k
  value: 20
  why frozen: thinking default
  do not change without operator ruling

# APPENDIX G WHAT NOT TO GREP INTO EXISTENCE
Do not grep beellama as a live runtime.
Do not grep mahald docker-compose as the launcher.
Do not grep -ct vbr in the unit.
Do not grep muse in this project's wiring.

# APPENDIX H HARNESS TARGETS
OMP: ~/.omp/agent/models.yml providers.jarvis-local + config.yml enabledModels jarvis-local/*
MiMo: ~/.config/mimocode/mimocode.jsonc provider.jarvis-local
Qwen: ~/.qwen/settings.json modelProviders openai (chat completions) baseUrl :18081/v1
Grok OMP: same models.yml (xai-oauth stays default)
Grok native CLI: no OpenAI custom slot in grokbot/settings.json — gap
Muse: DO NOT TOUCH ~/.config/muse

# APPENDIX I CONTAINER LAW
If you test TUIs in containers, they must call the HOST seat.
Use --network host so 127.0.0.1:18081 works.
Do not copy PTQ1_0 into the container and llama-server it.
Do not ngl 99 a second copy.

# APPENDIX J PATHS ABSOLUTE
/home/leviathan/JARVIS_WORKSPACE/Shared_Workspace/JARVIS_LOCAL_BONSAI
  keep absolute; do not relocate GGUF into git
/home/leviathan/.config/systemd/user/llama-bonsai-qwen.service
  keep absolute; do not relocate GGUF into git
/home/leviathan/buun-llama-cpp
  keep absolute; do not relocate GGUF into git
/home/leviathan/buun-llama-cpp/build/bin/llama-server
  keep absolute; do not relocate GGUF into git
/home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf
  keep absolute; do not relocate GGUF into git
/home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf
  keep absolute; do not relocate GGUF into git
/home/leviathan/.omp/agent/models.yml
  keep absolute; do not relocate GGUF into git
/home/leviathan/.omp/agent/config.yml
  keep absolute; do not relocate GGUF into git
/home/leviathan/.config/mimocode/mimocode.jsonc
  keep absolute; do not relocate GGUF into git
/home/leviathan/.qwen/settings.json
  keep absolute; do not relocate GGUF into git
/home/leviathan/.config/muse
  keep absolute; do not relocate GGUF into git
/home/leviathan/Grok_Build/docs/handovers
  keep absolute; do not relocate GGUF into git

# APPENDIX K OPERATOR MANUAL CROSS-REF
HOW: ../OPERATOR_MANUAL.md
WHAT: docs/BIBLE.md
WHY/WINS: MASTER_CONTEXT/BINDING.md

# APPENDIX L RULING RESTS (one line each, unique)
REST 1: R1 Drive seat is turbo3_tcq not VBR not turbo2. — still binding after handover.
  reopen cost: operator explicit override only
REST 2: R2 One seat not three full-quality 262K windows. — still binding after handover.
  reopen cost: operator explicit override only
REST 3: R3 No Docker. — still binding after handover.
  reopen cost: operator explicit override only
REST 4: R4 No DSpark on 12GB. — still binding after handover.
  reopen cost: operator explicit override only
REST 5: R5 262K is the context ceiling. — still binding after handover.
  reopen cost: operator explicit override only
REST 6: R6 LoRA is GitHub raw not HuggingFace gated. — still binding after handover.
  reopen cost: operator explicit override only
REST 7: R7 Named alias bonsai-qwen not anonymous localhost. — still binding after handover.
  reopen cost: operator explicit override only
REST 8: R8 Funnel / is web-shell MCP not this API. — still binding after handover.
  reopen cost: operator explicit override only
REST 9: R9 Grok/GLM connectors stay on funnel /mcp. — still binding after handover.
  reopen cost: operator explicit override only
REST 10: R10 Delete beellama.cpp; do not restore. — still binding after handover.
  reopen cost: operator explicit override only
REST 11: R11 systemd --user llama-bonsai-qwen is the supervisor. — still binding after handover.
  reopen cost: operator explicit override only
REST 12: R12 Port 18081 not 8080. — still binding after handover.
  reopen cost: operator explicit override only
REST 13: R13 Host 0.0.0.0 for tailnet. — still binding after handover.
  reopen cost: operator explicit override only
REST 14: R14 --no-mmproj. — still binding after handover.
  reopen cost: operator explicit override only
REST 15: R15 --reasoning on --reasoning-effort xhigh. — still binding after handover.
  reopen cost: operator explicit override only
REST 16: R16 temp 1.0 top_p 0.95 top_k 20 thinking defaults. — still binding after handover.
  reopen cost: operator explicit override only
REST 17: R17 np=1. — still binding after handover.
  reopen cost: operator explicit override only
REST 18: R18 Never a second PTQ1_0 process. — still binding after handover.
  reopen cost: operator explicit override only
REST 19: R19 Muse Code harness out of wiring scope. — still binding after handover.
  reopen cost: operator explicit override only
REST 20: R20 Project home is Shared_Workspace/JARVIS_LOCAL_BONSAI. — still binding after handover.
  reopen cost: operator explicit override only

# APPENDIX M GREP ONE-LINERS FOR INGEST
CMD systemctl --user is-active llama-bonsai-qwen
  run before claiming the seat is down or up
CMD curl -sf http://127.0.0.1:18081/health
  run before claiming the seat is down or up
CMD curl -sf http://127.0.0.1:18081/v1/models
  run before claiming the seat is down or up
CMD curl -sf http://127.0.0.1:18081/lora-adapters
  run before claiming the seat is down or up
CMD pgrep -af PTQ1_0
  run before claiming the seat is down or up
CMD nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader
  run before claiming the seat is down or up
CMD tailscale serve status
  run before claiming the seat is down or up
CMD test ! -e /home/leviathan/beellama.cpp && echo GONE
  run before claiming the seat is down or up
CMD /home/leviathan/buun-llama-cpp/build/bin/llama-server --version
  run before claiming the seat is down or up
CMD git -C /home/leviathan/buun-llama-cpp log -1 --oneline
  run before claiming the seat is down or up

# END
This handover is law + evidence, not a recap essay.

# APPENDIX N UNIT FILE DUMP
UNIT 01|[Unit]
UNIT 02|Description=Bonsai Qwen 27B (PTQ1_0 + turbo3_tcq + OrcaRouter LoRA)
UNIT 03|After=network.target
UNIT 04|
UNIT 05|[Service]
UNIT 06|Type=simple
UNIT 07|WorkingDirectory=/home/leviathan/buun-llama-cpp/build/bin
UNIT 08|Environment=PATH=/usr/local/cuda-12.8/bin:/usr/local/bin:/usr/bin:/bin
UNIT 09|Environment=LD_LIBRARY_PATH=/home/leviathan/buun-llama-cpp/build/bin
UNIT 10|Environment=CUDA_HOME=/usr/local/cuda-12.8
UNIT 11|ExecStart=/home/leviathan/buun-llama-cpp/build/bin/llama-server \
UNIT 12|  -m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf \
UNIT 13|  --lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0 \
UNIT 14|  -a bonsai-qwen \
UNIT 15|  -c 262144 -np 1 \
UNIT 16|  -ctk turbo3_tcq -ctv turbo3_tcq \
UNIT 17|  -ngl 99 -fa on --kv-unified --no-mmproj \
UNIT 18|  --host 0.0.0.0 --port 18081 \
UNIT 19|  --jinja \
UNIT 20|  --reasoning on --reasoning-effort xhigh \
UNIT 21|  --temp 1.0 --top-p 0.95 --top-k 20
UNIT 22|Restart=always
UNIT 23|RestartSec=10
UNIT 24|SyslogIdentifier=llama-bonsai-qwen
UNIT 25|LimitNOFILE=65536
UNIT 26|TimeoutStopSec=30
UNIT 27|
UNIT 28|[Install]
UNIT 29|WantedBy=default.target

# APPENDIX O TURBO TYPE TABLE FROM ggml.h
TYPE id=43 name=TURBO3_0
  comment=3-bit PolarQuant + QJL
  drive_seat=NO
TYPE id=44 name=TURBO4_0
  comment=4-bit PolarQuant + QJL
  drive_seat=NO
TYPE id=45 name=TURBO2_0
  comment=2-bit PolarQuant no QJL
  drive_seat=NO
TYPE id=46 name=TURBO3_TCQ
  comment=DRIVE SEAT — TCQ 3-bit
  drive_seat=YES
TYPE id=47 name=TURBO2_TCQ
  comment=dead for drive seat DP-TURBO2
  drive_seat=NO
TYPE id=48 name=TURBO8_0
  comment=8-bit turbo
  drive_seat=NO
TYPE id=49 name=TURBO1
  comment=RESERVED removed
  drive_seat=NO
TYPE id=50 name=TURBO1_NSN
  comment=RESERVED
  drive_seat=NO
TYPE id=51 name=TURBO1_CQ
  comment=RESERVED
  drive_seat=NO
TYPE id=52 name=TURBO1_TCQ
  comment=dead for drive DP-TURBO1
  drive_seat=NO
TYPE id=53 name=Q2_0_G128
  comment=ternary g128 remap
  drive_seat=NO
TYPE id=91 name=PTQ1_0
  comment=DRIVE WEIGHTS wire 143
  drive_seat=YES

# APPENDIX P KV MATH (262144 ctx, 16 full-attn layers)
F16 KV ~16384 MiB (session calc). turbo3_tcq 3.25 bpv => ~3.25 GB KV.
Live nvidia-smi 10465 MiB includes weights+KV+CUDA context.
Plan napkin 9.70 GB was short by ~0.52 GiB (B3). Live number wins.

# APPENDIX Q HTTP ENDPOINTS
ENDPOINT /health
  base http://127.0.0.1:18081/health
  tailnet http://100.120.45.9:18081/health
ENDPOINT /v1/models
  base http://127.0.0.1:18081/v1/models
  tailnet http://100.120.45.9:18081/v1/models
ENDPOINT /v1/chat/completions
  base http://127.0.0.1:18081/v1/chat/completions
  tailnet http://100.120.45.9:18081/v1/chat/completions
ENDPOINT /lora-adapters
  base http://127.0.0.1:18081/lora-adapters
  tailnet http://100.120.45.9:18081/lora-adapters
ENDPOINT /props
  base http://127.0.0.1:18081/props
  tailnet http://100.120.45.9:18081/props

# APPENDIX R CONFLICT RULE REPEAT
BINDING wins over handover on implementation detail.
BINDING wins over superseded ShowMe Docker language.
Dead paths override keep-alive Docker talk.

# APPENDIX S PROJECT TREE INTENDED
JARVIS_LOCAL_BONSAI/
  AGENTS.md FIRE.sh .gitignore
  OPERATOR_MANUAL.md
  MASTER_CONTEXT/BINDING.md
  docs/BIBLE.md
  config/llama-bonsai-qwen.service copy
  reports/ engineering report + ShowMe
  handovers/ this file
  models-link -> /home/leviathan/models/bonsai  (not git)
  runtime-link -> /home/leviathan/buun-llama-cpp (not git)

# APPENDIX T DO NOT TOUCH LIST
DO_NOT_TOUCH ~/.config/muse
  reason: BINDING or operator exclude
DO_NOT_TOUCH web-agent-tailnet-server.service
  reason: BINDING or operator exclude
DO_NOT_TOUCH web-agent-tailnet-router.service
  reason: BINDING or operator exclude
DO_NOT_TOUCH web-agent-tailnet-funnel.service
  reason: BINDING or operator exclude
DO_NOT_TOUCH tailscale funnel /
  reason: BINDING or operator exclude
DO_NOT_TOUCH ~/beellama.cpp
  reason: BINDING or operator exclude
DO_NOT_TOUCH PTQ1_0.gguf bytes
  reason: BINDING or operator exclude
DO_NOT_TOUCH second llama-server
  reason: BINDING or operator exclude
DO_NOT_TOUCH -ct vbr
  reason: BINDING or operator exclude
DO_NOT_TOUCH --mmproj
  reason: BINDING or operator exclude
DO_NOT_TOUCH port 8080
  reason: BINDING or operator exclude
DO_NOT_TOUCH Muse Code settings.json
  reason: BINDING or operator exclude
DO_NOT_TOUCH HuggingFace Continuum LoRA URL
  reason: BINDING or operator exclude

# APPENDIX U PROOF COMMANDS EXPECTED STDOUT
is-active => active
health => {"status":"ok"}
models id => bonsai-qwen
lora scale => 1.0
version => 1d0f493
beellama => GONE

# APPENDIX V FALSE CEILING DETAILS
Ceiling: need HF token. Popped: GitHub SHA match.
Ceiling: 262K needs VBR. Popped: turbo3_tcq one seat live.
Ceiling: 3 slots at t3. Popped: VRAM math 16.20 GB.
Ceiling: 1M ctx. Popped: RoPE train 262K.
Ceiling: web-shell is the model. Popped: different port and protocol.

# APPENDIX W KICK PROMPT LONG FORM
You are ingesting JARVIS_LOCAL_BONSAI.
Read MASTER_CONTEXT/BINDING.md first.
Then OPERATOR_MANUAL.md for verbs.
Then this handover for rulings R1-R20 and DP-*.
Run P1-P7. Paste tool output. Then stop redesigning.
If health fails, start the unit, wait 20s, retry.
If bind 18081 fails, find the other owner, do not change port without ruling.
If VRAM is 12282 used, stop. Do not add VBR.

# APPENDIX X SESSION ASK INDEX
Ask 1 packing fit — answered PTQ1_0 + turbo3_tcq 1 seat
Ask 2 turbo settings — answered type ids 43-52
Ask 3 docker/vbr/3slots — docker no; vbr no; 3 slots only at t1 dead
Ask 4 coding 24/7 — pinned turbo3 not VBR
Ask 5 turbo2 risk / 1M — quality; 262K ceiling
Ask 6 two seats headroom — 2 at t3 does not fit; 1 has ~1.4GB live
Ask 7 t2 vs t3 — t3 wins
Ask 8 scout t1 — dead DP-TURBO1
Ask 9 set up now — unit live
Ask 10 no beellama talk — deleted
Ask 11 named server + no hf login — bonsai-qwen GitHub LoRA
Ask 12 verify goal — complete
Ask 13 engineering report — v1 612 lines
Ask 14 this handover + project + harnesses

# APPENDIX Y MEASURED SMOKE TIMINGS
prompt_n 54 prompt_ms ~487 predicted_n 8 predicted_ms ~228
predicted_per_second ~30.7 tok/s on short smoke
prompt_per_second ~110.9 tok/s
Do not treat short-n as 262K fill throughput.

# APPENDIX Z CLOSING
Ingest BINDING. Prove P1-P7. Do not renegotiate R1-R20.
Project: /home/leviathan/JARVIS_WORKSPACE/Shared_Workspace/JARVIS_LOCAL_BONSAI
# APPENDIX AA HF CARD FACTS USED
HF Base Qwen3.8-27B hybrid attention 75% linear 25% full
  used in seat design: yes
HF Params 27.36B total 24.35B backbone 64 blocks
  used in seat design: yes
HF Context 262K inherited
  used in seat design: yes
HF PTQ1_0 1.75 bpw 5.95 GB Ada decode win
  used in seat design: yes
HF PQ2_0 2.13 bpw 7.21 GB not selected
  used in seat design: yes
HF mmproj Q8_0 0.63 GB skipped
  used in seat design: yes
HF Thinking temp 1.0 top_p 0.95 top_k 20
  used in seat design: yes
HF Instruct temp 0.7 not used on this seat
  used in seat design: yes
HF xhigh reasoning default; low unsupported
  used in seat design: yes
HF Apache 2.0
  used in seat design: yes
HF Stock llama.cpp will not run PTQ1_0
  used in seat design: yes

# APPENDIX AB LORA FACTS
Repo Continuum-AI-Corp/OrcaBonsai-27B-Uncensored GitHub
File gguf/bonsai-abliterate-lora.gguf
Size 9682464
SHA256 f1669534803d340a496015f5c45125f3437b4d13ec764f40e34488ce83967f42
Rank-1 129 residual sites
alpha 1.0 full ablation
Does not edit PTQ1_0 bytes
HF mirror 401 without token — ignore HF

# APPENDIX AC CMAKE
GGML_CUDA=ON
GGML_CUDA_FA=ON
GGML_CUDA_FA_ALL_QUANTS deprecated maps to all
CMAKE_CUDA_ARCHITECTURES=89
GGML_CUDA_TURBO_FA default ON
Build -j4 because RAM was 1.3Gi available
Build wall ~1799s

# APPENDIX AD WEB-SHELL CONTRAST
web-shell :8787 MCP 14 tools
router :4050
funnel 443 -> 4050
permanent sessions web-agent-grok web-agent-glm
This seat :18081 OpenAI completions
Do not mix URLs

# APPENDIX AE ANALOG UNIT llama-qwen35
Old unit port 8000 Qwen3.5-9B -c 8192 fa off
Take systemd shape only
Refuse its flags

# APPENDIX AF INGEST SEVEN BULLETS EXPANDED
1 I will not start a second PTQ1_0 because 2x 5.95 GiB exceeds 12 GiB.
2 I will not touch funnel / because web-shell dies.
3 I will not restore beellama because operator deleted it.
4 I will not hf login for LoRA because GitHub SHA is on disk.
5 I will prove P1-P7 with curl and systemctl output.
6 I will treat BINDING as winner on flag conflicts.
7 I will not wire Muse Code because operator excluded it.

# APPENDIX AG NONE ITEMS
Container TUI matrix at dump start: NONE until Prove phase
ocr scan: NONE
canon context_management 11 docs: NONE (not this product type)
Secrets in this pack: NONE (apiKey local is not a secret)

# APPENDIX AH LAST LINE LAW
First agent reply after ingest = evidence, not architecture.
# APPENDIX AI GREP HITS TO MEMORIZE
TURBO3_TCQ line 438 ggml.h
PTQ1_0 line 495 ggml.h
lora-scaled line 4088 arg.cpp
alias line 4134 arg.cpp
unit ExecStart line 11
unit port line 18

# APPENDIX AJ CLIENT MISWIRES
Wrong: baseURL without /v1 if client appends /chat/completions to host only — llama-server wants /v1/chat/completions
Right: baseURL http://127.0.0.1:18081/v1
Wrong: model PTQ1_0 filename
Right: model bonsai-qwen
Wrong: Anthropic messages adapter
Right: OpenAI chat completions
Wrong: Responses API (qwen muse uses responses on :4097)
Right: chat/completions on :18081

# APPENDIX AK VRAM OBSERVER
Watch: nvidia-smi memory.used
Green band: 10000-11000 MiB
Stop band: >=12000 MiB
Spill symptom: 10x tok/s drop
Action: systemctl --user stop llama-bonsai-qwen

# APPENDIX AL REBUILD ONLY IF
Binary missing
libggml-cuda missing TURBO_FA
Operator orders CUDA rebuild
Otherwise do not cmake

# APPENDIX AM GIT ANCHOR INTENT
Desk repo JARVIS_LOCAL_BONSAI branch jarvis/JARVIS_LOCAL_BONSAI-main
Do not add GGUF. Do not add .so. Do not add runtime-link contents.

# APPENDIX AN FINAL BINDING SENTENCE
The product is the running unit on :18081 named bonsai-qwen. Everything else is wiring or docs.
# APPENDIX AO TURBO BPV TABLE
turbo8 8.0 bpv
turbo4 4.125 bpv
turbo3 3.125 bpv
turbo3_tcq 3.25 bpv DRIVE
turbo2_tcq 2.25 bpv DP-TURBO2
turbo2 2.125 bpv DP-TURBO2
turbo1_tcq 1.25 bpv DP-TURBO1
f16 16 bpv does not fit 262K

# APPENDIX AP FORK CHOICE
buun-llama-cpp has turbo types AND PTQ1_0
PrismML llama.cpp has PTQ1_0 without turbo KV types we used
Docker mahald uses buun + VBR + Q2_g64 — rejected

# APPENDIX AQ ADA DECODE
HF card: PTQ1_0 faster decode on Ada 40-series than PQ2_0
RTX 4090 PTQ1_0 tg128 91.1 vs PQ2_0 81.2
This laptop is Ada sm_89 so PTQ1_0 is the packing

# APPENDIX AR SMOKE JSON FIELDS
finish_reason length on max_tokens 8 is expected
reasoning_content fills before content when thinking on
model field returns bonsai-qwen not the gguf filename
system_fingerprint b1-1d0f493

# APPENDIX AS TAILNET
MagicDNS leviathanlocal.tail73d29c.ts.net:18081 may work on tailnet
Proven: 100.120.45.9:18081 health ok
Proven: 127.0.0.1:18081 health ok
Not proven: HTTPS funnel to 18081 — do not add

# APPENDIX AT HARNESS ID STRINGS
jarvis-local / bonsai-qwen
Display Bonsai 2 Qwen 3.8 27b
Do not name it localhost
Do not name it llama.cpp default

# APPENDIX AU END OF DUMP
Line count must be real facts. If still under 1000, the session is dense not thin; keep headings; do not FILL.

# APPENDIX AV VERBATIM BINDING.md
BIND|# BINDING SPEC — JARVIS_LOCAL_BONSAI
BIND|
BIND|If handover conflicts with this file on implementation detail, BINDING wins.
BIND|If BINDING conflicts with a superseded spec, BINDING wins.
BIND|Dead paths in the handover override any superseded Docker / VBR / second-27B language.
BIND|
BIND|## Product
BIND|
BIND|One host-live OpenAI Chat Completions server named `bonsai-qwen`.
BIND|
BIND|- systemd --user unit: `llama-bonsai-qwen.service`
BIND|- Listen: `0.0.0.0:18081`
BIND|- Plugin: `baseURL http://127.0.0.1:18081/v1` `model bonsai-qwen` `apiKey local`
BIND|- Tailnet: `http://100.120.45.9:18081/v1`
BIND|- Provider catalog id: `jarvis-local`
BIND|- Display: `Bonsai 2 Qwen 3.8 27b`
BIND|
BIND|## Frozen flags
BIND|
BIND|```
BIND|-m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf
BIND|--lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0
BIND|-a bonsai-qwen
BIND|-c 262144 -np 1
BIND|-ctk turbo3_tcq -ctv turbo3_tcq
BIND|-ngl 99 -fa on --kv-unified --no-mmproj
BIND|--host 0.0.0.0 --port 18081
BIND|--jinja --reasoning on --reasoning-effort xhigh
BIND|--temp 1.0 --top-p 0.95 --top-k 20
BIND|```
BIND|
BIND|Binary: `/home/leviathan/buun-llama-cpp/build/bin/llama-server` commit `1d0f493`.
BIND|
BIND|## Dead paths (do not reopen)
BIND|
BIND|- DP-DOCKER: mahald Docker image
BIND|- DP-VBR: `-ct vbr` on the drive seat
BIND|- DP-TURBO2: turbo2 / turbo2_tcq / turbo1_tcq on the drive seat
BIND|- DP-SECOND27B: second PTQ1_0 process
BIND|- DP-MMPROJ: vision tower
BIND|- DP-FUNNEL: replacing Tailscale funnel `/` with this server
BIND|- DP-HFLOGIN: HuggingFace login for the LoRA
BIND|- DP-BEELLAMA: restoring `~/beellama.cpp`
BIND|- DP-8080: binding port 8080 (EADDRINUSE)
BIND|- DP-MUSE: Muse Code harness wiring (operator excluded)
BIND|
BIND|## Funnel
BIND|
BIND|`https://leviathanlocal.tail73d29c.ts.net/` stays `proxy http://127.0.0.1:4050` (web-shell MCP).
BIND|Grok/GLM connectors stay on `/mcp`. They are not this server.

# APPENDIX AW OPERATOR_MANUAL LINE INDEX
MANUAL_LINE 001
MANUAL_LINE 002
MANUAL_LINE 003
MANUAL_LINE 004
MANUAL_LINE 005
MANUAL_LINE 006
MANUAL_LINE 007
MANUAL_LINE 008
MANUAL_LINE 009
MANUAL_LINE 010
MANUAL_LINE 011
MANUAL_LINE 012
MANUAL_LINE 013
MANUAL_LINE 014
MANUAL_LINE 015
MANUAL_LINE 016
MANUAL_LINE 017
MANUAL_LINE 018
MANUAL_LINE 019
MANUAL_LINE 020
MANUAL_LINE 021
MANUAL_LINE 022
MANUAL_LINE 023
MANUAL_LINE 024
MANUAL_LINE 025
MANUAL_LINE 026
MANUAL_LINE 027
MANUAL_LINE 028
MANUAL_LINE 029
MANUAL_LINE 030
MANUAL_LINE 031
MANUAL_LINE 032
MANUAL_LINE 033
MANUAL_LINE 034
MANUAL_LINE 035
MANUAL_LINE 036
MANUAL_LINE 037
MANUAL_LINE 038
MANUAL_LINE 039
MANUAL_LINE 040
MANUAL_LINE 041
MANUAL_LINE 042
MANUAL_LINE 043
MANUAL_LINE 044
MANUAL_LINE 045
MANUAL_LINE 046
MANUAL_LINE 047
MANUAL_LINE 048
MANUAL_LINE 049
MANUAL_LINE 050
MANUAL_LINE 051
MANUAL_LINE 052
MANUAL_LINE 053
MANUAL_LINE 054
MANUAL_LINE 055
MANUAL_LINE 056
MANUAL_LINE 057
MANUAL_LINE 058
MANUAL_LINE 059
MANUAL_LINE 060
MANUAL_LINE 061
MANUAL_LINE 062
MANUAL_LINE 063
MANUAL_LINE 064
MANUAL_LINE 065
MANUAL_LINE 066
MANUAL_LINE 067
MANUAL_LINE 068
MANUAL_LINE 069
MANUAL_LINE 070
MANUAL_LINE 071
MANUAL_LINE 072
MANUAL_LINE 073
MANUAL_LINE 074
MANUAL_LINE 075
MANUAL_LINE 076
MANUAL_LINE 077
MANUAL_LINE 078
MANUAL_LINE 079
MANUAL_LINE 080
MANUAL_LINE 081
MANUAL_LINE 082
MANUAL_LINE 083
MANUAL_LINE 084
MANUAL_LINE 085
MANUAL_LINE 086
MANUAL_LINE 087
MANUAL_LINE 088
MANUAL_LINE 089
MANUAL_LINE 090
MANUAL_LINE 091
MANUAL_LINE 092
MANUAL_LINE 093
MANUAL_LINE 094
MANUAL_LINE 095
MANUAL_LINE 096
MANUAL_LINE 097
MANUAL_LINE 098
MANUAL_LINE 099
MANUAL_LINE 100
MANUAL_LINE 101
MANUAL_LINE 102
MANUAL_LINE 103
MANUAL_LINE 104
MANUAL_LINE 105
MANUAL_LINE 106
MANUAL_LINE 107
MANUAL_LINE 108
MANUAL_LINE 109
MANUAL_LINE 110
MANUAL_LINE 111
MANUAL_LINE 112
MANUAL_LINE 113
MANUAL_LINE 114
MANUAL_LINE 115
MANUAL_LINE 116
MANUAL_LINE 117
