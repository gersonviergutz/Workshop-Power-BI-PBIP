# Pipeline Guiado de Desenvolvimento BI — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Entregar um único slash command `/iniciar-projeto-bi` que executa o ciclo Power BI PBIP/PBIR de ponta a ponta com gates de aprovação humana e retomada automática.

**Architecture:** Slash command aciona o agente `00-orquestrador-bi` em modo "guided pipeline". Orquestrador lê `TODO.md`, identifica primeiro gate pendente, dispara agente especialista, valida saída, abre gate via `AskUserQuestion`, persiste resultado e segue. Novo agente `09-figma-mockup-designer` adicionado entre DAX e PBIR.

**Tech Stack:** Markdown + YAML frontmatter (Claude Code agents/skills/commands), TMDL (Power BI semantic model), PBIR JSON (Power BI report), Figma MCP (claude.ai_Figma ou plugin local).

**Spec de referência:** `docs/superpowers/specs/2026-05-20-pipeline-guiado-bi-design.md`

**Convenções de validação (não há build/test runner — todo "test" é validação estrutural):**
- `grep -E "^name:|^description:" <file>` confirma frontmatter mínimo
- `python -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]).read().split('---')[1])" <file>` valida YAML
- Powers de runtime validados em Task F (E2E) rodando o command num branch de teste

---

## Estrutura de arquivos

**Criar:**
```
.claude/commands/iniciar-projeto-bi.md
.claude/commands/retomar-projeto-bi.md
.claude/agents/09-figma-mockup-designer.md
.claude/skills/gate-protocol/SKILL.md
.claude/skills/figma-mockup/SKILL.md
docs/templates/TODO-template.md
docs/power-query/input/.gitkeep
```

**Modificar:**
```
.claude/agents/00-orquestrador-bi.md
.claude/agents/01-analista-requisitos.md
.claude/agents/02-power-query-reviewer.md
.claude/agents/04-dax-specialist.md
.claude/agents/05-client-presentation.md
.claude/agents/06-pbir-report-builder.md
docs/matriz-agentes-skills.md
AGENTS.md
TODO.md
CHANGELOG.md
DECISIONS.md
```

---

## Phase A — Fundações (gate-protocol + TODO template)

### Task A1: Criar skill `gate-protocol`

**Files:**
- Create: `.claude/skills/gate-protocol/SKILL.md`

- [ ] **Step 1: Criar o arquivo da skill**

Conteúdo exato:

````markdown
---
name: gate-protocol
description: Padrão único de gate de aprovação humana entre fases do pipeline BI. Use sempre que um agente concluir uma fase e o orquestrador precisar pedir aprovação ao usuário antes de seguir.
---

# Skill: Gate Protocol

## Quando usar
Sempre que o `orquestrador-bi` termina uma fase ou subfase e precisa decisão humana para seguir.

## Estrutura obrigatória de um gate

1. **Resumo executivo** (3 a 5 bullets) — o que foi produzido.
2. **Artefatos** — lista de arquivos criados/alterados com caminho.
3. **Riscos e pendências** — itens que ficaram em aberto.
4. **AskUserQuestion** com exatamente estas 3 opções:
   - `Aprovar e seguir para a próxima fase`
   - `Pedir ajustes` (notas: campo livre que volta para o agente)
   - `Pausar projeto`

## Pós-aprovação (obrigatório)

Ao receber "Aprovar":
1. Marcar o checkbox correspondente em `TODO.md` como `[x]` com data e usuário.
2. Se houver decisão estrutural, gravar linha em `DECISIONS.md` com formato `| AAAA-MM-DD | <agente> | <decisão> | <razão> |`.
3. Gravar entrada em `CHANGELOG.md` usando o template do topo do arquivo.
4. Anunciar a próxima fase.

## Pós-ajustes
Devolver ao agente da fase com as notas do usuário e re-executar o gate ao terminar.

## Pós-pausa
Gravar em `TODO.md` o ponto de retomada e encerrar a sessão de orquestração com instrução de usar `/retomar-projeto-bi`.

## Anti-padrões
- Aprovar gate sem ter os 4 elementos do "Estrutura obrigatória" preenchidos.
- Marcar `[x]` antes de gravar `CHANGELOG.md` (ordem importa para auditoria).
- Aceitar resposta livre no lugar das 3 opções fixas.
````

