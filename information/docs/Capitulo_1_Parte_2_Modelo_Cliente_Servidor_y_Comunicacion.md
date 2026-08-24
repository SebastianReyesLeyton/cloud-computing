# Capítulo 1 -- Parte 2

# Modelo Cliente--Servidor y Comunicación

## Objetivos de aprendizaje

Al finalizar esta sección el estudiante podrá:

-   Explicar el modelo Cliente--Servidor.
-   Diferenciar comunicación síncrona y asíncrona.
-   Comprender el ciclo de vida de una petición.
-   Identificar los elementos de un Request y un Response.

------------------------------------------------------------------------

# El modelo Cliente--Servidor

La mayoría de aplicaciones modernas siguen este modelo.

-   **Cliente:** solicita información o servicios.
-   **Servidor:** procesa la solicitud y devuelve una respuesta.

``` text
+-----------+        Solicitud        +-----------+
|  Cliente  | ----------------------> | Servidor  |
| (Browser) |                         |   API     |
|   Móvil   | <---------------------- |           |
+-----------+        Respuesta        +-----------+
```

El cliente no accede directamente a la base de datos; toda interacción
pasa por el servidor.

------------------------------------------------------------------------

# Flujo completo de una petición

Cuando un usuario hace clic en un botón:

1.  El cliente crea una solicitud.
2.  La solicitud viaja por Internet.
3.  El servidor recibe la petición.
4.  La API valida la información.
5.  Se ejecuta la lógica de negocio.
6.  Se consulta la base de datos si es necesario.
7.  Se construye una respuesta.
8.  El cliente recibe y presenta el resultado.

``` text
Usuario
   │
Navegador
   │
Internet
   │
API
   │
Servicio
   │
Base de datos
   │
Servicio
   │
API
   │
Navegador
```

------------------------------------------------------------------------

# Comunicación síncrona

En una comunicación síncrona el cliente espera la respuesta antes de
continuar.

Ejemplo:

-   Consultar el saldo bancario.
-   Iniciar sesión.
-   Buscar un producto.

**Ventajas**

-   Simplicidad.
-   Respuesta inmediata.
-   Fácil de depurar.

**Desventajas**

-   El cliente queda bloqueado esperando.
-   Si el servidor tarda, la experiencia empeora.

------------------------------------------------------------------------

# Comunicación asíncrona

El cliente envía la solicitud y continúa ejecutándose sin esperar.

Ejemplos:

-   Envío de correos.
-   Procesamiento de imágenes.
-   Generación de reportes.

**Ventajas**

-   Mayor escalabilidad.
-   Mejor experiencia para tareas largas.

**Desventajas**

-   Mayor complejidad.
-   Puede requerir colas de mensajes o eventos.

------------------------------------------------------------------------

# Request y Response

## Request

Una petición HTTP normalmente contiene:

-   Método (GET, POST, PUT, DELETE)
-   URL
-   Headers
-   Parámetros
-   Body

Ejemplo:

``` http
POST /students
Content-Type: application/json

{
  "name":"Ana",
  "program":"Ingeniería"
}
```

## Response

Una respuesta suele contener:

-   Código de estado.
-   Headers.
-   Body.

``` http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id":15,
  "message":"Estudiante creado"
}
```

------------------------------------------------------------------------

# Códigos de estado más comunes

    Código Significado
  -------- --------------------
       200 OK
       201 Recurso creado
       400 Solicitud inválida
       401 No autenticado
       403 Prohibido
       404 No encontrado
       500 Error interno

------------------------------------------------------------------------

# Resumen

El modelo Cliente--Servidor separa responsabilidades entre quien consume
y quien ofrece servicios. Comprender el flujo de una petición y la
diferencia entre comunicación síncrona y asíncrona es fundamental para
diseñar APIs modernas.

------------------------------------------------------------------------

# Preguntas de reflexión

1.  ¿Qué ventajas tiene separar cliente y servidor?
2.  ¿Qué operaciones de un sistema universitario serían asíncronas?
3.  ¿Qué código HTTP devolvería una autenticación fallida?

------------------------------------------------------------------------

# Taller práctico

1.  Dibuje el flujo de una petición desde un navegador hasta una base de
    datos.
2.  Clasifique diez operaciones de una aplicación entre síncronas y
    asíncronas.
3.  Escriba un ejemplo de Request y Response para registrar un curso.
