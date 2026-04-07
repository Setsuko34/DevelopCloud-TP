Livrables attendus :
Capture d'écran de votre console GCP (projet créé)
![console_GCP.png](Capture/console_GCP.png)

Capture d'écran du terminal : résultat de gcloud config list
![gcloud_config_list.png](Capture/gcloud_config_list.png)

Capture d'écran : bucket GCS créé et listé
![bucket_creer.png](Capture/bucket_creer.png)
![bucket_lister.png](Capture/bucket_lister.png)

Capture d'écran : réponse de votre application Flask en local ( docker run )
![docker_run.png](Capture/docker_run.png)

Réponses aux questions théoriques (dans ce fichier ou sur papier)

# 1.1 — Modèles de Service

| Description | Modèle |
| :--- | :--- |
| Vous gérez uniquement votre code, l'infrastructure est abstraite | **PaaS** |
| Vous gérez l'OS, le middleware et l'application sur des ressources virtualisées | **IaaS** |
| Vous utilisez l'application directement via un navigateur, sans rien gérer | **SaaS** |
| Exemple GCP : Google Compute Engine | **IaaS** |
| Exemple GCP : Google App Engine / Cloud Run | **PaaS** |
| Exemple GCP : Google Workspace (Gmail, Docs) | **SaaS** |

# 1.2 — Les 5 Caractéristiques NIST

1.  **NIST #1 Self-service à la demande :** Provisions des ressources automatique par l’utilisateur sans intervention du fournisseur.
2.  **NIST #2 Large accès réseau :** Accès au réseau depuis n’importe quel terminal.
3.  **NIST #3 Mutualisation des ressources :** Ressources physique qui peuvent êtres partagées (multitenant).
4.  **NIST #4 Élasticité rapide :** Scalabilité des ressources.
5.  **NIST #5 Service mesuré :** Suivi et controle de la consommation des services (pay as you go).

# 1.3 — Architecture Microservices vs Monolithe

| Affirmation | Monolithe | Microservices |
| :--- | :---: | :---: |
| Déploiement indépendant de chaque composant | | **x** |
| Couplage fort entre les modules | **x** | |
| Scalabilité horizontale de chaque service séparément | | **x** |
| Debugging centralisé plus simple | **x** | |
| Technologie agnostique (polyglot) | | **x** |

# 1.4 — Services GCP par catégorie

| Catégorie | Services GCP |
| :--- | :--- |
| **Calcul (Compute)** | Cloud Run, GKE, Compute Engine |
| **Stockage** | Cloud Storage, Persistent Disk |
| **Base de données** | Cloud SQL, BigQuery |
| **Réseau** | VPC, Cloud DNS |
| **Observabilité** | Cloud Logging |

## Partie 2 — Setup GCP & gcloud CLI (30 min)
# 2.1 — Vérification de l'installation
Exécutez les commandes suivantes et notez les résultats :
- Vérifier la version de gcloud installée
gcloud version
  - Résultat obtenu :
  ```bash
    C:\Program Files (x86)\Google\Cloud SDK>gcloud version
    Google Cloud SDK 561.0.0
    bq 2.1.29
    core 2026.03.13
    gcloud-crc32c 1.0.0
    gsutil 5.36
  ```

- Vérifier que Docker est installé
docker version
  - Résultat obtenu :
  ```bash
    C:\Program Files (x86)\Google\Cloud SDK>docker version
    Client:
    Version:           28.3.0
    API version:       1.51
    Go version:        go1.24.4
    Git commit:        38b7060
    Built:             Tue Jun 24 15:44:57 2025
    OS/Arch:           windows/amd64
    Context:           desktop-linux

     Server: Docker Desktop 4.43.1 (198352)
     Engine:
     Version:          28.3.0
     API version:      1.51 (minimum version 1.24)
     Go version:       go1.24.4
     Git commit:       265f709
     Built:            Tue Jun 24 15:44:25 2025
     OS/Arch:          linux/amd64
     Experimental:     false
     containerd:
     Version:          1.7.27
     GitCommit:        05044ec0a9a75232cad458027ca83437aae3f4da
     runc:
     Version:          1.2.5
     GitCommit:        v1.2.5-0-g59923ef
     docker-init:
     Version:          0.19.0
     GitCommit:        de40ad0
    ```

# 2.2 — Initialisation et configuration

Complétez les commandes manquantes ( `_______` ) puis exécutez-les :

