# Transformações em Power Query

Padrões para as transformações mais usadas: Merge vs Append, Group By, Expand, funções de List/Record/Text/Date e funções customizadas. Foco em como fazer "certo" cada operação comum.

## Sumário

1. [Merge (Join) vs Append (Union)](#1-merge-join-vs-append-union)
2. [Group By — agregação eficiente](#2-group-by--agregação-eficiente)
3. [Expand de colunas de merge](#3-expand-de-colunas-de-merge)
4. [List functions essenciais](#4-list-functions-essenciais)
5. [Record functions essenciais](#5-record-functions-essenciais)
6. [Text e Date helpers](#6-text-e-date-helpers)
7. [Funções customizadas avançadas](#7-funções-customizadas-avançadas)
8. [Padrão de tabela calendário](#8-padrão-de-tabela-calendário)

---

## 1. Merge (Join) vs Append (Union)

### Merge — quando juntar colunas de tabelas diferentes

Merge = JOIN do SQL. Combina colunas de duas tabelas com base em chave(s). Usado para lookup/enrichment.

**Tipos de Join:**

| JoinKind | Equivalente SQL | Uso |
|---|---|---|
| `JoinKind.LeftOuter` | LEFT JOIN | Manter todas as linhas da esquerda, trazer matches da direita (NULL se não achar) |
| `JoinKind.RightOuter` | RIGHT JOIN | Inverso do Left — raramente usado |
| `JoinKind.FullOuter` | FULL OUTER | Todas as linhas dos dois lados |
| `JoinKind.Inner` | INNER JOIN | Apenas linhas com match em ambos os lados |
| `JoinKind.LeftAnti` | NOT IN | Linhas da esquerda SEM match na direita (útil para diff/validação) |
| `JoinKind.RightAnti` | NOT IN (inverso) | Linhas da direita SEM match na esquerda |

**Padrão recomendado:**

```m
let
    Source = fVendas,
    
    mergeCliente = Table.NestedJoin(
        Source,              // Tabela esquerda (fato)
        {"ClienteId"},       // Colunas de chave esquerda
        dCliente,            // Tabela direita (dim)
        {"ClienteId"},       // Colunas de chave direita
        "cliente",           // Nome da coluna aninhada gerada
        JoinKind.Inner       // Tipo de join (Inner se a chave é sempre válida)
    ),
    
    expandCliente = Table.ExpandTableColumn(
        mergeCliente,
        "cliente",
        {"NomeCliente", "UfCliente"},  // Colunas a expandir
        {"NomeCliente", "UfCliente"}   // Nomes finais (pode renomear aqui)
    )
in
    expandCliente
```

**Dicas:**

- Use **Inner Join por padrão** quando a chave é obrigatória (FK válida sempre aponta para PK existente). Só use Left Outer quando há legitimamente chaves sem match.
- Sempre **filtre antes** do merge — reduz as duas tabelas.
- Nomeie a coluna de merge com prefixo claro (`cliente`, `produto`) em vez de "NewColumn" padrão.
- No Expand, renomeie colunas na hora (último parâmetro) para evitar conflitos.

### Append — quando empilhar linhas de tabelas iguais

Append = UNION ALL do SQL. Empilha tabelas com **mesma estrutura** (mesmas colunas e tipos).

**Uso típico:** consolidar vendas de múltiplas filiais, anos, arquivos CSV, abas de Excel.

```m
let
    vendasFilialA = ...,
    vendasFilialB = ...,
    vendasFilialC = ...,
    
    vendasConsolidadas = Table.Combine({vendasFilialA, vendasFilialB, vendasFilialC})
in
    vendasConsolidadas
```

**Cuidados:**

- Se as tabelas têm estrutura ligeiramente diferente (ex.: filial A tem coluna `Imposto` que filial B não tem), o `Table.Combine` gera `null` onde a coluna falta. Isso pode ser OK ou esconder bug. Padronize antes.
- Use `Table.Combine` com lista de tabelas, não faça 10 appends aninhados.
- Append de uma **pasta inteira** (Folder connector) é o padrão ideal para muitos CSVs: conecta na pasta e o Power Query já monta uma query que faz append automaticamente de todos os arquivos.

### Merge vs Append — resumo

| Você tem... | Use |
|---|---|
| Fato vendas + dim clientes | Merge (join) |
| Vendas_2023 + Vendas_2024 + Vendas_2025 | Append |
| Tabela principal + lookup de cotação de dólar | Merge |
| 20 arquivos Excel com mesma estrutura | Append (via Folder connector) |
| Produtos + categoria de produto (snowflake) | Merge |

---

## 2. Group By — agregação eficiente

### Quando usar Group By no Power Query

Para **pré-agregar** dados antes de carregar no modelo. Útil quando:
- Você não precisa do detalhe linha-a-linha no modelo (ex.: só precisa de vendas mensais, não cada venda)
- A fonte original é grande demais (bilhões de linhas) e agregar reduz o volume significativamente

### Padrão básico — agregações simples

```m
let
    Source = fVendas,
    
    agrupadoMensal = Table.Group(
        Source,
        {"AnoMes", "FilialId"},     // Colunas de agrupamento
        {
            {"TotalVendas", each List.Sum([Valor]), type number},
            {"QtdPedidos", each Table.RowCount(_), Int64.Type},
            {"TicketMedio", each List.Average([Valor]), type number}
        }
    )
in
    agrupadoMensal
```

**Dica de folding:** agregações simples (SUM, COUNT, AVG, MIN, MAX) geralmente folding em SQL. Agregações complexas (funções customizadas dentro) quase sempre quebram.

### Group By "all rows" — quando precisa do detalhe

Se você precisa fazer agregação + manter acesso às linhas originais (ex.: ranquear dentro do grupo), use o segundo parâmetro aninhado:

```m
agrupadoComDetalhe = Table.Group(
    Source,
    {"ClienteId"},
    {
        {"TotalCliente", each List.Sum([Valor]), type number},
        {"Detalhe", each _, type table}  // Mantém tabela original do grupo
    }
)
```

A coluna `Detalhe` fica como nested table — você pode expandir depois com `Table.ExpandTableColumn`.

### Group By ordenado (first/last do grupo)

Para pegar "primeiro" ou "último" registro de cada grupo (ex.: último status de cada pedido):

```m
let
    Source = fPedidos,
    ordenado = Table.Sort(Source, {{"PedidoId", Order.Ascending}, {"DataStatus", Order.Descending}}),
    
    agrupado = Table.Group(
        ordenado,
        {"PedidoId"},
        {
            {"UltimoStatus", each List.First([Status]), type text},
            {"DataUltimoStatus", each List.First([DataStatus]), type date}
        }
    )
in
    agrupado
```

**Cuidado:** a ordenação é crítica. Se não ordenar primeiro, `List.First` não garante ser o "último cronologicamente".

---

## 3. Expand de colunas de merge

### Renomeie colunas na hora do expand

Após um merge, você tem uma coluna aninhada (type: table ou record). Ao expandir, renomeie para padrão PascalCase direto:

```m
// ❌ Pior: expand default, depois rename
mergeCliente = Table.NestedJoin(Source, {"ClienteId"}, dCliente, {"ClienteId"}, "cliente", JoinKind.Inner),
expandCliente = Table.ExpandTableColumn(mergeCliente, "cliente", {"NomeCliente", "UfCliente"}),
// Agora preciso renomear, mais 1 step
renomeado = Table.RenameColumns(expandCliente, {{"NomeCliente", "NomeDoCliente"}, {"UfCliente", "EstadoCliente"}})

// ✅ Melhor: expand + rename em um step
expandCliente = Table.ExpandTableColumn(
    mergeCliente,
    "cliente",
    {"NomeCliente", "UfCliente"},          // Colunas de origem
    {"NomeDoCliente", "EstadoCliente"}     // Novos nomes
)
```

### Expand parcial — só o que precisa

Não expanda colunas que você não vai usar. Na UI, o botão de expand vem com TODAS marcadas por padrão. Desmarque as desnecessárias.

---

## 4. List functions essenciais

### `List.Contains` — existe na lista?

```m
__listaUfsSul = {"SC", "RS", "PR"},
filtroSul = Table.SelectRows(Source, each List.Contains(__listaUfsSul, [Uf]))
```

**Dica:** para lista grande reutilizada em múltiplos checks na mesma query, use `List.Buffer`.

### `List.Distinct` — remover duplicados

```m
uniqueClientes = List.Distinct(Table.Column(fVendas, "ClienteId"))
```

### `List.Sum`, `List.Average`, `List.Max`, `List.Min`, `List.Count`

Usados dentro de `Table.Group` e `Table.AddColumn`:

```m
totalCliente = Table.AddColumn(dCliente, "TotalVendas", each 
    List.Sum(Table.SelectRows(fVendas, (r) => r[ClienteId] = [ClienteId])[Valor])
)
```

**⚠️ Cuidado:** esse padrão é O(n*m) — para cada cliente, varre fVendas inteira. Para modelos grandes, é melhor usar `Table.Group` ou fazer a agregação em DAX (medida).

### `List.Accumulate` — reduzir lista

Útil para cálculos acumulados simples. Anti-padrão para transformar colunas (ver seção "Padrões problemáticos" em performance.md).

```m
// Somar lista
__total = List.Accumulate({1, 2, 3, 4, 5}, 0, (acc, cur) => acc + cur)  // Retorna 15
```

### `List.Generate` — criar lista recursivamente

```m
// Gerar próximos dias úteis a partir de uma data
proximos10DiasUteis = List.Generate(
    () => [data = dataInicial, diaUtilCount = 0],
    each [diaUtilCount] < 10,
    each [
        data = Date.AddDays([data], 1),
        diaUtilCount = if Date.DayOfWeek([data], Day.Monday) < 5 then [diaUtilCount] + 1 else [diaUtilCount]
    ],
    each [data]
)
```

---

## 5. Record functions essenciais

Records são "objetos" em M (like JSON objects). Usados quando você navega em APIs, dados aninhados, ou trabalha com metadados.

### `Record.Field` e `Record.FieldOrDefault`

```m
// Pegar campo (erro se não existir)
valor = Record.Field(meuRecord, "Nome")

// Pegar com default (seguro para campos opcionais de API)
valorSeguro = Record.FieldOrDefault(meuRecord, "Nome", "Sem nome")
```

**Uso típico em APIs:** o JSON retornado tem campos opcionais que às vezes vêm, às vezes não. `Record.FieldOrDefault` evita erro de query.

### `Record.ToTable` — de record para tabela

Útil quando você tem um record e quer desmontar em tabela chave-valor:

```m
tabelaMetadata = Record.ToTable([Nome = "Gerson", Cidade = "Schroeder", Uf = "SC"])
// Retorna: {Name = "Nome", Value = "Gerson"}, {Name = "Cidade", Value = "Schroeder"}, ...
```

### `Record.TransformFields`

Transforma múltiplos campos de um record:

```m
recordLimpo = Record.TransformFields(meuRecord, {
    {"Nome", Text.Upper},
    {"Cpf", each Text.Select(_, {"0".."9"})}
})
```

---

## 6. Text e Date helpers

### Text functions essenciais

```m
Text.Upper("gerson")                 // → "GERSON"
Text.Lower("GERSON")                 // → "gerson"
Text.Proper("gerson viergutz")       // → "Gerson Viergutz"
Text.Trim("  olá  ")                  // → "olá"
Text.TrimStart("  olá")              // → "olá"
Text.TrimEnd("olá  ")                // → "olá"
Text.Length("olá")                   // → 3
Text.Start("Dashmaker", 4)           // → "Dash"
Text.End("Dashmaker", 5)             // → "maker"
Text.Middle("Dashmaker", 4, 3)       // → "mak"
Text.Contains("Gerson", "son")       // → true
Text.Replace("0123-456", "-", "")    // → "0123456"
Text.Select("(11) 98765-4321", {"0".."9"})  // → "11987654321"
Text.Split("a,b,c", ",")             // → {"a", "b", "c"}
Text.Combine({"a", "b", "c"}, "-")   // → "a-b-c"
Text.PadStart("5", 3, "0")           // → "005"
Text.PadEnd("5", 3, "0")             // → "500"
```

### Date functions essenciais

```m
Date.Year(#date(2026, 4, 23))         // → 2026
Date.Month(#date(2026, 4, 23))        // → 4
Date.Day(#date(2026, 4, 23))          // → 23
Date.DayOfWeek(#date(2026, 4, 23), Day.Monday)   // → 3 (quinta, com segunda como 0)
Date.DayOfYear(#date(2026, 4, 23))    // → 113
Date.WeekOfYear(#date(2026, 4, 23))   // → 17
Date.QuarterOfYear(#date(2026, 4, 23))  // → 2
Date.StartOfMonth(#date(2026, 4, 23))   // → #date(2026, 4, 1)
Date.EndOfMonth(#date(2026, 4, 23))     // → #date(2026, 4, 30)
Date.AddDays(#date(2026, 4, 23), 7)     // → #date(2026, 4, 30)
Date.AddMonths(#date(2026, 4, 23), -1)  // → #date(2026, 3, 23)
Date.AddYears(#date(2026, 4, 23), 1)    // → #date(2027, 4, 23)
Date.ToText(#date(2026, 4, 23), "dd/MM/yyyy")   // → "23/04/2026"
Date.ToText(#date(2026, 4, 23), "MMMM yyyy")    // → "April 2026" (local default)
Date.ToText(#date(2026, 4, 23), "dd/MM/yyyy", "pt-BR")  // → "23/04/2026"
```

**Dica:** sempre passe o `culture` em `Date.ToText` para garantir PT-BR (`"pt-BR"`), senão pode retornar inglês em ambientes com locale diferente.

---

## 7. Funções customizadas avançadas

### Template básico

```m
// fxNomeDaFuncao: descrição curta do que faz
(param1 as [tipo], param2 as [tipo], ...) as [tipoRetorno] =>
let
    __passo1 = ...,
    __passo2 = ...,
    __resultado = ...
in
    __resultado
```

### Parâmetros opcionais

Use `optional` antes do nome:

```m
// fxFormatarCnpj: formata CNPJ. Se não vier, retorna vazio.
(optional cnpj as nullable text) as text =>
let
    __limpo = if cnpj = null then "" 
              else Text.Select(cnpj, {"0".."9"}),
    __padronizado = if Text.Length(__limpo) <> 14 then ""
                    else Text.PadStart(__limpo, 14, "0"),
    __formatado = if __padronizado = "" then ""
                  else Text.Middle(__padronizado, 0, 2) & "." &
                       Text.Middle(__padronizado, 2, 3) & "." &
                       Text.Middle(__padronizado, 5, 3) & "/" &
                       Text.Middle(__padronizado, 8, 4) & "-" &
                       Text.Middle(__padronizado, 12, 2)
in
    __formatado
```

### Documentação via metadados

Power Query suporta "type ascription" que gera documentação na UI:

```m
fxGeraCalendario = (dataInicio as date, dataFim as date) as table =>
let
    // ... implementação ...
in
    calendario;

// Documentação
fxGeraCalendarioTyped = Value.ReplaceType(fxGeraCalendario, type function (
    dataInicio as (type date meta [Documentation.FieldCaption = "Data Inicial", Documentation.SampleValues = {#date(2020,1,1)}]),
    dataFim as (type date meta [Documentation.FieldCaption = "Data Final", Documentation.SampleValues = {#date(2030,12,31)}])
) as table meta [
    Documentation.Name = "fxGeraCalendario",
    Documentation.LongDescription = "Gera tabela calendário completa entre duas datas com colunas Ano, Mes, MesNum, Trimestre, DiaSemana.",
    Documentation.Examples = {[
        Description = "Gera calendário de 2024",
        Code = "fxGeraCalendario(#date(2024,1,1), #date(2024,12,31))",
        Result = "Tabela com 366 linhas"
    ]}
])
```

Isso faz a função aparecer no painel de Funções com descrição e exemplos clicáveis.

---

## 8. Padrão de tabela calendário

Todo projeto Power BI precisa de uma. Aqui está o template Dashmaker completo:

```m
// fxGeraCalendario: gera tabela calendário completa
(dataInicio as date, dataFim as date) as table =>
let
    // Calcula quantos dias entre início e fim
    __totalDias = Duration.Days(dataFim - dataInicio) + 1,
    
    // Gera lista de todas as datas
    __listaDatas = List.Dates(dataInicio, __totalDias, #duration(1, 0, 0, 0)),
    
    // Converte para tabela
    __tabelaBase = Table.FromList(__listaDatas, Splitter.SplitByNothing(), {"Data"}),
    
    // Adiciona colunas derivadas
    __comAno = Table.AddColumn(__tabelaBase, "Ano", each Date.Year([Data]), Int64.Type),
    __comMesNum = Table.AddColumn(__comAno, "MesNum", each Date.Month([Data]), Int64.Type),
    __comMesNome = Table.AddColumn(__comMesNum, "MesNome", each Date.ToText([Data], "MMMM", "pt-BR"), type text),
    __comMesAbrev = Table.AddColumn(__comMesNome, "MesAbrev", each Date.ToText([Data], "MMM", "pt-BR"), type text),
    __comAnoMes = Table.AddColumn(__comMesAbrev, "AnoMes", each Date.ToText([Data], "yyyy-MM"), type text),
    __comTrimestre = Table.AddColumn(__comAnoMes, "Trimestre", each "T" & Text.From(Date.QuarterOfYear([Data])), type text),
    __comSemestre = Table.AddColumn(__comTrimestre, "Semestre", each "S" & Text.From(if Date.Month([Data]) <= 6 then 1 else 2), type text),
    __comDiaSemana = Table.AddColumn(__comSemestre, "DiaSemanaNum", each Date.DayOfWeek([Data], Day.Sunday) + 1, Int64.Type),
    __comDiaSemanaNome = Table.AddColumn(__comDiaSemana, "DiaSemanaNome", each Date.ToText([Data], "dddd", "pt-BR"), type text),
    __comDiaUtil = Table.AddColumn(__comDiaSemanaNome, "EhDiaUtil", each Date.DayOfWeek([Data], Day.Monday) < 5, type logical),
    __comDiaMes = Table.AddColumn(__comDiaUtil, "DiaMes", each Date.Day([Data]), Int64.Type),
    __comDiaAno = Table.AddColumn(__comDiaMes, "DiaAno", each Date.DayOfYear([Data]), Int64.Type),
    __comSemanaAno = Table.AddColumn(__comDiaAno, "SemanaAno", each Date.WeekOfYear([Data]), Int64.Type),
    
    // Tipo final na coluna Data
    __final = Table.TransformColumnTypes(__comSemanaAno, {{"Data", type date}})
in
    __final
```

**Como usar:**

1. Criar parâmetros `dataInicial` e `dataFinal` (tipo Date)
2. Criar a função `fxGeraCalendario` no grupo Funções
3. Criar query `dCalendario`:
   ```m
   dCalendario = fxGeraCalendario(dataInicial, dataFinal)
   ```
4. Opcional: criar versão com data máxima dinâmica:
   ```m
   dCalendarioDinamico = fxGeraCalendario(dataInicial, Date.From(DateTime.LocalNow()))
   ```

**Passos adicionais no modelo:**
- Marcar `dCalendario` como Date Table no Power BI Desktop
- Ocultar colunas técnicas (`MesNum`, `DiaSemanaNum`) e configurar Sort By em `MesNome` (ordenar por `MesNum`)

---

## Quando usar DAX em vez de Power Query

Power Query é ótimo para transformação de dados **na fonte/ETL**. Mas algumas coisas é melhor deixar em DAX:

| Tarefa | Onde fazer | Por quê |
|---|---|---|
| Agregação por dimensão (SUM por cliente) | DAX (medida) | Flexibilidade de contexto de filtro no visual |
| Ranking dinâmico | DAX (RANKX) | Reativo ao slicer |
| Cálculo baseado em período (YoY, MTD) | DAX (time intelligence) | Precisa da tabela calendário marcada |
| Limpeza de texto, padronização | Power Query | Fica barato na carga, não precisa recalcular |
| Merge/enrichment de tabelas | Power Query | Uma vez só, fica no modelo |
| Renomeação de colunas para modelo | Power Query | Impacta todo o modelo |
| Coluna calculada com lógica simples | Qualquer um | Prefira Power Query por compressão melhor |
| Coluna que depende de contexto de visual | DAX | Power Query é estático |

**Regra geral:** tudo que é transformação fixa do dado → Power Query. Tudo que depende de filtro/slicer/usuário → DAX.
