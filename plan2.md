# CareerAdapt AI 第二代开发计划

## 0. 文档使用规则

`Plan.md` / `history.md` 是 V1 MVP 历史档案；`plan2.md` / `history2.md` 是 V2 活动文档。V2每次开发必须先读本文件，再读 `history2.md` 最近相关记录、`docs/MVP_V1_HANDOFF.md`、`docs/V2_START_HERE.md` 和当前 Goal 对应设计文档。

状态标记：`[ ]` 未开始，`[~]` 进行中，`[x]` 已完成，`[!]` 阻塞，`[>]` 后置，`[-]` 不做或取消。

每次 V2 开发结束必须更新本文件的任务状态、更新 `history2.md`、记录验证结果，并保持 V1 主链路不被破坏。

## 1. 当前V1基线

- 当前 MVP 已完成：粘贴/文本型 PDF 导入、CareerProfile、JD解析、RequirementMatch、AI建议、Fact Guard、JobAdaptationDraft、verified ResumeBranch、双模板 A4 预览、浏览器打印 PDF 导出、PDF 来源追踪。
- 当前测试状态以 `history.md` 的 E1.1 记录为准：typecheck、lint、unit、build、e2e、C1 eval、C2 eval 均曾通过。
- 当前 Dexie 实际版本：v7。
- 当前稳定门槛：C2 `unsafeAllowed=0`，未确认事实进入正式材料数量为 0。
- 当前待收尾：V1 Git 检查点、快速启动/演示说明、比赛截图/视频/PPT、模型与开源组件说明、旧 Plan 状态补记。

## 2. V1收尾任务

| 任务 | 状态 | 是否阻塞V2代码 | 说明 |
|---|---|---|---|
| E1独立验收 | `[x]` | 否 | 已有 E1.1 记录；本地生成用例不进 Git。 |
| 快速启动脚本或启动说明 | `[ ]` | 是 | V2业务编码前至少要有可复现启动路径。 |
| `verify`脚本 | `[x]` | 否 | `package.json` 已有 `pnpm verify`。 |
| MVP v1.0 Git检查点 | `[ ]` | 是 | 当前工作树非干净；V2编码前应建立稳定检查点。 |
| Demo运行手册 | `[ ]` | 否 | 属比赛交付，可与V2规划并行，不与V2编码混做。 |
| 截图、视频、PPT | `[ ]` | 否 | 属比赛交付，不阻塞本次规划。 |
| 严重Bug修复 | `[!]` | 是 | 若出现事实安全、导出或数据损坏问题，优先于V2功能。 |
| 文档一致性修复 | `[x]` | 否 | 本轮已冻结V1文档并建立V2入口。 |

## 3. 第二代核心定位

第二代首先是一个可直接编辑、可套用正式模板、可针对岗位优化并稳定导出的智能简历工作台。

一句话价值主张：将用户已有简历、职业事实和岗位要求转化为可直接编辑、可切换正式模板、可针对岗位定制并稳定导出的智能简历工作台。

Workspace、多 Profile 和 Application 管理是后续扩展，不能压过“简历制作、编辑、模板、岗位定制、导出”主线。

## 4. 第二代目标用户

- 第一次找实习的大学生，需要把零散经历快速变成正式简历。
- 已有旧简历，希望导入后快速换模板和调整表达的用户。
- 需要针对多个岗位生成不同简历的用户。
- 对排版不熟悉但需要正式、稳定、可导出的简历的用户。
- 关注事实真实性、来源和AI修改可解释性的用户。

## 5. 不可破坏原则

- 内容与排版分离，正式事实层与表达层分离。
- Evidence Mapping 可追溯，所有事实内容修改继续经过 Fact Guard。
- 模板不得生成事实，样式修改不得修改事实或创建内容 Revision。
- AI不得静默写入正式事实，用户确认优先。
- 本地优先，日志脱敏，不把 API Key 写入源码或前端。
- 所有写操作可撤销，迁移幂等、可测试、无损。
- 不自动投递，不绕过平台规则，不编造经历。

## 6. 第二代非目标

