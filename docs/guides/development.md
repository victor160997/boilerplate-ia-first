# Guia de Desenvolvimento

<!--
  PROPÓSITO: Referência para o dia a dia de desenvolvimento.
  Padrões de código, fluxos, convenções e decisões que todo dev deve seguir.

  AUDIÊNCIA: Devs ativos no projeto e agentes de IA gerando código.
-->

## Fluxo TDD (obrigatório)

Este projeto segue **Test-Driven Development**. Toda implementação começa
com um teste falhando.

```bash
# 1. Escreva o teste
# 2. Rode e confirme que FALHA
pnpm test:watch src/services/user.service.test.ts

# 3. Implemente o mínimo para passar
# 4. Rode e confirme que PASSA
# 5. Refatore se necessário
```

> Para agentes de IA: **nunca escreva código de implementação antes do teste**.
> O fluxo é sempre: teste → falha confirmada → implementação → passou.

## Estrutura de um módulo

Cada feature é organizada em arquivos por responsabilidade:

```
src/
└── services/
    ├── user.service.ts          # Lógica de negócio
    ├── user.service.test.ts     # Testes do service (TDD)
    └── user.repository.ts       # Acesso ao banco
```

## Padrões de código

### Controllers
Responsabilidade única: receber request, validar schema, chamar service,
retornar response. **Sem lógica de negócio.**

```typescript
// ✅ Controller correto
export async function createUserController(req: Request, res: Response) {
  const body = createUserSchema.parse(req.body) // valida input
  const user = await userService.create(body, req.tenant.id) // delega ao service
  return res.status(201).json(user)
}
```

### Services
Contém toda a lógica de negócio. Não conhece HTTP. Recebe DTOs tipados,
retorna entidades de domínio ou erros tipados.

```typescript
// ✅ Service correto
export class UserService {
  async create(dto: CreateUserDto, tenantId: string): Promise<User> {
    // RN-001: verifica unicidade de email por tenant
    const exists = await this.repo.findByEmail(dto.email, tenantId)
    if (exists) throw new DomainError('EMAIL_ALREADY_EXISTS')

    // RN-010: verifica limite do plano
    await this.planGuard.assertCanAddUser(tenantId)

    return this.repo.create({ ...dto, tenantId })
  }
}
```

### Repositories
Acesso ao banco. Sempre recebe e filtra por `tenantId`. Nunca contém
lógica de negócio.

```typescript
// ✅ Repository correto — sempre com tenantId
export class UserRepository {
  async findByEmail(email: string, tenantId: string) {
    return this.db.user.findFirst({
      where: { email, tenantId } // tenantId OBRIGATÓRIO
    })
  }
}
```

### Erros de domínio
Use classes de erro tipadas. Nunca lance `new Error('string aleatória')`.

```typescript
// Defina em src/domain/errors.ts
export class DomainError extends Error {
  constructor(
    public readonly code: string,
    message?: string
  ) {
    super(message ?? code)
  }
}

export class NotFoundError extends DomainError {}
export class PlanLimitError extends DomainError {}
```

## Escrevendo testes

### Estrutura de um teste (TDD)

```typescript
// user.service.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { UserService } from './user.service'

describe('UserService', () => {
  describe('create', () => {
    it('should throw EMAIL_ALREADY_EXISTS when email is taken in tenant', async () => {
      // Arrange — configure o cenário
      const repo = { findByEmail: vi.fn().mockResolvedValue({ id: '1' }) }
      const service = new UserService(repo as any, planGuardMock)

      // Act + Assert — execute e verifique
      await expect(
        service.create({ email: 'joao@test.com', name: 'João' }, 'tenant-1')
      ).rejects.toThrow('EMAIL_ALREADY_EXISTS')
    })

    it('should create user when email is available', async () => {
      // Arrange
      const repo = {
        findByEmail: vi.fn().mockResolvedValue(null),
        create: vi.fn().mockResolvedValue({ id: 'new-id', email: 'joao@test.com' })
      }
      const service = new UserService(repo as any, planGuardMock)

      // Act
      const user = await service.create({ email: 'joao@test.com', name: 'João' }, 'tenant-1')

      // Assert
      expect(user.id).toBe('new-id')
      expect(repo.create).toHaveBeenCalledWith(
        expect.objectContaining({ tenantId: 'tenant-1' })
      )
    })
  })
})
```

## Convenções de nomenclatura

| Tipo         | Convenção       | Exemplo                    |
|--------------|-----------------|----------------------------|
| Arquivo      | kebab-case      | `user-service.ts`          |
| Classe       | PascalCase      | `UserService`              |
| Função/método| camelCase       | `createUser()`             |
| Constante    | SCREAMING_SNAKE | `MAX_RETRY_COUNT`          |
| Tipo/Interface| PascalCase     | `CreateUserDto`            |
| Arquivo teste| mesmo + .test   | `user-service.test.ts`     |

## Commits

Siga [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: adiciona endpoint de convite de usuário
fix: corrige validação de email duplicado (RN-001)
docs: atualiza glossário com termo "Convite"
test: adiciona testes de limite de plano (RN-010)
refactor: extrai planGuard para service dedicado
chore: atualiza dependências
```
