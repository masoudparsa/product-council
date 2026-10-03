# Product Council

A structured multi-agent Product + Technical Decision Council for Claude Code.

## How it works

The council does not simply collect opinions.

It uses:

1. Independent analysis
2. Evidence classification
3. Cross-examination
4. Steelman of opposing arguments
5. Adversarial Red Team review
6. Technical feasibility gate
7. Structured debate
8. Concession/change tracking
9. Chairman synthesis
10. Final decision stress test

The goal is better reasoning, not consensus.

## Council Members

- Product Manager
- User Advocate
- UX Expert
- Business Strategist
- Growth & Go-to-Market (marketing, distribution, positioning; sales lens for B2B/partner decisions)
- Data Analyst
- Technical Architect
- Security/Risk
- Red Team
- Product Chairman

## Modes

| | quick | standard (default) | deep |
|---|---|---|---|
| Members | PM, UX, Tech, Red Team + Chairman | all 9 + Chairman | all 9 + Chairman |
| Debate rounds | 0 (cross-exam only) | 1 | 2 |
| Feasibility gate | if implementation | if implementation | always |
| Experiment design | — | if Test first | always |
| Approx. agent calls | ~9–10 | ~20–27 | ~31–39 |

Quality has priority over cost: the skill cuts waste (filler, duplicate calls, repeated protocol text), never thinking (roles, challenges, context). Details in `skills/product-council/SKILL.md`.

## Structure

```text
agents/                         subagents (frontmatter: name, description, tools, model)
skills/product-council/
  SKILL.md                      orchestration
  debate-protocol.md            shared rules injected into rounds 2–3
  templates/decision-report.md  quick + full report templates
```

Chairman, Red Team, and Technical Architect run on `opus` (the highest-leverage reasoning roles); the other members run on `sonnet`. For maximum quality regardless of cost, set every `model:` to `opus`.
Technical Architect is read-only (`Read, Grep, Glob`). Business Strategist and Growth & Go-to-Market can use web search for real market and channel data.

## Installation

### Option A — Plugin (recommended)

In Claude Code:

```text
/plugin marketplace add <github-username>/product-council
/plugin install product-council@product-council
```

Update later with `/plugin marketplace update product-council`.

When installed as a plugin, the command is namespaced: `/product-council:product-council`.

### Option B — Manual copy

Copy into your global Claude folder (`%USERPROFILE%\.claude\` on Windows, `~/.claude/` on macOS/Linux):

```text
agents/*                 -> ~/.claude/agents/
skills/product-council/  -> ~/.claude/skills/product-council/   (including debate-protocol.md and templates/)
```

Then the command is `/product-council`.

## Usage

```text
/product-council
```

```text
/product-council quick
```

```text
/product-council standard
```

```text
/product-council deep
```

Example:

```text
/product-council deep

Should we let users share their data with a partner account?
```

The Technical Architect should inspect the actual repository when the decision concerns implementation.

The council informs the human decision-maker; it does not replace human product ownership.
