# 职适AI开发计划

依据：`职适AI_产品需求文档_PRD_v1.0.md`

当前目标：先完成“导入/粘贴 -> 职业母档案 -> JD解析 -> 经历匹配 -> AI建议 -> 岗位分支 -> 模板预览 -> PDF导出”的 MVP 纵向闭环。任何不能直接增强这条闭环的功能先后置。

## 状态标记

- `[ ]` 未开始
- `[~]` 进行中
- `[x]` 已完成
- `[!]` 阻塞，需要决策或外部条件
- `[>]` 后置，不影响当前 MVP 闭环

每次开发结束必须做三件事：

1. 在本文件更新对应任务状态。
2. 在 `history.md` 记录本次修改内容、验证结果、遗留问题。
3. 更新本文末尾“下次开发路线”。

## 当前进度

- [x] 项目开发协作文档初始化：创建 `Plan.md`、`history.md`、`Claude.md`。
- [x] Sprint 0：工程初始化、数据 Schema、示例数据、页面骨架。
- [x] 阶段A.1：阶段A收口与端到端集成修复。
- [x] 阶段B：职业母档案与 JD 解析工程实现。
- [x] 阶段B验收收口：真实模型联调通过（mimo-v2.5-pro via openai-compatible）、E2E 覆盖页面流程、幂等提交、revision 冲突、手动降级；coerce 层处理模型字段名差异；所有验证命令通过。
- [x] 阶段B.3真实模型联调完成：3/3 项测试通过（health check、profile-builder、jd-analyzer），Provider mimo-v2.5-pro，Schema coerce 层处理字段名差异。
- [x] 阶段C-C1：Evidence Matcher 与差距诊断完成；规则匹配、AI解释、人工覆盖、stale 判定、Dexie v3 持久化和迁移测试已覆盖。
- [x] 阶段C-C2：AI建议与 Fact Guard 完成；JobAdaptationDraft、resume-tailor、规则 Fact Guard、AI fact-guard、单条接受/拒绝/编辑/重新检测/撤销、Dexie v4 事务与迁移测试已覆盖。
- [x] 阶段D-D1：正式 ResumeBranch、版本历史与多岗位分支完成；未 stale 的 JobAdaptationDraft 可创建 verified 分支并同事务创建首个 ResumeRevision，支持两个岗位分支独立编辑、版本历史、恢复/撤销、syncStatus 更新提示和 legacy_unverified 只读迁移；模板、PDF 预览/导出、PDF 导入、求职材料、登录/云同步仍未开始。

## MVP交付标准

- 用户可以通过粘贴文本或文本型 PDF 生成可编辑职业母档案。
- 用户可以粘贴岗位 JD，得到职责、硬性条件、技能、优先级等结构化结果。
- 每条核心岗位要求可以关联真实经历，或明确显示“当前无证据”。
- AI 建议卡片必须显示原文、建议、原因、岗位依据、事实风险。
- 用户可以接受、拒绝、编辑建议，并可撤销。
- 同一母档案至少能保存 2 个不同岗位分支。
- 至少 2 套一页式模板可切换，内容不丢失。
- PDF 导出结果与预览基本一致，文本可复制。
- 未经用户确认的新事实不得进入最终导出。

## MVP执行约束

1. 每次开发必须保证已有纵向闭环仍可运行，不得为了新模块破坏已有演示路径。
2. 先支持粘贴文本，PDF 导入不得阻塞职业母档案、JD 分析和岗位适配闭环。
3. 所有进入最终简历的事实必须记录来源、确认状态和风险状态。
4. AI 输出必须经过 Schema 校验，不允许将自由文本直接写入核心数据。
5. AI 输出连续两次校验失败时，进入手动编辑或演示缓存降级，不得无限重试。
6. MVP 不建设向量数据库，先使用规则筛选、关键词匹配和大模型解释。
7. MVP 不实现母档案与岗位分支的复杂自动合并，只提供更新提示和手动同步。
8. MVP 只开发两套模板，未完成一页导出前不得新增模板。
9. API 密钥只能存在于服务端或本地环境变量中，不得写入前端代码或仓库。
10. 每个阶段完成后必须产出：可运行页面、验证记录、至少一张比赛素材截图。
11. 开发中统一文件名大小写，本文档统一使用 `Plan.md`、`history.md`、`Claude.md`。
12. 新功能若不能直接增强核心 Demo，默认进入后置功能池。

