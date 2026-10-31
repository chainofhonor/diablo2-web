# Diablo II v1.13c Reverse-Engineering Knowledge Base (`kb/`)

> **Ground Truth Invariant**: All documents in this knowledge base are reverse-engineered strictly and exclusively from the original Diablo II: Lord of Destruction v1.13c binary assembly (`D2Client.dll`, `D2Game.dll`, `D2Common.dll`, `D2CMP.dll`, `D2Multi.dll`, `D2MCPClient.dll`, `D2Gfx.dll`, `D2Win.dll`, `D2Lang.dll`, `Storm.dll`, `Fog.dll`, etc.) and official Blizzard 1.13c MPQ data tables (`Skills.txt`, `Missiles.txt`, `MonStats.txt`, `SuperUniques.txt`, `Objects.txt`, `Inventory.txt`, `States.txt`, `PetType.txt`, etc.) in `samples/d2/` (`~/d2-data/`) as the sole ground truth.

---

## 目录结构 / Directory Structure

```
kb/
├── README.md                                  # 知识库索引与架构概览 (本文件)
├── missile-velocity-range-113c.md             # 投射物速度/加速度 8.16 定点截断、Range/LevRange 寿命、SubLoop 与减速箭 (Slow Missiles)
├── audio/                                     # 音效表与声音记录层
│   ├── d2gs-0x2c-playsound-113c.md            # D2GS 0x2C 声音事件码分发 (0x6fb5d740 -> 0x6fad3f20 -> 0x6fad3200) 与 0xA4 守卫
│   └── sound-records-113c.md                  # Sounds.txt 载入后处理、PlaySound/SoundUpdate/StartSound/AllocChannel 与 D2sound 输出
├── combat/                                    # 战斗结算、抗性、吸血吸蓝与随机判定
│   ├── d2game-effective-resist-113c.md        # 有效抗性 0x6FCFB3C0：-敌抗、难度 ResistPenalty、玩家/佣兵上限 (物理 50 / 75+max ≤95)
│   ├── d2game-life-mana-steal-113c.md         # 生命/法力偷取 0x6FCFBA40：难度惩罚 LifeStealDivisor、怪物 Drain 效率与 /256 位移
│   └── d2game-sunitdmg-rng-rolls-113c.md      # SUnitDmg 伤害上下限 64 位种子掷骰与命中/闪避/格挡/暴击独立 rand(100) 判定
├── items/                                     # 物品系统、掉落管线与渲染规范
│   ├── belt-capacity-113c.md                  # 腰带容量计算 (Armor.txt belt -> belts.txt numboxes) 与卸下腰带溢出回退
│   ├── d2client-ethereal-item-rendering.md    # 无形装备 50% 半透明渲染与 GFX_DrawItem 调用规约
│   ├── d2client-ground-item-labels-113c.md    # 地面物品 Alt 标签文本拼装、孔数括号与成色着色规则 (0x6fb20510)
│   ├── d2client-unidentified-tooltip-reqlevel.md # 未辨识装备 Tooltip 需求等级显示门控
│   ├── d2game-treasure-class-drop-pipeline-113c.md # TreasureClassEx 掉落管线、NoDrop 多人缩放、品质掷骰与暗金单局防重 (0x6FC51070)
│   ├── d2game-vendor-gamble-113c.md           # D2Game 商人商店生成、城镇无人刷新生命周期与赌博机制逆向规范
│   └── item-addskill-tab-encoding-113c.md     # stat 188 技能系参数编码 tab + 8*class (func 10 / descfunc 14)
├── monsters/                                  # 怪物系统、AI 状态机、寻路与死亡演出
│   ├── d2client-boss-death-cltdo-113c.md      # Boss 死亡客户端状态机 (CltDoFunc 31/32/42/47/66)：安达利尔、血鸟、神鹰、议会、暗黑、督瑞尔
│   ├── d2client-boss-death-shows-113c.md      # Boss 死亡演出总览：拉达蒙特、召唤者、衣卒尔、尼拉塞克、山克、巴尔 (CltDo 29/45/52/79/82/91)
│   ├── d2client-champion-monster-name-113c.md # 头目怪(Champion)标志位、词缀与血条命名优先级
│   ├── d2client-monster-death-state-113c.md   # 怪物死亡状态切换 (MONMODE_DEATH 0x08 -> MONMODE_DEAD 0x0C) 与尸体留存判定
│   ├── d2client-monumod-death-fx-113c.md      # 精英怪 MonUMod 死亡特效分发 (0x6FAD1660)：火焰/闪电/冰冷强化、圣甲虫与自爆怪
│   ├── d2client-npc-interaction-range-113c.md # NPC 点击交互距离(#10407)与对话框每帧守卫(仅存活判定)
│   ├── d2common-path-wall-follow-113c.md      # D2Common 动态寻路左右手沿墙绕障 (sub_6FDA69E0) 与步进防抖
│   ├── d2game-monai-dispatch-table-113c.md    # D2Game.dll gpMonsterAiFunctionTable (0x6FD294B8) 全部 129 项怪物 AI 函数分发表
│   ├── diablo-baal-art-paging-921.md          # 暗黑破坏神与巴尔 DCC 高帧动画纹理分页防裁切规范 (#921)
│   └── monai-113c-profiles.md                 # MonStats.txt AI 参数 (aidel / aip1..8) 与行为配置总表
├── network/                                   # D2GS 网络报文逆向规范
│   └── d2gs-0x1a-0x1f-exp-attr-113c.md        # D2GS 0x1A..0x1F 属性/经验更新：0x1A/0x1B 增量累加 vs 0x1C 绝对值与 0x1D..0x1F BaseStat
├── objects/                                   # 场景物件与交互逻辑
│   ├── d2-npc-placeholder-objects-113c.md     # 城镇 NPC 静态占位 Object (InitFn 18/19) 与动态实体互斥隐藏
│   ├── d2client-object-interact-hittest-113c.md # 物件交互距离计算(#10594)与屏幕点击命中判定
│   └── d2game-roomtile-warp-113c.md           # 跨层楼梯/入口 (UNIT_TILE=5) 交互、寻路、12x12房间裁剪与同幕往返规范
├── quests/                                    # 五幕任务状态机与任务物品规范
│   ├── a2q6-staff-orifice-show-113c.md        # A2Q6 赫拉迪姆杖孔洞交互、光束震屏演出与督瑞尔房门开启 (OperateFn 25/48)
│   ├── a3-compelling-orb-smash-113c.md        # A3Q5 强制球体击碎 (OperateFn 53)、克林姆的意志消耗与憎恨囚牢入口开启
│   ├── a5q6-game-completion-113c.md           # A5Q6 巴尔毁灭之末传送门 (OperateFn 72) -> S2C 0x61 07 -> 胜利结算画面 (#924)
│   └── d2game-quest-item-grant-pickup-113c.md # 任务物品发放 (QUESTS_CreateItem 满包掉落地面) 与唯一性拾取前置校验 (0x6fcf7020)
├── render/                                    # 渲染、瓦片层级、墙壁遮挡淡出与光照
│   ├── d2client-drawmode-113c.md              # 绘制模式汇总、客户端 missile 动画速率 (AnimSpeed<<4)、CltDoFunc 25 64 环、States castoverlay
│   ├── d2client-tile-layer-order-113c.md      # 瓦片绘制顺序 (lower wall 16..19 先于地板)、上层墙遮挡淡出与 POPS 屋顶揭开 (#919, #1009)
│   ├── d2client-unit-light-records-113c.md    # 单位光源记录 (玩家/怪物/弹道/叠加层) 与 stat 89/90 回调
│   ├── d2gfx-draw-mode-113c.md                # D2gfx/D2Direct3D eDrawMode 混合模式映射 (Overlay Trans 恒等、Missile Trans 0/1/2->5/3/4)
│   └── dt1-material-flags-animated-113c.md    # DT1 +0x06 u16 材质标志与熔岩动画判定 (TILE_FLAGS_LAVA 0x100, #923)
├── ui/                                        # 用户界面、HUD、交易、镶嵌注入与大厅协议
│   ├── d2client-automap-waypoint-parity.md    # 小地图 39 个小站图元 (307 / 617..619) 地板层与对象层对齐规范
│   ├── d2client-charsheet-vitals-113c.md      # 角色属性面板 (C/A 键) 耐力/生命/法力双框布局与底栏血球上限聚合
│   ├── d2client-hireables-portrait-lifebar.md # 左上角随从/佣兵头像与血条尺寸及渲染规则
│   ├── d2client-level-entry-title-113c.md     # 切换区域 "Entering xxx" 居中标题 (0x6fb6f0d0)：EntryFile DC6、任务覆盖表、120 帧、textShifts[1]
│   ├── d2client-mercenary-screen-113c.md      # 佣兵装备/属性面板 (O 键界面) 坐标与排版
│   ├── d2client-npc-menu-quest-gates-113c.md  # NPC 菜单任务门控表 (0x6fba2330)：泰瑞尔前往哈洛加斯 = q[28] bit0 + 资料片
│   ├── d2client-player-corpse-pickup-113c.md  # 玩家尸体 (UNIT_PLAYER) 同步、悬停优先级 +1 与点击拾取距离规范 (#922)
│   ├── d2client-player-trade-flow-113c.md     # 玩家面对面交易全流程 (S2C 0x77/0x78/0x79, C2S 0x4F, storePage 2/3, 绿/红勾选框)
│   ├── d2client-quest-log-slots-113c.md       # 任务面板 (Q 键) 6 槽位坐标表 (0x6fba8100)、97px 步进、凹槽叠加层与命中矩形 (#914)
│   ├── d2client-quest-log-true-tomb-symbol-113c.md # A2Q6 塔拉夏真古墓符号 (invps.dc6) 门控 (0x50 tombOffset + qstsa2q61a 页码 7, #1008)
│   ├── d2client-useitem-imbue-socket-orifice-113c.md # useitem.cpp 单格放入窗口：恰西注入、拉苏克打孔、安雅刻名与赫拉迪姆杖孔洞
│   ├── d2client-vendor-stash-gold-display.md  # 交易/个人储藏箱金币框渲染布局与文本对齐
│   ├── d2game-player-busy-gate-113c.md        # 服务端 PLAYER_IsBusy 门控与小站/储物箱/方块/NPC 交互关闭报文规范 (#920)
│   └── d2multi-join-game-panel-113c.md        # 多人大厅加入游戏面板控件表与 MCP 0x06 协议
└── skills/                                    # 技能与弹道逆向工程参考
    ├── aura-srvdo-stat-routing-113c.md        # 服务端光环函数 (SrvDo065/066/081) 施法者 passivestat 与受众 aurastat 分离路由 (#1409)
    ├── aura-target-filter-113c.md             # 光环目标扫描 (0x6FCC0C70)：aurarangecalc 半径平方与 aurafilter 掩码位 (#1410)
    ├── client-state-ids-113c.md               # 客户端技能预测使用的真实 States.txt 行号 (blaze 13 / conviction 28 / stunned 21 / cold 11 等) 与持续帧
    ├── d2-skill-hotkeys-selectskill-respec-113c.md # 技能快捷键 (0x51/0x7B)、SelectSkill (0x3C) 服务端校验、洗点/等级归零移除节点 (#10302) 与槽位可选门控 (0x6faed110, #1490)
    ├── d2-skills-missiles-dispatch-tables.md  # 全技能与弹道分发函数总表与通用动作 (Dispatch Tables)
    ├── d2client-cltdofunc25-ring-113c.md      # SkillCltDoFunc 25 (0x6FB23CB0) 64 向圆环弹道 (霜之新星/新星/剧毒新星/战吼等)
    ├── missile-clthit31-icebreak-melt.md      # 冰冻碎裂 (shatter=1) 与冰片融水 (MissCltHitFunc[31] 0x6FB605B0 -> icebreak*melt, #1453)
    ├── player-srvdo-missile-volleys-113c.md   # 玩家 srvdofunc 齐射弹道 (多重箭/牙/炮轰/充能弹/火焰风暴/龙卷风/骨墙/陨石/狂怒等, #1417)
    ├── skill-damage-pct-calc-113c.md          # SrcDam 武器伤害加成 calc 列按 srvstfunc/srvdofunc 精确分发规范 (#1418)
    ├── bosses/                                # Boss 与超级暗金怪技能
    │   ├── skills-boss-act1-act2-act3.md      # Act 1-3 Boss 技能 (安达利尔、督瑞尔、墨菲斯托等)
    │   ├── skills-boss-act4-diablo.md         # Act 4 Boss 技能 (暗黑破坏神、混沌避难所封印怪等)
    │   └── skills-boss-act5-baal-ubers.md     # Act 5 & 6Boss 技能 (巴尔、地狱火三魔神等)
    ├── missiles/                              # 弹道机制与客户端轨迹渲染
    │   ├── d2client-meteor-cltdofunc-clthitfunc.md # 陨石下落轨迹计算、撞击闪光与地面火判定
    │   ├── d2game-srvdo-06-23-30-druid-113c.md # 服务端弹道 SrvDo 6 / 23-30 (熔岩巨石/火山/龙卷风/藤蔓/狂犬病传染归属, #1224-#1228)
    │   ├── d2game-srvhit-radius-113c.md       # 服务端弹道 pSrvHitFunc (0x6FD2DA48) 爆炸/溅射/弹跳半径 (sHitPar1 与技能 calc 等轴测半径, #1457)
    │   ├── missile-collision-113c.md          # CollideType 碰撞掩码表 (0x6FD2D928)、敌我过滤 (0x6FC58FB0) 与 Size 包围盒重叠 (#1422)
    │   └── missile-pierce-113c.md             # 弹道穿透 pierce_idx (stat 328)：创建期最多 4 次掷骰 (0x6FC8F660) 与命中消耗 (#1420)
    └── player/                                # 玩家七大职业技能树 (21 份文档，210 项技能)
        ├── skills-ama-bow-crossbow.md         # 亚马逊：弓与十字弓技能 (Bow & Crossbow)
        ├── skills-ama-javelin-spear.md        # 亚马逊：标枪与长矛技能 (Javelin & Spear)
        ├── skills-ama-passive-magic.md        # 亚马逊：被动与魔法技能 (Passive & Magic)
        ├── skills-ass-martial-arts.md         # 刺客：武术技能 (Martial Arts)
        ├── skills-ass-shadow-disciplines.md   # 刺客：暗影修行 (Shadow Disciplines)
        ├── skills-ass-traps.md                # 刺客：陷阱技能 (Traps)
        ├── skills-bar-combat.md               # 野蛮人：战斗技能 (Combat Skills)
        ├── skills-bar-masteries.md            # 野蛮人：战斗专家 (Combat Masteries)
        ├── skills-bar-warcries.md             # 野蛮人：战吼技能 (Warcries)
        ├── skills-dru-elemental.md            # 德鲁伊：元素技能 (Elemental Skills)
        ├── skills-dru-shapeshifting.md        # 德鲁伊：变形技能 (Shape Shifting)
        ├── skills-dru-summoning.md            # 德鲁伊：召唤技能 (Summoning)
        ├── skills-nec-curses.md               # 死灵法师：诅咒技能 (Curses)
        ├── skills-nec-poison-bone.md          # 死灵法师：毒素与白骨技能 (Poison & Bone)
        ├── skills-nec-summoning.md            # 死灵法师：召唤技能 (Summoning)
        ├── skills-pal-combat.md               # 圣骑士：作战技能 (Combat Skills)
        ├── skills-pal-defensive-auras.md      # 圣骑士：防御灵气 (Defensive Auras)
        ├── skills-pal-offensive-auras.md      # 圣骑士：攻击灵气 (Offensive Auras)
        ├── skills-sor-cold.md                 # 巫师：冰冻法术 (Cold Spells)
        ├── skills-sor-fire.md                 # 巫师：火焰法术 (Fire Spells)
        └── skills-sor-lightning.md            # 巫师：闪电法术 (Lightning Spells)
```

