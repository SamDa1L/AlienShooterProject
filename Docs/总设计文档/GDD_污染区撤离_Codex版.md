# 《代号：污染区撤离》Codex 开发版 V1.0（修正版）

> 基准文档：`污染区撤离_游戏策划案_V2.4.md`  
> 修正目标：将原 Codex 版重新对齐 V2.4 的产品基准，同时保留适合 Codex 辅助开发的技术路线、模块拆分、任务拆解和 Prompt 模板。  
> 项目类型：俯视角 / 等距视角 / 单机 PVE / 射击搜打撤 / 怪潮清屏 / 枪械改装 / 人类据点 / 轻量生存模拟 / 人物成长  
> 目标平台：PC / Steam  
> 推荐引擎：Unity 3D URP  
> 文档用途：立项执行、制作人拆分任务、程序架构设计、Codex 辅助开发  
> 当前版本：V1.0 修正版  
> 日期：2026-06-12

---

## 0. 修正结论

本修正版 Codex 文档必须服务于 V2.4 的核心方案，而不是重新发明一个新的工业生化搜打撤项目。

本项目应做成：

**单机 PVE 俯视角 / 等距视角射击搜打撤游戏。玩家从地下防空洞基地出发，进入郊区农场、旧工业工厂和封锁小城镇，在怪潮、人类据点、资源争夺和轻量生存压力中搜刮、战斗、撤离、改枪、成长。**

与旧 Codex 版相比，本修正版做出以下关键修正：

1. 首发地图从“废弃仓库区、实验室、下水道、处理厂”等泛工业地图，修正为 V2.4 指定的三张连续地图：**郊区农场、旧工业工厂、封锁小城镇**。
2. MVP 从“仓库区搜箱子打怪撤离”修正为：**防空洞基地 → 检查饥饿口渴疲劳 → 进入郊区农场 → 清怪 → 攻打农场掠夺者据点 → 搜刮食物水和装备 → 局内使用补给 → 撤离 → 回基地结算 → 人物成长 → 改枪 → 休息补给 → 再次出发**。
3. 将**饥饿、口渴、睡眠 / 疲劳**纳入 MVP 和首发核心开发范围。
4. 将**局内物品使用系统**纳入 MVP：食物、水、药品、能量饮料、咖啡因药片等必须可以在局内使用。
5. 将**人物成长系统**纳入首发核心：等级、五大属性、武器熟练度、轻量被动技能。
6. 将**人类据点与三方冲突**纳入核心差异化卖点，而不是只做怪物尸潮。
7. 保留 Unity 3D URP、XZ 平面移动、正交斜俯视相机、3D Collider、NavMesh、ScriptableObject、本地存档等技术路线。
8. Codex 任务必须以“小任务 + 清晰验收标准”的方式拆分，不允许一次性让 Codex 做完整游戏。

---

## 1. 项目基本信息

### 1.1 项目代号

中文代号：**污染区撤离**  
英文暂名：**Contamination Zone: Extraction**  
内部代号：**Project CZ-Evac**

### 1.2 游戏类型

- 俯视角 / 等距视角射击
- 单机 PVE
- 搜打撤
- Horde Shooter / 怪潮射击
- 枪械客制化
- 轻量生存模拟
- 人物成长
- 人类据点攻坚
- 基地经营与仓库积累

### 1.3 平台与商业模式

| 项目 | 方案 |
|---|---|
| 目标平台 | PC / Steam |
| 商业模式 | 单机买断制 |
| 首发形态 | 单机 PVE 完整版，必要时可走 EA |
| 目标预算 | 约 50 万人民币 |
| 开发模式 | 小团队独立开发 |
| 不建议 | 多人 PVP、开放世界、大型随机地图、真实军事模拟、赛季经济、联网市场 |

### 1.4 一句话定位

玩家从地下防空洞基地出发，穿越郊区农场、旧工业工厂和封锁小城镇，在异形怪潮、人类掠夺者、感染巢穴和末日资源争夺中搜刮、战斗、改枪、处理饥饿口渴疲劳状态、提升角色属性，并逐步打开通往城市、军事基地和机场的高危路线。

### 1.5 三个核心情绪

```text
爽：大量怪物被高火力撕开，血雾、血迹、爆炸、击退反馈直接。
贪：背包里越多好东西，越想多搜一个房间、一个箱子、一个据点。
慌：警戒值越高，怪潮越强，人类敌人越危险，撤离变成最后高潮。
```

---

## 2. 项目边界

### 2.1 首发必须做

1. 单机 PVE 俯视角 / 等距视角射击。
2. Unity 3D URP 项目。
3. 正交斜俯视相机。
4. XZ 平面移动，Y 轴作为高度。
5. 怪潮射击与强血腥反馈。
6. 搜刮、背包、仓库、撤离结算。
7. 三张连续地图：郊区农场、旧工业工厂、封锁小城镇。
8. 地下防空洞基地。
9. 人类据点攻坚。
10. 玩家、人类、感染怪物三方冲突。
11. 枪械改装。
12. 警戒值 / 污染警戒值系统。
13. 饥饿、口渴、睡眠 / 疲劳。
14. 局内可使用食物、水、药物、饮料等物品。
15. 人物等级、属性成长、武器熟练度、轻量被动技能。
16. 基地升级。
17. 本地存档。
18. Steam Demo / 商店页素材。

### 2.2 首发禁止做

以下内容不进入 50 万首发版本：

1. 多人联机。
2. 多人 PVP / PVPVE。
3. 开放世界。
4. 大型随机生成地图。
5. 复杂 NPC 社交。
6. 玩家交易市场。
7. 服务器仓库持久化。
8. 反作弊系统。
9. 复杂温度、疾病、精神值、生理需求等大生存模拟。
10. 复杂自由建造。
11. 大型家园系统。
12. 长篇剧情 CG。
13. 上百把枪和上百配件。
14. 高成本程序化肢解系统。

### 2.3 可延期内容

1. 双人合作。
2. 每日挑战。
3. 排行榜。
4. 创意工坊。
5. 地图编辑器。
6. 城市区域。
7. 军事基地。
8. 机场。
9. 家园系统。
10. 更多 Boss 和后期章节。

---

## 3. 技术路线

### 3.1 Unity 模板

Unity Hub 新建项目时选择：

```text
Universal 3D
或
3D (URP)
```

不要选择普通 2D、2D URP 或 HDRP。

原因：本项目虽然是俯视角画面，但底层逻辑更适合 3D：

```text
角色在 XZ 平面移动。
Y 轴表示高度。
墙体、障碍物、敌人、撤离点、掉落物均使用 3D Collider。
敌人寻路使用 NavMesh / AI Navigation。
相机使用 Orthographic 斜俯视。
枪线、爆炸、投射物、遮挡、血迹和尸体都按 3D 场景处理。
```

### 3.2 推荐 Unity 包

| 包 | 用途 | 优先级 |
|---|---|---|
| Universal RP | 渲染、光照、后处理 | 必装 |
| Input System | 键鼠、手柄输入 | 必装 |
| AI Navigation | 怪物和人类敌人寻路 | 必装 |
| TextMeshPro | UI 文本 | 必装 |
| Cinemachine | 相机跟随、震屏、镜头控制 | 推荐 |
| ProBuilder | 灰盒地图搭建 | 推荐 |
| Addressables | 中后期资源管理 | 中后期 |
| Unity Test Framework | EditMode / PlayMode 测试 | 推荐 |

### 3.3 初始相机参数

```text
Projection: Orthographic
Rotation X: 55-60
Rotation Y: 0
Rotation Z: 0
Orthographic Size: 10-14
Follow Target: Player
Movement Plane: XZ
Height Axis: Y
```

### 3.4 项目目录结构

```text
Assets/
  ArtTemplate/
    临时美术资源池，用于导入和筛选大量外部美术资源，不作为正式业务目录，不上传 Git。
  Game/
    Art/
      Characters/
        Player/
      Enemies/
      HumanEnemies/
      Weapons/
      Environments/
        Base_Bunker/
        Map_Farm/
        Map_Factory/
        Map_Town/
      VFX/
        BloodFeedback/
        CombatFeedback/
      UI/
      Icons/
    Audio/
      Weapons/
      Enemies/
      HumanEnemies/
      Survival/
      UI/
      Music/
    Materials/
    Prefabs/
      Player/
      Enemies/
      HumanEnemies/
      Weapons/
      Ammo/
      Loot/
      Consumables/
      Equipment/
      Strongholds/
      Extraction/
      UI/
      VFX/
    Scenes/
      Boot.unity
      MainMenu.unity
      Base_Bunker.unity
      Raid_Farm_01.unity
      Raid_Factory_01.unity
      Raid_Town_01.unity
    Scripts/
      Core/
      GameFlow/
      Player/
      Camera/
      Weapons/
      Ballistics/
      HitDetection/
      Enemies/
      HumanEnemies/
      Spawning/
      Factions/
      Inventory/
      Equipment/
      Hotbar/
      Storage/
      Loot/
      Consumables/
      Survival/
      CharacterGrowth/
      WeaponProficiency/
      Extraction/
      Threat/
      Strongholds/
      Missions/
      Base/
      Maps/
      ObjectPooling/
      BloodFeedback/
      CombatFeedback/
      Save/
      Settings/
      UI/
      Audio/
      DeveloperTools/
    ScriptableObjects/
      GameFlow/
      Weapons/
      Attachments/
      Ammo/
      Enemies/
      HumanEnemies/
      Spawning/
      Loot/
      Consumables/
      Equipment/
      Survival/
      CharacterGrowth/
      Proficiency/
      Missions/
      Maps/
      Strongholds/
      BaseUpgrades/
      ThreatLevels/
      DropTables/
      Difficulty/
      Settings/
    Tests/
      VersionValidation/
        VS_Combat.unity
        FVS_Combat.unity
      UI/
      Combat/
      Weapons/
      Enemies/
      Camera/
      Input/
      Systems/
      Performance/
  ProjectRef/
    参考模板资源，只用于分析和临时验证，不作为正式业务目录，不上传 Git。
  TmpTests/
    临时测试代码、测试脚本和测试辅助工具，不作为正式业务目录，不上传 Git。
```

