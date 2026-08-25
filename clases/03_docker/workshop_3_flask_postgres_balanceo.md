# Taller 3 - Tres APIs Flask Balanceadas con PostgreSQL

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:

- Flask
- Docker Compose
- Nginx
- Load Balancing
- PostgreSQL
- Python y SQL
- Stateless API

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Ejecutar tres instancias de la misma API.
- Balancear las solicitudes con Nginx.
- Consultar PostgreSQL desde Python.
- Compartir datos entre múltiples instancias.
- Comprender la importancia de separar estado y lógica de negocio.

---

# Arquitectura

```text
                    Cliente
                       |
                       v
                    Nginx
                 /    |    \
                v     v     v
              API1  API2  API3
                \     |     /
                 \    |    /
                 PostgreSQL
```

---

# Parte 1. PostgreSQL

Crear un servicio PostgreSQL:

```yaml
db:
  image: postgres:16
  environment:
    POSTGRES_DB: shop
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
```

Crear una tabla:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10,2) NOT NULL
);
```

Insertar por lo menos cinco productos.

---

# Parte 2. Flask + PostgreSQL

Instalar:

```text
Flask
psycopg[binary]
```

Crear:

```text
GET /products
```

La API debe consultar PostgreSQL y retornar los productos.

Además debe identificar la instancia que procesó la solicitud:

```json
{
  "served_by": "api-2",
  "products": [
    {"id": 1, "name": "Laptop", "price": 3500000}
  ]
}
```

---

# Parte 3. Variables de entorno

Las instancias deben recibir:

```text
INSTANCE
DB_HOST
DB_NAME
DB_USER
DB_PASSWORD
```

La aplicación no debe tener las credenciales escritas directamente en el código fuente.

---

# Parte 4. Tres APIs

```yaml
api1:
  build: ./api
  environment:
    INSTANCE: api-1
    DB_HOST: db
    DB_NAME: shop
    DB_USER: postgres
    DB_PASSWORD: postgres

api2:
  build: ./api
  environment:
    INSTANCE: api-2
    DB_HOST: db
    DB_NAME: shop
    DB_USER: postgres
    DB_PASSWORD: postgres

api3:
  build: ./api
  environment:
    INSTANCE: api-3
    DB_HOST: db
    DB_NAME: shop
    DB_USER: postgres
    DB_PASSWORD: postgres
```

Configurar Nginx para distribuir las solicitudes entre `api1`, `api2` y `api3`.

---

# Parte 5. Prueba de integración

```bash
docker compose up --build
```

Luego:

```bash
for i in {1..15}; do curl -s http://localhost:8080/products; echo; done
```

Comprobar que:

- Cambia `served_by`.
- Los datos de productos son los mismos.
- Las tres APIs consultan el mismo estado persistente.

---

# Parte 6. Prueba de tolerancia ante fallo

Detener una API:

```bash
docker compose stop api2
```

Repetir las solicitudes.

Después detener dos APIs y observar qué ocurre.

---

# Parte 7. Reto CRUD

Agregar:

```text
GET /products/<id>
POST /products
PUT /products/<id>
DELETE /products/<id>
```

El alumno debe demostrar que un registro creado desde una instancia puede ser consultado desde otra.

---

# Análisis

1. ¿Dónde se encuentra el estado del negocio?
2. ¿Por qué las APIs pueden ser stateless?
3. ¿Qué ocurre si una API reinicia?
4. ¿Qué ocurre si PostgreSQL reinicia?
5. ¿Dónde existen puntos únicos de falla?
6. ¿El balanceamiento de API soluciona el escalamiento de la base de datos?
7. ¿Qué mecanismos podrían añadirse en una arquitectura de producción?

---

# Entregables

- API Flask.
- Dockerfile.
- `docker-compose.yml`.
- `nginx.conf`.
- Script SQL.
- Evidencia del balanceamiento.
- Evidencia de consulta a PostgreSQL.
- Respuestas de análisis.
