# Orca 자율 오케스트레이션 가이드 — 추가분 (2026-08-05)

이 문서는 [`tinkerer0/orca-autonomous-coordinator`](https://github.com/tinkerer0/orca-autonomous-coordinator)
(2026-07-28 Initial public release)의 `ORCA_AUTONOMOUS_COORDINATION_GUIDE.md`를 실제로
몇 주 운용하면서 추가·확정된 규칙 3가지를 정리한 것이다. 게시본과 같이 읽으면 각각
**§6 (Ledger와 provenance)**, **§2 (운영 모델)**, **§8 (검증과 종료)**의 확장이고,
이 문서만 읽어도 쓸 수 있게 자체 완결로 적었다.

요약: 게시본은 "무엇을 기록하라"까지 말했는데, 운용해 보니 **기록 요구만으로는 어휘가
갈라지고 습관이 퇴행**했다. 그래서 ① ledger의 어휘·형식을 고정하고 읽히게 만들었고,
② 워커 위임을 강제 규칙에서 판단 기준으로 바꿨고, ③ 코디네이터 보고에 정직성 규범을
붙였다.

---

## 0. 층위 — 이 문서(와 게시본)를 읽는 법

이 체계는 "규칙집"이 아니라 가이드다 — 내용 대부분이 코디네이터가 상황을 보고 판단하라는
기준이기 때문이다. 다만 전부 재량은 아니다. 층이 셋 있고, 첫 번째 불변식 층만은 어기면
안 된다:

**불변식 (항상 강제):**

1. 구현한 쪽(모델·회사·세션)은 그 변경의 독립 검증자가 될 수 없다 — 코디네이터가 직접
   구현한 경우에도 같다.
2. 안전·비밀·비용·파괴적 행동 게이트는 provider 교체나 failover로 우회할 수 없다.
3. 소유가 불명확한 pane·다른 세션의 산출물은 protected — 건드리지 않는다.
4. ledger는 append-only — 과거 이벤트를 고치지 않고 새 이벤트로 정정한다.
5. `worker_done`의 machine-readable payload 없이는 완료로 인정하지 않는다(산문 보고는
   설명일 뿐이다).
6. 독립 검증을 확보할 수 없으면 결과에 `INDEPENDENCE_UNAVAILABLE`을 표시하고
   GO·merge·deploy·publish를 선언하지 않는다.

나머지 — direct vs 워커 분할, assurance 수준, provider 배치, worktree 격리 — 는 전부
**판단 기준**이다: 상황에 맞게 고르되, 어느 쪽을 골랐는지와 왜를 기록으로 남긴다.
pane layout·stage 어휘·capsule 템플릿 같은 **참고 스펙**은 세 번째 층이고, 프로젝트가
더 강한 규칙을 가지면 그쪽이 이긴다.

---

## 1. Ledger 관측성 — stage 어휘 고정 + topology 선언 + run 인덱스 (§6 확장)

### 왜 필요했나 (실측 배경)

한 워크스페이스에서 8일간 orchestration run 13개, event 765건을 쌓은 뒤 재봤더니:

- `stage` 값이 **104종**으로 파편화 — 상위 12종이 전체의 81%를 덮는데, 나머지는 한
  run이 `v13_4_implementation_complete`처럼 **버전·마일스톤을 stage 이름에 인코딩**해
  일회용 stage를 ~40종 만든 결과였다.
- run topology(누가 누구에게 무엇을 보내는가)는 13개 run 중 10개가 적었지만 전부
  1줄 자유문이었고, **가장 크고 최신인 run이 0회로 퇴행**했다. 즉 "기록하라"는 요구가
  스킬에 있어도 어휘와 위치를 못 박지 않으면 습관은 샌다.
- events.jsonl을 읽는 도구와 run 목록(cross-run 인덱스)은 0이었다 — write-only ledger.

교훈: **그래프 오케스트레이션 도구를 새로 만들 필요는 없다. 이미 찍히는 ledger를
채우고 읽히게 하면 된다.** 아래 세 가지가 그 최소 장치다.

### 1.1 stage 화이트리스트 (12종 + other)

`events.jsonl`의 `stage` 필드는 아래 12종만 쓴다. run의 생애주기를 그대로 따라간다.

| stage | 시점 |
|---|---|
| `run_created` | run 개시 |
| `plan_fixed` | 계약·계획 확정 (갱신 시 새로 append) |
| `split_requested` | pane/worker 분할 요청 |
| `agent_ready` | worker agent 기동 확인 |
| `dispatched` | 작업 전달 |
| `first_heartbeat` | 첫 생존 신호 (liveness ≠ 완료) |
| `worker_done` | worker 완료 payload 수신 |
| `review_done` | 독립 리뷰 라운드 종료 |
| `remediation_done` | 지적사항 수정 완료 |
| `coordinator_verified` | 코디네이터가 artifact를 직접 재확인 |
| `pane_closed` | owned pane 닫음 |
| `run_reconciled` | run 종료 대사(reconciliation) 완료 |

- 여기 없는 이벤트는 `stage: "other"` + **`meta.stageDetail` 필수**로 적는다.
- 같은 `stageDetail`이 3회 이상 재발하면 화이트리스트 승격을 검토한다 (ratchet).
- **버전·마일스톤·배치 라벨은 절대 stage 이름에 넣지 않는다** — `meta.milestone`,
  `meta.batchId`로. (위의 104종 파편화가 정확히 이걸 어겨서 생겼다.)
- 과거 run의 이벤트는 소급 수정하지 않는다. 읽는 쪽이 미등재 stage를 `other`로
  취급하면 된다.

### 1.2 topology 선언 — 첫 coordinator `plan_fixed`의 `meta.topology`

run에서 워커를 여는 순간, 그 run의 **첫 coordinator-task `plan_fixed` 이벤트**에
배선을 선언한다. 3~8줄이면 충분하고, 8줄로 안 그려지면 스키마를 키울 게 아니라
run을 쪼갤 신호다.

```json
{
  "stage": "plan_fixed",
  "taskKey": "coordinator",
  "meta": {
    "topology": {
      "nodes": ["C=coordinator@Codex", "W1=impl@Claude", "V1=verify@Grok"],
      "edges": ["C→W1 dispatch", "W1→C worker_done", "C→V1 artifact", "V1→C verdict"],
      "sharedArtifacts": ["src/foo.py (W1 write, V1 read-only)"]
    },
    "implementationCompany": "Anthropic",
    "verifierCompany": "xAI"
  }
}
```

- `implementationCompany` / `verifierCompany`는 topology 안에 녹이지 말고 **별도 키로
  유지**한다 — "구현한 쪽 ≠ 검증자" 불변식을 기계가 대조할 수 있는 자리다.
- 배선이 바뀌면(worker 추가·provider 교체) 과거 이벤트를 고치지 말고 **새 coordinator
  `plan_fixed`를 append**한다. **최신 것이 유효(last-wins)** — append-only ledger와
  일관된다.
- 워커가 없는 direct run이면 1줄 자유문도 유효하다 (예: `"coordinator direct,
  post-hoc independent verifier only"`). 구조화 비용 때문에 기입 자체를 포기하게
  만들지 않는 게 우선이다.

### 1.3 RUNS.md — run 종료 시 1줄 인덱스

`work/orchestration/RUNS.md`에 run당 1줄. run을 닫을 때 코디네이터가 append한다.

```
- <시작일[→종료일]> | <run-id> | <N> ev | topo:<topology 기입 수> | last: <마지막 stage> | <objective 1줄>
```

예:

```
- 2026-08-01→2026-08-03 | orch_feature_x_20260801 | 229 ev | topo:3 | last: run_reconciled | Implement X with frozen schema and independent verification
```

`topo:` 칼럼이 사실상 준수 카운터다 — `topo:0`인 run이 목록에 보이는 순간 누락이
드러난다. 기존 run들은 events.jsonl에서 스크립트로 한 번 시드하면 되고(읽기 전용
추출), 소급 편집은 하지 않는다.

### 1.4 감시 장치 (선택, 권장)

위 1.1~1.3은 전부 모델이 스킬을 따라 쓰는 것이라 **강제가 아니다**. 이미 돌고 있는
일일 헬스체크(cron 등)가 있다면 검사 두 개를 얹으면 습관 퇴행이 자동으로 보인다:

1. 정책 도입일 **이후 시작한** run에 `meta.topology`가 0회면 경고.
2. 마지막 이벤트가 48시간 지난(종료 추정) run이 RUNS.md에 미등재면 경고.

포인트는 "도입일 이후만"이다 — 과거 run을 소급 경고하면 만성 경고에 묻혀서
(alert fatigue) 정작 새 위반을 놓친다.

### 하지 말 것

- 과거 run에 topology 소급 백필 금지 (뷰가 결측을 관용 처리).
- stage 이름에 버전·마일스톤 접두어 금지.
- 화이트리스트·topology 규칙을 별도 문서로 만들지 말 것 — **SKILL.md 한 곳**에만
  (부록 A 패치 참조). 문서가 늘면 어긋나기 시작한다.
- 그래프 시각화 대시보드부터 만들지 말 것 — 텍스트 인덱스와 grep이 먼저다.

---

## 2. 위임은 판단제 — 강제 트리거를 "신호"로 (§2 보강)

초기에는 "새 파일 생성 시 위임", "3파일 이상이면 워커 분할" 같은 강제 트리거를
운용했는데, 실측 결과 규칙은 우회되거나 형식적 준수만 남았다. 현재 규칙:

- 새 파일·새 함수·다파일 변경은 **위임을 검토하라는 신호**일 뿐 강제가 아니다.
- direct vs 위임 판정 기준은 4축: **① 메인 컨텍스트 보존** (코디네이터 컨텍스트를
  대량 소모할 작업인가) **② 규모·전문성 ③ provider 가용성 ④ 검증자 확보**
  (불변식: 구현한 쪽은 그 변경의 검증자가 될 수 없다).
- 판정은 착수 시 한 번 하고, 어느 쪽을 골랐든 **보고에 한 줄 남긴다** — "위임 안
  했음"도 기록되면 나중에 패턴을 잴 수 있다.

사용자에게 묻는 것도 세 축으로 정리됐다:

| 축 | 처리 |
|---|---|
| 기술 판단 (뭐가 더 나은가) | 묻지 않고 정한 뒤 보고에 남김 |
| 취향·의도 (뭘 원하는가) | 새롭거나 모호한 것만 확인, 알려진 취향은 따름 |
| 권한 (비가역·비용·비밀·외부 전송·운영 자원·요청 범위 변경) | 능력과 무관하게 승인 |

---

## 3. 코디네이터 보고의 정직성 규범 (§8 확장)

게시본 §8의 "worker의 요약은 evidence이지 최종 사실이 아니다"를 코디네이터 자신의
보고까지 확장한 것이다. 보고 문장마다 라벨을 붙이는 의례가 아니라, 서술의 상한을
근거의 강도에 맞추는 계약이다.

- **강도** — 근거의 강도가 주장의 상한: `VERIFIED`(실행해서 결과를 관찰) /
  `INSPECTED`(파일·diff·정적 상태만 봄) / `UNVERIFIED`(확인 못 함) /
  `FAILED`(확인했더니 어긋남). 정적 관찰만으로 실행 결과를 주장하지 않는다.
- **주체** — 내가 직접 관찰한 것과 워커·외부 모델이 보고한 것을 구분한다. 위임
  결과를 재관찰하지 않았으면 "워커 보고"라고 밝힌다.
- **범위** — 부분을 전체로 말하지 않는다. 실패·미완이 하나라도 있으면 "완료"라고
  쓰지 않고 무엇이 되고 안 됐는지 나눈다.
- **미완** — 확인 못 한 것은 숨기지 말고, "무엇을 하면 확인되는지" 한 줄과 함께
  남긴다.
- **근거** — 실행으로 확인한 주장에는 실제 명령과 결정적 출력 한 줄을 붙인다.
- **약속** — "나중에 하겠다"는 실행 주체(사람·cron·hook·파일 기록)가 실재할 때만.
  세션이 끝나면 사라질 약속이면 장치에 걸고 무엇에 걸었는지 밝힌다.

---

## 부록 A — SKILL.md 패치 원문

게시본 repo의 `orca-autonomous-coordinator/SKILL.md`에 그대로 반영하면 되는 추가분.

**§6 (ledger 단락) 끝에 추가:**

```
Constrain `stage` to run_created, plan_fixed, split_requested, agent_ready, dispatched,
first_heartbeat, worker_done, review_done, remediation_done, coordinator_verified,
pane_closed, and run_reconciled, plus `other` with a required `meta.stageDetail`; keep
version, milestone, and batch labels in `meta`, never in the stage name. Declare run
topology in the run's first coordinator-task `plan_fixed` as `meta.topology` — nodes as
`role@provider`, edges, and shared exact paths — alongside `implementationCompany` and
`verifierCompany`; rewiring appends a new coordinator `plan_fixed`, and the latest wins.
```

**§11 (보고 단락) 끝에 추가:**

```
At run close, append a one-line summary (run id, dates, objective, closing stage) to
`work/orchestration/RUNS.md`.
```

---

*이 문서는 게시본과 마찬가지로 특정 프로젝트·모델 조합에 종속되지 않는다. 실측
수치는 한 워크스페이스의 운용 기록에서 나온 것이라 환경에 따라 다를 수 있고,
화이트리스트 12종도 정답이 아니라 출발점이다 — 각자 ledger에서 빈도를 재서 다듬는
쪽을 권한다.*
