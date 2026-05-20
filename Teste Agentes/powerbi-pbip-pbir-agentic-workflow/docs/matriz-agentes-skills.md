# Matriz Agente x Skills

| Agente | Skills obrigatórias | Pode alterar modelo? | Pode alterar relatório? |
|---|---|---:|---:|
| 00-orquestrador-bi | process-orchestration, pbip-repository-governance, requirements-discovery, tmdl, pbir, qa-validation, gate-protocol | Não | Não |
| 01-analista-requisitos | requirements-discovery, client-presentation, process-orchestration | Não | Não |
| 02-power-query-reviewer | power-query-m, tmdl, pbip-repository-governance, qa-validation | Sim, camada Power Query/TMDL relacionada | Não |
| 03-data-modeler | dimensional-modeling, tmdl, pbip-repository-governance, qa-validation | Sim, modelagem/relacionamentos | Não |
| 04-dax-specialist | dax, tmdl, dimensional-modeling, pbip-repository-governance, qa-validation | Sim, medidas/metadados DAX | Não |
| 05-client-presentation | client-presentation, dataviz-powerbi, requirements-discovery | Não | Não |
| 06-pbir-report-builder | pbir, tmdl, dataviz-powerbi, pbip-repository-governance, qa-validation | Não, exceto leitura TMDL | Sim |
| 07-bi-qa-validator | qa-validation, requirements-discovery, power-query-m, tmdl, dax, pbir, dataviz-powerbi, dimensional-modeling | Não por padrão | Não por padrão |
| 08-documentador | bi-documentation, requirements-discovery, tmdl, dax, pbir, dataviz-powerbi | Não | Não |
| 09-figma-mockup-designer | figma-mockup, client-presentation, dataviz-powerbi, requirements-discovery | Não | Não |

## Observação
Agentes que modificam arquivos técnicos precisam sempre carregar `pbip-repository-governance` e a skill técnica específica do objeto alterado.

## Skills transversais
- `gate-protocol` — carregada pelo `00-orquestrador-bi` em modo Guided Pipeline. Não pertence a um único agente.
