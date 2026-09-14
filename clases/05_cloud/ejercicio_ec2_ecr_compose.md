# Proyecto — Sistema de seguimiento de equipos en AWS

**Curso:** Computación en la Nube e Internet de las Cosas

**Tecnologías principales:** Amazon EC2, Amazon ECR, Amazon S3, PostgreSQL, Python + Flask, Nginx, Docker y Docker Compose.

---

# 1. Objetivo del ejercicio

Construir y desplegar una aplicación web sencilla para el **seguimiento y administración de equipos tecnológicos**.

La solución debe permitir registrar equipos, consultar equipos, registrar empleados, asignar equipos, registrar mantenimientos, consultar el historial de un equipo y asociar documentos o fotografías a cada equipo.

La arquitectura debe utilizar:

```text
                    INTERNET
                       |
                       v
                  Amazon EC2
                       |
                  Docker Compose
                       |
            +----------+----------+
            |                     |
            v                     v
          Nginx              Flask API
                                  |
                        +---------+---------+
                        |                   |
                        v                   v
                   PostgreSQL             S3
                   relacional          documentos
```

La imagen Docker de la API **no se debe construir dentro de EC2**. El estudiante deberá:

```text
Código fuente
     |
     v
Docker build
     |
     v
Amazon ECR
     |
     v
EC2
     |
     v
Docker Compose
     |
     v
Contenedor Flask
```

La intención es practicar un flujo cercano a **build once, deploy the same image**.

---

# 2. Arquitectura final esperada

```text
                          Usuario
                             |
                             v
                        Internet
                             |
                        TCP 80/443
                             |
                             v
                    +----------------+
                    |      EC2       |
                    |                |
                    | Docker Compose |
                    |                |
                    |   +--------+   |
                    |   | Nginx  |   |
                    |   +---+----+   |
                    |       |        |
                    |       v        |
                    |   +--------+   |
                    |   | Flask  |   |
                    |   | API    |   |
                    |   +---+----+   |
                    +-------|--------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
          PostgreSQL                Amazon S3
          relacional             documentos/archivos
```

La imagen de Flask será obtenida desde **Amazon ECR**.

```text
ECR
 |
 | docker pull
 v
EC2
 |
 v
Docker Compose
```

---

# 3. Requerimientos funcionales

## 3.1 Empleados

```http
GET /empleados
POST /empleados
GET /empleados/{id}
```

Ejemplo:

```json
{
  "id": 1,
  "nombre": "Ana Torres",
  "correo": "ana@empresa.com",
  "departamento": "Tecnología"
}
```

## 3.2 Equipos

```http
GET /equipos
POST /equipos
GET /equipos/{id}
PUT /equipos/{id}
```

Ejemplo:

```json
{
  "id": 10,
  "codigo_interno": "LAP-0010",
  "tipo": "Laptop",
  "marca": "Lenovo",
  "modelo": "ThinkPad E14",
  "numero_serie": "ABC123456",
  "estado": "DISPONIBLE"
}
```

## 3.3 Asignación

```http
POST /equipos/{id}/asignar
```

Body:

```json
{
  "empleado_id": 1
}
```

La aplicación debe verificar que el equipo y el empleado existan y que el equipo esté disponible.

## 3.4 Mantenimiento

```http
POST /equipos/{id}/mantenimientos
```

Ejemplo:

```json
{
  "tipo": "Preventivo",
  "descripcion": "Limpieza y actualización",
  "costo": 120000
}
```

## 3.5 Historial

```http
GET /equipos/{id}/historial
```

Ejemplo:

```json
{
  "equipo": "LAP-0010",
  "historial": [
    {
      "fecha": "2026-09-01",
      "evento": "Asignación",
      "detalle": "Ana Torres"
    },
    {
      "fecha": "2026-09-10",
      "evento": "Mantenimiento",
      "detalle": "Preventivo"
    }
  ]
}
```

---

# 4. Modelo de datos

