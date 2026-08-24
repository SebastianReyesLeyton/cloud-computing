# Caso de Estudio: Plataforma Inteligente de Gestión de Campus Universitario

## Contexto

La Universidad **SmartCampus** desea modernizar su infraestructura
tecnológica mediante el desarrollo de una plataforma centralizada que
permita integrar múltiples servicios académicos y administrativos. El
objetivo es ofrecer una solución escalable, mantenible y preparada para
operar tanto en la nube como con dispositivos del Internet de las Cosas
(IoT).

Actualmente, la universidad enfrenta varios problemas:

-   Cada facultad utiliza sistemas independientes que no se comunican
    entre sí.
-   El registro de asistencia se realiza manualmente.
-   Los salones inteligentes generan información que no es aprovechada.
-   Los estudiantes deben acceder a diferentes aplicaciones para
    consultar información académica.
-   No existe una plataforma central para enviar notificaciones en
    tiempo real.
-   La administración desea obtener indicadores sobre el uso de los
    espacios físicos y el consumo energético.

La nueva plataforma deberá integrar estos servicios en un único
ecosistema.

## Objetivos del sistema

La plataforma debe permitir:

-   Gestionar estudiantes, docentes y cursos.
-   Registrar asistencia automáticamente mediante dispositivos IoT.
-   Reservar aulas y laboratorios.
-   Administrar horarios.
-   Enviar notificaciones a estudiantes y docentes.
-   Monitorear sensores de temperatura, iluminación y ocupación.
-   Generar reportes estadísticos.
-   Integrarse con aplicaciones móviles y web.

## Actividades

### Actividad 1. Comprensión del problema

1.  ¿Quiénes son los actores del sistema?
2.  ¿Qué funcionalidades ofrece la plataforma?
3.  ¿Qué información debe almacenarse?
4.  ¿Qué procesos ocurren en tiempo real?
5.  ¿Qué módulos considera independientes?

### Actividad 2. Identificación de componentes

Proponga los componentes principales de la arquitectura y explique la
responsabilidad de cada uno.

### Actividad 3. Diseño de la arquitectura

Construya un diagrama de arquitectura de alto nivel indicando clientes,
APIs, servicios, bases de datos, servicios externos y dispositivos IoT.

### Actividad 4. Selección del tipo de API

Seleccione y justifique el uso de REST, GraphQL, WebSocket, gRPC o MQTT
para distintos escenarios del sistema.

### Actividad 5. Diseño de bases de datos

Clasifique qué información almacenaría en una base de datos relacional y
cuál en una base de datos NoSQL.

### Actividad 6. Integración mediante eventos

Identifique qué procesos deberían ejecutarse mediante mensajería
asíncrona y justifique el uso de una cola de mensajes.

### Actividad 7. Escalabilidad

Explique cómo escalaría la solución para soportar un incremento
significativo de usuarios.

### Actividad 8. Diseño de APIs

Diseñe endpoints REST, consultas GraphQL, tópicos MQTT, servicios gRPC o
canales WebSocket para dos módulos del sistema.

### Actividad 9. Patrones arquitectónicos

Indique dónde aplicaría los siguientes patrones:

-   API Gateway
-   Microservicios
-   Event-Driven Architecture
-   CQRS
-   Publish/Subscribe
-   Cache Aside
-   Circuit Breaker
-   Load Balancer

## Producto esperado

1.  Diagrama de arquitectura.
2.  Componentes identificados.
3.  Selección y justificación de APIs.
4.  Diseño de bases de datos.
5.  Diseño de interfaces.
6.  Estrategia de integración.
7.  Estrategia de escalabilidad.
8.  Justificación técnica de las decisiones.

## Objetivos de aprendizaje

-   Analizar requerimientos funcionales y no funcionales.
-   Diseñar arquitecturas de software.
-   Seleccionar el tipo de API adecuado.
-   Diferenciar comunicación síncrona y asíncrona.
-   Diseñar soluciones escalables para Cloud e IoT.
