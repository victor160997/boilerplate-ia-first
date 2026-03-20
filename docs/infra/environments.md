# Ambientes

<!--
  PROPÓSITO: Descreve cada ambiente de execução da aplicação —
  configurações, acesso, variáveis e diferenças entre eles.

  QUANDO ATUALIZAR: Ao adicionar ambiente, mudar URL, credenciais
  de acesso (referências, não os segredos em si) ou configurações.

  SEGURANÇA: Nunca escreva valores reais de secrets aqui.
  Documente onde encontrá-los (ex: Vault, 1Password, AWS Secrets Manager).
-->

## Local

Ambiente de desenvolvimento na máquina do dev.

| Item        | Valor                                    |
|-------------|------------------------------------------|
| URL         | `http://localhost:3000`                  |
| Banco       | PostgreSQL via Docker (`localhost:5432`) |
| Redis       | Redis via Docker (`localhost:6379`)      |
| Config      | Arquivo `.env` local (não commitado)     |

**Subir o ambiente:**
```bash
docker compose up -d
pnpm dev
```

**Variáveis necessárias** (copie de `.env.example`):
```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/app_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=any-secret-for-local
ANTHROPIC_API_KEY=sk-ant-...   # obtenha em console.anthropic.com
NODE_ENV=development
```

---

## Staging

Espelho de produção para testes de integração e QA.

| Item        | Valor                                      |
|-------------|--------------------------------------------|
| URL         | `https://staging.seudominio.com`           |
| Deploy      | Automático — push para `main` com CI verde |
| Banco       | PostgreSQL gerenciado (isolado de produção)|
| Logs        | [Link para ferramenta de logs - ex: Logtail, Datadog] |
| Secrets     | [Ex: GitHub Actions Secrets — prefixo `STAGING_`] |

**Acessar logs:**
```bash
# Exemplo com Railway
railway logs --environment staging

# Exemplo com AWS CloudWatch
aws logs tail /app/staging --follow
```

**Variáveis de ambiente:**
Gerenciadas em [plataforma de deploy] no ambiente `staging`.
Para adicionar/editar: [descreva o processo sem expor o painel].

---

## Production

Ambiente de produção real, com dados reais.

| Item        | Valor                                      |
|-------------|--------------------------------------------|
| URL         | `https://api.seudominio.com`               |
| Deploy      | Manual — tag `v*.*.*` após aprovação       |
| Banco       | PostgreSQL gerenciado com backup diário    |
| Logs        | [Link para ferramenta de logs]             |
| Secrets     | [Ex: AWS Secrets Manager / 1Password Vault]|
| Monitoramento | [Link para dashboard — ex: Grafana, Datadog] |

**Health check:**
```bash
curl https://api.seudominio.com/health
# Esperado: { "status": "ok", "version": "1.2.0" }
```

**Acesso ao banco (emergência):**
```bash
# Nunca conectar direto em produção para desenvolvimento
# Somente para incidentes críticos, com aprovação do time
# Credenciais em: [referência ao cofre de senhas]
```

---

## Diferenças entre ambientes

| Configuração          | Local       | Staging     | Production  |
|-----------------------|-------------|-------------|-------------|
| `NODE_ENV`            | development | production  | production  |
| Rate limiting         | Desativado  | Ativado     | Ativado     |
| Log level             | debug       | info        | warn        |
| Email real            | Não (mock)  | Não (sandbox)| Sim        |
| Cache Redis           | Opcional    | Ativado     | Ativado     |
| HTTPS                 | Não         | Sim         | Sim         |

---

## Variáveis de ambiente — referência completa

| Variável              | Obrigatória | Descrição                                    |
|-----------------------|-------------|----------------------------------------------|
| `DATABASE_URL`        | Sim         | Connection string PostgreSQL                 |
| `REDIS_URL`           | Sim         | Connection string Redis                      |
| `JWT_SECRET`          | Sim         | Segredo para assinar JWTs (min 32 chars)     |
| `ANTHROPIC_API_KEY`   | Sim         | API key do Claude (console.anthropic.com)    |
| `NODE_ENV`            | Sim         | `development` \| `production` \| `test`      |
| `PORT`                | Não (3000)  | Porta do servidor HTTP                       |
| `LOG_LEVEL`           | Não (info)  | `debug` \| `info` \| `warn` \| `error`       |
| `CORS_ORIGIN`         | Não (*)     | Origins permitidas no CORS                   |
