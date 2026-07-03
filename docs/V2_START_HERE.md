# V2 Start Here

## 默认阅读顺序

1. [`plan2.md`](../plan2.md)
2. [`history2.md`](../history2.md) 最近相关记录
3. [`docs/MVP_V1_HANDOFF.md`](MVP_V1_HANDOFF.md)
4. 当前 Goal：[`docs/v2/FIRST_GOAL.md`](v2/FIRST_GOAL.md)
5. 相关设计文档
6. 当前需要修改的源码

除非涉及 V1 迁移、V1历史Bug或旧设计争议，不默认读取完整 `Plan.md` 和 `history.md`。

## V2主线

V2不是优先做求职看板，也不是只堆模板数量。主线是：

```text
导入自己的简历
-> 提取并保留事实来源
-> 转换为统一简历内容模型
-> 进入可视化简历编辑器
-> 点击选择、双击编辑、拖动排序
-> 通过右侧面板调整样式
-> 切换正式模板
-> 根据岗位生成针对性建议
-> 事实安全与排版检查
-> 稳定导出 PDF
```

## 当前禁止事项

- 不开始V2业务代码，除非 `FIRST_GOAL.md` 已经过人工审核。
- 不修改业务Schema，不升级Dexie，不执行V2迁移。
- 不引入DOCX、OCR、多Profile、Application或模板市场作为首个Goal。
- 不做任意PDF一比一自由画布还原。
- 不自动投递，不绕过招聘平台规则。

## 设计文档导航

- 产品愿景：[`PRODUCT_VISION.md`](v2/PRODUCT_VISION.md)
- 简历工作台：[`RESUME_STUDIO_SPEC.md`](v2/RESUME_STUDIO_SPEC.md)
- 模板系统：[`TEMPLATE_SYSTEM.md`](v2/TEMPLATE_SYSTEM.md)
- 编辑交互：[`EDITOR_INTERACTION.md`](v2/EDITOR_INTERACTION.md)
- 导入导出：[`IMPORT_AND_EXPORT.md`](v2/IMPORT_AND_EXPORT.md)
- 领域架构：[`DOMAIN_AND_ARCHITECTURE.md`](v2/DOMAIN_AND_ARCHITECTURE.md)
- AI与安全：[`AI_AND_SAFETY.md`](v2/AI_AND_SAFETY.md)
- V1到V2迁移：[`V1_TO_V2_MIGRATION.md`](v2/V1_TO_V2_MIGRATION.md)
- 路线与待办：[`ROADMAP_AND_BACKLOG.md`](v2/ROADMAP_AND_BACKLOG.md)
- 首个Goal：[`FIRST_GOAL.md`](v2/FIRST_GOAL.md)
- 决策记录：[`DECISIONS.md`](v2/DECISIONS.md)

## 下一步

人工审核 `plan2.md` 与 `FIRST_GOAL.md`。审核通过后，单独启动 V2-G0a：Resume Studio 最小垂直切片。
