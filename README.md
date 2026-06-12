# github-workflows

Workflows reutilizáveis para CI/CD dos projetos PedeGás.

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
    uses: sostenesapollo/github-workflows/.github/workflows/docker-coolify-deploy.yml@v1
    permissions:
      contents: read
      packages: write
    with:
      app_name: Meu App
      repo_name: meu-repo
      coolify_deploy_uuid: xxxxxxxxxxxxxxxxxxxxxxxx
    secrets:
      COOLIFY_TOKEN: ${{ secrets.COOLIFY_TOKEN }}
      PRODUCTION_ENV: ${{ secrets.PRODUCTION_ENV }}
```

## Secrets no repositório do app

| Secret | Obrigatório | Descrição |
|--------|-------------|-----------|
| `COOLIFY_TOKEN` | Para deploy | Token da API Coolify |
| `PRODUCTION_ENV` | Opcional | Conteúdo do `.env.production` para o build |

## Inputs opcionais

| Input | Default | Descrição |
|-------|---------|-----------|
| `coolify_api_url` | `https://coolify.pedegas.com/api/v1/deploy` | URL da API |
| `ntfy_url` | `https://ntfy.sh/sostenes_alertas` | Notificações |
| `deploy_branch` | `master` | Branch que faz deploy |
| `dockerfile` | `./Dockerfile` | Caminho do Dockerfile |
| `docker_context` | `.` | Contexto do build |
| `create_env_production` | `true` | Gerar `.env.production` no build |
| `platforms` | `linux/amd64` | Plataformas Docker |
| `min_package_versions_to_keep` | `2` | Cleanup GHCR |

## Versionamento

Use `@v1`, `@v2`, etc. nos projetos. Atualize a tag ao mudar o workflow central.

```bash
git tag v1
git push origin v1
```

## Primeiro setup

```bash
gh repo create sostenesapollo/github-workflows --public --source=. --push
git tag v1 && git push origin v1
```

O repositório precisa ser **público** (ou na mesma org com acesso) para o `uses:` funcionar.
