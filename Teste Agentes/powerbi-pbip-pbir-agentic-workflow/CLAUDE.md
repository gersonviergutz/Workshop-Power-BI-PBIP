# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **not** a software codebase — there is no build, no tests, no package manager. It is an **agentic workflow kit** for delivering Power BI PBIP/PBIR projects (semantic model in TMDL + PBIR report). It ships ready-to-use:

- Specialized subagents under [.claude/agents/](.claude/agents/)
- Reusable skills under [.claude/skills/](.claude/skills/)
- Slash commands under [.claude/commands/](.claude/commands/) — entry point is `/iniciar-projeto-bi`
- Governance templates at the project root and under [docs/templates/](docs/templates/)

When a project is in flight, the artifacts being edited are:

- TMDL files under `powerbi/<NomeDoProjeto>.SemanticModel/definition/`
- PBIR JSON under `powerbi/<NomeDoProjeto>.Report/definition/`
- Markdown governance/documentation under [docs/](docs/)

A parallel set of skills also ships globally under `~/.claude/skills/` and via plugins (`pbip:`, `reports:`, `semantic-models:`, `power-bi-*`, etc.) — prefer those for low-level PBIP/TMDL/PBIR/DAX operations.

## Starting a new project

Run `/iniciar-projeto-bi` in a fresh Claude Code session. The `orquestrador-bi` agent will:
1. Read `AGENTS.md`, `PROJECT_CONTEXT.md`, `TODO.md`, `DECISIONS.md`, `CHANGELOG.md`
2. Show the project status and ask whether to start fresh or resume
3. Drive each phase end-to-end with explicit human gates between phases

To resume an interrupted project, use `/retomar-projeto-bi`.

## The mandatory read-before-acting protocol

Every action in this repo (including subagents you dispatch) **must** start by reading these four files, in this order:

1. [AGENTS.md](AGENTS.md) — agent scope rules, naming conventions, critical guardrails
2. [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) — client, scope, sources, constraints
3. [TODO.md](TODO.md) — current phase, gate status, pending items
4. [DECISIONS.md](DECISIONS.md) — accumulated technical/functional decisions

Then identify the **current phase** from `TODO.md` and act only within that phase's scope. After any edit:

- Append an entry to [CHANGELOG.md](CHANGELOG.md) using the template at the top of that file (Data / Agente / Arquivos alterados / Resumo / Impacto / Riscos / Próxima ação).
- If the edit reflects a structural choice, add a row to [DECISIONS.md](DECISIONS.md).
- Update phase/gate checkboxes in [TODO.md](TODO.md) via the `gate-protocol` skill.

Skipping these updates leaves the project untraceable and is the single most common failure mode in this workflow.

## Phased workflow with gates

Phases are sequential and gated. Do not start a phase until the previous gate is closed in `TODO.md`.

| Fase | Agente | Saída | Gates |
|---|---|---|---|
| 0. Análise do projeto | `orquestrador-bi` | resumo de status | 0 |
| 1. Requisitos | `analista-requisitos` | `docs/levantamento-requisitos/` | 1 |
| 2. Power Query review | `power-query-reviewer` | `docs/power-query/` + edits em partitions TMDL | 2a, 2b |
| 3. Modelagem | `data-modeler` | TMDL tables/relationships + `docs/dicionario-dados/` | 3a, 3b |
| 4. DAX | `dax-specialist` | `docs/dax/matriz-viabilidade.md` + medidas TMDL | 4a, 4b |
| 5. Mockup Figma | `figma-mockup-designer` | `docs/mockups/` (URL + screenshots) | 5 |
| 6. Relatório PBIR | `pbir-report-builder` | PBIR pages/visuals JSON | 6 |
| 7. QA | `bi-qa-validator` | `docs/qa/` | 7 |
| 8. Documentação | `documentador-bi` | `docs/documentacao/` | 8 |

Per-phase prompts at `prompts/03-…` through `prompts/08-…` continue available for surgical/manual use, but `/iniciar-projeto-bi` is the recommended path.

