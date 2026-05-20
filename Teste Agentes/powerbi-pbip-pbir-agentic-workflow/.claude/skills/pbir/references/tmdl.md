---
name: tmdl
version: 2026-04-24
summary: Skill para agentes que leem, analisam ou modificam arquivos TMDL em projetos Power BI PBIP/PBIR.
description: Use esta skill sempre que um agente precisar alterar objetos do modelo semântico Power BI em TMDL, incluindo tabelas, colunas, medidas, relacionamentos, partições, expressões M, roles, calculation groups, perspectivas, descrições, display folders e propriedades avançadas do Tabular Object Model.
allowed_agents:
  - power-query-reviewer
  - data-modeler
  - dax-specialist
  - pbir-report-builder
  - bi-qa-validator
  - documentador
last_reviewed: 2026-04-24
sources:
  - Microsoft Learn: Tabular Model Definition Language
  - Microsoft Learn: TMDL View in Power BI Desktop
  - Microsoft Learn: Power BI Desktop project semantic model folder
  - Microsoft Power BI Blog: TMDL View on the Web Preview
---

# Skill: TMDL para Power BI PBIP/PBIR

## 1. Objetivo da skill

Esta skill orienta agentes que precisam trabalhar com **Tabular Model Definition Language (TMDL)** em projetos Power BI no formato **PBIP/PBIR**.

Use esta skill quando o agente precisar:

- Ler ou modificar arquivos `.tmdl`.
- Criar ou alterar medidas DAX no modelo semântico.
- Criar ou revisar relacionamentos.
- Alterar tabelas, colunas, hierarquias, partições, expressões M ou propriedades do modelo.
- Organizar display folders, descrições, formatos e metadados.
- Validar impacto de alterações no modelo semântico antes de o relatório PBIR ser ajustado.

TMDL é uma representação textual do modelo tabular. Ele é otimizado para leitura humana, versionamento e edição como código. A sintaxe usa indentação para representar hierarquia de objetos, de forma parecida com YAML.

---

## 2. Quando usar esta skill

Use obrigatoriamente para agentes que modificam qualquer item dentro de:

```text
*.SemanticModel/definition/
*.Dataset/definition/
powerbi/semanticModel/definition/
```

Arquivos comuns:

```text
definition.pbism
model.tmdl
relationships.tmdl
expressions.tmdl
tables/*.tmdl
roles/*.tmdl
perspectives/*.tmdl
cultures/*.tmdl
```

Também use quando o agente alterar scripts TMDL em:

```text
TMDLScripts/
```

---

## 3. Agentes que devem usar esta skill

| Agente | Usa TMDL? | Motivo |
|---|---:|---|
| Power Query Reviewer | Sim | Pode alterar partições, expressões M e consultas dentro de TMDL. |
| Data Modeler | Sim | Cria e revisa tabelas, colunas, relacionamentos, cardinalidade e direções de filtro. |
| DAX Specialist | Sim | Cria medidas, formatos, descrições e display folders. |
| PBIR Report Builder | Sim | Precisa ler medidas, tabelas e campos disponíveis para criar visuais corretos. Só deve alterar TMDL se for autorizado. |
| BI QA Validator | Sim | Valida consistência entre requisito, modelo, DAX e relatório. |
| Documentador | Sim | Lê TMDL para documentar modelo, medidas e regras. |

---

## 4. Regra central de segurança

Nenhum agente deve modificar TMDL diretamente sem executar este ciclo:

```text
1. Diagnosticar
2. Mapear dependências
3. Propor plano de alteração
4. Alterar o menor escopo possível
5. Validar impacto
6. Atualizar documentação
7. Registrar mudança no CHANGELOG.md
```

Antes de qualquer alteração, o agente deve responder:

```text
- Qual objeto será alterado?
- Por que será alterado?
- Quais objetos dependem dele?
- Existe impacto em medidas, relacionamentos, visuais PBIR ou Power Query?
- A alteração exige aprovação humana?
```

