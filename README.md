# study-with-claude-code

**English** | [简体中文](README.zh-CN.md)

> An AI-driven, general-purpose study & review system — shipped as a Claude Code plugin (a set of skills). It builds a knowledge tree, runs a closed-loop review cycle, scores your mastery objectively, schedules reviews on the Ebbinghaus curve, and **gets to know how you learn** over time. Not tied to any tech stack — works for any subject.

## What it solves

It turns scattered "read it and forget it" learning into a loop that **compounds**:

```
build a tree → see what's due today → teach-then-test review → score + log weak points → reflect so it knows you better
 /swcc-plan        /swcc-daily            /swcc-go               /swcc-stop                  /swcc-compound
```

- **Auto-generated knowledge tree**: give it a topic and it decomposes it into a structured knowledge tree + knowledge system, optionally anchored to a PDF/doc; **after generation it mandatorily dispatches reviewer sub-agents to find and fill gaps**. You can incrementally update an existing topic — **your learned progress is never lost**.
- **Teach first, test after — starting from first principles**: `/swcc-go` walks you through the material first — for complex mechanisms it **starts from the essence / design philosophy** (giving you the right lens before the machinery), then offers several directions to dig deeper; only **then** does it quiz you Socratically + with a Feynman check. Weak points are **recorded only from the test phase**, so "not yet reviewed" is never mistaken for a weakness.
- **Objective, system-scored mastery**: `/swcc-stop` dispatches an **independent sub-agent** that scores you on *coverage + depth* — not self-assessment — and schedules the next review on the Ebbinghaus `[1,2,4,7,15]`-day curve.
- **It learns how you learn (the compounding part)**: `/swcc-compound` talks with you about *how it went, where you struggled, and why*, and distills a profile of **how you learn** (explanation preferences, recurring blind spots, what guidance works). go/weak/mock then teach you in an increasingly tailored way — adjusting style only, never relaxing rigor.
- **Capture the output**: `/swcc-blog` turns a learning session/discussion into readable blog posts (saved to `./blogs/` by default, ready to publish); `/swcc-deep` expands a concept across domains.
- **Your data is yours**: review data lives in `~/.study-with-cc/`, independent of the plugin, so updates never wipe it.

## Install

```bash
claude plugin marketplace add guoqiaoZhou/study-with-claude-code
claude plugin install study-with-claude-code
```

Try locally (without installing): `claude --plugin-dir /path/to/study-with-claude-code`

## Commands (12)

> Not sure which to use? Just run **`/swcc-help`** — it picks the right command for you based on "I want to do X → use this".

**Core loop**

| Command | Usage | What it does |
|------|------|------|
| `/swcc-plan` | `[topic] [level] [--update]` | Generate/update a roadmap-grade knowledge tree + knowledge system (reviewer sub-agents fill gaps) |
| `/swcc-go` | `[topic] [study\|test] [concept\|scenario\|mixed]` | Teach-then-test: teach from first principles → Socratic quiz |
| `/swcc-stop` | `stop` | End & archive: independent sub-agent scores you, logs weak points, schedules the next review |
| `/swcc-stats` | `[topic]` | Progress snapshot: mastery ratio, weak points, streak, stats |

**Daily & multi-topic**

| Command | Usage | What it does |
|------|------|------|
| `/swcc-daily` | `[topic]` | What to review today (cross-topic due digest, read-only, cron-friendly) |
| `/swcc-switch` | `[topic]` | Switch between topics + overview |
| `/swcc-weak` | `[topic]` | Drill weak points specifically (pair with `/swcc-stop` to archive) |

**Advanced & capture**

| Command | Usage | What it does |
|------|------|------|
| `/swcc-mock` | `[topic] [level]` | Mock interview + independently scored report |
| `/swcc-compound` | `[topic]` | Reflect on your learning: distill "how you learn" so the system fits you better |
| `/swcc-deep` | `[topic] [concept]` | Expand a concept across domains (optional web lookup) |
| `/swcc-blog` | `[split hint] [out:dir]` | Turn this session/discussion into blog posts |

- `level`: `p6`/`p7`/`p8`/`p9` (depth; default `p7`)
- ⚠️ `go`/`weak` must be used in the **same conversation** as `stop` — `stop` reads the current conversation to archive it.

## Typical flow

```
/swcc-plan <topic> p7      # build the tree (it'll ask whether to attach a PDF/material)
/swcc-daily               # see what's due today
/swcc-go → answer → /swcc-stop   # teach-then-test → archive & score
/swcc-weak → /swcc-stop          # drill your weak points
/swcc-compound            # periodic reflection so the system fits you better
/swcc-blog 3              # write up what you learned as 3 blog posts
```

## Data storage

```
~/.study-with-cc/
├── config.json            # global config + topic registry
├── learner-profile.md     # learner profile (cross-topic: how you learn; maintained by compound, read by skills to tune style)
└── topics/<topic>/
    ├── knowledge-tree.md / knowledge-system.md / progress.json / references.json
    ├── review-sessions/   # go/weak archives    mock-sessions/  # mock interview reports
    ├── reports/           # compound reflections deep-notes/     # deep-dive notes
```

Blogs are written to your **current working directory** `./blogs/` by default (easy to publish), not inside the plugin's data dir. Reference materials are recorded by path only (not copied); PDFs are read via native paginated reading (zero dependencies).

## License

MIT
