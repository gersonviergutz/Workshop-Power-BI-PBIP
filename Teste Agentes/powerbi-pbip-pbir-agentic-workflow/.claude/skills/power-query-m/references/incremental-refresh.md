# Incremental Refresh em Power Query

Refresh incremental é a feature que transforma modelos gigantes (100M+ linhas) em viáveis. Em vez de recarregar tudo, recarrega só as partições recentes/alteradas. Mas tem requisitos rigorosos.

## Sumário

1. [Como funciona](#1-como-funciona)
2. [Requisitos obrigatórios](#2-requisitos-obrigatórios)
3. [Parâmetros RangeStart e RangeEnd](#3-parâmetros-rangestart-e-rangeend)
4. [Padrão de query com filtro](#4-padrão-de-query-com-filtro)
5. [Configurar a policy](#5-configurar-a-policy)
6. [Detectar alterações (Detect data changes)](#6-detectar-alterações-detect-data-changes)
7. [Troubleshooting](#7-troubleshooting)
8. [Checklist rápido](#8-checklist-rápido)

---

## 1. Como funciona

Refresh completo tradicional processa **toda a tabela** a cada atualização. Para 10 anos de histórico, isso significa reprocessar dados antigos que nunca mudam. Desperdício.

Com Incremental Refresh, você define:
- **Janela arquivada** (ex.: "últimos 5 anos") — dados históricos que ficam no modelo mas não são reprocessados
- **Janela incremental** (ex.: "últimos 10 dias") — dados recentes que SÃO reprocessados a cada refresh

O Power BI automaticamente particiona a tabela e só atualiza partições da janela incremental. Ganho: refresh de 2h → 5min em modelos grandes.

---

## 2. Requisitos obrigatórios

### 🔴 Query Folding OBRIGATÓRIO

A fonte precisa suportar Query Folding até o filtro de data. Sem isso, Incremental Refresh não funciona — o Power BI vai simular, mas ainda baixa tudo em cada refresh, perdendo o sentido.

**Fontes que suportam (com folding):**
- SQL Server, Azure SQL, Azure Synapse
- PostgreSQL, MySQL, Oracle
- Snowflake, BigQuery
- Dataverse, OData compatível
- Databricks (com DirectQuery ou Import + folding)

**Fontes que NÃO suportam:**
- Excel, CSV, arquivos locais
- SharePoint Lists
- Web scraping via HTML
- APIs genéricas via Web.Contents (exceto OData específico)

**Power BI Premium/PPU** é recomendado para modelos grandes com Incremental Refresh — dá acesso a XMLA endpoint (gerenciar partições externamente), Enhanced Compute engine e partições maiores.

### 🔴 Filtro de data obrigatório

A query precisa ter um filtro que compara uma coluna de data com `RangeStart` e `RangeEnd` (exatamente esses nomes).

### 🔴 Parâmetros tipo DateTime (exatos)

Não podem ser Date. Não podem ter outro nome. Precisam ser `RangeStart` e `RangeEnd`, ambos **DateTime**.

---

## 3. Parâmetros RangeStart e RangeEnd

### Criar os parâmetros

No Power Query Editor: Home → Manage Parameters → New Parameter.

**RangeStart:**
- Name: `RangeStart` (case-sensitive!)
- Type: Date/Time
- Current Value: qualquer data de teste (ex.: 2020-01-01 00:00:00). O Power BI substitui em runtime.

**RangeEnd:**
- Name: `RangeEnd`
- Type: Date/Time
- Current Value: qualquer data de teste (ex.: 2025-01-01 00:00:00)

### ⚠️ Diferença entre DateTime e Date

Um erro muito comum: criar os parâmetros como Date em vez de DateTime. O Power BI só aceita DateTime. Se criou errado, delete e recrie.

### Case-sensitive

`RangeStart` ≠ `rangestart` ≠ `RangeSTART`. Tem que ser **exatamente** `RangeStart` e `RangeEnd`.

---

## 4. Padrão de query com filtro

### Filtro correto

```m
let
    Source = Sql.Database(servidorSQL, bancoSQL),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // Filtro crítico para Incremental Refresh
    filtroIncremental = Table.SelectRows(
        vw_vendas,
        each [data_venda] >= RangeStart and [data_venda] < RangeEnd
    ),
    
    // ... demais transformações depois do filtro
    tipos = Table.TransformColumnTypes(filtroIncremental, {
        {"data_venda", type date},
        {"valor", type number}
    })
in
    tipos
```

**Pontos críticos:**
- **`>= RangeStart`** (incluindo o início)
- **`< RangeEnd`** (EXclusivo — evita duplicar linhas na borda entre partições)
- A **coluna de data** na query (`[data_venda]`) deve estar em tipo Date ou DateTime — não texto.

### ❌ Padrões errados

```m
// ERRADO: usa <= RangeEnd (risco de duplicar linhas)
Table.SelectRows(Source, each [data] >= RangeStart and [data] <= RangeEnd)

// ERRADO: compara com tipos diferentes (data vs datetime)
Table.SelectRows(Source, each Date.From([data]) >= Date.From(RangeStart))

// ERRADO: Date.From quebra folding
Table.SelectRows(Source, each Date.From([data_venda]) >= Date.From(RangeStart))
```

### Validar que está folding

Depois de aplicar o filtro, clique com botão direito no step → **View Native Query**. Você DEVE ver o SQL com `WHERE data_venda >= @p0 AND data_venda < @p1`. Se não vir, o folding quebrou e Incremental não vai funcionar.

---

## 5. Configurar a policy

### No Power BI Desktop

1. Na View **Model** (ou no painel de campos), botão direito na tabela que vai ter Incremental → **Incremental refresh**
2. Ative "Set up incremental refresh for this table"
3. Configure:
   - **Archive data starting**: ex.: "5 years" (mantém 5 anos no modelo)
   - **Incrementally refresh data starting**: ex.: "10 days" (últimos 10 dias são re-processados)
4. Opções avançadas:
   - **Get the latest data in real time with DirectQuery** (apenas Premium)
   - **Only refresh complete days**: geralmente marcado, evita partição de dia em andamento
   - **Detect data changes**: exige coluna tipo DateTime que represente "última modificação" — veja seção 6

5. Clique em **Apply all**

### No serviço (após publicar)

- Configure o gateway (para fontes on-premises)
- Defina credenciais da fonte
- Agende refresh (geralmente daily ou hourly)
- Primeiro refresh processa histórico completo (demora). Próximos refreshes serão incrementais.

### Validação

Após o primeiro refresh incremental (segundo refresh no total), os logs do Power BI Service devem mostrar que só as partições da janela incremental foram processadas.

---

## 6. Detectar alterações (Detect data changes)

### Problema que resolve

Suponha que você configurou "refresh incremental de 10 dias". Se um pedido de 15 dias atrás teve status atualizado hoje, ele NÃO seria atualizado pelo refresh incremental padrão (está fora da janela).

Com **Detect data changes**, o Power BI olha uma coluna tipo DateTime de "última modificação" e reprocessa partições da janela arquivada QUANDO há alteração detectada.

### Como configurar

1. Precisa de uma coluna na tabela tipo DateTime que marca a última modificação de cada linha (ex.: `data_modificacao`, `last_updated_at`)
2. Nas opções de Incremental Refresh, marque **Detect data changes** e escolha essa coluna
3. A cada refresh, o Power BI compara o max dessa coluna com o último valor conhecido — se mudou, reprocessa partição

### Requisito adicional

A coluna de detecção precisa ser **confiável**: atualizada em TODA modificação da linha. Se o ETL não garante isso, Detect data changes pode dar falso "sem alteração".

---

## 7. Troubleshooting

### "Query folding is required for the partitioned refresh"

Folding quebrou em algum step ANTES do filtro com RangeStart/RangeEnd. Solução:
- Revisar a query step por step, validando View Native Query
- Mover transformações M-only para depois do filtro
- Confirmar que `[data_venda]` é Date/DateTime nativo, não conversão

### "Could not infer schema/type when applying incremental refresh"

A coluna de data tem tipo inconsistente. Solução:
- Garantir que o tipo na fonte é Date/DateTime
- Fazer `Table.TransformColumnTypes` ANTES do filtro, não depois

### Refresh continua lento após configurar

Indica que folding não está funcionando de verdade. Teste:
1. Olhe o log de refresh no Power BI Service (detalhe do dataset)
2. Se vê "Refreshing all partitions", o folding falhou e o Power BI caiu para full refresh
3. Retrabalhe a query

### Primeiro refresh demora muito

Isso é normal. O primeiro refresh processa TODO o histórico (5 anos, por exemplo). Próximos refreshes serão rápidos. Para não travar, considere:
- Fazer o primeiro refresh fora do horário de pico
- Em Premium: usar XMLA endpoint com Tabular Editor para carregar partições uma por uma

### "This dataset does not support incremental refresh"

Algum requisito foi violado:
- Fonte sem suporte a folding
- Tabela sem filtro usando RangeStart/RangeEnd
- Parâmetros criados como Date em vez de DateTime
- Nomes errados (ex.: `range_start` em vez de `RangeStart`)

---

## 8. Checklist rápido

Antes de publicar um modelo com Incremental Refresh:

- [ ] Fonte suporta Query Folding (SQL/OData/Dataverse/etc.)
- [ ] Parâmetros `RangeStart` e `RangeEnd` criados como **DateTime** (não Date)
- [ ] Nome dos parâmetros **EXATAMENTE** `RangeStart` e `RangeEnd` (case-sensitive)
- [ ] Filtro aplicado com `>= RangeStart and < RangeEnd` (atenção ao `<`, não `<=`)
- [ ] Filtro aplicado LOGO após Source, antes de merges e transformações pesadas
- [ ] View Native Query validado no step do filtro (folding OK)
- [ ] Coluna de data em tipo Date/DateTime nativo (sem conversão M)
- [ ] Policy configurada no Desktop (Archive + Incremental)
- [ ] Detect data changes configurado SE há coluna de última modificação confiável
- [ ] "Only refresh complete days" marcado (se aplicável)
- [ ] Primeiro refresh rodado com sucesso
- [ ] Segundo refresh é notavelmente mais rápido que o primeiro (prova que está incremental mesmo)

## Incremental Refresh com Polaris (XMLA endpoint)

Em Premium/PPU, você pode gerenciar partições diretamente via **XMLA endpoint** usando SSMS, Tabular Editor ou scripts TMSL. Isso abre cenários avançados:

- Processar partições específicas sob demanda
- Criar partições históricas adicionais fora do padrão da policy
- Backfill de dados históricos sem reprocessar tudo
- Debug de issues de partição

Para usuários avançados. Para começar, use a configuração padrão do Desktop.

## Referências externas

- [Microsoft Learn — Incremental refresh for Power BI](https://learn.microsoft.com/power-bi/connect-data/incremental-refresh-overview)
- [Chris Webb — Incremental refresh deep dives](https://blog.crossjoin.co.uk/category/incremental-refresh/)
- [Rick de Groot — Power Query Incremental Refresh](https://powerquery.how/)
