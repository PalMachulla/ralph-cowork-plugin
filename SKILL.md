---
name: ralph-cowork-plugin
description: Build a "Ralph for Cowork" plugin that brings autonomous iteration loops to Claude Cowork for non-technical users. Use when creating tools that enable folder-based autonomous work sessions with defined success criteria, iteration limits, and completion detection—extending the Ralph Wiggum methodology beyond Claude Code into general productivity workflows.
---

# Ralph-Cowork Plugin Development

This skill guides development of a Claude Cowork plugin that implements Ralph Wiggum-style autonomous loops for non-developers.

## Concept Overview

**Core insight**: Ralph Wiggum works because it compensates for AI's weakness in "stringing together longer sequences of actions" (METR research shows task completion horizons double every 7 months). Cowork brings Claude Code's agentic capabilities to non-technical users but lacks the iteration infrastructure.

**Goal**: Create a system where non-technical users can:
1. Define a task with clear success criteria
2. Point Claude at a folder
3. Let it iterate autonomously until done or stuck
4. Wake up to completed work

## Architecture Decision

Two implementation paths exist. Evaluate both before proceeding:

**Path A: Native Cowork Extension**
- Build as a Cowork-native feature request/prototype
- Leverages existing sandbox and file access
- Requires Anthropic collaboration or API access
- See `references/architecture.md` → "Native Extension" section

**Path B: Wrapper Application**
- Build standalone app that orchestrates Cowork sessions
- Uses Claude API + local file system
- Can ship independently
- See `references/architecture.md` → "Wrapper Application" section

## Development Workflow

### Phase 1: Define the Loop Mechanism

The core loop requires:

```
1. User defines: Task + Success Criteria + Max Iterations + Folder
2. System initiates Cowork session with folder access
3. Claude works on task
4. System checks for completion signal in output
5. If complete OR max iterations → Stop and report
6. If not complete → Feed same prompt + iteration context back
7. Repeat from step 3
```

Key difference from Ralph Wiggum: No Stop hook available in Cowork. Must implement externally via:
- API polling for session completion
- Output parsing for completion signals
- Session restart with accumulated context

### Phase 2: Success Criteria Engine

Non-technical users need guided success criteria creation. Implement a wizard:

```
Input: "I want to organize my receipts into a spreadsheet"

Wizard output:
- Task: Process receipt images in folder, extract data, create spreadsheet
- Success criteria: 
  - All .jpg/.png files processed
  - expenses.xlsx exists with columns: Date, Vendor, Amount, Category
  - Total row calculated
  - No unprocessed files remain
- Completion signal: <complete>ALL_RECEIPTS_PROCESSED</complete>
- Stuck signal: <stuck>CANNOT_READ_IMAGE: [filename]</stuck>
- Max iterations: 15
```

See `references/prompt-patterns.md` for success criteria templates.

### Phase 3: Context Persistence

Between iterations, preserve:
- Files modified (via folder state)
- Progress log (progress.txt in working folder)
- Iteration count and timestamps
- Error/stuck conditions encountered

Implement as JSON state file:
```json
{
  "task_id": "uuid",
  "prompt": "original task prompt",
  "iteration": 3,
  "max_iterations": 15,
  "started_at": "ISO timestamp",
  "last_iteration_at": "ISO timestamp",
  "status": "running|complete|stuck|max_iterations",
  "completion_signal": "<complete>...</complete>",
  "files_created": ["expenses.xlsx"],
  "files_modified": ["progress.txt"],
  "stuck_reasons": []
}
```

### Phase 4: User Interface

Design for non-technical users:

```
┌─────────────────────────────────────────────────────┐
│  Ralph for Cowork                                   │
├─────────────────────────────────────────────────────┤
│  📁 Working Folder: ~/Documents/Receipts           │
│     [Browse...]                                     │
│                                                     │
│  📝 What do you want done?                          │
│  ┌─────────────────────────────────────────────┐   │
│  │ Organize all receipt images into a          │   │
│  │ spreadsheet with date, vendor, amount...    │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ⚙️ Settings                                        │
│  Max iterations: [15 ▼]                            │
│  ☑ Stop if stuck for 3 iterations                 │
│  ☑ Create progress log                            │
│                                                     │
│  [✨ Generate Success Criteria]                     │
│                                                     │
│  Success Criteria (editable):                       │
│  ┌─────────────────────────────────────────────┐   │
│  │ • All image files processed                 │   │
│  │ • expenses.xlsx created with required cols  │   │
│  │ • Total calculated                          │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  [▶ Start Loop]  [⏹ Cancel]                        │
│                                                     │
│  ─────────────────────────────────────────────     │
│  Status: Iteration 3/15 - Processing receipt_7.jpg │
│  ████████░░░░░░░░ 47%                              │
└─────────────────────────────────────────────────────┘
```

### Phase 5: Safety Controls

Implement mandatory safeguards:

1. **Iteration limits**: Hard cap, default 20, max 100
2. **Cost estimation**: Show estimated API cost before start
3. **Stuck detection**: If same error 3x, pause and notify
4. **File backup**: Snapshot folder state before starting
5. **Kill switch**: Immediate stop button
6. **Scope limits**: Prevent access outside designated folder

## Implementation Checklist

- [ ] Choose architecture path (Native vs Wrapper)
- [ ] Implement core loop mechanism
- [ ] Build success criteria wizard
- [ ] Create context persistence layer
- [ ] Design and build UI
- [ ] Add safety controls
- [ ] Test with use cases from `references/use-cases.md`
- [ ] Document for end users
- [ ] Package for distribution

## Reference Files

- `references/architecture.md` - Technical implementation details
- `references/use-cases.md` - Example workflows for testing
- `references/prompt-patterns.md` - Success criteria templates
- `assets/templates/` - UI mockups and config templates

## Key Success Metrics

The plugin succeeds if:
1. Non-technical user can set up a loop in <2 minutes
2. 80% of common tasks complete without manual intervention
3. Users understand why a loop stopped (success vs stuck vs limit)
4. No runaway cost scenarios possible
