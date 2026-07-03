# CareerAdapt AI 第二代开发历史

## 使用规则

- 只记录V2规划和V2代码。
- V1历史见 `history.md`，V1基线见 `docs/MVP_V1_HANDOFF.md`。
- 每轮V2开发必须同步更新 `plan2.md`。
- 不复制设计文档正文；只记录目标、修改文件、核心变化、迁移、验证、Bug、遗留问题和下一步。
- 超过15条完整开发记录后，提出归档到 `docs/archive/history2-YYYY-QX.md`。

## 当前摘要

- 当前阶段：V2-G0a 已完成，Resume Studio 最小垂直切片可运行。
- 当前已完成：V1基线交接、V1任务迁移审计、V2文档入口、V2设计文档、首个Goal设计、G0a派生编辑视图模型、预览区直接编辑、模板A/B统一内容标识、G0a回归测试。
- 当前数据库版本：Dexie v7，未执行V2迁移。
- 当前下一步：启动 V2-G1 前先确认范围；继续保留 V1 稳定检查点、启动说明和 Demo 材料收尾。
- 当前阻塞：无 G0a 代码阻塞；完整 E2E 既有 flaky、C1/C2 全量评估仍待下一轮按需重跑。

## 开发记录模板

### YYYY-MM-DD：标题

目标：

修改文件：

核心变化：

数据迁移：

验证结果：

发现的Bug：

遗留问题：

下一步：

## 开发记录

### 2026-07-03：V2-G0a Resume Studio最小垂直切片

目标：

- 在已完成的 V1 MVP 上实现第一个 V2 开发Goal：可在正式简历预览区选择区块、显式编辑、保存、撤销，并保持模板和导出稳定。
- 严格遵守用户补充的 G0a 硬约束：不持久化 ResumeDocument、不新增 Dexie 表、不升级 Dexie v8、不建立第二套 Revision 系统。

修改文件：

- 文档：`docs/v2/FIRST_GOAL.md`、`RESUME_STUDIO_SPEC.md`、`EDITOR_INTERACTION.md`、`DOMAIN_AND_ARCHITECTURE.md`、`plan2.md`、`history2.md`。
- 新增派生视图模型：`src/domain/resumeDocument/mapper.ts`。
- 简历预览与模板：`src/domain/resumeRender/mapper.ts`、`src/components/resume/A4ResumePreview.tsx`、`src/components/resume/templates/templateRegistry.tsx`、`src/app/globals.css`。
- 工作台与Repository：`src/app/resume/ResumeWorkspace.tsx`、`src/services/storage/repositories.ts`。
- 测试：`tests/unit/resumeDocument.test.ts`、`tests/unit/branch.test.ts`、`tests/e2e/stageV2G0aResumeStudio.spec.ts`。

核心变化：

- `ResumeDocument` 仅作为由当前 `ResumeBranch/currentRevision` 派生的编辑视图模型，Mapper 映射全部 contentItem，并显式标记 `visible`、`renderable`、`editable`、`guardStatus`。
- 模板A/B统一使用 `contentItemId/sourceItemId`，预览层支持单击选中、编辑按钮、双击、Enter/F2、Escape、Ctrl/Cmd+Enter。
- 文本保存复用 `editResumeBranch`、`expectedRevision`、`operationId`、事务与 Fact Guard；冲突时显示错误，不静默覆盖。
- 分支切换、revision变化、撤销、恢复、保存成功和退出编辑模式会清理草稿、选中、错误和 pending operationId。
- 编辑 UI 使用 `.no-print` 与导出层隔离，PDF 不包含选中边框、按钮、textarea、错误提示或编辑工具栏。
- Repository 禁止 legacy、archived、invalid_reference、缺少 currentRevision 的内容编辑；`refreshResumeBranchSyncStatus` 仍允许 invalid_reference 分支刷新状态。

数据迁移：

