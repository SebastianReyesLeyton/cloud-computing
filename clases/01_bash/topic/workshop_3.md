# Taller 3 - Bash

**Curso:** Computación en la Nube e Internet de las Cosas

**Temas**:
    - Bash

---

# Objetivos

- Automatizar tareas mediante scripts.
- Utilizar variables.
- Implementar estructuras de control.
- Crear funciones.
- Recorrer archivos mediante bucles.

---

# Conceptos

- Scripts Bash
- Shebang
- Variables
- Parámetros
- Condicionales
- Bucles
- Funciones
- Código de salida
- Variables especiales

---

# Comandos y elementos

| Elemento | Uso |
|-----------|-----|
| #!/bin/bash | Shebang |
| if | Condicional |
| case | Selección |
| for | Bucle |
| while | Bucle |
| read | Leer entrada |
| exit | Finalizar |
| test | Comparaciones |
| function | Funciones |
| $1 $2 | Parámetros |
| $? | Código de salida |
| $0 | Nombre del script |
| $# | Número de argumentos |

---

# Ejercicio 1

Crear

```bash
nano saludo.sh
```

Contenido

```bash
#!/bin/bash

echo "Hola Mundo"
```

Ejecutar

```bash
chmod +x saludo.sh
./saludo.sh
```

---

# Ejercicio 2

Variables

```bash
#!/bin/bash

nombre="Juan"

echo "Hola $nombre"
```

---

# Ejercicio 3

Entrada

```bash
#!/bin/bash

read -p "Ingrese su nombre: " nombre

echo "Bienvenido $nombre"
```

---

# Ejercicio 4

Condicional

```bash
if [ -f archivo.txt ]
then
    echo "Existe"
else
    echo "No existe"
fi
```

---

# Ejercicio 5

For

```bash
for i in {1..10}
do
    echo $i
done
```

---

# Ejercicio 6

While

```bash
contador=1

while [ $contador -le 5 ]
do
    echo $contador
    contador=$((contador+1))
done
```

---

# Ejercicio 7

Funciones

```bash
saludar(){

echo "Hola"

}

saludar
```

---

# Ejercicio 8

Parámetros

```bash
#!/bin/bash

echo "Primer parámetro: $1"

echo "Segundo parámetro: $2"
```

Ejecutar

```bash
./script.sh Juan Pérez
```

---

# Ejercicio 9

Código de salida

```bash
ls

echo $?
```

Intentar

```bash
ls inexistente

echo $?
```

---

# Ejercicio 10

Proyecto integrador

Construir un script que:

- Cree una carpeta con la fecha actual.
- Copie todos los archivos `.txt`.
- Cuente cuántos archivos fueron copiados.
- Genere un archivo `reporte.log`.
- Muestre un mensaje de éxito o error según el código de salida.

---

# Desafío Final

Desarrollar un script denominado `backup.sh` que:

1. Solicite el nombre del directorio.
2. Verifique que exista.
3. Cree una carpeta `backup`.
4. Comprima el contenido usando `tar`.
5. Agregue la fecha al nombre del archivo.
6. Registre la operación en un archivo `backup.log`.
7. Muestre un resumen con:
   - Fecha.
   - Cantidad de archivos respaldados.
   - Tamaño del respaldo.
   - Estado del proceso.

---

# Preguntas de reflexión

1. ¿Qué ventaja tienen los scripts frente a ejecutar comandos manualmente?
2. ¿Cuándo utilizar un `for` y cuándo un `while`?
3. ¿Qué representa `$?`?
4. ¿Por qué es importante validar la existencia de archivos antes de manipularlos?
5. ¿Cómo podrían integrarse estos scripts en procesos de automatización para servidores Linux, servicios en la nube o dispositivos IoT?