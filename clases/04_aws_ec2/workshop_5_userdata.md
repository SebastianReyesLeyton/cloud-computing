# Workshop 5 — Automatización de EC2 con User Data

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- User Data
- Bash
- Automatización
- Provisionamiento
- Reproducibilidad
- EC2

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Comprender el propósito de User Data.
- Automatizar tareas de inicialización.
- Instalar software automáticamente.
- Crear un servidor reproducible.
- Comparar configuración manual frente a configuración automatizada.

---

# Parte 1. Configuración manual

En una EC2 nueva, realice manualmente:

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl status nginx
```

---

# Parte 2. Convertir la configuración en script

Cree un script como User Data:

```bash
#!/bin/bash

apt update
apt install -y nginx

systemctl enable nginx
systemctl start nginx

cat > /var/www/html/index.html <<'EOFHTML'
<h1>Servidor provisionado automáticamente</h1>
<p>EC2 + User Data</p>
EOFHTML
```

---

# Parte 3. Lanzar otra EC2

Cree una nueva instancia y coloque el script anterior en **User Data** durante el lanzamiento.

```text
Launch EC2
     |
     v
User Data
     |
     +--> apt update
     +--> install nginx
     +--> create website
     |
     v
Servidor listo
```

---

# Parte 4. Verificación

Desde el computador acceda a:

```text
http://<IP_PUBLICA>
```

---

# Parte 5. Análisis

| Enfoque | Configuración manual | User Data |
|---------|-----------------------|-----------|
| Repetibilidad | | |
| Tiempo | | |
| Errores humanos | | |
| Documentación | | |
| Automatización | | |

---

# Reto

Modifique el script para instalar Docker y dejar un contenedor ejecutándose automáticamente.

Resultado esperado:

```text
Nueva EC2
   |
User Data
   |
Docker
   |
Container
   |
Aplicación
```

---

# Entregable

- Script User Data.
- Captura del servidor funcionando.
- Comparación manual vs automatización.
- Explicación de qué tareas convendría automatizar en una arquitectura real.
