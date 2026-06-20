# OpenComputers-CE パリティ監査報告（Etalon 1.12 対比）

**対象:** `dev-MC1.20` vs Etalon `OpenComputers-Original` (MC 1.12.2)  
**方法:** モジュール単位のソース diff、Original との行単位比較  
**日付:** 2026-06-20  
**Fork PR:** https://github.com/ximaks00-hue/OpenComputers-CE/pull/5 （P0 修正バッチ）

---

## 概要

Etalon（1.12 Original）と CE（1.20 ポート）を比較し、**動作が意図的に異なるのではなく、ポート漏れ・ typo・ API 誤用**と判断できる問題を列挙します。各項目に **Original の正しい挙動**、**CE 修正前**、**CE 修正後**（該当する場合）を記載します。

---

## A. 修正済み — PR #5 `fix/p0-parity-audit-batch1`

### BUG-034 — `os.time()` が日時刻ではなくワールド経過時間を返すべき

**ファイル:** `server/machine/Machine.scala`

**問題:** CE は `getDayTime()`（0〜24000、日ごとにリセット）を `worldTime` に保存。Original は `getWorldTime()`（単調増加のワールド tick）。

**影響:** Lua `os.time()` が Minecraft の「日の時刻」になり、日が変わると値がリセットされる。

| | コード |
|---|--------|
| **Original (1.12)** | `worldTime = host.world.getWorldTime` |
| **CE 修正前** | `worldTime = host.getEnvironmentLevel.getDayTime` |
| **CE 修正後** | `worldTime = host.getEnvironmentLevel.getGameTime` |

---

### BUG-008 — `robot.compare()` / `drone.compare()` が fuzzy と BlockState を無視

**ファイル:** `server/component/traits/ContainerLevelControl.scala`

**問題:** CE は第2引数 `fuzzy` を読むが **使用せず**、ブロック ID のみ比較。Original は subtype（メタデータ / BlockState）も検証。

**影響:** 色違いウール、向き付きブロック等で誤って `true` を返す。

| | コード |
|---|--------|
| **Original (1.12)** | `subTypeMatches = fuzzy \|\| !item.getHasSubtypes \|\| metadata == getMetaFromState(state)` |
| **CE 修正前** | `args.optBoolean(1, false) // TODO` → `return result(idMatches)` のみ |
| **CE 修正後** | `val fuzzy = args.optBoolean(1, false)` + `blockStateMatchesStack(...)` で `getStateForPlacement` と world state を比較 |

```scala
// CE 修正後（抜粋）
val subTypeMatches = fuzzy || blockStateMatchesStack(item, stack, state, blockPos, side)
return result(idMatches && subTypeMatches)
```

---

### BUG-035 — ドローン API 名 typo: `getV1elocity`（数字の1）

**ファイル:** `server/component/Drone.scala`

**問題:** `@Callback` メソッド名が `getV1elocity` になり、Lua から `component.getMaxVelocity()` が呼べない。

| | コード |
|---|--------|
| **Original** | `def getMaxVelocity(...)` |
| **CE 修正前** | `def getV1elocity(...)` |
| **CE 修正後** | `def getMaxVelocity(...)` |

---

### BUG-037 — Lua レッドストーン map の Integer キー lookup 欠落

**ファイル:** `common/blockentity/traits/RedstoneAware.scala` — `getObjectFuzzy`

**問題:** 1.12→1.20 移植時、jnlua が `Integer` オブジェクトキーで map を渡す経路が **重複した dead branch** に置き換わり、`Integer.valueOf(key)` 検索が消えた。

**影響:** `redstone.setOutput({[1]=15, ...})` 等が **無音で失敗**。

| | コード |
|---|--------|
| **Original** | `else if (refMap.containsKey(new Integer(key)))` |
| **CE 修正前** | `else if (refMap.containsKey(key))` が **2回重複**（Integer 未対応） |
| **CE 修正後** | `Integer.valueOf(key)` および `Integer.valueOf(key) * 1.0` 分支を復元 |

