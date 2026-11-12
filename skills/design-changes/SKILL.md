---
name: design-changes
description: Interactive architecture review grounded in "A Philosophy of Software Design" (APOSD). Interviews the user about requirements, constraints, and context the code cannot show, then proposes at most 6 high-impact design changes that reduce complexity so the code is easier for humans and AI agents to understand and modify. Each change is designed twice with a recommendation and illustrative code. Use when the user asks to design code changes, improve architecture, reduce complexity, or runs /design-changes.
argument-hint: "[path, module, or feature to review]"
disable-model-invocation: true
---

# Design Changes (APOSD)

## Goal

Reduce complexity — so the system is easier to understand and modify for both humans and AI agents. "Complexity", its symptoms, causes, principles, and red flags are defined once in [references/aposd.md](references/aposd.md) and cited here by ID (`C*`, `K*`, `D*`, `R*`). Read that file before Phase 2.

## Ground rules

These apply to every phase.

1. **Read-only.** Do not modify code unless the user asks for implementation after the report.
2. **Evidence.** Every finding cites `path:line`. No finding without evidence.
3. **Cite, don't re-teach.** Refer to concepts by ID plus at most one clause; never paste definitions from the reference.
4. **Questions are for inputs, not decisions.** Ask about requirements, constraints, and facts the code cannot show. Never ask the user to pick a design, an alternative, whether to make a specific change, or to choose between options APOSD already decides — choosing designs is this skill's job.
5. **Say it once.** Each fact, snippet, and rationale appears once in the report. Later mentions point to it by label (e.g., "see S2.0", "per P3").
6. **At most 6 changes.** Fewer is better than padding. Rank by impact.

## Workflow

### Phase 1 — Scope

Use the argument as scope. If none was given, or the scope is a whole large repo, ask one question to confirm the boundary (folder, feature, or service).

### Phase 2 — Survey (silent)

Explore the scoped code before asking anything, so questions are specific to this codebase. Build working notes (not shown to the user):

- Module map: main modules, their interfaces, and dependency edges.
- Key flows: 2-3 representative paths through the code.
- Hotspots: files changed most often, if git history is available, e.g.:
  `git log --since="12 months ago" --name-only --format= -- <scope> | sort | uniq -c | sort -rn | head -20`
- Candidate findings: places matching red flags `R*`, each with evidence.

### Phase 3 — Interview

Ask one round of at most 15 questions. Ask only what the survey could not answer and what would change the ranking or designs; fewer is better - the cap is a limit, not a target. Ask a second round of at most 5 questions only if first-round answers open a gap that blocks Phase 4. Use a structured question tool if available; otherwise a single numbered message. Ask constraint, convention, and design-freedom questions as multiple choice with free form allowed, marking a sensible default inferred from the survey. Ask pain, surprise, and upcoming-work questions as open free text with no default. Pick the most relevant categories from the bank below and write concrete questions using real names from this codebase.

| Category           | What to learn                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pain & surprises   | Which changes feel slow or risky; what broke unexpectedly after a change; what you must remember when touching `X`; which areas you distrust despite tests (`C2`, `C3`).                                                                                                                                                                                                                                                                      |
| Upcoming work      | Concrete features or variations expected in the next 6-12 months (drives Frequency and `D4` scope)                                                                                                                                                                                                                                                                                                                                            |
| Constraints        | Public APIs and error behaviours other systems rely on (these block `D8`); performance budgets; fixed frameworks or platforms.                                                                                                                                                                                                                                                                                                                |
| Migration appetite | Incremental refactors only, or larger restructures acceptable; how much effort to invest (`D14`).                                                                                                                                                                                                                                                                                                                                             |
| Design freedom     | Which current choices (libraries, patterns, paradigms, module boundaries - name them from the survey) are mandated vs. team standard vs. historical/incidental; which new approaches are admissible (new dependencies, different paradigms, adopting or dropping a library/framework, changing code owned by other teams); tolerance for techniques the team hasn't used. Build options from survey findings; never ask which design to pick. |
| Conventions        | Rules mandated by lint, ADRs, org guidelines, or frameworks; error-reporting style (exceptions vs. result types) when the survey finds a mix (`D12`).                                                                                                                                                                                                                                                                                         |
| Survey checks      | Confirm or refute specific hypotheses from the survey: dead code, intentional duplication, deliberate module splits.                                                                                                                                                                                                                                                                                                                          |
| Out of scope       | Code being rewritten, deprecated, or already slated for refactor.                                                                                                                                                                                                                                                                                                                                                                             |