```bash
# Se connecter avec votre compte Google
gcloud auth login

# Vérifier que vous êtes bien authentifié (liste les comptes connectés)
gcloud auth list

# Définir votre projet comme projet par défaut
# (remplacer [MON_PROJET_ID] par votre vrai ID de projet)
gcloud config set project ynov-cloud-maxime

# Définir la région par défaut sur Paris
gcloud config set compute/region europe-west9

# Définir la zone par défaut
gcloud config set compute/zone europe-west9-a

# Vérifier la configuration complète
gcloud config list
```

**Question :** Quelle est la différence entre une région et une zone dans GCP ?
**Réponse :** Une Région englobe plusieurs zones, une zone est une infrastructure physique.

# 2.3 — Exploration de la console

Connectez-vous à la console GCP (console.cloud.google.com) et répondez :

**a) Dans IAM & Admin → IAM, quel est votre rôle sur le projet ?**
**Réponse :** Propriétaire

**b) Dans Facturation, quel montant de crédit vous reste-t-il ?**
**Réponse :** 254 €

**c) Dans APIs & Services → Tableau de bord, listez 3 APIs qui sont déjà activées par défaut :**
*   API 1 : Compute Engine API
*   API 2 : BigQuery API
*   API 3 : Cloud Storage API

**d) Activez les APIs nécessaires pour ce module en complétant la commande :**

```bash
gcloud services enable \
compute.googleapis.com \
run.googleapis.com \
container.googleapis.com \
artifactregistry.googleapis.com \
storage.googleapis.com

# Vérifier que les APIs sont bien activées
gcloud services list --enabled
```

# Partie 3 — Google Cloud Storage : Créer, Utiliser, Supprimer (30 min)

Cloud Storage est le service de stockage objet de GCP. L'équivalent d'AWS S3.

## 3.1 — Créer un bucket Cloud Storage

Un bucket GCS doit avoir un nom globalement unique (dans tout GCP, pas juste dans votre projet).
Convention de nommage à utiliser : `ynov-tp1-[votre-prenom]-[date]` Exemple : `ynov-tp1-alice-20032026`

```bash
# Remplacer [NOM_BUCKET] par votre nom unique
# --location : choisir une région GCP (utiliser europe-west9)
# --storage-class : utiliser STANDARD
gcloud storage buckets create gs://ynov-tp1-maxime-20032026 
--location=europe-west9   
--default-storage-class=STANDARD

# Vérifier que le bucket a été créé
gcloud storage buckets list
```

**Question :** Pourquoi les noms de buckets GCS doivent-ils être globalement uniques ?
**Réponse :** Comme ils sont accessibles via une URL publique, il ne peut pas y avoir de conflit de nom à l'échelle mondiale si les noms sont uniques.

## 3.2 — Uploader et lister des objets

```bash
# Créer un fichier texte local de test
echo "Hello GCP - TP1 YNOV $(date)" > test_tp1.txt

# Uploader le fichier vers votre bucket
gcloud storage cp test_tp1.txt gs://ynov-tp1-maxime-20032026/

# Lister les objets dans le bucket
gcloud storage ls gs://ynov-tp1-maxime-20032026/

# Télécharger le fichier avec un nouveau nom
gcloud storage cp gs://ynov-tp1-maxime-20032026/test_tp1.txt ./test_tp1_downloaded.txt

# Vérifier le contenu
cat test_tp1_downloaded.txt
```

## 3.3 — Métadonnées et permissions

```bash
# Obtenir les informations du bucket
gcloud storage buckets describe gs://[NOM_BUCKET]

# Répondre : quel est le storageClass de votre bucket ?
# Réponse : STANDARD
```

## 3.4 — Nettoyage

```bash
# Supprimer tous les objets du bucket, puis le bucket lui-même
# L'option -r supprime récursivement
gcloud storage rm -r gs://ynov-tp1-maxime-20032026
# Vérifier la suppression
gcloud storage buckets list
```

# Partie 4 — Compute Engine : Cycle de Vie d'une VM (25 min)

Compute Engine est le service IaaS de GCP. Équivalent d'AWS EC2. Attention aux coûts : Supprimez la VM à la fin de cet exercice.

## 4.1 — Créer une VM minimale

```bash
# Créer une VM de type e2-micro (la plus petite, éligible au Free Tier)
# --image-family : utiliser debian-12
# --image-project : utiliser debian-cloud
gcloud compute instances create tp1-vm 
--machine-type=e2-micro 
--image-family=debian-12 
--image-project=debian-cloud 
--zone=europe-west9-b 
--tags=http-server

# Lister les instances actives dans votre projet
gcloud compute instances list
```

