# CHANGELOG.md

## Não iniciado

Nenhuma alteração aplicada ainda.

## Modelo de registro

```text
Data:
Agente:
Arquivos alterados:
Resumo:
Impacto:
Riscos:
Próxima ação:
```
## 2026-04-24

- Criadas skills completas e vinculadas para todos os agentes.
- Atualizados metadados dos agentes com skills obrigatórias.
- Adicionada matriz `docs/matriz-agentes-skills.md`.

Data: 2026-04-24
Agente: Analista-requisitos
Arquivos alterados:
- `docs/levantamento-requisitos/levantamento-requisitos.md`
- `docs/levantamento-requisitos/requisitos-aprovados.md`
- `docs/levantamento-requisitos/matriz-rastreabilidade.md`
- `TODO.md`
- `DECISIONS.md`
Resumo: Consolidados requisitos do dashboard comercial/financeiro a partir da transcrição de reunião.
Impacto: Gate 1 passa a ter documentação estruturada, matriz de rastreabilidade e pendências explícitas para validação humana.
Riscos: Requisitos ainda não estão formalmente aprovados por Carlos, Mariana e Rafael; metas, fonte ERP, RLS e alguns critérios dependem de confirmação.
Próxima ação: Validar documentos do Gate 1 com os responsáveis de negócio antes de avançar para Power Query/modelagem.

Data: 2026-04-24
Agente: Analista-requisitos
Arquivos alterados:
- `docs/levantamento-requisitos/pacote-validacao-gate-1.md`
- `TODO.md`
- `DECISIONS.md`
Resumo: Criado pacote objetivo de validação do Gate 1 com checklist, perguntas para responsáveis e critérios de aprovação.
Impacto: Facilita coleta de aceite explícito antes de avançar para refatoração de Power Query/modelagem.
Riscos: Gate 1 permanece pendente até retorno de Carlos, Mariana, Rafael e TI sobre fonte ERP, metas, atingimento e RLS.
Próxima ação: Enviar o pacote de validação para revisão humana e registrar a decisão de aprovação ou ajustes.

Data: 2026-04-24
Agente: Analista-requisitos
Arquivos alterados:
- `docs/levantamento-requisitos/mensagem-validacao-gate-1.md`
- `docs/levantamento-requisitos/termo-aceite-gate-1.md`
Resumo: Criados artefatos de apoio para coleta de aprovação formal do Gate 1.
Impacto: Facilita envio aos responsáveis e registro estruturado de aceite, ressalvas ou reprovação.
Riscos: O Gate 1 continua pendente até preenchimento do termo de aceite por Carlos, Mariana, Rafael e TI.
Próxima ação: Coletar respostas dos responsáveis e atualizar o status do Gate 1 conforme decisão registrada.

Data: 2026-04-24
Agente: Orquestrador / Analista-requisitos
Arquivos alterados:
- `TODO.md`
- `DECISIONS.md`
- `docs/levantamento-requisitos/requisitos-aprovados.md`
- `docs/levantamento-requisitos/termo-aceite-gate-1.md`
- `docs/levantamento-requisitos/pacote-validacao-gate-1.md`
Resumo: Gate 1 registrado como aprovado pelo cliente sem ajustes.
Impacto: Fase 2 concluída e Fase 3 — Análise Power Query autorizada.
Riscos: Apesar da aprovação funcional, mudanças técnicas continuam dependentes de análise de impacto, especialmente fonte ERP, metas e RLS.
Próxima ação: Executar a revisão/refatoração de Power Query conforme escopo aprovado.

Data: 2026-04-24
Agente: Power-query-reviewer
Arquivos alterados:
- `docs/power-query/auditoria-power-query.md`
- `docs/power-query/plano-refatoracao-power-query.md`
- `TODO.md`
- `DECISIONS.md`
Resumo: Fase 3 executada como auditoria formal de Power Query; Gate 2 marcado como revisado.
Impacto: Queries, tipos, nomenclatura e riscos foram documentados; nenhuma consulta TMDL foi alterada.
Riscos: A fonte atual continua sendo Excel local hard-coded até uma refatoração controlada.
Próxima ação: Decidir se a próxima rodada aplicará a refatoração Power Query proposta ou seguirá para modelagem.