Utilizaremos PostgreSQL para los datos estructurados.

## employees

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    department VARCHAR(100)
);
```

## equipment

```sql
CREATE TABLE equipment (
    id SERIAL PRIMARY KEY,
    internal_code VARCHAR(50) UNIQUE NOT NULL,
    type VARCHAR(50) NOT NULL,
    brand VARCHAR(80),
    model VARCHAR(80),
    serial_number VARCHAR(100) UNIQUE,
    status VARCHAR(30) NOT NULL,
    acquisition_date DATE,
    employee_id INTEGER REFERENCES employees(id)
);
```

## maintenance

```sql
CREATE TABLE maintenance (
    id SERIAL PRIMARY KEY,
    equipment_id INTEGER NOT NULL REFERENCES equipment(id),
    maintenance_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    type VARCHAR(50),
    description TEXT,
    cost NUMERIC(12,2)
);
```

## documents

Esta tabla almacena metadatos del objeto almacenado en S3.

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    equipment_id INTEGER NOT NULL REFERENCES equipment(id),
    file_name VARCHAR(255) NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Relaciones:

```text
Empleado
   |
   +----< Equipo
             |
             +----< Mantenimiento
             |
             +----< Documento
```

---

# 5. ¿Qué va en PostgreSQL y qué va en S3?

## PostgreSQL

Guardar:

- Empleados.
- Equipos.
- Estados.
- Asignaciones.
- Mantenimientos.
- Metadatos de archivos.

## S3

Guardar:

- Fotografías.
- PDFs.
- Actas de entrega.
- Facturas.
- Evidencias de mantenimiento.

Ejemplo de clave S3:

```text
equipos/LAP-0010/acta-entrega.pdf
```

En PostgreSQL:

```text
file_name = acta-entrega.pdf
s3_key = equipos/LAP-0010/acta-entrega.pdf
```

---

# 6. Parte A — Preparar el proyecto Flask

Crear:

```text
cloud-equipment/
├── api/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── nginx/
│   └── nginx.conf
├── db/
│   └── init.sql
├── docker-compose.yml
└── .env.example
```

---

# 7. Crear la API Flask

`api/app.py`:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/health")
def health():
    return jsonify({
        "status": "ok",
        "service": "equipment-api"
    })

@app.get("/equipos")
def equipos():
    return jsonify([])

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

# 8. Dependencias

`api/requirements.txt`:

```text
Flask
psycopg2-binary
boto3
gunicorn
```

---

# 9. Dockerfile de la API

`api/Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

---

# 10. Parte B — Construir y probar localmente

Desde `api`:

```bash
docker build -t equipment-api:1.0 .
```

Ejecutar:

```bash
docker run --rm -p 5000:5000 equipment-api:1.0
```

Probar:

```bash
curl http://localhost:5000/health
```

Respuesta esperada:

```json
{
  "service": "equipment-api",
  "status": "ok"
}
```

No avanzar a ECR hasta que la imagen funcione localmente.

---

# 11. Parte C — Crear el repositorio en Amazon ECR

En AWS Console:

```text
Amazon ECR
   |
   +-- Repositories
        |
        +-- Create repository
```

Nombre:

```text
equipment-api
```

Para el laboratorio se recomienda mantenerlo privado.

---

# 12. Obtener región y Account ID

Configurar AWS CLI:

```bash
aws configure
```

Consultar identidad:

```bash
aws sts get-caller-identity
```

Supongamos:

```text
AWS_ACCOUNT_ID=123456789012
AWS_REGION=us-east-1
```

La URI será:

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api
```

---

# 13. Autenticar Docker contra ECR

AWS recomienda utilizar `get-login-password` y pasar el token a `docker login`.

```bash
aws ecr get-login-password \
  --region us-east-1 \
| docker login \
  --username AWS \
  --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

El token de autenticación tiene una validez de 12 horas. citeturn159889search1turn159889search3

Si aparece:

```text
Login Succeeded
```

