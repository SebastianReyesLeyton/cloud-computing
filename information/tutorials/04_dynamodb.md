# Tutorial: Dominando Amazon DynamoDB con Python

## 1. El "Por qué": ¿Qué es DynamoDB?
Amazon DynamoDB es una base de datos NoSQL de tipo Clave-Valor y Documento, totalmente gestionada (Serverless). 

### Conceptos clave:

- **Performance consistente**: Promete latencias de milisegundos de un solo dígito a cualquier escala.
- **Sin esquema (Schemaless)**: Cada fila (ítem) puede tener atributos diferentes, excepto por la Primary Key.
- **Escalabilidad**: No hay que elegir un tamaño de servidor. AWS maneja el tráfico por ti. 
- **La Primary Key (Clave Primaria) es sagrada**:
    - **Partition Key (PK)**: Determina en qué nodo físico se guardan los datos.
    - **Sort Key (SK)**: (Opcional) Permite ordenar los datos dentro de una misma Partition Key. 

## 2. Operaciones Principales con Boto3
Para este tutorial, imaginemos que estamos gestionando un inventario de Videojuegos.

### A. Inicialización

```python
import boto3
from botocore.exceptions import ClientError

# Usamos el recurso 'dynamodb' que es más amigable para Python que el 'client'
dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
table = dynamodb.Table('GamesInventory')
```

### B. Insertar un Ítem (put_item)
Si el ítem ya existe, lo sobrescribe completamente.

```python
def add_game(game_id, title, genre, rating):
    table.put_item(
        Item={
            'game_id': game_id,  # Partition Key
            'title': title,
            'genre': genre,
            'rating': rating
        }
    )
    print(f"🎮 Juego '{title}' añadido.")
```

### C. Obtener un Ítem (get_item)
Es la operación más rápida y barata. Requiere la clave primaria completa.

```python
def get_game(game_id):
    response = table.get_item(Key={'game_id': game_id})
    return response.get('Item', "Juego no encontrado")
```

### D. Actualizar un Ítem (update_item)
A diferencia de S3, aquí sí podemos modificar solo un atributo sin tocar el resto.

```python
def update_rating(game_id, new_rating):
    table.update_item(
        Key={'game_id': game_id},
        UpdateExpression="set rating = :r",
        ExpressionAttributeValues={':r': new_rating},
        ReturnValues="UPDATED_NEW"
    )
    print("⭐ Rating actualizado.")
```

### E. Eliminar un Ítem (delete_item)

```python
def delete_game(game_id):
    table.delete_item(Key={'game_id': game_id})
    print(f"🗑️ Juego {game_id} eliminado.")
```

## 3. Query vs Scan: El error que cuesta miles de dólares
NUNCA usar Scan si pueden usar Query.

- **Query (Eficiente)**: Busca usando la Partition Key. Es como ir directo al estante correcto en una biblioteca.
- **Scan (Costoso)**: Revisa todos los registros de la tabla uno por uno. Si tienes 1 millón de registros, pagas por leer el millón completo.

Ejemplo de Query:

```python
from boto3.dynamodb.conditions import Key

def query_by_id(game_id):
    # Solo busca en la partición específica
    response = table.query(
        KeyConditionExpression=Key('game_id').eq(game_id)
    )
    return response['Items']
```

## 4. Buenas prácticas
- **Nombres de Tablas**: Usa variables de entorno para los nombres de las tablas, no los escribas "a fuego" (hardcoded).
- **Manejo de Números**: DynamoDB devuelve los números como tipo Decimal de Python. A veces necesitarás convertirlos a `int` o `float` antes de usarlos en tu lógica.
- **Capacity Modes**:
    - **On-Demand**: Ideal para aprender (pagas solo por lo que usas).
    - **Provisioned**: Mejor para producción con tráfico estable (es más barato si sabes cuánto tráfico tendrás).

## Reto para los estudiantes:

- Creen un script que:
    - Inserte 5 juegos de diferentes géneros.
    - Use un `update_item` para cambiar el género de uno.
    - Use un `get_item` para mostrar el resultado.