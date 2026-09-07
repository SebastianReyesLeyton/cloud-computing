# EC2: Introducción a Amazon Elastic Compute Cloud

**Curso:** Computación en la Nube e Internet de las Cosas  
**Clase:** Introducción práctica a Amazon EC2  
**Objetivo general:** Comprender qué es Amazon EC2, cómo está compuesto, cuáles son sus conceptos asociados, cuándo utilizarlo y cómo se relaciona con un proceso de *Journey to Cloud (J2C)*.

---

# 1. Introducción

En las clases anteriores trabajamos con Docker, Docker Compose, APIs, Nginx, balanceamiento de carga y bases de datos. Hasta este punto, la infraestructura se ha ejecutado principalmente en el computador local del estudiante.

El siguiente paso consiste en llevar estos conceptos a una infraestructura real en la nube.

Para ello utilizaremos **Amazon EC2 (Elastic Compute Cloud)**.

La pregunta central de esta clase es:

> **¿Cómo paso de tener una aplicación ejecutándose en mi computador a tener un servidor ejecutándose en AWS?**

```text
Computador local
      |
      | Docker / Docker Compose
      v
Aplicación funcionando localmente
      |
      | Cloud
      v
Amazon EC2
      |
      +--> Sistema operativo
      +--> Red
      +--> Seguridad
      +--> Almacenamiento
      +--> Aplicación
```

Amazon EC2 proporciona capacidad de cómputo escalable bajo demanda y una instancia EC2 es un servidor virtual dentro de AWS. El tipo de instancia determina la combinación de recursos de cómputo, memoria, red y almacenamiento disponible.[1]

---

# 2. ¿Qué es Amazon EC2?

**Amazon Elastic Compute Cloud (Amazon EC2)** es un servicio de AWS que proporciona capacidad de cómputo escalable bajo demanda.

En términos simples:

> **EC2 permite alquilar y administrar servidores virtuales en la infraestructura de AWS.**

En lugar de comprar un servidor físico, instalarlo, conectarlo a Internet, mantenerlo y reemplazar sus componentes, podemos crear una instancia virtual dentro de AWS y administrarla remotamente.

Una instancia EC2 puede utilizarse para ejecutar:

- APIs.
- Aplicaciones web.
- Servidores Nginx.
- Aplicaciones Flask.
- Aplicaciones Java.
- Aplicaciones Node.js.
- Docker y Docker Compose.
- Procesamiento de datos.
- Workers y procesos de larga duración.
- Sistemas empresariales.
- Herramientas de CI/CD.
- Servidores de monitoreo.

La característica fundamental es que EC2 proporciona **control sobre el servidor y su sistema operativo**.

---

# 3. EC2 como Infraestructura como Servicio

EC2 se encuentra principalmente dentro del modelo **IaaS — Infrastructure as a Service**.

AWS administra la infraestructura física subyacente, mientras que el cliente administra una parte importante de lo que se encuentra por encima de ella.

```text
AWS
├── Centros de datos
├── Hardware físico
├── Energía
├── Redes físicas
└── Infraestructura física
        |
        v
      EC2
        |
        +--> Sistema Operativo
        +--> Aplicaciones
        +--> Configuración
        +--> Docker
        +--> Nginx
        +--> Runtime
        +--> Datos
```

Esto hace que EC2 sea especialmente útil para entender qué ocurre **debajo de servicios más administrados**.

---

# 4. ¿Para qué sirve EC2?

EC2 es útil cuando necesitamos control sobre un entorno de computación.

## 4.1 Ejecutar una API

```text
Internet
   |
   v
 EC2
   |
   v
 Nginx
   |
   v
Flask API
```

## 4.2 Ejecutar Docker

```text
EC2
 |
 +--> Docker Engine
       |
       +--> container api
       +--> container nginx
       +--> container worker
```

## 4.3 Montar un laboratorio

```text
EC2
 |
 +--> Ubuntu
 +--> Docker
 +--> Nginx
 +--> Python
```

## 4.4 Ejecutar procesos persistentes

Por ejemplo:

- Workers.
- Procesadores de archivos.
- Aplicaciones internas.
- Servidores web.
- Servicios de integración.

