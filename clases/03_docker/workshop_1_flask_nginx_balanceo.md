# Taller 1 - API Flask y Balanceamiento de Carga con Nginx

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:

- Python y Flask
- Dockerfile
- Docker Compose
- Nginx
- Reverse Proxy
- Load Balancing
- Escalamiento horizontal

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Construir una API sencilla con Flask.
- Crear una imagen Docker para una aplicación Python.
- Ejecutar varias instancias de la misma API.
- Configurar Nginx como balanceador.
- Identificar qué instancia respondió a cada solicitud.
- Explicar la relación entre balanceamiento y escalamiento horizontal.

---

# Conceptos que se trabajarán

- API REST
- Contenedor
- Imagen
- Servicio de Docker Compose
- Instancia
- Upstream de Nginx
- Round Robin
- Escalamiento horizontal
- Punto de entrada único

---

# Parte 1. Construcción de la API

Crear `app.py` con un endpoint `GET /`.

La respuesta debe tener el siguiente formato:

```json
{
  "message": "Hola desde Flask",
  "instance": "api-1"
}
```

El valor de `instance` debe venir de una variable de entorno.

```python
import os
from flask import Flask, jsonify

app = Flask(__name__)
INSTANCE = os.getenv("INSTANCE", "api-local")

@app.get("/")
def home():
    return jsonify({
        "message": "Hola desde Flask",
        "instance": INSTANCE
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Crear `requirements.txt`:

```text
Flask
```

---

# Parte 2. Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

Construir y probar la imagen individualmente antes de crear el balanceamiento.

---

# Parte 3. Docker Compose con tres instancias

Crear tres servicios a partir de la misma imagen:

```yaml
services:
  api1:
    build: ./api
    environment:
      INSTANCE: api-1

  api2:
    build: ./api
    environment:
      INSTANCE: api-2

  api3:
    build: ./api
    environment:
      INSTANCE: api-3

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

---

# Parte 4. Configuración de Nginx

Crear `nginx/nginx.conf`:

```nginx
upstream flask_backend {
    server api1:5000;
    server api2:5000;
    server api3:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://flask_backend;
    }
}
```

Completar el servicio Nginx:

```yaml
nginx:
  image: nginx:alpine
  ports:
    - "8080:80"
  volumes:
    - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
  depends_on:
    - api1
    - api2
    - api3
```

---

# Parte 5. Evidenciar el balanceamiento

Levantar:

```bash
docker compose up --build
```

Ejecutar:

```bash
for i in {1..12}; do curl -s http://localhost:8080/; echo; done
```

Identificar qué instancias aparecen.

Detener una instancia:

```bash
docker compose stop api2
```

Volver a probar las solicitudes y documentar el comportamiento.

---

# Parte 6. Preguntas de análisis

1. ¿Por qué el cliente solamente conoce `localhost:8080`?
2. ¿Qué función cumple `upstream`?
3. ¿Qué estrategia utiliza Nginx por defecto en este escenario?
4. ¿Qué ocurre al detener `api2`?
5. ¿Qué componente podría convertirse en un punto único de falla?
6. ¿Por qué este diseño representa escalamiento horizontal?
7. ¿Qué tendría que cambiar para tener cinco instancias?

---

# Reto

Agregar un endpoint `/health` que responda:

```json
{
  "status": "ok",
  "instance": "api-1"
}
```

Luego investigar cómo podría usarse en una estrategia de health check.

---

# Entregables

- `app.py`
- `requirements.txt`
- `Dockerfile`
- `docker-compose.yml`
- `nginx.conf`
- Evidencias del balanceamiento.
- Respuestas a las preguntas.