---

### BUG-038 — ロボット NBT ロード時、内部 Robot TE の座標未設定

**ファイル:** `common/blockentity/traits/Computer.scala` — `loadForServer`

**問題:** `RobotProxy` ロード時、内部 `robot` に `setLevel` のみ。Original は `setPos(getPos)`。CE では `clearRemoved` 前に **SaveHandler が座標 0,0,0 付近**で kernel 補助ファイルを探す可能性。

**影響:** Save&Quit 後のロボット Lua 状態（auxiliary save）が壊れる / 読めない。

| | コード |
|---|--------|
| **Original** | `proxy.robot.setPos(getPos)` |
| **CE 修正前** | `proxy.robot.setLevel(getLevel)` のみ |
| **CE 修正後** | `setLevel` + `proxy.robot.worldPosition = getBlockPos` |

---

### BUG-040 / BUG-041 — InputBuffer: キー入力タイミングと NPE

**ファイル:** `client/gui/traits/InputBuffer.scala`, `Robot.scala`, `Screen.scala`

**問題 (040):** CE は GLFW 移植で `pushQueuedKey` → `flushQueuedKey` を **tick 待ち**に変更したが、`containerTick()` / `tick()` から **一度も呼ばれていなかった**。Original は `handleKeyboardInput` で即 `keyDown`。

**影響:** `keyDown` が **keyReleased 時**にしか送られない。ターミナル入力・ゲームのキー repeat が 1.12 と不一致。

| | コード |
|---|--------|
| **CE 修正前 (Robot)** | `override def containerTick() = { super.containerTick() }` |
| **CE 修正後 (Robot)** | `flushQueuedKey(); super.containerTick()` |
| **CE 修正後 (Screen)** | `override def tick() = { flushQueuedKey(); super.tick() }` |

**問題 (041):** `keyReleased` で `buffer.keyUp` 呼び出し時、`buffer == null` チェックなし（`keyPressed` にはある）。

| | コード |
|---|--------|
| **CE 修正前** | `case Some(char) => buffer.keyUp(...)` |
| **CE 修正後** | `case Some(char) if buffer != null => buffer.keyUp(...)` |

---

### BUG-044 — SSD 容量計算 typo `1048`

**ファイル:** `integration/opencomputers/DriverFileSystem.scala`

**問題:** HDD は `* 1024`、SSD だけ `* 1048`（約 2.3% 過大）。

| | コード |
|---|--------|
| **CE 修正前** | `ssd.kiloBytes * 1048` |
| **CE 修正後** | `ssd.kiloBytes * 1024` |

---

### BUG-005 — Create 統合の dead code `Provider`

**ファイル:** `integration/create/DriverCreativeMotor.scala`

**問題:** `Blocks.FURNACE` を返す未使用 `EnvironmentProvider` が残存（ModCreate から未登録）。誤ったポート残骸。

| | コード |
|---|--------|
| **CE 修正前** | `object Provider extends EnvironmentProvider { ... Blocks.FURNACE ... }` |
| **CE 修正後** | オブジェクトごと削除 |

---

## B. 修正済み（別 PR、dev 未マージ）

### BUG-015 / BUG-019 — チャンクローダー

**ファイル:** `UpgradeChunkloader.scala`, `ChunkloaderUpgradeHandler.scala`

| 問題 | CE 修正前 | 修正内容 |
|------|-----------|----------|
| チケット座標 | `new ChunkPos(0, 0)` プレースホルダ | ロボット実座標の ticket |
| modded 次元 | `throw new Error("deprecated")` | ホワイトリスト方式に復帰 |

**PR:** #1 `fix/bug-015-017-chunkloader-agent-drops`

---

### BUG-017 — エージェント drop 重複

**ファイル:** `server/component/Agent.scala` — `endConsumeDrops`

| | |
|---|---|
| **Original** | `entity.capturedDrops.clear()` |
| **CE 修正前** | `captureDrops(null)` 後に clear なし |
| **CE 修正後** | `captured.clear()` 追加 |