---

# 5. Anatomía de una instancia EC2

Al crear una instancia EC2 no estamos seleccionando únicamente un "servidor".

Estamos configurando diferentes componentes:

```text
                    EC2 INSTANCE
                         |
        +----------------+----------------+
        |                |                |
       AMI        Instance Type       Storage
        |                |                |
       OS             CPU/RAM          EBS
        |
        +-----------------------------------+
        |                                   |
      Network                           Security
        |                                   |
      VPC                                SG / Key
        |
      Subnet
        |
      IP / DNS
```

Los principales componentes del lanzamiento son la AMI, el tipo de instancia, el par de claves, el grupo de seguridad, la VPC/subred y el almacenamiento EBS.[2]

---

# 6. Amazon Machine Image (AMI)

Una **Amazon Machine Image (AMI)** es una plantilla utilizada para crear una instancia.

Puede incluir:

- Sistema operativo.
- Software.
- Configuraciones.
- Componentes necesarios para iniciar el servidor.

AWS define las AMI como plantillas preconfiguradas que empaquetan los componentes necesarios para una instancia, incluido el sistema operativo y software adicional.[1]

```text
AMI Ubuntu
     |
     v
EC2 Instance
```

La AMI determina el punto de partida de nuestra máquina.

También es posible crear AMI propias y utilizarlas como base para múltiples instancias.[6]

---

# 7. Instance Type

El **Instance Type** determina los recursos de hardware virtual asignados a la instancia.

Entre ellos:

- CPU.
- Memoria.
- Capacidad de red.
- Capacidad de almacenamiento disponible según el tipo/configuración.

AWS agrupa los tipos de instancia en familias orientadas a diferentes cargas de trabajo.[3]

```text
General Purpose
      |
      +--> aplicaciones generales

Compute Optimized
      |
      +--> procesamiento intensivo

Memory Optimized
      |
      +--> grandes necesidades de RAM

Storage Optimized
      |
      +--> cargas intensivas de almacenamiento
```

La selección debe basarse en los requerimientos reales de la aplicación.

---

# 8. CPU y memoria

Una decisión básica al crear una instancia es:

> ¿Cuánto procesamiento necesita mi aplicación?

y:

> ¿Cuánta memoria necesita?

Ejemplo conceptual:

```text
Aplicación pequeña
CPU: baja
RAM: baja

Aplicación de procesamiento
CPU: alta
RAM: moderada

Base de datos grande
CPU: moderada
RAM: alta
```

La idea es realizar **capacity planning**, no elegir siempre la máquina más grande.

---

# 9. Region

AWS organiza su infraestructura geográficamente mediante **Regions**.

Una región representa un área geográfica independiente dentro de la infraestructura global de AWS.

Ejemplos:

- `us-east-1`
- `us-west-2`
- `eu-west-1`
- `sa-east-1`

La selección de región puede afectar:

- Latencia.
- Disponibilidad de servicios.
- Requisitos regulatorios.
- Precio.
- Proximidad de los usuarios.

La selección correcta debe responder a requisitos técnicos y de negocio.

---

# 10. Availability Zones

Dentro de una región existen **Availability Zones (AZ)**.

```text
Region
|
+-- AZ-1
|
+-- AZ-2
|
+-- AZ-3
```

Las Availability Zones proporcionan separación física y lógica para reducir el impacto de determinados fallos.

Esto permite construir arquitecturas distribuidas:

```text
              Load Balancer
                    |
          +---------+---------+
          |                   |
       EC2 AZ-A            EC2 AZ-B
```

Para cargas que requieren mayor disponibilidad es habitual distribuir recursos entre varias zonas.

---

# 11. VPC

Una **Virtual Private Cloud (VPC)** es una red virtual dentro de AWS.

Podemos verla como nuestra propia red privada dentro de la nube.

```text
AWS Region
|
+-- VPC
     |
     +-- Subnet A
     |     |
     |     +-- EC2
     |
     +-- Subnet B
           |
           +-- EC2
```

La VPC permite controlar:

- Direccionamiento IP.
- Subredes.
- Routing.
- Conectividad.
- Seguridad de red.

---

# 12. Subnet

