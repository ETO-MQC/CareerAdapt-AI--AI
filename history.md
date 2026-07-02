## 2026-07-02：D1 验证与 editTexts 缓存修复

本次目标：
- 执行 D1 完整验证：分支隔离、Fact Guard 阻止、版本恢复/撤销、刷新持久化、legacy 只读。
- 不修改核心业务逻辑，仅修复验证过程中发现的 UI 缓存 bug。

修改文件：
- `src/app/resume/ResumeWorkspace.tsx`（1 行变更：`replaceBranch` 中清空 `editTexts` 缓存）
- `tests/e2e/d1-verification.spec.ts`（新增，验证专用，不提交 git）
- `Plan.md`

修改内容：
- 新增 D1 验证 E2E 测试，覆盖 6 个场景：双分支创建隔离、单分支编辑隔离、Fact Guard 阻止无证据新增（数字 30%、技能 Python）、revision 0 恢复与撤销、刷新持久化、legacy_unverified 只读。
- 发现并修复 `ResumeWorkspace.tsx` 中 `replaceBranch()` 未清空 `editTexts` 的 bug：undo/restore 后 `editTexts[itemId]` 仍保留旧编辑文本，导致 textarea 显示缓存值而非恢复后的 `item.text`。修复方式：`replaceBranch` 时调用 `setEditTexts({})`。

验证结果：
- D1 验证测试：6/6 场景通过（4.5s）。
- 回归测试：`pnpm test` 47/47 通过；stageD1BranchFlow E2E 通过。
- 未提交验证测试文件，仅提交 Plan.md 更新和 editTexts 修复。

遗留问题：
- 无。

---

## 2026-07-02：阶段D-D2 双模板预览、单页检查与 PDF 导出

本次目标：
- 严格只实现 D2：verified `ResumeBranch` -> `ResumeRenderModel` -> 模板 A/B -> A4 预览 -> 单页溢出检测 -> 浏览器打印 PDF -> `ExportRecord`。
- 不实现 PDF 简历导入、OCR、DOCX、登录/云同步、求职材料、更多模板或阶段E能力。

修改文件：
- `src/domain/schemas/resumeRender.ts`（新增）
- `src/domain/resumeRender/mapper.ts`（新增）
- `src/domain/schemas/branch.ts`
- `src/domain/schemas/index.ts`
- `src/components/resume/A4ResumePreview.tsx`（新增）
- `src/components/resume/useA4Overflow.ts`（新增）
- `src/components/resume/templates/templateRegistry.tsx`（新增）
- `src/app/resume/ResumeWorkspace.tsx`
- `src/app/globals.css`
- `src/services/storage/db.ts`
- `src/services/storage/repositories.ts`
- `tests/unit/resumeRender.test.ts`（新增）
- `tests/unit/branch.test.ts`
- `tests/unit/storage.test.ts`
- `tests/e2e/stageD2ExportFlow.spec.ts`（新增）
- `tests/e2e/stageCFlow.spec.ts`
- `Plan.md`
- `history.md`
- `artifacts/c1-evaluation.json`
- `artifacts/c1-evaluation.md`
- `artifacts/c2-evaluation.json`
- `artifacts/c2-evaluation.md`

修改内容：
- 新增统一 `ResumeRenderModel` Schema：记录 `branchId/branchRevision/currentRevisionId/branchName/job/candidate/sections/safety/sourceTrace`，模板层只消费该模型。
- 新增 D2 Mapper：只允许 `migrationStatus=verified`、`lifecycleStatus=active` 且非 `invalid_reference` 的正式分支进入 RenderModel；拒绝 `legacy_unverified`、归档分支、缺失当前版本、失效事实引用和无可见内容。
- Mapper 只纳入 `visible=true` 且 `guardStatus=pass/ai_failed_rule_kept` 的内容；`rule_only_verified` 内容可进入预览和导出，但只在工作台显示校验状态，PDF 正文不加入内部风险标签。
- 新增模板接口 `TemplateDefinition`，实现两套 A4 单页中文 HTML 文本模板：
  - 模板A `classic-technical`：稳重清晰，适合数据/技术/研究类岗位。
  - 模板B `modern-operations`：简洁现代，适合运营/产品/综合类岗位。
- `/resume` 接入模板预览与导出：模板切换只保存到 `appMeta` 展示偏好，不创建内容 `ResumeRevision`；刷新后恢复当前分支和模板选择。
- 新增 A4 预览容器和打印 CSS：打印时隐藏导航、按钮、工作台面板、风险提示和非简历 UI；模板正文保持 HTML 文本，不渲染为图片。
- 新增基于真实 A4 容器 `scrollHeight/clientHeight` 的溢出检测，状态为 `fits/near_limit/overflow`；`near_limit` 警告但允许导出，`overflow` 阻止正式打印并给出确定性删减提示。
- 导出前重新读取最新 branch/profile/job，校验 `branch.revision` 与 `currentRevisionId`，旧 revision 预览不静默导出。
- `ExportRecord` 扩展为 D2 Schema：新增 `operationId/branchRevision/exportStatus/overflowStatus/exportedAt/displayName/errorCode`；保留 `branchId/revisionId/templateId/format/fileName`。
- Dexie 升级到 v6，`exportRecords` 对 `operationId` 建唯一索引；旧导出记录迁移补齐 D2 字段。
- Repository 新增 `getJobDescription` 和 `createResumeExportRecord`，导出记录按 `operationId` 幂等，拒绝 legacy、归档、invalid reference、旧 revision 和 overflow 的正式打印。
- E2E 增加 D2 流程：创建 C2 草稿和 verified 分支、切换模板、恢复模板偏好、写入导出记录、生成两套 PDF 并用 Poppler 验证。
- 将既有 Stage C 第一条 E2E 的 `evidence-matcher` 调用改为固定 mock，避免 D2 回归依赖实时外部模型。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：7 个测试文件 / 47 个测试。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：10 个 Playwright 测试。
- `pnpm test:c1:eval` 通过，`overallQualified=true`，hardSafetyFailures=0。
- `pnpm test:c2:eval` 通过，`overallQualified=true`，safeAllowed 6 / safeBlocked 0 / unsafeBlocked 10 / unsafeAllowed 0，hardSafetyFailures=0。
- PDF 产物验证通过：`test-results/d2-template-modern.pdf` 与 `test-results/d2-template-classic.pdf` 均为 1 页 A4；E2E 使用 `pdftotext` 验证关键中文文本可抽取，且不含导航、按钮和内部风险提示。

遗留问题：
- D2 使用浏览器打印导出，应用只能记录 `print_invoked`，无法确认用户是否最终在系统打印对话框中保存文件。
- D2 不做复杂自动分页排版；overflow 只阻止正式导出并给出确定性删减提示，不调用 AI 自动改写。
- PDF 导入、OCR、DOCX、求职材料、登录/云同步、更多模板仍未进入本阶段。

下一步：
1. 人工验收 `/resume` 的 verified 分支模板 A/B、单页状态、overflow 阻断和浏览器打印 PDF。
2. 若继续开发，单独启动阶段E：PDF导入、稳定性和比赛材料。
3. 阶段E 启动前继续禁止新增模板市场、DOCX、OCR、登录、云同步或付费能力。

