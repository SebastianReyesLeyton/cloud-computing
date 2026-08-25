# Taller 5 - MongoDB con Docker y Docker Compose

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:

- MongoDB
- Docker
- Docker Compose
- NoSQL
- Documentos
- Colecciones
- Scripts de inicialización

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Ejecutar MongoDB con Docker Compose.
- Utilizar `mongosh`.
- Crear manualmente una base y colecciones.
- Insertar y consultar documentos.
- Comprender el modelo documental.
- Automatizar la creación de colecciones y documentos con JavaScript.
- Reinicializar MongoDB desde cero y verificar la estructura resultante.

---

# Parte 1. MongoDB manual

Crear:

```yaml
services:
  mongo:
    image: mongo:8
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    ports:
      - "27017:27017"
```

Levantar:

```bash
docker compose up -d
```

Entrar:

```bash
docker compose exec mongo mongosh -u admin -p admin --authenticationDatabase admin
```

Crear la base:

```javascript
use tienda
```

Crear colección:

```javascript
db.createCollection("products")
```

Insertar:

```javascript
db.products.insertMany([
  { name: "Laptop", price: 3500000, stock: 5 },
  { name: "Mouse", price: 80000, stock: 20 },
  { name: "Keyboard", price: 150000, stock: 12 }
])
```

Consultar:

```javascript
db.products.find()
```

---

# Parte 2. Explorar colecciones y documentos

```javascript
show dbs
show collections
db.products.countDocuments()
db.products.find({ stock: { $gt: 10 } })
```

Analizar cómo un documento puede incluir estructuras anidadas.

---

# Parte 3. Automatización mediante archivo JavaScript

Crear:

```text
mongo/init/01-init.js
```

Contenido:

```javascript
db = db.getSiblingDB('tienda');

db.products.insertMany([
  { name: 'Laptop', price: 3500000, stock: 5 },
  { name: 'Mouse', price: 80000, stock: 20 },
  { name: 'Keyboard', price: 150000, stock: 12 }
]);

db.customers.insertMany([
  { name: 'Ana', email: 'ana@example.com' },
  { name: 'Luis', email: 'luis@example.com' }
]);
```

Montar el directorio:

```yaml
services:
  mongo:
    image: mongo:8
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
      - ./mongo/init:/docker-entrypoint-initdb.d:ro

volumes:
  mongo_data:
```

---

# Parte 4. Verificar la carga automática

Recrear la instancia desde cero:

```bash
docker compose down -v
```

Luego:

```bash
docker compose up -d
```

Entrar y consultar:

```bash
docker compose exec mongo mongosh -u admin -p admin --authenticationDatabase admin
```

```javascript
use tienda
show collections
db.products.find()
db.customers.find()
```

---

# Parte 5. Reto de modelado

Crear mediante script las colecciones:

```text
products
customers
orders
```

Una orden puede almacenar:

```javascript
{
  customerId: 1,
  date: ISODate("2026-08-24T00:00:00Z"),
  items: [
    { product: "Laptop", quantity: 1 },
    { product: "Mouse", quantity: 2 }
  ]
}
```

---

# Parte 6. Comparación PostgreSQL vs MongoDB

Responder:

1. ¿Qué diferencia existe entre tabla y colección?
2. ¿Qué diferencia existe entre fila y documento?
3. ¿Qué ventaja ofrecen los documentos anidados?
4. ¿Qué riesgos tiene duplicar datos en documentos?
5. ¿Qué tipos de aplicaciones podrían beneficiarse de MongoDB?
6. ¿Qué modelo resulta más adecuado para el ejercicio universitario y por qué?

---

# Entregables

- `docker-compose.yml`.
- Carpeta `mongo/init/`.
- Script `.js`.
- Evidencia de creación manual.
- Evidencia de carga automática.
- Consultas realizadas.
- Comparación PostgreSQL vs MongoDB.