Una **subnet** es una subdivisión de una VPC.

Una instancia EC2 se asocia a una subnet.

```text
VPC
|
+-- Public Subnet
|      |
|      +-- EC2 web
|
+-- Private Subnet
       |
       +-- Database
```

Separar recursos públicos y privados es una técnica fundamental de diseño de redes cloud.

---

# 13. Public IP y Private IP

Una instancia puede tener una dirección IP privada dentro de la VPC y, según la configuración, una dirección IP pública.

```text
Internet
   |
Public IP
   |
  EC2
   |
Private IP
   |
VPC
```

La IP privada sirve para la comunicación dentro de la red virtual.

La IP pública permite determinadas comunicaciones desde Internet cuando la ruta y las reglas de seguridad lo permiten.[2]

---

# 14. Elastic IP

Una **Elastic IP** es una dirección IPv4 pública estática que puede asociarse a recursos compatibles.

Es útil cuando se necesita una dirección pública estable, pero no debe utilizarse indiscriminadamente.

En arquitecturas modernas puede ser preferible utilizar DNS, un Load Balancer u otro servicio administrado en lugar de exponer directamente una instancia.

---

# 15. Security Group

Un **Security Group** funciona como un firewall virtual asociado a las instancias.

Permite controlar tráfico mediante reglas.

Ejemplo:

```text
Inbound

TCP 22   --> SSH
TCP 80   --> HTTP
TCP 443  --> HTTPS
```

AWS describe los Security Groups como firewalls virtuales que controlan protocolos, puertos y rangos de IP permitidos.[1]

### Regla fundamental

No debemos pensar:

> "Para que funcione, abro todos los puertos."

La estrategia correcta es:

> **Abrir únicamente los puertos necesarios y restringir el origen cuando sea posible.**

AWS advierte que permitir SSH desde `0.0.0.0/0` habilita conexiones desde cualquier IP y no es recomendable para producción.[4]

---

# 16. Key Pair

Un **Key Pair** permite autenticarse de manera segura en una instancia mediante el mecanismo soportado por la AMI.

En Linux es común utilizar SSH.

Conceptualmente:

```text
AWS
 |
 +--> Public Key
 |
 +--> EC2
```

El estudiante conserva la clave privada.

Ejemplo:

```bash
chmod 400 mi-clave.pem
```

Y posteriormente, cuando corresponda a la AMI utilizada:

```bash
ssh -i mi-clave.pem ubuntu@<IP_PUBLICA>
```

La clave privada no debe publicarse en repositorios ni compartirse.

---

# 17. EBS — Elastic Block Store

**Amazon EBS** proporciona almacenamiento de bloques persistente para EC2.

Podemos compararlo con un disco virtual.

```text
EC2
 |
 +--> Root EBS
 |
 +--> Data EBS
```

El volumen raíz suele contener el sistema operativo. También pueden adjuntarse volúmenes adicionales para datos.[1]

---

# 18. Instance Store

Algunos tipos de EC2 ofrecen **instance store**, un almacenamiento temporal.

AWS indica que los datos del instance store se eliminan cuando la instancia se detiene, hiberna o termina.[1]

Por ello:

```text
Datos importantes
      |
      v
EBS / S3 / Database
```

No se debe depender del instance store para información que necesite persistencia.

---

# 19. User Data

**User Data** permite proporcionar instrucciones para la inicialización de la instancia.

Por ejemplo:

```bash
#!/bin/bash

apt update
apt install -y nginx
systemctl enable nginx
systemctl start nginx
```

Esto permite automatizar parte de la configuración inicial del servidor.

```text
Launch EC2
    |
    v
User Data
    |
    +--> instalar paquetes
    +--> crear archivos
    +--> iniciar servicios
```

AWS documenta User Data como información disponible para la instancia durante el lanzamiento, incluyendo scripts shell para Linux y PowerShell para Windows.[1]

---

# 20. Tags

Los **Tags** agregan metadatos a los recursos.

Ejemplo:

```text
Name = api-laboratorio
Environment = dev
Team = backend
Project = cloud-course
```

Los tags ayudan a:

- Identificar recursos.
- Filtrar recursos.
- Organizar infraestructura.
- Gestionar costos.
- Automatizar operaciones.

