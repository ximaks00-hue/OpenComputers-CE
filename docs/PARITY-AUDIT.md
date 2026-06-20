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
| BUG-062 | Relay CC | `modem_message` multi-arg payload nested as one element | `Relay.scala:127` — CE regression |
| BUG-063 | Relay CC | Empty payload after `answerPort` → `payload(0)` IndexOutOfBounds | `Relay.scala:127` — CE regression |
| BUG-064 | UpgradeChunkloader | Custom dimensions → `throw new Error("deprecated")` in `isDimensionAllowed` | `UpgradeChunkloader.scala:127-131` — CE regression |
| BUG-065 | DebugCard | `setBlocks` block id from arg 3 not 6 | `DebugCard.scala:890` — CE regression |
| BUG-066 | Geolyzer / Solar / DebugCard | `canSeeSkyFromBelowWater` used instead of `canSeeSky` / `canBlockSeeSky` | `Geolyzer.scala:85`, `UpgradeSolarGenerator.scala:61`, `DebugCard.scala:866` — CE regression |
| BUG-067 | DebugCard | `getDimensionId` throws on modded dimensions | `DebugCard.scala:681-685` — CE regression |
| BUG-069 | Server (rack) | Missing `hasCapability` on mountable Server | `Server.scala:242-251` — CE regression |
| BUG-070 | Transposer/Robot | `compare()` ignores fuzzy flag — `optBoolean(1)` dead code | `ContainerLevelControl.scala:31-32` — CE regression (fix on batch1, not on dev) |
| BUG-020 | Client | FlatScreen skips back-face cull — text visible from behind | `ScreenRenderer.scala:174` — CE regression |

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
| client/PacketHandler | Phase 8 audited — opcode symmetry OK |
| common/entity/Drone | Parity OK |
| server/machine/ArgumentsImpl | Parity OK |
| common/blockentity/Case, Capacitor, PowerDistributor | Phase 8 parity OK |
| server/component/Server (rack) | Phase 8 audited (BUG-069) |
| server/component/DebugCard | Phase 8 audited (BUG-065, BUG-067; BUG-066 sky API) |
| server/component/traits/ContainerLevelControl | Phase 9 audited (BUG-070) |
| integration (AE2/IC2/TIS3D/JEI) | Phase 9 audited — disabled/missing (BUG-003/004) |
| common/EventHandler chunk unload | Phase 9 reconfirmed (BUG-056) |
| client/gui/Robot | Phase 9 reconfirmed (BUG-042) |

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

## Phase 6 — new CONFIRMED findings (2026-06-20)

| ID | Sev | Module | Issue | Evidence |
|----|-----|--------|-------|----------|
| BUG-062 | HIGH | Relay + CC | `RelayCCAdapter.queueMessage` nests multi-arg payload | Original flattens: `Array(Seq(header) ++ args: _*)`. CE: `header :+ payload` when `length > 1` → CC `modem_message` gets 4 args with last arg a table/seq. Breaks multi-part OC→CC relay messages. |
| BUG-063 | MED | Relay + CC | Empty `args` after answerPort strip | `if (payload.length > 1) payload else payload(0)` throws when `payload.isEmpty`. Original: event is `[name, port, answerPort]` only. |
| BUG-064 | HIGH | UpgradeChunkloader | Modded/custom dimensions crash whitelist check | CE maps only OVERWORLD/NETHER/END; `case _ => throw new Error("deprecated")`. Original: `world.provider.getDimension` for any dim. |

### Phase 6 — reviewed, parity OK

| Module | Notes |
|--------|-------|
| `Relay.scala` (core) | Wireless/linked relay, strength, repeater, `tryEnqueuePacket` — same as Original except BUG-062/063 |
| `RelayPeripheral.scala` | CC transmit/open/close — same packet shape as Original |
| `Microcontroller.scala` | Hub/snooper routing, energy pump, plug connect — equivalent (see INHERITED `outputSides` load) |
| `Rack.scala` | Node mapping, relay, mountable power, redstone fan-out — equivalent |
| `Keyboard.scala` (block TE) | Sided node, NBT — equivalent |
| `ControllerImpl.scala` (nanomachines) | Command range, wireless commands — equivalent |
| `NetSplitter.scala` | Invert + network reconnect on redstone — equivalent |
| `ChunkloaderUpgradeHandler.scala` | Ticket restore 3×3 shape — equivalent (Forge API port) |
| `FileSystem` component | Same managed-environment logic as Original |
| `InternetCard` | `checkAddress` / network callbacks — equivalent |

