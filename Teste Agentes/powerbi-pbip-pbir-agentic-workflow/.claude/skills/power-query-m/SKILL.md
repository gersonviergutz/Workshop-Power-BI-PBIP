---
name: power-query-m
description: Auditoria e padronização de Power Query (M) no padrão Dashmaker. Cobre nomenclatura (camelCase em queries, PascalCase em colunas, __ em parâmetros internos, snake_case em SQL, kebab-case em arquivos), organização (grupos, parâmetros, Fonte/Staging/Modelo, renomear steps), performance (Query Folding, ordem de etapas, Table.Buffer, redução antecipada), Incremental Refresh (RangeStart/RangeEnd), funções customizadas com fx, Privacy Levels, qualidade de dados. Use SEMPRE que o usuário pedir "revisa minha query", "audita meu Power Query", "refresh lento", "query folding quebrou", "padroniza meu M", "nomenclatura Power Query", "boas práticas M", "organiza minhas queries", "refatora esse M", ou compartilhar código M, .pq, .pbip, trechos de let...in. Dispara também para dúvidas de Table.Buffer, Merge vs Append, renomear etapas, dataInicial/dataFinal, Fontes/Staging/Modelo, Privacy Levels, performance em refresh. Entrega relatório categorizado (Crítico/Aviso/Info) com correções em M em PT-BR.
---

# Boas Práticas de Power Query (M)

Skill para auditar, reescrever e padronizar queries em Power Query (linguagem M) seguindo convenções de nomenclatura consistentes e boas práticas consolidadas da comunidade. Foca especialmente em performance (Query Folding), organização (grupos e parâmetros) e manutenibilidade (nomenclatura, comentários, funções customizadas).

## Por que essa skill existe

Power Query é frequentemente tratado como "drag-and-drop sem consequências", mas na prática é onde a maioria dos modelos perde performance e vira pesadelo de manutenção. As queries acumulam etapas com nomes default ("Linhas Filtradas", "Tipo Alterado1", "Coluna Personalizada12"), parâmetros hard-coded, Buffers desnecessários e quebras silenciosas de Query Folding. Quando o refresh começa a passar dos 30 minutos, já é tarde.

O Power Query também não tem um analisador oficial equivalente ao BPA do modelo semântico, então as regras ficam espalhadas em blogs e livros da comunidade. Essa skill condensa o essencial em um roteiro prático de auditoria em PT-BR, aplicando as convenções Dashmaker e os pontos validados ao longo dos anos por autores de referência.

## Antes de qualquer auditoria

1. **Entenda o escopo.** O usuário quer auditoria completa (nomenclatura + organização + performance + qualidade)? Ou só um recorte ("só renomeia minhas queries no padrão", "só investiga por que está lento")? Se não estiver claro, pergunte brevemente.

2. **Obtenha o código M.** Pode vir via:
   - **Upload de arquivo .pq ou .m** — leia direto
   - **PBIP/PBIR** — as queries M ficam em `SemanticModel/definition/tables/*.tmdl` dentro do bloco `partition ... = m source = ...`. Parâmetros e named expressions em `expressions.tmdl`.
   - **Cole inline** do Editor Avançado do Power Query (let ... in ...)
   - **Screenshot do editor** — pode trabalhar, mas avise que detecção de folding e nomes exatos fica limitada

3. **Carregue a referência relevante** (progressive disclosure — leia só o que for necessário para a tarefa):
   - `references/nomenclatura.md` — convenções Dashmaker completas (camelCase para queries, PascalCase para colunas, `__` para parâmetros internos, kebab-case, snake_case)
   - `references/organizacao.md` — grupos/pastas, parâmetros, Enable Load, padrão Fonte → Staging → Modelo, renomear steps
   - `references/performance.md` — Query Folding, ordem de etapas, Table.Buffer, redução antecipada, tipos de dados
   - `references/transformacoes.md` — Merge vs Append, List/Record functions, Group By, Expand com performance, funções customizadas
   - `references/incremental-refresh.md` — RangeStart/RangeEnd, patterns de particionamento, requisitos de folding
   - `references/qualidade-e-seguranca.md` — Privacy Levels, credenciais, validação, nulls, try/otherwise, normalização

