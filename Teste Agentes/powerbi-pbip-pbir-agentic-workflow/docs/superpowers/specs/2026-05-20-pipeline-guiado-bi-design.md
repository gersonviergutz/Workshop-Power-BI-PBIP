# Spec — Pipeline Guiado de Desenvolvimento BI

**Data:** 2026-05-20
**Autor:** Brainstorming session (gersonggv + Claude)
**Status:** Aguardando revisão humana

---

## 1. Objetivo

Substituir o fluxo manual atual (ler `prompts/01-…` e acionar agentes na mão) por um **único slash command** `/iniciar-projeto-bi` que executa o ciclo completo de desenvolvimento Power BI PBIP/PBIR de forma **interativa**, pausando em cada gate para aprovação humana e retomando do ponto correto se interrompido.

O comando preserva 100 % das regras de escopo dos agentes existentes (`AGENTS.md`, `docs/matriz-agentes-skills.md`) — ele apenas orquestra a sequência, não cruza limites.

---

## 2. Não-objetivos

- Não automatiza aprovações humanas — todo gate exige resposta explícita.
- Não substitui os prompts em `prompts/` — eles continuam válidos como ponto de entrada manual para fases isoladas.
- Não altera as skills existentes — apenas as consome.
- Não conecta automaticamente ao Power BI Desktop para validar refresh — refresh continua como pendência humana.
- Não cria projeto Figma do zero quando o cliente já tem design system — apenas o caso "começar do zero" é coberto na v1.

---

## 3. Diagnóstico do estado atual

### 3.1 Inventário

| Camada | Existe | Lacuna |
|---|---|---|
| Agentes (`.claude/agents/`) | 00–08 (9 agentes) | Sem agente de Figma |
| Skills (`.claude/skills/`) | 12 skills cobrindo todas as camadas | OK |
| Skills Figma (globais) | `figma:figma-generate-design`, `figma:figma-use`, `figma:figma-create-new-file` disponíveis | Não consumidas por nenhum agente |
| Prompts (`prompts/`) | 8 prompts texto (01–08) | São roteiros, não slash commands |
| `.claude/commands/` | **Não existe** | Comando precisa ser criado aqui |
| Estado/gates | `TODO.md` lista fases mas sem padrão de checkbox | Falta padrão estrito |
| Detecção de entrada (Fase 1) | Manual — agente lê `docs/transcricoes/` ou nada | Falta fallback (questionário/resumo) |
| Validação DAX × modelo | Implícita no agente DAX | Sem matriz formal de viabilidade pré-criação |
| Mockup visual | `05-client-presentation` faz mockup **textual** | Sem geração no Figma |

### 3.2 Pontos fortes a preservar

- Escopos de agente são rígidos e bem documentados em `AGENTS.md` § "Escopos protegidos"
- Convenções de nomenclatura (`fato_`, `dim_`, PascalCase sem acentos) já consolidadas
- Matriz de rastreabilidade `docs/levantamento-requisitos/matriz-rastreabilidade.md` já é o padrão de auditoria

---

## 4. Arquitetura proposta

### 4.1 Componentes novos

| Artefato | Tipo | Caminho | Função |
|---|---|---|---|
| `iniciar-projeto-bi` | Slash command | `.claude/commands/iniciar-projeto-bi.md` | Ponto de entrada único |
| `retomar-projeto-bi` | Slash command | `.claude/commands/retomar-projeto-bi.md` | Atalho para continuar do gate pendente |
| `09-figma-mockup-designer` | Agent | `.claude/agents/09-figma-mockup-designer.md` | Gera mockup no Figma a partir do PRD + KPIs |
| `gate-protocol` | Skill | `.claude/skills/gate-protocol/SKILL.md` | Padrão único de gate (entregável → resumo → AskUserQuestion → registra) |
| `figma-mockup` | Skill | `.claude/skills/figma-mockup/SKILL.md` | Como traduzir PRD em frames Figma |
| Template de TODO | Markdown | `docs/templates/TODO-template.md` | Estrutura de checkbox de gate |

### 4.2 Componentes modificados

