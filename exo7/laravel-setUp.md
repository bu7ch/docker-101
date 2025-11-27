# **EX07 – Environnement de développement Laravel avec Docker**

## **🎯 Objectif pédagogique**
Maîtriser la containerisation d'un environnement de développement web complet avec Docker Compose, en utilisant Laravel comme cas pratique.

**Compétences visées :**
- Orchestrer multiple services avec Docker Compose
- Configurer un environnement PHP/MySQL/Redis
- Comprendre les volumes pour le développement
- Gérer les dépendances entre conteneurs

---

## **📚 Prérequis**
- Docker et Docker Compose installés
- Connaissances basiques de PHP/Laravel
- Connaissances des commandes Linux

---

## **📁 Structure du projet à créer**
```
laravel-dev/
├── docker-compose.yml      # Orchestration des services
├── nginx/
│   └── default.conf        # Configuration du serveur web
├── mysql/
│   └── init.sql           # Initialisation de la BDD
├── php/
│   └── Dockerfile         # Image PHP personnalisée
├── .env                   # Variables d'environnement
└── src/                   # Code source Laravel (vide au départ)
```

---

## **📋 Instructions détaillées**

### **Étape 1 : Préparation de l'environnement**

1. **Créer la structure de dossiers**
```bash
mkdir laravel-dev
cd laravel-dev
mkdir -p nginx mysql php src
```

### **Étape 2 : Configuration Docker Compose**

2. **Créer `docker-compose.yml`**
```yaml
version: '3.8'

services:
  app:
    build:
      context: ./php
      dockerfile: Dockerfile
    volumes:
      - ./src:/var/www/html
    environment:
      - DB_HOST=mysql
      - REDIS_HOST=redis
    depends_on:
      - mysql
      - redis

  webserver:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./src:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: laravel
      MYSQL_DATABASE: laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: laravel
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  node:
    image: node:18-alpine
    volumes:
      - ./src:/app
    working_dir: /app
    command: ["sh", "-c", "npm install && npm run dev"]
    ports:
      - "5173:5173"
    depends_on:
      - app

volumes:
  mysql_data:

networks:
  default:
    driver: bridge
```

### **Étape 3 : Configuration des services**

3. **Créer `php/Dockerfile`**
```dockerfile
FROM php:8.2-fpm-alpine

RUN apk update && apk add --no-cache \
    git curl libpng-dev libxml2-dev \
    zip unzip oniguruma-dev nodejs npm \
    mariadb-dev linux-headers

RUN docker-php-ext-install \
    pdo_mysql mbstring exif pcntl \
    bcmath gd sockets

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN adduser -D -u 1000 laravel
USER laravel

WORKDIR /var/www/html

```

4. **Créer `nginx/default.conf`**
```nginx
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    client_max_body_size 100M;
}
```

5. **Créer `mysql/init.sql`**
```sql
CREATE DATABASE IF NOT EXISTS laravel;
GRANT ALL PRIVILEGES ON laravel.* TO 'laravel'@'%';
FLUSH PRIVILEGES;
```

### **Étape 4 : Déploiement initial**

6. **Lancer l'environnement**
```bash
docker compose up -d --build
```

7. **Vérifier que tout fonctionne**
```bash
docker compose ps
```

### **Étape 5 : Installation de Laravel**

8. **Installer Laravel dans le conteneur**
```bash
docker compose exec app sh -c "
composer create-project laravel/laravel . &&
php artisan key:generate &&
chmod -R 775 storage bootstrap/cache
"
```

9. **Configurer l'environnement Laravel**
```bash
# Copier le fichier d'exemple
cp src/.env.example src/.env

# Éditer le fichier .env pour qu'il corresponde à :
cat > src/.env << EOF
APP_NAME=LaravelDev
APP_ENV=local
APP_KEY=base64:... (généré par artisan)
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
EOF
```

### **Étape 6 : Finalisation**

10. **Installer les dépendances frontend**
```bash
docker compose exec node npm install
```

11. **Tester l'application**
```bash
# Tester l'accès web
curl http://localhost:8000

# Tester la base de données
docker compose exec app php artisan migrate:status

# Tester Redis
docker compose exec redis redis-cli ping
```

---

## **🔍 Points d'attention pédagogiques**

### **Concepts clés à expliquer :**
- **Réseaux Docker** : Comment les services communiquent entre eux
- **Volumes** : Persistance des données et partage de code
- **Dépendances** : Gestion du démarrage avec `depends_on`
- **Environnement** : Variables d'environnement pour la configuration

### **Questions de compréhension :**
1. Pourquoi utilise-t-on un volume pour le dossier `src` ?
2. Quel est le rôle de `depends_on` dans docker-compose ?
3. Comment PHP-FPM communique-t-il avec Nginx ?
4. Pourquoi créer un utilisateur non-root dans le Dockerfile ?

---

## **🐛 Débuggage common**

**Problème** : Laravel ne se connecte pas à MySQL
```bash
# Solution : Vérifier les logs MySQL
docker compose logs mysql
```

**Problème** : Nginx erreur 502
```bash
# Solution : Vérifier que PHP-FPM tourne
docker compose ps app
```

**Problème** : Permissions des fichiers
```bash
# Solution : Ajuster les permissions
docker compose exec app chmod -R 775 storage bootstrap/cache
```

---

## **🚀 Bonus (optionnel)**

### **Ajouter Xdebug pour le debugging**
```dockerfile
# Ajouter dans php/Dockerfile
RUN apk add --no-cache $PHPIZE_DEPS \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug
```

### **Ajouter un service Mailhog pour tester les emails**
```yaml
mailhog:
  image: mailhog/mailhog
  ports:
    - "8025:8025"
```

---

## **🧹 Nettoyage final**
```bash
docker compose down --volumes --remove-orphans
```

---