### Phase 6 — INHERITED (both ports)

| Issue | File |
|-------|------|
| `outputSides` saved to NBT but load discards `getBooleanArray` | `Microcontroller.scala:217` |
| Network packet dest load inverted on reload | `Network.scala:567-569` (BUG-058) |

---

## Phase 7 — new CONFIRMED findings (2026-06-20)

| ID | Sev | Module | Issue | Evidence |
|----|-----|--------|-------|----------|
| BUG-066 | HIGH | Geolyzer, UpgradeSolarGenerator, DebugCard | Wrong sky visibility API | Original: `world.canBlockSeeSky(pos)`. CE: `level.canSeeSkyFromBelowWater(pos)` — different semantics (underwater-skylight vs direct sky). Breaks `geolyzer.canSeeSky()`, `isSunVisible()`, solar panel generation, debug card sky query. Fix: `level.canSeeSky(pos)`. |

### Phase 7 — reviewed, parity OK

| Module | Notes |
|--------|-------|
| `DiskDrive.scala` | eject/media/isEmpty, floppy NBT sync — equivalent |
| `Assembler.scala` | template validate, energy assembly tick — equivalent |
| `Disassembler.scala` | queue, energy buffer, recipe disassembly — equivalent |
| `Printer.scala` | 3D print commit/cost/output merge — equivalent |
| `Raid.scala` | 3×HDD merge FS, wipe, label — equivalent |
| `Waypoint.scala` | label, Waypoints registry, particles — equivalent |
| `Charger.scala` | robot/drone/player charge, redstone speed — equivalent (equipment scan uses full `inventory.items` vs Original `mainInventory` only — minor scope change) |
| `Hologram.scala` | voxel volume, fill/setRaw, energy, SaveHandler — equivalent |
| `HologramRenderer.scala` | distance fade alpha **applied** via `setShaderColor` — unlike ScreenRenderer BUG-043 |
| `UpgradeDatabase.scala` | hash/index/copy/clone — equivalent |
| `Geolyzer.scala` (scan/analyze/store) | scan volume, event bus, store via `Block.getDrops` — 1.20 API port (intentional) |
| `EventHandlerVanilla` (geolyzer) | scan/analyze handlers — equivalent port |

### Phase 7 — INHERITED (both ports)

| Issue | File |
|-------|------|
| Hologram `getRenderBoundingBox` max-Z uses `translation.x` instead of `translation.z` | `Hologram.scala:457` (both ports) |

---

## Phase 8 — block infrastructure & DebugCard (2026-06-20)

**Scope:** `Case`, `Capacitor`, `PowerDistributor`, rack `Server`, `MotionSensor`, `PowerBalancer`, `Adapter`, `UpgradeDatabase`, `PacketHandler` client/server opcode map, `RobotMove` packet wire format, `DebugCard.WorldValue` callbacks.

**Method:** Line-by-line diff CE `dev-MC1.20` vs Etalon Original 1.12.2. No in-game verification in this phase — confidence tag **CONFIRMED** only where source proof is unambiguous.

---

### BUG-065 (HIGH) — DebugCard `setBlocks` wrong argument index

| | |
|---|---|
| **File** | `server/component/DebugCard.scala` — `WorldValue.setBlocks` |
| **CE (broken)** | Line 890: `args.checkString(3)` — reads **xMax** (corner coordinate) as block id |
| **Original (correct)** | Line 800: `args.checkString(6)` — block id is the **7th** argument (0-based index 6) |
| **Signature** | `function(x1,y1,z1, x2,y2,z2, id:string, meta:number)` |

**Why this is wrong:** Arguments 0–2 are `(xMin,yMin,zMin)`, 3–5 are `(xMax,yMax,zMax)`. The block resource id must come from index **6**. Using index 3 passes an integer coordinate to `ResourceLocation.tryParse`, which yields `null` or a nonsense block — area fill never places the intended block.

**Impact:** Debug-card `setBlocks` is completely broken on CE. Single-block `setBlock` (index 3 for id) is **unaffected** — only the area variant regressed during 1.20 port.

**Recommended fix:** One-line change: `args.checkString(6)` (and keep `args.checkInteger(7)` for meta).

