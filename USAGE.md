# Harness scripts (`bin/`)

Quick reference for firstmate's helper scripts. Each script's own header is the
source of truth — read it before first use. Scripts live in the repo's `bin/` but
operate on the active firstmate home selected by `FM_HOME` (see AGENTS.md §2).

Supervision scripts call `fm-guard.sh` first, so their output warns you when the
watcher is down or wakes are queued.

## Session start: bootstrap, detect, lock

| Script | Purpose | Usage |
|---|---|---|
| `fm-bootstrap.sh` | Detect missing tools / gh-auth / harness override, best-effort fleet refresh+prune, then install approved tools. Silent = all good. | `fm-bootstrap.sh` · `fm-bootstrap.sh install <tool>...` |
| `fm-harness.sh` | Detect this process's harness, or resolve the effective crewmate harness. | `fm-harness.sh` · `fm-harness.sh crew` |
| `fm-lock.sh` | Acquire/inspect the per-home session lock (records the session-stable harness PID). | `fm-lock.sh` · `fm-lock.sh status` |
| `fm-guard.sh` | Watcher-liveness guard called at the top of supervision scripts; warns (never blocks) when a task is in flight but the watcher beacon is stale or wakes are queued. | called internally |

## Project management

| Script | Purpose | Usage |
|---|---|---|
| `fm-project-mode.sh` | Resolve a project's delivery mode + yolo flag from `data/projects.md`. Prints `<mode> <yolo>`; unknown falls back to `no-mistakes off`. | `fm-project-mode.sh <project-name>` |
| `fm-fleet-sync.sh` | Fast-forward a clone's default branch to origin when safe and prune gone local branches. Never forces/stashes/discards unlanded work. `FM_FLEET_PRUNE=0` disables prune. | `fm-fleet-sync.sh [<project-dir>]` |
| `fm-ensure-agents-md.sh` | Worktree utility (for crewmates): ensure the `AGENTS.md` (real) + `CLAUDE.md` (symlink) convention; creates a skeleton or promotes an existing `CLAUDE.md`. | `fm-ensure-agents-md.sh [repo-or-worktree-dir]` |

## Task lifecycle

| Script | Purpose | Usage |
|---|---|---|
| `fm-brief.sh` | Scaffold a crewmate brief / scout brief / secondmate charter with the standard contract; fill the `{TASK}` placeholder afterward. | `fm-brief.sh <id> <repo> [--scout]` · `fm-brief.sh <id> --secondmate <project>...` |
| `fm-spawn.sh` | Spawn a direct report: crewmate in a treehouse worktree, or secondmate in its home. Records meta (`harness/kind/mode/yolo/...`). Supports batch `id=repo` pairs. | `fm-spawn.sh <id> <project-dir> [harness] [--scout]` · `fm-spawn.sh <id> [<home>] --secondmate` · `fm-spawn.sh <id1>=<repo1> <id2>=<repo2> [--scout]` |
| `fm-pr-check.sh` | Record a PR-ready task (`pr=` in meta) and arm the watcher's merged-PR poll. | `fm-pr-check.sh <id> <pr-url>` |
| `fm-review-diff.sh` | Review a crewmate branch against the authoritative base (fetches origin for remote-backed clones). | `fm-review-diff.sh <id> [--stat]` |
| `fm-merge-local.sh` | Approved local merge for a `local-only` task: clean fast-forward of the project's default branch to `fm/<id>`. Refuses a diverged branch. | `fm-merge-local.sh <id>` |
| `fm-promote.sh` | Promote a scout task to ship in place (flips `kind=` to ship, restoring teardown protection). Then send ship instructions. | `fm-promote.sh <id>` |
| `fm-teardown.sh` | Return the worktree / retire a secondmate home, kill the window, clear state, refresh the clone. Refuses if work is unlanded (scout: needs report). | `fm-teardown.sh <id> [--force]` |

## Supervision & signalling

| Script | Purpose | Usage |
|---|---|---|
| `fm-watch.sh` | The watcher backbone. Blocks until work is due, then exits with one reason line: `signal` / `stale` / `check` / `heartbeat`. Run in background; re-arm after each wake (singleton-safe). | `fm-watch.sh` (background) |
| `fm-wake-drain.sh` | Atomically drain durable queued wake records at the start of every wake/recovery turn. | `fm-wake-drain.sh` |
| `fm-supervise-daemon.sh` | Presence-gated (`/afk`) sub-supervisor: wraps the watcher, self-handles routine wakes in bash, escalates only captain-relevant events as one batched digest. | started by the `/afk` skill |
| `fm-peek.sh` | Print the bounded tail of a crewmate pane for cheap diagnosis. | `fm-peek.sh <window> [lines=40]` |
| `fm-send.sh` | Send one literal line (then Enter) to a crewmate window, or a special key. | `fm-send.sh <window> <text...>` · `fm-send.sh <window> --key Escape` |
| `fm-wake-lib.sh` | Shared durable wake-queue + portable (mkdir-based) lock helpers. | sourced by other scripts |

## Secondmates

| Script | Purpose | Usage |
|---|---|---|
| `fm-home-seed.sh` | Provision a persistent secondmate home (clone projects, copy charter, init no-mistakes projects, update the routing table). `-` durably leases a treehouse worktree. Transactional. | `fm-home-seed.sh <id> <home\|-> <project>...` · `fm-home-seed.sh validate` |
| `fm-backlog-handoff.sh` | Mechanically move in-scope queued backlog items from the main home into a secondmate's home backlog. Idempotent; refuses non-firstmate-home destinations and `## In flight` entries. | `fm-backlog-handoff.sh <secondmate-id> <item-key>...` |

## Common environment knobs

| Var | Effect |
|---|---|
| `FM_HOME` | Selects the operational home (`state/ data/ config/ projects/`). Unset = repo root. |
| `FM_STATE_OVERRIDE` / `FM_ROOT_OVERRIDE` | Legacy state-dir / whole-root overrides. |
| `FM_FLEET_PRUNE=0` | Skip gone-branch pruning during fleet sync. |
| `FM_FLEET_SYNC_BOOTSTRAP_TIMEOUT` | Bound (default 20s) on bootstrap's fleet refresh. |
| `FM_GUARD_GRACE` | Watcher-beacon staleness grace (default 300s). |
| `FM_HEARTBEAT` / `FM_HEARTBEAT_MAX` | Heartbeat base interval and backoff cap. |
| `FM_SIGNAL_GRACE` | Coalescing window for near-simultaneous signals. |
| `FM_SECONDMATE_CHARTER` / `FM_SECONDMATE_SCOPE` | Inline charter text / routing scope for brief + seed. |
| `FM_STALE_ESCALATE_SECS` / `FM_HEARTBEAT_SCAN_SECS` / `FM_ESCALATE_BATCH_SECS` / `FM_INJECT_SKIP` | Sub-supervisor timing and classification knobs. |
