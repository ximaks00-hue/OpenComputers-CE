# OpenComputers-CE — Parity Audit Documentation

Systematic comparison of **OpenComputers-CE** (`dev-MC1.20`) against the **Original 1.12.2** reference implementation.

| Document | Audience | Contents |
|----------|----------|----------|
| [PARITY-AUDIT.md](./PARITY-AUDIT.md) | Developers | English bug registry (BUG-001…060), module coverage, PR mapping |
| [PARITY-AUDIT-JA.md](./PARITY-AUDIT-JA.md) | Maintainers (@akki697222) | Full Japanese report with before/after code evidence |

## Methodology

1. Module-by-module source diff vs Etalon Original (`OpenComputers-Original` MC 1.12).
2. Each finding tagged **CONFIRMED** (line-level proof), **LIKELY**, or **INHERITED** (same in both ports).
3. Fixes split into **small reviewable PRs** on fork `ximaks00-hue/OpenComputers-CE`.
4. In-game verification tracked separately per PR (code audit precedes gameplay tests).

## Related PRs (fork)

| PR | Branch | Type |
|----|--------|------|
| [#1](https://github.com/ximaks00-hue/OpenComputers-CE/pull/1) | `fix/bug-015-017-chunkloader-agent-drops` | Fix |
| [#2](https://github.com/ximaks00-hue/OpenComputers-CE/pull/2) | `fix/bug-018-robot-gui-dispose` | Fix |
| [#3](https://github.com/ximaks00-hue/OpenComputers-CE/pull/3) | `fix/bug-024-cable-neighbor-notify` | Fix |
| [#4](https://github.com/ximaks00-hue/OpenComputers-CE/pull/4) | `fix/bug-027-entity-lookup-npe` | Fix |
| [#5](https://github.com/ximaks00-hue/OpenComputers-CE/pull/5) | `fix/p0-parity-audit-batch1` | Fix (P0 batch) |
| [#6](https://github.com/ximaks00-hue/OpenComputers-CE/pull/6) | `docs/parity-audit-registry` | Docs (this branch) |

*Last updated: 2026-06-20 — Phases 1–4 code audit complete.*
