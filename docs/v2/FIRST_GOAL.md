# First Goal: V2-G0a Resume Studio最小垂直切片

## 1. 当前问题

V1可以生成 verified ResumeBranch 并预览/导出，但用户不能在简历预览区自然地选择和直接编辑正文。已有编辑更偏工作流和表单，模板与内容还没有形成真正的Resume Studio体验。

## 2. 目标

在现有 `/resume` 工作台增加最小“编辑模式”：从 verified ResumeBranch 生成最小 ResumeDocument 或等价编辑模型，用户可以选中一个内容区块、双击编辑合法文本、保存后实时更新预览，并支持撤销最近一次内容修改。

## 3. 非目标

- 不实现完整拖拽。
- 不实现右侧全部样式设置。
- 不新增大量模板。
- 不实现DOCX。
- 不实现OCR。
- 不实现多Profile。
- 不实现Application。
- 不实现自由画布。
- 不执行任意PDF原版式还原。

## 4. 前置条件

- V1 E1验收通过。
- V1主链路可启动。
- 当前Git工作树建立稳定检查点或明确隔离未提交V1收尾。
- `unsafeAllowed=0`。
- `docs/MVP_V1_HANDOFF.md` 已确认。

## 4.1 G0a硬约束

- G0a不持久化ResumeDocument，不新增Dexie表，不升级Dexie v8。ResumeDocument仅作为由当前ResumeBranch和currentRevision派生的编辑视图模型。
- ResumeBranch/ResumeRevision继续作为唯一内容事实来源；不建立第二套内容Revision系统。
- 所有文本保存继续复用现有 `editResumeBranch`、`expectedRevision`、`operationId`、事务和Fact Guard路径。
- Mapper映射全部contentItem，不直接丢弃隐藏或暂不可渲染的内容项；每个block明确 `visible`、`renderable`、`editable` 和 `guardStatus`。
- 正式模板只渲染 `visible && renderable` 的内容。
- 模板A和模板B必须使用统一稳定内容标识：`contentItemId` 或等价ID；不得为两个模板分别实现两套编辑状态和保存逻辑。
- 编辑只允许作用于当前 `currentRevision`。旧Revision、`legacy_unverified`、`archived`、`invalid_reference` 和 revision不一致状态必须禁止编辑。
- 保存时发生 `expectedRevision` 冲突不得静默覆盖。
- 双击只是快捷入口；同时支持选中后的编辑按钮、Enter/F2进入、Escape取消、Ctrl/Cmd+Enter保存。
- G0a只使用明确保存和取消，不实现自动保存、失焦保存或防抖后台Revision。
- 分支切换、Revision变化、恢复、撤销、保存成功和离开编辑模式时，必须清除本地编辑草稿、选中状态、错误状态和pending operationId。
- 模板切换不得创建内容Revision。
- 编辑UI必须与导出层隔离；选中边框、编辑按钮、textarea、错误提示和编辑工具栏不得出现在PDF中。
- 如果实现过程中发现必须新增持久化ResumeDocument表、重构V1 Revision系统或引入复杂编辑器依赖，立即停止并报告，不得自行扩大G0a范围。

## 5. 用户流程

1. 用户打开 `/resume`。
2. 选择一个 verified ResumeBranch。
3. 打开编辑模式。
4. 单击某个经历/项目/技能区块，区块进入选中状态。
5. 双击正文，进入行内编辑。
6. 修改一条合法文本。
7. 点击保存。
8. Repository重新运行规则 Fact Guard。
9. 保存成功后预览立即更新。
10. 用户点击撤销，恢复上一版内容。
11. 导出仍走V1现有模板和导出链路。

## 6. 数据模型

首选最小派生模型：

```text
ResumeDocument
- id
- branchId
- profileId
- jobId
- sections
- templateId
- contentRevision
- presentationRevision
- sourceBranchRevision
```

`sections[].blocks[]` 包含 `blockId`、`contentItemId`、`text`、`type`、`factRefs`、`guardStatus`、`visible`、`order`。

## 7. Schema变化

G0a只允许新增或定义派生视图模型类型，不持久化 `ResumeDocument`，不新增Dexie表，不升级Dexie v8，不改变V1既有Schema语义。

## 8. Dexie迁移

G0a不执行Dexie迁移。若发现必须新增持久化表，停止并报告。

## 9. Mapper

新增 `mapBranchToResumeDocument`：

- 输入 verified ResumeBranch、CareerProfile、JobDescription、当前模板偏好。
- 对 `legacy_unverified`、`archived`、`invalid_reference` 标记为不可编辑；正式编辑入口必须拒绝。
- 保留factRefs、guardStatus、sourceTrace。
- 映射全部contentItem，并明确 `visible`、`renderable`、`editable`、`guardStatus`；模板只渲染 `visible && renderable` 内容。

## 10. Repository

G0a不新增第二套内容保存Repository。允许新增派生视图读取/helper，但文本保存必须复用现有内容写路径：

