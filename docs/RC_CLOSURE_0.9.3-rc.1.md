# CareerAdapt AI Core Beta RC Closure

## Snapshot

- Version: `0.9.3-rc.1`
- Planned tag: `release-v0.9.3-rc.1`（blocked，未创建）
- Starting HEAD: `4060210e2a535a5fb7983ea026aabcd33d955e2b`
- Starting working tree: clean
- Branch baseline: `main...origin/main [ahead 163]`
- Date: 2026-07-29

## Focused RC fixes

- A1 PASS：`derived_display` 只保留为 Review label。education、awards、skills、certificates、languages 及其余 canonical sections 不再从 derived title 生成正式 identity；缺少 identity 的 candidate 保留并强制确认，不生成 canonical structured item。
- A2 PASS：`conversation + needsNormalization=true` 在 UI、tool service 和 `WorkspaceRepository.confirmProfileIntake` 三层禁止普通采用/提交。UI 只提供重试整理、编辑后采用、补充细节、忽略；恶意强制 included 的 Draft 仍被 Repository 拒绝。
- A3 PASS：多 candidate 复用相同或高度重叠整段 narrative 时，全部降为 `needsConfirmation`；candidate-local field evidence 保留。
- Conversation Intake 在 A1–A3 后继续冻结；未新增 semantic field、completeness dimension、prompt feature、intake UI 或 normalizer architecture。

## Deterministic gates

- `pnpm install --frozen-lockfile`: PASS
- changed-files ESLint: PASS
- `pnpm typecheck`: PASS
- `pnpm lint`: PASS
- `pnpm test`: PASS，112 files / 780 tests
- `pnpm build`: PASS
- `pnpm test:c1:eval`: PASS，1/1 deterministic hard gate
- `pnpm test:c2:eval`: PASS，1/1 deterministic hard gate
- `git diff --check`: PASS
- Focused A1–A3: PASS，5 files / 80 tests

C1/C2 的外部 Judge 均返回 HTTP 401；这里只把 deterministic hard gate 记为 PASS。

## Real Provider smoke

- Provider: `openai-compatible`
- Model: `mimo-v2.5-pro`
- Result: FAIL
- Failure: HTTP 401
- Sensitive input/body: not recorded

| Task | Prompt version | Latency | Schema result |
| --- | --- | ---: | --- |
| health check | `health-check.v1` | 165 ms | no result; HTTP 401 |
| profile builder | `profile-builder.v1` | 91 ms | no result; HTTP 401 |
| JD analyzer | `jd-analyzer.v3-unit-ledger` | 120 ms | no result; HTTP 401 |
| evidence matcher | `evidence-matcher.v2` | 167 ms | no result; HTTP 401 |
| resume tailor | `resume-tailor.v3-minimal-output` | 141 ms | no result; HTTP 401 |
| fact guard semantic review | `fact-guard.v1` | 36 ms | no result; HTTP 401 |

`pnpm test:ai:real` 最终为 1 passed / 6 failed；通过项是 deterministic empty-candidate normalization，不是 Provider 成功。

## Golden Journeys

### J1 — File → Profile → General Resume → PDF: FAIL

- PASS：真实 PDF attachment 解析；真实 DOCX attachment 解析；external JSON canonical mapping。
- PASS：canonical fixture Preview→PDF，Render Coverage、两页分页、中文文本层、条目唯一性。
- FAIL：standard JSON v2 的 review→new Profile/General Resume commit 在确认边界进入“任务暂时中断”，没有产生确认卡。
- 未形成同一代表性 PDF/DOCX/JSON 各自贯穿到最终 PDF 的完整可重复 journey。

### J2 — Conversation → Profile → Resume: FAIL

- PASS：mock/deterministic 路径中，陌生用户 Rich Review→CareerProfile write-back、零 Resume→General Resume、export_ready 共 3/3。
- FAIL：真实 Provider 6 个核心调用全部 HTTP 401；没有真实 Provider 的成功 Semantic Intake 核心路径。

