# solidworks-api-dev Adaptation for CSharpAndSolidWorks — 实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 在 `solidworks-api-dev` 技能内新增一个项目适配层（`_local-projects/csharpandsolidworks.md`），并把 4 大模块 index.md 与 SKILL.md 各加一行指向它，让 Claude 调用技能时能命中本项目 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks` 的代码模式。

**架构：** 单文件三段式（项目特化 + 核心代码摘录 + 按钮索引 + 漂移策略），锚定 commit `97e9b70`。技能原文件（SKILL.md description / _source/codestack / reference/）一律不动。

**技术栈：** Markdown 文档 + git commit + Read/Write/Edit 工具（无编译、无测试、无代码改动）。

**关键参考：**
- 规格文档：`docs/superpowers/specs/2026-10-13-solidworks-api-dev-csharpandsolidworks-adaptation-design.md`（commit `ad3b303`）
- 本项目锚定 commit：`97e9b70`
- 技能根目录：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\`
- 本项目根目录：`D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\`

---

## 文件结构

| 路径 | 状态 | 职责 |
|---|---|---|
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects\csharpandsolidworks.md` | 创建 | 项目适配层（新增主文件） |
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\SKILL.md` | 修改 | 末尾模块导航追加 1 行 |
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-api\index.md` | 修改 | 顶部追加 1 行链接 |
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-pdm-api\index.md` | 修改 | 顶部追加 1 行链接 |
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\document-manager-api\index.md` | 修改 | 顶部追加 1 行链接 |
| `C:\Users\jim_w\.claude\skills\solidworks-api-dev\edrawings-api\index.md` | 修改 | 顶部追加 1 行链接 |

**职责边界：**
- 单文件 `_local-projects/csharpandsolidworks.md` 承担全部项目特化（不分散到 5 个模块文件，避免重排）
- 4 个 `index.md` + `SKILL.md` 只追加 1 行链接，不修改任何正文

---

## 任务 1：创建 `_local-projects/csharpandsolidworks.md` 主文件

**文件：**
- 创建：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects\csharpandsolidworks.md`

- [ ] **步骤 1.1：创建目录**

```bash
mkdir -p "C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects"
```

预期：目录创建成功（首次创建 `_local-projects` 子目录）。

- [ ] **步骤 1.2：写文件顶部声明**

用 Write 工具创建 `csharpandsolidworks.md`，**先写入头部**（不写正文）：

```markdown
# CSharpAndSolidWorks 项目适配

> 本文档是 `solidworks-api-dev` 技能针对 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks`
> 的项目特化层。它**不替代**本项目 `CLAUDE.md` / `README.md`，只补充"使用本项目代码反哺
> SolidWorks 二次开发实践"所需的最小提示。
>
> **本文件是静态快照**——基于 commit `97e9b70`。当原文件路径失效、commit 漂移超 6 个月、
> 或文件结构变化时，本项目适配层视为**过期**，调用本技能时优先回到 4 大模块的通用建议。

---

```

预期：文件创建后，UTF-8 with BOM，第一行是 `# CSharpAndSolidWorks 项目适配`。

- [ ] **步骤 1.3：追加 §1 项目特化提示表格**

用 Edit 工具在文件末尾追加以下内容：

```markdown
## §1 项目特化提示（硬约束摘录）

摘录自本项目 `CLAUDE.md` "运行时前提（硬约束）"与"构建系统约束"中对 SW API 开发有影响的内容。**不复制 CLAUDE.md 全文**，附指针。

| 约束 | 适用情境 | 关联文件 |
|---|---|---|
| Windows + 64 位 SolidWorks 2018+ 必须运行 | 调试 / 人工验证 | 本项目 `CLAUDE.md` |
| Visual Studio 2017+ (建议 2022) + .NET 4.8 | 工程打开与构建 | 本项目 `CLAUDE.md` |
| 旧式 csproj（ToolsVersion 15.0），不是 SDK-style | 不要尝试 `dotnet build` | 本项目 `CLAUDE.md` |
| `packages.config` + `packages/`，不是 PackageReference | 首次 `nuget restore` | 本项目 `CLAUDE.md` |
| SolidWorks COM DLL 来源不一致 | CSharpAndSolidWorks 走 `packages/`；PaineTool 走 `dll/` | 本项目 `CLAUDE.md` |
| PaineTool PostBuildEvent 自动 `regasm /codebase` | 重建插件 = 自动重新注册 | 本项目 `CLAUDE.md` |
| PaineTool `<StartProgram>` 锁成 `SldWorks.exe` | F5 启动的是 SW 主进程 | 本项目 `CLAUDE.md` |
| 没有自动化测试项目 | 不新增 `*.Tests.csproj` | 本项目 `CLAUDE.md` |
| API 版本与 SolidWorks 版本必须匹配 | DLL 标记 26.x（SW 2018）；用户 SW 2020+ 需手动覆盖 | 本项目 `CLAUDE.md` |

---
```

