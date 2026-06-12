
# Top Down Demo 完整技术拆解

## 一、文档目标

本文档针对当前项目中的俯视角参考场景进行完整技术拆解。分析对象是：

`Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Scenes/Top Down Demo/Top Down Demo.unity`

该场景来自 `JU TPS Controller` 模板，当前被放入 `Assets/ProjectRef` 作为参考内容。它不是最终类孤胆枪手项目的目标架构，但它已经包含俯视角射击、角色移动、鼠标瞄准、换枪、拾取、车辆驾驶、简单敌人、UI、后处理等完整 Demo 逻辑，非常适合作为后续重构和功能拆分的技术参考。

本文档重点拆解每个系统做什么、依赖哪些脚本、运行时如何连接、功能之间如何传递状态、哪些内容适合复用、哪些内容建议后续重构。

## 二、场景总体结构

### 2.1 场景根对象

| 对象 | 作用 |
|---|---|
| `Zombie Spawner` | 自动生成敌人或 AI 示例对象。 |
| `PlayGround` | 地形、障碍、射击靶、提示触发器、墙体等演示环境。 |
| `Pickable Items` | 可拾取武器、近战武器、投掷物、护甲、补给等。 |
| `Info Triggers` | 教学提示触发区域。 |
| `Waypoint Path` | AI 巡逻路径点集合。 |
| `Post Processing Volume` | 后处理体积，依赖 `com.unity.postprocessing`。 |
| `Pause` | 暂停逻辑对象。 |
| `Physical Cubes` | 可物理交互的测试方块。 |
| `Directional Light` | 场景主光源。 |

### 2.2 核心 Prefab

| Prefab | 作用 |
|---|---|
| `Top Down JU Character.prefab` | 玩家角色主 Prefab。 |
| `TopDown Camera Controller.prefab` | 俯视角摄像机控制器。 |
| `JUTPS Sidescroller User Interface.prefab` | 当前场景使用的 UI 根节点，内部包含输入、准星、血条、物品信息、暂停等通用 UI。 |
| `Car.prefab`、`Motorcycle.prefab`、`Bike.prefab` | 可驾驶车辆示例。 |
| `Patrol AI Sample.prefab` | 巡逻 AI 示例。 |
| 多个枪械、近战、投掷物 Prefab | 场景中的拾取物与角色背包内物品来源。 |

### 2.3 当前资源组织

模板内容全部位于：

`Assets/ProjectRef/Julhiecio TPS Controller`

这个目录应视为参考库。正式项目建议另建业务目录，例如 `Assets/Game`。后续不要直接在 `ProjectRef` 内做最终游戏业务逻辑，除非明确是在调试模板 Demo。

## 三、系统总览

核心运行链路如下：

```text
Unity Input System
        ↓
JUInputManager 全局输入缓存
        ↓
JUCharacterController 读取移动、瞄准、射击、换枪、拾取、上车输入
        ↓
角色移动 / 瞄准 / 动画 / 武器使用 / 拾取 / 驾驶
        ↓
Weapon、JUInventory、DriveVehicles、TDCameraController、UI 等系统根据角色状态同步表现
```

| 系统 | 核心脚本 | 主要职责 |
|---|---|---|
| 输入系统 | `JUInputManager.cs`、`JUTPSInputControlls.cs` | 统一读取新输入系统，转换为静态查询接口。 |
| 全局管理 | `JUGameManager.cs` | 查找玩家、记录移动端模式。 |
| 角色控制 | `JUCharacterController.cs`、`JUCharacterControllerCore.cs` | 移动、旋转、动画、IK、武器控制、状态管理。 |
| 俯视角摄像机 | `TDCameraController.cs`、`JUCameraSystemLib.cs` | 跟随玩家，根据玩家状态切换镜头状态。 |
| 鼠标/摇杆瞄准 | `AimOnMousePosition.cs`、`AimOnRightJoystickDirection.cs`、`AimControllSwitcher.cs` | 设置角色 `LookAtPosition`，决定开火方向。 |
| 背包与拾取 | `JUInventory.cs` | 管理所有物品、拾取、装备、丢弃、当前手持物。 |
| 换枪 | `ItemSwitchManager.cs` | 处理数字键、滚轮、上一件、下一件，调用角色或背包装备物品。 |
| 武器 | `Weapon.cs`、`Bullet.cs`、`Damager.cs` | 开火、弹药、散布、子弹实例、伤害、特效、音效。 |
| 近战与投掷 | `MeleeWeapon.cs`、`ThrowableItem.cs`、`Granade.cs` | 近战动画、伤害器启用、投掷物生成。 |
| 驾驶 | `DriveVehicles.cs`、`Vehicle`、`CarController.cs`、`MotorcycleController.cs`、`JUVehicleCharacterIK.cs` | 查找车辆、进入、退出、驾驶输入、车辆物理、驾驶 IK。 |
| AI | `PatrolAI.cs`、`ZombieAI.cs`、`JUCharacterArtificialInteligenceBrain.cs`、`WaypointPath.cs` | 视野检测、巡逻、追踪、攻击、路径跟随。 |
| UI | `Crosshair.cs`、`UIHealhBar.cs`、`UIItemInformation.cs`、`JUPauseGame.cs` | 准星、血条、当前物品、暂停、提示。 |
| 生成器 | `JUAutoInstantiate.cs` | 定时随机生成敌人或对象。 |
| 后处理 | `PostProcessVolume.cs` | 视觉后处理。 |

## 四、输入系统拆解

### 4.1 设计目的

输入系统的目的是把 Unity 新输入系统产生的动作事件，统一转成模板内部可以静态查询的接口。这样角色、车辆、武器和 UI 不需要直接关心具体输入设备。

### 4.2 相关脚本

| 脚本 | 作用 |
|---|---|
| `Assets/ProjectRef/Julhiecio TPS Controller/Inputs/JUTPSInputControlls.cs` | 由 `.inputactions` 自动生成的输入包装类。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Inputs/JUTPSInputControlls.inputactions` | 输入动作资源。 |
| `Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/JU Input/JUInputManager.cs` | 模板输入管理器。 |

### 4.3 输入缓存字段

| 缓存字段 | 含义 |
|---|---|
| `MoveHorizontal`、`MoveVertical` | 角色移动轴。 |
| `RotateHorizontal`、`RotateVertical` | 视角或摇杆瞄准轴。 |
| `PressedShooting`、`PressedShootingDown`、`PressedShootingUp` | 射击键按住、按下、抬起。 |
| `PressedAiming` | 瞄准键按住。 |
| `PressedReload` | 换弹键。 |
| `PressedPickup` | 拾取键。 |
| `PressedInteract` | 交互键，上车和下车使用。 |
| `PressedNextItem`、`PressedPreviousItem` | 切换物品。 |
| `PressedOpenInventoryDown` | 打开背包。 |

### 4.4 输入管理器运行流程

每帧 `JUInputManager.Update()` 做以下事情：

1. 判断最近使用的是手柄还是键鼠，写入 `IsUsingGamepad`。
2. 如果 `BlockStandardInputs` 为真，则停止更新默认输入。
3. 调用 `UpdateGetButtonDown()` 更新按下瞬间。
4. 调用 `UpdateGetButton()` 更新按住状态。
5. 通过输入事件回调更新按键抬起状态。
6. 调用 `UpdateAxis()` 更新移动轴和旋转轴。

其他系统通过静态接口读取输入：

```text
JUInput.GetAxis(...)
JUInput.GetButton(...)
JUInput.GetButtonDown(...)
JUInput.GetButtonUp(...)
```

### 4.5 输入到角色的连接

`JUCharacterController.Update()` 会调用 `ControllerInputs()`，内部读取：

```text
ShotInput        ← JUInput.GetButton(ShotButton)
ShotInputDown    ← JUInput.GetButtonDown(ShotButton)
ReloadInput      ← JUInput.GetButtonDown(ReloadButton)
AimInput         ← JUInput.GetButton(AimingButton)
HorizontalX      ← JUInput.GetAxis(MoveHorizontal)
VerticalY        ← JUInput.GetAxis(MoveVertical)
```

输入系统并不直接移动角色，而是只提供状态。真正的移动、开火、换弹、翻滚、蹲伏、拾取、上车逻辑都在角色或其他系统中执行。

## 五、全局管理系统拆解

### 5.1 `JUGameManager`

`JUGameManager` 挂在 UI 根节点上，负责记录玩家引用和移动端状态。

核心逻辑：

1. `Awake()` 设置静态 `Instance`。
2. `Start()` 如果 `PlayerController` 为空，则调用 `GetPlayer()`。
3. `GetPlayer()` 通过 `GameObject.FindGameObjectWithTag("Player")` 查找玩家，再获取 `JUCharacterController`。
4. `Update()` 根据 `SimulateMobileDevice` 更新 `IsMobileControls`。
5. `OnDestroy()` 清空静态玩家引用。

### 5.2 与其他系统的连接

| 系统 | 使用方式 |
|---|---|
| UI 血条 | 查找玩家的 `JUHealth`。 |
| 当前物品 UI | 通过 `JUGameManager.PlayerController` 获取当前手持物。 |
| 准星 | 通过玩家和摄像机状态更新。 |
| 瞄准控制切换 | 使用 `JUGameManager.IsMobileControls` 判断是否使用移动端瞄准。 |

### 5.3 风险点

`JUGameManager` 使用静态字段和标签查找，适合 Demo，但不适合大型项目。正式项目建议改成明确的场景服务或依赖注入，避免多玩家、切场景、复活时引用残留。

## 六、俯视角摄像机系统拆解

### 6.1 设计目的

俯视角摄像机负责跟随玩家，并根据玩家状态在普通、开火、瞄准、驾驶、死亡状态之间平滑切换镜头参数。

### 6.2 相关资源和脚本

| 内容 | 路径 |
|---|---|
| 摄像机 Prefab | `Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Game/Camera Prefabs/TopDown Camera Controller.prefab` |
| 俯视角控制脚本 | `Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Camera Controllers/TDCameraController.cs` |
| 摄像机基类 | `Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Libraries/JUCameraSystemLib.cs` |

### 6.3 核心类结构

`TDCameraController` 继承自 `JUCameraController`。

`JUCameraController` 提供通用能力：

| 能力 | 说明 |
|---|---|
| `TargetToFollow` | 摄像机跟随目标。 |
| `mCamera` | 子级真实 Camera 组件。 |
| `CurrentCameraState` | 当前镜头状态。 |
| `SetPivotCameraPosition()` | 移动摄像机枢轴。 |
| `SetCameraPosition()` | 移动真实 Camera。 |
| `SetFieldOfView()` | 设置视野。 |
| `SetCameraStateTransition()` | 平滑过渡镜头状态。 |
| `RotateCamera()` | 通用旋转逻辑，俯视 Demo 基本不依赖玩家自由旋转。 |

`CameraState` 是镜头状态数据结构，包含距离、跟随速度、视野、目标偏移、相机偏移、旋转灵敏度、角度限制和碰撞层。

### 6.4 俯视角状态机

| 状态 | 对应字段 | 触发条件 |
|---|---|---|
| 普通 | `NormalCameraState` | 玩家未开火、未瞄准、未驾驶、未死亡。 |
| 开火 | `FireModeCameraState` | `character.FiringMode == true`。 |
| 瞄准 | `AimModeCameraState` | `character.IsAiming == true`。 |
| 驾驶 | `DrivingVehicleCameraState` | `character.IsDriving == true`。 |
| 死亡 | `DeadPlayerCameraState` | `character.IsDead == true`。 |

`TDCameraController.Update()` 执行：

```text
UpdateCharacterState(PlayerTarget)
ChangeCameraStateAccordingCharacterState(CharacterState)
```

`FixedUpdate()` 移动摄像机枢轴，`LateUpdate()` 设置真实摄像机位置和视野。

### 6.5 摄像机如何找到玩家

`JUCameraController.Start()` 会通过 `GameObject.FindGameObjectWithTag("Player")` 查找玩家。`TDCameraController.Start()` 如果发现目标上有 `JUCharacterController`，则：

```text
PlayerTarget = JUCharacterController
TargetToFollow = PlayerTarget.HumanoidSpine
```

也就是说，摄像机真正跟随的是玩家骨骼中的 `HumanoidSpine`，不是角色根节点。

### 6.6 与角色状态的连接

摄像机只读取玩家状态，不直接控制玩家。它读取：

```text
IsAiming
IsDriving
FiringMode
IsDead
```

这些状态由 `JUCharacterController`、`DriveVehicles`、`Weapon`、`JUHealth` 等系统改变。摄像机根据这些状态选择不同的 `CameraState`。

## 七、角色控制系统拆解

### 7.1 设计目的

`JUCharacterController` 是整个 Demo 的中心系统，负责读取输入、移动、旋转、地面检测、动画、IK、武器使用、拾取、驾驶检查和生命状态处理。

### 7.2 核心脚本

| 脚本 | 作用 |
|---|---|
| `JUCharacterController.cs` | 角色主控制循环。 |
| `JUCharacterControllerCore.cs` | 角色基础字段、移动函数、物品函数、状态函数。 |
| `JUHealth.cs` | 生命、受伤、死亡事件。 |
| `JUSlipCapsule.cs` | 额外胶囊碰撞体，降低角色卡边。 |
| `JUFootPlacement.cs` | 脚部 IK。 |
| `BodyLeanInert.cs` | 身体倾斜惯性。 |
| `JUFootstep.cs` | 脚步声。 |
| `ProceduralDrivingAnimation.cs` | 驾驶时身体程序动画。 |

### 7.3 玩家 Prefab 关键挂载

`Top Down JU Character.prefab` 上核心脚本包括：

| 脚本 | 用途 |
|---|---|
| `JUCharacterController` | 主角色控制。 |
| `JUHealth` | 生命值。 |
| `JUInventory` | 背包和物品。 |
| `ItemSwitchManager` | 换枪和换物品。 |
| `DriveVehicles` | 上车、下车、驾驶状态。 |
| `WeaponAimRotationCenter` | 武器手持位置和旋转中心。 |
| `AimOnMousePosition` | 鼠标瞄准。 |
| `AimOnRightJoystickDirection` | 手柄右摇杆瞄准。 |
| `AimControllSwitcher` | 根据设备切换瞄准方式。 |
| `AdvancedRagdollController` | 布娃娃和物理死亡。 |
| `DamageableBody`、`DamageableBodyPart` | 身体部位受击。 |
| `Armor` | 护甲部件。 |

### 7.4 主循环拆解

`FixedUpdate()`：

```text
如果死亡、全局禁用移动、暂停，则返回
Movement()
SlopeSlide()
StepCorrectionMovement()
```

`Update()`：

```text
如果暂停，则返回
FootPlacementIKController()
GroundCheck()
HealthCheck()
SetAnimatorParameters()
SetupDefaultLayersWeights()
如果死亡，则返回
DrivingCheck()
WallAHeadCheck()
如果没有禁用移动：
    ControllerInputs()
    StepCorrectionCalculation()
    Rotate(HorizontalX, VerticalY)
    RefreshItemAimRotationPivot()
    WieldingIKWeightController()
    WeaponOrientator()
