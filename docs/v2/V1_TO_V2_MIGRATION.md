# V1 To V2 Migration

## 当前实际基线

- Dexie实际版本：v7。
- 当前表：`profiles`、`jobDescriptions`、`rawInputs`、`pdfImportSessions`、`pdfPageTexts`、`profileImportDrafts`、`jobAnalysisDrafts`、`draftCommits`、`requirementMatches`、`matchOperations`、`jobAdaptationDrafts`、`aiSuggestions`、`adaptationSnapshots`、`suggestionOperations`、`resumeBranches`、`resumeRevisions`、`resumeBranchOperations`、`aiLogs`、`exportRecords`、`appMeta`。
- V2预计新增：`resumeDocuments`、`resumeDocumentRevisions`、`presentationRevisions`、`templateDefinitions`或静态注册表、`layoutDiagnostics`、后续 `applications`。

## 保留策略

- `CareerProfile`：继续作为事实层，不迁移为模板内容。
- `JobDescription`：继续作为岗位要求来源。
- `ResumeBranch`：继续作为岗位版本实体；V2通过Mapper生成或持久化 `ResumeDocument`。
- `legacy_unverified`：继续只读，不进入正式编辑和导出。
- `ResumeRevision`：保留，作为V1分支历史。
- `ExportRecord`：保留，V2扩展导出状态时保持兼容。
- `PdfImportSession` 和 `PdfPageText`：保留，继续支持sourceQuote追踪。
- `AiLog`：保留元数据，可在未来做脱敏裁剪，不保存完整文本。

## 迁移原则

- 不删除旧数据。
- 不伪造缺失关联。
- 迁移幂等，以 `operationId` 或迁移版本防重复。
- 迁移前允许导出 JSON 备份。
- 迁移失败回滚，V1主链路仍能运行。
- 内容Revision与样式Revision分离。
- 模板偏好迁移到展示配置，不写内容历史。

## V1任务迁移审计表

