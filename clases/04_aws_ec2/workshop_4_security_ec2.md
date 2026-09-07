# Workshop 4 — Seguridad de red en EC2

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- Security Groups
- Puertos
- SSH
- HTTP
- HTTPS
- Principio de mínimo privilegio
- Network exposure

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Comprender cómo un Security Group controla el tráfico.
- Diferenciar puertos públicos y privados.
- Aplicar el principio de mínimo privilegio.
- Identificar configuraciones inseguras.
- Reducir la superficie de exposición de una EC2.

---

# Situación inicial

Suponga que la instancia tiene:

```text
22   SSH
80   HTTP
5000 Flask
5432 PostgreSQL
```

La pregunta es:

> ¿Deben estar los cuatro puertos públicos?

---

# Parte 1. Auditar la configuración

Liste los puertos que escucha el servidor:

```bash
sudo ss -tulpn
```

Identifique qué procesos están asociados a cada puerto.

---

# Parte 2. Analizar el Security Group

| Puerto | Servicio | ¿Necesita ser público? | ¿Por qué? |
|--------|----------|------------------------|-----------|
| 22 | SSH | | |
| 80 | HTTP | | |
| 5000 | Flask | | |
| 5432 | PostgreSQL | | |

---

# Parte 3. Principio de mínimo privilegio

Proponga una configuración donde:

```text
Internet
   |
   +--> 80/443 --> aplicación
   |
Administración
   |
   +--> 22 --> solo IP autorizada
```

Y la base de datos no sea accesible desde Internet:

```text
Internet -X-> 5432
```

---

# Parte 4. Análisis

Responda:

1. ¿Qué riesgo existe al abrir SSH al mundo?
2. ¿Qué diferencia existe entre Security Group y firewall dentro del sistema operativo?
3. ¿Por qué no debemos exponer PostgreSQL públicamente si no es necesario?
4. ¿Por qué Nginx puede ser el único punto público de entrada?

---

# Reto

Documente una propuesta de Security Group para una API Flask detrás de Nginx.

Debe incluir:

- Regla.
- Puerto.
- Protocolo.
- Origen.
- Justificación.

---

# Entregable

- Tabla de puertos.
- Capturas de Security Group.
- Propuesta de configuración segura.
- Respuestas de análisis.
