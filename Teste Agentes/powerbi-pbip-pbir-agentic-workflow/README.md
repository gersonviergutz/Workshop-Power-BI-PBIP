# Power BI PBIP/PBIR Agentic Workflow

Kit pronto para conduzir um projeto Power BI (PBIP/PBIR) do começo ao fim com agentes especializados no Claude Code. O fluxo é guiado, com **gates de aprovação humana** entre cada fase.

## Início rápido

1. Clone/copie esta pasta para um diretório novo (ou reabra-a vazia para começar).
2. Preencha [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) com cliente, escopo e fontes (campos `<preencher>`).
3. Abra o diretório no Claude Code.
4. Digite `/iniciar-projeto-bi`.

O orquestrador conduz o pipeline da Fase 0 (análise) até a Fase 8 (documentação), pausando em cada gate.

Se a sessão for interrompida, use `/retomar-projeto-bi` — o orquestrador lê `TODO.md`, identifica o primeiro gate `[ ]` e segue dali.

## Fluxo de fases

| Fase | Agente | Saída principal |
|---|---|---|
| 0. Análise do projeto | `orquestrador-bi` | resumo de status + escolha de modo |
| 1. Requisitos | `analista-requisitos` | `docs/levantamento-requisitos/` |
| 2. Power Query | `power-query-reviewer` | diagnóstico + edits em partitions TMDL |
| 3. Modelagem | `data-modeler` | TMDL + dicionário de dados |
| 4. DAX | `dax-specialist` | matriz de viabilidade + medidas |
| 5. Mockup Figma | `figma-mockup-designer` | URL Figma + screenshots |
| 6. Relatório PBIR | `pbir-report-builder` | páginas/visuais PBIR |
| 7. QA | `bi-qa-validator` | relatório de QA |
| 8. Documentação | `documentador-bi` | docs técnica e funcional |

Detecção automática de entrada na Fase 1: o agente verifica `docs/transcricoes/`, depois `docs/levantamento-requisitos/`, e em último caso oferece questionário guiado ou resumo livre.

## Pré-requisitos

- Claude Code instalado.
- Para a Fase 5 (Figma): MCP do Figma autenticado (`mcp__claude_ai_Figma` ou `mcp__plugin_figma_figma`). Se indisponível, o agente 09 aborta com instruções e o pipeline pode seguir com mockup textual via fallback.
- Para as fases técnicas: Power BI Desktop disponível para validação runtime de refresh e visuais.

## Estrutura

```text
.claude/
  agents/         Agentes especializados (00 a 09)
  skills/         Skills reutilizáveis (gate-protocol, figma-mockup, etc.)
  commands/       Slash commands (iniciar/retomar-projeto-bi)
docs/
  templates/      Templates (TODO, etc.)
  transcricoes/   Entrada opcional para a Fase 1
  superpowers/    Specs e plans de evolução do próprio kit
  padroes-nomenclatura/  Convenções TMDL/DAX
  matriz-agentes-skills.md   Matriz de escopo por agente
powerbi/          Será criado durante a Fase 3 quando o data-modeler iniciar TMDL
prompts/          Prompts manuais (uso cirúrgico, fora do pipeline guiado)
AGENTS.md         Regras de escopo dos agentes
CLAUDE.md         Guidance para Claude Code
PROJECT_CONTEXT.md  Contexto deste projeto (preencher)
TODO.md           Estado do pipeline (fonte da verdade dos gates)
DECISIONS.md      Decisões técnicas/funcionais
CHANGELOG.md      Histórico de alterações
```

## Regras críticas (escopo dos agentes)

- Nenhum agente deve modificar arquivos fora do seu escopo (definido em `AGENTS.md` e `docs/matriz-agentes-skills.md`).
- Toda alteração deve atualizar `CHANGELOG.md`.
- Toda decisão relevante deve atualizar `DECISIONS.md`.
- Todo KPI deve ser rastreável: **dor → pergunta de negócio → KPI → medida DAX → visual PBIR**.
- Antes de pedir aprovação no Gate 6 (PBIR), o orquestrador roda `pbip-validator` automaticamente.

## Skills locais incluídas

- `process-orchestration` — gates, TODOs, handoffs
- `gate-protocol` — padrão único de gate de aprovação humana
- `figma-mockup` — geração de mockup no Figma a partir do PRD
- `pbip-repository-governance` — segurança em alterações PBIP/PBIR/TMDL
- `requirements-discovery` — leitura de transcrições + extração de requisitos
- `power-query-m` — análise de M, performance, nomenclatura
- `tmdl` — alterações seguras em modelo semântico
- `dimensional-modeling` — estrela, fatos, dimensões, relacionamentos
- `dax` — criação e revisão de medidas
- `pbir` — criação e revisão de relatórios PBIR
- `dataviz-powerbi` — boas práticas de visualização
- `client-presentation` — narrativa executiva textual
- `qa-validation` — checklist final
- `bi-documentation` — documentação técnica/funcional

Skills globais e de plugins (`pbip:*`, `reports:*`, `semantic-models:*`, `power-bi-*`) são preferidas para operações low-level e são acionadas pelos agentes quando necessário.

## Para auditoria / evolução do kit

Specs e plans em `docs/superpowers/` documentam decisões de arquitetura do próprio kit (não dos projetos que ele entrega).