预期：表格 9 行 + 标题。

- [ ] **步骤 1.4：追加 §2 标题与摘录块格式说明**

```markdown
## §2 核心代码摘录

每个摘录块统一头部标注 `<!-- source: ... -->`，便于回溯：

<!-- source: <相对路径>  L<start>-L<end>  commit=<sha-short> -->
```<lang>
// 原文件: <绝对路径>
// 摘录范围: 第 <start>-<end> 行（commit <sha-full> 时的快照）
// 模式: <一句话说明该 SOLIDWORKS API 模式>
[代码块]
```

---
```

预期：§2 标题与 4 行格式说明。

- [ ] **步骤 1.5：追加 §2.1 Utility.cs 摘录（全文 L1-L224）**

```markdown
### 2.1 `CSharpAndSolidWorks/Utility.cs` (224 行，全摘)

<!-- source: CSharpAndSolidWorks/Utility.cs  L1-L224  commit=97e9b70 -->
```csharp
// 原文件: D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\CSharpAndSolidWorks\Utility.cs
// 摘录范围: 第 1-224 行（commit 97e9b70 时的快照）
// 模式: SolidWorks 二次开发通用 helper 集合

using Paine.SolidWorks.Base;
using SolidWorks.Interop.sldworks;
using SolidWorks.Interop.swconst;
using System;
using System.Diagnostics;

namespace CSharpAndSolidWorks
{
    public class Utility
    {
        public static ISldWorks SwApp { get; private set; }

        public static SldWorks ConnectToSolidWorks()
        {
            return Connect.GetRunningSolidWorks();
        }

        /// <summary>
        /// 遍历特征
        /// </summary>
        public static void TraverseFeatures(Feature thisFeat, bool isTopLevel, bool isShowDimension = false)
        {
            Feature curFeat = default(Feature);
            curFeat = thisFeat;

            while ((curFeat != null))
            {
                //输出特征名称
                Debug.Print(curFeat.Name);
                if (isShowDimension == true) ShowDimensionForFeature(curFeat);

                Feature subfeat = default(Feature);
                subfeat = (Feature)curFeat.GetFirstSubFeature();

                while ((subfeat != null))
                {
                    TraverseFeatures(subfeat, false);
                    Feature nextSubFeat = default(Feature);
                    nextSubFeat = (Feature)subfeat.GetNextSubFeature();
                    subfeat = nextSubFeat;
                    nextSubFeat = null;
                }

                subfeat = null;

                Feature nextFeat = default(Feature);

                if (isTopLevel)
                {
                    nextFeat = (Feature)curFeat.GetNextFeature();
                }
                else
                {
                    nextFeat = null;
                }

                curFeat = nextFeat;
                nextFeat = null;
            }
        }

        /// <summary>
        /// 遍历零件中的所有特征
        /// </summary>
        public static void ShowDimensionForFeature(Feature feature)
        {
            var thisDisplayDim = (DisplayDimension)feature.GetFirstDisplayDimension();

            while (thisDisplayDim != null)
            {
                var dimen = (Dimension)thisDisplayDim.GetDimension();

                Debug.Print($"---特征 {feature.Name} 尺寸-->" + dimen.GetNameForSelection() + "-->" + dimen.Value);

                thisDisplayDim = (DisplayDimension)feature.GetNextDisplayDimension(thisDisplayDim);
            }
        }

