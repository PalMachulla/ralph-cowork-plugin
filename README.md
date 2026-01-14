# Ralph for Cowork

> Autonomous iteration loops for Claude Cowork — bringing Ralph Wiggum methodology to non-technical users.

## What is this?

[Claude Cowork](https://www.anthropic.com) brings Claude Code's agentic capabilities to non-developers through the Claude Desktop app. This project extends Cowork with **autonomous iteration loops** — the same pattern that powers the [Ralph Wiggum plugin](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) for Claude Code.

**The problem**: AI agents struggle with long tasks not because they lack skills, but because they have difficulty [stringing together sequences of actions](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/). Task completion horizons are doubling every 7 months, but we need infrastructure to extend them further.

**The solution**: Define success criteria, point at a folder, let Claude iterate autonomously until done.

## Status

🚧 **Early Development** — This is a design document and skill specification. Implementation in progress.

## Use Cases

- **Receipt organization** → Expense spreadsheet
- **Meeting notes** → Consolidated summary with action items
- **Research papers** → Literature review synthesis
- **Messy downloads** → Organized folder structure
- **Raw data** → Analysis with insights and recommendations

See [use-cases.md](references/use-cases.md) for 10 detailed scenarios with success criteria.

## Architecture

Two implementation paths:

| Approach | Pros | Cons |
|----------|------|------|
| **Wrapper App** | Ship independently, full control | Separate API costs, build own sandbox |
| **Native Extension** | Uses Cowork sandbox, subscription-included | Requires Anthropic collaboration |

Recommended: Start with Wrapper for speed, design for Native compatibility.

See [architecture.md](references/architecture.md) for technical details.

## Core Concept

```
1. User defines: Task + Success Criteria + Max Iterations + Folder
2. System initiates session with folder access
3. Claude works on task
4. System checks for completion signal: <complete>TASK_DONE</complete>
5. If complete OR max iterations → Stop and report
6. If not complete → Feed same prompt + context back
7. Repeat
```

## Project Structure

```
ralph-cowork-plugin/
├── SKILL.md                      # Development guide (for Claude)
├── README.md                     # This file
├── references/
│   ├── architecture.md           # Technical implementation details
│   ├── use-cases.md              # Test scenarios
│   └── prompt-patterns.md        # Success criteria templates
└── assets/templates/
    ├── task-config.schema.json   # Task configuration schema
    ├── example-receipt-task.json # Example configuration
    └── session-state.schema.json # Runtime state schema
```

## Getting Started

This repo currently contains the **design specification**. To contribute:

1. Review [SKILL.md](SKILL.md) for the development roadmap
2. Check [architecture.md](references/architecture.md) for implementation options
3. Pick a component and start building

### Planned Components

- [ ] Core loop mechanism
- [ ] Success criteria wizard (NL → structured criteria)
- [ ] Context persistence layer
- [ ] Desktop UI (Tauri or Electron)
- [ ] Safety controls (backup, rate limiting, kill switch)

## Why "Ralph"?

Named after [Ralph Wiggum](https://en.wikipedia.org/wiki/Ralph_Wiggum) from The Simpsons. The philosophy: persistent iteration despite setbacks. Don't aim for perfect on first try — let the loop refine the work.

> "Ralph is a Bash loop." — Geoffrey Huntley

## Related Projects

- [ralph-wiggum](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) — Official Claude Code plugin
- [ralph-claude-code](https://github.com/frankbria/ralph-claude-code) — Community implementation with monitoring
- [ralph-wiggum-marketer](https://github.com/muratcankoylan/ralph-wiggum-marketer) — Ralph for content marketing

## Contributing

Contributions welcome! This is an early-stage project exploring how to make autonomous AI work accessible to non-developers.

## License

MIT

---

Built by [AIAKAKI](https://aiakaki.com)
