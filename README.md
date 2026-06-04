# Contract Copilot Engine

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[中文文档](README_ZH.md)

> AI-assisted contract review engine — framework is open source, domain knowledge is yours to fill.

Contract Copilot Engine is a contract review skill framework for AI coding assistants (Claude Code / Codex / Copilot CLI). It provides a multi-call pipeline architecture, contract-type routing, Playbook matching, and DOCX revision generation — without any industry-specific review standards or clause positions. Those are defined by you through structured Playbook JSON files.

## Core Philosophy

- **Engine open source** — pipeline architecture, routing logic, DOCX generation are free to use and modify
- **Domain knowledge filled by you** — clause standards, risk thresholds, approval tiers are defined in `config/playbooks/*.json`
- **Facilitate deals** — the goal is safe deal closure, not deal rejection

## Quick Start

### Prerequisites

- AI coding assistant (Claude Code / Codex / Copilot CLI)
- Python 3.9+
- Pandoc (for Markdown → DOCX conversion)

### Installation

```bash
git clone https://github.com/YOUR_USERNAME/contract-copilot-engine.git ~/.cc-switch/skills/legal/contract-copilot
cd ~/.cc-switch/skills/legal/contract-copilot
python3 -m pip install -r scripts/requirements.txt
```

### Configuration

1. Edit `config/reviewer_profile.json` with reviewer info
2. Create Playbook JSON files under `config/playbooks/` using `playbook.schema.json` structure

### Usage

Upload a contract file to your AI coding assistant; the skill auto-triggers:

```
Review this contract: /path/to/contract.docx
```

The engine executes 3 calls + DOCX output:
1. **Call 1 — Structure Scan**: contract type identification, Playbook matching, macro/meso-level checks
2. **Call 2 — Issue Identification**: clause-level comparison, structured issue list (P0/P1/P2)
3. **Call 3 — Deep Analysis**: conditional trigger, key pattern inspection + approval escalation + negotiation strategy
4. **DOCX Output**: track-changes version + review report

## Architecture

```
contract-copilot-engine/
├── SKILL.md                    # Routing entry point
├── references/                 # Review instructions
│   ├── call1-structure-scan.md
│   ├── call2-issue-identification.md
│   ├── call3-deep-analysis.md
│   ├── contract-routing.md     # 12-type contract routing
│   ├── docx-pipeline.md
│   ├── review-framework.md     # Macro/Meso/Micro framework
│   ├── revision-strategy.md
│   ├── priority-clauses.md
│   └── setup-dependencies.md
├── config/
│   ├── reviewer_profile.example.json
│   ├── review_memory.example.json
│   └── playbooks/
│       └── playbook.schema.json  # Schema definition (fill with your standards)
└── scripts/                    # Python scripts (DOCX track changes)
    ├── review/
    ├── docx/
    └── report/
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for details.

## Playbook System

The Playbook is the core extension point. Create one JSON per industry/contract type, defining:

- **Clause standards** — `standard[]` / `acceptable[]` / `redFlags[]`
- **Risk tiers** — Tier 1 (Deal-breaker) / Tier 2 (Negotiable) / Tier 3 (Optimization)
- **Approval escalation rules** — auto-escalation on deviation
- **Negotiation strategy** — fallback / commonCounter
- **Feedback corrections** — user corrections auto-written to `feedbackCorrections.records[]`, auto-injected on next review

See `config/playbooks/playbook.schema.json`.

## Customization

1. Create `contract-types/` directory with clause details per contract type
2. Create industry-specific Playbook JSONs under `config/playbooks/`
3. Define domain knowledge and review standards in AGENTS.md / CLAUDE.md

## Integrating Private Playbooks

This repo contains only the review engine. Place your private Playbook JSONs and contract-type files in your project directory, referenced via AI coding assistant project config:

```
your-project/
├── AGENTS.md                    # Domain knowledge (review standards)
├── config/playbooks/            # Private Playbook JSON (not in this repo)
│   ├── sale.json
│   ├── lease.json
│   └── ...
├── contract-types/              # Contract type files (not in this repo)
└── cases/                       # Contracts to review
```

## Author

**Luo Zhiqiang** — Real estate legal counsel, designer of the Contract Copilot system.

For commercial collaboration or customized Playbook development, contact via WeChat: **maoo8486**

---
## Contributing

Issues and PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT License — see [LICENSE](LICENSE).
