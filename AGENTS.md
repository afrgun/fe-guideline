# AGENTS.md — maintaining this pack

This repository is the **source** of a stack-agnostic AI guideline pack for frontend teams. It is not an application. Target projects copy this repo's content into a `guideline/` folder; `AGENTS.template.md` / `CLAUDE.template.md` become their root `AGENTS.md` / `CLAUDE.md`.

These instructions are for agents **editing the pack itself**. Do not follow `AGENTS.template.md` as if this were a target project.

## Layout

| Path | Purpose | Copied to targets? |
|---|---|---|
| `AGENTS.md`, `CLAUDE.md` | This file — maintainer instructions | No |
| `AGENTS.template.md`, `CLAUDE.template.md` | Root instructions for target repos | Yes (renamed) |
| `stack-profile.template.md` | Per-project facts template | Yes |
| `references/{nextjs,react,angular}.md` | Stack methods + code templates, same section numbering | Yes |
| `skills/tier-{1,2,3}/<bp-skill>/SKILL.md` | Skill source of truth (rules) | Yes (synced flat into agent folders) |
| `skills/tier-{1,2,3}/<bp-skill>/examples.md` | Worked examples for that skill — decisions and output formats, fictional `topup` feature, framework-free; stack code belongs in `references/` | Yes (synced with the skill) |
| `agents/*.md` | Reviewer subagents | Yes (synced into `.claude/agents/`) |
| `lint/` | ESLint boundaries config (v7) | Yes |
| `scripts/sync-skills.mjs` | Copies skills into agent folders of a target repo; `--check` for CI | Yes |
| `scaffold/` | Starting formats for project-owned files (page-map, second-brain, api-contracts, ASSUMPTIONS, `layer-agents/src/**` per-folder AGENTS.md) | Yes — copied once; the project's copies are never overwritten on upgrade |
| `hooks/` | Agent hooks (guard synced copies, lint edited file) + example settings | Yes |
| `PROMPTS.md` | Prompt cheat sheet for humans | Yes |
| `SKILLS.md` | Per-skill explanation for humans (purpose, contents, example prompt) | Yes |

Per-folder `scaffold/layer-agents/**/AGENTS.md` files are short local reminders that point to the owning skill — never restate a full rule table there; change the skill instead.

## Rules when editing

1. **One topic = one owning skill.** Other skills reference it; never duplicate rules or tables across skills, references, or templates.
2. **Skills stay framework-free.** Framework details live only in `references/<stack>.md` and the stack profile.
3. **Same section numbers in all three references** (§1 methods … §14 pitfalls). A change to one stack's section must be checked against the other two.
4. **New skill** → `bp-` prefix, frontmatter `name` equals folder name, registered in the routing table of `AGENTS.template.md`, the catalog in `README.md`, and `SKILLS.md`.
5. **Lint config and references must agree**: every import shown in a reference example must pass `lint/eslint.boundaries.mjs`. If you change element patterns or policies, rerun the test matrix below.
6. **No internal names**: no company, client, or internal project names; no machine paths, usernames, drive letters, credentials, or tokens anywhere.
7. Skills, agents, templates, and code: English. `README.md`, `lint/README.md`, and `references/*.md` prose may be Indonesian; code inside them is English.

## Verification before committing

| Check | How |
|---|---|
| Skill copies work | Copy the repo into a temp folder as `<tmp>/guideline/`, run `node <tmp>/guideline/scripts/sync-skills.mjs`, then `--check` → must print `OK` |
| Lint matrix (when `lint/` or layer layout changes) | In a temp project with `eslint@9`, `eslint-plugin-boundaries@^7`, `eslint-import-resolver-typescript@^4`, `typescript-eslint@^8`: allowed imports (domain→kernel, infrastructure→core/application, di→infrastructure, presentation→di, app→presentation/shared-ui, `*.routes.ts`→infrastructure, `src/*.ts` entry files) report 0 errors; forbidden imports (domain→infrastructure, cross-feature, presentation→infrastructure, presentation→app, core→domain, shared-ui→domain, application→react, unknown file under `src/`) each report an error |
| No internal names | `grep -rniE "<internal names>" .` returns nothing |
| Cross-references | Every `bp-*` mentioned in any file exists under `skills/`; no links to removed files |

Never run `scripts/sync-skills.mjs` at this repo's root without `--repo-root` — the script refuses by design.