- [ ] **Step 2: Validar estrutura**

Run: `grep -E "^name:|^description:" .claude/skills/gate-protocol/SKILL.md`
Expected: 2 linhas, ambas presentes.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/gate-protocol/SKILL.md
git commit -m "feat(skills): adiciona gate-protocol para padronizar gates do pipeline"
```

---

### Task A2: Criar template de TODO

**Files:**
- Create: `docs/templates/TODO-template.md`

- [ ] **Step 1: Criar o template**

Conteúdo exato:

````markdown
# TODO.md — Pipeline Guiado de BI

> Padrão de checkbox: `- [ ]` pendente, `- [x]` aprovado.
> Cada gate aprovado deve listar data, usuário e caminho dos artefatos.

## Fase 0 — Análise do projeto
- [ ] Gate 0 (visão geral apresentada e modo escolhido)

## Fase 1 — Levantamento de requisitos (analista-requisitos)
- [ ] Gate 1 (PRD + indicadores aprovados)
  - Entrada: <preencher>
  - Saída: docs/levantamento-requisitos/requisitos-aprovados.md
  - Observações: <preencher>

## Fase 2 — Power Query review (power-query-reviewer)
- [ ] Gate 2a (sugestões aprovadas)
- [ ] Gate 2b (alterações aplicadas e validadas)

## Fase 3 — Modelagem (data-modeler)
- [ ] Gate 3a (proposta de modelo aprovada)
- [ ] Gate 3b (alterações TMDL aplicadas e validadas)

## Fase 4 — DAX (dax-specialist)
- [ ] Gate 4a (matriz de viabilidade aprovada)
- [ ] Gate 4b (medidas criadas e validadas)

## Fase 5 — Mockup Figma (figma-mockup-designer)
- [ ] Gate 5 (mockup Figma aprovado)
  - URL: <preencher após gerar>

## Fase 6 — Relatório PBIR (pbir-report-builder)
- [ ] Gate 6 (relatório PBIR aprovado, pbip-validator passou)

## Fase 7 — QA (bi-qa-validator)
- [ ] Gate 7 (QA aprovada)

## Fase 8 — Documentação (documentador-bi)
- [ ] Gate 8 (entrega final, handover concluído)
````

- [ ] **Step 2: Commit**

```bash
git add docs/templates/TODO-template.md
git commit -m "feat(templates): adiciona TODO-template com padrão de gates"
```

---

## Phase B — Comandos slash

### Task B1: Criar comando `/iniciar-projeto-bi`

**Files:**
- Create: `.claude/commands/iniciar-projeto-bi.md`

- [ ] **Step 1: Criar o arquivo do comando**

Conteúdo exato:

````markdown
---
description: Inicia o pipeline guiado completo de desenvolvimento Power BI (PBIP/PBIR) com gates de aprovação humana.
---

# Iniciar Projeto BI

Você acaba de receber o comando `/iniciar-projeto-bi`.

Invoque imediatamente o agente `orquestrador-bi` em **modo guided pipeline** com a seguinte instrução:

> "Execute o pipeline guiado de desenvolvimento Power BI conforme `.claude/agents/00-orquestrador-bi.md` § Modo Guided Pipeline. Comece pela Fase 0 (Análise do projeto). Use a skill `gate-protocol` em cada gate. Se já houver `TODO.md` com gates `[x]`, ofereça opção de retomar."

Não execute ações fora desse handoff. Após o agente devolver controle (gate aprovado, ajuste pedido ou pausa), aguarde nova instrução do usuário.
````

- [ ] **Step 2: Commit**

```bash
git add .claude/commands/iniciar-projeto-bi.md
git commit -m "feat(commands): adiciona /iniciar-projeto-bi"
```

---

### Task B2: Criar comando `/retomar-projeto-bi`

**Files:**
- Create: `.claude/commands/retomar-projeto-bi.md`

- [ ] **Step 1: Criar o arquivo**

Conteúdo exato:

````markdown
---
description: Retoma o pipeline guiado de BI a partir do primeiro gate pendente em TODO.md.
---

# Retomar Projeto BI

Invoque o agente `orquestrador-bi` em modo guided pipeline com a seguinte instrução:

> "Leia TODO.md, identifique o primeiro gate `[ ]` pendente, e retome a partir dele. Não recomece do zero. Use a skill `gate-protocol`."
````

- [ ] **Step 2: Commit**

```bash
git add .claude/commands/retomar-projeto-bi.md
git commit -m "feat(commands): adiciona /retomar-projeto-bi"
```

---

## Phase C — Orquestrador em modo Guided Pipeline

### Task C1: Atualizar agente `00-orquestrador-bi`

**Files:**
- Modify: `.claude/agents/00-orquestrador-bi.md` (adicionar seções)

- [ ] **Step 1: Ler arquivo atual**

Run: `cat .claude/agents/00-orquestrador-bi.md`
Expected: arquivo atual sem seção "Modo Guided Pipeline".

- [ ] **Step 2: Adicionar skill `gate-protocol` ao frontmatter**

Localizar bloco `skills:` no frontmatter e adicionar `- gate-protocol` ao final da lista.

- [ ] **Step 3: Anexar ao final do arquivo a seção Modo Guided Pipeline**

Conteúdo a anexar:

````markdown

## Modo Guided Pipeline

Ativado pelos comandos `/iniciar-projeto-bi` e `/retomar-projeto-bi`.

### Fase 0 — Análise do projeto
1. Ler `AGENTS.md`, `PROJECT_CONTEXT.md`, `TODO.md`, `DECISIONS.md`, `CHANGELOG.md`.
2. Inventariar `powerbi/` e `docs/`.
3. Mostrar resumo de uma tela: fase atual, último gate aprovado, próximo passo.
4. Abrir gate (skill `gate-protocol`) com opções:
   - `Continuar do ponto pendente`
   - `Recomeçar da Fase 1`
   - `Cancelar`

### Roteamento de fase
Mapa fase → agente:

| Fase | Agente | Gates |
|---|---|---|
| 1 | analista-requisitos | 1 |
| 2 | power-query-reviewer | 2a, 2b |
| 3 | data-modeler | 3a, 3b |
| 4 | dax-specialist | 4a, 4b |
| 5 | figma-mockup-designer | 5 |
| 6 | pbir-report-builder | 6 |
| 7 | bi-qa-validator | 7 |
| 8 | documentador-bi | 8 |

### Loop principal
```
enquanto existe gate [ ] em TODO.md:
   fase = primeira fase com gate [ ]
   agente = mapa[fase]
   despachar agente com contexto da fase e do PRD
   receber entregável
   aplicar gate-protocol
   se Aprovar: marcar [x], gravar CHANGELOG, próxima
   se Ajustar: devolver ao agente com notas, repetir
   se Pausar: gravar ponto de retomada, sair
