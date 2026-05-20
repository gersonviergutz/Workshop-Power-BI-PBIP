---
name: dataviz-powerbi
description: Skill de boas práticas de visualização de dados, UX analítico e storytelling para dashboards Power BI.
---

# Skill: DataViz Power BI

# Visual Design System — Padrão para Páginas PBIR

Este arquivo é a **fonte de verdade** para criação de novas páginas e visuais no formato PBIR.
Sempre que o usuário pedir para criar uma página, gráfico, card ou qualquer visual,
use os templates e configurações abaixo como base.

---

## 1. Paleta de Cores

| Token | Hex | Uso |
|-------|-----|-----|
| GREEN_DARK | `#1A3C34` | Header bar, títulos de gráfico, accent bar de card |
| GREEN_MID | `#2E7D32` | Cor primária: barras de gráfico, linhas, accent bar principal |
| GREEN_LIGHT | `#558B2F` | Variação verde: 4ª cor de dados, accent bar alternativo |
| GREEN_SOFT | `#81C784` | Verde suave para backgrounds ou 4ª série |
| GREEN_PALE | `#A5D6A7` | Verde claro para áreas de destaque |
| ORANGE_ACC | `#F57C00` | Cor de destaque: cancelamentos, alertas, 2ª série |
| ORANGE_LIGHT | `#FFB74D` | Laranja suave para variações |
| BG_PAGE | `#F4F6F5` | Fundo da página (cinza muito claro) |
| WHITE | `#FFFFFF` | Fundo dos cards e visuais |
| TEXT_DARK | `#1A1A1A` | Texto principal |
| TEXT_GRAY | `#757575` | Texto secundário, labels, eixos |
| BORDER_COL | `#E0E4E2` | Bordas de visuais |
| SHADOW_COL | `#B0BEC5` | Cor da sombra |
| GRIDLINE | `#EEEEEE` | Linhas de grade dos gráficos |

### Ordem de cores para séries de dados (dataColors)
```json
["#2E7D32", "#F57C00", "#558B2F", "#81C784", "#1A3C34", "#A5D6A7", "#F57C00", "#FFB74D"]
```

---

## 2. Tipografia

| Classe | Fonte | Tamanho | Cor | Uso |
|--------|-------|---------|-----|-----|
| Callout (valor do card) | wf_standard-font, helvetica, arial, sans-serif | 22L (half-pts) | (herdado) | Valor principal do cardVisual |
| Title (título de gráfico) | Segoe UI Semibold | 10D (pts) | #1A3C34 | Título dentro do visualContainerObjects |
| Header (cabeçalho da página) | Segoe UI Semibold | 16px | #FFFFFF | TextBox no header bar |
| Label (label do card) | Segoe UI | 9L (half-pts) | #757575 | Label do cardVisual |
| Eixos de gráfico | (herdado do tema) | 9D (pts) | (herdado) | categoryAxis, valueAxis |
| Data labels | (herdado do tema) | 9D (pts) | #757575 | Nos gráficos de barra |
| Slicer header | Segoe UI | 9L (half-pts) | #1A1A1A | Cabeçalho do slicer |
| Slicer items | Segoe UI | 9L (half-pts) | #1A1A1A | Itens do dropdown |

---

## 3. Configuração da Página

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.1.0/schema.json",
  "name": "<PAGE_HEX_ID>",
  "displayName": "<Nome da Página>",
  "displayOption": "FitToPage",
  "height": 720,
  "width": 1280,
  "objects": {
    "background": [{
      "properties": {
        "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#F4F6F5'" } } } } },
        "transparency": { "expr": { "Literal": { "Value": "0D" } } }
      }
    }]
  }
}
```

---

## 4. Grid de Layout Padrão (1280 × 720)

```
┌──────────────────────────────────────────────────────────────────┐
│ HEADER BAR (shape verde escuro)           [Slicer1] [Slicer2]   │ y:0   h:64
│   TextBox: Título em branco 16px                                 │
├────────────┬────────────┬────────────┬───────────────────────────┤
│  Card 1    │  Card 2    │  Card 3    │  Card 4                   │ y:76  h:90
│  (294w)    │  (294w)    │  (294w)    │  (294w)                   │
├────────────┴────────────┴────────────┴───────────────────────────┤
│ Gráfico Esquerdo (600w)        │ Gráfico Direito (610w)         │ y:178 h:240
├────────────────────────────────┼────────────────────────────────│
│ Gráfico Esquerdo (600w)        │ Gráfico Direito (610w)         │ y:430 h:275
└────────────────────────────────┴────────────────────────────────┘
```

### Posições exatas

| Elemento | x | y | width | height | z |
|----------|---|---|-------|--------|---|
| Header bar (shape) | 0 | 0 | 1280 | 64 | 0 |
| TextBox título | 30 | 14 | 580 | 36 | 6 |
| Slicer 1 (Ano) | 900 | 6 | 170 | 55 | 5 |
| Slicer 2 (Mês) | 1085 | 6 | 170 | 55 | 5 |
| Card 1 | 30 | 76 | 294 | 90 | 3 |
| Card 2 | 340 | 76 | 294 | 90 | 3 |
| Card 3 | 651 | 76 | 294 | 90 | 3 |
| Card 4 | 961 | 76 | 294 | 90 | 3 |
| Gráfico sup. esquerdo | 30 | 178 | 600 | 240 | 2 |
| Gráfico sup. direito | 646 | 178 | 610 | 240 | 2 |
| Gráfico inf. esquerdo | 30 | 430 | 600 | 275 | 2 |
| Gráfico inf. direito | 646 | 430 | 610 | 275 | 2 |

### Margem padrão
- Margem esquerda/direita: **30px**
- Gap entre cards: **16px**
- Gap entre gráficos: **16px**
- Gap vertical entre linhas: **12px**

---

## 5. Template: Header Bar (Shape)

⚠️ **REGRA CRÍTICA para shapes:** A cor visível de uma shape vem EXCLUSIVAMENTE do `visualContainerObjects.background.color`. Os objects internos `fill` e `outline` devem ter `show: false`. Se você usar `fill.fillColor`, o Power BI ignora e renderiza a cor padrão do tema (geralmente roxa). Isso é um bug comum — a cor da shape é controlada pelo container, não pelo fill interno.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "<HEX_ID>",
  "position": { "x": 0, "y": 0, "z": 0, "height": 64, "width": 1280 },
  "visual": {
    "visualType": "shape",
    "objects": {
      "fill": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}}],
      "outline": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}}]
    },
    "visualContainerObjects": {
      "background": [{"properties":{"show":{"expr":{"Literal":{"Value":"true"}}},"color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#1A3C34'"}}}}},"transparency":{"expr":{"Literal":{"Value":"0L"}}}}}],
      "border": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}}]
    },
    "drillFilterOtherVisuals": true
  }
}
```

