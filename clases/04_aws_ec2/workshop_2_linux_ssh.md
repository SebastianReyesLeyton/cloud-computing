# Workshop 2 — Administración de Linux en EC2

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas:**

- Linux en EC2
- SSH
- Procesos
- Bash
- Usuarios
- Permisos
- Servicios
- Logs
- Recursos del sistema

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Administrar remotamente un servidor Linux.
- Analizar recursos de una instancia.
- Crear archivos y scripts Bash.
- Identificar procesos.
- Consultar logs.
- Trabajar con permisos.
- Comprender que EC2 entrega infraestructura, pero la administración del sistema sigue siendo responsabilidad del usuario.

---

# Conceptos que se trabajarán

- Shell.
- Bash.
- PID.
- Procesos.
- Servicios.
- Permisos.
- Logs.
- CPU.
- Memoria.
- Disco.
- SSH.

---

# Parte 1. Exploración del servidor

Ejecute:

```bash
pwd
ls -lah
uptime
df -h
free -h
ps aux
```

---

# Parte 2. Crear una estructura de trabajo

```bash
mkdir -p ~/ec2-lab/{scripts,logs,data}
cd ~/ec2-lab
```

Cree un archivo:

```bash
echo "EC2 laboratorio" > data/identificacion.txt
cat data/identificacion.txt
```

---

# Parte 3. Procesos

```bash
ps aux | head
ps aux | grep ssh
```

Analice:

- PID.
- Usuario.
- CPU.
- Memoria.
- Comando.

---

# Parte 4. Logs

En Ubuntu puede explorar:

```bash
sudo journalctl -n 30
sudo systemctl status ssh
```

---

# Parte 5. Bash

Cree `scripts/system_info.sh`:

```bash
#!/bin/bash

echo "Hostname: $(hostname)"
echo "Usuario: $(whoami)"
echo "Fecha: $(date)"
echo "Uptime:"
uptime

echo "Disco:"
df -h /

echo "Memoria:"
free -h
```

Dé permisos:

```bash
chmod +x scripts/system_info.sh
```

Ejecute:

```bash
./scripts/system_info.sh
```

---

# Parte 6. Reto

Modifique el script para recibir un argumento:

```bash
./system_info.sh /var/log
```

Debe reportar información relacionada con el filesystem donde se encuentra el argumento.

---

# Entregable

- Script `system_info.sh`.
- Evidencias de ejecución.
- Resumen de los recursos observados.
- Explicación de tres procesos encontrados en la instancia.