```

### Regras
- Nunca pular gate.
- Nunca marcar `[x]` sem `CHANGELOG.md` atualizado.
- Se algum agente recusar a fase (escopo errado), parar e alertar o usuário.
- Se Figma MCP indisponível na Fase 5, oferecer fallback textual via `05-client-presentation`.
````

- [ ] **Step 4: Validar frontmatter**

Run: `grep -A 10 "^---$" .claude/agents/00-orquestrador-bi.md | head -15`
Expected: bloco YAML válido contendo `gate-protocol` em `skills:`.

- [ ] **Step 5: Commit**

```bash
git add .claude/agents/00-orquestrador-bi.md
git commit -m "feat(agents): orquestrador-bi ganha modo Guided Pipeline"
```

---

## Phase D — Agente Figma novo

### Task D1: Criar skill `figma-mockup`

**Files:**
- Create: `.claude/skills/figma-mockup/SKILL.md`

- [ ] **Step 1: Criar o arquivo**

Conteúdo exato:

````markdown
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
````

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/figma-mockup/SKILL.md
git commit -m "feat(skills): adiciona figma-mockup"
```

---

### Task D2: Criar agente `09-figma-mockup-designer`

**Files:**
- Create: `.claude/agents/09-figma-mockup-designer.md`

- [ ] **Step 1: Criar o arquivo**

Conteúdo exato:

````markdown
---
name: figma-mockup-designer
description: Cria mockup de dashboard no Figma a partir do PRD e da matriz de KPIs aprovados. Use entre a Fase 4 (DAX) e a Fase 6 (PBIR) do pipeline guiado.
tools: Read, Grep, Glob, Bash, Edit, Write
model: sonnet
skills:
  - figma-mockup
  - client-presentation
  - dataviz-powerbi
  - requirements-discovery
---