Docker ya está autenticado contra ECR.

---

# 14. Etiquetar la imagen

```bash
docker tag \
  equipment-api:1.0 \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:1.0
```

Verificar:

```bash
docker images
```

---

# 15. Publicar la imagen en ECR

```bash
docker push \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:1.0
```

El flujo documentado por AWS es autenticarse, etiquetar la imagen con la URI del repositorio y ejecutar `docker push`. citeturn159889search3turn159889search6

---

# 16. Verificar ECR

En la consola:

```text
ECR
 |
 +-- equipment-api
      |
      +-- 1.0
```

O por CLI:

```bash
aws ecr describe-images \
  --repository-name equipment-api \
  --region us-east-1
```

---

# 17. Parte D — Crear el bucket S3

En AWS Console:

```text
S3
 |
 +-- Create bucket
```

Nombre sugerido:

```text
equipment-documents-<identificador-unico>
```

Mantener el bucket privado.

Las claves lógicas de los objetos pueden ser:

```text
equipos/LAP-0010/foto.jpg
equipos/LAP-0010/acta.pdf
```

---

# 18. Parte E — Crear EC2

Crear una instancia con una AMI Linux adecuada para el laboratorio.

Configurar:

- Instance Type pequeño apropiado para laboratorio.
- Public IP habilitada si se usará acceso público directo.
- Key Pair o EC2 Instance Connect.

Security Group inicial:

```text
TCP 22  -> solo IP administrativa
TCP 80  -> 0.0.0.0/0
```

No abrir:

```text
5432 -> Internet
```

si PostgreSQL se mantiene dentro de la EC2.

AWS recomienda restringir SSH a la IP o red administrativa cuando se utiliza SSH directo. citeturn159889search2

---

# 19. Conectarse a EC2

Por SSH:

```bash
ssh -i mi-clave.pem usuario@<IP_PUBLICA>
```

Verificar:

```bash
uname -a
```

---

# 20. Instalar Docker y Compose

Instalar Docker de acuerdo con la distribución Linux utilizada y su documentación oficial.

Verificar:

```bash
docker --version
```

```bash
docker compose version
```

Configurar el usuario para poder utilizar Docker sin `sudo` cuando corresponda y volver a iniciar sesión.

---

# 21. AWS CLI e IAM Role

Verificar:

```bash
aws --version
```

La EC2 necesitará permisos para:

1. autenticarse con ECR;
2. descargar la imagen;
3. acceder a S3.

Se recomienda asociar un **IAM Role** a la EC2 en lugar de copiar access keys dentro del servidor o de los contenedores.

Conceptualmente:

```text
EC2
 |
 IAM Role
 |
 +--> ECR Pull
 |
 +--> S3 Access
```

El Role debe seguir el principio de mínimo privilegio.

---

# 22. Parte F — Descargar la imagen desde ECR en EC2

En EC2:

```bash
aws ecr get-login-password \
  --region us-east-1 \
| docker login \
  --username AWS \
  --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

Después:

```bash
docker pull \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:1.0
```

Verificar:

```bash
docker images
```

---

# 23. Parte G — Preparar Docker Compose en EC2

```bash
mkdir -p ~/equipment-platform/{nginx,db}
cd ~/equipment-platform
```

Crear `docker-compose.yml`:

```yaml
services:

  api:
    image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:1.0
    container_name: equipment-api
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: equipment
      DB_USER: equipment
      DB_PASSWORD: change-me
      S3_BUCKET: equipment-documents-example
      AWS_REGION: us-east-1
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    container_name: equipment-db
    environment:
      POSTGRES_DB: equipment
      POSTGRES_USER: equipment
      POSTGRES_PASSWORD: change-me
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U equipment -d equipment"]
      interval: 5s
      timeout: 5s
      retries: 10

  nginx:
    image: nginx:alpine
    container_name: equipment-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - api

volumes:
  postgres_data:
```

**Nota didáctica:** la API utiliza `image:` y no `build:`. Esto obliga a consumir la imagen publicada en ECR.

---

# 24. Nginx

`nginx/nginx.conf`:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://api:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Dentro de la red de Compose, Nginx puede resolver `api` por el nombre del servicio.

---

# 25. PostgreSQL e inicialización

`db/init.sql`:

```sql
CREATE TABLE IF NOT EXISTS employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    department VARCHAR(100)
);

CREATE TABLE IF NOT EXISTS equipment (
    id SERIAL PRIMARY KEY,
    internal_code VARCHAR(50) UNIQUE NOT NULL,
    type VARCHAR(50) NOT NULL,
    brand VARCHAR(80),
    model VARCHAR(80),
    serial_number VARCHAR(100) UNIQUE,
    status VARCHAR(30) NOT NULL,
    acquisition_date DATE,
    employee_id INTEGER REFERENCES employees(id)
);

CREATE TABLE IF NOT EXISTS maintenance (
    id SERIAL PRIMARY KEY,
    equipment_id INTEGER NOT NULL REFERENCES equipment(id),
    maintenance_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    type VARCHAR(50),
    description TEXT,
    cost NUMERIC(12,2)
);

CREATE TABLE IF NOT EXISTS documents (
    id SERIAL PRIMARY KEY,
    equipment_id INTEGER NOT NULL REFERENCES equipment(id),
    file_name VARCHAR(255) NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO employees(name, email, department)
VALUES
('Ana Torres', 'ana@empresa.com', 'Tecnología'),
('Carlos Gómez', 'carlos@empresa.com', 'Finanzas')
ON CONFLICT (email) DO NOTHING;
```

**Importante:** el script de inicialización de la imagen oficial de PostgreSQL se ejecuta al inicializar un directorio de datos vacío. Si ya existe un volumen con una base creada, modificar `init.sql` no reconstruye automáticamente el esquema.

---

# 26. Variables de entorno

Para el laboratorio se puede utilizar un `.env` fuera del código fuente:

```env
DB_NAME=equipment
DB_USER=equipment
DB_PASSWORD=una-clave-segura
S3_BUCKET=equipment-documents-example
AWS_REGION=us-east-1
```

No subir credenciales reales a Git.

Una práctica posterior sería utilizar AWS Secrets Manager o Parameter Store.

---

# 27. Levantar todo con Docker Compose

```bash
docker compose up -d --pull always
```

Verificar:

```bash
docker compose ps
```

Logs generales:

```bash
docker compose logs
```

Solo API:

```bash
docker compose logs api
```

Solo Nginx:

```bash
docker compose logs nginx
```

---

# 28. Comprobar PostgreSQL

```bash
docker exec -it equipment-db psql -U equipment -d equipment
```

Dentro:

```sql
\dt
```

```sql
SELECT * FROM employees;
```

Salir:

```sql
\q
```

---

# 29. Comprobar la API

Desde EC2:

```bash
curl http://localhost/health
```

Desde el computador:

```text
http://<IP_PUBLICA_EC2>/health
```

Debe responder:

```json
{
  "service": "equipment-api",
  "status": "ok"
}
```

---

# 30. Conectar Flask con PostgreSQL

El servicio Flask debe utilizar:

```text
DB_HOST=db
```

No:

```text
DB_HOST=localhost
```

Dentro de un contenedor, `localhost` significa el propio contenedor. `db` significa el servicio PostgreSQL del Compose.

---

# 31. Conectar Flask con S3

Con `boto3`:

```python
import boto3
import os

s3 = boto3.client(
    "s3",
    region_name=os.getenv("AWS_REGION")
)
```

Subir un objeto:

```python
s3.upload_fileobj(
    file,
    os.getenv("S3_BUCKET"),
    s3_key
)
```

La aplicación debe utilizar la identidad IAM disponible para el entorno AWS en vez de hardcodear access keys.

---

# 32. Endpoint para documentos

Ejemplo conceptual:

```python
@app.post("/equipos/<int:equipment_id>/documentos")
def upload_document(equipment_id):
    file = request.files["file"]

    key = f"equipos/{equipment_id}/{file.filename}"

    s3.upload_fileobj(
        file,
        os.environ["S3_BUCKET"],
        key
    )

    # Guardar metadata en PostgreSQL

    return {
        "equipment_id": equipment_id,
        "s3_key": key
    }, 201
```

---

# 33. Flujo de una carga de archivo

```text
Usuario
   |
   v
Nginx
   |
   v
Flask
   |
   +--------------+
   |              |
   v              v
S3           PostgreSQL
archivo       metadata
```

---

# 34. Validación completa

## ECR

Debe existir:

```text
equipment-api:1.0
```

## EC2

```bash
docker compose ps
```

Debe mostrar:

```text
equipment-api
equipment-db
equipment-nginx
```

## API

```text
GET /health
```

## PostgreSQL

```sql
\dt
SELECT * FROM employees;
```

## S3

Debe existir al menos un objeto de prueba.

---

# 35. Reto 1 — Nueva versión de la API

Cambiar `/health` para devolver:

```json
{
  "status": "ok",
  "version": "2.0"
}
```

Construir:

```bash
docker build -t equipment-api:2.0 .
```

Etiquetar:

```bash
docker tag \
  equipment-api:2.0 \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:2.0
```

Publicar:

```bash
docker push \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:2.0
```

En EC2:

```bash
docker pull \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/equipment-api:2.0
```

Modificar Compose a `2.0` y ejecutar:

```bash
docker compose up -d
```

---

# 36. Reto 2 — No reconstruir en EC2

Durante la evaluación, verificar que:

- El código fuente de la API no sea necesario dentro de EC2 para desplegar la imagen.
- La API utilice `image:` y no `build:` en el Compose de despliegue.
- La imagen provenga de ECR.

---

# 37. Reto 3 — Health Check

Agregar un healthcheck a la API:

```yaml
healthcheck:
  test:
    ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"]
  interval: 10s
  timeout: 5s
  retries: 5
```

Responder:

> ¿Qué diferencia existe entre que el contenedor esté `running` y que la aplicación esté realmente saludable?

---

# 38. Reto 4 — Persistencia

Eliminar y volver a crear el contenedor PostgreSQL.

Verificar que los datos permanezcan debido al volumen:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Responder:

> ¿Qué persiste cuando el contenedor es eliminado?

---

# 39. Reto 5 — Escalabilidad conceptual

Evolucionar mentalmente la solución hacia:

```text
                 Load Balancer
                      |
          +-----------+-----------+
          |           |           |
        EC2-1       EC2-2       EC2-3
          |           |           |
        Flask       Flask       Flask
           \           |         /
            +----------+--------+
                       |
                   PostgreSQL
                       |
                       S3
```

Responder:

- ¿Qué componente puede escalar horizontalmente?
- ¿Qué componentes deben ser compartidos?
- ¿Por qué los archivos no deberían permanecer en el disco local de cada EC2?

---

# 40. Seguridad

La propuesta mínima debe seguir:

### Público

```text
80/443
```

### Administrativo

```text
22
```

Restringido a la red administrativa.

### No público

```text
5432
```

### ECR

Acceso autenticado.

### S3

Bucket privado.

### Secretos

No deben quedar hardcodeados en el código.

---

# 41. Errores frecuentes

## Docker no puede hacer pull desde ECR

Verificar:

```bash
aws sts get-caller-identity
```

Luego autenticar nuevamente en el registry.

También revisar el IAM Role de la EC2.

## La API no conecta a PostgreSQL

No usar `localhost`; usar:

```text
db
```

## No se puede acceder desde Internet

Revisar:

```text
Security Group -> TCP 80
```

Y:

```bash
docker compose ps
docker compose logs nginx
```

## El contenedor Flask termina

Revisar:

```bash
docker compose logs api
```

## ECR tiene la imagen pero EC2 no puede descargarla

Comprobar:

```bash
aws sts get-caller-identity
```

Y los permisos IAM sobre ECR.

---

# 42. Entregables

## 1. Diagrama

Debe mostrar:

- Usuario.
- EC2.
- Nginx.
- Flask.
- PostgreSQL.
- S3.
- ECR.

## 2. Código

- Flask.
- Dockerfile.
- Nginx.
- Docker Compose.
- SQL.

## 3. ECR

Repositorio:

```text
equipment-api
```

con al menos una versión.

## 4. Evidencias

Capturas de:

- EC2.
- ECR.
- S3.
- Docker Compose.
- API.
- PostgreSQL.

## 5. Explicación arquitectónica

El estudiante debe explicar:

> ¿Qué ocurre desde que hago `docker push` hasta que el usuario obtiene una respuesta de Flask?

---

# 43. Evaluación sugerida

| Criterio | Porcentaje |
|---|---:|
| Creación/configuración de EC2 | 15% |
| Construcción correcta de la API | 15% |
| Publicación de imagen en ECR | 20% |
| Despliegue con Docker Compose | 20% |
| Integración PostgreSQL | 10% |
| Integración S3 | 10% |
| Seguridad y explicación arquitectónica | 10% |
| **Total** | **100%** |

---

# 44. Preguntas de defensa

1. ¿Por qué utilizar ECR?
2. ¿Por qué no construir la imagen directamente en EC2?
3. ¿Qué diferencia existe entre ECR y S3?
4. ¿Por qué PostgreSQL y S3 tienen responsabilidades diferentes?
5. ¿Qué información debería estar en PostgreSQL?
6. ¿Qué información debería estar en S3?
7. ¿Por qué Flask utiliza `db` como hostname?
8. ¿Qué pasaría si eliminamos la EC2?
9. ¿Qué componente deberíamos cambiar para soportar varias instancias de Flask?
10. ¿Cómo evolucionaría esta arquitectura hacia alta disponibilidad?

---

# 45. Arquitectura final

```text
                         +----------------+
                         |     Usuario    |
                         +-------+--------+
                                 |
                                 v
                              Internet
                                 |
                                 v
                    +------------------------+
                    |       Amazon EC2       |
                    |                        |
                    |    Docker Compose      |
                    |                        |
                    |  +----------------+    |
                    |  |     Nginx      |    |
                    |  +-------+--------+    |
                    |          |             |
                    |          v             |
                    |  +----------------+    |
                    |  |   Flask API    |    |
                    |  +-------+--------+    |
                    +----------|-------------+
                               |
                     +---------+---------+
                     |                   |
                     v                   v
               PostgreSQL              S3
               relacional          documentos

                       ^
                       |
                 +-----+-----+
                 |    ECR    |
                 | Flask img |
                 +-----------+
```

---

# 46. Reflexión final

La finalidad del ejercicio no es solamente poner una API en AWS. El objetivo es que el estudiante comprenda un flujo completo de infraestructura:

```text
Desarrollo
    |
    v
Contenerización
    |
    v
Registro de imágenes
    |
    v
Despliegue
    |
    v
Red
    |
    v
Persistencia
    |
    +--> Base de datos
    |
    +--> Object Storage
    |
    v
Aplicación accesible por Internet
```

De esta manera se integran los conceptos trabajados durante el curso:

- Docker.
- Docker Compose.
- Flask.
- Nginx.
- PostgreSQL.
- S3.
- EC2.
- ECR.
- Networking.
- Seguridad.
- Persistencia.
- Arquitectura de software.
- Escalabilidad.

---

# Referencias oficiales

- Amazon EC2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html
- Amazon ECR: https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html
- AWS CLI `get-login-password`: https://docs.aws.amazon.com/cli/latest/reference/ecr/get-login-password.html
- Amazon S3: https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html
- Amazon RDS for PostgreSQL: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html
