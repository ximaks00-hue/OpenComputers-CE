# Parity Audit Registry (CE vs Original 1.12)

Source: systematic module-by-module comparison, 2026-06-20.  
Etalon: `Etalon/workspace/OpenComputers-Original/`  
CE base: `dev-MC1.20`

**Confidence:** CONFIRMED = proven by source diff; LIKELY = strong logic evidence; INHERITED = same in both ports.

---

## Fix batch 1 — `fix/p0-parity-audit-batch1`

| ID | Fix | File |
|----|-----|------|
| BUG-034 | `os.time()` uses `getGameTime()` not `getDayTime()` | `Machine.scala` |
| BUG-035 | `getV1elocity` → `getMaxVelocity` | `Drone.scala` |
| BUG-037 | `getObjectFuzzy` Integer-key lookup restored | `RedstoneAware.scala` |
| BUG-038 | Robot inner TE `worldPosition` on NBT load | `Computer.scala` |
| BUG-040 | `flushQueuedKey()` wired in Robot/Screen tick | `Robot.scala`, `Screen.scala` |
| BUG-041 | `keyReleased` null-safe buffer | `InputBuffer.scala` |
| BUG-008 | `compare()` fuzzy + BlockState parity | `ContainerLevelControl.scala` |
| BUG-044 | SSD capacity `1048` → `1024` | `DriverFileSystem.scala` |
| BUG-005 | Remove dead Create `Provider` | `DriverCreativeMotor.scala` |

---

## Open — CONFIRMED (not in batch 1)

| ID | Area | Issue |
|----|------|-------|
| BUG-017 | Agent | `endConsumeDrops` missing capture clear — branch `fix/bug-015-017-*` |
| BUG-011 | Machine | `canInteract` null-server bypass — branch `fix/parity-audit-regressions` |
| BUG-010 | Dev FS | `fromResource()` absolute path — branch `fix/parity-audit-regressions` |
| BUG-018 | Robot client | GUI dispose — branch `fix/bug-018`, LAN deferred |
| BUG-024 | Cable | neighbor notify — branch `fix/bug-024` |
| BUG-027 | Agent | `getEntitiesOfClass` NPE — branch `fix/bug-027` |
| BUG-043 | Client | ScreenRenderer distance fade alpha unused | `ScreenRenderer.scala` |
| BUG-042 | Client | Robot GUI buffer ComponentTracker race | `Robot.scala` |
| BUG-039 | Redstone | Output-enable neighbor notify differs | `RedstoneAware.scala` |
| BUG-045/046 | EventHandler unload/update-check — **see BUG-050/051** |
| BUG-050 | EventHandler world unload non-ticking chunks |
| BUG-051 | EventHandler update-check isOp |
| BUG-052 | SaveHandler dimension null NPE |
| BUG-053 | DriverLootDisk double savePath |
| BUG-054 | cofh energy_info wrong semantics |
| BUG-055 | Thermal wrench no-op |
| BUG-056 | Chunk unload double machine teardown |
| BUG-057 | CC Relay peripheral not registered |
| BUG-003/004 | Integration | AE2/IC2/TIS3D/JEI disabled | `Mods.scala`, `PowerAcceptor` |
| BUG-028 | Integration | ProjectRed bundled output export removed | `BundledRedstoneAware.scala` |
| BUG-047 | ComponentInventory | getCapability order inverted | `ComponentInventory.scala:157` |
| BUG-049 | ProjectRed | useWrench wrong Player type for IMC | `EventHandlerProjectRed.scala:10` |
| BUG-057 | ComputerCraft | Relay peripheral via wrong capability | `PeripheralProvider.scala` |
| BUG-058 | server/network | `newPacket(nbt)` dest load inverted | `Network.scala:567-569` |
| BUG-059 | server/network | `Packet.size` null/none = 4 bytes (was 1) | `Network.scala:709` |
| BUG-060 | transposer traits | `areStacksEquivalent` uses all item tags, not ore tags | `LevelInventoryAnalytics.scala:70-73` |
| BUG-061 | TerminalServer | Buffer max tier `Tier.Four` (190×60, 16-bit) vs Original `Tier.Three` (160×50, 8-bit) | `TerminalServer.scala:45-47` |

