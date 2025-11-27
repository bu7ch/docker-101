
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
=> app.py
```python
from flask import Flask, request, render_template
import redis, os
r = redis.Redis(host=os.getenv("REDIS_HOST","redis"), decode_responses=True)
app = Flask(__name__)

@app.route("/", methods=["GET","POST"])
def index():
    if request.method == "POST":
        vote = request.form.get("vote")
        if vote == "a": r.incr("votes_a")
        if vote == "b": r.incr("votes_b")
    va = int(r.get("votes_a") or 0)
    vb = int(r.get("votes_b") or 0)
    return render_template("index.html", a=va, b=vb)
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
=> index.html
```html
<!doctype html>
<html><body>
<h1>Vote A ou B</h1>
<form method="post">
  <button name="vote" value="a">A</button>
  <button name="vote" value="b">B</button>
</form>
<p>A : {{ a }} — B : {{ b }}</p>
</body></html>
```
=> requirement.txt
```bash
flask==3.0.2
redis==5.0.1
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
=> dockerfile
```bash
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
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
