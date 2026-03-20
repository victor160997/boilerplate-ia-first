# Boilerplate IA-First

> Ponto de entrada do projeto. Este README deve responder em 30 segundos:
> quem usa, o que faz, como rodar.

## O que é este projeto

<!-- Descreva o produto em 2-3 linhas. Evite jargão. -->
<!-- Exemplo: -->
Plataforma multi-tenant de gestão de [domínio], construída com uma abordagem
**IA-first** — toda a documentação, decisões arquiteturais e contexto do código
são escritos para serem consumidos tanto por humanos quanto por agentes de IA.

## Início rápido

```bash
# Clone e instale dependências
git clone <repo-url>
cd <projeto>
cp .env.example .env
npm install   # ou pnpm / yarn

# Suba o ambiente local
docker compose up -d
npm run dev
```

## Documentação

| O que você precisa              | Onde encontrar                                   |
|---------------------------------|--------------------------------------------------|
| Entender o sistema              | [docs/architecture/overview.md](docs/architecture/overview.md) |
| Regras de negócio               | [docs/domain/business-rules.md](docs/domain/business-rules.md) |
| Configurar ambiente             | [docs/guides/getting-started.md](docs/guides/getting-started.md) |
| Trabalhar com IA (Claude, etc.) | [docs/ai-context/system-prompt.md](docs/ai-context/system-prompt.md) |
| Endpoints da API                | [docs/api/endpoints.md](docs/api/endpoints.md) |
| Deploy e ambientes              | [docs/infra/deployment.md](docs/infra/deployment.md) |

## Stack principal

<!-- Liste as tecnologias-chave. Mantenha atualizado. -->
- **Runtime:** Node.js 20 / TypeScript
- **Framework:** (ex: Fastify, NestJS, Next.js)
- **Banco de dados:** (ex: PostgreSQL + Prisma)
- **Infra:** Docker + (ex: AWS / GCP / Railway)
- **IA:** Claude API (Anthropic)

## Contribuindo

Leia [CONTRIBUTING.md](CONTRIBUTING.md) antes de abrir um PR.
Para decisões arquiteturais, consulte [docs/architecture/decisions/](docs/architecture/decisions/).