| 原文档位置 | 原状态 | 仓库实际状态 | 分类 | 迁移目标 | 优先级 | 理由 |
|---|---|---|---|---|---|---|
| 阶段E-E1.1 | `[x]` | `history.md`记录已通过全量回归 | v1_completed | 不迁移 | P0 | E1独立验收已完成 |
| Sprint 1/2 | `[x]` | 文本和文本型PDF导入已实现 | v1_completed | 复用到G0/G4 | P0 | V2需要导入到ResumeDocument |
| Sprint 3 | `[x]` | JD解析已实现 | v1_completed | 复用到G5 | P0 | 岗位建议继续依赖 |
| Sprint 4 | `[x]` | Evidence Matcher和C1 eval已实现 | v1_completed | 复用到G5 | P0 | 不重复实现匹配 |
| Sprint 5 | `[x]` | AI建议和Fact Guard已实现 | v1_completed | 复用到G0/G5 | P0 | 编辑正文仍需安全门 |
| Sprint 6/D1 | `[~]` | Repository和页面已支持分支、撤销、恢复 | v1_completed | 状态补记 | P0 | 旧状态过期，应冻结不再误判 |
| Sprint 7/D2 | `[ ]` | 双模板预览和PDF导出已实现 | v1_completed | 状态补记 | P0 | 旧状态与实现冲突 |
| package.json | `[x]` | 已有 `pnpm verify` | v1_completed | 保留 | P0 | 可作为基础质量门 |
| V1收尾 | `[ ]` | 无一键启动说明 | v1_closeout | README或脚本 | P0 | V2编码前需要可复现启动 |
| V1收尾 | `[ ]` | 当前Git工作树非干净 | v1_closeout | 建立稳定检查点 | P0 | V2迁移前需要基线 |
| Sprint 8 | `[ ]` | Demo运行手册未完成 | v1_closeout | docs/demo或README补充 | P1 | 属比赛交付 |
| Sprint 8 | `[ ]` | 截图、视频、PPT未完成 | v1_closeout | 比赛材料 | P1 | 不阻塞V2规划 |
| Sprint 8 | `[ ]` | 模型和开源组件说明未完成 | v1_closeout | 交付说明 | P1 | 对比赛和合规有价值 |
| Sprint状态 | 冲突 | Sprint 6/7状态落后于实现 | v1_closeout | V1补记 | P1 | 防止重复迁移已完成事项 |
| 测试产物 | 未定义 | 本地E1黑盒用例和生成fixture未跟踪 | v1_closeout | `.gitignore`本地化 | P2 | 符合“测试用例不用git” |
| D1归档UI | `[ ]` | Repository有archive入口，UI未完成 | v2_migrate | G6或分支管理 | P2 | 不是Resume Studio首要能力 |
| 多Profile选择 | `[ ]` | 页面隐式 `profiles[0]` | v2_migrate | G6显式上下文 | P2 | 真实多档案前必须处理 |
| PDF直接下载 | `[ ]` | 当前浏览器打印 | v2_migrate | G3导出增强 | P1 | 用户体验关键 |
| 大量模板 | `[>]` | 仅2套模板 | v2_migrate | G2正式模板中心 | P0/P1 | 第一批4-6套，不一次十几套 |
| 预览区直接编辑 | `[>]` | 未实现 | v2_migrate | G0/G1 | P0 | V2首个可见价值 |
| 拖动排序 | `[>]` | 完整拖拽未实现 | v2_migrate | G1 | P1 | 结构化拖拽，不自由画布 |
| 样式属性面板 | `[>]` | 未实现 | v2_migrate | G1 | P1 | 样式配置不能污染事实 |
| 排版诊断 | `[ ]` | 仅overflow检测 | v2_migrate | G5 | P1 | 增强导出稳定 |
| Workspace正式实体 | `[ ]` | 有useWorkspace，无正式聚合 | v2_migrate | G6 | P2 | 与多Profile/Application配套 |
| Application实体 | `[ ]` | 不存在 | v2_migrate | G6 | P2 | 后续求职管理，不抢主线 |
| 模型路由和备用模型 | `[ ]` | 有Fallback/DemoCache，主备未产品化 | v2_migrate | G5/G8 | P2 | 不阻塞G0 |
| 页面profiles[0] | 未记录 | 多页面存在隐式首Profile | v2_migrate | G6或局部先修 | P2 | 防止数据串绑 |
| ResumeDocument | 未实现 | 当前只有RenderModel/Branch | v2_migrate | G0 | P0 | V2编辑核心 |
| DOCX导入导出 | `[>]` | 未实现 | defer | G4/G3后续 | P2 | 不进首个Goal |
| OCR | `[>]` | 未实现 | defer | G8 | P3 | 成本高，非P0 |
| 浏览器扩展 | `[>]` | 未实现 | defer | G8 | P3 | 需平台合规评估 |
| 云同步 | `[>]` | 未实现 | defer | G8 | P3 | 本地优先先稳定 |
| 向量检索 | `[>]` | 未建设 | defer | 长期评估 | P3 | 当前规则+LLM解释可用 |
| 模板市场 | `[>]` | 未实现 | defer | G8 | P3 | 正式模板中心稳定后再说 |
| 中英文双语/两页高级模板 | `[>]` | 未实现 | defer | G2/G3后续 | P2 | 先做核心正式模板 |
| Plan.md作为活动计划 | 旧规则 | 已冻结为V1档案 | drop | 使用plan2.md | P0 | V2不再向旧计划膨胀 |
| A4 probe作为正式路线 | 旧探针 | 仅技术探针 | drop | 使用Resume Studio导出 | P2 | 已被正式模板/导出取代 |
| 自动批量投递 | `[>]` | 未实现 | do_not_build | 不做 | - | 合规风险 |
| 验证码/风控绕过 | 禁止 | 未实现 | do_not_build | 不做 | - | 明确违规 |
| 无证据事实生成 | 禁止 | Fact Guard已阻断 | do_not_build | 不做 | - | 破坏核心信任 |
| 第一阶段任意PDF一比一自由画布 | 未实现 | 未实现 | do_not_build | 不做 | - | 成本高且破坏结构化事实 |

## 迁移统计

- v1_completed：8
- v1_closeout：7
- v2_migrate：13
- defer：7
- drop：2
- do_not_build：4

## 首次V2迁移停止条件

- V1启动说明和稳定检查点未确认时，不执行Dexie迁移。
- C1/C2 eval失败时停止。
- 迁移后V1主链路不可运行时回滚。
- 出现未确认事实进入正式材料时停止。
