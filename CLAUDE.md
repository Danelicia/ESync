## Agentes e skills

- Para qualquer pedido de código (mudança, bug, revisão, deploy, análise), use a skill `agent-router` antes de agir.
- Os 10 agentes ficam em `.claude/agents/`. Escolha o agente pelo domínio e o modelo pelo menor tier que resolve: ajuste visual e texto em `haiku`; feature e bug comum em `sonnet`; auth, permissão, migração e auditoria em `opus`.
- Mudança trivial não lê o projeto inteiro e não roda a suíte completa.
- Nunca commite segredos, `.env` ou credenciais.
- Não faça push nem abra PR sem pedido explícito.
- Mudança em auth, permissão, pagamento ou dado pessoal passa pelo `security-auditor`.
