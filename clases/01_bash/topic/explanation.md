# Clase 01

# Concepto: Bash

## Que es bash?

Bash es un lenguage de scripting que nos permite interactuar con el sistema operativo Unix/Linux. Dentro de las opciones que este brinda alguna son:

- Ejecutar comandos
- Automatizar tareas a traves de scripts
- Administrar archivos dentro de nuestros directorios
- Interactuar con herramientas (python, git, ...)

En entornos empresariales es realmente usado para construir o automatizar pasos para despliegues, ejecutar comandos repetitivos, manipular información.

## Comandos

https://www.w3schools.com/bash/bash_commands.php


## Actividades nivelación de Bash

Realizar los siguientes workshops:

- workshop_1.md
- workshop_2.md
- workshop_3.md

---

# Docker: Introducción a los Contenedores

## ¿Qué es Docker?

Docker es una plataforma de software de código abierto que permite **crear, distribuir y ejecutar aplicaciones dentro de contenedores**. Estos contenedores encapsulan una aplicación junto con todas sus dependencias, bibliotecas y configuraciones necesarias para funcionar de manera consistente en cualquier entorno.

En otras palabras, Docker elimina el clásico problema de:

> "En mi computador sí funciona."

Al ejecutar una aplicación dentro de un contenedor, se garantiza que el mismo software se comportará igual en el computador del desarrollador, en un servidor de pruebas o en un entorno de producción en la nube.

---

## ¿Qué es un contenedor?

Un **contenedor** es una unidad ligera de software que incluye:

- La aplicación.
- Sus bibliotecas.
- Sus dependencias.
- Variables de entorno.
- Configuraciones.
- Archivos necesarios para ejecutarse.

Todos estos elementos viajan juntos, haciendo que la aplicación sea portable y reproducible.

A diferencia de una máquina virtual, un contenedor **no contiene un sistema operativo completo**, sino que comparte el kernel del sistema operativo anfitrión.

---

## ¿Por qué utilizar Docker?

Antes de Docker era común encontrar problemas como:

- Diferencias entre sistemas operativos.
- Versiones distintas de librerías.
- Dependencias incompatibles.
- Instalaciones manuales complicadas.
- Errores durante el despliegue.

Docker soluciona estos problemas mediante la estandarización del entorno de ejecución.

### Beneficios principales

- Portabilidad.
- Rapidez.
- Bajo consumo de recursos.
- Fácil distribución.
- Escalabilidad.
- Automatización.
- Compatibilidad con plataformas Cloud.

---

## Arquitectura de Docker

Docker está compuesto por varios elementos.

```
               Docker Hub
                    │
                    │ pull / push
                    ▼
             +----------------+
             | Docker Engine  |
             +----------------+
                    │
      ┌─────────────┼─────────────┐
      │             │             │
      ▼             ▼             ▼
 Contenedor 1   Contenedor 2   Contenedor 3
```

Los componentes principales son:

- Docker Engine
- Docker Images
- Docker Containers
- Docker Registry (Docker Hub)
- Dockerfile
- Docker Compose

---

## Docker Engine

Es el motor principal de Docker.

Se encarga de:

- Descargar imágenes.
- Crear contenedores.
- Ejecutarlos.
- Administrar redes.
- Administrar almacenamiento.
- Gestionar recursos.

Cuando se ejecuta un comando como:

```bash
docker run nginx
```

quien realmente realiza el trabajo es Docker Engine.

---

## Docker Hub

Docker Hub es el repositorio oficial de imágenes Docker.

Su funcionamiento es similar a GitHub, pero en lugar de almacenar código fuente, almacena imágenes listas para ejecutarse.

Algunas imágenes populares son:

- Ubuntu
- Debian
- Alpine
- Nginx
- Apache
- MySQL
- PostgreSQL
- Redis
- MongoDB
- Node.js
- Python
- Java

Las imágenes pueden descargarse mediante:

```bash
docker pull nginx
```

---

## Imagen Docker

Una imagen es una plantilla inmutable que contiene todo lo necesario para crear un contenedor.

Una imagen incluye:

- Sistema operativo base.
- Librerías.
- Dependencias.
- Aplicación.
- Configuración inicial.

Puede compararse con un molde.

```
Imagen
   │
   ├────────► Contenedor 1
   ├────────► Contenedor 2
   └────────► Contenedor 3
```

Una misma imagen puede generar cientos o miles de contenedores.

---

## Contenedor Docker

Un contenedor es una instancia en ejecución de una imagen.

Ejemplo:

```bash
docker run nginx
```

Docker:

1. Busca la imagen.
2. Si no existe, la descarga.
3. Crea un contenedor.
4. Lo ejecuta.

---

## Diferencia entre Imagen y Contenedor

| Imagen | Contenedor |
|---------|------------|
| Es una plantilla | Es una instancia ejecutándose |
| Es inmutable | Puede modificarse mientras está activo |
| Se almacena en disco | Se ejecuta en memoria |
| Puede generar muchos contenedores | Proviene de una imagen |

