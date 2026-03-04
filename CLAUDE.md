# Factory — Software Development Toolbox

독립적으로 호출 가능한 9개 스킬의 소프트웨어 개발 도구 상자. 각 스킬은 `factory/<skill>/` 디렉토리에 작업 기록을 남기고, 다른 스킬은 이를 참고할 수 있지만 강제하지 않는다.

## 아키텍처

```
skills/    → 단일 레이어: 9개 독립 스킬 (factory-*)
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
- **patch** (1.0.0 → 1.0.1): 버그 수정, 오타, 문구 변경
- **minor** (1.0.0 → 1.1.0): 새 커맨드/스킬 추가, 기존 동작 변경
- **major** (1.x → 2.0): 호환성 깨지는 변경 (커맨드 삭제, 스킬 인터페이스 변경)

> 버전을 범프하지 않으면 marketplace update 시 캐시가 갱신되지 않아 사용자가 변경사항을 받지 못한다.

---

## 크로스 레퍼런스 업데이트 체크리스트

이름이나 경로를 변경할 때 아래 참조를 함께 업데이트해야 한다.

### 스킬 이름 변경 시

1. 각 스킬의 Scope Boundary — 다른 스킬 참조 (`factory-*`)
2. 각 스킬의 Recommended Inputs 섹션
3. 각 스킬의 완료 메시지 내 스킬 참조
4. `AGENTS.md` — 스킬 테이블
5. `README.md` — 스킬 테이블

### settings/rules 파일 변경 시

1. 참조하는 스킬: `factory-architect` (design-review), `factory-analysis` (gap-analysis), `factory-prd` (ears-format)

### settings/templates 파일 변경 시

1. 참조하는 스킬: `factory-analysis` (factory-analysis.md), `factory-architect` (factory-architect.md), 전체 스킬 (decision-agenda.md)
2. 템플릿 내부 attribution 코멘트

### factory/ 아티팩트 경로 변경 시

| 아티팩트 | 영향 범위 |
|----------|----------|
| `factory/architect/architect.md` | 6개 스킬 (tdd, develop, e2e, code-review, ui-design, analysis) |
| `factory/prd/prd.md` | 5개 스킬 (analysis, architect, tdd, develop, code-review) |
| `factory/research/research.md` | 3개 스킬 (prd, analysis, architect) |
| `factory/analysis/analysis.md` | 3개 스킬 (architect, develop, code-review) |
| `factory/ui-design/ui-design.md` | 1개 스킬 (tdd) |
| `factory/<skill>/decisions.md` | 해당 스킬 + 모든 하류 스킬 (Decision Agenda 패턴) |
