# EX06 – Réduire la taille d’image et scanner les CVE

Objectif  
Repartir de l’image Flask de l’ex03, basculer sur `python:3.11-alpine`, comparer tailles et vulnérabilités.

Étapes
1. Copie le dossier `ex03` vers `ex06-alpine-scan`.
2. Modifie le Dockerfile :
   ```dockerfile
   FROM python:3.11-alpine
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   COPY app.py .
   USER 1000
   EXPOSE 5000
   CMD ["python", "app.py"]
   ```
3. Build :
   ```bash
   docker build -t flask-alpine:1.0 .
   ```
4. Compare tailles :
   ```bash
   docker images | grep flask
   ```
5. Scan (Docker Desktop ≥ 4.30 ou CLI) :
   ```bash
   docker scout cves flask-alpine:1.0
   ```
6. Note le nombre de CVE critiques vs l’image `python:3.11-slim`.

Critères de réussite  
✓ Image alpine < 60 Mo  
✓ Aucune CVE « critical » (ou justifiée)  
✓ Conteneur démarre toujours avec `docker run -p 5000:5000 flask-alpine:1.0`
