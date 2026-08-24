# Explanation - Dockerfile y Docker Compose

**Curso:** Computación en la Nube e Internet de las Cosas

**Propósito:** Material conceptual de apoyo para comprender cómo se construyen imágenes Docker y cómo Docker Compose permite definir y ejecutar aplicaciones compuestas por uno o varios servicios.

---

# 1. Imagen, contenedor y servicio

Antes de estudiar `Dockerfile` y `docker-compose.yml` es fundamental diferenciar tres conceptos.

## Imagen

Una imagen es una plantilla inmutable que contiene el sistema de archivos, librerías, herramientas y configuración necesarios para iniciar un contenedor.

Ejemplos:

```bash
docker pull ubuntu
docker pull nginx:alpine
```

## Contenedor

Un contenedor es una instancia creada a partir de una imagen.

```text
Imagen
   |
   +----> Contenedor A
   +----> Contenedor B
   +----> Contenedor C
```

Una imagen puede producir múltiples contenedores.

## Servicio

En Docker Compose, un servicio representa la definición de un componente de la aplicación. El servicio especifica, entre otras cosas, qué imagen utilizar o cómo construirla, qué puertos exponer y qué volúmenes montar.

---

# 2. ¿Qué es un Dockerfile?

Un `Dockerfile` es un archivo de texto que contiene instrucciones para construir una imagen Docker.

Podemos visualizar el proceso así:

```text
Dockerfile
    |
    | docker build
    v
Imagen Docker
    |
    | docker run
    v
Contenedor
```

El `Dockerfile` describe principalmente **cómo debe construirse la imagen**.

---

# 3. Anatomía de un Dockerfile

Un ejemplo sencillo:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y nginx

WORKDIR /app

COPY . /app

ENV APP_ENV=development

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Cada instrucción tiene una responsabilidad diferente.

---

# 4. `FROM`

```dockerfile
FROM ubuntu:24.04
```

Define la imagen base desde la cual se construirá la nueva imagen.

Otros ejemplos:

```dockerfile
FROM nginx:alpine
FROM python:3.13-slim
FROM node:24-alpine
```

La elección de la imagen base afecta:

- Tamaño de la imagen.
- Herramientas disponibles.
- Compatibilidad.
- Seguridad.
- Tiempo de construcción.

En general, una imagen base debe ser suficientemente completa para el propósito del servicio, pero no más grande de lo necesario.

---

# 5. `RUN`

```dockerfile
RUN apt-get update && apt-get install -y curl
```

`RUN` ejecuta comandos **durante la construcción de la imagen**.

Ejemplo:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y curl
```

Es posible combinar instrucciones para reducir capas innecesarias:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

Conceptualmente:

```text
Docker build
     |
     +--> RUN comando 1
     |
     +--> RUN comando 2
     |
     +--> RUN comando 3
     v
 Imagen terminada
