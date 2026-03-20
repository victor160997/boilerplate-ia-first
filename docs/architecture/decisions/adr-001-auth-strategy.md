# ADR-001: Estratégia de Autenticação

<!--
  ADR = Architecture Decision Record.
  Registra UMA decisão arquitetural importante: o contexto, as opções
  consideradas, a decisão tomada e as consequências.

  NUNCA altere um ADR após aprovado. Se a decisão mudar, crie um novo ADR
  que supersede este (ex: "ADR-005 supersede ADR-001").

  Template baseado em: https://adr.github.io/madr/
-->

## Status

`Aprovado` — 2026-03-20

## Contexto

O sistema precisa autenticar usuários em múltiplos tenants de forma segura.
Usuários podem pertencer a mais de um tenant e precisam trocar de contexto
sem novo login. A API também precisa suportar autenticação machine-to-machine
para integrações.

## Opções consideradas

### Opção A: Session-based (cookies)
- **Prós:** Simples de invalidar, sem exposição de token no client
- **Contras:** Requer armazenamento de sessão (Redis obrigatório), dificulta
  APIs stateless, problemas com CORS em multi-tenant

### Opção B: JWT stateless
- **Prós:** Sem estado no servidor, escala horizontalmente, padrão de mercado
- **Contras:** Não é possível invalidar antes do expiry sem blocklist

### Opção C: JWT de curta duração + Refresh Token (escolhida)
- **Prós:** Melhor dos dois mundos — JWT leve para requests, refresh token
  permite invalidação, suporta rotação de tokens
- **Contras:** Levemente mais complexo de implementar

## Decisão

**Opção C: JWT (15min) + Refresh Token (7 dias)**

- Access token: JWT assinado com RS256, expira em 15 minutos
- Refresh token: opaque token armazenado no Redis com TTL de 7 dias
- O payload do JWT inclui: `userId`, `tenantId`, `role`, `iat`, `exp`
- Troca de tenant: emite novo par de tokens sem nova senha

### Fluxo de autenticação

```
Cliente                    API
  │                         │
  │── POST /auth/login ─────►│
  │   { email, password }   │
  │                         │── valida credenciais
  │                         │── gera access + refresh token
  │◄── 200 { accessToken, ──│
  │         refreshToken }  │
  │                         │
  │── GET /api/resource ────►│ (header: Bearer <accessToken>)
  │                         │── valida JWT
  │◄── 200 { data } ────────│
  │                         │
  │── POST /auth/refresh ───►│ (body: { refreshToken })
  │                         │── valida no Redis
  │                         │── emite novos tokens (rotação)
  │◄── 200 { accessToken, ──│
  │         refreshToken }  │
```

## Consequências

- **Positivas:** API completamente stateless para requests normais;
  invalidação possível via Redis; suporta troca de tenant sem re-login
- **Negativas:** Requer Redis como dependência obrigatória;
  access token válido por até 15min após revogação (janela aceitável)
- **Ações necessárias:**
  - Implementar middleware de validação JWT em `src/middleware/auth.ts`
  - Configurar Redis para armazenamento de refresh tokens
  - Documentar rotação de tokens no guia de desenvolvimento
