# VS版本阶段一迁移候选清单

## 一、文档定位

本文档是 `VS版本` 阶段一的程序交付物，用于记录从 `TopDownDemo` 参考内容中筛选出的可迁移对象、可参考思路、目标落点、原创命名和禁止迁移范围。

本文档只服务后续 `VS版本` 开发，不代表最终正式游戏架构，不扩展策划玩法，不直接替代具体代码实现。

## 二、阶段一目标

阶段一目标是完成迁移前置判断，保证后续阶段开发时不会盲目复制参考模板。

阶段一需要达成以下结果：

1. 明确哪些内容可以直接复制后改名。
2. 明确哪些内容只能参考思路后重写。
3. 明确每个可用内容进入 `Assets/Game` 后的正式目录。
4. 明确每个可用内容进入项目后的原创命名方向。
5. 明确哪些模板内容不能进入 `VS版本` 正式架构。
6. 明确 `Assets/Game/Tests/Combat/TestCombat.unity` 后续搭建需要的最小资源来源。

## 三、扫描范围

| 扫描对象 | 扫描用途 |
|---|---|
| `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Scenes/Top Down Demo/Top Down Demo.unity` | 确认参考场景实际使用的对象、脚本、预制体、材质、音效和场景资源。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Game/Camera Prefabs/TopDown Camera Controller.prefab` | 确认俯视角摄像机结构、跟随方式、视野参数和状态配置。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Textures/UI/HUD` | 确认准星、血条、槽位等基础 HUD 素材是否可迁移。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Items/Weapons/Guns` | 确认基础枪械模型和预制体是否可作为 VS 占位武器来源。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Effects/Bullet Hit` | 确认基础命中特效是否可作为 VS 占位反馈来源。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Audio/Guns` | 确认基础开火、换弹、空仓等武器音效是否可迁移。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Audio/FX` | 确认命中、冲击、提示等战斗反馈音效是否可迁移。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Models/Prototype Models` | 确认灰盒地面、墙体、平台、障碍物等测试场景资源是否可迁移。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Scripts` | 确认可参考的摄像机、瞄准、生命、伤害、准星、敌人追踪脚本思路。 |

## 四、场景依赖扫描结论

`Top Down Demo.unity` 直接依赖的参考资源类型包括：

| 类型 | 结论 |
|---|---|
| 场景资源 | 依赖 `NavMesh.asset`、灯光数据和后处理配置，但 VS 阶段不需要原样复制完整场景烘焙数据。 |
| 摄像机预制体 | 依赖 `TopDown Camera Controller.prefab`，可参考或复制参数结构，但正式预制体必须重新命名。 |
| 玩家预制体 | 依赖 `Top Down JU Character.prefab`，耦合角色控制、背包、IK、驾驶等系统，不建议直接成为正式玩家预制体。 |
| 武器预制体 | 依赖多把枪械、近战和投掷物，VS 阶段只允许选择一把基础枪械作为占位武器来源。 |
| UI 预制体 | 依赖模板 UI 根节点和移动端控制界面，VS 阶段只允许拆出准星、血条、弹药显示等最小表现资源。 |
| 怪物预制体 | 依赖 `Zombi AI Sample.prefab` 和复杂 AI 脚本，VS 阶段只参考追踪压力，不直接继承完整 AI。 |
| 车辆预制体 | 依赖 Bike、Car、Motorcycle 等车辆内容，VS 阶段全部排除。 |
| 音频资源 | 枪声、命中、脚步等音效可筛选后复制并改名，优先用于开火和命中反馈。 |
| 脚本资源 | 多数脚本存在模板命名和高耦合依赖，只能作为设计参考，不直接复制为正式脚本。 |

## 五、可直接复制后改名的候选资源

以下内容允许在后续阶段从 `Assets/ProjectRef` 复制到 `Assets/Game`，但复制后必须重新归类、重新命名，并清除对参考目录的依赖。