**Suggested PR:** `fix/bug-065-debugcard-setblocks`

---

### BUG-067 (HIGH) — DebugCard `getDimensionId` crashes on modded dimensions

| | |
|---|---|
| **File** | `server/component/DebugCard.scala` — `WorldValue.getDimensionId` |
| **CE (broken)** | Lines 681–685: match only `OVERWORLD`/`NETHER`/`END`; `case _ => throw new Error("deprecated")` |
| **Original (correct)** | Line 624: `result(world.provider.getDimension)` — numeric id for **any** dimension |

**Why this is wrong:** During MC 1.20 port, vanilla dimension ids were hard-coded (0, -1, 1) as a stopgap. Modded dimensions (Twilight Forest, Aether, custom datapack dims, etc.) hit the default case and **crash the Lua callback** with an uncaught `Error`, not a clean Lua error.

**Same root cause class as BUG-064** (UpgradeChunkloader whitelist). Both need a 1.20-safe numeric dimension key — e.g. registry hash, Forge dimension id mapping, or documented breaking change to `getDimension()` string API (which CE already exposes correctly at line 697).

**Impact:** Any debug-card script calling `getDimensionId()` in a modded dimension terminates the computer.

**Recommended fix:** Return a stable int per dimension (parity with 1.12 behaviour) or deprecate numeric id and document migration to `getDimension()` string — but **do not throw** on unknown dims.

**Suggested PR:** `fix/bug-064-067-dimension-api` (batch with chunkloader)

---

### BUG-069 (MED) — Rack `Server` missing `hasCapability`

| | |
|---|---|
| **File** | `server/component/Server.scala` — `ICapabilityProvider` |
| **CE** | Lines 242–251: only `getCapability`; iterates components, returns first present `LazyOptional` |
| **Original** | Lines 237–244: **`hasCapability`** probes components; **`getCapability`** only after `hasCapability` check |

