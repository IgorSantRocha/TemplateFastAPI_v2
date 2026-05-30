# Patch de Cursor Rules para TemplateFastAPI

Este pacote adiciona uma estrutura de rules para configurar o Cursor no TemplateFastAPI.

## Arquivos incluídos

```text
.cursor/rules/*.mdc
AGENTS.md
.cursorrules
docs/cursor/README.md
```

## Como instalar

Copie tudo para a raiz do projeto.

Exemplo:

```bash
cp -r .cursor AGENTS.md .cursorrules docs /caminho/do/seu/projeto/
```

## Objetivo

Fazer o Cursor seguir seu padrão de código:

```text
Endpoint -> Service -> CRUD -> Banco
Endpoint -> Service -> Core Client -> API externa
```

## O que foi documentado nas rules

- Arquitetura do projeto.
- Regras para endpoints FastAPI.
- Regras para services.
- Regras para CRUD.
- Regras para schemas e models.
- Regras para core e integrações OAuth2/API Key/mTLS.
- Regras para testes.
- Regras de segurança.
- Regras para documentação README.
- Workflow de tarefas no Cursor.
- Estilo Python.

## Recomendação

Mantenha `.cursor/rules/` como fonte principal.

O arquivo `.cursorrules` foi incluído apenas como fallback curto para compatibilidade com projetos ou versões que ainda leem esse arquivo.