否则：
    腿部动画层权重归零
Events.UpdateRuntimeEventsCallbacks(this)
```

### 7.5 移动逻辑

移动由 `HorizontalX`、`VerticalY`、`VelocityMultiplier`、`MovementMode`、`DoFreeMovement()`、`DoFireModeMovement()`、`Rotate()`、`GroundCheck()`、`SlopeSlide()`、`StepCorrectionCalculation()` 等函数共同完成。

`Movement()` 内部流程：

1. 调用 `LocomotionModeController()` 更新移动模式。
2. 如果正在布娃娃，停止瞄准、开火、驾驶和 IK。
3. 根据 `RootMotion` 决定 Animator 是否使用根运动。
4. 如果翻滚，则强制执行翻滚移动。
5. 如果 `CanMove == false`，速度归零并返回。
6. 执行空中移动控制。
7. 判断 `IsMoving`。
8. 调用普通移动和开火模式移动。
9. 处理换枪动画计时，恢复开火 IK。

### 7.6 角色旋转逻辑

`Rotate(HorizontalX, VerticalY)` 分两种情况：

| 情况 | 行为 |
|---|---|
| 普通移动 | 根据摄像机朝向和移动输入计算目标方向，角色朝移动方向转。 |
| 开火模式 | 如果 `FiringMode == true`，角色朝 `LookAtPosition` 或摄像机前方目标点转。 |

俯视角 Demo 中，鼠标瞄准脚本会持续写入 `TPSCharacter.LookAtPosition = AimPosition`。因此开火模式下，角色会朝鼠标指向位置旋转。

### 7.7 动画和 IK

角色动画由 `Animator` 和多个脚本共同驱动。`SetAnimatorParameters()` 会更新移动、速度、开火、蹲伏、趴下、跳跃、物品姿势等参数。

IK 分为手部 IK、武器旋转中心、脚部 IK 和驾驶 IK。`WieldingIKWeightController()` 根据当前手持物、开火状态、换枪状态、驾驶状态调整 IK 权重。

## 八、瞄准系统拆解

### 8.1 设计目的

俯视角射击的关键是让角色身体和武器朝鼠标或摇杆指向的位置旋转。Demo 将瞄准位置作为一个世界坐标传给角色：

```text
JUCharacterController.LookAtPosition
```

之后角色旋转和武器朝向都会围绕这个位置更新。

### 8.2 鼠标瞄准

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Character Controllers/Additionals/AimOnMousePosition.cs`

设计逻辑：

1. 获取鼠标屏幕坐标。
2. 从摄像机向鼠标位置发射射线。
3. 与地面或目标层碰撞后得到世界坐标。
4. 将该世界坐标写入 `TPSCharacter.LookAtPosition`。
5. 根据是否二维模式调整坐标平面。

在俯视角中，它是角色朝鼠标转向的核心。

### 8.3 右摇杆瞄准

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Character Controllers/Additionals/AimOnRightJoystickDirection.cs`

设计逻辑：

1. 读取 `RotateHorizontal` 和 `RotateVertical`。
2. 将摇杆方向转换为角色周围的瞄准方向。
3. 生成一个距离角色一定半径的目标点。
4. 写入 `TPSCharacter.LookAtPosition`。

### 8.4 瞄准方式切换

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Character Controllers/Additionals/AimControllSwitcher.cs`

每帧判断：

```text
如果没有使用手柄，且不是移动端：启用鼠标瞄准，禁用摇杆瞄准
否则：禁用鼠标瞄准，启用摇杆瞄准
```

判断依据来自：

```text
JUInputManager.IsUsingGamepad
JUGameManager.IsMobileControls
```

### 8.5 与开火的连接

角色开火时，`WeaponOrientator()` 会使用 `GetLookPosition()` 和当前武器 `Shoot_Position` 计算射击方向。鼠标或摇杆瞄准只是提供目标点，真正开火仍由角色和武器系统完成。

## 九、背包、拾取和物品系统拆解

### 9.1 设计目的

背包系统负责统一管理角色拥有的所有物品，包括右手可持物、左手可持物、所有物品、护甲、当前装备物品、拾取范围内物品、顺序槽位和快捷切换。

### 9.2 核心脚本

| 脚本 | 作用 |
|---|---|
| `JUInventory.cs` | 背包和拾取主逻辑。 |
| `ItemSystemLib.cs` | `JUItem`、`JUHoldableItem` 等基础物品类。 |
| `Weapon.cs` | 枪械物品。 |
| `MeleeWeapon.cs` | 近战物品。 |
| `ThrowableItem.cs` | 投掷物。 |
| `Armor.cs` | 护甲。 |

### 9.3 玩家背包配置

`Top Down JU Character` 上的 `JUInventory` 配置了：

| 字段 | 说明 |
|---|---|
| `HoldableItensRightHand` | 右手可用物品列表，包含枪械、近战、投掷物等。 |
| `HoldableItensLeftHand` | 左手可用物品列表，用于双持。 |
| `AllHoldableItems` | 所有可手持物。 |
| `AllItems` | 全部物品，包含护甲等非手持物。 |
| `SequenceSlot` | 顺序槽位，用于快捷切换。 |
| `EnablePickup` | 启用拾取。 |
| `CheckerRadious` | 拾取检测半径，当前为 `1.5`。 |
| `UseDefaultInputToPickUp` | 使用默认输入拾取。 |
| `AutoEquipPickedUpItems` | 拾取后自动装备。 |
| `HoldTimeToPickUp` | 拾取按住时间。 |

### 9.4 物品基础类

`JUItem` 包含物品分类、图标、是否解锁、数量、最大数量、名称和切换 ID。

`JUHoldableItem` 在此基础上增加武器旋转中心、是否单次使用、是否持续使用、是否阻止开火姿态、身体收纳模型、使用时间、手持位置索引、是否左手物品、是否强制双持、双持配对物品、手持姿势和另一只手 IK 位置。

### 9.5 拾取流程

```text
玩家按拾取键
    ↓
JUInventory 在 CheckerRadious 范围内查找 ItemLayer 上的物品
    ↓
选择最近或可拾取物品
    ↓
AddPickedItemData() 把地面物品数据合并到背包内对应物品
    ↓
如果 AutoEquipPickedUpItems 为真，则 EquipItem()
    ↓
地面物品被移除或隐藏
```

### 9.6 装备流程

`EquipItem(ID)` 根据物品类型分流：

| 类型 | 行为 |
|---|---|
| 护甲 | 直接激活护甲对象。 |
| 非手持普通物品 | 直接激活对象。 |
| `JUHoldableItem` | 调用角色 `SwitchToItem()` 或背包自己的 `SwitchToItem()`。 |

关键连接：

```text
JUInventory.EquipItem(ID)
    ↓
JUCharacterController.SwitchToItem(ItemSwitchID, RightHand)
    ↓
更新 HoldableItemInUseRightHand / HoldableItemInUseLeftHand
    ↓
刷新武器模型、IK、动画层、当前武器引用
```

## 十、换枪系统拆解

### 10.1 设计目的

换枪系统负责根据输入切换当前手持物。它不直接处理武器开火，而是选择哪个 `JUHoldableItem` 当前在手上。

### 10.2 核心脚本

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Weapon Management/Weapon Switch/ItemSwitchManager.cs`

### 10.3 玩家配置

`Top Down JU Character` 上的 `ItemSwitchManager` 配置：

| 字段 | 当前值 | 作用 |
|---|---|---|
| `IsPlayer` | 真 | 表示这是玩家换枪器。 |
| `UseOldInputSystem` | 假 | 使用新输入系统。 |
| `ItemToEquipOnStart` | `-1` | 开局不强制装备某物品。 |
| `EnableNextAndPreviousWeaponSwitch` | 真 | 允许上一件、下一件。 |
| `EnableAlphaNumericWeaponSwitch` | 真 | 允许数字键切换。 |
| `EnableMouseScrollWeaponSwitch` | 真 | 允许鼠标滚轮切换。 |
| `ScrollThreshold` | `0.1` | 滚轮阈值。 |

### 10.4 输入流程

每帧 `Update()` 调用输入控制：

```text
如果 UseOldInputSystem 为真：OldInput_ItemSwitchController()
否则：NewInput_ItemSwitchController()
```

新输入模式下，逻辑包括：

| 输入 | 行为 |
|---|---|
| 下一件 | `NextItem()` |
| 上一件 | `PreviousItem()` |
| 数字键 | `SwitchToItem(对应 ID)` |
| 鼠标滚轮上 | 下一件 |
| 鼠标滚轮下 | 上一件 |

### 10.5 换枪执行链路

```text
ItemSwitchManager.NextItem() / PreviousItem() / SwitchToItem()
    ↓
