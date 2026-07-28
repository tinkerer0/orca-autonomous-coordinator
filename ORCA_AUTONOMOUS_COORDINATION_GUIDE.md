# Orca 자율 오케스트레이션 운영 가이드

이 문서는 특정 프로젝트나 특정 모델 조합에 종속되지 않는 Orca 운영 기준이다. 목표는
사용자가 하나의 main session에 작업을 맡기면 그 session이 단일 coordinator가 되어 직접
수행, worker 분할, provider 대체, 독립 검증, pane 정리까지 자율적으로 끝내는 것이다.

핵심 최적화 대상은 worker 수가 아니라 **검증된 완료까지의 wall-clock과 재작업 위험**이다.

## 1. 배포물

복사 가능한 Codex skill은 다음 경로에 있다.

```text
orca-autonomous-coordinator/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── decision-policy.md
    ├── pane-layouts.md
    └── worker-capsule.md
```

Codex coordinator에서 자동 발견되게 설치하려면:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R orca-autonomous-coordinator \
  "${CODEX_HOME:-$HOME/.codex}/skills/"
```

다른 agent 제품을 main coordinator로 쓸 때는 `SKILL.md`와 세 reference의 정책을 해당
제품의 global instruction 또는 project instruction에 적용한다. 제품마다 instruction
경로와 skill discovery 방식이 다르므로 한 경로를 공통 정답으로 가정하지 않는다.

설치 후 명시적 호출 예:

```text
Use $orca-autonomous-coordinator to finish this task with the appropriate workers and independent verification.
```

모든 material task에서 자동 적용하려면 global `AGENTS.md`에 다음처럼 짧은 activation
clause만 둔다. 상세 규칙은 skill에 한 번만 유지해 중복과 drift를 줄인다.

```markdown
## Orca coordination

- The user-facing session is the single coordinator.
- For material work, use `$orca-autonomous-coordinator` to choose direct execution or bounded
  orchestration, allocate capabilities, preserve independent verification, and clean up panes.
