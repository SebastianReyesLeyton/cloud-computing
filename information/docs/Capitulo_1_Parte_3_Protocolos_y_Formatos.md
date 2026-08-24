# Capítulo 1 -- Parte 3

# Protocolos de Comunicación y Formatos de Intercambio

## Objetivos de aprendizaje

Al finalizar esta sección podrá:

-   Comprender cómo viajan los datos por una red.
-   Diferenciar TCP y UDP.
-   Explicar HTTP, HTTPS, HTTP/2 y HTTP/3.
-   Identificar el papel de DNS, IP y puertos.
-   Comparar JSON, XML, Protocol Buffers y MessagePack.

------------------------------------------------------------------------

# Introducción

Cuando una aplicación solicita información a otra, los datos atraviesan
múltiples dispositivos y protocolos antes de llegar a su destino. Estos
protocolos establecen las reglas para que la comunicación sea confiable
y comprensible.

------------------------------------------------------------------------

# Direcciones IP

Una dirección IP identifica un dispositivo dentro de una red.

Ejemplos:

-   IPv4: `192.168.1.10`
-   IPv6: `2001:db8::1`

Las aplicaciones no se comunican con nombres como `api.ejemplo.com`;
primero necesitan conocer su dirección IP.

------------------------------------------------------------------------

# DNS

El **Domain Name System (DNS)** traduce nombres de dominio en
direcciones IP.

``` text
Cliente
   |
www.universidad.edu
   |
Servidor DNS
   |
200.10.20.30
   |
Servidor Web
```

------------------------------------------------------------------------

# Puertos

Un puerto identifica un servicio dentro de un mismo equipo.

    Puerto Servicio
  -------- ----------
        80 HTTP
       443 HTTPS
        22 SSH
        25 SMTP
      3306 MySQL

------------------------------------------------------------------------

# TCP

TCP (**Transmission Control Protocol**) garantiza que los datos lleguen
completos y en orden.

Características:

-   Orientado a conexión.
-   Confiable.
-   Retransmisión de paquetes.
-   Control de errores.

Ideal para:

-   APIs REST.
-   GraphQL.
-   SOAP.
-   gRPC.
-   Transferencia de archivos.

------------------------------------------------------------------------

# UDP

UDP (**User Datagram Protocol**) prioriza la velocidad sobre la
confiabilidad.

Características:

-   Sin conexión.
-   Menor latencia.
-   No garantiza entrega.

Ideal para:

-   Video en tiempo real.
-   Audio.
-   Juegos en línea.
-   Streaming.

------------------------------------------------------------------------

# Comparativa TCP vs UDP

  Característica        TCP        UDP
  --------------------- ---------- -----
  Confiable             ✔          ✘
  Rápido                Medio      ✔
  Reenvío de paquetes   ✔          ✘
  Orden garantizado     ✔          ✘
  Tiempo real           Limitado   ✔

------------------------------------------------------------------------

# HTTP

HTTP es el protocolo más utilizado para intercambiar información entre
clientes y servidores web.

Ejemplo:

``` http
GET /students HTTP/1.1
Host: api.universidad.edu
```

------------------------------------------------------------------------

# HTTPS

HTTPS añade una capa de seguridad mediante **TLS/SSL**, cifrando la
información para protegerla frente a terceros.

Se utiliza para:

-   Bancos.
-   Comercio electrónico.
-   Plataformas educativas.
-   Redes sociales.

------------------------------------------------------------------------

# HTTP/2

Principales mejoras:

-   Multiplexación.
-   Compresión de cabeceras.
-   Mejor rendimiento.

Permite múltiples solicitudes simultáneas sobre una misma conexión.

------------------------------------------------------------------------

# HTTP/3

HTTP/3 utiliza **[QUIC](https://www.f5.com/glossary/quic-http3)**, construido sobre UDP.

Ventajas:

-   Menor latencia.
-   Reconexión más rápida.
-   Mejor desempeño en redes móviles.

------------------------------------------------------------------------

# Formatos de intercambio

## JSON

Formato ligero y legible.

``` json
{
  "id": 10,
  "name": "Ana"
}
```

Ventajas:

-   Fácil de leer.
-   Muy utilizado.
-   Compatible con casi todos los lenguajes.

------------------------------------------------------------------------

## XML

Formato estructurado basado en etiquetas.

``` xml
<student>
  <id>10</id>
  <name>Ana</name>
</student>
```

Común en SOAP y sistemas empresariales.

------------------------------------------------------------------------

## Protocol Buffers

Formato binario desarrollado por Google.

Ventajas:

-   Muy compacto.
-   Muy rápido.
-   Utilizado por gRPC.

------------------------------------------------------------------------

## MessagePack

Formato binario compatible conceptualmente con JSON, optimizado para
tamaño y velocidad.

------------------------------------------------------------------------

# Comparativa de formatos

  Formato            Legible          Tamaño   Velocidad
  ------------------ --------- ------------- -----------
  JSON               ✔                 Medio        Alta
  XML                ✔                Grande       Media
  Protocol Buffers   ✘           Muy pequeño    Muy alta
  MessagePack        Parcial         Pequeño        Alta

------------------------------------------------------------------------

# Ejemplo en Python

``` python
import json

student = {
    "id": 1,
    "name": "Juan",
    "program": "Ingeniería"
}

texto = json.dumps(student)
print(texto)
```

------------------------------------------------------------------------

# Buenas prácticas

-   Utilizar HTTPS en producción.
-   Elegir JSON cuando la interoperabilidad sea prioritaria.
-   Utilizar Protocol Buffers cuando el rendimiento sea crítico.
-   Documentar claramente el formato de intercambio.

------------------------------------------------------------------------

# Errores comunes

-   Enviar datos sensibles por HTTP.
-   Confundir IP con dominio.
-   Usar UDP cuando se requiere confiabilidad.
-   Elegir XML sin una necesidad real.

------------------------------------------------------------------------

# Resumen

Los protocolos definen cómo se comunican las aplicaciones, mientras que
los formatos determinan cómo se representan los datos. Elegir
correctamente ambos aspectos es clave para construir sistemas
eficientes, seguros y escalables.

------------------------------------------------------------------------

# Taller práctico

1.  Compare TCP y UDP para una plataforma de videoconferencias.
2.  Investigue el puerto utilizado por PostgreSQL.
3.  Convierta un objeto Python a JSON utilizando el módulo `json`.
4.  Explique en qué casos elegiría Protocol Buffers sobre JSON.
