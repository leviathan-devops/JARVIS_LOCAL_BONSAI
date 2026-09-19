# MIGRATION_MANIFEST — JARVIS_LOCAL_BONSAI
Timestamp: 2026-09-19T01:23Z

## Moves / copies (docs only; GGUF not moved)

| dest | source | note |
|---|---|---|
| reports/Bonsai_Qwen_Local_Provider_Engineering_Report_v1.md | Shared_Workspace/reports/... | copy |
| reports/Bonsai_Ternary2_27B_Seat_ShowMe.md | Shared_Workspace/reports/... | copy |
| config/llama-bonsai-qwen.service | ~/.config/systemd/user/... | copy; live unit stays in systemd |
| models-link | /home/leviathan/models/bonsai | symlink, gitignored |
| runtime-link | /home/leviathan/buun-llama-cpp | symlink, gitignored |

Excluded: GGUF bytes, libggml-cuda.so, CUDA build tree (too large; live paths remain absolute).

Live unit NOT moved out of systemd (would break enablement).