## 比赛版五阶段执行路线

原 Sprint 0-8 保留为详细任务池；真实开发按以下五个阶段推进，避免长时间只做底层、看不到完整产品。

### 阶段A：底座与最小技术验证

目标：工程可运行，Schema 冻结，示例母档案可显示，IndexedDB 可读写，静态 A4 简历可导出 PDF，AI 结构化接口可返回合法 JSON。

对应任务：Sprint 0。

完成后才能进入阶段B。

### 阶段B：职业母档案与JD解析

目标：打通“粘贴简历文本 -> 生成母档案草稿 -> 用户校对 -> 保存母档案 -> 粘贴 JD -> 生成岗位要求 -> 用户校对”。

对应任务：Sprint 1、Sprint 3 的核心路径。PDF 导入暂不阻塞。

内部检查点：

- [x] B1 职业母档案链路：原文保存、隐私确认、服务端白名单AI解析/手动降级、草稿确认、Dexie事务幂等提交、刷新恢复。
- [x] B2 岗位JD链路：原文保存、隐私确认、服务端白名单AI解析/手动降级、要求确认/删除提示、Dexie事务幂等提交、刷新恢复。
- [x] B3 真实模型联调与验收收口：profile-builder 和 jd-analyzer 均通过真实模型测试；E2E 覆盖页面流程；单元测试覆盖幂等提交、revision 冲突、手动降级、刷新恢复；`pnpm typecheck/lint/test/build/test:e2e/test:ai:real` 全部通过。

### 阶段C：经历匹配、AI建议与Fact Guard

目标：打通“岗位要求 -> 匹配用户经历 -> 显示满足/部分满足/无证据 -> 生成可解释建议 -> 检测新增事实 -> 用户接受或拒绝”。

对应任务：Sprint 4、Sprint 5。向量检索后置，MVP 默认规则筛选 + 关键词匹配 + 大模型解释。

内部检查点：

- [x] C1 Evidence Matcher 与差距诊断：仅使用正式 CareerProfile 中已确认事实；匹配等级与风险等级分离；规则评估、AI评估、人工覆盖分层保存；统一通过 `resolveEffectiveMatch` 计算有效结果；旧匹配可按 profileVersion、jobVersion、matcherVersion、candidateSetHash 判定 stale。
- [x] C1 AI辅助自动验收：15个脱敏验收案例覆盖strong/weak/transferable/none/团队风险/硬约束缺口/未确认排除/白名单外ID/stale/Provider失败/Prompt注入；确定性硬校验（ID白名单、事实确认、no-evidence、stale、resolve一致性、禁止总分、禁止新增事实、风险约束）；独立AI语义Judge（c1-evaluator，独立prompt，不修改结果）；`pnpm test:c1:eval` 输出 `artifacts/c1-evaluation.json` 和 `artifacts/c1-evaluation.md`；AI辅助验收不替代人工验收。
- [x] C1.1 AI验收校准与Matcher质量收口：收紧规则Matcher（参与/协助等限定词降级、团队上下文检测、独立性不匹配检测）；改进匹配解释结构（[支持]/[缺失]/[判定]/[风险]）；增加expectedDisposition区分合法/非法案例；Prompt注入区分inputContainsInjection与modelFollowedInjection并清理注入文本；Judge一致性校验（criticalFailures/score阈值）+一次重试；新报告统计positiveCasesPassed/negativeCasesCorrectlyRejected/hardSafetyFailures/semanticCasesPassed/judgeInvalid/overallQualified；所有安全断言通过。
- [x] C2 AI建议与 Fact Guard：只读取 `resolveEffectiveMatch` 得到且实时未 stale 的匹配；创建 `JobAdaptationDraft` 而非正式 `ResumeBranch`；resume-tailor 只使用 `usedEvidenceRefs` 中的已确认事实；规则 Fact Guard 先执行，再调用 AI fact-guard 复核；支持单条接受、拒绝、编辑后重检、重新检测和撤销，全部经 Dexie 事务和 `expectedRevision`/`operationId` 保护。
- [x] C2.1 AI建议与 Fact Guard 验收收口：16个脱敏验收案例覆盖合法措辞优化、合法删减、合法排序、新增数字、新增工具/技能、参与变主导、协助变独立、了解变熟练、团队成果变个人、stale阻断、Prompt注入、Provider失败降级、合法新增（证据存在）、编辑后重检、建议范围隔离、复合风险；确定性硬校验（usedEvidenceRefs白名单、事实确认状态、新增实体/数字/技能检测、表达强度升级检测、ownership风险、blocked_high_risk不可接受、stale状态、Provider失败降级）；独立AI语义Judge（c2-judge.v2，只评价Fact Guard和建议安全性，不修改建议）；工作流验证（accept/reject/edit-recheck/revoke、expectedRevision/operationId幂等、scope隔离：建议只修改JobAdaptationDraft不修改CareerProfile不创建ResumeBranch）；`pnpm test:c2:eval` 输出 `artifacts/c2-evaluation.json` 和 `artifacts/c2-evaluation.md`；不加入 `pnpm verify`；AI辅助验收不替代人工验收。

