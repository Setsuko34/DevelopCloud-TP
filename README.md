# Développer pour le Cloud - YNOV Campus

Ce repository contient l'ensemble des travaux pratiques et projets réalisés dans le cadre du cours **Développer pour le Cloud** au sein d'YNOV Campus Montpellier (Master 2).

## Structure du projet

Le repository est organisé par dossiers correspondants aux différentes étapes du cursus :

- [**tp1-app/**](./tp1-app/) : Introduction aux fondamentaux de Google Cloud Platform (GCP) et déploiement d'une première application Flask.
- [**tp2-app/**](./tp2-app/) : Docker Avancé, Cloud Run et Networking. Mise en place d'une stack micro-services avec Redis et PostgreSQL.
- **MD/** : Supports de cours et instructions des TP et réponses aux questions au format Markdown.
- **PDF/** : Énoncés originaux des TP au format PDF.
- **Capture/** : Captures d'écran justifiant la réalisation des étapes clés (preuves de déploiement, tests de performance, etc.).

## Compétences acquises

- **Conteneurisation** : Docker, Dockerfiles optimisés (multi-stage build), Docker Compose.
- **Google Cloud Platform (GCP)** :
    - **Compute** : Cloud Run (Serverless), Artifact Registry.
    - **Storage** : Cloud Storage avec gestion du versioning et cycle de vie.
    - **Networking** : Configuration de VPC, sous-réseaux et règles de pare-feu.
- **Base de données & Cache** : PostgreSQL pour la persistance, Redis pour l'optimisation des performances (TTL, gestion de cache).
- **CI/CD & Stratégies de déploiement** : Traffic splitting (Canary deployment) sur Cloud Run.

## Prérequis

Pour exécuter les projets localement ou les déployer :
- Docker et Docker Compose
- Google Cloud SDK (`gcloud`)
- Node.js / Python (selon les modules)