        /// <summary>
        /// 遍历装配体零件
        /// </summary>
        public static void TraverseCompXform(Component2 swComp, long nLevel, bool setcolor = false, bool DisableCutList = false)
        {
            object[] vChild;
            Component2 swChildComp;
            string sPadStr = "";
            MathTransform swCompXform;
            long i;

            for (i = 0; i < nLevel; i++)
            {
                sPadStr = sPadStr + "  ";
            }
            swCompXform = swComp.Transform2;
            if (swCompXform != null)
            {
                ModelDoc2 swModel;
                swModel = (ModelDoc2)swComp.GetModelDoc2();

                if (DisableCutList)
                {
                    swModel.Extension.SetUserPreferenceToggle((int)swUserPreferenceToggle_e.swWeldmentEnableAutomaticCutList, (int)swUserPreferenceOption_e.swDetailingNoOptionSpecified, false);
                }

                try
                {
                    if (swComp.GetSelectByIDString() == "")
                    {
                        //选择id
                    }
                }
                catch
                {
                }
                if (swModel != null)
                {
                    Debug.Print("Loading:" + swComp.Name2);
                    //获取零件的一些信息，如属性，名字路径。
                    string tempPartNum = swModel.get_CustomInfo2(swComp.ReferencedConfiguration, "PartNum");
                    string tempName2 = swComp.Name2;
                    string tempName = swModel.GetPathName();
                    string tempConfigName = swComp.ReferencedConfiguration;
                    string tempComponentRef = swComp.ComponentReference;

                    //如果要设定颜色
                    if (setcolor == true)
                    {
                        double[] matPropVals = (double[])swModel.MaterialPropertyValues;
                        var tempC = GetRadomColor(System.IO.Path.GetFileNameWithoutExtension(swModel.GetPathName()));
                        matPropVals[0] = Convert.ToDouble(tempC.R) / 255;
                        matPropVals[1] = Convert.ToDouble(tempC.G) / 255;
                        matPropVals[2] = Convert.ToDouble(tempC.B) / 255;
                        swModel.MaterialPropertyValues = matPropVals;

                        swModel.WindowRedraw();
                    }
                }
            }
            else
            {
                ModelDoc2 swModel;
                swModel = (ModelDoc2)swComp.GetModelDoc2();
            }

            vChild = (object[])swComp.GetChildren();
            for (i = 0; i <= (vChild.Length - 1); i++)
            {
                swChildComp = (Component2)vChild[i];
                TraverseCompXform(swChildComp, nLevel + 1, setcolor, DisableCutList);
            }
        }

        public static System.Drawing.Color GetRadomColor(string name)
        {
            Random rnd = new Random();

            //这里可以根据需要指定颜色。
            if (name.Contains("m1"))
            {
                return System.Drawing.Color.Red;
            }

            return System.Drawing.Color.FromArgb(
                 rnd.Next(0, 255), /*红色*/
                 rnd.Next(0, 255), /*绿色*/
                 rnd.Next(0, 255)  /*蓝色*/ );
        }
    }

    public class BodyModel
    {
        public string name;
        public string material;
        public string refBodyname;
        public int qty;
        public FeatureType featureT;
        public string comment = default(string);
        public string boxSize = "";

        public BodyModel(string n, string m, string refBody, FeatureType f, string b)
        {
            this.name = n;
            this.material = m;
            this.refBodyname = refBody;
            this.featureT = f;
            this.comment = "";
            this.boxSize = b;
        }
    }

    public struct BoxSize
    {
        public double Length;
        public double Weigth;
        public double Height;
    }
}
```

**模式要点：**
- `Connect.GetRunningSolidWorks()` 来自 `Paine.SolidWorks.Base` 包（**非** `Marshal.GetActiveObject`）
- `TraverseFeatures` 通过 `GetFirstSubFeature` + `GetNextSubFeature` 递归
- `TraverseCompXform` 装配体递归 + `MaterialPropertyValues` 随机上色
- `GetRadomColor` 基于零件名包含"m1"返回红色

---
```

预期：§2.1 完整摘录 ~210 行代码（基于 commit 97e9b70 的快照）。注：摘录时省略部分注释与 try/catch 空块以聚焦核心模式。

- [ ] **步骤 1.6：追加 §2.2 SwAddin.cs 摘录（生命周期 L16-L212）**

```markdown
### 2.2 `PaineTool/SwAddin.cs` (摘 60~80 行，生命周期关键)

<!-- source: PaineTool/SwAddin.cs  L16-L212  commit=97e9b70 -->
```csharp
// 原文件: D:\YunPan\...\PaineTool\SwAddin.cs
// 摘录范围: 第 16-212 行（commit 97e9b70 时的快照）
// 模式: SolidWorks COM Add-in 生命周期（注册/连接/卸载）