---

## 6. Template: TextBox (Título da Página)

⚠️ **REGRA CRÍTICA para textbox:** O campo `paragraphs` deve ser um **array nativo de objetos JSON**, NÃO uma string JSON encodada dentro de `expr.Literal`. Se você usar `{"expr":{"Literal":{"Value":"[{...}]"}}}`, o Power BI não consegue parsear e renderiza o textbox vazio. O formato correto é `"paragraphs": [{"textRuns": [...]}]` — array direto, sem wrapper de expressão.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "<HEX_ID>",
  "position": { "x": 30, "y": 14, "z": 6, "height": 36, "width": 580 },
  "visual": {
    "visualType": "textbox",
    "objects": {
      "general": [{"properties":{
        "paragraphs": [{"textRuns":[{"value":"<TÍTULO DA PÁGINA>","textStyle":{"fontFamily":"Segoe UI Semibold","fontSize":"16px","fontWeight":"bold","color":"#FFFFFF"}}]}]
      }}]
    },
    "visualContainerObjects": {
      "background": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}}]
    },
    "drillFilterOtherVisuals": true
  }
}
```

---

## 7. Template: Slicer (Dropdown)

⚠️ **REGRA CRÍTICA de altura do slicer:** Quando o header do slicer está visível (`header.show: true`), a altura mínima deve ser **55px**. Com `height: 40`, o Power BI não consegue renderizar o título + o dropdown juntos — o título fica cortado ou o dropdown some. Use `height: 40` APENAS se `header.show: false` (slicer sem título). Com título visível, use no mínimo 55px.

| header.show | height mínimo | Uso |
|-------------|---------------|-----|
| `true` | **55** | Padrão — slicer com título visível |
| `false` | 40 | Slicer compacto sem título (ex: dentro de header bar) |

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "<HEX_ID>",
  "position": { "x": "<X>", "y": 12, "z": 5, "height": 55, "width": 170 },
  "visual": {
    "visualType": "slicer",
    "query": {
      "queryState": {
        "Values": {
          "projections": [{
            "field": { "Column": { "Expression": { "SourceRef": { "Entity": "<ENTIDADE>" } }, "Property": "<COLUNA>" } },
            "queryRef": "<ENTIDADE>.<COLUNA>",
            "nativeQueryRef": "<COLUNA>",
            "active": true
          }]
        }
      }
    },
    "objects": {
      "data": [{"properties":{"mode":{"expr":{"Literal":{"Value":"'Dropdown'"}}}}}],
      "header": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"true"}}},
        "fontFamily":{"expr":{"Literal":{"Value":"'Segoe UI'"}}},
        "textSize":{"expr":{"Literal":{"Value":"9L"}}},
        "fontColor":{"solid":{"color":{"expr":{"Literal":{"Value":"'#1A1A1A'"}}}}}
      }}],
      "selection": [{"properties":{"selectAllCheckboxEnabled":{"expr":{"Literal":{"Value":"true"}}}}}],
      "items": [{"properties":{
        "textSize":{"expr":{"Literal":{"Value":"9L"}}},
        "fontColor":{"solid":{"color":{"expr":{"Literal":{"Value":"'#1A1A1A'"}}}}}
      }}]
    },
    "visualContainerObjects": {
      "dropShadow": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"true"}}},
        "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#B0BEC5'"}}}}},
        "preset":{"expr":{"Literal":{"Value":"'Custom'"}}},
        "shadowSpread":{"expr":{"Literal":{"Value":"0L"}}},
        "shadowBlur":{"expr":{"Literal":{"Value":"8L"}}},
        "angle":{"expr":{"Literal":{"Value":"90L"}}},
        "shadowDistance":{"expr":{"Literal":{"Value":"3L"}}},
        "transparency":{"expr":{"Literal":{"Value":"75L"}}}
      }}],
      "background": [{"properties":{"show":{"expr":{"Literal":{"Value":"true"}}},"color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#FFFFFF'"}}}}}}}],
      "border": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"true"}}},
        "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#E0E4E2'"}}}}},
        "radius":{"expr":{"Literal":{"Value":"8L"}}}
      }}],
      "subTitle": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}}],
      "padding": [{"properties":{
        "top":{"expr":{"Literal":{"Value":"2L"}}},
        "bottom":{"expr":{"Literal":{"Value":"0L"}}},
        "left":{"expr":{"Literal":{"Value":"4L"}}},
        "right":{"expr":{"Literal":{"Value":"4L"}}}
      }}]
    },
    "drillFilterOtherVisuals": true
  }
}
```

