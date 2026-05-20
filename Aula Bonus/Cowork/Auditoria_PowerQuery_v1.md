# Auditoria Power Query — Power BI Workshop Cowork

**Projeto:** `Power BI Workshop Cowork.pbip`
**Data:** 23/04/2026
**Engine:** PBIP / TMDL
**Queries auditadas:** 5 (Vendas, Produtos, Clientes, Localização, Calendário)

---

## Resumo executivo

| Métrica | Valor |
|---|---|
| Queries auditadas | 5 |
| Total de violações | 18 (🔴 3 / 🟡 10 / 🔵 5) |
| Parâmetros definidos | 0 |
| Staging queries | 0 |
| Grupos/pastas | 0 |
| Funções customizadas | 0 |
| Query Folding | N/A (fonte Excel — folding não se aplica) |

**Top 3 prioridades:**

1. **Parametrizar o caminho do Excel** — hoje o caminho absoluto `C:\Users\gerso\OneDrive - Minhas Planilhas\...` está replicado em 4 queries. Qualquer outro usuário ou deploy no Serviço quebra.
2. **Consolidar a fonte Excel em uma única query staging** — hoje `Excel.Workbook(File.Contents(...))` é chamado 4 vezes abrindo o mesmo arquivo 4×. Basta 1 staging compartilhada.
3. **Corrigir o erro em `Localização`** — a query tipa `Column1..Column4` ANTES de promover cabeçalhos. Etapa inútil que desperdiça processamento.

---

## 🔴 Críticos

### 🔴 1. Caminho de arquivo hard-coded e replicado em 4 queries
**Queries:** `Vendas`, `Produtos`, `Clientes`, `Localização`
**Regra:** Parâmetros usados em vez de valores hard-coded (caminhos, servidores, datas).

**Problema:** O caminho `C:\Users\gerso\OneDrive - Minhas Planilhas\Minhas Planilhas\Comunidade Dashmaker\Cursos\Workshop Power BI na era da IA Generativa\Material Workshop\Banco de Dados\Vendas Exemplo.xlsx` está literalmente copiado em 4 queries.

**Impacto:**
- Qualquer mudança de pasta/máquina exige editar 4 queries.
- Impossível alguém além do Gerson abrir o projeto sem reconfigurar tudo.
- Deploy no Power BI Service exige gateway com exatamente o mesmo caminho.
- Perde a portabilidade entre Dev/Homolog/Prod.

**Correção:** Criar parâmetro `caminhoArquivoVendas` e uma query staging `Fonte_VendasExemplo` (com Enable Load = OFF) que abre o Excel uma única vez. As demais queries passam a referenciar essa staging.

```m
// 1) Parâmetro (criar em Gerenciar Parâmetros)
// Nome: caminhoArquivoVendas
// Tipo: Text
// Valor atual: C:\Users\gerso\OneDrive - Minhas Planilhas\...\Vendas Exemplo.xlsx

// 2) Query staging: Fonte_VendasExemplo  (Enable Load = OFF)
let
    fonteExcel = Excel.Workbook(File.Contents(caminhoArquivoVendas), null, true)
in
    fonteExcel

// 3) Cada query final vira simples referência:
let
    origem         = Fonte_VendasExemplo,
    navegarVendas  = origem{[Item = "Vendas", Kind = "Sheet"]}[Data],
    promoverCabec  = Table.PromoteHeaders(navegarVendas, [PromoteAllScalars = true]),
    tiparColunas   = Table.TransformColumnTypes(promoverCabec, {
                        {"ID_Venda", type text},
                        {"Data", type date},
                        {"ID_Cliente", type text},
                        {"ID_Produto", type text},
                        {"Quantidade", Int64.Type},
                        {"Preço_Unitário", type number},
                        {"Valor_Total", type number},
                        {"Forma_Pagamento", type text},
                        {"Status", type text}
                    })
in
    tiparColunas
```

---

### 🔴 2. Erro de lógica em `Localização` — tipar antes de promover cabeçalhos
**Query:** `Localização`
**Regra:** Ordem correta de transformações / etapas sem efeito prático.

**Problema:** A query tem uma etapa que tipa `Column1..Column4` como `text` ANTES de promover cabeçalhos:

```m
#"Tipo de coluna alterado" = Table.TransformColumnTypes(#"Navegação 1",
    {{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}}),
#"Cabeçalhos Promovidos" = Table.PromoteHeaders(#"Tipo de coluna alterado", [PromoteAllScalars=true]),
#"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos", ...)
```

