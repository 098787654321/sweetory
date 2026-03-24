# Sweetory

Projet web complet pour une patissiere:
- Frontend client (catalogue, configurateur canvas, commande, suivi)
- Frontend vendeur (login, dashboard, gestion produits/options/commandes)
- Backend Java (Servlets Jakarta sur Tomcat)
- Base de donnees relationnelle (MySQL/MariaDB)

## Structure

```text
sweetory/
|- client/
|  |- backend/
|  |- database/
|  \- frontend/
|     |- index.html
|     |- catalogue.html
|     |- configurateur.html
|     |- commande.html
|     |- suivi.html
|     |- css/
|     \- js/
|- vendeur/
|  |- backend/
|  |- database/
|  \- frontend/
|     |- login.html
|     |- dashboard.html
|     |- gestion-produits.html
|     |- gestion-options.html
|     |- commandes.html
|     |- css/
|     \- js/
|- backend/
|  \- java/
|     |- pom.xml
|     \- src/main/java/com/sweetory
|- database/
|  |- sweetory_schema.sql
|  \- sweetory_seed.sql
\- logo/
   \- logo-favicon.jpg
```

## Frontend rapide

1. Pour un apercu visuel simple, ouvrir `client/frontend/index.html` directement.
2. Pour tester les fonctions connectees au backend (`/api/*`), servir les dossiers `client/` et `vendeur/` sur le meme host que Tomcat (ex: `http://localhost:8080/...`).
3. Entree admin:
   - Email: `admin@sweetory.com`
   - Mot de passe: `sweetory123`

## Backend Java (Tomcat)

Pre-requis:
- JDK 25
- Maven 3.9+
- Apache Tomcat 11 (Jakarta Servlet 6+)

Commandes:

```bash
cd backend/java
mvn clean package
```

Resultat:
- WAR genere: `backend/java/target/sweetory-api.war`
- Deposer ce WAR dans `webapps` de Tomcat.

## API principale

Client:
- `GET /api/cakes`
- `GET /api/options`
- `POST /api/orders/create`
- `GET /api/orders?phone=...`
- `GET /api/orders/{id}`

Vendeur:
- `POST /api/admin/login`
- `POST /api/admin/logout`
- `GET /api/admin/session`
- `GET /api/admin/cakes`
- `POST /api/admin/cakes/add`
- `PUT /api/admin/cakes/update`
- `DELETE /api/admin/cakes/delete`
- `GET /api/admin/orders`
- `GET /api/admin/options`
- `POST /api/admin/options`
- `DELETE /api/admin/options`
- `PATCH /api/admin/orders/status`

Le frontend vendeur (`vendeur/frontend/js/admin.js`) est connecte a ces routes backend en HTTP (plus de fallback localStorage).

## Base de donnees

1. Executer `database/sweetory_schema.sql`
2. Executer `database/sweetory_seed.sql`

La couche Java utilise maintenant une persistance MySQL JDBC (avec fallback automatique InMemory si MySQL indisponible).

Variables de configuration (optionnelles):
- `SWEETORY_DB_ENABLED` (`true` par defaut)
- `SWEETORY_DB_URL` (defaut: `jdbc:mysql://127.0.0.1:3306/sweetory?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC`)
- `SWEETORY_DB_USER` (defaut: `root`)
- `SWEETORY_DB_PASSWORD` (defaut: vide)

### Activation rapide persistance reelle (Mac)

```bash
# 1) Initialiser la base (schema + seed)
mysql -u root -p < database/sweetory_schema.sql
mysql -u root -p < database/sweetory_seed.sql

# 2) Build backend (le driver MySQL est inclus dans le WAR)
cd backend/java
mvn -DskipTests clean package

# 3) (Optionnel) Forcer la config DB via variables d'environnement
export SWEETORY_DB_ENABLED=true
export SWEETORY_DB_URL='jdbc:mysql://127.0.0.1:3306/sweetory?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC'
export SWEETORY_DB_USER='root'
export SWEETORY_DB_PASSWORD=''
```

Ensuite redeployer `target/sweetory-api.war` dans Tomcat.
Si MySQL n'est pas accessible, l'application bascule automatiquement en InMemory (sans arret du site), mais les donnees ne seront alors pas conservees au redemarrage.

## Publication Internet 100% gratuite (Oracle Cloud Always Free)

Option recommandee pour rester a 0 DA et garder une vraie base MySQL persistante.

### 1) Creer une VM gratuite Oracle Cloud

- Creer un compte Oracle Cloud Free Tier
- Creer une VM Ubuntu (Always Free, shape Ampere A1 si disponible)
- Ouvrir les ports `80` (HTTP) et `22` (SSH) dans la security list / NSG

### 2) Sur la VM, installer Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin git
sudo usermod -aG docker $USER
newgrp docker
```

### 3) Deployer Sweetory avec Docker Compose

```bash
git clone <URL_DE_TON_REPO_GIT> sweetory
cd sweetory

# Optionnel: personnaliser les mots de passe MySQL
export MYSQL_PASSWORD='change-me-strong'
export MYSQL_ROOT_PASSWORD='change-me-root-strong'

docker compose up -d --build
```

Le site sera en ligne sur:
- `http://<IP_PUBLIQUE_VM>/client/frontend/index.html`
- `http://<IP_PUBLIQUE_VM>/vendeur/frontend/login.html`

### 4) Mettre a jour

```bash
cd sweetory
git pull
docker compose up -d --build
```
