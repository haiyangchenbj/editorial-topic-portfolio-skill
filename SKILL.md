---
name: editorial-topic-portfolio
description: Evaluate a portfolio of technology, AI, data, cloud, or enterprise-software content topics. Normalizes Notion or Markdown/JSON/CSV inputs, applies factual, timeliness, originality, argument-capacity, positioning, and privacy gates, scores eligible topics, selects one primary and two backups, produces merge/watch/covered/abandon decisions, and prepares a confirmed Notion writeback preview. Never writes to Notion without two explicit confirmations.
description_zh: 选题组合评估与写作排期
description_en: Editorial topic portfolio
version: 1.0.0
disable: false
agent_created: true
read_when:
  - editorial topic portfolio
  - topic prioritization
  - 选题组合
  - 本周写什么
---

# Editorial Topic Portfolio

Evaluate a batch of technology / AI / data / cloud / enterprise-software topics and output the current cycle's 1 primary-write + 2 backups, plus merge / watch / short-note / covered-track / abandon decisions. Uses Routing + Prompt Chaining: pass hard gates first, then score and solve the portfolio.

## When to use

- You need to determine this / next week's writing组合 from a Notion topic database or local files.
- You need to decide whether a topic is an independent long-form, merges into another article, shrinks to a short note, stays on watch, or closes.
- You need to check competition density, originality space, fact readiness, timeliness window, and positioning fit.
- You need to generate a Notion field-change preview before writeback.

## Do not use

- Writing the body directly; hand the primary topic to `industry-deep-dive-pipeline`.
- Generating WeChat layout, summaries, covers, social assets, or publish content directly.
- Writing to Notion without confirmation.
- Ranking purely by热度, a single number, or headline spreadability.

## Inputs

### Supported sources

- Markdown, JSON, CSV: convert directly to a standard topic record.
- Notion: read via a private adapter; credentials, database ID, and field mapping come only from secure environment variables or private config.

### Required fields

- topic title
- source date and captured date
- lifecycle status
- topic type
- source/evidence
- hypothesis or value judgment
- related published content

Read `references/evaluation-gates.md`, `references/scoring-model.md`, and `references/portfolio-rules.md` for fields and rules.

## Workflow

### Step 1: [Deterministic] Normalize input

1. Route Notion or file input.
2. Run:

```bash
python scripts/normalize_topics.py --input <input.md|input.json|input.csv> --output <run-dir>/01-normalized-topics.json
```

3. Deduplicate titles, URLs, event subjects, and synonymous topics.
4. Keep a stable topic ID or Notion page ID; title is display-only.

### Step 2: [Deterministic + LLM] Pre-screen

1. Run G0 input-completeness check.
2. Classify event / trend / framework / case / policy / research.
3. Check lifecycle status; identify published, adopted, watch, and closed items.
4. Output `02-gate-precheck.json`.

### Step 3: [LLM + Search] Research evidence and competition

1. Re-verify key facts, dates, policy status, product status, and amount caliber.
2. Search Chinese/English topic, synonymous judgments, analytical frameworks, signature expressions, counter-arguments, and historical counterexamples.
3. Check overlap with published, in-progress, and same-batch topics.
4. For watch items, set trigger conditions, required evidence, and re-check date.
5. Output `03-evidence-review.md`.

### Step 4: [LLM] Apply hard gates

For each topic output G0–G5:

- G0 input completeness.
- G1 fact and status.
- G2 timeliness and trigger.
- G3 originality space.
- G4 long-form capacity.
- G5 positioning and privacy boundary.

Output `04-gate-results.json`. A BLOCK cannot be offset by a high score.

### Step 5: [Deterministic] Calculate scores

For topics passing hard gates or recoverable HOLD, fill 0–5 raw scores:

- Originality space & information gain: 20.
- Reader judgment value: 20.
- Fact & evidence readiness: 15.
- Timeliness & window: 15.
- Content positioning fit: 15.
- Portfolio contribution: 10.
- Execution feasibility: 5.

