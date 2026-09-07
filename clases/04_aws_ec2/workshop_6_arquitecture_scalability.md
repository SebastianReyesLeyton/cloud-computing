# Workshop 6 — EC2 en una arquitectura escalable

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- EC2
- Load Balancer
- Auto Scaling
- Availability Zones
- Stateless applications
- Escalabilidad horizontal
- Alta disponibilidad
- Arquitectura cloud

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Identificar las limitaciones de una única EC2.
- Diseñar una arquitectura con múltiples instancias.
- Comprender el propósito de un Load Balancer.
- Relacionar EC2 con Auto Scaling.
- Analizar alta disponibilidad y escalabilidad.
- Identificar problemas de estado en aplicaciones distribuidas.

---

# Parte 1. Arquitectura inicial

```text
Internet
   |
   v
EC2
 |
 +-- Nginx
 |
 +-- Flask
```

Analice:

- ¿Qué sucede si EC2 falla?
- ¿Qué sucede si aumenta el tráfico?
- ¿Dónde existe un Single Point of Failure?

---

# Parte 2. Evolución

Diseñe:

```text
                 Load Balancer
                /      |      \
               /       |       \
             EC2      EC2      EC2
              |        |        |
            Flask    Flask    Flask
```

Identifique:

- ¿Qué componente distribuye el tráfico?
- ¿Dónde se ejecuta cada instancia?
- ¿Por qué conviene distribuir instancias entre Availability Zones?

---

# Parte 3. Stateless

Explique qué ocurre si una petición llega a EC2-1 y la siguiente llega a EC2-2:

```text
Request 1 -> EC2-1
Request 2 -> EC2-2
```

¿Qué pasa si la sesión está guardada únicamente en memoria de EC2-1?

---

# Parte 4. Persistencia

Analice una arquitectura donde cada instancia mantiene archivos locales:

```text
EC2-1 -> archivo.txt
EC2-2 -> archivo.txt
```

¿Son necesariamente el mismo archivo?

Proponga una estrategia de almacenamiento compartido o administrado.

---

# Parte 5. Auto Scaling

Investigue y documente:

- ¿Qué problema resuelve Auto Scaling?
- ¿Qué diferencia existe entre escalar verticalmente y horizontalmente?
- ¿Qué métrica podría utilizarse como señal de escalamiento?

Ejemplos:

```text
CPU > umbral
```

o:

```text
Número de solicitudes > umbral
```

---

# Parte 6. Diseño

Construya una propuesta:

```text
                 Internet
                    |
                    v
              Load Balancer
                    |
          +---------+---------+
          |         |         |
         EC2       EC2       EC2
          |         |         |
       Flask     Flask     Flask
          \         |         /
           \        |        /
             Base de datos
```

Indique qué elementos deberían ser:

- Públicos.
- Privados.
- Escalables.
- Persistentes.
- Monitoreados.

---

# Reto final

Explique cómo evolucionaría el laboratorio:

```text
Docker local
    |
    v
Docker en EC2
    |
    v
Múltiples EC2
    |
    v
Load Balancer
    |
    v
Auto Scaling
```

Relacione esta evolución con **Journey to Cloud (J2C)**.

---

# Entregable

- Diagrama de arquitectura.
- Respuestas de análisis.
- Comparación entre arquitectura de una instancia y múltiples instancias.
- Explicación de escalabilidad horizontal.
- Propuesta de evolución hacia una arquitectura más administrada.