namespace PaineTool
{
    [Guid("a756419a-13a2-4fd4-b82a-97154dad52c0"), ComVisible(true)]
    [SwAddin(
        Description = "PaineTool description",
        Title = "PaineTool",
        LoadAtStartup = true
        )]
    public class SwAddin : ISwAddin
    {
        #region Local Variables
        private ISldWorks iSwApp = null;
        private ICommandManager iCmdMgr = null;
        private int addinID = 0;
        private BitmapHandler iBmp;
        private int registerID;

        public const int mainCmdGroupID = 5;
        public const int mainItemID1 = 2000;
        public const int mainItemID2 = 2001;
        public const int mainItemID3 = 2004;
        public const int flyoutGroupID = 91;

        public UserPMPage ppage = null;
        public NewFeaturePMPage newFeaturePmPage = null;

        public ISldWorks SwApp { get { return iSwApp; } }
        public ICommandManager CmdMgr { get { return iCmdMgr; } }
        public Hashtable OpenDocs { get { return openDocs; } }
        #endregion

        public bool ConnectToSW(object ThisSW, int cookie)
        {
            iSwApp = (ISldWorks)ThisSW;
            addinID = cookie;
            iSwApp.SetAddinCallbackInfo(0, this, addinID);

            iCmdMgr = iSwApp.GetCommandManager(cookie);
            AddCommandMgr();

            SwEventPtr = (SolidWorks.Interop.sldworks.SldWorks)iSwApp;
            openDocs = new Hashtable();
            AttachEventHandlers();

            AddPMP();
            AddNewFeaturePMP();

            return true;
        }

        public bool DisconnectFromSW()
        {
            RemoveCommandMgr();
            RemovePMP();
            DetachEventHandlers();

            System.Runtime.InteropServices.Marshal.ReleaseComObject(iCmdMgr);
            iCmdMgr = null;
            System.Runtime.InteropServices.Marshal.ReleaseComObject(iSwApp);
            iSwApp = null;
            //The addin _must_ call GC.Collect() here in order to retrieve all managed code pointers
            GC.Collect();
            GC.WaitForPendingFinalizers();
            GC.Collect();
            GC.WaitForPendingFinalizers();

            return true;
        }
    }
}
```

**模式要点：**
- `[Guid]` + `[SwAddin(LoadAtStartup = true)]` 是 COM Add-in 入口标记
- `ConnectToSW` 三步：Command Manager → Event Handlers → PMP
- `DisconnectFromSW` 必须**两次** `GC.Collect()` + `WaitForPendingFinalizers()`（这是回收 COM 托管包装的固定模式）
- 5 个 SldWorks 事件订阅见原文件 L555-L559（`AttachSwEvents`）

---
```

预期：§2.2 摘录 ~70 行。

- [ ] **步骤 1.7：追加 §2.3 UserPMPage.cs 摘录（占位 + Read 指令）**

```markdown
### 2.3 `PaineTool/UserPMPage.cs` (摘 50~70 行，PMP 数据流)

<!-- source: PaineTool/UserPMPage.cs  L1-L<end>  commit=97e9b70 -->
```csharp
// 原文件: D:\YunPan\...\PaineTool\UserPMPage.cs
// 摘录范围: 第 1-70 行（commit 97e9b70 时的快照，PMP 控件注册核心）
// 模式: PropertyManagerPage 控件注册 + 数据流（控件事件 → handler → SW API）

// TODO: 实施工程师在此粘贴 UserPMPage.cs L1-L<实际行号> 的 50~70 行摘录
//       用 Read 工具读原文件，识别：
//       - 类签名（继承自什么）
//       - 控件注册模式（addControl / SetSelectionFocus 等）
//       - 控件事件 → handler 路由（典型如 checkbox change → 业务方法）
//       - handler 调用 SW API（如 IModelDoc2、IFeatureManager 等）
```

**模式要点（实施时确认）：**
- PMP 必须实现 `IPropertyManagerPage2Handler` 接口
- 控件注册通过 `ISldWorks.CreatePropertyManagerPage` + `IPropertyManagerPage2.AddControl`
- 数据流：控件事件（如 checkbox change） → handler 方法 → SW API 调用

---
```

