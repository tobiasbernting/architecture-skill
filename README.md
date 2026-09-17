# architecture-skill

Agent skills for architecture documentation, installable with [`npx skills`](https://skills.sh/).

## Skills

### `architecture-grill`

Interviews you about a repo's architecture, then produces Context/Container/Component-level C4 diagrams (Mermaid) and any ADRs the interview actually warrants. Tuned for a .NET backend + React/Vite SPA deployed as an Azure Static Web App, but adapts if the repo looks different.

Requires the `grilling` skill (falls back to an inline interview loop if unavailable):

```
npx skills add mattpocock/skills@grilling -y
npx skills add tobiasbernting/architecture-skill@architecture-grill -y
```