Deduct for fact uncertainty, heating competition, over-broad angle, recent repetition, and public risk to get the net score. Run:

```bash
python scripts/calculate_scores.py --input <run-dir>/04-gate-results.json --output <run-dir>/05-scores.json
```

### Step 6: [LLM + Deterministic] Build portfolio

1. Select exactly 1 `WRITE_NOW` and 2 `BACKUP`.
2. Constraint: primary + backups cannot all belong to the same company, event, or argument framework.
3. Mark remaining topics `MERGE`, `WATCH`, `SHORT_NOTE`, `COVERED_TRACK`, or `ABANDON`.
4. State the fact / originality / timeliness / portfolio reason for each decision.
5. Output `06-portfolio-decision.md`.

### Gate A: [Human] Confirm portfolio

Pause and confirm:

- 1 primary + 2 backups.
- P0–P3 adjustments.
- Merge / watch / covered / abandon handling.
- Manual-override reason and validity period.
- Planned writeback fields.

No change-set generation before confirmation.

### Step 7: [Deterministic] Generate change preview

Generate `07-change-preview.md` and `07-change-set.json`, listing line by line:

- page ID or stable topic ID.
- old/new values of status, priority, competition density, originality, value, positioning fit, evaluation date.
- changes to recommended reason, suggested angle, and risk reminder.
- idempotency evaluation block.

Run:

```bash
python scripts/validate_change_set.py --input <run-dir>/07-change-set.json --output <run-dir>/07-change-set-validation.json --enforce
```

### Gate B: [Human] Confirm writeback

Explicitly list target database, page count, fields, and actions. Batch writeback only after the second confirmation.

### Step 8: [Deterministic] Writeback

Notion writeback must obey:

- `NOTION_TOKEN`, `NOTION_TOPIC_DATABASE_ID`, and mapping come from secure config.
- Use page ID; forbid fuzzy title writes.
- Preview and dry-run send no PATCH.
- On failure, pause the batch and record succeeded/failed/skipped.
- Evaluation blocks are idempotently replaced by run ID and date; never append duplicates.

Default read-only adapter:

```bash
NODE_PATH=<managed-node-workspace> node scripts/notion_topic_adapter.js read > <run-dir>/08-notion-read.json
```

Writeback must be driven by a separate, reviewed change-set; never make live writeback an unconditional automatic action.

### Step 9: [Deterministic] Readback

- Re-query the updated page IDs.
- Compare preview against actual fields.
- On inconsistency, retry once and switch request path for cross-verification.
- Do not mark complete on readback inconsistency.

Output `09-readback-verification.md`.

### Step 10: [Deterministic] Handoff primary

Convert the primary into `industry-deep-dive-pipeline`'s case brief, preserving facts, competition, core hypothesis, suggested angle, risk, counter-arguments, and series relationship. Do not write the body directly.

## Hard Rules

1. Hard gates before scoring; a BLOCK cannot be offset by a high score.
2. Facts, status, and dates must return to reliable sources; search summaries are leads only.
3. `lifecycle status` and `recommendation priority` must stay separate.
4. Each cycle must be 1 primary + 2 backups, unless a manual override with recorded reason.
5. Published topics default to `COVERED_TRACK`, unless there is clear new fact or new judgment.
6. Watch items must state trigger conditions, required evidence, and re-check date.
7. Write using page ID; title must not be the sole write key.
8. Any Notion writeback needs both portfolio confirmation and writeback confirmation.
9. Credentials, database IDs, private topics, and private positioning must not enter the generic skill or public templates.
10. Tool first-failure must retry and cross-verify; re-run after changes.
11. Do not auto-rollback adopted, published, or closed status.
12. Before completion, record output, verification, residual risk, and next task.

## Failure Handling

