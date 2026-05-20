# Auditoria Power Query - Plano de Implementacao por Sprint

## Escopo
- Projeto: Power BI Workshop Vs Code
- Tabelas analisadas e padronizadas: Calendario, Clientes, Localizacao, Produtos, Vendas
- Fonte principal: Excel.Workbook (arquivo local)

## O que foi executado

### 1) Correcao e padronizacao do codigo M (executado)
- Padronizacao de steps para nomes descritivos em todas as tabelas.
- Introducao de variavel unica de caminho por query (`pCaminhoArquivoVendas`) para reduzir acoplamento no bloco M.
- Remocao de etapa redundante de tipagem antes de `PromoteHeaders` em Localizacao.
- Refatoracao da tabela de calendario com variaveis mais claras (`dataInicial`, `dataFinal`, `inicioAno`, `fimAno`, `quantidadeDias`, `listaDatas`).
- Ajuste de tipos no calendario para chaves temporais inteiras (`MesNum` e `AnoMesINT` como `Int64.Type`).
- Ajuste de `Date.DayOfWeek(_, Day.Monday)` para padrao explicito de inicio semanal.

### 2) Entrega de padrao Dashmaker nas 5 tabelas (executado)
- Estrutura M mais legivel e consistente.
- Ordem de transformacoes simplificada.
- Codigo pronto para evoluir para parametrizacao centralizada de ambiente.

### 3) Validacao tecnica pos-edicao (executado)
- Validacao de erros nos 5 arquivos TMDL alterados: sem erros.

## Arquivos alterados
- Power BI Workshop Vs Code.SemanticModel/definition/tables/Calendario.tmdl
- Power BI Workshop Vs Code.SemanticModel/definition/tables/Clientes.tmdl
- Power BI Workshop Vs Code.SemanticModel/definition/tables/Localizacao.tmdl
- Power BI Workshop Vs Code.SemanticModel/definition/tables/Produtos.tmdl
- Power BI Workshop Vs Code.SemanticModel/definition/tables/Vendas.tmdl

## Pendencias relevantes (proximo nivel)
- Parametrizacao centralizada no modelo (em vez de variavel local repetida):
  - `pCaminhoArquivoVendas`
  - `dataInicial`
  - `dataFinal`
- Migracao de nomes de colunas para PascalCase sem underscore (ex.: `ID_Cliente` -> `IdCliente`) com avaliacao de impacto em relacionamentos e medidas.
- Organizacao formal em camadas Fonte -> Staging -> Modelo (atualmente o projeto esta mais direto).
- Se houver requisito de historico incremental: definir `RangeStart` e `RangeEnd`.

## Plano por sprint

## Sprint Rapida (1 a 2 dias)
Objetivo: aumentar robustez sem alterar comportamento funcional do modelo.
- Centralizar caminho de arquivo em parametro compartilhado de ambiente.
- Garantir que todas as tabelas consumam esse parametro unico.
- Revisar nomes de steps restantes para consistencia total.
- Checklist de validacao:
  - Atualizacao no Desktop sem erro.
  - Refresh manual concluido.
  - Quantidade de linhas por tabela sem variacao inesperada.

## Sprint Completa (3 a 5 dias)
Objetivo: padronizacao estrutural total Dashmaker com foco em manutencao.
- Reestruturar queries em Fonte/Staging/Modelo.
- Renomear colunas para PascalCase sem underscore.
- Revisar metadados de formatacao e summarization em colunas numericas/chaves.
- Preparar base para incremental refresh (se aplicavel).
- Checklist de validacao:
  - Refresh no Desktop e Servico.
  - Relacionamentos sem quebra.
  - Medidas DAX sem regressao.
  - Performance de refresh comparada (antes/depois).

## Riscos e mitigacao
- Risco: renomeacao de colunas impactar DAX e relacionamentos.
  - Mitigacao: executar renomeacao em lote com mapa de impacto e teste regressivo.
- Risco: dependencia de caminho local em publicacao.
  - Mitigacao: uso de parametro central + governanca de ambiente.
- Risco: diferenca de localidade/acentos em nomes de planilha e colunas.
  - Mitigacao: manter nomes de abas originais e padronizar apenas no modelo final.

## Criterio de pronto (DoD)
- Sem caminhos hard-coded repetidos no M.
- Steps legiveis em todas as queries.
- Tipos essenciais definidos explicitamente.
- Refresh concluindo sem erro no ambiente alvo.