---

## 5. Escopo do TMDL

TMDL pode representar objetos do modelo tabular, incluindo:

- Modelo.
- Tabelas.
- Colunas.
- Medidas.
- Hierarquias.
- Partições.
- Expressões M.
- Relacionamentos.
- Roles/RLS.
- Culturas/traduções.
- Perspectivas.
- Calculation groups.
- Propriedades avançadas do Tabular Object Model.

A skill deve considerar que nem todas as propriedades aparecem na interface do Power BI Desktop. Algumas propriedades avançadas podem ser acessadas apenas por TMDL, Tabular Editor, TOM ou TMDL View.

---

## 6. Estrutura típica em PBIP com TMDL

Um projeto PBIP com modelo semântico em TMDL costuma ter estrutura semelhante a:

```text
Projeto.SemanticModel/
├── definition.pbism
├── diagramLayout.json
├── .pbi/
│   ├── editorSettings.json
│   ├── localSettings.json
│   ├── cache.abf
│   └── unappliedChanges.json
├── definition/
│   ├── model.tmdl
│   ├── expressions.tmdl
│   ├── relationships.tmdl
│   ├── tables/
│   │   ├── fato_Vendas.tmdl
│   │   ├── dim_Cliente.tmdl
│   │   ├── dim_Produto.tmdl
│   │   └── dim_Calendario.tmdl
│   ├── roles/
│   ├── perspectives/
│   └── cultures/
└── TMDLScripts/
```

Observações:

- `definition.pbism` é obrigatório.
- O formato TMDL usa a pasta `definition/`.
- Arquivos `.pbi/localSettings.json` e `.pbi/cache.abf` normalmente não devem ser versionados.
- `unappliedChanges.json` pode conter alterações pendentes do Power Query salvas pelo Power BI Desktop.

---

## 7. Atenção especial ao `unappliedChanges.json`

Antes de alterar Power Query, partições ou expressões M via TMDL, verificar se existe:

```text
.pbi/unappliedChanges.json
```

Risco:

- O Power BI Desktop pode sobrescrever consultas do modelo se houver alterações pendentes no Power Query.
- Se o agente editar TMDL fora do Desktop enquanto existir `unappliedChanges.json`, as alterações podem ser perdidas ao aplicar mudanças pendentes.

Regra:

```text
Se existir unappliedChanges.json:
1. Avisar o usuário.
2. Não modificar expressões M sem confirmação.
3. Sugerir abrir o PBIP no Power BI Desktop e aplicar ou descartar alterações pendentes.
```

---

## 8. Regras de edição TMDL

### 8.1 Preservar indentação

TMDL usa indentação para representar hierarquia. Não reformatar arquivos inteiros sem necessidade.

Regra:

```text
- Alterar apenas os blocos necessários.
- Preservar indentação existente.
- Não misturar tabs e espaços dentro do mesmo bloco.
```

### 8.2 Não renomear objetos sem rastrear dependências

Renomear tabela, coluna ou medida pode quebrar:

- Medidas DAX.
- Relacionamentos.
- Visuais PBIR.
- Tooltips.
- Bookmarks.
- Filtros de página.
- RLS.
- Perspectivas.
- Traduções.

Checklist obrigatório antes de renomear:

```text
1. Buscar o nome antigo no repositório inteiro.
2. Verificar arquivos TMDL.
3. Verificar arquivos PBIR.
4. Verificar medidas DAX.
5. Verificar relacionamentos.
6. Verificar roles/RLS.
7. Documentar substituições.
```

### 8.3 Não excluir objetos sem análise de uso

Antes de excluir tabela, coluna, medida ou relacionamento:

```text
- Procurar referências no repositório.
- Verificar uso em DAX.
- Verificar uso no PBIR.
- Verificar uso em relações.
- Verificar uso em roles.
- Verificar se o objeto é técnico, oculto ou usado como chave.
```

Se houver dúvida, marcar como `isHidden: true` em vez de excluir, quando fizer sentido.