| Artefato | Mudança |
|---|---|
| `.claude/agents/00-orquestrador-bi.md` | Adiciona modo "guided pipeline" com lógica de detecção/retomada |
| `.claude/agents/01-analista-requisitos.md` | Adiciona ramos: transcrição / PRD existente / questionário guiado / resumo livre |
| `.claude/agents/02-power-query-reviewer.md` | Adiciona checagem de `docs/power-query/input/` antes de revisar |
| `.claude/agents/04-dax-specialist.md` | Adiciona etapa explícita de matriz "KPI × Viabilidade" antes de criar medidas |
| `.claude/agents/05-client-presentation.md` | Reduz escopo a "narrativa executiva textual"; mockup visual sai para o agente 09 |
| `.claude/agents/06-pbir-report-builder.md` | Lê o arquivo de saída do Figma como referência visual |
| `docs/matriz-agentes-skills.md` | Adiciona linha do agente 09 |
| `AGENTS.md` | Adiciona § "Escopo protegido — Figma Mockup Designer" |

### 4.3 Modelo de estado

`TODO.md` é a fonte da verdade. Padrão estrito:

```markdown
## Fase 1 — Levantamento de requisitos
- [x] Gate 1 aprovado em 2026-05-20 por gersonggv
  - Entrada: docs/transcricoes/Transcrição da Reunião.txt
  - Saída: docs/levantamento-requisitos/requisitos-aprovados.md
  - Observações: …
## Fase 2 — Power Query review
- [ ] Gate 2a (aprovar sugestões) — pendente
- [ ] Gate 2b (aprovar alterações aplicadas) — pendente
```

O orquestrador identifica o **primeiro gate `[ ]`** e retoma dali.

---

## 5. Fluxo detalhado (end-to-end)

### Fase 0 — Análise do projeto

```
1. Lê AGENTS.md, PROJECT_CONTEXT.md, TODO.md, DECISIONS.md, CHANGELOG.md
2. Inventaria: powerbi/, docs/, agentes carregados
3. Apresenta resumo de uma tela: fase atual, último gate aprovado, próximo passo
4. AskUserQuestion: [Continuar do ponto pendente | Recomeçar da Fase 1 | Cancelar]
```

### Fase 1 — Requisitos (analista-requisitos)

**Detecção de entrada (cascata):**

```
SE existe arquivo em docs/transcricoes/*.{txt,md,docx}
   → opção A: usar transcrição
SENÃO SE existe docs/levantamento-requisitos/requisitos-aprovados.md ou PRD.md
   → opção B: usar PRD existente (pula direto pro Gate 1)
SENÃO
   → AskUserQuestion:
       1. "Tenho transcrição — onde está?" (pede caminho ou cola texto)
       2. "Quero responder um questionário guiado" (12 perguntas: setor, processos, dores, fontes, decisões, frequência, KPIs já usados, sazonalidade, stakeholders, frequência de uso, dispositivos, segurança)
       3. "Vou colar um resumo livre" (textarea)
       4. "Cancelar"
```

Saída obrigatória: `docs/levantamento-requisitos/requisitos-aprovados.md` + `matriz-rastreabilidade.md` (com dor → pergunta → KPI mapeados).

**Gate 1:** orquestrador mostra (a) número de dores, (b) número de KPIs propostos, (c) pendências para cliente, e pergunta:

> `[Aprovar Gate 1 | Pedir ajustes (descrever) | Pausar projeto]`

### Fase 2 — Power Query review (power-query-reviewer)

```
Pré-condição: docs/power-query/input/ contém arquivo .pq, .m, .xlsx ou screenshot
SE vazio → AskUserQuestion:
   "Onde está a fonte? [colar M aqui | apontar caminho | usar o que já está em TMDL partitions]"
```

**Gate 2a** — aprovar sugestões (relatório `docs/power-query/diagnostico.md`)
**Gate 2b** — aprovar alterações aplicadas nas partitions TMDL (`Bash diff` mostrado antes do commit)

### Fase 3 — Modelagem (data-modeler)

**Gate 3a** — aprovar proposta de modelo (relatório com tabelas, relacionamentos, cardinalidade, direção de filtro)
**Gate 3b** — aprovar alterações TMDL aplicadas

Pendência conhecida registrada como aviso: refresh em Power BI Desktop fica como tarefa humana.

### Fase 4 — DAX (dax-specialist) — **com etapa nova**

