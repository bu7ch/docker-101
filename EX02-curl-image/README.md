 # EX02 – Créer une image « curl » ultra-légère  

Objectif  
Écrire un Dockerfile qui produit une image capable de télécharger une URL via curl.  

Étapes  
1. Dans un dossier `curl-image`, crée le Dockerfile :  
   ```dockerfile
   FROM alpine:3.19
   RUN apk add --no-cache curl
   ENTRYPOINT ["curl"]
   ```  
2. Build :  
   ```bash
   docker build -t my-curl:1.0 .
   ```  
3. Test :  
   ```bash
   docker run --rm my-curl:1.0 https://httpbin.org/ip
   ```  
4. Tag latest :  
   ```bash
   docker tag my-curl:1.0 my-curl:latest
   ```  

Critères de réussite  
✓ Image < 10 Mo (`docker images`)  
✓ Commande ci-dessus retourne une IP publique JSON
