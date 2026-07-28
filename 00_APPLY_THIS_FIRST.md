# LLM에게 주는 적용 지시

사용자는 이 폴더의 Orca 자율 오케스트레이션 방식을 자신의 환경에 실제로 적용하려고 한다.
설명만 하지 말고 아래 절차대로 설치·검증하라.

## 목표

- 사용자가 대화하는 main session 하나를 single coordinator로 둔다.
- Coordinator가 direct 실행과 multi-agent orchestration을 자율적으로 선택한다.
- 특정 provider를 고정 역할로 배치하지 않고 capability, availability, tool access,
  latency/cost, 최근 검증 품질로 배치한다.
- Material 작업은 구현과 다른 company/session의 독립 검증을 보존한다.
- Quota, tool, context, quality failure가 발생하면 bounded failover로 자동 재구성한다.
- Worker pane을 열었다면 provenance, ledger, exact cleanup까지 완료한다.

## 적용 절차

1. 이 파일과 다음 파일을 원문 전체로 읽는다.
   - `ORCA_AUTONOMOUS_COORDINATION_GUIDE.md`
   - `orca-autonomous-coordinator/SKILL.md`
   - `orca-autonomous-coordinator/references/*.md`
   - `orca-autonomous-coordinator/agents/openai.yaml`
2. 현재 환경의 agent 제품과 공식 global instruction/skill 경로를 확인한다. 경로를 추측하지
   않는다.
3. Codex-compatible 환경이면 `orca-autonomous-coordinator` 폴더 전체를
   `${CODEX_HOME}/skills/` 또는 `~/.codex/skills/` 아래에 설치한다.
4. 기존 global instruction을 덮어쓰지 말고 다음 activation clause만 병합한다.

```markdown
## Orca coordination

- The user-facing session is the single coordinator.
- For material work, use `$orca-autonomous-coordinator` to choose direct execution or bounded
  orchestration, allocate capabilities, preserve independent verification, and clean up panes.
- Project instructions may add stricter safety, data, cost, Git, or release gates.
```

5. Claude, Grok 또는 다른 제품이 main coordinator라면 그 제품이 공식 지원하는 global
   instruction/skill 방식에 같은 activation과 bundle policy를 적용한다. 지원 여부가 불명확하면
   임의 경로를 만들지 말고 사용 가능한 방식과 미적용 부분을 보고한다.
6. 기존 project instruction과 안전 규칙은 삭제하거나 약화하지 않는다. Project-specific
   data, secret, cost, Git, release, destructive-action gate가 이 bundle보다 강하면 그것을
   우선한다.
7. 기존 완료 작업, commit, 산출물은 재실행하거나 수정하지 않는다. 이 설치를 이유로 worker
   pane을 열거나 project source code를 변경하지 않는다.
8. 설치된 skill을 공식 validator가 있으면 검증한다. 없으면 frontmatter, reference path,
   `agents/openai.yaml`의 구조와 링크를 확인한다.
9. 변경한 exact path, 검증 명령과 exit status, 적용하지 못한 부분을 사용자에게 간단히
   보고한다.

## 기존 session 처리

새 session부터 확실히 적용된다. 기존 main session에는 다음 한 문장만 전달하면 된다.

```text
완료된 작업은 그대로 두고, 다음 새 작업부터 새 Orca coordinator 규칙을 적용해. 지금은 재작업하지 말고 대기해.
```

기존 bounded worker는 coordinator로 승격하지 말고 원래 역할을 유지한다.

## 사용자가 말할 문장

이 폴더를 첨부한 뒤 다음처럼 요청하면 된다.

```text
00_APPLY_THIS_FIRST.md를 읽고, 기존 설정과 완료된 작업은 보존하면서 이대로 설치·적용해줘.
```