**PR:** #1

---

### BUG-027 — `getEntitiesOfClass(..., null)` NPE (MC 1.20)

**ファイル:** `LevelAware.scala`, `agent/Player.scala`, `DebugCard.scala`

| | |
|---|---|
| **Original** | `findNearestEntityWithinAABB`（Predicate なし） |
| **CE 修正前** | `getEntitiesOfClass(clazz, bounds, null)` → **NPE** |
| **CE 修正後** | 2 引数 overload + null-safe helper |

**PR:** #4

---

### BUG-024 — ケーブル `neighborChanged`

**ファイル:** `common/block/Cable.scala`

| | |
|---|---|
| **Original** | 常に `notifyBlockUpdate` + `super.neighborChanged` |
| **CE 修正前** | state 変更時のみ `setBlock`、`super` なし |
| **CE 修正後** | 未変更時 `sendBlockUpdated` + `super.neighborChanged` |

**PR:** #3

---

### BUG-018 — ロボット GUI ゴースト

**ファイル:** `blockentity/Robot.scala`, `client/gui/Robot.scala`

| | |
|---|---|
| **Original** | `robotGui.robot == this` → GUI 閉じる |
| **CE 修正前** | `inventoryContainer.otherInventory == this`（常に false） |
| **CE 修正後** | `robotGui.robot == this` |

**PR:** #2（LAN 2人テスト待ち）

---

### BUG-001, 005, 010, 011, 013 — PR #39 系

| ID | 内容 |
|----|------|
| BUG-001 | ESC ポーズ中 Lua 停止 |
| BUG-010 | dev classpath `fromResource` 相対パス |
| BUG-011 | `canInteract` の `getCurrentServer == null` bypass 削除 |
| BUG-013 | パーティクル packet count byte 消費 |

**PR:** `fix/parity-audit-regressions` → upstream #39

---

## C. 未修正 — CONFIRMED（コード上確実）

### BUG-043 — ScreenRenderer 距離フェード未適用

**ファイル:** `client/renderer/tileentity/ScreenRenderer.scala`

`alpha` を計算して `draw(stack, alpha, buffer)` に渡すが、`draw()` 内で **テキスト描画に alpha を使っていない**。Original は `GL14.glBlendColor` でフェード。

---

### BUG-042 — Robot GUI buffer の ComponentTracker レース

**ファイル:** `client/gui/Robot.scala`

GUI 構築時に **一度だけ** `ComponentTracker.get(...).orNull`。Original は `robot.components` を直接参照。tracker 未登録時、画面なしレイアウト（108px）とスロット位置（256px）が不一致。

---

### BUG-039 — レッドストーン output 有効化時の neighbor 通知

**ファイル:** `RedstoneAware.scala` — `onRedstoneOutputEnabledChanged`

| Original | CE |
|----------|-----|
| `notifyNeighborsOfStateChange(..., true)` | `updateNeighborsAt` のみ |

オブザーバー / コンパレータの挙動差の可能性。

---

### BUG-045 — ワールド unload 時 TE dispose 不完全

**ファイル:** `EventHandler.scala` ~438行

CE は `chunkMap` + `getTickingChunk != null` のみ。Original は `loadedTileEntityList` 全件。non-ticking chunk の TE が dispose されない可能性。

---

### BUG-046 — バージョン通知の権限チェック

**ファイル:** `EventHandler.scala` ~253行

| Original | CE |
|----------|-----|
| `!dedicated \|\| canSendCommands` | `isOp` のみ |

---

### BUG-003 / BUG-004 — 統合・電源

- `PowerAcceptor`: IC2 / AE2 traits コメントアウト
- `Mods.scala`: `ModAppEng`, `ModTIS3D` 等コメントアウト
- JEI プラグイン全体コメントアウト

意図的 WIP の可能性あり。機能欠落として記録。

---

### BUG-028 — ProjectRed bundled **出力** 未エクスポート