说明：正式业务代码和正式资源统一放入 `Assets/Game`。`Assets/Game/Scenes` 只放正式场景，不放测试场景，正式场景命名不能出现 `Test`。`Assets/Game/Tests` 只放测试场景和测试场景所需场景级资源，不放测试代码。`Assets/TmpTests` 只放临时测试代码，不作为正式业务目录。`Assets/ProjectRef` 只作为参考资源区，不允许把参考项目命名体系扩散到正式业务代码。`Assets/ArtTemplate` 只作为临时美术资源池，只有确认使用的美术资源才整理后移动到 `Assets/Game/Art` 下对应目录。

### 3.5 命名空间建议

```csharp
CZ.Core
CZ.GameFlow
CZ.Player
CZ.Camera
CZ.Weapons
CZ.Ballistics
CZ.HitDetection
CZ.Enemies
CZ.HumanEnemies
CZ.Spawning
CZ.Factions
CZ.Inventory
CZ.Equipment
CZ.Hotbar
CZ.Storage
CZ.Loot
CZ.Consumables
CZ.Survival
CZ.CharacterGrowth
CZ.WeaponProficiency
CZ.Extraction
CZ.Threat
CZ.Strongholds
CZ.Missions
CZ.Base
CZ.Maps
CZ.ObjectPooling
CZ.BloodFeedback
CZ.CombatFeedback
CZ.Save
CZ.Settings
CZ.UI
CZ.Audio
CZ.DeveloperTools
```

---

## 4. 核心玩法循环

### 4.1 局外循环

```text
地下防空洞基地
-> 查看仓库
-> 查看饥饿 / 口渴 / 疲劳
-> 进食 / 饮水 / 休息 / 使用药物
-> 接取任务
-> 整理武器、弹药、护甲、医疗品、食物、水、工具
-> 改装武器
-> 分配属性点或被动技能
-> 选择地图
-> 出发进入地表战区
```

### 4.2 局内循环

```text
进入地图
-> 探索区域
-> 搜刮资源点
-> 拾取武器、弹药、食物、水、药物、材料
-> 根据状态直接使用部分物品
-> 遭遇感染怪物或人类敌人
-> 攻打、绕开、偷袭或引怪摧毁人类据点
-> 警戒值上升
-> 继续深入 / 原地补给 / 转移路线 / 前往撤离点
-> 激活撤离点
-> 防守撤离倒计时
-> 成功撤离或死亡
```

### 4.3 长线循环

```text
初期进入郊区农场
-> 完成农场关键任务
-> 解锁旧工业工厂
-> 在工厂修复道路、夺取钥匙卡或清理巢穴
-> 解锁封锁小城镇
-> 在小城镇找到城市、军事基地、机场线索
-> 通过搜刮、任务、改枪、成长、生存管理和基地升级持续增强
```

---

## 5. 多版本垂直切片

V2.4 的版本推进不再只设置单一 MVP，而是按 `VS版本`、`FVS版本`、`MVP版本`、`CBT版本`、`Alpha版本`、`Beta版本`、`EA版本`、`正式上线版本` 逐步扩大范围。每个版本都必须继承前一版本的验证结果，再增加新模块或提升质量。

### 5.1 版本推进链路

```text
VS版本
-> FVS版本
-> MVP版本
-> CBT版本
-> Alpha版本
-> Beta版本
-> EA版本
-> 正式上线版本
```

### 5.2 VS版本目标

`VS版本` 是最低调的基础验证版本，只用于验证俯视角核心操作是否成立。

必须验证：

1. 玩家移动是否顺手。
2. 鼠标瞄准或基础朝向是否清楚。
3. 基础射击是否能命中目标。
4. 基础怪物是否能追踪玩家。
5. 命中、扣血、死亡是否能形成最小反馈。
6. 基础 HUD 是否能表达血量、弹药和准星。

暂不包含：基地、搜打撤、仓库、人类据点、复杂生存、完整成长、正式任务和正式美术。

### 5.3 FVS版本目标

`FVS版本` 在 VS 版本基础上扩展基础战斗，用于验证五到十分钟战斗是否有爽感。

必须验证：

1. 二到三把武器是否有明显差异。
2. 二到三种怪物是否形成基础压力。
3. 基础刷怪节奏是否成立。
4. 血迹、血雾、爆炸、击退等战斗反馈是否足够直接。
5. 基础掉落和拾取是否能形成战斗奖励。
6. 玩家是否愿意重复打一局。

### 5.4 MVP版本目标

`MVP版本` 才开始验证完整最小闭环，不要求首发内容完整，但必须验证出击、战斗、搜刮、撤离、结算、成长和再次出发。

MVP 必须验证：

```text
防空洞基地
-> 进入郊区农场
-> 清怪
-> 攻打小型农场掠夺者据点
-> 搜刮食物、水、药品、武器、弹药
-> 局内直接使用食物、水、药品
-> 警戒值上升
-> 撤离
-> 回基地结算
-> 整理仓库
-> 获得经验
-> 分配属性点
-> 简单改枪
-> 休息恢复疲劳
-> 再次出发
```

### 5.5 MVP内容范围

| 模块 | MVP 内容 |
|---|---|
| 基地 | 简版地下防空洞 UI，不做 3D 漫游 |
| 地图 | 郊区农场灰盒地图 |
| 据点 | 1 个小型农场掠夺者营地 |
| 资源点 | 农舍、谷仓、加油站、医疗箱、储物箱 |
| 撤离点 | 防空洞入口、林地小路 |
| 怪物 | 感染爬行者、感染犬类、膨胀自爆体 |
| 人类敌人 | 掠夺者、哨兵、霰弹枪手 |
| 武器 | 手枪、突击步枪、霰弹枪 |
| 改装 | 基础配件雏形 |
| 生存 | 饥饿、口渴、疲劳的最小实现 |
| 可用物品 | 罐头、瓶装水、能量饮料、简易医疗包 |
| 成长 | 角色等级、经验、五大属性雏形 |
| 搜打撤 | 背包、仓库、撤离、死亡损失、结算 |
| 警戒值 | 时间、开枪、据点警报、撤离触发增长 |
| 存档 | 本地保存仓库、装备、任务、等级、状态 |

### 5.6 CBT版本目标

`CBT版本` 用于小范围封闭测试，重点验证真实玩家是否理解流程并愿意重复游玩。

必须验证：

1. 外部玩家不依赖开发者解释也能完成基础流程。
2. 玩家能理解出击、搜刮、撤离、结算和死亡损失。
3. 存档、仓库、撤离和结算没有高频阻断问题。
4. 基础难度、资源收益和警戒值节奏不明显失衡。
5. 玩家对武器手感、怪物压力和 UI 可读性有明确反馈。

### 5.7 Alpha版本目标

`Alpha版本` 用于把首发主要内容骨架全部串联起来，允许内容粗糙，但核心模块不能缺席。

必须包含：

1. 郊区农场、旧工业工厂、封锁小城镇的可玩骨架。
2. 人类据点和三方冲突基础逻辑。
3. 更多怪物、人类敌人、武器和配件。
4. 初版任务链和地图推进路线。
5. 更完整的生存资源、局内消耗品、人物成长和武器熟练度。

### 5.8 Beta版本目标

`Beta版本` 用于内容锁定后的整体打磨，原则上不再随意增加大型系统。

重点工作：

1. 修复主要 Bug。
2. 优化性能和加载。
3. 平衡武器、敌人、掉落、经济和生存消耗。
4. 完成新手引导、设置菜单、UI 可读性和主要音效反馈。
5. 准备 Steam 页面素材、截图和预告片。

### 5.9 EA版本目标

`EA版本` 用于 Steam 抢先体验，需要稳定、可重复游玩，并能支撑持续更新。

必须具备：

1. 可付费游玩的稳定版本。
2. 清晰更新路线图。
3. 社区反馈收集机制。
4. 持续补丁计划。
5. 正式上线前内容补齐清单。

### 5.10 正式上线版本目标

正式上线版本是完整商业交付版本。

必须完成：

1. 首发三张大地图。
2. 首发主要敌人、据点和 Boss。
3. 首发武器、配件和改装。
4. 首发任务内容。
5. 完整搜打撤、生存、成长、仓库、基地和撤离系统。
6. 新手引导、设置、成就、存档和平台适配。
7. Steam 商店、宣传图、预告片和发售文案。