| 候选内容 | 参考来源 | 目标目录 | 原创命名方向 | 用途 | 优先级 |
|---|---|---|---|---|---|
| 俯视角摄像机参数结构 | `Prefabs/Game/Camera Prefabs/TopDown Camera Controller.prefab` | `Assets/Game/Prefabs/Player` 或 `Assets/Game/Prefabs/Camera` | `CZ_CombatCameraRig`、`TopDownCombatCameraRig` | 快速确定摄像机高度、角度、视野和跟随方式。 | 高 |
| 准星纹理 | `Textures/UI/HUD/Crosshair.png`、`Crosshair00.png`、`Crosshair01.png` | `Assets/Game/Art/UI` | `CZ_Crosshair_Core`、`CZ_Crosshair_Spread` | 建立 VS 基础准星显示。 | 高 |
| 血条纹理 | `Textures/UI/HUD/Bar.png`、`OutlineBar.png` | `Assets/Game/Art/UI` | `CZ_HudHealth_Bar`、`CZ_HudHealth_Frame` | 建立玩家生命显示。 | 高 |
| 弹药或槽位纹理 | `Textures/UI/HUD/Slot.png`、`SlotOutline.png`、`EmptySlot.png` | `Assets/Game/Art/UI` | `CZ_HudAmmo_Slot`、`CZ_HudAmmo_Frame` | 建立基础弹药和当前武器显示。 | 中 |
| 基础命中特效 | `Prefabs/Effects/Bullet Hit/Default Bullet Hit.prefab` | `Assets/Game/Prefabs/VFX` | `CZ_BulletImpact_Default` | 命中地面、墙体或怪物时显示反馈。 | 高 |
| 人形命中特效 | `Prefabs/Effects/Bullet Hit/Human Skin Bullet Hit.prefab` | `Assets/Game/Prefabs/VFX` | `CZ_BulletImpact_Flesh` | 怪物受击反馈占位。 | 中 |
| 基础枪械模型 | `Demos/Models/Items/Weapons Models/Guns/P226/Model/P226 PISTOLA.fbx` | `Assets/Game/Art/Weapons` | `CZ_BasicSidearm_Model` | VS 阶段单武器占位模型。 | 高 |
| 基础枪械材质 | `Demos/Models/Items/Weapons Models/Guns/P226/Materials` | `Assets/Game/Materials` 或 `Assets/Game/Art/Weapons` | `CZ_BasicSidearm_Mat` | 配套基础枪械模型显示。 | 高 |
| 基础枪械预制体 | `Demos/Demo Prefabs/Items/Weapons/Guns/P226.prefab` | `Assets/Game/Prefabs/Weapons` | `CZ_BasicSidearm_Weapon` | 作为拆解参考或复制后重建正式武器。 | 中 |
| 测试靶预制体 | `Demos/Demo Prefabs/ShootingTarget/ShootingTarget.prefab` | `Assets/Game/Prefabs/Enemies` 或 `Assets/Game/Tests/Combat` | `CZ_CombatTarget_Dummy` | 在怪物完成前验证命中和扣血。 | 中 |
| 灰盒地面模型 | `Demos/Models/Prototype Models/MeshTerrain.fbx`、`Plataform.fbx` | `Assets/Game/Art/Environments` 或 `Assets/Game/Tests/Combat` | `CZ_TestArena_Floor` | 搭建 `TestCombat` 测试地面。 | 高 |
| 灰盒障碍模型 | `Demos/Models/Prototype Models/Stair01.fbx`、`Stair02.fbx`、`Ramp.fbx`、`SemiSpheres.fbx` | `Assets/Game/Art/Environments` 或 `Assets/Game/Tests/Combat` | `CZ_TestArena_Blocker`、`CZ_TestArena_Obstacle` | 搭建射击遮挡和移动障碍。 | 中 |
| 灰盒材质 | `Demos/Materials/Prototype Props` | `Assets/Game/Materials` 或 `Assets/Game/Tests/Combat` | `CZ_TestArena_GridGray`、`CZ_TestArena_WallBlack` | 测试场景视觉区分。 | 中 |
| 枪械音效 | `Audio/Guns/PistolShot.wav`、`pistol-shot-1.mp3`、`empty-gun-shot.wav`、`Reload.ogg` | `Assets/Game/Audio/Weapons` | `CZ_Audio_Weapon_PistolShot`、`CZ_Audio_Weapon_Empty` | 开火、空仓、换弹反馈。 | 中 |
| 命中音效 | `Audio/FX/bullet-hit.ogg`、`hit-marker.ogg`、`madpancake__hit-impact.ogg` | `Assets/Game/Audio/Weapons` 或 `Assets/Game/Audio/Enemies` | `CZ_Audio_Hit_Default`、`CZ_Audio_HitMarker` | 命中反馈和击中提示。 | 中 |
| 基础玩家模型 | `Demos/Models/Characters/JU Mannequin/JU Mannequin.fbx` | `Assets/Game/Art/Characters/Player` | `CZ_Player_Prototype_Model` | 如果暂时没有正式玩家模型，可作为 VS 占位外观。 | 低 |
| 基础怪物动画 | `Animations/Humanoid Zombie Animations` | `Assets/Game/Art/Enemies` | `CZ_Infected_Walk`、`CZ_Infected_Run`、`CZ_Infected_Attack` | 如果后续需要临时怪物表现，可筛选后复制。 | 低 |

