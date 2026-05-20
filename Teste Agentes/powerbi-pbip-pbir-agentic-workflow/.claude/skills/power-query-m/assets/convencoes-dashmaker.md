# Convenções de Nomenclatura — Padrão Dashmaker (Cola Rápida)

Resumo das convenções aplicadas em projetos Power Query/Power BI no padrão Dashmaker. Use como referência rápida sem precisar reler a documentação completa.

## Quadro universal

| Convenção | Formato | Exemplos |
|---|---|---|
| **PascalCase** | Inicial maiúscula, sem separador | `MesAno`, `PrecoUnitario`, `DataFaturamento` |
| **camelCase** | Primeira minúscula, demais com inicial maiúscula | `fDados`, `dimProdutos`, `geraCalendario` |
| **snake_case** | Tudo minúsculo, separado por `_` | `data_faturamento`, `cliente_nome`, `vw_vendas` |
| **SCREAMING_SNAKE_CASE** | Tudo maiúsculo, separado por `_` | `DIM_CLIENTES`, `FACT_SALES` |
| **kebab-case** | Tudo minúsculo, separado por `-` | `curso-power-bi-avancado`, `projeto-calendario-v5` |
| **SCREAMING-KEBAB-CASE** | Tudo maiúsculo, separado por `-` | `DIM-CLIENTES`, `FACT-SALES` |

## Aplicação por contexto

| Contexto | Convenção | Exemplo |
|---|---|---|
| **Query / tabela** | camelCase | `fVendas`, `dCliente` |
| **Função customizada** | `fx` + camelCase | `fxGeraCalendario`, `fxLimpaTexto` |
| **Parâmetro externo** | camelCase | `dataInicial`, `caminhoServidor`, `nomeAmbiente` |
| **Parâmetro interno escalar** (M e DAX) | `__` + PascalCase | `__ValorAnterior`, `__DataCorte` |
| **Parâmetro interno vetorial** (M e DAX) | `__` + camelCase | `__listaClientes`, `__windowTresMeses` |
| **Coluna** | PascalCase | `PrecoUnitario`, `DataFaturamento` |
| **Medida DAX** | Livre com espaço | `[Total Vendas]`, `[Margem %]` |
| **Arquivo externo** | kebab-case | `base-dados.xlsx`, `projeto-vendas.pbip` |
| **SQL** | snake_case (ou SCREAMING_SNAKE_CASE) | `venda_id`, `VALOR_VENDAS` |
| **Incremental Refresh** (fixo) | PascalCase | `RangeStart`, `RangeEnd` |
| **Step do Power Query** | camelCase | `tiposAjustados`, `filtroDataInicial` |

## Prefixos de query (Dashmaker)

| Prefixo | Tipo | Exemplos |
|---|---|---|
| `f` | Fato | `fVendas`, `fEstoque`, `fMovimentacao` |
| `d` | Dimensão | `dCliente`, `dProduto`, `dCalendario` |
| `fx` | Função customizada | `fxGeraCalendario`, `fxLimpaTexto` |
| `src` | Fonte bruta (staging layer 1) | `srcVendasSQL`, `srcClientesExcel` |
| `stg` | Staging (tratamento) | `stgVendasTratado`, `stgClientesMerge` |
| `vw` | View SQL importada direto | `vw_vendas`, `vw_clientes` |
| (sem prefixo) | Parâmetros e auxiliares | `dataInicial`, `nomeAmbiente` |

## Regra de ouro de medidas vs colunas

```
[Sales Amount]   → medida (com espaço)
[SalesAmount]    → coluna (sem espaço, PascalCase)
```

Sempre que o nome tiver espaço, é medida. Sempre que for PascalCase sem espaço, é coluna. Convenção do padrão SQLBI adotada globalmente.

## Exemplos completos

### Query M seguindo padrão

```m
// Parâmetros (criados via Manage Parameters)
// - servidorSQL: Text = "sql-prod-01"
// - bancoSQL: Text = "dw_vendas"
// - dataInicial: Date = #date(2023, 1, 1)

let
    Source = Sql.Database(servidorSQL, bancoSQL),
    vw_vendas = Source{[Schema="dbo", Item="vw_vendas"]}[Data],
    
    // Filtra primeiro (mantém Query Folding)
    filtroData = Table.SelectRows(vw_vendas, each [data_venda] >= dataInicial),
    
    // Seleciona colunas necessárias
    colunasSelecionadas = Table.SelectColumns(filtroData, 
        {"venda_id", "cliente_id", "produto_id", "data_venda", "valor"}),
    
    // Tipa cedo
    tiposAjustados = Table.TransformColumnTypes(colunasSelecionadas, {
        {"venda_id", Int64.Type},
        {"cliente_id", Int64.Type},
        {"produto_id", Int64.Type},
        {"data_venda", type date},
        {"valor", type number}
    }),
    
    // Renomeia para PascalCase (padrão de coluna)
    renomeado = Table.RenameColumns(tiposAjustados, {
        {"venda_id", "VendaId"},
        {"cliente_id", "ClienteId"},
        {"produto_id", "ProdutoId"},
        {"data_venda", "DataVenda"},
        {"valor", "ValorTotal"}
    })
in
    renomeado
```

### Função customizada seguindo padrão

```m
// fxLimpaCpf: remove caracteres não numéricos de CPF e padroniza para 11 dígitos
(cpf as nullable text) as nullable text =>
let
    __limpo = if cpf = null then null else Text.Select(cpf, {"0".."9"}),
    __padronizado = if __limpo = null or Text.Length(__limpo) = 0 then null
                    else Text.PadStart(__limpo, 11, "0")
in
    __padronizado
```

### Medida DAX seguindo padrão

```dax
Total Vendas = 
VAR __Receita = SUM(fVendas[ValorTotal])
VAR __QtdePedidos = DISTINCTCOUNT(fVendas[VendaId])
RETURN
    DIVIDE(__Receita, __QtdePedidos)
```

### Caminho de arquivo seguindo padrão

```
\\servidor\compartilhado\projeto-vendas\base-dados-2025.xlsx
C:/dashmaker/projeto-bi-empresa/arquivo-fonte.pbip
```

### SQL seguindo padrão (dentro de Value.NativeQuery)

```m
Source = Sql.Database(servidorSQL, bancoSQL),
customQuery = Value.NativeQuery(Source, "
    SELECT 
        venda_id,
        cliente_id,
        data_venda,
        valor_total
    FROM dbo.fato_vendas
    WHERE data_venda >= @dataCorte
", [dataCorte = dataInicial])
```

## Exceções aceitáveis

- **`RangeStart`, `RangeEnd`** — nomes fixos obrigatórios para Incremental Refresh (PascalCase apesar do padrão camelCase para parâmetros)
- **Fontes com convenção própria** — se o DWH usa snake_case e o modelo é "thin" (sem transformações significativas), pode fazer sentido manter snake_case. Documente a decisão.
- **Legado em produção** — sistema que usa outra convenção há anos pode não compensar refatorar. Considere aplicar o padrão Dashmaker apenas em queries novas.