---

# 21. Ciclo de vida

Una instancia tiene diferentes estados.

```text
pending
   |
   v
running
   |
   +----> stopping ----> stopped
   |
   +----> shutting-down --> terminated
```

### Stop

Detiene la ejecución de la instancia y mantiene la instancia en un estado recuperable según su configuración.

### Reboot

Reinicia el sistema operativo.

### Terminate

Termina la instancia y elimina los recursos efímeros asociados. Algunos volúmenes pueden conservarse si fueron configurados para ello.

---

# 22. SSH y administración de Linux

Después de crear la instancia podemos administrar el sistema remotamente.

Ejemplos:

```bash
uname -a
```

```bash
df -h
```

```bash
free -h
```

```bash
ps aux
```

```bash
ip addr
```

La práctica debe mostrar que la EC2 es un servidor real sobre el cual podemos instalar, configurar y administrar software.

---

# 23. EC2 y Docker

Una conexión importante con las clases anteriores es:

> **Docker puede ejecutarse dentro de EC2.**

```text
AWS
|
+-- EC2
     |
     +-- Ubuntu
          |
          +-- Docker
               |
               +-- Nginx
               +-- Flask
               +-- Worker
```

Esto permite pasar de:

```text
Docker en mi PC
```

a:

```text
Docker en un servidor AWS
```

---

# 24. EC2 + Docker Compose

También podemos instalar Docker Compose dentro de EC2 y ejecutar la infraestructura que se construyó localmente en las clases anteriores.

```text
EC2
 |
 +-- Docker Compose
       |
       +-- nginx
       +-- api-1
       +-- api-2
```

De esta manera el estudiante conecta tres niveles de conocimiento:

```text
Aplicación
    +
Contenedores
    +
Infraestructura Cloud
```

---

# 25. EC2 + Nginx + Flask

La arquitectura vista anteriormente puede llevarse a AWS:

```text
Internet
   |
   v
 EC2
   |
   v
 Nginx
   |
   +--------+
   |        |
   v        v
Flask-1   Flask-2
```

Posteriormente puede evolucionar hacia:

```text
Internet
   |
   v
AWS Load Balancer
   |
   +--------+--------+
   |        |        |
  EC2      EC2      EC2
   |        |        |
 Flask    Flask    Flask
```

AWS ofrece Elastic Load Balancing para distribuir tráfico entre múltiples instancias.[1]

---

# 26. Auto Scaling

Una sola instancia no siempre es suficiente.

```text
Usuarios = 100
       |
       v
      EC2
```

Si la carga aumenta:

```text
Usuarios = 10.000
       |
       v
      EC2
       X
```

Podemos necesitar varias instancias:

```text
             Load Balancer
                  |
       +----------+----------+
       |          |          |
      EC2        EC2        EC2
```

Amazon EC2 Auto Scaling ayuda a mantener el número adecuado de instancias disponibles para la carga de una aplicación.[1]

---

# 27. EC2 y Elastic Load Balancing

EC2 y ELB se complementan.

```text
              Internet
                  |
                  v
           Load Balancer
          /      |       \
         /       |        \
       EC2      EC2       EC2
```

Esto conecta directamente con el trabajo que realizamos con Nginx y Docker Compose.

La diferencia es que ahora parte de la responsabilidad de balanceo puede ser delegada a un servicio administrado de AWS.

---

# 28. Monitoreo

Crear una EC2 no significa que la infraestructura esté terminada.

Debemos saber:

- ¿Cuánto CPU utiliza?
- ¿Cuánta memoria consume?
- ¿Está activa?
- ¿Está recibiendo tráfico?
- ¿Está fallando?
- ¿Cuánto cuesta?

AWS ofrece **Amazon CloudWatch** para monitorear instancias EC2 y otros recursos.[1]

```text
Application
     |
     v
Infrastructure
     |
     v
Monitoring
     |
     v
Alerts
```

---

# 29. IAM y EC2

**IAM (Identity and Access Management)** controla quién puede realizar operaciones sobre recursos AWS.

Debemos diferenciar:

### Autenticación

¿Quién eres?

### Autorización

¿Qué puedes hacer?