---

## Open — LIKELY

| ID | Issue |
|----|-------|
| BUG-036 | Drone `getOffset()` BlockPos vs entity position distance |
| BUG-021 | Machine `canInteract` isOp vs canSendCommands |

---

## INHERITED (both ports)

| Issue | File |
|-------|------|
| `suck()` ignores `mayInteract` result | `ContainerLevelControl.scala:107` |
| `getBundledOutput` array returns `_bundledInput` | `BundledRedstoneAware.scala:95` |

---

## Maintainer / not reproduced

| ID | Notes |
|----|-------|
| BUG-016/009/025 | Network nil/boolean NBT — reverted or not reproduced |
| BUG-006/007 | Immibis microblocks — dropped |

---

## Module coverage

| Module | Status |
|--------|--------|
| server/machine | Audited |
| server/component, agent | Audited |
| server/network | Phase 4 audited (BUG-058,059) |
| server/component/Transposer, MotionSensor | Parity OK (port renames) |
| common/blockentity, traits | Audited |
| client/gui, InputBuffer | Audited + batch1 |
| client/renderer | Audited (BUG-020,043 open) |
| integration | Phase 3 partial |
| common/EventHandler, SaveHandler | Phase 3 audited |
| server/network/WirelessNetwork | Phase 5 parity OK |
| server/component/Agent | Phase 5 audited (BUG-017 on dev) |
| common/component/TerminalServer | Phase 5 audited (BUG-061) |
| server/machine/luaj, luac | Phase 5 OK (worldTime via BUG-034) |
| client/PacketHandler | Phase 5 partial (server-side gaps inherited) |
| common/entity/Drone | Parity OK |
| server/machine/ArgumentsImpl | Parity OK |

---

## Phase 3 — new CONFIRMED findings (2026-06-20)

| ID | Sev | Module | Issue | Original vs CE |
|----|-----|--------|-------|----------------|
| BUG-047 | MED | ComponentInventory | `getCapability`: components before host | Original: host first |
| BUG-048 | MED | ComponentInventory | No `hasCapability` override | Original has both |
| BUG-049 | HIGH | projectred | `useWrench(player: agent.Player)` — IMC expects `net.minecraft.Player` | Registration fails silently |
| BUG-050 | HIGH | EventHandler | World unload skips non-ticking chunks | Original: `loadedTileEntityList` all |
| BUG-051 | LOW | EventHandler | Update check `isOp` only | Original: `!dedicated \|\| canSendCommands` |
| BUG-052 | HIGH | SaveHandler | `ResourceLocation.tryParse` null → NPE in `load()` | Original: int dimension |
| BUG-053 | MED | DriverLootDisk | Double `Settings.savePath` prefix in legacy loot floppies | Original: `"loot/" + tag` only |
| BUG-054 | HIGH | cofh DriverEnergyInfo | `getCurSpeed()` / `getMaxSpeed()` instead of energy/tick API | Original: `IEnergyInfo.getInfoEnergyPerTick` |
| BUG-055 | MED | cofh EventHandlerFoundation | Thermal wrench always `true`, no durability/simulate | Original: `toolUsed` / `isUsable` |
| BUG-056 | MED | EventHandler + BaseBlockEntity | Chunk unload: `dispose()` + `scheduleClose()` double teardown | Original: scheduleClose only |
| BUG-057 | HIGH | computercraft | Relay: custom Forge cap vs `ComputerCraftAPI.registerPeripheralProvider` | CC cannot see Relay modem |

Note: BUG-050/051 overlap Phase 2 BUG-045/046 with line-level proof in EventHandler.

---

## Phase 4 — new CONFIRMED findings (2026-06-20)