### 8.4 Alterar o menor escopo possível

Evitar reescrever arquivos inteiros. Preferir patches pequenos.

Formato recomendado de resposta do agente:

```text
Alterei apenas:
- tables/fato_Vendas.tmdl: inclusão de 3 medidas.
- relationships.tmdl: ajuste de direção de filtro em 1 relacionamento.
```

---

## 9. Boas práticas de nomenclatura

### 9.1 Tabelas

Padrão recomendado:

```text
dim_Cliente
dim_Produto
dim_Calendario
dim_Vendedor
fato_Vendas
fato_Estoque
aux_Metas
stg_Vendas
```

Regras:

- `dim_` para dimensões.
- `fato_` para fatos.
- `aux_` para tabelas auxiliares de análise.
- `stg_` para staging ou tabelas intermediárias.
- Evitar nomes genéricos como `Tabela1`, `Consulta1`, `Base`, `Planilha1`.

### 9.2 Colunas

Padrão recomendado:

```text
ID Cliente
ID Produto
Data Emissão
Valor Bruto
Valor Líquido
Quantidade
Status Pedido
```

Regras:

- Usar nomes compreensíveis para usuário final quando a coluna estiver visível.
- Ocultar colunas técnicas.
- Não expor chaves substitutas sem necessidade.
- Evitar prefixos técnicos em campos que aparecem no painel de dados do usuário final.

### 9.3 Medidas

Padrão recomendado:

```text
Receita Bruta
Receita Líquida
Quantidade Vendida
Ticket Médio
Margem Bruta
Margem Bruta %
Atingimento Meta %
Variação Receita %
```

Regras:

- Medidas devem ser orientadas ao negócio.
- Evitar prefixos como `med_`, `calc_`, `m_` no nome exibido ao usuário.
- Usar display folders para organização.
- Toda medida importante deve ter descrição.

### 9.4 Display folders sugeridos

```text
01. Vendas
02. Quantidade
03. Margem
04. Ticket Médio
05. Metas
06. Tempo
07. Rankings
08. Clientes
09. Produtos
99. Técnicas
```

---

## 10. Boas práticas para medidas DAX em TMDL

Ao criar medidas:

- Usar medidas base antes de medidas derivadas.
- Usar `DIVIDE()` para divisões.
- Usar variáveis para legibilidade.
- Evitar duplicação de lógica.
- Adicionar `formatString` coerente.
- Adicionar descrição funcional com `///` quando aplicável.
- Colocar a medida em display folder.

Exemplo conceitual:

```tmdl
/// Receita líquida total considerando devoluções e descontos comerciais.
measure 'Receita Líquida' =
    SUM ( fato_Vendas[Valor Líquido] )
    formatString: "R$ #,0.00"
    displayFolder: "01. Vendas"
```

Medida derivada:

```tmdl
/// Ticket médio por pedido, calculado pela divisão entre receita líquida e quantidade de pedidos.
measure 'Ticket Médio' =
    DIVIDE (
        [Receita Líquida],
        [Quantidade de Pedidos]
    )
    formatString: "R$ #,0.00"
    displayFolder: "04. Ticket Médio"
```

Regra:

```text
Nunca criar medida se ela não estiver ligada a:
- Um requisito aprovado;
- Uma pergunta de negócio;
- Um visual planejado;
- Uma necessidade técnica documentada.
```

---

## 11. Boas práticas para relacionamentos

O agente de modelagem deve priorizar:

```text
- Modelo estrela.
- Fatos no centro.
- Dimensões nas bordas.
- Relacionamentos 1:* entre dimensão e fato.
- Filtro single-direction sempre que possível.
- Muitos-para-muitos apenas com justificativa.
- Bidirecional apenas quando necessário e documentado.
```

Checklist antes de alterar relacionamento:

```text
1. Identificar tabela fato.
2. Identificar tabela dimensão.
3. Confirmar granularidade das tabelas.
4. Confirmar unicidade da chave na dimensão.
5. Validar tipo de dados das chaves.
6. Validar direção de filtro.
7. Avaliar impacto em medidas existentes.
8. Documentar decisão.
```

Toda alteração em relacionamento deve atualizar:

```text
docs/dicionario-dados/relacionamentos.md
DECISIONS.md
CHANGELOG.md
```

---

## 12. Boas práticas para Power Query dentro de TMDL

Quando TMDL contiver partições com expressão M:

```tmdl
partition 'Tabela-Partition' = m
    mode: import
    source =
        let
            Source = ...
        in
            ...
```

Regras:

- Não alterar M sem usar também a skill `power-query-m`.
- Verificar query folding quando fonte permitir.
- Não remover coluna usada em relacionamento, DAX ou PBIR.
- Padronizar etapas M sem quebrar dependências.
- Evitar transformar tipos no final se isso comprometer performance.
- Registrar qualquer mudança em fonte, filtro, merge, append ou tipo de dado.

---

## 13. Propriedades importantes que o agente deve preservar

Ao editar TMDL, preservar quando existirem:

```text
lineageTag
summarizeBy
formatString
isHidden
isAvailableInMdx
sourceColumn
dataType
sortByColumn
displayFolder
description
annotations
changedProperties
```

Não remover propriedades desconhecidas. Se o agente não souber o impacto, deve preservar.

---

## 14. Descrições com `///`

Use descrições para objetos importantes:

```tmdl
/// Tabela fato com pedidos faturados, devoluções e valores comerciais.
table fato_Vendas
```

```tmdl
/// Receita bruta antes de descontos e devoluções.
measure 'Receita Bruta' =
    SUM ( fato_Vendas[Valor Bruto] )
```

Regras:

- Toda medida final voltada ao usuário deve ter descrição.
- Tabelas fato e dimensão devem ter descrição.
- Colunas técnicas podem ter descrição quando houver regra relevante.

---

## 15. Compatibilidade com PBIR

Alterações em TMDL podem impactar diretamente o relatório PBIR.

Antes de alterar nomes, remover campos ou mudar medidas, buscar referências em:

```text
*.Report/definition/
*.Report/report.json
*.Report/pages/
*.Report/visuals/
powerbi/report/
```

Riscos comuns:

```text
- Visual quebrado por medida renomeada.
- Filtro quebrado por coluna removida.
- Tooltip quebrado por campo ocultado ou excluído.
- Bookmark inconsistente.
- Página usando medida antiga.
```

Regra:

```text
Se uma alteração TMDL afetar PBIR, chamar o agente pbir-report-builder ou bi-qa-validator.
```

---

## 16. Scripts TMDL

TMDL scripts possuem comando no topo, seguido de um ou mais objetos.

Comandos comuns:

```text
createOrReplace
alter
create
replace
delete
```

Regra de segurança:

- Preferir `createOrReplace` apenas quando o objeto inteiro for conhecido.
- Evitar `delete` sem validação explícita.
- Para alterações pequenas, preferir modificar o arquivo específico no projeto PBIP em vez de gerar script amplo.

Exemplo conceitual:

```tmdl
createOrReplace
    table dim_Calendario
        column Data
            dataType: dateTime
```

---

## 17. Fluxo obrigatório para agentes modificadores

### 17.1 Antes da alteração

O agente deve gerar:

```md
## Diagnóstico TMDL

### Arquivos analisados

### Objetos impactados

### Dependências encontradas

### Riscos

### Plano de alteração
```

### 17.2 Durante a alteração

O agente deve:

```text
- Editar apenas arquivos necessários.
- Preservar propriedades existentes.
- Não reformatar tudo.
- Não misturar responsabilidades de outros agentes.
```

### 17.3 Depois da alteração

O agente deve gerar:

```md
## Validação pós-alteração

### Arquivos alterados

### Objetos criados/alterados

### Dependências validadas

### Riscos remanescentes

### Próximos passos
```

