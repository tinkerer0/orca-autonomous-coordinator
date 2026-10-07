# Orca Autonomous Coordinator

Orca에서 한 coordinator가 에이전트의 작업 분담과 독립 검토를 조율하도록 만든 운영 지침.

> **이전 공개판** — 2026-08-05까지 공개한 지침과 설치 예시를 보존합니다. 현재 운영 중인 같은 이름의 스킬과 버전이 다르므로, 아래 문서는 이 공개판을 기준으로 읽어주세요.

[운영 가이드](ORCA_AUTONOMOUS_COORDINATION_GUIDE.md) · [당시 적용 안내](00_APPLY_THIS_FIRST.md) · [스킬 본문](orca-autonomous-coordinator/SKILL.md) · [MIT 라이선스](LICENSE)

<details>
<summary>당시 사용법과 설계 기록 보기</summary>

아래는 이전 README를 보존한 기록입니다. 사용·설치·검증 상태에 관한 표현도 당시 기준입니다.

A reusable operating policy for letting one user-facing LLM coordinate the other agents
available in Orca.

The coordinator decides whether to work directly or delegate, assigns providers by capability
and availability, preserves independent verification, performs bounded failover, isolates risky
work in worktrees, and cleans up owned worker panes.

This repository contains coordination instructions and a Codex-compatible skill. It does not
include or modify Orca itself.

## Updates

- **2026-08-05** — the skill's ledger contract gained a fixed `stage` vocabulary, a run
  topology declaration (`meta.topology`), and a one-line-per-run `RUNS.md` index (SKILL.md §6,
  §11). The Korean addendum
  [`ORCA_GUIDE_UPDATE_2026-08-05.md`](ORCA_GUIDE_UPDATE_2026-08-05.md) explains the
  measurements behind the change and two policy-layer updates (delegation by judgment, honest
  reporting norms), and states which parts of this system are hard invariants versus judgment
  guidance.

## Easiest installation

Give this folder or its ZIP archive to an LLM with local file access and say:

```text
Read 00_APPLY_THIS_FIRST.md and apply it while preserving my existing settings and completed work.
```

The LLM-facing installer contract is in
[`00_APPLY_THIS_FIRST.md`](00_APPLY_THIS_FIRST.md).

## Manual Codex installation

From the extracted directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R orca-autonomous-coordinator \
  "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Then add the short activation clause from
[`ORCA_AUTONOMOUS_COORDINATION_GUIDE.md`](ORCA_AUTONOMOUS_COORDINATION_GUIDE.md)
to the user's global `AGENTS.md` without replacing existing instructions. Open a new main
session, or explicitly invoke:

```text
Use $orca-autonomous-coordinator to coordinate this task safely and efficiently.
```

Other agent hosts should use their documented global-instruction or skill mechanism. Do not
invent a universal configuration path.

## Operating model

- One user-facing main session acts as the coordinator.
- Short, sequential, or same-file work stays direct.
- Only disjoint, worthwhile slices become worker tasks.
- Provider roles are selected from capabilities, tools, availability, cost/latency, and recent
  verified quality instead of a permanent vendor roster.
- Material changes receive verification from a different company and session.
- Quota, tool, runtime, and quality failures use a finite retry/reallocation budget.
- Existing panes and concurrent changes remain protected.
- A worker run ends with evidence validation, exact pane cleanup, and an integrated report.

## Included files

```text
README.md
LICENSE
00_APPLY_THIS_FIRST.md
ORCA_AUTONOMOUS_COORDINATION_GUIDE.md
ORCA_GUIDE_UPDATE_2026-08-05.md
orca-autonomous-coordinator/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── decision-policy.md
    ├── pane-layouts.md
    └── worker-capsule.md
```

The detailed operator guide is currently written in Korean. The installed skill and its
references are in English.

## Safety boundaries

This policy does not authorize an LLM to bypass secrets, sensitive-data handling, costs,
destructive actions, external publication, deployment, or project-specific approval gates.
Stricter project rules always win.

Orca command syntax and pane behavior may vary by version. The coordinator must load the
version-matched Orca orchestration guide before manipulating panes.

## Validation

For Codex installations with the official `skill-creator` validator available:

```bash
uv run --with pyyaml --no-project python \
  /path/to/skill-creator/scripts/quick_validate.py \
  orca-autonomous-coordinator
```

An existing Python environment with `PyYAML` may be used instead of `uv`.

Before publishing a modified bundle, also check that it contains no personal paths, credentials,
project-specific names, run ledgers, or generated work artifacts.

## 기여·피드백

제가 이런 분야를 접한 지 얼마 안 돼서 부족한 점이 많습니다. 고칠 점이나 알려주실 내용이
있다면 issue나 PR로 남겨주시면 너무 감사하겠습니다.

## License

MIT. See [`LICENSE`](LICENSE).

</details>

상태 확인: 2026-10-07.
