# Roadmap And Backlog

## V1收尾池

| 项目 | 用户问题 | 价值 | 优先级 | 复杂度 | 风险 | 依赖 | 复用V1 | 里程碑 | 验收标准 |
|---|---|---|---|---|---|---|---|---|---|
| 快速启动说明 | 新环境不易复现 | 降低交接成本 | P0 | S | low | README | 是 | V1 closeout | 新用户按说明可启动 |
| V1稳定检查点 | V2前基线不清 | 可回滚 | P0 | S | medium | Git状态 | 是 | V1 closeout | 工作树清楚并有检查点 |
| Demo运行手册 | 比赛演示路径散 | 可复现演示 | P1 | M | low | 示例数据 | 是 | V1 closeout | 3-4分钟脚本可跑 |
| 模型与开源清单 | 交付合规不足 | 降低风险 | P1 | S | low | package/AI配置 | 是 | V1 closeout | 列明模型、库和用途 |

## V2 P0

| 项目 | 用户问题 | 价值 | 优先级 | 复杂度 | 风险 | 依赖 | 复用V1 | 里程碑 | 验收标准 |
|---|---|---|---|---|---|---|---|---|---|
| ResumeDocument最小模型 | 分支不可直接编辑 | 编辑核心 | P0 | M | high | ResumeBranch | 是 | G0 | 分支可映射为文档 |
| 预览区直接编辑 | 表单编辑不直观 | 用户可见提升 | P0 | L | high | Fact Guard | 是 | G0 | 单击选择、双击编辑、保存同步 |
| 内容/样式Revision分离 | 样式会污染事实历史 | 安全可维护 | P0 | M | high | Branch revision | 部分 | G0/G1 | 样式不创建内容Revision |
| 模板注册底座 | 2套模板不可扩展 | 后续模板中心 | P0 | M | medium | RenderModel | 是 | G2 | 模板元数据和能力可注册 |
| PDF导出安全门 | 导出失败影响成品 | 稳定交付 | P0 | M | high | ExportRecord | 是 | G3 | overflow阻断且记录错误 |

## V2 P1

| 项目 | 用户问题 | 价值 | 优先级 | 复杂度 | 风险 | 依赖 | 复用V1 | 里程碑 | 验收标准 |
|---|---|---|---|---|---|---|---|---|---|
| 拖动排序 | 调整经历不顺手 | 编辑效率 | P1 | M | medium | Editor state | 部分 | G1 | 拖动不改事实 |
| 样式属性面板 | 模板不够可调 | 成品感 | P1 | M | medium | StyleConfig | 部分 | G1 | 字号/间距/颜色可调 |
| 第一批正式模板 | 求职场景覆盖不足 | 直接价值 | P1 | M | medium | Template registry | 是 | G2 | 4套模板可切换 |
| PDF直接下载 | 打印框不稳定 | 导出体验 | P1 | L | high | 模板CSS | 是 | G3 | 可直接生成可复制PDF |
| 区块级岗位建议 | 建议和预览分离 | 修改更直观 | P1 | M | high | C1/C2 | 是 | G5 | 建议定位到block |
| 排版诊断 | 不知为何溢出 | 降低失败 | P1 | M | medium | layout metrics | 部分 | G5 | 给出拥挤/留白/ATS提示 |

## V2 P2

- DOCX内容导入：复杂度L，风险medium，G4后评估。
- DOCX导出：复杂度L，风险high，依赖PDF导出稳定。
- 多Profile显式管理：复杂度L，风险medium，G6。
- Application求职管理：复杂度L，风险medium，G6。
- 申请材料包：复杂度M，风险medium，G7。
- 面试问题和STAR草稿：复杂度M，风险medium，G7。

## V2 P3

- OCR。
- 浏览器扩展。
- 云同步。
- 职业能力图谱。
- 语音面试。
- 模板市场。

## defer

- 向量数据库。
- 任意复杂PDF样式近似映射。
- PNG批量导出。
- 中英文双语完整模板。

## do_not_build

- 自动批量投递。
- 验证码破解、Cookie窃取、反风控绕过。
- 无证据事实生成。
- 第一阶段任意PDF一比一自由画布编辑。
