
# EX05 – Orchestrer une mini-app avec Docker Compose

Objectif  
Déployer en 1 commande :
- un front Python (vote) sur port 5000
- Redis (backend)

Fichiers fournis
```
vote/
  app.py
  requirements.txt
  templates/index.html
docker-compose.yml
```

docker-compose.yml (minimal)
```yaml
version: "3.9"
services:
  redis:
    image: redis:7-alpine
  vote:
    build: ./vote
    ports:
      - "5000:5000"
    environment:
      REDIS_HOST: redis
    depends_on:
      - redis
```

Étapes
1. Place-toi à la racine du dossier `ex05-vote-compose`.
2. Build & up :
   ```bash
   docker compose up --build -d
   ```
3. Ouvre http://localhost:5000, vote A ou B.
4. Vérifie les clés Redis :
   ```bash
   docker compose exec redis redis-cli keys '*'
   ```

Critères de réussite  
✓ Page de vote accessible  
✓ Redis contient des clés `votes_a` / `votes_b`

Nettoyage
```bash
docker compose down --volumes
```