- Project instructions may add stricter safety, data, cost, Git, or release gates.
```

## 2. 운영 모델

Coordinator는 먼저 아래 계약을 고정한다.

- 목표와 명시적 제외 범위
- 산출물과 acceptance criteria
- 읽기·쓰기·외부 action의 권한
- 사용자가 직접 결정해야 하는 gate
- 동시에 진행 중인 session과 보호할 파일 또는 pane

그다음 assurance와 topology를 **서로 별도로** 선택한다.

### Assurance

| Path | 대상 | 최소 종료 조건 |
|---|---|---|
| `Fast` | 좁고 가역적인 low-risk 작업 | targeted check + coordinator diff/artifact review |
| `Standard` | material하지만 경계가 안정적인 작업 | 구현과 독립된 verifier + coordinator 재확인 |
| `Full` | 민감정보, 비용, 외부/비가역 action, 중대한 claim | 가장 강한 독립 검증 + 필요한 user gate |

`Full`이라고 반드시 병렬 구현할 필요는 없다. 같은 파일이나 순서 의존 작업은 coordinator가
직접 구현하고 강한 독립 verifier만 붙이는 편이 더 안전하고 빠를 수 있다.

### Direct와 orchestration

다음이면 direct가 기본이다.

- 같은 파일을 연속 수정한다.
- shared state 또는 불안정한 interface에 의존한다.
- 작업이 짧아 pane 준비·회수·merge 비용이 더 크다.
- 정확한 분리가 불가능하다.

다음 조건을 모두 만족할 때만 worker를 연다.

- 동시에 준비된 독립 slice가 둘 이상이거나 독립 specialist review가 필요하다.
- exact write path가 겹치지 않는다.
- interface와 acceptance criteria가 안정적이다.
- 병렬 이득이 setup, supervision, recovery, merge 비용보다 크다.

## 3. Provider 배치

Claude, Codex, Grok 같은 이름을 영구 역할표로 고정하지 않는다. 매 task에서 다음 신호로
capability를 배치한다.

- 필요한 추론·구현·research·UI·tool capability
- 현재 tool/runtime 접근 가능성
- quota와 실제 availability
- 필요한 context와 latency/cost
- 최근 산출물의 검증된 품질
- 구현과 검증 사이의 company/session 독립성

사용자가 특정 provider나 역할을 지정하면 우선한다. 그 외의 routine 배치는 coordinator가
묻지 않고 정한다.

Material 또는 high-risk 변경은 구현한 model/company와 다른 company의 새 session이
검증해야 한다. 이를 확보할 수 없으면 결과를 `INDEPENDENCE_UNAVAILABLE`로 표시하고 local
draft까지만 보존한다. 이 상태에서 `GO`, merge, deploy, publish를 선언하지 않는다.

## 4. Worktree와 pane 사용

Main session은 논리적 coordinator다. 실제 worker가 반드시 main session의 현재 worktree를
공유할 필요는 없다.

다음 경우에는 별도 worktree가 기본이다.

- active tree가 dirty다.
- 다른 session이 관련 파일을 작업 중이다.
- 구현이 speculative하거나 rollback 경계를 분리해야 한다.
- verifier가 깨끗한 diff를 독립적으로 읽어야 한다.

격리는 coordinator 역할을 바꾸지 않는다. Main session이 task contract, dispatch,
evidence collection, merge 판단을 계속 소유한다.

기본 pane topology:

| Worker 수 | Layout intent | 비고 |
|---:|---|---|
| 0 | `C` | coordinator 100% |
| 1 | `C \| W1` | 대략 50:50 |
| 2 | `C \| (W1 / W2)` | coordinator를 절반 유지 |
| 3 | 2x2 | ratio API가 없으면 대략 균등 |
| 4+ | wave 또는 별도 owned tab | 깊은 중첩을 피함 |

Orca version에 따라 split direction의 의미와 지원 옵션이 달라질 수 있다. 실행 전 설치된
Orca의 version-matched `orchestration` guide를 읽고, runtime이 equal split만 제공하면
비율을 근삿값으로만 보고한다.

## 5. Worker 계약

모든 dispatch는 적어도 다음을 포함한다.

- finite role이며 coordinator가 되거나 자체 orchestration을 열지 않는다.
- objective, exclusions, acceptance evidence
- exact read/write paths 또는 허용 action
- external side effect, commit, push, deploy, publish 권한
- secret, sensitive data, cost, destructive action, domain gate
- 가장 좁은 관련 검증과 evidence 형식
- 다른 session의 변경을 되돌리지 않는 규칙

`worker_done`은 prose body만으로 완료되지 않는다. `taskId`, `dispatchId`,
`orchestrationRunId`,
수정한 파일/action, evidence, `assume`, `risk`, `untested`, 그리고 권한이 있을 때의 verdict를
완전한 machine-readable payload로 보내야 한다. Coordinator는 body-only 또는 malformed
payload를 quarantine하고, 이미 인정된 작업을 다시 하지 않는 format-only recovery를 한 번만
허용한다.

## 6. Ledger와 provenance

첫 pane mutation 전에 run-scoped, local-only, append-only ledger를 만든다. Project가
validator나 위치를 정했다면 그것을 우선하고, 둘 다 없다면
`work/orchestration/<run-id>/events.jsonl`의 append-only JSONL을 기본값으로 사용한다. 이
ledger는 local evidence이므로 publish artifact에 섞지 않는다. 최소 기록 항목은 다음과 같다.

- run, task, dispatch, event ID와 timestamp
- coordinator/worker의 exact runtime identity
- provider company, role, attempt, state
- tab, pane, PTY, worktree, split parent, open/nesting order
- dispatch scope와 authority
- retry/failover 이유
- `worker_done`, 검증 verdict, close, reconciliation

Pane title이나 화면상의 이름만으로 ownership을 판단하지 않는다. 사용자가 만든 pane,
unrelated session, ownership이 불명확한 pane은 protected로 분류한다.

## 7. 자동 장애 대응

장애를 먼저 분류하고 bounded recovery를 적용한다.

| 장애 | 기본 대응 |
|---|---|
| side effect가 없는 transient runtime/transport | 동일 구성 1회 retry |
| quota/auth/tool/provider unavailable | 동일 실패를 반복하지 않고 capable provider로 1회 재배치 |
| quality/acceptance failure | slice 보강 또는 provider 교체 1회 |
| malformed completion payload | 원본 격리 후 format-only recovery 1회 |
| worker stall | exact liveness 1회 확인, reminder 1회 후 회수/재배치 |
| path/interface collision | 새 dispatch 중단 후 serialize 또는 repartition |
| safety/gate violation | 해당 경로 중단; 우회 배치 금지 |

Quota probe가 없는 runtime에서 availability를 추측하지 않는다. 제한된 attempt로 관측하고
실패를 ledger에 남긴다. 안전이나 비용 gate는 provider 교체로 우회할 수 없다.

## 8. 검증과 종료

Worker의 요약은 evidence이지 최종 사실이 아니다. Coordinator는 실제 artifact를 읽고
artifact 유형에 맞춰 확인한다.

- code: diff, targeted tests, relevant static/runtime check
- document: structure, render when layout matters, content consistency
- research: primary source와 citation binding, independent fact check
- UI: screenshot/reference comparison과 interaction/state check
- data: schema, counts, invariants, reconciliation
- external action: 필요 시 preview, 실행 후 read-back과 durable receipt

종료 순서는 다음과 같다.

1. 권위 있는 `worker_done`을 회수한다.
2. payload와 실제 artifact를 검증한다.
3. agent를 surviving outer shell로 종료한다.
4. owned pane을 reverse nesting/open order로 닫는다.
5. coordinator 생존과 exact identity를 매 close 후 재대조한다.
6. protected/unresolved pane은 보존한다.
7. 관측한 `opened`, `closed`, `unresolved`, `remaining_owned`를 보고한다.

`remaining_owned=0`은 추측하지 않고 pre-final pane audit으로 입증한다.

## 9. 최소 smoke test

실제 프로젝트에 적용하기 전에 side effect가 없는 임시 task로 다음을 확인한다.

1. 격리된 임시 worktree 또는 temporary directory에서 worker 하나가 작은 JSON artifact를
   만든다.
2. 다른 company/session verifier가 read-only로 schema와 hash를 확인한다.
3. Coordinator가 complete `worker_done` payload를 validate한다.
4. Artifact와 ledger를 다시 읽어 acceptance를 확인한다.
5. Worker pane을 모두 닫고 `remaining_owned=0`을 관측한다.
6. Temporary artifact를 삭제하고 원래 worktree의 status/hash가 변하지 않았는지 확인한다.

의도적으로 body-only completion을 한 번 주입해 quarantine과 format-only recovery가
작동하는지도 검증할 수 있다. 이 경우 원본 실패 evidence를 삭제하지 않는다.

## 10. 적용 원칙

- Global instruction에는 activation과 공통 invariants만 둔다.
- Skill에는 재사용 가능한 coordinator policy를 둔다.
- Project instruction에는 data, Git, cost, release, domain gate 같은 더 강한 overlay만 둔다.
- Runtime command syntax는 version-matched Orca guide를 source of truth로 둔다.
- 같은 규칙을 여러 파일에 복제하지 않고 한 canonical source를 링크한다.

이 분리는 모델이 더 자율적으로 판단하게 하면서도, 프로젝트별 안전 경계와 실행 evidence를
잃지 않게 한다.
