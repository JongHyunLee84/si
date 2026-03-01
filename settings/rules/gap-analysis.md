# Gap Analysis Framework

## Purpose
Evaluate implementation approaches with structured multi-criteria analysis.
Eliminates subjective "recommendations" — only evidence-based decisions.

## Banned Vocabulary
These words are FORBIDDEN in analysis output:
- "recommended" → use "mandatory" / "not-required" / "conditional"
- "consider" → use "must" / "must-not" / "conditional-on [trigger]"
- "might" / "could" / "should consider" → state the fact or mark "unknown"

## 3-Option Rule
ALWAYS evaluate exactly 3 implementation approaches:

| Criteria | Option A: Extend Existing | Option B: New Module | Option C: Hybrid |
|----------|--------------------------|---------------------|------------------|
| Effort | S / M / L / XL | S / M / L / XL | S / M / L / XL |
| Risk | High / Medium / Low | High / Medium / Low | High / Medium / Low |
| Maintainability | High / Medium / Low | High / Medium / Low | High / Medium / Low |
| Alignment | How well fits current architecture | How well fits current architecture | How well fits current architecture |

## T-shirt Sizing Guide
- **S**: 1-2 files, <100 lines changed, no new dependencies
- **M**: 3-5 files, 100-500 lines, may add internal module
- **L**: 6-10 files, 500-2000 lines, may add dependency
- **XL**: 10+ files, 2000+ lines, architecture-level change

## Risk Classification
- **High**: Data model change, security boundary, external API, breaking change
- **Medium**: New module/service, significant logic change, performance-sensitive
- **Low**: UI-only, additive change, isolated scope, well-tested area

## Gap Tags
Each identified gap MUST be tagged:
- `Missing` — does not exist, must be built
- `Unknown` — insufficient information, research needed (mark for Architect phase)
- `Constraint` — exists but blocked by technical/business limitation

## Escalation Checklist
ANY "Yes" → immediate user notification:
- [ ] Architecture layer change required?
- [ ] External dependency addition required?
- [ ] Existing data model modification required?
- [ ] Security or performance impact?