**ファイル:** `BundledRedstoneAware.scala`

Original: `IBundledTile.getBundledSignal`, Charset capability。  
CE: import コメントアウト、NBT 保存のみ。**読取は可、外部ケーブルへの出力不可**。

---

### BUG-020 — FlatScreen back-face cull スキップ

**ファイル:** `ScreenRenderer.scala:174`

`if (!isFlatScreen)` のみ cull。FlatScreen 背面からテキストが見える可能性。

---

## D. LIKELY / INHERITED

| ID | 内容 |
|----|------|
| BUG-036 | Drone `getOffset()` — BlockPos vs entity 座標 |
| BUG-021 | Machine `canInteract` — `isOp` vs `canSendCommands` |
| INHERITED | `suck()` が `mayInteract` 結果を無視（1.12 も同様） |
| INHERITED | `getBundledOutput: Array = _bundledInput`（1.12 も同様 typo） |

---

## E. Maintainer 判断 / 未再現

| ID | 内容 |
|----|------|
| BUG-006/007 | Immibis マイクロブロック — 削除 |
| BUG-009/012/014/025 | Network NBT — revert |
| BUG-016 | nil part size 4 vs 1 — in-game 未再現 |

---

## H. Phase 3 — 新規 CONFIRMED（2026-06-20、未修正）

| ID | 重要度 | モジュール | 問題 |
|----|--------|------------|------|
| BUG-047/048 | MED | ComponentInventory | `getCapability` が components→host の順（Original は逆）。`hasCapability` 未 override |
| BUG-049 | HIGH | ProjectRed | `useWrench(player: agent.Player)` — IMC は `net.minecraft.Player` を要求 → 登録失敗 |
| BUG-050 | HIGH | EventHandler | ワールド unload が non-ticking chunk の TE をスキップ |
| BUG-051 | LOW | EventHandler | 更新通知が `isOp` のみ（Original: `!dedicated \|\| canSendCommands`） |
| BUG-052 | HIGH | SaveHandler | `ResourceLocation.tryParse` が null → `dimension.toString` で NPE |
| BUG-053 | MED | DriverLootDisk | legacy loot ディスクで `savePath` 二重 prefix |
| BUG-054 | HIGH | cofh DriverEnergyInfo | `getCurSpeed()` / `getMaxSpeed()` を energy/tick として返す |
| BUG-055 | MED | Thermal wrench | `EventHandlerFoundation` が常に `true`（耐久/simulate なし） |
| BUG-056 | MED | EventHandler + BaseBlockEntity | chunk unload で `dispose()` + `scheduleClose()` 二重 teardown |
| BUG-057 | HIGH | ComputerCraft | Relay が Forge capability のみ（`registerPeripheralProvider` なし） |

---

## I. Phase 4 — 新規 CONFIRMED（2026-06-20、未修正）

| ID | 重要度 | モジュール | 問題 |
|----|--------|------------|------|
| BUG-058 | HIGH | Network | `newPacket(nbt)` の dest 読込が **反転**（`contains("dest")` → null）。Hub/Switch キューが reload 後に broadcast 化。**1.12 も同様（INHERITED）** |
| BUG-059 | MED | Network | `Packet.size` で null/none が 4 バイト（Original は 1）。CE regression — パケット拒否・relay コスト増 |
| BUG-043 | MED | ScreenRenderer | 距離 fade の `alpha` を `draw()` が未使用（Phase 3 で記載、Phase 4 で行番号確認） |
| BUG-060 | MED | LevelInventoryAnalytics | `areStacksEquivalent` が全 item tag 交差（Original は OreDictionary のみ） |

**Phase 4 parity OK:** Transposer, MotionSensor, Adapter, Hub（キュー自体は OK、dest は Network 側）、Machine.convertArg（HashMap 追加は改善）

---

## J. Phase 5 — 新規 CONFIRMED / 再確認（2026-06-20）

