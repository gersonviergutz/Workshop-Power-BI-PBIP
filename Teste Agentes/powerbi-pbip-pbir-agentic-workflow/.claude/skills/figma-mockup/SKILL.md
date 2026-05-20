---
name: figma-mockup
description: Traduz PRD + matriz de KPIs aprovados em mockup de dashboard no Figma. Use quando o agente figma-mockup-designer precisa gerar visual a partir de requisitos.
---

# Skill: Figma Mockup

## Quando usar
Após Gate 4b (medidas DAX aprovadas), antes da construção do relatório PBIR.

## Pré-condição
Figma MCP autenticado. Verificar:
1. Tentar `mcp__claude_ai_Figma__whoami` — se 200, ok.
2. Senão tentar `mcp__plugin_figma_figma__authenticate`.
3. Se ambos falharem, abortar a fase e instruir autenticação.

## Entrada
- `docs/levantamento-requisitos/requisitos-aprovados.md`
- `docs/dax/matriz-viabilidade.md`
- (opcional) `docs/apresentacoes-cliente/*.md` para narrativa

## Processo
1. Extrair lista de KPIs aprovados (status 🟢 e 🟡).
2. Agrupar KPIs em "páginas" lógicas (visão executiva, vendas detalhadas, geográfico, etc.).
3. Para cada página, definir layout em grid 12 colunas (cards de KPI no topo, gráfico principal central, tabela/lista lateral).
4. Chamar `mcp__claude_ai_Figma__create_new_file` ou `figma:figma-generate-design` com prompt detalhado por página.
5. Capturar URL do arquivo Figma criado.
6. Exportar thumbnails de cada página via `mcp__claude_ai_Figma__get_screenshot`.
7. Gravar `docs/mockups/figma-url.md` com URL, paleta e lista de páginas.
8. Gravar thumbnails em `docs/mockups/screenshots/`.

## Saída
- `docs/mockups/figma-url.md` (estrutura: URL + paleta + lista de páginas + KPIs por página)
- `docs/mockups/screenshots/*.png`

## Anti-padrões
- Gerar mockup com KPI que ainda não tem medida DAX aprovada.
- Usar paleta/tipografia divergente do design system do cliente (se houver `docs/padroes-design/`).
