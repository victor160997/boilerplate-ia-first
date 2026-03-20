# Guia de Início Rápido

<!--
  PROPÓSITO: Levar um dev (ou agente de IA configurando ambiente) do zero
  ao servidor rodando localmente no menor número de passos possível.

  AUDIÊNCIA: Devs novos no projeto.

  MANUTENÇÃO: Teste este guia do zero periodicamente. Se precisar de mais
  de 15 minutos para ter o ambiente rodando, algo está errado.
-->

## Pré-requisitos

Certifique-se de ter instalado:

| Ferramenta   | Versão mínima | Verificar com        |
|--------------|---------------|----------------------|
| Node.js      | 20.x          | `node --version`     |
| pnpm         | 9.x           | `pnpm --version`     |
| Docker       | 24.x          | `docker --version`   |
| Docker Compose | 2.x         | `docker compose version` |
| Git          | 2.x           | `git --version`      |

## Passo 1: Clone e instale dependências

```bash
git clone <url-do-repositorio>
cd <nome-do-projeto>
pnpm install
```

## Passo 2: Configure as variáveis de ambiente

```bash
cp .env.example .env
```

Abra o `.env` e preencha os valores obrigatórios:

```env
# Banco de dados (funciona com o Docker Compose padrão)
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/app_dev"

# Redis (funciona com o Docker Compose padrão)
REDIS_URL="redis://localhost:6379"

# JWT — gere com: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
JWT_SECRET="troque-por-um-segredo-real"

# Claude API — obtenha em console.anthropic.com
ANTHROPIC_API_KEY="sk-ant-..."
```

## Passo 3: Suba os serviços de infraestrutura

```bash
docker compose up -d
```

Isso sobe: PostgreSQL, Redis (e qualquer outro serviço definido no `docker-compose.yml`).

Verifique se estão rodando:
```bash
docker compose ps
```

## Passo 4: Rode as migrations

```bash
pnpm db:migrate
```

Opcionalmente, popule com dados de exemplo:
```bash
pnpm db:seed
```

## Passo 5: Inicie o servidor de desenvolvimento

```bash
pnpm dev
```

O servidor estará disponível em: `http://localhost:3000`

## Verificando que tudo funciona

```bash
# Health check
curl http://localhost:3000/health
# Esperado: { "status": "ok" }

# Rode os testes
pnpm test
```

## Comandos úteis

```bash
pnpm dev          # Servidor com hot-reload
pnpm test         # Todos os testes
pnpm test:watch   # Testes em modo watch (ideal para TDD)
pnpm lint         # Verificar lint
pnpm typecheck    # Verificar tipos TypeScript
pnpm db:studio    # Abre Prisma Studio (UI para o banco)
pnpm db:reset     # Reseta o banco e roda seed (CUIDADO: destrutivo)
```

## Problemas comuns

**Porta em uso:**
```bash
# Verifique o que está usando a porta
lsof -i :3000
```

**Erro de conexão com banco:**
```bash
# Certifique-se que o Docker está rodando
docker compose ps
docker compose up -d postgres
```

**Erro de migration:**
```bash
# Resete o estado das migrations (só em desenvolvimento!)
pnpm db:reset
```
