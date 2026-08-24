# Taller 1 - Bash

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:
    - Bash
    - Comandos avanzados de consola
    - Pipes y redirecciones
    - Archivos `.sh`
    - Scripts con y sin argumentos


---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Comprender el funcionamiento de comandos Bash y su combinación mediante pipes.
- Analizar información de archivos de texto desde la terminal.
- Utilizar filtros para encontrar información relevante en logs.
- Procesar columnas de texto mediante `awk`.
- Crear archivos Bash ejecutables.
- Ejecutar scripts `.sh` mediante Bash y mediante permisos de ejecución.
- Crear scripts que funcionen sin argumentos.
- Crear scripts que reciban y procesen argumentos.
- Validar argumentos y archivos antes de procesarlos.

---

# Conceptos que se trabajarán

- Bash
- Shell
- Shebang (`#!/bin/bash`)
- Entrada y salida estándar
- Pipes (`|`)
- Redirecciones (`>`, `>>`)
- Composición de comandos
- Procesamiento de texto
- Logs
- Variables en Bash
- Argumentos posicionales (`$1`, `$2`, etc.)
- Cantidad de argumentos (`$#`)
- Todos los argumentos (`$@`)
- Permisos de ejecución
- Código de salida de comandos

---

# Comandos que se utilizarán

| Comando | Descripción |
|----------|-------------|
| `grep` | Buscar texto dentro de archivos |
| `awk` | Procesar columnas y patrones de texto |
| `sort` | Ordenar resultados |
| `uniq` | Agrupar resultados repetidos |
| `cut` | Extraer partes de una línea |
| `wc` | Contar líneas, palabras y caracteres |
| `cat` | Mostrar o crear archivos de texto |
| `echo` | Imprimir texto |
| `head` | Mostrar las primeras líneas |
| `tail` | Mostrar las últimas líneas |
| `ps` | Mostrar procesos |
| `df` | Mostrar uso de disco |
| `hostname` | Mostrar el nombre del equipo |
| `whoami` | Mostrar el usuario actual |
| `date` | Mostrar fecha y hora |
| `chmod` | Cambiar permisos de un archivo |

---

# Parte 1. Bash avanzado: análisis de logs

## Contexto

Suponga que usted es administrador de un servidor y debe analizar un archivo de eventos llamado `access.log`.

Crear el archivo:

```bash
cat > access.log <<'LOGEND'
2026-08-24 08:15:20 INFO user=ana action=login ip=192.168.1.10
2026-08-24 08:16:02 ERROR user=juan action=login ip=192.168.1.15
2026-08-24 08:16:18 INFO user=ana action=download ip=192.168.1.10
2026-08-24 08:17:33 WARN user=carlos action=login ip=192.168.1.22
2026-08-24 08:18:04 ERROR user=juan action=upload ip=192.168.1.15
2026-08-24 08:19:27 INFO user=ana action=logout ip=192.168.1.10
2026-08-24 08:20:11 ERROR user=juan action=login ip=192.168.1.15
LOGEND
```

## Actividad

Resolver los siguientes requerimientos utilizando únicamente herramientas de Bash.

### A. Detectar errores

Mostrar únicamente las líneas que tengan nivel `ERROR`.

```bash
grep 'ERROR' access.log
```

### B. Contar errores

```bash
grep 'ERROR' access.log | wc -l
```

### C. Identificar usuarios con errores

Utilizar `awk` para obtener el campo `user=` de las líneas con error.

```bash
grep 'ERROR' access.log | awk '{for(i=1;i<=NF;i++) if($i ~ /^user=/) print $i}'
```

### D. Contar errores por usuario

```bash
grep 'ERROR' access.log | awk '{for(i=1;i<=NF;i++) if($i ~ /^user=/) print $i}' | sort | uniq -c
```

### E. Identificar la IP más asociada a errores

Construir un pipeline que extraiga las IP de los eventos `ERROR`, las ordene y cuente sus repeticiones.

