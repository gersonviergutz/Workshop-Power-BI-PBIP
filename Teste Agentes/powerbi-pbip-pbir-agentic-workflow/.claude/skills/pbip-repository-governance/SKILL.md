---
name: pbip-repository-governance
description: Skill para governança de repositórios Power BI Developer Mode, PBIP, PBIR e TMDL com controle de versão e segurança de alterações.
---

# Skill: PBIP Repository Governance

## Quando usar
Use em qualquer agente que leia ou modifique arquivos de projeto Power BI em modo Developer.

## Objetivo
Evitar alterações destrutivas em arquivos PBIP/PBIR/TMDL e manter o projeto versionável, rastreável e seguro.

## Estrutura esperada
Um projeto PBIP normalmente contém:

```text
projeto.pbip
report/
semanticModel/
```

O relatório pode usar PBIR e o modelo semântico pode usar TMDL. Não presumir uma estrutura única sem verificar os arquivos reais.

## Regras gerais
- Nunca alterar arquivos binários.
- Nunca reformatar JSON/TMDL inteiro sem necessidade.
- Preferir mudanças pequenas e localizadas.
- Antes de renomear objeto, procurar dependências no repositório.
- Antes de excluir objeto, listar impacto.
- Preservar encoding, indentação e estrutura existente.
- Não inserir segredos, tokens, senhas ou credenciais.
- Não alterar conexão de fonte de dados sem registrar risco.

## Arquivos que exigem cuidado extra
- `*.pbip`
- `definition.pbism`
- `definition.pbir`
- `*.tmdl`
- arquivos JSON de páginas, visuais, bookmarks e filtros.

## Checklist antes de alteração
1. Qual arquivo será alterado?
2. O arquivo é gerado pelo Power BI Desktop?
3. A alteração é estável para controle de versão?
4. Existe dependência em DAX, relacionamento ou visual?
5. A alteração deveria ser feita no Power BI Desktop em vez de texto?
6. A alteração tem reversão clara?

## Validação mínima
Após qualquer alteração textual:
- Verificar sintaxe básica.
- Procurar referências quebradas.
- Verificar se nomes alterados ainda aparecem em medidas/visuais.
- Registrar no `CHANGELOG.md`.

## Política de segurança
- Não expor caminhos locais sensíveis.
- Não versionar credenciais.
- Não criar arquivos com dados reais sensíveis.
- Para exemplos, usar dados fictícios.
