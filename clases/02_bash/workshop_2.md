# Taller 2 - Construcción de imágenes con Dockerfile

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:

- Dockerfile
- Construcción de imágenes
- Ubuntu como imagen base
- Capas de imágenes
- `COPY`
- `RUN`
- `WORKDIR`
- `CMD`
- `ENTRYPOINT`

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Construir una imagen Docker a partir de Ubuntu.
- Comprender la estructura básica de un `Dockerfile`.
- Diferenciar instrucciones de construcción y ejecución.
- Incorporar scripts Bash dentro de una imagen.
- Ejecutar una imagen personalizada.
- Analizar las diferencias entre `CMD` y `ENTRYPOINT`.

---

# Conceptos que se trabajarán

- Imagen Docker
- Contenedor
- Dockerfile
- Imagen base
- Capas
- Contexto de construcción
- `FROM`
- `RUN`
- `WORKDIR`
- `COPY`
- `ENV`
- `EXPOSE`
- `CMD`
- `ENTRYPOINT`
- Build
- Run

---

# Comandos que se utilizarán

| Comando | Descripción |
|----------|-------------|
| `docker pull` | Descargar una imagen |
| `docker images` | Listar imágenes |
| `docker build` | Construir una imagen |
| `docker run` | Crear y ejecutar un contenedor |
| `docker ps` | Mostrar contenedores activos |
| `docker ps -a` | Mostrar todos los contenedores |
| `docker logs` | Mostrar logs de un contenedor |
| `docker exec` | Ejecutar un comando dentro de un contenedor |
| `docker rm` | Eliminar un contenedor |
| `docker rmi` | Eliminar una imagen |

---

# Parte 1. Crear una imagen Ubuntu para el laboratorio

Crear el archivo `Dockerfile`:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y \
    bash \
    coreutils \
    grep \
    gawk \
    procps \
    nano \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY . /app

CMD ["bash"]
```

Construir:

```bash
docker build -t ubuntu-bash-lab .
```

Ejecutar:

```bash
docker run -it --name ubuntu-lab ubuntu-bash-lab
```

Dentro del contenedor:

```bash
cd /app
ls -la
```

---

# Parte 2. Ejercicio CMD vs ENTRYPOINT

Crear tres imágenes pequeñas y observar qué sucede cuando se pasan argumentos al comando `docker run`.

## Caso A: solo CMD

```dockerfile
FROM ubuntu:latest
CMD ["echo", "Hola desde CMD"]
```

Construir:

```bash
docker build -t ejemplo-cmd .
```

Ejecutar:

```bash
docker run --rm ejemplo-cmd
```

Después probar:

```bash
docker run --rm ejemplo-cmd "Mensaje reemplazado"
```

Registrar qué ocurrió.

## Caso B: solo ENTRYPOINT

```dockerfile
FROM ubuntu:latest
ENTRYPOINT ["echo"]
```

Construir y probar:

```bash
docker build -t ejemplo-entrypoint .
docker run --rm ejemplo-entrypoint "Hola"
```

Después probar:

```bash
docker run --rm ejemplo-entrypoint "Mensaje adicional"
```

## Caso C: ENTRYPOINT + CMD

```dockerfile
FROM ubuntu:latest
ENTRYPOINT ["echo"]
CMD ["Mensaje por defecto"]
```

Probar:

```bash
docker run --rm ejemplo-combinado
docker run --rm ejemplo-combinado "Mensaje personalizado"
```

## Preguntas de análisis

1. ¿Cuál instrucción se comporta como valor por defecto?
2. ¿Cuál instrucción define el programa principal del contenedor?
3. ¿Qué sucede con `CMD` cuando se pasa un comando al final de `docker run`?
4. ¿Por qué `ENTRYPOINT` es útil cuando queremos que el contenedor se comporte como una herramienta concreta?
5. ¿Qué combinación utilizaría para crear un script que tenga un ejecutable fijo pero permita modificar parámetros?

---

# Entregables

- `Dockerfile` funcional.
- Imagen `ubuntu-bash-lab`.
- Tres ejemplos de `CMD`/`ENTRYPOINT`.
- Tabla o conclusión escrita con las diferencias observadas.
- Evidencia de construcción y ejecución.