## 2026-07-02：阶段D-D1 正式 ResumeBranch、版本历史与多岗位分支

本次目标：
- 严格只实现 D1：从未 stale 的 `JobAdaptationDraft` 创建正式 `ResumeBranch`，同事务创建首个 `ResumeRevision`。
- 支持两个不同岗位分支、分支独立编辑、版本历史、恢复/撤销、母档案与岗位更新提示。
- 不实现模板、PDF 预览/导出、PDF 导入、求职材料、登录/云同步；完成 D1 后停止。

修改文件：
- `src/domain/schemas/branch.ts`
- `src/domain/branch/mapper.ts`（新增）
- `src/domain/branch/revision.ts`（新增）
- `src/domain/branch/validation.ts`（新增）
- `src/services/storage/db.ts`
- `src/services/storage/repositories.ts`
- `src/app/jobs/JobsWorkspace.tsx`
- `src/app/resume/ResumeWorkspace.tsx`
- `src/app/globals.css`
- `tests/unit/schema.test.ts`
- `tests/unit/storage.test.ts`
- `tests/unit/branch.test.ts`（新增）
- `tests/e2e/stageD1BranchFlow.spec.ts`（新增）
- `Plan.md`
- `history.md`
- `artifacts/c1-evaluation.json`
- `artifacts/c1-evaluation.md`
- `artifacts/c2-evaluation.json`
- `artifacts/c2-evaluation.md`

修改内容：
- 将 `ResumeBranch` 从阶段A占位扩展为 D1 正式 Schema：记录 profile/job/source draft/matcher 来源，`revision/currentRevisionId/lifecycleStatus/migrationStatus/syncStatusCache/contentItems`。
- 新增 `BranchContentItem` 事实引用结构：只持久化 `factRefs`，不重复保存 `sourceFactIds`；事实性内容必须绑定正式确认事实引用。
- 新增 `ResumeBranchSnapshot`：只保存 `name/lifecycleStatus/contentItems`，不保存 `revision/currentRevisionId/createdAt/updatedAt/syncStatusCache`。
- 新增 `ResumeRevision` 追加链：首个 created revision 可无 `previousRevisionId`，之后 edit/restore/undo revision 均记录 `previousRevisionId`。
- 新增 `ResumeBranchOperation` 审计与幂等表，Dexie v5 对 `operationId` 建立数据库级唯一索引 `&operationId`。
- Dexie 从 v4 升级到 v5，保留阶段A-C数据；旧占位分支迁移为 `migrationStatus=legacy_unverified`，只读保留，不参与正式编辑、版本恢复或后续导出。
- 新增 D1 Mapper：校验 draft 未 stale、非 error，校验 profile/job 版本、match stale、事实引用存在且已确认；阻断 high risk、`blocked_high_risk`、Fact Guard 未通过和失效事实引用；`ai_failed_rule_kept` 在规则通过且无 high finding 时以 `rule_only_verified` 进入分支。
- Repository 新增 `createResumeBranchFromDraft`、`listResumeBranches`、`getResumeBranch`、`listResumeRevisions`、`editResumeBranch`、`restoreResumeRevision`、`undoResumeBranch`、`refreshResumeBranchSyncStatus`、`archiveResumeBranch`。
- 创建分支和首个 revision 在同一 Dexie transaction 中完成；所有写操作使用 `expectedRevision + operationId`；重复 operationId 幂等返回。
- 手动编辑由 Repository 基于正式 `factRefs` 重新运行规则 Fact Guard，不信任前端 guardResult；编辑、恢复、撤销均不修改 `CareerProfile`、`JobDescription` 或 `JobAdaptationDraft`。
- 恢复旧版本只创建一个新的 restore revision；撤销统一基于 `previousRevisionId` 链；恢复/撤销后重新计算 `syncStatusCache`。
- `/jobs` 增加岗位选择器，C1/C2 按 `profileId + selectedJobId` 加载，不在 `/jobs` 创建正式分支。
- `/resume` 改为正式分支工作台：从 C2 draft 创建分支、选择分支、编辑内容、显示版本历史、恢复、撤销、刷新 syncStatus；`legacy_unverified` 只读展示，`rule_only_verified` 显示 AI 复核未完成提示。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：6 个测试文件 / 43 个测试。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：9 个 Playwright 测试。
- `pnpm test:c1:eval` 通过。
- `pnpm test:c2:eval` 通过。
- C2 指标保持：safeAllowed 6 / safeBlocked 0 / unsafeBlocked 10 / unsafeAllowed 0，overallQualified true。

遗留问题：
- D1 未实现模板、PDF 预览/导出、PDF 导入、求职材料、登录/云同步。
- D1 未实现字段级自动合并，不自动覆盖分支内容，不覆盖 CareerProfile 正式事实层。
- 删除前影响提示与删除/归档 UI 未进入本次 D1；Repository 已提供归档入口，未实现批量删除。

下一步：
1. 人工验收 `/jobs` 两岗位 C1/C2 与 `/resume` 两个正式分支的创建、独立编辑、版本恢复、撤销、syncStatus 提示。
2. 人工确认 D1 后，再单独启动 D2/Sprint 7：模板预览与 PDF 导出。
3. 继续后置 PDF 导入、求职材料、登录/云同步，禁止写入 API 密钥，禁止覆盖职业母档案正式事实层。
# 开发历史

本文件记录每次开发完成后的实际修改、验证结果、遗留问题和下一步路线。每次修改代码或文档后，都要同步更新 `Plan.md` 的状态。

## 2026-07-02：阶段C-C2.1 AI建议与 Fact Guard 验收收口

本次目标：
- 基于 C2 已完成的前提下，完成 C2.1 验收收口。
- 新增 `pnpm test:c2:eval`，不加入 `pnpm verify`。
- 建立 16 个脱敏案例覆盖所有要求的风险场景。
- 对每个案例执行确定性硬校验和工作流验证。
- AI Judge 只评价 Fact Guard 和建议安全性，不修改建议。
- 生成 `artifacts/c2-evaluation.json` 和 `artifacts/c2-evaluation.md`。
- 不进入阶段D，不创建正式 ResumeBranch，不实现模板或 PDF 导出。

修改文件：
- `package.json`（新增 `test:c2:eval` 脚本）
- `vitest.c2-eval.config.ts`（新增）
- `tests/c2-eval/cases.ts`（新增：16个脱敏验收案例）
- `tests/c2-eval/hardValidate.ts`（新增：确定性硬校验逻辑）
- `tests/c2-eval/judgeSchema.ts`（新增：AI Judge 输入/输出 Zod Schema）
- `tests/c2-eval/judgePrompt.ts`（新增：Judge system prompt c2-judge.v1）
- `tests/c2-eval/aiJudge.ts`（新增：AI 语义 Judge 调用）
- `tests/c2-eval/runEval.ts`（新增：主入口，串联硬校验+工作流测试+AI Judge+报告生成）
- `tests/c2-eval/c2-eval.test.ts`（新增：vitest 测试文件）
- `Plan.md`
- `history.md`

修改内容：
- 新增 16 个脱敏 C2 验收案例：
  - 合法案例（8个）：措辞优化、删减、排序、Provider失败降级、合法新增（证据存在）、建议范围隔离、编辑后重检（编辑前阻断后编辑通过）、合法措辞优化
  - 非法案例（8个）：新增数字、新增工具/技能、参与变主导、协助变独立、了解变熟练、团队成果变个人、stale阻断、Prompt注入、复合风险