```

---

# 6. `WORKDIR`

```dockerfile
WORKDIR /app
```

Define el directorio de trabajo para las instrucciones posteriores y para el proceso principal del contenedor.

Por ejemplo:

```dockerfile
WORKDIR /app
COPY . .
```

En este caso, el contenido del contexto se copia en `/app`.

`WORKDIR` es preferible a realizar múltiples instrucciones `cd`.

---

# 7. `COPY`

```dockerfile
COPY . /app
```

Copia archivos desde el **contexto de construcción** hacia la imagen.

Ejemplo:

```text
proyecto/
├── Dockerfile
├── monitor.sh
├── analiza.sh
└── access.log
```

Con:

```dockerfile
COPY . /app
```

los archivos del contexto se incorporan a `/app` dentro de la imagen.

Es recomendable utilizar `.dockerignore` para evitar copiar información innecesaria.

Ejemplo:

```text
.git
__pycache__
*.log
.env
node_modules
```

---

# 8. `ENV`

Permite definir variables de entorno dentro de la imagen.

```dockerfile
ENV APP_ENV=development
```

Estas variables pueden ser consultadas dentro del contenedor:

```bash
echo "$APP_ENV"
```

Una variable definida con `ENV` forma parte de la configuración de la imagen y puede ser modificada durante la ejecución utilizando opciones de Docker o Compose.

---

# 9. `EXPOSE`

```dockerfile
EXPOSE 80
```

Documenta que la aplicación dentro del contenedor utiliza el puerto 80.

**Importante:** `EXPOSE` no publica por sí mismo el puerto hacia el host.

Para publicar el puerto se utiliza, por ejemplo:

```bash
docker run -p 8080:80 nginx
```

La relación es:

```text
Host                  Contenedor
8080  --------------> 80
```

---

# 10. `CMD`

`CMD` define el comando por defecto que se ejecutará cuando se inicie el contenedor.

Ejemplo:

```dockerfile
CMD ["bash"]
```

Otro ejemplo:

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

`CMD` puede ser reemplazado cuando se pasan argumentos al final de `docker run`.

Ejemplo:

```dockerfile
FROM ubuntu:24.04
CMD ["echo", "Hola desde CMD"]
```

Por defecto:

```bash
docker run --rm mi-imagen
```

produce:

```text
Hola desde CMD
```

Pero si ejecutamos:

```bash
docker run --rm mi-imagen echo "Otro mensaje"
```

el comando indicado al ejecutar el contenedor reemplaza el `CMD`.

Una forma útil de recordarlo es:

> `CMD` proporciona un comportamiento predeterminado que puede ser reemplazado al ejecutar el contenedor.

---

# 11. `ENTRYPOINT`

`ENTRYPOINT` define el ejecutable principal del contenedor.

Ejemplo:

```dockerfile
ENTRYPOINT ["echo"]
```

Entonces:

```bash
docker run --rm mi-imagen "Hola"
```

equivale conceptualmente a ejecutar:

```text
echo Hola
```

Cuando se utiliza la forma exec, los argumentos entregados a `docker run` se agregan al `ENTRYPOINT`.

---

# 12. Diferencia entre `CMD` y `ENTRYPOINT`

| Característica | `CMD` | `ENTRYPOINT` |
|----------------|-------|--------------|
| Define comportamiento por defecto | Sí | Sí |
| Puede ser reemplazado fácilmente por argumentos de `docker run` | Sí | No de la misma forma |
| Define el ejecutable principal | No necesariamente | Sí |
| Útil para parámetros por defecto | Sí | Sí, especialmente combinado con `CMD` |

La idea más importante es:

```text
ENTRYPOINT = programa principal
CMD         = argumentos o valores por defecto
```

Por ejemplo:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Sin argumentos:

```bash
docker run mi-app
```

se obtiene conceptualmente:

```text
python app.py
```

Con un argumento distinto:

```bash
docker run mi-app otro.py
```

se obtiene:

```text
python otro.py
```

Esto permite crear imágenes que funcionan como herramientas reutilizables.

---

# 13. Formato `exec` y formato `shell`

Docker permite escribir instrucciones como:

```dockerfile
CMD ["echo", "Hola"]
```

Esta es la forma **exec**.

También existe:

```dockerfile
CMD echo Hola
```

Esta es la forma **shell**.

Para servicios y procesos principales suele ser conveniente utilizar la forma exec porque evita depender innecesariamente de un shell para iniciar el proceso.

---

# 14. Construcción de una imagen

Supongamos:

```text
proyecto/
├── Dockerfile
└── monitor.sh
```

Ejecutamos:

```bash
docker build -t mi-imagen .
```

La estructura conceptual es:

```text
                Contexto de build
                       |
             +---------+---------+
             |                   |
        Dockerfile          monitor.sh
             |                   |
             +---------+---------+
                       |
                       v
                 docker build
                       |
                       v
                  mi-imagen
