# Decision policy

## Assurance and topology

- `Fast`: narrow, reversible, low-risk. Keep direct unless a specialist check is cheaper than
  likely rework. Perform targeted evidence and coordinator review.
- `Standard`: material but bounded. Use direct or limited orchestration and an implementer-
  independent verifier, then coordinator artifact/evidence recheck.
- `Full`: sensitive data, cost, external or irreversible action, consequential domain claim, or
  broad parallel change. Use the strongest applicable design review, user gates, and independent
  verification. Full assurance does not require parallel implementation.

Parallelize only disjoint slices with stable interfaces. Serialize same-file, shared-state,
unstable-API, or order-dependent work. Existing protected panes never authorize touching them;
display readability may justify waves or a new owned tab instead of deep nesting.

## Bounded failure ladder

Classify the failure before acting.

| Failure | Autonomous action |
|---|---|
| Transient transport/runtime failure with no unsafe side effect | Retry the same configuration once. |
| Quota, authentication, missing tool, or provider unavailable | Do not repeat unchanged; reallocate once to a capable provider. |
| Quality or acceptance failure | Refine the slice or replace the provider once; preserve original evidence. |
| Malformed completion payload | Quarantine it and request one format-only recovery without redoing accepted work. |
| Worker stall | Check exact liveness once, send one completion reminder when applicable, then recover or reallocate. |
| Path/interface collision | Stop new dispatches and serialize or repartition. |
| Safety/gate violation | Stop the affected path; do not route around the gate. |

Default to one transient retry and one provider reallocation per task. Shorten the budget for
unsafe or non-idempotent work. Do not silently extend it. Record every attempt and reason.

When the preferred verifier is unavailable, rearrange implementer/verifier assignments to leave
another company available. If no different-company/session verifier remains, retain the local
draft and advisory checks but label the result `INDEPENDENCE_UNAVAILABLE`. A coordinator may
create or locally commit that clearly labeled draft only when project Git policy otherwise
permits it; the commit is a recoverable checkpoint, not verified completion or merge readiness.
Do not emit `GO`, merge, deploy, publish, or perform the gated external action.

## Scope and continuity

When new user input changes meaning, acceptance, authority, or exact ownership:

1. Stop issuing new dispatches.
2. Let in-flight workers reach a safe checkpoint or collect their current status.
3. Mark superseded results stale for acceptance without deleting evidence.
4. Re-freeze the task contract and replan.
5. Reuse completed evidence only when it still proves the new acceptance criteria.

A status question alone does not trigger draining.

Before coordinator context becomes unsafe, persist the task contract, plan, ledger, pane
ownership, attempts, completed evidence, open gates, and next action. Transfer to exactly one
capable successor and stop the outgoing coordinator after observed takeover.

## Evidence adapters

| Artifact | Minimum evidence |
|---|---|
| Code | Diff, targeted tests, relevant runtime or static checks |
| Research | Primary sources, citation binding, independent fact cross-check |
| Document | Structural inspection, render when layout matters, content consistency |
| UI/design | Reference comparison, screenshot, interaction or state check |
| Data | Schema, counts, invariants, reconciliation |
| External action | Preview when needed, then read-back and durable receipt |
| Operations | Dry-run when available, state re-query, rollback or residual-risk statement |

Use the strongest project or connector-specific rule when it exceeds this table.

## User-owned gates

Escalate sensitive-data release, secret handling, unapproved external send/publish/deploy,
purchase or cost-threshold crossing, access-control changes, destructive or irreversible action,
regulated or consequential domain judgment, and meaning-changing scope expansion. Do not re-ask
for an external effect already explicitly authorized unless a narrower platform gate requires it.