---

## Dockerfile

Un Dockerfile es un archivo de texto que contiene las instrucciones necesarias para construir una imagen Docker.

Ejemplo:

```Dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python","main.py"]
```

Cada línea representa una instrucción para construir la imagen.

---

## Docker Compose

Docker Compose permite ejecutar múltiples contenedores simultáneamente.

Ejemplo:

```yaml
services:

  web:
    image: nginx

  db:
    image: postgres
```

Con un solo comando:

```bash
docker compose up
```

se crean ambos servicios automáticamente.

---

## Ciclo de vida de un contenedor

```
Imagen

   │

docker run

   ▼

Contenedor creado

   │

docker start

   ▼

Contenedor ejecutándose

   │

docker stop

   ▼

Contenedor detenido

   │

docker rm

   ▼

Eliminado
```

---

## Redes en Docker

Los contenedores pueden comunicarse entre sí mediante redes virtuales.

Docker crea automáticamente una red llamada:

```
bridge
```

También es posible crear redes personalizadas.

Ejemplo:

```bash
docker network create mi_red
```

---

## Volúmenes

Los contenedores son efímeros.

Cuando un contenedor se elimina, toda la información almacenada dentro desaparece.

Para evitar la pérdida de información existen los **volúmenes**.

```
Contenedor
      │
      ▼
 Volumen Docker
```

Ejemplo:

```bash
docker volume create datos_mysql
```

Los volúmenes permiten almacenar información de manera persistente.

---

## Registros (Logs)

Cada contenedor genera registros sobre su ejecución.

Estos registros son muy útiles para:

- Detectar errores.
- Monitorear aplicaciones.
- Analizar comportamiento.
- Depurar problemas.

Consultar los registros:

```bash
docker logs mi_contenedor
```

---

## Inspección de contenedores

Docker almacena gran cantidad de información sobre cada contenedor.

Puede consultarse mediante:

```bash
docker inspect mi_contenedor
```

Esta información incluye:

- Dirección IP.
- Variables de entorno.
- Puertos.
- Redes.
- Volúmenes.
- Configuración interna.

---

## Recursos administrados por Docker

Docker administra distintos tipos de recursos.

### Imágenes

```bash
docker images
```

### Contenedores

```bash
docker ps
```

### Redes

```bash
docker network ls
```

### Volúmenes

```bash
docker volume ls
```

### Uso de disco

```bash
docker system df
```

---

## Flujo típico de trabajo con Docker

```
Escribir aplicación

        │

Crear Dockerfile

        │

docker build

        │

Crear imagen

        │

docker run

        │

Crear contenedor

        │

Pruebas

        │

docker push

        │

Docker Hub

        │

Servidor Cloud

        │

docker pull

        │

Aplicación desplegada
```

---

## Ventajas de Docker

- Portabilidad entre sistemas operativos.
- Reproducibilidad de entornos.
- Menor consumo de recursos que una máquina virtual.
- Arranque rápido de aplicaciones.
- Fácil integración con herramientas DevOps.
- Ideal para arquitecturas de microservicios.
- Facilita el desarrollo colaborativo.
- Compatible con Kubernetes y plataformas Cloud.

---

## Limitaciones de Docker

- Comparte el kernel del sistema operativo anfitrión.
- No reemplaza completamente una máquina virtual.
- Requiere una adecuada administración de imágenes.
- Puede generar consumo elevado de almacenamiento si no se eliminan imágenes y contenedores antiguos.

---

## Casos de uso

Docker es ampliamente utilizado para:

- Desarrollo de software.
- Microservicios.
- Integración continua (CI).
- Despliegue continuo (CD).
- Computación en la nube.
- Internet de las Cosas (IoT).
- Ciencia de datos.
- Inteligencia Artificial.
- Automatización de pruebas.
- Laboratorios académicos.

---

## Docker en Computación en la Nube

Docker es una de las tecnologías fundamentales en la computación en la nube.

Plataformas como:

- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)

permiten desplegar aplicaciones Docker de manera sencilla mediante servicios especializados.

Entre los servicios más utilizados se encuentran:

- Amazon ECS
- Amazon EKS
- Azure Container Apps
- Azure Kubernetes Service (AKS)
- Google Kubernetes Engine (GKE)
- Cloud Run

Esto permite escalar aplicaciones automáticamente, reducir costos y simplificar el despliegue de software.

---

## Resumen

Docker es una plataforma que permite empaquetar aplicaciones y todas sus dependencias en contenedores ligeros y portables. Gracias a esta tecnología es posible desarrollar, probar y desplegar aplicaciones de forma consistente en cualquier entorno, convirtiéndose en una herramienta esencial para el desarrollo moderno, la administración de infraestructura y la computación en la nube.

---

# Referencias para estudio

- Bash: https://www.youtube.com/watch?v=tK9Oc6AEnR4
- Docker: https://www.youtube.com/watch?v=9eTVZwMZJsA
- Tutorial con ejercicios bash: https://www.w3schools.com/bash/index.php
