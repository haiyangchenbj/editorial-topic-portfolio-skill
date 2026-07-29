# 选题组合评估与写作排期（Editorial Topic Portfolio）

> 对一批科技 / AI / 数据 / 云 / 企业软件选题做组合评估，过硬门禁、评分，输出当前周期 1 个主写 + 2 个备选，以及合并 / 观察 / 已覆盖 / 放弃决定与经确认的 Notion 回写预览。

[![ClawHub](https://img.shields.io/badge/ClawHub-editorial--topic--portfolio--skill-blue)](https://clawhub.ai/haiyangchenbj/editorial-topic-portfolio-skill)
[![GitHub](https://img.shields.io/badge/GitHub-haiyangchenbj-black)](https://github.com/haiyangchenbj/editorial-topic-portfolio-skill)

---

## 它做什么

采用 Routing + Prompt Chaining：归一化 Notion 或 Markdown/JSON/CSV 选题输入，施加 G0–G5 硬门禁（事实、时效、原创、长文承载力、定位、隐私），以 100 分模型评分，求解恰好 1 主写 + 2 备选的组合，并产出经确认的 Notion 变更预览——未经两次确认绝不写入。

## 何时使用

- 需要从 Notion 选题库或本地文件确定本周 / 下周写作组合。
- 需要合并 / 观察 / 短观察 / 已覆盖跟踪 / 放弃处置。
- 需要回写前的 Notion 字段变更预览。

## 何时不使用

- 直接写正文 → 主写交给 `industry-deep-dive-pipeline`。
- 微信排版 / 发布内容 → 用发布套件。
- 未经确认直接写 Notion。

## 关键硬规则

- 硬门禁先于评分；BLOCK 不得被高分抵消。
- 每轮恰好 1 主写 + 2 备选，除非人工覆盖并记录原因。
- 双确认（组合 + 回写）后才允许任何 Notion 写入。
- 凭据、数据库 ID、私人选题留私有层——绝不进入通用 Skill。

## 目录结构

```
editorial-topic-portfolio/
├── SKILL.md
├── SKILL_zh.md
├── README.md
├── README_zh.md
├── _meta.json
├── references/   # 门禁、评分模型、组合规则、Notion 适配边界、回放
├── scripts/      # 归一化、评分、变更集校验、Notion 适配器
└── templates/    # 变更预览、评估报告、组合决策、映射样例、私有 profile 样例
```

## 许可证

MIT