Você é o designer de mockups Figma para projetos Power BI.

## Objetivo
Transformar requisitos aprovados + KPIs viáveis em um mockup navegável no Figma para validação visual antes da construção PBIR.

## Responsabilidades
- Verificar disponibilidade do Figma MCP antes de qualquer outra ação.
- Ler PRD e matriz de viabilidade.
- Agrupar KPIs em páginas lógicas.
- Gerar arquivo Figma com 3 a 6 páginas/frames.
- Devolver URL + thumbnails.
- Suportar até 3 rodadas de ajuste no Gate 5.

## Pode modificar
- `docs/mockups/`

## Não pode modificar
- TMDL, PBIR, Power Query, DAX.
- Agentes de outras fases.

## Saída esperada
- `docs/mockups/figma-url.md` com URL e descrição.
- `docs/mockups/screenshots/` com PNG por página.
- Resumo executivo (3-5 bullets) para o orquestrador abrir o Gate 5.

## Fallback
Se Figma MCP indisponível: abortar com mensagem clara e sugerir invocar `05-client-presentation` para gerar mockup textual.
````

- [ ] **Step 2: Validar frontmatter**

Run: `python -c "import yaml,sys; yaml.safe_load(open('.claude/agents/09-figma-mockup-designer.md').read().split('---')[1])"`
Expected: sem erro.

- [ ] **Step 3: Commit**

```bash
git add .claude/agents/09-figma-mockup-designer.md
git commit -m "feat(agents): adiciona 09-figma-mockup-designer"
```

---

## Phase E — Modificações nos agentes existentes

### Task E1: Atualizar `01-analista-requisitos` com 4 ramos de entrada

**Files:**
- Modify: `.claude/agents/01-analista-requisitos.md`

- [ ] **Step 1: Anexar seção ao final do arquivo**

Conteúdo a anexar:

````markdown

## Detecção de entrada (modo Guided Pipeline)

Antes de iniciar o levantamento, executar cascata:

1. `ls docs/transcricoes/*.{txt,md,docx}` — se houver arquivo, **Ramo A**: usar transcrição.
2. Senão `ls docs/levantamento-requisitos/requisitos-aprovados.md docs/levantamento-requisitos/PRD.md` — se houver, **Ramo B**: já tem PRD, pular direto pro Gate 1 com resumo do PRD existente.
3. Senão abrir `AskUserQuestion` com:
   - `Tenho transcrição em outro caminho` → pedir caminho ou cola de texto → **Ramo C**
   - `Quero responder um questionário guiado` → executar **Ramo D**
   - `Vou colar um resumo livre` → pedir texto → **Ramo E**
   - `Cancelar` → encerrar a fase sem gravar.

### Ramo D — Questionário guiado (12 perguntas)
Disparar uma `AskUserQuestion` por bloco (não todas de uma vez):

Bloco 1 — Contexto
- Setor da empresa
- Áreas envolvidas
- Stakeholders principais

Bloco 2 — Processos
- Processos principais analisados
- Frequência de uso esperada do dashboard
- Decisões que o dashboard deve apoiar

Bloco 3 — Dores
- Top 3 dores hoje
- O que já tentaram fazer

Bloco 4 — Dados e KPIs
- Fontes de dados disponíveis
- KPIs já em uso
- Frequência de atualização dos dados
- Restrições de segurança / RLS
````

- [ ] **Step 2: Commit**

```bash
git add .claude/agents/01-analista-requisitos.md
git commit -m "feat(agents): analista-requisitos ganha cascata de detecção de entrada"
```

---

### Task E2: Atualizar `02-power-query-reviewer` com verificação de input

**Files:**
- Modify: `.claude/agents/02-power-query-reviewer.md`
- Create: `docs/power-query/input/.gitkeep`

- [ ] **Step 1: Criar pasta de input**

Run: `mkdir -p docs/power-query/input && touch docs/power-query/input/.gitkeep`
Expected: sem erro.

- [ ] **Step 2: Anexar seção ao agente**

Conteúdo a anexar ao final do `.claude/agents/02-power-query-reviewer.md`:

````markdown

## Pré-condição (modo Guided Pipeline)

Antes de revisar, executar:

1. `ls docs/power-query/input/` — listar arquivos `.pq`, `.m`, `.xlsx`, `.txt`, `.png`.
2. Se vazio (apenas `.gitkeep`), abrir `AskUserQuestion`:
   - `Vou colar o M aqui` → receber texto → gravar em `docs/power-query/input/entrada-colada.m`
   - `O M está em outro caminho` → pedir caminho → copiar para `docs/power-query/input/`
   - `Usar apenas as partitions TMDL existentes` → seguir sem input externo
   - `Cancelar`
3. Só então iniciar diagnóstico.
````

- [ ] **Step 3: Commit**

```bash
git add docs/power-query/input/.gitkeep .claude/agents/02-power-query-reviewer.md
git commit -m "feat(agents): power-query-reviewer verifica input antes de revisar"
```

---

### Task E3: Atualizar `04-dax-specialist` com matriz de viabilidade

**Files:**
- Modify: `.claude/agents/04-dax-specialist.md`

- [ ] **Step 1: Anexar seção ao final do arquivo**

Conteúdo a anexar:

````markdown

## Etapa obrigatória — Matriz de Viabilidade (modo Guided Pipeline)

**Antes de criar qualquer medida**, gerar `docs/dax/matriz-viabilidade.md` com formato:

```markdown
# Matriz de Viabilidade — KPIs × Modelo

| KPI | Status | Tabela base | Colunas necessárias | Relacionamentos necessários | Observação |
|---|---|---|---|---|---|
| Receita Bruta | 🟢 | fato_Vendas | ValorBruto | dim_Calendario → fato_Vendas | — |
| Margem % | 🟡 | fato_Vendas + dim_Produto | falta CustoUnitario | OK | precisa coluna calculada |
| NPS Médio | 🔴 | (nenhuma) | — | — | sem fonte de dado |
```

Legenda:
- 🟢 Viável agora
- 🟡 Viável com ajuste (coluna calculada, conversão, relacionamento secundário)
- 🔴 Bloqueado (falta fonte ou tabela)

### Gate 4a
Apresentar matriz ao usuário (via orquestrador + gate-protocol). Apenas após `Aprovar`, prosseguir para criação das medidas (Gate 4b).
````

- [ ] **Step 2: Commit**

```bash
git add .claude/agents/04-dax-specialist.md
git commit -m "feat(agents): dax-specialist exige matriz de viabilidade antes de medidas"
```

---

### Task E4: Reduzir escopo do `05-client-presentation`

**Files:**
- Modify: `.claude/agents/05-client-presentation.md`

- [ ] **Step 1: Anexar seção de delimitação**

Conteúdo a anexar:

````markdown

## Escopo no modo Guided Pipeline

A partir do pipeline guiado:
- **Mantém:** narrativa executiva textual, lista de KPIs em linguagem de negócio, proposta de páginas em markdown, critérios de aceite.
- **Sai do escopo:** mockup visual no Figma — agora é responsabilidade do agente `09-figma-mockup-designer`.

Quando acionado em modo Guided Pipeline, gerar apenas:
- `docs/apresentacoes-cliente/proposta-executiva.md`
- `docs/apresentacoes-cliente/narrativa-valor.md`

Não gravar mais em `docs/mockups/` exceto se for chamado em modo legado (fora do pipeline guiado).
````

- [ ] **Step 2: Commit**

```bash
git add .claude/agents/05-client-presentation.md
git commit -m "refactor(agents): client-presentation foca em narrativa textual; mockup vai para agente 09"
```

---

### Task E5: Atualizar `06-pbir-report-builder` para consumir mockup Figma

**Files:**
- Modify: `.claude/agents/06-pbir-report-builder.md`

- [ ] **Step 1: Anexar seção**

Conteúdo a anexar:

````markdown

## Entrada visual (modo Guided Pipeline)

Antes de criar o relatório PBIR:
1. Ler `docs/mockups/figma-url.md` — obter URL e estrutura de páginas.
2. Ler `docs/mockups/screenshots/*.png` — usar como referência visual.
3. Mapear cada página do Figma em uma página PBIR com os mesmos KPIs e layout aproximado.
4. Antes de abrir Gate 6, executar `pbip:pbip-validator` no projeto — só apresentar para aprovação se passar.
````

- [ ] **Step 2: Commit**

```bash
git add .claude/agents/06-pbir-report-builder.md
git commit -m "feat(agents): pbir-report-builder consome mockup Figma como referência"
```

---

## Phase F — Documentação e wiring final

