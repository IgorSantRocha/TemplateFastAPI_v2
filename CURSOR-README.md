# Cursor Rules para projetos FastAPI

Este repositório contém um conjunto de **Cursor Rules** para padronizar o desenvolvimento de projetos FastAPI, especialmente projetos baseados no padrão do `TemplateFastAPI`.

A ideia é permitir que o time copie estas regras para projetos novos ou já existentes e faça o Cursor seguir uma arquitetura consistente.

## Documentação detalhada

Para entender como aplicar estas rules em um projeto já existente, leia:

[Guia detalhado de aplicação em projetos existentes](./docs/cursor/README.md)

Esse guia explica como copiar os arquivos, como adaptar projetos legados, como migrar endpoints aos poucos para `services/` e como manter a documentação atualizada conforme o projeto evolui.

## Histórico de alterações nos projetos que usarem estas rules

Além de manter o README principal atualizado, estas rules orientam o Cursor a criar ou atualizar uma seção de **Alterações recentes** dentro do projeto de destino.

Essa seção funciona como um resumo simples de release notes, para que o time consiga entender rapidamente o que mudou no projeto sem precisar analisar commits ou procurar em várias pastas.

Formato recomendado no README do projeto de destino:

```md
## Alterações recentes

| Data | Tipo | Módulo/Pasta | Alteração | Impacto |
| ---- | ---- | ------------ | --------- | ------- |
| 2026-05-30 | Adicionado | `services/` | Criada camada de services para regras de negócio. | Endpoints passam a chamar services antes do CRUD. |
| 2026-05-30 | Adicionado | `tests/` | Criada estrutura inicial de testes. | Projeto passa a ter testes por camada. |
```

Essa orientação está definida principalmente na rule:

```text
.cursor/rules/095-project-readme-sync-auto.mdc
```

Ela também indica quando uma alteração pede novas pastas no template, como `services/`, `tests/`, `docs/`, `examples/` ou uma pasta específica de integração. Quando uma nova pasta tiver responsabilidade própria, o Cursor deve avaliar se ela também precisa de um `README.md` interno.

## Estrutura do pacote

```text
.cursor/
└── rules/
    ├── 000-project-context-always.mdc
    ├── 010-architecture-always.mdc
    ├── 020-fastapi-endpoints-auto.mdc
    ├── 030-services-auto.mdc
    ├── 040-crud-auto.mdc
    ├── 050-schemas-models-auto.mdc
    ├── 060-core-integrations-auto.mdc
    ├── 070-tests-auto.mdc
    ├── 080-security-config-always.mdc
    ├── 090-docs-readme-auto.mdc
    ├── 095-project-readme-sync-auto.mdc
    ├── 100-task-workflow-agent.mdc
    └── 110-python-style-auto.mdc

AGENTS.md
.cursorrules
docs/
└── cursor/
    └── README.md
```

## Como instalar em um projeto

Copie os arquivos deste repositório para a raiz do projeto FastAPI.

Exemplo:

```bash
cp -r .cursor AGENTS.md .cursorrules docs /caminho/do/projeto/
```

A estrutura no projeto de destino deve ficar parecida com:

```text
meu-projeto-fastapi/
├── .cursor/
│   └── rules/
├── AGENTS.md
├── .cursorrules
├── docs/
│   └── cursor/
├── api/
├── core/
├── crud/
├── db/
├── models/
├── schemas/
├── services/
├── tests/
└── README.md
```

O Cursor precisa abrir o projeto pela **raiz**, onde está a pasta `.cursor/`.

## Padrão arquitetural esperado

O fluxo principal do projeto deve ser:

```text
Endpoint -> Service -> CRUD -> Banco
Endpoint -> Service -> Core Client -> API externa
```

Responsabilidades por camada:

```text
api/       = rotas HTTP, Depends, Query, Path, response_model
services/  = regra de negócio, validações, orquestração e integrações
crud/      = acesso ao banco, filtros, paginação e operações persistentes
models/    = modelos SQLAlchemy
schemas/   = schemas Pydantic de entrada, atualização e resposta
core/      = configurações, segurança, clients externos e utilitários globais
db/        = sessão, base e conexão com banco
tests/     = testes automatizados por camada
```

## O que cada rule faz

### `000-project-context-always.mdc`

Contexto global do projeto. Define stack, arquitetura esperada e comportamento geral do Cursor.

Use para reforçar:

```text
FastAPI
SQLAlchemy
Pydantic
HTTPX
Pytest
Endpoint -> Service -> CRUD -> Banco
```

---

### `010-architecture-always.mdc`

Define a separação de responsabilidades entre as camadas.

Evita misturar:

```text
regra de negócio em endpoint
SQL direto em service
HTTP externo em endpoint
schema dentro de model
model dentro de schema
```

---

### `020-fastapi-endpoints-auto.mdc`

Aplicada em arquivos de endpoint.