### 5.11 多版本验收标准

必须逐步回答“是”的问题：

1. `VS版本`：玩家移动、瞄准、射击、打怪是否顺手？
2. `FVS版本`：五到十分钟战斗是否有爽感和重复游玩动力？
3. `MVP版本`：玩家是否能完成一次出击、搜刮、撤离、结算和再次出发？
4. `CBT版本`：外部玩家是否能理解流程并稳定游玩？
5. `Alpha版本`：首发主要模块是否全部串联？
6. `Beta版本`：内容是否基本锁定，性能和数值是否进入可发布状态？
7. `EA版本`：公开玩家反馈是否支撑继续优化和更新？
8. `正式上线版本`：游戏是否达到完整首发商业交付质量？

---

## 6. 首发正式版内容范围

### 6.1 地图

首发正式版只做三张大地图：

1. 郊区农场。
2. 旧工业工厂。
3. 封锁小城镇。

三张地图必须是连续推进关系：

```text
地下防空洞基地
-> 郊区农场
-> 旧工业工厂
-> 封锁小城镇
-> 后续城市 / 军事基地 / 机场线索
```

### 6.2 敌人

| 类型 | 数量建议 |
|---|---:|
| 感染 / 变异敌人 | 8-10 种 |
| 人类敌人 | 6-9 种 |
| 精英怪 | 3 个 |
| 据点头目 | 3 个 |
| Boss | 1-2 个 |

### 6.3 武器与配件

| 内容 | 数量建议 |
|---|---:|
| 武器 | 12-16 把 |
| 配件 | 40-60 个 |
| 武器类型 | 手枪、冲锋枪、突击步枪、霰弹枪、轻机枪、电磁 / 狙击、火焰喷射器、榴弹发射器、近战、特殊实验武器 |

### 6.4 生存系统

首发包含：

1. 饥饿。
2. 口渴。
3. 睡眠 / 疲劳。
4. 10-15 种食物和水资源。
5. 3-5 种疲劳恢复或临时压制物品。
6. 5-8 种局内可用药物。
7. 基地休息功能。
8. 食物储备和净水储备。
9. 生存状态 UI。
10. 局内消耗品使用 UI。
11. 生存相关任务。
12. 生存相关基地升级。

### 6.5 人物成长

首发包含：

1. 等级上限 30 级。
2. 五大属性：体能、敏捷、枪械、生存、工程。
3. 20-30 个轻量被动技能。
4. 8 类武器熟练度。
5. 属性点分配界面。
6. 武器熟练度界面。
7. 成长与任务、撤离、击杀、探索挂钩。

### 6.6 人类据点

| 地图 | 据点 |
|---|---|
| 郊区农场 | 农场掠夺者营地、加油站小据点 |
| 旧工业工厂 | 工厂武装营地、维修间据点 |
| 封锁小城镇 | 超市幸存者堡垒、警局军火点、小型军方检查站 |

### 6.7 任务数量

首发建议 40-60 个任务，包括：

1. 主线推进任务。
2. 地图支线任务。
3. 据点任务。
4. 重复任务。
5. 动态事件任务。
6. 生存资源任务。
7. 局内使用任务。
8. 成长挑战任务。

---

## 7. 玩家控制与操作设计

### 7.1 PC 默认键位

| 操作 | 键位 |
|---|---|
| 移动 | WASD |
| 瞄准 | 鼠标指向 |
| 射击 | 鼠标左键 |
| 精确瞄准 / 副模式 | 鼠标右键 |
| 换弹 | R |
| 互动 / 搜刮 / 开门 | E |
| 使用战术道具 | F |
| 切换消耗品 | Q |
| 使用当前快捷消耗品 | 建议为数字键或独立快捷键 |
| 切换武器 | 1 / 2 / 3 或鼠标滚轮 |
| 冲刺 | Shift |
| 背包 | Tab |
| 地图 | M |
| 暂停菜单 | Esc |

### 7.2 手感原则

1. 移动必须灵活，但不能像滑冰。
2. 鼠标指向哪里，武器就朝向哪里。
3. 换弹时允许移动，但移动速度下降。
4. 使用物品时不能射击，部分物品降低移动速度。
5. 食物和水使用时间短中等，药片和注射器较快。
6. 霰弹枪、机枪、步枪、冲锋枪必须有明显差异。
7. 命中反馈优先级高于模型精细度。

---

## 8. 战斗系统

### 8.1 战斗目标

战斗系统服务以下体验：

1. 射击反馈直接。
2. 怪物数量多。
3. 血腥反馈强烈。
4. 武器差异明显。
5. 人类枪战具有策略性。
6. 玩家需要走位、控弹药、利用环境。
7. 战斗节奏从探索逐步升级到混乱爆发。
8. 饥饿、口渴、疲劳影响战斗表现，但不取代玩家操作。
9. 局内可用消耗品让玩家在危险中做选择。

### 8.2 武器实现路线

第一版建议：

```text
手枪 / 步枪 / 冲锋枪 / 机枪：Hitscan
霰弹枪：多条 Hitscan Pellet
榴弹 / 火箭 / 电浆：Projectile
火焰 / 毒气：Area Effect
```

### 8.3 命中反馈链

每次命中应触发：

```text
命中判定
-> 伤害结算
-> 敌人受击硬直 / 击退
-> 受击 VFX
-> 命中音效
-> 血迹或火花贴花
-> 武器手感反馈：枪口火焰、弹壳、镜头轻震、后坐力
```

### 8.4 成熟向暴力表现边界

优先做：

1. 血迹。
2. 血雾。
3. 尸体。
4. 爆炸痕迹。
5. 焦痕。
6. 击退。
7. 死亡动画。
8. 简单残肢预制件。

谨慎做：

1. 动态断肢。
2. 复杂布娃娃堆积。
3. 高成本程序化肢解。

首发不做：

1. 全动态断肢模拟。
2. 内脏物理模拟。
3. 特写处决系统。

---

## 9. 枪械客制化系统

### 9.1 设计目标

枪械改装不是拟真军事模拟，而是改变武器手感和战斗策略。

玩家应能明显感受到：

1. 枪更稳。
2. 弹匣更大。
3. 换弹更慢或更快。
4. 子弹带火焰、腐蚀、电磁或穿透效果。
5. 武器适合清怪或打精英。
6. 某些改装适合攻打人类据点。
7. 某些改装适合撤离点防守。

### 9.2 改装槽位

建议主武器最多 5-7 个槽位：

```text
Muzzle      枪口
Sight       瞄具
Magazine    弹匣
Stock       枪托
Underbarrel 下挂
AmmoType    弹种
Core        核心模块
```

为了降低首发复杂度，MVP 只做：

```text
Muzzle
Magazine
Stock
Core
```

### 9.3 配件属性方向

| 配件类型 | 典型效果 |
|---|---|
| 消音器 | 降低噪音，略降伤害或射程 |
| 制退器 | 降低后坐力，增加噪音 |
| 扩容弹匣 | 增加弹匣容量，降低换弹速度 |
| 快速弹匣 | 提升换弹速度，容量较小 |
| 稳定枪托 | 降低散布和后坐力 |
| 轻量枪托 | 提升移动速度，稳定性下降 |
| 穿甲核心 | 提高穿透，降低射速 |
| 火焰核心 | 附加燃烧，增加弹药消耗 |
| 爆裂核心 | 小范围爆裂，噪音和警戒值提高 |

### 9.4 改枪流派

| 流派 | 配件方向 | 体验 |
|---|---|---|
| 稳定流 | 制退器 + 稳定枪托 + 标准弹匣 | 新手友好，持续输出稳定 |
| 清怪流 | 高射速核心 + 扩容弹匣 + 火焰弹种 | 清怪强，耗弹快，警戒涨得快 |
| 穿甲流 | 穿甲核心 + 重型枪管 + 高威力弹 | 打精英和重甲人类强，机动差 |
| 潜入流 | 消音器 + 轻量枪托 + 快速弹匣 | 警戒上升慢，火力较弱 |
| 撤离防守流 | 扩容弹匣 + 稳定枪托 + 压制核心 | 适合撤离倒计时防守 |
| 据点突入流 | 霰弹模块 + 快速弹匣 + 近距增伤 | 适合农舍、超市、警局室内战 |

---

## 10. 搜打撤系统

### 10.1 基础规则

玩家进入地图前可携带：

1. 武器。
2. 弹药。
3. 护甲。
4. 医疗品。
5. 食物。
6. 水。
7. 疲劳抑制品。
8. 工具。

成功撤离：

```text
背包物资进入仓库。
任务完成。
获得经验和金钱。
获得人物成长进度。
解锁基地升级 / 新任务 / 新地图路线。
```

死亡：

```text
本局未带出的搜刮物全部丢失。
局内已使用的物品不返还。
带入武器有概率损坏。
护甲耐久下降。
消耗品损失。
角色等级不清零。
仓库基础物资不清零。
死亡后增加疲劳值。
```

### 10.2 保险格规则

保险格不是 V2.4 的核心系统，但可以作为降低劝退的可选机制。

建议处理：

1. 默认普通难度可开放小型保险格。
2. 高难模式可以禁用保险格。
3. 保险格只能放小型配件、任务样本、情报、稀有材料。
4. 不能放大型武器、护甲、大型任务箱。
5. 保险格不要削弱“成功撤离”的核心价值。