---

## 8. Template: Card Visual (KPI)

### Configurações do Card

| Propriedade | Valor | Notas |
|-------------|-------|-------|
| visualType | `cardVisual` | Tipo moderno de card (não `card` legado) |
| Valor fontSize | `22L` | Half-points (11pt real) |
| Valor fontFamily | `wf_standard-font, helvetica, arial, sans-serif` | Fonte padrão do sistema |
| Valor bold | `true` | Negrito |
| Label fontSize | `9L` | Half-points (4.5pt real) |
| Label fontFamily | `Segoe UI` | Regular |
| Label matchValueAlignment | `true` | Alinha label com valor |
| Layout calloutSize | `54D` | Tamanho do callout |
| Layout contentOrder | `callout_referenceLabel_image` | Ordem do conteúdo |
| Layout autoGrid | `true` | Grid automático |
| Padding | `15L top`, `5L bottom`, `10L left/right` | Custom com paddingIndividual |
| Background (container) | `show: false` | Sem fundo visível no container |
| Border (container) | `show: false` | Sem borda no container |
| DropShadow (container) | `show: false` | Sem sombra no container |
| **Título (container)** | **`show: false`** | **SEM título no card** |
| **SubTítulo (container)** | **`show: false`** | **SEM subtítulo no card** |
| Accent bar | cor customizada por card | Verde, laranja, etc. |
| Outline | `false` | Sem contorno extra |

### Accent bars por KPI (sugestão)

| KPI | Cor | Hex |
|-----|-----|-----|
| Receita / Faturamento | Verde primário | `#2E7D32` |
| Margem / Percentual | Laranja accent | `#F57C00` |
| Ticket Médio / Valor médio | Verde escuro | `#1A3C34` |
| Volume / Quantidade | Verde oliva | `#558B2F` |

