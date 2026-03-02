---
name: si:save
description: 현재 SI 워크플로우 진행 상태를 명시적으로 저장
user-invocable: true
---

# SI Save

Save the current SI workflow state for session recovery.

## Execution

1. Read `tasks/si-progress.json`
2. Update `updatedAt` to current timestamp
3. Ask user: "현재 작업에 대해 메모할 내용이 있습니까?" (optional)
4. If user provides notes, append to `notes` field
5. Write updated `tasks/si-progress.json`
6. Display confirmation:

```
── SI State Saved ───────────────────────
Project:    [name]
Phase:      [current]
Completed:  [list]
Saved at:   [timestamp]
Notes:      [first 50 chars of notes...]
─────────────────────────────────────────
```

## When to Use
- Before ending a session
- Before switching to a different task
- After making significant progress within a phase
- As a manual checkpoint (complementing auto-save on phase transitions)

## Recovery
Next session: SessionStart hook will automatically detect `tasks/si-progress.json` and load the saved state.
Or run `/si:start` to see the full status dashboard.