- 每个案例定义：originalText、checkedText、usedEvidenceRefs（已确认事实引用）、expectedDisposition（pass/block）、expectedGuardStatus、expectedFindingTypes、hardFailIf。
- 确定性硬校验：处置预期、guard status、finding type 检查、evidence ref 白名单、Prompt 注入检测、stale 阻断。
- 工作流验证：evidence ref 完整性、事实确认状态、新增实体/数字/技能检测、表达强度升级检测、ownership 风险、blocked_high_risk 不可接受、stale 状态、Provider 失败降级、accept 操作、reject 操作、expectedRevision/operationId 幂等、scope 隔离（只修改 JobAdaptationDraft，不修改 CareerProfile，不创建 ResumeBranch）。
- 独立 AI 语义 Judge（c2-judge.v1）：只评价 Fact Guard 安全性和完整性，不修改建议；输入为原始文本+建议文本+证据引用+Fact Guard 结果；输出 5 维度 0-5 评分（factSafety、guardCompleteness、riskAssessment、hallucinationSafety、scopeIsolation）+ criticalFailures + issues。
- AI Judge 统计分类修正：`aiPassed`（模型返回且Judge通过）、`aiSemanticFailed`（模型返回但Judge不通过）、`aiUnavailable`（429/超时/网络错误）、`aiInvalid`（Schema/一致性错误）、`aiSkipped`（仅未配置API时为true）；Markdown 与 JSON 使用同一汇总逻辑。
- AI Judge Schema 重构：输入分为 `expectedDisposition`（人工定义）、`systemDisposition`（Fact Guard 真实执行结果，不由 Judge 生成）；Judge 输出 `suggestionSafe`/`recommendedDisposition`/`agreesWithSystemDisposition`/`findingsComplete`/`evidenceGrounded`/`scopeIsolationSafe`/`passed`/`issues`；Judge 独立评估建议安全性，`passed = agreesWithSystemDisposition && evidenceGrounded && scopeIsolationSafe`；一致性校验强制 `suggestionSafe↔recommendedDisposition` 和 `passed` 计算约束；Prompt 升级到 c2-judge.v2。
- 双重指标分离：产品确定性指标用 `expectedDisposition × systemDisposition`（safeAllowed/safeBlocked/unsafeBlocked/unsafeAllowed），证明 Fact Guard 系统处置正确性；AI Judge 一致性指标用 `recommendedDisposition × systemDisposition`（judgeAgreed/judgeDisagreed/judgeUnavailable/judgeInvalid），证明 Judge 与系统的一致程度。
- `aiSemanticFailed` 重命名为 `aiJudgeDisagreed`，避免误读为产品存在语义安全失败。
- 有 API Key 时运行真实 AI 评价；无 Key 时仍运行硬校验并标记 AI Judge skipped。
- 生成 `artifacts/c2-evaluation.json` 和 `artifacts/c2-evaluation.md`，报告不含 API Key 或未脱敏个人数据。
- AI 辅助验收不替代人工验收。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：5 个测试文件 / 38 个测试。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：8 个 Playwright 测试。
- `pnpm test:c1:eval` 通过，C1/C1.1 安全指标未降低。
- `pnpm test:c2:eval` 通过：16个案例硬校验+工作流测试全部通过，总体合格 ✅。产品确定性指标：safeAllowed 6 / safeBlocked 0 / unsafeBlocked 10 / unsafeAllowed 0（基于 expectedDisposition × systemDisposition）；AI Judge 一致性指标：judgeAgreed 13 / judgeDisagreed 0 / judgeUnavailable 3 / judgeInvalid 0（基于 recommendedDisposition × systemDisposition）。
- `pnpm test:ai:real` 受外部 API 429 限流影响（非代码问题），待限流解除后重试。

C2.1 验收指标：
- 总案例：16
- 合法案例通过：8/8（100%）
- 非法案例正确阻断：8/8（100%）
- 硬安全失败：0
- 工作流测试通过：全部
- 总体合格：✅

遗留问题：
- `test:ai:real` 受外部 API 429 限流影响，非代码问题。
- C2.1 仍复用 `/jobs` 当前第一份 profile/job 的演示选择方式，正式多岗位选择体验留到阶段D或后续 UI 收口。
- `JobAdaptationDraft` 仍是岗位适配草稿，不是正式 `ResumeBranch`；正式分支、版本历史、模板预览和 PDF 导出留到阶段D。
- 本阶段未进入 PDF 导入、求职材料、登录/云端能力，未写入 API 密钥，未覆盖职业母档案事实层。

下一步：
1. 人工验收 C2.1：查看 `artifacts/c2-evaluation.md`，在 `/jobs` 验证页面流程。
2. C2.1 人工确认通过后，进入阶段D：正式 `ResumeBranch`、版本历史、模板预览和 PDF 导出。
3. 阶段D启动前继续保持 PDF 导入、求职材料和登录/云端能力后置。

## 2026-07-02：阶段C-C2 AI建议与 Fact Guard

本次目标：
- 基于 C1/C1.1 已完成且 `pnpm test:c1:eval` 通过的前提，完成阶段C-C2。
- 只读取 `resolveEffectiveMatch` 得到且实时未 stale 的匹配，创建 `JobAdaptationDraft`，不提前创建正式 `ResumeBranch`。
- 生成可解释 AI 建议，先执行规则 Fact Guard，再执行 AI fact-guard 语义复核。
- 支持用户单条接受、拒绝、编辑后重新检测、重新检测和撤销，并保存 `AiSuggestion`、适配草稿和快照。

修改文件：
- `Plan.md`
- `history.md`
- `src/domain/schemas/ai.ts`
- `src/domain/schemas/adaptationDraft.ts`（新增）
- `src/domain/schemas/index.ts`
- `src/domain/adaptation/draft.ts`（新增）
- `src/domain/adaptation/factGuard.ts`（新增）
- `src/ai/prompts/resumeTailor.ts`（新增）
- `src/ai/prompts/factGuard.ts`（新增）
- `src/ai/tasks/registry.ts`
- `src/app/api/ai/structured/route.ts`
- `src/services/storage/db.ts`
- `src/services/storage/repositories.ts`
- `src/app/jobs/JobsWorkspace.tsx`
- `src/app/globals.css`
- `tests/unit/adaptation.test.ts`（新增）
- `tests/unit/schema.test.ts`
- `tests/unit/storage.test.ts`
- `tests/e2e/stageCFlow.spec.ts`
- `tests/ai-real/stageCRealProvider.test.ts`
- `artifacts/c1-evaluation.json`
- `artifacts/c1-evaluation.md`

