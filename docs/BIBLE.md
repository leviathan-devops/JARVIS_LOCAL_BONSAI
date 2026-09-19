# BIBLE — JARVIS_LOCAL_BONSAI
WHAT is true. HOW is OPERATOR_MANUAL.md. WHY forks are BINDING.md.

## Architecture
Host-live llama-server from spiritbuun/buun-llama-cpp 1d0f493.
PTQ1_0 = ggml type 91 wire 143. turbo3_tcq = ggml type 46.
LoRA rank-1 runtime ablation, weights of PTQ1_0 bit-identical.
n_ctx 262144. n_slots 1. Hybrid attention 75% linear 25% full.
VRAM live 10465/12282 MiB. Headroom 1412 MiB.

## Iron laws
1. One PTQ1_0 process.
2. turbo3_tcq pinned, VBR off.
3. Funnel / stays web-shell.
4. LoRA from GitHub raw SHA f1669534803d340a496015f5c45125f3437b4d13ec764f40e34488ce83967f42
5. Alias bonsai-qwen. Provider jarvis-local.
6. No Muse Code wiring.
7. No ~/beellama.cpp restore.

## Research that survived
PTQ1_0 faster decode on Ada (RTX 40) than PQ2_0. Card is sm_89 Ada.
turbo3_tcq 3.25 bpv; chat timings kv_bpv=3.25 measured.
262K is training ceiling; no 1M.
DSpark not on this seat.
Orca web-shell named sessions are web-agent-* MCP, not this API.

## Config surfaces
Dev: cmake CUDA 12.8 ARCH=89 TURBO_FA default ON.
User: systemd unit flags listed in BINDING.md.
Client: OpenAI baseURL /v1 model bonsai-qwen.
Harness: jarvis-local in OMP models.yml, mimocode.jsonc, qwen settings.

## Anti-patterns
See BINDING dead paths DP-*
