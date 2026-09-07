# Workshop 3 — Docker en EC2 y despliegue de una aplicación

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- EC2
- Docker
- Docker Compose
- Nginx
- Flask
- Puertos
- Publicación de aplicaciones

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Instalar Docker en una instancia EC2.
- Ejecutar contenedores dentro de EC2.
- Publicar una aplicación hacia Internet.
- Comprender la diferencia entre puerto del contenedor y puerto del host.
- Integrar EC2 con los conocimientos anteriores de Docker y Flask.
- Utilizar un Security Group para permitir tráfico HTTP.

---

# Arquitectura

```text
Internet
   |
   v
EC2
 |
 +-- Docker
      |
      +-- Nginx
      |
      +-- Flask
```

---

# Parte 1. Instalar Docker

Siga la documentación oficial de Docker para la distribución utilizada o el método recomendado por ella.

Verifique:

```bash
docker --version
sudo docker run hello-world
```

---

# Parte 2. Crear una API Flask

Cree:

```text
app/
├── app.py
├── requirements.txt
└── Dockerfile
```

`app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def home():
    return {"message": "Hola desde EC2"}

@app.get("/health")
def health():
    return {"status": "ok"}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt`:

```text
flask
```

`Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# Parte 3. Construir la imagen

```bash
docker build -t ec2-flask-api .
docker images
```

---

# Parte 4. Ejecutar el contenedor

```bash
docker run -d --name api -p 5000:5000 ec2-flask-api
docker ps
curl http://localhost:5000/health
```

---

# Parte 5. Publicar la aplicación

Configure el Security Group para permitir temporalmente el puerto requerido para el laboratorio.

Pruebe desde su computador:

```text
http://<IP_PUBLICA>:5000
```

---

# Parte 6. Docker Compose

Cree:

```yaml
services:
  api:
    build: .
    container_name: api
    ports:
      - "5000:5000"
```

Ejecute:

```bash
docker compose up -d --build
docker compose ps
```

---

# Parte 7. Nginx

Como extensión, agregue Nginx y haga que actúe como reverse proxy hacia Flask:

```text
Internet
   |
  :80
   |
 Nginx
   |
  :5000
   |
 Flask
```

El usuario final debería entrar solamente por:

```text
http://<IP_PUBLICA>
```

---

# Reto

Cambie la respuesta de Flask para incluir:

- Hostname del contenedor.
- Hostname de EC2.
- Hora actual.

Explique la diferencia entre el host EC2 y el contenedor.

---

# Entregable

- Dockerfile.
- `docker-compose.yml`.
- Configuración Nginx.
- Evidencia de la API accesible desde Internet.
- Explicación del flujo:

```text
Internet -> Security Group -> EC2 -> Docker -> Nginx -> Flask
```