Garante que endpoints sejam leves, contendo apenas:

```text
payload
Depends
Query/Path
response_model
chamada para service
retorno
```

---

### `030-services-auto.mdc`

Aplicada em `services/**/*.py`.

Define que services devem concentrar:

```text
validações
normalizações
regras de negócio
duplicidades
transições de status
chamadas para CRUDs
chamadas para clients externos em core/
```

---

### `040-crud-auto.mdc`

Aplicada em `crud/**/*.py`.

Define que CRUD deve ser responsável apenas por banco:

```text
get
get_multi
create
update
remove
filtros
paginação
ordenação
```

Não deve conter regra de negócio pesada.

---

### `050-schemas-models-auto.mdc`

Aplicada em `schemas/**/*.py` e `models/**/*.py`.

Padroniza a separação entre:

```text
Pydantic schemas
SQLAlchemy models
```

Sugere schemas no padrão:

```text
Base
Create
Update
Response/InDbBase
```

---

### `060-core-integrations-auto.mdc`

Aplicada em `core/**/*.py` e arquivos relacionados a integrações externas.

Define o padrão para consumo de APIs externas com:

```text
OAuth2
API Key
Bearer Token
Basic Auth
Client Secret
mTLS
certificados
headers extras
tratamento de erro externo
```

---

### `070-tests-auto.mdc`

Aplicada em `tests/**/*.py`.

Define padrão de testes com:

```text
pytest
pytest-asyncio
httpx AsyncClient
AsyncMock
fixtures em conftest.py
```

Separação recomendada:

```text
tests/services/
tests/api/api_v1/endpoints/
```

---

### `080-security-config-always.mdc`

Regra global de segurança.

Impede gerar ou versionar:

```text
senhas reais
tokens reais
api keys reais
client secrets reais
certificados reais
chaves privadas reais
.env com dados sensíveis
```

---

### `090-docs-readme-auto.mdc`

Aplicada em arquivos de documentação.

Define como escrever READMEs:

```text
README principal = visão geral
README de subpasta = detalhes específicos
sem redundância desnecessária
links relativos entre documentos
```

---

### `095-project-readme-sync-auto.mdc`

Aplicada em arquivos de código e documentação do projeto.

Essa rule orienta o Cursor a manter o `README.md` do projeto atualizado com informações reais depois que um módulo começar a ser desenvolvido.

Ela cobre casos como:

```text
novo módulo criado
novo endpoint criado
novo service criado
nova integração externa criada
novo fluxo de autenticação criado
mudança relevante na estrutura
adição de testes
mudança em variáveis de ambiente
```

A regra também impede documentação inventada: o Cursor deve documentar apenas o que existe no projeto ou o que acabou de ser implementado.

---

### `100-task-workflow-agent.mdc`

Regra para uso em tarefas maiores no Agent Mode.

Ajuda o Cursor a trabalhar em etapas:

```text
analisar estrutura
identificar camadas afetadas
criar/alterar arquivos necessários
atualizar testes
atualizar documentação
explicar o que mudou
```

---

### `110-python-style-auto.mdc`

Aplicada em arquivos Python.

Define padrões de estilo:

```text
tipagem
imports limpos
async/await correto
funções menores
tratamento de erro explícito
sem secrets hardcoded
sem pass silencioso
```

## Arquivos complementares

### `AGENTS.md`

Resumo global para agentes de código.

Fica na raiz do projeto de destino e serve como instrução geral para ferramentas que leem arquivos de contexto.

### `.cursorrules`

Fallback curto para compatibilidade com fluxos antigos.

O foco principal deve continuar sendo:

```text
.cursor/rules/*.mdc
```

### `docs/cursor/README.md`

Guia detalhado para aplicar as rules em projetos existentes.

Acesse:

[docs/cursor/README.md](./docs/cursor/README.md)

## Prompts úteis para usar no Cursor

```text
Leia as rules do projeto e analise se a estrutura atual segue o padrão Endpoint -> Service -> CRUD -> Banco. Não altere arquivos ainda. Apenas gere um plano de ajustes.
```

```text
Refatore este endpoint para mover a regra de negócio para services/, mantendo o CRUD apenas para acesso ao banco. Entregue os arquivos completos alterados.
```

```text
Crie um novo módulo seguindo as rules do projeto: model, schema, crud, service, endpoint, testes e documentação mínima.
```

```text
Após implementar este módulo, atualize o README.md principal com informações reais do projeto e adicione links para READMEs específicos quando necessário.
```

## Recomendação para o time

Use este repositório como base comum.

Para cada projeto novo ou existente:

1. Copie `.cursor/`, `AGENTS.md`, `.cursorrules` e `docs/`.
2. Abra o projeto no Cursor pela raiz.
3. Peça primeiro uma análise da estrutura atual.
4. Migre endpoints para `services/` gradualmente.
5. Atualize testes e README conforme os módulos forem evoluindo.
