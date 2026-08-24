# Workshop 3 - Docker: Introducción a los Contenedores

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:
    - Docker

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Comprender el concepto de contenedor.
- Diferenciar imágenes y contenedores.
- Descargar imágenes desde Docker Hub.
- Ejecutar y administrar contenedores.
- Inspeccionar recursos Docker.

---

# Conceptos que se trabajarán

- Docker
- Imagen
- Contenedor
- Docker Hub
- Dockerfile
- Redes
- Volúmenes
- Logs
- Persistencia

---

# Comandos que se utilizarán

| Comando | Descripción |
|----------|-------------|
| docker --version | Verificar instalación |
| docker pull | Descargar imágenes |
| docker images | Listar imágenes |
| docker run | Crear contenedores |
| docker ps | Mostrar contenedores |
| docker stop | Detener contenedores |
| docker start | Iniciar contenedores |
| docker rm | Eliminar contenedores |
| docker rmi | Eliminar imágenes |
| docker exec | Ejecutar comandos |
| docker logs | Ver registros |
| docker inspect | Información detallada |
| docker network ls | Redes |
| docker volume ls | Volúmenes |
| docker system df | Uso de disco |
| docker system prune | Limpiar recursos |

---

# Parte 1. Introducción a Docker

## Conceptos fundamentales

- Imagen
- Contenedor
- Dockerfile
- Docker Hub

## ¿Por qué utilizar Docker?

- Portabilidad.
- Escalabilidad.
- Aislamiento.
- Integración con CI/CD.
- Despliegue de microservicios.

---

# Parte 2. Explorando Docker

Ejecute:

```bash
docker --version
docker images
docker system df
```

Analice la salida obtenida.

---

# Parte 3. Descarga de imágenes

Descargue:

```bash
docker pull ubuntu
docker pull nginx
```

Verifique:

```bash
docker images
```

---

# Parte 4. Primer contenedor

Ejecute:

```bash
docker run -it ubuntu bash
```

Dentro del contenedor:

```bash
pwd
ls
whoami
exit
```

---

# Parte 5. Contenedor Web

Ejecute:

```bash
docker run -d -p 8080:80 nginx
```

Verifique:

```bash
docker ps
```

Acceda desde el navegador:

```
http://localhost:8080
```

---

# Parte 6. Administración

Practique los siguientes comandos:

```bash
docker stop
docker start
docker logs
docker exec
docker inspect
docker rm
docker rmi
```

Explique qué hace cada uno.

---

# Parte 7. Recursos Docker

Explore:

```bash
docker network ls
docker volume ls
docker system df
docker system prune
```

Analice qué recursos administra cada comando.

---

# Reto Integrador

Implemente un entorno compuesto por:

- Un contenedor Ubuntu.
- Un contenedor Nginx.
- Acceda al contenedor Ubuntu mediante `docker exec`.
- Consulte los logs del contenedor Nginx.
- Detenga ambos contenedores.
- Elimine los contenedores.
- Elimine las imágenes descargadas.

Documente cada comando utilizado y describa el propósito de cada uno.

---

# Preguntas de reflexión

1. ¿Cuál es la diferencia entre una imagen y un contenedor?
2. ¿Por qué Docker facilita el despliegue en la nube?
3. ¿Qué ventajas ofrece Docker frente a instalar aplicaciones directamente en el sistema operativo?
4. ¿En qué situaciones utilizaría un volumen Docker?
5. ¿Qué riesgos existen al ejecutar `docker system prune` en un servidor de producción?