### J3 — Re-import / Reconcile: FAIL

- PASS：same resume semantic idempotency；revised resume 的 unresolved conflict reload；只提交真实 delta；OpenDataLoader provenance reload，共 3/3。
- Unit gate 覆盖 exact duplicate、evidence extension、compatible update、conflict 和 keep existing。
- FAIL：本次没有取得 `use imported` 与 `keep both as distinct` 的完整执行证据，不能按 RC 条件宣称全矩阵通过。

### J4 — Branch Isolation: FAIL

- PASS：当前 Agent route 能为新岗位创建独立 Revision，且 existing-job route 不改变 General source，2/2。
- Unit gate 覆盖 Repository operationId、factRefs 与 sibling branch isolation。
- FAIL：完整 General + Job A + Job B 相互隔离 browser journey 在当前 Jobs UI 找不到旧第三个 tab，未证明 Job A/Job B 双向不污染。

### J5 — Persistence / Reload: FAIL

- PASS：library picker→ResumeBranch→reload。
- FAIL：canonical Profile UI 使用 `award/skill/certificate/language` 单数 data-section-type，而 v2 catalog 为 plural。
- FAIL：两条结构编辑保存/切换回归未通过；没有形成 Profile、Job、General、Job Resume、Tailoring、Revision、Export 的单一 reload/new-session 全链证据。

### J6 — Failure Recovery: FAIL

- PASS：active Profile 切换清 stale Resume；mutation claim grounding；typed ignore 不生成 user message。
- FAIL：纠正未点击的 confirmation 后没有重建可见确认按钮。
- FAIL：最终 Job PDF export-failure/retry journey 无法越过已漂移的 Jobs source-resume 入口。
- Unit gate 覆盖 stale revision/version、duplicate operationId 和多类 provider/tool failure，但不足以代替完整 browser recovery journey。

## Final artifact verification

- Canonical/general presentation PDF: PASS
  - HTTP 200 PDF
  - Render Coverage 无 warning
  - 中文可由 `pdftotext` 提取
  - 两页内容顺序完整
  - 关键条目无 duplicate
- Final Job Resume PDF: FAIL
  - 当前 Job Resume 创建/导出 browser flow 无法到达最终下载。
  - 因此不能把“Revision 创建成功”或 canonical fixture PDF 代替最终 Job artifact verification。

## Defects

### P0: 0

未观察到事实丢失、跨 Profile 写入或分支污染的已证实 P0；但未通过的 journeys 不能作为“无此风险”的完整证明。

### P1: 7

1. Real Provider credentials/configuration return HTTP 401，阻断真实 AI core smoke。
2. Standard JSON v2 import 在新 Profile/General Resume confirmation 边界中断。
3. J3 缺少 `use imported` / `keep both` 的完整 release execution evidence。
4. General + Job A + Job B browser isolation journey 与当前 Jobs UI 脱节。
5. Persistence 路径存在 canonical section type 单复数漂移和结构编辑保存回归。
6. Confirmation correction recovery 未重建可执行确认按钮。
7. 最终 Job Resume 创建→PDF→failure retry 路径无法到达，最终 artifact 未验证。

### P2: 2

1. 一批历史 manual-first Playwright selectors 与当前 AI-first shell/Jobs UI 不一致，长组合批次无法作为现行 release gate。
2. 新 deterministic CI 只在本地等价命令验证，尚未取得远端 GitHub Actions 运行；planned RC tag 因 blocked 未创建。

## Release decision

Is CareerAdapt AI Core Beta-ready?

**NO**

Beta Core 不冻结；在 7 个 P1 关闭、六条 Golden Journey 全通过、至少一个真实 Provider 核心路径成功且最终 Job PDF 完成 artifact verification 前，不进入真实 Beta 用户测试。