## 六、只能参考思路后重写的候选逻辑

以下内容不建议直接复制脚本到 `Assets/Game`，但可以提取设计思路，用当前项目命名空间重新实现。

| 参考逻辑 | 参考脚本 | 正式实现目录 | 正式命名方向 | 重写要点 |
|---|---|---|---|---|
| 鼠标射线转世界瞄准点 | `AimOnMousePosition.cs` | `Assets/Game/Scripts/Player` | `PlayerAimController`、`MouseWorldAimProvider` | 只保留鼠标射线命中地面得到瞄准点的思路，不使用模板静态字段和动作系统。 |
| 摄像机状态跟随 | `TDCameraController.cs`、`JUCameraSystemLib.cs` | `Assets/Game/Scripts/Camera` | `TopDownCameraRig`、`CombatCameraFollower` | 只保留高度、距离、视野、跟随目标和插值移动，不继承模板状态机。 |
| 摄像机目标查找 | `JUCameraSystemLib.cs` | `Assets/Game/Scripts/Camera` | `CameraFollowTargetBinder` | 不使用模板通过 `Player` 标签自动绑定的隐式方式，改为场景显式引用或启动器绑定。 |
| 武器枪口方向修正 | `WeaponAimRotationCenter.cs` | `Assets/Game/Scripts/Weapons` | `WeaponMuzzleAimAligner` | 只保留枪口朝向瞄准点的数学关系，不保留模板武器挂点列表管理。 |
| 单武器开火链路 | 模板枪械预制体和武器管理脚本 | `Assets/Game/Scripts/Weapons` | `BasicWeaponController`、`WeaponFireInput` | 只做一把武器、射速、弹药、开火事件，不接入完整背包和换枪。 |
| 子弹忽略发射者 | `Damager.cs` | `Assets/Game/Scripts/HitDetection` | `HitResolver`、`DamageQuery` | 只保留忽略发射者碰撞体和命中标签过滤思路。 |
| 基础生命扣血死亡 | `JUHealth.cs` | `Assets/Game/Scripts/Player`、`Assets/Game/Scripts/Enemies` | `HealthComponent`、`DeathStateController` | 重写为轻量生命组件，保留扣血、死亡事件、最大生命限制。 |
| 准星精度反馈 | `Crosshair.cs` | `Assets/Game/Scripts/UI` | `CrosshairView`、`CombatAimReticle` | 只保留准星跟随鼠标、准星颜色、散布大小变化思路。 |
| 怪物追踪压力 | `ZombieAI.cs` | `Assets/Game/Scripts/Enemies` | `EnemyChaseController`、`BasicInfectedEnemy` | 不继承复杂 AI 基类，只实现追踪玩家、接近、受击死亡。 |
| 命中反馈播放 | `Damager.cs`、命中特效预制体 | `Assets/Game/Scripts/CombatFeedback` | `HitFeedbackSpawner`、`CombatFeedbackPlayer` | 只保留命中点生成特效和音效播放，不保留模板表面系统。 |
| 开火镜头反馈 | 相机控制器和枪械脚本 | `Assets/Game/Scripts/CombatFeedback`、`Assets/Game/Scripts/Camera` | `CameraShakeReceiver`、`WeaponFireFeedback` | VS 阶段只做轻量震动或可选后坐反馈。 |

