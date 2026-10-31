# `src/server` — 1.13c 权威游戏模拟引擎 (`D2Game.dll`) / 1.13c Authoritative Game Simulation Engine (`D2Game.dll`)

**[简体中文](#简体中文) | [English](#english)**

---

<a id="简体中文"></a>

## 简体中文

> [!WARNING]
> **服务端开发状态说明（Work in Progress）**：
> `src/server` 模块**目前尚未开发完成（Work in Progress / Not Completed Yet）**。虽然基础的 25fps 模拟循环、战斗公式、部分技能执行器、掉落管线与 `.d2s` 存档读写框架已搭建并通过单元测试，但尚未与前端完整串联为可独立游玩的本地/单机全流程服务端。
> **当前进行完整游戏体验，请按照主文档（[根目录 `README.md`](../../README.md)）使用单客户端模式（Single-Client Mode），通过 WebSocket 桥连接原版 C++ PvPGN + D2GS (v1.13c) 服务器。**

`src/server` 是 **Diablo II: Lord of Destruction (v1.13c)** 的确定性服务端权威逻辑模块，设计目标是 1:1 复刻原版 `D2Game.dll` 的 25fps (40ms/tick) 帧循环、战斗管线、221 个技能、怪物 AI、掉落表（TreasureClass）与存档系统。

### 1. 模块职责与架构边界

- **对应原版 DLL**：`D2Game.dll`。
- **TypeScript 工程约束（`tsconfig.server.json`）**：
  - `lib: ["ES2022", "WebWorker"]`, `types: []` —— **严禁使用任何浏览器 DOM API 或 Node.js API**（由 `tests/arch/boundaries.test.ts` Rules 3 & 4 强制校验）。
  - **纯确定性模拟（Deterministic Simulation）**：模块内严禁调用 `Math.random`、`Date.now` 或 `performance.now`；所有随机性必须由种子的 `D2Rng` 驱动，所有时间推进必须由 25fps 离散帧号（`tickCount`）驱动。
  - 仅允许依赖 `src/common` 与 `src/netproto`（门面入口 `src/netproto/index.ts`），严禁导入 `src/client` 或 `src/baker`。

---

### 2. 目录结构

```text
src/server/
├── index.ts             # 模块导出入口 (GameInstance, LocalLoopbackAdapter, LocalLobbyProvider, MultiClientRoomHost, .d2s 读写器)
├── engine.ts            # 顶层模拟引擎导出门面
├── loopback-adapter.ts  # LocalLoopbackAdapter & LocalLobbyProvider：将单机 GameInstance 封装为标准 GameServerAdapter 与 LobbyProvider 契约接口
├── room-host.ts         # MultiClientRoomHost：多客户端 (1..8 玩家) 25Hz 权威房间宿主，支持 TsFramer 与二进制 D2GS 双协议接入
├── net/                 # 多人网络同步层 (aoi.ts 视野过滤、party.ts 组队/敌对关系管理、session.ts 客户端帧会话、trade-manager-113c.ts 城镇玩家面对面安全交易状态机与 10x4 原子提交/回滚)
├── engine/              # 25fps 核心模拟引擎 (game-instance.ts, game-engine.ts, combat-pipeline, missile-engine, aura-scanner, stat-list, state-hash)
├── ai/                  # 怪物 AI 决策树、仇恨/索敌、视野碰撞与动态房间流式加载 (room-streamer)
├── skills/              # 七大职业共 221 个技能与通用技能服务端执行器 (Amazon, Sorceress, Necromancer, Paladin, Barbarian, Druid, Assassin)
├── items/               # 1.13c 掉落管线 (TC NoDrop/Quality/Affix)、恰西注入 (Charsi Imbue)、符文之语、宝石镶嵌、赫拉迪克方块合成与 NPC 商店服务 (cubemain-data-113c.ts 151 行 Cubemain.txt / 146 条启用配方；cube-engine-113c.ts C2S 0x28 镶嵌、Gems.txt gemapplytype、78 条 Runes.txt 符文之语与 0x4F 0x18 方块合成)
├── world/               # 场景交互物体 (神殿/宝箱/传送门/小站)、任务状态机 (Quests)、NPC 对话与治疗/赌博交易
└── save/                # 原版 1.13c .d2s v96 二进制角色存档读写器 (Checksum, Attributes, Skills, QuestData, Waypoints, ItemList)
```

---

### 3. 核心子系统详解（持续完善中）

#### 3.1 25fps 核心引擎与战斗管线 (`engine/`)
- **`GameInstance` (`engine/game-instance.ts`) & `GameEngine` (`engine/game-engine.ts`)**：维护权威世界状态，按 40ms 固定步长推进（`GameInstance` 是唯一的线上/线下权威运行时；`GameEngine` 仅为遗留测试工具，生产代码不得实例化，见 #1328）：
  1. 玩家输入指令队列消费（移动、`UNIT_TILE=5` 跨层楼梯交互与 `12×12`/`16×16` 房间裁剪、普通攻击、技能施放、拾取/穿戴/药水、NPC 交互、阿卡拉洗点 `Quest 41` 与恰西注入）。
  2. 状态持续时间衰减（中毒、冰冻、诅咒、战吼、神殿 Buff）、`States.txt` `curse` 槽位互斥覆写（`0x6FCC00B0`，吸引 `Attract` 不可覆写，见 #1411）与生命/法力自然回复。
  3. 光环扫描器 (`engine/aura-scanner.ts`) 周期性范围索敌与属性挂载（严格区分 `SrvDo065` 防御光环与 `SrvDo066`/`SrvDo081` 攻击光环施法者 `passivestat` 与受众 `aurastat` 路由，并按 `aurarangecalc` 半径平方与 `aurafilter` 掩码过滤，见 #1409、#1410）。
  4. 投射物引擎 (`engine/missile-engine.ts`) 步进碰撞检测与子投射物分裂（对齐 `Vel + lvl * VelLev / 8` 初速与 `8.16` 定点步进、`Range + lvl * LevRange` 寿命、`SubLoop`、`Slow Missiles` 减速、`g_aMissileCollideTypes[9]` 碰撞掩码、`Size` 足迹重叠与 `pierce_idx` 最多 4 次穿透判定，见 #1420–#1422）。
  5. 怪物与召唤物 (`ai/summon-manager.ts`) AI 决策与动作帧推进。
  6. 状态总线 (`engine/state-bus.ts`)、单位属性列表 (`engine/stat-list.ts`) 与确定性状态哈希 (`engine/state-hash.ts`) 收集本帧脏状态并生成增量事件。
- **战斗伤害管线 (`engine/combat-pipeline.ts`)**：严格遵循 1.13c 命中率公式（AR vs DR + 等级差修正）、攻击者 `D2SeedStrc` 逐系伤害区间 `[min, max]` 掷骰与命中/闪避/格挡/暴击/压碎独立 `rand(100)` 判定（#1414）、有效抗性折算 `0x6FCFB3C0`（免疫 `1/5` 穿抗、难度 `ResistPenalty`、玩家/佣兵 `75% + max <= 95%` 与物理 `50%` 上限，#1412）、生命/法力偷取 `0x6FCFBA40`（难度 `LifeStealDivisor`、怪物 `Drain` 效率与 `>> 8` 定点换算，#1413）、碾压性打击 (Crushing Blow)、致命一击 (Deadly Strike / Critical Strike) 与撕开伤口 (Open Wounds)。

#### 3.2 全职业技能系统 (`skills/`)
- **`skills/registry.ts` & `skills/skills.ts`**：基于 `Skills.txt` 的 `srvdofunc` / `srvstfunc` 分发机制，实现亚马逊、法师、死灵法师、圣骑士、野蛮人、德鲁伊、刺客七大职业共 **221 个技能**（含协同加成公式、按各 `srvstfunc`/`srvdofunc` 精确映射 `SrcDam` 武器伤害加成 `calc1`/`calc2`/`calc4` [#1418]、毒素技能 `EType=pois` 按帧长 `ELen` 缩放总伤害 [#1419]、玩家 `srvdofunc` 齐射发射器 `engine/player-srvdo-113c.ts` [#1417]、施法延迟 Cooldown、法力消耗与尸体交互）。

#### 3.3 怪物 AI 与房间流式激活 (`ai/`)
- **`ai/monster-ai.ts`**：基于 `MonStats.txt` 的 AI 参数（`aidel`, `aip1..aip8`）与 129 项 `gpMonsterAiFunctionTable` 分发表驱动沉沦魔复活、巫师远程风筝、冥河娃娃自爆、精英怪暗金词缀（闪电强化、火焰强化死亡尸爆、冰冷强化死亡冰环、特别快速、狂热灵气、免疫计算）及 Act Boss 专属 AI 与死亡演出（安达利尔火柱震屏落石、督瑞尔死亡爆虫等）。
- **`ai/room-streamer.ts`**：围绕玩家所在子格坐标按九宫格动态激活/休眠 DRLG 房间并按 `MonDen` 与 `SuperUniques.txt` 刷怪。

#### 3.4 1.13c 物品掉落、商人赌博与存档系统 (`items/`, `save/`)
- **掉落管线、商人库存与恰西注入 (`items/drop-pipeline.ts`, `items/imbue.ts`)**：复刻 `TreasureClassEx.txt` 递归寻路、多人难度 `NoDrop` 指数衰减、MF (Magic Find) 边际递减有效值转换（Unique / Set / Rare / Magic 四级判定与同局暗金防重降级 `rare×3`）、`MagicPrefix.txt` / `MagicSuffix.txt` 词缀池权重抽样、商人普通商店城镇无人清空刷新与赌博按玩家隔离重掷生命周期（#1354–#1357），以及 Act 1 任务 3 恰西注入（`ilvl = clvl + 4` 亮金物品词缀与 `StaffMods` 生成）。
- **`.d2s` 存档兼容 (`save/d2s-reader.ts`, `save/d2s-writer.ts`)**：支持直接导入或导出与原版 Diablo II v1.13c 二进制逐字节兼容的 `.d2s` 角色存档（含 `0xAA55AA55` 文件头、循环移位累加校验和、`gf` 属性比特流、`if` 技能表、96 字节任务位图与 `JM` 物品列表）。

---

### 4. 常用验证命令

```bash
# 单独对 src/server 进行类型检查（验证零 DOM / 零 Node 依赖）
npx tsc -p tsconfig.server.json --noEmit

# 运行架构边界与掉落管线等服务端单测
npx vitest run tests/arch/boundaries.test.ts tests/drop-pipeline.test.ts
```

---
---

<a id="english"></a>

## English

> [!WARNING]
> **Server-Side Development Status Notice (Work in Progress)**:
> The `src/server` module is **currently under active development and NOT completed yet**. While foundational building blocks—such as the 25fps deterministic tick loop, combat formulas, skill handlers, TreasureClass drop pipeline, and `.d2s` binary save reader/writer—are in place and unit-tested, it does not yet provide a complete standalone server for end-to-end gameplay.
> **To play the game today, please follow the Single-Client Mode guide in the [root `README.md`](../../README.md) to connect the web client to a native C++ PvPGN + D2GS (v1.13c) server over the WebSocket bridge.**

`src/server` is the deterministic authoritative server simulation module for **Diablo II: Lord of Destruction (v1.13c)**, designed as a 1:1 port of `D2Game.dll`'s 25fps (40ms/tick) game loop, combat pipeline, 221 skills, monster AI, TreasureClass drop tables, and `.d2s` save system.

### 1. Module Responsibilities & Architectural Boundaries

- **Original DLL Counterpart**: `D2Game.dll`.
- **TypeScript Compiler Constraints (`tsconfig.server.json`)**:
  - `lib: ["ES2022", "WebWorker"]`, `types: []` — **Zero browser DOM globals and zero Node.js APIs** (enforced by `tests/arch/boundaries.test.ts` Rules 3 & 4).
  - **Strictly Deterministic Simulation**: Calling `Math.random`, `Date.now`, or `performance.now` inside `src/server` is strictly forbidden; all randomness must flow through seeded `D2Rng` instances, and all time progression is driven by discrete 25fps ticks (`tickCount`).
  - May only import `src/common` and `src/netproto` (via `src/netproto/index.ts`); importing `src/client` or `src/baker` is forbidden.

---

### 2. Directory Structure

```text
src/server/
├── index.ts             # Module barrel export (GameInstance, LocalLoopbackAdapter, LocalLobbyProvider, MultiClientRoomHost, .d2s reader/writer)
├── engine.ts            # Top-level simulation engine facade
├── loopback-adapter.ts  # LocalLoopbackAdapter & LocalLobbyProvider wrapping singlePlayer GameInstance behind GameServerAdapter & LobbyProvider
├── room-host.ts         # MultiClientRoomHost multiplexing 1..8 clients over a 25Hz multiplayer GameInstance (TsFramer & binary D2GS)
├── net/                 # Multiplayer network layer (aoi.ts spatial filtering, party.ts party/hostile roster, session.ts client wire session, trade-manager-113c.ts town player-to-player safe trade state machine with atomic 10x4 commit/rollback)
├── engine/              # 25fps core simulation engine (game-instance.ts, game-engine.ts, combat-pipeline, missile-engine, aura-scanner, stat-list, state-hash)
├── ai/                  # Monster AI decision trees, aggro/target acquisition, line-of-sight & room-streamer
├── skills/              # Server-side executors for all 221 skills across 7 classes (Amazon, Sorceress, Necromancer, Paladin, Barbarian, Druid, Assassin)
├── items/               # 1.13c drop pipeline (TC NoDrop/Quality/Affix), Charsi Imbue, runewords, gem socketing, Horadric Cube recipes & NPC vendor services (cubemain-data-113c.ts: 151-row Cubemain.txt / 146 enabled recipes; cube-engine-113c.ts: C2S 0x28 socketing, Gems.txt gemapplytype, 78 Runes.txt runewords & 0x4F 0x18 transmute)
├── world/               # Interactive world objects (shrines/chests/portals/waypoints), quest state machines & NPC dialogue/healing/gambling
└── save/                # Authentic 1.13c .d2s v96 binary character save reader/writer (Checksum, Attributes, Skills, QuestData, Waypoints, ItemList)
```

---

### 3. Core Subsystems (Work in Progress)

#### 3.1 25fps Core Engine & Combat Pipeline (`engine/`)
- **`GameInstance` (`engine/game-instance.ts`) & `GameEngine` (`engine/game-engine.ts`)**: Maintains authoritative world state and advances in fixed 40ms steps (`GameInstance` is the single online/offline authoritative runtime; `GameEngine` is a legacy test-only harness that production code must not instantiate, see #1328):
  1. Consumes queued player input commands (movement, `UNIT_TILE=5` roomtile warps & `12×12`/`16×16` room pruning, normal attacks, skill casts, item pickup/equip/potions, NPC interactions, Akara respec `Quest 41` & Charsi Imbue).
  2. Ticks state durations (poison, freeze, curses, warcries, shrine buffs), enforces `States.txt` `curse` slot exclusivity (`0x6FCC00B0`, with un-overridable Attract, see #1411), and applies natural life/mana regeneration.
  3. Runs `engine/aura-scanner.ts` for periodic radial target scanning and stat application (separating `SrvDo065` defensive vs. `SrvDo066`/`SrvDo081` offensive caster `passivestat` and target `aurastat` routing, filtered by `aurarangecalc` squared sub-tile radius and `aurafilter` flags, see #1409, #1410).
  4. Advances `engine/missile-engine.ts` for projectile step collision and sub-missile spawning (`Vel + lvl * VelLev / 8` velocity and `8.16` fixed-point stepping, `Range + lvl * LevRange` lifetime, `SubLoop`, `Slow Missiles`, `g_aMissileCollideTypes[9]` collision masks, `Size` footprint overlap, and up to 4 `pierce_idx` rolls, see #1420–#1422).
  5. Steps monster and summon (`ai/summon-manager.ts`) AI decisions and action frames.
  6. Flushes dirty state deltas through `engine/state-bus.ts`, `engine/stat-list.ts`, and `engine/state-hash.ts` into incremental `ServerEvent` emissions.
- **Combat Damage Pipeline (`engine/combat-pipeline.ts`)**: Implements 1.13c hit-chance formulas (AR vs. DR + level difference), attacker `D2SeedStrc` per-channel `[min, max]` damage range rolls and independent `rand(100)` rolls for hit/dodge/avoid/evade/weapon-block/shield-block/crit/CB (#1414), effective resistance resolution `0x6FCFB3C0` (`1/5` pierce vs. `>= 100` immunes, difficulty `ResistPenalty`, player/merc `75% + max <= 95%` and `50%` physical caps, #1412), life/mana steal `0x6FCFBA40` (difficulty `LifeStealDivisor`, monster `Drain` effectiveness, and `>> 8` fixed-point scaling, #1413), Crushing Blow, Deadly Strike / Critical Strike, and Open Wounds.

#### 3.2 All-Class Skill System (`skills/`)
- **`skills/registry.ts` & `skills/skills.ts`**: Dispatches `Skills.txt` `srvdofunc` / `srvstfunc` handlers across all 7 classes (**221 skills** total), including synergy formulas, per-handler `SrcDam` weapon damage % calc column selection (`calc1`/`calc2`/`calc4`, #1418), poison `EType=pois` total damage scaling by `ELen` (#1419), player `srvdofunc` multi-missile volley emitters (`engine/player-srvdo-113c.ts`, #1417), casting delays (cooldowns), mana costs, and corpse interactions.

#### 3.3 Monster AI & Room Streaming (`ai/`)
- **`ai/monster-ai.ts`**: Uses `MonStats.txt` AI parameters (`aidel`, `aip1..aip8`) and the 129-entry `gpMonsterAiFunctionTable` dispatch table to drive Fallen shaman revives, ranged kiting, Stygian Doll death explosions, Unique monster boss modifiers (Lightning Enchanted, Fire Enchanted death explosion, Cold Enchanted death Frost Nova, Extra Fast, Fanaticism aura, immunities), and Act Boss AI & death sequences (Andariel fire pillars/screen shake/falling rocks, Duriel maggot burst, etc.).
- **`ai/room-streamer.ts`**: Dynamically activates and hibernates DRLG rooms in a 3×3 neighborhood around players, spawning monsters according to `MonDen` and `SuperUniques.txt`.

#### 3.4 1.13c Item Drop Pipeline, Vendor/Gamble & `.d2s` Save System (`items/`, `save/`)
- **Drop Pipeline, Vendor/Gamble & Charsi Imbue (`items/drop-pipeline.ts`, `items/imbue.ts`)**: Implements `TreasureClassEx.txt` recursive tree traversal, player-count `NoDrop` exponent scaling, diminishing-returns Magic Find (MF) conversion across Unique / Set / Rare / Magic rolls (with per-game unique deduplication degrading duplicates to `3×` durability rare), weighted affix sampling from `MagicPrefix.txt` / `MagicSuffix.txt`, town-shared vendor inventory refresh when the town empties & per-player gamble reroll lifecycles (#1354–#1357), and Act 1 Quest 3 Charsi Imbue rare item generation (`ilvl = clvl + 4` with `StaffMods`).
- **`.d2s` Binary Save Compatibility (`save/d2s-reader.ts`, `save/d2s-writer.ts`)**: Reads and writes byte-authentic Diablo II v1.13c `.d2s` character save files (`0xAA55AA55` header, rotate-add checksum, `gf` stat bitstreams, `if` skill arrays, 96-byte quest bitmaps, and `JM` item lists).

---

### 4. Common Verification Commands

```bash
# Run strict typecheck on src/server (verifies zero DOM / zero Node dependencies)
npx tsc -p tsconfig.server.json --noEmit

# Run architectural boundary and drop pipeline unit tests
npx vitest run tests/arch/boundaries.test.ts tests/drop-pipeline.test.ts
```
