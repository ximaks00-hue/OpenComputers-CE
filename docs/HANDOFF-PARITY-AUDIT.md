# Parity Audit — Handoff Document

Transfer guide for continuing the OpenComputers-CE vs Original 1.12.2 parity audit in a new chat or by another maintainer.

**Last updated:** 2026-06-20  
**Branch:** `docs/parity-audit-registry`  
**Docs PR:** https://github.com/ximaks00-hue/OpenComputers-CE/pull/6

---

## 1. Goal

Systematic **static source audit** of **OpenComputers-CE** (MC 1.20) vs **OpenComputers Original** (MC 1.12.2).

- Find **real bugs** with line-level proof (CE file:lines vs Original file:lines).
- Record in audit registry; **do not fix on `dev-MC1.20`** during audit phase.
- Fixes go to **separate PR branches** (#1–#5 on fork).
- **In-game tests deferred** until explicitly requested.

---

## 2. Paths

| Resource | Path |
|----------|------|
| CE (audited) | `workspace/OpenComputers-CE/` |
| Etalon Original | `Etalon/workspace/OpenComputers-Original/` |
| Registry EN | `workspace/OpenComputers-CE/docs/PARITY-AUDIT.md` |
| Registry JA | `workspace/OpenComputers-CE/docs/PARITY-AUDIT-JA.md` |
| Test lab mirror | `docs/07-PARITY-AUDIT.md` (test lab repo root) |
| Fork | `ximaks00-hue/OpenComputers-CE` |
| Upstream | `akki697222/OpenComputers-CE` |

**Compare base:** `dev-MC1.20`  
**Docs branch:** `docs/parity-audit-registry`

---

## 3. Methodology

1. Pick **phase scope** (module list).
2. Read CE + Original sources; compare **semantics**, not just API renames.
3. Classify: **CONFIRMED** / **INHERITED** / **LIKELY** / parity OK.
4. Assign **BUG-NNN** (next free: **082+**).
5. Update `PARITY-AUDIT.md`, `PARITY-AUDIT-JA.md`, sync `07-PARITY-AUDIT.md`.
6. Commit on `docs/parity-audit-registry`; push PR #6.

---

## 4. Phases completed (1–14)

| Phase | Scope | Key CONFIRMED |
|-------|--------|---------------|
| 1–2 | Machine, Agent, GUI, cables | BUG-017, 018, 024, 027, 042, 043 |
| 3 | EventHandler, SaveHandler, integration | BUG-047–057, 050–052 |
| 4 | Network, transposer, ScreenRenderer | BUG-058, 059, 060 |
| 5 | Wireless, TerminalServer, Lua | BUG-061 |
| 6 | Relay CC, Chunkloader | BUG-062, 063, 064 |
| 7–8 | DebugCard, Server rack | BUG-065, 066, 067, 069 |
| 9 | compare, integration, EventHandler, Robot GUI | BUG-070 |
| 10 | LinkedCard, Hub, Keyboard, fromResource | BUG-010 |
| 11 | CC DriverPeripheral, DiskDrive, Rack | BUG-071, 072 |
| 12 | Upgrades, Trade, Loot | BUG-073–076 |
| 13 | Agent/Player, InternetCard, Sound, blockentity | BUG-077–081, BUG-021 |
| 14 | Manual, upgrades, Geolyzer, projectred | BUG-082, 083 |

**Open CONFIRMED count:** ~57 (BUG-003 … BUG-083) on `dev-MC1.20`.

---

## 5. Fix PR map

| PR | Branch | Content |
|----|--------|---------|
| #1 | `fix/bug-015-017-*` | Chunkloader, agent drops |
| #2 | `fix/bug-018-*` | Robot GUI dispose |
| #3 | `fix/bug-024-*` | Cable neighbor notify |
| #4 | `fix/bug-027-*` | Entity lookup NPE |
| #5 | `fix/p0-parity-audit-batch1` | P0 batch |
| #6 | `docs/parity-audit-registry` | Documentation |

Also: `fix/parity-audit-regressions` — BUG-010, BUG-011.

**Most fix branches not merged to `dev-MC1.20`.**

---

## 6. Fix priority (recommended)

1. BUG-066 → BUG-065 → BUG-062/063  
2. BUG-077 (yaw/pitch) + BUG-073/074  
3. BUG-071 → BUG-064/067/075  
4. BUG-069, BUG-010/076, BUG-078/079/080/081  
5. Merge PR #5 + #1–#4  

---

## 7. Phase 15 — continue here

| Module | Notes |
|--------|-------|
| `server/machine/Machine.scala` | close/stop/sleep lifecycle beyond BUG-011/021/034 |
| Print / 3D printer | Not deeply audited |
| Remaining `server/component/*` | Printer, MotionSensor edge cases |
| `integration/cofh/*` | Reverify BUG-054/055 |

New findings → **BUG-084+**.

---

## 8. Prompt for new chat

```
Parity audit OpenComputers-CE vs Original 1.12.
CE: workspace/OpenComputers-CE/
Etalon: Etalon/workspace/OpenComputers-Original/
Registry: docs/PARITY-AUDIT.md, PARITY-AUDIT-JA.md, HANDOFF-PARITY-AUDIT.md
Branch: docs/parity-audit-registry (PR #6)
Base: dev-MC1.20

STATIC diff only. No in-game tests. No fixes on dev.
Phases 1–14 done. Open BUG-003…083.
Continue Phase 15 per HANDOFF §7.
```

---

## 9. Test lab (future)

Under `d:\Testing OC\`: runbooks, PAR scenarios, GameTests, release gate — for post-audit verification only.
