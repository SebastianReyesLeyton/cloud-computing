# Balanceamiento de carga, Reverse Proxy y Arquitecturas Distribuidas con Docker

**Curso:** Computación en la Nube e Internet de las Cosas

## 1. Introducción

Cuando una aplicación deja de ser utilizada por unos pocos usuarios y comienza a recibir muchas solicitudes concurrentes, aparece una pregunta fundamental de arquitectura:

> ¿Cómo conseguimos que varias instancias de una aplicación trabajen juntas como si fueran un único servicio para el usuario?

Una de las respuestas más importantes es el **balanceamiento de carga (load balancing)**.

En una arquitectura distribuida, un cliente normalmente no necesita conocer cuál instancia concreta atenderá su solicitud. El cliente se conecta a un punto de entrada único y un componente intermedio decide hacia qué backend enviar cada petición.

En esta clase se utilizarán:

- Python y Flask para construir APIs sencillas.
- Docker para empaquetar las aplicaciones.
- Docker Compose para ejecutar varios servicios.
- Nginx como reverse proxy y load balancer.
- PostgreSQL como base de datos relacional.
- Python para consultar información desde PostgreSQL.
- MongoDB como ejemplo de base de datos documental.

El objetivo no es solamente ejecutar contenedores, sino comprender cómo una decisión de infraestructura afecta atributos de calidad del software y la arquitectura completa.

---

# 2. ¿Qué es el balanceamiento de carga?

El **balanceamiento de carga** consiste en distribuir las solicitudes de los clientes entre varias instancias capaces de prestar el mismo servicio.

Conceptualmente:

```text
                 Cliente
                    |
                    v
            +----------------+
            |  Load Balancer |
            +----------------+
              /      |      \
             /       |       \
            v        v        v
         API 1     API 2     API 3
```

En lugar de que todos los clientes accedan directamente a una única instancia:

```text
Cliente -> API única
```

se puede tener:

```text
Cliente -> Load Balancer -> API 1
                         -> API 2
                         -> API 3
```

El balanceador decide qué instancia atenderá cada solicitud según una estrategia de distribución.

---

# 3. ¿Por qué necesitamos balanceamiento de carga?

Una sola instancia puede convertirse en un cuello de botella.

Supongamos una API Flask que puede procesar aproximadamente 100 solicitudes por segundo y que en determinado momento recibe 300 solicitudes por segundo.

Si existe una sola instancia:

```text
300 req/s -> API 1
```

La aplicación puede comenzar a responder lentamente o incluso dejar de responder correctamente.

Si tenemos tres instancias similares:

```text
                  +--> API 1
300 req/s -> LB ---+--> API 2
                  +--> API 3
```

podemos distribuir aproximadamente 100 solicitudes por segundo a cada instancia, dependiendo de la estrategia y de las características reales de cada backend.

Esto no significa que simplemente multiplicar instancias siempre multiplique exactamente el rendimiento. El resultado depende de la capacidad de los backends, la base de datos, la red, el patrón de tráfico, las conexiones y muchos otros factores.

---

# 4. Funciones principales de un Load Balancer

Un load balancer puede encargarse de:

- Recibir solicitudes del cliente.
- Seleccionar un backend.
- Distribuir tráfico.
- Detectar backends no disponibles.
- Evitar temporalmente enviar tráfico a instancias que fallaron.
- Terminar conexiones HTTP o TLS, dependiendo de la arquitectura.
- Implementar políticas de afinidad de sesión, cuando sean necesarias.
- Exponer un único punto de entrada.
- Facilitar escalamiento horizontal.

En arquitecturas modernas también puede integrarse con:

- Health checks.
- TLS termination.
- Rate limiting.
- Autenticación.
- Observabilidad.
- Enrutamiento por host o por ruta.

---

# 5. Estrategias de balanceamiento

## 5.1 Round Robin

Distribuye las solicitudes de forma secuencial.

```text
Solicitud 1 -> API 1
Solicitud 2 -> API 2
Solicitud 3 -> API 3
Solicitud 4 -> API 1
Solicitud 5 -> API 2
```

Es sencilla y muy útil cuando las instancias tienen capacidades similares y las solicitudes tienen costos parecidos.

### Ventajas

- Fácil de entender.
- Fácil de configurar.
- Adecuado para backends homogéneos.

### Desventajas

- No considera automáticamente que una instancia pueda estar más ocupada que otra.
- No considera por sí mismo el costo real de cada solicitud.

---

## 5.2 Least Connections

Envía la nueva solicitud hacia el backend que tenga menos conexiones activas.

