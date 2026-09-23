---
title: Onboarding
---
# Onboarding — levantar el proyecto en local

```bash
mkdir micasaestuya && cd micasaestuya
git clone git@github.com:marlonbdez/micasaestuya-web.git
git clone git@github.com:marlonbdez/micasaestuya-api.git
git clone git@github.com:marlonbdez/micasaestuya-infra.git
# los tres deben quedar como hermanos en la misma carpeta

cp micasaestuya-infra/.env.example micasaestuya-infra/.env
# ajustar micasaestuya-infra/.env si hace falta

# Abrir micasaestuya-web/ o micasaestuya-api/ en VS Code -> "Reopen in Container"
# (usa micasaestuya-infra/docker-compose.yml por debajo vía devcontainer.json)

# Primera vez: poblar Redis con datos de localización
docker exec express npm run redis:seed
```

Puertos locales: 3000 (web), 3001 (api), 27017 (mongo), 6379 (redis).

Cada repo tiene también su propio .env.example (micasaestuya-web/, micasaestuya-api/) para correr sin Docker si se prefiere.

Guía completa y detallada (troubleshooting, comandos útiles): README de [micasaestuya-infra](https://github.com/marlonbdez/micasaestuya-infra).
