# Design — solidworks-api-dev Skill Adaptation for CSharpAndSolidWorks

- **Author**: Architect (via Claude brainstorming session 2026-10-13)
- **Status**: Draft for user review
- **Date**: 2026-10-13
- **Anchor commit**: `97e9b70` (本项目当前 commit)

---

## 1. 概述

`solidworks-api-dev` 技能是基于 [xarial/codestack](https://github.com/xarial/codestack) 本地化的 SolidWorks API 知识库（位于 `C:\Users\jim_w\.claude\skills\solidworks-api-dev\`）。

本项目 `CSharpAndSolidWorks` 是该技能最具体的"用户"——一个独立的 SolidWorks 二次开发资料库，覆盖独立应用、COM Add-in、PMP、MacroFeature、eDrawing 集成、零件对比、射线求交等常见场景。

本次优化的目标：**让技能在本项目语境下更可用**——通过新增项目适配层（不污染通用部分），让 Claude 调用技能时能直接命中本项目代码模式与硬约束，同时把本项目 Utility.cs / SwAddin.cs 等真实样例反哺为通用参考。

---

## 2. 范围

### 2.1 In-Scope（方案 A：「最小化大改」）

| 项 | 内容 |
|---|---|
| 新增文件 | `solidworks-api-dev/_local-projects/csharpandsolidworks.md` |
| 新增内容 | §1 项目特化提示 + §2 核心代码摘录 + §3 按钮演示索引 + §4 漂移策略 |
| SKILL.md 改动 | 末尾"模块导航"加 1 行 |
| 4 大模块 index.md 改动 | 顶部各加 1 行 |
| 摘录粒度 | Utility.cs 全文、SwAddin.cs 60~80 行、NewFeature 60~80 行、UserPMPage 50~70 行、4 个子工具各 30~50 行 |

### 2.2 Out-of-Scope（不做）

- ❌ 重排 4 大模块正文（solidworks-api / pdm / document-manager / edrawings）
- ❌ 重写 reference/interop-assemblies.md / csharp-vs-vbnet.md / api-help-links.md
- ❌ 修改 SKILL.md description / 触发场景
- ❌ 修改 _source/codestack/ 任何内容
- ❌ 自动化漂移检测
- ❌ 自动化测试（与本项目 CLAUDE.md 一致："没有自动化测试项目"）

---

## 3. 现有结构（仅读，不动）

```
C:\Users\jim_w\.claude\skills\solidworks-api-dev\
├── SKILL.md                              ← 仅末尾"模块导航"追加 1 行
├── _source/codestack\...                  ← 不动
├── solidworks-api/index.md               ← 顶部加 1 行链接
├── solidworks-pdm-api/index.md           ← 顶部加 1 行链接
├── document-manager-api/index.md         ← 顶部加 1 行链接
├── edrawings-api/index.md                ← 顶部加 1 行链接
└── reference/...                          ← 不动
```

参考 SKILL.md 现有结构（line 1-100），description 明确声明是为 SEPDM-Mate 项目服务；本项目新增是**平行**的第二个项目适配层。

---

## 4. 新增文件 `_local-projects/csharpandsolidworks.md`

### 4.1 文件级 frontmatter / 顶部声明

```markdown
# CSharpAndSolidWorks 项目适配

> 本文档是 `solidworks-api-dev` 技能针对 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks`
> 的项目特化层。它**不替代**本项目 `CLAUDE.md` / `README.md`，只补充"使用本项目代码反哺
> SolidWorks 二次开发实践"所需的最小提示。
>
> **本文件是静态快照**——基于 commit `97e9b70`。当原文件路径失效、commit 漂移超 6 个月、
> 或文件结构变化时，本项目适配层视为**过期**，调用本技能时优先回到 4 大模块的通用建议。
```

### 4.2 §1 项目特化提示（硬约束摘录）

摘录自本项目 `CLAUDE.md` "运行时前提（硬约束）"与"构建系统约束"中对 SW API 开发有影响的内容，**不复制 CLAUDE.md 全文**，附指针。

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

### 4.3 §2 核心代码摘录

每个摘录块统一头部标注 `<!-- source: ... -->`，格式：

```markdown
<!-- source: <相对路径>  L<start>-L<end>  commit=<sha-short> -->
```<lang>
// 原文件: <绝对路径>
// 摘录范围: 第 <start>-<end> 行（commit <sha-full> 时的快照）
// 模式: <一句话说明该 SOLIDWORKS API 模式>
[代码块]
```
```

#### 2.1 `CSharpAndSolidWorks/Utility.cs` (224 行, 全摘)

4 个核心 helper + 2 个数据结构：

- `ConnectToSolidWorks()` — 调用 `Paine.SolidWorks.Base.Connect.GetRunningSolidWorks()`（**注意：不是 `Marshal.GetActiveObject`**，是包内封装）
- `TraverseFeatures(Feature, bool, bool)` — 递归遍历 `GetFirstSubFeature` + `GetNextSubFeature`
- `ShowDimensionForFeature(Feature)` — `GetFirstDisplayDimension` + `GetNextDisplayDimension` 链
- `TraverseCompXform(Component2, long, bool, bool)` — 装配体递归遍历 + 随机上色 (`MaterialPropertyValues`)
- `GetRadomColor(string)` — 基于零件名包含"m1"返回红色
- `BodyModel` 数据类 + `BoxSize` 结构体

#### 2.2 `PaineTool/SwAddin.cs` (710 行, 摘 60~80 行)

类签名 + 生命周期关键：

```text
L16-30:  [Guid("a756419a-13a2-4fd4-b82a-97154dad52c0")] + [SwAddin(LoadAtStartup = true)]
L161-192: ConnectToSW(object ThisSW, int cookie)
        → SetAddinCallbackInfo → GetCommandManager → AddCommandMgr()
        → AttachEventHandlers (5 个 SldWorks events)
        → AddPMP() + AddNewFeaturePMP()
L194-212: DisconnectFromSW()
        → RemoveCommandMgr + RemovePMP + DetachEventHandlers
        → Marshal.ReleaseComObject + GC.Collect()/WaitForPendingFinalizers() 两次（关键！）
```

#### 2.3 `PaineTool/UserPMPage.cs` (摘 50~70 行)

PMP 控件注册 + 控件事件 → handler → SW API 数据流。

#### 2.4 `PaineTool/NewFeature/` (摘 60~80 行)

`NewFeatureHandler.cs`（`ISwComFeature`） + `NewFeaturePMPage.cs`（PMP 集成）。这是新增 MacroFeature 的模板。

#### 2.5 各子工具（各 30~50 行）

- `CSharpAndSolidWorks/ComparePart/` — `IPartDoc` 比较 API
- `CSharpAndSolidWorks/InsertDrawingNote/` — `IDrawingDoc.InsertNote`
- `CSharpAndSolidWorks/GetRayIntersectionWithBody/` — 射线求交
- `CSharpAndSolidWorks/MyeDrawing/` (VB.NET) — eDrawing Control 嵌入

#### 2.6 `CSharpAndSolidWorks/FormMain.cs` (236 KB)

**不摘代码**。改为 §3 按钮→API 索引表。

### 4.4 §3 按钮演示索引（FormMain.cs 不摘，改为索引表）

按 README §"功能详细说明" 与 FormMain.Designer.cs 中按钮事件方法名整理。**关键一行**（基于 grep 失败，需进一步核查按钮命名约定后再补全）：

| 用户意图 | FormMain 按钮名 | 调用 API | 摘录见 |
|---|---|---|---|
| 连接到 SolidWorks | （自动） | `Connect.GetRunningSolidWorks()` | §2.1 |
| 打开零件 | TBD | `ISldWorks.OpenDoc` | §2.1 |
| 读取自定义属性 | TBD | `IModelDoc2.CustomPropertyManager` | §2.1 |
| 遍历特征 | TBD | `IFeatureManager` + `IFeature` 递归 | §2.1 |
| ... | | | |

> **TODO 字段**：实施阶段用 Grep 抽取 `FormMain.Designer.cs` 中按钮事件实际命名后再写完整表格。本规格不固化按钮名（避免与真实代码漂移）。

### 4.5 §4 漂移策略

```markdown
## §4 漂移策略

本文件所有摘录头部 `commit=` 标注指向**快照 commit**（当前 `97e9b70`）。
当本项目 commit 变化涉及已摘录片段时：

- **优先信任原文件**——用户代码始终以 git HEAD 为准，本摘录视为过期
- **不自动同步**——避免每次本项目 commit 都触发技能文件改动
- **手工维护**——由维护者周期性（建议每 6 个月或在重大重构后）评估并：
  - 在摘录代码块上方追加 `<!-- drifted since <new commit>: see D:\path -->` 标注
  - 或重写整个摘录块并更新 `commit=` 字段
- **过期判定**：当原文件被删除 / 路径失效 / 摘录代码与原文件 > 30% 不一致 → 标记为过期
```

---

## 5. SKILL.md 与 4 大模块 index.md 的最小改动

### 5.1 SKILL.md 末尾"模块导航"追加 1 行

在现有"5. 跨模块参考"之后追加：

```markdown
### 6. [项目适配](./_local-projects/csharpandsolidworks.md)
- [CSharpAndSolidWorks](./_local-projects/csharpandsolidworks.md)：本仓库 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks` 的项目特化与代码反哺（基于 commit `97e9b70` 的静态快照）
```

### 5.2 4 大模块 index.md 顶部各加 1 行

模板（4 个文件同样适用）：

```markdown
> **本项目适配**：参见 [CSharpAndSolidWorks 项目实践](../_local-projects/csharpandsolidworks.md)
> ——基于 `D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks` commit `97e9b70` 的静态摘录
> 与按钮演示索引。
```

### 5.3 不修改 SKILL.md description

维持原 description 不变——避免改变技能的触发场景。本项目特化只在技能被触发后**作为补充资料**出现。

---

## 6. 数据流（用户 → 技能 → 本项目）

```
用户: "如何用 SolidWorks API 做 MacroFeature？"
 │
         ▼ 触发技能 solidworks-api-dev（命中 description 中 MacroFeature 相关关键词）
         │
         ▼ Claude 读 SKILL.md 模块导航
         │
         ▼ 看到 solidworks-api → 读 index.md 顶部
         │   顶部链接: → ../_local-projects/csharpandsolidworks.md
         │
         ▼ 读 _local-projects/csharpandsolidworks.md
         │   §2.4: PaineTool/NewFeature/ 摘录 (60~80 行)
         │   §3: 按钮演示索引 — NewFeature 按钮 → CreateNewFeature 入口
         │
         ▼ 结合 _source/codestack/solidworks-api/ 通用 MacroFeature 教程
         │
         ▼ 返回给用户: 通用模式 + 本项目 NewFeaturePMPage / NewFeatureHandler 具体骨架
```

---

## 7. 验证清单（人工执行，无自动化）

| 步骤 | 验证内容 | 通过标准 |
|---|---|---|
| V1 文件存在性 | `_local-projects/csharpandsolidworks.md` 已生成 | 文件存在且 UTF-8 with BOM |
| V2 路径连通 | §2 摘录块头部的 `source:` 路径在本项目 git 中能找到 | `git ls-files` 命中所有引用文件 |
| V3 引用闭环 | §3 按钮索引表与 §2 摘录代码互相链接 | 每个 API 至少有一处摘录或注释 |
| V4 4 大模块 index.md | 各加 1 行本项目链接 | `git diff` 显示预期行 |
| V5 SKILL.md | 末尾模块导航加 1 行 | `git diff` 显示预期行 |
| V6 触发回归 | 模拟用户问 "如何做 MacroFeature"，看技能是否引用本项目 NewFeature | 输出含 `NewFeature` 或 `_local-projects/csharpandsolidworks.md` |
| V7 漂移边界 | 故意改本项目 Utility.cs 1 行，看技能不立即失效 | 摘录头部 `commit=` 标注仍指向原 commit |
| V8 上游同步 | 模拟 `git pull --rebase` 更新 _source/codestack | 本次新增的 `_local-projects/` 不受影响 |

---

## 8. 完成标准（Definition of Done）

满足以下**全部**条件时交付完成：

1. `_local-projects/csharpandsolidworks.md` 文件存在，UTF-8 with BOM
2. SKILL.md 末尾模块导航有 1 行指向新文件
3. 4 大模块 `index.md` 顶部各有 1 行指向新文件
4. V1~V8 全部通过
5. 用户审查通过书面规格（即本文件）
6. 若技能目录是 git 仓库，commit 改动；非 git 则仅本地生效

---

## 9. 失败回退

- 若任何 V1~V8 失败，**保留修改但标注失败项**，不强行 commit
- 用户可选择：a) 修复后重试；b) 接受现状 commit；c) 全量回退（`git checkout` 或删除新文件）

---

## 10. 维护协议

- **触发更新**：
  - 本项目新增 SolidWorks API 演示按钮 → 更新 §3 索引表
  - 本项目代码结构变化（如 NewFeature 模板重写）→ 评估 §2.4 是否需重写
  - 上游 xarial/codestack 重大更新 → 不影响本文件（_source/ 与 _local-projects/ 物理隔离）
- **不触发更新**：
  - 本项目小修小补（变量重命名、注释调整）——摘录为快照，无需同步
  - 技能 _source/codestack/ 更新——本文件不依赖 _source 内容

---

## 11. 引用

- 本项目 CLAUDE.md：`D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\CLAUDE.md`
- 本项目 README.md：`D:\YunPan\Sync\Data\Projects\github\CSharpAndSolidWorks\README.md`
- 技能 SKILL.md：`C:\Users\jim_w\.claude\skills\solidworks-api-dev\SKILL.md`
- 上游源：`https://github.com/xarial/codestack`
- 锚定 commit：`97e9b70`（`chore(docs): add CLAUDE.md commands and verification sections; expand gitignore`）