# Levantamento de Requisitos — Dashboard Comercial

**Projeto:** Dashboard Comercial — Workshop Power BI Cowork
**Cliente:** Empresa fictícia de venda de eletrônicos
**Data:** 23/04/2026
**Participantes:** Ana (BA), Carlos (Gerente Comercial), Mariana (Coord. Vendas), Rafael (Analista Financeiro)
**Status:** Rascunho para validação

---

## 1. Visão geral

Projeto de consolidação de indicadores comerciais e financeiros em dashboard único no Power BI, substituindo relatórios manuais exportados do ERP. Objetivo: fonte única da verdade, redução de retrabalho, visibilidade rápida de meta vs. realizado.

---

## 2. Dores mapeadas

| # | Dor | Origem | Impacto |
|---|---|---|---|
| D1 | Divergência de números entre áreas (comercial, financeiro, e-commerce) | Múltiplas planilhas manuais | Discussão recorrente em reuniões |
| D2 | Falta de visão rápida meta vs. realizado | Depende de consolidação manual | Diretoria sem resposta imediata |
| D3 | Baixa visibilidade do desempenho por vendedor | Ausência de modelo consolidado | Gestão de equipe reativa |
| D4 | Dificuldade de separar pedido, faturamento, cancelamento e devolução | Mesma fonte, conceitos misturados | Impacto direto no fechamento contábil |
| D5 | Processo manual de consolidação | Exportações + planilhas | Baixa produtividade, risco operacional |

---

## 3. Glossário de conceitos (crítico para evitar discussão posterior)

| Termo | Definição |
|---|---|
| **Pedido Aprovado** | Pedido com aprovação comercial — visão do comercial. Entra na receita aprovada. |
| **Faturamento Emitido** | Nota fiscal emitida — visão do financeiro. Entra na receita faturada. |
| **Receita Faturada Líquida** | Valor das notas emitidas menos valor das devoluções. |
| **Cancelamento** | Pedido aprovado que não virou nota. Não entra em faturamento; aparece como perda comercial. |
| **Devolução** | Nota emitida e posteriormente devolvida. Reduz receita faturada. |
| **Cliente Positivado** | Cliente que comprou no período (mesmo que uma única vez). |
| **Cliente Novo** | Cliente cuja primeira compra registrada na empresa ocorreu dentro do período. |
| **Cliente Reativado** | Cliente com compra no período atual e sem compras nos **180 dias** anteriores à nova compra. |
| **Cliente Inativo** | Cliente sem compra por mais de N dias (faixas: 30, 60, 90, 180+). |
| **Ticket Médio** | Receita Faturada Líquida ÷ Quantidade de Pedidos. |
| **Margem Bruta** | (Receita Faturada Líquida − Custo dos Produtos Vendidos) ÷ Receita Faturada Líquida. |
| **Curva ABC de Clientes** | Classificação A/B/C por concentração de receita (Pareto). |

---

## 4. Catálogo de KPIs

### 4.1 Receita e vendas

| Código | KPI | Fórmula | Granularidade mín. | Status implementação |
|---|---|---|---|---|
| K01 | Receita de Pedidos Aprovados | Σ ValorTotal onde Status ∈ {Aprovado, Faturado} | Dia | ✅ Implementável |
| K02 | Receita Faturada | Σ ValorTotal onde Status = Faturado | Dia | ✅ Implementável |
| K03 | Receita Faturada Líquida | K02 − K05 | Dia | ✅ Implementável |
| K04 | Receita Cancelada | Σ ValorTotal onde Status = Cancelado | Dia | ✅ Implementável |
| K05 | Receita Devolvida | Σ ValorTotal onde Status = Devolvido | Dia | ✅ Implementável |
| K06 | Quantidade de Pedidos | DISTINCTCOUNT(IdVenda) | Dia | ✅ Implementável |
| K07 | Quantidade Vendida | Σ Quantidade | Dia | ✅ Implementável |
| K08 | Ticket Médio | K03 ÷ K06 | Período | ✅ Implementável |

### 4.2 Custos e margem

| Código | KPI | Fórmula | Status |
|---|---|---|---|
| K09 | Custo Total | Σ (Quantidade × CustoUnitario do produto) | ✅ Implementável |
| K10 | Lucro Bruto | K03 − K09 | ✅ Implementável |
| K11 | Margem Bruta % | K10 ÷ K03 | ✅ Implementável |

### 4.3 Meta