### 10.3 撤离点类型

| 类型 | 示例 | 风险 |
|---|---|---|
| 固定撤离点 | 防空洞入口、工厂正门、城镇入口检查站 | 低中 |
| 随机撤离点 | 林地小路、排水出口、车站隧道 | 中 |
| 条件撤离点 | 加油站后门、员工宿舍后门、诊所地下室 | 中高 |
| 高风险撤离点 | 公路检查点、小广场信号点、封锁桥入口 | 高 |

### 10.4 撤离倒计时

```text
基础撤离：15-30 秒
中风险撤离：30-45 秒
高风险撤离：45-60 秒
警戒值高时追加怪潮、人类追击或精英怪加入
```

---

## 11. 警戒值 / 污染警戒值系统

### 11.1 命名

为避免概念混乱，系统内部建议统一叫：

```text
ThreatLevel / 警戒值
```

UI 文案可以根据地图包装为：

```text
污染警戒
感染活跃度
区域警戒
封锁等级
```

### 11.2 设计目标

警戒值连接“爽感”和“撤离压力”。

玩家越吵、越贪、停留越久、打据点越激烈，警戒值越高。警戒值越高，怪物越多，人类据点反应越强，撤离越难。

### 11.3 增长来源

```text
时间流逝：缓慢增加
开枪：按武器噪音增加
爆炸：大量增加
打开高价值箱：中量增加
破解门禁：中量增加
攻击人类据点：中量增加
触发据点警报：大量增加
击杀精英怪：大量增加
携带高价值样本：持续增加
激活撤离点：短时间快速增加
```

### 11.4 等级效果

| 等级 | 名称 | 效果 |
|---|---|---|
| 0 | 平静 | 少量巡逻怪，适合搜刮 |
| 1 | 异动 | 小规模怪物从边缘出现 |
| 2 | 警戒 | 刷怪频率提高，快速怪出现 |
| 3 | 爆发 | 精英怪加入，人类据点警戒增强 |
| 4 | 失控 | 撤离倒计时延长，刷怪点更多 |
| 5 | 封锁 | 大量怪潮追击，撤离极难，人类敌人可能封锁出口 |

### 11.5 伪代码

```csharp
public enum ThreatSource
{
    PassiveTime,
    Gunfire,
    Explosion,
    LootHighValueContainer,
    HumanStrongholdAlarm,
    EliteKilled,
    ExtractionActivated,
    MissionEvent
}

public sealed class ThreatManager : MonoBehaviour
{
    public float ThreatValue { get; private set; }
    public int ThreatLevel => Mathf.Clamp(Mathf.FloorToInt(ThreatValue / 100f), 0, 5);

    public event Action<int> OnThreatLevelChanged;
    public event Action<float> OnThreatValueChanged;

    [SerializeField] private float passiveThreatPerSecond = 0.4f;
    private int lastThreatLevel;

    public void AddThreat(float amount, ThreatSource source)
    {
        ThreatValue = Mathf.Max(0f, ThreatValue + amount);
        OnThreatValueChanged?.Invoke(ThreatValue);
        CheckLevelChanged();
    }

    private void Update()
    {
        AddThreat(passiveThreatPerSecond * Time.deltaTime, ThreatSource.PassiveTime);
    }

    private void CheckLevelChanged()
    {
        int current = ThreatLevel;
        if (current == lastThreatLevel) return;
        lastThreatLevel = current;
        OnThreatLevelChanged?.Invoke(current);
    }
}
```

---

## 12. 轻量生存模拟系统

### 12.1 系统定位

首发只做三项：

1. 饥饿。
2. 口渴。
3. 睡眠 / 疲劳。

不做：

1. 体温。
2. 精神值。
3. 复杂疾病。
4. 营养素。
5. 排泄。
6. 感染病程。

设计目标是：

```text
有压力，但不烦人。
强化搜刮价值，而不是打断射击爽感。
让局内补给品产生“现在用还是带回”的取舍。
```

### 12.2 饥饿系统

饥饿值降低来源：

1. 出击时间流逝。
2. 冲刺。
3. 近战攻击。
4. 负重过高。
5. 受伤恢复。
6. 长时间连续作战。

饥饿状态分级：

| 状态 | 效果 |
|---|---|
| 饱腹 | 体力恢复小幅提高 |
| 正常 | 无明显效果 |
| 饥饿 | 体力恢复降低 |
| 严重饥饿 | 最大体力降低，近战伤害降低 |
| 极度饥饿 | 移动速度小幅降低，负重上限降低 |

饥饿不会直接导致死亡。

### 12.3 口渴系统

口渴值降低来源：

1. 时间流逝。
2. 冲刺。
3. 高负重。
4. 火焰区域或高温区域。
5. 使用部分刺激剂。
6. 长时间战斗。

口渴状态分级：

| 状态 | 效果 |
|---|---|
| 水分充足 | 体力消耗小幅降低 |
| 正常 | 无明显效果 |
| 口渴 | 体力恢复降低 |
| 严重口渴 | 射击稳定性下降 |
| 脱水 | 移动速度下降，瞄准扩散增加 |

口渴不会直接导致死亡，但会明显影响战斗手感。

### 12.4 睡眠 / 疲劳系统

疲劳值增加来源：

1. 每次出击。
2. 长时间局内行动。
3. 负重撤离。
4. 受伤。
5. 死亡后恢复。
6. 使用刺激剂。
7. 高强度战斗。

疲劳状态分级：

| 状态 | 效果 |
|---|---|
| 精力充沛 | 搜刮速度和射击稳定性小幅提高 |
| 正常 | 无明显效果 |
| 疲劳 | 换弹速度小幅降低 |
| 严重疲劳 | 射击准确性降低，体力恢复降低 |
| 极度疲劳 | 移动速度降低，瞄准扩散增加，搜刮速度降低 |

疲劳恢复方式：

1. 回防空洞睡眠。
2. 使用简易床铺。
3. 使用睡眠药物。
4. 在基地消耗时间休息。
5. 使用咖啡因、能量饮料、疲劳抑制剂临时压制疲劳。
6. 在局内特殊安全房进行短休。

### 12.5 生存状态实现建议

核心类：

```text
SurvivalStatusComponent
HungerStatus
ThirstStatus
FatigueStatus
SurvivalEffectApplier
SurvivalStatusDefinition
SurvivalThresholdDefinition
SurvivalStatusUI
BaseRestService
```

`SurvivalStatusComponent` 不直接改武器和移动数值，而是输出统一 Modifier，由角色属性系统汇总。

---

## 13. 局内物品使用系统

### 13.1 系统定位

局内物品使用系统连接：

1. 生存模拟。
2. 搜打撤。
3. 背包管理。
4. 战斗风险。
5. 长期仓库积累。

玩家在局内搜到的部分物品可以直接使用，也可以带回基地。

核心选择：

```text
现在用掉，保证活着撤离；
还是带回基地，作为长期储备？
```

### 13.2 局内可用物品

| 类型 | 示例 | 效果 |
|---|---|---|
| 食物 | 罐头、压缩饼干、肉干、军粮 | 恢复饥饿 |
| 水 | 瓶装水、净水袋、电解质饮料 | 恢复口渴 |
| 医疗品 | 简易医疗包、恢复针、止痛药 | 恢复生命或解除负面状态 |
| 疲劳品 | 能量饮料、咖啡因药片、疲劳抑制剂 | 临时压制疲劳 |
| 净化工具 | 净水片、净化剂 | 将可疑水转化为可饮用水 |
| 任务药剂 | 特殊样本药剂 | 任务用途或特殊状态处理 |

### 13.3 使用方式

1. 背包右键使用。
2. 放入快捷栏后按快捷键使用。
3. 拾取时选择“使用 / 收入背包”。
4. 对低价值补给可在设置中开启自动使用。
5. 对水源、床铺、医疗柜等场景点进行场景互动使用。

### 13.4 使用时间与风险

| 物品 | 使用时间 | 风险 |
|---|---:|---|
| 瓶装水 | 1-3 秒 | 可缓慢移动 |
| 能量饮料 | 1-2 秒 | 可缓慢移动 |
| 药片 | 1 秒 | 风险较低 |
| 注射器 | 0.8-1.5 秒 | 快速但可能有副作用 |
| 罐头 | 3-5 秒 | 移动速度下降，容易被打断 |
| 军粮 | 4-5 秒 | 恢复多，但使用慢 |
| 简易医疗包 | 3-4 秒 | 被攻击可中断 |
| 大型医疗包 | 5-8 秒 | 战斗中风险高 |
| 局内短休 | 20-60 秒 | 只能在安全区使用 |

### 13.5 核心类

```text
ConsumableItemDefinition
ItemUseEffect
UseItemAction
ItemUseChannel
QuickSlotConsumableUI
PickupOrUsePrompt
UseInterruptedByDamage
ConsumableEffectResolver
SurvivalRestoreEffect
TemporaryBuffEffect
```

---

## 14. 人物成长系统

### 14.1 系统定位

人物成长是首发核心系统之一，提供长期目标。

采用：

```text
等级 + 属性点 + 武器熟练度 + 少量被动技能
```

不做：

