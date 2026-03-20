# Contratos da API

<!--
  PROPÓSITO: Define os schemas de dados (contratos) usados na API.
  Complementa endpoints.md com definições precisas de tipos.

  ESTE ARQUIVO É A FONTE DA VERDADE para:
  - Tipos TypeScript gerados/mantidos manualmente
  - Schemas Zod de validação
  - Documentação para consumidores da API

  QUANDO ATUALIZAR: Sempre que um schema mudar.
  Mudanças breaking devem criar nova versão (v2) da rota correspondente.
-->

## Convenções de tipos

| Tipo TS         | Formato JSON         | Exemplo                          |
|-----------------|----------------------|----------------------------------|
| `string` (UUID) | `string`             | `"usr_01J..."`                   |
| `Date`          | ISO 8601 string      | `"2026-03-20T15:00:00Z"`         |
| `enum`          | `string` (uppercase) | `"ADMIN"`, `"MEMBER"`            |
| `null`          | `null` ou ausente    | Campo omitido = não aplica       |

---

## Entidades principais

### User

```typescript
// Schema Zod — src/domain/dtos.ts
export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(200),
  role: z.enum(['OWNER', 'ADMIN', 'MEMBER']),
  tenantId: z.string().uuid(),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
})

export type User = z.infer<typeof UserSchema>
```

```json
// Exemplo JSON
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "joao@empresa.com",
  "name": "João Silva",
  "role": "ADMIN",
  "tenantId": "tenant-uuid-aqui",
  "createdAt": "2026-03-20T15:00:00Z",
  "updatedAt": "2026-03-20T15:00:00Z"
}
```

---

### Tenant

```typescript
export const TenantSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  plan: z.enum(['FREE', 'PRO', 'ENTERPRISE']),
  createdAt: z.string().datetime(),
})

export type Tenant = z.infer<typeof TenantSchema>
```

---

### Invitation

```typescript
export const InvitationSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(['ADMIN', 'MEMBER']),
  tenantId: z.string().uuid(),
  status: z.enum(['PENDING', 'ACCEPTED', 'EXPIRED']),
  expiresAt: z.string().datetime(),
  createdAt: z.string().datetime(),
})

export type Invitation = z.infer<typeof InvitationSchema>
```

---

## DTOs de input (requests)

### CreateUserInvitationDto

```typescript
export const CreateInvitationDto = z.object({
  email: z.string().email({ message: 'Email inválido' }),
  role: z.enum(['ADMIN', 'MEMBER']).default('MEMBER'),
})
```

---

## Formato de erro padrão

Todos os erros seguem este contrato:

```typescript
export const ErrorResponseSchema = z.object({
  error: z.string(),       // Código de erro em SCREAMING_SNAKE_CASE
  message: z.string(),     // Mensagem legível por humano
  details: z.unknown().optional(), // Dados adicionais (ex: campos inválidos)
})
```

```json
// Exemplo — erro de validação
{
  "error": "VALIDATION_ERROR",
  "message": "Dados de entrada inválidos",
  "details": {
    "email": "Email inválido"
  }
}

// Exemplo — erro de negócio
{
  "error": "PLAN_LIMIT_EXCEEDED",
  "message": "Seu plano Free permite até 5 usuários. Faça upgrade para adicionar mais."
}
```

## Versionamento de contratos

Mudanças breaking (remover campo, mudar tipo) requerem nova versão da rota:
- `v1`: `/v1/users` — contrato atual
- `v2`: `/v2/users` — nova versão (quando necessário)

Adicionar campos opcionais não é breaking e não requer nova versão.