Ejemplo:

```text
Usuario
   |
   v
IAM
   |
   +--> puede crear EC2
   +--> puede detener EC2
   +--> puede consultar EC2
```

Esto es diferente al acceso SSH al sistema operativo.

```text
AWS Account
   |
   +--> IAM
   |
   v
EC2
   |
   +--> Linux user
```

---

# 30. IAM Role para EC2

Una instancia puede utilizar un **IAM Role** para acceder a determinados servicios AWS sin guardar credenciales estáticas dentro del servidor.

Ejemplo conceptual:

```text
EC2
 |
 IAM Role
 |
 +--> leer S3
 +--> enviar métricas
 +--> consultar otros servicios
```

La idea es aplicar el principio de mínimo privilegio y evitar distribuir claves de acceso estáticas innecesariamente.

---

# 31. Red: Route Table e Internet Gateway

Una arquitectura EC2 involucra otros conceptos de red:

```text
Region
  |
  v
Availability Zone
  |
  v
VPC
  |
  v
Subnet
  |
  +--> Route Table
  |
  +--> Internet Gateway
  |
  +--> Security Group
```

### Internet Gateway

Permite la conectividad entre una VPC e Internet cuando las rutas y demás configuraciones lo permiten.

### Route Table

Determina hacia dónde se dirige el tráfico.

Ejemplo conceptual:

```text
Destination       Target

0.0.0.0/0         Internet Gateway
10.0.0.0/16       local
```

---

# 32. Public subnet vs Private subnet

Una arquitectura frecuente separa recursos públicos y privados.

```text
                 Internet
                    |
             Load Balancer
                    |
          +---------+---------+
          | Public Subnet     |
          +-------------------+
                    |
                    v
          +-------------------+
          | Private Subnet    |
          |                   |
          | EC2 / Services    |
          | Database          |
          +-------------------+
```

No todos los servidores deben estar expuestos directamente a Internet.

---

# 33. DNS

Normalmente no queremos que el usuario recuerde una IP como:

```text
54.123.45.67
```

Preferimos:

```text
api.midominio.com
```

DNS permite trabajar con nombres en lugar de depender de IPs directamente.

Una arquitectura puede ser:

```text
Usuario
   |
api.midominio.com
   |
   v
DNS
   |
   v
Load Balancer
   |
   v
EC2
```

---

# 34. EBS frente a S3

Es importante diferenciar almacenamiento de bloques y almacenamiento de objetos.

### EBS

Disco virtual asociado a una instancia.

Útil para:

- Sistema operativo.
- Archivos de aplicación.
- Datos que requieren almacenamiento de bloques.

### S3

Almacenamiento de objetos.

Útil para:

- Imágenes.
- Videos.
- Backups.
- Archivos.
- Logs.
- Objetos.

```text
Servidor / disco
      |
      v
     EBS

Archivos / objetos
      |
      v
      S3
```

---

# 35. Costos

EC2 debe entenderse también desde la perspectiva financiera.

AWS ofrece diferentes modalidades de compra, entre ellas:

- On-Demand.
- Savings Plans.
- Reserved Instances.
- Spot Instances.
- Capacity Reservations.
- Dedicated Hosts.

Las condiciones y precios dependen, entre otros factores, de la región, tipo de instancia, sistema operativo y recursos utilizados.[1]

Durante el laboratorio debe existir una regla:

> **Crear recursos, utilizarlos y revisar su estado y costos al finalizar.**

---

# 36. ¿Cuándo utilizar EC2?

EC2 es particularmente útil cuando necesitamos:

## Control del sistema operativo

Cuando necesitamos instalar y configurar software específico.

## Aplicaciones heredadas

Cuando una aplicación requiere configuraciones de servidor particulares.

## Procesos persistentes

Cuando necesitamos servicios o procesos de larga duración.

## Migraciones

Cuando una aplicación existente corre sobre un servidor tradicional y queremos llevarla a AWS con cambios limitados.

## Laboratorios y aprendizaje

Es una excelente plataforma para aprender:

- Linux.
- Redes.
- Docker.
- Seguridad.
- Servidores.
- Cloud Computing.

---

# 37. ¿Cuándo NO utilizar EC2 directamente?