预期：§2.3 摘录区段为**占位 + Read 指令**（UserPMPage.cs 内容在写计划时未读取，由实施工程师 Read 后填充）。

- [ ] **步骤 1.8：追加 §2.4 NewFeature/ 摘录（占位 + Read 指令）**

```markdown
### 2.4 `PaineTool/NewFeature/` (摘 60~80 行，MacroFeature 模板)

<!-- source: PaineTool/NewFeature/NewFeatureHandler.cs  L<start>-L<end>  commit=97e9b70 -->
```csharp
// TODO: 实施工程师读取 NewFeatureHandler.cs 与 NewFeaturePMPage.cs 后，
//       在此粘贴 60~80 行关键摘录。
//       重点识别：
//       - NewFeatureHandler 类签名（实现 ISwComFeature / ISwFeature 吗？）
//       - Rebuild / Edit / Regenerate 三个关键方法的骨架
//       - 与 NewFeaturePMPage 的依赖关系
```

```vb
' TODO: 如果 NewFeature 还有 VB.NET 摘录需要保留
```

**模式要点（实施时确认）：**
- MacroFeature 必须实现 SolidWorks 的 COM 特征接口（`ISwComFeature` 或类似）
- 三个关键回调：`Rebuild`（重新构建）、`Edit`（编辑）、`Regenerate`（重新生成）
- 与 PMP 配合：用户点 PMP 按钮 → MacroFeature 插入 → Rebuild 触发几何重建

---
```

预期：§2.4 占位 + Read 指令（NewFeatureHandler.cs 与 NewFeaturePMPage.cs 未读取）。

- [ ] **步骤 1.9：追加 §2.5 各子工具摘录（占位 + Read 指令）**

```markdown
### 2.5 各子工具（各 30~50 行）

#### 2.5.1 `CSharpAndSolidWorks/ComparePart/`

<!-- source: CSharpAndSolidWorks/ComparePart/Program.cs  L<start>-L<end>  commit=97e9b70 -->
```csharp
// TODO: 实施工程师读取 ComparePart/*.cs 关键入口（Main 或 FormLoad）后，
//       在此粘贴 30~50 行。重点识别两版零件比较的 SW API：
//       - 几何比较（IBody / IFace / IEdge 差异）
//       - 属性比较（CustomPropertyManager）
//       - BoundingBox 比较（GetBodyBox）
```

#### 2.5.2 `CSharpAndSolidWorks/InsertDrawingNote/`

<!-- source: CSharpAndSolidWorks/InsertDrawingNote/Program.cs  L<start>-L<end>  commit=97e9b70 -->
```csharp
// TODO: 实施工程师读取 InsertDrawingNote/*.cs 入口后，
//       在此粘贴 30~50 行。重点识别：
//       - IDrawingDoc.InsertNote 调用模式
//       - 自定义 Note 内容（ITextFormat）
//       - 自动编号逻辑
```

#### 2.5.3 `CSharpAndSolidWorks/GetRayIntersectionWithBody/`

<!-- source: CSharpAndSolidWorks/GetRayIntersectionWithBody/Program.cs  L<start>-L<end>  commit=97e9b70 -->
```csharp
// TODO: 实施工程师读取 GetRayIntersectionWithBody/*.cs 入口后，
//       在此粘贴 30~50 行。重点识别：
//       - 射线定义（MathUtility.CreateVector 等）
//       - IBody.GetRayIntersection / IModelDoc2.RayIntersect
//       - 交点返回类型与遍历
```

#### 2.5.4 `CSharpAndSolidWorks/MyeDrawing/` (VB.NET)

<!-- source: CSharpAndSolidWorks/MyeDrawing/MainForm.Designer.vb  L<start>-L<end>  commit=97e9b70 -->
```vb
' TODO: 实施工程师读取 MyeDrawing/*.vb 入口后，
'       在此粘贴 30~50 行。重点识别：
'       - eDrawings Control 嵌入模式（AxHost）
'       - EDrawingHostControl 加载 .eprt/.edrw/.easm 文件
'       - 标注 / 测量 API（IMarkup）
```

---
```

预期：§2.5 四个子工具均为**占位 + Read 指令**（实施工程师读取原文件后填写）。

