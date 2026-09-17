# ADR Format

ADRs live in `docs/adr/`, one file per decision: `docs/adr/{yyyy-mm-dd}-{short-slug}.md`. Date-prefixed, not sequentially numbered — avoids collisions across parallel branches/repos.

If `docs/adr/` already exists with a different convention (sequential numbers, a different template), match the existing one instead of introducing a second standard.

## Template

```md
---
status: accepted
date: {yyyy-mm-dd}
---

# {Short, specific title — the decision itself, not the topic}

## Context

{What situation forced this decision? What constraints applied? Enough for
someone with no memory of the discussion to understand why this came up.}

## Decision

{What was decided, stated plainly, followed by enough detail to act on it.}

## Considered Options

{Only if the rejected alternatives are worth remembering — omit entirely if
there was really only one reasonable path.}

## Consequences

**Good:**
- {non-obvious upside}

**Bad:**
- {non-obvious downside / debt taken on}
```

Keep it short — a dense paragraph beats five sparse headings. Omit any section that has nothing non-obvious to say; don't fill sections just because the template has them.

`status` is `accepted` by default: these are written up after a decision has already been reached, not proposals awaiting approval. Use `proposed` only if the user says this one genuinely isn't settled yet. Use `superseded by ADR {filename}` when a later decision replaces this one — link both directions.

## When to write one

**All three** must be true, or skip it:

1. **Hard to reverse** — meaningful cost to changing this later.
2. **Surprising without context** — a future reader would wonder "why did they do it this way?"
3. **Result of a real trade-off** — genuine alternatives existed and one was picked for specific reasons.

If a decision is easy to reverse, skip it — it'll just get reversed. If it's not surprising, nobody will wonder why. If there was no real alternative, there's nothing to record beyond "we did the obvious thing."

## What qualifies (examples)

- **Architectural shape** — "This service owns its own SQL database; nothing else writes to it directly."
- **Integration patterns** — "The SPA calls the managed Functions API synchronously; anything slower than 2s runs as a queued background job instead."
- **Technology/lock-in choices** — hosting platform, datastore, auth provider, message bus. Not every NuGet/npm package — only the ones that would take real effort to swap out.
- **Boundary decisions** — what this service explicitly does *not* do, and where that responsibility actually lives instead.
- **Deliberate deviations** — anything where a reasonable reader would assume the opposite of what was actually done.
- **Invisible constraints** — compliance requirements, partner SLAs, cost ceilings that shaped the design but leave no trace in the code.
- **Non-obvious rejections** — if GraphQL was seriously considered and REST was chosen for specific reasons, write it down, or someone proposes GraphQL again in six months.

## What doesn't qualify

- Business requirements — already documented elsewhere; link, don't restate.
- Anything readable directly from the code.
- The current API contract (endpoints/params/schemas) — that belongs in a generated OpenAPI/schema spec, not an ADR. An ADR can record the *policy* around the contract (versioning strategy, breaking-change rules) — never the current shape of it.
- Cross-team integration requests — handled by the org's existing integration-request process, not an ADR.
