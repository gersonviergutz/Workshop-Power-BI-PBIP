# AGENTS.md

## Contexto

Este repositório contém projetos Power BI em formato PBIP/PBIR, com modelo semântico em TMDL.

## Objetivo

Apoiar o ciclo completo de desenvolvimento de soluções Power BI:

1. Levantamento de requisitos.
2. Análise de Power Query.
3. Modelagem de dados.
4. Criação de medidas DAX.
5. Criação de apresentação para cliente.
6. Criação/revisão de relatórios PBIR.
7. Validação técnica e documentação.

## Fluxo obrigatório

1. Ler `PROJECT_CONTEXT.md`.
2. Ler `TODO.md`.
3. Ler `DECISIONS.md`.
4. Identificar a fase atual.
5. Trabalhar apenas no escopo da fase.
6. Antes de editar, apresentar diagnóstico e plano.
7. Após editar, atualizar `CHANGELOG.md`.
8. Registrar decisões relevantes em `DECISIONS.md`.
9. Atualizar `TODO.md` com status da fase.

## Regras gerais

- Nunca alterar arquivos sem explicar o plano.
- Nunca alterar modelo sem verificar dependências.
- Nunca excluir objetos sem listar impacto.
- Alterações em TMDL precisam ser documentadas.
- Alterações em PBIR precisam preservar JSON válido.
- Alterações em Power Query precisam considerar impacto em modelo, medidas e visuais.
- Todo KPI precisa estar vinculado a uma pergunta de negócio.
- Todo visual precisa estar vinculado a uma decisão ou análise.
- Se houver dúvida de regra de negócio, registrar pendência para validação humana.

## Escopos protegidos

### Power Query Reviewer
Pode analisar e sugerir alterações em consultas M e expressões relacionadas.
Não pode alterar medidas DAX, visuais PBIR ou relacionamentos sem envolver o agente responsável.

### Data Modeler
Pode revisar e propor alterações em tabelas, relacionamentos, cardinalidade, direções de filtro e documentação do modelo.
Não pode criar layout de relatório nem alterar consultas M sem envolver o agente Power Query.

### DAX Specialist
Pode criar e revisar medidas DAX alinhadas aos requisitos aprovados.
Não pode criar visuais PBIR nem alterar relacionamentos sem revisão.

### PBIR Report Builder
Pode criar/revisar páginas, visuais, filtros, tooltips e navegação PBIR.
Não pode criar medidas DAX finais sem envolver o agente DAX.

### Figma Mockup Designer
Pode criar arquivos no Figma via MCP e gravar metadados em `docs/mockups/`.
Não pode alterar TMDL, PBIR, Power Query, DAX nem agentes de outras fases.
Depende de Figma MCP autenticado — sem ele a fase é abortada.

## Pipeline Guiado
O comando `/iniciar-projeto-bi` executa as fases 1–8 em sequência com gates de aprovação humana.
Estado persistido em `TODO.md`. Padrão de gate em `.claude/skills/gate-protocol/SKILL.md`.
Retomada: `/retomar-projeto-bi`.

## Convenções de nomenclatura

### Tabelas

- `stg_` para staging.
- `dim_` para dimensões.
- `fato_` para fatos.
- `aux_` para auxiliares.
- `param_` para parâmetros.
- `fn_` para funções Power Query.

### Medidas

- Nome orientado ao negócio.
- Sem prefixos técnicos desnecessários.
- Usar display folders.
- Descrever medidas críticas.

Exemplos:

- `[Receita Bruta]`
- `[Receita Líquida]`
- `[Quantidade Vendida]`
- `[Ticket Médio]`
- `[Margem %]`
- `[Atingimento Meta %]`

## Critérios de aceite

Antes de concluir qualquer tarefa:

- O requisito foi atendido?
- O modelo continua consistente?
- A alteração é rastreável?
- Os nomes estão padronizados?
- O impacto foi documentado?
- Existe risco conhecido?
- Há pendência para validação humana?