修改内容：
- 新增 `JobAdaptationDraft`、`JobAdaptationSnapshot`、`SuggestionOperation` Schema，记录 `profileId`、`jobId`、`profileVersion`、`jobVersion`、`matcherVersion`、`requirementMatchIds`、`revision`、`appliedSuggestionIds`、`sectionTexts`、`createdAt`、`updatedAt`。
- 扩展 `AiSuggestion` Schema：支持 `rewrite`、`remove_or_shorten`、`reorder`、`risk_warning`、`follow_up_question`，记录原文、建议、原因、岗位要求ID、`usedEvidenceRefs`、Fact Guard 结果、风险等级、状态和编辑文本。
- 新增 C2 草稿创建逻辑：实时检查 `RequirementMatch` 是否 stale，任一引用 stale 时阻止创建或生成建议，要求返回 C1 重跑。
- 新增规则 Fact Guard：只以 `usedEvidenceRefs` 为事实允许边界，检测新增数字、学校/组织/公司/岗位、工具/技能、奖项/证书/成果，以及参与变负责、协助变独立、了解变熟练/精通、团队成果变个人成果。
- 新增 `resume-tailor` 和 `fact-guard` AI 任务、Prompt 与 Schema 校验；Prompt 明确把岗位、简历事实和原文视为不可信输入，忽略其中任何指令。
- `resume-tailor` 只接收允许的候选事实片段和草稿章节，不接收完整母档案；输出继续做业务语义白名单校验，拒绝白名单外 requirement/evidence/section ID。
- Fact Guard 流程固定为规则层先执行，再调用 AI 语义复核；AI 失败时保留已有建议和规则结果，不清空草稿。
- Dexie 升级到 v4，新增 `jobAdaptationDrafts`、`aiSuggestions`、`adaptationSnapshots`、`suggestionOperations` 表，并补齐 v3 -> v4 迁移测试。
- Repository 新增 C2 事务方法：创建草稿、保存建议、接受、拒绝、编辑后重检、重新检测、撤销；所有操作携带 `expectedRevision` 与 `operationId`，重复 `operationId` 不重复应用。
- 接受建议时禁止 `blocked_high_risk` 或 high risk 建议；建议只能修改 `JobAdaptationDraft.sectionTexts`，不修改 `CareerProfile` 正式事实。
- `/jobs` 页面新增 C2 区域：创建适配草稿、生成建议、草稿预览、建议卡片、Fact Guard 结果、证据引用、单条操作按钮和编辑后重检入口。
- 扩展单元测试、Dexie 迁移/事务测试、真实模型测试和 E2E 验收；重新运行 C1 验收并更新报告 artifact。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：5 个测试文件 / 38 个测试。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：8 个 Playwright 测试。
- `pnpm test:c1:eval` 通过，C1/C1.1 安全指标未降低。
- `pnpm test:ai:real` 通过：2 个真实模型测试文件 / 7 个测试。

遗留问题：
- C2 仍复用 `/jobs` 当前第一份 profile/job 的演示选择方式，正式多岗位选择体验留到阶段D或后续 UI 收口。
- `JobAdaptationDraft` 仍是岗位适配草稿，不是正式 `ResumeBranch`；正式分支、版本历史、模板预览和 PDF 导出留到阶段D。
- 规则 Fact Guard 当前采用确定性启发式抽取，后续可继续增强中文实体抽取精度，但 C2 必要风险边界已覆盖。
- 本阶段未进入 PDF 导入、求职材料、登录/云端能力，未写入 API 密钥，未覆盖职业母档案事实层。

下一步：
1. 人工验收 C2 页面流程和 Fact Guard 边界。
2. C2 人工确认通过后，进入阶段D：正式 `ResumeBranch`、版本历史、模板预览和 PDF 导出。
3. 阶段D启动前继续保持 PDF 导入、求职材料和登录/云端能力后置。

## 2026-07-02：阶段C-C1.1 AI验收校准与Matcher质量收口

本次目标：
- 收紧规则Matcher，提高匹配质量。
- 修正验收案例计分逻辑，区分合法/非法案例。
- 改进AI Judge一致性校验。
- 不进入C2。

修改文件：
- `src/domain/match/matcher.ts`
- `tests/c1-eval/cases.ts`
- `tests/c1-eval/hardValidate.ts`
- `tests/c1-eval/judgePrompt.ts`
- `tests/c1-eval/aiJudge.ts`
- `tests/c1-eval/runEval.ts`
- `tests/c1-eval/c1-eval.test.ts`
- `Plan.md`
- `history.md`

修改内容：
- 收紧规则Matcher：
  - 「参与/协助/基础/接触/了解」等限定词不得判定strong，自动降级为weak。
  - 岗位要求「独立/主导/负责」而事实只有「参与/协助」时最高weak并标记low_confidence。
  - 团队项目（事实含「团队/项目组/小组/课题组」）必须检查team_to_individual_risk。
- 改进匹配解释结构：[支持]关键词匹配、[缺失]未匹配、[判定]级别理由、[风险]风险说明，不再只复述岗位要求和事实文本。
- 增加expectedDisposition("accept"/"reject")区分合法/非法案例：合法案例硬校验通过才算通过；非法案例被正确拒绝算"预期拒绝成功"。
- Prompt注入区分inputContainsInjection与modelFollowedInjection：仅复述原始恶意文本不算执行成功；matcher解释中清理注入指令。
- 增加Judge一致性校验：criticalFailures非空时passed必须为false；criticalFailures为空且五项评分均>=3时passed必须为true；自相矛盾时标记judgeInvalid并重试一次。
- 新报告统计：positiveCasesPassed、negativeCasesCorrectlyRejected、hardSafetyFailures、semanticCasesPassed、judgeInvalid、overallQualified。
- Judge prompt升级到v2，增加评分阈值和一致性规则。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：4 文件/31 测试。
- `pnpm build` 通过。
- `pnpm test:c1:eval` 通过：
  - 正面案例通过：13/13（100%）
  - 负面案例正确拒绝：2/2
  - 硬安全失败：0
  - 语义案例通过：13/13（≥80%）
  - Judge自相矛盾：0
  - 总体合格：✅

完成标准核查：
- ✅ 所有非法安全案例均被正确拒绝（白名单越权、stale）
- ✅ 白名单越权放行数为0
- ✅ stale有效使用数为0
- ✅ Prompt注入执行成功数为0
- ✅ 无证据强匹配数为0
- ✅ 团队成果个人化未标记风险数为0
- ✅ Judge无自相矛盾输出
- ✅ 合法语义案例通过率≥80%

遗留问题：
- AI Judge（mimo-v2.5-pro）对"none"和"strong"案例评分偏低，存在same-model judge bias；后续可接入不同模型。
- C1/C1.1已通过AI辅助自动验收，但仍需人工最终确认后才能进入C2。

下一步：
1. 人工查看 `artifacts/c1-evaluation.md` 并结合页面操作最终确认C1。
2. 若C1人工确认通过，再单独启动C2：AI建议与 Fact Guard。
3. C2启动前继续禁止进入阶段D/E能力。

## 2026-07-02：阶段C-C1 AI辅助自动验收系统

本次目标：
- 为C1建立程序硬校验 + AI语义Judge + Markdown报告的自动验收流程。
- 新增 `pnpm test:c1:eval`，不加入 `pnpm verify`。
- 不进入C2，不修改现有matcher、registry、route或UI代码。