计算下一个已解锁物品 ID
    ↓
SwitchCharacterItem(character, SwitchID)
    ↓
JUCharacterController.SwitchToItem()
    ↓
更新当前手持物、隐藏旧物品、显示新物品
    ↓
触发换枪动画和 IK 权重过渡
```

### 10.6 设计要点

换枪系统依赖数组索引、`ItemSwitchID` 和顺序槽位三套概念。这个设计功能强，但容易复杂。正式类孤胆枪手项目可简化为固定武器槽：主武器、副武器、近战、投掷物、特殊武器。

## 十一、枪械和开火系统拆解

### 11.1 设计目的

枪械系统负责判断能否开火、计算射击方向、根据精度和散布偏移射线、生成子弹 Prefab、扣除弹药、播放枪声、枪口火焰、退壳、后坐力和换弹反馈。

### 11.2 核心脚本

| 脚本 | 作用 |
|---|---|
| `Weapon.cs` | 枪械主体。 |
| `Bullet.cs` | 子弹飞行、命中、反弹、销毁。 |
| `Damager.cs` | 近战或触发器伤害。 |
| `WeaponAimRotationCenter.cs` | 武器挂点、武器位置、程序动画参考。 |

### 11.3 武器核心字段

| 字段 | 说明 |
|---|---|
| `RaycastingLayers` | 射线检测层。 |
| `BulletsPerMagazine` | 弹匣容量。 |
| `TotalBullets` | 备用弹药。 |
| `BulletsAmounts` | 当前弹匣子弹数。 |
| `InfiniteAmmo` | 无限弹药。 |
| `Fire_Rate` | 射速间隔。 |
| `Precision` | 精度恢复速度。 |
| `LossOfAccuracyPerShot` | 每发增加的散布。 |
| `BulletPrefab` | 子弹 Prefab。 |
| `MuzzleFlashParticlePrefab` | 枪口火焰。 |
| `Shoot_Position` | 枪口位置。 |
| `FireMode` | 开火模式。 |
| `AimMode` | 瞄准模式。 |
| `CameraAimingPosition` | 瞄准时镜头偏移。 |
| `CameraFOV` | 瞄准 FOV。 |
| `RecoilForce` | 后坐力。 |
| `ShootAudio` | 枪声。 |
| `ReloadAudio` | 换弹声。 |
| `EmptyMagazineAudio` | 空枪声。 |

### 11.4 开火入口

角色系统最终调用：

```text
HoldableItemInUseRightHand.UseItem()
```

如果当前手持物是 `Weapon`，则进入 `Weapon.UseItem()`：

```text
如果 CanUseItem 且 BulletsAmounts > 0：Shot()
否则如果没有子弹：播放空枪声和轻微武器摆动
base.UseItem()
```

### 11.5 `Shot()` 流程

```text
如果 CanUseItem 为假，返回
根据 FireMode 判断普通枪械或霰弹枪
如果启用 camera direction correction：
    从摄像机位置沿 ShootDirection 做 Raycast
    若命中，修正 ShootDirection，使枪口朝命中点射击
如果射击方向和枪口 forward 差异过大：
    回退到枪口 forward
加入随机散布 ShotErrorProbability
从 Shoot_Position 发射 Raycast
如果命中：
    让枪口 LookAt 命中点
    BulletSpawn(... 命中点 ...)
否则：
    BulletSpawn(... 远方向 ...)
播放枪声
生成弹壳
扣除弹药
增加散布
执行后坐力
生成枪口火焰
```

### 11.6 子弹生成

`BulletSpawn()` 做以下事情：

```text
Instantiate(BulletPrefab, ShootStart, ShootDirection)
如果子弹上有 Bullet 组件：
    设置 FinalPoint
    设置 FinalPointNormal
    设置 Owner 为开火角色
    忽略开火者自身碰撞体
Destroy(bullet, 10f)
```

`Bullet.cs` 支持物理移动、高精度碰撞、命中特效、冲击力、反弹和所有者忽略。

### 11.7 换弹逻辑

`Weapon.Reload()` 会根据当前弹匣和总弹药计算补充数量。如果当前弹匣未满且 `TotalBullets > 0`，则从备用弹药转移到当前弹匣，并播放换弹音效。

角色中还包含自动换弹逻辑：当玩家按射击但弹匣为空、备用弹药大于零时，可以触发换弹。

## 十二、近战、投掷和伤害系统拆解

### 12.1 近战系统

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Weapon Management/MeleeWeapon.cs`

| 字段 | 说明 |
|---|---|
| `AttackAnimatorParameterName` | 攻击动画参数。 |
| `DamagerToEnable` | 攻击时启用的伤害器。 |
| `EnableHealthLoss` | 使用时是否损耗武器耐久。 |
| `MeleeWeaponHealth` | 近战武器耐久。 |
| `DamagePerUse` | 每次使用损耗。 |

流程：

```text
玩家按射击
    ↓
角色判断当前物品是 MeleeWeapon
    ↓
触发 Animator 攻击参数
    ↓
动画事件或脚本启用 Damager
    ↓
Damager 碰撞到目标后造成伤害
```

### 12.2 投掷物系统

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Weapon Management/ThrowableItem.cs`

| 字段 | 说明 |
|---|---|
| `AnimationTriggerParameterName` | 投掷动画参数。 |
| `ThrowForce` | 前向投掷力。 |
| `ThrowUpForce` | 向上投掷力。 |
| `RotationForce` | 旋转力。 |
| `SecondsToDestroy` | 自动销毁时间。 |
| `PositionToThrow` | 相对角色的生成位置。 |
| `DirectionToThrow` | 相对角色的投掷方向。 |

流程：

```text
玩家按射击
    ↓
角色判断当前物品是 ThrowableItem
    ↓
触发投掷动画
    ↓
动画事件调用 UseItem 或 ThrowThis
    ↓
生成投掷物 Prefab
    ↓
给 Rigidbody 添加力和旋转力
```

### 12.3 通用伤害器 `Damager`

`Damager` 支持碰撞、触发器和射线伤害。关键字段包括 `Damage`、`HitMinTime`、`TagsToDamage`、`RaycastDistance`、`RaycastLayer`、`IgnoreRootColliders`、`HitParticlesList` 和 `HitSoundsAudioSource`。

造成伤害时会尝试找到目标身上的：

```text
JUHealth
DamageableBodyPart
DamageableBody
```

然后调用相应伤害方法。

### 12.4 生命系统

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Gameplay/Character Controllers/Additionals/JUHealth.cs`

| 字段 | 说明 |
|---|---|
| `Health` | 当前生命值。 |
| `MaxHealth` | 最大生命值。 |
| `BloodScreenEffect` | 是否启用血屏。 |
| `BloodHitParticle` | 受击粒子。 |
| `OnDeath` | 死亡事件。 |
| `IsDead` | 是否死亡。 |

流程：

```text
DoDamage(damage, hitPosition)
    ↓
Health -= damage
    ↓
生成受击特效
    ↓
CheckHealthState()
    ↓
如果 Health <= 0 且未死亡：IsDead = true，触发 OnDeath
```

## 十三、车辆驾驶系统拆解

### 13.1 设计目的

车辆系统允许玩家自动检测附近车辆、按交互键进入车辆、把角色绑定到驾驶座位置、禁用或限制角色常规移动、用输入控制车辆、按交互键退出车辆并恢复角色控制。

### 13.2 核心脚本

| 脚本 | 作用 |
|---|---|
| `DriveVehicles.cs` | 角色上车、下车、驾驶状态管理。 |
| `JUVehicleEngine.cs` | `Vehicle` 基类，处理输入、轮子、引擎、物理。 |
| `CarController.cs` | 汽车控制器，继承 `Vehicle`。 |
| `MotorcycleController.cs` | 摩托控制器，继承 `Vehicle`。 |
| `JUVehicleCharacterIK.cs` | 驾驶座位置、手脚 IK、下车点。 |
| `ProceduralDrivingAnimation.cs` | 驾驶时角色身体倾斜和 IK 表现。 |

### 13.3 玩家上的驾驶配置

`Top Down JU Character` 的 `DriveVehicles` 配置包含：

| 字段 | 当前含义 |
|---|---|
| `EnterVehiclesEnabled` | 允许上车。 |
| `ExitVehiclesEnabled` | 允许下车。 |
| `DisableCharacterOnEnter` | 上车后是否隐藏或禁用角色。当前不禁用。 |
| `DelayToReenableAction` | 进入或退出后的短暂冷却。 |
| `UseDefaultInputs` | 使用默认交互输入。 |
| `CheckNearVehicles` | 附近车辆检测配置。 |
| `GroundLayer` | 下车落点检测层。 |
| `OnEnterVehicle`、`OnExitVehicle` | 进入和退出事件。 |

`CheckNearVehicles` 内部包含检测层、车辆区域标签、检测半径、是否自动检查、自动检查间隔和是否检查障碍。

### 13.4 查找车辆流程

`FindNearVehicles()`：

```text
以角色碰撞体中心为球心
    ↓
Physics.OverlapSphere 查找 CheckNearVehicles.Layer 中的碰撞体
    ↓
筛选 CompareTag("VehicleArea") 的触发器
    ↓
按距离排序，取最近车辆区域
    ↓
如果 AvoidObstacles 为真，则 Linecast 检查障碍
    ↓
从车辆区域父级获取 Vehicle
    ↓
获取 JUVehicleCharacterIK
    ↓
写入 NearestVehicle 和 NearestVehicleCharacterIK
```

车辆 Prefab 必须具备触发区域 Collider、Tag `VehicleArea`、父级 `Vehicle` 组件和 `JUVehicleCharacterIK`。

### 13.5 上车流程

玩家按交互键后触发 `OnPressEnterVehicleButton()`。如果当前未驾驶：

```text
TryDriveNearestVehicle()
    ↓
检查 EnterVehiclesEnabled
    ↓
检查 NearestVehicle 是否存在
    ↓
检查车辆是否允许角色驾驶
    ↓
DriveVehicle(vehicle, vehicleCharacterIK)
```

`DriveVehicle()` 设置当前车辆和当前车辆 IK，进入上车状态，触发 `OnStartEnterVehicle`，最终调用 `OnCharacterStartDriving()`。

`OnCharacterStartDriving()` 会设置 `IsDriving = true`、启用车辆控制、禁用脚步声、让角色速度跟随车辆，并触发 `OnEnterVehicle`。

### 13.6 驾驶中更新

`DriveVehicles.Update()` 中，如果 `IsDriving == true`，调用 `UpdateDrivingState()`：

```text
角色 Rigidbody 速度 = 当前车辆 Rigidbody 速度
如果车辆 IK 提供驾驶座 Transform：
    角色位置 = 驾驶座位置
    角色旋转 = 驾驶座旋转
否则：
    角色位置 = 车辆位置
    角色旋转 = 车辆旋转
```

驾驶状态下，角色自身不再按普通移动输入移动，而是被车辆座位绑定。

### 13.7 下车流程

```text
驾驶中按 Interact
    ↓
ExitVehicle()
    ↓
JUVehicleCharacterIK.GetExitPosition()
    ↓
找到可用下车点
    ↓
OnCharacterStopDriving()
    ↓
IsDriving = false，车辆 ControlsEnabled = false
    ↓
角色位置移动到下车点
    ↓
摄像机回到普通或开火状态
```