### JSON completo

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "<HEX_ID>",
  "position": { "x": "<X>", "y": 76, "z": 3, "height": 90, "width": 294 },
  "visual": {
    "visualType": "cardVisual",
    "query": {
      "queryState": {
        "Data": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<NOME_MEDIDA>" } },
            "queryRef": "<TABELA_MEDIDAS>.<NOME_MEDIDA>",
            "nativeQueryRef": "<NOME_MEDIDA>"
          }]
        }
      },
      "sortDefinition": {
        "sort": [{ "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<NOME_MEDIDA>" } }, "direction": "Descending" }],
        "isDefaultSort": true
      }
    },
    "objects": {
      "accentBar": [{"properties":{"color":{"solid":{"color":{"expr":{"Literal":{"Value":"'<COR_ACCENT>'"}}}}}},"selector":{"metadata":"<TABELA_MEDIDAS>.<NOME_MEDIDA>"}}],
      "outline": [
        {"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}}},
        {"properties":{"weight":{"expr":{"Literal":{"Value":"1D"}}}},"selector":{"id":"default"}}
      ],
      "layout": [
        {"properties":{
          "maxTiles":{"expr":{"Literal":{"Value":"10L"}}},
          "cellPadding":{"expr":{"Literal":{"Value":"2L"}}},
          "calloutSize":{"expr":{"Literal":{"Value":"54D"}}},
          "showFixedSize":{"expr":{"Literal":{"Value":"false"}}},
          "contentOrder":{"expr":{"Literal":{"Value":"'callout_referenceLabel_image'"}}},
          "autoGrid":{"expr":{"Literal":{"Value":"true"}}}
        }},
        {"properties":{"paddingUniform":{"expr":{"Literal":{"Value":"0L"}}}},"selector":{"id":"default"}}
      ],
      "label": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"true"}}},
        "fontSize":{"expr":{"Literal":{"Value":"9L"}}},
        "fontFamily":{"expr":{"Literal":{"Value":"'Segoe UI'"}}},
        "bold":{"expr":{"Literal":{"Value":"false"}}},
        "matchValueAlignment":{"expr":{"Literal":{"Value":"true"}}}
      },"selector":{"id":"default"}}],
      "spacing": [{"properties":{"verticalSpacing":{"expr":{"Literal":{"Value":"3L"}}}},"selector":{"id":"default"}}],
      "value": [{"properties":{
        "fontSize":{"expr":{"Literal":{"Value":"22L"}}},
        "fontFamily":{"expr":{"Literal":{"Value":"'wf_standard-font, helvetica, arial, sans-serif'"}}},
        "bold":{"expr":{"Literal":{"Value":"true"}}}
      },"selector":{"id":"default"}}],
      "padding": [{"properties":{
        "paddingSelection":{"expr":{"Literal":{"Value":"'Custom'"}}},
        "topMargin":{"expr":{"Literal":{"Value":"15L"}}},
        "leftMargin":{"expr":{"Literal":{"Value":"10L"}}},
        "rightMargin":{"expr":{"Literal":{"Value":"10L"}}},
        "bottomMargin":{"expr":{"Literal":{"Value":"5L"}}},
        "paddingIndividual":{"expr":{"Literal":{"Value":"true"}}}
      },"selector":{"id":"default"}}],
      "divider": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}}},"selector":{"id":"default"}}],
      "cardCalloutArea": [{"properties":{
        "paddingIndividual":{"expr":{"Literal":{"Value":"false"}}},
        "paddingTop":{"expr":{"Literal":{"Value":"0L"}}},
        "paddingBottom":{"expr":{"Literal":{"Value":"0L"}}}
      },"selector":{"id":"default"}}]
    },
    "visualContainerObjects": {
      "title": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"false"}}},
        "text":{"expr":{"Literal":{"Value":"'<TÍTULO DO CARD>'"}}},
        "titleWrap":{"expr":{"Literal":{"Value":"true"}}},
        "fontColor":{"solid":{"color":{"expr":{"Literal":{"Value":"'#757575'"}}}}},
        "fontSize":{"expr":{"Literal":{"Value":"10D"}}},
        "bold":{"expr":{"Literal":{"Value":"true"}}},
        "fontFamily":{"expr":{"Literal":{"Value":"'''Segoe UI Semibold'', wf_segoe-ui_semibold, helvetica, arial, sans-serif'"}}}
      }}],
      "subTitle": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}},"fontSize":{"expr":{"Literal":{"Value":"'12'"}}}}}],
      "padding": [{"properties":{
        "top":{"expr":{"Literal":{"Value":"0L"}}},
        "bottom":{"expr":{"Literal":{"Value":"0L"}}},
        "left":{"expr":{"Literal":{"Value":"0L"}}},
        "right":{"expr":{"Literal":{"Value":"0L"}}}
      }}],
      "background": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"false"}}},
        "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#FFFFFF'"}}}}},
        "transparency":{"expr":{"Literal":{"Value":"0D"}}}
      }}],
      "dropShadow": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"false"}}},
        "preset":{"expr":{"Literal":{"Value":"'Custom'"}}},
        "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#B0BEC5'"}}}}},
        "transparency":{"expr":{"Literal":{"Value":"75L"}}},
        "shadowSpread":{"expr":{"Literal":{"Value":"0L"}}},
        "shadowBlur":{"expr":{"Literal":{"Value":"8L"}}},
        "angle":{"expr":{"Literal":{"Value":"90L"}}},
        "shadowDistance":{"expr":{"Literal":{"Value":"3L"}}}
      }}],
      "border": [{"properties":{
        "show":{"expr":{"Literal":{"Value":"false"}}},
        "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#E0E4E2'"}}}}},
        "radius":{"expr":{"Literal":{"Value":"10L"}}},
        "width":{"expr":{"Literal":{"Value":"1D"}}}
      }}],
      "spacing": [
        {"properties":{
          "customizeSpacing":{"expr":{"Literal":{"Value":"true"}}},
          "spaceBelowSubTitle":{"expr":{"Literal":{"Value":"16D"}}}
        }},
        {"properties":{"verticalSpacing":{"expr":{"Literal":{"Value":"2D"}}}},"selector":{"id":"default"}}
      ]
    },
    "drillFilterOtherVisuals": true
  }
}
```

---

## 9. Template: Gráfico (Chart)

### Princípios de visualização limpa (SWD — Storytelling with Data)

1. **Sem título de eixo** — `showAxisTitle: false` em categoryAxis e valueAxis (o título do gráfico já dá contexto)
2. **Sem subtítulo** — `subTitle show: false` no `visualContainerObjects`
3. **Font dos eixos: 9D** — pequena e discreta, cor cinza (#757575)
4. **DataLabels vs Eixo — NUNCA ambos** — se dataLabels ON → valueAxis OFF + gridlines OFF. Se valueAxis ON → dataLabels OFF + gridlines sutis (#EEEEEE)
5. **Data labels** — preferencialmente em gráficos de barra/coluna, tamanho 9D, cor #757575. Quando ativados, ocultar valueAxis.
6. **Legenda** — posição Top, tamanho 9D, apenas quando há 2+ séries. Para 2 séries, prefira label direto na série.
7. **Título narrativo** — `Segoe UI Semibold`, 10D, cor #1A3C34. Comunique o insight ("Receita cresceu 12%") ao invés de apenas o tema ("Receita Mensal"). Sempre com `titleWrap: true`.
8. **Sort** — sempre explícito (Ascending para temporal, Descending para ranking)
9. **Cor estratégica** — cinza (#999999) para contexto/referência, cor primária (#2E7D32) para destaque. Máximo 3 cores por gráfico.
10. **Sem marcadores em linhas** — `showMarker: false` por padrão (exceto ponto de destaque isolado)
11. **Barras > espaço** — barras devem ser mais largas que o gap entre elas
12. **Sem 3D, gradientes ou sombras em barras** — simplicidade sempre
13. **Sem borda do gráfico** se não agrega informação — use espaço branco para separar

### ⚠️ Query Wells — REGRA CRÍTICA

| visualType | Well do Eixo/Categoria | Well dos Valores |
|------------|------------------------|------------------|
| `lineChart` | `Category` | `Y` |
| `clusteredBarChart` | `Category` | `Y` |
| `clusteredColumnChart` | `Category` | `Y` |
| `slicer` | `Values` | — |
| `cardVisual` | — | `Data` |

**NUNCA use `"Values"` como query well em gráficos (lineChart, clusteredBarChart, clusteredColumnChart).** O well correto para medidas nesses visuais é `"Y"`. Se usar "Values", o gráfico renderiza sem dados.

### Template de query para gráficos (1 série)

```json
"query": {
  "queryState": {
    "Category": {
      "projections": [{
        "field": { "Column": { "Expression": { "SourceRef": { "Entity": "<ENTIDADE_CATEGORIA>" } }, "Property": "<COLUNA_CATEGORIA>" } },
        "queryRef": "<ENTIDADE_CATEGORIA>.<COLUNA_CATEGORIA>",
        "active": true
      }]
    },
    "Y": {
      "projections": [{
        "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<NOME_MEDIDA>" } },
        "queryRef": "<TABELA_MEDIDAS>.<NOME_MEDIDA>"
      }]
    }
  },
  "sortDefinition": {
    "sort": [{ "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<NOME_MEDIDA>" } }, "direction": "<Ascending|Descending>" }]
  }
}
```

### Template de query para gráficos (2 séries no Y)

```json
"query": {
  "queryState": {
    "Category": {
      "projections": [{
        "field": { "Column": { "Expression": { "SourceRef": { "Entity": "<ENTIDADE_CATEGORIA>" } }, "Property": "<COLUNA_CATEGORIA>" } },
        "queryRef": "<ENTIDADE_CATEGORIA>.<COLUNA_CATEGORIA>",
        "active": true
      }]
    },
    "Y": {
      "projections": [
        {
          "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<MEDIDA_1>" } },
          "queryRef": "<TABELA_MEDIDAS>.<MEDIDA_1>"
        },
        {
          "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<TABELA_MEDIDAS>" } }, "Property": "<MEDIDA_2>" } },
          "queryRef": "<TABELA_MEDIDAS>.<MEDIDA_2>"
        }
      ]
    }
  },
  "sortDefinition": {
    "sort": [{ "field": { "Column": { "Expression": { "SourceRef": { "Entity": "<ENTIDADE_CATEGORIA>" } }, "Property": "<COLUNA_CATEGORIA>" } }, "direction": "Ascending" }]
  }
}
```

### VisualContainerObjects padrão (aplicar em TODOS os gráficos)

```json
"visualContainerObjects": {
  "dropShadow": [{"properties":{
    "show":{"expr":{"Literal":{"Value":"true"}}},
    "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#B0BEC5'"}}}}},
    "preset":{"expr":{"Literal":{"Value":"'Custom'"}}},
    "shadowSpread":{"expr":{"Literal":{"Value":"0L"}}},
    "shadowBlur":{"expr":{"Literal":{"Value":"8L"}}},
    "angle":{"expr":{"Literal":{"Value":"90L"}}},
    "shadowDistance":{"expr":{"Literal":{"Value":"3L"}}},
    "transparency":{"expr":{"Literal":{"Value":"75L"}}}
  }}],
  "background": [{"properties":{
    "show":{"expr":{"Literal":{"Value":"true"}}},
    "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#FFFFFF'"}}}}},
    "transparency":{"expr":{"Literal":{"Value":"0D"}}}
  }}],
  "border": [{"properties":{
    "show":{"expr":{"Literal":{"Value":"true"}}},
    "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#E0E4E2'"}}}}},
    "radius":{"expr":{"Literal":{"Value":"8L"}}},
    "width":{"expr":{"Literal":{"Value":"1D"}}}
  }}],
  "title": [{"properties":{
    "show":{"expr":{"Literal":{"Value":"true"}}},
    "text":{"expr":{"Literal":{"Value":"'<TÍTULO DO GRÁFICO>'"}}},
    "titleWrap":{"expr":{"Literal":{"Value":"true"}}},
    "fontColor":{"solid":{"color":{"expr":{"Literal":{"Value":"'#1A3C34'"}}}}},
    "fontSize":{"expr":{"Literal":{"Value":"10D"}}},
    "bold":{"expr":{"Literal":{"Value":"true"}}},
    "fontFamily":{"expr":{"Literal":{"Value":"'''Segoe UI Semibold'', wf_segoe-ui_semibold, helvetica, arial, sans-serif'"}}}
  }}],
  "subTitle": [{"properties":{"show":{"expr":{"Literal":{"Value":"false"}}},"fontSize":{"expr":{"Literal":{"Value":"'12'"}}}}}],
  "spacing": [{"properties":{
    "customizeSpacing":{"expr":{"Literal":{"Value":"true"}}},
    "verticalSpacing":{"expr":{"Literal":{"Value":"2D"}}},
    "spaceBelowSubTitle":{"expr":{"Literal":{"Value":"16D"}}}
  }}],
  "padding": [{"properties":{
    "top":{"expr":{"Literal":{"Value":"14D"}}},
    "bottom":{"expr":{"Literal":{"Value":"16D"}}},
    "left":{"expr":{"Literal":{"Value":"16D"}}},
    "right":{"expr":{"Literal":{"Value":"16D"}}}
  }}]
}
```

### Objects padrão para eixos (aplicar em TODOS os gráficos)

```json
"objects": {
  "categoryAxis": [{"properties":{
    "showAxisTitle":{"expr":{"Literal":{"Value":"false"}}},
    "fontSize":{"expr":{"Literal":{"Value":"9D"}}},
    "innerPadding":{"expr":{"Literal":{"Value":"30L"}}}
  }}],
  "valueAxis": [{"properties":{
    "showAxisTitle":{"expr":{"Literal":{"Value":"false"}}},
    "fontSize":{"expr":{"Literal":{"Value":"9D"}}},
    "gridlineShow":{"expr":{"Literal":{"Value":"true"}}},
    "gridlineColor":{"solid":{"color":{"expr":{"Literal":{"Value":"'#EEEEEE'"}}}}}
  }}]
}
```

### Cores por série (dataPoint)

Para gráficos com 2 séries (ex: valor atual vs comparação):
```json
"dataPoint": [
  {"properties":{"fill":{"solid":{"color":{"expr":{"Literal":{"Value":"'#2E7D32'"}}}}}},"selector":{"metadata":"<MEDIDA_1>"}},
  {"properties":{"fill":{"solid":{"color":{"expr":{"Literal":{"Value":"'#F57C00'"}}}}}},"selector":{"metadata":"<MEDIDA_2>"}}
]
```

Para gráficos com 1 série:
```json
"dataPoint": [{"properties":{"fill":{"solid":{"color":{"expr":{"Literal":{"Value":"'#2E7D32'"}}}}}}}]
```

### Data labels (apenas para gráficos de barra)

```json
"dataLabels": [{"properties":{
  "show":{"expr":{"Literal":{"Value":"true"}}},
  "fontSize":{"expr":{"Literal":{"Value":"9D"}}},
  "color":{"solid":{"color":{"expr":{"Literal":{"Value":"'#757575'"}}}}}
}}]
```

### Legenda (para gráficos com 2+ séries)

```json
"legend": [{"properties":{
  "show":{"expr":{"Literal":{"Value":"true"}}},
  "fontSize":{"expr":{"Literal":{"Value":"9D"}}},
  "position":{"expr":{"Literal":{"Value":"'Top'"}}}
}}]
```

### Line chart: estilos de linha

```json
"lineStyles": [
  {"properties":{"strokeWidth":{"expr":{"Literal":{"Value":"3D"}}},"lineStyle":{"expr":{"Literal":{"Value":"'solid'"}}}},"selector":{"metadata":"<MEDIDA_PRINCIPAL>"}},
  {"properties":{"strokeWidth":{"expr":{"Literal":{"Value":"2D"}}},"lineStyle":{"expr":{"Literal":{"Value":"'dashed'"}}}},"selector":{"metadata":"<MEDIDA_COMPARAÇÃO>"}}
]
```

---

## 10. Tema Customizado (JSON para RegisteredResources)

Salvar como `Coinest_Inspired_Green.json` em `StaticResources/RegisteredResources/`:

```json
{
  "name": "Coinest_Inspired_Green",
  "dataColors": ["#2E7D32", "#F57C00", "#558B2F", "#81C784", "#1A3C34", "#A5D6A7", "#F57C00", "#FFB74D"],
  "background": "#F4F6F5",
  "foreground": "#1A1A1A",
  "tableAccent": "#2E7D32",
  "textClasses": {
    "callout": { "fontSize": 24, "fontFace": "Segoe UI Semibold", "color": "#1A1A1A" },
    "title": { "fontSize": 14, "fontFace": "Segoe UI Semibold", "color": "#1A3C34" },
    "header": { "fontSize": 12, "fontFace": "Segoe UI Semibold", "color": "#1A1A1A" },
    "label": { "fontSize": 10, "fontFace": "Segoe UI", "color": "#757575" }
  },
  "visualStyles": {
    "*": {
      "*": {
        "background": [{ "color": { "solid": { "color": "#FFFFFF" } }, "transparency": 0 }],
        "border": [{ "show": true, "color": "#E0E4E2", "radius": 8 }],
        "dropShadow": [{ "show": true, "color": "#B0BEC5", "shadowBlur": 8, "shadowDistance": 3, "transparency": 75 }]
      }
    }
  }
}
```

---

## 11. Boas Práticas de Data Storytelling (baseado em Cole Nussbaumer Knaflic — Storytelling with Data)

Estas regras são fundamentais para criar visualizações limpas, claras e eficazes. Cada elemento desnecessário em um gráfico consome carga cognitiva do público. O objetivo é reduzir o "ruído" para que os dados falem por si.

### 11.1 Regra de Ouro: Rótulos de Dados vs Eixo de Valores

**Quando há rótulos de dados (dataLabels: show true), o eixo de valores (valueAxis) deve ser ocultado (show: false).** Mostrar ambos é redundância — a mesma informação aparece duas vezes. Escolha um:

| Cenário | dataLabels | valueAxis | gridlines |
|---------|-----------|-----------|-----------|
| Valores exatos são importantes (barras, colunas) | `show: true` | `show: false` | `show: false` |
| Tendência geral importa mais (linhas, áreas) | `show: false` | `show: true` | sutil `#EEEEEE` |
| Poucos pontos de dados (≤5 barras) | `show: true` | `show: false` | `show: false` |
| Muitos pontos de dados (>12) | `show: false` | `show: true` | sutil |

