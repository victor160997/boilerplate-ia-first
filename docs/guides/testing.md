# Guia de Testes

<!--
  PROPÓSITO: Definir a estratégia de testes do projeto — o quê testar,
  como escrever, como rodar e o que é cobertura suficiente.

  FILOSOFIA: Testes devem dar confiança para refatorar e liberar.
  Não testamos implementação, testamos comportamento.

  AUDIÊNCIA: Devs e agentes de IA que vão escrever testes.
-->

## Pirâmide de testes

```
        /\
       /  \  E2E (poucos)
      /────\  Testa fluxos completos do usuário
     /      \
    /────────\  Integração (moderados)
   /          \  Testa módulos com suas dependências reais
  /────────────\
 /              \  Unitários (maioria)
/────────────────\  Testa funções/classes de forma isolada
```

## Tipos de teste e quando usar

### Testes unitários (`*.test.ts`)
- **O quê:** services, domain logic, utils, value objects
- **Dependências:** sempre mockadas (vi.fn())
- **Velocidade:** milissegundos
- **Localização:** ao lado do arquivo testado

```typescript
// src/services/plan-guard.service.test.ts
import { describe, it, expect, vi } from 'vitest'
import { PlanGuardService } from './plan-guard.service'
import { PlanLimitError } from '../domain/errors'

describe('PlanGuardService', () => {
  it('should throw PlanLimitError when user limit is reached', async () => {
    // Arrange
    const tenantRepo = {
      getPlanUsage: vi.fn().mockResolvedValue({ users: 5, plan: { maxUsers: 5 } })
    }
    const guard = new PlanGuardService(tenantRepo as any)

    // Act + Assert
    await expect(guard.assertCanAddUser('tenant-1'))
      .rejects
      .toThrow(PlanLimitError)
  })

  it('should pass when user limit is not reached', async () => {
    // Arrange
    const tenantRepo = {
      getPlanUsage: vi.fn().mockResolvedValue({ users: 3, plan: { maxUsers: 5 } })
    }
    const guard = new PlanGuardService(tenantRepo as any)

    // Act + Assert — não lança erro
    await expect(guard.assertCanAddUser('tenant-1')).resolves.toBeUndefined()
  })
})
```

### Testes de integração (`*.integration.test.ts`)
- **O quê:** repositories, endpoints HTTP, fluxos multi-módulo
- **Dependências:** banco real (via Docker), Redis real
- **Velocidade:** segundos
- **Localização:** `src/__tests__/integration/`

```typescript
// src/__tests__/integration/user.integration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { createTestDb, cleanDb } from '../helpers/db'
import { UserRepository } from '../../repositories/user.repository'

describe('UserRepository (integration)', () => {
  const db = createTestDb()

  beforeAll(() => db.connect())
  afterAll(() => db.disconnect())

  it('should isolate users by tenant', async () => {
    await cleanDb(db)
    const repo = new UserRepository(db)

    // Cria usuário no tenant A
    await repo.create({ email: 'a@test.com', name: 'A', tenantId: 'tenant-a' })

    // Busca no tenant B — não deve encontrar
    const result = await repo.findByEmail('a@test.com', 'tenant-b')
    expect(result).toBeNull()
  })
})
```

### Testes E2E (`*.e2e.test.ts`)
- **O quê:** fluxos críticos completos (login → ação → verificação)
- **Dependências:** servidor completo rodando
- **Velocidade:** segundos a minutos
- **Localização:** `src/__tests__/e2e/`

## Comandos

```bash
pnpm test                    # Roda todos os testes (unit + integration)
pnpm test:watch              # Modo watch — ideal para TDD
pnpm test:coverage           # Gera relatório de cobertura
pnpm test:e2e                # Roda apenas testes E2E
pnpm test src/services/user  # Roda apenas testes de um módulo
```

## Metas de cobertura

| Camada      | Cobertura mínima | Observação                         |
|-------------|------------------|------------------------------------|
| Services    | 90%              | Lógica de negócio — crítico        |
| Domain      | 95%              | Erros, entidades, value objects    |
| Repositories| 80%              | Cobertos por integração            |
| Controllers | 70%              | Cobertos por E2E                   |
| Utils       | 85%              | Funções puras — fáceis de testar   |

## Boas práticas

### Nomeação de testes (AAA)
```typescript
it('should [resultado esperado] when [condição]', async () => {
  // Arrange — configure o cenário
  // Act     — execute a ação
  // Assert  — verifique o resultado
})
```

### O que NÃO testar
- Código de terceiros (Prisma, bibliotecas)
- Configurações (apenas se tiver lógica)
- Getters/setters simples sem lógica

### Dados de teste
Use factories em vez de fixtures estáticas:
```typescript
// src/__tests__/factories/user.factory.ts
export const makeUser = (overrides = {}) => ({
  id: 'user-1',
  email: 'test@test.com',
  name: 'Test User',
  tenantId: 'tenant-1',
  role: 'MEMBER' as const,
  ...overrides
})
```

## Referência de regras de negócio

Ao escrever um teste que implementa uma regra, referencie o ID:
```typescript
// RN-001: unicidade de email por tenant
it('should throw when email already exists in same tenant', ...)

// RN-010: limite de usuários por plano
it('should throw PlanLimitError when free plan user limit is reached', ...)
```
