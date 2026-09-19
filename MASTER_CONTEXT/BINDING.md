# BINDING SPEC — JARVIS_LOCAL_BONSAI

If handover conflicts with this file on implementation detail, BINDING wins.
If BINDING conflicts with a superseded spec, BINDING wins.
Dead paths in the handover override any superseded Docker / VBR / second-27B language.

## Product

One host-live OpenAI Chat Completions server named `bonsai-qwen`.

- systemd --user unit: `llama-bonsai-qwen.service`
- Listen: `0.0.0.0:18081`
- Plugin: `baseURL http://127.0.0.1:18081/v1` `model bonsai-qwen` `apiKey local`
- Tailnet: `http://100.120.45.9:18081/v1`
- Provider catalog id: `jarvis-local`
- Display: `Bonsai 2 Qwen 3.8 27b`

## Frozen flags

```
-m /home/leviathan/models/bonsai/Ternary-Bonsai-2-27B-PTQ1_0.gguf
--lora-scaled /home/leviathan/models/bonsai/bonsai-abliterate-lora.gguf:1.0
-a bonsai-qwen
-c 262144 -np 1
-ctk turbo3_tcq -ctv turbo3_tcq
-ngl 99 -fa on --kv-unified --no-mmproj
--host 0.0.0.0 --port 18081
--jinja --reasoning on --reasoning-effort xhigh
--temp 1.0 --top-p 0.95 --top-k 20
```

Binary: `/home/leviathan/buun-llama-cpp/build/bin/llama-server` commit `1d0f493`.

## Dead paths (do not reopen)

- DP-DOCKER: mahald Docker image
- DP-VBR: `-ct vbr` on the drive seat
- DP-TURBO2: turbo2 / turbo2_tcq / turbo1_tcq on the drive seat
- DP-SECOND27B: second PTQ1_0 process
- DP-MMPROJ: vision tower
- DP-FUNNEL: replacing Tailscale funnel `/` with this server
- DP-HFLOGIN: HuggingFace login for the LoRA
- DP-BEELLAMA: restoring `~/beellama.cpp`
- DP-8080: binding port 8080 (EADDRINUSE)
- DP-MUSE: Muse Code harness wiring (operator excluded)

## Funnel

`https://leviathanlocal.tail73d29c.ts.net/` stays `proxy http://127.0.0.1:4050` (web-shell MCP).
Grok/GLM connectors stay on `/mcp`. They are not this server.