Regra prática para PBIR:
- Em `clusteredBarChart` e `clusteredColumnChart`: prefira dataLabels ON + valueAxis OFF + gridlines OFF
- Em `lineChart`: prefira valueAxis ON + gridlines sutis + dataLabels OFF (exceto ponto de destaque)

### 11.2 Eliminação de Clutter (Desordem Visual)

Remova sistematicamente tudo que não carrega informação:

| Elemento | Ação | Justificativa |
|----------|------|---------------|
| **Borda do gráfico** | `border show: false` ou cor muito sutil | Bordas não informam; use espaço em branco |
| **Gridlines** | Remover ou tornar muito sutis (#EEEEEE, 1px) | Se tem dataLabels, remover completamente |
| **Marcadores de linha** | Não usar em lineChart (exceto destaque) | Poluem a visualização sem agregar |
| **Títulos de eixo** | `showAxisTitle: false` sempre | O contexto já está no título do gráfico |
| **Tick marks** | Reduzir ao mínimo necessário | Menos marcas = menos ruído visual |
| **Efeitos 3D** | Jamais usar | Distorcem a comparação de valores |
| **Gradientes/sombras em barras** | Não usar | Simplicidade > decoração |
| **SubTítulo do visual** | `subTitle show: false` | Raramente necessário |
| **Legenda separada** | Preferir label direto na série | Evita "ida e volta" do olhar |

### 11.3 Uso Estratégico de Cor

Cor deve ser usada com parcimônia e intenção. A filosofia SWD é: **projete em cinza primeiro, depois adicione UMA cor de destaque.**

**Regras práticas para PBIR:**

1. **1 série de dados** → Use apenas a cor primária (#2E7D32). Sem legenda.
2. **2 séries comparativas** → Cor primária para o destaque + cinza (#999999) para a referência (ex: ano atual vs anterior)
3. **Destaque estratégico** → Use cor forte (#F57C00 laranja ou #2E7D32 verde) APENAS no elemento que conta a história. O restante em cinza neutro.
4. **Nunca usar mais de 3 cores** em um único gráfico — se precisar de mais, repense a visualização.
5. **Evite vermelho+verde juntos** — inacessível para daltônicos. Use laranja (#F57C00) no lugar de vermelho.
6. **Consistência** — a mesma medida deve ter a mesma cor em todos os gráficos da página.

### 11.4 Escolha de Tipo de Gráfico

| Objetivo | Tipo recomendado | Tipo a evitar |
|----------|-----------------|---------------|
| Comparar categorias | `clusteredBarChart` (horizontal) | Pie chart, donut |
| Tendência temporal | `lineChart` | Barras para séries temporais longas |
| Ranking (top N) | `clusteredBarChart` (horizontal, sort desc) | Tabela, column chart |
| Parte do todo | Stacked bar 100% | Pie chart |
| Comparar 2 períodos | `lineChart` com 2 séries (atual + anterior) | Tabela lado a lado |
| Valor único (KPI) | `cardVisual` ou texto simples | Gauge, donut com 1 valor |
| Desvio/variação | Barra positiva/negativa | Linhas para desvios |

**Regras de ouro para tipo de gráfico:**
- Se há **1 ou 2 números isolados**, use `cardVisual` ou texto simples — não gaste um gráfico.
- **Barras horizontais** são preferíveis a colunas quando os nomes das categorias são longos.
- **Barras devem ser mais largas que o espaço entre elas** — isso facilita a comparação.
- **Nunca use rótulos de eixo X na diagonal** — se não cabem, use barras horizontais ou abrevie.
- **Pie charts são desencorajados** — o olho humano é ruim em comparar ângulos/áreas.

### 11.5 Títulos que Contam a História

Em vez de títulos descritivos ("Vendas por Mês"), use títulos que comuniquem o insight:

| ❌ Título descritivo | ✅ Título narrativo |
|---------------------|---------------------|
| Receita por Estado | SP lidera com 26% da receita total |
| Vendas Mensais | Receita cresceu 12% vs ano anterior |
| Cancelamentos por Mês | Taxa de cancelamento estável em ~8% |

**Implementação em PBIR:** Use o campo `text` em `visualContainerObjects.title` para comunicar o insight principal. Se o dado é dinâmico, use o título descritivo como fallback — mas sempre que possível, inclua o "e daí?" ("so what?").

### 11.6 Hierarquia Visual e Princípios de Gestalt

Use estes princípios para organizar a informação na página:

- **Proximidade**: Agrupe visualmente elementos relacionados (cards de KPI em linha, gráficos relacionados lado a lado)
- **Similaridade**: Use mesma cor/estilo para dados da mesma natureza
- **Enclosure**: Use fundo/borda para agrupar (header bar = grupo de navegação)
- **Continuidade**: Alinhe gráficos em grid — o olhar segue linhas naturais
- **Leitura em Z**: Coloque o KPI mais importante no canto superior esquerdo (Card 1)

**Posição = importância:**
- Topo-esquerda → Informação mais importante
- Cards → Resumo executivo (visão em 3 segundos)
- Gráficos superiores → Análise principal
- Gráficos inferiores → Detalhamento / drill-down

### 11.7 Atributos Pré-Atentivos

Atributos visuais processados pelo cérebro em menos de 250ms. Use UM atributo por gráfico:

| Atributo | Como usar em PBIR |
|----------|-------------------|
| **Cor** | dataPoint fill para destacar uma barra/série |
| **Tamanho** | fontSize maior no card principal (22L vs 18L) |
| **Posição** | Card 1 no canto superior esquerdo |
| **Orientação** | Barras horizontais para ranking, verticais para temporal |

**Regra: use no máximo 1 atributo de destaque por gráfico.** Se tudo está destacado, nada está destacado.

### 11.8 Framework Narrativo

Toda página de dashboard deve seguir o framework "O quê → E daí → E agora":

1. **O quê?** → Cards de KPI mostram os números-chave
2. **E daí?** → Gráficos mostram o contexto (tendência, comparação, composição)
3. **E agora?** → Título narrativo ou callout indica a ação necessária

---

## 12. Regras SWD Aplicadas ao JSON PBIR

Tradução direta das regras de Data Storytelling para propriedades JSON dos visuais:

### 12.1 Gráfico de Barras/Colunas com DataLabels (sem eixo de valores)

```json
"objects": {
  "dataLabels": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "fontSize": {"expr": {"Literal": {"Value": "9D"}}},
    "color": {"solid": {"color": {"expr": {"Literal": {"Value": "'#757575'"}}}}}
  }}],
  "categoryAxis": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "showAxisTitle": {"expr": {"Literal": {"Value": "false"}}},
    "fontSize": {"expr": {"Literal": {"Value": "9D"}}}
  }}],
  "valueAxis": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "false"}}}
  }}]
}
```

### 12.2 Gráfico de Linha sem DataLabels (com eixo de valores sutil)

```json
"objects": {
  "categoryAxis": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "showAxisTitle": {"expr": {"Literal": {"Value": "false"}}},
    "fontSize": {"expr": {"Literal": {"Value": "9D"}}}
  }}],
  "valueAxis": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "showAxisTitle": {"expr": {"Literal": {"Value": "false"}}},
    "fontSize": {"expr": {"Literal": {"Value": "9D"}}},
    "gridlineShow": {"expr": {"Literal": {"Value": "true"}}},
    "gridlineColor": {"solid": {"color": {"expr": {"Literal": {"Value": "'#EEEEEE'"}}}}}
  }}],
  "lineStyles": [{"properties": {
    "showMarker": {"expr": {"Literal": {"Value": "false"}}}
  }}],
  "legend": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "position": {"expr": {"Literal": {"Value": "'Top'"}}},
    "fontSize": {"expr": {"Literal": {"Value": "9D"}}}
  }}]
}
```

### 12.3 Destaque com Cor Estratégica (1 série destaque + 1 cinza)

```json
"dataPoint": [
  {"properties": {"fill": {"solid": {"color": {"expr": {"Literal": {"Value": "'#2E7D32'"}}}}}},
   "selector": {"metadata": "Medidas.Receita Concluída"}},
  {"properties": {"fill": {"solid": {"color": {"expr": {"Literal": {"Value": "'#999999'"}}}}}},
   "selector": {"metadata": "Medidas.Receita Ano Anterior"}}
]
```

### 12.4 Container Limpo (sem borda, sem sombra desnecessária)

Para gráficos com dataLabels onde o visual já é auto-explicativo:

```json
"visualContainerObjects": {
  "border": [{"properties": {"show": {"expr": {"Literal": {"Value": "false"}}}}}],
  "dropShadow": [{"properties": {"show": {"expr": {"Literal": {"Value": "false"}}}}}],
  "background": [{"properties": {
    "show": {"expr": {"Literal": {"Value": "true"}}},
    "color": {"solid": {"color": {"expr": {"Literal": {"Value": "'#FFFFFF'"}}}}},
    "transparency": {"expr": {"Literal": {"Value": "0D"}}}
  }}]
}
```

---

## 13. Checklist de Criação de Página

Ao criar uma nova página, siga esta ordem:

1. **Gerar hex IDs únicos** para a página e cada visual
2. **Criar page.json** com fundo `#F4F6F5` e `displayOption: FitToPage`
3. **Header bar** (shape) — z:0, verde escuro, full width
4. **TextBox** — z:6, título branco sobre o header
5. **Slicers** — z:5, dropdown, posicionados no canto direito do header, **height ≥ 55 se header.show: true**
6. **Cards** — z:3, 4 em linha com gap de 16px, **SEM título (show: false)**
7. **Gráficos** — z:2, grid 2×2 abaixo dos cards, **query well "Y" para medidas**
8. **Atualizar pages.json** — adicionar ao `pageOrder`
9. **OBRIGATÓRIO: subTitle show: false** em TODOS os visuais
10. **Nunca usar** títulos de eixo (`showAxisTitle: false`)
11. **Gridlines**: remover se há dataLabels; manter sutil se não há
12. **Sempre usar** sort explícito
13. **Sempre usar** `drillFilterOtherVisuals: true`
14. **Schema** — `visualContainer/2.8.0` em todos os visuais
15. **Query wells**: `"Y"` para gráficos, `"Values"` para slicers, `"Data"` para cards
16. **Títulos de gráfico**: incluir `titleWrap: true` e `fontFamily` no visualContainerObjects.title
17. **Spacing/Padding**: incluir `spacing` e `padding` no visualContainerObjects dos gráficos
18. **SWD — DataLabels vs Eixo**: se dataLabels ON → valueAxis OFF + gridlines OFF
19. **SWD — Cor estratégica**: cinza para contexto, cor forte apenas para destaque
20. **SWD — Sem marcadores**: lineChart com `showMarker: false` (exceto ponto de destaque)
21. **SWD — Barras > espaço**: barras mais largas que o gap entre elas
22. **SWD — Sem 3D/gradientes/sombras desnecessárias**: simplicidade sempre
23. **SWD — Hierarquia visual**: KPI mais importante no Card 1 (topo-esquerda)
24. **SWD — Título narrativo**: quando possível, título comunica o insight, não apenas o tema
25. **Slicer com título**: height mínimo de 55px quando `header.show: true` (40px corta o título)
