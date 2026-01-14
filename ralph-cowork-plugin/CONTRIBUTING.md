# Contributing to Ralph for Cowork

Thanks for your interest in contributing! This project is in early development, so there's lots of room to shape its direction.

## Current State

This repo is currently a **design specification** — documentation and schemas that define how the tool should work. We're looking for contributors to help build the actual implementation.

## How to Contribute

### 1. Pick a Component

Check the roadmap in [SKILL.md](SKILL.md):

- **Core loop mechanism** — The iteration logic that drives autonomous sessions
- **Success criteria wizard** — Convert natural language to structured criteria
- **Context persistence** — Track state between iterations
- **Desktop UI** — User interface (Tauri recommended for lightweight, Electron for speed)
- **Safety controls** — Backup, rate limiting, cost estimation, kill switch

### 2. Discuss First

Before building, open an issue to discuss your approach. This helps avoid duplicate work and ensures alignment with the project direction.

### 3. Implementation Guidelines

- Follow the schemas in `assets/templates/` for data structures
- Use TypeScript for type safety
- Test against use cases in `references/use-cases.md`
- Document your work

### 4. Pull Request Process

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/loop-controller`)
3. Make your changes
4. Test thoroughly
5. Submit PR with clear description

## Architecture Decisions

We're pursuing a **hybrid approach**:

1. **Phase 1**: Standalone wrapper app (ship fast, learn from users)
2. **Phase 2**: Propose native Cowork integration to Anthropic
3. **Phase 3**: Port to native if accepted

See [architecture.md](references/architecture.md) for details.

## Questions?

Open an issue or reach out to the maintainers.
