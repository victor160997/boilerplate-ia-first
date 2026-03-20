# ADR-002: Abordagem Multi-tenant

<!--
  Registra a decisão sobre como isolar dados entre tenants.
  Esta é uma das decisões mais críticas do sistema — mudar depois é caro.
-->

## Status

`Aprovado` — 2026-03-20

## Contexto

O sistema atende múltiplas organizações (tenants) que não devem ver dados
umas das outras. Precisamos escolher uma estratégia de isolamento que equilibre
segurança, custo operacional e complexidade de desenvolvimento.

## Opções consideradas

### Opção A: Banco de dados separado por tenant
- **Prós:** Isolamento máximo, backup/restore independente, compliance mais simples
- **Contras:** Alto custo operacional, difícil de gerenciar em escala,
  migrações complexas (N bancos para migrar)

### Opção B: Schema separado por tenant (PostgreSQL schemas)
- **Prós:** Bom isolamento, migrações por schema
- **Contras:** Suporte limitado em ORMs (Prisma tem limitações),
  conexão pooling complexo

### Opção C: Row-level isolation com tenantId (escolhida)
- **Prós:** Simples de implementar, suporte nativo em ORMs, fácil de migrar,
  baixo custo operacional
- **Contras:** Risco de vazamento de dados se queries incorretas (mitigado
  por middleware e convenção)

## Decisão

**Opção C: Row-level isolation com `tenantId`**

Todas as tabelas de dados de negócio possuem coluna `tenantId` (UUID).
Um middleware extrai o `tenantId` do JWT e o injeta no contexto da request.
Todos os repositories DEVEM receber e aplicar o `tenantId` como filtro obrigatório.

### Schema padrão

```sql
-- Toda tabela de negócio segue este padrão
CREATE TABLE "User" (
  "id"        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  "tenantId"  UUID NOT NULL REFERENCES "Tenant"("id"),
  "email"     TEXT NOT NULL,
  "createdAt" TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Unicidade SEMPRE inclui tenantId
  UNIQUE ("tenantId", "email")
);

-- Índice obrigatório em tenantId para performance
CREATE INDEX "User_tenantId_idx" ON "User"("tenantId");
```

### Padrão de repository

```typescript
// ✅ CORRETO — sempre filtrar por tenantId
class UserRepository {
  async findAll(tenantId: string) {
    return this.db.user.findMany({
      where: { tenantId }
    })
  }
}

// ❌ ERRADO — jamais buscar sem tenantId
class UserRepository {
  async findAll() {
    return this.db.user.findMany() // NUNCA faça isso
  }
}
```

### Tabelas sem tenantId (globais)

Apenas tabelas de sistema não possuem `tenantId`:
- `Tenant` — o próprio registro do tenant
- `Plan` — planos de cobrança
- `Migration` — controle de migrações

## Consequências

- **Positivas:** Operação simples (único banco), migrações unificadas,
  custo baixo
- **Negativas:** Desenvolvedores precisam estar cientes do padrão e nunca
  esquecer o filtro — mitigado por lint rule e code review
- **Ações necessárias:**
  - Criar middleware `src/middleware/tenant.ts`
  - Adicionar ESLint rule para detectar queries Prisma sem `tenantId`
  - Documentar padrão no guia de desenvolvimento
  - Adicionar verificação no template de PR