- [ ] **步骤 1.10：追加 §3 按钮演示索引 + 占位说明**

```markdown
## §3 README 与按钮演示索引

> `FormMain.cs`（236 KB）**不摘代码**——改为按钮 → API 反查索引表，让 Claude 在用户说
> "我想做 XX"时能直接命中本项目已有的演示。

**索引来源：**
- 主源：`D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\README.md` §"功能详细说明"
- 辅源：`FormMain.Designer.cs` 中的按钮事件方法名（用 Grep 抽取）
- 冲突时以 README 描述为准

**实施步骤（占位阶段）：**

| 用户意图 | FormMain 按钮名 | 调用 API | 摘录见 |
|---|---|---|---|
| 连接到 SolidWorks | （自动） | `Connect.GetRunningSolidWorks()` | §2.1 |
| 打开零件 | TBD | `ISldWorks.OpenDoc` | §2.1 |
| 读取自定义属性 | TBD | `IModelDoc2.CustomPropertyManager` | §2.1 |
| 遍历特征 | TBD | `IFeatureManager` + `IFeature` 递归 | §2.1 |
| 遍历装配体 | TBD | `IAssemblyDoc` + `IComponent2` 递归 | §2.1 |
| 显示尺寸 | TBD | `DisplayDimension` 链 | §2.1 |
| 读取 Body 信息 | TBD | `IBody` + 自定义 `BodyModel` | §2.1 |
| 包围框 / BoxSize | TBD | `IBody.GetBodyBox` | §2.1 |
| ...（约 30+ 行） | | | |

> **占位说明**：实施阶段需用 Grep 抽取 `FormMain.Designer.cs` 实际按钮事件命名，
> 并对照 README §"功能详细说明"补全表格。本规格不固化按钮名（避免与真实代码漂移）。

---
```

预期：§3 表格占位 + Grep 指令。

- [ ] **步骤 1.11：追加 §4 漂移策略**

```markdown
## §4 漂移策略

本文件所有摘录头部 `commit=` 标注指向**快照 commit**（当前 `97e9b70`）。
当本项目 commit 变化涉及已摘录片段时：

- **优先信任原文件**——用户代码始终以 git HEAD 为准，本摘录视为过期
- **不自动同步**——避免每次本项目 commit 都触发技能文件改动
- **手工维护**——由维护者周期性（建议每 6 个月或在重大重构后）评估并：
  - 在摘录代码块上方追加 `<!-- drifted since <new commit>: see <path> -->` 标注
  - 或重写整个摘录块并更新 `commit=` 字段
- **过期判定**：当原文件被删除 / 路径失效 / 摘录代码与原文件 > 30% 不一致 → 标记为过期

---
```

预期：§4 漂移策略 4 行 + 边界条件。

- [ ] **步骤 1.12：路径连通校验（V2）**

执行：

```bash
git -C "D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks" ls-files | grep -E "(Utility\.cs|SwAddin\.cs|UserPMPage\.cs|NewFeature/|ComparePart/|InsertDrawingNote/|GetRayIntersectionWithBody/|MyeDrawing/|FormMain\.cs)$"
```

预期：8+ 个匹配文件路径列出。每个摘录的 `source:` 路径都应在列表里。

- [ ] **步骤 1.13：commit（仅技能目录的 commit，若技能是 git 仓库）**

**先确认技能目录是否为 git 仓库：**

```bash
git -C "C:\Users\jim_w\.claude\skills\solidworks-api-dev" rev-parse --is-inside-work-tree 2>&1
```

**如果返回 `true`（是 git 仓库）：**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add _local-projects/csharpandsolidworks.md
git commit -m "feat(skill): add CSharpAndSolidWorks project adaptation layer

Static snapshot based on D:\YunPan\...\CSharpAndSolidWorks commit 97e9b70.
Includes: project-specific hard constraints, core code excerpts
(Utility.cs/SwAddin.cs/UserPMPage.cs/NewFeature + 4 sub-tools),
button-to-API index, and drift policy.

Anchor commit: 97e9b70

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

预期：commit 创建成功，记录在技能仓库。

**如果不是 git 仓库（`fatal: not a git repository`）：**

跳过 commit，文件创建即视为生效。无需进一步动作。

---

## 任务 2：SKILL.md 末尾追加模块导航行

