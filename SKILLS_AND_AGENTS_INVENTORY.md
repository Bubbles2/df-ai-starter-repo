# Skills & Agents Inventory

> Inventory of all Claude skill and agent definitions found across the project folders in `C:\AI_Donncha\testClaude`.
> Generated 2026-06-16.
>
> **Excluded:** `node_modules` copies (npm packages, not yours) and duplicate `.claude/worktrees/...` copies.
> The top-level `.claude` folder holds only `settings.local.json` — no skills/agents.

---

## Agents

### `arcade-ged/.claude/agents/` and `arcade-ged-V1/.claude/agents/` (identical)
- **code-reviewer** — Read-only, evidence-based code reviewer. Triggers after a chunk of work or on request for bug/security/performance/regression review. *(tools: Read, Grep, Glob, Bash; model: opus)*

### `ArcMob/.claude/agents/`, `ArcMobWord/.claude/agents/`, `sub_agent_test/.claude/agents/`
- **DocsExplorer** — Documentation lookup specialist; fetches docs for libraries/frameworks in parallel. *(WebFetch, WebSearch, Skill, MCPSearch; sonnet)* — *present in ArcMob, ArcMobWord, oldstuff/MobArc_Lp*
- **frontend-ui-engineer** — Build/refactor/review React + Next.js + Tailwind UI components; accessibility, responsive design. *(sonnet)*
- **senior-code-reviewer** — Thorough quality/architecture/security/performance review after a feature, fix, or refactor. *(opus)*
- **test-engineer** — Writes/reviews/validates tests across unit, integration, and e2e layers. *(Read, Write, Edit, Bash, Glob, Grep)*

*Note: `ArcMob` and `sub_agent_test` lack DocsExplorer; `oldstuff/MobArc_Lp` has only DocsExplorer.*

### `TestOrchestrator/.claude/agents/` (coordinated CRM-app agent team)
- **dev-orchestrator** — Entry point for all dev tasks; coordinates the other agents in sequence with built-in testing and review. *(sonnet)*
- **backend-architect** — Scaffolds/implements Express REST APIs with Axios-compatible responses, per CLAUDE.md. *(sonnet)*
- **frontend-engineer** — React components, Redux Toolkit slices, Material UI, React Router v7. *(sonnet)*
- **code-reviewer** — Audits code for correctness/quality/security and adherence to project standards. *(sonnet)*
- **playwright-test-engineer** — Designs/implements/reviews Playwright e2e, UI, and integration tests. *(sonnet)*

### `erp-dashboard/.claude/agents/`
- *Empty* — folder exists but contains no agent definitions.

---

## Skills

### `arcade-ged/.claude/skills/`, `arcade-ged-V1/.claude/skills/`, `superpower/mobile1/.claude/skills/` (identical — React + Express + Sequelize + PostgreSQL stack)
- **api-structure** — Scaffold a new API domain in the Express/Sequelize/PostgreSQL backend.
- **clean-typescript** — Write clean, correct TypeScript; types, signatures, state modelling.
- **code-review** — Read-only, evidence-based code review (never modifies files). *(includes `code-review/agents/openai.yaml` config)*
- **modern-browser-apis** — Prefer native browser APIs over libraries for size/perf.
- **modern-react-components** — Build modern React 18+ components, avoid state/useEffect pitfalls.
- **react-web-express-starter** — Bootstrap a fresh React (web) + Vite project with an optional Express + (Sequelize or pg) + axios backend.
- **web-security** — Enforce baseline web security across trust boundaries.

### `ArcMob/.claude/skills/` and `ArcMobWord/.claude/skills/` (React Native / Arcade stack)
- **arcade-api** (`arcade_api`) — Reusable JS/TS service modules connecting UI to authenticated Arcade HTTP APIs.
- **clean-typescript** — Clean, efficient TypeScript best practices.
- **modern-accessible-html-jsx** — Clean, accessible, semantically correct HTML & JSX.
- **modern-best-practice-react-components** — Modern React components avoiding common pitfalls.
- **npm-package-manager** — Safe npm installs/updates/audits, lockfile & supply-chain discipline.
- **react-npm-compat** — Check whether an npm package is compatible before installing.
- **web-security** — Enforce web security / avoid vulnerabilities.

**ArcMobWord adds two more:**
- **digitech-design** — Digitech-branded UI/assets: colors, type, fonts, UI kit for prototyping/production.
- **docx** — Create/read/edit/manipulate Word `.docx` documents.

### `oldstuff/MobArc_Lp/.claude/skills/`
- Subset of the ArcMob set: **clean-typescript**, **modern-accessible-html-jsx**, **modern-best-practice-react-components**, **npm-package-manager**, **react-npm-compat**, **web-security**.

### `arcade-ged-prompted_experiment/.claude/skills/`
- **aa** (`modern-best-practice-react-native-expo-components`) — Build clean, modern React Native Expo components avoiding pitfalls. *(folder is named `aa`)*

### `sub_agent_test/.claude/skills/`
- **cmd-dev-manage** — Orchestration-only command coordinating frontend-ui-engineer, senior-code-reviewer, and optionally test-engineer agents (writes no code itself). *(slash-command style skill; arg hint `[test]`)*

### `MyAgents/fitness-landing/` (skills live in **three** parallel folders: `.claude/skills`, `.agents/skills`, `.augment/skills`)
- **gemini-image-gen** *(in `.claude/skills` only)* — Generate images via the Google Gemini API (gemini-2.5-flash-image).
- **vercel-react-best-practices** *(in `.agents/skills` and `.augment/skills`)* — React/Next.js performance optimization guidelines from Vercel Engineering. ~70 `rules/` sub-files (rendering, re-render, bundle, server, async, JS micro-opts).
- **vercel-react-native-skills** *(in `.agents/skills` and `.augment/skills`)* — React Native & Expo performance best practices (list perf, animations, native modules). ~45 `rules/` sub-files.

---

## Summary by location

| Project | Agents | Skills |
|---|---|---|
| `arcade-ged` / `arcade-ged-V1` | code-reviewer | 7 (React+Express+Sequelize+PG) |
| `arcade-ged-prompted_experiment` | — | 1 (`aa`) |
| `ArcMob` | 4 (frontend-ui, senior-reviewer, test-eng, DocsExplorer) | 7 |
| `ArcMobWord` | 4 (same as ArcMob) | 9 (+digitech-design, +docx) |
| `erp-dashboard` | — (empty) | — |
| `MyAgents/fitness-landing` | — | 3 (gemini + 2 Vercel skills, mirrored across 3 dirs) |
| `oldstuff/MobArc_Lp` | 1 (DocsExplorer) | 6 |
| `sub_agent_test` | 3 (frontend-ui, senior-reviewer, test-eng) | 1 (cmd-dev-manage) |
| `superpower/mobile1` | — | 7 (same as arcade-ged) |
| `superpower/mobile2` | — (settings only) | — |
| `TestOrchestrator` | 5 (orchestrated CRM team) | — |

### Patterns
- The **arcade-ged / arcade-ged-V1 / superpower/mobile1** skill sets are identical (one web-stack template copied around).
- The **ArcMob / ArcMobWord / sub_agent_test** agent sets are largely identical.
- `TestOrchestrator` is the only project with a full multi-agent orchestration team.
- `MyAgents/fitness-landing` is the only one using `.agents` / `.augment` folders alongside `.claude`.
