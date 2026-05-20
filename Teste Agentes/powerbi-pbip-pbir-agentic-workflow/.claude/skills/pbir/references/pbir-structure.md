# PBIP/PBIR Complete Structure Reference

This document contains the full file hierarchy, JSON schemas, and TMDL syntax for Power BI projects in the modern PBIP/PBIR format.

## Table of Contents

1. [Project Hierarchy](#1-project-hierarchy)
2. [PBIP Project File](#2-pbip-project-file)
3. [Report Structure (PBIR)](#3-report-structure-pbir)
4. [Semantic Model Structure](#4-semantic-model-structure)
5. [TMDL Syntax](#5-tmdl-syntax)
6. [Expression Patterns](#6-expression-patterns)
7. [Visual Types Reference](#7-visual-types-reference)
8. [Schema URLs](#8-schema-urls)

---

## 1. Project Hierarchy

```
project-name/
├── project-name.pbip                           # Project manifest
├── .gitignore                                  # Auto-generated ignore rules
│
├── project-name.Report/                        # Report artifact
│   ├── definition.pbir                         # Report metadata + semantic model reference
│   ├── .platform                               # Fabric integration metadata
│   ├── .pbi/
│   │   ├── localSettings.json                  # Personal settings (GITIGNORED)
│   │   └── cache.abf                           # Local cache (GITIGNORED)
│   ├── definition/
│   │   ├── version.json                        # Report format version
│   │   ├── report.json                         # Global report settings, themes, resources
│   │   ├── pages/
│   │   │   ├── pages.json                      # Page order and active page
│   │   │   └── <PageSection>/
│   │   │       ├── page.json                   # Page dimensions, background, display options
│   │   │       └── visuals/
│   │   │           └── <VisualHexId>/
│   │   │               └── visual.json         # Individual visual definition
│   │   └── bookmarks/
│   │       ├── bookmarks.json                  # Bookmark list
│   │       └── <BookmarkId>.bookmark.json      # Individual bookmark state
│   ├── CustomVisuals/
│   │   └── <VisualId>/
│   │       ├── package.json
│   │       └── resources/
│   │           ├── .pbiviz.json
│   │           ├── *.js
│   │           ├── *.css
│   │           └── icon.png
│   └── StaticResources/
│       ├── RegisteredResources/                # User images, custom themes
│       └── SharedResources/                    # Base themes
│
└── project-name.SemanticModel/                 # Semantic model artifact
    ├── definition.pbism                        # Semantic model metadata
    ├── .platform                               # Fabric integration metadata
    ├── diagramLayout.json                      # Model diagram view layout
    ├── .pbi/
    │   ├── localSettings.json                  # GITIGNORED
    │   └── cache.abf                           # GITIGNORED
    └── definition/
        ├── database.tmdl                       # Compatibility level
        ├── model.tmdl                          # Model config, culture, table refs
        ├── relationships.tmdl                  # All relationships
        ├── tables/
        │   ├── Table1.tmdl                     # Table schema + M query
        │   ├── Table2.tmdl
        │   └── MeasureTable.tmdl               # Dedicated measure table
        └── cultures/
            └── pt-BR.tmdl                      # Linguistic metadata
```

---

## 2. PBIP Project File

**File:** `project-name.pbip`

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/pbip/pbipProperties/1.0.0/schema.json",
  "version": "1.0",
  "artifacts": [
    {
      "report": {
        "path": "project-name.Report"
      }
    }
  ],
  "settings": {
    "enableAutoRecovery": true
  }
}
```

---

## 3. Report Structure (PBIR)

### 3.1 definition.pbir

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definitionProperties/2.0.0/schema.json",
  "version": "4.0",
  "datasetReference": {
    "byPath": {
      "path": "../project-name.SemanticModel"
    }
  }
}
```

### 3.2 .platform

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/gitIntegration/platformProperties/2.0.0/schema.json",
  "metadata": {
    "type": "Report",
    "displayName": "Display Name"
  },
  "config": {
    "version": "2.0",
    "logicalId": "uuid-string"
  }
}
```

`type` can be: `"Report"`, `"SemanticModel"`, `"Dashboard"`

### 3.3 report.json

Global report settings, themes, custom visuals, and resource packages.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/report/3.2.0/schema.json",
  "themeCollection": {
    "baseTheme": {
      "name": "CY24SU06",
      "reportVersionAtImport": {
        "visual": "2.8.0",
        "report": "3.2.0",
        "page": "2.3.1"
      },
      "type": "SharedResources"
    },
    "customTheme": {
      "name": "theme-name",
      "reportVersionAtImport": { ... },
      "type": "RegisteredResources"
    }
  },
  "publicCustomVisuals": ["VisualId1", "VisualId2"],
  "resourcePackages": [
    {
      "name": "SharedResources",
      "type": "SharedResources",
      "items": [
        {"name": "base-theme.json", "path": "BaseThemes/base-theme.json", "type": "BaseTheme"}
      ]
    },
    {
      "name": "RegisteredResources",
      "type": "RegisteredResources",
      "items": [
        {"name": "image.svg", "path": "RegisteredResources/image.svg", "type": "Image"},
        {"name": "custom-theme.json", "path": "RegisteredResources/custom-theme.json", "type": "CustomTheme"}
      ]
    }
  ],
  "settings": {
    "useStylableVisualContainerHeader": true,
    "defaultFilterActionIsDataFilter": true,
    "defaultDrillFilterOtherVisuals": true,
    "allowChangeFilterTypes": true,
    "useEnhancedTooltips": true,
    "filterPaneHiddenInEditMode": false
  }
}
```

### 3.4 pages.json

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/pagesMetadata/1.0.0/schema.json",
  "pageOrder": ["PageSection1", "PageSection2", "PageSection3"],
  "activePageName": "PageSection1"
}
```

### 3.5 page.json

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.1.0/schema.json",
  "name": "PageSection1",
  "displayName": "Visão Geral",
  "displayOption": "FitToPage",
  "height": 720.00,
  "width": 1280.00,
  "objects": {
    "background": [{
      "properties": {
        "color": {
          "solid": {
            "color": {"expr": {"Literal": {"Value": "'#FFFFFF'"}}}
          }
        },
        "transparency": {"expr": {"Literal": {"Value": "0D"}}},
        "image": {
          "image": {
            "name": {"expr": {"Literal": {"Value": "'bg.png'"}}},
            "url": {"expr": {"ResourcePackageItem": {"PackageName": "RegisteredResources", "PackageItem": {"ItemName": "bg.png"}}}},
            "scaling": {"expr": {"Literal": {"Value": "'Fit'"}}}
          }
        }
      }
    }]
  }
}
```

**displayOption values:** `"FitToPage"`, `"FitToWidth"`, `"ActualSize"`, custom dimensions via height/width

### 3.6 visual.json

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "abc123def456",
  "position": {
    "x": 100.0,
    "y": 50.0,
    "z": 5000,
    "height": 300.0,
    "width": 400.0,
    "tabOrder": 1000
  },
  "visual": {
    "visualType": "clusteredColumnChart",
    "query": {
      "queryState": {
        "Category": {
          "projections": [{
            "field": {
              "Column": {
                "Expression": {"SourceRef": {"Entity": "Products"}},
                "Property": "CategoryName"
              }
            },
            "queryRef": "Products.CategoryName",
            "active": true
          }]
        },
        "Y": {
          "projections": [{
            "field": {
              "Measure": {
                "Expression": {"SourceRef": {"Entity": "Sales"}},
                "Property": "TotalRevenue"
              }
            },
            "queryRef": "Sales.TotalRevenue"
          }]
        }
      },
      "sortDefinition": {
        "sort": [{
          "field": {
            "Measure": {
              "Expression": {"SourceRef": {"Entity": "Sales"}},
              "Property": "TotalRevenue"
            }
          },
          "direction": "Descending"
        }],
        "isDefaultSort": true
      }
    },
    "objects": {
      "dataLabels": [{
        "properties": {
          "show": {"expr": {"Literal": {"Value": "true"}}},
          "fontSize": {"expr": {"Literal": {"Value": "12D"}}},
          "color": {"solid": {"color": {"expr": {"ThemeDataColor": {"ColorId": 1, "Percent": 0}}}}}
        }
      }],
      "categoryAxis": [{
        "properties": {
          "show": {"expr": {"Literal": {"Value": "true"}}}
        }
      }],
      "valueAxis": [{
        "properties": {
          "show": {"expr": {"Literal": {"Value": "true"}}}
        }
      }]
    }
  }
}
```

**Query well slots** (depend on visual type):
- `Category` / `Axis` — dimension fields
- `Values` — measure fields
- `Series` / `Legend` — legend/series fields
- `Rows`, `Columns` — for matrix visuals
- `Tooltips` — tooltip fields

### 3.7 bookmarks.json

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/bookmarksMetadata/1.0.0/schema.json",
  "items": [
    {"name": "Bookmark1"},
    {"name": "Bookmark2"}
  ]
}
```

### 3.8 Individual bookmark

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/bookmark/2.0.0/schema.json",
  "displayName": "Show Product View",
  "name": "Bookmark1",
  "options": {
    "targetVisualNames": ["visual-hex-id-1", "visual-hex-id-2"]
  },
  "explorationState": {
    "version": "1.0",
    "activeSection": "PageSection1",
    "sections": {
      "PageSection1": {
        "visualContainers": {
          "visual-hex-id-1": {
            "isVisible": true
          }
        }
      }
    }
  }
}
```

---

## 4. Semantic Model Structure

### 4.1 definition.pbism

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/semanticModel/definitionProperties/1.0.0/schema.json",
  "version": "4.2",
  "settings": {}
}
```

### 4.2 diagramLayout.json

```json
{
  "version": "1.1.0",
  "diagrams": [{
    "ordinal": 0,
    "scrollPosition": {"x": -200, "y": 100},
    "nodes": [
      {
        "location": {"x": 50, "y": 100},
        "nodeIndex": "TableName",
        "size": {"height": 300, "width": 250},
        "zIndex": 0
      }
    ]
  }]
}
```

---

## 5. TMDL Syntax

### 5.1 database.tmdl

```tmdl
database
	compatibilityLevel: 1600
```

### 5.2 model.tmdl

```tmdl
model Model
	culture: pt-BR
	defaultPowerBIDataSourceVersion: powerBI_V3
	sourceQueryCulture: pt-BR
	dataAccessOptions
		legacyRedirects
		returnErrorValuesAsNull

	annotation __PBI_TimeIntelligenceEnabled = 0
	annotation PBI_QueryOrder = ["Table1","Table2","Table3"]
	annotation PBI_ProTooling = ["MCP-PBIModeling","DevMode"]

	ref table Table1
	ref table Table2
	ref table MeasureTable

	ref cultureInfo pt-BR
```

### 5.3 Table with columns and partitions

```tmdl
/// Description of the table
table Products
	lineageTag: a1b2c3d4-e5f6-7890-abcd-ef1234567890

	/// Product unique identifier
	column ProductID
		dataType: int64
		lineageTag: 11111111-2222-3333-4444-555555555555
		summarizeBy: none
		sourceColumn: ProductID

		annotation SummarizationSetBy = Automatic

	column ProductName
		dataType: string
		lineageTag: aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
		summarizeBy: none
		sourceColumn: ProductName

		annotation SummarizationSetBy = Automatic

	column Price
		dataType: double
		lineageTag: 12345678-abcd-ef01-2345-678901234567
		summarizeBy: sum
		sourceColumn: Price

		annotation SummarizationSetBy = Automatic
		annotation PBI_FormatHint = {"isGeneralNumber":true}

	column DateAdded
		dataType: dateTime
		formatString: dd/MM/yyyy
		lineageTag: fedcba98-7654-3210-fedc-ba9876543210
		summarizeBy: none
		sourceColumn: DateAdded

		annotation SummarizationSetBy = Automatic

	partition Products = m
		mode: import
		source =
			let
			    Fonte = Sql.Database("server.database.windows.net", "MyDB"),
			    dbo_Products = Fonte{[Schema="dbo",Item="Products"]}[Data]
			in
			    dbo_Products

		annotation PBI_ResultType = Table
```

### 5.4 Measure table

```tmdl
table Measures
	lineageTag: 99999999-8888-7777-6666-555555555555

	measure 'Total Revenue' = SUM(Sales[Amount])
		formatString: #,0.00
		displayFolder: Revenue
		lineageTag: aaaa1111-bbbb-2222-cccc-3333dddd4444

	measure 'Profit Margin %' = DIVIDE([Total Revenue] - [Total Cost], [Total Revenue], 0)
		formatString: 0.00%;-0.00%;0.00%
		displayFolder: Profitability
		lineageTag: eeee5555-ffff-6666-0000-111122223333

	measure 'YTD Revenue' = TOTALYTD([Total Revenue], Calendar[Date])
		formatString: R$\ #,0.00;-R$\ #,0.00;R$\ #,0.00
		displayFolder: Time Intelligence
		lineageTag: 44445555-6666-7777-8888-999900001111

		annotation PBI_FormatHint = {"currencyCulture":"pt-BR"}
```

### 5.5 relationships.tmdl

```tmdl
relationship 12345678-abcd-ef01-2345-678901234567
	fromColumn: Sales.ProductID
	toColumn: Products.ProductID

relationship 87654321-dcba-10fe-5432-109876543210
	fromColumn: Sales.DateKey
	toColumn: Calendar.DateKey
```

Relationships are many-to-one by default (from = many side, to = one side).

### 5.6 cultures (linguistic metadata)

```tmdl
cultureInfo pt-BR

	linguisticMetadata =
		{
		  "Version": "2.0.0",
		  "Language": "pt-BR"
		}
```

---

## 6. Expression Patterns

Used in visual.json and page.json property values:

| Pattern | Syntax | Example |
|---------|--------|---------|
| String literal | `{"expr":{"Literal":{"Value":"'text'"}}}` | `'#FF0000'`, `'Fit'` |
| Number literal | `{"expr":{"Literal":{"Value":"13D"}}}` | `0D`, `100D` |
| Boolean | `{"expr":{"Literal":{"Value":"true"}}}` | `true`, `false` |
| Theme color | `{"expr":{"ThemeDataColor":{"ColorId":0,"Percent":0}}}` | ColorId 0-12, Percent -100 to 100 |
| Column ref | `{"Column":{"Expression":{"SourceRef":{"Entity":"Table"}},"Property":"Col"}}` | |
| Measure ref | `{"Measure":{"Expression":{"SourceRef":{"Entity":"Table"}},"Property":"Meas"}}` | |
| Resource ref | `{"expr":{"ResourcePackageItem":{"PackageName":"RegisteredResources","PackageItem":{"ItemName":"file.png"}}}}` | |

---

## 7. Visual Types Reference

Common `visualType` values:

| visualType | Description |
|------------|-------------|
| `clusteredColumnChart` | Clustered column chart |
| `clusteredBarChart` | Clustered bar chart |
| `stackedColumnChart` | Stacked column chart |
| `stackedBarChart` | Stacked bar chart |
| `lineChart` | Line chart |
| `areaChart` | Area chart |
| `lineClusteredColumnComboChart` | Line + column combo |
| `pieChart` | Pie chart |
| `donutChart` | Donut chart |
| `treemap` | Treemap |
| `table` | Table |
| `matrix` | Matrix |
| `card` | Card (single value) |
| `multiRowCard` | Multi-row card |
| `kpi` | KPI |
| `slicer` | Slicer |
| `map` | Map |
| `filledMap` | Filled/choropleth map |
| `scatter` | Scatter chart |
| `gauge` | Gauge |
| `waterfallChart` | Waterfall chart |
| `funnel` | Funnel chart |
| `image` | Image |
| `textbox` | Text box |
| `shape` | Shape |
| `actionButton` | Button |
| `bookmarkNavigator` | Bookmark navigator |
| `pageNavigator` | Page navigator |

---

## 8. Schema URLs

All PBIR JSON files declare their schema. Key URLs:

| File | Schema URL |
|------|-----------|
| .pbip | `https://developer.microsoft.com/json-schemas/fabric/pbip/pbipProperties/1.0.0/schema.json` |
| definition.pbir | `https://developer.microsoft.com/json-schemas/fabric/item/report/definitionProperties/2.0.0/schema.json` |
| definition.pbism | `https://developer.microsoft.com/json-schemas/fabric/item/semanticModel/definitionProperties/1.0.0/schema.json` |
| report.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/report/3.2.0/schema.json` |
| pages.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/pagesMetadata/1.0.0/schema.json` |
| page.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.1.0/schema.json` |
| visual.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json` |
| bookmarks.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/bookmarksMetadata/1.0.0/schema.json` |
| bookmark.json | `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/bookmark/2.0.0/schema.json` |
| .platform | `https://developer.microsoft.com/json-schemas/fabric/gitIntegration/platformProperties/2.0.0/schema.json` |
