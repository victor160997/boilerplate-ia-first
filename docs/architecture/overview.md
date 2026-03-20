# Visão Geral da Arquitetura

<!--
  PROPÓSITO: Dar a qualquer pessoa (ou IA) uma visão completa do sistema
  em menos de 10 minutos de leitura.

  QUANDO ATUALIZAR: Sempre que mudar um componente, fluxo principal,
  tecnologia-chave ou padrão arquitetural.

  AUDIÊNCIA: Devs novos no projeto, agentes de IA, revisores de PR.
-->

## Diagrama de alto nível

```
                        ┌─────────────────────────────────┐
                        │           Clientes               │
                        │  Web App  │  Mobile  │  API ext  │
                        └─────────────┬───────────────────┘
                                      │ HTTPS
                        ┌─────────────▼───────────────────┐
                        │         API Gateway              │
                        │   (auth, rate limit, routing)    │
                        └──┬──────────────────────────┬───┘
                           │                          │
              ┌────────────▼──────┐      ┌────────────▼──────┐
              │   Serviço Core    │      │  Serviço de IA    │
              │  (CRUD, negócio)  │      │  (Claude API)     │
              └────────┬──────────┘      └───────────────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
   ┌──────▼──┐  ┌──────▼──┐  ┌────▼────┐
   │PostgreSQL│  │  Redis  │  │  S3/CDN │
   │(dados)   │  │(cache/  │  │(arquivos│
   │          │  │ filas)  │  │)        │
   └──────────┘  └─────────┘  └─────────┘
```

## Componentes principais

### API Gateway
- Responsável por: autenticação JWT, rate limiting, roteamento
- Tecnologia: [ex: Nginx / Kong / Traefik]
- Não contém lógica de negócio

### Serviço Core
- Responsável por: todas as regras de negócio, CRUD, integrações
- Estrutura interna: arquitetura em camadas (Controller → Service → Repository)
- Multi-tenant: todo acesso ao banco filtra por `tenantId`

### Serviço de IA
- Responsável por: orquestrar chamadas ao Claude API, manter histórico de conversas
- Padrão: system prompt fixo + contexto dinâmico por tenant
- Ver: [docs/ai-context/system-prompt.md](../ai-context/system-prompt.md)

## Padrões arquiteturais

### Multi-tenancy
Isolamento por `tenantId` em row-level. Cada request carrega o tenant
extraído do JWT. Middleware valida e injeta no contexto.

```typescript
// Exemplo: toda query DEVE receber tenantId
const users = await db.user.findMany({
  where: { tenantId: ctx.tenant.id }
})
```

### Autenticação
JWT de curta duração (15min) + refresh token (7 dias).
Decisão completa em [decisions/adr-001-auth-strategy.md](decisions/adr-001-auth-strategy.md).

### Tratamento de erros
Erros de domínio são classes tipadas (`DomainError`, `NotFoundError`, etc.).
Nunca expor stack trace em produção.

## Estrutura de pastas do código

```
src/
├── controllers/     # Recebe request, valida input, chama service
├── services/        # Lógica de negócio pura, sem HTTP
├── repositories/    # Acesso ao banco, sempre com tenantId
├── domain/          # Entidades, value objects, erros de domínio
├── middleware/       # Auth, tenant, logging
├── lib/             # Clientes externos (Prisma, Redis, Claude)
└── utils/           # Funções puras sem efeito colateral
```

## Decisões arquiteturais (ADRs)

- [ADR-001: Estratégia de Autenticação](decisions/adr-001-auth-strategy.md)
- [ADR-002: Abordagem Multi-tenant](decisions/adr-002-multi-tenant.md)
