# Restrições para Geração de IA

<!--
  PROPÓSITO: Lista de restrições e diretrizes específicas para quando
  agentes de IA (como Claude Code) geram ou modificam código neste projeto.

  ESTE ARQUIVO É LIDO POR:
  - Devs que usam Claude Code no projeto
  - Pipelines de CI com verificação assistida por IA
  - Qualquer agente que modifique este codebase

  DIFERENÇA DE CLAUDE.md:
  - CLAUDE.md: contexto geral e arquitetura
  - Este arquivo: restrições explícitas e verificáveis para geração de código
-->

## Restrições de segurança (CRÍTICAS — nunca violar)

### SC-001: tenantId obrigatório em queries
Toda query Prisma em tabelas de negócio DEVE incluir `tenantId` no `where`.

```typescript
// ❌ PROIBIDO
const users = await db.user.findMany()
const user = await db.user.findFirst({ where: { email } })

// ✅ OBRIGATÓRIO
const users = await db.user.findMany({ where: { tenantId } })
const user = await db.user.findFirst({ where: { email, tenantId } })
```

### SC-002: Nunca expor stack trace em produção
Erros devem ser tratados e retornados como mensagens genéricas em produção.

```typescript
// ❌ PROIBIDO em handlers de API
return res.status(500).json({ error: err.stack })

// ✅ OBRIGATÓRIO
logger.error(err) // log interno com stack
return res.status(500).json({ error: 'Internal server error' })
```

### SC-003: Validação de input nos controllers
Todo input externo deve ser validado com schema (Zod) antes de chegar
ao service.

```typescript
// ❌ PROIBIDO
const { email } = req.body // sem validação

// ✅ OBRIGATÓRIO
const { email } = createUserSchema.parse(req.body)
```

### SC-004: Sem segredos hardcoded
Nunca gere código com API keys, senhas ou segredos literais.

```typescript
// ❌ PROIBIDO
const client = new Anthropic({ apiKey: 'sk-ant-...' })

// ✅ OBRIGATÓRIO
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })
```

---

## Restrições de qualidade

### QC-001: Sem `any` no TypeScript
Use tipos precisos ou `unknown` com type guard.

```typescript
// ❌ PROIBIDO
function process(data: any) { ... }

// ✅ OBRIGATÓRIO
function process(data: unknown) {
  if (!isValidData(data)) throw new Error('Invalid data')
  ...
}
```

### QC-002: Erros de domínio tipados
Nunca lance `new Error('string')` em services. Use classes de erro do domínio.

```typescript
// ❌ PROIBIDO
throw new Error('User not found')

// ✅ OBRIGATÓRIO
throw new NotFoundError('USER_NOT_FOUND')
```

### QC-003: Testes antes da implementação (TDD)
Ao gerar código de feature ou bugfix, sempre gere o teste primeiro,
confirme que ele falha, depois gere a implementação.

---

## Restrições de estrutura

### ST-001: Separação de camadas
- Controllers não chamam repositories diretamente
- Services não importam de `controllers/`
- Repositories não importam de `services/`

### ST-002: Novos módulos seguem estrutura existente
Ao criar um novo módulo, siga o padrão:
```
src/[camada]/
  [nome].ts            # implementação
  [nome].test.ts       # testes unitários (TDD)
```

---

## Checklist para revisão de código gerado por IA

Antes de aceitar código gerado por IA, verifique:

- [ ] Toda query tem `tenantId`?
- [ ] Input está sendo validado com schema?
- [ ] Erros são tipados (não `new Error('string')`)?
- [ ] Nenhum `any` introduzido?
- [ ] Nenhum segredo hardcoded?
- [ ] Teste foi escrito antes da implementação?
- [ ] Camadas respeitadas (controller → service → repository)?
