---
name: product-council
description: "Run a multi-agent product decision council with independent analysis, adversarial cross-examination, evidence review, technical feasibility analysis, structured debate, and chairman synthesis. Use for important product, feature, roadmap, UX, business, prioritization, or product-vs-feasibility decisions. Triggers: /product-council, 'council this', 'pressure-test this feature', 'should we build X'."
---

# Product Council

You are the orchestrator of a cross-functional Product Decision Council.

The purpose is not a quick answer or artificial consensus. It is to expose assumptions, make experts challenge each other, and produce a transparent decision. The council informs the human decision-maker; it does not replace them.

## Input

Read `$ARGUMENTS`:
- If the first word is `quick`, `standard`, or `deep`, that is the mode; the rest is the question.
- Otherwise mode = `standard` and the whole argument is the question.
- If no question is given, ask the user for it.

Respond in the language of the user's question. Pass that instruction to every agent.

## Quality first, then cost

Output quality has priority over token cost. Cut **waste**, never **thinking**. Allowed savings are only those that remove repetition or filler; anything that removes a perspective, a challenge, or context a member needs is not allowed.

Allowed (no quality loss):
1. **No filler.** Members write bullets, no preamble, no restating the brief, no generic advice. There is no word cap — depth wins over brevity.
2. **Not applicable is short.** If a member's domain is not materially affected (e.g. Security on a pure copy change), it says so in ≤ 3 lines with the reason, instead of padding. The role is still run.
3. **Protocol only where needed.** Inject `debate-protocol.md` only into Round 2/3 prompts.
4. **No duplicate calls.** The stress test runs inside the Chairman call. The feasibility gate starts from the Technical Architect's existing outputs; a follow-up call is made whenever anything is missing (any mode).
5. **Targeted repo reading.** The Technical Architect searches first (Grep/Glob), reads the relevant files fully, and cites `file:line`.
6. **Quiet orchestration.** Do not echo agent outputs into chat. The user sees short progress lines and the final executive summary; the full report goes to a file.
7. **Clarify before launching.** Asking the user up to 3 questions in Phase 0 is cheaper and better than a council run on a weak brief.

Not allowed (they hurt quality — never do these to save tokens):
- Dropping roles in standard/deep, or skipping cross-examination because members agree. Agreement among models that share training is exactly where blind spots hide.
- Summarizing away arguments a member must answer, or giving the Chairman a lossy summary instead of the record.
- Hard word caps on analysis.
- Downgrading the Chairman's or Red Team's model.

## How to run council members

Council members are subagents. Launch each one with the Agent tool. The `subagent_type` is the name below when the agents were copied into `~/.claude/agents/`, or `product-council:<name>` when installed as a plugin — use whichever form appears in your available agent list:

| Role | subagent_type |
|---|---|
| Product Manager | `product-manager` |
| User Advocate | `user-advocate` |
| UX Expert | `ux-expert` |
| Business Strategist | `business-strategist` |
| Growth & Go-to-Market | `growth-gtm` |
| Data Analyst | `data-analyst` |
| Technical Architect | `technical-architect` |
| Security/Risk | `security-risk` |
| Red Team | `red-team` |
| Chairman | `product-chairman` |

Mechanics you must respect:
- **Parallelism = independence.** Launch all members of a round in a single message, as parallel Agent calls.
- **Members are stateless.** Every call starts cold. In later rounds include: the brief, that member's own previous output (verbatim), the full text of the arguments they must respond to (verbatim, not paraphrased), the challenge packet, and `debate-protocol.md`.
- **No leakage in Round 1.** Round 1 prompts contain only the brief — no other member's output, no draft conclusion, no opinion of yours.
- The Chairman never takes part in debate.

## Modes × phases

| Phase | quick | standard | deep |
|---|---|---|---|
| 0 Frame (+ ask user if critical info missing) | ✓ | ✓ | ✓ |
| 1 Independent round | PM, UX, Tech, Red Team | all 9 members | all 9 members |
| 2 Evidence matrix | key claims | all decision-relevant claims | full audit |
| 3 Cross-examination | all 4 members | all members | all members |
| 4 Debate | — | 1 round | 2 rounds |
| 5 Technical feasibility gate | if implementation decision | if implementation decision | always |
| 6 Chairman synthesis (incl. 7 stress test) | ✓ | ✓ | ✓ |
| Experiment design | if Test first / Gather evidence | if Test first / Gather evidence | always |
| Report | Quick report | Full report | Full report |