### 13.8 车辆物理基类

`Vehicle` 定义在 `JUVehicleEngine.cs` 中，负责轮子数据、引擎配置、输入平滑、默认输入、控制开关、输入处理、油门、转向、刹车、轮子同步、防侧翻和重心更新。

`CarController` 继承 `Vehicle`，主要负责多轴轮子配置和汽车轮子同步。

`MotorcycleController` 继承 `Vehicle`，额外包含摩托倾斜、循环或特技逻辑、车身倾角模拟、前后轮配置。

### 13.9 摄像机和驾驶的连接

角色进入驾驶后：

```text
JUCharacterController.IsDriving = true
```

`TDCameraController` 每帧读到该状态后切换到：

```text
DrivingVehicleCameraState
```

车辆系统不直接操作摄像机，只改变角色状态；摄像机通过状态自动切换。

## 十四、AI 系统拆解

### 14.1 设计目的

Demo 中的 AI 主要用于展示模板的第三人称敌人能力，包括视野检测、目标筛选、路径巡逻、追踪目标、进入攻击状态、控制角色开火和目标丢失后返回巡逻。

### 14.2 核心脚本

| 脚本 | 作用 |
|---|---|
| `PatrolAI.cs` | 巡逻 AI，能沿 Waypoint 行走并攻击玩家。 |
| `ZombieAI.cs` | 僵尸 AI，逻辑更偏向追逐攻击。 |
| `JUCharacterArtificialInteligenceBrain.cs` | AI 路径和视野基础工具。 |
| `WaypointPath.cs` | 巡逻路径点。 |
| `JUAutoInstantiate.cs` | 生成 AI 实例。 |

### 14.3 Waypoint 路径

场景中 `Waypoint Path` 下有多个子对象。`WaypointPath` 会收集这些 Transform，并生成路径点数组 `WaypointPathPositions`。它支持刷新路径点、按路径移动、开始时反转路径，以及路径结束后停止、循环或反向。

### 14.4 巡逻 AI 流程

`PatrolAI.Start()` 中每 `0.5` 秒调用一次 `CheckTargets()`。

`PatrolAI.Update()` 流程：

```text
如果角色死亡：禁用 AI
CheckEndEvents()
根据目标距离决定是否奔跑
如果 currentTarget 不为空：
    HuntTheTargetState()
    如果距离小于 AttackAtDistance 且目标可见且可攻击：
        EnterAttackModeState()
    否则：
        ExitAttackModeState()
否则：
    如果有 WaypointPath：
        如果角色仍处于 FiringMode：
            追踪最后目标或附近位置，并在一段时间后退出开火模式
        否则：
            FollowWaypointPathState()
    否则：
        根据 Destination 或 IdleState 处理
UpdateCurrentTarget()
```

### 14.5 目标检测

`CheckTargets()`：

```text
计算 fieldViewPosition
    ↓
FieldOfView.CheckViewCollider(...)
    ↓
根据 TargetTags 选择目标，默认 TargetTags 包含 Player
    ↓
如果目标是 JUCharacterBrain，则保存 targetJuCharacter
    ↓
判断目标是否死亡，更新 isCurrentTargetAttackable
```

### 14.6 追踪和攻击

`HuntTheTargetState()` 根据目标是否可见决定追踪目标、追踪最后可见位置或返回巡逻。

`EnterAttackModeState()` 会让 AI 角色看向目标并进入开火模式：

```text
character.LookAtPosition = smoothedTargetPosition + Vector3.up * AimUpOffset
character.FiringMode = true
character.FiringModeIK = true
```

AI 的攻击不是独立系统，而是复用 `JUCharacterController` 的开火和物品逻辑。AI 只负责设置目标、移动和触发开火状态。

### 14.7 自动生成器

场景中的 `Zombie Spawner` 挂载 `JUAutoInstantiate`：

| 字段 | 当前配置 |
|---|---|
| `StartInstantiateOnAwake` | 真。 |
| `TimeToSpawn` | `5` 秒后开始。 |
| `Repeat` | 真。 |
| `RepeatingTime` | `1` 秒。 |
| `SwitchToRandomInstantiate` | 真。 |
| `SpawnArea` | `20 x 0 x 20`。 |
| `RandomRotation` | 真。 |
| `Quantity` | `1`。 |
| `InstancesLimit` | `10`。 |

该生成器适合作为怪物刷新器原型，但正式项目建议重构为波次系统。

## 十五、UI 系统拆解

### 15.1 UI 根节点

当前场景使用的 UI Prefab 是：

`Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Game/UI Interfaces/JUTPS Sidescroller User Interface.prefab`

虽然名称包含 `Sidescroller`，但内部挂载了通用系统：`JUGameManager`、`JUInputManager`、`JUGameSettings`、`JUPauseGame`、`Crosshair`、`UIHealhBar`、`UIItemInformation` 和移动端按钮脚本。

### 15.2 准星系统

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/UI/Crosshair.cs`

准星负责读取玩家当前武器精度、根据精度缩放准星、可选择跟随鼠标位置、用射线检测准星指向对象、根据目标是否可射击改变颜色，并根据是否持有武器、是否瞄准、是否开火模式决定隐藏。

关键静态字段：

| 字段 | 说明 |
|---|---|
| `AimingOnTarget` | 是否瞄准可攻击目标。 |
| `AimingOnFriend` | 是否瞄准友方或不可射击对象。 |
| `ObjectOnCrosshairPoint` | 准星指向对象。 |

### 15.3 血条 UI

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/UI/UIHealhBar.cs`

流程：

```text
Start()
    如果 IsPlayerHealthBar 为真：通过 Player 标签查找 JUHealth
Update()
    healthValueNormalized = Health / MaxHealth
    平滑更新 Image.fillAmount
    根据生命比例设置颜色
    如果有文字，则显示 000/MaxHealth
    如果生命上升或下降，短暂改成治疗或受伤颜色
```

### 15.4 当前物品 UI

脚本：

`Assets/ProjectRef/Julhiecio TPS Controller/Scripts/UI/UIItemInformation.cs`

流程：

```text
如果 Player 为空：从 JUGameManager.PlayerController 获取
如果 Player.Inventory 为空：返回
CurrentItem = Player.HoldableItemInUseRightHand
如果没有当前物品：显示 Hand
如果是 Weapon：显示图标、名称、数量、当前弹匣/总弹药、弹匣比例
如果是普通手持物或投掷物：显示数量比例
如果是 MeleeWeapon：显示耐久比例
```

### 15.5 暂停系统

`JUPauseGame` 和 `JU_UIPause` 管理暂停。暂停时通常会影响 `Time.timeScale`、鼠标锁定、UI 面板显示和角色控制。`JUCharacterController` 会检查 `JUPauseGame.IsPaused` 并提前返回。

## 十六、场景标签、层和物理连接

### 16.1 关键 Tags

| Tag | 用途 |
|---|---|
| `Player` | 查找玩家、AI 目标、血条绑定、摄像机跟随。 |
| `Enemy` | 敌人标记和伤害筛选。 |
| `Bullet` | 子弹标记，避免错误物理伤害。 |
| `Skin` | 身体受击部位。 |
| `VehicleArea` | 车辆进入检测区域。 |
| `DeadZone` | 掉落或死亡区域。 |
| `RagdollZone` | 布娃娃测试区域。 |
| `Grass`、`Stone`、`Tiles` | 脚步声或表面反馈。 |
| `CoverTrigger` | 掩体触发相关。 |
| `Distractable` | 可吸引 AI 的对象。 |

### 16.2 关键 Layers

| Layer 索引 | 名称 | 用途 |
|---|---|---|
| 8 | `Vehicle` | 车辆主体。 |
| 9 | `Character` | 角色。 |
| 10 | `Bullet` | 子弹。 |
| 11 | `Terrain` | 地形。 |
| 12 | `VehicleMeshCollider` | 车辆 Mesh 碰撞。 |
| 13 | `VehicleWheel` | 车辆轮子。 |
| 14 | `Item` | 可拾取物。 |
| 15 | `Bones` | 布娃娃骨骼。 |
| 16 | `Walls` | 墙体。 |
| 17 | `Trigger` | 触发器。 |

层索引不能随意变，因为很多 LayerMask 序列化保存的是整数位掩码，不是字符串。

### 16.3 物理矩阵

模板同步了 `ProjectSettings/DynamicsManager.asset`，其中包含自定义 Layer Collision Matrix。该矩阵会影响子弹命中、角色碰撞、车辆碰撞、拾取检测和上车检测。

## 十七、运行时完整链路示例

### 17.1 玩家移动链路

```text
键盘或手柄输入
    ↓
JUTPSInputControlls 产生动作值
    ↓
JUInputManager.UpdateAxis()
    ↓
JUCharacterController.ControllerInputs()
    ↓
HorizontalX / VerticalY
    ↓
JUCharacterController.FixedUpdate()
    ↓
Movement()
    ↓
DoFreeMovement() 或 DoFireModeMovement()
    ↓
Rigidbody / Animator / IK 更新
```

### 17.2 鼠标瞄准链路

```text
鼠标位置
    ↓
AimOnMousePosition
    ↓
摄像机射线命中地面或目标
    ↓
TPSCharacter.LookAtPosition
    ↓
JUCharacterController.Rotate()
    ↓
角色朝瞄准点旋转
    ↓
WeaponOrientator()
    ↓
武器朝瞄准点对齐
```

### 17.3 开火链路

```text
玩家按下射击
    ↓
JUInputManager.PressedShooting / PressedShootingDown
    ↓
JUCharacterController.ControllerInputs()
    ↓
进入 FiringMode，调用武器使用逻辑
    ↓
Weapon.UseItem()
    ↓
Weapon.Shot()
    ↓
从摄像机方向修正枪口方向
    ↓
BulletSpawn()
    ↓
Bullet 移动并检测命中
    ↓
命中目标后调用 JUHealth 或 DamageableBodyPart
    ↓
UI 血条和准星反馈更新
```

### 17.4 换枪链路

```text
鼠标滚轮 / 数字键 / 下一件上一件输入
    ↓
ItemSwitchManager.Update()
    ↓
NewInput_ItemSwitchController()
    ↓
NextItem() / PreviousItem() / SwitchToItem()
    ↓
计算目标 ItemSwitchID
    ↓
JUCharacterController.SwitchToItem()
    ↓
隐藏旧物品，显示新物品
    ↓
更新 HoldableItemInUseRightHand
    ↓
更新 WeaponInUseRightHand / MeleeWeaponInUseRightHand
    ↓
动画层和 IK 权重过渡
    ↓
UIItemInformation 显示新物品
```

### 17.5 拾取链路

```text
玩家接近物品
    ↓
JUInventory 在 ItemLayer 和 CheckerRadious 内检测物品
    ↓
玩家按 PickupButton
    ↓
JUInventory.PickUp()
    ↓
AddPickedItemData()
    ↓
增加数量或解锁物品
    ↓
AutoEquipPickedUpItems 为真则 EquipItem()
    ↓
当前手持物和 UI 更新
```

### 17.6 上车链路

```text
DriveVehicles 自动定时 FindNearVehicles()
    ↓
OverlapSphere 查找 VehicleArea 触发器
    ↓
记录 NearestVehicle
    ↓
玩家按 Interact
    ↓
TryDriveNearestVehicle()
    ↓
DriveVehicle()
    ↓
进入上车状态
    ↓
OnCharacterStartDriving()
    ↓
IsDriving = true，车辆 ControlsEnabled = true
    ↓
TDCameraController 切换 DrivingVehicleCameraState
    ↓
车辆读取输入并移动
```