1. 复杂职业。
2. 大型技能树。
3. 上百个天赋节点。
4. 无限制数值膨胀。

### 14.2 经验来源

1. 击杀怪物。
2. 击杀人类敌人。
3. 击杀精英怪。
4. 摧毁感染巢穴。
5. 完成任务。
6. 成功撤离。
7. 首次探索新区域。
8. 攻破人类据点。
9. 带出高价值物品。
10. 完成高警戒值撤离。

经验奖励必须鼓励成功撤离，而不是无限刷怪。

### 14.3 等级成长

首发等级上限：30 级。

升级奖励：

1. 少量基础属性提升。
2. 1 点属性点。
3. 每若干等级获得 1 个被动技能点。

成长节奏：

| 等级段 | 节奏 | 目的 |
|---|---|---|
| 1-5 | 快速 | 新手期反馈明显 |
| 6-15 | 中速 | 逐步形成玩法偏好 |
| 16-30 | 慢速 | 服务后期地图和高难挑战 |

### 14.4 五大属性

| 属性 | 影响内容 | 适合玩家 |
|---|---|---|
| 体能 | 生命、体力、负重、抗性 | 重甲、机枪、高负重搜刮 |
| 敏捷 | 移动、冲刺、脱离包围 | 跑图、游走、快速撤离 |
| 枪械 | 准确性、后坐力、换弹、武器切换 | 正面战斗、精英击杀 |
| 生存 | 饥饿、口渴、疲劳、医疗效率 | 长时间探索、低补给挑战 |
| 工程 | 搜刮、开锁、维修、工具效率 | 据点攻坚、任务和资源最大化 |

### 14.5 成长限制

为避免破坏难度，建议上限：

1. 移动速度最高提升 15%。
2. 射击准确性最高提升 20%。
3. 后坐力控制最高提升 20%。
4. 换弹速度最高提升 15%。
5. 负重最高提升 25%。
6. 饥饿和口渴消耗最多降低 25%。
7. 疲劳增长最多降低 25%。
8. 局内消耗品使用速度最多提升 20%。

### 14.6 被动技能示例

| 技能 | 效果 |
|---|---|
| 快手换弹 | 换弹速度提升 8% |
| 稳定射击 | 连续射击时后坐力累积降低 |
| 轻装突入 | 负重低于 50% 时移动速度提升 |
| 负重老兵 | 高负重时移动惩罚降低 |
| 深区幸存者 | 饥饿、口渴消耗降低 |
| 短眠恢复 | 睡眠恢复疲劳效率提升 |
| 现场补给 | 局内使用食物和水的速度提升 |
| 药品熟手 | 局内使用医疗品速度提升，治疗效果小幅提高 |
| 搜刮专家 | 开箱和搜索尸体速度提升 |
| 撤离专家 | 撤离倒计时期间体力恢复提升 |
| 据点攻坚 | 对人类敌人造成轻微额外压制 |
| 怪潮猎手 | 连续击杀怪物时短时间提升换弹速度 |

### 14.7 核心类

```text
CharacterLevelData
ExperienceService
AttributePointSystem
CharacterAttributeDefinition
DerivedStatsCalculator
PassiveSkillDefinition
PassiveSkillState
CharacterGrowthSaveData
CharacterGrowthUI
```

---

## 15. 武器熟练度系统

### 15.1 熟练度类型

1. 手枪熟练度。
2. 冲锋枪熟练度。
3. 突击步枪熟练度。
4. 霰弹枪熟练度。
5. 机枪熟练度。
6. 狙击 / 电磁武器熟练度。
7. 榴弹 / 爆炸武器熟练度。
8. 火焰武器熟练度。

### 15.2 提升方式

1. 使用该类武器击杀敌人。
2. 使用该类武器完成任务。
3. 使用该类武器成功撤离。
4. 使用该类武器击杀精英或据点头目。

### 15.3 奖励方向

1. 降低该类武器后坐力。
2. 提高该类武器换弹速度。
3. 降低移动射击惩罚。
4. 解锁少量专属被动。

### 15.4 核心类

```text
WeaponProficiencyDefinition
WeaponProficiencyState
WeaponProficiencyService
WeaponCategoryProgress
ProficiencyBonusApplier
WeaponProficiencySaveData
WeaponProficiencyUI
```

---

## 16. 人类据点系统

### 16.1 系统目标

人类据点是 V2.4 的重要差异化系统。

它让游戏不只是打尸潮，而是出现：

1. 资源争夺。
2. 枪战压制。
3. 偷袭和绕路。
4. 引怪破局。
5. 三方混战。
6. 高价值战利品风险收益。

### 16.2 据点玩法

玩家遇到据点时可以选择：

1. 绕开：安全，但错过资源。
2. 偷袭：优先击杀哨兵，降低警报风险。
3. 强攻：快速夺取资源，但引发高强度战斗。
4. 引怪：把附近怪物引入据点，制造三方混战。
5. 等待：观察巡逻路线，寻找入口。
6. 消耗补给后继续强攻：在据点内部搜到食物、水或药物后直接使用，支撑后续战斗。

### 16.3 据点奖励

1. 枪械。
2. 弹药。
3. 武器配件。
4. 医疗品。
5. 食物。
6. 水。
7. 燃油。
8. 药物。
9. 钥匙。
10. 地图情报。
11. 任务物品。
12. 高价值材料。

### 16.4 据点风险

1. 枪声提高警戒值。
2. 警报吸引附近怪物。
3. 人类敌人可能封锁出口。
4. 据点内部可能触发感染突破事件。
5. 玩家负重增加后撤离难度提高。
6. 长时间战斗可能错过撤离窗口。
7. 高强度战斗会加快疲劳积累和水分消耗。
8. 在据点内部使用物品容易被敌人打断。

### 16.5 核心类

```text
StrongholdController
StrongholdDefinition
StrongholdAlarmSystem
PatrolRoute
CoverPoint
HumanEnemyAgent
HumanEnemySenses
HumanRangedAttackController
HumanSuppressionController
StrongholdLootTable
StrongholdEventTrigger
```

---

## 17. 三方冲突系统

### 17.1 阵营

首发阵营：

1. Player。
2. Infected。
3. HumanHostile。
4. Neutral / Environment。

### 17.2 仇恨矩阵

```text
Player vs Infected：敌对
Player vs HumanHostile：敌对
Infected vs HumanHostile：敌对
Infected vs Infected：友方
HumanHostile vs HumanHostile：友方
```

### 17.3 触发方式

1. 玩家攻击人类据点。
2. 据点警报响起。
3. 玩家枪声过大。
4. 怪物巡逻队靠近人类据点。
5. 玩家故意引怪。
6. 高警戒值导致怪潮刷新。
7. 任务事件触发感染突破。

### 17.4 核心类

```text
FactionId
FactionRelationshipTable
ThreatTargetSelector
AggroSource
CombatNoiseEmitter
FactionCombatEvent
```

---

## 18. 敌人与 AI

### 18.1 感染怪物

MVP：

1. 感染爬行者：基础近战怪，数量多，低血量。
2. 感染犬类：快速怪，打断搜刮和走位。
3. 膨胀自爆体：靠近爆炸，死亡产生范围危险。

首发扩展：

1. 撕裂者。
2. 酸液喷吐者。
3. 装甲屠夫。
4. 巢母触须。
5. 工厂变异体。
6. 实验体 Alpha。

### 18.2 人类敌人

MVP：

1. 人类掠夺者。
2. 人类哨兵。
3. 人类霰弹枪手。

首发扩展：

1. 人类步枪手。
2. 人类机枪手。
3. 人类投掷兵。
4. 人类狙击手。
5. 人类盾牌兵。
6. 人类据点头目。

### 18.3 AI 状态机

感染怪：

```text
Idle
-> Patrol
-> Alert
-> Chase
-> Attack
-> Stagger
-> Death
```

快速怪增加：

```text
LeapPrepare
-> LeapAttack
-> Recover
```

远程怪增加：

```text
KeepDistance
-> Aim
-> Spit
-> Relocate
```

人类敌人增加：

```text
Patrol
-> Suspicious
-> Alert
-> TakeCover
-> Shoot
-> Reload
-> ThrowGrenade
-> Retreat
-> CallAlarm
-> Death
```

### 18.4 刷怪导演

输入：

1. 警戒值。
2. 玩家位置。
3. 玩家血量。
4. 玩家弹药量。
5. 背包价值。
6. 是否激活撤离点。
7. 附近刷怪点可用性。
8. 是否处于据点战。

输出：

1. 生成小怪波次。
2. 生成精英怪。
3. 从侧翼施压。
4. 暂停刷怪给玩家喘息。
5. 撤离点最终围攻。
6. 据点警报后引入感染怪。

---

## 19. 地图与关卡设计

### 19.1 地图结构原则

1. 固定地图主体。
2. 随机战利品点。
3. 随机锁门 / 事件。
4. 随机撤离点状态。
5. 随机精英怪刷新。
6. 不做完全随机生成大地图。
7. 单局时长控制在 10-20 分钟。

### 19.2 地图一：郊区农场

定位：新手期和 MVP 主要地图。

主要区域：

1. 防空洞地表出口。
2. 废弃农舍。
3. 谷仓与粮仓。
4. 玉米地 / 麦田。
5. 牲畜棚。
6. 灌溉渠。
7. 小树林。
8. 废弃加油站。
9. 农场掠夺者营地。
10. 通往工厂的公路检查点。