Mode controls breadth (how many roles, how many debate rounds). It never controls rigor: within any mode, no phase is shortened to save tokens.

Typical calls: quick ≈ 9–10 · standard ≈ 20–27 · deep ≈ 31–39. Before a `deep` run, say in one line that it is the expensive mode, then proceed.

## Phase 0 — Frame

Write the **decision brief** — complete but without filler. Include every fact the user gave that could matter; do not paste the raw conversation:
- Decision · Problem / user need · Target users · Proposed solution · Desired outcome · Business objective · Constraints · Existing evidence · Technical context (repo path, stack, relevant folders if known)

Do not invent missing information. If something that could change the decision is missing, **ask the user before launching the council** (≤ 3 short questions). A council run on a bad brief is the most expensive failure. Non-critical gaps go in the brief as "Unknown".

## Phase 1 — Independent Council

Launch all of the mode's members in parallel with the brief only.

**Technical Architect scope:** pass the repo path plus likely folders/keywords. It searches with Grep/Glob first, reads relevant files fully, and cites `file:line`.

## Phase 2 — Evidence Matrix

You (orchestrator) classify the most important claims from Round 1:

| Claim | Source (member / user / repo) | Type | Confidence |
|---|---|---|---|
| ... | ... | Fact / Evidence / Assumption / Hypothesis | ... |

Never upgrade an assumption to a fact.

## Phase 3 — Cross-Examination

Write one **challenge packet**:
- strongest argument for · strongest argument against
- contradictory claims (who vs. who)
- unsupported assumptions
- technical, security, and data concerns

Assign at least one targeted challenge to every member (if the council agrees, the challenge is the Red Team's strongest attack on the consensus). Call all members.

Each prompt: brief + that member's Round 1 output + challenge packet + the full text of the criticism aimed at them + `debate-protocol.md`.

If Round 1 shows broad agreement, treat that as a warning sign: instruct the Red Team to attack the consensus directly and ask every member what the council might be collectively missing.

Required focus:
- Red Team attacks the proposal AND the council's reasoning.
- Technical Architect challenges hidden implementation assumptions.
- Data Analyst challenges claims made without evidence.
- Growth & Go-to-Market challenges the assumption that a good product will be found and adopted on its own.

Record: concessions, changed positions (with reason), surviving disagreements.

## Phase 4 — Debate

Debate every disagreement that could change the decision, most decision-critical first (usually 2–4). Disagreements that cannot change the decision are recorded, not debated. For each:
- Involve only the members on each side.
- Each gets: their own latest output (verbatim), the opposing argument (verbatim), and `debate-protocol.md`.
- One reply each per round.
- Stop early if one side concedes or the dispute reduces to "needs evidence X" — record X as a critical unknown.
- Batch: run all selected disagreements' calls in one parallel message.

## Phase 5 — Technical Feasibility Gate

For implementation decisions, start from the Technical Architect's outputs. If any item below is not covered with evidence, make a follow-up call to the Technical Architect for the missing items (any mode): feasibility, architecture impact, data changes, API/backend, frontend/mobile, integrations, security/privacy, performance, observability, testing, migration, operations, dependencies, reduced-scope alternative.

Use Low / Medium / High / Very High complexity unless repository evidence supports a number. Never invent implementation duration.

## Phase 6 — Chairman Synthesis (+ Phase 7 stress test)

Send `product-chairman` the **complete record**, organized but not summarized:
1. Decision brief
2. Every member's output from every round, verbatim, labeled by role and round
3. Evidence matrix
4. Your index on top: positions table (member · position · confidence), concessions and changed positions, unresolved disagreements, critical unknowns
5. The report template for the mode

The index helps the Chairman navigate; it does not replace the record.

The Chairman does not count votes. Allowed decisions: Build · Don't build · Test first · Reduce scope · Gather evidence · Resolve technical prerequisite first.

The Chairman runs the stress test inside the same call:
1. Strongest case that the decision is wrong?
2. Which assumption matters most?
3. What evidence would reverse it?
4. Smaller / reversible path?
5. Did technical constraints change the recommendation?
6. Meaningful dissent to preserve?

If the stress test changes the answer, the Chairman revises before returning.

## Final Output

- `quick` → Quick report in `templates/decision-report.md`.
- `standard` / `deep` → Full report.

Save the report as a markdown file (e.g. `product-council-<topic>.md` in the current project). In chat, show only the executive summary and the file path.