### 17.7 下车链路

```text
驾驶中按 Interact
    ↓
ExitVehicle()
    ↓
JUVehicleCharacterIK.GetExitPosition()
    ↓
找到可用下车点
    ↓
OnCharacterStopDriving()
    ↓
IsDriving = false，车辆 ControlsEnabled = false
    ↓
角色位置移动到下车点
    ↓
摄像机回到普通或开火状态
```

### 17.8 AI 攻击链路

```text
PatrolAI 定时 CheckTargets()
    ↓
FieldOfView 检测 Player
    ↓
currentTarget = Player
    ↓
Update() 中 HuntTheTargetState()
    ↓
NavMesh 路径追踪目标
    ↓
进入 AttackAtDistance
    ↓
EnterAttackModeState()
    ↓
AI 角色 LookAtPosition 指向玩家
    ↓
AI 角色 FiringMode = true
    ↓
复用 JUCharacterController 和 Weapon 开火逻辑
```

## 十八、后处理和渲染

场景中存在 `Post Processing Volume`，挂载 `Packages/com.unity.postprocessing/PostProcessing/Runtime/PostProcessVolume.cs`。当前项目已嵌入 `Packages/com.unity.postprocessing`，并在 `ProjectSettings/ProjectSettings.asset` 中加入 `UNITY_POST_PROCESSING_STACK_V2`。

模板原本更接近 Built-in Render Pipeline。为了让 Demo 优先原样运行，当前项目同步了模板的图形和质量设置。若后续正式项目使用 URP，需要重新处理材质 Shader、后处理、光照和摄像机设置。

## 十九、功能设计方案拆解

### 19.1 自研俯视角玩家控制需要设计的功能

| 功能 | 设计内容 |
|---|---|
| 移动输入 | WASD 或摇杆输入，输出二维移动向量。 |
| 世界方向转换 | 根据摄像机朝向把输入转成世界移动方向。 |
| 角色移动 | Rigidbody 或 CharacterController。 |
| 鼠标瞄准 | 摄像机射线到地面平面。 |
| 身体旋转 | 开火模式下朝鼠标点，普通模式朝移动方向。 |
| 动画参数 | 移动速度、方向、开火、翻滚、死亡。 |
| 状态管理 | 普通、开火、换弹、翻滚、死亡、驾驶。 |

### 19.2 自研武器系统需要设计的功能

| 功能 | 设计内容 |
|---|---|
| 武器数据 | 名称、图标、射速、弹匣、伤害、散布、子弹 Prefab。 |
| 武器状态 | 当前弹匣、备用弹药、冷却计时、是否换弹。 |
| 开火入口 | 玩家输入或 AI 调用统一接口。 |
| 命中检测 | 射线或实体子弹。 |
| 子弹生成 | 生成位置、方向、所有者忽略。 |
| 反馈 | 枪声、枪口火焰、弹壳、后坐力、震屏。 |
| UI | 当前武器、弹药、准星散布。 |

### 19.3 自研换枪系统需要设计的功能

| 功能 | 设计内容 |
|---|---|
| 武器槽 | 主武器、副武器、近战、投掷物等。 |
| 切换输入 | 数字键、滚轮、上一件、下一件。 |
| 切换条件 | 死亡、换弹、翻滚、驾驶时是否允许切换。 |
| 切换过程 | 收起旧武器、显示新武器、动画过渡。 |
| 数据同步 | 更新当前武器引用和 UI。 |

### 19.4 自研车辆系统需要设计的功能

| 功能 | 设计内容 |
|---|---|
| 可交互检测 | 车辆范围触发器和交互提示。 |
| 上车状态 | 播放动画、锁定角色输入、绑定座位。 |
| 驾驶输入 | 油门、刹车、转向、手刹。 |
| 车辆物理 | 轮子、速度、重心、防侧翻。 |
| 下车状态 | 找可用落点、恢复角色控制。 |
| 摄像机 | 驾驶模式镜头参数。 |
| UI | 上车提示、速度或车辆状态。 |

### 19.5 自研敌人系统需要设计的功能

| 功能 | 设计内容 |
|---|---|
| 生成 | 刷怪点、波次、数量上限。 |
| 寻路 | NavMesh、流场或简单追踪。 |
| 攻击 | 近战、远程、冲撞、自爆。 |
| 受击 | 生命、硬直、击退、死亡。 |
| 掉落 | 弹药、金币、经验、武器。 |
| 难度 | 随波次提升速度、血量、数量、精英怪。 |

## 二十、适合复用与不建议直接复用的内容

### 20.1 适合复用的技术思想

| 内容 | 原因 |
|---|---|
| 状态驱动摄像机 | 逻辑清晰，适合俯视角。 |
| 鼠标射线转世界瞄准点 | 是俯视角射击基础。 |
| 武器射线修正枪口方向 | 能解决俯视角枪口和鼠标点不一致问题。 |
| 子弹所有者忽略 | 必须保留。 |
| 准星随精度变化 | 有即时反馈。 |
| 拾取半径检测 | 可用于类孤胆枪手拾取补给。 |
| 上车状态切换 | 如果游戏保留车辆，可参考。 |
| 波次生成器雏形 | 可作为刷怪器最小原型。 |

### 20.2 不建议直接作为最终架构的内容

| 内容 | 原因 |
|---|---|
| `JUCharacterController` 巨型类 | 移动、动画、武器、IK、物理高度耦合。 |
| `JUInventory` 全能背包 | 同时管理拾取、护甲、手持物、UI、槽位，复杂度高。 |
| 静态全局 `JUGameManager` | 多场景和正式架构下容易残留引用。 |
| AI 巡逻系统 | 更偏 TPS 巡逻敌人，不适合大规模怪潮。 |
| UI 根节点混合输入管理 | UI、输入、设置、全局管理放在同一个 Prefab，不利于维护。 |
| 车辆系统与角色 IK 高耦合 | 如非核心玩法，建议独立封装。 |

## 二十一、建议的后续重构路线

### 21.1 第一阶段：保持模板可运行

目标是不破坏 `ProjectRef`。

建议：

1. 不直接修改模板核心脚本。
2. 用文档理解运行链路。
3. 需要测试时只打开 Demo 场景。
4. 新业务代码放到 `Assets/Game`。

### 21.2 第二阶段：抽取最小俯视角原型

建议模块：

```text
Assets/Game/Scripts/Input
Assets/Game/Scripts/Player
Assets/Game/Scripts/Camera
Assets/Game/Scripts/Weapons
Assets/Game/Scripts/Enemies
Assets/Game/Scripts/Pickups
Assets/Game/Scripts/UI
Assets/Game/Scripts/GameFlow
```

优先实现：玩家移动、鼠标瞄准、摄像机跟随、单武器开火、敌人追踪、伤害死亡、简单 UI、刷怪器。

### 21.3 第三阶段：按需求迁移模板功能

| 模板功能 | 迁移方式 |
|---|---|
| 武器参数 | 提取成自己的 `WeaponData`。 |
| 子弹逻辑 | 参考 `Bullet.cs`，重写简化版。 |
| 准星 | 参考 `Crosshair.cs` 的散布反馈。 |
| 拾取 | 参考 `JUInventory` 的检测思路，不复用全背包。 |
| 车辆 | 如确实需要，再独立迁移。 |
| AI | 参考视野和路径工具，但重写怪潮 AI。 |

## 二十二、总结

`Top Down Demo` 是一个功能齐全的模板演示场景。它的核心是 `JUCharacterController`，围绕它连接了输入、摄像机、瞄准、武器、背包、换枪、驾驶、生命、AI 和 UI。

它的优点是功能完整、运行链路成熟、演示内容丰富。它的问题是模板化程度高，很多系统互相耦合，不适合直接作为最终类孤胆枪手项目架构。

最合理的使用方式是：

```text
ProjectRef 保持为参考实现
正式游戏另建独立架构
从模板中拆思想，不直接继承复杂耦合
优先实现类孤胆枪手最小核心循环
```

对于后续项目，最应该复用的是俯视角瞄准、状态驱动摄像机、武器射线修正、准星精度反馈和拾取检测思路；最应该重写的是角色巨型控制器、全能背包、AI 巡逻系统和混合式 UI 根节点。

## 二十三、关键资源与场景实例精确清单

本章节用于把前面的大系统拆解落到具体资源上，方便后续在 Unity 中逐个定位和验证。这里列出的内容不是建议全部照搬到正式项目，而是说明当前 `Top Down Demo` 能运行所依赖的关键实例、Prefab 和脚本连接。

### 23.1 场景中直接实例化的核心 Prefab

| 场景实例 | 资源路径 | 技术职责 |
|---|---|---|
| `Top Down JU Character` | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Character Prefabs/Top Down JU Character.prefab` | 玩家角色，承载移动、瞄准、开火、背包、换枪、生命、布娃娃、驾驶能力。 |
| `TopDown Camera Controller` | `Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Game/Camera Prefabs/TopDown Camera Controller.prefab` | 俯视角摄像机，读取玩家状态后切换不同相机状态。 |
| `JUTPS Sidescroller User Interface` | `Assets/ProjectRef/Julhiecio TPS Controller/Prefabs/Game/UI Interfaces/JUTPS Sidescroller User Interface.prefab` | Demo 当前使用的 UI 根节点，虽然名字是横版 UI，但被该俯视角场景复用。 |
| `Car` | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Vehicles/Car.prefab` | 四轮车辆示例，供 `DriveVehicles` 上车驾驶。 |
| `Motorcycle` | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Vehicles/Motorcycle.prefab` | 摩托车示例，供角色进入驾驶状态。 |
| `Bike` | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Vehicles/Bike.prefab` | 自行车示例，同属车辆系统。 |
| `Patrol AI Sample` | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/AI/Patrol AI Sample.prefab` | 巡逻 AI 示例，包含角色控制器、背包、武器和感知攻击逻辑。 |
| 多个拾取武器 | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Items/Weapons/Guns` | 场景中可拾取枪械来源。 |
| 多个近战武器 | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Items/Melee Weapons` | 可拾取近战武器来源。 |
| 多个投掷物 | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Items/Throwable` | 可拾取手雷等投掷物来源。 |
| 多个护甲 | `Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Prefabs/Items/Armor` | 可拾取或可装备护甲来源。 |

### 23.2 场景中重复出现的演示资源

| 资源类型 | 数量特征 | 作用 |
|---|---:|---|
| `Plataform.fbx` | 场景中约二十个实例 | 构成 Demo 地面、平台、墙体或结构块。 |
| `ShootingTarget.prefab` | 场景中约十二个实例 | 射击靶，用于验证开火、命中、伤害反馈。 |
| `MeshTerrain.fbx` | 场景中约七个实例 | 构成演示地图地形。 |
| `InfoIcon.prefab` | 场景中约七个实例 | 信息提示或教学交互标记。 |
| `Bottle.prefab` | 场景中约六个实例 | 可被射击或物理影响的演示物。 |
| `Katana.prefab` | 场景中约四个实例 | 近战拾取物示例。 |
| `Granade.prefab` | 场景中约三个实例 | 投掷物拾取示例。 |

这些资源服务于模板演示，不一定都要进入正式类孤胆枪手项目。正式项目最有价值的是它们所验证的交互链路：拾取、装备、射击、命中、伤害、物理反馈和 UI 显示。

## 二十四、玩家 Prefab 组件级拆解

`Top Down JU Character.prefab` 是整个 Demo 的中心。它本身包含约九十多个子对象和三十多个脚本组件。其核心问题是功能高度集中，优点是 Demo 运行链路完整。

### 24.1 玩家根对象关键脚本

