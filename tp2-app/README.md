# TP 2 - Docker Avancé, Cloud Run & Networking GCP

Ce dossier contient l'application Node.js/TypeScript développée pour le TP 2, mettant en œuvre des concepts avancés de conteneurisation et de déploiement cloud.

## Architecture de l'application

L'application est composée de trois services orchestrés par Docker Compose :

1.  **Web** : Serveur Express en TypeScript (Image optimisée via multi-stage build).
2.  **Database** : PostgreSQL 16 pour la persistance des visites.
3.  **Cache** : Redis 7 pour la mise en cache des compteurs avec TTL.

## Commandes principales

### Développement local

```bash
# Lancer la stack complète avec Docker Compose
docker-compose up -d --build

# Consulter les logs
docker-compose logs -f

# Arrêter les services et supprimer les volumes
docker-compose down -v
```

### Build & Registry (GCP)

```bash
# Builder l'image multi-stage
docker build -t tp2-app:v1 .

# Tagger pour Artifact Registry
PROJECT_ID=$(gcloud config get-value project)
docker tag tp2-app:v1 europe-west9-docker.pkg.dev/${PROJECT_ID}/tp2-registry/tp2-app:v1

# Pousser l'image
docker push europe-west9-docker.pkg.dev/${PROJECT_ID}/tp2-registry/tp2-app:v1
```

### Déploiement Cloud Run

```bash
# Déployer sur Cloud Run
gcloud run deploy tp2-service \
  --image=europe-west9-docker.pkg.dev/${PROJECT_ID}/tp2-registry/tp2-app:v1 \
  --region=europe-west9 \
  --allow-unauthenticated
```

## Endpoints de test

- `GET /` : Message de bienvenue et version.
- `GET /health` : Vérification de l'état de l'application et de la DB.
- `GET /db` : Incrémentation directe du compteur en base de données.
- `GET /cached` : Lecture du compteur via Redis (TTL 10s) avec fallback vers PostgreSQL.
