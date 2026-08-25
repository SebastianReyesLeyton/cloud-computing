# Taller 4 - PostgreSQL con Docker y Docker Compose

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:

- PostgreSQL
- Docker
- Docker Compose
- SQL
- Esquemas
- Tablas
- Volúmenes
- Scripts de inicialización

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Crear PostgreSQL mediante Docker Compose.
- Administrar la base mediante `psql`.
- Construir manualmente una base de datos y su esquema.
- Crear tablas, relaciones y registros.
- Persistir información con volúmenes.
- Automatizar la creación mediante archivos `.sql`.
- Verificar la inicialización del contenido desde cero.

---

# Parte 1. Construcción manual

Crear:

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: universidad
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    ports:
      - "5432:5432"
```

Levantar:

```bash
docker compose up -d
```

Entrar:

```bash
docker compose exec postgres psql -U admin -d universidad
```

---

# Parte 2. Crear manualmente la estructura

Ejecutar:

```sql
CREATE SCHEMA academia;
```

Crear:

```sql
CREATE TABLE academia.students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL
);
```

Crear otra tabla:

```sql
CREATE TABLE academia.courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    credits INTEGER NOT NULL
);
```

Insertar datos y consultar:

```sql
SELECT * FROM academia.students;
SELECT * FROM academia.courses;
```

---

# Parte 3. Automatización

Crear:

```text
postgres/init/01-schema.sql
postgres/init/02-data.sql
```

`01-schema.sql`:

```sql
CREATE SCHEMA IF NOT EXISTS academia;

CREATE TABLE IF NOT EXISTS academia.students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL
);

CREATE TABLE IF NOT EXISTS academia.courses (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    credits INTEGER NOT NULL
);
```

`02-data.sql`:

```sql
INSERT INTO academia.students (name, email) VALUES
('Ana', 'ana@example.com'),
('Luis', 'luis@example.com'),
('Marta', 'marta@example.com');

INSERT INTO academia.courses (name, credits) VALUES
('Cloud Computing', 3),
('Internet of Things', 4);
```

---

# Parte 4. Montar los scripts

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: universidad
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/init:/docker-entrypoint-initdb.d:ro

volumes:
  postgres_data:
```

---

# Parte 5. Regla importante de inicialización

Los scripts de `/docker-entrypoint-initdb.d` son ejecutados por la imagen oficial durante la inicialización de un directorio de datos vacío.

Para probar nuevamente la carga automática desde cero:

```bash
docker compose down -v
```

Luego:

```bash
docker compose up -d
```

Verificar esquemas:

```bash
docker compose exec postgres psql -U admin -d universidad -c "\\dn"
```

Verificar tablas:

```bash
docker compose exec postgres psql -U admin -d universidad -c "\\dt academia.*"
```

Consultar datos:

```bash
docker compose exec postgres psql -U admin -d universidad -c "SELECT * FROM academia.students;"
```

---

# Parte 6. Reto

Automatizar:

- `teachers`.
- `courses`.
- `enrollments`.
- Llaves foráneas.
- Datos de prueba.

El esquema debe quedar completamente reconstruible eliminando el volumen y levantando nuevamente Compose.

---

# Entregables

- `docker-compose.yml`.
- Directorio `postgres/init/`.
- Scripts `.sql`.
- Evidencia manual.
- Evidencia automática.
- Evidencia de esquemas, tablas y datos.
