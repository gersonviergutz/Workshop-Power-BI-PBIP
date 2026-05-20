# CHANGELOG.md

> Toda alteração feita por um agente deve gerar uma entrada aqui, seguindo o template abaixo.

## Template

```
## AAAA-MM-DD — <Título curto da mudança>

- Agente: <nome do agente ou pessoa>
- Arquivos alterados:
  - Criados: <lista>
  - Modificados: <lista>
  - Removidos: <lista>
- Resumo: <1–3 frases descrevendo a mudança>
- Impacto: <quem/o que é afetado>
- Riscos: <pendências, dependências, validações pendentes>
- Próxima ação: <quem deve fazer o quê em seguida>
```

---

## 2026-05-20 — Pipeline Guiado de BI (release inicial do kit)

- Agente: brainstorming + writing-plans (humano: gersonggv)
- Arquivos alterados:
  - Criados: `.claude/commands/iniciar-projeto-bi.md`, `.claude/commands/retomar-projeto-bi.md`, `.claude/agents/09-figma-mockup-designer.md`, `.claude/skills/gate-protocol/SKILL.md`, `.claude/skills/figma-mockup/SKILL.md`, `docs/templates/TODO-template.md`
  - Modificados: `.claude/agents/00-orquestrador-bi.md`, `.claude/agents/01-analista-requisitos.md`, `.claude/agents/02-power-query-reviewer.md`, `.claude/agents/04-dax-specialist.md`, `.claude/agents/05-client-presentation.md`, `.claude/agents/06-pbir-report-builder.md`, `AGENTS.md`, `DECISIONS.md`, `docs/matriz-agentes-skills.md`
- Resumo: Adoção de pipeline guiado interativo com gates explícitos via `/iniciar-projeto-bi`. Novo agente `09-figma-mockup-designer` para mockup no Figma entre as fases de DAX e PBIR.
- Impacto: Fluxo recomendado para novos projetos passa por `/iniciar-projeto-bi`. Os prompts em `prompts/` permanecem disponíveis para uso manual cirúrgico.
- Riscos: Dependência de Figma MCP autenticado na Fase 5 (fallback textual via `05-client-presentation` disponível).
- Próxima ação: Rodar `/iniciar-projeto-bi` em sessão Claude Code limpa para iniciar um projeto real.

---

<!-- Adicione abaixo as entradas dos próximos commits/agentes -->