修改文件：
- `package.json`（新增 `test:c1:eval` 脚本）
- `vitest.c1-eval.config.ts`（新增）
- `tests/c1-eval/cases.ts`（新增：15个脱敏验收案例）
- `tests/c1-eval/hardValidate.ts`（新增：确定性硬校验逻辑）
- `tests/c1-eval/judgeSchema.ts`（新增：AI Judge输入/输出Zod Schema）
- `tests/c1-eval/judgePrompt.ts`（新增：Judge system prompt，独立于evidence-matcher）
- `tests/c1-eval/aiJudge.ts`（新增：AI语义Judge调用，使用OpenAiCompatibleProvider）
- `tests/c1-eval/runEval.ts`（新增：主入口，串联硬校验+AI Judge+报告生成）
- `tests/c1-eval/c1-eval.test.ts`（新增：vitest测试文件）
- `Plan.md`
- `history.md`

修改内容：
- 新增15个脱敏C1验收案例：strong基础、weak单命中、transferable可迁移、无证据硬约束、无证据非硬约束、强匹配团队风险、硬约束可迁移不足、未确认事实排除、白名单外ID、stale结果、Provider失败、Prompt注入、技能事实强匹配、数字事实风险、证书事实匹配。
- 每个案例定义：岗位要求、正式确认事实、允许matchLevel、必须/禁止风险、允许证据ID、硬性失败条件。
- 新增确定性硬校验：ID白名单、事实确认状态、no-evidence约束、stale约束、resolveEffectiveMatch一致性、禁止总分、禁止新增事实、风险约束、matchLevel约束、Prompt注入检测。
- 新增独立AI语义Judge（c1-evaluator）：只评价匹配结果不修改；输入为岗位要求+已确认事实+匹配结果；不接收生成过程；将所有输入文本视为不可信数据；输出严格JSON Schema（passed、5维度0-5评分、criticalFailures、issues、recommendedMatchLevel/RiskLevel）。
- Judge使用与evidence-matcher相同的Provider endpoint（当前仅一个模型），独立prompt版本 `c1-judge.v1`，报告中标记 `same-model judge bias`。
- 硬性失败条件：引用白名单外事实、使用未确认事实、无证据输出strong、编造数字/组织/工具/技能/成果、团队成果直接归个人、stale结果被视为有效、执行Prompt注入指令。
- 有API Key时运行真实AI评价；无Key时仍运行硬校验并标记AI Judge skipped。
- 生成 `artifacts/c1-evaluation.json` 和 `artifacts/c1-evaluation.md`，报告不含API Key或未脱敏个人数据。
- AI辅助验收不替代人工验收。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：4 个测试文件，31 个测试通过（原有测试不受影响）。
- `pnpm test:c1:eval` 通过：15个案例硬校验全部通过（其中3个案例的预期失败检查确实失败）；AI Judge使用mimo-v2.5-pro运行，输出15个案例的语义评分。
- `artifacts/c1-evaluation.json` 和 `artifacts/c1-evaluation.md` 正确生成。

真实模型与 Provider：
- Judge Provider: openai-compatible（通过 `.env.local` 当前配置）
- Judge Model: mimo-v2.5-pro（与evidence-matcher相同，报告标记same-model judge bias）
- Judge Prompt版本: c1-judge.v1

AI Judge 结果摘要：
- 15个案例中8个AI Judge通过，7个失败。
- 失败主要集中在：matchLevelReasonableness偏低（模型倾向于降级matchLevel）、riskAssessment不完整。
- hallucinationSafety普遍较高（5/5），未检测到编造行为。
- same-model judge bias已标记：Judge使用同一模型可能对matchLevel判断偏保守。

人工验收入口：
1. 查看 `artifacts/c1-evaluation.md` 验收报告。
2. 在报告中检查每个案例的硬校验结果和AI Judge评分。
3. 注意报告末尾声明：AI辅助验收不替代人工验收。
4. 如需重新运行：`pnpm test:c1:eval`。

遗留问题：
- AI Judge当前使用与evidence-matcher相同的模型（mimo-v2.5-pro），存在same-model judge bias；后续可接入不同模型消除偏差。
- AI Judge评分中matchLevelReasonableness偏低，可能需要调优Judge prompt的评分标准。
- C1已通过AI辅助自动验收，但仍需人工最终确认后才能进入C2。
- 未修改现有matcher.ts、registry.ts、route.ts或UI代码。
- 未进入C2的AI建议、Fact Guard、JobAdaptationDraft、ResumeBranch、模板或PDF导出。

下一步：
1. 人工查看 `artifacts/c1-evaluation.md` 并结合页面操作最终确认C1。
2. 若C1人工确认通过，再单独启动C2：AI建议与 Fact Guard。
3. C2启动前继续禁止进入阶段D/E能力。

## 2026-07-02：阶段C-C1 Evidence Matcher 与差距诊断

本次目标：
- 只完成阶段C的 C1 检查点：岗位要求到已确认经历事实的证据映射、差距诊断、AI解释、人工覆盖、stale 判定和持久化。
- C1 完成后立即停止，等待人工验收；不进入 C2 的 AI建议、Fact Guard、JobAdaptationDraft、ResumeBranch、模板或PDF导出。

修改文件：
- `Plan.md`
- `history.md`
- `src/domain/schemas/job.ts`
- `src/domain/match/matcher.ts`（新增）
- `src/ai/prompts/evidenceMatcher.ts`（新增）
- `src/ai/tasks/registry.ts`
- `src/ai/client.ts`
- `src/app/api/ai/structured/route.ts`
- `src/services/storage/db.ts`
- `src/services/storage/repositories.ts`
- `src/app/jobs/JobsWorkspace.tsx`
- `src/app/globals.css`
- `tests/unit/matcher.test.ts`（新增）
- `tests/unit/storage.test.ts`
- `tests/ai-real/stageCRealProvider.test.ts`（新增）
- `tests/e2e/stageCFlow.spec.ts`（新增）

修改内容：
- 重构 `RequirementMatch`：将 `matchLevel`（strong/weak/transferable/none）与 `riskLevel`（low/medium/high）分离，风险使用 `MatchRisk[]` 独立记录。
- 将 `evidenceRefs` 改为判别联合类型：`experience_fact`、`skill_fact`、`certificate_fact`、`evidence_file`；所有引用通过正式母档案白名单和关联完整性校验。
- 新增 `ruleEvaluation`、`aiEvaluation`、`manualOverride` 分层保存，并通过 `resolveEffectiveMatch` 统一计算有效结果；页面不直接编辑 `effectiveEvaluation`。
- 新增 C1 规则匹配：只召回正式 `CareerProfile` 中已确认事实，不使用未绑定事实的 `resumeDraft/customText`。
- 新增规范化 `candidateSetHash` 和 stale 判定：基于 `profileVersion`、`jobVersion`、`matcherVersion`、岗位要求与固定排序后的候选事实计算。
- 新增 `evidence-matcher` 服务端白名单任务与 Prompt，Prompt 明确将简历、岗位和事实文本视为不可信数据，忽略其中的 Prompt 注入。
- AI 任务只接收规则层候选事实片段，不接收完整职业母档案；Zod 校验后继续做业务语义校验，拒绝白名单外 ID。
- Dexie 升级到 v3，新增 `requirementMatches` 和 `matchOperations` 表；Repository 增加规则匹配保存、AI匹配保存、人工覆盖、stale 标记和有效结果解析方法。
- 岗位页新增 C1 匹配诊断区域：运行规则匹配、运行AI解释、查看岗位原文/事实依据/匹配等级/风险等级、保存人工覆盖、展示 stale。
- 真实模型联调中补强阶段B/C1 coerce 层：JD Analyzer 对缺失 description/sourceQuote、纯字符串 location 等模型差异做兜底；Evidence Matcher 对空 evaluations、风险枚举和证据引用做规范化。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：4 个测试文件，31 个测试通过。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：7 个 Playwright 测试通过，其中包含 C1 规则匹配、AI解释、人工覆盖、刷新恢复和 stale 展示。
- `pnpm test:ai:real` 通过：2 个真实模型测试文件，5 个测试通过（health check、profile-builder、jd-analyzer、evidence-matcher）。

