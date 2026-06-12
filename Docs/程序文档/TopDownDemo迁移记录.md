# Top Down Demo 迁移记录

## 目标

将参考模板工程中的 `Top Down Demo` 场景迁移到当前项目，作为类孤胆枪手项目的俯视角玩法参考，不作为最终游戏目录结构。

## 源工程

`F:\UnityTestProjects\JU TPS 3 - Third Person Shooter GameKit Vehicle Physics 3.0.52`

## 目标工程

`F:\UnityTestProjects\AlienShooterProject`

## 迁移资源位置

所有模板资源统一放在：

`Assets/ProjectRef/Julhiecio TPS Controller`

迁移后的目标场景：

`Assets/ProjectRef/Julhiecio TPS Controller/Demos/Demo Scenes/Top Down Demo/Top Down Demo.unity`

## 已处理内容

- 完整复制 `Assets/Julhiecio TPS Controller` 到 `Assets/ProjectRef/Julhiecio TPS Controller`。
- 保留全部 `.meta` 文件，维持原模板 GUID 引用。
- 添加嵌入包目录：`Packages/com.unity.postprocessing`。
- 在 `Packages/manifest.json` 添加 `com.unity.postprocessing` 依赖。
- 同步模板项目的以下设置：
  - `ProjectSettings/TagManager.asset`
  - `ProjectSettings/DynamicsManager.asset`
  - `ProjectSettings/InputManager.asset`
  - `ProjectSettings/NavMeshAreas.asset`
  - `ProjectSettings/GraphicsSettings.asset`
  - `ProjectSettings/QualitySettings.asset`
- 在 `ProjectSettings/ProjectSettings.asset` 合并 `UNITY_POST_PROCESSING_STACK_V2` 宏。
- 将 `ProjectSettings/EditorBuildSettings.asset` 的启用场景改为迁移后的 Top Down Demo。
- 修正模板中会实际执行的编辑器默认路径：
  - `Assets/ProjectRef/Julhiecio TPS Controller/Editor/Editor Scripts/Create Functions/JUTPSQuickSetup.cs`
  - `Assets/ProjectRef/Julhiecio TPS Controller/Scripts/Effects/JUFootstep.cs`

## 备份位置

迁移前的目标工程关键配置备份在：

`Temp/MigrationBackups/TopDownDemo_20260611_233653`

## 验证状态

- 已完成迁移后 GUID 依赖扫描：`Temp/DependencyScan/PostMigrationTargetScan.json`
- 扫描结果：场景递归依赖总数 `650`，其中 `Assets/ProjectRef` 资源 `505`，Post Processing 包资源 `124`。
- 未发现由于移动到 `Assets/ProjectRef` 导致的新增丢失引用。
- Unity 当前实例曾完成脚本编译成功，未在最近日志中发现 `error CS` 编译错误。
- `Packages/packages-lock.json` 在当前 Unity 实例中尚未刷新出 `com.unity.postprocessing`；重新打开 Unity 后应由嵌入包/manifest 解析。

## 注意事项

- 模板源工程本身存在若干未解析 GUID，多数指向动画、字体或材质残留引用；迁移后未新增这类问题。
- 目标工程从 URP 设置临时切换为模板 Built-in 设置，以优先保证 Demo 原样运行。
- 后续正式游戏开发时，建议将参考模板内容继续保持在 `Assets/ProjectRef`，业务代码另建独立目录。