| ID | 重要度 | 問題 |
|----|--------|------|
| BUG-061 | MED | TerminalServer 仮想 buffer: CE `Tier.Four` (190×60, 16bit) vs Original `Tier.Three` (160×50, 8bit) |

**dev-MC1.20 上で再確認（fix 分支あり）:** BUG-017 Agent drops, BUG-018 Robot GUI, BUG-011/021 canInteract

**parity OK:** Agent swing/use/place ロジック, WirelessNetwork, luaj/luac OSAPI（BUG-034 で修正）

---

## K. Phase 6 — 新規 CONFIRMED（2026-06-20）

| ID | 重要度 | モジュール | 問題 |
|----|--------|------------|------|
| BUG-062 | HIGH | Relay + CC | `modem_message` の複数 payload が 1 要素にネスト（Original は flatten） |
| BUG-063 | MED | Relay + CC | payload 空で `payload(0)` → IndexOutOfBounds |
| BUG-064 | HIGH | UpgradeChunkloader | カスタム次元で `throw new Error("deprecated")`（Original は任意 dimension ID） |
| BUG-066 | HIGH | Geolyzer / Solar / DebugCard | `canSeeSkyFromBelowWater` を使用（Original は `canBlockSeeSky`）— 天空判定が誤る |

---

## L. Phase 7 — 新規 CONFIRMED（2026-06-20）

| ID | 重要度 | モジュール | 問題 |
|----|--------|------------|------|
| BUG-066 | HIGH | Geolyzer / Solar / DebugCard | `canSeeSkyFromBelowWater` を使用（Original は `canBlockSeeSky`） |

### BUG-066 — 天空 visibility API の誤り

**ファイル:** `Geolyzer.scala:85`, `UpgradeSolarGenerator.scala:61`, `DebugCard.scala:866`

| | |
|---|---|
| **Original** | `world.canBlockSeeSky(pos)` — ブロック位置から直接天空が見えるか |
| **CE** | `level.canSeeSkyFromBelowWater(pos)` — **水中から上方の光**を判定する別 API |

**なぜ誤りか:** 1.20 ポート時にメソッド名の類似で誤選択。水中・半透明ブロック下での挙動が Original と異なる。

**影響:** `geolyzer` の天空判定、ソーラー発電、`debug.card` の `canSeeSky` が誤った結果を返す。

**推奨修正:** `level.canSeeSky(pos)` に統一（3 ファイル各 1 行）。

**parity OK:** DiskDrive, Assembler, Disassembler, Printer, Raid, Waypoint, Charger, Hologram (+ HologramRenderer は fade 適用済み), UpgradeDatabase, Geolyzer scan/store

---

## M. Phase 8 — ブロック基盤 & DebugCard（2026-06-20）

**監査範囲:** Case, Capacitor, PowerDistributor, rack Server, MotionSensor, PowerBalancer, Adapter, UpgradeDatabase, PacketHandler opcode 対称, RobotMove パケット, DebugCard.WorldValue

**方法:** CE `dev-MC1.20` と Etalon Original 1.12.2 のソース diff。in-game 検証なし — **CONFIRMED** はソース上確実なもののみ。

---

### BUG-065 (HIGH) — DebugCard `setBlocks` 引数インデックス誤り

**ファイル:** `server/component/DebugCard.scala` — `WorldValue.setBlocks`

| | |
|---|---|
| **CE（誤）** | 890 行: `args.checkString(3)` — **xMax**（座標）を block id として読む |
| **Original（正）** | 800 行: `args.checkString(6)` — 7 番目の引数が block id |
| **シグネチャ** | `function(x1,y1,z1, x2,y2,z2, id:string, meta:number)` |

**なぜ誤りか:** 引数 0–5 は両コーナー座標。block id は index **6**。index 3 は整数座標のため `ResourceLocation.tryParse` が失敗または無意味なブロックになる。

**影響:** エリア一括 `setBlocks` が CE で完全に動作しない。単一 `setBlock`（index 3 = id）は **影響なし**。

**推奨修正:** `args.checkString(6)` に 1 行変更。