### 阶段D：岗位分支、模板和导出

目标：创建两个岗位分支，保存不同内容选择和排序，切换两套模板，一页预览，并导出两份 PDF。

对应任务：Sprint 6、Sprint 7。分支同步只做更新提示和手动同步。

内部检查点：

- [x] D1 正式 ResumeBranch、版本历史与多岗位分支：只允许未 stale、非 error 的 `JobAdaptationDraft` 创建 verified 分支；分支只持久化正式事实引用 `factRefs`，不复制 CareerProfile 正式事实层；创建分支与首个 `ResumeRevision` 在同一 Dexie v5 事务中完成；写操作使用 `expectedRevision` + `operationId` 幂等保护；手动文本编辑由 Repository 基于正式 factRefs 重新运行规则 Fact Guard；`ai_failed_rule_kept` 在规则通过且无 high finding 时以 `rule_only_verified` 进入分支并提示未完成 AI 复核；旧占位分支迁移为 `legacy_unverified` 只读保留；恢复/撤销通过不可变追加 revision 链完成，`syncStatusCache` 为派生缓存且不进入 snapshot。
- [ ] D2 模板预览与 PDF 导出：尚未开始。

### 阶段E：PDF导入、稳定性和比赛材料

目标：补齐文本型 PDF 导入、AI 缓存、异常降级、截图、演示视频、模型与开源组件说明、产品说明书和 PPT。

对应任务：Sprint 2、Sprint 8。

## 补充量化验收指标

- 事实安全：未经用户确认的新增数字、奖项、组织、工具和成果进入最终导出的数量为 0。
- 完整闭环时间：使用预设演示数据，从导入经历到生成第一份岗位简历不超过 5 分钟。
- 多岗位差异性：同一母档案生成的两份岗位简历，在经历排序、核心关键词或内容选择上至少存在 3 处可解释差异。
- 导出可靠性：两套模板均能输出单页、中文正常、文本可复制的 PDF。

## Sprint 0：工程与数据底座

状态：`[x]`

目标：让项目能运行，核心数据结构能序列化，示例档案能在页面中显示，并完成 AI、事实来源和 PDF 导出的最小技术验证。

任务清单：