4. **Asset `assets/convencoes-dashmaker.md`** — tabela única de referência rápida das convenções. Consulte sem reler a referência de nomenclatura inteira.

## Sistema de severidade

Como Power Query não tem BPA oficial, essa skill usa uma escala equivalente:

- **🔴 Crítico** — quebra refresh, inviabiliza Incremental Refresh, expõe credenciais, causa lentidão severa (>30min), ou viola Privacy Levels. Corrigir antes do deploy.
- **🟡 Aviso** — performance degradada, manutenibilidade comprometida, padrões inconsistentes, folding quebrado sem motivo claro. Corrigir na próxima sprint.
- **🔵 Info** — polish, documentação, melhorias opcionais (renomear steps, adicionar comentários).

## Fluxo padrão de auditoria

### Passo 1 — Inventário

Levante do projeto:
- **Lista de queries** (nome, tipo inferido: Fonte/Staging/Modelo/Função, Enable Load sim/não, grupo/pasta)
- **Parâmetros** existentes (nome, tipo, valor default, uso)
- **Funções customizadas** (nome, entrada/saída, onde é chamada)
- **Fontes de dados** (tipo: SQL/Excel/CSV/API/Pasta, Privacy Level)
- **Etapas por query** — nome de cada step (default vs renomeado)

### Passo 2 — Aplicação das regras

Para cada categoria selecionada, percorra o checklist da referência e capture violações:
- **Regra violada** (nome claro, ex.: "Query Folding quebrado no step de filtro de data")
- **Severity** (🔴/🟡/🔵)
- **Objeto afetado** (ex.: "Query `vendas` — step `Linhas Filtradas1`")
- **Por que importa** (impacto real: refresh lento / manutenção difícil / credencial exposta)
- **Código M corrigido** sempre que possível (não só "renomeie a etapa")

### Passo 3 — Relatório

Sempre use essa estrutura (ajuste volumes conforme tamanho da auditoria):

```markdown
# Auditoria Power Query — [Nome do projeto]

## Resumo executivo
- Queries auditadas: [N]
- Total de violações: [N] (🔴 [x] / 🟡 [y] / 🔵 [z])
- Query Folding preservado em: [X de Y queries]
- Top 3 prioridades: [...]

## 🔴 Críticos
### [Regra] — query `[nome]` step `[nome]`
**Problema:** [...]
**Impacto:** [...]
**Correção:**
​```m
// M original
...
// M corrigido
...
​```

## 🟡 Avisos
[...]

## 🔵 Infos
[...]

## Checklist de boas práticas atendidas
[Lista das validações que passaram — ajuda a dar visibilidade do que está OK]
```

Se o relatório ficar grande (>200 linhas), salve como `.md` em `/mnt/user-data/outputs/` e apresente via `present_files`.

## Checklist mestre rápido

Quando o usuário pedir auditoria express, percorra este checklist compacto:

### Organização
- [ ] Queries nomeadas em camelCase (`fDados`, `dCliente`, `fxGeraCalendario`)
- [ ] Colunas em PascalCase (`SalesAmount`, `DataFaturamento`)
- [ ] Parâmetros em camelCase externo (`dataInicial`) ou `__PascalCase`/`__camelCase` interno
- [ ] Queries agrupadas em pastas (Parâmetros, Funções, Fontes, Staging, Dimensões, Fatos)
- [ ] Queries intermediárias/staging com "Enable load" DESMARCADO
- [ ] Parâmetros usados em vez de valores hard-coded (servidores, caminhos, datas)
- [ ] Etapas com nomes descritivos (não "Tipo Alterado1", "Linhas Filtradas12")
- [ ] Funções customizadas com prefixo `fx` (`fxLimpaTexto`, `fxGeraCalendario`)

### Performance (Query Folding)
- [ ] Query Folding preservado nas fontes com suporte (SQL, OData, Dataverse)
- [ ] Filtros aplicados ANTES de expansões, merges e joins
- [ ] Colunas desnecessárias removidas CEDO (primeiro ou segundo step)
- [ ] Tipos de dados definidos logo após Source (cedo, não tardio)
- [ ] `Table.Buffer` usado apenas onde faz sentido (ciente de que quebra folding)
- [ ] Sem transformações M-only pesadas antes de ponto de folding possível

