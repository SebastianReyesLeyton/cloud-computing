# Taller 4 - Bash

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:
    - Bash

---

# Objetivos

Al finalizar este workshop el estudiante será capaz de:

- Comprender el funcionamiento de las funciones en Bash.
- Utilizar pipes para combinar comandos.
- Analizar procesos activos en Linux.
- Buscar información utilizando filtros.
- Procesar archivos de texto y JSON desde la terminal.
- Resolver problemas comunes de administración utilizando herramientas de consola.

---

# Conceptos que se trabajarán

- Funciones en Bash
- Pipes (|)
- Entrada y salida estándar
- Procesos en Linux
- PID
- Logs
- Formato JSON
- Filtrado de información
- Procesamiento de texto

---

# Comandos que se utilizarán

| Comando | Descripción |
|----------|-------------|
| grep | Buscar texto dentro de archivos |
| ps | Mostrar procesos |
| htop | Monitor interactivo de procesos |
| kill | Finalizar procesos |
| awk | Procesar columnas de texto |
| jq | Procesar archivos JSON |
| wc | Contar líneas, palabras y caracteres |
| cat | Mostrar archivos |
| echo | Imprimir texto |

---

# Parte 1. Funciones en Bash

## Ejemplo

```bash
saludar() {
    echo "Hola desde Bash"
}

saludar
```

### Actividad

1. Cree una función llamada `presentarse`.
2. La función debe imprimir su nombre y el programa académico.
3. Ejecute la función tres veces.

---

# Parte 2. Pipes

## Ejemplo

```bash
ls | wc -l
```

```bash
cat archivo.txt | grep error
```

### Actividad

Explique con sus propias palabras qué hace cada uno de los comandos anteriores.

---

# Parte 3. Investigación de comandos

Investigue los siguientes comandos y complete una tabla con:

- ¿Qué hace?
- Sintaxis.
- Ejemplo.
- Caso de uso.

Comandos:

- grep
- ps
- htop
- kill
- awk
- jq

---

# Parte 4. Ejercicio práctico 1
## Proceso sospechoso consumiendo recursos

### Contexto

Los usuarios reportan lentitud en un servidor Linux desplegado en la nube.

### Objetivo

Identificar el proceso responsable, analizar su consumo de recursos y finalizarlo correctamente.

### Actividades

1. Abra el monitor de procesos utilizando `htop`.
2. Identifique un proceso con alto consumo de CPU o memoria.
3. Registre:
   - PID.
   - Nombre.
   - %CPU.
   - %MEM.
4. Verifique el proceso utilizando:

```bash
ps aux
```

5. Filtre únicamente dicho proceso mediante `grep`.

6. Utilice `awk` para mostrar únicamente:

- Usuario
- PID
- CPU
- MEM

7. Finalice el proceso usando `kill`.

8. Verifique que el proceso ya no existe.

---

# Parte 5. Ejercicio práctico 2
## Análisis de logs JSON

### Contexto

Un microservicio genera registros en formato JSON y presenta múltiples errores HTTP 500.

### Archivo

Crear el archivo:

```text
logs.json
```

Contenido:

```json
{"level":"info","service":"api","status":200,"message":"OK"}
{"level":"error","service":"api","status":500,"message":"Database error"}
{"level":"error","service":"auth","status":500,"message":"Token invalid"}
{"level":"info","service":"auth","status":200,"message":"Login success"}
```

---

### Actividades

1. Mostrar únicamente las líneas con errores.

```bash
grep error logs.json
```

2. Mostrar únicamente:

- service
- status
- message

```bash
jq '{service,status,message}' logs.json
```

3. Mostrar únicamente los errores HTTP 500.

```bash
jq 'select(.status==500)' logs.json
```

4. Contar cuántos errores existen.

```bash
jq 'select(.status==500)' logs.json | wc -l
```

5. Identificar el servicio con mayor número de errores.

---

# Reto Integrador

Un servidor Linux presenta bajo rendimiento y múltiples errores registrados en archivos JSON.

Realice un procedimiento que permita:

- Identificar procesos problemáticos.
- Filtrar información relevante.
- Contar errores HTTP 500.
- Identificar el servicio afectado.
- Elaborar un breve informe con los hallazgos.

---

# Preguntas de reflexión

1. ¿Qué ventajas ofrecen los pipes?
2. ¿Cuándo utilizar grep en lugar de jq?
3. ¿Qué ventajas ofrece awk?
4. ¿Qué riesgos tiene finalizar procesos incorrectos con kill?
5. ¿Qué utilidad tiene el análisis de logs en ambientes Cloud?