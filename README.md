# CI/CD og Deploy-oppsett

Dette repoet koordinerer deploy av tre tjenester i vårt system: `backend`, `frontend`, og `detection-api`. Alle tjenester containeriseres og kjøres sammen med Docker Compose på en virtuell maskin (VM).

## Docker Container Registry

Vi bruker [GitHub Container Registry (GHCR)](https://ghcr.io) for å bygge og distribuere Docker-images. Hvert tjenesterepo har en GitHub Actions workflow som:

1. Bygger et Docker-image fra repoets kode.
2. Tagger og pusher imaget til `ghcr.io/bachelor-group-13/<reponavn>:latest`.
3. Trigger et deploy-workflow i dette repoet.

## Deploy-mekanisme

Ved hvert push til `main`-grenen i `backend`, `frontend` eller `detection-api` skjer følgende:

- Nye Docker-images pushes til GHCR.
- Et `workflow_dispatch`-kall trigges i `inneparkert-deployment`.
- GitHub Actions logger inn på vår VM via SSH og kjører:

```bash
docker-compose --env-file .env pull
docker-compose --env-file .env up -d
```

Dette sikrer at alle tjenester alltid kjører den nyeste versjonen etter hvert push.

## Struktur på VM

På VM-en forventes følgende struktur:

```
~/inneparkert/
├── .env                    # Miljøvariabler for docker-compose
├── docker-compose.yml      # Starter hele applikasjonen
```

> `.env` inneholder alle secrets og miljøvariabler og er ikke versjonskontrollert.

## Sikkerhet

- Runtime-secrets (som JWT-nøkler, DB-passord etc.) lagres **kun i `.env` på VM**.
- Ingen secrets injiseres i workflows eller GitHub-logg.
- Docker på VM er logget inn mot GHCR for å kunne hente private images.
