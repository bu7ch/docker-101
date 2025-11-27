 
# EX03 – Containeriser une micro-api Flask  

Objectif  
Packager une appli Python (Flask) qui répond « Hello <name> » sur `/hello/<name>`  

Fichiers fournis  
```
app.py
requirements.txt
```  

app.py  
```python
from flask import Flask
app = Flask(__name__)

@app.route("/hello/<name>")
def hello(name):
    return f"Hello {name} from Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5001)

```  

requirements.txt  
```
flask==3.0.2
```  

Étapes  
1. Écris le Dockerfile :  
   ```dockerfile
   FROM python:3.11-slim
   WORKDIR /app
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   COPY app.py .
   EXPOSE 5000
   CMD ["python", "app.py"]
   ```  
2. Build :  
   ```bash
   docker build -t flask-demo:1.0 .
   ```  
3. Run :  
   ```bash
   docker run -d -p 5000:5000 --name flask flask-demo:1.0
   ```  
4. Test :  
   ```bash
   curl http://localhost:5000/hello/Alice
   ```  

Critères de réussite  
✓ Réponse « Hello Alice from Docker ! »  
✓ `docker logs flask` sans erreur