第一阶段不做：任意PDF一比一自由画布还原、OCR、复杂云同步、自动申请、批量投递、验证码绕过、模板市场、付费系统、完整移动端设计器、语音面试评分、从零重写全部V1代码。

## 7. V1迁移摘要

完整审计表见 [`docs/v2/V1_TO_V2_MIGRATION.md`](docs/v2/V1_TO_V2_MIGRATION.md)。

- V1已完成：文本/PDF导入、JD分析、匹配、建议、Fact Guard、多岗位分支、两套模板、A4预览和浏览器打印导出。
- V1收尾：启动说明、Git检查点、Demo材料、模型与开源清单、旧 Sprint 状态补记。
- V2 P0：Resume Studio、直接编辑、ResumeDocument、正式模板中心底座、稳定导出增强。
- V2 P1：拖拽排序、样式面板、排版诊断、区块级岗位建议、第一批正式模板。
- 后置：DOCX、OCR、云同步、浏览器扩展、Application看板、模板市场。
- 不做：自动批量投递、验证码绕过、无证据事实生成、第一阶段任意PDF完全还原。

## 8. 第二代能力地图

| 能力 | 用户问题 | 价值 | 优先级 | 复用V1 | 依赖 | 复杂度 | 风险 | 验收标准 |
|---|---|---|---|---|---|---|---|---|
| Resume Studio统一简历模型 | V1分支可预览但不可直接编辑 | 形成可编辑内容核心 | P0 | Branch/RenderModel | G0 | M | high | verified分支可映射为可编辑文档 |
| 所见即所得编辑器 | 用户不想在表单和预览间来回猜 | 直接修改简历正文 | P0 | ResumeWorkspace/Fact Guard | G0/G1 | L | high | 单击选中、双击编辑、保存预览同步 |
| 正式模板中心 | 2套模板不足以覆盖求职场景 | 提升成品感 | P0/P1 | 模板A/B | G2 | M | medium | 模板元数据、缩略图、第一批模板可切换 |
| 岗位针对性修改 | 建议卡片与简历区块联动不足 | 修改更直观 | P1 | C1/C2 | G5 | M | high | 建议定位到区块并保留依据 |
| 排版诊断 | 用户不知道为什么溢出或不正式 | 降低导出失败 | P1 | overflow检测 | G5 | M | medium | 可给出拥挤、留白、字号、ATS提示 |
| 稳定PDF/DOCX导出 | 浏览器打印不够稳定 | 可下载、可验证 | P1 | ExportRecord/打印CSS | G3 | L | high | PDF直接生成，打印fallback |
| 导入和结构映射 | 旧简历内容要进入新编辑模型 | 降低迁移成本 | P0/P1 | PDF导入/Mapper | G0/G4 | M | high | PDF导入生成ResumeDocument |
| 多Profile和Workspace | 页面隐式取profiles[0] | 支持真实多用户/多档案 | P2 | useWorkspace | G6 | L | medium | 显式上下文，不串数据 |
| Application求职管理 | 简历之外还需管理投递 | 长期闭环 | P2 | Job/Branch | G6 | L | medium | Application实体与分支绑定 |
| 申请材料包 | 简历之后需要自我介绍等 | 提高求职材料一致性 | P2 | Material思路 | G7 | M | medium | 人工确认后生成材料 |
| 面试准备 | 投递后要准备面试 | 延伸价值 | P3 | JD/Profile | G7 | M | medium | 生成问题和STAR草稿 |
| 职业能力图谱 | 长期成长建议 | 长期差异化 | P3 | Evidence | G8 | XL | high | 不影响主线 |
| 评估、隐私和可观测性 | AI安全需要可量化 | 保持可信 | P0 | C1/C2 eval | 全程 | M | high | 安全指标不下降 |

## 9. 第二代里程碑

### V2-G0：Resume Studio基础

目标：定义最小 ResumeDocument 或等价编辑模型，将 verified ResumeBranch 映射到可编辑简历文档，支持区块选中、合法文本直接编辑、保存、实时预览、撤销，并不破坏V1 PDF导出。非目标：不做完整拖拽、不做样式全量面板、不新增大量模板、不做DOCX/OCR/多Profile/Application。