---

### BUG-067 (HIGH) — DebugCard `getDimensionId` が modded 次元でクラッシュ

**ファイル:** `DebugCard.scala` — `WorldValue.getDimensionId`（681–685 行）

| | |
|---|---|
| **CE** | OVERWORLD/NETHER/END のみ; `case _ => throw new Error("deprecated")` |
| **Original** | `world.provider.getDimension` — 任意次元の数値 ID |

**なぜ誤りか:** 1.20 ポート時の暫定 hard-code。modded 次元で Lua コールバックが未捕捉 `Error` でコンピュータ停止。

**BUG-064（Chunkloader）と同根。** 1.20 向け安定 int キーが必要。

**推奨修正:** 数値 ID を返すか、`getDimension()` 文字列 API への移行を文書化 — **未知次元で throw しない**。

---

### BUG-069 (MED) — rack `Server` に `hasCapability` なし

**ファイル:** `server/component/Server.scala`（242–251 行）

| | |
|---|---|
| **CE** | `getCapability` のみ |
| **Original** | `hasCapability` + `getCapability`（237–244 行） |

**なぜ問題か:** 多くの mod は `hasCapability` を先に呼ぶ。未 override だと mountable Server が false を返し、コンポーネント capability があっても検出されない。

**関連:** BUG-047/048（blockentity ComponentInventory）と同種。

**推奨修正:** Original と同様の `hasCapability` override を追加。

---

### Phase 8 — parity OK

Case（tier-4 + creative Tier.Five は意図的拡張）, Capacitor, PowerDistributor, Server コア, MotionSensor, PowerBalancer, Adapter, UpgradeDatabase, RobotMove パケット, PacketHandler opcode 対称

### Phase 8 — 監査メモ

- **BUG-037** は `dev-MC1.20` 未マージ（`getObjectFuzzy` dead code）
- **BUG-038** Robot `worldPosition` — `clearRemoved` で通常は OK; batch1 分支に fix
- **BUG-039** redstone output-enable neighbor 通知差

### Phase 8 — 推奨 fix 優先度

1. BUG-066（天空 API）→ 2. BUG-065 → 3. BUG-062/063 → 4. BUG-064/067 → 5. BUG-069

### Phase 9 予定

AE2/IC2/TIS3D 無効コード, ContainerLevelControl, Robot GUI BUG-042, EventHandler BUG-056, batch1 マージ状況

**INHERITED:** Microcontroller `outputSides` NBT 未復元; Hologram bounding box max-Z typo

---

## F. PR 一覧（fork: ximaks00-hue）

| PR | ブランチ | 内容 | 状態 |
|----|----------|------|------|
| #1 | fix/bug-015-017-* | チャンクローダー、agent drops | in-game PASS（一部） |
| #2 | fix/bug-018-* | Robot GUI dispose | LAN 検証待ち |
| #3 | fix/bug-024-* | Cable neighbor | コードのみ |
| #4 | fix/bug-027-* | Entity lookup NPE | コードのみ |
| #5 | fix/p0-parity-audit-batch1 | P0 修正 10 ファイル | コードのみ |
| #6 | docs/parity-audit-registry | 本監査レポート（docs/） | ドキュメントのみ |

---

## G. 検証状況

- **コード監査:** Phase 1–8 完了（Phase 8: Case/Capacitor/PowerDistributor/Server/Adapter/PacketHandler/DebugCard）
- **in-game 検証:** #1 一部 PASS。P0 batch (#5) および Phase 3–8 項目は **未検証**
- **推奨 fix 優先:** BUG-066 → BUG-065 → BUG-062/063 → BUG-064/067 → BUG-069 → BUG-058 → BUG-049/057/054
- **推奨テスト順:** os.time → compare → redstone map → robot reload → InputBuffer → 既存 PR 群

---

*English registry: `docs/PARITY-AUDIT.md`（CE リポジトリ） / テストラボ: `docs/07-PARITY-AUDIT.md`*