| ID | Sev | Module | Issue | Evidence |
|----|-----|--------|-------|----------|
| BUG-058 | HIGH | Network | `newPacket(nbt)` inverts dest: `if (contains("dest")) null else getString` | Save writes dest when targeted; load drops it. **Inherited** in Original 1.12. Breaks switch/hub queue after world reload. |
| BUG-059 | MED | Network | `Packet.size`: `null \| ResultWrapper.unit \| None => 4` vs Original `=> 1` | CE rejects packets sooner; relay/linked-card energy cost inflated for sparse packets. **CE regression**. |
| BUG-043 | MED | ScreenRenderer | Distance fade `alpha` computed but `draw()` never applies it to text | Original used `GL14.glBlendColor`; CE `draw(stack, alpha, …)` ignores `alpha` at lines 369–372. |
| BUG-060 | MED | LevelInventoryAnalytics | `areStacksEquivalent` intersects all `ItemStack.getTags` | Original: `OreDictionary.getOreIDs` only. Broader/narrower matches in 1.20. |

### Phase 4 — reviewed, parity OK

| Module | Notes |
|--------|-------|
| `Transposer.scala` | `WorldInventoryAnalytics` → `LevelInventoryAnalytics`; logic equivalent |
| `MotionSensor.scala` | 1.20 entity/ray API port; same algorithm |
| `Adapter.scala` | Same neighbor side resolution as Original |
| `Hub.scala` | Queue save/load OK; dest bug comes from `Network.newPacket(nbt)` |
| `Machine.convertArg` | CE adds `HashMap` + `CompoundTag` in signals — improvement |
| `ArgumentsImpl` | Intentional OC1 compat TODOs — both ports |

### Phase 4 — INHERITED (same in Original)

| Issue | File |
|-------|------|
| `suck()` calls `mayInteract` but discards result | `ContainerLevelControl.scala:119-121` |
| `getBundledOutput` array returns `_bundledInput` | `BundledRedstoneAware.scala:95` |
| Byte wire format sent as short (FIXME) | `Network.scala:711` — both ports |
| Keyboard key_up on disconnect TODO | `Keyboard.scala:23-24` |

---

## Phase 5 — new CONFIRMED findings (2026-06-20)

| ID | Sev | Module | Issue | Evidence |
|----|-----|--------|-------|----------|
| BUG-061 | MED | TerminalServer | Virtual screen buffer uses **Tier.Four** resolution/color depth | Original `Tier.Three` → CE gives 190×60 @ 16-bit instead of 160×50 @ 8-bit. Remote terminal UI exceeds 1.12 capability. |

### Phase 5 — reconfirmed on `dev-MC1.20` (fix branches exist)

| ID | File | Proof |
|----|------|-------|
| BUG-017 | `Agent.scala:314-327` | `endConsumeDrops` never clears captured drops list after consume (Original `capturedDrops.clear()` line 327) |
| BUG-018 | `Robot.scala:404` | `robotGui.inventoryContainer.otherInventory == this` always false; Original uses `robotGui.robot == this` |
| BUG-011 | `Machine.scala:205` | `getCurrentServer == null` short-circuits to allow interaction — not in Original |
| BUG-021 | `Machine.scala:209` | `config.isOp` vs Original `config.canSendCommands` |

### Phase 5 — reviewed, parity OK

| Module | Notes |
|--------|-------|
| `Agent.scala` swing/use/place | Same algorithm; sneaky via `Pose.CROUCHING` (port of `setSneaking`) |
| `WirelessNetwork.scala` | Dimension key + obstruction logic equivalent |
| `WirelessNetworkCard.scala` | join/update/leave wireless network equivalent |
| `luaj/OSAPI.scala`, `luac/OSAPI.scala` | Both read `machine.worldTime` — fixed by BUG-034 in PR #5 |
| `Terminal.scala` | Client-side range check + GUI; port of 1.12 `GuiType.Terminal` flow |
| `onRobotStateRequest` | No distance check — **inherited** in Original |

---

*CE repo: `docs/PARITY-AUDIT.md` (PR #6) · Test lab mirror: `docs/07-PARITY-AUDIT.md`*
