---
name: agent-router
description: Roteia cada tarefa para um dos 10 agentes especialistas (testes, backend, frontend, segurança, performance, automação, devops, banco de dados, qualidade de código, docs/copy) e escolhe o modelo mais barato que dá conta. Use em qualquer pedido de mudança de código, revisão, bug, deploy ou análise.
---

# Agent Router

Você decide, antes de gastar tokens, **qual agente resolve** e **qual modelo ele usa**.
Objetivo: mudança simples (ex.: mover um botão, trocar um texto, ajustar uma cor) não pode consumir um modelo caro nem explorar o repositório inteiro.

## Os 10 agentes

Definidos em `.claude/agents/` (um arquivo por agente):

| Agente | Quando usar | Modelo padrão |
|---|---|---|
| `qa-tester` | testes, cobertura, reproduzir bug, validar fluxo | sonnet |
| `backend-engineer` | APIs, regras de negócio, serviços, integrações | sonnet |
| `frontend-engineer` | telas, componentes, layout, estado de UI | sonnet |
| `security-auditor` | auth, permissões, injeção, segredos, LGPD, dependências | opus |
| `performance-optimizer` | consultas lentas, bundle, cache, memória, latência | sonnet |
| `automation-engineer` | scripts, cron, workflows, integrações automatizadas | sonnet |
| `devops-engineer` | Docker, CI/CD, deploy, variáveis de ambiente, cloud | sonnet |
| `database-engineer` | schema, migrações, índices, queries | opus para migração, sonnet no resto |
| `code-quality` | refatoração, duplicação, legibilidade, revisão de diff | sonnet |
| `docs-copy` | README, comentários, textos de interface, docs | haiku |

## Passo a passo

1. **Classifique** a tarefa em um agente. Se ela cruza dois domínios, o agente dono da mudança principal resolve e o outro só revisa o trecho dele.
2. **Estime a complexidade** e escolha o modelo pelo menor tier que resolve:
   - **haiku**: ajuste visual pontual (posição, espaçamento, cor, tamanho), troca de texto, renomear, editar uma linha de config, docs.
   - **sonnet**: feature nova, bug com causa conhecida, teste novo, refatoração local, otimização com medição.
   - **opus**: causa desconhecida, mudança em auth/permissão/dados, migração, múltiplos módulos acoplados, auditoria de segurança.
3. **Delegue** com o Agent tool usando `subagent_type` igual ao nome do agente e, se quiser forçar o tier, o parâmetro `model` (`haiku`, `sonnet` ou `opus`). Passe no prompt: o objetivo, os arquivos conhecidos e o critério de pronto.
4. **Escale só se precisar.** Se o agente barato falhar ou o escopo crescer além do previsto, suba um tier e diga por quê. Não repita a mesma tentativa no mesmo modelo.
5. **Reporte** ao usuário: qual agente e modelo foram usados, o que mudou e o que foi verificado.

## Regras de economia de tokens

- Mudança trivial não lê o projeto inteiro: use `Grep` e leia só o trecho alvo.
- Não rode suíte completa para mudança de texto ou cor; rode o teste que cobre o arquivo.
- Não contrate `opus` para o que `haiku` faz. Opus é para risco, não para volume.
- Não reescreva arquivo inteiro para mexer em uma linha. Use edição pontual.
- Agrupe tarefas independentes em uma só rodada, em paralelo.
- Se a tarefa for só pergunta ("como funciona X"), responda direto, sem agente.

## Regras de segurança (valem para todos os agentes)

- Nunca imprima, commite ou copie segredos, tokens ou `.env`.
- Nenhum agente faz push, abre PR, apaga dados ou muda produção sem pedido explícito do usuário.
- Mudança em auth, permissão, pagamento ou dados pessoais passa pelo `security-auditor` antes de ser dada como pronta.

## Formato da resposta final

```
Agente: <nome> · Modelo: <haiku|sonnet|opus> (motivo em uma frase)
Mudança: <o que mudou, com arquivo:linha>
Verificação: <o que foi testado e o resultado real>
Riscos/pendências: <ou "nenhum">
```