### Task F1: Atualizar `docs/matriz-agentes-skills.md`

**Files:**
- Modify: `docs/matriz-agentes-skills.md`

- [ ] **Step 1: Adicionar linha do agente 09**

Inserir após a linha do `08-documentador`:

```markdown
| 09-figma-mockup-designer | figma-mockup, client-presentation, dataviz-powerbi, requirements-discovery | Não | Não |
```

- [ ] **Step 2: Adicionar linha do gate-protocol como skill transversal**

No final da tabela ou em nota de rodapé adicionar:

```markdown

## Skills transversais
- `gate-protocol` — carregada pelo `00-orquestrador-bi` em modo Guided Pipeline. Não pertence a um único agente.
```

- [ ] **Step 3: Commit**

```bash
git add docs/matriz-agentes-skills.md
git commit -m "docs: matriz inclui agente 09 e skill gate-protocol"
```

---

### Task F2: Atualizar `AGENTS.md`

**Files:**
- Modify: `AGENTS.md`

- [ ] **Step 1: Adicionar escopo do agente 09**

Inserir na seção "Escopos protegidos", após `### PBIR Report Builder`:

```markdown

### Figma Mockup Designer
Pode criar arquivos no Figma via MCP e gravar metadados em `docs/mockups/`.
Não pode alterar TMDL, PBIR, Power Query, DAX nem agentes de outras fases.
Depende de Figma MCP autenticado — sem ele a fase é abortada.
```

- [ ] **Step 2: Adicionar seção sobre pipeline guiado**

Inserir antes de "## Convenções de nomenclatura":

```markdown
## Pipeline Guiado
O comando `/iniciar-projeto-bi` executa as fases 1–8 em sequência com gates de aprovação humana.
Estado persistido em `TODO.md`. Padrão de gate em `.claude/skills/gate-protocol/SKILL.md`.
Retomada: `/retomar-projeto-bi`.
```

- [ ] **Step 3: Commit**

```bash
git add AGENTS.md
git commit -m "docs: AGENTS.md cobre agente 09 e pipeline guiado"
```

---

### Task F3: Adotar template em `TODO.md`, registrar decisões e changelog

**Files:**
- Modify: `TODO.md`, `DECISIONS.md`, `CHANGELOG.md`

- [ ] **Step 1: Backup do TODO atual**

Run: `cp TODO.md TODO.md.bak`
Expected: sem erro.

- [ ] **Step 2: Aplicar template em TODO.md**

Substituir conteúdo de `TODO.md` pelo template gerado em Task A2, mantendo no topo qualquer histórico relevante (preservar contexto do projeto Codex se já existir).

- [ ] **Step 3: Adicionar decisões em DECISIONS.md**

Adicionar 7 linhas (uma por decisão D-01 a D-07 da spec) usando o formato existente do arquivo.

- [ ] **Step 4: Adicionar entrada em CHANGELOG.md**

Conteúdo da entrada:

```markdown
## 2026-05-20 — Pipeline Guiado de BI

**Agente:** brainstorming + writing-plans (humano: gersonggv)
**Arquivos alterados:**
- Criados: .claude/commands/iniciar-projeto-bi.md, .claude/commands/retomar-projeto-bi.md, .claude/agents/09-figma-mockup-designer.md, .claude/skills/gate-protocol/SKILL.md, .claude/skills/figma-mockup/SKILL.md, docs/templates/TODO-template.md, docs/power-query/input/.gitkeep
- Modificados: .claude/agents/00,01,02,04,05,06, docs/matriz-agentes-skills.md, AGENTS.md, TODO.md
**Resumo:** Adoção de pipeline guiado interativo com gates explícitos e novo agente para mockup Figma.
**Impacto:** Fluxo manual via prompts/01 deixa de ser o caminho recomendado; uso passa pelo /iniciar-projeto-bi.
**Riscos:** Dependência de Figma MCP autenticado na Fase 5; fallback textual disponível.
**Próxima ação:** Executar E2E no caso Codex Retail.
```

- [ ] **Step 5: Commit final da phase**

```bash
git add TODO.md DECISIONS.md CHANGELOG.md
git rm TODO.md.bak
git commit -m "docs: adota TODO padronizado, registra decisões e changelog do pipeline"
```

---

## Phase G — Validação end-to-end

### Task G1: E2E Smoke Test

