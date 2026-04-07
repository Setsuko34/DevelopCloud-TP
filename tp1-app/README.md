# TP1 — Fondamentaux GCP

Travaux Pratiques 1 du cours Cloud — YNOV.
Objectif : prendre en main les services fondamentaux de Google Cloud Platform et conteneuriser une application Flask avec Docker.

## Structure du projet

```
TP1/
├── tp1-app/
│   ├── app.py              # Application Flask (routes / et /health)
│   ├── requirements.txt    # Dépendances Python (Flask, Gunicorn)
│   ├── Dockerfile          # Image Docker Python 3.12-slim
│   └── .dockerignore
├── Capture/                # Captures d'écran des livrables GCP
├── TP.md                   # Réponses aux exercices
└── README.md
```

## Application Flask

L'application expose deux routes :

| Route | Description |
|-------|-------------|
| `GET /` | Retourne un JSON avec message, hostname, environnement et version |
| `GET /health` | Health check — retourne `{"status": "ok"}` avec HTTP 200 |

## Lancer l'application avec Docker

### Build de l'image

```bash
cd tp1-app
docker build -t tp1-flask:v1 .
```

### Démarrer le conteneur

```bash
docker run -d -p 8080:8080 --name tp1-container -e APP_ENV=development tp1-flask:v1
```

### Tester

```bash
curl http://localhost:8080/
curl http://localhost:8080/health
```

### Arrêter et supprimer le conteneur

```bash
docker stop tp1-container && docker rm tp1-container
```

## Variables d'environnement

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| `PORT` | `8080` | Port d'écoute de l'application |
| `APP_ENV` | `production` | Environnement applicatif |

## Parties du TP

| Partie | Sujet |
|--------|-------|
| 1 | Théorie Cloud (modèles de service, NIST, microservices, GCP) |
| 2 | Setup GCP & gcloud CLI |
| 3 | Google Cloud Storage (buckets, objets) |
| 4 | Compute Engine (cycle de vie d'une VM) |
| 5 | Docker — conteneurisation de l'application Flask |
| 6 | IAM & Service Accounts GCP |
| 7 | Docker — inspection et debug de conteneurs |

## Prérequis

- [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) (`gcloud` CLI)
- [Docker](https://docs.docker.com/get-docker/) 20+
- Compte GCP avec un projet actif
