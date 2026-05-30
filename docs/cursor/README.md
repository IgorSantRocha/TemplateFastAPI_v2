# Cursor Rules - TemplateFastAPI

Esta pasta documenta a configuração sugerida do Cursor para o TemplateFastAPI.

## Estrutura criada

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
    ├── 100-task-workflow-agent.mdc
    └── 110-python-style-auto.mdc

AGENTS.md
.cursorrules
```

## Como usar

Copie os arquivos para a raiz do seu projeto FastAPI.

Depois, no Cursor:

1. Abra o projeto pela raiz correta.
2. Confirme que existe `.cursor/rules/` na raiz.
3. Abra um novo chat/agent.
4. Peça algo como: `Leia as rules do projeto e ajuste o endpoint de cars seguindo o padrão`.

## Tipos de rules usadas

### Always Apply

Usadas para regras globais, carregadas em toda conversa quando o Cursor aplica esse modo corretamente.

Arquivos:

- `000-project-context-always.mdc`
- `010-architecture-always.mdc`
- `080-security-config-always.mdc`

### Auto Attached por glob

Usadas quando arquivos específicos entram no contexto.

Exemplos:

- `api/**/*.py` para endpoints.
- `services/**/*.py` para services.
- `crud/**/*.py` para CRUD.
- `tests/**/*.py` para testes.

### Agent Requested

Usada para tarefas maiores, quando o agente deve aplicar workflow de implementação/refatoração.

Arquivo:

- `100-task-workflow-agent.mdc`

## Regras principais do template

O padrão arquitetural obrigatório é:

```text
Endpoint -> Service -> CRUD -> Banco
Endpoint -> Service -> Core Client -> API externa
```

## Cuidados

- Não deixe rules gigantes demais.
- Não coloque segredo em rules.
- Não duplique conteúdo dos READMEs inteiros dentro das rules.
- Prefira rules pequenas, objetivas e específicas por pasta.
- Se o Cursor não aplicar uma rule automaticamente, mencione o arquivo com `@nome-da-rule.mdc` no chat.
