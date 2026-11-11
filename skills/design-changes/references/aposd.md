# APOSD Reference

Concepts from _A Philosophy of Software Design_ (John Ousterhout). The skill cites these by ID (e.g. `C2`, `D1`, `R3`) instead of re-explaining them.

---

## Complexity

Complexity is anything about the structure of a system that makes it hard to understand or modify. It accumulates in small increments, so every change either pays it down or adds to it.

| ID  | Symptom              | Meaning                                                                             |
| :-- | :------------------- | :---------------------------------------------------------------------------------- |
| C1  | Change amplification | A simple change needs edits in many places.                                         |
| C2  | Cognitive load       | A developer must hold a lot in their head to make a change.                         |
| C3  | Unknown unknowns     | It is not obvious what must change, or what a change will break. The worst symptom. |

| ID  | Cause        | Meaning                                                                            |
| :-- | :----------- | :--------------------------------------------------------------------------------- |
| K1  | Dependencies | Code cannot be understood or changed in isolation.                                 |
| K2  | Obscurity    | Important information is not obvious (vague names, hidden coupling, missing docs). |

## Design Principles

| ID  | Principle                              | Core Idea                                                                                                                                                                                        |
| :-- | :------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| D1  | Deep modules                           | Best modules give a lot of functionality behind a simple interface. Interface cost must be small relative to what it hides.                                                                      |
| D2  | Information hiding                     | Each module encapsulates design decisions (formats, algorithms, protocols). Leakage is when one decision is reflected in several modules.                                                        |
| D3  | Avoid temporal decomposition           | Structure by the knowledge each module needs, not by the order operations run (e.g., read + parse + write as separate modules often leak the same format).                                       |
| D4  | Somewhat general-purpose               | Interface general enough to serve current and near-future needs without special cases; implementation tuned to today's use. Ask: "What is the simplest interface that covers all current needs?" |
| D5  | Different layer, different abstraction | Adjacent layers should offer different abstractions. Pass-through methods and pass-through variables signal a layer adds nothing.                                                                |
| D6  | Pull complexity downwards              | It is better for a module's implementation to absorb complexity than to push it to every caller (via config knobs, exceptions, or required call sequences).                                      |
| D7  | Better together or better apart        | Combine code that shares information, simplifies the interface, or removes duplication. Separate general-purpose from special-purpose code.                                                      |
| D8  | Define errors out of existence         | Redefine semantics so error cases disappear; otherwise mask them low, aggregate them in one handler, or crash on truly unrecoverable ones.                                                       |
| D9  | Design it twice                        | Consider radically different designs before committing; the first idea is rarely the best.                                                                                                       |
| D10 | Comments describe what code cannot     | Interface comments state abstractions and contracts; implementation comments state why. Writing comments first exposes poor abstractions early.                                                  |
| D11 | Precise names                          | A name should create a clear, specific image. Hard-to-name things often have muddled responsibilities.                                                                                           |
| D12 | Consistency                            | Similar things done in similar ways lower cognitive load.                                                                                                                                        |
| D13 | Obvious code                           | A reader can understand behaviour quickly without deep study. Event-driven control flow, generic containers, and mismatched types/declarations hurt obviousness.                                 |
| D14 | Strategic programming                  | Optimise for long-term structure, investing a modest ongoing fraction of effort, rather than tactical "just make it work" patches.                                                               |

---

## Red Flags

| ID  | Red Flag                                                                    | Points To |
| :-- | :-------------------------------------------------------------------------- | :-------- |
| R1  | Shallow module – interface nearly as complex as implementation              | D1        |
| R2  | Information leakage – one decision encoded in several modules               | D2        |
| R3  | Temporal decomposition                                                      | D3        |
| R4  | Overexposure – common use requires knowing rare features                    | D1, D6    |
| R5  | Pass-through method or variable                                             | D5        |
| R6  | Repetition – same nontrivial code in several places                         | D7        |
| R7  | Special-general mixture – special-case logic inside a general mechanism     | D4, D7    |
| R8  | Conjoined methods – one cannot be understood without the other              | D7        |
| R9  | Comment repeats code                                                        | D10       |
| R10 | Implementation details in interface docs                                    | D2, D10   |
| R11 | Vague name                                                                  | D11       |
| R12 | Hard to pick a name                                                         | D11, D7   |
| R13 | Hard to describe – interface doc must be long to be complete                | D1        |
| R14 | Nonobvious code                                                             | D13       |
| R15 | Exception/error cases pushed to callers that could be handled or eliminated | D6, D8    |

---

## Agent Lens

The same properties that help humans help AI agents, often more strongly, because agents work within a limited context window and rely on search:

- **D1, D2**: An agent can read a small interface instead of a whole implementation, so fewer files enter context.
- **C3**: Hidden dependencies cause agents to make edits that compile but break distant behaviour.
- **D11, D12**: Precise, consistent names make `grep` and semantic search land on the right code.
- **D13, D10**: Obvious code and contract-level comments reduce wrong assumptions about behaviour.
- **C1**: Every extra edit site is another chance for an agent to miss one.
