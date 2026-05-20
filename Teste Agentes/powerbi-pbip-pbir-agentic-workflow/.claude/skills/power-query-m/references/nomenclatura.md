# Nomenclatura em Power Query (Padrão Dashmaker)

Convenções de nomenclatura consistentes para queries, colunas, parâmetros, funções, arquivos e SQL no contexto Power Query/Power BI. Baseado nos padrões Dashmaker/Minhas Planilhas, com referência às convenções universais de naming.

## Por que ter um padrão

Sem padrão, um projeto Power Query vira rapidamente uma mistura de `Query1`, `Tabela_vendas_final_v3`, `DIM Clientes`, `fDados`, `PRODUTOS_DIM`. Isso custa tempo em toda leitura e manutenção do código.

Um padrão consistente:
- Permite identificar na hora o que é coluna, o que é medida, o que é função, o que é parâmetro
- Evita conflito em DAX (`[SalesAmount]` coluna vs `[Sales Amount]` medida)
- Facilita busca em projetos grandes (filtrar por prefixo `f`, `d`, `fx`)
- Reduz fricção quando novos devs entram no projeto

---

## Convenções universais (referência rápida)

| Convenção | Definição | Exemplos |
|---|---|---|
| **PascalCase** | Início de cada palavra em maiúscula sem separador | `MesAno`, `PrecoUnitario`, `DataFaturamento` |
| **camelCase** | Primeira palavra em minúscula, subsequentes com início maiúsculo | `fDados`, `dimProdutos`, `geraCalendario` |
| **snake_case** | Tudo minúsculo separado por `_` | `data_faturamento`, `cliente_nome`, `vw_vendas` |
| **SCREAMING_SNAKE_CASE** | snake_case todo maiúsculo | `DIM_CLIENTES`, `FACT_SALES` |
| **kebab-case** | Tudo minúsculo separado por `-` | `curso-power-bi-avancado`, `projeto-calendario-v5` |
| **SCREAMING-KEBAB-CASE** | kebab-case todo maiúsculo | `DIM-CLIENTES`, `FACT-SALES` |

---

## Padrão Dashmaker adotado

Cada contexto dentro do ecossistema Power BI/Power Query tem uma convenção específica. Misturar convenções entre contextos é o que mais quebra consistência.

### 1. Queries em geral — camelCase

Use **camelCase** para nomes de tabelas, listas, parâmetros externos e funções no Power Query. Sempre **sem acentuação** e **sem espaços**.

✅ **Correto:**
- `fDados` (fato genérica)
- `fVendas` (fato de vendas)
- `dProdutos` (dimensão de produtos)
- `dCliente` (dimensão de cliente)
- `dCalendario` (dimensão calendário)
- `fxGeraCalendario` (função)
- `fxLimpaTexto` (função)
- `dataInicial` (parâmetro)
- `caminhoServidor` (parâmetro)
- `tipoAlterado` (step)

❌ **Errado:**
- `f Dados` (espaço)
- `F_DADOS` (SCREAMING_SNAKE_CASE)
- `Produtos Dimensão` (acentuação + espaço)
- `produtos dimensão` (acentuação)

**Exceção — limitação técnica:**

Alguns nomes são fixos por imposição do Power BI e devem ser respeitados exatamente:
- `RangeStart` — parâmetro obrigatório para Incremental Refresh
- `RangeEnd` — parâmetro obrigatório para Incremental Refresh

Esses ficam em PascalCase por design do próprio Power BI.

### 2. Prefixos recomendados para queries

Adote prefixos curtos para facilitar identificação visual no painel de queries:

| Prefixo | Tipo de query | Exemplos |
|---|---|---|
| `f` | Fato | `fVendas`, `fEstoque`, `fMovimentacao` |
| `d` | Dimensão | `dCliente`, `dProduto`, `dCalendario` |
| `fx` | Função customizada | `fxLimpaTexto`, `fxGeraCalendario`, `fxConverteMoeda` |
| `vw` | View de banco (mantém padrão SQL) | `vw_vendas`, `vw_clientes` |
| (sem prefixo) | Parâmetros, staging, queries auxiliares | `dataInicial`, `stgVendasBruto` |

