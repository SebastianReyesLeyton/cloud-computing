# Tutorial: Gestión de Objetos en Amazon S3 con Python

Recordemos que Amazon S3 (Simple Storage Service) no es un "sistema de carpetas", es un almacenamiento de objetos, el cual no tiene jerarquías reales, sino llaves (keys) que simulan rutas para los objetos que se cargan.

En este tutorial aprenderemos a manipular archivos (objetos) en la nube. Un ingeniero no sube archivos arrastrándolos a la consola; escribe scripts que lo hacen automáticamente.

## 1. Configuración Inicial
Asegúrate de tener boto3 instalado y tu sesión iniciada con el perfil que configuramos anteriormente.
```python
import boto3
from botocore.exceptions import ClientError

# Inicializamos el recurso S3
s3 = boto3.client('s3')
BUCKET_NAME = 'tu-nombre-de-bucket-unico' # S3 requiere nombres únicos globales
```

## 2. Operaciones CRUD en S3
### A. Encontrar (Listar) un Bucket
Antes de operar, verificamos si el bucket existe en nuestra cuenta.

```python
def find_bucket(name):
    response = s3.list_buckets()
    buckets = [b['Name'] for b in response['Buckets']]
    if name in buckets:
        print(f"✅ Bucket '{name}' encontrado.")
        return True
    print("❌ Bucket no encontrado.")
    return False
```

### B. Subir un archivo
Subiremos un archivo local indicando su nombre en el bucket (Key).
```python
def upload_file(file_name, object_name=None):
    if object_name is None:
        object_name = file_name
    s3.upload_file(file_name, BUCKET_NAME, object_name)
    print(f"📤 Archivo {object_name} subido.")
```

### C. Verificar si un archivo existe
No descargamos el archivo para saber si está ahí; usamos `head_object` para consultar solo los metadatos (ahorra costos y tiempo).
```python
def file_exists(object_key):
    try:
        s3.head_object(Bucket=BUCKET_NAME, Key=object_key)
        return True
    except ClientError:
        return False
```

### D. Obtener (Descargar) un archivo
```python
def download_file(object_key, download_path):
    s3.download_file(BUCKET_NAME, object_key, download_path)
    print(f"📥 Archivo {object_key} descargado en {download_path}.")
```

### E. Modificar un archivo
En S3, los objetos son inmutables. "Modificar" significa sobrescribir el objeto con una versión nueva usando la misma llave.
```python
def update_file(object_key, new_content):
    # Sobrescribe el contenido del archivo con un string
    s3.put_object(Body=new_content, Bucket=BUCKET_NAME, Key=object_key)
    print(f"🔄 Archivo {object_key} actualizado.")
```

### F. Eliminar un archivo
```python
def delete_file(object_key):
    s3.delete_object(Bucket=BUCKET_NAME, Key=object_key)
    print(f"🗑️ Archivo {object_key} eliminado.")
```

### G. Obtener (Extraer el contenido) de un archivo

Asi como puedes descargar el archivo alojado en s3 dentro de tu disco duro, tambien puedes obtener el contenido del objeto directamente en la memoria de tu programa Python para procesarlo. `get_object` es fundamental cuando tu componente o servicio necesita leer un archivo (como un JSON de configuración o un CSV) para procesar sus datos sin necesidad de guardarlo localmente. Este método nos devuelve un "Streaming Body". Es decir, una conexión abierta al archivo en AWS para que podamos leer su contenido.

```python
def get_object_content(object_key):
    try:
        # Solicitamos el objeto a AWS
        response = s3.get_object(Bucket=BUCKET_NAME, Key=object_key)
        
        # El contenido viene en el campo 'Body'
        # Debemos leerlo (.read()) y decodificarlo (usualmente 'utf-8')
        content = response['Body'].read().decode('utf-8')
        
        print(f"📖 Contenido de {object_key} leído con éxito.")
        return content
        
    except ClientError as e:
        print(f"❌ Error al obtener el objeto: {e}")
        return None
```

## 3. URLs Firmadas (Presigned URLs)
### ¿Qué es y por qué usarla?
Por seguridad, tus buckets deben ser privados. Si quieres que un usuario descargue un archivo, no debes hacer el bucket público. Por ello, surgió una técnica denominada **Presigned URL** es una URL temporal generada con tus credenciales que le da permiso a alguien de ver un objeto específico por un tiempo limitado (ej. 5 minutos). Es como un "pase de invitado" que caduca.

**Generación de URL firmada:**

```python
def generate_presigned_url(object_key, expiration=3600):
    try:
        response = s3.generate_presigned_url('get_object',
                                            Params={'Bucket': BUCKET_NAME,
                                                    'Key': object_key},
                                            ExpiresIn=expiration)
        return response
    except ClientError as e:
        print(e)
        return None

# Uso:
url = generate_presigned_url("reporte.pdf")
print(f"🔗 URL temporal (válida por 1 hora): {url}")
```

## Actividad
- Construir un bucket
- Cargue un archivo .csv y json
- Crear un script de Python que descargue el archivo de S3, valide formato (dependiendo del tipo de archivo) y contenido del archivo. En caso tal de que el archivo no haga match con los tipos esperados, debe eliminar el archivo de esa ruta y colocarlo en la ruta: `fail_structure/<filename>`