- [x] 确认应用形态：优先 Web MVP；Tauri、本地优先能力后置评估。
- [x] 确认技术栈：建议 React/Next.js 或 Vite React + TypeScript；本地存储优先。
- [x] 建立目录结构：`components`、`domain`、`ai`、`services`、`tests`。
- [x] 定义核心 Schema：`CareerProfile`、`Experience`、`Evidence`、`JobDescription`、`JobRequirement`、`RequirementMatch`、`ResumeBranch`、`AiSuggestion`、`ResumeRevision`。
- [x] 定义事实来源结构 `FactProvenance`：`sourceType`、`sourceId`、`sourceText`、`confidence`、`confirmedByUser`、`riskLevel`、`createdAt`。
- [x] 明确所有可能进入最终简历的事实必须具有来源、确认状态和风险状态。
- [x] 建立统一 AI Service：`AiProvider`、结构化输出、Prompt 版本、Schema 校验、一次自动修复/重试、失败降级、请求日志、演示缓存。
- [x] 建立 Mock AI Provider 和演示缓存 Provider，保证无 API 时可演示主链路。
- [x] 准备脱敏示例数据：1 份学生母档案、2 份岗位 JD。
- [x] 示例数据从第一天固定为比赛案例：数据分析实习 JD、外贸/跨境运营实习 JD。
- [x] 搭建页面骨架：首页/项目空间、职业母档案、岗位工作区、简历工作台、模板导出。
- [x] 建立基础持久化接口：保存、读取、更新、导出 JSON；MVP 默认使用 IndexedDB/Dexie。
- [x] 完成最小 PDF 技术探针：静态示例数据 -> A4 HTML 简历 -> 浏览器打印导出 PDF，验证中文字体、单页尺寸和文本可复制。
- [x] 建立最小验证：Schema 校验、示例数据加载、空状态显示。
- [x] 阶段A.1收口：页面统一从 IndexedDB/Repository 读取 workspace 数据，首次加载 seed 演示 workspace，提供 loading/error/empty 状态。
- [x] 阶段A.1收口：A4 导出探针优先读取应用 workspace 数据，并明确区分固定探针数据与应用 workspace 数据。
- [x] 阶段A.1收口：补齐 provider 异常、Schema 结构化输出、DemoCache、FallbackAiProvider、Repository 写入和 AI 日志持久化测试。

完成定义：

- 项目可本地启动。
- 示例母档案可显示。
- 核心实体可被创建、序列化、反序列化。
- 关键页面入口存在，但业务功能可以先是占位。
- AI 结构化调用可通过 Mock 或演示缓存返回合法 JSON。
- 静态示例简历可导出单页、中文正常、文本可复制的 PDF。

## Sprint 1：职业母档案编辑器与粘贴导入

状态：`[x]`

目标：用户可以从粘贴文本或空白表单创建、编辑、保存完整职业母档案。

任务清单：

- [x] 实现粘贴文本导入入口。
- [x] 建立手动分类/半自动分类流程，先不依赖 PDF。
- [x] 实现基本信息、教育、实习、项目、竞赛/校园、技能、证书/作品草稿确认路径。
- [>] 每条经历区分”事实原文”和”简历表达稿”；阶段B草稿确认已包含事实原文与 sourceQuote，完整表达稿编辑后置到阶段C/D前收口。
- [>] 经历复制、排序和完整富编辑后置到阶段C/D前收口；阶段B保留草稿确认。
- [x] 支持本地持久化和自动保存。
- [x] 添加低置信度字段提示和用户确认状态。

完成定义：

- 可创建、编辑、保存、重新打开完整职业母档案。
- 用户输入不会因为刷新或 AI 请求失败而丢失。
- 不制造虚假评分，不诱导用户编造经历。

## Sprint 2：PDF文本提取与 Profile Builder

状态：`[ ]`

目标：支持文本型 PDF 导入，并生成可校对的结构化 JSON。

任务清单：

- [ ] 实现文件类型、大小、加密状态校验。
- [ ] 提取文本型 PDF 的正文和基础段落信息。
- [ ] 接入或模拟 Profile Builder，将简历文本转为职业母档案草稿。
- [ ] 使用 Schema 校验模型输出。
- [ ] 输出不合规时自动重试一次；失败则进入手动分类。
- [ ] 解析确认页展示原文对照、字段置信度和未分类内容。
- [ ] 扫描件/OCR 提示后置，不作为 MVP 阻塞。

完成定义：

- 常见文本型 PDF 可以生成可校对 JSON。
- PDF 解析失败时可降级为粘贴文本。
- 用户确认前不写入正式母档案事实层。

## Sprint 3：JD Analyzer 与岗位工作区

状态：`[x]`

目标：用户粘贴 JD 后，系统能输出结构化岗位要求，并允许人工编辑。

任务清单：

- [x] 实现 JD 粘贴和岗位名称/公司手动填写。
- [x] 定义 `JobRequirement` 分类：职责、硬性条件、技能、软能力、加分项、风险/不确定。
- [x] 解析岗位职责、硬性条件、技能、优先级、原文位置。
- [x] 支持低置信度和不确定要求标记。
- [x] 支持用户确认和删除要求条目，删除前提示影响。
- [x] 保存 JD 原文和解析结果。
- [>] 要求合并与完整行内富编辑后续增强；阶段B先完成确认/删除/提交闭环。