**Regra prática:** no painel de queries, ordenado alfabeticamente, fica visível: primeiro `d` (dims), depois `f` (fatos), depois `fx` (funções), depois o resto. Organização natural.

### 3. Nomes de colunas — PascalCase

Use **PascalCase** para TODAS as colunas. Sem acentuação, sem espaços.

✅ **Correto:**
- `SalesAmount`
- `PrecoUnitario`
- `DataAtual`
- `DataFaturamento`
- `NomeCliente`
- `UfResidencia`
- `CodigoProduto`

❌ **Errado:**
- `Sales Amount` (espaço)
- `preco_unitario` (snake_case em coluna)
- `Preço Unitário` (acentuação + espaço)
- `salesAmount` (camelCase — reservado para queries/parâmetros)

**Por que PascalCase em colunas:** quando você escreve `f_Vendas[PrecoUnitario]` em DAX fica claro que é coluna. Se fosse `f_Vendas[preco unitario]` teria que usar aspas e nomes com espaço em todo código DAX.

### 4. Nomes de medidas — livre com espaço

Para **medidas DAX**, use nomenclatura livre, amigável, com espaços. Essa é a forma mais eficaz de diferenciar medida de coluna no código.

✅ **Correto:**
- `[Sales Amount]` — **medida**
- `[Total Vendas]`
- `[Ticket Médio]`
- `[Margem %]`
- `[Receita Líquida R$]`

vs.

- `[SalesAmount]` — **coluna**
- `[PrecoUnitario]` — **coluna**

**A regra fundamental:**
- Com espaço → medida
- Sem espaço (PascalCase) → coluna

Essa convenção é padrão SQLBI (Marco Russo / Alberto Ferrari) e adotada pela comunidade mundial.

### 5. Parâmetros internos em M e DAX — prefixo `__`

Para variáveis internas de queries (dentro de `let` no M, ou dentro de `VAR` no DAX), use prefixo `__` (2 underscores) para separar visualmente dos identificadores normais.

Isso deixa claro quando um identificador é "variável local de cálculo" versus "step da query" ou "medida referenciada".

#### Variáveis escalares — `__PascalCase`

Use `__` + PascalCase para valores escalares (único valor: número, texto, data).

**Em DAX:**
```dax
Total Ajustado = 
VAR __ValorAnterior = CALCULATE([Total], PREVIOUSMONTH('d_Calendario'[Data]))
VAR __AjusteFator = 1.05
RETURN
    __ValorAnterior * __AjusteFator
```

**Em M:**
```m
let
    Source = Sql.Database("srv", "db"),
    __DataCorte = #date(2024, 1, 1),
    __Ambiente = "Producao",
    Filtrado = Table.SelectRows(Source, each [Data] >= __DataCorte)
in
    Filtrado
```

#### Variáveis vetoriais — `__camelCase`

Use `__` + camelCase para valores vetoriais (lista, tabela, coleção).

**Em DAX:**
```dax
Vendas Ultimos 3 Meses = 
VAR __windowTresMeses = 
    DATESINPERIOD('d_Calendario'[Data], MAX('d_Calendario'[Data]), -3, MONTH)
RETURN
    CALCULATE([Total Vendas], __windowTresMeses)
```

**Em M:**
```m
let
    Source = Sql.Database("srv", "db"),
    __listaClientesVip = {"CLI001", "CLI002", "CLI003"},
    Filtrado = Table.SelectRows(Source, each List.Contains(__listaClientesVip, [CodigoCliente]))
in
    Filtrado
```

**Resumo do `__`:**
- `__PascalCase` → escalar (um valor)
- `__camelCase` → vetor (lista, tabela, record)

### 6. Nomes de arquivos — kebab-case

Para arquivos externos, pastas, caminhos de rede — use **kebab-case**.

✅ **Correto:**
```
C:/arquivos/projeto-oficial/base-dados.xlsx
//servidor/compartilhado/dashboards-marketing/vendas-2025-q4.csv
\\rede\comercial\relatorio-mensal-setembro.xlsx
```

❌ **Errado:**
```
C:/arquivos/Projeto Oficial/Base_Dados.xlsx  (espaços + snake misturado)
//servidor/Compartilhado/DashboardsMarketing/Vendas2025Q4.csv  (PascalCase em caminho)
```