Como logo depois os cabeçalhos são promovidos (Column1 vira `ID_Local`, Column2 vira `Cidade` etc.) e em seguida há um novo `Table.TransformColumnTypes` com os nomes corretos, o primeiro passo **não agrega nada**. Pior: se o Excel mudar a quantidade de colunas, essa etapa pode até quebrar.

**Impacto:** Processamento desperdiçado; ruído no Applied Steps; risco de quebra silenciosa se a planilha ganhar colunas.

**Correção:** Remover a etapa `#"Tipo de coluna alterado"` — idêntica à estrutura limpa das outras queries.

```m
let
    origem            = Fonte_VendasExemplo,
    navegarLocaliz    = origem{[Item = "Localização", Kind = "Sheet"]}[Data],
    promoverCabec     = Table.PromoteHeaders(navegarLocaliz, [PromoteAllScalars = true]),
    tiparColunas      = Table.TransformColumnTypes(promoverCabec, {
                            {"ID_Local", type text},
                            {"Cidade",   type text},
                            {"Estado",   type text},
                            {"Região",   type text}
                        })
in
    tiparColunas
```

---

### 🔴 3. Coluna `Data Atual` do Calendário retorna tipo errado
**Query:** `Calendário`
**Regra:** Tipagem consistente de colunas.

**Problema:** A coluna `Data Atual` é declarada no `#table` como `text`, mas a lógica mistura string e data:

```m
if Date.IsInCurrentDay(_) then "Dia Atual" else Text.From(_)
```

Além de forçar `text` para uma coluna com conteúdo de data, o nome `Data Atual` é ambíguo — sugere uma data, mas entrega a string "Dia Atual" ou um texto com a data. Isso quebra qualquer ordenação cronológica, comparações ou filtros por data usando essa coluna.

**Impacto:** Usuários que tentarem usar `Data Atual` como eixo de gráfico vão se deparar com ordenação alfabética (ex.: `01/10/2026` antes de `02/01/2026`).

**Correção recomendada:** Manter duas colunas distintas:

```m
// no type table
DataAtualFlag = Logical.Type,   // True/False se é a data de hoje
RotuloData    = text            // "Hoje" OU texto da data formatado

// na transform:
Date.IsInCurrentDay(_),                                  // DataAtualFlag
if Date.IsInCurrentDay(_) then "Hoje"
else Date.ToText(_, "dd/MM/yyyy")                        // RotuloData
```

Benefício: `DataAtualFlag` é um boolean limpo para marcar o hoje; `RotuloData` é um texto com formato previsível.

---

## 🟡 Avisos

### 🟡 4. Nomenclatura de queries não segue padrão Dashmaker
**Queries:** todas.

**Problema:** Nomes atuais são `Vendas`, `Produtos`, `Clientes`, `Localização`, `Calendário`. O padrão Dashmaker distingue fato/dimensão por prefixo.

**Correção:**

| Atual | Padrão Dashmaker | Tipo |
|---|---|---|
| `Vendas` | `fVendas` | Fato |
| `Produtos` | `dProduto` | Dimensão |
| `Clientes` | `dCliente` | Dimensão |
| `Localização` | `dLocalizacao` | Dimensão |
| `Calendário` | `dCalendario` | Dimensão (Date) |

Observação: remover acentos do identificador da query (`dLocalizacao`, `dCalendario`) evita problemas em scripts XMLA, deploy via Fabric API e referências em ferramentas externas.

---

### 🟡 5. Colunas com `snake_case` e acentos
**Queries:** todas as 4 de dados.

**Problema:** `ID_Venda`, `ID_Cliente`, `ID_Produto`, `Nome_Produto`, `Preço_Unitário`, `Custo_Unitário`, `Forma_Pagamento`, `Região`.

O padrão Dashmaker para colunas é **PascalCase sem acento e sem underscore**. Acentos em nomes de colunas também geram atrito em DAX, consultas MDX/DAX externas e em ferramentas de governança.

**Correção sugerida (reexpor via `Table.RenameColumns` na staging final):**

| Atual | Recomendado |
|---|---|
| `ID_Venda` | `IdVenda` |
| `ID_Cliente` | `IdCliente` |
| `ID_Produto` | `IdProduto` |
| `Nome_Produto` | `NomeProduto` |
| `Nome_Cliente` | `NomeCliente` |
| `Preço_Unitário` | `PrecoUnitario` |
| `Custo_Unitário` | `CustoUnitario` |
| `Valor_Total` | `ValorTotal` |
| `Forma_Pagamento` | `FormaPagamento` |
| `ID_Local` | `IdLocal` |
| `Região` | `Regiao` |