## Agent scope is enforced — do not cross lanes

Each agent in [.claude/agents/](.claude/agents/) declares which skills it must load and which file types it may edit. The boundaries below are non-negotiable; violating them is treated as a defect:

- **Power Query Reviewer** — edits M expressions inside TMDL partitions only. Cannot touch DAX, PBIR, or relationships.
- **Data Modeler** — edits TMDL tables, relationships, hierarchies, calendar marking. Cannot edit M or PBIR.
- **DAX Specialist** — edits TMDL measures only. Cannot edit relationships or PBIR.
- **Figma Mockup Designer** — edits files in Figma via MCP and writes to `docs/mockups/`. Cannot touch TMDL, PBIR, M, or DAX.
- **PBIR Report Builder** — edits PBIR JSON only. May read TMDL but not modify it; cannot author final DAX measures.
- **Orquestrador / QA / Documentador** — read-only on technical artifacts unless explicitly authorized in `TODO.md`.

The full matrix lives at [docs/matriz-agentes-skills.md](docs/matriz-agentes-skills.md). When an agent needs work outside its lane, it must hand off via the orchestrator, not edit the file itself.

## Power BI artifact layout (when a project is in flight)

```
powerbi/
  <NomeDoProjeto>.pbip                          # PBIP entry point
  <NomeDoProjeto>.SemanticModel/
    definition.pbism
    definition/
      database.tmdl
      model.tmdl
      relationships.tmdl
      cultures/
      tables/                                   # one .tmdl per table
        fato_*.tmdl
        dim_*.tmdl
  <NomeDoProjeto>.Report/
    definition.pbir
    definition/                                 # PBIR JSON pages/visuals
    StaticResources/
```

Naming conventions in force (see [docs/padroes-nomenclatura/](docs/padroes-nomenclatura/) and AGENTS.md):

- Tables: `fato_*`, `dim_*`, `stg_*`, `aux_*`, `param_*`, `fn_*`
- Columns: PascalCase, **no accents, no underscores** (e.g. `IdCliente`, `AnoMesINT`)
- Measures: business-oriented names, no technical prefixes, organized via display folders.

When renaming, `sourceColumn` in TMDL and Power Query column references must be preserved.

## Editing PBIP/TMDL/PBIR safely

For any TMDL/PBIR/M/DAX edit, prefer the installed plugin skills over hand-editing:

- TMDL syntax & BIM↔TMDL: invoke `pbip:tmdl` skill
- PBIR JSON schemas: invoke `pbip:pbir-format` skill
- DAX authoring/measures: invoke `power-bi-dax` or `dax` skill (multi-line DAX with VAR/RETURN cannot be passed via `-e`; use `--file` or stdin)
- Power Query M: invoke `power-query-m` or `semantic-models:power-query`
- Modeling operations: `power-bi-modeling`
- Report scaffolding/visuals: `power-bi-report`, `power-bi-visuals`, `power-bi-pages`, `power-bi-themes`, `power-bi-filters`
- Validation: `pbip:pbip-validator` agent and `qa-validation` skill before declaring a gate closed

Critical rules from AGENTS.md and `pbip-repository-governance`:

- Never delete tables, columns, or measures without first listing downstream impact.
- TMDL relationship and partition changes require a refresh test in Power BI Desktop — flag this as a runtime pendency in TODO.md, do not assume validation.
- PBIR edits must keep the JSON valid; preserve the `definition.pbir`/`definition/` structure.

## Linking business and technical artifacts

Every KPI must be traceable through this chain:

```
dor (pain) → pergunta de negócio → KPI → medida DAX → visual PBIR
```

The traceability matrix lives at `docs/levantamento-requisitos/matriz-rastreabilidade.md` and is populated by `analista-requisitos` in Fase 1. Adding a measure or visual without a corresponding row there is a QA failure.

## Working language

All governance documents, decisions, changelog entries, and user-facing artifacts are written in **Portuguese (pt-BR)**. Match that language when editing those files. Code identifiers (TMDL/PBIR/DAX) follow the conventions above regardless of conversation language.
