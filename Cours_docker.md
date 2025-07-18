# Cours Complet Docker - Débutant à Avancé

## Table des Matières
1. [Introduction et Concepts Fondamentaux](#introduction)
2. [Installation et Configuration](#installation)
3. [Commandes Docker de Base](#commandes-base)
4. [Images Docker](#images)
5. [Conteneurs](#conteneurs)
6. [Dockerfiles](#dockerfiles)
7. [Volumes et Stockage](#volumes)
8. [Réseaux Docker](#reseaux)
9. [Docker Compose](#compose)
10. [Bonnes Pratiques](#bonnes-pratiques)
11. [Concepts Avancés](#avance)
12. [Déploiement et Production](#production)
13. [Sécurité](#securite)
14. [Monitoring et Logs](#monitoring)
15. [Projets Pratiques](#projets)

---

## 1. Introduction et Concepts Fondamentaux {#introduction}

### Qu'est-ce que Docker ?
Docker est une plateforme de containerisation qui permet d'empaqueter une application et ses dépendances dans un conteneur léger et portable.

### Concepts Clés

**Conteneur** : Instance d'exécution d'une image Docker
**Image** : Modèle read-only utilisé pour créer des conteneurs
**Dockerfile** : Script contenant les instructions pour construire une image
**Registry** : Dépôt d'images Docker (Docker Hub, par exemple)

### Avantages de Docker
- **Portabilité** : "Ça marche sur ma machine" → "Ça marche partout"
- **Isolation** : Chaque conteneur est isolé
- **Efficacité** : Moins de ressources qu'une VM
- **Scalabilité** : Facilite la montée en charge
- **Cohérence** : Même environnement du développement à la production

---

## 2. Installation et Configuration {#installation}

### Vérification de l'installation
```bash
docker --version
docker info
docker run hello-world
```

### Configuration de base
```bash
# Voir les informations système
docker system info

# Configurer Docker pour démarrer automatiquement
# (Sur Windows, Docker Desktop se lance automatiquement)
```

---

## 3. Commandes Docker de Base {#commandes-base}

### Commandes essentielles

#### Gestion des images
```bash
# Lister les images
docker images
docker image ls

# Télécharger une image
docker pull nginx
docker pull node:18

# Supprimer une image
docker rmi nginx
docker image rm nginx

# Rechercher une image
docker search nginx
```

#### Gestion des conteneurs
```bash
# Lancer un conteneur
docker run nginx
docker run -d nginx  # en arrière-plan (detached)
docker run -it ubuntu bash  # interactif avec terminal

# Lister les conteneurs
docker ps          # conteneurs actifs
docker ps -a       # tous les conteneurs

# Arrêter un conteneur
docker stop <container_id>
docker stop <container_name>

# Supprimer un conteneur
docker rm <container_id>
docker rm <container_name>

# Supprimer tous les conteneurs arrêtés
docker container prune
```

#### Examiner les conteneurs
```bash
# Voir les logs
docker logs <container_id>
docker logs -f <container_id>  # en temps réel

# Accéder au terminal d'un conteneur
docker exec -it <container_id> bash
docker exec -it <container_id> sh

# Inspecter un conteneur
docker inspect <container_id>

# Voir les statistiques
docker stats <container_id>
```

---

## 4. Images Docker {#images}

### Comprendre les images
Les images Docker sont composées de couches (layers) empilées.

```bash
# Voir l'historique d'une image
docker history nginx

# Inspecter une image
docker image inspect nginx

# Voir les couches d'une image
docker image inspect nginx | grep -A 10 "Layers"
```

### Gérer les images
```bash
# Taguer une image
docker tag nginx:latest mon-nginx:1.0

# Construire une image depuis un Dockerfile
docker build -t mon-app .
docker build -t mon-app:v1.0 .

# Pousser une image vers un registry
docker push mon-utilisateur/mon-app:v1.0
```

---

## 5. Conteneurs {#conteneurs}

### Options de lancement importantes
```bash
# Port mapping
docker run -p 8080:80 nginx
docker run -p 127.0.0.1:8080:80 nginx

# Variables d'environnement
docker run -e ENV_VAR=value nginx
docker run --env-file .env nginx

# Nom du conteneur
docker run --name mon-nginx nginx

# Redémarrage automatique
docker run --restart always nginx
docker run --restart unless-stopped nginx

# Limitation des ressources
docker run --memory="512m" --cpus="1.0" nginx
```

### Cycles de vie des conteneurs
```bash
# Créer sans démarrer
docker create --name mon-conteneur nginx

# Démarrer un conteneur créé
docker start mon-conteneur

# Redémarrer
docker restart mon-conteneur

# Pause/Unpause
docker pause mon-conteneur
docker unpause mon-conteneur

# Arrêter proprement
docker stop mon-conteneur

# Forcer l'arrêt
docker kill mon-conteneur
```

---

## 6. Dockerfiles {#dockerfiles}

### Structure d'un Dockerfile
```dockerfile
# Dockerfile basique
FROM node:18-alpine

# Métadonnées
LABEL maintainer="votre-email@example.com"
LABEL version="1.0"

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers
COPY package*.json ./
COPY . .

# Installer les dépendances
RUN npm install

# Exposer un port
EXPOSE 3000

# Définir l'utilisateur
USER node

# Commande par défaut
CMD ["npm", "start"]
```

### Instructions principales

#### FROM
```dockerfile
FROM node:18-alpine
FROM ubuntu:22.04
FROM scratch  # image vide
```

#### RUN
```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN npm install
RUN pip install -r requirements.txt
```

#### COPY et ADD
```dockerfile
COPY source destination
COPY package*.json ./
COPY . .

ADD https://example.com/file.tar.gz /app/
# ADD peut décompresser automatiquement
```

#### ENV et ARG
```dockerfile
# Variables d'environnement
ENV NODE_ENV=production
ENV PORT=3000

# Arguments de build
ARG VERSION=1.0
ARG BUILD_DATE
```

#### EXPOSE
```dockerfile
EXPOSE 3000
EXPOSE 8080/tcp
EXPOSE 53/udp
```

#### USER
```dockerfile
USER node
USER 1000:1000
```

#### VOLUME
```dockerfile
VOLUME /app/data
VOLUME ["/app/data", "/app/logs"]
```

#### ENTRYPOINT vs CMD
```dockerfile
# CMD peut être écrasé
CMD ["npm", "start"]
CMD npm start

# ENTRYPOINT ne peut pas être écrasé
ENTRYPOINT ["npm", "start"]

# Combinaison
ENTRYPOINT ["npm"]
CMD ["start"]
```

### Dockerfile multi-étapes
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Production
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Optimisation des Dockerfiles
```dockerfile
# Mauvais
COPY . .
RUN npm install

# Bon (cache des layers)
COPY package*.json ./
RUN npm install
COPY . .
```

---

## 7. Volumes et Stockage {#volumes}

### Types de stockage
1. **Volumes** : Gérés par Docker (recommandé)
2. **Bind mounts** : Répertoires de l'hôte
3. **tmpfs** : Mémoire temporaire

### Volumes Docker
```bash
# Créer un volume
docker volume create mon-volume

# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect mon-volume

# Utiliser un volume
docker run -v mon-volume:/app/data nginx
docker run --mount source=mon-volume,target=/app/data nginx

# Supprimer un volume
docker volume rm mon-volume
docker volume prune  # supprimer tous les volumes non utilisés
```

### Bind mounts
```bash
# Windows
docker run -v C:\mon\chemin:/app/data nginx
docker run -v ${PWD}:/app nginx

# Linux/Mac
docker run -v /home/user/data:/app/data nginx
docker run -v $(pwd):/app nginx
```

### Volumes dans Docker Compose
```yaml
version: '3.8'
services:
  app:
    image: nginx
    volumes:
      - app-data:/app/data
      - ./config:/app/config:ro  # read-only
      - type: tmpfs
        target: /app/tmp

volumes:
  app-data:
    driver: local
```

---

## 8. Réseaux Docker {#reseaux}

### Types de réseaux
1. **bridge** : Réseau par défaut
2. **host** : Utilise le réseau de l'hôte
3. **none** : Pas de réseau
4. **overlay** : Pour Docker Swarm
5. **macvlan** : Attribution d'adresse MAC

### Gestion des réseaux
```bash
# Lister les réseaux
docker network ls

# Créer un réseau
docker network create mon-reseau
docker network create --driver bridge mon-reseau-bridge

# Inspecter un réseau
docker network inspect mon-reseau

# Connecter un conteneur à un réseau
docker run --network mon-reseau nginx
docker network connect mon-reseau mon-conteneur

# Déconnecter
docker network disconnect mon-reseau mon-conteneur

# Supprimer un réseau
docker network rm mon-reseau
```

### Communication entre conteneurs
```bash
# Créer un réseau personnalisé
docker network create app-network

# Lancer des conteneurs sur ce réseau
docker run -d --name database --network app-network postgres
docker run -d --name webapp --network app-network nginx

# Les conteneurs peuvent se parler par leur nom
# webapp peut accéder à database via "http://database:5432"
```

### Exposition des ports
```bash
# Port mapping
docker run -p 8080:80 nginx           # localhost:8080 → conteneur:80
docker run -p 127.0.0.1:8080:80 nginx # seulement localhost
docker run -P nginx                    # ports aléatoires

# Voir les ports mappés
docker port mon-conteneur
```

---

## 9. Docker Compose {#compose}

### Qu'est-ce que Docker Compose ?
Docker Compose permet de définir et gérer des applications multi-conteneurs avec un fichier YAML.

### Fichier docker-compose.yml basique
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - database
    environment:
      - ENV=production
    networks:
      - app-network

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

### Commandes Docker Compose
```bash
# Démarrer les services
docker-compose up
docker-compose up -d  # en arrière-plan
docker-compose up --build  # reconstruire les images

# Arrêter les services
docker-compose down
docker-compose down -v  # supprimer aussi les volumes

# Voir les logs
docker-compose logs
docker-compose logs web  # logs d'un service spécifique

# Lister les services
docker-compose ps

# Exécuter une commande dans un service
docker-compose exec web bash
docker-compose exec database psql -U user -d myapp

# Redémarrer un service
docker-compose restart web

# Construire les images
docker-compose build
docker-compose build --no-cache
```

### Exemple complet - Application MEAN
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:5.0
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongodb_data:/data/db
    ports:
      - "27017:27017"
    networks:
      - mean-network

  backend:
    build: ./backend
    restart: unless-stopped
    depends_on:
      - mongodb
    environment:
      - NODE_ENV=production
      - MONGODB_URL=mongodb://admin:password@mongodb:27017/myapp?authSource=admin
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    networks:
      - mean-network

  frontend:
    build: ./frontend
    restart: unless-stopped
    depends_on:
      - backend
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "4200:4200"
    networks:
      - mean-network

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    depends_on:
      - frontend
      - backend
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    networks:
      - mean-network

volumes:
  mongodb_data:

networks:
  mean-network:
    driver: bridge
```

---

## 10. Bonnes Pratiques {#bonnes-pratiques}

### Dockerfile
```dockerfile
# ✅ Utiliser des images officielles
FROM node:18-alpine

# ✅ Utiliser des tags spécifiques
FROM node:18.16.0-alpine

# ✅ Créer un utilisateur non-root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# ✅ Optimiser les couches
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# ✅ Nettoyer après installation
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# ✅ Utiliser .dockerignore
# Créer un fichier .dockerignore
```

### Fichier .dockerignore
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
.vscode
.DS_Store
```

### Sécurité
```dockerfile
# ✅ Scanner les vulnérabilités
# docker scan mon-image

# ✅ Utiliser des images minimales
FROM alpine:3.18
FROM distroless/java:11

# ✅ Ne pas exposer de données sensibles
# Éviter COPY .env ou ADD secrets.txt

# ✅ Utiliser des secrets Docker
# docker run --secret id=mysecret,src=./secret.txt mon-app
```

### Performance
```dockerfile
# ✅ Multi-stage builds
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS production
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]

# ✅ Optimiser la taille des images
RUN npm ci --only=production && npm cache clean --force
```

---

## 11. Concepts Avancés {#avance}

### Docker Swarm
```bash
# Initialiser un swarm
docker swarm init

# Ajouter un worker
docker swarm join --token <token> <manager-ip>:2377

# Déployer un service
docker service create --name web --replicas 3 -p 80:80 nginx

# Mettre à jour un service
docker service update --image nginx:alpine web

# Scaler un service
docker service scale web=5
```

### Healthchecks
```dockerfile
# Dans le Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

```yaml
# Dans docker-compose.yml
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Secrets et Configs
```bash
# Créer un secret
echo "mot-de-passe-secret" | docker secret create db-password -

# Utiliser un secret
docker service create --name db --secret db-password postgres
```

### Registry privé
```bash
# Démarrer un registry local
docker run -d -p 5000:5000 --name registry registry:2

# Taguer et pousser vers le registry privé
docker tag mon-app localhost:5000/mon-app
docker push localhost:5000/mon-app
```

---

## 12. Déploiement et Production {#production}

### Environnements
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    image: mon-app:latest
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### CI/CD avec Docker
```yaml
# .github/workflows/docker.yml
name: Docker Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t mon-app:${{ github.sha }} .
    
    - name: Run tests
      run: docker run --rm mon-app:${{ github.sha }} npm test
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push mon-app:${{ github.sha }}
```

---

## 13. Sécurité {#securite}

### Sécurisation des conteneurs
```bash
# Utiliser un utilisateur non-root
docker run --user 1000:1000 nginx

# Limiter les capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Mode read-only
docker run --read-only nginx

# Pas de nouveaux privilèges
docker run --security-opt=no-new-privileges nginx
```

### Scan de sécurité
```bash
# Scanner une image (Docker Desktop)
docker scan mon-image:latest

# Utiliser des outils tiers
# trivy image mon-image:latest
# snyk container test mon-image:latest
```

---

## 14. Monitoring et Logs {#monitoring}

### Gestion des logs
```bash
# Voir les logs
docker logs conteneur
docker logs -f conteneur  # suivre en temps réel
docker logs --tail 50 conteneur  # 50 dernières lignes

# Configuration des logs
docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 nginx
```

### Monitoring avec Prometheus
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

---

## 15. Projets Pratiques {#projets}

### Projet 1 : Application Web Simple
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - redis
      - db

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db-data:
```

### Projet 2 : Stack ELK (Elasticsearch, Logstash, Kibana)
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    depends_on:
      - elasticsearch
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    depends_on:
      - elasticsearch
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

volumes:
  elasticsearch-data:
```

### Projet 3 : Microservices avec API Gateway
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - product-service
      - order-service

  user-service:
    build: ./user-service
    environment:
      - DB_HOST=user-db
    depends_on:
      - user-db

  product-service:
    build: ./product-service
    environment:
      - DB_HOST=product-db
    depends_on:
      - product-db

  order-service:
    build: ./order-service
    environment:
      - USER_SERVICE_URL=http://user-service:3000
      - PRODUCT_SERVICE_URL=http://product-service:3000
    depends_on:
      - user-service
      - product-service

  user-db:
    image: postgres:13
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

  product-db:
    image: postgres:13
    environment:
      POSTGRES_DB: products
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

---

## Commandes de Référence Rapide

### Images
```bash
docker images                    # Lister les images
docker pull <image>              # Télécharger une image
docker rmi <image>               # Supprimer une image
docker build -t <tag> .          # Construire une image
docker tag <image> <new-tag>     # Taguer une image
```

### Conteneurs
```bash
docker run <image>               # Lancer un conteneur
docker run -d <image>            # Lancer en arrière-plan
docker run -it <image> bash      # Lancer avec terminal interactif
docker ps                        # Conteneurs actifs
docker ps -a                     # Tous les conteneurs
docker stop <container>          # Arrêter un conteneur
docker rm <container>            # Supprimer un conteneur
docker exec -it <container> bash # Accéder au terminal
```

### Volumes
```bash
docker volume create <volume>    # Créer un volume
docker volume ls                 # Lister les volumes
docker volume rm <volume>        # Supprimer un volume
```

### Réseaux
```bash
docker network create <network>  # Créer un réseau
docker network ls                # Lister les réseaux
docker network rm <network>      # Supprimer un réseau
```

### Docker Compose
```bash
docker-compose up                # Démarrer les services
docker-compose up -d             # Démarrer en arrière-plan
docker-compose down              # Arrêter les services
docker-compose build             # Construire les images
docker-compose logs              # Voir les logs
```

### Nettoyage
```bash
docker system prune              # Nettoyer le système
docker container prune           # Supprimer les conteneurs arrêtés
docker image prune               # Supprimer les images non utilisées
docker volume prune              # Supprimer les volumes non utilisés
docker network prune             # Supprimer les réseaux non utilisés
```

---

## Prochaines Étapes

1. **Pratiquez** chaque section avec des exemples concrets
2. **Créez** vos propres Dockerfiles pour vos applications
3. **Explorez** Docker Hub pour découvrir des images utiles
4. **Apprenez** Kubernetes pour l'orchestration à grande échelle
5. **Intégrez** Docker dans votre pipeline CI/CD

## Ressources Complémentaires

- [Documentation officielle Docker](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile best practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Compose documentation](https://docs.docker.com/compose/)

---

*Ce cours couvre les aspects essentiels de Docker. Pratiquez régulièrement et n'hésitez pas à expérimenter avec différentes configurations pour maîtriser parfaitement cet outil puissant.*