### V2-G1：所见即所得编辑器

目标：单击选择、双击编辑、拖动排序、显示隐藏、右侧属性面板、字号/间距/主题色、内容Revision与展示配置分离、undo/redo。非目标：自由绝对坐标画布、任意图形设计器。

### V2-G2：正式模板中心

目标：模板注册、元数据、缩略图、岗位适用类型、ATS标记、第一批4至6套正式模板、单栏/双栏、模板偏好持久化。第一批不一次开发十几套。

### V2-G3：导出增强

目标：PDF直接下载、浏览器打印fallback、一页/两页控制、预览导出一致、PDF golden tests、文件名和导出记录。DOCX作为独立子阶段评估。

### V2-G4：导入和版式映射

目标：PDF/DOCX内容导入到 ResumeDocument，识别标题、栏目、单栏/双栏、基础字号/颜色，复杂版式提示。任意PDF完全自由编辑后置。

### V2-G5：岗位智能优化和排版诊断

目标：区块级AI建议、inline diff、内容建议、排版建议、模板推荐、拥挤/留白/ATS诊断，所有内容建议继续受 Fact Guard 约束。

### V2-G6：多Profile、Workspace和Application

目标：显式 Profile/Workspace 上下文、Application实体、岗位机会管理、求职状态和时间线。必须在 Resume Studio 稳定后启动。

### V2-G7：申请材料和面试准备

目标：求职信、邮件/私信草稿、申请问题、面试问题、STAR回答和复盘，人工确认后才回流事实层。

### V2-G8：长期适配器和同步

OCR、浏览器扩展、网页JD、云同步、加密导入导出、模板市场、语音面试。

## 10. 当前准备实施的里程碑

当前只准备 V2-G0a：Resume Studio 最小垂直切片。详细设计见 [`docs/v2/FIRST_GOAL.md`](docs/v2/FIRST_GOAL.md)。

完成定义：

- 打开一个现有 verified ResumeBranch。
- 选中一个简历区块。
- 双击或点击编辑进入行内编辑。
- 修改合法文本后 Repository 重新运行 Fact Guard。
- 保存后实时更新预览。
- 内容修改创建内容 Revision。
- 模板选择和样式不创建内容 Revision。
- 支持撤销最近一次内容修改。
- 分支切换和恢复后清理本地编辑状态。
- V1两套模板和PDF导出正常。

## 11. 安全和回归门槛

每个V2 Sprint至少运行：typecheck、lint、unit test、build、相关E2E、C1 eval、C2 eval、数据迁移测试、V1主链路回归、PDF导出回归。

必须继续保证：`unsafeAllowed=0`、未确认事实进入正式材料数量为0、模板层生成新事实数量为0、样式操作修改事实数量为0、迁移丢失数据数量为0、编辑器保存后预览不同步数量为0、一个分支的数据不得显示到另一个分支、一个Profile的数据不得绑定到错误Application。

## 12. 当前决策和开放问题

已决定事项见 [`docs/v2/DECISIONS.md`](docs/v2/DECISIONS.md)。

开放问题：

- ResumeDocument最终字段是否单独持久化，还是先由 ResumeBranch 派生。
- 是否引入 Tiptap/Lexical，或第一阶段保持自定义结构编辑。
- 是否在 G1 引入 dnd-kit。
- PDF直接下载采用客户端、服务端还是本地 Headless Chromium。
- 是否支持两页简历作为 G3 P1。
- 第一批模板最终数量为4套还是6套。
- DOCX优先级是否进入G4。
- 多Profile和Application何时进入G6。
- 是否需要领域事件表。

## 13. 下一开发路线

当前应执行的一个Goal：人工审核 `docs/v2/FIRST_GOAL.md`，确认后再启动 V2-G0a。

该Goal后的候选Goal：V2-G1 所见即所得编辑器增强，补齐拖拽排序和右侧属性面板。

当前禁止进入：V2业务代码、Schema/Dexie迁移、DOCX、OCR、Application、自由画布、模板市场、自动投递、依赖升级。