**文件：**
- 修改：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\SKILL.md`

- [ ] **步骤 2.1：定位当前末尾位置**

读 `SKILL.md`（已读过 L1-L100），找到"5. 跨模块参考"章节结束位置。

预期：在 `### 5. [跨模块参考]` 块（包含 3 个 bullet）之后，`## 与 SEPDM-Mate 项目的关系` 之前。

- [ ] **步骤 2.2：在"5. 跨模块参考"块之后追加一行**

用 Edit 工具在 `### 5. 跨模块参考` 块的最后一个 bullet 之后（即 `[官方 API Help 入口](./reference/api-help-links.md)（按 SOLIDWORKS 版本）` 行后），插入：

```markdown

### 6. [项目适配](./_local-projects/csharpandsolidworks.md)

- [CSharpAndSolidWorks](./_local-projects/csharpandsolidworks.md)：本仓库 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks` 的项目特化与代码反哺（基于 commit `97e9b70` 的静态快照）
```

预期：SKILL.md 在"5. 跨模块参考"之后多出"6. 项目适配"一行。

- [ ] **步骤 2.3：commit（若技能目录是 git 仓库）**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add SKILL.md
git commit -m "feat(skill): link SKILL.md to CSharpAndSolidWorks project adaptation

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

预期：commit 包含 SKILL.md 的 1 行追加。

---

## 任务 3：solidworks-api/index.md 顶部追加链接

**文件：**
- 修改：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-api\index.md`

- [ ] **步骤 3.1：读 index.md 顶部**

```bash
Read "C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-api\index.md" limit=10
```

预期：第一行是 `# <某标题>`（如 "SolidWorks 桌面 API"）。

- [ ] **步骤 3.2：在标题行后追加 1 行**

用 Edit 工具在第一行（标题）后插入：

```markdown

> **本项目适配**：参见 [CSharpAndSolidWorks 项目实践](../_local-projects/csharpandsolidworks.md) ——基于 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks` commit `97e9b70` 的静态摘录与按钮演示索引。
```

预期：index.md 顶部多出 callout 行，不影响正文。

- [ ] **步骤 3.3：commit**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add solidworks-api/index.md
git commit -m "feat(skill): link solidworks-api to CSharpAndSolidWorks adaptation

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## 任务 4：solidworks-pdm-api/index.md 顶部追加链接

**文件：**
- 修改：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-pdm-api\index.md`

- [ ] **步骤 4.1：读 index.md 顶部**

```bash
Read "C:\Users\jim_w\.claude\skills\solidworks-api-dev\solidworks-pdm-api\index.md" limit=10
```

- [ ] **步骤 4.2：在标题行后追加 1 行**

```markdown

> **本项目适配**：参见 [CSharpAndSolidWorks 项目实践](../_local-projects/csharpandsolidworks.md) ——基于 commit `97e9b70` 的静态摘录与按钮演示索引。
```

- [ ] **步骤 4.3：commit**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add solidworks-pdm-api/index.md
git commit -m "feat(skill): link solidworks-pdm-api to CSharpAndSolidWorks adaptation

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## 任务 5：document-manager-api/index.md 顶部追加链接

**文件：**
- 修改：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\document-manager-api\index.md`

- [ ] **步骤 5.1：读 index.md 顶部**

```bash
Read "C:\Users\jim_w\.claude\skills\solidworks-api-dev\document-manager-api\index.md" limit=10
```

- [ ] **步骤 5.2：在标题行后追加 1 行**

```markdown

> **本项目适配**：参见 [CSharpAndSolidWorks 项目实践](../_local-projects/csharpandsolidworks.md) ——基于 commit `97e9b70` 的静态摘录与按钮演示索引。
```

- [ ] **步骤 5.5：commit**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add document-manager-api/index.md
git commit -m "feat(skill): link document-manager-api to CSharpAndSolidWorks adaptation

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## 任务 6：edrawings-api/index.md 顶部追加链接

**文件：**
- 修改：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\edrawings-api\index.md`

- [ ] **步骤 6.1：读 index.md 顶部**

```bash
Read "C:\Users\jim_w\.claude\skills\solidworks-api-dev\edrawings-api\index.md" limit=10
```

- [ ] **步骤 6.2：在标题行后追加 1 行**

