# TP3 — LogiStream : GKE, Apache Kafka & CI/CD

Plateforme de suivi de livraisons en temps réel pour la flotte de camions LogiStream.  
800 camions actifs → 80 événements GPS/seconde → traitement en temps réel via Kafka sur GKE.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          FLOTTE LOGISTREAM                              │
│                                                                         │
│   🚛 TRK-001          🚛 TRK-002          🚛 TRK-003                   │
│   Paris-Lyon          Lyon-Marseille      Bordeaux-Paris                │
│       │                    │                    │                       │
│       └────────────────────┴────────────────────┘                       │
│                    Position GPS (toutes les 10s)                        │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │ HTTP / Mobile App
                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        GKE — namespace: kafka                           │
│                                                                         │
│  ┌─────────────────────┐                                                │
│  │    GPS Producer     │  Simule les 800 camions                        │
│  │  (1 réplica)        │  Envoie vers Kafka toutes les 10s              │
│  └──────────┬──────────┘                                                │
│             │                                                           │
│             ▼                                                           │
│  ┌──────────────────────────────────────────────────────────┐           │
│  │               Apache Kafka (Strimzi — KRaft)             │           │
│  │                                                          │           │
│  │   ┌─────────────────────┐   ┌──────────────────────┐    │           │
│  │   │   truck-positions   │   │   delivery-alerts    │    │           │
│  │   │   6 partitions      │   │   3 partitions       │    │           │
│  │   │   rétention 24h     │   │   rétention 7 jours  │    │           │
│  │   └─────────────────────┘   └──────────────────────┘    │           │
│  │                                                          │           │
│  │   Broker 0    Broker 1    Broker 2                       │           │
│  │   (logistream-kafka-dual-role-0/1/2)                     │           │
│  └──────────────────────────┬───────────────────────────────┘           │
│                             │                                           │
│                             ▼                                           │
│  ┌──────────────────────────────────────────────────────────┐           │
│  │           Tracker Consumer (3 réplicas)                  │           │
│  │        Consumer Group: tracker-service-group             │           │
│  │                                                          │           │
│  │   Instance 1        Instance 2        Instance 3         │           │
│  │   partitions 0,1    partitions 2,3    partitions 4,5     │           │
│  │                                                          │           │
│  │   • Détection camion à l'arrêt (vitesse < 5 km/h)        │           │
│  │   • Détection carburant bas (< 20%)                      │           │
│  │   • Notification dispatcher                              │           │
│  └──────────────────────────┬───────────────────────────────┘           │
│                             │                                           │
└─────────────────────────────┼───────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────┐           ┌───────────────────────┐
│   API Gateway       │           │   Cloud Monitoring    │
│   (2 réplicas)      │           │                       │
│   LoadBalancer :80  │           │  • Consumer lag       │
│   → HPA (2-10 pods) │           │  • CPU / RAM pods     │
│                     │           │  • Alertes email      │
└─────────────────────┘           └───────────────────────┘
         │
         ▼
┌─────────────────────┐
│     Dashboard       │
│  Suivi temps réel   │
│  positions camions  │
└─────────────────────┘
```

---

## Pipeline CI/CD

```
┌──────────┐    push main    ┌─────────────────────────────────────────┐
│  GitHub  │ ─────────────► │           GitHub Actions                │
└──────────┘                │                                         │
                            │  Job 1: Tests & Lint                    │
                            │    • npm ci (producer + consumer)       │
                            │    • kubeval k8s/*.yaml                 │
                            │           │                             │
                            │           ▼                             │
                            │  Job 2: Build & Push                    │
                            │    • docker build producer              │
                            │    • docker build consumer              │
                            │    • push → Artifact Registry           │
                            │           │                             │
                            │           ▼                             │
                            │  Job 3: Deploy sur GKE                  │
                            │    • kubectl set image (producer)       │
                            │    • kubectl set image (consumer)       │
                            │    • kubectl rollout status             │
                            └─────────────────────────────────────────┘
                                          │
                                          ▼
                            ┌─────────────────────────┐
                            │   Artifact Registry     │
                            │   europe-west9          │
                            │   gps-producer:sha      │
                            │   tracker-consumer:sha  │
                            └─────────────────────────┘
```

---

## Structure du projet

```
tp3-app/
├── producer/
│   ├── gps-producer.js       # Simule les positions GPS des camions
│   ├── package.json
│   └── Dockerfile
├── consumer/
│   ├── tracker-consumer.js   # Traite les positions, détecte anomalies
│   ├── package.json
│   └── Dockerfile
├── k8s/
│   ├── configmap.yaml        # Configuration centralisée (Kafka, env)
│   ├── secret.yaml           # Credentials BDD, JWT (non commité)
│   ├── api-gateway.yaml      # Deployment + Service LoadBalancer
│   ├── tracker-service.yaml  # Deployment + Service ClusterIP
│   ├── hpa.yaml              # HorizontalPodAutoscaler (2-10 pods)
│   └── kafka-apps.yaml       # Deployments producer + consumer
└── kafka/
    ├── cluster.yaml          # Cluster Kafka 3 brokers (KRaft)
    └── topics.yaml           # truck-positions, delivery-alerts, delivery-events
```

---

## Déploiement

### Prérequis

```bash
gcloud components install kubectl
gcloud auth configure-docker europe-west9-docker.pkg.dev
```

### 1. Créer le cluster GKE

```bash
gcloud container clusters create-auto logistream-cluster \
  --region=europe-west9 \
  --project=$(gcloud config get-value project)

gcloud container clusters get-credentials logistream-cluster --region=europe-west9
kubectl create namespace logistream
```

### 2. Appliquer les manifests Kubernetes

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/api-gateway.yaml
kubectl apply -f k8s/tracker-service.yaml
kubectl apply -f k8s/hpa.yaml
```

### 3. Installer Kafka (Strimzi)

```bash
kubectl create namespace kafka
kubectl apply -f https://strimzi.io/install/latest?namespace=kafka -n kafka
kubectl apply -f kafka/cluster.yaml -n kafka
kubectl apply -f kafka/topics.yaml
```

### 4. Déployer le Producer et Consumer

```bash
kubectl apply -f k8s/kafka-apps.yaml
```

### 5. Vérifier

```bash
kubectl get pods -n kafka
kubectl logs -f deployment/gps-producer -n kafka
kubectl logs -f deployment/tracker-consumer -n kafka
```

---

## Métriques clés

| Métrique | Valeur |
|---|---|
| Camions actifs | 800 |
| Événements GPS/seconde | 80 |
| Partitions `truck-positions` | 6 |
| Réplicas consumer | 3 (2 partitions/instance) |
| Réplicas API Gateway | 2 (HPA: max 10) |
| Rétention positions GPS | 24h |
| Rétention alertes | 7 jours |