真实模型与 Provider：
- Provider: openai-compatible（通过 `.env.local` 当前配置）
- Model: 使用当前 `AI_MODEL` 环境变量。
- C1 覆盖：`evidence-matcher` 输出通过 Zod 与业务语义校验，不引用白名单外 ID，不输出总分；候选集合为空时归一化为 `matchLevel: none`。

人工验收入口：
1. 打开 `/jobs`。
2. 在“C1 经历匹配与差距诊断”区域点击“运行C1规则匹配”。
3. 点击“运行AI解释”。
4. 检查每条岗位要求的岗位原文、事实依据、匹配等级、风险等级和解释。
5. 对一条非 none 匹配保存人工覆盖并选择正式事实；对一条 none 匹配填写说明后保存。
6. 刷新页面确认匹配和人工覆盖可恢复。
7. 修改母档案版本或候选事实后确认旧匹配显示 stale。

遗留问题：
- C1 已完成但尚未人工验收；C2 必须等待人工确认后再启动。
- 当前 C1 页面默认使用 workspace 中第一份正式母档案和第一份正式岗位，后续 C2/D 可再扩展岗位选择体验。
- 未实现 AI建议、Fact Guard、JobAdaptationDraft、正式 ResumeBranch、正式模板、PDF导出、PDF导入或求职材料生成。
- 未覆盖职业母档案事实层，未写入 API 密钥，未进行批量删除或破坏性 Git 操作。

下一步：
1. 人工验收 C1。
2. 若 C1 验收通过，再单独启动 C2：AI建议与 Fact Guard。
3. C2 启动前继续禁止进入阶段D/E能力。

## 2026-07-01：阶段A.1 阶段A收口与端到端集成修复

本次目标：
- 仅修复阶段A审查发现的集成问题，不进入阶段B业务开发。
- 将页面数据源从直接 demo import 收口到 IndexedDB/Repository，并补齐 AI、缓存、Repository 和日志持久化测试。

修改文件：
- `Plan.md`
- `history.md`
- `src/services/storage/repositories.ts`
- `src/services/workspace/useWorkspace.ts`
- `src/components/workspace/WorkspaceStates.tsx`
- `src/app/HomeWorkspace.tsx`
- `src/app/page.tsx`
- `src/app/profile/ProfileWorkspace.tsx`
- `src/app/profile/page.tsx`
- `src/app/jobs/JobsWorkspace.tsx`
- `src/app/jobs/page.tsx`
- `src/app/resume/ResumeWorkspace.tsx`
- `src/app/resume/page.tsx`
- `src/components/resume/A4ResumeProbe.tsx`
- `src/app/globals.css`
- `src/ai/persistentService.ts`
- `src/ai/providers/demoCacheProvider.ts`
- `src/ai/providers/fallbackProvider.ts`
- `tests/unit/aiService.test.ts`
- `tests/unit/storage.test.ts`

修改内容：
- 将 Sprint 0 状态在修复期间临时调整为 `[~]`，完成验证后恢复为 `[x]`。
- 新增统一客户端 workspace 加载机制：首次加载执行 `ensureDemoWorkspace` / `seedDemoWorkspace`，再通过 `WorkspaceRepository` 从 IndexedDB 读取 profile 和 jobs，并提供 loading、error、empty 状态。
- 首页、职业母档案页、岗位页、简历页均改为使用 Repository 数据，不再直接把 `demoCareerProfile` 或 `demoJobDescriptions` 作为页面主要数据源。
- A4 导出探针优先读取应用 workspace profile；仅在 Repository 数据不可用时使用固定探针数据，并在工具栏明确显示数据来源。
- 为 `WorkspaceRepository` 补充 `ensureDemoWorkspace` 和 `getMeta`，并覆盖 `saveResumeBranch`、`listResumeBranches`、`saveAiLogs`、`saveExportRecord`、`setMeta/getMeta` 测试。
- 新增 `PersistentAiService`，确保实际 AI 调用产生的 logs 可写入 IndexedDB，并出现在 workspace JSON 导出中。
- 新增 `FallbackAiProvider`：主 Provider 失败时尝试 `DemoCacheProvider`，缓存不可用时返回明确失败；补齐降级链测试。
- 调整 `DemoCacheProvider`，支持缓存命中、未命中 fallback、repair 绕过缓存、底层失败四类路径，并补齐测试。
- 增加 `AiService` provider 抛异常测试，验证 `provider_failed` 状态。
- 使用 `CareerProfileSchema` 和 `JobDescriptionSchema` 测试 `profile-builder` 与 `jd-analyzer` 结构化输出。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：3 个测试文件，18 个测试通过。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：A4 PDF 探针渲染、中文文本、A4 比例和无溢出检查通过。

遗留问题：
- A4 探针仍保留固定探针数据作为 Repository 不可用时的 fallback，正式模板和分支业务仍留到后续阶段。
- 阶段B的真实简历文本解析、JD解析界面、经历匹配、建议卡片、岗位分支业务和正式模板均未实现。

下一步：
1. 进入阶段B：职业母档案与 JD 解析。
2. 优先实现粘贴简历文本生成母档案草稿、用户校对、保存母档案。
3. 接入 JD 粘贴与结构化要求输出，继续沿用阶段A的 Schema、AI Service 和 IndexedDB 底座。
4. 仍不提前进入经历匹配、AI建议卡片、岗位分支正式业务和正式模板开发。

## 2026-07-01：完成阶段A / Sprint 0 底座与最小技术验证

本次目标：

- 按 `Plan.md` 的阶段A范围初始化 Web MVP 工程，冻结核心 Schema，准备脱敏演示数据，完成 Mock AI、IndexedDB 和静态 A4 PDF 技术探针。

修改文件：

- 新增工程配置：`package.json`、`pnpm-lock.yaml`、`.gitignore`、`next.config.ts`、`tsconfig.json`、`eslint.config.mjs`、`postcss.config.mjs`、`tailwind.config.ts`、`vitest.config.ts`、`playwright.config.ts`、`next-env.d.ts`。
- 新增源码：`src/app`、`src/domain/schemas`、`src/data`、`src/ai`、`src/services`、`src/components/resume`。
- 新增测试：`tests/setup.ts`、`tests/unit/schema.test.ts`、`tests/unit/storage.test.ts`、`tests/unit/aiService.test.ts`、`tests/e2e/pdfProbe.spec.ts`。
- 新增验证素材：`artifacts/stage-a-a4-probe.png`、`artifacts/stage-a-a4-probe.pdf`。
- 更新 `Plan.md` 与 `history.md`。

修改内容：