核心资源：

1. 基础食物。
2. 瓶装水。
3. 燃油。
4. 低级弹药。
5. 农具材料。
6. 低级医疗品。

关键任务：

1. 修复防空洞发电机。
2. 搜刮农场医疗箱。
3. 清理谷仓感染体。
4. 消灭农场掠夺者头目。
5. 回收加油站燃油。
6. 找到通往工厂的公路钥匙。

### 19.3 地图二：旧工业工厂

定位：从郊区生存过渡到城市灾变。

主要区域：

1. 工厂正门。
2. 货车装卸区。
3. 废弃生产车间。
4. 化工储罐区。
5. 地下排水系统。
6. 员工宿舍。
7. 机械维修间。
8. 工厂武装营地。
9. 感染巢穴核心。
10. 通往小城镇的封锁公路。

核心资源：

1. 金属零件。
2. 电子元件。
3. 机械零件。
4. 武器配件。
5. 污染样本。
6. 刺激剂。
7. 咖啡因药片。
8. 能量饮料。

### 19.4 地图三：封锁小城镇

定位：首发最高危区域。

主要区域：

1. 城镇入口检查站。
2. 超市。
3. 诊所。
4. 警局。
5. 住宅区。
6. 学校。
7. 小广场。
8. 废弃车站。
9. 掠夺者大型据点。
10. 军方临时营地。
11. 感染巢穴核心。
12. 通往城市的封锁大桥。

核心资源：

1. 高级药品。
2. 军粮。
3. 瓶装水。
4. 枪械。
5. 蓝图。
6. 军方物资。
7. 长期食物储备。
8. 城市区域线索。

---

## 20. 基地系统

### 20.1 基地表现

首发建议基地以 UI 为主，不做完整 3D 漫游，降低成本。

基地是地下防空洞，包含：

1. 仓库。
2. 工作台。
3. 医疗区。
4. 发电机房。
5. 通讯终端。
6. 地图墙。
7. 交易角落。
8. 出入口。
9. 食物储备区。
10. 净水储备区。
11. 床铺 / 休息区。
12. 角色成长面板。

### 20.2 基地升级

| 升级 | 功能 |
|---|---|
| 修复发电机 | 解锁高级工作台功能 |
| 扩建仓库 | 提高储存上限 |
| 加固入口 | 降低基地风险事件 |
| 修复通讯 | 解锁更多任务 |
| 建立医疗站 | 降低死亡后惩罚 |
| 修复地图系统 | 显示更多撤离点和资源点 |
| 改造武器台 | 解锁高级枪械改装 |
| 建立食物储备区 | 提高食物保存上限 |
| 建立净水储备区 | 提高水资源保存上限 |
| 修复床铺 | 提高疲劳恢复效率 |
| 建立训练角落 | 解锁部分人物成长功能 |
| 建立情报终端 | 显示高价值资源点概率 |

---

## 21. 任务系统

### 21.1 任务类型

1. 回收任务。
2. 击杀任务。
3. 破坏任务。
4. 下载任务。
5. 据点任务。
6. 探索任务。
7. 限时任务。
8. 防守任务。
9. 路线任务。
10. 生存任务。
11. 局内使用任务。
12. 成长任务。

### 21.2 地图推进任务

郊区农场：

1. 修复防空洞发电机。
2. 获取农场区域地图。
3. 找到通往工厂的道路。
4. 打开公路检查点铁门。
5. 带回第一批食物和饮水储备。

旧工业工厂：

1. 获取工厂钥匙卡。
2. 修复工厂供电。
3. 清理地下排水巢穴。
4. 打开通往小城镇的封锁路障。
5. 带回刺激剂样本和休息区修复材料。

封锁小城镇：

1. 找到城市封锁桥路线图。
2. 获取军方通讯设备。
3. 调查感染扩散源头。
4. 解锁后续城市区域线索。
5. 搜索超市和诊所，带回长期生存物资。

---

## 22. 数据驱动设计

### 22.1 ScriptableObject 类型

建议建立：

```text
WeaponDefinition
AttachmentDefinition
AmmoDefinition
EnemyDefinition
HumanEnemyDefinition
LootItemDefinition
ConsumableItemDefinition
SurvivalStatusDefinition
SurvivalThresholdDefinition
MissionDefinition
MapDefinition
StrongholdDefinition
ExtractionDefinition
BaseUpgradeDefinition
SpawnWaveDefinition
ThreatLevelDefinition
CharacterAttributeDefinition
PassiveSkillDefinition
WeaponProficiencyDefinition
```

### 22.2 WeaponDefinition 示例

```csharp
public enum WeaponPlatform
{
    Pistol,
    SMG,
    AssaultRifle,
    Shotgun,
    LMG,
    SniperOrRailgun,
    Flamethrower,
    Launcher,
    Experimental
}

[CreateAssetMenu(menuName = "CZ/Weapons/Weapon Definition")]
public sealed class WeaponDefinition : ScriptableObject
{
    public string id;
    public string displayName;
    public WeaponPlatform platform;
    public GameObject prefab;
    public Sprite icon;

    public float damage;
    public float fireRate;
    public int magazineSize;
    public float reloadTime;
    public float spreadAngle;
    public float recoil;
    public int penetration;
    public float noise;
    public float weight;

    public bool usesHeat;
    public float heatPerShot;
    public float heatCooldown;

    public AttachmentSlot[] allowedSlots;
}
```

### 22.3 ConsumableItemDefinition 示例

```csharp
public enum ConsumableCategory
{
    Food,
    Water,
    Medical,
    FatigueSuppressor,
    Purification,
    TemporaryBuff,
    MissionDrug
}

[CreateAssetMenu(menuName = "CZ/Consumables/Consumable Item Definition")]
public sealed class ConsumableItemDefinition : ScriptableObject
{
    public string id;
    public string displayName;
    public Sprite icon;
    public ConsumableCategory category;
    public float weight;
    public float useDuration;
    public bool canUseInRaid;
    public bool canUseInBase;
    public bool canBeInterrupted;
    public ItemUseEffect[] effects;
}
```

### 22.4 Character Growth 示例

```csharp
public enum CharacterAttributeType
{
    Body,
    Agility,
    Firearms,
    Survival,
    Engineering
}

[Serializable]
public sealed class CharacterGrowthSaveData
{
    public int level;
    public int currentExp;
    public int availableAttributePoints;
    public int availablePassiveSkillPoints;
    public Dictionary<CharacterAttributeType, int> attributes;
    public List<string> unlockedPassiveSkillIds;
}
```

### 22.5 SaveData 字段

```csharp
[Serializable]
public sealed class SaveData
{
    public int version;

    public int currency;
    public List<StoredItemData> stash;
    public List<WeaponInstanceData> weapons;
    public List<string> completedMissionIds;
    public List<BaseUpgradeState> baseUpgrades;
    public List<string> unlockedMapIds;

    public CharacterGrowthSaveData characterGrowth;
    public SurvivalSaveData survival;
    public List<WeaponProficiencySaveData> weaponProficiencies;
}
```

---

## 23. 关键系统组件清单

### 23.1 Core

```text
GameBootstrap
GameStateMachine
ServiceRegistry
SceneLoader
EventBus
TimeService
RandomService
```

### 23.2 Player

```text
PlayerController
PlayerAimController
PlayerHealth
PlayerInteractor
PlayerLoadout
PlayerInventoryBridge
PlayerStatsController
```

### 23.3 Weapons

```text
WeaponController
WeaponInstance
WeaponDefinition
AttachmentDefinition
HitscanWeapon
ProjectileWeapon
ShotgunWeapon
AmmoPool
ReloadController
RecoilController
WeaponEventEmitter
```

### 23.4 Enemies

```text
EnemyAgent
EnemyHealth
EnemyAttackController
EnemySenses
EnemyStateMachine
EnemyDeathHandler
EnemyDropTable
```

### 23.5 HumanEnemies

```text
HumanEnemyAgent
HumanEnemyHealth
HumanEnemySenses
HumanRangedAttackController
HumanCoverController
HumanAlarmCaller
HumanSuppressionController
```

### 23.6 Factions

```text
FactionId
FactionRelationshipTable
ThreatTargetSelector
AggroSource
CombatNoiseEmitter
```

### 23.7 Loot / Inventory

```text
LootItemDefinition
LootContainer
LootDropper
InventoryGrid
StashInventory
OptionalInsuranceContainer
PickupPrompt
ItemTooltip
```

### 23.8 Consumables / Survival

```text
ConsumableItemDefinition
UseItemAction
ItemUseEffect
QuickSlotConsumableUI
SurvivalStatusComponent
HungerStatus
ThirstStatus
FatigueStatus
SurvivalEffectApplier
BaseRestService
```

### 23.9 Character Growth

```text
ExperienceService
CharacterLevelData
AttributePointSystem
DerivedStatsCalculator
PassiveSkillDefinition
PassiveSkillService
CharacterGrowthUI
```

### 23.10 Weapon Proficiency

```text
WeaponProficiencyDefinition
WeaponProficiencyService
WeaponCategoryProgress
ProficiencyBonusApplier
WeaponProficiencyUI
```

### 23.11 Extraction / Threat

