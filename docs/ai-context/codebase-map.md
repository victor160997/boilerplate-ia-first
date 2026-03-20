# Mapa do Codebase

<!--
  PROPÓSITO: Guia de navegação do código para humanos e agentes de IA.
  Responde: "onde está X?" e "o que faz Y?".

  DIFERENÇA DE overview.md:
  - overview.md: arquitetura e decisões de design
  - Este arquivo: localização física de responsabilidades no código

  QUANDO ATUALIZAR: Sempre que um módulo importante for criado, movido
  ou tiver sua responsabilidade alterada.

  MANUTENÇÃO: Percorrer este arquivo vs. a estrutura real de pastas
  mensalmente para manter sincronizado.
-->

## Estrutura de pastas

```
src/
├── controllers/          # Handlers HTTP — input/output apenas
│   ├── auth.controller.ts
│   ├── user.controller.ts
│   └── tenant.controller.ts
│
├── services/             # Lógica de negócio — coração do sistema
│   ├── auth.service.ts
│   ├── user.service.ts
│   ├── tenant.service.ts
│   └── plan-guard.service.ts   # Verifica limites de plano (RN-010)
│
├── repositories/         # Acesso ao banco — sempre com tenantId
│   ├── user.repository.ts
│   └── tenant.repository.ts
│
├── domain/               # Tipos, entidades e erros do domínio
│   ├── errors.ts         # DomainError, NotFoundError, PlanLimitError...
│   ├── entities.ts       # Types das entidades de negócio
│   └── dtos.ts           # Input/Output schemas (Zod)
│
├── middleware/            # Express/Fastify middleware
│   ├── auth.ts           # Valida JWT, injeta user no ctx
│   ├── tenant.ts         # Extrai tenantId do JWT, injeta no ctx
│   └── error-handler.ts  # Trata erros de domínio → HTTP response
│
├── lib/                  # Clientes externos e infraestrutura
│   ├── prisma.ts         # Cliente Prisma com middleware de tenant
│   ├── redis.ts          # Cliente Redis
│   └── anthropic.ts      # Cliente Claude API
│
├── routes/               # Definição de rotas HTTP
│   ├── auth.routes.ts
│   ├── user.routes.ts
│   └── index.ts          # Agrega todas as rotas
│
├── utils/                # Funções puras sem efeito colateral
│   ├── jwt.ts            # assinar/verificar tokens
│   ├── crypto.ts         # hash de senha, tokens seguros
│   └── pagination.ts     # helpers de paginação
│
└── __tests__/            # Testes de integração e E2E
    ├── integration/
    ├── e2e/
    └── helpers/          # factories, db helpers, mocks reutilizáveis
```

## Onde está cada responsabilidade?

| Responsabilidade                        | Arquivo                                  |
|-----------------------------------------|------------------------------------------|
| Autenticação JWT (login, refresh)       | `src/services/auth.service.ts`           |
| Validação de token em cada request      | `src/middleware/auth.ts`                 |
| Extração de tenant do JWT               | `src/middleware/tenant.ts`               |
| Criação e gestão de usuários            | `src/services/user.service.ts`           |
| Verificação de limites de plano         | `src/services/plan-guard.service.ts`     |
| Queries ao banco (usuários)             | `src/repositories/user.repository.ts`   |
| Tratamento centralizado de erros HTTP   | `src/middleware/error-handler.ts`        |
| Erros de domínio tipados                | `src/domain/errors.ts`                  |
| Schemas de validação (Zod)              | `src/domain/dtos.ts`                    |
| Cliente Prisma com log e tenant         | `src/lib/prisma.ts`                     |
| Integração com Claude API               | `src/lib/anthropic.ts`                  |
| Hashing de senhas                       | `src/utils/crypto.ts`                   |

## Arquivos críticos — leia antes de tocar

### `src/lib/prisma.ts`
Cliente Prisma com middleware que loga queries lentas e pode injetar
`tenantId` automaticamente. **Qualquer mudança aqui afeta TODO o acesso
ao banco.**

### `src/middleware/tenant.ts`
Extrai e valida o `tenantId` de cada request. Se falhar, todas as queries
ficam sem filtro de tenant. **Criticidade: máxima.**

### `src/domain/errors.ts`
Define a hierarquia de erros. O `error-handler.ts` converte estas classes
em responses HTTP. Adicionar um erro aqui requer atualizar o handler.

## Fluxo de uma request típica

```
Request HTTP
    │
    ▼
auth.ts (middleware)     ← valida JWT, injeta req.user
    │
    ▼
tenant.ts (middleware)   ← extrai tenantId, injeta req.tenant
    │
    ▼
*.routes.ts              ← roteamento
    │
    ▼
*.controller.ts          ← valida body (Zod), chama service
    │
    ▼
*.service.ts             ← lógica de negócio, regras de domínio
    │
    ▼
*.repository.ts          ← query com tenantId obrigatório
    │
    ▼
Prisma → PostgreSQL
```
