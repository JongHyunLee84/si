# SI Workflow Plugin

SI 스타일 소프트웨어 개발 워크플로우를 Claude Code 플러그인으로 구현.

## Pipeline

```
Research → PRD → Analysis → Architect → TDD → Develop → E2E → Acceptance
```

## 설치

```bash
claude plugins add ~/Documents/si
```

## 커맨드

| Command | Phase | Description |
|---------|-------|-------------|
| `/si:start` | - | 오케스트레이터: Phase 추적, 게이트, 라우팅 |
| `/si:research` | 1 | 시장/기술 리서치 |
| `/si:prd` | 2 | 인터뷰 기반 요구사항 정의 |
| `/si:analysis` | 3 | 코드/시스템 분석, 갭 분석 |
| `/si:architect` | 4 | 기술 설계, ADR, 수용 기준 |
| `/si:tdd` | 5 | Red-Green-Refactor 사이클 |
| `/si:develop` | 6 | 구현 가이드 |
| `/si:e2e` | 7 | E2E 테스트 |
| `/si:acceptance` | 8 | 준수율 채점 + 최종 검증 |
| `/si:save` | - | 진행 상태 저장 |

## 게이트 (사용자 승인 필요)

1. Research 완료 후 — 진행 가치 판단
2. PRD 완료 후 — 요구사항 확인
3. Architect 완료 후 — GO/NO-GO 설계 리뷰

## 산출물

각 Phase가 `tasks/` 디렉토리에 산출물을 생성:

- `tasks/si-progress.json` — Phase 추적
- `tasks/research-report.md` — 리서치 결과
- `tasks/requirements.md` — 요구사항 정의서
- `tasks/analysis.md` — 분석 보고서
- `tasks/design.md` — 설계 문서
- `tasks/research.md` — 기술 조사 버퍼
- `tasks/acceptance-report.md` — 최종 검증 리포트

## 차용 패턴

- **cc-sdd**: EARS 요구사항, Gap 분석, GO/NO-GO 리뷰
- **shinpr**: 에스컬레이션 체크, 정량 채점, 금지 어휘
- **ATDD**: Given/When/Then 수용 기준

## License

MIT