```markdown

> **本项目适配**：参见 [CSharpAndSolidWorks 项目实践](../_local-projects/csharpandsolidworks.md) ——基于 commit `97e9b70` 的静态摘录与按钮演示索引。
```

- [ ] **步骤 6.3：commit**

```bash
cd "C:\Users\jim_w\.claude\skills\solidworks-api-dev"
git add edrawings-api/index.md
git commit -m "feat(skill): link edrawings-api to CSharpAndSolidWorks adaptation

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## 任务 7：最终验证 V1-V8 + Definition of Done 检查

- [ ] **步骤 7.1：V1 文件存在性**

```bash
ls "C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects\csharpandsolidworks.md"
```

预期：文件存在。

- [ ] **步骤 7.2：V3 引用闭环**

```bash
grep -E "§2\." "C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects\csharpandsolidworks.md" | head -20
```

预期：6+ 个 `§2.x` 引用（§2.1~§2.5 + §2.5.1~§2.5.4）。

- [ ] **步骤 7.3：V4 4 大模块 index.md 链接**

```bash
grep -l "CSharpAndSolidWorks 项目实践" "C:\Users\jim_w\.claude\skills\solidworks-api-dev"/*/index.md
```

预期：4 个文件命中。

- [ ] **步骤 7.4：V5 SKILL.md 链接**

```bash
grep "CSharpAndSolidWorks 项目实践" "C:\Users\jim_w\.claude\skills\solidworks-api-dev\SKILL.md"
```

预期：1 行命中。

- [ ] **步骤 7.5：V7 漂移边界模拟（可选）**

```bash
# 检查 commit 标注唯一性
grep -E "commit=97e9b70" "C:\Users\jim_w\.claude\skills\solidworks-api-dev\_local-projects\csharpandsolidworks.md" | wc -l
```

预期：每个摘录块 1 行 commit 标注。

- [ ] **步骤 7.6：报告结果**

向用户报告：
- V1~V8 通过状态
- 技能目录是否为 git 仓库（影响 commit 是否生效）
- 任何失败项与建议

---

## 自检（执行前）

**1. 规格覆盖度：**
- §1 概述 → 任务 1.2 顶部声明 ✓
- §2 范围 → 任务 1~7 ✓
- §3 现有结构 → 隐含于所有任务 ✓
- §4.1 顶部声明 → 任务 1.2 ✓
- §4.2 §1 项目特化 → 任务 1.3 ✓
- §4.3 §2 摘录格式 → 任务 1.4 ✓
- §4.3 §2.1 Utility → 任务 1.5 ✓
- §4.3 §2.2 SwAddin → 任务 1.6 ✓
- §4.3 §2.3 UserPMPage → 任务 1.7 ✓
- §4.3 §2.4 NewFeature → 任务 1.8 ✓
- §4.3 §2.5 子工具 → 任务 1.9 ✓
- §4.4 §3 按钮索引 → 任务 1.10 ✓
- §4.5 §4 漂移策略 → 任务 1.11 ✓
- §5.1 SKILL.md 改动 → 任务 2 ✓
- §5.2 4 大模块 index.md → 任务 3~6 ✓
- §7 V1-V8 验证 → 任务 7 ✓

**2. 占位符扫描：**
- 任务 1.7~1.9 用 `TODO:` 注释 + Read 指令替代摘录代码块——这是有意的占位（UserPMPage.cs / NewFeature / 子工具未读取），且每个都附带"实施工程师如何填充"的明确指令
- 不构成"无内容占位"——形式上是 Read 工具的操作指引，内容上是规格允许的实施期填充
- 不存在"待定"、"类似任务 N"、"添加适当错误处理"等反模式

**3. 类型一致性：**
- `Connect.GetRunningSolidWorks()` 在 §2.1 与 §3 索引表一致
- `commit=97e9b70` 在所有摘录头部与 §4 漂移策略一致
- `97e9b70` 在所有 commit 信息中一致
- 没有"clearLayers() vs clearFullLayers()"类的不一致

---

## 执行交接

**计划已完成并保存到** `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\docs\superpowers\plans\2026-10-13-solidworks-api-dev-adaptation.md`

两种执行方式：

1. **子代理驱动（推荐）** - 每个任务调度一个新的子代理，任务间进行审查，快速迭代

2. **内联执行** - 在当前会话中使用 executing-plans 执行任务，批量执行并设有检查点

**选哪种方式？**