| Scenario | Action |
|---|---|
| Input field missing | Mark `NEEDS_INPUT`, stop scoring that item |
| Fact or policy unverifiable | `BLOCK_FACT` or `HOLD_EVIDENCE`, exclude from primary/backup |
| Timeliness expired, no gain | `BLOCK_STALE` or `COVERED_TRACK` |
| Insufficient originality space | `MERGE` or `BLOCK_SATURATED` |
| Single-event capacity insufficient | `SHORT_NOTE` or `BLOCK_THIN` |
| Positioning / privacy conflict | `BLOCK_BOUNDARY` or `PRIVATE_ONLY` |
| Portfolio not exactly 1+2 | Block change-set, re-solve or wait for manual override |
| Notion token/mapping missing | Stop, read nothing, write nothing |
| Notion writeback partial failure | Stop batch, record success/fail/skip, no blind full retry |
| Readback inconsistent | Retry once and cross-verify; if still inconsistent, stay incomplete |
| Tool first-failure | Retry same op 1–2 times, then switch tool stack |

## Output Format

```text
<run-dir>/
├── 01-normalized-topics.json
├── 02-gate-precheck.json
├── 03-evidence-review.md
├── 04-gate-results.json
├── 05-scores.json
├── 06-portfolio-decision.md
├── 07-change-preview.md
├── 07-change-set.json
├── 07-change-set-validation.json
├── 08-writeback-result.json
└── 09-readback-verification.md
```

Every cycle report must include: primary, two backups, other disposals, hard-gate blocks, net score, writeback status, and next-step handoff.

## References

- `references/evaluation-gates.md`: G0–G5 hard gates.
- `references/scoring-model.md`: 100-point model and deductions.
- `references/portfolio-rules.md`: portfolio, lifecycle, and idempotency rules.
- `references/notion-adapter-interface.md`: private Notion adapter boundary.
- `references/replay-evaluation.md`: two historical replays.

## Pitfalls

- Confusing status with recommendation priority.
- Fuzzy-matching Notion pages by title and writing the wrong object.
- Appending notes so every run piles up duplicates.
- Inferring global market share or pricing power from one aggregator platform's call volume.
- Re-listing a published topic as primary.
- High score but unstable fact / policy / amount caliber.
- Ranking only by news heat, ignoring reader judgment value and originality space.
- Writing to Notion before letting the user confirm the portfolio.

## Verification

- [ ] Notion/file input both convert to a standard topic record.
- [ ] Every topic has G0–G5 result, base score, deduction, net score, and disposition.
- [ ] Exactly 1 primary + 2 backups per cycle, or explicit manual override.
- [ ] Primary and backups have no same-event / same-framework internal competition.
- [ ] Fact blocks did not enter P0/P1.
- [ ] Change-set uses only page ID or stable topic ID.
- [ ] Preview, portfolio confirmation, and writeback confirmation all recorded.
- [ ] Repeated runs do not append duplicate evaluation blocks.
- [ ] Post-writeback readback field consistency 100%.
- [ ] Real token, database ID, and private topic data never entered the skill directory.

---

## 中文摘要（Chinese Summary）

本 Skill 对一批科技 / AI / 数据 / 云 / 企业软件选题进行组合评估，输出当前周期 1 个主写 + 2 个备选，以及合并 / 观察 / 短观察 / 已覆盖跟踪 / 放弃决定。采用 Routing + Prompt Chaining，先过硬门禁，再评分和组合求解。

**关键约束（双语要点 / Bilingual key points）：**

- **硬门禁优先 Hard gates first**：G0–G5 先于评分；BLOCK 不得被高分抵消。每轮必须恰好 1 主写 + 2 备选，除非人工覆盖并记录原因。
- **双确认 Two confirmations**：组合确认（Gate A）与回写确认（Gate B）均须记录；未经确认绝不写 Notion。
- **凭据隔离 Credential isolation**：token、数据库 ID、私人选题与私有定位不得进入通用 Skill 或公开模板，只走安全环境变量 / 私有配置。
- **幂等回写 Idempotent writeback**：用 page ID 写入，评估块按 run ID + 日期幂等替换，禁止重复追加；回写后必须回读核验（一致率 100%）。
- **工具失败 Tool failure**：首次失败重试 1–2 次并换工具栈，修改后必须重跑。
