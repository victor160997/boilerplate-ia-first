# Endpoints da API

<!--
  PROPÓSITO: Referência de todos os endpoints HTTP da API.
  Deve ser a fonte da verdade sobre rotas, métodos e autenticação.

  FORMATO: Cada endpoint tem método, rota, autenticação, parâmetros
  e exemplo de request/response.

  QUANDO ATUALIZAR: Sempre que um endpoint for adicionado, removido
  ou tiver sua interface alterada.

  COMPLEMENTAR COM: docs/api/contracts.md (schemas detalhados dos bodies)
-->

## Autenticação

Todos os endpoints (exceto os de auth) exigem o header:
```
Authorization: Bearer <accessToken>
```

## Convenções

- Base URL: `https://api.seudominio.com/v1`
- Responses de erro seguem o formato: `{ "error": "CODE", "message": "..." }`
- Paginação: `?page=1&limit=20` → `{ "data": [...], "total": N, "page": N }`
- Datas: ISO 8601 (`2026-03-20T15:00:00Z`)

---

## Auth

### POST /auth/login
Autentica um usuário e retorna tokens de acesso.

**Autenticação:** Pública

**Body:**
```json
{
  "email": "joao@empresa.com",
  "password": "senha123"
}
```

**Response 200:**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "tok_...",
  "user": {
    "id": "usr_123",
    "email": "joao@empresa.com",
    "name": "João Silva"
  }
}
```

**Erros:**
- `401 INVALID_CREDENTIALS` — email ou senha incorretos
- `403 ACCOUNT_SUSPENDED` — conta suspensa

---

### POST /auth/refresh
Gera novos tokens usando o refresh token.

**Autenticação:** Pública

**Body:**
```json
{ "refreshToken": "tok_..." }
```

**Response 200:** Mesmo formato do login

**Erros:**
- `401 INVALID_REFRESH_TOKEN`
- `401 REFRESH_TOKEN_EXPIRED`

---

### POST /auth/logout
Invalida o refresh token atual.

**Autenticação:** Bearer token

**Response:** `204 No Content`

---

## Usuários

### GET /tenants/:tenantId/users
Lista todos os usuários do tenant.

**Autenticação:** Bearer token — roles: `ADMIN`, `OWNER`

**Query params:**
- `page` (default: 1)
- `limit` (default: 20, max: 100)
- `search` — busca por nome ou email

**Response 200:**
```json
{
  "data": [
    {
      "id": "usr_123",
      "email": "joao@empresa.com",
      "name": "João Silva",
      "role": "MEMBER",
      "createdAt": "2026-03-20T15:00:00Z"
    }
  ],
  "total": 42,
  "page": 1
}
```

---

### POST /tenants/:tenantId/users/invite
Convida um novo usuário para o tenant.

**Autenticação:** Bearer token — roles: `ADMIN`, `OWNER`

**Body:**
```json
{
  "email": "novo@empresa.com",
  "role": "MEMBER"
}
```

**Response 201:**
```json
{
  "invitationId": "inv_456",
  "email": "novo@empresa.com",
  "expiresAt": "2026-03-22T15:00:00Z"
}
```

**Erros:**
- `400 EMAIL_ALREADY_EXISTS` — usuário já está no tenant (RN-001)
- `402 PLAN_LIMIT_EXCEEDED` — limite de usuários atingido (RN-010)

---

## [Adicione novos endpoints seguindo o padrão acima]

<!--
  Template:

  ### METHOD /rota
  Descrição de uma linha.

  **Autenticação:** Pública | Bearer token [roles: ROLE1, ROLE2]

  **Body/Params:**
  ```json
  { ... }
  ```

  **Response XXX:**
  ```json
  { ... }
  ```

  **Erros:**
  - `STATUS CODE_ERROR` — descrição
-->