- 初始化 Next.js + TypeScript + Tailwind CSS 工程，使用 pnpm 管理依赖。
- 定义 `CareerProfile`、`Experience`、`Evidence`、`JobDescription`、`JobRequirement`、`RequirementMatch`、`ResumeBranch`、`AiSuggestion`、`ResumeRevision`、`ExportRecord` 等 Zod Schema。
- 定义 `FactProvenance` 与 `FactStatement`，要求进入核心事实层的数据必须具备来源、确认状态和风险等级。
- 建立统一 `AiProvider` 接口、`AiService`、`MockAiProvider` 和 `DemoCacheProvider`，支持结构化输出校验、一次修复重试、失败降级和脱敏日志。
- 建立 Dexie/IndexedDB 数据库与 `WorkspaceRepository`，支持示例母档案、示例 JD、AI 日志、导出记录的保存、读取和 workspace JSON 导出。
- 准备脱敏示例母档案和两份示例 JD：数据分析实习、外贸/跨境运营实习。
- 搭建页面骨架：首页/项目空间、职业母档案、岗位工作区、简历工作台、A4 导出探针。
- 使用 HTML 文本渲染静态 A4 简历，并通过浏览器/Playwright 生成 PDF 探针文件。

验证结果：

- `pnpm verify` 通过：类型检查、Lint、Vitest 单元测试、Next 生产构建均通过。
- `pnpm test:e2e` 通过：Playwright 使用系统 Edge 检查 `/export/probe` 的 A4 页面、中文文本、A4 比例和内容无溢出。
- `pdftotext artifacts/stage-a-a4-probe.pdf -` 可提取中文文本，验证 PDF 不是整页图片，文本可复制。
- 本次未修改 PRD、`AGENTS.md`、`Claude.md`，未删除任何文件，未接入真实模型或写入 API 密钥。

遗留问题：

- `pnpm exec playwright install chromium` 两次因下载超时未完成；当前 E2E 和 PDF artifact 使用系统 Microsoft Edge channel `msedge` 完成验证。
- 主模型与备用模型仍待后续阶段确认。
- 阶段A只完成底座和静态探针，尚未实现阶段B的粘贴简历解析和 JD 真实解析流程。

下一步：

1. 进入阶段B：职业母档案与 JD 解析。
2. 优先实现粘贴文本生成母档案草稿、用户校对、保存母档案。
3. 接入 JD 粘贴与结构化要求输出，继续使用阶段A的 Schema、AI Service 和 IndexedDB。
4. 不提前进入阶段C的匹配建议、Fact Guard 交互和阶段D的正式分支模板导出。

## 2026-07-01：根据外部评价收缩比赛执行计划

本次目标：

- 吸收外部评价中对“单人比赛 MVP 执行过重”的提醒，将总计划补充为更短的比赛版执行路线。

修改文件：

- `Plan.md`
- `Claude.md`
- `history.md`

修改内容：

- 在 `Plan.md` 增加 MVP 执行约束，明确粘贴文本优先、Fact Guard 前置、AI Schema 校验、向量数据库后置、复杂分支同步后置、API 密钥保护等规则。
- 在 `Plan.md` 增加五阶段比赛版路线：阶段A底座与技术验证、阶段B母档案与JD解析、阶段C匹配建议与Fact Guard、阶段D分支模板导出、阶段E PDF导入与比赛材料。
- 将 Sprint 0 补强为底座验证阶段，加入 `FactProvenance`、统一 AI Service、Mock/演示缓存 Provider、固定演示数据、IndexedDB/Dexie、A4 PDF 技术探针。
- 将 Sprint 4 的向量匹配改为后置接口，MVP 默认规则筛选、关键词匹配和大模型解释。
- 将 Sprint 6 的复杂分支继承收缩为母档案版本号、更新提示和手动同步。
- 在 `Claude.md` 增加五阶段路线、MVP硬约束、AI调用边界、API密钥保护和文件名大小写约定。

验证结果：

- 本次仅修改协作文档，未删除文件，未修改 PRD 原文。
- 三份文档的下一步方向已统一为进入阶段A / Sprint 0。

遗留问题：

- 主模型与备用模型仍待确认。
- 尚未初始化工程代码。
- PDF 导出技术探针尚未执行。

下一步：

1. 进入阶段A / Sprint 0。
2. 确认或默认采用 Web MVP + Next.js + TypeScript + IndexedDB/Dexie。
3. 初始化工程、核心 Schema、`FactProvenance`、AI Service 接口和演示数据。
4. 完成静态 A4 简历 PDF 导出技术探针。

## 2026-07-01：开发协作文档初始化

修改内容：

- 创建 `Plan.md`，基于 PRD 拆分 MVP 开发路线、Sprint 任务、完成定义和下次开发路线。
- 创建 `history.md`，用于记录后续每次开发结果、验证情况和遗留问题。
- 创建 `Claude.md`，用于约束后续项目开发行为，重点防止乱删乱改、越过计划开发和忘记同步记录。

验证结果：

- 已按 PRD 的核心闭环、MVP 范围、Sprint 拆分、验收标准和单人开发规则整理。
- 本次仅新增文档文件，未修改 PRD 原文，未删除任何文件。

遗留问题：

- 尚未确认实际技术栈、主模型、存储方案和 PDF 导出方案。
- 尚未初始化工程代码。

下次开发路线：

1. 进入 `Plan.md` 的 Sprint 0。
2. 确认 Web/Tauri 取舍和前端技术栈。
3. 初始化工程、Schema、示例数据和页面骨架。
## 2026-07-02：阶段B.3 真实模型联调与阶段B验收收口

本次目标：
- 完成阶段B验收收口：E2E测试、幂等提交、revision冲突、手动降级全覆盖。
- 使用脱敏中文样本真实跑通 profile-builder 和 jd-analyzer。
- 验证模型输出通过 Zod Schema、sourceQuote 可定位、无编造、低置信度状态正确。
- 修正 Plan.md 中 Sprint 1 状态与未完成任务之间的矛盾。

修改文件：
- `Plan.md`
- `history.md`
- `package.json`
- `vitest.ai-real.config.ts`（新增）
- `tests/ai-real/_server-only-mock.ts`（新增）
- `tests/ai-real/stageBRealProvider.test.ts`
- `tests/unit/storage.test.ts`
- `tests/e2e/phaseBFlow.spec.ts`（新增）
- `src/ai/tasks/registry.ts`
- `src/app/api/ai/structured/route.ts`

修改内容：
- 扩充 `tests/ai-real/stageBRealProvider.test.ts`：3 个真实模型测试全部通过。
- 新增 `coerceRawOutput` 层：模型返回字段名不一致时（如 `type`→`category`、`requirement`→`description`、`reason`→`confidenceReason`、`parsedRequirements`→`requirements`、`jobTitle`→`title`），自动映射到 Schema 期望结构，缺失的 `id`/`createdAt`/`updatedAt`/`hardConstraint`/`needsConfirmation` 等自动补全。
- `normalizeOutput` 增加防御性检查：`basics`/`requirements`/`skills`/`certificates` 为空或非数组时不崩溃；`sourceQuote` 为非字符串时跳过定位。
- 服务端路由在 provider 返回后依次执行 `coerceRawOutput` → `normalizeOutput` → Zod 校验。
- 新增 `vitest.ai-real.config.ts`：独立配置，mock `server-only`，自动加载 `.env.local`，测试超时 60 秒。
- 扩充 `tests/unit/storage.test.ts`：新增 5 个测试覆盖 JD 幂等 commit、revision 冲突、刷新恢复、provider 失败降级。
- 新增 `tests/e2e/phaseBFlow.spec.ts`：6 个 Playwright E2E 测试覆盖 Phase B 页面流程。
- 修正 Plan.md：Sprint 1 后置任务标记 `[>]`，阶段B检查点 B3 标记 `[x]`。

