# Editorial Topic Portfolio

> Evaluate a batch of technology / AI / data / cloud / enterprise-software topics, pass hard gates, score, and output the current cycle's 1 primary-write + 2 backups with merge / watch / covered / abandon decisions and a confirmed Notion writeback preview.

[![ClawHub](https://img.shields.io/badge/ClawHub-editorial--topic--portfolio--skill-blue)](https://clawhub.ai/haiyangchenbj/editorial-topic-portfolio-skill)
[![GitHub](https://img.shields.io/badge/GitHub-haiyangchenbj-black)](https://github.com/haiyangchenbj/editorial-topic-portfolio-skill)

---

## What it does

A Routing + Prompt-Chaining skill that normalizes Notion or Markdown/JSON/CSV topic inputs, applies G0–G5 hard gates (fact, timeliness, originality, argument-capacity, positioning, privacy), scores eligible topics on a 100-point model, and solves a portfolio of exactly 1 primary + 2 backups. Produces a confirmed Notion change-set preview — never writes without two explicit confirmations.

## When to use

- You need this / next week's writing组合 from a Notion topic database or local files.
- You need merge / watch / short-note / covered-track / abandon dispositions.
- You need a Notion field-change preview before writeback.

## When not to use

- Writing the body directly → hand the primary to `industry-deep-dive-pipeline`.
- WeChat layout / publish content → use the publishing suite.
- Writing to Notion without confirmation.

## Hard rules (key)

- Hard gates before scoring; a BLOCK cannot be offset by a high score.
- Exactly 1 primary + 2 backups per cycle, unless a manual override with recorded reason.
- Two confirmations (portfolio + writeback) before any Notion write.
- Credentials, database IDs, private topics stay in the private layer — never in the generic skill.

## File structure

```
editorial-topic-portfolio/
├── SKILL.md
├── SKILL_zh.md
├── README.md
├── README_zh.md
├── _meta.json
├── references/
│   ├── evaluation-gates.md
│   ├── scoring-model.md
│   ├── portfolio-rules.md
│   ├── notion-adapter-interface.md
│   └── replay-evaluation.md
├── scripts/
│   ├── normalize_topics.py
│   ├── calculate_scores.py
│   ├── validate_change_set.py
│   └── notion_topic_adapter.js
└── templates/
    ├── change-preview.template.md
    ├── evaluation-report.template.md
    ├── portfolio-decision.template.md
    ├── notion-mapping.example.json
    ├── private-profile.example.json
    ├── topic-record.template.json
    └── writeback-result.template.json
```

## License

MIT