Es útil cuando las solicitudes pueden tener duraciones significativamente diferentes.

---

## 5.3 IP Hash / Sticky Sessions

La decisión se relaciona con la dirección IP del cliente o con una clave equivalente, intentando mantener a un cliente sobre una misma instancia.

Puede utilizarse cuando existe estado en memoria de la sesión, aunque muchas arquitecturas modernas prefieren diseñar servicios **stateless** y almacenar el estado fuera de las instancias.

---

## 5.4 Weighted Load Balancing

Permite asignar diferentes pesos a los backends.

Ejemplo:

```text
API 1 -> peso 3
API 2 -> peso 1
```

La primera instancia puede recibir aproximadamente más tráfico porque tiene una mayor capacidad.

---

# 6. Balanceamiento de carga y escalabilidad horizontal

Una de las ideas centrales que queremos enseñar es la diferencia entre:

**Escalamiento vertical**

```text
Servidor pequeño -> Servidor más potente
```

y

**Escalamiento horizontal**

```text
API 1 + API 2 + API 3 + ...
```

El balanceador permite que las múltiples instancias se presenten al cliente como un servicio único.

Esto es fundamental para arquitecturas distribuidas y para aplicaciones que necesitan crecer agregando más instancias.

---

# 7. Requerimientos no funcionales relacionados

El balanceamiento de carga no es un atributo de calidad por sí mismo. Es un mecanismo arquitectónico que puede contribuir al cumplimiento de varios **requerimientos no funcionales (RNF)**.

## 7.1 Rendimiento

Puede mejorar la capacidad de la arquitectura para atender múltiples solicitudes concurrentes al distribuir el trabajo entre diferentes instancias.

## 7.2 Escalabilidad

Facilita agregar o retirar instancias sin cambiar el punto de entrada usado por el cliente.

Ejemplo:

```text
Antes:
LB -> API 1

Después:
LB -> API 1
  -> API 2
  -> API 3
```

## 7.3 Disponibilidad

Si existen varias instancias y el balanceador detecta fallos, puede evitar enviar solicitudes hacia un backend no saludable.

Importante: el balanceador por sí solo no garantiza alta disponibilidad. También se necesita redundancia suficiente, monitoreo, despliegue adecuado y eliminación de puntos únicos de falla.

## 7.4 Tolerancia a fallos

La arquitectura puede continuar atendiendo solicitudes si una instancia falla, siempre que existan otras instancias saludables.

## 7.5 Mantenibilidad

Permite aislar ciertas tareas operativas. Por ejemplo, puede retirarse temporalmente una instancia del tráfico para actualizarla.

## 7.6 Flexibilidad

Facilita cambiar la cantidad y configuración de backends sin modificar directamente el cliente.

---

# 8. Principios de calidad del software relacionados

Desde una perspectiva de arquitectura, el balanceamiento puede apoyar principios como:

## Separación de responsabilidades

El backend se concentra en la lógica de negocio, mientras otro componente puede encargarse del ingreso y distribución del tráfico.

## Bajo acoplamiento

El cliente no necesita conocer la ubicación de cada instancia backend.

## Modularidad

La capa de entrada y la capa de aplicación pueden evolucionar independientemente hasta cierto punto.

## Elasticidad

Es posible aumentar o disminuir la cantidad de instancias según la demanda.

## Disponibilidad

Se puede evitar que la caída de una única instancia implique necesariamente la caída completa del servicio.

## Observabilidad

El punto de entrada central puede convertirse en un lugar importante para recopilar métricas y registros de acceso.

---

# 9. ¿Dónde se ubica el Load Balancer en la arquitectura?

Una arquitectura sencilla podría ser:

```text
Cliente
   |
   v
+----------------+
| Nginx / LB     |
+----------------+
   |      |      |
   v      v      v
 API-1   API-2   API-3
   |      |      |
   +------+------+
          |
          v
      Base de datos
```

El cliente ve un único endpoint, por ejemplo:

```text
http://localhost/api
```

pero Nginx puede distribuir las solicitudes entre varios contenedores Flask.

---

# 10. ¿Qué es un Reverse Proxy?

Un **reverse proxy** es un servidor que recibe solicitudes de los clientes y las reenvía a uno o más servidores backend.

El cliente no necesita conocer directamente la dirección del backend.

```text
Cliente -> Reverse Proxy -> Backend
```

Un reverse proxy puede ofrecer funciones como:

- Enrutamiento.
- Terminación TLS.
- Ocultamiento de la topología interna.
- Compresión.
- Caché.
- Control de acceso.
- Headers.
- Logging.
- Rate limiting.
- Balanceamiento de carga.