完成定义：

- JD 可结构化展示并人工校正。
- 每条要求保留原文依据。
- 不对岗位匹配输出虚假精确总分。

## Sprint 4：Evidence Matcher 与差距诊断

状态：`[x]`

目标：将岗位要求与真实经历建立证据映射，显示满足、部分满足、无证据和风险。

任务清单：

- [x] 定义匹配等级和风险等级：`matchLevel` 为 strong/weak/transferable/none，`riskLevel` 为 low/medium/high，风险用 `MatchRisk[]` 独立记录。
- [x] 实现基础规则匹配：技能、关键词、经历类型、岗位方向；只召回正式母档案中已确认事实。
- [>] 预留可替换的语义检索接口；MVP 不建设向量数据库，默认使用规则筛选、关键词匹配和大模型解释。
- [x] 生成匹配解释：岗位依据、经历事实依据、证据片段。
- [x] 标记差距、风险和无证据状态。
- [x] 保存 `RequirementMatch`，并保存规则评估、AI评估、人工覆盖和操作日志。
- [x] 实现 stale 判定：基于 profileVersion、jobVersion、matcherVersion、规范化 candidateSetHash 实时计算。

完成定义：

- 每条核心要求显示对应经历或“当前无证据”。
- 匹配结果可解释、可追溯。
- 不把团队成果自动归为用户个人成果。

## Sprint 5：Resume Tailor、AI建议卡片与 Fact Guard

状态：`[x]`

目标：生成可解释修改建议，用户可以接受、拒绝、编辑，新增事实会被拦截。

任务清单：

- [x] 定义建议类型：`rewrite`、`remove_or_shorten`、`reorder`、`risk_warning`、`follow_up_question`。
- [x] 实现建议卡片字段：原文、建议、修改原因、岗位依据、事实依据、Fact Guard 结果、风险等级、状态、编辑文本。
- [x] 实现单条接受、拒绝、编辑后重新检测、重新检测和撤销；不实现批量接受。
- [x] 实现 Fact Guard：新增数字、学校/组织/公司/岗位、工具/技能、奖项/证书/成果、参与到负责、协助到独立、了解到熟练/精通、团队成果到个人成果自动标记。
- [x] 高风险或 `blocked_high_risk` 建议禁止接受。
- [x] 所有建议只作用于 `JobAdaptationDraft`，不直接覆盖职业母档案正式事实。

完成定义：

- 至少生成 3 类建议。
- 每条建议有原因、依据和风险标记。
- 未确认新事实不会进入正式简历预览或导出。
- stale 匹配禁止生成或应用建议，必须返回 C1 重跑。
- 建议状态更新、草稿文本修改和快照保存使用 Dexie 事务，重复 `operationId` 不重复应用。

## Sprint 6：岗位分支、版本历史与撤销

状态：`[~]`

目标：同一职业母档案可以保存多个岗位分支，每个分支独立维护版本。

任务清单：

- [x] 实现正式 `ResumeBranch` 创建与命名；旧占位分支迁移为 `legacy_unverified` 只读保留。
- [x] 分支引用母档案 Experience/Fact/Skill/Certificate ID，不复制正式事实层。
- [x] 分支保存岗位专属内容选择、排序、显示状态和表达文本；模板配置未进入 D1。
- [x] 创建分支时记录母档案版本号、岗位版本、源 `JobAdaptationDraft` 和草稿 revision。
- [x] 实现 `ResumeRevision` 快照，snapshot 只保存可恢复业务内容，不保存版本控制元数据或 `syncStatusCache`。
- [x] 实现撤销/恢复；除首个创建 revision 外均记录 `previousRevisionId`，恢复旧版本只追加一个 restore revision。
- [x] 母档案事实或岗位更新后只刷新 `syncStatusCache` 并提示，不自动覆盖分支内容。
- [ ] 删除前影响提示与删除/归档 UI 后续收口；D1 已提供 Repository 归档入口但未实现批量删除或删除 UI。
- [>] 字段级自动合并、复杂冲突检测、双向同步后置。

完成定义：

- 至少 2 个不同岗位分支可独立保存。
- 接受/拒绝/编辑建议后可撤销。
- 母档案和分支职责清晰，不互相静默覆盖。

## Sprint 7：模板预览与 PDF 导出

