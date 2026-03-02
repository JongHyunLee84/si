# SI Workflow Plugin

멀티 플랫폼 소프트웨어 개발 워크플로우 — Research → PRD → Analysis → Architect → UI Design → TDD → Develop → E2E → Acceptance 9단계.

## 아키텍처

```
skills/    → Layer 1 (워크플로우 진입점, si-N-*) → Layer 2 (능력 엔진, si-*) 위임
AGENTS.md  → Codex/Cursor/Copilot 진입점
```

---

## 필수 규칙 — 버전 범프

`skills/`, `hooks/`, `settings/` 내 파일을 변경할 때 **반드시** 아래 두 파일의 `version`을 함께 범프한다.

| 파일 | 필드 |
|------|------|
| `.claude-plugin/plugin.json` | `"version"` |
| `.claude-plugin/marketplace.json` | `plugins[0].version` |

**SemVer 기준:**
- **patch** (0.2.0 → 0.2.1): 버그 수정, 오타, 문구 변경
- **minor** (0.2.0 → 0.3.0): 새 커맨드/스킬 추가, 기존 동작 변경
- **major** (0.x → 1.0): 호환성 깨지는 변경 (커맨드 삭제, 스킬 인터페이스 변경)

> 버전을 범프하지 않으면 marketplace update 시 캐시가 갱신되지 않아 사용자가 변경사항을 받지 못한다.

---

## 크로스 레퍼런스 업데이트 체크리스트

이름이나 경로를 변경할 때 아래 참조를 함께 업데이트해야 한다.

### 워크플로우 스킬 이름 변경 시

1. `skills/si-start/SKILL.md` — Phase Router 테이블 (9개 엔트리)
2. 각 워크플로우 스킬의 완료 메시지 (si-start 참조)
3. `skills/si-7-develop/SKILL.md` — si-6-tdd, si-4-architect 참조
4. `skills/si-8-e2e/SKILL.md` — si-9-acceptance 참조
5. `skills/si-9-acceptance/SKILL.md` — si-7-develop, si-8-e2e, si-4-architect 참조
6. `AGENTS.md` — 스킬 테이블
7. `README.md` — 스킬 테이블
8. `settings/templates/analysis.md` — attribution 코멘트
9. `settings/templates/design.md` — attribution 코멘트

### 능력 스킬 이름 변경 시

1. 호출하는 워크플로우 스킬의 "Follow the `si-*` skill" 문구
2. 스킬 자체 `SKILL.md` frontmatter `name:` 필드
3. `AGENTS.md` — 능력 스킬 테이블
4. `README.md` — 능력 스킬 테이블

### settings/rules 파일 변경 시

1. 참조하는 워크플로우 스킬: `si-start`(design-review), `si-3-analysis`(gap-analysis), `si-4-architect`(design-review)
2. `skills/si-prd/SKILL.md` — ears-format 참조

### settings/templates 파일 변경 시

1. 참조하는 워크플로우 스킬: `si-3-analysis`(analysis.md), `si-4-architect`(design.md), `si-start`(si-progress.json)
2. 템플릿 내부 attribution 코멘트

### tasks/ 아티팩트 경로 변경 시

| 아티팩트 | 영향 범위 |
|----------|----------|
| `tasks/si-progress.json` | 모든 워크플로우 스킬 + session-start hook (가장 영향 큼) |
| `tasks/design.md` | 6개 워크플로우 스킬 + 3개 능력 스킬 |
| `tasks/requirements.md` | 5개 워크플로우 스킬 + 2개 능력 스킬 |
| `tasks/<phase>/` | 해당 phase 스킬 + 다운스트림 phase (서브리포트 참조) |
| `tasks/architect/research.md` | si-4-architect 내부 (기존 `tasks/research.md` 대체) |