## 七、不进入 VS 版本的模板内容

以下内容明确排除，不允许作为 `VS版本` 正式架构直接迁入：

| 排除内容 | 参考路径或对象 | 排除原因 |
|---|---|---|
| 巨型角色控制器 | `JUCharacterController.cs`、`Top Down JU Character.prefab` | 耦合移动、动画、装备、动作状态、驾驶、IK 等大量内容，不适合 VS 最小验证。 |
| 全能背包系统 | `JUInventory.cs`、物品拾取和装备链路 | VS 阶段只需要单武器，不需要完整背包、物品格和装备槽。 |
| 完整换枪系统 | `ItemSwitchManager.cs`、多武器 Prefab | VS 阶段只验证一把基础武器，不需要多武器切换。 |
| 车辆驾驶系统 | `DriveVehicles.cs`、Bike、Car、Motorcycle 等 Prefab | VS 阶段不包含驾驶，不应引入车辆物理和驾驶状态。 |
| 复杂 IK 系统 | `JUFootPlacement.cs`、持枪 IK 和脚步 IK | VS 阶段优先验证战斗闭环，不需要复杂动画校正。 |
| 静态全局管理器 | 模板全局管理类和静态入口 | 当前项目需要保持独立模块连接，不继承模板全局状态。 |
| 混合 UI 根节点 | `JUTPS TopDown User Interface.prefab`、`JUTPS Sidescroller User Interface.prefab` | UI 根节点混合了背包、快捷栏、移动端控制等内容，VS 阶段只拆出 HUD 思路。 |
| 移动端虚拟摇杆 | `Controls On Screen UI.prefab`、移动端 Rig | 当前 VS 验证以键鼠为主，不引入移动端控制。 |
| TPS 巡逻 AI | `PatrolAI.cs`、`WaypointPath.cs` | VS 阶段只需要基础追踪怪物，不需要巡逻路径和复杂感知。 |
| 后处理完整配置 | `PC DESKTOP.asset` 和模板渲染设置 | VS 阶段不依赖模板后处理作为战斗闭环条件。 |
| 原场景烘焙数据 | `NavMesh.asset`、`LightingData.asset` | `TestCombat` 会重新搭建，不继承参考场景烘焙数据。 |
| 教程提示触发器 | `UIMenssengerTrigger.cs`、InfoIcon 等对象 | VS 阶段不做模板教程提示。 |

## 八、正式命名对照建议

后续阶段复制或重写时，必须使用当前项目原创命名。以下是建议对照表：