| 脚本 | 挂载位置 | 主要职责 | 与其他系统连接 |
|---|---|---|---|
| `JUCharacterController` | 玩家根对象 | 移动、旋转、动画参数、开火状态、装备 IK、死亡检查、驾驶状态同步。 | 读取 `JUInput`，引用 `JUInventory`、`JUHealth`、`DriveVehicles`、武器对象、动画器。 |
| `JUInventory` | 玩家根对象 | 保存所有可装备物品、拾取检测、自动装备、槽位顺序、当前手持物。 | 被 `JUCharacterController` 和 `ItemSwitchManager` 调用。 |
| `ItemSwitchManager` | 玩家根对象 | 监听换枪输入，调用角色切换到下一个、上一个或指定物品。 | 读取 `JUInput` 或鼠标滚轮，调用 `JUCharacterController.SwitchToItem`。 |
| `DriveVehicles` | 玩家根对象 | 查找附近车辆、进入车辆、退出车辆、驾驶中同步角色到座位。 | 读取交互输入，修改角色移动、碰撞、动画和当前车辆引用。 |
| `JUHealth` | 玩家根对象 | 生命值、死亡状态、受伤反馈。 | 被子弹、伤害器、身体部位和 UI 血条读取或调用。 |
| `DamageableBody` | 玩家根对象 | 管理身体部位伤害倍率。 | 关联多个 `DamageableBodyPart`。 |
| `AdvancedRagdollController` | 玩家根对象 | 死亡或重击后的布娃娃切换。 | 被角色控制器和伤害系统读取状态。 |
| `AimOnMousePosition` | 玩家根对象 | 用鼠标屏幕坐标计算世界瞄准点。 | 写入角色 `LookAtPosition`。 |
| `AimOnRightJoystickDirection` | 玩家根对象 | 用右摇杆方向计算世界瞄准点。 | 写入角色 `LookAtPosition`，可自动进入射击模式。 |
| `AimControllSwitcher` | 玩家根对象 | 根据当前输入设备在鼠标瞄准和摇杆瞄准之间切换。 | 依赖 `JUInputManager.IsUsingGamepad`。 |
| `ProceduralDrivingAnimation` | 玩家根对象 | 驾驶时做角色身体、手脚或头部的程序动画。 | 依赖 `DriveVehicles` 的当前驾驶状态。 |
| `BodyLeanInert` | 玩家根对象 | 角色身体惯性倾斜。 | 读取移动或驾驶方向。 |
| `JUFootstep` | 玩家根对象 | 脚步声。 | 驾驶时会被 `DriveVehicles` 禁用。 |
| `JUFootPlacement` | 玩家根对象或脚部系统 | 脚部 IK 贴地。 | 被角色控制器在动画 IK 阶段驱动。 |
| `ResizableCapsuleCollider` | 玩家根对象 | 按蹲伏、趴下等姿态调整胶囊体。 | 与移动姿态和碰撞检测有关。 |

### 24.2 玩家主要数值配置

| 字段 | 当前意义 | 技术影响 |
|---|---|---|
| `Speed` | 基础移动速度，当前配置约为 `3`。 | 决定普通移动的线速度来源。 |
| `WalkSpeed` | 行走速度倍率，当前约为 `0.5`。 | 按住走路键或部分状态下减速。 |
| `RunSpeed` | 跑步速度倍率，当前约为 `1`。 | 正常移动速度倍率。 |
| `SprintingSpeedMax` | 冲刺最高倍率，当前约为 `1.9`。 | 影响冲刺加速后的最大移动速度。 |
| `RotationSpeed` | 旋转插值速度，当前约为 `5`。 | 影响角色朝移动方向或瞄准点转身的快慢。 |
| `FireModeMaxTime` | 射击模式保持时间，当前约为 `2` 秒。 | 停止射击后角色不会立刻退出射击姿态。 |
| `MovementAffectsWeaponAccuracy` | 移动是否影响武器精度。 | 开启后跑动会扩大准星和弹道误差。 |
| `OnMovePrecision` | 移动状态下额外精度影响值。 | 与准星 UI 和武器散布相关。 |
| `RagdollWhenDie` | 死亡时是否进入布娃娃。 | 决定死亡表现和物理状态。 |
| `EnableStepCorrection` | 是否启用台阶修正。 | 对台阶、坡面和障碍移动稳定性有影响。 |

### 24.3 玩家背包配置

| 配置项 | 当前作用 | 说明 |
|---|---|---|
| `HoldableItensRightHand` | 右手可持有物数组。 | 包含多把枪、近战武器、投掷物等。 |
| `HoldableItensLeftHand` | 左手可持有物数组。 | 用于双持或副手物品。 |
| `AllHoldableItems` | 所有可持有物合集。 | 便于统一遍历、隐藏、刷新。 |
| `AllItems` | 背包中所有物品合集。 | 包括武器、护甲、补给等。 |
| `SequenceSlot` | 数字槽位顺序。 | 换枪时按槽位或索引查找物品。 |
| `EnablePickup` | 是否允许拾取。 | 关闭后角色不会检测附近物品。 |
| `CheckerRadious` | 拾取检测半径，当前约 `1.5`。 | 以玩家为中心扫描附近可拾取对象。 |
| `UseDefaultInputToPickUp` | 是否使用默认拾取输入。 | 开启后按模板输入触发拾取。 |
| `AutoEquipPickedUpItems` | 拾取后是否自动装备。 | 当前适合 Demo 快速验证，但正式项目需要更细规则。 |
| `HoldTimeToPickUp` | 拾取按键保持时间。 | 防止误拾取。 |

### 24.4 玩家换枪配置

| 配置项 | 当前作用 |
|---|---|
| `IsPlayer` | 运行时根据 `Player` 标签判断，玩家才响应换枪输入。 |
| `UseOldInputSystem` | 当前为新输入系统路径，旧输入系统分支不作为主路径。 |
| `ItemToEquipOnStart` | 开始时默认装备物品编号，当前为 `-1` 表示不强制装备。 |
| `EnableNextAndPreviousWeaponSwitch` | 开启上一个、下一个武器切换。 |
| `EnableAlphaNumericWeaponSwitch` | 开启数字键直接切换槽位。 |
| `EnableMouseScrollWeaponSwitch` | 开启鼠标滚轮切换。 |
| `ScrollThreshold` | 滚轮触发阈值，当前约为 `0.1`。 |

### 24.5 玩家驾驶配置

| 配置项 | 当前作用 |
|---|---|
| `EnterVehiclesEnabled` | 是否允许进入车辆。 |
| `ExitVehiclesEnabled` | 是否允许退出车辆。 |
| `DisableCharacterOnEnter` | 上车后是否直接禁用角色对象；当前一般不禁用，而是让角色跟随座位。 |
| `DelayToReenableAction` | 上下车状态恢复延迟，当前约 `0.2` 秒。 |
| `UseDefaultInputs` | 是否使用默认交互输入。 |
| `MaxVehicleSpeedToEnter` | 车辆速度超过该值时不能上车，避免高速抢车。 |
| `MaxVehicleSpeedToExit` | 车辆速度超过该值时不能下车，当前值很高，Demo 基本允许下车。 |
| `MaxCharacterSpeedToEnter` | 玩家自身速度超过该值时不能上车。 |
| `CheckNearVehicles` | 附近车辆检测配置，使用范围扫描和标签过滤。 |

## 二十五、摄像机 Prefab 组件级拆解

`TopDown Camera Controller.prefab` 是独立摄像机控制器。它的根对象负责相机状态和跟随，子对象是真正的 `Camera`。它不是简单把摄像机固定在玩家上方，而是通过 `CameraState` 计算枢轴位置和真实相机位置。

### 25.1 摄像机对象结构

| 对象 | 组件 | 作用 |
|---|---|---|
| `TopDown Camera Controller` | `TDCameraController` | 根据玩家状态选择不同 `CameraState`。 |
| `TopDown Camera Controller` | `JUApplyCameraSettings` | 应用模板中的相机设置。 |
| 子级摄像机对象 | `Camera` | 真正负责渲染。 |
| 子级摄像机对象 | `PostProcessLayer` | 后处理层，依赖 `com.unity.postprocessing`。 |
| 子级摄像机对象 | `Shaker` | 处理相机震动或反馈。 |

### 25.2 摄像机关键字段

| 字段 | 作用 |
|---|---|
| `mCamera` | 指向真实摄像机对象。 |
| `TargetToFollow` | 运行时目标，启动后会被替换为玩家脊柱 `HumanoidSpine`。 |
| `NormalCameraState` | 普通移动状态使用的相机配置。 |
| `FireModeCameraState` | 射击姿态使用的相机配置。 |
| `AimModeCameraState` | 瞄准或开镜状态使用的相机配置。 |
| `DrivingVehicleCameraState` | 驾驶车辆时使用的相机配置。 |
| `DeadPlayerCameraState` | 玩家死亡时使用的相机配置。 |
| `CameraCollisionLayerMask` | 摄像机碰撞遮挡检测层。 |
| `CrosshairRaycastLayerMask` | 准星射线检测层。 |
| `CameraRecoilReaction` | 是否允许武器后坐力影响相机。 |

### 25.3 摄像机状态切换优先级

`TDCameraController.UpdateCharacterState` 不是互斥 `else if`，而是按顺序覆盖状态：普通、瞄准、射击、驾驶、死亡。最终优先级可以理解为：

```text
死亡 > 驾驶 > 射击 > 瞄准 > 普通
```

因此当玩家同时满足射击和驾驶时，驾驶状态会覆盖射击状态；当玩家死亡时，死亡状态会覆盖所有状态。这种优先级对正式项目很重要，后续自研时建议改成明确状态机，避免状态相互覆盖不透明。

### 25.4 摄像机运行时三段更新

| 生命周期 | 调用内容 | 原因 |
|---|---|---|
| `Start` | 找到玩家控制器，把跟随目标改成玩家脊柱。 | 让镜头跟随身体中心，而不是脚底或根节点。 |
| `Update` | 读取玩家状态，切换当前相机状态。 | 状态变化需要每帧响应输入和角色状态。 |
| `FixedUpdate` | 移动相机枢轴到目标位置。 | 枢轴跟随可以与物理移动节奏保持一致。 |
| `LateUpdate` | 设置真实相机位置和视野。 | 摄像机最终位置放在角色移动之后，减少画面抖动。 |

## 二十六、UI Prefab 组件级拆解

当前场景实例化的是 `JUTPS Sidescroller User Interface.prefab`。这说明模板作者复用了横版 UI 根节点来服务俯视角 Demo，因此正式项目不要被名称误导，应以实际挂载脚本为准。

### 26.1 UI 根节点职责

| 脚本或对象 | 作用 |
|---|---|
| `JUInputManager` | 输入系统的运行时入口，负责把 Unity 输入动作转换成模板静态输入状态。 |
| `JUGameManager` | 保存全局玩家、摄像机、暂停、移动控制、移动端控制等状态。 |
| `JUGameSettings` | 模板设置项入口，如画质、声音、控制相关设置。 |
| `JUPauseGame` | 暂停和恢复游戏。 |
| `Crosshair` | 准星显示、颜色、扩散、跟随鼠标。 |
| `UIHealhBar` | 玩家血条显示。 |
| `UIItemInformation` | 当前装备物品、弹药、耐久或数量显示。 |
| `InventoryUIManager` | 背包 UI 管理。 |
| `MobileRig`、`JoystickVirtual`、`ButtonVirtual` | 移动端虚拟按键和摇杆。 |
| `HitMarkerEffect` | 命中反馈 UI。 |
| `BloodScreen` | 受伤屏幕效果。 |

### 26.2 UI 中存在的缺失脚本现象

