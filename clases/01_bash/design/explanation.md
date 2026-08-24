# Diseño de Arquitecturas de Software a Alto Nivel

## Objetivos de aprendizaje

Al finalizar esta guía el estudiante será capaz de:

- Comprender qué es una arquitectura de software.
- Diseñar una arquitectura de alto nivel.
- Identificar los principales componentes de un sistema.
- Diferenciar los principales tipos de APIs.
- Comprender cuándo utilizar REST y GraphQL.
- Diferenciar persistencia y caché.
- Seleccionar la mejor estrategia de almacenamiento según el problema.

---

# ¿Qué es una Arquitectura de Software?

La arquitectura de software es la **estructura de alto nivel** de un sistema.

Describe:

- Los componentes principales.
- Cómo se comunican.
- Qué responsabilidad tiene cada componente.
- Cómo fluye la información.
- Qué tecnologías podrían utilizarse.

No describe el código fuente.

En cambio, responde preguntas como:

- ¿Cuántos servicios tendrá el sistema?
- ¿Dónde estará la base de datos?
- ¿Cómo llegan las peticiones?
- ¿Cómo se comunican los componentes?
- ¿Qué sucede cuando un usuario realiza una acción?

Puede compararse con el plano arquitectónico de una casa.

Antes de construir una casa se diseñan:

- Habitaciones.
- Cocina.
- Baños.
- Pasillos.
- Instalaciones eléctricas.

Lo mismo ocurre con un sistema software.

---

# ¿Qué es una Arquitectura de Alto Nivel?

Una arquitectura de alto nivel representa únicamente los componentes principales.

No interesa conocer:

- Clases
- Métodos
- Funciones
- Variables

Solo interesa observar cómo está organizado el sistema.

Ejemplo:

```text
           Usuario

              │

              ▼

       Aplicación Web

              │

              ▼

          API REST

              │

      ┌───────┴────────┐
      │                │

Servicio Usuarios   Servicio Cursos

      │                │

      └───────┬────────┘

              ▼

        Base de Datos
```

Este diagrama permite entender rápidamente cómo funciona el sistema.

---

# Pasos para diseñar una arquitectura

## Paso 1. Comprender el problema

Antes de diseñar cualquier sistema es necesario responder preguntas como:

- ¿Quién utilizará el sistema?
- ¿Qué necesita hacer?
- ¿Qué información se almacenará?
- ¿Qué volumen de usuarios tendrá?
- ¿Debe funcionar en tiempo real?

Nunca se debe comenzar diseñando bases de datos o APIs sin comprender el problema.

---

## Paso 2. Identificar los actores

Los actores son quienes interactúan con el sistema.

Ejemplo:

- Cliente
- Administrador
- Profesor
- Estudiante
- Sensor IoT
- Sistema externo

---

## Paso 3. Identificar funcionalidades

Preguntarse:

¿Qué debe hacer el sistema?

Ejemplo:

- Registrar usuarios.
- Consultar productos.
- Realizar pagos.
- Enviar notificaciones.
- Registrar asistencia.

Cada funcionalidad suele convertirse posteriormente en un componente.

---

## Paso 4. Agrupar funcionalidades

En lugar de crear un único sistema enorme, las funcionalidades pueden agruparse.

Ejemplo

```
Usuarios

Cursos

Pagos

Reportes

Notificaciones
```

Cada grupo podría convertirse en un servicio independiente.

---

## Paso 5. Identificar componentes

Una arquitectura moderna suele contener componentes como:

- Cliente Web
- Aplicación móvil
- API
- Base de datos
- Caché
- Broker de mensajes
- Microservicios
- Balanceador
- Servicios externos

No todos los sistemas necesitan todos estos componentes.

---

## Paso 6. Diseñar el flujo

Ejemplo.

```
Cliente

↓

API

↓

Servicio

↓

Base de datos

↓

Respuesta
```

Este flujo representa el recorrido de una petición.

---

# Tipos de APIs

Una API (**Application Programming Interface**) es un conjunto de reglas que permite que dos aplicaciones se comuniquen entre sí.

Ejemplo.

Una aplicación móvil solicita al servidor la información de un estudiante.

```
Aplicación

↓

API

↓

Servidor

↓

Base de datos
```

La API actúa como intermediario.

---

# Tipos principales de APIs

Existen varios estilos de APIs.

Los más utilizados son:

- REST
- GraphQL
- gRPC
- WebSocket
- MQTT

Cada uno resuelve problemas distintos.

---

# REST

REST (**Representational State Transfer**) es el estilo de API más utilizado actualmente.

Se basa en recursos.

Ejemplo:

```
/students

/courses

/teachers

/payments
```

Cada recurso representa una entidad.

Las operaciones se realizan mediante los métodos HTTP.

| Método | Acción |
|----------|---------|
| GET | Consultar |
| POST | Crear |
| PUT | Actualizar |
| DELETE | Eliminar |

