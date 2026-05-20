# TODO.md — Pipeline Guiado de BI

> Padrão de checkbox: `- [ ]` pendente, `- [x]` aprovado.
> Cada gate aprovado deve listar data, usuário e caminho dos artefatos.

---

## Histórico — Projeto Codex Retail (legado, fase manual)

Este histórico preserva o estado do projeto Codex Retail antes da adoção do pipeline guiado em 2026-05-20. Decisões e gates abaixo continuam válidos como referência histórica.

### Status geral (legado)

- [x] Fase 1 — Levantamento de requisitos (documentado)
- [x] Fase 2 — Aprovação dos requisitos (Gate 1 aprovado)
- [x] Fase 3 — Análise Power Query
- [x] Fase 4 — Modelagem de dados (concluída estruturalmente)
- [ ] Fase 5 — Apresentação e mockup ao cliente
- [ ] Fase 6 — Criação de medidas DAX
- [ ] Fase 7 — Criação/revisão PBIR
- [ ] Fase 8 — QA final
- [ ] Fase 9 — Documentação e entrega

### Pendências runtime (legado)

| Item | Responsável | Status | Observação |
|---|---|---|---|
| Confirmar conexão direta com banco do ERP | TI do cliente | Pendente | Citado por Rafael — impacta frequência de refresh |
| Confirmar origem/granularidade das metas comerciais | Comercial | Pendente | Mencionado mas não detalhado |
| Base de estoque disponível? | TI do cliente | Pendente | Para fase 2 (análise de estoque) |
| Validar refresh após refatoração Power Query | Power BI Desktop / Humano | Pendente | Abrir PBIP e atualizar dados para validar runtime das partições M alteradas |
| Validar unicidade das chaves das dimensões | Data Modeler / Power BI Desktop | Pendente no runtime | Estrutura de relacionamento concluída; confirmar `dim_Cliente[IdCliente]`, `dim_Produto[IdProduto]`, `dim_Localizacao[IdLocal]` e `dim_Calendario[Data]` após refresh |
| Atualizar documentação final após DAX, PBIR e QA | Documentador BI | Pendente | Documentação criada nesta rodada é parcial e deve ser revisada ao final do ciclo |

---

## Pipeline Guiado (template novo — adotado em 2026-05-20)

A partir desta data novos projetos devem usar a estrutura abaixo. O projeto Codex Retail pode ser remapeado para esta estrutura ao retomar via `/retomar-projeto-bi`.

> **Remapeamento Codex Retail → pipeline guiado:** a antiga "Fase 5 — Apresentação e mockup ao cliente" (legado) foi dividida em duas fases novas: **Fase 5 — Mockup Figma** (agente 09) faz o mockup visual; a narrativa executiva textual permanece com `05-client-presentation` mas roda como entregável paralelo, não como fase com gate próprio.

## Fase 0 — Análise do projeto
- [ ] Gate 0 (visão geral apresentada e modo escolhido)

## Fase 1 — Levantamento de requisitos (analista-requisitos)
- [ ] Gate 1 (PRD + indicadores aprovados)
  - Entrada: <preencher>
  - Saída: docs/levantamento-requisitos/requisitos-aprovados.md
  - Observações: <preencher>

## Fase 2 — Power Query review (power-query-reviewer)
- [ ] Gate 2a (sugestões aprovadas)
- [ ] Gate 2b (alterações aplicadas e validadas)

## Fase 3 — Modelagem (data-modeler)
- [ ] Gate 3a (proposta de modelo aprovada)
- [ ] Gate 3b (alterações TMDL aplicadas e validadas)

## Fase 4 — DAX (dax-specialist)
- [ ] Gate 4a (matriz de viabilidade aprovada)
- [ ] Gate 4b (medidas criadas e validadas)

## Fase 5 — Mockup Figma (figma-mockup-designer)
- [ ] Gate 5 (mockup Figma aprovado)
  - URL: <preencher após gerar>

## Fase 6 — Relatório PBIR (pbir-report-builder)
- [ ] Gate 6 (relatório PBIR aprovado, pbip-validator passou)

## Fase 7 — QA (bi-qa-validator)
- [ ] Gate 7 (QA aprovada)

## Fase 8 — Documentação (documentador-bi)
- [ ] Gate 8 (entrega final, handover concluído)
