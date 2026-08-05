---
name: orca-autonomous-coordinator
description: Coordinate work autonomously inside Orca across available agent providers. Use when the user-facing session must choose direct versus multi-agent execution, open or reuse worker panes, allocate implementation and independent-verification roles, recover from quota/tool/context/quality failures, supervise worktree-isolated workers, or clean up an orchestration run. Do not use for a purely direct task that opens no worker and needs no provider failover.
---

# Orca Autonomous Coordinator

Act as the single user-facing coordinator. Optimize verified end-to-end completion time and
rework risk, not worker count. Treat provider names as interchangeable implementations of
capabilities, never as a permanent roster.

## Coordinate

1. Freeze the task contract: goal, exclusions, deliverables, acceptance evidence, user-owned
   gates, and project or connector overlays.
2. Select `Fast`, `Standard`, or `Full` assurance independently from execution topology. Keep
   short, sequential, same-file, shared-state, or unstable-interface work direct. Open workers
   only when independent slices or specialist review repay setup, recovery, and merge cost.
3. Allocate roles from required capability, tool/runtime access, availability, context,
   latency/cost, and recent verified output. Honor explicit user assignments. Preserve a
   different company and session for material or high-risk verification.
4. Before dispatch, read [references/worker-capsule.md](references/worker-capsule.md) and inject
   a task-specific safety capsule. Make every worker finite, non-coordinating, and limited to the
   authority required for its slice.
5. Before opening a pane, read [references/pane-layouts.md](references/pane-layouts.md), load the
   version-matched Orca `orchestration` guide, and load the project's canonical Orca lifecycle
   when one exists. Let the live guide own command syntax and the project add stricter safety.
6. Create a run-scoped, local-only append-only ledger before the first pane mutation. Prefer a
   project-provided validated ledger command and designated path. When neither exists, use
   append-only JSONL at `work/orchestration/<run-id>/events.jsonl` and keep it out of published
   artifacts. Record ownership, topology, dispatch provenance, attempts, failovers, completion,
   verification, and exact close/reconciliation evidence. Constrain `stage` to run_created,
   plan_fixed, split_requested, agent_ready, dispatched, first_heartbeat, worker_done,
   review_done, remediation_done, coordinator_verified, pane_closed, and run_reconciled, plus
   `other` with a required `meta.stageDetail`; keep version, milestone, and batch labels in
   `meta`, never in the stage name. Declare run topology in the run's first coordinator-task
   `plan_fixed` as `meta.topology` — nodes as `role@provider`, edges, and shared exact paths —
   alongside `implementationCompany` and `verifierCompany`; rewiring appends a new coordinator
   `plan_fixed`, and the latest wins.
7. Maintain at most one outstanding blocking wait per coordinator inbox receiver. One wait may
   cover several dispatches; validate every returned `taskId` and `dispatchId`. Treat heartbeat
   and visible activity as liveness, not completion. Apply
   [references/decision-policy.md](references/decision-policy.md); never retry an unchanged
   failure indefinitely.
8. Require `worker_done` to carry a complete machine-readable payload. Quarantine body-only or
   malformed completion evidence, preserve it, and allow at most one bounded formatting recovery
   before applying the normal failure budget.
9. Inspect the actual artifact and rerun proportionate checks. Adapt evidence to the artifact
   type instead of assuming every task is code.
10. Exit each agent to its surviving outer shell, close owned panes in reverse nesting/open
    order, reconcile exact identities, and preserve protected or unresolved panes.
11. Report one integrated result with assurance path, topology, task-to-provider mapping,
    substitutions, evidence, residual risks, user decisions, and observed cleanup counts. At
    run close, append a one-line summary (run id, dates, objective, closing stage) to
    `work/orchestration/RUNS.md`.

## Preserve autonomy safely

- Do not ask the user to choose routine providers, layouts, retry order, or worker count.
- Do not invoke every provider by ritual or retain idle panes for hypothetical work.
- Do not accept an implementer's self-verdict as independent verification.
- Do not weaken sensitive-data, external-side-effect, cost, destructive-action, or domain gates
  during failover.
- Do not infer provider availability when Orca exposes no quota probe. Mark it unknown, attempt
  only within budget, and record the observed failure.
- Do not claim exact pane ratios when the runtime exposes only equal splits.
- Do not edit generated memory as the policy source. Keep required behavior in global/project
  guidance, this skill, the injected capsule, and durable run evidence.

## Stop conditions

Stop and preserve evidence when the user changes meaning or authority, a safety invariant fails,
the bounded failover budget is exhausted, or required independent verification remains
unavailable. Escalate only the decision the coordinator cannot own.
