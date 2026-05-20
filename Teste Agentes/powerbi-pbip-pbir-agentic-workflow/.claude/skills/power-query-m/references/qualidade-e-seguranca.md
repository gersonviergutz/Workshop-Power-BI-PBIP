# Qualidade e Segurança em Power Query

Tratamento de nulls e erros, Privacy Levels, gerenciamento de credenciais, validação de dados e padrões de robustez. A pior hora para descobrir que tem credencial hard-coded é quando o arquivo já foi publicado ou enviado por e-mail.

## Sumário

1. [Privacy Levels (Níveis de Privacidade)](#1-privacy-levels-níveis-de-privacidade)
2. [Credenciais e autenticação](#2-credenciais-e-autenticação)
3. [Tratamento de nulls](#3-tratamento-de-nulls)
4. [Tratamento de erros (try/otherwise)](#4-tratamento-de-erros-tryotherwise)
5. [Validação de dados](#5-validação-de-dados)
6. [Normalização de strings](#6-normalização-de-strings)
7. [Padrões de robustez](#7-padrões-de-robustez)

---

## 1. Privacy Levels (Níveis de Privacidade)

### O que são

Privacy Levels controlam como o Power Query combina dados de fontes diferentes. Impedem, por exemplo, que dados privados de um SQL interno sejam enviados para uma API pública durante um merge.

Os níveis:
- **None** — sem classificação (evite em produção)
- **Public** — pode ser compartilhado com qualquer fonte (ex.: API pública de cotação de moeda)
- **Organizational** — só pode combinar com outras fontes Organizational (ex.: SQL Server interno da empresa)
- **Private** — NÃO pode combinar com nenhuma outra fonte (ex.: dados de RH sensíveis)

### Onde configurar

Arquivo → Opções e Configurações → Opções → Arquivo Atual → Privacy → escolher comportamento.

Três modos:
1. **Always combine data according to your Privacy Level settings for each source** (padrão recomendado para produção)
2. **Combine data according to each file's Privacy Level setting** (flexível mas manual)
3. **Always ignore Privacy Level settings** (rápido mas arriscado — nunca use em produção)

### Erro clássico: Formula.Firewall

Quando você faz merge de duas fontes com níveis incompatíveis, aparece:

```
Formula.Firewall: Query 'X' (step 'Y') references other queries or steps, 
so it may not directly access a data source. Please rebuild this data combination.
```

**Soluções:**

1. **Ajustar Privacy Levels:** revisar classificação das fontes (às vezes uma está como Private quando deveria ser Organizational)
2. **Reescrever a query:** combinar as fontes em uma única função, sem intermediárias
3. **Usar Table.Buffer:** materializa uma das tabelas antes do merge, pode contornar em alguns casos
4. **Ignorar (desenvolvimento):** temporariamente setar "Always ignore Privacy Level settings" para depurar — NUNCA em produção

### 🔴 Sempre definir Privacy Levels em produção

Modelos publicados sem Privacy Levels configurados podem:
- Enviar dados sensíveis para APIs externas durante transformações
- Quebrar refresh no serviço quando o engine do Gateway aplica regras diferentes
- Gerar logs de segurança que dificultam auditoria

**Regra:** na primeira conexão com cada fonte, defina o nível imediatamente.

---

## 2. Credenciais e autenticação

### 🔴 Nunca hard-code credenciais

**ERRADO:**
```m
Source = Sql.Database("srv01", "db1", [Query="SELECT * FROM tabela"]),
// Ou pior:
auth = [Username="gerson@empresa.com", Password="senha123"]  // NUNCA
```

Credenciais hard-coded em M:
- Ficam visíveis no arquivo .pbip/.pbix (texto plano para `.pbip`!)
- Vão para Git se o arquivo for versionado
- Ficam expostas em backups
- Violam qualquer política de segurança séria

### Abordagens corretas

**1. Credenciais gerenciadas pelo Power BI Desktop (local):**
Na primeira conexão com a fonte, o Power BI pergunta credenciais e as armazena no Windows Credential Manager (Gerenciador de Credenciais). Ficam criptografadas por usuário.

**2. Credenciais gerenciadas pelo Power BI Service:**
Após publicar, configure as credenciais no Dataset Settings → Data source credentials.

**3. On-premises data gateway:**
Para fontes locais (SQL Server interno, arquivos em rede), credenciais ficam no gateway e NÃO saem do ambiente. Ideal para segurança corporativa.

**4. Azure Key Vault (avançado):**
Para cenários complexos, credenciais podem vir do Key Vault via conexão específica. Requer Premium/PPU.

### Privacy Level e credencial

São coisas separadas:
- **Credencial** — COMO você autentica (Windows, OAuth2, Key)
- **Privacy Level** — O QUE pode ser combinado

Uma fonte pode ter credencial "Windows" e Privacy Level "Organizational" — são configs independentes.

### Web.Contents com header de autenticação

Padrão comum para APIs com token:

**❌ Anti-pattern (hard-coded):**
```m
Source = Web.Contents("https://api.exemplo.com/dados", [
    Headers = [Authorization = "Bearer abc123...chaveexposta"]
])
```

**✅ Correto (via parâmetro + credencial customizada):**
```m
// Parâmetro "apiToken" (type Text) criado separadamente
Source = Web.Contents("https://api.exemplo.com/dados", [
    Headers = [Authorization = "Bearer " & apiToken]
])
```

Mesmo assim, o token em parâmetro fica no arquivo. Para máxima segurança, armazene o token em variável de ambiente via Azure Function como proxy, ou use Web.Contents com autenticação OAuth2 quando a API suportar.

---

## 3. Tratamento de nulls

### Nulls em Power Query

Null em M representa "ausência de valor" (equivalente ao NULL em SQL). É diferente de:
- String vazia (`""`)
- Zero (`0`)
- Erro (`Error`)

### Verificações básicas

```m
// Testar se é null
valorSeguro = if valor = null then 0 else valor

// Operador "??" não existe em M. Use:
valorComDefault = if [coluna] = null then "Sem Valor" else [coluna]

// Ou a função nativa (cuidado — só funciona em alguns contextos):
valor = Value.ReplaceType(potencialmenteNulo, type nullable text)
```

### Limpeza de nulls em coluna

Cenários:

**Substituir nulls por default:**
```m
Table.ReplaceValue(Source, null, "Desconhecido", Replacer.ReplaceValue, {"NomeCliente"})
```

**Remover linhas com null em coluna crítica:**
```m
Table.SelectRows(Source, each [ClienteId] <> null)
```

**Default múltiplas colunas de uma vez:**
```m
Table.ReplaceValue(
    Source, 
    null, 
    "N/A", 
    Replacer.ReplaceValue, 
    {"Status", "Categoria", "Origem"}
)
```

### Cuidado com null em merge

Quando duas colunas de merge têm null, o Power Query pode "matchar" nulls entre si (diferente do SQL padrão). Resultado: duplicação inesperada.

**Prevenção:** antes do merge, filtre nulls da chave ou substitua por valor sentinel:

```m
// Opção 1: filtrar antes do merge
keyValida = Table.SelectRows(Source, each [ClienteId] <> null),
merge = Table.NestedJoin(keyValida, {"ClienteId"}, dCliente, {"ClienteId"}, "cli", JoinKind.LeftOuter)

// Opção 2: substituir null por sentinel
keyAjustada = Table.ReplaceValue(Source, null, -1, Replacer.ReplaceValue, {"ClienteId"})
```

### Tipos nullable vs non-nullable

Ao tipar colunas, você pode especificar se aceitam null:

```m
// Aceita null (padrão do PQ na maioria dos casos)
Table.TransformColumnTypes(Source, {{"Valor", type nullable number}})

// NÃO aceita null (a query vai falhar se algum null aparecer)
Table.TransformColumnTypes(Source, {{"Valor", type number}})
```

Usar `non-nullable` em colunas críticas é uma forma de "contratar" o dado: se o valor vier null, a query falha explicitamente em vez de propagar silenciosamente.

---

## 4. Tratamento de erros (try/otherwise)

### Erros em M

Diferente de null, erro é um estado distinto. Acontece em:
- Divisão por zero
- Conversão de tipo falha (`"abc"` → Int64.Type)
- Acesso a campo inexistente
- Parse de data mal-formatada

### `try / otherwise`

Padrão para capturar erro e retornar valor default:

```m
resultadoSeguro = try expressaoQuePodeFalhar otherwise valorDefault
```

**Exemplo — divisão segura:**
```m
Table.AddColumn(Source, "Margem", each 
    try [Lucro] / [Receita] otherwise null,
    type number
)
```

**Exemplo — conversão segura:**
```m
Table.AddColumn(Source, "CpfNumerico", each 
    try Number.FromText(Text.Select([Cpf], {"0".."9"})) otherwise null,
    Int64.Type
)
```

### `try` sem `otherwise`

Retorna um record com campos `HasError` e `Value`/`Error`:

```m
tentativa = try risky(x),
// tentativa = [HasError = true, Error = [Reason = "...", Message = "...", Detail = ...]]
// ou
// tentativa = [HasError = false, Value = resultado]

resultadoFinal = if tentativa[HasError] then "Falhou" else tentativa[Value]
```

Útil quando você quer reagir ao tipo de erro específico, não só "falhou → default".

### Replace Errors (na tabela)

Para substituir erros em uma coluna inteira:

```m
Table.ReplaceErrorValues(Source, {{"ColunaRisco", "Erro"}})
```

**⚠️ Cuidado:** `Table.ReplaceErrorValues` tende a quebrar Query Folding. Em fontes SQL, prefira tratar o erro com `try/otherwise` em step antes ou fazer cast seguro no SQL fonte.

### Padrão Dashmaker: fallback + log

Para operações críticas, combine try/otherwise com logging:

```m
conversaoCpf = Table.AddColumn(Source, "CpfLimpo", each
    let
        __tentativa = try fxLimpaCpf([CpfOriginal])
    in
        if __tentativa[HasError] then null
        else __tentativa[Value],
    type nullable text
),

// Opcional: coluna com flag de erro para auditoria
flagErroCpf = Table.AddColumn(conversaoCpf, "TeveErroCpf", each
    [CpfLimpo] = null and [CpfOriginal] <> null,
    type logical
)
```

A coluna `TeveErroCpf` permite fazer visual no Power BI de "linhas com problema" sem travar o modelo.

---

## 5. Validação de dados

### Validação em linha

Padrões para validar dados críticos durante a carga:

```m
// Validar CPF (11 dígitos)
validaCpf = Table.AddColumn(Source, "CpfValido", each
    if [Cpf] = null then false
    else Text.Length(Text.Select([Cpf], {"0".."9"})) = 11,
    type logical
)

// Validar email (regex simples)
validaEmail = Table.AddColumn(Source, "EmailValido", each
    if [Email] = null then false
    else Text.Contains([Email], "@") and Text.Contains([Email], "."),
    type logical
)

// Validar range de valor
validaValor = Table.AddColumn(Source, "ValorSuspeito", each
    [Valor] > 1000000 or [Valor] < 0,
    type logical
)
```

### Validação de schema

Cenário: a fonte pode mudar (coluna nova, coluna removida, rename). Para detectar antes de quebrar o modelo:

```m
// Validar que colunas esperadas existem
colunasEsperadas = {"VendaId", "ClienteId", "Data", "Valor"},
colunasReais = Table.ColumnNames(Source),
faltantes = List.Difference(colunasEsperadas, colunasReais),

// Se houver faltantes, gere erro explícito
validacao = if List.Count(faltantes) > 0 
            then error "Colunas faltantes: " & Text.Combine(faltantes, ", ")
            else Source
```

Coloque essa validação logo após o Source. Se a fonte mudar, a query falha com mensagem clara em vez de propagar bugs silenciosos.

### Validação de tipos

```m
// Verificar se coluna de data contém apenas datas válidas
validaData = Table.AddColumn(Source, "DataValida", each
    try Date.From([DataTexto]) = null false otherwise false,
    type logical
)
```

Ou mais direto — quebra com erro se houver data inválida:

```m
// Força conversão estrita (se falhar, a query quebra — ok em alguns casos)
Table.TransformColumnTypes(Source, {{"Data", type date}})
```

### Qualidade via Power Query Data Profiling

No Power Query Editor: View → **Column quality / Column distribution / Column profile**. Mostra no cabeçalho das colunas:
- % de valores válidos / errados / vazios
- Valores distintos / únicos
- Perfil de distribuição

Útil para auditoria inicial de uma nova fonte, mas não substitui validação automatizada no código.

---

## 6. Normalização de strings

### Trimming (remover espaços)

```m
Table.TransformColumns(Source, {{"Nome", Text.Trim, type text}})
```

### Casing (maiúsculas/minúsculas)

```m
// Tudo maiúsculo
Table.TransformColumns(Source, {{"Uf", Text.Upper, type text}})

// Tudo minúsculo
Table.TransformColumns(Source, {{"Email", Text.Lower, type text}})

// Primeira letra de cada palavra maiúscula
Table.TransformColumns(Source, {{"NomeCliente", Text.Proper, type text}})
```

### Remover acentos

Power Query não tem função nativa. Crie `fxRemoveAcentos`:

```m
// fxRemoveAcentos: remove acentuação de uma string
(texto as nullable text) as nullable text =>
let
    __mapa = {
        {"á", "a"}, {"à", "a"}, {"â", "a"}, {"ã", "a"}, {"ä", "a"},
        {"é", "e"}, {"è", "e"}, {"ê", "e"}, {"ë", "e"},
        {"í", "i"}, {"ì", "i"}, {"î", "i"}, {"ï", "i"},
        {"ó", "o"}, {"ò", "o"}, {"ô", "o"}, {"õ", "o"}, {"ö", "o"},
        {"ú", "u"}, {"ù", "u"}, {"û", "u"}, {"ü", "u"},
        {"ç", "c"}, {"ñ", "n"},
        {"Á", "A"}, {"À", "A"}, {"Â", "A"}, {"Ã", "A"}, {"Ä", "A"},
        {"É", "E"}, {"È", "E"}, {"Ê", "E"}, {"Ë", "E"},
        {"Í", "I"}, {"Ì", "I"}, {"Î", "I"}, {"Ï", "I"},
        {"Ó", "O"}, {"Ò", "O"}, {"Ô", "O"}, {"Õ", "O"}, {"Ö", "O"},
        {"Ú", "U"}, {"Ù", "U"}, {"Û", "U"}, {"Ü", "U"},
        {"Ç", "C"}, {"Ñ", "N"}
    },
    __substituido = if texto = null then null
                    else List.Accumulate(__mapa, texto, (acc, par) => Text.Replace(acc, par{0}, par{1}))
in
    __substituido
```

### Padrão "chave normalizada"

Para merges entre tabelas onde a chave pode ter variação de espaços/case/acentos:

```m
// Antes do merge, criar coluna normalizada nas duas tabelas
Table.AddColumn(fVendas, "ChaveNorm", each 
    fxRemoveAcentos(Text.Upper(Text.Trim([Cliente]))),
    type text
)

Table.AddColumn(dCliente, "ChaveNorm", each 
    fxRemoveAcentos(Text.Upper(Text.Trim([Nome]))),
    type text
)

// Merge usando a chave normalizada
merge = Table.NestedJoin(fVendasNorm, {"ChaveNorm"}, dClienteNorm, {"ChaveNorm"}, ...)
```

### Limpeza de códigos (CEP, CPF, CNPJ, telefone)

```m
// Remove tudo que não é dígito
Text.Select([Cpf], {"0".."9"})

// Padroniza com leading zeros
Text.PadStart(Text.Select([Cep], {"0".."9"}), 8, "0")
```

---

## 7. Padrões de robustez

### Usar tipos explícitos em `Table.AddColumn`

Sem tipo, a coluna fica tipo Any — folding quebra e o motor precisa inferir em cada linha.

```m
// ❌ Sem tipo
Table.AddColumn(Source, "Ano", each Date.Year([Data]))

// ✅ Com tipo
Table.AddColumn(Source, "Ano", each Date.Year([Data]), Int64.Type)
```

### Verificar cardinalidade após merge

Como mencionado em performance, toda merge tem risco de duplicar linhas. Adicione validação:

```m
__linhasAntes = Table.RowCount(Source),
merge = Table.NestedJoin(Source, ..., ..., ..., "join", JoinKind.LeftOuter),
expand = Table.ExpandTableColumn(merge, "join", {...}),
__linhasDepois = Table.RowCount(expand),

// Falha explicitamente se houve duplicação
validacao = if __linhasDepois <> __linhasAntes 
            then error "Merge duplicou linhas: " & Text.From(__linhasAntes) & " → " & Text.From(__linhasDepois)
            else expand
```

Para produção, considere remover a validação (para não impactar performance) ou deixar só em queries críticas.

### `Table.Schema` — inspecionar estrutura programaticamente

```m
// Retorna tabela com info sobre cada coluna (nome, tipo, nullable, etc.)
esquema = Table.Schema(Source)
```

Útil para queries de auditoria ou documentação automatizada.

### Date.From(DateTime.LocalNow()) vs DateTime.LocalNow()

**Cuidado com timezone.** `DateTime.LocalNow()` retorna timezone do servidor que roda o refresh:
- No Power BI Desktop: seu timezone local
- No Power BI Service: UTC (geralmente)
- No Gateway: timezone do servidor do gateway

Para consistência, prefira:

```m
// Data atual (sem hora) considerando UTC
dataAtualUtc = Date.From(DateTime.FromFileTime(DateTime.ToText(DateTime.UtcNow())))

// Ou converter para timezone específico
dataAtualBr = Date.From(DateTimeZone.SwitchZone(DateTimeZone.UtcNow(), -3))  // UTC-3 (BR)
```

---

## Checklist rápido de qualidade e segurança

- [ ] Privacy Levels definidos em todas as fontes (nenhuma como "None" em produção)
- [ ] Credenciais NUNCA hard-coded no M (usar Credential Manager, Gateway ou OAuth)
- [ ] Tokens de API em parâmetros (mesmo assim, idealmente via proxy)
- [ ] Colunas críticas filtram nulls explicitamente quando apropriado
- [ ] Operações que podem falhar protegidas com `try/otherwise`
- [ ] Validação de schema na entrada (colunas esperadas existem)
- [ ] Merges validam cardinalidade (antes = depois em 1:1, ou validação intencional de 1:N)
- [ ] Strings críticas normalizadas (trim, upper, sem acentos)
- [ ] Tipos explícitos em `Table.AddColumn` e outros steps
- [ ] Timezone considerado em campos de "agora" / "atual"