在 Prefab 序列化信息中，UI 根节点存在部分脚本显示为缺失。这通常来自模板依赖的第三方 UI、输入、设置或编辑器组件没有被完整解析。当前 Demo 能否运行，要以 Unity 控制台实际报错为准。技术拆解层面应注意：

1. 缺失脚本大多集中在 UI Prefab，而不是玩家核心控制器。
2. 如果运行时没有阻断错误，可以先把它们视为模板 UI 的非核心残留。
3. 正式项目不建议直接继承这个 UI 根节点，因为它把输入、设置、暂停、背包、移动端控制、准星和血条混在一起。
4. 后续应拆成 `输入服务`、`HUD`、`暂停菜单`、`背包界面`、`移动端控制` 等独立模块。

## 二十七、核心功能之间的连接矩阵

| 功能 | 上游输入 | 中间处理 | 下游结果 |
|---|---|---|---|
| 玩家移动 | `JUInputManager` 的移动轴 | `JUCharacterController.ControllerInputs`、`Movement` | 刚体速度、动画速度、身体倾斜、脚步声。 |
| 鼠标瞄准 | 鼠标屏幕坐标 | `AimOnMousePosition` 从摄像机发射射线 | 写入 `JUCharacterController.LookAtPosition`。 |
| 手柄瞄准 | 右摇杆方向 | `AimOnRightJoystickDirection` 计算角色周围目标点 | 写入 `LookAtPosition`，可触发射击模式。 |
| 瞄准方式切换 | 当前输入设备 | `AimControllSwitcher` 启停鼠标或摇杆瞄准组件 | 避免两个瞄准源同时改写目标点。 |
| 摄像机跟随 | 玩家角色状态 | `TDCameraController` 切换 `CameraState` | 相机位置、相机视野、死亡或驾驶镜头。 |
| 开火 | 射击按键 | `JUCharacterController` 调用当前手持武器 `UseItem` | `Weapon.Shot` 生成子弹和特效。 |
| 换枪 | 数字键、滚轮、上一把、下一把 | `ItemSwitchManager` 调用角色切换物品 | 背包刷新当前手持物，角色动画和 IK 更新。 |
| 拾取 | 拾取按键和半径检测 | `JUInventory.PickUpNearbyItem` | 物品进入背包，可自动装备。 |
| 伤害 | 子弹、近战、投掷物、物理碰撞 | `Damager`、`Bullet`、`DamageableBodyPart` | 调用 `JUHealth` 扣血，死亡后布娃娃。 |
| 驾驶 | 交互按键和附近车辆检测 | `DriveVehicles` 设置当前车辆与座位 IK | 角色停用移动，车辆接管运动，摄像机进入驾驶状态。 |
| AI 巡逻 | 路径点和感知检测 | `PatrolAI`、`WaypointPath`、`NavMeshAgent` | 巡逻、看见玩家、追踪、攻击。 |
| UI 显示 | 玩家、相机、武器、生命值 | `Crosshair`、`UIHealhBar`、`UIItemInformation` | 准星、血条、弹药和当前物品显示。 |

## 二十八、核心时序拆解

### 28.1 场景启动时序

```text
Unity 加载 Top Down Demo 场景
    ↓
实例化场景中的玩家、摄像机、UI、车辆、AI、拾取物和环境对象
    ↓
UI 根节点上的 JUInputManager 初始化输入动作
    ↓
JUGameManager 建立全局玩家、摄像机和暂停引用
    ↓
TopDown Camera Controller 在 Start 中寻找 TargetToFollow 的 JUCharacterController
    ↓
摄像机把跟随目标从玩家根对象切换到 HumanoidSpine
    ↓
玩家 JUCharacterController 缓存 Animator、Rigidbody、Collider、Health、Inventory 等引用
    ↓
ItemSwitchManager 延迟约 0.2 秒执行初始装备逻辑
    ↓
DriveVehicles 订阅 Interact 输入事件，准备上车或下车
    ↓
UI 脚本开始读取玩家生命、武器、准星射线和当前物品
```

### 28.2 每帧输入到角色状态的时序

```text
玩家输入键盘、鼠标或手柄
    ↓
JUTPSInputControlls 产生 Unity Input System 动作值
    ↓
JUInputManager 把动作值写入模板静态输入接口
    ↓
JUCharacterController.ControllerInputs 读取移动、射击、瞄准、换弹、跳跃、翻滚、蹲伏等按钮
    ↓
角色更新 HorizontalX、VerticalY、FiringMode、IsAiming 等状态
    ↓
FixedUpdate 中 Movement 根据状态计算速度
    ↓
Update 中 Rotate、WeaponOrientator、WieldingIKWeightController 更新朝向、武器和 IK
    ↓
Animator 参数被刷新，模型表现跟随逻辑状态变化
```

### 28.3 每帧摄像机时序

```text
角色状态变化
    ↓
TDCameraController.Update 读取 IsAiming、FiringMode、IsDriving、IsDead
    ↓
根据状态选择 Normal、FireMode、AimMode、Driving、Dead 中的一个 CameraState
    ↓
FixedUpdate 将相机枢轴移动到玩家脊柱附近
    ↓
LateUpdate 将真实摄像机移动到 CameraState 计算的位置
    ↓
SetFieldOfView 应用当前状态视野
    ↓
画面最终呈现俯视角跟随效果
```

### 28.4 每帧 UI 时序

```text
UI 根节点保持激活
    ↓
Crosshair 获取当前摄像机和玩家
    ↓
如果 FollowMousePosition 开启，准星跟随鼠标屏幕位置
    ↓
Crosshair 根据当前武器精度调整四个准星片的位置或缩放
    ↓
Crosshair 用相机射线检测准星下对象
    ↓
根据对象标签切换普通颜色、可射击颜色或不可射击颜色
    ↓
UIHealhBar 读取 JUHealth 当前生命值
    ↓
UIItemInformation 读取当前手持物、弹药、数量或耐久
```

## 二十九、摄像机设计方案拆解

### 29.1 当前模板方案

当前方案不是固定相机，而是“目标跟随 + 状态切换 + 枢轴插值”。摄像机只关心玩家状态，不直接处理战斗逻辑。

| 子功能 | 当前实现 | 优点 | 风险 |
|---|---|---|---|
| 跟随目标 | 启动后跟随玩家 `HumanoidSpine`。 | 跟随位置稳定，角色高度变化时不容易丢目标。 | 依赖人形骨骼命名和玩家引用。 |
| 状态切换 | 根据普通、射击、瞄准、驾驶、死亡切换 `CameraState`。 | 新状态可扩展，调参方便。 | 状态优先级隐含在代码顺序里。 |
| 相机移动 | 枢轴在 `FixedUpdate`，真实相机在 `LateUpdate`。 | 避免部分抖动，适配物理移动。 | 如果角色非物理移动，可能需要调整。 |
| 后坐力反馈 | 武器调用相机的反冲反馈。 | 射击反馈明确。 | 俯视角项目中过强后坐力可能影响可读性。 |
| 准星射线 | 相机提供准星检测层。 | UI 和武器能共享目标检测思路。 | 如果 UI、武器、瞄准使用不同层，可能命中不一致。 |

### 29.2 正式项目建议方案

正式类孤胆枪手项目建议设计自己的 `TopDownCameraRig`，只保留模板思想，不直接继承大类。

| 功能 | 建议设计 |
|---|---|
| 跟随 | 只跟随玩家中心点，支持固定高度、固定俯角和轻微平滑。 |
| 瞄准偏移 | 根据鼠标与屏幕中心距离，让摄像机向瞄准方向轻微偏移。 |
| 状态 | 明确枚举 `普通`、`战斗`、`驾驶`、`死亡`、`剧情锁定`。 |
| 震动 | 独立 `CameraShakeService`，武器、爆炸、受伤都发事件。 |
| 遮挡 | 优先用地形透明化或简单射线，而不是复杂 TPS 碰撞。 |
| 参数 | 用 `ScriptableObject` 保存不同镜头配置。 |

## 三十、人物逻辑设计方案拆解

### 30.1 当前模板人物逻辑

`JUCharacterController` 同时处理输入读取、移动、旋转、射击模式、动画层、武器朝向、IK、脚步、布娃娃、物理伤害和驾驶状态。这种做法适合模板演示，但不适合作为正式项目核心。

| 子功能 | 当前实现方式 | 后续拆分建议 |
|---|---|---|
| 输入读取 | 角色控制器直接读 `JUInput`。 | 改成 `PlayerInputReader` 输出输入状态。 |
| 移动 | 控制器在 `FixedUpdate` 内计算刚体速度。 | 改成 `PlayerMotor`，只负责位移、碰撞和速度。 |
| 旋转 | 根据移动方向或 `LookAtPosition` 转身。 | 改成 `PlayerAimController` 提供朝向目标。 |
| 射击模式 | `FiringMode` 由射击和瞄准输入维持。 | 改成显式战斗状态，由武器系统请求进入。 |
| 动画 | 控制器直接写 Animator 参数和层权重。 | 改成 `PlayerAnimationPresenter`。 |
| 武器 IK | 控制器直接调整手臂、持枪点、瞄准点。 | 先简化，俯视角可减少复杂 IK。 |
| 生命死亡 | 控制器读取 `JUHealth` 和布娃娃。 | 改成 `Health` 事件驱动死亡流程。 |
| 驾驶 | 控制器和 `DriveVehicles` 双向改状态。 | 改成 `PlayerVehicleInteractor`，通过状态机切换控制权。 |

### 30.2 正式项目人物最小功能集

类孤胆枪手玩法的玩家角色最小可行功能建议如下：

1. 读取移动输入。
2. 用平面向量移动角色。
3. 用鼠标世界点决定角色朝向。
4. 按住或点击射击触发当前武器。
5. 支持武器切换。
6. 支持拾取补给、弹药、武器。
7. 支持生命、受伤、死亡和无敌短帧。
8. 支持简单动画状态：待机、移动、射击、死亡。
9. 后续再加入翻滚、冲刺、护甲、车辆和复杂 IK。

## 三十一、换枪逻辑设计方案拆解

### 31.1 当前模板换枪链路

```text
数字键、滚轮、上一把、下一把输入
    ↓
ItemSwitchManager.Update 判断是否是玩家、是否允许换枪
    ↓
NewInput_ItemSwitchController 或 OldInput_ItemSwitchController 读取具体输入
    ↓
调用 JUCharacterController.SwitchToNextItem、SwitchToPreviousItem 或 SwitchToItem
    ↓
JUCharacterController 转发给 JUInventory 找到目标物品
    ↓
JUInventory 关闭当前手持物，启用目标手持物
    ↓
刷新当前右手或左手物品引用
    ↓
角色动画层、手臂 IK、武器持握位置和 UI 信息刷新
```

### 31.2 当前模板换枪规则

| 规则 | 当前实现意义 |
|---|---|
| 玩家才响应输入 | `ItemSwitchManager` 会根据对象标签判断 `IsPlayer`。 |
| 死亡不能换枪 | 死亡、布娃娃、翻滚、近战攻击时直接返回。 |
| 支持数字键 | 数字键 `1` 到 `9` 对应槽位索引。 |
| 支持滚轮 | 滚轮正负方向切换下一把或上一把。 |
| 支持上一把和下一把按钮 | 适配键盘、手柄和移动端。 |
| 支持开始默认装备 | `ItemToEquipOnStart` 可指定开局装备，但当前玩家配置为不强制。 |
| 支持双手 | 右手和左手物品数组分开，但 Demo 主要围绕右手武器。 |

### 31.3 正式项目换枪建议

