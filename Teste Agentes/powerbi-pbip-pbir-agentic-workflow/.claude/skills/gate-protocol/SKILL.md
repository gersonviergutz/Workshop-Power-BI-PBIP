---
name: gate-protocol
description: Padrão único de gate de aprovação humana entre fases do pipeline BI. Use sempre que um agente concluir uma fase e o orquestrador precisar pedir aprovação ao usuário antes de seguir.
---

# Skill: Gate Protocol

## Quando usar
Sempre que o `orquestrador-bi` termina uma fase ou subfase e precisa decisão humana para seguir.

## Estrutura obrigatória de um gate

1. **Resumo executivo** (3 a 5 bullets) — o que foi produzido.
2. **Artefatos** — lista de arquivos criados/alterados com caminho.
3. **Riscos e pendências** — itens que ficaram em aberto.
4. **AskUserQuestion** com exatamente estas 3 opções:
   - `Aprovar e seguir para a próxima fase`
   - `Pedir ajustes` (notas: campo livre que volta para o agente)
   - `Pausar projeto`

## Pós-aprovação (obrigatório)

Ao receber "Aprovar":
1. Marcar o checkbox correspondente em `TODO.md` como `[x]` com data e usuário.
2. Se houver decisão estrutural, gravar linha em `DECISIONS.md` com formato `| AAAA-MM-DD | <agente> | <decisão> | <razão> |`.
3. Gravar entrada em `CHANGELOG.md` usando o template do topo do arquivo.
4. Anunciar a próxima fase.

## Pós-ajustes
Devolver ao agente da fase com as notas do usuário e re-executar o gate ao terminar.

## Pós-pausa
Gravar em `TODO.md` o ponto de retomada e encerrar a sessão de orquestração com instrução de usar `/retomar-projeto-bi`.

## Anti-padrões
- Aprovar gate sem ter os 4 elementos do "Estrutura obrigatória" preenchidos.
- Marcar `[x]` antes de gravar `CHANGELOG.md` (ordem importa para auditoria).
- Aceitar resposta livre no lugar das 3 opções fixas.