Data: 2026-04-24
Agente: Power-query-reviewer
Arquivos alterados:
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Vendas.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Produtos.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Clientes.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Localização.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Calendário.tmdl`
- `docs/power-query/refatoracao-power-query-aplicada.md`
- `docs/power-query/plano-refatoracao-power-query.md`
- `DECISIONS.md`
Resumo: Refatoração controlada de Power Query aplicada em TMDL.
Impacto: Steps M foram padronizados, seleção explícita de colunas foi adicionada, `Localização` passou a promover cabeçalhos antes da tipagem, `Calendário` deixou de usar `DateTime.LocalNow()` e `MesNum`/`AnoMesINT` foram ajustados para inteiro.
Riscos: A fonte ainda é Excel local e a mesma planilha segue aberta por múltiplas queries; validação de refresh no Power BI Desktop ainda é necessária.
Próxima ação: Abrir o PBIP no Power BI Desktop, executar refresh e decidir se será criada fonte/staging compartilhada ou se o projeto seguirá para modelagem.

Data: 2026-04-24
Agente: Data Modeler
Arquivos alterados:
- `docs/dicionario-dados/diagnostico-modelagem.md`
- `TODO.md`
- `DECISIONS.md`
Resumo: Fase 4 executada como diagnóstico estrutural do modelo semântico TMDL, sem alterar arquivos de modelo.
Impacto: Fato provisória, dimensões, relacionamentos atuais, lacunas de calendário, riscos de cardinalidade, filtros e impactos para DAX foram documentados.
Riscos: Cardinalidade real ainda depende de refresh/inspeção dos dados; modelo ainda não está pronto para medidas DAX finais sem correção estrutural.
Próxima ação: Aplicar rodada controlada de correções TMDL de modelagem, começando por relacionamento com calendário, propriedades de dimensão e validação de chaves.

Data: 2026-04-24
Agente: Documentador BI
Arquivos alterados:
- `docs/documentacao/documentacao-funcional.md`
- `docs/documentacao/documentacao-tecnica.md`
- `docs/documentacao/catalogo-medidas-e-objetos.md`
- `docs/documentacao/guia-uso.md`
- `docs/documentacao/release-notes.md`
- `README.md`
- `TODO.md`
Resumo: Criada documentação funcional e técnica parcial do projeto no estado atual, sem alterar Power Query, TMDL, PBIR ou DAX.
Impacto: Cliente, usuários e manutenção passam a ter visão consolidada de requisitos, modelo atual, objetos existentes, medidas previstas, guia de uso e limitações conhecidas.
Riscos: A documentação ainda não é final porque DAX, PBIR, RLS, metas e QA não foram concluídos.
Próxima ação: Atualizar a documentação final após correções de modelagem, criação de medidas, construção PBIR e QA.

Data: 2026-04-24
Agente: Data Modeler
Arquivos alterados:
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/relationships.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Vendas.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Produtos.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Clientes.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Localização.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/Calendário.tmdl`
- `docs/dicionario-dados/relacionamentos.md`
- `docs/dicionario-dados/correcao-modelagem-aplicada.md`
- `docs/documentacao/documentacao-tecnica.md`
- `docs/documentacao/guia-uso.md`
- `docs/documentacao/release-notes.md`
- `TODO.md`
- `DECISIONS.md`
Resumo: Fase 4 concluída estruturalmente com correções TMDL de relacionamento, propriedades de calendário, ocultação de chaves técnicas e documentação dos relacionamentos.
Impacto: O modelo passa a ter caminho temporal entre `Vendas` e `Calendário`, atributos de calendário deixam de somar indevidamente e o Gate 3 fica concluído em nível estrutural.
Riscos: Refresh, unicidade real das chaves e validação runtime ainda precisam ser executados no Power BI Desktop; RLS, metas, DAX e PBIR seguem pendentes para fases posteriores.
Próxima ação: Validar refresh do PBIP e avançar para a próxima fase do fluxo.

Data: 2026-04-24
Agente: Data Modeler
Arquivos alterados:
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/model.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/relationships.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/fato_Vendas.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/dim_Produto.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/dim_Cliente.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/dim_Localizacao.tmdl`
- `powerbi/Power BI Workshop Codex.SemanticModel/definition/tables/dim_Calendario.tmdl`
- `docs/padroes-nomenclatura/nomenclatura-modelo.md`
- `docs/dicionario-dados/relacionamentos.md`
- `docs/dicionario-dados/correcao-modelagem-aplicada.md`
- `docs/documentacao/documentacao-tecnica.md`
- `docs/documentacao/catalogo-medidas-e-objetos.md`
- `docs/documentacao/guia-uso.md`
- `docs/documentacao/release-notes.md`
- `TODO.md`
- `DECISIONS.md`
- `README.md`
Resumo: Padronizada nomenclatura de tabelas e colunas do modelo semântico.
Impacto: Tabelas usam `fato_`/`dim_`; colunas usam PascalCase sem acentos e sem underscores; relacionamentos foram atualizados para os novos nomes.
Riscos: O PBIP precisa ser aberto e atualizado no Power BI Desktop para validar runtime após a renomeação; próximas fases de DAX/PBIR devem usar os novos nomes.
Próxima ação: Validar refresh no Power BI Desktop antes de criar medidas DAX.

## 2026-05-20 — Pipeline Guiado de BI

**Agente:** brainstorming + writing-plans (humano: gersonggv)
**Arquivos alterados:**
- Criados: .claude/commands/iniciar-projeto-bi.md, .claude/commands/retomar-projeto-bi.md, .claude/agents/09-figma-mockup-designer.md, .claude/skills/gate-protocol/SKILL.md, .claude/skills/figma-mockup/SKILL.md, docs/templates/TODO-template.md, docs/power-query/input/.gitkeep
- Modificados: .claude/agents/00,01,02,04,05,06, AGENTS.md, DECISIONS.md, docs/matriz-agentes-skills.md, TODO.md
**Resumo:** Adoção de pipeline guiado interativo com gates explícitos e novo agente para mockup Figma.
**Impacto:** Fluxo manual via prompts/01 deixa de ser o caminho recomendado; uso passa pelo /iniciar-projeto-bi.
**Riscos:** Dependência de Figma MCP autenticado na Fase 5; fallback textual disponível.
**Próxima ação:** Executar E2E no caso Codex Retail.
