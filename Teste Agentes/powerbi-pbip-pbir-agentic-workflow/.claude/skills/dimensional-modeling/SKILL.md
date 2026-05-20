---
name: dimensional-modeling
description: Skill para modelagem dimensional, modelo estrela, relacionamento, granularidade e desenho de modelo semântico Power BI.
---

# Skill: Dimensional Modeling para Power BI

## Quando usar
Use ao identificar fatos, dimensões, chaves, granularidade, relacionamentos e regras de modelagem.

## Objetivo
Criar modelos simples, performáticos e compreensíveis para análise de dados.

## Princípios
- Preferir modelo estrela.
- Fatos no centro, dimensões ao redor.
- Evitar snowflake sem necessidade real.
- Evitar relacionamento muitos-para-muitos sem justificativa.
- Evitar filtro bidirecional como solução padrão.
- Separar fatos transacionais, snapshots e metas.
- Garantir granularidade clara para cada fato.
- Criar tabela calendário adequada para inteligência de tempo.

## Classificação de tabelas

### Fato
Contém eventos ou valores mensuráveis.
Exemplos:
- Vendas
- Pedidos
- Estoque
- Metas
- Financeiro

### Dimensão
Contém atributos para segmentação.
Exemplos:
- Cliente
- Produto
- Calendário
- Vendedor
- Canal
- Região

### Auxiliar
Tabela de suporte, parâmetro ou bridge.
Só usar quando necessário.

## Checklist de granularidade
Para cada fato, responder:
- Uma linha representa o quê?
- Pedido?
- Item do pedido?
- Cliente por mês?
- Produto por loja por dia?
- Meta por vendedor por mês?

## Checklist de relacionamento
- Chaves têm unicidade no lado dimensão?
- Há nulos nas chaves?
- Há duplicidade inesperada?
- A direção de filtro é single sempre que possível?
- O relacionamento ativo é o principal para análise?
- Há necessidade de relacionamento inativo para USERELATIONSHIP?

## Tabela calendário
Recomendações:
- Criar `dim_Calendario` quando houver análise temporal.
- Ter coluna Date contínua.
- Ter Ano, Mês, Número do Mês, Trimestre, Semana, Dia.
- Marcar como tabela de data no Power BI quando aplicável.
- Evitar várias tabelas calendário sem necessidade.

## Entregáveis
- Diagrama conceitual do modelo.
- Lista de fatos e dimensões.
- Lista de relacionamentos.
- Dicionário de dados.
- Decisões de modelagem registradas.
