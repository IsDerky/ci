# ci

Workflows reutilizables de GitHub Actions para mis repos. Cada repo deja un `ci.yml` cortito que llama a estos con `@v1`, así un cambio (versión de una action, caché, lo que sea) se hace una vez aquí y no en diez sitios.

| Workflow | Qué hace |
|---|---|
| `app-node.yml` | install + lint/typecheck/test/build con bun, npm o pnpm |
| `imagen.yml` | docker build, arranca el contenedor y comprueba que responde; opcional subir a GHCR |
| `deploy.yml` | despliegue en el servidor con el runner self-hosted del servicio |
| `ansible.yml` | ansible-lint con las colecciones del `requirements.yml` |

## Uso

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: IsDerky/ci/.github/workflows/app-node.yml@v1
    with:
      gestor: bun
      typecheck: true
      test: true

  deploy:
    needs: ci
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: IsDerky/ci/.github/workflows/deploy.yml@v1
    with:
      servicio: oficina
```

Las entradas de cada uno están documentadas arriba del propio fichero.

### Despliegue

`deploy.yml` corre en un runner self-hosted con la etiqueta del servicio y solo llama a `sudo deploy-repo <servicio> <sha>`. El runner no ejecuta código del repo: `deploy-repo` (root) hace su propio fetch, comprueba que el commit está en main y lanza el instalador del repo. Aunque el que llama se olvide del `if`, el job no corre fuera de un push a la rama por defecto.

## Versiones

`v1` es una etiqueta móvil que apunta al último `v1.x.y`. Los cambios que rompan algo van a `v2`.

```sh
git tag v1.0.1 && git tag -f v1 && git push origin v1.0.1 && git push -f origin v1
```

## Licencia

MIT