| 设计点 | 建议 |
|---|---|
| 武器栏 | 先做两个主武器槽、一个副武器槽、一个投掷物槽即可。 |
| 数据来源 | 用 `WeaponData` 定义武器，不让场景物体本身成为背包数据。 |
| 当前武器 | 用 `WeaponController.CurrentWeapon` 单一引用管理。 |
| 切换流程 | 输入请求切换，武器系统检查是否能切，播放收枪和掏枪，最后激活目标武器。 |
| UI | UI 监听武器切换事件，不主动扫描背包大数组。 |
| 禁止状态 | 换弹、死亡、上车、过场、硬直时禁止切换。 |

## 三十二、枪械与开火设计方案拆解

### 32.1 当前模板开火链路

```text
JUCharacterController 检测射击输入
    ↓
角色进入 FiringMode
    ↓
当前手持物如果是 Weapon，则调用 Weapon.UseItem
    ↓
WeaponControl 判断射速、弹药、是否可使用
    ↓
Weapon.Shot 计算射击方向、散布、摄像机修正、命中点
    ↓
BulletSpawn 或直接 Instantiate 子弹
    ↓
子弹设置 owner，忽略发射者碰撞体
    ↓
生成枪口火焰、弹壳、音效和后坐力
    ↓
扣除弹药，重置射速计时
```

### 32.2 当前武器核心字段

| 字段 | 作用 |
|---|---|
| `BulletsPerMagazine` | 单个弹匣容量。 |
| `TotalBullets` | 备用弹药数量。 |
| `BulletsAmounts` | 当前弹匣内剩余子弹。 |
| `NumberOfShotgunBulletsPerShot` | 霰弹枪单次射击生成的弹丸数量。 |
| `InfiniteAmmo` | 是否无限弹药。 |
| `Fire_Rate` | 射击间隔。 |
| `Precision` | 武器基础精度。 |
| `LossOfAccuracyPerShot` | 每次射击造成的精度损失或散布。 |
| `BulletPrefab` | 子弹 Prefab。 |
| `MuzzleFlashParticlePrefab` | 枪口火焰 Prefab。 |
| `Shoot_Position` | 枪口发射位置。 |
| `RaycastingLayers` | 射击修正或命中检测使用的层。 |
| `FireMode` | 自动、半自动、栓动、霰弹等开火模式。 |
| `AimMode` | 无瞄准、相机靠近、开镜等瞄准模式。 |
| `RecoilForce` | 武器模型后坐位移。 |
| `CameraRecoilMultiplier` | 相机后坐反馈倍率。 |
| `ShootAudio`、`ReloadAudio` | 射击与换弹音效。 |

### 32.3 正式项目武器建议

正式项目建议把模板的 `Weapon` 拆成三层：

| 层级 | 职责 |
|---|---|
| 武器数据层 | 伤害、射速、弹匣、散布、后坐力、弹道类型、音效、特效。 |
| 武器运行层 | 当前弹药、射速计时、是否换弹、当前持有者、开火请求。 |
| 武器表现层 | 枪口火焰、音效、动画、弹壳、屏幕震动、准星变化。 |

这样做可以让同一个武器数据被玩家、敌人、掉落物和商店共同使用，也便于后续做强化、词条、稀有度和升级系统。

## 三十三、开车逻辑设计方案拆解

### 33.1 当前模板上车链路

```text
DriveVehicles 每隔一定时间扫描角色附近碰撞体
    ↓
只保留标签匹配 EnterVehiclesAreaTag 的触发区域
    ↓
按距离排序，找到最近车辆区域
    ↓
可选 Linecast 检查玩家和车辆之间是否被障碍阻挡
    ↓
从触发区域父级获取 Vehicle
    ↓
从车辆获取 JUVehicleCharacterIK
    ↓
玩家按下 Interact
    ↓
检查 CanEnterVehicle、车辆速度、玩家速度、布娃娃状态
    ↓
设置 CurrentVehicle 和 CurrentVehicleCharacterIK
    ↓
StartEnterVehicleState 禁止重复交互
    ↓
延迟结束后 EndEnterVehicleState
    ↓
OnCharacterStartDriving 关闭脚步、收起武器、禁用移动、修改碰撞和动画层
    ↓
车辆系统接管移动
```

### 33.2 当前模板驾驶中状态

| 行为 | 当前实现意义 |
|---|---|
| 角色速度同步 | 角色刚体速度等于当前车辆刚体速度。 |
| 角色位置同步 | 如果车辆有座位 IK，则角色位置和旋转跟随座位。 |
| 无座位时兜底 | 角色位置和旋转跟随车辆根节点。 |
| 武器收起 | 上车时调用 `SwitchToItem(-1)`。 |
| 移动禁用 | 上车后调用 `DisableLocomotion`。 |
| 胶囊体触发 | 上车时角色碰撞体变为触发器，避免和车辆互相顶开。 |
| 重力关闭 | 上车时角色刚体关闭重力。 |
| 动画层关闭 | 上车时关闭普通上半身、持枪等动画层。 |
| 摄像机切换 | 玩家 `IsDriving` 为真后，摄像机进入驾驶状态。 |

### 33.3 当前模板下车链路

```text
玩家驾驶中按下 Interact
    ↓
DriveVehicles 判断 CanExitVehicle 和车辆速度
    ↓
如果车辆有 JUVehicleCharacterIK，则调用 GetExitPosition 找安全下车点
    ↓
如果没有专用下车点，则使用车辆侧向偏移作为兜底位置
    ↓
OnCharacterStopDriving 恢复角色移动、碰撞、重力和动画层
    ↓
StartExitVehicleState 标记正在下车
    ↓
延迟后 EndExitVehicleState 清空 CurrentVehicle 和 IK 引用
    ↓
触发 OnExitVehicle 事件
    ↓
摄像机从驾驶状态回到普通或战斗状态
```

### 33.4 正式项目是否保留车辆

对于类孤胆枪手项目，车辆不是最小核心循环的一部分。建议决策如下：

| 项目目标 | 建议 |
|---|---|
| 如果核心是室内刷怪、清房间、升级武器 | 暂时不要保留车辆系统。 |
| 如果核心是开放地图、生存逃离、载具碾压 | 可以保留车辆，但要独立成扩展模块。 |
| 如果只是 Demo 参考 | 只学习“控制权切换”和“摄像机状态切换”。 |
| 如果正式保留 | 车辆系统应和玩家武器、背包、IK 解耦。 |

## 三十四、AI 与刷怪设计方案拆解

### 34.1 当前模板 AI 链路

```text
Patrol AI Sample 启动
    ↓
读取 Waypoint Path 或 AI 路径
    ↓
PatrolAI 每帧检查可见目标
    ↓
FieldOfView 按角度、距离和 SensorLayerMask 找目标
    ↓
如果看见玩家，则记录 currentTarget 和 lastVisiblePosition
    ↓
距离较远时追踪，距离进入攻击范围时进入攻击模式
    ↓
攻击模式下让角色朝目标瞄准并使用武器
    ↓
如果丢失目标，则回到路径或最后可见点
```

### 34.2 当前 AI 与类孤胆枪手需求的差异

| 当前 AI | 类孤胆枪手更需要 |
|---|---|
| 巡逻、视野、看见玩家再攻击。 | 大量敌人持续追踪玩家。 |
| 使用 `NavMeshAgent` 和复杂状态。 | 简单、高性能、可批量生成的移动。 |
| 适合少量有枪敌人。 | 适合大量近战怪、远程怪、精英怪和 Boss。 |
| 关注 TPS 可见性和路径点。 | 关注波次、包围、刷怪点、掉落和难度曲线。 |
| 每个 AI 都可能有完整角色控制器。 | 敌人控制器应轻量化，避免模板玩家控制器级别的开销。 |

### 34.3 正式项目刷怪建议

| 模块 | 建议功能 |
|---|---|
| `EnemySpawner` | 按时间、波次、玩家位置和刷怪区域生成敌人。 |
| `EnemyDirector` | 控制当前场上敌人数量、强度预算和波次节奏。 |
| `EnemyMotor` | 简单追踪、避障、分离，必要时使用 NavMesh。 |
| `EnemyAttack` | 近战碰撞、远程弹幕、自爆、冲刺等攻击方式。 |
| `EnemyHealth` | 血量、受击硬直、死亡、掉落。 |
| `DropTable` | 弹药、补给、金币、经验、武器掉落。 |
| `DifficultyScaler` | 随时间提升敌人数量、速度、生命和精英比例。 |

## 三十五、后续落地功能清单

基于这个 Demo 的拆解，正式项目建议按以下优先级落地，而不是一次性迁移模板全部系统。

### 35.1 第一优先级：类孤胆枪手核心循环

| 功能 | 必要性 | 设计目标 |
|---|---|---|
| 玩家平面移动 | 必须 | 键鼠或手柄控制角色在平面移动。 |
| 鼠标世界瞄准 | 必须 | 鼠标指向哪里，角色和枪口朝向哪里。 |
| 俯视角摄像机 | 必须 | 稳定跟随玩家，能看到战斗区域。 |
| 单武器开火 | 必须 | 射线或子弹命中敌人，有音效和特效。 |
| 敌人追踪 | 必须 | 敌人能主动靠近玩家。 |
| 生命与伤害 | 必须 | 玩家和敌人都能受伤、死亡。 |
| 简单刷怪 | 必须 | 周期性生成敌人，形成战斗压力。 |
| 基础 HUD | 必须 | 显示生命、弹药、准星。 |

### 35.2 第二优先级：扩展战斗体验

| 功能 | 设计目标 |
|---|---|
| 多武器切换 | 支持主武器、副武器、投掷物。 |
| 换弹 | 弹匣制、换弹时间、缺弹反馈。 |
| 拾取补给 | 弹药、血包、护甲、临时增益。 |
| 敌人类型 | 近战怪、远程怪、快速怪、坦克怪。 |
| 波次系统 | 每波敌人数量和强度递增。 |
| 掉落奖励 | 击杀后掉落资源或升级材料。 |
| 命中反馈 | 受击闪白、伤害数字、击退、屏幕震动。 |

### 35.3 第三优先级：模板可选内容

| 功能 | 建议状态 |
|---|---|
| 复杂 IK 持枪 | 可选，俯视角不一定需要。 |
| 布娃娃 | 可选，可用于死亡表现，但注意性能。 |
| 护甲部位 | 可选，前期可用简单护甲值代替。 |
| 车辆驾驶 | 可选，只有地图和玩法需要时再做。 |
| 移动端虚拟摇杆 | 可选，先确定目标平台。 |
| 完整背包 UI | 可选，前期只做武器栏和掉落拾取即可。 |
| TPS 巡逻 AI | 不建议作为主 AI，只能作为少量持枪敌人参考。 |

## 三十六、最终拆解结论

`Top Down Demo` 的真正价值不在于把所有脚本当成正式项目底座，而在于它已经验证了一套完整链路：输入进入角色，角色控制移动和瞄准，武器根据瞄准点开火，子弹造成伤害，UI 反馈状态，摄像机根据角色状态切换镜头，车辆和 AI 作为扩展系统接入角色状态。

对于当前类孤胆枪手项目，建议保留 `Assets/ProjectRef` 作为可运行参考，不要在其内部直接改正式业务逻辑。正式代码应另建在 `Assets/Game` 下，并优先重写以下最小模块：

```text
输入读取
玩家移动
鼠标瞄准
俯视角摄像机
武器开火
敌人追踪
生命伤害
刷怪波次
HUD 显示
```

模板中最值得吸收的是：状态驱动摄像机、鼠标射线瞄准、武器方向修正、子弹忽略发射者、准星精度反馈、拾取半径扫描和上车控制权切换。模板中最应该避免直接继承的是：巨型角色控制器、全能背包、混合 UI 根节点、静态全局管理器、复杂 TPS AI 和高耦合车辆 IK。