Isso também requer ajuste nos relacionamentos.

---

### 🟡 6. Nomes de etapas (steps) não descritivos e mistos PT/EN
**Queries:** todas.

**Problema:** `#"Navegação 1"`, `#"Tipo de coluna alterado"`, `#"Tipo Alterado"`, `#"Cabeçalhos Promovidos"` versus `#"Cabeçalhos promovidos"` (inconsistência de case), numeração residual `1`.

**Correção:** adote camelCase descritivo e consistente entre queries:

```m
origem → fonteExcel → navegarVendas → promoverCabec → tiparColunas
```

Ganho: quem lê o applied steps entende o fluxo sem abrir cada etapa.

---

### 🟡 7. Ausência do padrão Fonte → Staging → Modelo
**Queries:** todas.

**Problema:** Cada uma das 4 queries abre o mesmo arquivo Excel diretamente. Não há separação entre queries de fonte (cruas), staging (tratadas) e modelo (expostas).

**Correção:** estrutura recomendada:

```
📁 Parâmetros
   - caminhoArquivoVendas

📁 Fontes          (Enable Load = OFF)
   - Fonte_VendasExemplo

📁 Staging         (Enable Load = OFF)
   - Stg_Vendas, Stg_Produtos, Stg_Clientes, Stg_Localizacao

📁 Fatos
   - fVendas

📁 Dimensões
   - dProduto, dCliente, dLocalizacao, dCalendario
```

As staging querem aplicar limpezas (trim, remoção de nulos, tipagem). Os modelos finais só renomeiam e expõem.

---

### 🟡 8. `Enable Load` provavelmente ligado em todas as queries intermediárias
**Queries:** todas.

**Problema:** Como não há staging, não dá para dizer definitivamente pelo TMDL, mas na estrutura atual tudo carrega no modelo. Quando você adotar staging (item 7), cada query intermediária deve estar com **"Enable Load" desmarcado** para não criar tabelas ocultas no modelo.

**Como fazer:** botão direito na query → **"Enable Load"** → desmarcar → as queries ficam em itálico no painel.

---

### 🟡 9. Inconsistências de formatação entre queries
**Queries:** Vendas & Produtos vs. Clientes & Localização.

**Problema:**
- Vendas/Produtos: `[PromoteAllScalars = true]` (com espaços).
- Clientes/Localização: `[PromoteAllScalars=true]` (sem espaços).
- Clientes/Localização: nomes como `#"Cabeçalhos Promovidos"` (Title Case) e `#"Tipo Alterado"` vs. `#"Cabeçalhos promovidos"` e `#"Tipo de coluna alterado"` nas outras.
- Indentação mista.

**Impacto:** Não quebra nada, mas dificulta revisão e sugere que queries foram criadas em momentos/máquinas diferentes. Padronize com a sugestão do item 6.

---

### 🟡 10. `Calendário` — coluna `Semana` usa `Date.WeekOfMonth`
**Query:** `Calendário`, coluna `Semana`.

**Problema:** A coluna chamada `Semana` é calculada com `Date.WeekOfMonth(_)`, que retorna **1 a 5** (semana dentro do mês). Se o usuário esperar "semana do ano" (1–53, convencional em relatórios YoY e ISO), o comportamento vai surpreender.

**Correção:** escolher conscientemente e renomear para remover ambiguidade:

```m
// Se for semana do ano:
SemanaDoAno = Int64.Type,   // no type table
...
Date.WeekOfYear(_)          // na transform

// Se for semana do mês:
SemanaDoMes = Int64.Type,
...
Date.WeekOfMonth(_)
```

---

### 🟡 11. Calendário — cabeçalho com `Data Atual` contendo espaço
**Query:** `Calendário`.

**Problema:** No `type table` existe o campo `Data Atual = text` — nome de coluna com espaço gera necessidade de escape em várias situações DAX (ex.: `'Calendário'[Data Atual]`) e é inconsistente com as outras colunas (`AnoAtual`, `MesAtual`).

**Correção:** renomear para `DataAtual` ou, se seguir a sugestão do item 3, separar em `DataAtualFlag` e `RotuloData`.

---

### 🟡 12. Ausência de parâmetros `RangeStart` / `RangeEnd` (Incremental Refresh)
**Query:** `fVendas` (futura).

