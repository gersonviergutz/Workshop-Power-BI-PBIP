# Template Specification: Documentação Power BI

Complete visual and structural specification extracted from the reference .docx template. Every measurement, color, font size, and spacing value is exact. Follow these specs precisely when generating the document.

## Table of Contents

1. [Page Setup](#1-page-setup)
2. [Color Palette](#2-color-palette)
3. [Typography](#3-typography)
4. [Cover Page (Section 1)](#4-cover-page-section-1)
5. [Content Pages (Section 2)](#5-content-pages-section-2)
6. [Headers & Footers](#6-headers--footers)
7. [Table of Contents](#7-table-of-contents)
8. [Tables](#8-tables)
9. [Notes & Blockquotes](#9-notes--blockquotes)
10. [Lists](#10-lists)
11. [Section Catalog](#11-section-catalog)
12. [Logo Asset](#12-logo-asset)

---

## 1. Page Setup

| Property | Value | Notes |
|----------|-------|-------|
| Page Size | 12240 × 15840 DXA | US Letter (8.5" × 11") |
| Top Margin | 1100 DXA | ~0.76" |
| Bottom Margin | 1100 DXA | ~0.76" |
| Left Margin | 1200 DXA | ~0.83" |
| Right Margin | 1200 DXA | ~0.83" |
| Header Distance | 708 DXA | ~0.49" |
| Footer Distance | 708 DXA | ~0.49" |
| Gutter | 0 DXA | No gutter |
| Content Width | 9840 DXA | 12240 - 1200 - 1200 |
| Orientation | Portrait | Default |

```javascript
page: {
  size: { width: 12240, height: 15840 },
  margin: { top: 1100, right: 1200, bottom: 1100, left: 1200, header: 708, footer: 708, gutter: 0 }
}
```

---

## 2. Color Palette

### Primary Colors

| Name | Hex | Usage |
|------|-----|-------|
| Deep Purple | #4A148C | H1 headings, table header background, header border |
| Medium Purple | #7B1FA2 | H2 headings, cover subtitle |
| Light Orange (Peach) | #FFF3E0 | Table alternating row fill |
| Dark Text | #212121 | Body text, table data text |
| Gray Text | #616161 | Cover meta text, notes, header/footer text |
| White | #FFFFFF | Table header text |
| Light Gray | #CCCCCC | Footer top border |

### Color Application Map

```
Cover Title:        #4A148C (bold, 28pt)
Cover Subtitle:     #7B1FA2 (bold, 15pt)
Cover Meta:         #616161 (italic/regular, 13pt/9pt)
H1 Headings:        #4A148C (bold, 14pt)
H2 Headings:        #7B1FA2 (bold, 12pt)
Body Text:          #212121 (regular, 9pt)
Table Header BG:    #4A148C
Table Header Text:  #FFFFFF (bold, 9pt)
Table Data Text:    #212121 (regular, 9pt)
Table Alt Row BG:   #FFF3E0
Table Normal Row:   no fill (white)
Notes:              #616161 (italic, 9pt)
Header Text:        #616161 (italic, 8pt)
Footer Text:        #616161 (regular, 8pt)
Hyperlinks:         #0563C1
```

---

## 3. Typography

### Font Family

| Element | Font | Fallback |
|---------|------|----------|
| Default (body) | Arial | System default |
| Headings H1 | Theme Major (Calibri Light) | Arial |
| All other text | Theme Minor (Calibri) | Arial |

In docx-js, use `"Arial"` for all elements for maximum compatibility.

### Font Sizes (in half-points for docx-js `sz` property)

| Element | Points | Half-points (sz) |
|---------|--------|-------------------|
| Cover Title ("DOCUMENTAÇÃO COMPLETA") | 28pt | 56 |
| Cover Subtitle ("Modelo Semântico Power BI") | 15pt | 30 |
| Cover Model Name | 13pt | 26 |
| Cover Version | 11pt (default) | 22 |
| Cover Footer Note | 9pt | 18 |
| Heading 1 | 14pt | 28 |
| Heading 2 | 12pt | 24 |
| Body / Table Data | 9pt | 18 |
| Header Text | 8pt | 16 |
| Footer Text | 8pt | 16 |
| Notes (Nota:) | 9pt | 18 |

### Default Document Font

```javascript
styles: {
  default: {
    document: {
      run: { font: "Arial", size: 22 } // 11pt default
    }
  }
}
```

---

## 4. Cover Page (Section 1)

The cover page is the first section with its own header (centered logo, larger) and no footer. Section break separates it from the content.

### Cover Layout

All content is center-aligned with vertical spacing creating visual hierarchy:

```
[Logo in header - centered, 787400 × 442592 EMU]

                    [vertical space]

          DOCUMENTAÇÃO COMPLETA              <- 28pt, bold, #4A148C, center
            spacing: after 200 DXA

         Modelo Semântico Power BI           <- 15pt, bold, #7B1FA2, center
            spacing: after 100 DXA

            {Model Name Here}               <- 13pt, italic, #616161, center
            spacing: after 400 DXA

      Versão X.X  |  DD de Mês de AAAA      <- 11pt, regular, #616161, center
            spacing: after 60 DXA

   Gerado automaticamente a partir dos       <- 9pt, italic, #616161, center
       arquivos PBIR/TMDL
```

### Cover Page Section Properties

```javascript
// Section 1: Cover page
{
  properties: {
    page: { size: { width: 12240, height: 15840 }, margin: { ... } },
    // Header reference: header1 (centered large logo)
  },
  children: [
    // Empty paragraphs for vertical spacing before content
    // ... cover content paragraphs ...
  ]
}
```

### Cover Content docx-js Implementation

```javascript
// Title
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { after: 200 },
  children: [new TextRun({
    text: "DOCUMENTAÇÃO COMPLETA",
    bold: true,
    color: "4A148C",
    size: 56, // 28pt
    font: "Arial"
  })]
}),

// Subtitle
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { after: 100 },
  children: [new TextRun({
    text: "Modelo Semântico Power BI",
    bold: true,
    color: "7B1FA2",
    size: 30, // 15pt
    font: "Arial"
  })]
}),

// Model Name
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { after: 400 },
  children: [new TextRun({
    text: modelName, // e.g. "Dashboard Estoque"
    italics: true,
    color: "616161",
    size: 26, // 13pt
    font: "Arial"
  })]
}),

// Version & Date
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { after: 60 },
  children: [new TextRun({
    text: `Versão ${version}  |  ${dateStr}`,
    color: "616161",
    font: "Arial"
  })]
}),

// Auto-generated note
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [new TextRun({
    text: "Gerado automaticamente a partir dos arquivos PBIR/TMDL",
    italics: true,
    color: "616161",
    size: 18, // 9pt
    font: "Arial"
  })]
})
```

---

## 5. Content Pages (Section 2)

### Heading Styles

```javascript
paragraphStyles: [
  {
    id: "Heading1",
    name: "Heading 1",
    basedOn: "Normal",
    next: "Normal",
    quickFormat: true,
    run: { size: 28, bold: true, color: "4A148C", font: "Arial" },
    paragraph: { spacing: { before: 300, after: 200 }, outlineLevel: 0 }
  },
  {
    id: "Heading2",
    name: "Heading 2",
    basedOn: "Normal",
    next: "Normal",
    quickFormat: true,
    run: { size: 24, bold: true, color: "7B1FA2", font: "Arial" },
    paragraph: { spacing: { before: 240, after: 160 }, outlineLevel: 1 }
  }
]
```

### Body Text

```javascript
// Standard body paragraph
new Paragraph({
  spacing: { after: 100 },
  children: [new TextRun({
    text: "...",
    color: "212121",
    size: 18, // 9pt
    font: "Arial"
  })]
})
```

---

## 6. Headers & Footers

### Header 1 (Cover Page Only)

- Logo: centered horizontally, anchored to margin
- Logo size: 787400 × 442592 EMU (~2.17" × 1.22")
- Position: aligned center, vertical offset -284480 EMU (above paragraph)
- No text, no border

### Header 2 (Content Pages)

- Logo: left-aligned, smaller
- Logo size: 474476 × 266700 EMU (~1.31" × 0.74")
- Position: aligned left, vertical offset -136102 EMU
- Text: right-aligned, italic
- Text content: `"{Model Name}  |  Documentação do Modelo Semântico"`
- Text format: italic, 8pt (sz=16), color #616161
- Bottom border: single line, size 6, color #4A148C, space 4

```javascript
// Content page header paragraph
new Paragraph({
  border: { bottom: { style: BorderStyle.SINGLE, size: 6, space: 4, color: "4A148C" } },
  alignment: AlignmentType.RIGHT,
  children: [
    // Logo as ImageRun (anchor/inline)
    new TextRun({
      text: `${modelName}  |  Documentação do Modelo Semântico`,
      italics: true,
      color: "616161",
      size: 16, // 8pt
      font: "Arial"
    })
  ]
})
```

### Footer (Content Pages Only)

- Top border: single line, size 1, color #CCCCCC, space 4
- Text: centered
- Content: "Página {PAGE}" with auto page number field
- Text format: regular, 8pt (sz=16), color #616161

```javascript
new Paragraph({
  border: { top: { style: BorderStyle.SINGLE, size: 1, space: 4, color: "CCCCCC" } },
  alignment: AlignmentType.CENTER,
  children: [
    new TextRun({ text: "Página ", color: "616161", size: 16 }),
    new TextRun({ children: [PageNumber.CURRENT], color: "616161", size: 16 })
  ]
})
```

---

## 7. Table of Contents

- Auto-generated from Heading 1 and Heading 2 styles
- TOC 1 style: spacing after 100 DXA
- TOC 2 style: spacing after 100 DXA, indent left 200 DXA
- Tab stop: right-aligned with dot leader at position 9830 DXA
- Preceded by H1 "Sumário"

```javascript
new TableOfContents("Sumário", {
  hyperlink: true,
  headingStyleRange: "1-2"
})
```

---

## 8. Tables

All tables in the document follow the same visual pattern. This is critical for consistency.

### Table Properties

```javascript
{
  width: { size: 9840, type: WidthType.DXA }, // Full content width
  borders: {
    top: { style: BorderStyle.SINGLE, size: 4, color: "auto" },
    bottom: { style: BorderStyle.SINGLE, size: 4, color: "auto" },
    left: { style: BorderStyle.SINGLE, size: 4, color: "auto" },
    right: { style: BorderStyle.SINGLE, size: 4, color: "auto" },
    insideHorizontal: { style: BorderStyle.SINGLE, size: 4, color: "auto" },
    insideVertical: { style: BorderStyle.SINGLE, size: 4, color: "auto" }
  },
  // Cell margins are set at table level
  // Left/Right cell margin: 10 DXA (very tight, content relies on cell-level margins)
}
```

### Header Row (First Row)

```javascript
new TableRow({
  tableHeader: true,
  children: columns.map(col =>
    new TableCell({
      width: { size: col.width, type: WidthType.DXA },
      shading: { fill: "4A148C", type: ShadingType.CLEAR },
      margins: { top: 80, bottom: 80, left: 120, right: 120 },
      children: [new Paragraph({
        children: [new TextRun({
          text: col.header,
          bold: true,
          color: "FFFFFF",
          size: 18, // 9pt
          font: "Arial"
        })]
      })]
    })
  )
})
```

### Data Rows (Alternating)

Pattern: Row 1 (after header) = white (no fill), Row 2 = #FFF3E0, Row 3 = white, Row 4 = #FFF3E0, etc.

```javascript
new TableRow({
  children: columns.map((col, ci) =>
    new TableCell({
      width: { size: col.width, type: WidthType.DXA },
      shading: rowIndex % 2 === 1
        ? { fill: "FFF3E0", type: ShadingType.CLEAR }
        : undefined, // no shading = white
      margins: { top: 80, bottom: 80, left: 120, right: 120 },
      children: [new Paragraph({
        children: [new TextRun({
          text: cellValue,
          color: "212121",
          size: 18, // 9pt
          font: "Arial"
        })]
      })]
    })
  )
})
```

### Standard Table Column Widths

These are the column width patterns used across the document. Adapt proportionally based on actual number of columns, always summing to 9840 DXA.

**2-column tables** (e.g., Visão Geral):
- Property: 4000 DXA, Value: 5840 DXA

**3-column tables** (e.g., Arquitetura):
- Col 1: 2200, Col 2: 1600, Col 3: 6040

**4-column tables** (e.g., Colunas):
- Col 1: 2400, Col 2: 1200, Col 3: 1100, Col 4: 5140

**4-column tables** (e.g., Medidas DAX):
- Col 1: 1600, Col 2: 3500, Col 3: 1200, Col 4: 3540

**4-column tables** (e.g., Fontes de Dados):
- Col 1: 1600, Col 2: 2200, Col 3: 2200, Col 4: 3840

**4-column tables** (e.g., Páginas):
- Col 1: 1000, Col 2: 2000, Col 3: 1200, Col 4: 5640

**2-column tables** (e.g., Visuais Customizados):
- Col 1: 4000, Col 2: 5840

---

## 9. Notes & Blockquotes

### Note Text (e.g., "Nota: O caminho é absoluto...")

```javascript
new Paragraph({
  spacing: { after: 100 },
  children: [new TextRun({
    text: "Nota: ...",
    italics: true,
    color: "616161",
    size: 18, // 9pt
    font: "Arial"
  })]
})
```

### File Path Blockquote

File paths are shown as regular paragraphs with specific formatting. The path text uses the same italic gray style as notes.

---

## 10. Lists

Bullet lists use the following numbering configuration:

```javascript
numbering: {
  config: [{
    reference: "bullets",
    levels: [
      { level: 0, format: LevelFormat.BULLET, text: "●", alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 720, hanging: 360 } } } },
      { level: 1, format: LevelFormat.BULLET, text: "○", alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 1440, hanging: 360 } } } },
      { level: 2, format: LevelFormat.BULLET, text: "■", alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 2160, hanging: 360 } } } }
    ]
  }]
}
```

---

## 11. Section Catalog

Each section has a specific heading level and content type. Below is the complete catalog:

### H1 Sections (Heading 1)

| # | Title | Content |
|---|-------|---------|
| - | Sumário | Table of Contents |
| 1 | Visão Geral do Modelo | 2-col summary table |
| 2 | Arquitetura do Modelo (Star Schema) | Intro paragraph + 3-col table |
| 3+ | Tabela {name} ({type}) | Description paragraph + H2 subsections |
| N | Relacionamentos | 5-col relationships table + H2 Observações |
| N+1 | Catálogo de Medidas DAX | Intro paragraph + 4-col measures table |
| N+2 | Padrões DAX Utilizados | Intro paragraph + 3-col patterns table |
| N+3 | Convenções de Nomenclatura | Intro paragraph + 3-col conventions table |
| N+4 | Fontes de Dados | Intro paragraph + 4-col sources table + H2 Caminho |
| N+5 | Páginas do Relatório | Intro paragraph + 4-col pages table + H2s |

### H2 Sections (Heading 2)

| Parent | Title | Content |
|--------|-------|---------|
| Table sections | Colunas | 4-col columns table |
| Table sections | Fonte de Dados (Power Query) | Description paragraph |
| Table sections | Fonte de Dados | Description paragraph |
| Relacionamentos | Observações | Bullet list |
| Fontes de Dados | Caminho do Arquivo | Path blockquote + Note |
| Páginas do Relatório | Visuais Customizados | 2-col table |
| Páginas do Relatório | Bookmarks | Description paragraph |

---

## 12. Logo Asset

The SIA Inteligência logo is bundled at `assets/sia-logo.png`.

| Property | Value |
|----------|-------|
| File | sia-logo.png |
| Format | PNG, RGBA |
| Dimensions | 1546 × 869 pixels |
| Cover Header Size | 787400 × 442592 EMU |
| Content Header Size | 474476 × 266700 EMU |

To use in docx-js:

```javascript
const fs = require("fs");
const logoPath = "<skill-path>/assets/sia-logo.png";
const logoBuffer = fs.readFileSync(logoPath);

// Cover page header logo
new ImageRun({
  data: logoBuffer,
  transformation: { width: 787400/914400*72, height: 442592/914400*72 },
  // Or use EMU directly in floating/anchor mode
  type: "png"
})
```

Note: For headers, images need to be added as inline or anchor elements in the header XML. The docx-js Header object supports ImageRun children.
