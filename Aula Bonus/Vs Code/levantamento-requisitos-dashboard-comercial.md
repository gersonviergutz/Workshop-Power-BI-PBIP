# Levantamento de Requisitos - Dashboard Comercial

## 1. Contexto do projeto
Baseado na transcrição da reunião de levantamento (23/04/2026), o objetivo do dashboard é criar uma fonte única da verdade para área Comercial e Financeira, eliminando divergências de números e acelerando a tomada de decisão.

## 2. Objetivos de negócio
- Unificar conceitos de venda entre Comercial e Financeiro.
- Monitorar meta versus realizado com visão executiva.
- Acompanhar performance por vendedor, produto, cliente e região.
- Evidenciar perdas por cancelamento e devolução.
- Reduzir esforço manual de consolidação de relatórios.

## 3. Dores identificadas
- Divergência de números entre áreas.
- Falta de visão rápida de meta realizada versus planejada.
- Baixa visibilidade do desempenho individual de vendedores.
- Dificuldade para separar pedido, faturamento, cancelamento e devolução.
- Processo manual de atualização e fechamento.

## 4. KPIs priorizados (visão executiva)
- Receita de pedidos aprovados.
- Receita faturada líquida.
- Receita cancelada.
- Receita devolvida.
- Meta comercial.
- Percentual de atingimento da meta.
- Quantidade de pedidos.
- Ticket médio.
- Lucro bruto.
- Margem bruta %.
- Quantidade de clientes ativos.
- Positivação de clientes.

## 5. Regras de negócio definidas
- Receita Faturada Líquida = Receita faturada - devoluções.
- Receita Cancelada = valor de pedidos cancelados (perda comercial).
- Lucro Bruto = Receita Faturada Líquida - Custo dos Produtos Vendidos.
- Margem Bruta % = Lucro Bruto / Receita Faturada Líquida.
- Cliente positivado = cliente com compra no mês.
- Cliente novo = primeira compra registrada na empresa.
- Cliente reativado = cliente que voltou a comprar após mais de 180 dias sem compra.

## 6. Análises e páginas esperadas (fase 1)
- Visão executiva.
- Desempenho por vendedor.
- Produtos e categorias.
- Clientes e carteira.
- Indicadores financeiros básicos.

## 7. Filtros obrigatórios
- Período (Ano/Mês).
- Vendedor.
- Canal de venda.
- Região/Estado/Cidade.
- Cliente.
- Segmento de cliente.
- Produto/Categoria.
- Status do pedido.
- Forma de pagamento.

## 8. Segurança e atualização
- Atualização desejada: idealmente de hora em hora (mínimo diário).
- Requisito de segurança: RLS por perfil (Diretoria, Gerentes Regionais, Vendedores).

## 9. Mapeamento para o modelo atual (gap analysis)
Modelo atual possui: `Vendas`, `Clientes`, `Produtos`, `Calendário`.

Cobertura atual:
- Financeiro/comercial básico: sim (valor, quantidade, status, custo produto, cliente, produto, região via clientes).

Gaps de dados para 100% dos requisitos:
- Meta comercial por período/equipe/vendedor: não há tabela de metas.
- Vendedor: não há coluna/tabela de vendedor na fato.
- Canal de venda: não há coluna/tabela no modelo.
- Segmento de cliente: disponível em Clientes (ok).

## 10. Entrega desta etapa
Foram criadas medidas DAX de fase 1 no modelo para suportar:
- Receita e perdas (cancelamento/devolução).
- Faturamento líquido.
- Pedidos e ticket médio.
- Clientes ativos e positivação.
- Custos, lucro e margem.
- Novos e reativados (180 dias).
- Faixas de inatividade (30/60/90/180 dias).

## 11. Próximos passos recomendados
1. Incluir tabela de metas para habilitar Atingimento real.
2. Incluir dimensão de Vendedor e Canal de venda.
3. Definir tabela de permissões para RLS.
4. Validar domínio dos valores de `Vendas[Status]` para padronizar semântica dos KPIs.
