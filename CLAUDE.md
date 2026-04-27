# CLAUDE.md

## Context Bridge Protocol

Tinkersbin uses a context bridge between Claude Web Chat (architecture) and Claude Code (implementation). Communication happens through the `.context/` directory.

### For Claude Web Chat (architect)
1. Read `.context/project.md` for project scope and constraints
2. Read `.context/status.md` for current state
3. Read `.context/decisions/` for prior architectural decisions
4. Write handoffs as **intent and expected outcomes**, not as prescriptive code recipes. The user has explicitly asked that web chat operate as architect/designer and not specify how code should be written. Interface signatures and concrete API shapes are appropriate; function bodies, algorithms, and stylistic prescriptions are not.

### For Claude Code (builder)
1. Read `.context/project.md` for project scope and constraints
2. Read `.context/status.md` for current state
3. Read `.context/decisions/` for architectural decisions that constrain your work — these are constraints, not suggestions
4. Read `.context/handoff-to-code.md` for your current task

### Your Workflow (Claude Code)
1. Read the handoff and understand what's being asked
2. Read relevant ADRs — these are constraints, not suggestions
3. Build what's requested; surface real choices in `handoff-to-chat.md` rather than guessing silently
4. Update `.context/status.md` with current component states
5. Write `.context/handoff-to-chat.md` with:
   - What you built (file/module descriptions, not full code listings)
   - Decisions you made within the latitude given (e.g. specific column types, similarity approach for pHash) and your reasoning
   - Deviations from the handoff (if any) with explanation
   - Questions for web chat
6. Commit with `Author: code` in the message and push

### Rules
- Follow ADRs. If an ADR seems wrong for the task, note it in your handoff — don't ignore it
- Don't modify `.context/project.md` or `.context/decisions/` — those are web-chat's domain
- Do update `.context/status.md` and `.context/handoff-to-chat.md`
- Commit messages must include "Author: code"
- The user's code style is governed by ADR-006. Read it before writing PHP, SQL, or Python.
- The user has 30 years of experience and strong opinions. He prefers single-line PDO queries, no ORM, no framework ceremony, functionality over convention. Don't introduce abstractions that don't earn their place.
- If on a feature branch, stay on that branch — do NOT merge to main.
