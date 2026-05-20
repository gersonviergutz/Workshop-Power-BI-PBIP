# Organização de Projeto Power Query

Estrutura de grupos, uso de parâmetros, Enable Load, separação Fonte → Staging → Modelo, renomeação de etapas e comentários. Aqui mora a maior diferença entre um projeto que cresce bem e um que vira "bagunça em M".

## Sumário

1. [Grupos e pastas no painel de queries](#1-grupos-e-pastas-no-painel-de-queries)
2. [Uso de parâmetros](#2-uso-de-parâmetros)
3. [Enable Load em queries intermediárias](#3-enable-load-em-queries-intermediárias)
4. [Padrão Fonte → Staging → Modelo](#4-padrão-fonte--staging--modelo)
5. [Renomear etapas (steps)](#5-renomear-etapas-steps)
6. [Comentários em M](#6-comentários-em-m)
7. [Funções customizadas](#7-funções-customizadas)

---

## 1. Grupos e pastas no painel de queries

### 🟡 Toda query deve estar em um grupo

**Problema:** Projeto com 30+ queries na raiz (sem agrupamento) vira caos. Scroll infinito, impossível achar onde está o bug, impossível saber o que é dim, o que é fato, o que é intermediário.

**Correção:** Organize em grupos (pastas) via botão direito no painel de queries → "New Group".

**Estrutura recomendada para projetos médios/grandes:**

```
📁 Parâmetros
   • dataInicial
   • dataFinal
   • caminhoServidor
   • nomeAmbiente
   • RangeStart
   • RangeEnd

📁 Funções
   • fxGeraCalendario
   • fxLimpaTexto
   • fxConverteMoeda

📁 Fontes
   • srcVendasSQL
   • srcClientesExcel
   • srcEstoqueAPI

📁 Staging
   • stgVendasTratado
   • stgClientesMerge
   • stgEstoquePivot

📁 Dimensões (carregadas no modelo)
   • dCalendario
   • dCliente
   • dProduto

📁 Fatos (carregadas no modelo)
   • fVendas
   • fEstoque
```

**Para projetos pequenos** (até ~15 queries):

```
📁 Parâmetros
📁 Funções
📁 Modelo (com dims e fatos juntos)
```

**Princípio:** cada grupo responde a UMA pergunta rápida. "Onde estão os parâmetros?" → pasta Parâmetros. "Onde fica o tratamento intermediário?" → Staging.

---

## 2. Uso de parâmetros

### 🔴 Nunca hard-code valores críticos

**Problema:** Query com servidor, caminho, filial, ambiente hard-coded no meio do código M. Quando precisar apontar para outro servidor (ex.: desenvolvimento → produção), você precisa editar todas as queries uma por uma. Erro garantido.

**Exemplo ruim:**
```m
let
    Source = Sql.Database("sql-prod-01.empresa.com.br", "dw_vendas"),
    Vendas = Source{[Schema="dbo", Item="fVendas"]}[Data],
    Filtrado = Table.SelectRows(Vendas, each [Data] >= #date(2024, 1, 1))
in
    Filtrado
```

**Corrigido com parâmetros:**
```m
// Parâmetro: servidorSQL = "sql-prod-01.empresa.com.br"
// Parâmetro: bancoSQL = "dw_vendas"
// Parâmetro: dataCorte = #date(2024, 1, 1)

let
    Source = Sql.Database(servidorSQL, bancoSQL),
    Vendas = Source{[Schema="dbo", Item="fVendas"]}[Data],
    Filtrado = Table.SelectRows(Vendas, each [Data] >= dataCorte)
in
    Filtrado
```

### 🟡 Parâmetros típicos que TODO projeto deveria ter

| Parâmetro | Tipo | Uso |
|---|---|---|
| `servidorSQL` | Text | Nome/IP do servidor |
| `bancoSQL` | Text | Nome do banco de dados |
| `caminhoServidor` | Text | Pasta compartilhada base (`\\srv\compartilhado\`) |
| `nomeAmbiente` | Text | `"Desenvolvimento"`, `"Homologacao"`, `"Producao"` |
| `dataInicial` | Date | Corte inferior de dados (para limitar histórico) |
| `dataFinal` | Date | Corte superior (geralmente `DateTime.LocalNow()` em dinâmica) |
| `RangeStart` | DateTime | Incremental Refresh (fixo) |
| `RangeEnd` | DateTime | Incremental Refresh (fixo) |

**Como criar:** Power Query Editor → Home → Manage Parameters → New Parameter.

**Boa prática extra:** use parâmetros tipados (Text, Date, Decimal Number) em vez de apenas Text — evita bugs de comparação e formatação.

### 🔵 Parâmetros de configuração (lookup em tabela)

Para cenários com múltiplos ambientes ou parametrizações complexas, crie uma `dParametros` como query:

```m
let
    Source = Excel.Workbook(File.Contents(caminhoConfig), null, true),
    Parametros = Source{[Item="Config"]}[Data],
    #"Tipo Alterado" = Table.TransformColumnTypes(Parametros, {{"Chave", type text}, {"Valor", type text}})
in
    #"Tipo Alterado"
```

E consulte valores via função:
```m
// fxLerParametro: função utilitária
(chave as text) as any =>
let
    valor = Table.SelectRows(dParametros, each [Chave] = chave){0}[Valor]
in
    valor
```

Útil quando os parâmetros mudam com frequência sem precisar abrir o Power Query.

---

## 3. Enable Load em queries intermediárias

### 🟡 Queries de staging/auxiliares devem ter Enable Load DESMARCADO

**Problema:** Toda query com "Enable load" marcado é carregada no modelo semântico, ocupando memória e poluindo o Field List. Queries intermediárias (staging, fontes brutas, queries de lookup) não deveriam ir para o modelo.

**Como identificar:** no painel de queries, o nome em itálico indica Enable Load desmarcado.

**Como corrigir:** botão direito na query → desmarque "Enable load".

**Regra prática:**

| Tipo de query | Enable Load |
|---|---|
| Fonte bruta (`srcVendasSQL`) | ❌ Desmarcado |
| Staging (`stgVendasTratado`) | ❌ Desmarcado |
| Dimensão final (`dCliente`) | ✅ Marcado |
| Fato final (`fVendas`) | ✅ Marcado |
| Parâmetro | N/A (parâmetros não carregam) |
| Função (`fxLimpaTexto`) | N/A (funções não carregam) |
| Lookup auxiliar não usado direto | ❌ Desmarcado |

**Cuidado comum:** desmarcar Enable Load em uma query que outra query depende (via Reference) NÃO faz a dependente quebrar — ela continua funcionando porque a referência é resolvida em build time. Só significa que a intermediária não vai para o modelo.

---

## 4. Padrão Fonte → Staging → Modelo

### 🟡 Separe responsabilidades em camadas de queries

**Problema:** Uma única query fazendo tudo (conexão + limpeza + merge + renomeação + tipagem + formatação final) vira impossível de manter. Quando precisa mudar um filtro, você revisa 40 steps pra entender onde fazer.

**Solução: três camadas de queries.**

#### Camada 1 — Fonte (srcXXX ou sem prefixo específico)

- Só conecta com a origem e traz o bruto
- **Poucos ou zero steps** de transformação
- Enable Load **desmarcado**
- Nome prefixado: `srcVendasSQL`, `srcClientesExcel`, ou apenas os table references que vêm do banco sem wrap

**Exemplo:**
```m
let
    Source = Sql.Database(servidorSQL, bancoSQL),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data]
in
    vw_vendas
```

#### Camada 2 — Staging (stgXXX)

- Recebe dados de uma Fonte (via Reference)
- Faz **tratamento, limpeza, merges, tipagem**
- Enable Load **desmarcado**
- Nome prefixado: `stgVendasTratado`, `stgClientesMerge`

**Exemplo:**
```m
let
    Source = srcVendasSQL,
    ColunasSelecionadas = Table.SelectColumns(Source, {"venda_id", "cliente_id", "data_venda", "valor"}),
    TipoAlterado = Table.TransformColumnTypes(ColunasSelecionadas, {
        {"venda_id", Int64.Type},
        {"cliente_id", Int64.Type},
        {"data_venda", type date},
        {"valor", type number}
    }),
    MergeClientes = Table.NestedJoin(TipoAlterado, {"cliente_id"}, dCliente, {"ClienteId"}, "Cliente", JoinKind.LeftOuter),
    ExpandNome = Table.ExpandTableColumn(MergeClientes, "Cliente", {"NomeCliente"}, {"NomeCliente"})
in
    ExpandNome
```

#### Camada 3 — Modelo (dXXX, fXXX)

- Recebe Staging (via Reference)
- Faz **seleção final de colunas, renomeação para PascalCase, ordenação**
- Enable Load **marcado**
- Nome prefixado: `dCliente`, `fVendas`

**Exemplo:**
```m
let
    Source = stgVendasTratado,
    ColunasFinais = Table.SelectColumns(Source, {"venda_id", "cliente_id", "data_venda", "valor", "NomeCliente"}),
    Renomeado = Table.RenameColumns(ColunasFinais, {
        {"venda_id", "VendaId"},
        {"cliente_id", "ClienteId"},
        {"data_venda", "DataVenda"},
        {"valor", "ValorTotal"}
    })
in
    Renomeado
```

**Benefícios dessa separação:**
- **Reuso:** várias fatos podem referenciar a mesma `stgClientesMerge`
- **Troubleshoot:** sabe exatamente em qual camada procurar um problema
- **Query Folding:** a camada Fonte costuma manter folding até onde possível
- **Refresh controlado:** fontes pesadas podem ter tratamento em cache intermediário

**Quando NÃO usar essa separação:** projetos pequenos (<10 queries) ou queries triviais (só conectar e carregar). Força não compensa.

---

## 5. Renomear etapas (steps)

### 🟡 Etapas com nomes default = dívida técnica

**Problema:** Queries com steps nomeados `Changed Type1`, `Removed Columns3`, `Filtered Rows7`, `Added Custom12` são ilegíveis. Pra entender a query, você precisa ler cada step e inferir o que faz. Multiplica por 20 queries e é o seu dia inteiro perdido.

**Correção:** renomeie TODA etapa para algo descritivo. Use botão direito na etapa → Rename.

**Padrão sugerido (em PT-BR, descritivo):**

| Step default | Renomeado |
|---|---|
| `Source` | `Conexao` ou deixe `Source` (é idiomático) |
| `Navigation` | `TabelaFonte` |
| `Changed Type` | `TiposAjustados` |
| `Removed Columns` | `ColunasRemovidas` (ou algo mais específico: `RemoveColunasAuditoria`) |
| `Filtered Rows` | `FiltroDataValida` (ou outro filtro específico) |
| `Added Custom` | `AddColunaMargem` |
| `Grouped Rows` | `AgrupadoPorCliente` |
| `Merged Queries` | `MergeComProduto` |
| `Expanded Product` | `ExpandeNomeProduto` |

**Convenção Dashmaker:** use camelCase para steps (consistente com queries/parâmetros), sem acentos e sem espaços.

**Exemplo antes:**
```m
let
    Source = Sql.Database(...),
    #"Navigation" = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Navigation, ...),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [Data] >= dataInicial),
    #"Removed Columns" = Table.RemoveColumns(#"Filtered Rows", {"ColunaAuditoria1", "ColunaAuditoria2"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Removed Columns", ...)
in
    #"Changed Type1"
```

**Depois:**
```m
let
    Source = Sql.Database(...),
    tabelaFonte = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    tiposAjustados = Table.TransformColumnTypes(tabelaFonte, ...),
    filtroDataInicial = Table.SelectRows(tiposAjustados, each [Data] >= dataInicial),
    removeAuditoria = Table.RemoveColumns(filtroDataInicial, {"ColunaAuditoria1", "ColunaAuditoria2"}),
    tiposFinais = Table.TransformColumnTypes(removeAuditoria, ...)
in
    tiposFinais
```

**Efeito colateral bom:** identificadores sem `#"..."` ficam muito mais limpos. A sintaxe `#"Filtered Rows"` só é necessária quando o nome tem espaço/caractere especial. camelCase sem espaço dispensa isso.

---

## 6. Comentários em M

### 🔵 Comente o "porquê", não o "o quê"

**Problema:** Power Query é self-documenting em 70% dos casos (o nome da função diz o que faz). Mas há lógica não-óbvia que precisa ser explicada, especialmente regras de negócio e workarounds.

**Sintaxe de comentários em M:**
- `// comentário de uma linha`
- `/* comentário de múltiplas linhas */`

**Onde usar:**

✅ **Sim — comentar:**
- Regra de negócio específica: `// Filial 99 é código interno de ajuste, não considerar nas vendas`
- Workaround: `// Table.Buffer aqui porque o merge com dClientes estava duplicando linhas sem isso`
- Dependência externa: `// Depende da view vw_vendas_ajustada que é atualizada pelo ETL do SAP toda madrugada`
- Decisões de performance: `// Ordenado por data aqui para otimizar o merge subsequente`

❌ **Não — não comentar:**
- O óbvio: `// Remove colunas` em cima de `Table.RemoveColumns(...)` (redundante)
- Cada step (polui o código)

**Exemplo bem comentado:**
```m
let
    Source = Sql.Database(servidorSQL, bancoSQL),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // Filtro inicial: filial 99 é código interno de ajuste, nunca vai pro dashboard
    semFilialAjuste = Table.SelectRows(vw_vendas, each [filial] <> 99),
    
    // Buffer aqui porque o merge com dClientes abaixo estava duplicando linhas
    // sem a materialização prévia (investigado em 2024-10, ver issue #123)
    vendasBuffer = Table.Buffer(semFilialAjuste),
    
    mergeClientes = Table.NestedJoin(vendasBuffer, {"cliente_id"}, dCliente, {"ClienteId"}, "cli", JoinKind.Inner)
in
    mergeClientes
```

### 🔵 Descrição da query (via UI)

Além de comentários inline, use a propriedade **Description** da query (botão direito → Properties → Description). Aparece no tooltip do Field List no Power BI Desktop e serve como documentação "visível".

Bom conteúdo para Description:
- Fonte dos dados
- Granularidade (1 linha = ?)
- Filtros aplicados
- Dependências externas
- Atualização (diária? mensal?)

---

## 7. Funções customizadas

### 🟡 Use funções para lógica reutilizável

**Problema:** Mesma lógica de tratamento (ex.: limpeza de CNPJ, conversão de moeda, padronização de UF) repetida em 5+ queries. Quando a regra muda, você edita 5 lugares e esquece um.

**Solução:** extraia em função customizada, prefixo `fx`.

**Exemplo — função de limpeza de CNPJ:**

```m
// fxLimpaCnpj: remove caracteres não numéricos de CNPJ e padroniza para 14 dígitos
(cnpj as nullable text) as nullable text =>
let
    limpo = if cnpj = null then null
            else Text.Select(cnpj, {"0".."9"}),
    padronizado = if limpo = null or Text.Length(limpo) = 0 then null
                  else Text.PadStart(limpo, 14, "0")
in
    padronizado
```

**Uso:**
```m
let
    Source = ...,
    CnpjLimpo = Table.AddColumn(Source, "CnpjNormalizado", each fxLimpaCnpj([cnpj_original]))
in
    CnpjLimpo
```

### Convenções para funções

- **Nome:** `fx` + camelCase (`fxLimpaCnpj`, `fxGeraCalendario`, `fxConverteMoeda`)
- **Grupo:** sempre em pasta "Funções"
- **Enable Load:** N/A (funções não carregam)
- **Documentação:** use comentário inline no início (como no exemplo acima)
- **Parâmetros:** tipados (`cnpj as nullable text`, não `cnpj as any`) — ajuda o Editor a detectar erros e habilita tooltip com tipo esperado
- **Retorno:** tipado quando possível (`as nullable text`, `as number`)

### Funções "clássicas" para ter em todo projeto

- `fxGeraCalendario(dataInicio, dataFim)` — cria tabela calendário completa
- `fxLimpaTexto(texto)` — normaliza texto (trim, upper/lower, remove acentos)
- `fxConverteMoeda(valor, moedaOrigem, moedaDestino, data)` — conversão com lookup de câmbio

**Padrão de calendário Dashmaker** (template útil):
```m
// fxGeraCalendario: gera tabela calendário entre duas datas
(dataInicio as date, dataFim as date) as table =>
let
    __totalDias = Duration.Days(dataFim - dataInicio) + 1,
    __listaDatas = List.Dates(dataInicio, __totalDias, #duration(1, 0, 0, 0)),
    __tabela = Table.FromList(__listaDatas, Splitter.SplitByNothing(), {"Data"}),
    __comColunas = Table.AddColumn(__tabela, "Ano", each Date.Year([Data]), Int64.Type),
    __comMes = Table.AddColumn(__comColunas, "MesNum", each Date.Month([Data]), Int64.Type),
    __comMesNome = Table.AddColumn(__comMes, "MesNome", each Date.ToText([Data], "MMMM"), type text),
    __comTrimestre = Table.AddColumn(__comMesNome, "Trimestre", each "T" & Text.From(Date.QuarterOfYear([Data])), type text),
    __comDiaSemana = Table.AddColumn(__comTrimestre, "DiaSemana", each Date.DayOfWeek([Data], Day.Monday) + 1, Int64.Type),
    __comDiaSemanaNome = Table.AddColumn(__comDiaSemana, "DiaSemanaNome", each Date.ToText([Data], "dddd"), type text),
    __tipado = Table.TransformColumnTypes(__comDiaSemanaNome, {{"Data", type date}})
in
    __tipado
```

Note o uso de `__camelCase` para variáveis internas (vetoriais, como `__listaDatas`, `__tabela`) e `__PascalCase` quando aplicável para escalares (como `__totalDias`).

---

## Checklist final de organização

Ao terminar a auditoria de organização, a query do usuário deveria:

- [ ] Estar em um grupo apropriado (Parâmetros/Funções/Fontes/Staging/Modelo)
- [ ] Ter Enable Load coerente (desmarcado em intermediárias, marcado em finais)
- [ ] Usar parâmetros em vez de valores hard-coded
- [ ] Seguir o padrão Fonte → Staging → Modelo (quando o tamanho do projeto justifica)
- [ ] Ter TODAS as etapas renomeadas em camelCase descritivo
- [ ] Ter comentários `//` explicando regras de negócio e workarounds
- [ ] Ter Description preenchida nas queries expostas ao modelo
- [ ] Usar funções `fx` para lógica que se repete em 3+ lugares

Se o projeto respeita tudo isso, você passou do "vibe coder" para "dev de Power Query que entrega produção sem dor".
