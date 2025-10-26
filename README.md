---

# Plateforme Universitaire Libre et Composable (Open University Platform)

Ce projet est une preuve de concept (PoC) pour une architecture technique complète dédiée à la gestion universitaire. Elle est construite entièrement à partir de solutions open-source, déployées dans des conteneurs Docker et orchestrées avec Docker Compose.

L'objectif est de fournir une plateforme modulaire, sécurisée et facile à reproduire, tout en s'appuyant sur des pratiques modernes d'automatisation et d'observabilité.

## Architecture

La plateforme est conçue autour d'une architecture microservices où chaque service est un conteneur indépendant :

-   **Reverse Proxy :** **Traefik** est utilisé comme point d'entrée unique. Il route le trafic vers les services appropriés en se basant sur les noms de domaine et gère la découverte automatique des services.
-   **Services applicatifs :** Chaque application (Moodle, Drupal, etc.) tourne dans son propre conteneur.
-   **Bases de données :** Chaque service nécessitant une base de données utilise son propre conteneur de base de données dédié pour un meilleur isolement.
-   **Monitoring :** Une stack de monitoring complète est intégrée avec **Prometheus** pour la collecte de métriques, **Grafana** pour la visualisation et **cAdvisor** pour la surveillance des conteneurs.
-   **Persistance :** Les données sont rendues persistantes grâce à des volumes Docker nommés, garantissant qu'aucune donnée n'est perdue lors du redémarrage des conteneurs.

## Services Inclus

| Service | Description | URL d'accès |
| :--- | :--- | :--- |
| **Traefik** | Reverse Proxy et Dashboard de routage | `http://localhost:8080` |
| **Drupal** | CRM, portail vitrine et gestion des admissions | `http://crm.university.local` |
| **Moodle** | Plateforme d'enseignement en ligne (LMS) | `http://lms.university.local` |
| **Nextcloud** | Partage et gestion de fichiers | `http://files.university.local` |
| **Keycloak** | Gestion des identités et authentification unique (SSO) | `http://sso.university.local` |
| **n8n** | Automatisation des workflows et échange de données | `http://workflow.university.local` |
| **Kasm** | Laboratoires et bureaux à distance via le navigateur | `http://labs.university.local` |
| **Prometheus** | Collecte de métriques et monitoring | `http://monitoring.university.local`|
| **Grafana**| Visualisation des données et tableaux de bord | `http://grafana.university.local` |
| **cAdvisor** | Métriques de performance des conteneurs | `http://cadvisor.university.local` |
| **MySQL Exporter** | Export des métriques de la base de données Moodle | (Utilisé par Prometheus) |

## Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :
-   [Docker Engine](https://docs.docker.com/engine/install/)
-   [Docker Compose](https://docs.docker.com/compose/install/)

## Installation

Suivez ces étapes pour configurer et lancer la plateforme.

### 1. Cloner le projet
Clonez ce dépôt sur votre machine locale.

### 2. Configurer l'environnement

Créez un fichier `.env` à la racine du projet en copiant le modèle ci-dessous. **Remplissez toutes les valeurs `...` avec des mots de passe sécurisés.**

```env
# Drupal Variables
DRUPAL_DB_NAME=drupaldb
DRUPAL_DB_USER=drupaluser
DRUPAL_DB_PASSWORD=...

# Moodle Variables
MOODLE_DB_NAME=moodledb
MOODLE_DB_USER=moodleuser
MOODLE_DB_PASSWORD=...
MOODLE_DB_ROOT_PASSWORD=...

# Nextcloud Variables
NEXTCLOUD_DB_NAME=nextclouddb
NEXTCLOUD_DB_USER=nextclouduser
NEXTCLOUD_DB_PASSWORD=...

# Keycloak Variables
KEYCLOAK_DB_NAME=keycloakdb
KEYCLOAK_DB_USER=keycloakuser
KEYCLOAK_DB_PASSWORD=...
KEYCLOAK_ADMIN_USER=admin
KEYCLOAK_ADMIN_PASSWORD=...

# n8n Variables
N8N_DB_NAME=n8ndb
N8N_DB_USER=n8nuser
N8N_DB_PASSWORD=...

# KASM Variables
KASM_ADMIN_PASSWORD=admin
```

### 3. Configurer les fichiers externes

Assurez-vous que les fichiers de configuration suivants sont présents à la racine du projet :
-   `prometheus.yml` (pour la configuration des cibles Prometheus)
-   `mysqld-exporter.cnf` (pour la connexion à la base de données Moodle)

### 4. Modifier le fichier `hosts`

Pour que votre machine puisse accéder aux services via leurs noms de domaine, vous devez modifier votre fichier `hosts`.

-   **Sur Linux/macOS :** `/etc/hosts`
-   **Sur Windows :** `C:\Windows\System32\drivers\etc\hosts`

Ajoutez la ligne suivante à la fin du fichier :
```
127.0.0.1 crm.university.local portail.university.local lms.university.local files.university.local sso.university.local workflow.university.local monitoring.university.local cadvisor.university.local grafana.university.local labs.university.local
```

### 5. Lancer la plateforme

Ouvrez un terminal à la racine du projet et exécutez la commande suivante :

```bash
docker-compose up -d --build
```
-   `-d` lance les conteneurs en arrière-plan.
-   `--build` est nécessaire pour construire l'image personnalisée de Drupal.

Le premier démarrage peut prendre plusieurs minutes, car Docker doit télécharger toutes les images et Kasm doit initialiser ses propres images de bureau.

## Utilisation des services

Une fois les conteneurs démarrés, vous pouvez accéder aux services via les URLs listées dans le tableau ci-dessus.

-   **Grafana :**
    -   Utilisateur : `admin`
    -   Mot de passe : `admin` (il vous sera demandé de le changer à la première connexion).
-   **Kasm :**
    -   Utilisateur : `kasm_admin@kasm.local`
    -   Mot de passe : celui que vous avez défini dans `KASM_ADMIN_PASSWORD`.

## Monitoring et Observabilité

-   **Traefik Dashboard :** `http://localhost:8080` pour voir l'état du routage.
-   **Prometheus :** `http://monitoring.university.local` pour explorer les métriques. Allez dans **Status -> Targets** pour vérifier que tous les services sont bien surveillés.
-   **Grafana :** `http://grafana.university.local` pour les tableaux de bord. N'hésitez pas à importer des dashboards depuis la communauté Grafana (par exemple, ID `13978` pour Docker, `7362` pour MySQL).
