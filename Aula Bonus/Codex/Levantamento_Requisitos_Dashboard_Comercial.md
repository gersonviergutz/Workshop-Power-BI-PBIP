# Levantamento de Requisitos - Dashboard Comercial

## 1. Contexto

Cliente fictício de venda de eletrônicos. O objetivo do projeto é consolidar a visão comercial e financeira em uma única fonte da verdade, reduzindo divergências entre áreas e substituindo consolidações manuais por indicadores padronizados no Power BI.

## 2. Dores Identificadas

- Divergência entre números do comercial, financeiro e e-commerce.
- Falta de visão rápida de meta versus realizado.
- Baixa visibilidade do desempenho por vendedor.
- Dificuldade em separar pedido aprovado, faturamento, cancelamento e devolução.
- Consolidação manual de relatórios exportados do ERP.
- Falta de visão regional, de carteira de clientes e de produtos com baixa margem.

## 3. Objetivos do Dashboard

- Monitorar receita faturada, pedidos aprovados, cancelamentos, devoluções e margem.
- Acompanhar desempenho comercial por período, produto, cliente e região.
- Identificar produtos com alta venda e baixa margem.
- Identificar clientes ativos, novos, reativados e inativos.
- Apoiar diretoria, gestão comercial, coordenação de vendas e financeiro com regras documentadas.

## 4. Modelo Atual Encontrado

Modelo disponível no PBIP:

```text
dCalendario 1 -> N fVendas N -> 1 dProduto
                         N -> 1 dCliente
                         N -> 1 dLocalizacao
```

Tabelas atuais:

- `fVendas`: `VendaId`, `Data`, `ClienteId`, `LocalId`, `ProdutoId`, `Quantidade`, `PrecoUnitario`, `ValorTotal`, `FormaPagamento`, `Status`.
- `dProduto`: `ProdutoId`, `NomeProduto`, `Categoria`, `PrecoUnitario`, `CustoUnitario`.
- `dCliente`: `ClienteId`, `NomeCliente`, `Tipo`, `Segmento`, `LocalId`.
- `dLocalizacao`: `LocalId`, `Cidade`, `Estado`, `Regiao`.
- `dCalendario`: calendário analítico.

## 5. Requisitos Funcionais

### Visão Executiva

- Receita de pedidos aprovados.
- Receita faturada líquida.
- Receita cancelada.
- Receita devolvida.
- Quantidade de pedidos.
- Clientes ativos.
- Ticket médio.
- Lucro bruto.
- Margem bruta.
- Evolução mensal e anual.

### Produtos e Categorias

- Receita por produto e categoria.
- Quantidade vendida.
- Margem por produto e categoria.
- Ranking de produtos por receita.
- Produtos com alta venda e baixa margem.

### Clientes e Carteira

- Top clientes por receita.
- Clientes ativos no período.
- Clientes novos.
- Clientes reativados.
- Clientes sem compra há 60, 90 e 180 dias.
- Participação de receita para análise de concentração.

### Regional

- Receita por região, estado e cidade.
- Clientes ativos por região.
- Ticket médio por região.
- Margem por região.
- Ranking de cidades.

## 6. Requisitos Ainda Não Atendidos Pela Base Atual

Estes requisitos foram identificados na reunião, mas exigem campos/tabelas que ainda não existem no modelo atual:

- Meta comercial: requer tabela de metas por período, vendedor, região ou categoria.
- Vendedor: requer dimensão `dVendedor` e chave em `fVendas`.
- Canal de venda: requer coluna ou dimensão de canal.
- Faturamento real por nota fiscal: requer status fiscal, nota emitida ou tabela de faturamento.
- Devolução real: idealmente requer tipo de movimento ou tabela própria de devoluções.
- Estoque e produtos parados: requer fato de estoque ou saldo por produto.
- RLS por vendedor/região: requer tabela de usuários/perfis e mapeamento de vendedor/região.

## 7. Regras de Negócio Implementadas Como Aproximação

- `Receita Faturada Líquida`: considera vendas não canceladas nem devolvidas, menos receita devolvida.
- `Receita Cancelada`: usa `Status` contendo "CANCEL".
- `Receita Devolvida`: usa `Status` contendo "DEVOL".
- `Receita Pedidos Aprovados`: considera vendas sem status de cancelamento.
- `CPV`: `Quantidade * CustoUnitario` via relacionamento com `dProduto`.
- `Cliente Novo`: primeira compra registrada dentro do período selecionado.
- `Cliente Reativado`: cliente com compra no período, compras anteriores, mas sem compra nos 180 dias anteriores ao início do período.
- `Cliente Inativo`: cliente com última compra anterior a 60/90/180 dias da data de referência selecionada.

## 8. Medidas Criadas

### Base

- `Receita Total`
- `Receita Pedidos Aprovados`
- `Quantidade Vendida`
- `# Pedidos`
- `Ticket Médio`

### Financeiro

- `Receita Cancelada`
- `Receita Devolvida`
- `Receita Faturada Bruta`
- `Receita Faturada Líquida`
- `# Pedidos Cancelados`
- `CPV`
- `Lucro Bruto`
- `% Margem Bruta`
- `% Receita Cancelada`

### Clientes

- `# Clientes Ativos`
- `# Clientes Novos`
- `# Clientes Reativados`
- `# Clientes Sem Compra 60D`
- `# Clientes Sem Compra 90D`
- `# Clientes Sem Compra 180D`

### Inteligência Temporal

- `Receita MTD`
- `Receita YTD`
- `Receita Ano Anterior`
- `Var YoY Receita`
- `% Var YoY Receita`

### Ranking e Concentração

- `Rank Produto Receita`
- `Rank Cliente Receita`
- `Rank Cidade Receita`
- `% Participação Receita`

## 9. Próximas Tabelas Recomendadas

- `dVendedor`: vendedor, gerente, região, equipe, status.
- `fMetas`: período, vendedor, categoria, região, valor meta.
- `dCanalVenda`: canal, tipo de canal, agrupamento.
- `fFaturamento`: nota fiscal, data emissão, valor emitido, devolução, cliente, produto.
- `fEstoque`: produto, data, saldo, cobertura, giro.
- `dUsuarioRLS`: e-mail, perfil, vendedor, região.

## 10. Priorização

### Fase 1

- Visão executiva.
- Indicadores financeiros básicos.
- Produtos e categorias.
- Clientes e carteira.
- Regional com a base disponível.

### Fase 2

- Metas comerciais.
- Desempenho por vendedor.
- Canal de venda.
- RLS.
- Estoque e produtos parados.

