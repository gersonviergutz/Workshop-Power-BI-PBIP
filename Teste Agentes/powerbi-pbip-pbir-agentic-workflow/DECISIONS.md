# DECISIONS.md

Registre aqui decisões técnicas, funcionais e de modelagem que tomarem-se durante o ciclo de desenvolvimento.

## Decisões fundacionais do kit (não excluir)

| Data | Decisão | Contexto | Impacto | Responsável |
|---|---|---|---|---|
| 2026-05-20 | D-01: Um único slash command `/iniciar-projeto-bi` | Alternativa avaliada: vários comandos por fase | Pedido explícito do usuário; comandos por fase entram só como atalhos opcionais | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-02: `TODO.md` como fonte da verdade do estado | Alternativa avaliada: JSON paralelo / arquivo `.state` | Já é o padrão do projeto, evita duplicação | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-03: Novo agente 09 para Figma | Alternativa avaliada: estender agente 05 (`client-presentation`) | Pedido do usuário; mantém escopos isolados entre narrativa e mockup visual | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-04: Matriz de viabilidade antes de criar medidas | Alternativa avaliada: criar medidas e validar depois | Pedido do usuário; evita medidas órfãs sem rastreabilidade KPI ↔ modelo | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-05: `AskUserQuestion` com 3 opções fixas em todo gate | Alternativa avaliada: texto livre nos gates | Padronização + análise de telemetria futura | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-06: Gate de PBIR roda `pbip-validator` antes de pedir aprovação humana | Alternativa avaliada: aprovar e validar depois | Evita aprovação cega de relatório com defeitos estruturais | brainstorming + writing-plans (gersonggv) |
| 2026-05-20 | D-07: Verificação de Figma MCP antes da Fase 5 | Alternativa avaliada: tentar gerar e tratar erro | UX melhor — instruções claras antes do agente rodar | brainstorming + writing-plans (gersonggv) |

## Decisões do projeto atual

<!-- Adicione abaixo decisões tomadas durante o ciclo deste projeto.
     Formato:
     | AAAA-MM-DD | <decisão> | <contexto / alternativas> | <impacto> | <agente / pessoa> |
-->

| Data | Decisão | Contexto | Impacto | Responsável |
|---|---|---|---|---|
