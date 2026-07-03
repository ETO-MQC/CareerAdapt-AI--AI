# CareerAdapt AI V1 Handoff

本文是 V2 读取 V1 信息的主要入口，不复制完整 `history.md`。

## 1. V1一句话定位

第一代 MVP 以职业母档案为事实中心，把简历文本/PDF、岗位JD、经历匹配、AI建议、Fact Guard、岗位分支、双模板预览和PDF导出串成可演示的纵向闭环。

## 2. V1完整链路

```text
简历文本/PDF导入
-> CareerProfile
-> JD解析
-> RequirementMatch
-> AI建议
-> Fact Guard
-> JobAdaptationDraft
-> ResumeBranch
-> 双模板
-> A4预览
-> PDF导出
```

## 3. 已完成核心模块

- 文本导入和文本型PDF导入。
- PDF来源追踪：`fileHash`、`normalizedTextHash`、`aiInputHash`、`sourceQuote`、`pdf_import` locator。
- CareerProfile、JobDescription、RequirementMatch、JobAdaptationDraft、AiSuggestion、ResumeBranch、ResumeRevision、ExportRecord。
- JD Analyzer、Evidence Matcher、Resume Tailor、Fact Guard。
- 多岗位 verified ResumeBranch，版本历史，撤销/恢复。
- 两套模板：`classic-technical`、`modern-operations`。
- A4预览、overflow检测、浏览器打印PDF导出。
- C1/C2安全评估体系。

## 4. 当前主要Schema

- `src/domain/schemas/common.ts`：`FactProvenance`、`FactStatement`、PDF locator、风险等级。
- `profile.ts`：`CareerProfile`、`Experience`、`Evidence`、`Skill`、`Certificate`。
- `job.ts`：`JobDescription`、`JobRequirement`、`RequirementMatch`、匹配证据引用。
- `importDraft.ts`：`RawInputDocument`、`PdfImportSession`、`PdfPageText`、`ProfileImportDraft`、`JobAnalysisDraft`、`DraftCommit`。
- `adaptationDraft.ts`：`JobAdaptationDraft`、`JobAdaptationSnapshot`、`SuggestionOperation`。
- `ai.ts`：`AiSuggestion`、`FactGuardResult`、`AiLog`、AI任务输出。
- `branch.ts`：`ResumeBranch`、`ResumeRevision`、`ResumeBranchOperation`、`ExportRecord`。
- `resumeRender.ts`：`ResumeRenderModel`、模板ID、渲染区块、overflow状态。

## 5. 当前Dexie版本和表

当前实际 Dexie 版本：v7。

核心表：`profiles`、`jobDescriptions`、`rawInputs`、`pdfImportSessions`、`pdfPageTexts`、`profileImportDrafts`、`jobAnalysisDrafts`、`draftCommits`、`requirementMatches`、`matchOperations`、`jobAdaptationDrafts`、`aiSuggestions`、`adaptationSnapshots`、`suggestionOperations`、`resumeBranches`、`resumeRevisions`、`resumeBranchOperations`、`aiLogs`、`exportRecords`、`appMeta`。

## 6. Repository边界

`WorkspaceRepository` 是 V1 本地数据边界，负责：

- seed demo workspace、读写 profile/job/raw input/import draft。
- PDF import session 和 page text 管理。
- draft commit 幂等提交。
- RequirementMatch 保存、AI评估、人工覆盖、stale判断。
- JobAdaptationDraft 创建和建议操作。
- ResumeBranch 创建、编辑、恢复、撤销、归档、sync status。
- ExportRecord 幂等写入。
- workspace JSON 导出。

V2不得绕过 Repository 直接写核心事实数据。

## 7. AI Provider和安全

- 统一 AI Service，任务白名单在 `src/ai/tasks/registry.ts`。
- Provider：Mock、DemoCache、Fallback、OpenAI-compatible。
- Prompt集中在 `src/ai/prompts/`。
- AI日志只保存任务、provider、模型、prompt版本、hash、长度、状态和错误码，不保存完整简历文本、API Key或本地路径。
- C1/C2 eval 不替代人工验收，但安全指标不得下降。

## 8. 当前页面

- `/`：项目空间摘要。
- `/profile`：简历文本/PDF导入、Profile草稿确认。
- `/jobs`：JD解析、匹配、建议、生成分支。
- `/resume`：分支选择、模板预览、编辑、撤销、导出。
- `/export/probe`：V1 A4探针入口。

注意：多处页面仍隐式使用 `workspace.profiles[0]`，V2多Profile前必须显式化上下文。

## 9. 当前测试入口

- `pnpm verify`：typecheck、lint、unit、build。
- `pnpm test:e2e`：Playwright端到端测试。
- `pnpm test:ai:real`：真实模型可选联调。
- `pnpm test:c1:eval`：C1匹配安全评估。
- `pnpm test:c2:eval`：C2建议与Fact Guard安全评估。

## 10. 不可降低指标

- C2 `unsafeAllowed=0`。
- 未确认事实进入正式材料数量为0。
- 模板层生成新事实数量为0。
- 样式操作修改事实数量为0。
- 迁移丢失数据数量为0。

## 11. V1限制

- 没有大量模板库。
- 没有预览区双击编辑。
- 没有完整样式属性面板。
- 没有完整拖拽编辑。
- 没有DOCX导入导出。
- OCR后置。
- 不支持任意PDF原版式高还原编辑。
- PDF导出主要依赖浏览器打印，不是直接下载。
- 多Profile显式管理未完成。
- 完整求职过程管理和Application实体未完成。

## 12. V1待完成交付事项

- 快速启动脚本或启动说明。
- V1稳定 Git 检查点。
- Demo运行手册。
- 比赛截图、视频、PPT。
- 模型与开源组件说明。
- 第一代旧 Sprint 状态补记。

## 13. V2可复用模块

CareerProfile事实层、PDF文本导入、JD解析、Evidence Matcher、Resume Tailor、Fact Guard、ResumeBranch/Revision基础、ExportRecord、DemoCache/Fallback、C1/C2评估体系、A4预览和两套模板。

## 14. V2需重构或适配模块

ResumeBranch到ResumeDocument的编辑模型、模板注册与元数据、可视化编辑状态、样式配置、PDF直接导出、多Profile上下文、Application实体、导入到新编辑模型的Mapper。

## 15. 迁移注意事项

- 不删除旧数据，不伪造缺失关联。
- legacy_unverified 继续只读。
- ResumeRevision和ExportRecord继续保留。
- PDFImportSession和AI日志默认保留或脱敏裁剪，不保存原始文件Blob。
- 任何V2迁移必须幂等、可回滚、可测试，迁移后V1主链路仍能运行。

## 16. 文档入口

- V1：[`Plan.md`](../Plan.md)、[`history.md`](../history.md)、[`职适AI_产品需求文档_PRD_v1.0.md`](../职适AI_产品需求文档_PRD_v1.0.md)
- V2：[`plan2.md`](../plan2.md)、[`history2.md`](../history2.md)、[`docs/V2_START_HERE.md`](V2_START_HERE.md)、[`docs/v2`](v2)