- 派生视图读取：从当前 `ResumeBranch/currentRevision` 即时映射，不持久化。
- 文本保存：调用 `editResumeBranch({ branchId, expectedRevision, operationId, edits })`。
- 撤销：复用 `undoResumeBranch`。
- 冲突：`expectedRevision` 不一致时抛出/展示冲突错误，不静默覆盖。

## 11. 页面变化

`/resume` 增加：

- 编辑模式开关。
- 区块选中边框。
- 双击进入inline textarea/input。
- 保存/取消按钮。
- Fact Guard错误展示。
- 撤销入口复用现有撤销。
- 编辑按钮和 Enter/F2 快捷键。
- Escape取消，Ctrl/Cmd+Enter保存。

不改首页、profile、jobs业务流程。

## 12. 状态管理

- selection：`branchId + revision + blockId`。
- draftText：同样带revision key。
- 保存成功、分支切换、Revision变化、恢复版本、撤销、离开编辑模式后清理本地草稿、选中、错误和pending operationId。
- 模板切换不清空内容，只重新渲染。
- 不实现自动保存、失焦保存或防抖后台Revision。

## 13. Fact Guard

- 内容修改必须重新运行规则 Fact Guard。
- 高风险阻断，不写入正式分支。
- 合法修改创建内容Revision。
- 样式、模板、选中状态不运行Fact Guard。

## 14. Revision

- 内容修改创建 ResumeRevision。
- `previousRevisionId` 保留。
- 撤销追加新revision，不直接回滚数据库。
- 模板选择不创建内容Revision。

## 15. 测试

单元测试：

- Mapper拒绝legacy/archived/invalid_reference。
- Mapper保留factRefs和guard状态。
- Mapper映射隐藏或不可渲染contentItem，并正确标记visible/renderable/editable。
- 合法编辑创建revision。
- 高风险编辑被Fact Guard阻断。
- 分支切换清理草稿key。
- 模板切换不创建Revision。
- 相同operationId不重复创建Revision。
- expectedRevision冲突阻止覆盖。

E2E：

- 打开verified分支，选中区块。
- 双击编辑合法文本，保存后预览同步。
- 选中后编辑按钮、Enter/F2、Escape、Ctrl/Cmd+Enter可用。
- 高风险新增数字/技能被阻断。
- 撤销后文本恢复，编辑缓存不残留。
- 模板A/B编辑内容一致。
- 分支隔离。
- 旧Revision不可编辑。
- legacy/archived/invalid_reference不可编辑。
- 编辑控件不进入PDF。
- 导出入口仍可用。

回归：

- `pnpm verify`
- 相关E2E
- `pnpm test:c1:eval`
- `pnpm test:c2:eval`

## 16. V1回归

- 文本/PDF导入不受影响。
- JD解析、匹配、建议、分支创建不受影响。
- V1两套模板仍可切换。
- 浏览器打印导出仍可记录ExportRecord。

## 17. 数据安全

- 不写API Key。
- 不发送新增外部AI请求，除非用户触发已有AI任务。
- 不把未确认事实写入正式材料。
- 不修改CareerProfile事实层。

## 18. 停止条件

- C2 `unsafeAllowed` 不为0。
- 发现必须新增持久化ResumeDocument表、升级Dexie、重构V1 Revision系统或引入复杂编辑器依赖。
- 合法编辑后预览不同步。
- 撤销后出现旧草稿残留。
- 编辑一个分支影响另一个分支。
- legacy_unverified可编辑。
- 导出回归失败。

## 19. 预计修改文件

- `src/domain/schemas/*`：可选新增最小ResumeDocument Schema。
- `src/domain/resumeDocument/*`：新增Mapper。
- `src/services/storage/repositories.ts`：新增编辑入口或复用封装。
- `src/app/resume/ResumeWorkspace.tsx`：编辑模式UI。
- `src/components/resume/*`：可编辑预览组件。
- `tests/unit/*`：Mapper和Repository测试。
- `tests/e2e/*`：G0a端到端测试。
- `plan2.md`、`history2.md`。

## 20. 执行顺序

1. 复核V1状态和目标分支数据。
2. 写最小Mapper测试。
3. 实现ResumeDocument派生模型。
4. 接入Repository合法编辑路径。
5. 在预览区实现选中和双击编辑。
6. 接入保存、取消、错误状态。
7. 接入撤销和缓存清理。
8. 跑单元测试和E2E。
9. 跑V1回归和C1/C2 eval。
10. 更新 `plan2.md` 和 `history2.md`。

## 21. 完成定义

- 用户能在 `/resume` 预览区直接编辑一条合法内容。
- 保存后预览即时更新。
- 高风险事实修改被阻断。
- 内容修改创建Revision。
- 撤销恢复正确。
- 分支切换或恢复后无本地编辑缓存残留。
- V1模板切换和PDF导出仍可用。
- 未进入DOCX/OCR/多Profile/Application/自由画布。

## 22. 后续Goal

G1：补齐拖拽排序、显示隐藏、右侧属性面板、样式配置和更完整undo/redo。

G2：正式模板中心和第一批4套模板。