**Por que kebab-case em arquivos:**
- Maioria dos sistemas de arquivos trata hífen como separador seguro
- URLs funcionam naturalmente (`projeto-oficial.com.br`)
- Linux/Mac são case-sensitive mas tolerantes a hífen
- Sem espaços evita problemas de escape em scripts

### 7. SQL — snake_case ou SCREAMING_SNAKE_CASE

Quando escrever SQL dentro de `Value.NativeQuery` ou referenciar objetos do banco, use as convenções do mundo SQL: **snake_case** (mais comum) ou **SCREAMING_SNAKE_CASE** (bancos mais antigos, Oracle clássico).

✅ **Correto:**
```sql
SELECT
    venda_id,
    valor_vendas,
    data_faturamento
FROM vendas
WHERE data_faturamento >= '2025-01-01';
```

```sql
SELECT
    VENDA_ID,
    VALOR_VENDAS
FROM DIM_VENDAS;
```

❌ **Errado (misturar convenção M no SQL):**
```sql
SELECT
    VendaId,
    ValorVendas
FROM vendas;
```

**Regra:** respeite a convenção do banco de origem. Se é SQL Server com snake_case, mantenha snake_case no código. Se é legacy Oracle SCREAMING_SNAKE, mantenha assim. A transformação para PascalCase acontece no Power Query (step de Renomear Colunas), não no SQL.

---

## Tabela resumo (cola rápida)

| Contexto | Convenção | Exemplo |
|---|---|---|
| Query / tabela | camelCase | `fVendas`, `dCliente` |
| Função customizada | `fx` + camelCase | `fxGeraCalendario` |
| Parâmetro externo | camelCase | `dataInicial`, `caminhoServidor` |
| Parâmetro interno escalar (M e DAX) | `__` + PascalCase | `__ValorAnterior`, `__DataCorte` |
| Parâmetro interno vetorial (M e DAX) | `__` + camelCase | `__listaClientes`, `__windowTresMeses` |
| Coluna | PascalCase | `PrecoUnitario`, `DataFaturamento` |
| Medida DAX | Livre com espaço | `[Total Vendas]`, `[Margem %]` |
| Arquivo externo | kebab-case | `base-dados.xlsx`, `projeto-vendas.pbip` |
| SQL | snake_case (ou SCREAMING_SNAKE_CASE) | `venda_id`, `VALOR_VENDAS` |
| Incremental Refresh (fixo) | PascalCase | `RangeStart`, `RangeEnd` |

---

## Como aplicar em auditoria

Ao revisar um projeto existente, vá por categoria:

1. **Listar queries violando o padrão** (ex.: `PRODUTOS`, `Query1`, `tabela_vendas`)
2. **Listar colunas violando o padrão** (ex.: `Nome do Cliente`, `preco_unitario`)
3. **Listar medidas violando o padrão** (ex.: `TotalVendas` — deveria ter espaço)
4. **Identificar variáveis internas sem `__`** (difíceis de diferenciar de steps)
5. **Propor renomeações em batch**, priorizando impacto (queries > colunas > medidas > steps internos)

**Cuidado com renomeações:** cada rename de query, coluna ou medida pode quebrar referências em relatórios, outras queries e medidas DAX. Sempre:
- No Power Query: o próprio engine atualiza referências internas dentro do mesmo arquivo
- Para relatórios: revisar visuais após rename de colunas visíveis
- Para modelo: o Tabular Editor mostra "dependent objects" para cada objeto — use antes de renomear

Prefira fazer renomeação em lote (uma PR só, não spread ao longo de dias) para consolidar o impacto.

---

## Quando quebrar o padrão

O padrão existe para acelerar, não para atravancar. Casos em que é OK quebrar:

- **Limitação técnica:** `RangeStart`, `RangeEnd` (Incremental Refresh), colunas automáticas de Add Column from Examples que vêm com nomes genéricos e serão renomeadas logo depois
- **Legado inegociável:** modelo antigo em produção com convenção diferente — avalie se vale refatorar tudo ou padronizar só novas queries
- **Fonte externa padronizada:** se o DWH já entrega tudo em snake_case e o modelo é thin (sem transformações), pode fazer sentido manter snake_case nas tabelas em vez de renomear tudo no Power Query

Documente essas exceções no início do projeto (README.md junto do .pbip, ou descrição do modelo).
