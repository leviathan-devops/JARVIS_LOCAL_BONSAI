# OPERATOR MANUAL — llama-bonsai-qwen
**Machinery:** systemd --user unit `llama-bonsai-qwen` + OpenAI HTTP on :18081
**Version:** binary 1d0f493 · unit 29 lines · walked 2026-09-19T01:18Z
**Operator job:** keep the named OpenAI endpoint up and plug clients into it.
**WHY:** MASTER_CONTEXT/BINDING.md and docs/BIBLE.md (do not repeat here)
**WHAT:** reports/Bonsai_Qwen_Local_Provider_Engineering_Report_v1.md

## §1 WHAT THIS IS FOR
ONE outcome: `curl -sf http://127.0.0.1:18081/v1/models` returns id `bonsai-qwen` and a client can complete.
USE when you need a local 27B OpenAI endpoint.
DO NOT USE as Grok/GLM MCP connector, as a second GPU load, or as funnel `/`.

## §2 BEFORE YOU START
Check each:
- [ ] `systemctl --user is-active llama-bonsai-qwen` prints `active` OR you will start it (A-2)
- [ ] `test -f /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf`
- [ ] `test -f /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf`
- [ ] `test -x /home/leviathan/buun-llama-cpp/build/bin/llama-server`
- [ ] GPU is the RTX 4080 Laptop (do not start if another 27B already owns VRAM)

Walked: all four files exist; unit is enabled.

## §3 QUICK START
1. `systemctl --user start llama-bonsai-qwen`
2. `curl -sf http://127.0.0.1:18081/health` expect `{"status":"ok"}`
3. Point client: baseURL `http://127.0.0.1:18081/v1` model `bonsai-qwen` apiKey `local`
4. Send one chat. Expect HTTP 200. Thinking may fill `reasoning_content` first.
5. Tailnet: `http://100.120.45.9:18081/v1` — not the funnel MCP URL.
6. Stop only if you mean it: `systemctl --user stop llama-bonsai-qwen`
7. Status: `systemctl --user status llama-bonsai-qwen --no-pager`

## §4 FULL PROCEDURE
A-1 PRECONDITION: unit file exists at ~/.config/systemd/user/llama-bonsai-qwen.service
    ACTION: `systemctl --user daemon-reload && systemctl --user enable llama-bonsai-qwen`
    EXPECTED: `enabled`
    IF NOT: copy config/llama-bonsai-qwen.service into ~/.config/systemd/user/ then retry
    PROVES: boot autostart is armed

A-2 PRECONDITION: no second PTQ1_0 process (`pgrep -af PTQ1_0` empty or this unit only)
    ACTION: `systemctl --user start llama-bonsai-qwen`
    EXPECTED: `is-active` = active within 30s
    IF NOT: `journalctl --user -u llama-bonsai-qwen -n 80`
    PROVES: process running

A-3 PRECONDITION: A-2 active
    ACTION: `curl -sf http://127.0.0.1:18081/health`
    EXPECTED: `{"status":"ok"}`
    IF NOT: wait 20s (model load); if bind error, see §6 SYM-BIND
    PROVES: HTTP is up

A-4 ACTION: `curl -sf http://127.0.0.1:18081/v1/models`
    EXPECTED: `"id":"bonsai-qwen"`
    IF NOT: unit argv missing `-a bonsai-qwen`
    PROVES: alias

A-5 ACTION: `curl -sf http://127.0.0.1:18081/lora-adapters`
    EXPECTED: `"scale":1.0` and path ending bonsai-abliterate-lora.gguf
    IF NOT: LoRA file missing; do not hf login; GitHub raw
    PROVES: uncensor adapter loaded

A-6 ACTION: POST /v1/chat/completions with model bonsai-qwen
    EXPECTED: HTTP 200; timings.kv_bpv 3.25
    IF NOT: §6 SYM-SLOW or SYM-OOM
    PROVES: tokens generate

A-7 ACTION: `nvidia-smi --query-gpu=memory.used --format=csv,noheader`
    EXPECTED: used ~10400-11000 MiB, not 12282
    IF NOT: spill — stop the unit
    PROVES: VRAM budget

## §5 WHAT YOU SHOULD SEE
health: {"status":"ok"}
lora-adapters: [{"id":0,"path":"/home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf","scale":1.0,...}]
journal: llama_server: listening on http://0.0.0.0:18081
journal: llama_server: model loaded

## §6 WHEN IT GOES WRONG
| SYMPTOM | ACTION |
|---|---|
| couldn't bind ... port: 8080 | Do not use 8080. Unit must say 18081. |
| couldn't bind ... port: 18081 | `ss -ltnp \| awk '/18081/'` then stop the other owner |
| health connection refused | A-2 start; wait load (~20s) |
| /lora-adapters [] | check LoRA path; GitHub raw not HF |
| nvidia-smi used 12282 | STOP unit; never start a second 27B |
| tokens 10x slower | host RAM spill; stop; check free VRAM |
| funnel MCP broken | you stole `/`; restore proxy 127.0.0.1:4050 |
| Grok connector fails on :18081 | wrong URL; Grok uses funnel /mcp |
| unit inactive after reboot | linger + enable; `loginctl enable-linger $USER` |

## §7 RECOVERY
ABORT: `systemctl --user stop llama-bonsai-qwen`
ROLLBACK: none for GPU weights; files stay on disk
RESUME: A-2 start then A-3 health
If unit crash-loops: `journalctl --user -u llama-bonsai-qwen -n 100`

## §8 THE SECOND RUN
Do not rebuild CUDA. Do not re-download GGUF. Just A-2 if inactive, A-3 if unsure.

## §9 THE LIMITS
Cannot run two PTQ1_0 seats on this 12 GB card.
Cannot extend context past 262144.
Cannot serve funnel `/`.
Cannot be the Grok MCP connector.
Cannot load mmproj without eating 0.63 GB you do not have spare.
Muse Code is out of scope.

## §10 THE TRAPS
Using 8080. HuggingFace login for LoRA. Pointing Grok at 18081. Starting Docker mahald. Copying GGUF into git.

## §11 VERIFICATION
systemctl --user is-active llama-bonsai-qwen == active
curl -sf http://127.0.0.1:18081/health -> {"status":"ok"}
curl -sf http://127.0.0.1:18081/v1/models id==bonsai-qwen
curl -sf http://127.0.0.1:18081/lora-adapters scale==1.0

## §12 THE DRILL
Walked 2026-09-19: A-3 A-4 A-5 A-7 all green. Questions remaining: OMP/MiMo default model steal (unruled).