```text
RaidStateMachine
ThreatManager
ThreatEventSource
ExtractionZone
ExtractionRequirement
ExtractionCountdownUI
SpawnDirector
SpawnPoint
WaveBudget
```

### 23.12 Strongholds

```text
StrongholdDefinition
StrongholdController
StrongholdAlarmSystem
PatrolRoute
CoverPoint
StrongholdLootTable
StrongholdEventTrigger
```

### 23.13 UI

```text
HUDController
AmmoWidget
HealthWidget
ArmorWidget
ThreatWidget
SurvivalStatusWidget
QuickConsumableWidget
BackpackUI
StashUI
WeaponBenchUI
MissionBoardUI
CharacterGrowthUI
WeaponProficiencyUI
ExtractionMarkerUI
```

---

## 24. UI / UX

### 24.1 局内 HUD

必须显示：

1. 生命值。
2. 护甲值。
3. 当前武器。
4. 当前弹药 / 备用弹药。
5. 换弹状态。
6. 背包负重。
7. 小地图。
8. 任务目标。
9. 警戒值。
10. 撤离点状态。
11. 饥饿状态。
12. 口渴状态。
13. 疲劳状态。
14. 快捷消耗品栏。
15. 稀有物品拾取提示。

饥饿、口渴、疲劳不需要占据过多屏幕空间。建议小图标 + 颜色提示，只有进入负面阶段时明显提醒。

### 24.2 背包界面

物品需要显示：

1. 名称。
2. 稀有度。
3. 重量。
4. 价值。
5. 是否可局内使用。
6. 使用效果。
7. 使用时间。
8. 是否有副作用。
9. 是否任务物品。
10. 是否可带出。

右键菜单：

1. 使用。
2. 放入快捷栏。
3. 拆分。
4. 丢弃。
5. 标记带出。

### 24.3 基地界面

基地 UI 包含：

1. 仓库。
2. 工作台。
3. 任务。
4. 地图。
5. 商人。
6. 医疗。
7. 食物与饮水。
8. 休息。
9. 角色成长。
10. 武器熟练度。
11. 出发。

---

## 25. 性能优化重点

1. 敌人对象池。
2. 子弹对象池。
3. 血迹对象池。
4. 掉落物对象池。
5. 粒子数量限制。
6. 尸体数量限制。
7. 怪物 AI 分批更新。
8. 远距离敌人低频更新。
9. 人类敌人掩体点预计算。
10. 地图区域分块加载。
11. 生存状态只做数值结算，不做复杂模拟。
12. 人物成长属性通过统一属性系统计算，避免散落在各模块中。
13. 大量血迹使用贴花池和生命周期控制。
14. 撤离点怪潮刷怪使用预算制，避免瞬时刷爆。

---

## 26. 开发里程碑

### 26.1 M0：项目搭建，1 周

1. Unity URP 项目创建。
2. Git 仓库和 LFS 配置。
3. 基础目录结构。
4. Boot / MainMenu / Base_Bunker / Raid_Farm_01 场景。
5. 输入系统配置。
6. 灰盒地面和相机。

### 26.2 M1：战斗原型，2 周

1. 玩家移动和瞄准。
2. 相机跟随。
3. Hitscan 武器。
4. 换弹和弹药。
5. 1 种感染怪追踪玩家。
6. 敌人受击、死亡、掉落。
7. 基础血迹和枪口反馈。

### 26.3 M2：郊区农场局内循环，2 周

1. 郊区农场灰盒地图。
2. LootContainer。
3. 背包 UI。
4. 警戒值。
5. 刷怪导演。
6. 撤离点倒计时。
7. 成功 / 失败结算。

### 26.4 M3：生存与局内物品使用，2 周

1. 饥饿值。
2. 口渴值。
3. 疲劳值。
4. 罐头、瓶装水、能量饮料、医疗包。
5. 背包使用。
6. 快捷栏使用。
7. 使用中断。
8. 生存状态 UI。

### 26.5 M4：人类据点与三方冲突，2 周

1. 农场掠夺者营地。
2. 人类哨兵。
3. 人类霰弹枪手。
4. 据点警报。
5. 简单掩体行为。
6. 感染怪与人类敌人互相敌对。
7. 引怪进入据点。

### 26.6 M5：局外成长，2 周

1. 仓库。
2. 武器台。
3. 配件装配。
4. 角色等级。
5. 属性点。
6. 基础被动技能。
7. 本地存档。
8. 任务板。

### 26.7 M6：垂直切片打磨，2 周

1. 音效。
2. VFX。
3. UI 打磨。
4. 难度调优。
5. 性能测试。
6. Demo 打包。

---

## 27. 预算优先级

在 50 万预算下，优先级必须明确：

```text
战斗手感
> 枪械反馈
> 怪物压迫
> 搜打撤闭环
> 局内补给使用
> 轻量生存压力
> 人类据点
> 人物成长
> 改枪成长
> 三张地图内容量
> 美术精修
> 剧情包装
```

预算建议：

| 模块 | 预算区间 |
|---|---:|
| 程序开发 | 18-22 万 |
| 美术资源 | 12-15 万 |
| 特效与动画 | 5-7 万 |
| UI / 图标 / 商店视觉 | 3-5 万 |
| 音效与音乐 | 2-4 万 |
| 测试与优化 | 2-3 万 |
| Steam / 宣传 / 杂费 | 2-4 万 |
| 风险预留 | 5 万左右 |

---

## 28. Codex 开发指令原则

给 Codex 的任务必须小、明确、有验收标准。

不要这样写：

```text
帮我做一个类似孤胆枪手的游戏。
帮我把搜打撤系统做出来。
帮我做完整 AI。
帮我做完整生存系统。
```

应该拆成：

```text
做玩家移动。
做鼠标瞄准。
做武器定义。
做射线射击。
做敌人血量。
做敌人 NavMesh 追踪。
做战利品箱。
做撤离倒计时。
做饥饿数值。
做局内使用罐头。
做人类哨兵巡逻。
做角色经验结算。
```

推荐任务格式：

```text
目标：实现某个小系统。
上下文：说明项目已有内容。
文件：明确要创建或修改的路径。
要求：列出具体行为。
验收：写出 Play 后如何确认成功。
限制：禁止使用什么、不要改什么。
```

---

## 29. Codex 任务拆分清单

### 29.1 第一批：基础框架

1. 创建 `_Project` 目录结构。
2. 创建 `CZ.Core`、`CZ.Player`、`CZ.Weapons` 等 asmdef。
3. 实现 `GameBootstrap`。
4. 实现 `InputReader`。
5. 实现 `PlayerController`。
6. 实现 `TopDownCameraFollow`。
7. 实现 `MouseAimController`。

### 29.2 第二批：武器

1. 创建 `WeaponDefinition`。
2. 创建 `AttachmentDefinition`。
3. 实现 `WeaponInstance`。
4. 实现 `WeaponController`。
5. 实现 `HitscanWeapon`。
6. 实现 `ShotgunWeapon`。
7. 实现弹药和换弹。
8. 实现基础枪口火焰、弹壳、音效事件接口。

### 29.3 第三批：敌人

1. 创建 `EnemyDefinition`。
2. 实现 `EnemyHealth`。
3. 实现 `EnemyAgent`，使用 NavMeshAgent 追踪玩家。
4. 实现近战攻击。
5. 实现受击硬直。
6. 实现死亡掉落。
7. 实现 3 种感染怪 prefab。

### 29.4 第四批：背包与战利品

1. 创建 `LootItemDefinition`。
2. 实现 `LootContainer`。
3. 实现 `InventoryGrid` 数据结构。
4. 实现基础背包 UI。
5. 实现拾取与丢弃。
6. 实现仓库。
7. 可选实现保险格。

### 29.5 第五批：搜打撤循环

1. 实现 `RaidStateMachine`。
2. 实现 `ThreatManager`。
3. 实现 `SpawnDirector`。
4. 实现 `ExtractionZone`。
5. 实现撤离倒计时 UI。
6. 实现成功撤离结算。
7. 实现死亡失败结算。

### 29.6 第六批：生存与局内使用

1. 实现 `SurvivalStatusComponent`。
2. 实现饥饿值。
3. 实现口渴值。
4. 实现疲劳值。
5. 实现 `ConsumableItemDefinition`。
6. 实现 `UseItemAction`。
7. 实现罐头恢复饥饿。
8. 实现瓶装水恢复口渴。
9. 实现能量饮料临时压制疲劳。
10. 实现简易医疗包恢复生命。
11. 实现快捷消耗品栏。
12. 实现使用中断。

### 29.7 第七批：人类据点

1. 实现 `HumanEnemyDefinition`。
2. 实现人类哨兵巡逻。
3. 实现人类远程射击。
4. 实现简单掩体点。
5. 实现据点警报。
6. 实现 `StrongholdController`。
7. 实现据点战利品箱。
8. 实现人类与感染怪互相敌对。

### 29.8 第八批：人物成长

1. 实现经验获取。
2. 实现角色等级。
3. 实现五大属性。
4. 实现属性点分配。
5. 实现衍生属性计算。
6. 实现基础被动技能。
7. 实现武器熟练度经验。
8. 实现成长 UI。
9. 实现成长存档。

### 29.9 第九批：基地系统