```
Passo 4.1: lê PRD + KPIs, lê model.tmdl + relationships.tmdl + tables/*.tmdl
Passo 4.2: para cada KPI propõe classificação:
   🟢 Viável agora — colunas e relacionamentos existem
   🟡 Viável com ajuste — falta coluna calculada / formato / relacionamento secundário
   🔴 Bloqueado — falta tabela ou fonte de dado
Passo 4.3: gera docs/dax/matriz-viabilidade.md
```

**Gate 4a** — aprovar matriz de viabilidade (decide o que fica fora do escopo desta release)
**Gate 4b** — aprovar medidas criadas em TMDL (`tables/*.tmdl` — measures organizadas em display folders)

### Fase 5 — Mockup Figma (**novo** — figma-mockup-designer)

```
Pré-condição: Figma MCP autenticado
   - Tenta mcp__claude_ai_Figma__whoami (servidor claude.ai) OU
     mcp__plugin_figma_figma__authenticate (plugin local)
   - SE ambos falharem → exibe instruções de autenticação e aborta a fase
Entrada: docs/levantamento-requisitos/requisitos-aprovados.md + docs/dax/matriz-viabilidade.md
Saída:
   - Novo arquivo Figma criado via figma:figma-generate-design
   - URL salva em docs/mockups/figma-url.md
   - Screenshot exportado para docs/mockups/screenshots/
```

**Gate 5** — aprovar mockup. Orquestrador mostra URL + thumbnails. Se ajustes pedidos, agente re-executa `use_figma` para iterar.

### Fase 6 — PBIR report builder

Lê `docs/mockups/figma-url.md` + screenshots como **referência visual**, lê medidas TMDL aprovadas, gera páginas/visuais PBIR.

**Gate 6** — aprovar relatório (orquestrador roda `pbip:pbip-validator` antes do gate; só apresenta para aprovação se passar).

### Fase 7 — QA (bi-qa-validator)

Valida cadeia completa: PRD → matriz rastreabilidade → modelo → medidas → visuais.
Saída: `docs/qa/relatorio-qa.md` com status verde/amarelo/vermelho por requisito.

**Gate 7** — aprovar QA. Se houver vermelhos, opção de voltar à fase correspondente.

### Fase 8 — Documentação (documentador-bi)

Gera `docs/documentacao/` (técnica, funcional, catálogo de medidas, dicionário de dados, release notes).

**Encerramento:** orquestrador commita `CHANGELOG.md`, fecha último gate, exibe checklist de handover.

---

## 6. Protocolo padrão de gate

Todo gate segue este formato (definido na skill `gate-protocol`):

1. **Resumo executivo** (3–5 bullets) do que o agente produziu
2. **Caminho dos artefatos** gerados/alterados
3. **Riscos/pendências** detectados
4. **`AskUserQuestion`** com 3 opções padrão:
   - `Aprovar e seguir para a próxima fase`
   - `Pedir ajustes` (campo livre — orquestrador devolve ao agente)
   - `Pausar projeto` (registra ponto de retomada em TODO.md)
5. **Pós-aprovação:** atualiza `TODO.md` (marca `[x]`), grava `DECISIONS.md` se for decisão estrutural, grava `CHANGELOG.md` com diff resumido.

---

## 7. Riscos e mitigações

| Risco | Probabilidade | Mitigação |
|---|---|---|
| Figma MCP não autenticado | Alta | Verificação prévia + instrução de `mcp__plugin_figma_figma__authenticate` |
| TODO.md fica inconsistente após erro no meio do gate | Média | Cada gate é transação: marca `[x]` só **depois** de gravar artefatos |
| Agente DAX classifica KPI como 🟢 mas modelo não fecha em runtime | Média | Gate 4b inclui pedido para usuário rodar 1 medida no Power BI Desktop antes de aprovar |
| Mockup Figma gerado não bate com expectativa visual | Média | Loop de iteração no Gate 5 (até 3 rounds antes de pausar) |
| Cascade de ajustes em fase tardia força refazer fases anteriores | Baixa | QA (Fase 7) tem botão "voltar para fase X" — re-abre gate específico |
| Power Query reviewer ainda não tem entrada e o usuário não tem fonte estruturada | Alta no primeiro projeto | Fallback: agente trabalha apenas com partitions TMDL existentes e registra pendência |
| Skill `figma:figma-generate-design` não está disponível no ambiente do usuário | Baixa-Média | Fase 5 detecta ausência e oferece fallback textual (mantém comportamento atual do agente 05) |