**Problema:** Tabelas fato tendem a crescer. Sem `RangeStart`/`RangeEnd` criados e filtro de data no M, não há como habilitar Incremental Refresh se o volume aumentar.

**Nota:** Com fonte Excel NÃO há Query Folding, então Incremental Refresh teria ganho limitado — só vale a pena se a fonte for migrada para SQL/Dataverse. Registre isso como débito técnico caso a fonte mude.

---

### 🟡 13. `Calendário` fixo a partir de 01/01/2021
**Query:** `Calendário`.

```m
MenorData = #date(2021,01,01),
MaiorData = Date.From(Date.EndOfYear(DateTime.LocalNow())),
```

**Problema:** A data inicial está hard-coded. Se a tabela de vendas passar a ter registros anteriores, o Calendário não cobre. Se a fonte só começa em uma data futura, o Calendário está gerando anos desnecessários.

**Correção:** derivar dinamicamente da fonte de fatos:

```m
// No começo do Calendário, após criar a staging de vendas:
    MenorData = List.Min(Stg_Vendas[Data]),
    MaiorData = Date.From(Date.EndOfYear(DateTime.LocalNow())),
```

Ou expor como parâmetros `anoInicialCalendario` e `anoFinalCalendario` se quiser controle manual.

---

## 🔵 Infos

### 🔵 14. Colunas potencialmente redundantes no modelo
- `Preço_Unitário` existe em `Vendas` e em `Produtos`. Em geral o preço da venda é o praticado naquela transação (Vendas) e o preço tabelado é da dimensão (Produtos). Documente a diferença ou remova um dos dois para evitar confusão.
- `Valor_Total` em Vendas pode ser uma **medida DAX** (`SUMX(Vendas, Vendas[Quantidade] * Vendas[Preço_Unitário])`) em vez de coluna armazenada — economiza memória no VertiPaq.

### 🔵 15. Tipagem de `Custo_Unitário` como `Int64`
`Custo_Unitário` em `Produtos` está como `Int64.Type`. Moeda/custo geralmente é decimal. Verifique se a fonte realmente só tem inteiros ou se a importação está truncando centavos.

### 🔵 16. Relacionamentos com nomes `AutoDetected_*`
Os 3 relacionamentos foram criados automaticamente pelo Power BI. Não é erro, mas sugere que não houve revisão manual (cardinalidade, cross-filter direction, integridade referencial).

### 🔵 17. Ausência total de comentários nas queries de dados
Somente `Calendário` tem comentários. Considere adicionar pelo menos um bloco inicial em cada query descrevendo: fonte, periodicidade de atualização, dono do dado.

```m
let
    // Fonte: Vendas Exemplo.xlsx (Onedrive Dashmaker)
    // Atualização: manual, quando novo mês é fechado
    // Dono: Gerson
    ...
```

### 🔵 18. `annotation __PBI_TimeIntelligenceEnabled = 0` — boa prática já aplicada
Bom: Auto Date/Time está desligado no modelo. Isso evita inflar o PBIX com calendários ocultos para cada coluna data — mantenha assim e continue usando o `Calendário` explícito.

---

## Checklist de boas práticas atendidas ✅

- [x] Auto Date/Time desabilitado no modelo (`__PBI_TimeIntelligenceEnabled = 0`)
- [x] Tabela calendário explícita criada via M
- [x] Tipagem explícita de colunas em todas as queries (não ficou `Any`)
- [x] Cultura definida como `pt-BR` no modelo
- [x] Separação fato/dimensão (mesmo sem o prefixo `f`/`d`)
- [x] Uso de `List.Dates` + `#table` no Calendário (padrão performático)

---

## Plano de ação sugerido (ordem de execução)

1. **Criar parâmetro `caminhoArquivoVendas`** e query staging `Fonte_VendasExemplo` com Enable Load OFF.
2. **Refatorar Localização** removendo o step redundante.
3. **Apontar as 4 queries de dados** para a staging `Fonte_VendasExemplo`.
4. **Criar grupos no Query Editor:** Parâmetros, Fontes, Staging, Fatos, Dimensões.
5. **Renomear queries** para padrão `f`/`d` (impacta relacionamentos → ajuste).
6. **Renomear etapas** para camelCase descritivo.
7. **Corrigir Calendário:** desambiguar `Semana` e refatorar `Data Atual`.
8. **(Opcional) Renomear colunas** para PascalCase sem acento — impacto em DAX e visuais, faça com cuidado.
9. **Revisar relacionamentos** criados por autodetect.

---
