# AGENTS.md - TemplateFastAPI

Este projeto usa Cursor Rules em `.cursor/rules/`.

## Regra principal

Siga sempre este fluxo:

```text
Endpoint -> Service -> CRUD -> Banco
Endpoint -> Service -> Core Client -> API externa
```

## Prioridades

1. Preserve o padrão atual do projeto.
2. Não coloque regra de negócio em endpoints.
3. Não coloque chamada HTTP externa em endpoints.
4. Não coloque regra de negócio em CRUD.
5. Use `services/` para regras, validações e orquestração.
6. Use `core/` para clients externos, autenticação, OAuth2, API keys e certificados.
7. Use `tests/` para exemplos e validação de comportamento.
8. Atualize README quando mudar estrutura ou padrão.

## Segurança

Nunca exponha tokens, senhas, API keys, secrets, certificados reais ou `.env` real.