---

## 8. Critérios de aceite da implementação

- [ ] `/iniciar-projeto-bi` aparece como slash command e abre a Fase 0
- [ ] Comando detecta corretamente todos os 4 cenários da Fase 1 (transcrição / PRD existente / questionário / resumo)
- [ ] Cada gate usa `AskUserQuestion` com as 3 opções padrão
- [ ] `TODO.md` é atualizado após cada gate aprovado
- [ ] `CHANGELOG.md` recebe entrada padronizada por fase
- [ ] Re-execução do comando retoma do primeiro gate `[ ]`
- [ ] Agente 09 (Figma) cria arquivo no Figma e devolve URL
- [ ] Agente DAX entrega matriz de viabilidade **antes** de criar medidas
- [ ] Agente 06 (PBIR) consegue ler `docs/mockups/figma-url.md`
- [ ] Nenhum agente cruza escopo definido em `AGENTS.md`
- [ ] Testar com o caso fictício atual (Codex Retail) end-to-end uma vez antes de declarar pronto

---

## 9. Arquivos a criar/modificar (consolidado)

**Criar:**
```
.claude/commands/iniciar-projeto-bi.md
.claude/commands/retomar-projeto-bi.md
.claude/agents/09-figma-mockup-designer.md
.claude/skills/gate-protocol/SKILL.md
.claude/skills/figma-mockup/SKILL.md
docs/templates/TODO-template.md
docs/mockups/figma-url.md           (placeholder, populado em runtime)
docs/dax/matriz-viabilidade.md      (placeholder, populado em runtime)
docs/power-query/input/.gitkeep
```

**Modificar:**
```
.claude/agents/00-orquestrador-bi.md          (modo guided pipeline)
.claude/agents/01-analista-requisitos.md      (4 ramos de entrada)
.claude/agents/02-power-query-reviewer.md     (verificação de input)
.claude/agents/04-dax-specialist.md           (matriz viabilidade)
.claude/agents/05-client-presentation.md      (escopo reduzido)
.claude/agents/06-pbir-report-builder.md      (consome mockup Figma)
docs/matriz-agentes-skills.md                 (linha do agente 09)
AGENTS.md                                     (§ escopo do agente 09)
TODO.md                                       (adotar padrão de checkbox)
CHANGELOG.md                                  (entrada cobrindo esta mudança)
DECISIONS.md                                  (decisão de adotar pipeline guiado)
```

---

## 10. Decisões registradas neste design

| # | Decisão | Alternativa descartada | Razão |
|---|---|---|---|
| D-01 | Um único slash command `/iniciar-projeto-bi` | Vários comandos por fase | Pedido explícito do usuário; comandos por fase entram só como atalhos opcionais |
| D-02 | `TODO.md` como fonte da verdade do estado | JSON paralelo / arquivo `.state` | Já é o padrão do projeto, evita duplicação |
| D-03 | Novo agente 09 para Figma | Estender agente 05 | Pedido do usuário; mantém escopos isolados |
| D-04 | Matriz de viabilidade antes de criar medidas | Criar medidas e validar depois | Pedido do usuário; evita medidas órfãs |
| D-05 | `AskUserQuestion` com 3 opções fixas em todo gate | Texto livre | Padronização + análise de telemetria futura |
| D-06 | Gate de PBIR roda `pbip-validator` antes de pedir aprovação humana | Aprovar e validar depois | Evita aprovação cega |
| D-07 | Verificação de Figma MCP antes da Fase 5 | Tentar gerar e tratar erro | UX melhor — instruções claras antes do agente rodar |

---

## 11. Próximos passos

1. Usuário revisa esta spec (próximo gate desta sessão de brainstorming)
2. Após aprovação → skill `superpowers:writing-plans` gera plano de implementação detalhado com ordem, dependências e checkpoints
3. Plano executado por `superpowers:executing-plans` ou `superpowers:subagent-driven-development`