| Código | KPI | Fórmula | Status |
|---|---|---|---|
| K12 | Meta Comercial | Valor lido da tabela `dMeta` | ⚠️ **Requer tabela de metas** |
| K13 | % Atingimento de Meta | K01 ÷ K12 (visão comercial) OU K03 ÷ K12 (visão financeira) | ⚠️ Depende K12 |
| K14 | Gap de Meta | K12 − K01 (ou K03) | ⚠️ Depende K12 |

### 4.4 Clientes

| Código | KPI | Regra | Status |
|---|---|---|---|
| K15 | Clientes Ativos | DISTINCTCOUNT(IdCliente) com pedido no período | ✅ Implementável |
| K16 | Clientes Positivados | Igual K15 (mesmo conceito) | ✅ Implementável |
| K17 | Clientes Novos | 1ª compra histórica ocorreu dentro do período | ✅ Implementável |
| K18 | Clientes Reativados | Compra no período + gap > 180d da compra anterior | ✅ Implementável |
| K19 | Clientes Inativos 30–59d | Sem compra entre 30 e 59 dias da data de referência | ✅ Implementável |
| K20 | Clientes Inativos 60–89d | Faixa 60–89 | ✅ Implementável |
| K21 | Clientes Inativos 90–179d | Faixa 90–179 | ✅ Implementável |
| K22 | Clientes Inativos 180d+ | Sem compra há 180+ dias | ✅ Implementável |
| K23 | Classificação ABC | A: top ~80% receita · B: próximos ~15% · C: restantes | ✅ Implementável (coluna calc ou medida) |

### 4.5 Produtos

| Código | KPI | Fórmula | Status |
|---|---|---|---|
| K24 | Receita por Produto | K03 agrupada por `NomeProduto` | ✅ |
| K25 | Receita por Categoria | K03 agrupada por `Categoria` | ✅ |
| K26 | Margem por Produto | (K03 − K09) ÷ K03, com filtro por produto | ✅ |
| K27 | Margem por Categoria | Idem por categoria | ✅ |
| K28 | Top Produtos por Receita | TOPN sobre K24 | ✅ |
| K29 | Top Produtos por Margem | TOPN sobre K26 | ✅ |
| K30 | Produtos Alta Venda + Baixa Margem | K24 acima da mediana E K26 abaixo da mediana | ✅ (medida flag) |

### 4.6 Vendedor *(fase 1)*

| Código | KPI | Status |
|---|---|---|
| K31 | Vendas por Vendedor | ❌ **Bloqueado — sem tabela/coluna de Vendedor** |
| K32 | Meta por Vendedor | ❌ Bloqueado |
| K33 | % Atingimento por Vendedor | ❌ Bloqueado |
| K34 | Ticket Médio por Vendedor | ❌ Bloqueado |
| K35 | Pedidos por Vendedor | ❌ Bloqueado |
| K36 | Clientes Atendidos por Vendedor | ❌ Bloqueado |
| K37 | Clientes Positivados por Vendedor | ❌ Bloqueado |
| K38 | Recompra por Vendedor | ❌ Bloqueado |
| K39 | Cancelamentos por Vendedor | ❌ Bloqueado |

### 4.7 Regional

| Código | KPI | Status |
|---|---|---|
| K40 | Receita por Região / Estado / Cidade | ✅ (Cidade/Estado/Região já em `dCliente`) |
| K41 | Ticket Médio por Região | ✅ |
| K42 | Margem por Região | ✅ |
| K43 | Clientes Ativos por Região | ✅ |
| K44 | Ranking de Cidades | ✅ |

---

## 5. Filtros obrigatórios do dashboard

| Filtro | Origem | Fase |
|---|---|---|
| Período (Ano/Mês/Data) | `dCalendario` | 1 |
| Vendedor | ❌ não existe | 1 (bloqueado) |
| Canal de Venda (Loja/E-commerce/Televendas/Representantes) | ❌ não existe | 1 (bloqueado) |
| Região / Estado / Cidade | `dCliente.Regiao / Estado / Cidade` | 1 |
| Cliente | `dCliente.NomeCliente` | 1 |
| Segmento de Cliente | `dCliente.Segmento` | 1 |
| Produto | `dProduto.NomeProduto` | 1 |
| Categoria | `dProduto.Categoria` | 1 |
| Status do Pedido | `fVendas.Status` | 1 |
| Forma de Pagamento | `fVendas.FormaPagamento` | 1 |

---

## 6. Telas planejadas