**Translate decisions into facts.** If you are tempted to ask about a specific change, ask about the fact that would decide it instead:

| Don't ask                                                                | Ask instead                                                                                                            |
| :----------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| "Should I merge `OrderService` into `OrderRepository`?"                  | "Is the `OrderService`/`OrderRepository` split required by a framework or team rule?"                                  |
| "Option A (result types) or option B (exceptions)?"                      | "Do other services depend on the exceptions `OrderService` throws today?"                                              |
| "Should we replace the `BaseHandler` inheritance tree with composition?" | "Is `BaseHandler` inheritance mandated, a team standard, or historical? Which alternative paradigms would you accept?" |

Record answers as a numbered **Context Profile** (`P1`, `P2`, …), tagging each item **[hard]** (a constraint no design may violate) or **[context]**. Record mandated design choices and inadmissible approaches as **[hard]**. Record incidental or historical choices as **[context]** and note that they may be removed. Unanswered items with a default take it and are marked "(default)"; unanswered items without a default are omitted.

### Phase 4 — Diagnose and rank

1. For each candidate finding, record: symptoms (`C*`), cause (`K*`), red flags (`R*`), evidence.
2. Group findings that share a root cause into one change; fix the cause, not each symptom.
3. Drop changes that violate a **[hard]** item from the Context Profile, require an approach the user marked inadmissible, target out-of-scope code, or rest on a survey hypothesis the user refuted.
4. Score each change: **Impact = Reach × Frequency × Severity**, each 1-3.
   - Reach: how many modules/files the problem touches.
   - Frequency: how often that code changes (churn + upcoming work from the interview).
   - Severity: 1 = mild `C2`, 2 = strong `C1`/`C2`, 3 = `C3` or hurts agents per the Agent lens.
   - Tie-break by lower migration cost.
5. Keep the top 6 or fewer.

### Phase 5 — Design it twice

For each selected change, apply `D9`:

1. Produce 2-3 alternatives that differ fundamentally - different module boundaries or interface shapes, not parameter tweaks. At least one should be a bold option that uses the widest design freedom the user granted (`P*`); if it goes beyond that freedom, label it "stretch" in the Designs table and never recommend it.
2. Evaluate every alternative on these **criteria**:
   - Interface simplicity / depth (`D1`)
   - Information hiding (`D2`)
   - Complexity removed (`C1`-`C3`)
   - Respects constraints and conventions (`P*`)
   - Migration cost and risk
3. Recommend one. The rationale cites criteria and `P*` IDs.

### Phase 6 — Report

Produce the report using the template below, then offer next steps in one line: implement a change, deep-dive one, or re-rank with new constraints.

## Report template

````markdown
# Design Review: <scope>

<2-3 sentences: the dominant source of complexity and the expected outcome.>

| #   | Change             | Concepts | Impact | Cost | Recommended |
| --- | ------------------ | -------- | ------ | ---- | ----------- |
| 1   | <imperative title> | R2, D2   | 18     | M    | B: <name>   |

## Context Profile

- P1 [hard]: <constraint or context>

## Change 1: <imperative title>

**Problem** - <symptoms and cause in 1-3 sentences, citing `path:line`.>

**S1.0 Current**

```<lang>
<real code, trimmed to ≤20 lines, `...` marks elisions>
```

**Designs**

|                                 | A: <name>                  | B: <name>                  |
| ------------------------------- | -------------------------- | -------------------------- |
| Idea                            | <one line>                 | <one line>                 |
| Interface                       | `<signature-level sketch>` | `<signature-level sketch>` |
| <one row per Phase 5 criterion> | ✓ / ~ / ✗ + ≤8 words       | …                          |

**Recommended: B** - <rationale citing criteria and P-ids; do not restate the Problem.>

**S1.1 Recommended sketch**

```<lang>
<new interface plus one call site, ≤25 lines>
```

**Migration** - <ordered steps, each leaving the system working.>
**Risks** - <only non-obvious ones; omit the line if none.>

## Considered, not ranked

- ≤3 one-line findings that missed the cut, with why.>
````

### Snippet Rules

- `S<n>.0` is real code from the repo; `S<n>.1` is the recommended design only. Non-recommended alternatives appear as signatures inside the Designs table, never as full snippets.
- Never show the same code twice. If a later change touches code already shown, reference its label.
- Snippets show interfaces and call sites, not full implementations.
- Comments inside snippets only for what the code cannot show itself.
