---
name: architecture-grill
description: Interview the user about this repo's architecture, then produce Context/Container/Component-level C4 diagrams and any ADRs that are actually warranted. Use when the user wants to document architecture, catch up documentation on a repo, or asks to "grill" them about architecture/design decisions.
---

# Architecture Grill

**Prerequisite**: this skill uses a `grilling` skill for Phase 2 — install it first if it isn't already (e.g. `npx skills add mattpocock/skills@grilling`). If it's unavailable for some reason, fall back to the inline instructions at the start of Phase 2.

Stack this skill assumes (adjust if the repo looks different): **.NET backend**, **React + Vite SPA frontend**, deployed as an **Azure Static Web App**. If the repo doesn't match, note the mismatch and adapt rather than forcing it.

This skill has three phases, run in order: **explore**, **grill**, **produce**. Don't skip to producing documents from assumptions — everything written must come from either the code (explore) or the user (grill), never invented.

## Phase 1 — Explore the repo first

Facts come from the repo, not from asking the user things you can find yourself. Look for:

**Backend (.NET)**
- `*.sln`, `*.csproj` — how many deployable services live here? One repo can hold more than one.
- `Program.cs` / `Startup.cs` — registered `HttpClient`s, DI registrations hinting at external dependencies (queues, caches, DB providers).
- `appsettings*.json` — connection strings (redact values, keep the *kind*: "SQL Server", "Service Bus", "Redis"), configured auth (Azure AD, JWT).
- `Controllers/` or minimal-API endpoint maps — rough shape of what this service exposes.
- Existing OpenAPI/Swagger setup — if present, that's the source of truth for the contract; don't re-document endpoints by hand.

**Frontend (Vite + React SPA)**
- `package.json` — confirm Vite/React, note any notable libs (state management, a design-system package from another team, GraphQL client, etc).
- `src/` — where it calls the backend (an `api/` or `services/` folder is typical), and whether it calls anything *besides* this repo's backend (other teams' APIs, third-party widgets).
- `vite.config.ts` — proxy rules often reveal what backend(s) it talks to in dev.

**Deployment (Azure Static Web App)**
- `staticwebapp.config.json` — routing rules, configured auth providers, and whether `/api/*` is proxied to **managed Azure Functions** (a separate deployable unit living alongside the SPA) vs. a fully separate backend. This materially changes the Container diagram — a managed-functions API is its own container even though it deploys alongside the SPA.
- Any `infra/`, `*.bicep`, or GitHub Actions / Azure DevOps pipeline files — confirms actual deployment topology rather than assuming it.

**Existing docs**
- `docs/adr/` — read what's already there. Match its numbering/format if one exists rather than introducing a second convention. Note the highest existing ADR to avoid collisions if sequential; if date-prefixed, no numbering to track.
- `docs/architecture/` or any existing diagram (Mermaid, draw.io, image) — if a Container diagram already exists, you'll be **updating it in place** in Phase 3, not redrawing from scratch.
- `CONTEXT.md` if present — domain vocabulary; use the same terms the user already uses for things.

Summarize what you found in a few lines before moving to Phase 2 — this grounds the interview and lets the user correct you early if you misread something.

## Phase 2 — Grill the user

Everything in Phase 1 tells you *what exists*. It doesn't tell you *why*, and why is the only thing worth writing down. Use the `grilling` skill to interview the user about this repo's architecture — feed it the Phase 1 summary as context so it doesn't re-ask anything already answered by the code, and steer it toward the categories below rather than a generic interview.

If `grilling` isn't installed: ask everything you can ask right now as one numbered batch, each with your best-guess recommended answer so the user can confirm or correct rather than write an essay; wait for their reply; ask the next round shaped by what you learned; stop once nothing new is left to ask.

Target these categories — skip any that Phase 1 already resolved or that clearly don't apply to this repo:

1. **Shape decisions**: Why this many services in this repo? Why this datastore/queue/cache? Was anything migrated from something else, and why?
2. **Integration decisions**: For each external system found in Phase 1 (including other teams' APIs/widgets) — is it critical or optional/degradable? Sync or async? Who owns it?
3. **Contract policy** (not the current endpoint list — that's generated, not asked about): versioning strategy, breaking-change policy, REST vs. something else, and *why*.
4. **Boundary decisions**: What does this service explicitly *not* own or do? What's deliberately out of scope here vs. handled elsewhere (including the org's cross-team integration-request process)?
5. **Deployment decisions**: Why Azure Static Web Apps specifically — e.g. managed Functions vs. a separately hosted API, why that split, any constraint that forced it (cost, team ownership, latency).
6. **Constraints invisible in code**: compliance, performance SLAs, anything that would look "wrong" to a future engineer without this context.

For each answer, silently run it against the litmus test before treating it as ADR material:

> **All three must be true**: (1) hard to reverse, (2) surprising without context, (3) the result of a real trade-off between genuine alternatives.

If an answer is just "that's the obvious/default choice," don't push for an ADR on it — move on. You're hunting for the decisions that would make a new hire go "huh, why?", not cataloguing every choice made.

## Phase 3 — Produce

### C4 diagrams — pick the level(s) the findings actually warrant

Don't default to one level. For each diagram, update the existing file in place if Phase 1 found one; otherwise create it fresh. Use **Mermaid `flowchart`, not `C4Context`/`C4Container`/`C4Component` syntax** throughout — Mermaid's native C4 renderer lays out badly (crossing lines) even on small diagrams; `flowchart` with labeled `subgraph` boundaries gives a correct auto-layout and still reads as C4. See `C4-FORMAT.md` for the exact pattern and worked examples at all three levels for this stack.

| Level | File | Draw/update it when |
|---|---|---|
| **Context** | `docs/architecture/context.md` | Phase 1/2 surfaced a new external system (another team's API, a third-party provider) not already on it, or this is the first run and none exists yet. Changes rarely. |
| **Container** | `docs/architecture/container.md` | Almost always worth producing/updating — this is the main "solution level" view: what talks to what across the SPA, managed Functions (if any), backend service(s), datastores, and external systems. |
| **Component** | `docs/architecture/components/{container-slug}.md` | Only for a container Phase 2 revealed to be genuinely complex (an authz engine, an orchestrator, similar). Don't produce one by default for every container — ask before drawing it if it's not obvious from the interview. |

Relationship labels carry the facts that matter: protocol (`HTTPS/JSON`), and criticality if it's not simply "required" (`optional, degrades gracefully if unavailable`).

### ADRs

For every decision from Phase 2 that passed the litmus test, write one ADR using the format in `ADR-FORMAT.md` (MADR-based). One file per decision, `docs/adr/{yyyy-mm-dd}-{short-slug}.md`. Status `accepted` by default — you're recording a decision already reached, not proposing one.

If this repo is a backend service in a larger system (Phase 1 will usually show this — check for a sibling "system repo" reference, a `CONTEXT-MAP.md`, or just ask), flag which of the drafted ADRs actually belong in the **system repo** instead of here: anything that trips a cross-cutting scope trigger (affects more than this one service, changes the frontend↔backend contract, new integration another service also depends on, security/compliance/cost impact). Don't silently commit those here — tell the user which ones to move.

### Output, in order

1. What Phase 1 found (brief).
2. The full grill transcript's conclusions are implicit in the ADRs — don't repeat them separately.
3. Each drafted ADR, with its target path, and a flag if it belongs in the system repo instead.
4. Each diagram produced or updated (Context/Container/Component as applicable), with its target path and a one-line reason it was the right level for what changed.
5. Anything from Phase 2 that did **not** pass the litmus test — name it briefly so the user knows it was considered and deliberately excluded, not missed.
