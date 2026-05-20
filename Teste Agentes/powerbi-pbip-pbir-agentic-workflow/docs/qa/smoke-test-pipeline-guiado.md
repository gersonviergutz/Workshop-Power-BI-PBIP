# Smoke Test — Pipeline Guiado de BI

**Data:** 2026-05-20
**Executor:** Agente automatizado (subagent driven, fases A–G)
**Plano executado:** `docs/superpowers/plans/2026-05-20-pipeline-guiado-bi.md`

## Cobertura desta validação

Esta validação cobre apenas verificações estruturais (presença de arquivos, frontmatter YAML válido). Os testes interativos (Steps 3 e 4) requerem sessão Claude Code humana e ficam **diferidos**.

## Step 1 — Arquivos criados existem

Verificação executada via `for f in <lista>; do test -f "$f"; done`. Resultado:

| Arquivo | Status |
|---|---|
| `.claude/commands/iniciar-projeto-bi.md` | OK |
| `.claude/commands/retomar-projeto-bi.md` | OK |
| `.claude/agents/09-figma-mockup-designer.md` | OK |
| `.claude/skills/gate-protocol/SKILL.md` | OK |
| `.claude/skills/figma-mockup/SKILL.md` | OK |
| `docs/templates/TODO-template.md` | OK |

Adicional: `docs/power-query/input/.gitkeep` — OK.

## Step 2 — Frontmatter YAML válido

Verificação via `grep -E "^name:|^description:"` em cada arquivo novo. Todos os 5 arquivos com frontmatter possuem ambos os campos:

| Arquivo | name | description |
|---|---|---|
| `.claude/agents/09-figma-mockup-designer.md` | figma-mockup-designer | presente |
| `.claude/skills/gate-protocol/SKILL.md` | gate-protocol | presente |
| `.claude/skills/figma-mockup/SKILL.md` | figma-mockup | presente |
| `.claude/commands/iniciar-projeto-bi.md` | (n/a — comandos usam só description) | presente |
| `.claude/commands/retomar-projeto-bi.md` | (n/a — comandos usam só description) | presente |

Orquestrador modificado:
- `.claude/agents/00-orquestrador-bi.md` — frontmatter `skills:` agora inclui `gate-protocol` (verificado por grep).

## Step 3 — Smoke test do slash command `/iniciar-projeto-bi` — DIFERIDO

**Status:** Não executado.
**Motivo:** Requer sessão Claude Code interativa para invocar o slash command e validar:
- Agente `orquestrador-bi` é invocado.
- Fase 0 é apresentada (resumo do projeto).
- `AskUserQuestion` abre com 3 opções (Continuar / Recomeçar / Cancelar).

**Ação para o humano (gersonggv):** Em uma nova sessão Claude Code limpa, digitar `/iniciar-projeto-bi` e validar os 3 critérios acima. Se falhar, retornar à Task C1 ou B1.

## Step 4 — Smoke test do agente `09-figma-mockup-designer` — DIFERIDO

**Status:** Não executado.
**Motivo:** Requer (a) sessão interativa, (b) Figma MCP autenticado.

**Ação para o humano:** Após autenticar Figma MCP (`mcp__claude_ai_Figma__whoami` ou `mcp__plugin_figma_figma__authenticate`), em sessão separada solicitar:
> Acione o agente figma-mockup-designer com o PRD atual.

Validar:
- Agente verifica autenticação Figma antes de qualquer outra ação.
- Se autenticado: gera arquivo no Figma e retorna URL + thumbnails em `docs/mockups/`.
- Se não: aborta com mensagem clara sugerindo `05-client-presentation` como fallback textual.

## Conclusão

Estrutura entregue e estruturalmente válida. Validação funcional (E2E interativa) é responsabilidade humana e deve ocorrer no caso Codex Retail antes de declarar o pipeline pronto para produção.