**Question :** Quelle est la différence entre `--machine-type e2-micro` et `--machine-type n2-standard-4` en termes de coût et d'usage ?
**Réponse :** e2-micro est plus petite et adaptée au dev, n2-standard-4 est plus puissante mais plus cher et adapté à la prod.

## 4.2 — Se connecter à la VM via SSH

```bash
# Connexion SSH via gcloud (gère automatiquement les clés SSH)
gcloud compute ssh tp1-vm --zone=europe-west9-b

# Une fois connecté à la VM, vérifier l'OS
uname -a
cat /etc/os-release

# Quitter la VM
exit
```

## 4.3 — Supprimer la VM (OBLIGATOIRE)

```bash
# Supprimer la VM pour éviter les frais
# --quiet : ne pas demander de confirmation
gcloud compute instances delete tp1-vm 
--zone=europe-west9-b 
--quiet


# Vérifier que la VM n'existe plus
gcloud compute instances list
```

# Partie 5 — Docker : Conteneuriser une Application Flask (30 min)

Cette partie est 100% locale (pas de GCP). On prépare l'application qu'on déploiera sur GCP au Cours 2.

## 5.1 — L'application Flask

Créez un dossier `tp1-app` et les fichiers suivants :

`tp1-app/app.py` — à compléter :

```python
from flask import Flask, jsonify
import os
import socket

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        "message": "Hello from YNOV Cloud TP1",
        "hostname": socket.gethostname(),
        "environment": os.getenv("APP_ENV", "development"),
        # TODO: Ajouter un champ "version" avec la valeur "1.0.0"
        "version": "1.0.0"
    })

@app.route('/health')
def health():
    # TODO: Retourner un JSON {"status": "ok"} avec le code HTTP 200
    return jsonify({"status": "ok"}), 200

if __name__ == '__main__':
    # TODO: Lire le port depuis la variable d'environnement PORT (défaut: 8080)
    port = int(os.getenv("PORT", 8080))
    app.run(host='0.0.0.0', port=port)
```

`tp1-app/requirements.txt` — à compléter :

```text
flask==3.0.3 # Utiliser la version 3.0.3
gunicorn==21.2.0
```

## 5.2 — Écrire le Dockerfile

`tp1-app/Dockerfile` — à compléter :

```dockerfile
# Utiliser Python 3.12 slim comme image de base
FROM python:3.12-slim

# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier en premier le fichier requirements.txt (optimisation du cache Docker)
# Pourquoi copier requirements.txt séparément avant le reste du code ?
COPY requirements.txt .

# Installer les dépendances Python
RUN pip install --no-cache-dir -r requirements.txt

# Copier le reste du code source
COPY . .

# Exposer le port 8080
EXPOSE 8080

# Variable d'environnement par défaut
ENV APP_ENV=production

# Commande de démarrage avec gunicorn (serveur WSGI production)
# Format : gunicorn --bind 0.0.0.0:[PORT] [MODULE]:[OBJET_APP]
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]

```

**Question :** Pourquoi copie-t-on `requirements.txt` avant le code source dans le Dockerfile ?
**Réponse :** Comme cela ne change pas, sa permet a docker de les garder en cache.

## 5.3 — Créer un .dockerignore

`tp1-app/.dockerignore` — à compléter :

```text
# Exclure les fichiers inutiles pour réduire la taille de l'image
__pycache__
*.pyc
*.pyo
.env
venv # Exclure les environnements virtuels Python
.git
.gitignore
```

## 5.4 — Build et Run

```bash
cd tp1-app

# Builder l'image Docker
# -t : nommer l'image [nom]:[tag]
docker build -t tp1-flask:v1 .

# Vérifier que l'image a été créée
docker images | grep tp1-flask

# Lancer le conteneur
# -d : mode détaché (background)
# -p [port_local]:[port_conteneur] : mapping de ports
# --name : nommer le conteneur
# -e : variable d'environnement
docker run -d -p 8080:8080 --name tp1-container -e APP_ENV=development tp1-flask:v1

# Vérifier que le conteneur tourne
docker ps

# Tester l'application
curl http://localhost:8080/
curl http://localhost:8080/health
```

**Question :** Quelle est la différence entre une image Docker et un conteneur Docker ?
**Réponse :** L'image c'est un modèle qui contient le code et les dépendances,etc..., le conteneur c'est une instance de l'image.

## 5.5 — Nettoyage Docker