Por eso un reverse proxy puede actuar también como load balancer, pero los conceptos no son idénticos.

---

# 11. Diferencia entre Reverse Proxy y Load Balancer

La diferencia principal está en la **función que se quiere destacar**.

## Reverse Proxy

Su responsabilidad fundamental es actuar como intermediario entre clientes y servidores internos.

```text
Cliente -> Reverse Proxy -> Backend
```

Puede existir incluso un único backend.

## Load Balancer

Su objetivo fundamental es distribuir tráfico entre múltiples backends.

```text
Cliente -> Load Balancer -> Backend 1
                       -> Backend 2
                       -> Backend 3
```

## Un mismo componente puede hacer ambas cosas

Nginx puede funcionar como:

```text
                Nginx
       +---------------------+
       | Reverse Proxy       |
       | + Load Balancer     |
       +---------------------+
          /        |        \
       API 1     API 2     API 3
```

En otras palabras:

> Reverse proxy describe el rol de intermediario. Load balancing describe la capacidad de distribuir tráfico entre múltiples destinos.

---

# 12. Ventajas del Reverse Proxy

- Oculta la topología interna.
- Proporciona un punto de entrada único.
- Simplifica routing.
- Puede centralizar TLS.
- Permite aplicar políticas de seguridad.
- Puede servir contenido estático.
- Puede agregar caché y compresión.
- Puede centralizar logs.
- Puede realizar balanceamiento.

# 13. Desventajas del Reverse Proxy

- Agrega complejidad.
- Puede convertirse en un punto único de falla si no se replica.
- Introduce latencia adicional.
- Requiere configuración y monitoreo.
- Una mala configuración puede bloquear todo el tráfico.

---

# 14. Ventajas del Load Balancer

- Permite escalamiento horizontal.
- Distribuye trabajo.
- Puede aumentar disponibilidad.
- Puede retirar instancias enfermas del tráfico.
- Permite crecer sin exponer cada backend directamente.

# 15. Desventajas del Load Balancer

- Añade un componente adicional.
- Puede aumentar el costo operativo.
- Requiere configuración correcta de health checks y timeouts.
- Puede convertirse en un cuello de botella si se diseña incorrectamente.
- La persistencia de sesiones puede complicar la arquitectura.

---

# 16. ¿Cuándo usar un Load Balancer?

Es recomendable cuando:

- Existen varias instancias de una aplicación.
- Se necesita escalamiento horizontal.
- Hay una carga significativa.
- Se requiere tolerancia a fallos entre instancias.
- Se desea ocultar la ubicación de los backends.
- Se necesita distribuir tráfico entre diferentes servicios o servidores.

Ejemplos:

- APIs con mucho tráfico.
- Aplicaciones web empresariales.
- Microservicios.
- Plataformas de comercio electrónico.
- Sistemas con crecimiento dinámico.

---

# 17. ¿Cuándo usar Reverse Proxy?

Es recomendable cuando se necesita:

- Un único punto de entrada.
- Routing por URL.
- Routing por dominio.
- Terminación TLS.
- Caching.
- Compresión.
- Seguridad de acceso.
- Ocultar servidores internos.
- Servir contenido estático.

Ejemplo:

```text
/           -> frontend
/api        -> backend
/images     -> servidor estático
/auth       -> servicio de autenticación
```

---

# 18. ¿Cuándo usar ambos?

Una aplicación real puede utilizar las dos funciones simultáneamente.

```text
                      Internet
                         |
                         v
              +----------------------+
              | Nginx                |
              | Reverse Proxy + LB   |
              +----------------------+
                  /      |       \
                 /       |        \
                v        v         v
             Flask-1  Flask-2   Flask-3
                 \        |        /
                  \       |       /
                   +------v------+
                          |
                       PostgreSQL
```

Esta arquitectura será reproducida progresivamente en los workshops.

---

# 19. Persistencia y balanceamiento

Al utilizar múltiples instancias de Flask hay una decisión importante:

> ¿Dónde vive el estado de la aplicación?

Si una instancia almacena información importante solamente en memoria:

```text
Cliente -> API 1 -> sesión en memoria

Siguiente solicitud -> API 2
```

podríamos perder contexto.

Por eso las aplicaciones que serán balanceadas suelen diseñarse como **stateless** siempre que sea posible, dejando el estado compartido en servicios externos como:

- PostgreSQL.
- Redis.
- MongoDB.
- Sistemas de almacenamiento externos.

La base de datos se convierte así en un recurso compartido por varias instancias de API.

---

# 20. Balanceamiento y bases de datos

