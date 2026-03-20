# CLAUDE.md — Contexto do Projeto para Claude Code

<!--
  Este arquivo é carregado AUTOMATICAMENTE pelo Claude Code em toda conversa.
  Escreva aqui o contexto que qualquer IA precisa saber antes de tocar no código.

  Regras:
  - Seja objetivo. Cada linha ocupa contexto do LLM.
  - Prefira fatos ("usamos Prisma") a intenções ("pretendemos usar...").
  - Atualize sempre que a arquitetura mudar.
-->

## Identidade do projeto

**Nome:** [Nome do Projeto]
**Domínio:** [Ex: Gestão de frotas / E-commerce / SaaS B2B]
**Tipo:** Multi-tenant SaaS
**Stage:** [Ex: MVP / Beta / Produção]

## Arquitetura em uma linha

```
[Cliente Web/Mobile] → [API Gateway] → [Serviços] → [PostgreSQL + Redis]
```

Detalhes em [docs/architecture/overview.md](docs/architecture/overview.md).

## Fluxo de desenvolvimento (TDD obrigatório)

**Red → Green → Refactor. Sempre.**
1. Escreva o teste que descreve o comportamento esperado
2. Rode e confirme que FALHA (`pnpm test:watch`)
3. Implemente o mínimo para passar
4. Rode e confirme que PASSA
5. Refatore se necessário

Nunca gere código de implementação sem um teste falhando antes.

## Convenções de código

- **Linguagem:** TypeScript strict mode
- **Formatação:** Prettier (`.prettierrc`)
- **Lint:** ESLint (`eslint.config.ts`)
- **Testes:** Vitest — arquivos `*.test.ts` ao lado do módulo
- **Commits:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- **Branches:** `feat/<ticket>-descricao`, `fix/<ticket>-descricao`

## Regras que NUNCA devem ser quebradas

1. Nunca commitar segredos ou `.env` com valores reais.
2. Toda alteração em schema de banco exige migration (nunca editar diretamente).
3. Endpoints públicos exigem autenticação, sem exceção.
4. Multi-tenancy: toda query ao banco deve filtrar por `tenantId`.

## O que a IA deve evitar

- Gerar código sem considerar o `tenantId` em queries.
- Sugerir `any` no TypeScript.
- Criar arquivos fora da estrutura de pastas definida em [docs/architecture/overview.md](docs/architecture/overview.md).
- Remover tratamento de erro existente.

## Arquivos-chave para entender o sistema

| Arquivo                              | Por que é importante                       |
|--------------------------------------|--------------------------------------------|
| `src/lib/prisma.ts`                  | Cliente de banco com middleware de tenant  |
| `src/lib/auth.ts`                    | Lógica central de autenticação             |
| `src/middleware/tenant.ts`           | Extração e validação do tenant na request  |
| `docs/domain/business-rules.md`      | Regras de negócio que guiam o código       |
| `docs/ai-context/constraints.md`     | Restrições específicas para geração de IA  |

## Contexto adicional

- [Regras de negócio](docs/domain/business-rules.md)
- [Glossário do domínio](docs/domain/glossary.md)
- [Restrições para IA](docs/ai-context/constraints.md)
- [Mapa do codebase](docs/ai-context/codebase-map.md)
