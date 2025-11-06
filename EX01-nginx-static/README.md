
# EX01 – Servir une page statique avec nginx

Objectif  
Lancer le serveur web nginx dans un conteneur et remplacer la page d’accueil par une page perso sans reconstruire d’image.

Étapes
1. Crée un répertoire `site` contenant `index.html` :
   ```html
   <h1>Bonjour depuis Docker</h1>
   ```
2. Lance le conteneur :
   ```bash
   docker run -d --name mon-nginx -p 8080:80 -v "$(pwd)/site:/usr/share/nginx/html" nginx
   ```
3. Ouvre http://localhost:8080 dans ton navigateur.
4. Observe les logs : `docker logs mon-nginx`
5. Modifie `index.html` sur l’hôte → rafraîchis le navigateur (pas besoin de restart).

Critères de réussite  
✓ Page « Bonjour depuis Docker » visible  
✓ `docker ps` montre le conteneur UP  
✓ `curl localhost:8080` retourne ton HTML

Nettoyage
```bash
docker stop mon-nginx && docker rm mon-nginx
```
