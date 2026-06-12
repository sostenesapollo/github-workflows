# github-workflows

Workflows reutilizáveis para CI/CD.

## Workflows

### `docker-coolify-deploy.yml`

Build Docker → push GHCR → cleanup de tags antigas → deploy no Coolify + notificações ntfy.

## Uso em um projeto

Crie `.github/workflows/deploy.yml`:

```yaml
name: Build & Push Docker

on:
  push:
    branches: [master]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - 'scripts/**'
  workflow_dispatch:

concurrency:
  group: docker-build-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: sostenesapollo/github-workflows/.github/workflows/docker-coolify-deploy.yml@v2
    permissions:
      contents: read
      packages: write
    with:
      app_name: Meu App
      repo_name: meu-repo
      coolify_deploy_uuid: xxxxxxxxxxxxxxxxxxxxxxxx
    secrets:
      COOLIFY_TOKEN: ${{ secrets.COOLIFY_TOKEN }}
      COOLIFY_API_URL: ${{ secrets.COOLIFY_API_URL }}
      NTFY_URL: ${{ secrets.NTFY_URL }}
      PRODUCTION_ENV: ${{ secrets.PRODUCTION_ENV }}
```

## Secrets

Configure no **repositório do app** ou na **organização** (recomendado para valores compartilhados entre vários apps).

| Secret | Obrigatório | Descrição |
|--------|-------------|-----------|
| `COOLIFY_TOKEN` | Para deploy | Token da API Coolify |
| `COOLIFY_API_URL` | Para deploy | URL base da API de deploy (ex. `https://coolify.exemplo.com/api/v1/deploy`) |
| `NTFY_URL` | Para notificações | URL completa do tópico ntfy (ex. `https://ntfy.sh/meu-topico-secreto`) |
| `PRODUCTION_ENV` | Opcional | Conteúdo do `.env.production` para o build |

**Dica:** coloque `COOLIFY_TOKEN`, `COOLIFY_API_URL` e `NTFY_URL` como secrets de **organização** para não repetir em cada repo. `coolify_deploy_uuid` e `PRODUCTION_ENV` ficam por app (input + secret do repo).

## Inputs opcionais

| Input | Default | Descrição |
|-------|---------|-----------|
| `deploy_branch` | `master` | Branch que faz deploy |
| `dockerfile` | `./Dockerfile` | Caminho do Dockerfile |
| `docker_context` | `.` | Contexto do build |
| `create_env_production` | `true` | Gerar `.env.production` no build |
| `platforms` | `linux/amd64` | Plataformas Docker |
| `min_package_versions_to_keep` | `2` | Cleanup GHCR |

## Versionamento

Use `@v1`, `@v2`, etc. nos projetos. Atualize a tag ao mudar o workflow central.

```bash
git tag v2
git push origin v2
```

## Primeiro setup

```bash
gh repo create sostenesapollo/github-workflows --public --source=. --push
git tag v2 && git push origin v2
```

O repositório precisa ser **público** (ou na mesma org com acesso) para o `uses:` funcionar.
