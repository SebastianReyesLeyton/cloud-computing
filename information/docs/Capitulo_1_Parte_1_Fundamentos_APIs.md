# Capítulo 1 -- Parte 1

# Fundamentos de las APIs y la Comunicación entre Aplicaciones

## Objetivos de aprendizaje

Al finalizar esta sección el estudiante será capaz de:

-   Comprender por qué existen las APIs.
-   Explicar el problema que resuelven.
-   Identificar la evolución histórica de las APIs.
-   Relacionar las APIs con situaciones cotidianas.
-   Reconocer la importancia de las APIs en el desarrollo de software
    moderno.

------------------------------------------------------------------------

# Motivación

Las aplicaciones modernas rara vez funcionan de manera aislada. Un
sistema de comercio electrónico consulta servicios de pago, una
aplicación móvil obtiene información desde un servidor y una plataforma
de videoconferencias integra autenticación, almacenamiento y mensajería.

Las APIs hacen posible esta comunicación entre aplicaciones, ocultando
la complejidad interna de cada sistema y ofreciendo una interfaz
estandarizada para intercambiar información.

------------------------------------------------------------------------

# ¿Por qué existen las APIs?

Antes de las APIs, integrar sistemas era complejo y costoso. Cada
aplicación debía conocer detalles internos de las demás, generando un
fuerte acoplamiento y dificultando el mantenimiento.

Una API define un contrato de comunicación: especifica qué operaciones
están disponibles, cómo solicitarlas y qué respuestas esperar. Gracias a
ello, los sistemas pueden evolucionar de forma independiente.

------------------------------------------------------------------------

# Analogía cotidiana

Imagine un restaurante.

-   El cliente representa la aplicación consumidora.
-   El mesero representa la API.
-   La cocina representa el sistema interno.

El cliente no entra a la cocina ni conoce cómo se prepara la comida.
Solo realiza un pedido mediante el mesero y recibe una respuesta. Del
mismo modo, una aplicación interactúa con otra a través de una API.

------------------------------------------------------------------------

# Evolución de las APIs

## RPC

Las primeras soluciones buscaban ejecutar procedimientos remotos como si
fueran funciones locales.

## SOAP

Introdujo contratos formales mediante XML y WSDL, ampliamente utilizado
en sistemas empresariales.

## REST

Popularizó el uso de HTTP y recursos, simplificando la construcción de
servicios web.

## GraphQL

Permitió que el cliente solicitara únicamente los datos necesarios.

## WebSockets

Incorporó comunicación bidireccional y en tiempo real.

## gRPC

Optimizó la comunicación entre servicios mediante HTTP/2 y Protocol
Buffers.

## WebRTC

Facilitó la comunicación multimedia directa entre navegadores y
dispositivos.

------------------------------------------------------------------------

# APIs en la actualidad

Las APIs son la base de:

-   Aplicaciones móviles.
-   Plataformas web.
-   Microservicios.
-   Computación en la nube.
-   Internet de las Cosas.
-   Inteligencia Artificial.
-   Integraciones entre organizaciones.

------------------------------------------------------------------------

# Resumen

Una API es un contrato de comunicación entre aplicaciones. Su objetivo
es desacoplar sistemas, facilitar la integración y permitir que
distintas tecnologías colaboren de forma segura y eficiente.

------------------------------------------------------------------------

# Preguntas de reflexión

1.  ¿Qué problemas existirían si las APIs no existieran?
2.  ¿Por qué es importante desacoplar los sistemas?
3.  ¿Qué ventajas ofrece una API frente al acceso directo a una base de
    datos?
4.  ¿En qué aplicaciones utiliza APIs diariamente sin darse cuenta?

------------------------------------------------------------------------

# Taller práctico

1.  Identifique cinco aplicaciones que utilicen APIs.
2.  Dibuje un diagrama simple de comunicación entre un cliente y un
    servidor.
3.  Investigue una API pública y describa qué servicios ofrece.