Es importante aclarar que balancear las APIs no significa automáticamente balancear la base de datos.

Podemos tener:

```text
             Nginx
               |
        +------+------+ 
        |      |      |
      API1   API2   API3
        |      |      |
        +------+------+ 
               |
          PostgreSQL
```

Aquí el tráfico HTTP está balanceado, pero PostgreSQL continúa siendo un único servicio.

En ambientes reales se pueden utilizar técnicas adicionales para escalar la base de datos, tales como:

- Réplicas de lectura.
- Connection pooling.
- Particionamiento.
- Sharding, cuando sea apropiado.
- Servicios administrados.

Estas técnicas son un tema diferente y no deben confundirse con el balanceamiento de APIs.

---

# 21. Ejemplo didáctico con Nginx

Supongamos tres instancias Flask:

```text
flask-1:5000
flask-2:5000
flask-3:5000
```

Nginx define un upstream:

```nginx
upstream flask_backend {
    server api1:5000;
    server api2:5000;
    server api3:5000;
}
```

Y luego:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://flask_backend;
    }
}
```

El navegador utiliza:

```text
http://localhost
```

pero Nginx distribuye las solicitudes entre los tres servicios.

---

# 22. ¿Cómo demostrar que realmente existe balanceamiento?

Una técnica pedagógica muy útil es hacer que cada instancia responda con un identificador distinto.

Por ejemplo:

```json
{
  "instance": "api-1"
}
```

Luego:

```json
{
  "instance": "api-2"
}
```

y:

```json
{
  "instance": "api-3"
}
```

Al ejecutar varias solicitudes contra el mismo endpoint, el estudiante puede observar cómo cambia la instancia que responde.

Este mecanismo permite hacer visible un comportamiento que normalmente sería transparente para el usuario.

---

# 23. Conceptos de arquitectura que se deben reforzar

Durante la práctica es importante que el estudiante diferencie:

- Imagen Docker.
- Contenedor.
- Servicio Docker Compose.
- Instancia de una aplicación.
- Reverse proxy.
- Load balancer.
- Backend.
- API.
- Base de datos.
- Escalamiento horizontal.
- Estado y statelessness.
- Punto único de falla.

---

# 24. Relación con atributos de calidad

| Atributo de calidad | ¿Cómo ayuda la arquitectura? |
|---|---|
| Rendimiento | Distribuye el procesamiento entre varias instancias |
| Escalabilidad | Permite agregar instancias |
| Disponibilidad | Permite continuar operando si otras instancias siguen saludables |
| Tolerancia a fallos | Reduce el impacto de fallos individuales |
| Mantenibilidad | Permite retirar y actualizar instancias individualmente |
| Seguridad | El reverse proxy puede centralizar controles |
| Observabilidad | Puede centralizar logs y métricas de acceso |
| Flexibilidad | Facilita cambiar backends sin afectar al cliente |

---

# 25. Resumen conceptual

La idea central de la clase es:

```text
                    CLIENTE
                       |
                       v
              +----------------+
              | NGINX          |
              | Reverse Proxy  |
              | Load Balancer  |
              +----------------+
                 /     |     \
                v      v      v
             Flask1 Flask2 Flask3
                \      |      /
                 \     |     /
                  +----v----+
                       |
                  PostgreSQL
```

El estudiante deberá comprender que:

1. Docker permite empaquetar la aplicación.
2. Docker Compose permite ejecutar múltiples servicios.
3. Flask implementa el backend.
4. Nginx puede funcionar como reverse proxy.
5. Nginx también puede balancear tráfico.
6. PostgreSQL puede convertirse en el estado compartido de múltiples APIs.
7. La arquitectura resultante soporta mejor ciertos requerimientos no funcionales, aunque introduce nuevas decisiones y posibles puntos de falla.

---

# 26. Preguntas de análisis

1. ¿Por qué una sola instancia Flask puede convertirse en un cuello de botella?
2. ¿Cuál es la diferencia entre escalamiento vertical y horizontal?
3. ¿Por qué un reverse proxy puede existir sin balanceamiento?
4. ¿Por qué un load balancer normalmente trabaja con varias instancias?
5. ¿Por qué es conveniente que una API balanceada sea stateless?
6. ¿Qué ocurre si una de las tres instancias Flask se detiene?
7. ¿Qué ocurre si Nginx se detiene?
8. ¿El balanceamiento de APIs resuelve el problema de escalabilidad de PostgreSQL?
9. ¿Qué requerimientos no funcionales podrían verse afectados por una mala configuración del load balancer?
10. ¿Qué componentes adicionales serían necesarios para llevar esta arquitectura a producción?