### Fase 1 (MVP)
1. **Visão Executiva** — KPIs principais (K01, K02, K03, K06, K08, K11, K13), meta vs. realizado, tendência mensal
2. **Desempenho por Vendedor** — *bloqueado até dados disponíveis*
3. **Produtos & Categorias** — top produtos, quadrante venda×margem, ranking
4. **Clientes & Carteira** — ABC, positivação, novos, reativados, inatividade
5. **Indicadores Financeiros Básicos** — faturado, cancelado, devolvido, margem

### Fase 2
6. **Regional** — mapa, ranking de cidades, heatmap por região
7. **Análise de Estoque** — *depende de ingestão da tabela de estoque*

---

## 7. Requisitos técnicos

### 7.1 Atualização
- **Ideal:** de hora em hora (comercial)
- **Mínimo:** diária (financeiro)
- **Ação pendente:** validar com TI se ERP permite conexão direta ao banco; caso contrário, rotina de exportação automatizada

### 7.2 Segurança (RLS)
| Perfil | Escopo | Regra |
|---|---|---|
| Diretoria | Tudo | `TRUE()` |
| Gerente Regional | Apenas sua(s) região(ões) | `[Regiao] IN VALUES(dUsuarios[Regiao])` |
| Vendedor | Apenas sua carteira | `[IdVendedor] = LOOKUPVALUE(dUsuarios[IdVendedor], dUsuarios[Email], USERPRINCIPALNAME())` |

*Requer tabela `dUsuarios` com mapeamento Email ↔ IdVendedor/Região — pendente.*

### 7.3 Histórico
- Mínimo 24 meses para permitir comparação YoY
- Fato atual cobre de 01/01/2021 até hoje (via `dCalendario`)

---

## 8. Gap analysis — o que falta no modelo atual

| Gap | Impacto | Severidade | Sugestão |
|---|---|---|---|
| Sem dimensão `dVendedor` e sem `IdVendedor` em `fVendas` | Bloqueia K31–K39 e filtro por vendedor | 🔴 Alta | Criar `dVendedor` (IdVendedor, NomeVendedor, Equipe, Regiao) + adicionar `IdVendedor` em `fVendas` |
| Sem coluna `CanalVenda` em `fVendas` | Bloqueia filtro e segmentação por canal | 🔴 Alta | Adicionar coluna `CanalVenda` em `fVendas` |
| Sem tabela `dMeta` | Bloqueia K12–K14 | 🔴 Alta | Criar `dMeta` (IdVendedor OU Regiao, AnoMes, ValorMeta) |
| Sem tabela `dUsuarios` para RLS | Bloqueia segurança por perfil | 🟡 Média | Criar tabela auxiliar com mapeamento Email → papel |
| Valores de `Status` em `fVendas` não documentados | Impede validar medidas (K01, K02, K04, K05) | 🟡 Média | Listar distintos: `EVALUATE DISTINCT(fVendas[Status])` |
| Custo vem de `dProduto.CustoUnitario` atual | Pode distorcer margem histórica se preços mudaram | 🔵 Baixa | Considerar SCD Tipo 2 no futuro ou congelar custo na linha do fato |

---

## 9. Plano de fases consolidado

**Fase 1 — MVP (entregar primeiro):**
- Modelo atual + criação de `dVendedor`, `dMeta`, coluna `CanalVenda` em `fVendas`
- Telas: Executiva, Produtos, Clientes, Financeiros Básicos
- Filtros: todos os documentados (exceto estoque)
- RLS básica (diretoria vs. vendedor)

**Fase 2 — Expansão:**
- Tela Regional completa
- Tela de Vendedor (se dados vierem após Fase 1)
- Tela de Estoque

**Fase 3 — Otimização:**
- Incremental Refresh em `fVendas`
- Migração Excel → SQL
- RLS por gerente regional
- Aggregations para performance em grandes volumes

---

## 10. Próximos passos

1. ✅ Validar glossário (seção 3) com todos os stakeholders
2. ⚠️ Confirmar valores distintos da coluna `Status`
3. ⚠️ Solicitar à TI: conexão direta ao ERP, cadastro de vendedores, tabela de metas
4. ⚠️ Definir critérios A/B/C da curva ABC (ex.: A = 80% receita acumulada, B = 15%, C = 5%)
5. 🔧 Criar tabela `_Medidas` no modelo com as medidas implementáveis (entregue junto com este doc)
6. 🔧 Construir Fase 1 após validação

---

**Aprovação:**

- [ ] Carlos (Gerente Comercial)
- [ ] Mariana (Coordenadora de Vendas)
- [ ] Rafael (Analista Financeiro)
- [ ] Ana (BA) — responsável