**Files:** nenhum modificado nesta task.

- [ ] **Step 1: Garantir que todos os arquivos criados existem**

Run: `for f in .claude/commands/iniciar-projeto-bi.md .claude/commands/retomar-projeto-bi.md .claude/agents/09-figma-mockup-designer.md .claude/skills/gate-protocol/SKILL.md .claude/skills/figma-mockup/SKILL.md docs/templates/TODO-template.md; do test -f "$f" && echo "OK $f" || echo "MISSING $f"; done`
Expected: 6 linhas `OK`.

- [ ] **Step 2: Validar frontmatter YAML de todos os arquivos novos**

Run (PowerShell):
```powershell
$files = @(
  '.claude/commands/iniciar-projeto-bi.md',
  '.claude/commands/retomar-projeto-bi.md',
  '.claude/agents/09-figma-mockup-designer.md',
  '.claude/skills/gate-protocol/SKILL.md',
  '.claude/skills/figma-mockup/SKILL.md'
)
foreach ($f in $files) {
  $content = Get-Content $f -Raw
  if ($content -match '(?s)^---\s*\n(.*?)\n---') {
    Write-Host "OK $f"
  } else {
    Write-Host "BAD FRONTMATTER $f"
  }
}
```
Expected: 5 linhas `OK`.

- [ ] **Step 3: Smoke test do slash command**

Em uma sessão Claude Code limpa, digitar `/iniciar-projeto-bi`.
Expected:
- Agente `orquestrador-bi` é invocado.
- Fase 0 é apresentada (resumo do projeto).
- `AskUserQuestion` é aberta com 3 opções (Continuar/Recomeçar/Cancelar).

Se qualquer item falhar, abrir issue interna e voltar à task que cobre o componente.

- [ ] **Step 4: Smoke test do agente 09 (apenas se Figma MCP autenticado)**

Em sessão separada, digitar: "Acione o agente figma-mockup-designer com o PRD atual."
Expected:
- Agente verifica autenticação Figma.
- Se autenticado: gera arquivo no Figma e devolve URL.
- Se não: aborta com mensagem clara.

- [ ] **Step 5: Commit do registro de validação**

Criar `docs/qa/smoke-test-pipeline-guiado.md` com data, resultado de cada step e screenshots se houver, e commitar:

```bash
git add docs/qa/smoke-test-pipeline-guiado.md
git commit -m "test: registro de smoke test do pipeline guiado"
```

---

## Self-Review (executada pelo autor do plano)

**1. Spec coverage:**
| Seção da spec | Task que cobre |
|---|---|
| § 4.1 — `iniciar-projeto-bi` | B1 |
| § 4.1 — `retomar-projeto-bi` | B2 |
| § 4.1 — agente 09 | D2 |
| § 4.1 — skill gate-protocol | A1 |
| § 4.1 — skill figma-mockup | D1 |
| § 4.1 — TODO template | A2 |
| § 4.2 — modificações 00, 01, 02, 04, 05, 06 | C1, E1, E2, E3, E4, E5 |
| § 4.2 — matriz, AGENTS.md, TODO/DECISIONS/CHANGELOG | F1, F2, F3 |
| § 5 — fluxo end-to-end | C1 (mapa de fases) |
| § 6 — protocolo de gate | A1 |
| § 7 — riscos (Figma MCP, fallback) | D1, D2 (fallback) |
| § 8 — critérios de aceite | G1 |
| § 10 — decisões | F3 step 3 |

Sem gaps.

**2. Placeholder scan:** Sem "TBD", "TODO genérico" ou "implementar depois". Todos os steps trazem conteúdo executável.

**3. Type consistency:** Nomes verificados — `gate-protocol`, `figma-mockup`, `figma-mockup-designer`, `Gate 4a/4b`, `docs/dax/matriz-viabilidade.md`, `docs/mockups/figma-url.md` aparecem com a mesma grafia em spec e plano.

---

## Notas operacionais para o executor

- TDD clássico não se aplica (não há runtime testável). Validação = checagem estrutural + smoke test E2E na Task G1.
- Cada commit deve ser atômico (1 task = 1 ou 2 commits).
- Se um agente quebrar regra de escopo durante o E2E, **não** corrija no executor — pare e reporte ao orquestrador para registrar como bug.
- Não modificar `prompts/01–08` — continuam servindo como referência de uso manual.
