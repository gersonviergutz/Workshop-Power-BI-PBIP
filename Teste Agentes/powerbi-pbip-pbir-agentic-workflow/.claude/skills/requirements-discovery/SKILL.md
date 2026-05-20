---
name: requirements-discovery
description: Skill para transformar transcrições, anotações e conversas com cliente em levantamento de requisitos, dores, KPIs e critérios de aceite para Power BI.
---

# Skill: Requirements Discovery para Projetos Power BI

## Quando usar
Use quando houver transcrição de reunião, ata, briefing, mensagens do cliente, áudio transcrito ou anotações soltas.

## Objetivo
Converter linguagem de negócio em requisitos claros, mensuráveis e rastreáveis para construção de dashboard.

## Processo

### 1. Leitura e classificação
Classificar trechos em:
- Dor operacional.
- Dor gerencial.
- Pedido explícito.
- Necessidade implícita.
- Regra de negócio.
- Fonte de dados citada.
- Indicador citado.
- Decisão esperada.
- Pendência de validação.

### 2. Extração de dores
Para cada dor, registrar:
- Dor declarada.
- Evidência textual.
- Área impactada.
- Impacto no negócio.
- Frequência do problema.
- Consequência se não resolver.

### 3. Perguntas de negócio
Converter dores em perguntas, por exemplo:
- Quanto vendemos por período?
- Qual vendedor está abaixo da meta?
- Quais clientes reduziram compra?
- Quais produtos têm maior margem?
- Onde há ruptura, atraso ou perda de oportunidade?

### 4. KPIs candidatos
Para cada KPI:
- Nome do indicador.
- Objetivo.
- Fórmula de negócio.
- Granularidade.
- Filtros necessários.
- Dimensões de análise.
- Fonte de dados.
- Frequência de atualização.
- Critério de aceite.

### 5. Regras de negócio
Separar regras confirmadas de hipóteses.

Use marcação:
- `[CONFIRMADO]`
- `[HIPÓTESE]`
- `[PENDENTE CLIENTE]`

## Matriz obrigatória
Gerar matriz:

```md
| Dor | Pergunta de negócio | KPI | Fórmula de negócio | Fonte | Visual sugerido | Prioridade | Status |
|---|---|---|---|---|---|---|---|
```

## Entregáveis
- `docs/levantamento-requisitos/levantamento-requisitos.md`
- `docs/levantamento-requisitos/matriz-rastreabilidade.md`
- `docs/levantamento-requisitos/pendencias-cliente.md`

## Critérios de qualidade
- Nenhum KPI sem pergunta de negócio.
- Nenhuma pergunta de negócio sem decisão esperada.
- Nenhuma regra crítica deve ficar implícita.
- Distinguir claramente necessidade real de solução pedida pelo cliente.