### F. Generar un reporte

Crear `reporte.txt` con:

1. Cantidad total de registros.
2. Cantidad de errores.
3. Cantidad de advertencias.
4. Usuario con más errores.
5. IP con más errores.

Debe utilizar pipes y redirecciones.

## Desafío

Construir un comando o conjunto de comandos que determine automáticamente si existe una situación potencialmente sospechosa cuando un mismo usuario tenga dos o más eventos `ERROR`.

---

# Parte 2. Ejecutar archivos Bash

## Creación de un script

Crear `saludo.sh`:

```bash
#!/bin/bash

echo "Hola desde mi primer script Bash"
echo "Usuario: $(whoami)"
echo "Directorio actual: $(pwd)"
```

## Ejecutarlo mediante Bash

```bash
bash saludo.sh
```

## Ejecutarlo directamente

```bash
chmod +x saludo.sh
./saludo.sh
```

## Preguntas

1. ¿Qué función cumple `#!/bin/bash`?
2. ¿Qué diferencia existe entre `bash saludo.sh` y `./saludo.sh`?
3. ¿Qué ocurre si el archivo no tiene permisos de ejecución?
4. ¿Qué información devuelve `$?` después de ejecutar un comando?

---

# Parte 3. Ejercicio: script sin argumentos

## Nombre

**Monitor básico del contenedor**

## Objetivo

Crear un archivo `monitor.sh` que se ejecute de esta forma:

```bash
./monitor.sh
```

No debe requerir argumentos.

## Requisitos

Debe mostrar:

1. Hostname.
2. Usuario actual.
3. Fecha y hora.
4. Directorio actual.
5. Espacio en disco.
6. Cantidad aproximada de procesos activos.
7. Un mensaje final indicando que el diagnóstico terminó correctamente.

Una estructura inicial puede ser:

```bash
#!/bin/bash

echo "===== MONITOR DEL CONTENEDOR ====="
echo "Hostname: $(hostname)"
echo "Usuario: $(whoami)"
echo "Fecha: $(date)"
echo "Directorio: $(pwd)"

echo "===== DISCO ====="
df -h

echo "===== PROCESOS ====="
ps aux | wc -l

echo "===== FIN DEL REPORTE ====="
```

## Extensión

Agregar una condición que muestre una advertencia cuando el uso de una partición supere un umbral definido por el estudiante.

---

# Parte 4. Ejercicio: script con argumentos

## Nombre

**Analizador de texto**

Crear `analiza.sh` y ejecutarlo así:

```bash
./analiza.sh access.log
```

## Requisitos

El script debe:

1. Verificar que se haya proporcionado al menos un argumento.
2. Verificar que el archivo exista.
3. Mostrar el nombre del archivo analizado.
4. Mostrar cantidad de líneas.
5. Mostrar cantidad de palabras.
6. Mostrar cantidad de caracteres.
7. Mostrar cantidad de líneas con `ERROR`.
8. Mostrar cantidad de líneas con `WARN`.

## Variables importantes

Primer argumento:

```bash
$1
```

Cantidad de argumentos:

```bash
$#
```

Todos los argumentos:

```bash
$@
```

## Casos que deben probarse

### Sin argumentos

```bash
./analiza.sh
```

Debe mostrar cómo utilizar correctamente el script.

### Archivo inexistente

```bash
./analiza.sh no_existe.log
```

Debe informar el error.

### Archivo válido

```bash
./analiza.sh access.log
```

Debe mostrar el análisis solicitado.

## Extensión avanzada

Permitir múltiples archivos:

```bash
./analiza.sh access.log otro.log eventos.log
```

El script debe presentar un resumen independiente para cada archivo.

---

# Entregables

- `access.log`
- `reporte.txt`
- `saludo.sh`
- `monitor.sh`
- `analiza.sh`
- Evidencia de ejecución de cada actividad.
