
# EX04 – Partager un fichier entre hôte et conteneur

Objectif  
Faire lire un fichier `data.txt` situé sur l’hôte par un conteneur Alpine, sans copier l’image.

Étapes
1. Crée `data.txt` :
   ```
   Ligne 1
   Ligne 2
   ```
2. Lance :
   ```bash
   docker run -v "$(pwd)/data.txt:/tmp/data.txt:ro" alpine cat /tmp/data.txt
   ```
3. Modifie `data.txt` sur l’hôte, relance la commande → changement immédiat.

Bonus  
Utilise un volume nommé :
```bash
docker volume create shared
docker run -v shared:/data alpine sh -c 'echo toto > /data/f.txt'
docker run -v shared:/data alpine cat /data/f.txt
```

Critères de réussite  
✓ Affichage des 2 lignes  
✓ Aucune erreur de permission
