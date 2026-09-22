# Phone "server floor" for the Jarvis bot — research report
Ticket dc_1790044703758_9660 (#git-repos). 2026-09-22.

## TL;DR (the decision)

**Do Tier 1 now (guaranteed in a few hours, 0 data loss):** the phone runs ONLY the Discord
gateway daemon (Termux + Bun), receives @mentions, writes tickets into a SHARED sqlite, and
backs off whenever the main comp is up. The comp answers all queued tickets the moment it
returns. Nothing on the phone needs Orca, OMP, or the API model — so none of the risky
unknowns block it.

**Tier 2 is the "full live answering" version** (phone also runs Orca + OMP TUI + Laguna) —
do it AFTER the comp is back, because it hits three unknowns at once (Orca-on-Termux,
bun:sqlite-on-Termux, cross-host session merge).

The one non-negotiable: **one Discord token = one gateway.** The phone must only hold the
token while the comp is down. Every design below is built around that mutual exclusion.

---

## What is actually alive on the comp today (grounded)

The "bot backend" is NOT one process. It is a systemd **user-service stack** (all die when
the comp powers off):

```
jarvis-discord-gateway.service   bun run src/daemon/index.ts      (the gateway + ticket phone)
jarvis-tui.service               jarvis-tui-run.sh                (headed OMP TUI / droid desk)
jarvis-manager.service           fleet supervisor, REST :4210
jarvis-gateway-watchdog.service  gateway liveness (event-based)
jarvis-seat-watchdog.service     bonsai seat HTTP /health
jarvis-stuck-ticket-watchdog     artifact-first ticket-age verdict
  (+ timers: gateway-watchdog, seat-watchdog, stuck-ticket, hand-cron)
```

Environment on the gateway: `JARVIS_CHANNELS_MD` (CHANNELS.md registry), `DISCORD_TOKEN_FILE`,
`JARVIS_STALL_MS=300000`, `JARVIS_ORCA_BIN`, `JARVIS_ORCA_TERMINAL=term_<uuid>`.

Local model = Bonsai 27B, seat `llama-bonsai-qwen` at :18081 (needs the RTX 4080 —
**not on the phone**). So the phone's OMP, if any, must be **API-based**: poolside Laguna
direct + NVIDIA Laguna fallback (per your spec).

### The ticket store (the piece that survives handoff)

`discord-tickets.sqlite` (WAL), one table `tickets`, one row per ticket. Row carries:
`guild_id/guild_name, channel_id/channel_name, message_id, author, content, channel_summary,
recent_messages, status, created_at, last_kick_at, response, resolved_at`. Indexes on
`status / channel_id / author_id / created_at`. **`DISCORD_TICKETS_DB` env overrides the
path** — this is the single switch that makes the store shareable across hosts.

The daemon's internal flow:

```
gateway.onMessage --+
                    v
           ticket row in sqlite
                    v
              Bridge.emit  -> outbox file (state/bridge-outbox/ticket_<id>.json)
                    v
              bridge.deliver(ticket)  -> socket into the OMP terminal
                    (deliver FAIL is NON-FATAL: ticket stays pending in outbox + sqlite)
```

Two durable artifacts, not one: the **sqlite row** (source of truth for "a ticket exists and
is pending") and the **outbox file** (the delivery queue). The outbox lives in `PKG_ROOT/
state/bridge-outbox/` — also host-relative, so it too must be shared or re-created on handoff.

---

## The handoff problem, decomposed

When the comp is OFF, all of: gateway, TUI, manager, watchdogs, Bonsai seat, sqlite, outbox,
worktree, OMP sessions, mnemopi memory — are gone. For the phone to take over:

| Concern            | What must move to the phone | Hard? |
|--------------------|-----------------------------|-------|
| **Token / gateway**  | the sealed token + the daemon   | MED  (mutual exclusion) |
| **Ticket queue**     | sqlite + outbox (shared)        | EASY  |
| **Work (files)**     | worktree, git commit/pull       | EASY  |
| **OMP TUI + model**  | OMP + Orca + API model          | HARD  (Tier 2) |
| **Session context**  | OMP JSONL transcript + memory   | HARDEST |

### Token mutual exclusion (the crux)

A Discord **bot token opens exactly one gateway socket.** A second socket on the same token
gets kicked. This is a natural lock: the phone's daemon can run continuously but only
*holds* the token while the comp's gateway is down. On comp return, the comp's gateway takes
the token and the phone backs off. Two ways to make the handoff deterministic:

1. **Sentinel (recommended, deterministic).** The comp's gateway (or a tiny companion)
   writes a heartbeat file (e.g. `COMP_UP` with a timestamp, in a shared location: git-repo,
   rsync, or NAS) every 30s. The phone's daemon treats a **stale** sentinel as "comp down"
   and connects; a **fresh** sentinel as "comp up" and it sleeps. No dependence on Discord's
   kick behavior.
2. **Token-kick (self-healing, no shared file).** The phone's daemon always tries to connect;
   if Discord rejects/kicks it (token held), it sleeps + retries. Comp down => the token is
   free => the phone's next retry succeeds. *Needs a one-time live test of the exact kick
   behavior.*

**Security note:** the token file ends up on the phone. The sentinel (or token-kick) design
is what prevents two gateways from both being "active". Keep the token file's permissions tight
on both devices.

---

## Tier 1 — "0 data loss" (build this now, ~1–2 h phone-side)

**Phone runs: Termux + Bun + the gateway daemon. Nothing else.**

Comp side (low-risk prep, do now or on return):
1. Keep the `discord-tickets.sqlite` + `state/bridge-outbox/` as the shared artifacts.
2. Decide the share mechanism (pick one):
   - **git** the two artifacts into a small repo (commit the sqlite + outbox every ~5s while
     active). Simplest, no new infra. WAL makes this safe-ish; commit on a quiescent interval.
   - **rsync / syncthing** to a NAS or to the phone over the network.
   - (A NAS-mounted path that both can write = the cleanest, but requires the NAS.)
3. (Optional) add a tiny `COMP_UP` heartbeat writer to the comp (a 10-line cron/timer that
   writes the timestamp every 30s) for the sentinel design.

Phone side (operator, on the phone):
4. Install **Termux** + **bun-on-Termux** (the `bun-on-termux` / `bun-termux` projects give a
   working glibc wrapper for Bun on Android). Verify `bun --version`.
5. Copy the `discord-ticket-phone` package + `config/.discord-token` to the phone (Termux storage).
6. Point `DISCORD_TICKETS_DB` (and the outbox dir) at the shared location.
7. Run a **sentinel-gated** version of the daemon: `while true; do if comp_down; then
   DISCORD_TICKETS_DB=... bun run src/daemon/index.ts & fi; sleep 10; done` — or better,
   a small phone-side wrapper that starts/stops the daemon based on the sentinel.
8. **Keep the phone alive** (the real operational risk, see Risks): Termux:Boot,
   `termux-wake-lock`, keep charging, keep screen on, disable battery optimization, and a
   foreground service if needed.

Why Tier 1 has no model/Orca risk: the phone only *receives* + writes tickets. It does not
run OMP, so it never needs the Bonsai seat, Orca, or the API model. The "answer" happens on
the comp on return — the comp's session already has all pre-off context, and the queued tickets
are in the shared sqlite. **0 data loss; a gap in answering, but no loss.**

---

## Tier 2 — "full live answering" (build after comp returns)

Phone runs: **Orca serve + OMP TUI + poolside Laguna (NVIDIA fallback)**. The droid desk is
drivable from the phone.

### The Orca pairing model (verified, issue #18783)

`orca serve` advertises **either** a mobile-scoped pairing link (`--mobile-pairing`) **or** a
runtime-environment pairing link — **mutually exclusive** per process (open bug #18783, no fix
yet). Consequence:

- A `--mobile-pairing` server accepts the phone (the phone is a **mobile client**) but CANNOT
  be a peer environment (a mobile client is refused `worktree.list` and can't host a worker).
- Without the flag, hosts pair as environments (cross-host dispatch) but the phone can't pair.

So the phone is a **client** that pairs to a `--mobile-pairing` server. For the phone to be
self-sufficient (comp off), **the phone must run its own `orca serve --mobile-pairing`** and
its own Orca app pairs to it. That is a big unknown: **does `orca serve` even run on Termux?**
Orca is a Node/Bun app — it needs the same Bun-on-Termux runtime plus the Orca binary. If
that fails, fall back to a **raw Termux + tmux + OMP TUI** (no Orca) — simpler and more likely
to work.

### Phone OMP config (API model only — no local weights)

Configure the phone's OMP `modelRoles` to:
- primary = `poolside/laguna-s-2.1:high` (inference.poolside.ai),
- fallback = NVIDIA Laguna (then openrouter),
- **no local Bonsai provider** (the seat is GPU-bound on the comp).
This is exactly the chain you specified. Caveat: the poolside Laguna lane is *currently
rate-limited* (the same 429 that has been blocking my subagents), so the phone's OMP will
likely run mostly on the NVIDIA fallback right now — expected, not broken.

### Session-context resync (the HARDEST piece, greenfield)

OMP sessions are **local JSONL per cwd** in `~/.omp/`. There is **no built-in cross-host
session merge** (the `omp-session-move` / `omp-worktree-session-split-recovery` skills are
same-host). On comp return, the comp's session would be "behind" the phone's. Options:
- **Tier 2a (files only):** git pull for the worktree. The comp's session resumes with
  pre-off context; the phone's *ticket conversations* are NOT merged into the session, but
  every ticket + its answer is in the shared sqlite (which the operator can re-read). Simplest.
- **Tail-append:** a sync script appends the phone's session JSONL *tail* (new rows since the
  comp went off) to the comp's session file, keyed by row timestamp. Feasible, greenfield.
- **Full merge + memory:** plus rsync mnemopi banks and `threads/store.sqlite`. Most work.

---

## Risks (honest)

1. **Android kills the daemon.** By far the #1 operational risk. Android's battery management
   reaps background processes. Mitigations: Termux:Boot, `termux-wake-lock`, charging,
   screen-on, disable optimization, foreground service. If the phone sleeps, the "backup"
   is not alive. **This must be tested on the actual phone before relying on it.**
2. **`bun:sqlite` on Termux.** The daemon uses the native `bun:sqlite` binding. Under
   bun-on-Termux's glibc wrapper, native bindings *may* not load. Test `bun -e 'require(
   "bun:sqlite").openFile(...)"'` on the phone FIRST. If it fails: swap the sqlite for a
   JSONL queue (small code change) or use a plain file per ticket.
3. **Token conflict / double-gateway.** Two devices hold the token. The sentinel (preferred)
   or token-kick prevents double-active. Verify the exact Discord kick behavior once live.
4. **Orca-on-Termux (Tier 2 only).** Unknown whether `orca serve` runs on Termux. If not,
   use raw Termux + tmux + OMP (no Orca).
5. **Cross-host session merge (Tier 2 only).** No built-in mechanism. Tier 1 sidesteps it
   (queue only).

---

## Verified vs. needs-phone-side-testing

**Verified (on the comp, evidence-backed):** systemd service stack; `DISCORD_TICKETS_DB`
override; outbox + sqlite dual-durable; deliver-failure non-fatal; `--mobile-pairing`
mutual-exclusion (issue #18783); bot is in exactly "Admin's server" + "Jarvis-Server"
guilds (so a #test-chan in a *different* guild is unreachable by the bot — root cause of the
"merge probe" delivery fault, reported).

**Needs testing on the phone (the unknowns):**
- `bun --version` + `bun:sqlite` load under Termux.
- `orca serve --mobile-pairing` runs on Termux (Tier 2).
- The sentinel / token-kick handoff, end to end, with the comp actually powered off.
- Phone keep-alive under real battery pressure.

**Bottom line for the next few hours:** ship Tier 1. It is the only tier that is *guaranteed*
within the window and it gives you 0 data loss. Tier 2 (live answering from the phone) is a
first-class follow-on once the comp is back and the phone's Termux stack is proven.
