# Deploy

<!--
  PROPÓSITO: Guia completo de como fazer deploy em cada ambiente.
  Deve ser suficiente para um dev fazer deploy sem ajuda.

  AUDIÊNCIA: Devs, DevOps e agentes de IA em pipelines de CD.

  QUANDO ATUALIZAR: Sempre que o processo de deploy mudar.
-->

## Visão geral

```
Push para branch → CI (testes + lint + typecheck) → CD (deploy automático)

main   → staging (automático após CI verde)
tag    → production (manual, após aprovação)
```

## Ambientes

| Ambiente   | Branch/Trigger | URL                        | Deploy       |
|------------|----------------|----------------------------|--------------|
| Local      | —              | localhost:3000             | Manual       |
| Staging    | `main`         | staging.seudominio.com     | Automático   |
| Production | tag `v*.*.*`   | api.seudominio.com         | Manual       |

Ver detalhes de cada ambiente em [environments.md](environments.md).

---

## Deploy para Staging

O deploy para staging é **automático** após um push para `main` com CI verde.

Para acompanhar:
```bash
# Ver status do pipeline (GitHub Actions)
gh run list --branch main

# Ver logs do deploy em andamento
gh run view <run-id> --log
```

Se o deploy falhar, verifique:
1. Logs do CI: `gh run view <run-id> --log-failed`
2. Logs da aplicação no staging (ver environments.md)

---

## Deploy para Production

### Pré-requisitos
- [ ] Staging testado e estável
- [ ] PR aprovado por pelo menos 1 reviewer
- [ ] Migrations revisadas (se houver)
- [ ] Changelog atualizado

### Passo 1: Criar a tag de versão

```bash
# Certifique-se de estar em main atualizado
git checkout main && git pull

# Crie a tag seguindo semver
git tag v1.2.0
git push origin v1.2.0
```

### Passo 2: Acompanhar o deploy

```bash
gh run list --workflow=deploy-production.yml
```

### Passo 3: Verificar o deploy

```bash
# Health check
curl https://api.seudominio.com/health

# Verificar versão deployada
curl https://api.seudominio.com/version
```

### Rollback

Em caso de problema em produção:

```bash
# Opção 1: Redeployar a tag anterior
git tag v1.1.1-hotfix v1.1.1
git push origin v1.1.1-hotfix

# Opção 2: Via painel da plataforma de deploy
# (Railway / Render / AWS) — revertir para o deploy anterior
```

---

## Migrations em produção

Migrations rodam **automaticamente** como parte do pipeline de deploy,
antes de iniciar a nova versão da aplicação.

```yaml
# .github/workflows/deploy-production.yml (trecho)
- name: Run migrations
  run: pnpm db:migrate:deploy
  env:
    DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}

- name: Start application
  run: pnpm start
```

> **ATENÇÃO:** Migrations irreversíveis (DROP, rename) devem ser revisadas
> manualmente antes do deploy. Marque no PR com a label `breaking-migration`.

---

## Checklist de deploy em produção

- [ ] CI verde em `main`
- [ ] Testado em staging
- [ ] Migrations revisadas
- [ ] Variáveis de ambiente de produção atualizadas (se necessário)
- [ ] Health check passando após o deploy
- [ ] Time avisado no canal de deploys
