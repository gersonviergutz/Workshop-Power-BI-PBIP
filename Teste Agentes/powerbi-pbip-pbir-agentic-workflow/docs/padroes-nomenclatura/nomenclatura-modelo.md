# Nomenclatura do Modelo Semântico

Data: 2026-04-24  
Agente: Data Modeler  
Status: aplicado em TMDL

## Padrão adotado

| Objeto | Padrão | Exemplo |
|---|---|---|
| Fatos | `fato_` + nome no plural quando representa eventos/transações | `fato_Vendas` |
| Dimensões | `dim_` + entidade de negócio no singular | `dim_Cliente`, `dim_Produto` |
| Tabelas auxiliares futuras | `aux_` + finalidade | `aux_Metas` |
| Colunas | PascalCase, sem underscore e sem acentos | `IdCliente`, `PrecoUnitario` |
| Chaves técnicas | PascalCase e ocultas no modelo quando não forem eixos analíticos | `IdProduto`, `IdLocal` |

## Mapeamento aplicado

### Tabelas

| Nome anterior | Nome novo | Tipo |
|---|---|---|
| `Vendas` | `fato_Vendas` | Fato provisória |
| `Produtos` | `dim_Produto` | Dimensão |
| `Clientes` | `dim_Cliente` | Dimensão |
| `Localização` | `dim_Localizacao` | Dimensão |
| `Calendário` | `dim_Calendario` | Dimensão calendário |

### Colunas principais

| Tabela nova | Nome anterior | Nome novo |
|---|---|---|
| `fato_Vendas` | `ID_Venda` | `IdVenda` |
| `fato_Vendas` | `ID_Cliente` | `IdCliente` |
| `fato_Vendas` | `ID_Produto` | `IdProduto` |
| `fato_Vendas` | `Preço_Unitário` | `PrecoUnitario` |
| `fato_Vendas` | `Valor_Total` | `ValorTotal` |
| `fato_Vendas` | `Forma_Pagamento` | `FormaPagamento` |
| `dim_Produto` | `ID_Produto` | `IdProduto` |
| `dim_Produto` | `Nome_Produto` | `NomeProduto` |
| `dim_Produto` | `Preço_Unitário` | `PrecoUnitario` |
| `dim_Produto` | `Custo_Unitário` | `CustoUnitario` |
| `dim_Cliente` | `ID_Cliente` | `IdCliente` |
| `dim_Cliente` | `Nome_Cliente` | `NomeCliente` |
| `dim_Cliente` | `ID_Local` | `IdLocal` |
| `dim_Localizacao` | `ID_Local` | `IdLocal` |
| `dim_Localizacao` | `Região` | `Regiao` |
| `dim_Calendario` | `MesNum` | `MesNumero` |
| `dim_Calendario` | `AnoMesINT` | `AnoMesNumero` |
| `dim_Calendario` | `DiaAtualTF` | `EhDiaAtual` |
| `dim_Calendario` | `EMesAtualTF` | `EhMesAtual` |
| `dim_Calendario` | `EAnoAtualTF` | `EhAnoAtual` |
| `dim_Calendario` | `Data Atual` | `DataAtual` |

## Observações técnicas

- `sourceColumn` foi preservado para manter compatibilidade com a origem Power Query.
- As expressões M não foram renomeadas nesta rodada.
- Não havia medidas DAX nem visuais PBIR finais dependentes dos nomes antigos.
- O refresh no Power BI Desktop continua necessário para validar runtime após a renomeação.