### Manutenibilidade
- [ ] Comentários com `//` explicando o "porquê" em lógica não-óbvia
- [ ] Sem credenciais/tokens hard-coded no código M
- [ ] Nomes de arquivos externos em kebab-case
- [ ] Queries em SQL (Value.NativeQuery) em snake_case ou SCREAMING_SNAKE_CASE
- [ ] Separação clara entre Fonte, Staging (tratamento) e Modelo (carga)

### Incremental Refresh (quando aplicável)
- [ ] Parâmetros `RangeStart` e `RangeEnd` definidos (nomes exatos, case-sensitive)
- [ ] Filtro de data com Query Folding verificado
- [ ] Policy configurada no desktop e publicada no serviço

### Qualidade e Segurança
- [ ] Privacy Levels definidos em todas as fontes
- [ ] Credenciais via gateway ou "Use Current Credentials" (nunca hard-coded)
- [ ] Tratamento de nulls/erros em colunas críticas (try/otherwise)

## Validação de Query Folding (ponto crítico #1)

Esse é o ponto #1 de performance em Power Query. Sempre ensine o usuário a validar:

**Como verificar** (Power BI Desktop):
1. Abra o Power Query Editor
2. Clique com o botão direito em cada etapa (step)
3. Se aparecer **"View Native Query"** (Exibir Consulta Nativa) HABILITADO → folding OK até essa etapa
4. Se aparecer **cinza/desabilitado** → folding quebrou nessa etapa

**O que quebra folding (lista essencial):**
- `Table.Buffer` — sempre quebra
- `Table.AddColumn` com lógica M complexa (depende do caso)
- `List.Accumulate`, `List.Generate` usados para gerar colunas
- `Table.TransformColumns` com funções M customizadas
- Merge com query M-only (sem folding do outro lado)
- Tipos `Any` ou conversões M-specific
- `Table.FromList`, `Table.FromRecords` no meio da query

**Quando Buffer faz sentido** (apesar de quebrar folding):
- Queries pequenas referenciadas múltiplas vezes (evita re-execução)
- Proteger contra issues de Privacy Level em merges complexos
- Forçar materialização antes de transformações sequenciais que dependem do mesmo set

## Adaptações ao contexto Dashmaker

- **Idioma do relatório:** sempre PT-BR. Mantenha identificadores M em inglês (é linguagem), mas steps renomeados em PT.
- **Prefixos de query recomendados:**
  - `f` para fato (`fVendas`, `fEstoque`)
  - `d` para dimensão (`dCliente`, `dProduto`, `dCalendario`)
  - `fx` para função customizada (`fxLimpaTexto`, `fxGeraCalendario`)
  - `vw` para views de banco importadas direto (`vw_vendas` quando nomeadas em SQL-style)
  - Parâmetros sem prefixo, camelCase (`dataInicial`, `caminhoServidor`, `nomeAmbiente`)
- **Prefixo interno `__`** para variáveis `let` dentro de queries (`let __ValorAnterior = ...`, `let __listaClientes = ...`). Diferencia visualmente das steps de transformação.
- **Grupos padronizados:** `Parâmetros`, `Funções`, `Fontes`, `Staging`, `Dimensões`, `Fatos` (ou recorte equivalente). Sem grupo = query na raiz, geralmente queries finais/expostas ao modelo.

## Referências externas úteis

Quando o usuário pedir mais profundidade, aponte:
- **Rick de Groot** (powerquery.how) — tutoriais profundos sobre M, code library
- **Chris Webb** (blog.crossjoin.co.uk) — performance, folding, internals do engine
- **Imke Feldmann** (thebiccountant.com) — funções customizadas avançadas
- **Ken Puls** (livro "M is for Data Monkey", ExcelGuru blog) — fundamentos sólidos
- **Microsoft Learn** — docs oficiais de Power Query M
- **PowerQuery.how Code Library** — snippets prontos

## Idioma

Sempre responda em PT-BR. Se o usuário escrever em outra língua, siga a dele. Identificadores M (`let`, `in`, `Table.SelectRows`) permanecem em inglês porque é linguagem.