Ejemplo.

```
GET /students
```

Obtiene todos los estudiantes.

```
GET /students/15
```

Obtiene únicamente el estudiante 15.

```
POST /students
```

Crea un nuevo estudiante.

---

## Ventajas de REST

- Fácil de aprender.
- Muy utilizado.
- Compatible con cualquier lenguaje.
- Excelente para operaciones CRUD.
- Amplio soporte de herramientas.

---

## Desventajas

A veces devuelve más información de la necesaria.

Ejemplo.

Un estudiante solo necesita su nombre.

REST devuelve:

- Nombre
- Dirección
- Teléfono
- Fecha nacimiento
- Correo
- Programa
- Estado

Aunque no toda esa información sea utilizada.

---

# GraphQL

GraphQL fue desarrollado por Facebook.

Su principal ventaja es que el cliente decide exactamente qué información desea recibir.

Ejemplo.

```graphql
query {

 student(id:15){

    name

    email

 }

}
```

La respuesta será únicamente:

```json
{
 "name":"Juan",
 "email":"juan@correo.com"
}
```

No se envía información innecesaria.

---

## Ventajas de GraphQL

- Menor cantidad de datos.
- Una sola consulta puede obtener información relacionada.
- Ideal para aplicaciones móviles.
- Reduce múltiples llamadas al servidor.

---

## Desventajas

- Mayor complejidad.
- Requiere mayor conocimiento.
- Puede ser más difícil de asegurar.

---

# ¿Cuándo usar REST?

REST es recomendable cuando:

- Se construyen APIs sencillas.
- Existen operaciones CRUD.
- Se desea simplicidad.
- Los datos no cambian constantemente.

Ejemplos:

- Inventarios
- Productos
- Usuarios
- Cursos

---

# ¿Cuándo usar GraphQL?

GraphQL es recomendable cuando:

- Existen aplicaciones móviles.
- El cliente necesita distintos datos dependiendo de la pantalla.
- Existen muchas relaciones entre entidades.
- Se desea reducir el tráfico de red.

Ejemplos.

- Redes sociales.
- Aplicaciones móviles.
- Dashboards.
- Sistemas con muchas consultas.

---

# Persistencia

La persistencia consiste en almacenar información de manera permanente.

Ejemplos:

- Usuarios.
- Productos.
- Ventas.
- Cursos.
- Pedidos.

La información permanece incluso después de apagar el servidor.

Generalmente se utiliza una base de datos.

```
Aplicación

↓

Base de datos

↓

Información permanente
```

---

# Caché

La caché es un almacenamiento temporal.

Su objetivo es responder más rápido.

No reemplaza la base de datos.

Ejemplo.

```
Cliente

↓

Cache

↓

Base de datos
```

Si la información ya existe en la caché:

- No es necesario consultar la base de datos.

---

# Persistencia vs Caché

| Persistencia | Caché |
|---------------|---------|
| Permanente | Temporal |
| Fuente oficial | Copia de información |
| Puede sobrevivir reinicios | Puede perderse |
| Mayor capacidad | Menor capacidad |
| Más lenta | Mucho más rápida |

---

# ¿Cuándo usar Persistencia?

Cuando la información no puede perderse.

Ejemplos.

- Usuarios.
- Compras.
- Facturas.
- Pagos.
- Historia clínica.
- Matrículas.

---

# ¿Cuándo usar Caché?

Cuando la información se consulta frecuentemente.

Ejemplos.

- Catálogo de productos.
- Cursos.
- Configuración.
- Noticias.
- Ranking.
- Resultados de búsqueda.

---

# Ejemplo práctico

Supongamos un sistema universitario.

Cada estudiante consulta constantemente el listado de programas académicos.

Los programas cambian muy pocas veces al año.

Sin caché:

```
1000 estudiantes

↓

1000 consultas

↓

Base de datos
```

Con caché:

```
1000 estudiantes

↓

Cache

↓

1 consulta inicial

↓

Base de datos
```

La carga del servidor disminuye considerablemente.

---

# Resumen

| Concepto | Propósito |
|-----------|-----------|
| Arquitectura | Organizar el sistema |
| API | Comunicar aplicaciones |
| REST | API basada en recursos |
| GraphQL | API donde el cliente define la información |
| Persistencia | Almacenamiento permanente |
| Caché | Almacenamiento temporal para acelerar consultas |

---

# Buenas prácticas

Al diseñar una arquitectura recuerde:

- Comprender primero el problema.
- Identificar actores y funcionalidades.
- Diseñar componentes con responsabilidades claras.
- Elegir el tipo de API según las necesidades del sistema.
- Utilizar persistencia para información crítica.
- Utilizar caché para mejorar el rendimiento.
- Evitar incorporar tecnologías innecesarias.
- Justificar cada decisión arquitectónica.