1. 实现防空洞基地 UI。
2. 实现仓库。
3. 实现武器台。
4. 实现食物与饮水处理。
5. 实现休息恢复疲劳。
6. 实现基地升级。
7. 实现任务板。
8. 实现地图选择路线图。

---

## 30. 第一阶段 Codex Prompt 示例

### Prompt 1：玩家控制器

```text
你是 Unity C# 开发助手。请在现有 Unity 3D URP 项目中实现一个俯视角玩家控制器。

上下文：
- 角色在 XZ 平面移动，Y 是高度。
- 相机是 Orthographic，从斜上方看。
- 项目使用 Input System，但脚本可先兼容 Keyboard.current 和 Mouse.current。

请创建：
Assets/_Project/Scripts/Player/PlayerController.cs
Assets/_Project/Scripts/Player/MouseAimController.cs

要求：
1. WASD 移动。
2. 鼠标指向地面时，角色朝向鼠标所在点。
3. 支持 Rigidbody 移动。
4. 暴露 moveSpeed、rotationSpeed、groundMask。
5. 写清楚必要的组件依赖。
6. 不要使用 FindObjectOfType。

验收：
- 挂到 Player prefab 后可以移动和瞄准。
- 鼠标移到角色周围不同方向时，角色旋转正确。
- Console 不出现 NullReferenceException。
```

### Prompt 2：武器数据与射击

```text
请为项目实现基础武器系统。

创建：
- WeaponDefinition.cs
- AttachmentDefinition.cs
- WeaponInstance.cs
- WeaponController.cs
- HitscanWeapon.cs

要求：
1. WeaponDefinition 使用 ScriptableObject。
2. AttachmentDefinition 使用 ScriptableObject。
3. WeaponInstance 可以根据基础武器和配件计算最终属性。
4. HitscanWeapon 使用 Physics.Raycast 做命中。
5. 支持 fireRate、damage、spreadAngle、magazineSize、reloadTime、noise。
6. 射击时触发 OnWeaponFired 事件，方便后续 VFX 和警戒值监听。
7. 不写 UI。

验收：
- 可以在 Inspector 指定 WeaponDefinition。
- 左键射击能打中 EnemyHealth。
- 弹匣耗尽后不能射击，按 R 换弹。
```

### Prompt 3：警戒值系统

```text
请实现 ThreatManager 警戒值系统。

创建：
- ThreatManager.cs
- ThreatSource.cs

要求：
1. ThreatValue 从 0 开始。
2. 每秒按 passiveThreatPerSecond 增长。
3. AddThreat(float amount, ThreatSource source) 可由开枪、爆炸、开箱、据点警报、撤离点调用。
4. 每 100 点提升 1 级，等级 0-5。
5. 等级变化时触发 C# event。
6. 提供 Inspector Debug 显示。

验收：
- 开枪事件可以增加 ThreatValue。
- ThreatLevel 达到新等级时 Debug.Log 能收到通知。
- ThreatLevel 不超过 5。
```

### Prompt 4：撤离点

```text
请实现基础 ExtractionZone。

要求：
1. 玩家进入 Trigger 后按 E 激活。
2. 激活后开始倒计时。
3. 倒计时期间通知 SpawnDirector 进入撤离压力模式。
4. 倒计时结束后调用 RaidStateMachine.CompleteExtraction()。
5. 如果玩家离开区域，可以配置是否取消倒计时。
6. 倒计时受 ThreatLevel 影响。

验收：
- 玩家进入撤离区能看到倒计时。
- 倒计时结束后进入撤离成功状态。
- ThreatLevel 越高，倒计时越长或刷怪压力越大。
```

### Prompt 5：生存状态系统

```text
请实现轻量生存状态系统。

创建：
- SurvivalStatusComponent.cs
- SurvivalStatusType.cs
- SurvivalStage.cs
- SurvivalStatusUI.cs 可先用 Debug.Log 替代

要求：
1. 包含 Hunger、Thirst、Fatigue 三项。
2. Hunger 和 Thirst 随局内时间下降。
3. Fatigue 随局内时间和出击次数上升。
4. 每项状态有 5 个阶段。
5. 状态阶段变化时触发事件。
6. 不要直接修改 PlayerController 数值，只输出可查询的 Modifier。

验收：
- Play 后等待一段时间，Hunger 和 Thirst 会下降。
- Fatigue 会随时间上升。
- 阶段变化时能 Debug.Log。
```

### Prompt 6：局内消耗品使用

```text
请实现局内消耗品使用系统的第一版。

创建：
- ConsumableItemDefinition.cs
- ItemUseEffect.cs
- UseItemAction.cs

要求：
1. ConsumableItemDefinition 使用 ScriptableObject。
2. 支持 Food、Water、Medical、FatigueSuppressor 四类。
3. 支持 useDuration。
4. 使用期间玩家不能射击。
5. 被攻击时可中断使用。
6. 食物恢复 Hunger。
7. 水恢复 Thirst。
8. 医疗包恢复生命。
9. 能量饮料临时压制 Fatigue 负面效果。

验收：
- 背包里选中罐头并使用，可以恢复 Hunger。
- 使用瓶装水可以恢复 Thirst。
- 使用医疗包可以恢复生命。
- 使用过程中受到伤害会中断。
```

### Prompt 7：人类哨兵与据点警报

```text
请实现人类据点的第一版哨兵逻辑。

创建：
- HumanEnemyAgent.cs
- HumanEnemySenses.cs
- StrongholdAlarmSystem.cs

要求：
1. 人类哨兵按 PatrolRoute 巡逻。
2. 看到玩家后进入 Alert。
3. Alert 后调用 StrongholdAlarmSystem.RaiseAlarm()。
4. RaiseAlarm 后通知 ThreatManager 增加警戒值。
5. RaiseAlarm 后可唤醒据点内其他人类敌人。
6. 不需要复杂掩体，只要能巡逻、发现、报警。

验收：
- 玩家进入哨兵视野，哨兵能发现玩家。
- 据点警报被触发。
- ThreatValue 增加。
- 其他人类敌人进入战斗状态。
```

### Prompt 8：人物经验与属性点

```text
请实现人物成长系统第一版。

创建：
- ExperienceService.cs
- CharacterGrowthState.cs
- CharacterAttributeType.cs
- DerivedStatsCalculator.cs

要求：
1. 角色有 level、currentExp、availableAttributePoints。
2. 获得经验后可升级。
3. 每次升级获得 1 点属性点。
4. 五大属性为 Body、Agility、Firearms、Survival、Engineering。
5. 属性点可分配到五大属性。
6. DerivedStatsCalculator 根据属性输出移动速度、准确性、负重、生存消耗等修正。
7. 不需要完整 UI，可先用 Debug Inspector 或测试方法验证。

验收：
- 完成一次撤离后获得经验。
- 经验足够时升级。
- 升级后获得属性点。
- 分配属性点后衍生属性发生变化。
```

---

## 31. 测试与验收

### 31.1 EditMode 测试

应测试：

```text
WeaponInstance 属性合成
InventoryGrid 放置 / 移除物品
ThreatManager 等级计算
ConsumableItemDefinition 效果解析
SurvivalStatus 阶段变化
CharacterGrowth 升级与属性点
WeaponProficiency 经验累计
Mission 条件判断
SaveData 序列化
```

### 31.2 PlayMode 测试

应测试：

```text
玩家生成
玩家移动和鼠标瞄准
武器射击命中敌人
敌人追踪玩家
人类哨兵发现玩家
据点警报触发
LootContainer 开箱
局内使用罐头 / 水 / 医疗包 / 能量饮料
ExtractionZone 倒计时完成
Raid 成功 / 失败结算
回基地后仓库和人物成长保存
```

### 31.3 每日构建检查

```text
项目可打开。
所有场景无 Missing Script。
Console 无红色报错。
可打包 Windows 版本。
MVP 垂直切片流程可玩通。
玩家至少能完成一次：出发 -> 搜刮 -> 战斗 -> 使用补给 -> 撤离 -> 结算 -> 成长。
```

---

## 32. Steam 发布准备

最低需要：

1. Capsule 主图。
2. 截图 5-8 张。
3. 30-60 秒短预告片。
4. 游戏简介。
5. 标签。
6. 成熟内容说明。
7. 配置需求。
8. Demo 包。

Steam 页面卖点建议突出：

1. 俯视角血腥怪潮射击。
2. 单机 PVE 搜打撤。
3. 局内搜到食物、水、药品可直接使用。
4. 人类据点与三方混战。
5. 枪械改装。
6. 饥饿、口渴、疲劳轻量生存。
7. 人物成长。
8. 三张连续地图，从农场推进到工厂和小城镇。

---

## 33. 最终执行原则

本项目不是开放世界生存，也不是多人搜打撤。

本项目的第一优先级是：

```text
俯视角射击爽感 + 搜刮撤离张力 + 局内补给取舍 + 人类据点变化 + 轻量成长循环
```

任何新增系统都必须服从以下判断：

1. 是否增强射击爽感？
2. 是否增强搜打撤取舍？
3. 是否增强三张地图的推进感？
4. 是否增强生存资源价值？
5. 是否增强撤离后的长期成长？
6. 是否能在 50 万预算和小团队能力内完成？

如果答案是否定，则延后或删除。
