# Regras de Negócio

<!--
  PROPÓSITO: Centralizar as regras de negócio do domínio em linguagem clara,
  independente de tecnologia. Este arquivo é a fonte da verdade sobre
  O QUE o sistema faz (não como faz).

  AUDIÊNCIA: Product managers, devs, QA e agentes de IA gerando código.

  QUANDO ATUALIZAR: Sempre que uma regra mudar ou uma nova for adicionada.
  Cada regra deve ter um ID estável para referência em código e testes.

  FORMATO:
  - RN-XXX: identificador único e estável
  - Título em negrito
  - Descrição clara
  - Exemplo concreto
  - Referência ao código que implementa (quando existir)
-->

## Usuários e Acesso

### RN-001: Unicidade de email por tenant
Um email só pode estar cadastrado uma vez por tenant. O mesmo email pode
existir em tenants diferentes (são usuários independentes).

**Exemplo:** `joao@empresa.com` pode existir no tenant A e no tenant B,
mas não pode ter dois registros no mesmo tenant A.

**Implementado em:** `src/services/user.service.ts` → `createUser()`

---

### RN-002: Papéis de usuário
Cada usuário tem exatamente um papel (role) por tenant:
- `OWNER` — criador do tenant, acesso total, não pode ser removido
- `ADMIN` — acesso total exceto deletar o tenant
- `MEMBER` — acesso às funcionalidades do produto

**Exemplo:** Um usuário pode ser `ADMIN` no tenant A e `MEMBER` no tenant B.

---

### RN-003: Convite por email
Novos usuários só podem acessar um tenant via convite. Convites expiram em
**48 horas**. Um convite aceito não pode ser reutilizado.

---

## Tenants e Planos

### RN-010: Limites por plano
Cada plano define limites que são verificados antes de operações de criação:

| Plano   | Usuários | Projetos | Storage |
|---------|----------|----------|---------|
| Free    | 5        | 3        | 1 GB    |
| Pro     | 50       | Ilimitado| 50 GB   |
| Enterprise | Ilimitado | Ilimitado | Custom |

**Exemplo:** Tenant no plano Free tentando adicionar o 6º usuário deve
receber erro `PLAN_LIMIT_EXCEEDED`.

**Implementado em:** `src/services/plan-guard.service.ts`

---

### RN-011: Downgrade de plano
Um tenant não pode fazer downgrade se estiver acima dos limites do plano
de destino. O sistema deve informar quantos recursos precisam ser removidos.

**Exemplo:** Tenant Pro com 20 usuários não pode fazer downgrade para Free
(limite 5). Erro deve informar: "Remova 15 usuários antes do downgrade."

---

## [Adicione aqui as regras do seu domínio]

<!--
  Template para nova regra:

  ### RN-XXX: Nome da regra
  Descrição clara em linguagem de negócio.

  **Exemplo:** Caso concreto que ilustra a regra.

  **Implementado em:** `src/services/xxx.service.ts` → `nomeDoMetodo()`
-->