- 未新增 Dexie 表。
- 未升级 Dexie，当前仍为 v7。
- 未持久化 ResumeDocument。
- 未执行 V2 数据迁移。

验证结果：

- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm exec vitest run tests/unit/resumeDocument.test.ts tests/unit/branch.test.ts` 通过：8/8。
- `pnpm exec playwright test tests/e2e/stageV2G0aResumeStudio.spec.ts --project=chromium` 通过：2/2。
- `pnpm test` 通过：58/58。
- `pnpm build` 通过。

发现的Bug：

- 首次 e2e 发现点击“编辑”后 textarea 未自动获得焦点，导致 Escape 不会触发取消；已通过给编辑 textarea 增加 `autoFocus` 修复。

遗留问题：

- 未全量重跑完整 E2E、C1 eval、C2 eval。
- V1 Git 稳定检查点、快速启动说明、Demo材料、模型与开源组件说明仍待收尾。
- G1 是否引入拖拽库和右侧属性面板需另行确认。

下一步：

- 若继续 V2，建议单独确认 V2-G1 范围后再启动；不得在 G1 前自动进入 DOCX、OCR、多Profile、Application 或模板市场。

### 2026-07-03：第二代规划体系初始化

目标：

- 基于已完成的第一代 MVP，建立第二代独立开发计划体系。
- 完成第一代任务迁移审计，确立第二代 Resume Studio 主线。
- 只创建规划和导航文档，不启动第二代业务代码。

修改文件：

- 新建 `plan2.md`、`history2.md`。
- 新建 `docs/MVP_V1_HANDOFF.md`、`docs/V2_START_HERE.md`。
- 新建 `docs/v2/PRODUCT_VISION.md`、`RESUME_STUDIO_SPEC.md`、`TEMPLATE_SYSTEM.md`、`EDITOR_INTERACTION.md`、`IMPORT_AND_EXPORT.md`、`DOMAIN_AND_ARCHITECTURE.md`、`AI_AND_SAFETY.md`、`V1_TO_V2_MIGRATION.md`、`ROADMAP_AND_BACKLOG.md`、`FIRST_GOAL.md`、`DECISIONS.md`。
- 更新 `Plan.md`、`history.md`、`AGENTS.md`、`Claude.md`、`README.md`、`.gitignore`、`tests/README.md`。

核心变化：

- `Plan.md` 与 `history.md` 冻结为V1历史档案。
- `plan2.md` 与 `history2.md` 成为V2活动文档。
- 第二代主线确定为“导入 -> 可视化编辑 -> 正式模板 -> 岗位定制 -> 稳定导出”。
- 首个推荐Goal确定为 V2-G0a：Resume Studio 最小垂直切片。
- 本地/生成型验收用例通过 `.gitignore` 保持不进 Git。

数据迁移：

- 本次未修改业务 Schema。
- 本次未升级 Dexie。
- 本次未执行 V2 数据迁移。

验证结果：

- 已审查 PRD、`Plan.md`、`history.md`、`Claude.md`、`AGENTS.md`、`package.json`、当前 Schema、Dexie v7、Repository、页面、测试脚本和 Git 工作树状态。
- `pnpm verify` 通过：typecheck、lint、55个单元测试、生产构建全部通过。
- 本次为文档规划变更，未运行E2E、C1 eval或C2 eval；未开始V2业务代码。

发现的Bug：

- 未修改业务代码，未确认新的产品Bug。
- 审计发现页面仍存在隐式 `profiles[0]` 使用，应迁移到 V2 G6 或更早的显式上下文处理。

遗留问题：

- V1 Git 稳定检查点、快速启动说明、Demo材料、模型与开源组件说明仍待收尾。
- V2首个Goal需要人工审核后才能开始编码。

下一步：

- 人工审核 `plan2.md` 和 `docs/v2/FIRST_GOAL.md`。
- 审核通过后，单独启动 V2-G0a；不得在同一轮进入 DOCX、OCR、多Profile、Application 或模板市场。
