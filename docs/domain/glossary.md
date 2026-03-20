# Glossário do Domínio

<!--
  PROPÓSITO: Dicionário dos termos usados no projeto. Garante que devs,
  produto e agentes de IA falem a mesma língua.

  REGRAS:
  - Um termo, uma definição. Sem ambiguidade.
  - Se o código usa um nome diferente do negócio, documente ambos.
  - Ordem alfabética dentro de cada seção.

  AUDIÊNCIA: Qualquer pessoa (ou IA) que precise entender o que um termo
  significa no contexto deste sistema específico.
-->

## Termos do Domínio

### Tenant
Uma organização/empresa que usa o sistema. Todos os dados são isolados
por tenant. Equivale a "workspace" ou "account" em outros sistemas.

**No código:** `tenantId: string` (UUID)
**Não confundir com:** usuário (um tenant tem muitos usuários)

---

### Member / Membro
Usuário com papel `MEMBER` dentro de um tenant. Acesso às funcionalidades
do produto sem permissões administrativas.

---

### Owner
Usuário que criou o tenant. Tem acesso total e é o único que pode deletar
o tenant ou transferir ownership. Não pode ser removido por admins.

---

### Convite (Invite)
Mecanismo de adição de novos usuários a um tenant. Gerado por `ADMIN` ou
`OWNER`, enviado por email, válido por 48 horas.

**No código:** entidade `Invitation` com status `PENDING | ACCEPTED | EXPIRED`

---

### Plano (Plan)
Define os limites de uso do tenant (usuários, projetos, storage).
Exemplos: `FREE`, `PRO`, `ENTERPRISE`.

---

### Role / Papel
Conjunto de permissões atribuído a um usuário dentro de um tenant.
Valores possíveis: `OWNER`, `ADMIN`, `MEMBER`. Ver RN-002.

---

## Termos Técnicos do Projeto

### Context (Contexto de IA)
Conjunto de informações passadas ao modelo de IA antes de uma interação.
Inclui system prompt, histórico e dados do tenant.

---

### ADR (Architecture Decision Record)
Documento que registra uma decisão arquitetural importante com seu contexto,
alternativas consideradas e consequências. Imutável após aprovação.
Ver: `docs/architecture/decisions/`

---

### RN (Regra de Negócio)
Identificador de uma regra documentada em `docs/domain/business-rules.md`.
Usado em comentários de código e testes para rastreabilidade.

**Exemplo de uso em código:**
```typescript
// RN-010: verifica limite do plano antes de criar usuário
await planGuard.assertCanAddUser(tenantId)
```

---

## [Adicione novos termos aqui]

<!--
  Template:

  ### NomeDoTermo
  Definição clara e objetiva.

  **No código:** como aparece no código (nome de variável, type, enum, etc.)
  **Não confundir com:** termo similar com significado diferente (se aplicável)
-->