```bash
# Arrêter le conteneur
docker stop tp1-container

# Supprimer le conteneur (une fois arrêté)
docker rm tp1-container

# Vérifier qu'il n'y a plus de conteneur actif
docker ps
```

# Partie 6 — IAM & Service Accounts GCP (25 min)

IAM (Identity and Access Management) est le système de contrôle d'accès de GCP. Il répond à la question : "Qui peut faire quoi sur quelle ressource ?"

## 6.1 — Explorer les rôles IAM

```bash
# Lister les membres IAM de votre projet
gcloud projects get-iam-policy $(gcloud config get-value project)

# Lister les rôles prédéfinis disponibles pour Cloud Storage
gcloud iam roles list --filter="name:roles/storage" --format="table(name,title)"
```

**Question :** Quelle est la différence entre `roles/storage.admin` et `roles/storage.objectViewer` ?
**Réponse :** `roles/storage.admin` donne tous les droits (création, suppression, modification), tandis que `roles/storage.objectViewer` ne donne que des droits de lecture.

## 6.2 — Créer un Service Account

Un Service Account est une identité pour les applications (pas pour les humains). Convention : `[application]-sa@[project].iam.gserviceaccount.com`

```bash
PROJECT_ID=$(gcloud config get-value project)

# Créer un Service Account pour l'application Flask
gcloud iam service-accounts create tp1-app-sa \
--display-name="TP1 Flask App Service Account" \
--description="SA utilisé par l'application Flask pour accéder à GCS"

# Vérifier la création
gcloud iam service-accounts list
# Résultat attendu : liste incluant tp1-app-sa@[project].iam.gserviceaccount.com

PS C:\Users\maxf3\Desktop\Dev\DevelopCloud\TP1> gcloud iam service-accounts list                                                                                                 
DISPLAY NAME                            EMAIL                                                 DISABLED                                                                           
TP1 Flask App Service Account           tp1-app-sa@ynov-cloud-maxime.iam.gserviceaccount.com  False
--- ----- --- ------- -------           ----------------------------------------------------  -------
```

## 6.3 — Attribuer un rôle au Service Account

Principe du moindre privilège : donner uniquement les droits nécessaires, rien de plus.

```bash
PROJECT_ID=$(gcloud config get-value project)
SA_EMAIL="tp1-app-sa@${PROJECT_ID}.iam.gserviceaccount.com"

# Donner uniquement le droit de LIRE les objets GCS (pas d'écriture, pas d'admin)
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member="serviceAccount:${SA_EMAIL}" \
--role="roles/storage.objectViewer" # Utiliser objectViewer (lecture seule)

# Vérifier les bindings du projet
gcloud projects get-iam-policy ${PROJECT_ID} \
--flatten="bindings[].members" \
--filter="bindings.members:tp1-app-sa"
```

**Question :** Pourquoi ne faut-il pas donner le rôle `roles/owner` ou `roles/editor` à un Service Account applicatif ?
**Réponse :** C'est contraire au moindre privilège et ça peut mettre en danger le proget si le service account est compromis.

## 6.4 — Générer et utiliser une clé de Service Account

```bash
PROJECT_ID=$(gcloud config get-value project)
SA_EMAIL="tp1-app-sa@${PROJECT_ID}.iam.gserviceaccount.com"

# Générer une clé JSON (fichier de credentials)
gcloud iam service-accounts keys create /tmp/tp1-sa-key.json --iam-account=${SA_EMAIL}

# Vérifier que le fichier est créé
ls -lh /tmp/tp1-sa-key.json

# Activer le Service Account dans gcloud (différent de GOOGLE_APPLICATION_CREDENTIALS
# qui est réservé aux SDKs Python/Java/Go, pas à la CLI gcloud)
gcloud auth activate-service-account --key-file=/tmp/tp1-sa-key.json

# Vérifier que gcloud utilise bien le SA
gcloud auth list

# Tester : essayer de créer un bucket avec ce SA
# Cela doit ÉCHOUER car le SA n'a que roles/storage.objectViewer (lecture seule)
gcloud storage buckets create gs://test-sa-${PROJECT_ID} --location=europe-west9 2>&1 || echo "Accès refusé (attendu : le SA n'a pas les droits de création de bucket)"

# Remettre l'authentification de votre compte utilisateur
gcloud auth login

# Supprimer la clé (bonne pratique : éviter les clés qui traînent)
rm /tmp/tp1-sa-key.json
gcloud iam service-accounts keys list --iam-account=${SA_EMAIL}
```

