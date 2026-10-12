# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

SolidWorks 二次开发示例资料库（中文）。覆盖 C# / VB.NET 调用 SolidWorks COM API 的常见场景：独立应用、COM Add-in、自定义 PropertyManager Page、MacroFeature、eDrawing 集成、零件对比、射线求交等。完整使用文档见 `README.md`，本文件只补充"读 README 前需要先知道的事"。

## 运行时前提（硬约束）

- **Windows + 64 位 SolidWorks（2018+）必须安装并实际运行**。所有 API 走 COM（`SolidWorks.Interop.sldworks`），主程序通过 `Utility.ConnectToSolidWorks()` 抓取已运行的 `SldWorks` 进程；不启动 SolidWorks 直接跑 exe 只会拿到 `null`。
- **Visual Studio 2017 或更高（建议 2022）**，带 ".NET 桌面开发" 工作负载。目标 **.NET Framework 4.8**。
- **没有测试项目**。验证靠"在 SolidWorks 中运行 → 输出窗口 / 肉眼 / 文件比对"，无自动化测试工程可跑。

## 解决方案结构（6 个工程）

| 工程 | 类型 | 语言 | 入口 | 用途 |
| --- | --- | --- | --- | --- |
| CSharpAndSolidWorks | WinExe | C# | `Program.cs` → `FormMain.cs` | 独立应用，按钮演示每个 API |
| PaineTool | COM Library (Add-in) | C# | `SwAddin.cs`（`[SwAddin]`、`[Guid("a756419a-13a2-4fd4-b82a-97154dad52c0")]`、`ISwAddin`） | SW 启动时自动加载的插件 |
| InsertDrawingNote | 子工具 | C# | — | 工程图自动插入 Note |
| ComparePart | 子工具 | C# | — | 两版零件对比 |
| GetRayIntersectionWithBody | 子工具 | C# | — | 射线求交 |
| MyeDrawing | 子工具 | VB.NET | — | eDrawing 集成（轻量化预览） |

`PaineTool/NewFeature/` 是 SolidWorks 自定义 MacroFeature 模板（`NewFeatureHandler.cs` + `NewFeaturePMPage.cs`）。新增 MacroFeature 按这个目录结构抄。

## 构建系统约束（与常规 .NET 不同）

- **旧式 csproj（ToolsVersion 15.0），不是 SDK-style**。不要期待 `dotnet build`，用 MSBuild 或 VS 的"生成解决方案"（`Ctrl+Shift+B`）。
- **`packages.config` + `packages/`（.gitignore 排除），不是 PackageReference**。克隆后 `packages/` 不存在，首次构建需 VS 自动还原或手动 `nuget restore` 下载。
- **SolidWorks COM 引用 DLL 来源不一致**——这是历史遗留：
  - `CSharpAndSolidWorks.csproj` 里的 `SolidWorks.Interop.sldworks` 等指向 `..\packages\Paine.SolidWorks.Base.2018.0.0.1\lib\…`（受包管理）。
  - `PaineTool.csproj` 里的 `SolidWorks.Interop.*` 直接指向 `..\dll\…`（仓库内备份）。
  - 扩展新工程时**优先复用 CSharpAndSolidWorks.csproj 的写法**（走 packages），与 README 一致。
- **PaineTool 的 PostBuildEvent 会自动 `regasm /codebase` 注册 Add-in**（配合 `<RegisterForComInterop>true</RegisterForComInterop>`）。重建插件 = 自动重新注册；卸载用 `regasm /u`（见 README §"卸载插件"）。
- **PaineTool.csproj 的调试启动程序被锁成 SolidWorks 主程序**（`<StartProgram>D:\Program Files\SOLIDWORKS Corp\SOLIDWORKS\\SldWorks.exe</StartProgram>`）。F5 启动的是 SW 主进程，插件被加载；不要为插件单独配"启动外部程序"。

## 架构入口（修改前先读这几个文件）

- `CSharpAndSolidWorks/Utility.cs`（≈7.8 KB）：连接、遍历、常用 API 封装，几乎所有功能页都引用。新功能**先复用/扩展这里**的 helper，不要直接调 COM。
- `CSharpAndSolidWorks/FormMain.cs`（236 KB）：主窗体，每个按钮一个示例。新增 API 演示按"按钮 → 方法"的模式追加，不要新建并行结构。
- `PaineTool/SwAddin.cs`（≈26 KB）：插件生命周期（`ConnectToSW` / `DisconnectFromSW`），命令注册、事件订阅、PMP 装载都在这里。
- `PaineTool/UserPMPage.cs` + `PaineTool/NewFeature/`：PMP 与 MacroFeature 范例。

具体 API 代码片段（连接、文档、属性、特征、装配、工程图、批量导出等）已在 README §"功能详细说明"中给出，**本文件不重复**。

## API 版本与 SolidWorks 版本必须匹配

DLL 标记 26.x（SolidWorks 2018）。完整对照见 README §"API 版本对照表"。如果目标用户的 SW 是 2020+ 且出现"找不到方法 / 接口不匹配"：
1. 用 SW 安装目录 `api\redist` 下的 interop DLL 覆盖 `packages/Paine.SolidWorks.Base.2018.0.0.1/lib/` 与 `dll/`；
2. 同步修改 csproj 中对应 HintPath；
3. **不要**为了"兼容"把 `EmbedInteropTypes` 盲改成 `True`——这会引入版本绑死问题，按各 csproj 已显式声明的 `EmbedInteropTypes` 设置为准。

## 仓库内测试用资产

- `CSharpAndSolidWorks/TemplateModel/`：大量 `.SLDPRT` / `.sldasm` / `.DRWDOT` / `.dwg` 示例文档，作为功能演示的输入。
- 根目录 `TMYTest.SLDPRT`：README 测试 2（读取属性）的样本。
- `dll/`：SW 2018 interop DLL 的备份。
- `Solidworks API Help CN.xlsx`、`如何测试.mp4`：本地 API 帮助和测试演示视频。