EC2 no siempre es la mejor opción.

Ejemplos:

| Necesidad | Alternativa que puede ser más adecuada |
|---|---|
| Ejecutar una función ocasional | AWS Lambda |
| Base de datos relacional administrada | Amazon RDS |
| Almacenamiento de objetos | Amazon S3 |
| Contenedores administrados | ECS/EKS u otros servicios administrados |

La pregunta arquitectónica no debería ser:

> "¿Puedo hacerlo con EC2?"

Sino:

> **"¿Qué nivel de control necesito y qué responsabilidades quiero administrar?"**

---

# 38. EC2 frente a servicios administrados

Podemos visualizar la diferencia así:

```text
Más control
    ^
    |
   EC2
    |
    |
 Servicios administrados
    |
    v
Menos administración de infraestructura
```

Con EC2 podemos controlar:

- Sistema operativo.
- Paquetes.
- Servicios.
- Docker.
- Configuración.

Pero también debemos ocuparnos de:

- Parches.
- Hardening.
- Monitoreo.
- Actualizaciones.
- Capacidad.
- Sistema operativo.

---

# 39. Relación con J2C — Journey to Cloud

En esta clase utilizaremos **J2C** en el sentido de **Journey to Cloud**: el proceso de transformación y migración de aplicaciones e infraestructura hacia la nube.

El término aparece en la industria para describir estrategias de migración y transformación hacia cloud.[5]

La relación con EC2 es directa: EC2 puede ser uno de los primeros destinos cuando una aplicación tradicional se está migrando desde infraestructura física o virtual hacia AWS.

---

# 40. J2C y estrategias de migración

Un proceso simplificado puede verse así:

```text
On-Premise
    |
    v
Assessment
    |
    v
Migration
    |
    v
EC2
    |
    v
Modernization
    |
    v
Cloud Native
```

Por ejemplo, una aplicación tradicional:

```text
Servidor físico
|
+-- Ubuntu
+-- Nginx
+-- Flask
+-- PostgreSQL
```

podría migrarse inicialmente como:

```text
AWS
|
+-- EC2
     |
     +-- Ubuntu
     +-- Nginx
     +-- Flask
     +-- PostgreSQL
```

El objetivo de esta primera etapa podría ser migrar con modificaciones limitadas.

---

# 41. Lift and Shift

Una estrategia común es **Lift and Shift**.

La idea es:

> Llevar una aplicación a la nube modificándola lo menos posible.

```text
ANTES
Servidor físico
      |
      +-- Aplicación

DESPUÉS
EC2
      |
      +-- Aplicación
```

Esta estrategia puede acelerar una migración, aunque no necesariamente aprovecha todas las capacidades cloud.

---

# 42. Modernización posterior

Después de migrar, la arquitectura puede evolucionar:

```text
EC2 monolítico
      |
      v
Contenedores
      |
      v
Servicios administrados
      |
      v
Arquitectura cloud-native
```

Por ejemplo:

```text
EC2
 |
 +-- Flask
 +-- PostgreSQL
```

puede evolucionar hacia:

```text
Load Balancer
      |
      v
Containers / ECS
      |
      v
RDS PostgreSQL
      |
      v
S3
```

EC2 puede ser entonces un paso intermedio dentro de una estrategia más amplia de J2C.

---

# 43. Ejemplo integrador

Supongamos que una universidad tiene una aplicación académica funcionando en un servidor local.

### Arquitectura actual

```text
Servidor
|
+-- Ubuntu
+-- Nginx
+-- Flask
+-- PostgreSQL
```

### Primera etapa de migración

```text
AWS
|
+-- EC2
     |
     +-- Ubuntu
     +-- Docker
          |
          +-- Nginx
          +-- Flask
          +-- PostgreSQL
```

### Segunda etapa

```text
AWS
|
+-- Load Balancer
|
+-- EC2 / Containers
|
+-- RDS PostgreSQL
|
+-- S3
```

Este ejemplo permite discutir cómo una migración puede evolucionar desde una estrategia cercana al entorno original hacia una arquitectura más administrada.

---

# 44. Objetivos de la práctica

Al finalizar la práctica, el estudiante debería poder:

```text
1. Ingresar a AWS
      |
2. Seleccionar una región
      |
3. Seleccionar una AMI
      |
4. Seleccionar un Instance Type
      |
5. Configurar Key Pair
      |
6. Configurar VPC/Subnet
      |
7. Configurar Security Group
      |
8. Configurar almacenamiento
      |
9. Crear la instancia
      |
10. Conectarse por SSH
      |
11. Verificar Linux
      |
12. Instalar Docker
      |
13. Ejecutar una aplicación
```

---

# 45. Checklist de creación de EC2

Antes de crear:

- [ ] Región seleccionada.
- [ ] AMI seleccionada.
- [ ] Instance Type seleccionado.
- [ ] Key Pair configurado.
- [ ] VPC seleccionada.
- [ ] Subnet seleccionada.
- [ ] Security Group configurado.
- [ ] Puertos revisados.
- [ ] Disco EBS revisado.
- [ ] Tags definidos.

Después de crear:

- [ ] Instancia en estado `running`.
- [ ] IP pública identificada, si aplica.
- [ ] Conectividad comprobada.
- [ ] SSH probado.
- [ ] Sistema operativo verificado.
- [ ] Costos y recursos revisados.

---

# 46. Preguntas de análisis

1. Si tengo una API Flask que funciona en Docker Compose en mi computador, ¿qué componentes necesito para ejecutarla dentro de EC2?
2. ¿Por qué necesito un Security Group?
3. ¿Qué diferencia existe entre IP privada, IP pública y DNS?
4. ¿Qué función cumple una AMI?
5. ¿Por qué no debería guardar información crítica únicamente en almacenamiento temporal?
6. ¿Qué ventajas ofrece ejecutar Docker dentro de EC2?
7. ¿Qué diferencia existe entre detener y terminar una instancia?
8. ¿Cuándo sería mejor utilizar RDS que PostgreSQL instalado directamente en EC2?
9. ¿Qué cambiaría si la aplicación necesitara tres instancias EC2?
10. ¿Cómo encaja EC2 dentro de un proceso J2C?

---

# 47. Pregunta final de arquitectura

Considere:

```text
                    Internet
                       |
                       v
                  EC2 Instance
                       |
                    Docker
                       |
              +--------+--------+
              |                 |
           Nginx             Flask
                                |
                                v
                           PostgreSQL
```

Analice:

1. ¿Cuál es el punto único de falla?
2. ¿Qué ocurriría si la EC2 se detiene?
3. ¿Qué componente debería escalar primero?
4. ¿Dónde debería almacenarse la información persistente?
5. ¿Sería conveniente poner PostgreSQL dentro de la misma EC2?
6. ¿Qué cambiaría al utilizar RDS?
7. ¿Cómo evolucionaría esta arquitectura hacia múltiples EC2?

---

# 48. Resumen

EC2 es un servicio de computación que permite crear servidores virtuales en AWS.

Para comprender una instancia EC2 debemos conocer:

```text
AMI
Instance Type
Region
Availability Zone
VPC
Subnet
Security Group
Key Pair
EBS
Public / Private IP
User Data
Tags
IAM
Monitoring
Auto Scaling
Load Balancing
```

La idea más importante de la clase es:

> **EC2 no es simplemente una máquina virtual; es un componente de una arquitectura de infraestructura cloud.**

Aprender EC2 significa comprender cómo se relacionan:

```text
Compute
+
Networking
+
Security
+
Storage
+
Identity
+
Monitoring
+
Scalability
```

---

# Referencias

[1] AWS — **What is Amazon EC2?**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html

[2] AWS — **Tutorial: Launch a test EC2 instance and connect to it**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/tutorial-launch-a-test-ec2-instance.html

[3] AWS — **Amazon EC2 instance types**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html

[4] AWS — **Get started with Amazon EC2**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html

[5] Accenture — **Journey to Cloud (J2C) approach**  
https://s3-us-west-2.amazonaws.com/naspovaluepoint/1651951499_AR3086%20Accenture%20Master%20Agreement%20Executed.pdf

[6] AWS — **AMI types and characteristics in Amazon EC2**  
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ComponentsAMIs.html
