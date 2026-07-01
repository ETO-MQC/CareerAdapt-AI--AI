# 开发历史

本文件记录每次开发完成后的实际修改、验证结果、遗留问题和下一步路线。每次修改代码或文档后，都要同步更新 `Plan.md` 的状态。

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
