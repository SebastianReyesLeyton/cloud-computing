# Workshop 1 — Crear y conectar una instancia EC2

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- Amazon EC2
- AMI
- Instance Type
- Region y Availability Zone
- Key Pair
- Security Group
- VPC y Subnet
- IP pública
- SSH

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Crear una instancia EC2 desde la consola de AWS.
- Identificar los componentes principales de una instancia.
- Seleccionar una AMI y un tipo de instancia.
- Configurar un Key Pair.
- Configurar un Security Group básico.
- Identificar la IP pública de una instancia.
- Conectarse a una instancia Linux mediante SSH.

---

# Conceptos que se trabajarán

- EC2 como IaaS.
- AMI.
- Instance Type.
- Region y Availability Zone.
- VPC y Subnet.
- Security Group.
- Key Pair.
- SSH.
- Public IP y Private IP.

---

# Parte 1. Preparación

Antes de crear la instancia, responder:

1. ¿Qué diferencia existe entre una AMI y un Instance Type?
2. ¿Qué función cumple el Security Group?
3. ¿Por qué SSH requiere una autenticación segura?
4. ¿Por qué debemos seleccionar cuidadosamente la región?

---

# Parte 2. Crear la instancia

Desde AWS Console:

1. Ingrese a EC2.
2. Seleccione una región.
3. Seleccione **Launch Instance**.
4. Asigne un nombre, por ejemplo:

```text
curso-ec2-<nombre>
```

5. Seleccione una AMI Linux, por ejemplo Ubuntu.
6. Seleccione un Instance Type pequeño apropiado para laboratorio.
7. Cree o seleccione un Key Pair.
8. Configure un Security Group que permita SSH desde una fuente restringida cuando sea posible.
9. Revise el almacenamiento.
10. Lance la instancia.

---

# Parte 3. Verificar el estado

La instancia debe llegar al estado:

```text
running
```

Identifique:

- Instance ID.
- Public IPv4 address.
- Private IPv4 address.
- Availability Zone.
- AMI.
- Instance Type.
- Security Group.

---

# Parte 4. Conectarse por SSH

En Linux/macOS:

```bash
chmod 400 mi-clave.pem
```

Después:

```bash
ssh -i mi-clave.pem ubuntu@<IP_PUBLICA>
```

El nombre del usuario depende de la AMI seleccionada.

---

# Parte 5. Verificar el sistema

Una vez conectado:

```bash
whoami
hostname
uname -a
df -h
free -h
ip addr
```

---

# Parte 6. Actividad de análisis

Construya una tabla con:

| Elemento | Valor | ¿Para qué sirve? |
|----------|-------|------------------|
| AMI | | |
| Instance Type | | |
| Region | | |
| Availability Zone | | |
| VPC | | |
| Subnet | | |
| Public IP | | |
| Private IP | | |
| Security Group | | |
| Key Pair | | |

---

# Reto

Explique con sus propias palabras qué sucede desde que presiona **Launch Instance** hasta que obtiene acceso por SSH.

---

# Entregable

- Captura de la instancia en estado `running`.
- Captura de la conexión SSH.
- Tabla de componentes.
- Respuestas de análisis.
