---
name: qa-validation
description: Skill para validação final de projetos Power BI PBIP/PBIR: requisitos, modelo, DAX, PBIR, qualidade visual e publicação.
---

# Skill: QA Validation para Power BI

## Quando usar
Use antes de concluir fases, antes de publicar ou após alterações relevantes em Power Query, TMDL, DAX ou PBIR.

## Objetivo
Garantir consistência entre requisitos aprovados, modelo, medidas, visuais e documentação.

## Validação por rastreabilidade
Verificar:

```text
Dor → Pergunta de negócio → KPI → Medida DAX → Visual PBIR → Página → Critério de aceite
```

## Checklist de requisitos
- Todas as dores prioritárias foram tratadas?
- Todos os KPIs aprovados foram implementados?
- Existem requisitos sem dono?
- Existem pendências do cliente?

## Checklist de Power Query
- Consultas críticas revisadas.
- Tipos de dados coerentes.
- Query folding considerado.
- Colunas desnecessárias removidas.
- Nomenclatura padronizada.

## Checklist de modelo
- Fatos e dimensões claras.
- Relacionamentos consistentes.
- Direção de filtro justificada.
- Tabela calendário adequada.
- Chaves ocultas quando apropriado.

## Checklist DAX
- Medidas derivam de requisitos aprovados.
- Medidas base reaproveitadas.
- Formatos corretos.
- Display folders definidos.
- Descrições incluídas em KPIs principais.

## Checklist PBIR/DataViz
- Páginas têm objetivo claro.
- Visuais usam medidas corretas.
- Títulos são claros.
- Filtros são coerentes.
- Não há poluição visual excessiva.

## Checklist de publicação
- Atualização testada.
- Credenciais e parâmetros revisados.
- RLS revisado, se existir.
- Performance aceitável.
- Documentação final criada.
- Pendências conhecidas registradas.

## Saída obrigatória

```md
# QA Final

## Resultado geral
Aprovado / Aprovado com ressalvas / Reprovado

## Itens críticos

## Itens importantes

## Melhorias futuras

## Pendências do cliente

## Checklist de publicação
```
