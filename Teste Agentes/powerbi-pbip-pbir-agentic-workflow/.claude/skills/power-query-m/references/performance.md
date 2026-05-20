# Performance em Power Query

Regras e padrões para evitar que o refresh do modelo fique lento. Foco em **Query Folding** (o principal fator de performance), ordem correta de etapas, uso consciente de `Table.Buffer`, redução antecipada de dados e tipos de dados definidos cedo.

## Sumário

1. [Query Folding — o ponto #1 de performance](#1-query-folding--o-ponto-1-de-performance)
2. [Ordem correta de etapas](#2-ordem-correta-de-etapas)
3. [Redução antecipada de linhas e colunas](#3-redução-antecipada-de-linhas-e-colunas)
4. [Tipos de dados definidos cedo](#4-tipos-de-dados-definidos-cedo)
5. [Table.Buffer e List.Buffer](#5-tablebuffer-e-listbuffer)
6. [Merge e Append eficientes](#6-merge-e-append-eficientes)
7. [Padrões problemáticos](#7-padrões-problemáticos)

---

## 1. Query Folding — o ponto #1 de performance

### O que é Query Folding

Quando você conecta a uma fonte que suporta SQL (SQL Server, PostgreSQL, Oracle, Snowflake, OData, Dataverse, etc.), o Power Query tenta traduzir suas transformações M em SQL nativo para executar **no servidor**, não no seu Power BI Desktop/Gateway. Isso é Query Folding.

**Sem folding:** o Power Query baixa todos os dados brutos do servidor e aplica as transformações localmente (lento, pesado).

**Com folding:** só os dados já transformados vêm, muito menos rede e memória.

### 🔴 Query Folding quebrado = refresh lento

**Como verificar:**

No Power Query Editor, clique com botão direito em cada etapa. Olhe **"View Native Query"** (Exibir Consulta Nativa):

- ✅ **Habilitado** (azul/clicável) → folding OK até essa etapa
- ❌ **Desabilitado** (cinza) → folding quebrou nessa ou numa etapa anterior

### O que sempre quebra folding

| Função/padrão | Comportamento |
|---|---|
| `Table.Buffer` | **Sempre quebra.** Materializa em memória local. |
| `List.Buffer` | **Sempre quebra** (para listas no meio de transformações) |
| Step `Replace Errors` (Table.ReplaceErrorValues) | Geralmente quebra |
| Capitalização/trim em colunas (`Text.Trim`, `Text.Proper`) | Depende da fonte — muitas vezes quebra |
| Merge com query que não tem folding | Quebra |
| `Table.Pivot` / `Table.Unpivot` | Frequentemente quebra (especialmente com nomes dinâmicos) |
| Funções customizadas em `Table.AddColumn` ou `Table.TransformColumns` | **Sempre quebra** |
| Índice (`Table.AddIndexColumn`) | Sempre quebra |
| `Table.Group` com funções complexas | Geralmente quebra |
| Conversão de tipos `Any` → tipo específico | Depende |

### O que preserva folding (geralmente)

- `Table.SelectRows` com condições simples (comparações, IN)
- `Table.SelectColumns` (SELECT colunas)
- `Table.RemoveColumns` (complemento de SELECT)
- `Table.RenameColumns`
- `Table.Sort`
- Merge com outras queries que também têm folding (vira JOIN no SQL)
- `Table.Group` com agregações simples (SUM, AVG, COUNT)
- Filtros de data (`each [Data] >= dataInicial`)
- `Table.Distinct`
- `Table.FirstN` / `Table.LastN` simples

### Regras práticas para preservar folding

1. **Comece pelas etapas que folding gosta:** filtro, seleção de colunas, rename, sort
2. **Deixe etapas "problemáticas" para o FIM** da query: buffer, operações com funções customizadas, replace errors
3. **Se precisa quebrar folding, quebre uma vez só:** evite intercalar folding/não-folding repetidamente
4. **Valide em cada step do projeto:** a cada nova etapa, dê botão direito e veja se View Native Query ainda funciona

### Quando NÃO há folding na fonte

Fontes como **Excel, CSV, JSON local, SharePoint Lists, Web API em alguns casos** não têm SQL engine por trás — então folding não existe. Nessas fontes, todas as transformações rodam localmente e folding não se aplica.

Nesse caso, o foco de performance vira:
- Minimizar linhas/colunas carregadas
- Evitar transformações desnecessárias
- Usar `Table.Buffer` estrategicamente

**Referência:** [Query Folding (Microsoft)](https://learn.microsoft.com/power-query/power-query-folding)

---

## 2. Ordem correta de etapas

### 🟡 A ordem das etapas afeta performance

**Princípio:** o Power Query executa etapas de cima pra baixo. Se você filtra 80% dos dados na etapa 15, mas faz um merge pesado na etapa 3, o merge processou 5x mais dados do que o necessário.

### Ordem ideal (quando possível)

1. **Source** — conexão
2. **Navigation** — selecionar tabela
3. **Select Columns** — eliminar colunas desnecessárias
4. **Filter Rows** — eliminar linhas desnecessárias (filtro por data, status, flag)
5. **Change Type** — tipar colunas
6. **Rename Columns** — padronizar nomes para PascalCase
7. **Merges / Joins** — fazer lookups com outras queries
8. **Expand** — expandir colunas de merge
9. **Add Columns** — colunas calculadas baseadas no resultado anterior
10. **Group By / Pivot / Unpivot** — agregações e pivotagens
11. **Sort / Final Cleanup** — ajustes finais

### Exemplo de má ordem (pesado)

```m
let
    Source = Sql.Database(...),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // ❌ Merge ANTES de filtrar — processa todas as vendas do histórico
    MergeProduto = Table.NestedJoin(vw_vendas, {"produto_id"}, dProduto, {"ProdutoId"}, "prod"),
    ExpandProduto = Table.ExpandTableColumn(MergeProduto, "prod", {"NomeProduto", "Categoria"}),
    
    // ❌ Adicionar coluna calculada antes de filtrar
    AddColunaMargem = Table.AddColumn(ExpandProduto, "Margem", each [valor] - [custo]),
    
    // Só agora filtra — tarde demais
    FiltroData = Table.SelectRows(AddColunaMargem, each [data_venda] >= dataInicial)
in
    FiltroData
```

### Exemplo de boa ordem

```m
let
    Source = Sql.Database(...),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // ✅ Filtra PRIMEIRO (folding gosta disso)
    filtroData = Table.SelectRows(vw_vendas, each [data_venda] >= dataInicial),
    
    // ✅ Remove colunas não necessárias
    colunasSelecionadas = Table.SelectColumns(filtroData, 
        {"venda_id", "produto_id", "cliente_id", "data_venda", "valor", "custo"}),
    
    // ✅ Tipa cedo
    tiposAjustados = Table.TransformColumnTypes(colunasSelecionadas, {
        {"venda_id", Int64.Type},
        {"produto_id", Int64.Type},
        {"cliente_id", Int64.Type},
        {"data_venda", type date},
        {"valor", type number},
        {"custo", type number}
    }),
    
    // Só agora merge (em volume já reduzido)
    mergeProduto = Table.NestedJoin(tiposAjustados, {"produto_id"}, dProduto, {"ProdutoId"}, "prod"),
    expandProduto = Table.ExpandTableColumn(mergeProduto, "prod", {"NomeProduto", "Categoria"}),
    
    // Última etapa: cálculo final
    addMargem = Table.AddColumn(expandProduto, "Margem", each [valor] - [custo], type number)
in
    addMargem
```

**Ganho típico:** 30-70% mais rápido em fontes com folding, pela redução de dados processados.

---

## 3. Redução antecipada de linhas e colunas

### 🟡 Regra de ouro: "Remover antes de transformar"

**Colunas que não vão para o modelo não deveriam sobreviver ao primeiro step de limpeza.** Idem para linhas.

### Colunas

**Problema típico:** tabela fonte com 80 colunas, usuário carrega tudo porque "pode precisar depois". O modelo fica 3x maior que o necessário e o Vertipaq (compressão) sofre.

**Correção:** `Table.SelectColumns` logo após navegação:

```m
let
    Source = Sql.Database(...),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // Só as colunas que o modelo vai usar
    colunasNecessarias = Table.SelectColumns(vw_vendas, {
        "venda_id", "cliente_id", "produto_id",
        "data_venda", "valor", "custo", "quantidade"
    })
    // ... resto
```

Evite `Table.RemoveColumns` longa: é mais legível listar o que FICA do que listar o que SAI.

### Linhas

**Problema típico:** query traz 10 anos de histórico quando o modelo só precisa dos últimos 3. Por segurança ou preguiça.

**Correção:** filtro de data PRIMEIRO:

```m
let
    Source = Sql.Database(...),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // Filtro de data logo após fonte (preserva folding)
    filtroData = Table.SelectRows(vw_vendas, each [data_venda] >= dataInicial),
    
    // ... resto
```

**Caso especial — linhas irrelevantes:** filtre também status inválido, flags de teste, etc.:

```m
filtroStatus = Table.SelectRows(filtroData, each [status] <> "CANCELADA" and [ambiente] <> "TESTE")
```

### Ganho esperado

| Redução | Ganho aproximado |
|---|---|
| 80 colunas → 10 | ~70% menos memória no modelo |
| 10 anos de dados → 3 anos | ~70% menos linhas |
| Filtro de status | Varia (10-30%) |

Tudo isso composto pode reduzir um modelo de 500MB pra 50MB.

---

## 4. Tipos de dados definidos cedo

### 🟡 Tipe as colunas logo após carregar

**Problema:** deixar colunas como `Any` (tipo não definido) até o meio da query força o engine a inferir tipos em cada step, atrasa processamento e pode causar erros sutis.

**Correção:** `Table.TransformColumnTypes` logo após selecionar colunas e filtrar linhas básicas.

**Não use o botão "Detect Data Types" do Power Query** em produção — ele faz detecção por amostragem e pode errar em colunas mistas. Sempre explicite:

```m
tiposAjustados = Table.TransformColumnTypes(colunasSelecionadas, {
    {"venda_id", Int64.Type},
    {"cliente_id", Int64.Type},
    {"data_venda", type date},
    {"data_faturamento", type datetime},
    {"valor", type number},        // Decimal Number
    {"quantidade", Int64.Type},
    {"ativo", type logical},
    {"status", type text}
})
```

### Tipos Power Query mais usados

| Tipo em M | UI no Power Query | Uso |
|---|---|---|
| `Int64.Type` | Whole Number | IDs, contagens |
| `type number` | Decimal Number | Valores monetários, percentuais |
| `Percentage.Type` | Percentage | Valores que serão exibidos como % |
| `type date` | Date | Datas (sem hora) |
| `type datetime` | Date/Time | Timestamp |
| `type time` | Time | Hora isolada |
| `type text` | Text | Strings |
| `type logical` | True/False | Booleanos |
| `Currency.Type` | Fixed Decimal Number | Moeda (maior precisão que `number`) |

### Erro comum: texto com número

Se uma coluna é um código (CPF, CNPJ, CEP, código do produto "00123"), **SEMPRE trate como texto**. Converter pra número remove zeros à esquerda e quebra merges posteriores.

```m
// ❌ Errado
{"cep", Int64.Type}  // "01234-567" vira 1234567, perdeu zero à esquerda

// ✅ Certo
{"cep", type text}
```

---

## 5. Table.Buffer e List.Buffer

### 🟡 Buffer quebra folding — use com consciência

`Table.Buffer(tabela)` materializa a tabela em memória local e **sempre quebra Query Folding**. Não é ruim por si só — é uma ferramenta que resolve problemas específicos.

### Quando Buffer AJUDA

1. **Tabela pequena referenciada múltiplas vezes:**
   ```m
   // dParametros é pequena (50 linhas) mas usada em 15 queries
   // Com Buffer, ela carrega 1x em memória e todas as queries acessam a versão materializada
   dParametrosBuffer = Table.Buffer(dParametros)
   ```

2. **Evitar re-execução em merges subsequentes:**
   ```m
   // tabelaLookup vai ser usada em 3 merges seguidos nessa query
   tabelaLookupBuffer = Table.Buffer(tabelaLookup),
   merge1 = Table.NestedJoin(source, {"key"}, tabelaLookupBuffer, {"Key"}, ...),
   merge2 = Table.NestedJoin(merge1, ...),
   merge3 = Table.NestedJoin(merge2, ...)
   ```

3. **Resolver issues de Privacy Level em Combine:**
   quando duas fontes com Privacy Levels diferentes geram erro "Formula.Firewall", Buffer antes do merge resolve em muitos casos.

4. **Garantir ordenação antes de agrupamento:**
   ```m
   ordenado = Table.Sort(source, {"data"}),
   ordenadoBuffer = Table.Buffer(ordenado),  // Materializa a ordenação
   agrupado = Table.Group(ordenadoBuffer, ...)  // Ordem preservada
   ```

### Quando Buffer PREJUDICA

1. **Primeira etapa em fonte grande com folding:** elimina folding de toda a query.
   ```m
   // ❌ Ruim: folding quebra logo no começo
   Source = Sql.Database(...),
   vendas = Source{[...]}[Data],
   vendasBuffer = Table.Buffer(vendas)  // Não faça isso com tabela grande
   ```

2. **Tabela muito grande:** Buffer carrega tudo na RAM local. Se a tabela tem 50 milhões de linhas, você vai estourar memória.

3. **Sem motivo claro:** copy/paste de stackoverflow achando que "Buffer sempre ajuda".

### List.Buffer

Mesma lógica, mas para listas:

```m
// ✅ Bom uso
__listaClientesVip = List.Buffer(Table.Column(dClienteVip, "ClienteId")),
filtroVip = Table.SelectRows(vendas, each List.Contains(__listaClientesVip, [ClienteId]))
```

Sem Buffer, `List.Contains` rodaria `Table.Column` pra cada linha → desastroso. Com Buffer, a lista é materializada 1x e reutilizada.

---

## 6. Merge e Append eficientes

### 🟡 Merges custam caro, principalmente sem folding

**Regras:**

1. **Filtre as tabelas ANTES do merge** — reduza as duas pontas
2. **Use Inner Join quando possível** — Left Outer traz nulls desnecessários
3. **Mantenha folding nas DUAS tabelas sendo merged** — se uma quebrou, o SQL não consegue mais traduzir
4. **Para merges múltiplos com a mesma tabela, Buffer a tabela direita** — veja seção anterior

### Append — em geral é leve

Append concatena linhas (`Table.Combine({t1, t2, t3})`). Costuma folding bem em SQL (vira UNION ALL) e é barato em memória.

**Uso típico:** combinar múltiplas abas de Excel, múltiplos CSVs, múltiplos arquivos de uma pasta.

### Append vs Merge — qual usar

| Cenário | Use |
|---|---|
| Consolidar vendas de várias filiais com mesma estrutura | Append |
| Adicionar atributos de cliente na fato de vendas | Merge |
| Juntar tabelas de ano anterior com ano atual | Append |
| Fazer lookup de tabela de preço por produto/data | Merge |

### Sempre verifique o resultado do merge

**Cardinalidade inesperada** é o bug mais comum:
- Você espera 1:1 (uma venda → um produto) e o merge traz 1:N porque dProduto tem duplicados
- Resultado: a fato duplica linhas e os totais de vendas ficam inflados

**Como validar:**
```m
// Conta linhas antes
__linhasAntes = Table.RowCount(vendas),
mergeProd = Table.NestedJoin(vendas, {"produto_id"}, dProduto, {"ProdutoId"}, "prod", JoinKind.LeftOuter),
expand = Table.ExpandTableColumn(mergeProd, "prod", {"NomeProduto"}),
// Conta linhas depois
__linhasDepois = Table.RowCount(expand)
```

Se `__linhasDepois > __linhasAntes`, há duplicação no merge. Investigue dProduto.

---

## 7. Padrões problemáticos

### 🔴 `Table.AddColumn` com lógica pesada em loop

```m
// ❌ Ruim: cada linha chama função customizada complexa
comMargem = Table.AddColumn(vendas, "Margem", each 
    fxCalculoComplexo([valor], [custo], [imposto], [desconto])
)
```

Se `fxCalculoComplexo` é pesada, isso executa 1M de vezes para 1M de linhas. Folding sempre quebrado.

**Alternativa:** fazer o cálculo no SQL fonte (view ou stored procedure), ou transformar em cálculo vetorial:

```m
// ✅ Melhor (folding preservado)
comMargem = Table.AddColumn(vendas, "Margem", each [valor] - [custo] - [imposto] + [desconto], type number)
```

### 🔴 `List.Accumulate` para transformar tabela

```m
// ❌ Anti-pattern: gerar coluna via List.Accumulate
comTotalAcumulado = Table.AddColumn(vendas, "TotalAcumulado", each 
    List.Accumulate({1..Table.RowCount(vendas)}, 0, (acc, i) => acc + vendas{i-1}[valor])
)
```

Isso é O(n²) — tempo explode com o volume. Evite.

**Alternativa:** agregação via `Table.Group` ou cálculo DAX (RUNNINGSUM) no modelo em vez de no Power Query.

### 🟡 Múltiplos `Table.AddColumn` em sequência

```m
// ❌ Menos eficiente
c1 = Table.AddColumn(source, "A", each [x] * 2),
c2 = Table.AddColumn(c1, "B", each [x] + [y]),
c3 = Table.AddColumn(c2, "C", each [A] + [B])
```

O engine cria 3 steps intermediários. Melhor combinar quando possível:

```m
// ✅ Mais eficiente
adicionaTodas = Table.AddColumn(source, "A", each [x] * 2, type number),
adicionaB = Table.AddColumn(adicionaTodas, "B", each [x] + [y], type number),
adicionaC = Table.AddColumn(adicionaB, "C", each [A] + [B], type number)
```

(No fim é similar; a diferença real é quando a lógica pode ser fundida num único `Table.TransformColumns` ou `Table.SelectColumns` com cálculo inline.)

### 🟡 `Pivot`/`Unpivot` com colunas dinâmicas

`Table.Pivot(tabela, List.Distinct(...), ...)` é ok mas:
- Quebra folding na maioria das fontes
- Torna a query frágil (se uma nova categoria aparece, a estrutura muda)

**Alternativa:** se for possível, faça pivot no DWH/SQL (view com CASE WHEN).

---

## Checklist rápido de performance

Ao auditar uma query por performance, cheque:

- [ ] Query Folding preservado até onde faz sentido (botão direito em cada step → View Native Query)
- [ ] Filtros aplicados ANTES de merges/expands/add columns pesados
- [ ] Colunas desnecessárias removidas no step 2 ou 3 (não no fim)
- [ ] Tipos definidos logo após fonte (não tardio)
- [ ] `Table.Buffer` usado só onde faz sentido (tabela pequena referenciada múltiplas vezes, resolver folding/privacy issue, forçar ordenação)
- [ ] Sem `List.Accumulate` / `List.Generate` pesadas para transformar colunas
- [ ] Merges com ambas tabelas filtradas antes
- [ ] Funções customizadas em `Table.AddColumn` justificadas (ou movidas para SQL)
- [ ] `Pivot` dinâmico avaliado (considerar migrar pro DWH)

Se os 9 itens acima estão OK, você está no top 10% de performance em Power Query.