E atualizar:

```text
CHANGELOG.md
DECISIONS.md, se houver decisão técnica
TODO.md, se houver pendência
```

---

## 18. Checklist para revisão TMDL

Use esta lista antes de finalizar qualquer tarefa:

```text
[ ] O arquivo TMDL continua com indentação válida.
[ ] O objeto alterado existe no escopo correto.
[ ] Não houve renomeação sem busca de dependências.
[ ] Medidas DAX referenciam objetos existentes.
[ ] Relacionamentos usam colunas com tipos compatíveis.
[ ] Colunas técnicas desnecessárias foram ocultadas, não expostas.
[ ] Medidas possuem formato adequado.
[ ] Medidas relevantes possuem descrição.
[ ] Display folders foram aplicados quando necessário.
[ ] Alterações em Power Query consideraram unappliedChanges.json.
[ ] Alterações que afetam PBIR foram sinalizadas.
[ ] CHANGELOG.md foi atualizado.
[ ] DECISIONS.md foi atualizado se houve decisão arquitetural.
```

---

## 19. Critérios de qualidade

Uma alteração TMDL é considerada boa quando:

```text
- Resolve um requisito claro.
- É pequena e rastreável.
- Mantém o modelo semanticamente consistente.
- Preserva propriedades existentes.
- Não quebra PBIR.
- Não cria ambiguidade de relacionamento.
- Não duplica lógica DAX.
- Está documentada.
```

Uma alteração TMDL é considerada ruim quando:

```text
- Renomeia objetos sem rastrear dependências.
- Remove propriedades desconhecidas.
- Reescreve arquivos inteiros sem necessidade.
- Cria medida sem requisito.
- Cria relacionamento bidirecional sem justificativa.
- Usa muitos-para-muitos para resolver problema de modelagem mal definido.
- Altera Power Query sem validar impacto no modelo.
```

---

## 20. Integração com outras skills

Esta skill deve ser combinada com:

```text
power-query-m
```

quando alterar partições ou expressões M.

```text
dimensional-modeling
```

quando alterar tabelas, chaves, cardinalidade ou relacionamentos.

```text
dax
```

quando criar ou revisar medidas.

```text
pbir
```

quando a alteração impactar visuais, filtros, páginas, bookmarks ou tooltips.

```text
qa-validation
```

quando concluir alteração e validar consistência do fluxo completo.

---

## 21. Prompt interno recomendado para agentes

Sempre que um agente for modificar TMDL, ele deve seguir este prompt mental:

```text
Estou alterando o modelo semântico. Antes de editar, preciso:

1. Identificar objeto e arquivo.
2. Verificar dependências no repositório.
3. Verificar impacto em PBIR.
4. Verificar se há unappliedChanges.json.
5. Aplicar alteração mínima.
6. Preservar propriedades existentes.
7. Documentar mudança.
8. Listar riscos remanescentes.
```

---

## 22. Exemplo de saída esperada do agente

```md
## Diagnóstico

Identifiquei que a medida `Receita` está sem descrição, sem display folder e com nome pouco específico para o requisito aprovado de Receita Líquida.

## Plano

- Criar nova medida `Receita Líquida`.
- Manter `Receita` temporariamente para evitar quebra de visuais.
- Adicionar descrição e formatString.
- Sinalizar ao PBIR Report Builder para substituir o uso da medida antiga nos visuais.

## Alterações realizadas

- `tables/fato_Vendas.tmdl`: criada medida `Receita Líquida`.
- `CHANGELOG.md`: registrado ajuste.
- `TODO.md`: criada pendência para revisar visuais que ainda usam `Receita`.

## Riscos

- A medida antiga ainda pode estar em uso no PBIR.
```

---

## 23. Regra final

TMDL deve ser tratado como **código-fonte crítico do modelo semântico**.

Portanto:

```text
Não edite como texto solto.
Edite como contrato de dados, regra de negócio e camada semântica do Power BI.
```
