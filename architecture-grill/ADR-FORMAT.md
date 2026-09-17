# ADR Format

## The litmus

**All three** must hold, or the decision earns no ADR:

1. **Hard to reverse** — changing course later costs real time or real risk.
2. **Surprising without context** — a future reader asks "why on earth did they do it this way?"
3. **A real trade-off** — genuine alternatives existed, and one won for specific reasons.

An easily-reversed decision just gets reversed. An unsurprising one prompts no question. One with no alternative records nothing beyond "we did the obvious thing." Where an ADR fails the litmus, a line in the PR description carries it instead.

## What passes

- **Architectural shape** — "This service owns its SQL database; nothing else writes to it directly."
- **Integration patterns** — "The SPA calls this API synchronously; anything slower than 2s becomes a queued job."
- **Lock-in choices** — hosting platform, datastore, auth provider, message bus. The ones costing a quarter to swap, rather than every NuGet and npm package.
- **Boundaries** — what this service does not do, and where that responsibility actually sits.
- **Deliberate deviations** — anything where a reasonable reader would assume the opposite of what was done.
- **Invisible constraints** — compliance rules, partner SLAs, cost ceilings that shaped the design and left no trace in the code.
- **Non-obvious rejections** — GraphQL seriously considered and REST chosen for specific reasons, or the suggestion returns in six months.

## What fails

Business requirements (documented already — link them), anything readable straight from the code, the current API contract (the generated OpenAPI spec owns it; an ADR may own the *policy* around it), and cross-team integration requests (the org's own process owns those).

## The file

`docs/adr/{yyyy-mm-dd}-{short-slug}.md` — date-prefixed, so parallel branches never collide on a number. Where `docs/adr/` already runs a different convention, follow the one in use.

```md
---
status: accepted
date: {yyyy-mm-dd}
---

# {The decision itself, specific — not the topic it concerns}

## Context

{What forced this decision, and under what constraints. Enough that someone
with no memory of the discussion follows why it came up.}

## Decision

{What was decided, plainly, then enough detail to act on it.}

## Considered Options

{The rejected alternatives worth remembering, one line each. Where only one
path was ever reasonable, the section goes.}

## Consequences

**Good:**
- {a non-obvious upside}

**Bad:**
- {a non-obvious downside, or debt taken on}
```

A dense paragraph beats five sparse headings: sections with nothing non-obvious to say come out. `status` is `accepted` — these record decisions already reached, and `proposed` applies only where the user says this one is still open. A later decision replacing this one sets `superseded by ADR {filename}`, linked both ways.