**Question :** Quelle alternative aux clés JSON GCP recommande-t-on en production pour éviter de stocker des fichiers de credentials ?
**Réponse (indice : mécanisme "keyless") :** Utiliser les Services Accounts GCP pour l'authentification keyless.

## 6.5 — Supprimer le Service Account (nettoyage)

```bash
gcloud iam service-accounts delete tp1-app-sa@${PROJECT_ID}.iam.gserviceaccount.com --quiet
```

# Partie 7 — Docker : Commandes d'Inspection et Debug (20 min)

Cette partie étend le TP Docker avec les commandes essentielles pour débugger un conteneur en production.

## 7.1 — Relancer le conteneur Flask

```bash
cd tp1-app
docker run -d -p 8080:8080 --name tp1-debug -e APP_ENV=debug tp1-flask:v1

# Vérifier qu'il tourne
docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                         NAMES
80ac3a021a14   tp1-flask:v1   "gunicorn --bind 0.0…"   4 seconds ago   Up 4 seconds   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   tp1-debug
```

## 7.2 — Inspecter un conteneur

```bash
# Voir la configuration complète du conteneur (JSON détaillé)
docker inspect tp1-debug

# Extraire uniquement l'adresse IP du conteneur
docker inspect tp1-debug --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# Voir les variables d'environnement injectées
docker inspect tp1-debug --format='{{range .Config.Env}}{{println .}}{{end}}'
```

**Question :** quelle valeur a la variable APP_ENV dans le conteneur ?
**Réponse :** APP_ENV = debug

## 7.3 — Exécuter des commandes dans un conteneur actif

```bash
# Ouvrir un shell interactif dans le conteneur (comme SSH)
docker exec -it tp1-debug /bin/sh # Utiliser /bin/sh (pas bash sur slim)

# Une fois dans le conteneur :
# - Vérifier les processus actifs
ps aux

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2  32948 23168 ?        Ss   14:03   0:00 /usr/local/bin/python3.12 /usr/local/bin/gunicorn --bind 0.0.0.0:8080 app:app
root         7  0.0  0.3  43380 31104 ?        S    14:03   0:00 /usr/local/bin/python3.12 /usr/local/bin/gunicorn --bind 0.0.0.0:8080 app:app
root        14  0.0  0.0   2672  1536 pts/0    Ss   14:24   0:00 /bin/sh
root       120  0.0  0.0   6784  3712 pts/0    R+   14:24   0:00 ps aux

# - Vérifier les fichiers copiés
ls -la /app/

total 28
drwxr-xr-x 1 root root 4096 Mar 20 14:03 .
drwxr-xr-x 1 root root 4096 Mar 20 14:03 ..
-rwxr-xr-x 1 root root  168 Mar 20 13:12 .dockerignore
-rwxr-xr-x 1 root root  754 Mar 20 13:12 Dockerfile
drwxr-xr-x 2 root root 4096 Mar 20 14:03 __pycache__
-rwxr-xr-x 1 root root  739 Mar 20 11:56 app.py
-rwxr-xr-x 1 root root   58 Mar 20 11:56 requirements.txt

# - Vérifier les variables d'environnement
env | grep APP

APP_ENV=debug

# - Tester la connexion réseau depuis le conteneur
wget -qO- http://localhost:8080/health

{"status":"ok"}

# Quitter le conteneur (sans l'arrêter)
exit

# Exécuter une commande sans ouvrir un shell interactif
docker exec tp1-debug env | grep APP

APP_ENV=debug
```

## 7.4 — Surveiller les ressources consommées

```bash
# Voir les stats CPU/RAM en temps réel (Ctrl+C pour quitter)
docker stats tp1-debug

# Voir les statistiques une seule fois (pas de flux continu)
docker stats --no-stream tp1-debug
```

**Question :** combien de RAM consomme votre application Flask au repos ?
**Réponse :** 80MiB environ

```bash
# Voir les logs avec filtre temporel
docker logs --since="5m" tp1-debug # Logs des 5 dernières minutes
docker logs --tail=20 tp1-debug # 20 dernières lignes
```

## 7.5 — Analyser la taille de l'image par couche

```bash
# Voir l'historique des layers et leur taille
docker history tp1-flask:v1
```

**Question :** quelle couche est la plus volumineuse et pourquoi ?
**Réponse :** C'est l'image de base qui comporte l'OS

```bash
# Nettoyage
docker stop tp1-debug && docker rm tp1-debug
```