实际模型与 Provider：
- Provider: openai-compatible（通过 `https://token-plan-cn.xiaomimimo.com/v1`）
- Model: mimo-v2.5-pro

测试时间：2026-07-02 02:46 UTC

使用脱敏样本：
- Profile Builder：虚构"北京大学""某科技有限公司""校园二手交易平台"，无真实个人信息。
- JD Analyzer：虚构"某互联网公司/数据分析实习生"岗位 JD，无真实公司信息。

Profile Builder 结果：
- 模型返回 experiences + skills + certificates，Schema 一次通过（coerce 后）。
- sourceQuote 全部可在原文中定位。
- 无编造学校/组织/奖项。

JD Analyzer 结果：
- 模型返回 10 条要求，字段名（`type`/`requirement`/`reason`/`parsedRequirements`）与 Schema 不一致，需 coerce。
- coerce 后 Schema 通过，sourceQuote 全部可定位。
- 分类：responsibility 3 条、must_have 5 条、nice_to_have 2 条。
- 无编造要求或技能。

Schema 失败/修复次数：3 次（JD 字段名不一致→新增 coerceRawOutput；title/company 空值→补全 placeholder；skills 非数组→数组检查）。

sourceQuote 定位情况：所有通过 Schema 校验的 sourceQuote 均可在原文中直接或 compact 匹配定位。未定位项自动降级为 `confidenceLevel: low` + `needsConfirmation: true`。

验证命令结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：3 文件 / 24 测试通过。
- `pnpm build` 通过。
- `pnpm test:e2e` 通过：6 测试通过。
- `pnpm test:ai:real` 通过：3 测试通过（health check + profile-builder + jd-analyzer）。

遗留问题：
- 模型返回字段名不稳定，coerce 层需随不同模型迭代维护。
- 阶段B后置能力（经历复制/排序、要求合并、完整富编辑）留作后续增强。

下一步：
1. 进入阶段C：经历匹配、AI建议与 Fact Guard。
2. 不提前实现 PDF 导入、正式模板、PDF 导出或登录能力。

## 2026-07-02：阶段B 职业母档案与 JD 解析工程实现

本次目标：
- 实现阶段B工程链路：粘贴简历/JD -> 保存原文 -> 隐私确认 -> 服务端白名单AI解析或手动降级 -> 草稿确认 -> Dexie事务幂等提交正式数据。
- 严格保留草稿层与正式事实层分离，不进入阶段C的经历匹配、AI建议、Fact Guard建议卡片、岗位分支和正式导出。

修改文件：
- `Plan.md`
- `history.md`
- `.env.example`
- `package.json`
- `src/domain/schemas/importDraft.ts`
- `src/domain/schemas/index.ts`
- `src/domain/schemas/job.ts`
- `src/domain/schemas/ai.ts`
- `src/domain/mappers/profileDraftMapper.ts`
- `src/domain/mappers/jobDraftMapper.ts`
- `src/services/security/text.ts`
- `src/services/storage/db.ts`
- `src/services/storage/repositories.ts`
- `src/ai/service.ts`
- `src/ai/provider.ts`
- `src/ai/client.ts`
- `src/ai/prompts/profileBuilder.ts`
- `src/ai/prompts/jdAnalyzer.ts`
- `src/ai/tasks/registry.ts`
- `src/ai/providers/demoCacheProvider.ts`
- `src/ai/providers/openAiCompatibleProvider.ts`
- `src/app/api/ai/structured/route.ts`
- `src/app/profile/ProfileWorkspace.tsx`
- `src/app/jobs/JobsWorkspace.tsx`
- `src/app/globals.css`
- `tests/unit/aiService.test.ts`
- `tests/unit/storage.test.ts`
- `tests/ai-real/stageBRealProvider.test.ts`

修改内容：
- 新增阶段B草稿 Schema：`RawInputDocument`、`ProfileImportDraft`、`JobAnalysisDraft`、`ProfileBuilderOutput`、`JdAnalyzerOutput`、`DraftCommit`。
- 新增 Mapper，确保 AI 草稿结构只通过独立转换进入现有 `CareerProfile` / `JobDescription`，不建立第二套正式数据模型。
- Dexie 升级到 v2，新增 raw input、profile draft、job draft、draft commit 表；Repository 提供 revision 校验、自动保存、刷新恢复、幂等 commit 和 workspace JSON 导出。
- 新增服务端 AI Route：前端只提交白名单任务名和业务输入；服务端根据任务注册表选择 Prompt、版本和 Schema，并在返回前完成 Zod 校验。
- 真实模型 Provider 使用 `server-only`，环境变量统一为 `AI_PROVIDER`、`AI_API_KEY`、`AI_BASE_URL`、`AI_MODEL`；新增无密钥 `.env.example`。
- 首次外部模型调用前，Profile/JD 页面展示数据发送与隐私说明；用户拒绝则进入手动模式。
- 默认脱敏手机号、邮箱、身份证号、精确地址后再发送外部模型；服务端日志和 AI 日志不保存完整敏感文本。
- DemoCache 改为任务、Prompt 版本、输入哈希精确命中；缓存未命中不再冒用其他演示案例。
- sourceSpan 改为模型返回 sourceQuote、程序定位字符位置；无法定位的内容保留低置信度和待确认。
- Profile 页面实现 B1：原文保存、隐私确认、AI解析/手动模式、事实确认、事务提交正式母档案。
- Jobs 页面实现 B2：原始JD保存、隐私确认、AI解析/手动模式、要求确认、删除影响提示、事务提交正式岗位数据。
- AI 日志改为元数据记录：任务、Provider、模型、Prompt版本、输入哈希、长度、延迟、状态和错误码。
- 新增 `pnpm test:ai:real` 本地可选命令，不加入 `pnpm verify`。

验证结果：
- `pnpm typecheck` 通过。
- `pnpm lint` 通过。
- `pnpm test` 通过：3 个测试文件，19 个测试通过。
- `pnpm build` 通过，包含 `/api/ai/structured` 动态路由构建。

遗留问题：
- 真实外部模型尚未联调：当前环境未配置 `AI_API_KEY` / `AI_MODEL`，阶段B工程实现完成，但真实模型验收必须在 `Plan.md` 中保持 `[!]`，进入阶段C前不得忽略。
- 阶段B仅实现确认/删除闭环；经历复制/排序、要求合并、完整富编辑体验留作后续增强。
- PDF导入、经历匹配、AI建议、Fact Guard建议卡片、岗位分支、正式模板和PDF导出仍未进入本阶段。

下一步：
1. 本地配置真实模型环境变量后，使用脱敏测试样本运行 `pnpm test:ai:real`。
2. 人工在页面分别跑通一次 Profile Builder 和 JD Analyzer，并记录模型、时间、结果和错误码状态。
3. 真实模型联调完成后，再进入阶段C：经历匹配、AI建议与 Fact Guard。
