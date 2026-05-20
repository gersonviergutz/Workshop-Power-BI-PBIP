---
name: process-orchestration
description: Skill para orquestrar o ciclo completo de projetos Power BI PBIP/PBIR com gates, TODOs, handoffs, riscos e critérios de aceite.
---

# Skill: Process Orchestration para Power BI PBIP/PBIR

## Quando usar
Use esta skill quando o agente precisar coordenar fases, controlar pendências, definir próximos passos, registrar decisões ou validar passagem entre etapas.

## Objetivo
Garantir que o projeto siga um fluxo controlado, rastreável e auditável, reduzindo alterações aleatórias no modelo, no relatório ou nos requisitos.

## Artefatos controlados
- `TODO.md`
- `DECISIONS.md`
- `CHANGELOG.md`
- `PROJECT_CONTEXT.md`
- `docs/levantamento-requisitos/`
- `docs/qa/`

## Gates obrigatórios

### Gate 1 — Requisitos levantados
Critérios:
- Transcrições/anotações foram lidas.
- Dores foram identificadas.
- Perguntas de negócio foram mapeadas.
- KPIs candidatos foram listados.
- Pendências para o cliente foram registradas.

### Gate 2 — Requisitos aprovados
Critérios:
- Existe arquivo de requisitos aprovados.
- Cada KPI tem fórmula de negócio ou critério de cálculo.
- Cada KPI tem granularidade, dimensão de análise e fonte provável.
- Existe matriz de rastreabilidade.

### Gate 3 — Power Query revisado
Critérios:
- Consultas críticas foram analisadas.
- Tipos de dados foram validados.
- Etapas pesadas foram identificadas.
- Riscos de query folding foram documentados.
- Impactos no modelo foram listados.

### Gate 4 — Modelo validado
Critérios:
- Fatos e dimensões estão identificadas.
- Relacionamentos foram validados.
- Cardinalidade e direção de filtro foram justificadas.
- Tabela calendário foi considerada.
- Chaves técnicas e de negócio foram documentadas.

### Gate 5 — DAX criado e revisado
Critérios:
- Medidas derivam de requisitos aprovados.
- Medidas base foram criadas antes das derivadas.
- Formatação e display folders foram definidos.
- Medidas têm descrição funcional quando relevante.

### Gate 6 — Relatório PBIR implementado
Critérios:
- Páginas respondem perguntas de negócio.
- Visuais usam medidas aprovadas.
- Layout segue boas práticas de DataViz.
- Filtros e navegação foram verificados.

### Gate 7 — QA final
Critérios:
- Requisitos, DAX e visuais estão rastreados.
- Não há alteração não documentada.
- Riscos e pendências foram registrados.
- Existe checklist para publicação.

## Regra de handoff entre agentes
Todo handoff deve conter:

```md
## Handoff

### Fase concluída

### Arquivos analisados

### Arquivos alterados

### Decisões registradas

### Pendências

### Riscos

### Próximo agente recomendado
```

## Política de alteração
Antes de modificar arquivos, o agente deve produzir:
1. Diagnóstico.
2. Plano.
3. Arquivos afetados.
4. Riscos.
5. Critério de reversão.

Depois de modificar, deve atualizar:
- `CHANGELOG.md`
- `TODO.md`
- `DECISIONS.md`, quando houver decisão estrutural.

## Níveis de prioridade
- **P0 Crítico:** quebra modelo, relatório ou cálculo principal.
- **P1 Alto:** afeta resultado, performance ou entendimento executivo.
- **P2 Médio:** melhora governança, nomenclatura ou manutenção.
- **P3 Baixo:** melhoria estética, documentação ou refinamento.