| 参考命名或对象 | 正式命名方向 | 正式目录 |
|---|---|---|
| `TopDown Camera Controller` | `CZ_CombatCameraRig` | `Assets/Game/Prefabs/Player` 或 `Assets/Game/Prefabs/Camera` |
| `TDCameraController` | `TopDownCameraRig` | `Assets/Game/Scripts/Camera` |
| `AimOnMousePosition` | `MouseWorldAimProvider` | `Assets/Game/Scripts/Player` |
| `WeaponAimRotationCenter` | `WeaponMuzzleAimAligner` | `Assets/Game/Scripts/Weapons` |
| `Damager` | `HitResolver` 或 `DamageVolume` | `Assets/Game/Scripts/HitDetection` |
| `JUHealth` | `HealthComponent` | `Assets/Game/Scripts/Player`、`Assets/Game/Scripts/Enemies` |
| `Crosshair` | `CrosshairView` | `Assets/Game/Scripts/UI` |
| `ZombieAI` | `EnemyChaseController` | `Assets/Game/Scripts/Enemies` |
| `Zombi AI Sample` | `CZ_BasicInfectedEnemy` | `Assets/Game/Prefabs/Enemies` |
| `P226` | `CZ_BasicSidearm_Weapon` | `Assets/Game/Prefabs/Weapons` |
| `Default Bullet Hit` | `CZ_BulletImpact_Default` | `Assets/Game/Prefabs/VFX` |
| `Human Skin Bullet Hit` | `CZ_BulletImpact_Flesh` | `Assets/Game/Prefabs/VFX` |
| `Crosshair.png` | `CZ_Crosshair_Core` | `Assets/Game/Art/UI` |
| `Bar.png` | `CZ_HudHealth_Bar` | `Assets/Game/Art/UI` |
| `MeshTerrain` | `CZ_TestArena_Floor` | `Assets/Game/Tests/Combat` 或 `Assets/Game/Art/Environments` |
| `Black Wall` | `CZ_TestArena_WallBlack` | `Assets/Game/Materials` |

## 九、阶段二优先使用组合

为了让 `Assets/Game/Tests/Combat/TestCombat.unity` 尽快成立，阶段二建议优先处理以下组合：

| 组合 | 内容 | 原因 |
|---|---|---|
| 场景灰盒组合 | 地面、墙体、障碍、出生点、怪物点位 | 最快建立移动和射击验证区域。 |
| 摄像机组合 | 俯视角高度、旋转、视野、跟随目标 | 直接决定俯视角手感。 |
| 玩家组合 | 玩家胶囊体、移动脚本、瞄准点脚本 | 先用占位体也可以验证移动射击。 |
| 武器组合 | 基础手枪模型、开火脚本、枪口点、音效 | 先完成一把武器即可验证战斗闭环。 |
| 命中组合 | 轻量生命组件、命中解析、命中特效 | 保证开火后能看到结果。 |
| 怪物组合 | 占位怪物、追踪脚本、生命组件 | 形成最小打怪压力。 |
| HUD 组合 | 准星、生命条、弹药文本 | 提供最基础可读反馈。 |

## 十、阶段一验收结果

| 验收项 | 状态 | 说明 |
|---|---|---|
| 候选清单中每项都有目标目录 | 已完成 | 文档第五节已经列出目标目录。 |
| 候选清单中每项都有新命名 | 已完成 | 文档第五节和第八节已经列出原创命名方向。 |
| 候选清单中没有保留模板业务命名 | 已完成 | 正式命名方向均以当前项目语义和 `CZ` 规则为准。 |
| 候选清单没有把 `Assets/ProjectRef` 作为运行路径 | 已完成 | `Assets/ProjectRef` 只作为扫描来源和复制来源。 |
| 已排除不进入 VS 的模板内容 | 已完成 | 文档第七节已经列出排除范围和原因。 |

## 十一、后续执行要求

1. 阶段二开始后，应先创建或整理 `Assets/Game/Tests/Combat/TestCombat.unity`。
2. 如果复制参考资源，必须复制到 `Assets/Game` 下目标目录后再使用。
3. 不允许让 `TestCombat.unity` 直接引用 `Assets/ProjectRef`。
4. 不允许把模板脚本直接放入正式业务目录后继续保留原命名空间。
5. 如果某项资源复制后仍依赖大量模板脚本，应放弃直接复制，改为当前项目重建。
6. 如果某项资源只是临时测试脚本，应放入 `Assets/TmpTests`，不能放入 `Assets/Game/Tests`。
7. 阶段二到阶段七开发期间，如发现新候选资源，应回写本文档或另建对应迁移记录。