**Why this matters:** Forge/NeoForge mods often call `hasCapability(cap, side)` before `getCapability`. Without override, `Server` inherits default `false` from the capability helper chain even when an installed component (e.g. energy, fluid, item handler from another mod's card) would answer `true`.

**Impact:** Rack-mounted servers may be invisible to WAILA/JADE probes, cable mods, or automation that checks capabilities on the mountable — even though direct `getCapability` on CE might still work.

**Recommended fix:** Add `hasCapability` mirroring Original's `components.exists { case Some(c: ICapabilityProvider) => c.getCapability(...).isPresent }` pattern (1.20: use `LazyOptional.isPresent`).

**Related:** BUG-047/048 on blockentity `ComponentInventory` — same class of capability probe ordering issue.

**Suggested PR:** `fix/bug-069-server-hasCapability`

---

### Phase 8 — reviewed, parity OK

| Module | What was verified | Notes |
|--------|-------------------|-------|
| `Case.scala` | Tier load/save, creative infinite power, inventory slots, connector sides | CE adds tier-4 case + creative `Tier.Five` — **intentional extension**, not a bug |
| `Capacitor.scala` | Connector nodes, energy buffer, sided connectivity | Equivalent |
| `PowerDistributor.scala` | Six connector nodes, `PowerBalancer` tick, NBT | Equivalent |
| `Server.scala` (core) | Machine lifecycle, rack mount, analyze, power | Equivalent except BUG-069 |
| `MotionSensor.scala` | Entity scan ray algorithm | 1.20 API port, logic same |
| `PowerBalancer.scala` | Multi-connector synchronized buffer distribution | Equivalent (nested locks preserved) |
| `Adapter.scala` | Side open, block driver scan, neighbor notify | Equivalent |
| `UpgradeDatabase.scala` | hash/index/copy/clone callbacks | Equivalent |
| `RobotMove` packet | Server `writeUTF(dimension)` + coords; client `onRobotMove` read order | Correct 1.20 port of int dimension id |
| `PacketHandler` | All `PacketType` enum values have client receive + server receive handlers | Opcode symmetry verified |

---

### Phase 8 — audit notes (not new functional bugs)

| Note | Detail |
|------|--------|
| **BUG-037 not merged to `dev-MC1.20`** | `RedstoneAware.getObjectFuzzy` (lines 40–51) contains dead duplicate branches. Fix exists on `fix/p0-parity-audit-batch1` only. Functional int + double key paths remain — low runtime risk. |
| **BUG-038 load path** | `Computer.loadForServer` sets `robot.setLevel` only; `worldPosition` updated in `RobotProxy.clearRemoved`. Usually OK on chunk load (Robot recreated at saved pos); edge case for moved robots without proxy recreate — fix on batch1 branch. |
| **BUG-039** | `onRedstoneOutputEnabledChanged`: CE `updateNeighborsAt` vs Original `notifyNeighborsOfStateChange(..., true)` — may differ for observers; tracked separately. |
| **Creative tier renumbering** | Original creative case = `Tier.Four`; CE = `Tier.Five` with new regular tier-4 case. Documented as intentional in `ItemUtils` / `Case.isCreative`. |

---

### Phase 8 — recommended fix priority (code audit)

1. **BUG-066** — sky API (3 files, one-line each) — highest player-visible impact  
2. **BUG-065** — DebugCard `setBlocks` (one line)  
3. **BUG-062/063** — Relay CC `queueMessage` flatten (one line)  
4. **BUG-064 + BUG-067** — dimension API batch  
5. **BUG-069** — Server `hasCapability`  
6. Remaining open registry items (Network, integration, Agent, Robot GUI, …)

---

### Phase 9 — pending audit scope

- ~~Integration disabled code paths~~ — **done Phase 9** (see below)  
- ~~`ContainerLevelControl` deep pass~~ — **done** (BUG-070)  
- ~~Client GUI Robot ComponentTracker~~ — **reconfirmed** (BUG-042)  
- ~~`EventHandler` chunk unload double teardown~~ — **reconfirmed** (BUG-056)  
- Merge status: `fix/p0-parity-audit-batch1` vs `dev-MC1.20` — **gap list below**  
- Phase 10: `LinkedCard`, `Switch` block TE, `traits/Keyboard` disconnect, dev FS `fromResource`

---

## Phase 9 — transposer traits, integration, EventHandler, GUI (2026-06-20)

**Scope:** `ContainerLevelControl.compare`, `LevelInventoryAnalytics` (reconfirm BUG-060), `Mods.scala` integration proxies, `PowerAcceptor`, JEI plugin stubs, `EventHandler.onChunkUnloaded`, `client/gui/Robot.scala` buffer source, fix-branch merge gap on `dev-MC1.20`.

**Method:** Source diff only; no in-game verification.

---

### BUG-070 (HIGH) — `compare()` fuzzy flag not implemented on `dev-MC1.20`

| | |
|---|---|
| **File** | `server/component/traits/ContainerLevelControl.scala` |
| **CE (broken)** | Lines 31–32: `args.optBoolean(1, false) // TODO` then `return result(idMatches)` — fuzzy arg **discarded** |
| **Original** | `InventoryWorldControl.scala:31-32`: `subTypeMatches = fuzzy \|\| !hasSubtypes \|\| metadata match`; returns `idMatches && subTypeMatches` |
| **Fix branch** | `fix/p0-parity-audit-batch1` implements `blockStateMatchesStack()` for 1.20 BlockState parity |

**Why wrong:** Robot/transposer `compare(side, fuzzy)` API documents fuzzy matching. CE always compares block type id only — wrong for colored/variant blocks when `fuzzy=false` should require exact state match.

**Impact:** Autonomous building/mining scripts get false positives/negatives vs 1.12.

**Note:** Tracked as BUG-008 on fix branch; **still open on `dev-MC1.20`**.

---

### BUG-020 (MED) — FlatScreen back-face cull skipped — reconfirmed

| | |
|---|---|
| **File** | `client/renderer/tileentity/ScreenRenderer.scala:174-182` |
| **CE** | Back-face test wrapped in `if (!isFlatScreen)` — flat tier screens skip cull |
| **Original** | `ScreenRenderer.scala:57-60` — cull **always** applied (no flat exception) |

**Impact:** Tier-2 flat screens show text when viewed from behind the block.

---

### BUG-056 (MED) — Chunk unload double machine teardown — reconfirmed with CE delta

**Original `onChunkUnload`:** iterates chunk **entity lists** only → `scheduleClose(machine)` for `MachineHost` entities. Tile entities still get `onChunkUnload` → `dispose()` → `Computer.dispose` → `machine.stop()` separately.

**CE `onChunkUnloaded` (lines 461-494):** **additionally** iterates `chunk.getBlockEntities` for `MachineHost` → `scheduleClose`, **plus** AABB entity scan → `scheduleClose` again.

**CE `BaseBlockEntity.onChunkUnloaded`:** calls `dispose()` → for computers `machine.stop()` (same as Original `TileEntity`).

**Double path on CE:** For a Case/RobotProxy in unloading chunk:
1. `BaseBlockEntity.onChunkUnloaded` → `Computer.dispose` → `machine.stop()`
2. `EventHandler.onChunkUnloaded` → `scheduleClose(machine)` → next tick `tryClose()` → `close()` again

**Why it matters:** `stop()` then `tryClose()`/`close()` may duplicate signal teardown, filesystem flush, or node removal — race risk on fast chunk cycles.

**CE regression vs Original:** Extra block-entity `scheduleClose` loop not present in Original 1.12 chunk unload handler.

---

### BUG-042 (MED) — Robot GUI buffer via ComponentTracker — reconfirmed

| | |
|---|---|
| **CE** | `client/gui/Robot.scala:39-43` — `inventoryContainer.info.screenBuffer.flatMap(ComponentTracker.get(...)).orNull` evaluated **once** at GUI init |
| **Original** | `Robot.scala:28-30` — `robot.components.collect { case Some(buffer: TextBuffer) => buffer }.headOption.orNull` — direct reference |

**Why wrong:** If screen component registers in tracker after GUI open, `buffer == null` → 108px layout (`noScreenHeight`) while robot has screen → slot/label misalignment (256px vs 108px).

---

### Phase 9 — integration audit (BUG-003/004 reconfirmed)

| Integration | Original `Mods.Proxies` | CE `dev-MC1.20` | Status |
|-------------|-------------------------|-------------------|--------|
| Applied Energistics 2 | `ModAppEng` active | `//integration.appeng.ModAppEng` commented; **no `integration/appeng/` package** | Disabled — no ME network power/items |
| TIS3D | `ModTIS3D` active | `//integration.tis3d.ModTIS3D` commented; **no `integration/tis3d/` package** | Disabled |
| IC2 power | `PowerAcceptor` traits | `// with power.AppliedEnergistics2` only; IC2 traits absent | Disabled (BUG-003) |
| JEI | `ModJEI` + plugin | All `integration/jei/*.scala` **fully commented** | Disabled (BUG-004) |
| ComputerCraft | `ModComputerCraft` | Active | OK (but Relay registration BUG-057) |
| ProjectRed | `ModProjectRed` | Active | OK (but wrench BUG-049, bundled BUG-028) |

**Conclusion:** Not merely “WIP stubs” — AE2/TIS3D integration **source removed or never ported**; enabling requires re-implementation, not uncommenting one line.

---

### Phase 9 — fix branches not merged to `dev-MC1.20` (reverified)

| ID | File | Still broken on dev |
|----|------|---------------------|
| BUG-008/070 | `ContainerLevelControl.scala:31-32` | fuzzy compare TODO |
| BUG-027 | `LevelAware.scala:87` | `getEntitiesOfClass(..., null)` NPE risk |
| BUG-035 | `Drone.scala:105` | `getV1elocity` typo method name |
| BUG-037 | `RedstoneAware.scala:40-51` | dead duplicate `getObjectFuzzy` branches |
| BUG-038 | `Computer.scala:154` | `setLevel` only, no `worldPosition` on load |
| BUG-034 | `Machine.scala` | `getDayTime` for os.time (if not merged) |

Verify before release: merge `fix/p0-parity-audit-batch1` and open fix PRs #1–#4.

---

### Phase 9 — reviewed, parity OK

| Module | Notes |
|--------|-------|
| `ContainerLevelControl` drop/suck | Same structure as Original `InventoryWorldControl`; `mayInteract` discard **INHERITED** |
| `LevelInventoryAnalytics` | Same as Phase 4 except BUG-060 ore-tag equivalence |
| `Hub` trait | Queue/relay logic equivalent; dest reload still BUG-058 |
| `LevelAware.entitiesInBounds` | Line 74 uses 2-arg `getEntitiesOfClass` — OK; only `closestEntity` line 87 broken |

---

*CE repo: `docs/PARITY-AUDIT.md` (PR #6) · Test lab mirror: `docs/07-PARITY-AUDIT.md`*