状态：`[ ]`

目标：提供两套一页式中文简历模板，支持实时预览和文本可复制 PDF 导出。

任务清单：

- [ ] 建立模板渲染数据接口。
- [ ] 实现模板 A：稳重清晰，适合数据/技术/研究类岗位。
- [ ] 实现模板 B：简洁表达，适合运营/产品/综合类岗位。
- [ ] 实现实时预览和模板切换。
- [ ] 实现分页检查、内容溢出提示和删减建议。
- [ ] 实现 PDF 导出，保留当前版本和导出记录。
- [ ] 导出失败时保留状态并展示原因。

完成定义：

- 两套模板切换时内容不丢失。
- 成功导出文本可复制 PDF。
- 预览与导出结果基本一致。

## Sprint 8：求职材料、演示数据与稳定性收口

状态：`[ ]`

目标：完成比赛演示闭环，准备可录屏、可复现、可解释的提交版本。

任务清单：

- [ ] 生成自我介绍、打招呼文案、面试问题预测。
- [ ] 准备演示模式：脱敏母档案、数据分析 JD、外贸运营 JD。
- [ ] 缓存演示链路关键 AI 输出，支持离线降级。
- [ ] 按 MVP 验收表逐项测试。
- [ ] 记录关键截图：导入、JD解析、匹配、建议、Fact Guard、分支对比、模板、PDF。
- [ ] 完成 3-4 分钟 Demo 脚本和录屏。
- [ ] 整理模型、开源组件、第三方 API 来源清单。
- [ ] 完成隐私说明和风险说明。

完成定义：

- 能稳定演示完整 MVP 闭环。
- 能导出两份岗位差异明显的 PDF。
- Demo 可在网络异常时通过示例数据和缓存结果继续展示。

## 后置功能池

状态：`[>]`

- [>] 扫描件 OCR。
- [>] Word/DOCX 导入与导出。
- [>] 英文简历和多语言。
- [>] 更多模板市场。
- [>] 投递看板深度能力。
- [>] 浏览器扩展。
- [>] 高校端、企业端、完整 ATS。
- [>] 订阅、点数、支付。
- [>] 自动批量投递相关能力：原则上不做，不绕过平台规则。

## 技术与产品决策记录

- [x] 首版应用形态：默认 Web MVP；Tauri 不进入当前 MVP。
- [x] 推荐技术栈：Next.js + TypeScript + Tailwind CSS + Zod + Zustand + Dexie/IndexedDB。
- [x] 数据库存储：MVP 默认 IndexedDB，使用 Dexie 封装；暂不引入账号、云同步和 PostgreSQL。
- [x] PDF 导出方案：默认 A4 打印 CSS + 浏览器打印导出 PDF；Puppeteer/Playwright 服务端导出后置。
- [x] 模型方案：阶段A 使用 `AiProvider`、Mock 实现和演示缓存实现，暂不接真实模型。
- [>] Embedding/向量数据库：当前后置。
- [ ] 主模型与备用模型：待确认。
- [x] API 密钥管理：阶段A 未接入真实 API，未向前端代码或仓库写入任何密钥。
- [x] 是否开放注册：阶段A 使用本地匿名体验，不开放注册。
- [>] DOCX 是否进入 P0：当前后置。

## 下次开发路线

阶段D-D1 已完成，等待人工验收后再决定是否进入 D2/Sprint 7。模板、PDF 预览/导出、PDF 导入、求职材料、登录/云同步均未开始。

1. 人工验收 D1：在 `/jobs` 为两个不同岗位分别运行 C1/C2，在 `/resume` 从 C2 草稿创建两个正式分支，验证独立编辑、版本历史、恢复、撤销、syncStatus 提示和 `legacy_unverified` 只读行为。
2. 复核 `artifacts/c1-evaluation.md` 与 `artifacts/c2-evaluation.md`，确认 C1/C2 回归仍通过，C2 指标保持 safeAllowed 6 / safeBlocked 0 / unsafeBlocked 10 / unsafeAllowed 0。
3. D1 人工确认通过后，再单独启动 D2/Sprint 7：模板预览与 PDF 导出。
4. 进入 D2 前仍不实现 PDF 导入、求职材料、登录/云端能力，不写入 API 密钥，不覆盖 CareerProfile 正式事实层。