---

## 1. 战斗结算、物品系统与渲染规范 (`combat/`, `items/`, `network/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`combat/d2game-effective-resist-113c.md`](./combat/d2game-effective-resist-113c.md) | `D2Game.dll` `0x6FCFB3C0` (`SUNITDMG_ApplyResistances`) | 有效抗性计算：免疫 (`>= 100`) 时仅 `passive_mastery` 之前穿抗类的 `-敌抗` 按 `1/5` 生效，非免疫全额扣减；玩家与佣兵享受难度抗性惩罚 (`DifficultyLevels.txt` `ResistPenalty`: 0 / -40 / -100) 且上限受 `max*resist` 限制（物理 50%，元素/毒 `75% + max <= 95%`，下限 `-100%`）(Issue #1412)。 |
| [`combat/d2game-life-mana-steal-113c.md`](./combat/d2game-life-mana-steal-113c.md) | `D2Game.dll` `0x6FCFBA40` (`SUNITDMG_ApplyLeeches`) | 生命/法力偷取：基于实际造成的物理伤害（扣减 DR 后），先后应用难度系数 (`DifficultyLevels.txt` `LifeStealDivisor`/`ManaStealDivisor`) 与受击怪物 `MonStats.txt` `Drain` 效率百分比，并以 `>> 8` (`/256`) 换算固定点生命/法力回复 (Issue #1413)。 |
| [`combat/d2game-sunitdmg-rng-rolls-113c.md`](./combat/d2game-sunitdmg-rng-rolls-113c.md) | `D2Game.dll` `0x6FCBF210` (`SUNITDMG_RollDamage`), `0x6FCFC530` | 物理与五系元素伤害区间 `[min, max]` 使用攻击者 `D2SeedStrc` 逐系独立掷骰 (`min + rand(max - min + 1)`)；命中率、亚马逊闪避三姐妹 (`Dodge`/`Avoid`/`Evade`)、武器格挡、盾牌格挡、暴击 (`CS`/`DS`) 与压碎性打击 (`CB`) 各自消耗独立 `rand(100)`，严禁复用同一次随机值 (Issue #1414)。 |
| [`items/belt-capacity-113c.md`](./items/belt-capacity-113c.md) | `D2Common.dll` `#10690` (`INVENTORY_GetBeltCapacity`), `Armor.txt`, `belts.txt` | 腰带总格数由穿戴腰带的 `Armor.txt` `belt` 索引查 `belts.txt` `numboxes` 决定（裸装默认 4 格，轻型 8 格，中型 12 格，重型/精英 16 格）；卸下或更换小容量腰带时第 `numboxes..15` 槽位的药水自动退回背包或掉落地面。 |
| [`items/d2client-ethereal-item-rendering.md`](./items/d2client-ethereal-item-rendering.md) | `D2Client.dll` `0x6FB1B5A0` (`GFX_DrawItem`), `D2gfx.dll` `0x6FA8B080` | 无形装备 (`IFLAG_ETHEREAL = 0x00400000`) 的 2D UI 精灵以 `DRAWMODE_TRANS50 = 1` (50% 透明度) 渲染；嵌入无形装备中的凹槽宝石/符文维持 100% 不透明 (`DRAWMODE_NORMAL = 5`)。 |
| [`items/d2client-ground-item-labels-113c.md`](./items/d2client-ground-item-labels-113c.md) | `D2Client.dll` `0x6fb20510` (`DrawGroundItemLabels`) | 按住 Alt 时地面物品标签文本拼装（金币数量前缀、孔数/无形灰字、辨识前后词缀名）与碰撞避让布局规范。 |
| [`items/d2client-unidentified-tooltip-reqlevel.md`](./items/d2client-unidentified-tooltip-reqlevel.md) | `D2Client.dll` `0x6FB43D89`, `D2Common.dll` `0x6FD76190` | 未辨识物品在 Tooltip 中严格抑制 `Required Level` (需求等级) 行；只有在辨识后 (`IFLAG_IDENTIFIED = 0x10`) 才参与词缀需求等级计算。护身符未辨识时在物品栏中不生效。 |
| [`items/d2game-treasure-class-drop-pipeline-113c.md`](./items/d2game-treasure-class-drop-pipeline-113c.md) | `D2Game.dll` `0x6FC51070`, `0x6FC51470`, `0x6FC4F2D0` | `TreasureClassEx.txt` 递归寻路、多人难度 `NoDrop` 指数衰减、MF 边际递减有效值转换、暗金物品单局防重位图 (`pGame->dwUniqueFlags`) 及同局二次命中降级为 3 倍耐久亮金 (Failed Unique) 规范。 |
| [`items/d2game-vendor-gamble-113c.md`](./items/d2game-vendor-gamble-113c.md) | `D2Game.dll` `0x6FCAF4E0..0x6FCB0320`, `0x6FCDCB20..0x6FCDF810` | 商人普通商店库存 (`pNpcRecord->pInventory`) 城镇共享、开关窗口不重掷、仅当城镇内最后一名玩家离开时清空刷新；赌博库存 (`pGambleList`) 按玩家隔离、每次打开重掷、连续 12 次放置失败终止、单次 `ilvl` 掷骰、Failed Unique (`rare×3`) / Failed Set (`magic×2`) 降级规范 (Issue #1357)。 |
| [`items/item-addskill-tab-encoding-113c.md`](./items/item-addskill-tab-encoding-113c.md) | `D2Common.dll` `0x6FD98470` (property func 10), `D2Client.dll` `0x6FB052E2` (descfunc 14) | `skilltab` par (线性 `class*3+tab`, 0..20) 写入 stat 188 时编码为 `tab + 8*class` (Save Param Bits 16)；显示时 `class = param>>3`、`tab = param&7`，`tab>=3` 不显示。法师火系 = 8 (曾被误当线性键 8 = 死灵召唤, #925)。 |
| [`network/d2gs-0x1a-0x1f-exp-attr-113c.md`](./network/d2gs-0x1a-0x1f-exp-attr-113c.md) | `D2Client.dll` `0x6fb5e080..0x6fb5e310` | D2GS `0x1A` (u8) / `0x1B` (u16) 经验包为相对增量累加 (`AddStat`)，`0x1C` (u32) 为绝对总经验覆盖 (`SetStat`)；`0x1D`/`0x1E`/`0x1F` 为属性 BaseStat 更新。 |

---

## 2. 怪物系统、AI 与死亡演出 (`monsters/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`monsters/d2client-boss-death-cltdo-113c.md`](./monsters/d2client-boss-death-cltdo-113c.md) | `D2Client.dll` `CltDoFunc` `31`, `32`, `42`, `47`, `66` | Boss 死亡客户端状态机：安达利尔火柱落石震屏 (`CltDo 31`)、血鸟灵魂喷发 (`CltDo 32`)、血鹰巢穴破壳 (`CltDo 42`)、崔凡克议会灵魂飞升 (`CltDo 47`)、暗黑破坏神与督瑞尔死亡演出 (`CltDo 66`)。 |
| [`monsters/d2client-boss-death-shows-113c.md`](./monsters/d2client-boss-death-shows-113c.md) | `D2Client.dll` `CltDoFunc` `29`, `45`, `52`, `79`, `82`, `91` | 拉达蒙特、召唤者、衣卒尔、尼拉塞克、督军山克投石火雨与巴尔触手消融等 Boss 死亡视觉特效与弹道序列。 |
| [`monsters/d2client-champion-monster-name-113c.md`](./monsters/d2client-champion-monster-name-113c.md) | `D2Game.dll` `0x6FC43D5E`, `D2Client.dll` `0x6FAD0620` | Champion 头目怪生成时设置 `0x04` 标志，并带有专属 `MonUMod.txt` 词缀 (36: Ghostly, 37: Fanatic, 38: Possessed, 39: Berserker)。客户端解析显示名与血条颜色时，`0x04` 优先级高于普通的 `0x08` (Unique)，血条固定为蓝色 (`color = 3`)。 |
| [`monsters/d2client-monster-death-state-113c.md`](./monsters/d2client-monster-death-state-113c.md) | `D2Client.dll` `0x6FAD1480`, `MonStats2.txt` | 怪物从 `MONMODE_DEATH (8)` 动画播放完毕自动流转至 `MONMODE_DEAD (12)` 尸体态的帧事件、无尸体怪物 (`corpseSel=0`) 隐藏与碎冰隐藏规则。 |
| [`monsters/d2client-monumod-death-fx-113c.md`](./monsters/d2client-monumod-death-fx-113c.md) | `D2Client.dll` `0x6FAD1660` (`MONUMOD_ClientDeathDispatch`) | 精英怪 `MonUMod` 死亡客户端特效分发：火焰强化尸爆、冰冷强化霜之新星、闪电强化充能弹、圣甲虫与冥河娃娃自爆。 |
| [`monsters/d2client-npc-interaction-range-113c.md`](./monsters/d2client-npc-interaction-range-113c.md) | `D2Common.dll` `#10407` (`0x6FDCFCD0`), `D2Client.dll` `0x6facafea`, `0x6fac9ef0`, `0x6faf83b0` | 点击怪物/NPC 时以单位尺寸修正距离 `#10407` 判定：MonStats `interact` 置位阈值 `2`，否则 `5`；走近后的待定交互要求 `#10407 <= 2` 且 `#10324(..., 0x804) == 0`。NPC 对话框打开后每帧守卫 `0x6faf83b0` 只检查玩家/NPC 存活 (玩家模式 `0x11`、NPC 模式 `0xC`)，**不做距离复检** (Issue #890)。 |
| [`monsters/d2common-path-wall-follow-113c.md`](./monsters/d2common-path-wall-follow-113c.md) | `D2Common.dll` `0x6FDA69E0` | `D2Common` 动态寻路左右手沿墙绕障步进算法与防抖动终止条件。 |
| [`monsters/d2game-monai-dispatch-table-113c.md`](./monsters/d2game-monai-dispatch-table-113c.md) | `D2Game.dll` `0x6FD294B8` (`gpMonsterAiFunctionTable`) | 全部 129 项怪物 AI 函数虚拟地址、对应 `MonAI.txt` 行号与行为语义映射总表。 |
| [`monsters/diablo-baal-art-paging-921.md`](./monsters/diablo-baal-art-paging-921.md) | `src/baker/packers/pack-entity-assets.ts` | 暗黑破坏神与巴尔等超大尺寸多方向 DCC 精灵图集分页打包与 UV 采样防裁切规范 (Issue #921)。 |
| [`monsters/monai-113c-profiles.md`](./monsters/monai-113c-profiles.md) | `MonStats.txt`, `MonAI.txt` | 怪物 AI 参数 (`aidel`, `aip1..aip8`) 在三难度下的决策概率、冷却与索敌半径解析。 |

---

## 3. 场景物件、跨层楼梯与五幕任务 (`objects/`, `quests/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`objects/d2-npc-placeholder-objects-113c.md`](./objects/d2-npc-placeholder-objects-113c.md) | `D2Game.dll` `0x6fc89140`, `D2Client.dll` `0x6fad5120`, `Objects.txt` (`InitFn 18/19`) | 城镇 NPC 静态占位 Object (`121/122` 杰海因、`378/379` 赫拉铁力、`461/462` 夸尔凯克、`542/543` 安雅，`Token=ss`, `Selectable=0`) 与 `0xAC AssignNPC` 动态 NPC 实体 (`hcIdx = 201, 253, 515, 512`) 双向互斥隐藏，消除双重实体幽灵克隆 (Issue #900)。 |
| [`objects/d2client-object-interact-hittest-113c.md`](./objects/d2client-object-interact-hittest-113c.md) | `D2Common.dll` `#10594` (`0x6fd81000`), `D2Client.dll` `0x6fb568e0` | 物件交互距离严格按 `Objects.txt` 中的 `SizeX`/`SizeY` 计算包围盒并剔除 4 个极端角点（如艾维斯之树 `Id=30` 的 5×5 占地允许站在边缘交互）；屏幕命中测试分支于 `CollisionSubst` 固定包围盒与 DCC/DC6 动画帧扩展边界。 |
| [`objects/d2game-roomtile-warp-113c.md`](./objects/d2game-roomtile-warp-113c.md) | `D2Client.dll` `0x6facaed0..0x6facafe7`, `D2Game.dll` `0x6fcee150`, `0x6fcf02bc..0x6fcf0322`, `0x6fd019c0` | `UNIT_TILE (5)` 远距点击发送 `MoveToUnit(5, id)` 走近至 `#10407 <= 4` 且无 `0x804` 阻挡后发 `0x13`；同幕跨层在 `UnitReassign` 时立即同步当前层、`RoomHide` 按真实 `12×12`/`16×16` 房间尺寸裁剪、同层重复 `lvlWarpId` 按坐标就近匹配 (Issue #1488)。 |
| [`quests/a2q6-staff-orifice-show-113c.md`](./quests/a2q6-staff-orifice-show-113c.md) | `D2Game.dll` `OperateFn 25/48`, `D2Client.dll` `useitem.cpp` | A2Q6 赫拉迪姆杖孔洞 (`Horadric Orifice`, `Id=152`) 交互打开 `useitem` 模式 0 窗口，放入 `hst` 确认 (`C2S 0x44`) 后触发光束、震屏与督瑞尔房门开启演出。 |
| [`quests/a3-compelling-orb-smash-113c.md`](./quests/a3-compelling-orb-smash-113c.md) | `D2Game.dll` `0x6fc69130` (`OperateFn 53`) | A3Q5 装备克林姆的意志 (`qf2`) 击碎强制球体 (`Compelling Orb`, `Id=404`) 的状态流转、武器销毁与崔凡克楼梯入口开启。 |
| [`quests/a5q6-game-completion-113c.md`](./quests/a5q6-game-completion-113c.md) | `D2Game.dll` `0x6fc7fd00` (`OperateFn 72`), `D2Client.dll` `0x6fb5f940`, `0x6fb399a0` | 击败巴尔后点击毁灭之末传送门 (`LastPortal`)：先传送回哈洛加斯并保存存档，再发送 `S2C 0x61 07` 播放结局动画、客户端回发 `0x48` 与 `0x69 LeaveGame` 并弹出头衔晋升胜利画面 (Issue #924)。 |
| [`quests/d2game-quest-item-grant-pickup-113c.md`](./quests/d2game-quest-item-grant-pickup-113c.md) | `D2Game.dll` `0x6fc95df0` (`QUESTS_CreateItem`), `0x6fcf7020` (`ITEMS_PickUpGroundItem`) | NPC 发放任务物品时若背包无空位 (`bDropIfFull=1`) 自动掉落在脚下地面并播放 `cantCarry1`；地面拾取任务物品前先做全容器唯一性检查 (`0x6fcf2f00`) 与背包空间预检，严禁在校验失败时误删地面物品。 |

---

## 4. 用户界面、HUD 与大厅协议 (`ui/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`ui/d2client-automap-waypoint-parity.md`](./ui/d2client-automap-waypoint-parity.md) | `D2Client.dll` `0x6FB12580` (`AUTOMAP_RevealRoom`), `0x6FB11D00..0x6FB11E60` (`AUTOMAP_AddRoomUnits`) | 小地图小站图元与 `Objects.txt` / `AutoMap.txt` 严格对齐：`AutoMap == 0` 的第 1–4 幕及哈洛加斯小站仅由地板层 (`Tile="fl", Style=0, Seq=54 -> 307`)、保存式城镇底图 (`40, 103, 109`) 或术士峡谷地板 (`617..619`) 表达，绝不向 `pObjects` 重复注入偏移的 `307`；仅 `Id = 402` (限 Level 74) 与第 5 幕野外/洞穴/神殿小站 (`494, 496, 511, 539`) 向 `pObjects` 写入单一 `307` (Issue #867)。 |
| [`ui/d2client-charsheet-vitals-113c.md`](./ui/d2client-charsheet-vitals-113c.md) | `D2Client.dll` `0x6FB6CEA0` (`UI_DrawCharSheet`), `0x6FB8D6F0`, `0x6FAD7BD0`, `0x6FAD7A90`, `D2Common.dll` `#10913/#10186/#10578` | 角色属性面板耐力/生命/法力双框严格按 `0x6FB8D6F0` 表项 `[7]..[13]`：**左框 (`ox+250`) 绘制上限值**（有加成时蓝字）、**右框 (`ox+291`) 绘制当前值**（白字）；底栏生命/法力球与耐力条上限统一通过 `StatOpEngine` 聚合装备、护身符、被动与状态加成，避免将满血当前值按未穿戴裸装 `BaseStats` 截断 (Issue #903)。 |
| [`ui/d2client-hireables-portrait-lifebar.md`](./ui/d2client-hireables-portrait-lifebar.md) | `D2Client.dll` `0x6FB0AC70..0x6FB0C7F0` (`Hireables.cpp`) | 界面左上角随从与佣兵头像框绘制规范：46×41 像素头像、46×5 像素血条、三色生命百分比阈值 (绿 $\ge 50\%$, 黄 $25\%\sim 49\%$, 红 $< 25\%$) 与 56px 水平排布步进。 |
| [`ui/d2client-level-entry-title-113c.md`](./ui/d2client-level-entry-title-113c.md) | `D2Client.dll` `0x6fb6f0d0`, `0x6fb8be88` | 切换关卡时绘制 120 帧居中 `EntryFile` DC6 横幅 (`Entering The Blood Moor` 等)，支持 A1Q1/A1Q5 任务状态覆盖表且进程首次进图不显示 (Issue #1007)。 |
| [`ui/d2client-mercenary-screen-113c.md`](./ui/d2client-mercenary-screen-113c.md) | `D2Client.dll` `0x6fb4ae80` (`UI_DrawMercScreen`) | 佣兵装备属性面板 (按 `O` 键打开) 的 800×600 与 640×480 布局、头部/衣服/左右手装备格坐标、12 项属性文本排版与抗性金字/红字门控。 |
| [`ui/d2client-npc-menu-quest-gates-113c.md`](./ui/d2client-npc-menu-quest-gates-113c.md) | `D2Client.dll` `0x6fba3ba0`, `0x6fba2330`, `0x6faf5cc0` | NPC 菜单静态项与 9 项任务门控动态项（瓦瑞夫/马席夫东行、卡夏/夸尔凯克雇佣、恰西注入、拉苏克打孔、安雅刻名、泰瑞尔前往哈洛加斯 `q[28] bit0 + expansion`）。 |
| [`ui/d2client-player-corpse-pickup-113c.md`](./ui/d2client-player-corpse-pickup-113c.md) | `D2Game.dll` `D2GAME_CORPSE_Handler_6FC7FBD0`, `sub_6FC828D0`, `sub_6FC80440`; `D2Client.dll` `0x6fb01e80`, `0x6fb55440`, `0x6facb7f0`, `0x6facab20`, `0x6fac94e0` | 玩家尸体为新 GUID 的 `UNIT_PLAYER`（`0x59` → `0x0D` code 9 → `0x74`）；悬停仅跳过本地玩家、死亡玩家优先级 +1；不可攻击的非怪物单位点击进入交互处理 `C2S 0x13`（与左键技能无关），服务端 8 格内拾取、50 格内自动走近 (Issue #922)。 |
| [`ui/d2client-player-trade-flow-113c.md`](./ui/d2client-player-trade-flow-113c.md) | `D2Client.dll` `0x6fb094c0..0x6fb0a230`, `0x6fb4a173`; `D2Game.dll` `0x6fc90750..0x6fc927c0` | 玩家面对面交易完整状态机：居中请求/等待弹窗 (`0x77 0x00/0x01`)、`0x78 TradeInit` 双栏开启、`storePage=2/3` (`bInvPage=1/2`) 交易格同步、`0x79` 金币同步、物品变动重置绿勾与防满包原子交换。 |
| [`ui/d2client-keyconfig-ui-policy-113c.md`](./ui/d2client-keyconfig-ui-policy-113c.md) | `D2Client.dll` `0x6FBA4F10`, `0x6FBCA2E0`, `0x6FAE8BB0`, `0x6FB72790`, `0x6FB8D538`, `0x6FAE9E20` | 57 条键位命令表与 default.key、模态输入门控、UI 开关策略矩阵、死亡锁定、自动地图淡出与聊天命令。 |
| [`ui/d2client-quest-log-slots-113c.md`](./ui/d2client-quest-log-slots-113c.md) | `D2Client.dll` `0x6facf1e0..0x6facf73c`, `0x6fba8100`, `0x6facc7a0` | 任务面板 6 个图标槽位精确像素坐标 (`drawX = 26/123/220`, `drawY = 121/218`, 步进 97px)、`questsockets.dc6` 边框后置叠加与命中测试矩形 (Issue #914)。 |
| [`ui/d2client-quest-log-true-tomb-symbol-113c.md`](./ui/d2client-quest-log-true-tomb-symbol-113c.md) | `D2Client.dll` `0x6facc740`, `0x6facf46c..0x6facf5e3` | A2Q6 塔拉夏真古墓符号 (`invps.dc6` 7 帧) 绘制门控：接收 `S2C 0x50` `tombOffset (0..6)` 且 A2Q6 处于第 7 页 (`qstsa2q61a`) 时以 `DRAWMODE_MODULATE (3)` 绘制在文本下方 (Issue #1008)。 |
| [`ui/d2client-useitem-imbue-socket-orifice-113c.md`](./ui/d2client-useitem-imbue-socket-orifice-113c.md) | `D2Client.dll` `0x6fb4f400..0x6fb50480` (`UI\useitem.cpp`) | 单格放入窗口模块：模式 0 为赫拉迪姆杖孔洞 (`horadricback.dc6` -> `C2S 0x44`)，模式 1 按交互 NPC 分发恰西注入 (`154`)、拉苏克打孔 (`511`) 与安雅个性化刻名 (`512`) (`upgrade.dc6` -> `C2S 0x38`)。 |
| [`ui/d2client-vendor-stash-gold-display.md`](./ui/d2client-vendor-stash-gold-display.md) | `D2Client.dll` `0x6FB3CFB0` (`HUD_DrawPanelGoldBox`) | 储藏箱 (`nMode=0`)、物品栏 (`nMode=1`) 与商人商店 (`nMode=2`) 底部凹槽金币框绘制规范，涵盖 `STAT_GOLDBANK` 与 `STAT_GOLD` 读取、右对齐排版及商店模式下跳过金币按钮渲染。 |
| [`ui/d2game-player-busy-gate-113c.md`](./ui/d2game-player-busy-gate-113c.md) | `D2Game.dll` `PLAYER_IsBusy`, `D2Client.dll` `0x6fb5a380` | 服务端 `PLAYER_IsBusy` 门控原理：小站 (`0x49 level=0`)、储物箱 (`0x4F 0x12`)、方块 (`0x4F 0x17`)、NPC (`0x30`) 与交易 (`0x4F 0x02`) 关闭时必须发送对应释放包，否则后续地面拾取与祭坛/宝箱交互将被静默拒绝 (Issue #920)。 |
| [`ui/d2multi-join-game-panel-113c.md`](./ui/d2multi-join-game-panel-113c.md) | `D2Multi.dll` `0x6F9E3280`, `D2MCPClient.dll` `0x6FA26280` | 多人游戏大厅加入面板 (`joingamebckg.dc6` 373×373 拼接)、控件描述符表 (`0x6F9EA510`)、`MCP_GAMEINFO (0x06)` 二进制通信报文解析与右侧详情列表格式化规范。 |

---

## 5. 技能与弹道系统 (`skills/`, `missile-velocity-range-113c.md`)

### 5.1 分发总表与核心机制
- [`skills/d2-skills-missiles-dispatch-tables.md`](./skills/d2-skills-missiles-dispatch-tables.md): 全职业与怪物技能分发函数表 (`gpSkillSrvStartFnTable`, `gpSkillSrvDoFnTable`, `g_apfnSkillCltStFunc`, `g_apfnSkillCltDoFunc`, `g_apfnMissileCltDoFunc`, `g_apfnMissileCltHitFunc`) 及普通攻击、踢击、投掷、解除召唤、辨识/传送门书卷等通用玩家动作。
- [`skills/d2-skill-hotkeys-selectskill-respec-113c.md`](./skills/d2-skill-hotkeys-selectskill-respec-113c.md): 技能快捷键 (`0x51`/`0x7B`)、服务端 `SelectSkill (0x3C)` 校验 (`0x6FCEF480` 要求 `pSkill != NULL` 且 `slvl > 0` 或有效聚气)、`SKILLS_SetSkillLevel` (`#10302`) 等级归零摘除链表节点、阿卡拉与赦免徽章 (`toa`) 洗点重置 (`0x6FCC1280`)、以及客户端 `IsSkillHotkeySlotSelectable` (`0x6FAED110`) 与 `PruneZeroLevelSkillHotkeys` (`0x6FAED320`) 门控规范 (Issue #1490)。
- [`skills/aura-srvdo-stat-routing-113c.md`](./skills/aura-srvdo-stat-routing-113c.md): 服务端光环函数 (`SrvDo065` 防御/增益光环 vs `SrvDo066`/`SrvDo081` 攻击光环) 施法者 `passivestat` 与目标 `aurastat` 严格分离路由规范 (Issue #1409)。
- [`skills/aura-target-filter-113c.md`](./skills/aura-target-filter-113c.md): 光环径向扫描 (`0x6FCC0C70`)：`aurarangecalc` 子格欧氏距离平方判定与 `aurafilter` 掩码位（玩家/怪物/亡灵/友军/敌军/城镇禁扫/视线遮挡）过滤规范 (Issue #1410)。
- [`skills/client-state-ids-113c.md`](./skills/client-state-ids-113c.md): 客户端技能预测使用的真实 `States.txt` 行号 (`blaze 13`、`conviction 28`、`redemption 50`、`stunned 21`、`cold 11`、`progressive_* 122..127`) 与持续帧计算 (Issues #1045–#1049)。
- [`skills/d2client-cltdofunc25-ring-113c.md`](./skills/d2client-cltdofunc25-ring-113c.md): `SkillCltDoFunc 25` (`0x6FB23CB0`) 64 向圆环弹道生成（霜之新星、新星、剧毒新星、战吼家族、暗黑火环等）。
- [`skills/missile-clthit31-icebreak-melt.md`](./skills/missile-clthit31-icebreak-melt.md): 冰冻碎裂状态 (`freeze=1`, `shatter=107`, `uberminion=184`) 冰片飞溅与 `MissCltHitFunc[31]` (`0x6FB605B0`) 落地生成 `icebreaksmallmelt`/`icebreaklargemelt` 融水动画规范 (Issue #1453)。
- [`skills/player-srvdo-missile-volleys-113c.md`](./skills/player-srvdo-missile-volleys-113c.md): 玩家 `srvdofunc` 多弹道齐射与特殊发射器（多重箭/牙/冲击波扇形步进、炮轰索敌、充能弹、火焰风暴/小旋风/龙卷风、骨墙、天堂之拳、火墙/陨石、刀刃之怒等，Issue #1417）。
- [`skills/skill-damage-pct-calc-113c.md`](./skills/skill-damage-pct-calc-113c.md): `SrcDam` 武器攻击技能伤害加成百分比 (`dwEnDmgPct`) 按各 `srvstfunc`/`srvdofunc` 精确选择 `calc1`、`calc2`（炮轰/急刺/白热/狂怒）、`calc4`（牺牲）或协同公式（复仇/狂乱/专心/狂暴）规范 (Issue #1418)。
- [`missile-velocity-range-113c.md`](./missile-velocity-range-113c.md): 投射物初速 `Vel + lvl * VelLev / 8`、`8.16` 定点截断步进、`Accel` 加速度、减速箭 (`slowmissiles` stat 179) 缩放、`Range + lvl * LevRange` 帧寿命与 `SubLoop` 循环帧校准 (Issue #1421)。

### 5.2 弹道与特效机制 (`skills/missiles/`)
- [`skills/missiles/d2client-meteor-cltdofunc-clthitfunc.md`](./skills/missiles/d2client-meteor-cltdofunc-clthitfunc.md): 巫师陨石技能的客户端 3D 轨迹计算 (`CltDoFunc=9`, $vx=15, vz=-25$)、等轴测投影、撞击闪光、屏幕震颤与 18 块地面火分散生成机制。
- [`skills/missiles/d2game-srvdo-06-23-30-druid-113c.md`](./skills/missiles/d2game-srvdo-06-23-30-druid-113c.md): 服务端弹道 `SrvDo 6` 与 `23..30`（熔岩巨石/火焰风暴尾迹、火山裂口、藤蔓、龙卷风周期范围伤害、火山喷发与狂犬病传染归属，Issues #1224–#1228）。
- [`skills/missiles/d2game-srvhit-radius-113c.md`](./skills/missiles/d2game-srvhit-radius-113c.md): 服务端弹道 `gpMissileSrvHitFnTable` (`0x6FD2DA48`, 59 项 `pSrvHitFunc`) 命中爆炸/溅射/弹跳半径解析（`sHitPar1`、`dHitPar1` 与 `Skills.txt` `aurarangecalc`/`calc1`/`calc2` 等轴测地面半径，Issue #1457）。
- [`skills/missiles/missile-collision-113c.md`](./skills/missiles/missile-collision-113c.md): `g_aMissileCollideTypes[9]` (`0x6FD2D928`) 碰撞掩码表、敌我单位过滤回调 (`0x6FC58FB0`) 与 `Size` 占地足迹重叠检测 (`0x6FDCF6A0`, Issue #1422)。
- [`skills/missiles/missile-pierce-113c.md`](./skills/missiles/missile-pierce-113c.md): 弹道穿透 `pierce_idx` (`stat 328`)：弹道创建期最多 4 次 `SEED_Rand % 100 < item_pierce + skill_pierce` 掷骰 (`0x6FC8F660`) 与命中单位时的递减消耗 (`0x6FC58F40`, Issue #1420)。

### 5.3 怪物与 Boss 技能 (`skills/bosses/`)
- [`skills/bosses/skills-boss-act1-act2-act3.md`](./skills/bosses/skills-boss-act1-act2-act3.md): 第一幕至第三幕关卡 Boss 与超级暗金怪技能 (安达利尔、血乌、女伯爵、格瑞斯华尔德、母牛之王、拉达蒙特、钻地虫、召唤者、督瑞尔、议会成员、墨菲斯托)。
- [`skills/bosses/skills-boss-act4-diablo.md`](./skills/bosses/skills-boss-act4-diablo.md): 第四幕 Boss 与混沌避难所封印怪 (衣卒尔、海法斯特、暗黑破坏神、暗黑克隆体)。
- [`skills/bosses/skills-boss-act5-baal-ubers.md`](./skills/bosses/skills-boss-act5-baal-ubers.md): 第五幕关卡 Boss 与地狱火事件三魔神 (山克、尼拉塞克、古代人、巴尔、超级墨菲斯托、超级暗黑破坏神、超级巴尔、莉莉丝、超级督瑞尔、超级衣卒尔)。

### 5.4 玩家职业技能树 (`skills/player/`, 21 份文档 / 210 项技能)
- **亚马逊 (Amazon)**:
  - [`skills/player/skills-ama-bow-crossbow.md`](./skills/player/skills-ama-bow-crossbow.md) — 弓与十字弓技能 (Magic Arrow, Multiple Shot, Guided Arrow, Strafe 等)
  - [`skills/player/skills-ama-passive-magic.md`](./skills/player/skills-ama-passive-magic.md) — 被动与魔法技能 (Inner Sight, Dodge, Avoid, Evade, Valkyrie, Pierce 等)
  - [`skills/player/skills-ama-javelin-spear.md`](./skills/player/skills-ama-javelin-spear.md) — 标枪与长矛技能 (Jab, Power Strike, Charged Strike, Lightning Fury 等)
- **刺客 (Assassin)**:
  - [`skills/player/skills-ass-martial-arts.md`](./skills/player/skills-ass-martial-arts.md) — 武术技能 (Tiger Strike, Dragon Talon, Phoenix Strike 等)
  - [`skills/player/skills-ass-shadow-disciplines.md`](./skills/player/skills-ass-shadow-disciplines.md) — 暗影修行 (Burst of Speed, Fade, Weapon Block, Shadow Master 等)
  - [`skills/player/skills-ass-traps.md`](./skills/player/skills-ass-traps.md) — 陷阱技能 (Fire Blast, Lightning Sentry, Death Sentry 等)
- **野蛮人 (Barbarian)**:
  - [`skills/player/skills-bar-combat.md`](./skills/player/skills-bar-combat.md) — 战斗技能 (Bash, Leap Attack, Concentrate, Frenzy, Whirlwind 等)
  - [`skills/player/skills-bar-masteries.md`](./skills/player/skills-bar-masteries.md) — 战斗专家 (Sword Mastery, Iron Skin, Natural Resistance 等被动机制)
  - [`skills/player/skills-bar-warcries.md`](./skills/player/skills-bar-warcries.md) — 战吼技能 (Howl, Shout, Battle Orders, Battle Command, War Cry 等)
- **德鲁伊 (Druid)**:
  - [`skills/player/skills-dru-elemental.md`](./skills/player/skills-dru-elemental.md) — 元素技能 (Firestorm, Fissure, Tornado, Hurricane, Armageddon 等)
  - [`skills/player/skills-dru-shapeshifting.md`](./skills/player/skills-dru-shapeshifting.md) — 变形技能 (Werewolf, Werebear, Feral Rage, Fury 等)
  - [`skills/player/skills-dru-summoning.md`](./skills/player/skills-dru-summoning.md) — 召唤技能 (Raven, Oak Sage, Spirit Wolf, Summon Grizzly 等)
- **死灵法师 (Necromancer)**:
  - [`skills/player/skills-nec-curses.md`](./skills/player/skills-nec-curses.md) — 诅咒技能 (Amplify Damage, Iron Maiden, Decrepify, Lower Resist 等)
  - [`skills/player/skills-nec-poison-bone.md`](./skills/player/skills-nec-poison-bone.md) — 毒素与白骨技能 (Teeth, Corpse Explosion, Bone Spear, Bone Spirit, Poison Nova 等)
  - [`skills/player/skills-nec-summoning.md`](./skills/player/skills-nec-summoning.md) — 召唤技能 (Raise Skeleton, Clay Golem, Iron Golem, Revive 等)
- **圣骑士 (Paladin)**:
  - [`skills/player/skills-pal-combat.md`](./skills/player/skills-pal-combat.md) — 作战技能 (Smite, Charge, Blessed Hammer, Holy Shield, Zeal 等)
  - [`skills/player/skills-pal-defensive-auras.md`](./skills/player/skills-pal-defensive-auras.md) — 防御灵气 (Prayer, Cleansing, Defiance, Vigor, Salvation 等)
  - [`skills/player/skills-pal-offensive-auras.md`](./skills/player/skills-pal-offensive-auras.md) — 攻击灵气 (Might, Holy Fire, Thorns, Blessed Aim, Fanaticism, Conviction 等)
- **巫师 (Sorceress)**:
  - [`skills/player/skills-sor-cold.md`](./skills/player/skills-sor-cold.md) — 冰冻法术 (Ice Bolt, Frost Nova, Blizzard, Frozen Orb, Cold Mastery 等)
  - [`skills/player/skills-sor-fire.md`](./skills/player/skills-sor-fire.md) — 火焰法术 (Fire Bolt, Warmth, Fire Ball, Meteor, Hydra, Fire Mastery 等)
  - [`skills/player/skills-sor-lightning.md`](./skills/player/skills-sor-lightning.md) — 闪电法术 (Charged Bolt, Static Field, Teleport, Nova, Chain Lightning, Lightning Mastery 等)

---

## 6. 渲染与光照 (`render/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`render/d2client-tile-layer-order-113c.md`](./render/d2client-tile-layer-order-113c.md) | `D2Client.dll` `0x6fb2a72e`, `0x6fb3b110`, `0x6fb29f20`; `D2Common.dll` `#10916` | 房间墙壁按 `nTileType` 分流：`16..19` (`*_WALL_DOWN` 下层悬崖壁) 在地板层**之前**绘制（消除 Act 4 高原红石柱穿模），`15` 为屋顶，其余（含 `10/11` 门框）为上层墙；上层墙遮挡玩家淡出与 POPS 屋顶/建筑区域揭开规范 (Issues #919, #1009)。 |
| [`render/d2client-unit-light-records-113c.md`](./render/d2client-unit-light-records-113c.md) | `D2Client.dll` `0x6fac92f5`, `0x6fac8bc0`, `0x6fac89a0`, `0x6fad0510`, `0x6fb63647`, `0x6fb1b360`, `0x6fb1b49f`, `0x6fb23410` | 单位光源记录与状态叠加层：玩家半径 13 + stat 89 (≤0 不变)、stat 90 颜色；怪物 MonStats2 光源与 MonUMod 3 (半径 7，Champion 随机 B/G/R)；弹道光仅在 Light Quality 高时创建、Flicker 与结束淡出；叠加层 `States.txt`/`states.bin` (`wOverlay[0..3]` + `pgsvoverlay` 3 行展开) 映射至 `overlay.bin` 绘制参数与 `InitRadius → Radius` 光源 (Issue #830, #878)。光照网格管线见 [`docs/lighting_grid_113c.md`](../docs/lighting_grid_113c.md) |
| [`render/d2gfx-draw-mode-113c.md`](./render/d2gfx-draw-mode-113c.md) | `D2Client.dll` `0x6FB1B11D`, `0x6FB1B2B2`, `0x6FB1CF32`; `D2gfx.dll` `#10041` (`0x6FA8B080`); `D2Direct3D.dll` `0x6F886B90` | Overlay.txt `Trans` 原值即 eDrawMode (恒等)，Missiles.txt `Trans` 0/1/2 → 5/3/4；各 eDrawMode 的 D3D 混合状态 (0-2 α25/50/75、3 加法 ONE/ONE、4 乘法 ZERO/SRCCOLOR、5/6/7/≥8 不透明)；烘焙不做黑色抠像 (Issue #1050-#1055) |
| [`render/d2client-drawmode-113c.md`](./render/d2client-drawmode-113c.md) | `D2Client.dll` `0x6FB633F7`, `0x6FB23CB0`, `0x6FB50DB0`, `0x6FB22C80`, `0x6FB23760`; `D2Direct3D.dll` `0x6F886CF4`; `D2Gdi.dll` `0x6F87633C` | eDrawMode 汇总 (Overlay Trans 恒等、Missile Trans 0/1/2→5/3/4、3=加法 ONE/ONE、4=乘法 ZERO/SRCCOLOR)；客户端 missile 视觉帧速 = `AnimSpeed<<4` 覆盖 (`animrate` 仅服务端)；CltDoFunc 25 客户端自生成 64 发半径 30 的环 (Range+LevRange*lvl)；States.txt castoverlay 在 state 设置时以 LOOPEX(4)/AURAEX(9) 播放后交接 overlay1/2 (Issue #1057) |
| [`render/dt1-material-flags-animated-113c.md`](./render/dt1-material-flags-animated-113c.md) | `D2Common.dll` `0x6fd62f20`, `0x6fd62ff0`; `D2CMP.dll` `#10051` (`0x6fe25300`) | DT1 `+0x06` 为 u16 材质标志 (byte +7 是其高字节，非独立 animated 字节)；仅 `TILE_FLAGS_LAVA` (`0x100`) 动画，Act 5 雪地 `0x400` 为普通随机变体地块 (Issue #923) |

---

## 7. 音效表与声音记录层 (`audio/`)

| 文档 | 核心逆向地址 / 模块 | 关键机制与规范 |
|---|---|---|
| [`audio/d2gs-0x2c-playsound-113c.md`](./audio/d2gs-0x2c-playsound-113c.md) | `D2Client.dll` `0x6fb5d740` (`0x2C`), `0x6fb5d760` (`0xA4`), `0x6fad3f20` (外层 `10..93`), `0x6fad3200` (玩家 `1..83`), `.rdata:0x6fba7694` | `S2C 0x2C PlaySound` 的 `u16` 字段为声音事件码而非 `Sounds.txt` 行号（如 `10 -> 2673 shrine_refill` vol 120，`13 -> 2634 object_trap_trigger`，`14 -> 2635 object_trap_release`，`93 -> 2553 object_corpse_loot`，以及 7 职业 22 槽位语音表）；`S2C 0xA4 MonsterSpecialSkill` 不播放任何声音 (Issue #944) |
| [`audio/sound-records-113c.md`](./audio/sound-records-113c.md) | `D2Client.dll` `0x6fb0e8e1` (表载入后处理), `0x6fb38a70` (PlaySound), `0x6fb38dd0` (SoundUpdate), `0x6fb0d7a0` (StartSound), `0x6fb0ce70` (AllocChannel), `0x6fb0d1d0` (UpdateChannel), `0x6fb0ee70` (变体选取); `D2Common.dll` `#10628`; `D2sound.dll` `#10047/#10008/#10028` | `Sounds.txt` 4699 行 groupBase/Group Size/nBlocks 后处理；200 条声音记录池、Compound 复用、本地玩家 +80 字节加法、屏幕坐标 `(Δcx, 2·Δcy, 640)`；同 id 2 tick 去抖比较计划开始时刻；16 通道 = 4 立体声 + 12 单声道分区抢占；淡入启动失败的循环记录 vol 0 永不重启等二进制怪癖 (Issue #943) |
| [`progression/d2game-levelup-vitals-gold-113c.md`](./progression/d2game-levelup-vitals-gold-113c.md) | `D2Game.dll` `0x6FD0B090` (level-up), `0x6FCEC960`/`0x6FD0AEE0`/`0x6FD0AD10`/`0x6FD0AE20` (stat alloc), `0x6FC68BA0` (MERCS_SetStats); `D2Common.dll` `#10049`, `#11060`, `#10913` | Level-up adds CharStats per-level gains in 1/256 fixed point and fully refills life (if alive)/mana/stamina to the accrued max; stat points refill nothing (incremental `+LPV<<6` to current); merc per-level stats (Str/Dex/Dmg `/8`, Resist `/4`, HP/Def/AR full, HP>=40); inventory gold `clvl*10000`, stash gold constant 2,500,000. |
| [`network/d2gs-c2s-dispatch-and-sizes-113c.md`](./network/d2gs-c2s-dispatch-and-sizes-113c.md) | `D2Net.dll` `0x6FBF6750` / `0x6FBF65D0` (var-len resolvers), `0x6FBFA900` / `0x6FBFABD8` (size tables); `D2Game.dll` `0x6FCEC550` / `0x6FD1A078` (C2S dispatch) | D2Net S2C/C2S size tables and variable-length resolvers (min-buffer per case, C2S `0x6C = u8@1+7`, `0x39` fixed 5); D2Game C2S handler table with per-handler size checks, always-reject handlers (`0x2C/0x2D/0x39/0x45/0x52`) and no-ops; S2C `0x22` playerbody skip byte `@0xB`, `0x73` missile `+0x10` word and stat `0x148`, merc `0x9E..0xA2` set/add split. |
| [`items/d2common-item-bitstream-props-reqs-113c.md`](./items/d2common-item-bitstream-props-reqs-113c.md) | `D2Common.dll` `0x6fd7a600`/`0x6fd7c0c0` (decode), `0x6fd79d90` (generic stat read), `0x6fd7813e` (stat-list encode), `0x6fdeb0cc` (Properties func table), `#11151` `0x6fd74610`, `#10244` `0x6fd76db0`, `0x6fd76190` | Stat values `(raw-SaveAdd)<<ValShift` (maxhp/maxmana/maxstamina/hp,mana per level are 8.8); encoder skips 0-bit/zero stats, always emits chained followers (17/48/50/52/54/57) and saturates values; func 7 writes 17+18, func 5 writes 21/23/159; compact ears read the save realm bit; socket cap by ItemTypes MaxSock tiers and inv size; signed MulDiv str/dex `item_req_percent` with -10 ethereal; crafted level req = maxAffix+10+3n then max with base. |
| [`combat/d2game-tohit-defense-anim-rate-regen-113c.md`](./combat/d2game-tohit-defense-anim-rate-regen-113c.md) | `D2Common.dll` `#10621` (`0x6FD81EA0`), `#10672` (`0x6FD82890`), `0x6FD83180..0x6FD835E0`, `0x6FD823E0`; `D2Game.dll` `0x6FCFDE90`, `0x6FCFB1F0`, `0x6FCFC530`, `0x6FCFAC10`, `0x6FCFC0B0`/`0x6FCFBD00`, `0x6FC97950`/`0x6FC97A50`/`0x6FC97CB0` | AR = tohit+5*dex-35+ToHitFactor (no %); hit-chance single additive %AR pool (119 + mastery + skill + vs-montype) plus ignore/fractional target AC, demon/undead tohit, def vs missile/melee; defense dex/4 + (171+16)% + armor_override%; single additive physical ED pool clamped -90; PvP 17%/merc 25% damage modifier; DR/MDR per channel before resist; FHR/FBR rate base 50 (Holy Shield FBR 100); stamina/mana/life per-frame regen rules (no base life regen). |
| [`skills/d2common-skillcalc-callbacks-113c.md`](./skills/d2common-skillcalc-callbacks-113c.md) | `Fog.dll` `#10253` (`0x6FF69E90`); `D2Common.dll` `0x6FDA1070` (SkillCalc), `0x6FDBA790` (MissCalc), `0x6FD9DC30` (dm), `#10121`/`#11091` (elem dmg), `0x6FD9FCB0` (skill bonus); `D2Game.dll` `0x6FCBF850` (ConsumeMana) | Calc bytecode opcodes (`/0` -> 0, `^` exp<=0 -> 1); all 73 SkillCalc / 43 MissCalc codes; dm = `trunc(110*lvl/(lvl+6))*(b-a)/100+a` capped at `b`; edmn/edmx always synergised (min skipped when `<=256 && EMinLev1==0`), enma/exma add elemental mastery stat 329..332; mps = `M*25/2<<shift>>8`; math/madm/macr = mastery passivecalc; oskill own-class cap 3, `item_elemskill` 126, clamp 99; mana 0/0 = no cost. |
| [`world/d2common-act-spawn-location-raytrace-113c.md`](./world/d2common-act-spawn-location-raytrace-113c.md) | `D2Common.dll` `#10816` (`0x6FD8CDF0`), `#10942`, `0x6FD6A130`, `0x6FD69B60`, `0x6FDE4F48`, `0x6FD5C45F`, `0x6FD9AD20` (`#10864`/`#10479`), act link tables `0x6FDDCBE8..0x6FDDD278` | Spawn location: Levels.txt `Position` levels pick a seeded style-30..33 special tile (`pTileInfo`, index 0 = entry, 11 = town portal) then tile*5+3 + free-coordinate search (mask `0x1C09`, r=50); other levels use waypoint -> warp -> centre. `COLLISION_RayTrace` is begin-inclusive major-axis Bresenham with 0-start accumulator; room exit = hit. |
| [`audio/d2client-soundenviron-music-areaspeech-113c.md`](./audio/d2client-soundenviron-music-areaspeech-113c.md) | `D2Client.dll` `0x6faff860` (SoundEnvironTick), `0x6fb38810`/`0x6fb38770` (range stops), `0x6fb30450`/`0x6faff770` (rain gate/intensity), `0x6fb30648` (thunder), `0x6faf5660` (music), `0x6fad3e20` + `.data:0x6fba76b0` (area-entry speech); D2Common `#10396`, `#10215` | Day/night = env cycle index 1..3 (sunrise/sunset count as night); swap crossfade 250 ticks both ways (new ambience vol 0 → 255); other changes StopWithFade ambience ids 52..71 / event ids 72..201; rain gated by Levels.txt Rain, vol = intensity·255, ±6/tick; event z = 640; thunder delay 25+rand(50) at ±200 offset; music 75-tick room debounce, per-song resume at next Block marker every 125 ticks; 14 first-visit class monologues. |
| [`network/d2net-mcp-ping-storm-comp-113c.md`](./network/d2net-mcp-ping-storm-comp-113c.md) | `D2Net.dll` `0x6FBF6D60`/`0x6FBF7290`; `D2Client.dll` `0x6FAC4B00` (0x6D), `0x6FAF22E0` (0x8F), `0x6FAC4A50` (0x68); `D2MCPClient.dll` `0x6FA2A900` (S2C table), `0x6FA26250` (0x0A); `D2Multi.dll` `0x6F9E33ED`; `Storm.dll` `0x6FC12784`/`0x6FC36814`; `D2Win.dll` `0x6F8EA9A6` | D2Net stamps pong `@0x0D` with GetTickCount; ping every 5000 ms with `@5 = avgPing/2` (10-sample mean); 0x68 logon constants `0xED5DCC50`/`0x2185EDD6` + `0x91A519B6`; full MCP S2C handler table (0x0A = u16 req + u32 status, JOINGAME success iff IP != 0); GAMEINFO classes 0-based (>6 = no class); Storm decompress masks 0x08->0x01->0x80->0x40 only; MPQ priorities 1000/3000/5000. |
| [`ui/d2client-control-panel-hud-geometry-113c.md`](./ui/d2client-control-panel-hud-geometry-113c.md) | `D2Client.dll` `0x6FAD7290` (ctrl panel), `0x6FAD6970` (exp bar), `0x6FAD78B0` (stamina), `0x6FAD6B20` (vital smoothing), `0x6FAD8320`/`0x6FAD6880` (run/menu), `0x6FB13000`/`0x6FB14990` (mini panel), `0x6FAD8D30` (skill buttons), `0x6FAD8E60` (pop-belt) | Bottom control-panel HUD geometry & math at 800x600 / 640x480: frames drawn, exp-bar 119-px two-line fill, stamina yellow/red/blue rules, smoothed life/mana/stamina display, run/menu/mini-panel/skill-button/pop-belt positions, and mini-panel shift/hide when side panels are open. |
| [`ui/d2win-d2lang-fonts-strings-113c.md`](./ui/d2win-d2lang-fonts-strings-113c.md) | `D2Lang.dll` `0x6FC09360`, `0x6FC0A7B0`, `0x6FC09450`, `0x6FC09050`; `D2Win.dll` `0x6F8F2FE0`, `0x6F8F3110`, `0x6F8F2800`; `D2Launch.dll` `0x6FA60C08`; `D2Multi.dll` `0x6F9E5182` | String tbl lookup (patch 10000+ > expansion 20000+ > string, out-of-range idx -> entry 500, missing key -> "-not xlated" text), 14 font ids + D2Client usage (quest log = FontFormal11/FontExocet10), tbl glyph records, locale line metrics (truncating /10), colour codes, campfire overlay table, name filter, difficulty gate. |
| [`network/d2client-s2c-unit-queue-and-c2s-interact-113c.md`](./network/d2client-s2c-unit-queue-and-c2s-interact-113c.md) | `D2Client.dll` `0x6FB8DE60` (S2C table), `0x6FB5CE20`/`0x6FB5C630`/`0x6FB5BC20` (per-unit queue), `0x6FADAC10` (interact/pickup), `0x6FAC86A0` (walk adjust), `0x6FB5ECA0`/`0x6FB5EBE0`, `0x6FB5CA40`, `0x6FADB1E0` | S2C table `{handler,size,unitCallback}`; unit-callback opcodes (0x0C..0x10, 0x4C, 0x4D, 0x67..0x72) are queued on `unit+0xD8` and drained after the unit update; interact sender per unit type (monster 0x13 200 ms throttle, item 0x16 toCursor = inventory open, Compelling Orb needs wielded `qf2` / Hellforge mode 2 needs `hfh` else "impossible" sound and no packet); walk min-step 3/4/5 + 7-frame hold gate; 0x18/0x95 revive only from mode 17 -> mode 5; own-merc 0x0A ignored; 0x4B never for self. |
| [`render/d2win-pl2-layout-state-colorshift-113c.md`](./render/d2win-pl2-layout-state-colorshift-113c.md) | `D2Win.dll` `0x6F8EEF20` (pl2 loader), `#10164` (`0x6F8EEE50`); `D2Client.dll` `0x6FB22130`, `0x6FB1C5F8`, `0x6FB1C6EC` | PL2 file offsets copied by D2Win (128-table contiguous hue block at `0x53500`); States.txt colorshift N → `hueVariations[N-1]` (highest colorpri, first max wins), overrides the monster colormap; poison (104) suppressed on the local player in 3D modes; state light-r/g/b becomes the local player's light colour (else white). |
| [`render/d2cmp-direction-tables-113c.md`](./render/d2cmp-direction-tables-113c.md) | `D2CMP.dll` `0x6FE2E7D8..0x6FE2EBD8` (DCC), `0x6FE2F7D8..0x6FE2FBD8` (COF) | 64-direction → DCC (permuted) and COF (sequential) direction tables for 4/8/16/32/64 directions; port tables match all 640 entries. |
| [`render/d2gfx-gamma-contrast-113c.md`](./render/d2gfx-gamma-contrast-113c.md) | `D2gfx.dll` `0x6FA8AEE0`; `D2Direct3D.dll` `0x6F88A640`, `0x6F88AE00`; `Fog.dll` `#10198` (`0x6FF712C0`) | Registry Gamma (default 155) / Contrast (default 100, 0 in render mode 4) options; D3D builds a 256-entry 16-bit gamma ramp via Fog #10198 and SetGammaRamp. |
| [`render/d2client-hittest-overlay-height-shake-113c.md`](./render/d2client-hittest-overlay-height-shake-113c.md) | `D2Client.dll` `0x6FB568E0`, `0x6FB1A550`, `0x6FB01E80`, `0x6FB1B300`, `0x6FB1AA30`, `0x6FB39F00`, `0x6FB3AFD0`, `0x6FB3AD30`, `0x6FB641F0`, `0x6FB6C210`; `D2Common.dll` `#10172` (`0x6FD80830`) | Unit hover hit-test per type (players/monsters = COF header box ±16 px, noGfxHitTest PtInRect, no radial fallback); overlay y = yOffset + Height[#10172] (player→Height2, monster→OverlayHeight-1, -1→+75); shadow skipped by unit flag 0x20 (MonStats2 Shadow=0) or state 146; complete screen-shake writer list + quake loop sound 452; Duriel death CltDo 36 schedule table. |
