---
name: architecture-grill
description: Grill the user about a repo's architecture, then write the C4 diagrams and ADRs it warrants. Use when documenting a repo's architecture, writing an ADR, or drawing a C4 diagram.
---

# Architecture Grill

Three phases in order: **recon**, **grill**, **write**. Every line written traces back to the repo (recon) or the user (grill).

**Prerequisite**: the `grilling` skill — `npx skills add mattpocock/skills@grilling`.

**Stack assumed**: .NET backend(s), containerized, on Azure Red Hat OpenShift (ARO); React + Vite SPA on Azure Static Web Apps. Where the repo differs, name the mismatch and adapt.

## Altitude

The word governing every judgement here. *Altitude* is how far down the detail goes, and the two failures are symmetric: too high states what any reader would guess, too low restates the code and config and goes stale within a sprint. Documentation earns its place in the band between — the **why** the code cannot carry.

Three altitudes are fixed for this stack:

- **Hosting** belongs in a diagram box label (`.NET, containerized`) or a boundary (`Azure Red Hat OpenShift`). OpenShift `Deployment`s, `Route`s and `Service`s fly below C4; they live in the manifests.
- **The frontend↔backend relationship** carries its protocol (`HTTPS/JSON`). Endpoints, params and schemas fly below C4 too; they live in the generated OpenAPI spec. The *policy* around the contract — versioning strategy, breaking-change rules — is ADR material; the contract's current shape never is.
- **Business requirements** sit outside this skill altogether: already documented, so link them.

## Phase 1 — Recon

Read the repo before asking the user anything the repo answers. `*.csproj`, `package.json`, `Dockerfile` and the `src/` layout are findable unaided; these are the reads whose *significance* is not self-evident:

- `staticwebapp.config.json` — whether `/api/*` routes to SWA **managed Functions** or straight to OpenShift. Managed Functions are a separate container with their own failure mode, which changes the Container diagram, so establish which it is.
- `appsettings*.json` — record each dependency's *kind* ("SQL Server", "Service Bus", "Redis"); leave the connection-string values out.
- `Program.cs` / `Startup.cs` — DI and `HttpClient` registrations are the real external-dependency list, usually longer than anyone remembers.
- OpenShift manifests or Helm charts (`deploy/`, `helm/`, `.openshift/` — naming varies) — what is actually deployed, and how it is exposed.
- Existing OpenAPI/Swagger setup — the contract's source of truth; its presence settles that endpoints stay out of what you write.
- `docs/adr/` — match the convention already in use rather than starting a second one.
- `docs/architecture/` — any diagram here is **living**: Phase 3 edits it in place.
- `CONTEXT.md` — the user's own vocabulary; reuse those exact terms.

Recon completes on a summary the user can correct: every service, datastore and external system found, each named.

## Phase 2 — Grill

Recon establishes what exists. The grill establishes why, which is the only part worth writing down. Run the `grilling` skill seeded with the recon summary, so it asks nothing the code already answered.

Without `grilling` available: ask the whole answerable set as one numbered round, each question carrying your best guess so the user confirms rather than composes; let the next round follow from the answers; stop when a round yields nothing new.

Cover these, skipping whatever recon settled:

1. **Shape** — why this service count, this datastore, this queue. What was migrated from what, and why.
2. **Integrations** — per external system from recon: critical or degradable, sync or async, who owns it.
3. **Contract policy** — versioning strategy, breaking-change rules, REST or otherwise, and the reasoning.
4. **Boundaries** — what this service deliberately does not own, and where that responsibility sits instead. Cross-team integration requests ride the org's existing process: name them and move on.
5. **Deployment** — why OpenShift over Azure PaaS (Functions, App Service, AKS): portability, a platform-team standard, on-prem parity, lock-in. And why SWA for the frontend.
6. **Invisible constraints** — compliance, SLAs, cost ceilings: whatever would read as a mistake to someone without the context.

Run each answer against the **litmus** in [`ADR-FORMAT.md`](ADR-FORMAT.md) as it arrives. An answer that fails the litmus still earns its keep by shaping the diagram — it just earns no ADR.

The grill completes when every service, datastore and external system from recon carries a *why*, or an explicit "nobody remembers" on the record.

## Phase 3 — Write

**Diagrams.** Levels, files, shapes and the flowchart mechanics live in [`C4-FORMAT.md`](C4-FORMAT.md). Pick the level from what the findings actually changed:

| Level | Draw or update it when |
|---|---|
| **Context** | recon or the grill surfaced an external system missing from it, or none exists yet |
| **Container** | nearly always — the solution-level view, and where most findings land |
| **Component** | the grill exposed one container as genuinely complex (an authz engine, an orchestrator); confirm with the user before drawing it |

**ADRs.** One per litmus-passing decision, following [`ADR-FORMAT.md`](ADR-FORMAT.md). Where this repo is one service inside a larger system (a `CONTEXT-MAP.md`, a sibling system repo, or ask), say which ADRs belong in that **system repo** instead: anything reaching past this one service — the frontend↔backend contract, an integration another service also depends on, security, compliance, cost. Hand those to the user for placement.

**Report**, in order:

1. The recon summary.
2. Each ADR, with its path, flagged where it belongs in the system repo.
3. Each diagram, with its path and a one-line reason that level was the right altitude.
4. Every grill finding the litmus rejected, each named, so the user sees what was weighed rather than missed.
