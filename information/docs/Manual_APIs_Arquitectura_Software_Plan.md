# Manual de APIs para Arquitectura de Software

## Propósito

Este manual está diseñado como material de apoyo para un curso
universitario de Arquitectura de Software, Computación en la Nube o
Desarrollo de APIs. Su objetivo es enseñar no solo qué es cada
tecnología, sino también **cuándo utilizarla y cómo justificar su
elección** dentro del diseño de un sistema.

## Estructura del manual

### Capítulo 1. Comunicación entre aplicaciones

**Objetivo:** Comprender por qué existen las APIs y cómo evolucionaron.

**Contenido:** - ¿Qué es una API? - Cliente y servidor. - Comunicación
síncrona vs asíncrona. - Request y Response. - Protocolos de
comunicación. - HTTP, HTTP/2 y HTTP/3. - JSON, XML y Protocol Buffers. -
¿Qué es un endpoint? - Anatomía de una petición HTTP. - Diagramas del
flujo de comunicación.

------------------------------------------------------------------------

### Capítulo 2. REST y RESTful

-   Historia de REST.
-   ¿Qué significa REST?
-   REST vs RESTful.
-   Principios REST.
-   Recursos.
-   URIs.
-   Verbos HTTP.
-   Códigos de respuesta.
-   Versionamiento.
-   Buenas prácticas.
-   Ventajas y desventajas.
-   Casos de uso.
-   Ejemplo completo en Python.

------------------------------------------------------------------------

### Capítulo 3. GraphQL

-   Motivación de GraphQL.
-   Arquitectura.
-   Queries.
-   Mutations.
-   Subscriptions.
-   Schema.
-   Tipos.
-   Ventajas y desventajas.
-   Casos de uso.
-   Ejemplo en Python.

------------------------------------------------------------------------

### Capítulo 4. WebSockets

-   Comunicación Full Duplex.
-   Handshake.
-   Ciclo de vida de la conexión.
-   Broadcast.
-   Escalabilidad.
-   Casos de uso.
-   Chat y dashboards en tiempo real.
-   Ejemplo en Python.

------------------------------------------------------------------------

### Capítulo 5. WebRTC (RTC APIs)

-   ¿Qué es WebRTC?
-   Peer-to-Peer.
-   ICE.
-   STUN.
-   TURN.
-   Señalización.
-   Audio y video.
-   Compartición de pantalla.
-   Casos de uso.
-   Ejemplo con Python.

------------------------------------------------------------------------

### Capítulo 6. gRPC

-   RPC.
-   Protocol Buffers.
-   HTTP/2.
-   Unary.
-   Client Streaming.
-   Server Streaming.
-   Bidirectional Streaming.
-   Rendimiento.
-   Casos de uso.
-   Ejemplo completo en Python.

------------------------------------------------------------------------

### Capítulo 7. SOAP

-   Historia.
-   XML.
-   Envelope.
-   Header.
-   Body.
-   WSDL.
-   WS-Security.
-   Ventajas y desventajas.
-   Casos de uso.
-   Ejemplo en Python.

------------------------------------------------------------------------

### Capítulo 8. Comparativa completa

Comparación entre:

-   REST
-   RESTful
-   GraphQL
-   SOAP
-   WebSocket
-   WebRTC
-   gRPC

Incluye tablas comparativas sobre: - Transporte. - Formato de datos. -
Tiempo real. - Streaming. - Rendimiento. - Complejidad. - Seguridad. -
Escalabilidad. - Compatibilidad con IoT. - Uso en microservicios. -
Aplicaciones móviles.

------------------------------------------------------------------------

### Capítulo 9. ¿Cuál API debo usar?

Guía práctica basada en escenarios reales:

-   Inventarios.
-   Comercio electrónico.
-   Chats.
-   Videollamadas.
-   Sistemas bancarios.
-   Microservicios.
-   Aplicaciones móviles.
-   Plataformas IoT.

Incluye diagramas de decisión y criterios técnicos.

------------------------------------------------------------------------

### Capítulo 10. Caso de estudio: SmartCampus

Aplicación práctica del conocimiento adquirido.

Los estudiantes deberán seleccionar y justificar el uso de:

-   REST.
-   GraphQL.
-   WebSocket.
-   WebRTC.
-   gRPC.
-   SOAP.

para los diferentes módulos de una plataforma universitaria inteligente.

------------------------------------------------------------------------

## Características del manual

Cada capítulo incluirá:

-   Explicaciones detalladas.
-   Diagramas ASCII.
-   Diagramas de secuencia.
-   Diagramas de arquitectura.
-   Ejemplos completos en Python.
-   Buenas prácticas.
-   Errores comunes.
-   Casos de uso reales.
-   Preguntas de reflexión.
-   Ejercicios.
-   Mini retos.

## Resultado esperado

Al finalizar el manual, el estudiante será capaz de:

-   Comprender los principales estilos de APIs.
-   Seleccionar la tecnología más adecuada para un problema específico.
-   Diseñar arquitecturas de software justificadas técnicamente.
-   Implementar ejemplos funcionales en Python.
-   Analizar ventajas y desventajas de cada alternativa.
-   Aplicar estos conocimientos al diseño de sistemas modernos para la
    nube y el Internet de las Cosas.
