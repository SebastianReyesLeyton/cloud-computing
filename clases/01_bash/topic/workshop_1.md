# Taller 1 - Bash

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:
    - Bash

---

# Objetivos

Al finalizar este taller el estudiante será capaz de:

- Comprender qué es Bash y la terminal.
- Navegar por el sistema de archivos.
- Crear, copiar, mover y eliminar archivos y directorios.
- Visualizar el contenido de archivos.
- Utilizar ayuda integrada del sistema.

---

# Conceptos que se trabajarán

- Shell
- Terminal
- Bash
- Sistema de archivos Linux
- Directorio raíz (/)
- Directorio personal (~)
- Ruta absoluta
- Ruta relativa
- Variables de entorno básicas

---

# Comandos que se utilizarán

| Comando | Descripción |
|----------|-------------|
| pwd | Mostrar directorio actual |
| ls | Listar archivos |
| ls -l | Lista detallada |
| ls -a | Mostrar archivos ocultos |
| cd | Cambiar directorio |
| mkdir | Crear directorio |
| touch | Crear archivo vacío |
| cp | Copiar archivos |
| mv | Mover o renombrar |
| rm | Eliminar archivos |
| rm -r | Eliminar directorios |
| cat | Mostrar contenido |
| less | Visualizar archivos largos |
| clear | Limpiar pantalla |
| history | Historial |
| man | Manual |
| echo | Mostrar texto |

---

# Parte 1. Conociendo la terminal

## Ejercicio 1

Abra una terminal.

Ejecute:

```bash
pwd
```

**Preguntas**

1. ¿Qué significa `pwd`?
2. ¿Qué directorio aparece?

---

## Ejercicio 2

Liste el contenido.

```bash
ls
```

Luego:

```bash
ls -l
```

Finalmente:

```bash
ls -la
```

**Preguntas**

- ¿Qué diferencia observa?
- ¿Qué son los archivos ocultos?

---

# Parte 2. Navegación

Crear el siguiente árbol:

```text
bash_basico
├── documentos
├── scripts
└── datos
```

Comandos sugeridos:

```bash
mkdir bash_basico
cd bash_basico
mkdir documentos scripts datos
```

Verificar con

```bash
ls
```

---

# Parte 3. Creación de archivos

Crear los siguientes archivos.

```bash
touch documentos/notas.txt

touch scripts/script1.sh

touch datos/datos.csv
```

Verificar con

```bash
ls -R
```

---

# Parte 4. Copiar y mover

Copiar

```bash
cp documentos/notas.txt datos/
```

Mover

```bash
mv datos/notas.txt documentos/notas_copia.txt
```

Renombrar

```bash
mv documentos/notas.txt documentos/apuntes.txt
```

---

# Parte 5. Visualizar contenido

Escribir texto

```bash
echo "Hola Bash" > documentos/apuntes.txt
```

Leer

```bash
cat documentos/apuntes.txt
```

Abrir

```bash
less documentos/apuntes.txt
```

---

# Parte 6. Eliminación

Eliminar

```bash
rm documentos/apuntes.txt
```

Eliminar directorio

```bash
rm -r datos
```

---

# Parte 7. Ayuda

Consultar ayuda

```bash
man ls
```

Consultar historial

```bash
history
```

---

# Reto Final

Construya la siguiente estructura.

```text
proyecto
├── codigo
│   ├── main.py
│   └── util.py
├── datos
│   ├── clientes.csv
│   └── ventas.csv
└── documentos
    └── informe.md
```

Después:

- Copiar `ventas.csv` a documentos.
- Renombrarlo como `ventas_backup.csv`.
- Mostrar el árbol creado usando `ls -R`.

---

# Preguntas de reflexión

1. ¿Cuál es la diferencia entre una ruta absoluta y una relativa?
2. ¿Qué hace `cd ..`?
3. ¿Para qué sirve `man`?
4. ¿Qué riesgo tiene `rm -r`?