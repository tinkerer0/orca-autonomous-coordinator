# Pane layouts and lifecycle

Load the version-matched Orca `orchestration` guide before commands. Treat this file as layout
intent; the live guide and project lifecycle own exact syntax and stricter safety.

## Default layouts

| Concurrent workers | Layout intent | Effective target |
|---:|---|---:|
| 0 | Coordinator only | 100% |
| 1 | `C \| W1` | about 50:50 |
| 2 | `C \| (W1 / W2)` | about 50:25:25 |
| 3 | Balanced 2x2 | about 25% each without ratio support |
| 4+ | Waves or another coordinator-owned tab | Avoid deeper unreadable nesting |

For two workers, split once into coordinator and worker columns, then split only the worker
column. Do not shrink the coordinator to a quarter. For three workers, split each half at most
once. Plan the full topology before opening panes.

Do not hard-code `horizontal` or `vertical` as semantic left/right or top/bottom across Orca
versions. Use the live guide or observed topology. When no ratio API exists, describe the layout
as approximate and never claim exact pixel proportions.

## Ownership and durable ledger

Before the first pane mutation:

1. Resolve the exact live coordinator identity and target tab.
2. Inventory once and classify coordinator-owned, ledger-owned, protected, and unresolved panes.
3. Keep user-created, unrelated-session, and ownership-unknown panes protected.
4. Create a run-scoped append-only ledger.

Record at least:

- run ID, event ID, timestamp, task and dispatch IDs;
- role, provider company, attempt number, and state;
- pane key, PTY, tab, leaf, worktree, split parent and direction;
- open and nesting order;
- outer-shell survival, shell and agent PID/PGID/TTY when uniquely observed;
- failover reason, `worker_done`, verification, exact close, and reconciliation result.

Titles and runtime handles are hints, not ownership proof. Persist pending split requests and
circuit state before retry decisions.

## Opening and dispatch

- Open only workers with concrete ready tasks.
- Use outer-shell-first visible splits when the active lifecycle requires them.
- Confirm shell and agent identity before task creation or dispatch.
- Inject the worker safety capsule and exact receiver/task/dispatch provenance.
- Prefer an isolated worktree when the active tree is dirty, another session owns related files,
  or speculative implementation must not disturb the coordinator's state.
- Reuse a completed pane only for a concrete immediate next task; otherwise close it in the same
  lifecycle cycle.

## Collection and cleanup

1. Collect authoritative `worker_done` or preserve the dispatch as unresolved.
2. Validate the complete machine-readable payload and quarantine malformed evidence.
3. Verify the reported artifact independently.
4. Exit the agent to the surviving outer shell; never intentionally exit the PTY before
   exact-close.
5. Close in reverse nesting order, then reverse open order.
6. After every close, confirm coordinator survival and reconcile exact owned identities.
7. Preserve exited, protected, stale-but-unresolved, or ownership-ambiguous panes.
8. Report observed `opened`, `closed`, `retired_exited_pending_close`, `unresolved`, and
   `remaining_owned` counts. Never infer zero.