```

El punto final (`.`) representa el contexto de construcción actual.

---

# 15. ¿Qué es Docker Compose?

Docker Compose permite definir y administrar aplicaciones compuestas por uno o varios servicios mediante un archivo YAML, normalmente llamado:

```text
docker-compose.yml
```

o:

```text
compose.yml
```

La idea principal es pasar de ejecutar manualmente muchos comandos a describir la aplicación de manera declarativa.

En lugar de:

```bash
docker run ...
docker run ...
docker run ...
```

podemos definir los servicios y luego ejecutar:

```bash
docker compose up
```

---

# 16. Anatomía de `docker-compose.yml`

Ejemplo:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Las partes principales son:

```text
services
   |
   +-- web
       |
       +-- image
       +-- ports
```

---

# 17. `services`

```yaml
services:
```

Es la sección donde se definen los componentes de la aplicación.

Por ejemplo:

```yaml
services:
  web:
    image: nginx:alpine

  database:
    image: postgres:18
```

Aquí existen dos servicios:

- `web`
- `database`

---

# 18. `image`

```yaml
image: nginx:alpine
```

Indica qué imagen se utilizará para crear los contenedores del servicio.

Ejemplos:

```yaml
image: nginx:alpine
image: redis:8
image: postgres:18
```

---

# 19. `build`

Cuando queremos construir nuestra propia imagen podemos utilizar:

```yaml
build: .
```

Ejemplo:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
```

Compose buscará el `Dockerfile` en el contexto especificado.

También se puede indicar una ruta específica:

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

---

# 20. `ports`

```yaml
ports:
  - "8080:80"
```

Relaciona un puerto del host con un puerto del contenedor.

```text
Host             Contenedor
8080  ----------> 80
```

Por eso Nginx será accesible mediante:

```text
http://localhost:8080
```

---

# 21. `volumes`

Permiten montar información entre el host y el contenedor.

Ejemplo:

```yaml
volumes:
  - ./html:/usr/share/nginx/html:ro
```

La estructura es:

```text
./html                         /usr/share/nginx/html
host        ---------------->  contenedor
```

`ro` significa **read-only**.

Una aplicación puede utilizar volúmenes para:

- Compartir archivos.
- Persistir información.
- Inyectar configuraciones.
- Facilitar desarrollo local.

---

# 22. `environment`

Permite proporcionar variables de entorno al servicio.

```yaml
environment:
  APP_ENV: development
  DEBUG: "true"
```

Dentro del contenedor:

```bash
echo "$APP_ENV"
```

---

# 23. `depends_on`

Permite expresar dependencias de inicio entre servicios.

Ejemplo:

```yaml
services:
  api:
    image: mi-api
    depends_on:
      - database

  database:
    image: postgres:18
```

Esto expresa una relación de dependencia entre los servicios.

**Importante:** `depends_on` no significa que la aplicación dependiente necesariamente pueda atender solicitudes inmediatamente. La disponibilidad real de un servicio puede requerir mecanismos adicionales como healthchecks y lógica de reintentos.

---

# 24. Redes en Docker Compose

Compose crea una red para los servicios del proyecto y permite que los contenedores se comuniquen mediante el nombre del servicio.

Por ejemplo:

```yaml
services:
  api:
    image: mi-api

  database:
    image: postgres:18
```

La API puede comunicarse con el servicio de base de datos utilizando:

```text
database
```

como nombre de host.

No es necesario utilizar `localhost` para referirse al otro contenedor.

---

# 25. `docker compose up`

Para iniciar todos los servicios:

```bash
docker compose up
```

En segundo plano:

```bash
docker compose up -d
```

Si se modificó el `Dockerfile` y se desea reconstruir la imagen:

```bash
docker compose up --build
```

---

# 26. `docker compose down`

Detiene y elimina los contenedores y recursos creados por Compose para el proyecto:

```bash
docker compose down
```

Esto es especialmente útil para limpiar el entorno del laboratorio.

---

# 27. Otros comandos útiles de Compose

Ver servicios:

```bash
docker compose ps
```

Ver logs:

```bash
docker compose logs
```

Ver logs de un servicio:

```bash
docker compose logs web
```

Seguir logs en tiempo real:

```bash
docker compose logs -f web
```

Ejecutar comandos dentro de un servicio:

```bash
docker compose exec web sh
```

---

# 28. Dockerfile vs Docker Compose

Una comparación útil para los estudiantes:

| Aspecto | Dockerfile | Docker Compose |
|---------|------------|----------------|
| Propósito | Construir una imagen | Definir y ejecutar servicios |
| Formato | Texto con instrucciones Docker | YAML |
| Describe | Cómo se construye una imagen | Cómo se ejecuta una aplicación compuesta |
| Principal comando | `docker build` | `docker compose up` |
| Puede definir varios servicios | No | Sí |
| Define puertos de publicación | No directamente | Sí |
| Define volúmenes de ejecución | No como mecanismo de Compose | Sí |
| Puede utilizar un Dockerfile | Es el propio Dockerfile | Sí, mediante `build` |

Una forma sencilla de recordarlo:

```text
Dockerfile
    ↓
Construye una IMAGEN

Docker Compose
    ↓
Organiza y ejecuta SERVICIOS
```

---

# 29. Flujo completo

Un proyecto típico puede tener esta estructura:

```text
proyecto/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── scripts/
│   ├── monitor.sh
│   └── analiza.sh
└── html/
    └── index.html
```

El flujo puede ser:

```text
Dockerfile
    |
    | docker build
    v
Imagen personalizada
    |
    | docker compose up
    v
Servicio / Contenedor
    |
    +---- puertos
    +---- volúmenes
    +---- variables
    +---- red
```

---

# 30. Buenas prácticas para este laboratorio

## Fijar versiones

En lugar de depender de etiquetas que cambian, para ejercicios reproducibles conviene fijar versiones concretas cuando sea necesario.

Ejemplo:

```dockerfile
FROM nginx:alpine
```

o, para mayor reproducibilidad, utilizar una referencia de versión concreta:

```dockerfile
FROM nginx:1.27-alpine
```

## Evitar imágenes innecesariamente grandes

Usar imágenes ligeras cuando sean apropiadas.

## No incluir secretos en la imagen

Evitar copiar contraseñas, llaves privadas o archivos `.env` directamente a una imagen.

## Utilizar `.dockerignore`

Evitar enviar al contexto de construcción archivos innecesarios.

## Un proceso principal por contenedor

Diseñar el contenedor alrededor de una responsabilidad principal y utilizar Compose para combinar servicios cuando corresponda.

---

# 31. Ejemplo integrador: Nginx personalizado

`Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY html/ /usr/share/nginx/html/
```

`docker-compose.yml`:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
```

Estructura:

```text
proyecto/
├── Dockerfile
├── docker-compose.yml
└── html/
    └── index.html
```

Ejecutar:

```bash
docker compose up --build -d
```

Abrir:

```text
http://localhost:8080
```

Detener:

```bash
docker compose down
```

---

# 32. Preguntas de comprensión

1. ¿Qué problema resuelve un `Dockerfile`?
2. ¿Qué problema resuelve Docker Compose?
3. ¿Cuál es la diferencia entre una imagen y un contenedor?
4. ¿Para qué sirve `FROM`?
5. ¿En qué momento se ejecuta `RUN`?
6. ¿Qué función cumple `COPY`?
7. ¿Qué diferencia existe entre `CMD` y `ENTRYPOINT`?
8. ¿Qué ocurre cuando se agrega un comando al final de `docker run`?
9. ¿Qué representa `8080:80`?
10. ¿Por qué un contenedor puede comunicarse con otro utilizando el nombre del servicio?
11. ¿Qué problema resuelve un volumen?
12. ¿Qué diferencia existe entre `image` y `build` en Compose?

---

# 33. Idea clave para recordar

```text
Dockerfile = ¿Cómo construyo la imagen?

Docker Image = ¿Qué contiene la imagen?

Container = ¿Dónde se ejecuta la imagen?

Docker Compose = ¿Cómo organizo y ejecuto varios servicios?
```