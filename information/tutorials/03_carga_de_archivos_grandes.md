# Tutorial: Optimización de Cargas Pesadas en S3 (Multipart Upload)

Cuando un archivo supera los 100 MB, la mejor práctica es dividirlo en partes más pequeñas, subirlas en paralelo y dejar que S3 las reensamble al final.

## 1. El Concepto: ¿Por qué Multipart?

- **Resiliencia**: Si falla la subida de una parte, solo reintentas esa parte, no todo el archivo.
- **Velocidad**: Podemos subir varias partes al mismo tiempo (paralelismo).
- **Límite**: S3 permite archivos de hasta 5 TB, pero una sola operación PUT solo llega a 5 GB. El multipart es obligatorio para archivos mayores a 5 GB.

## 2. La forma automática (S3 Transfer Manager)

Boto3 incluye un gestor de transferencias que decide automáticamente cuándo usar multipart basándose en el tamaño del archivo. Es la opción recomendada para la mayoría de los casos.

```python
import boto3
import os
from boto3.s3.transfer import TransferConfig

s3 = boto3.client('s3')

def upload_large_file(file_path, bucket, object_name):
    # Configuración del umbral: 
    # Si el archivo mide más de 20MB, usa Multipart con partes de 20MB cada una.
    GB = 1024 ** 3
    config = TransferConfig(
        multipart_threshold=20 * 1024 * 1024, # 20MB
        max_concurrency=10,                    # Hilos en paralelo
        use_threads=True
    )
    
    try:
        print(f"🚀 Iniciando subida de {file_path}...")
        s3.upload_file(file_path, bucket, object_name, Config=config)
        print("✅ Subida completada con éxito.")
    except Exception as e:
        print(f"❌ Error: {e}")

# upload_large_file('video_4k_pesado.mp4', 'mi-bucket-multimedia', 'videos/video.mp4')
```

## 3. La forma manual (Control total)
A veces necesitas control absoluto (por ejemplo, para pausar y reanudar subidas). El flujo es: Inicio -> Subida de Partes -> Finalización.

```python
def manual_multipart_upload(file_path, bucket, key):
    file_size = os.path.getsize(file_path)
    part_size = 10 * 1024 * 1024  # 10 MB por parte
    
    # 1. Iniciar la subida multipart
    mpu = s3.create_multipart_upload(Bucket=bucket, Key=key)
    upload_id = mpu['UploadId']
    parts = []

    try:
        with open(file_path, 'rb') as f:
            part_number = 1
            while True:
                data = f.read(part_size)
                if not data:
                    break
                
                # 2. Subir cada parte
                print(f"📦 Subiendo parte {part_number}...")
                part = s3.upload_part(
                    Body=data, Bucket=bucket, Key=key,
                    PartNumber=part_number, UploadId=upload_id
                )
                
                parts.append({'PartNumber': part_number, 'ETag': part['ETag']})
                part_number += 1

        # 3. Finalizar la subida (reensamblaje)
        s3.complete_multipart_upload(
            Bucket=bucket, Key=key, UploadId=upload_id,
            MultipartUpload={'Parts': parts}
        )
        print("🏁 Archivo reensamblado en S3.")

    except Exception as e:
        # Si algo falla, es vital abortar para no generar costos de almacenamiento fantasma
        s3.abort_multipart_upload(Bucket=bucket, Key=key, UploadId=upload_id)
        print(f"⚠️ Subida abortada debido a error: {e}")
```

## 4. El "Costo Fantasma"
Cuando una subida multipart falla y no se llama a abort_multipart_upload, las partes que alcanzaron a subirse se quedan en S3 "flotando". No las ves en la consola, pero AWS te las cobra.

**Solución Profesional**: Configura una Lifecycle Policy en el bucket de S3 para "Abort incomplete multipart uploads" después de 1 o 2 días. Así, AWS limpia los restos automáticamente por ti.

## Actividad:
- Busquen un archivo de más de 100MB en su equipo.
- Implementen el script con TransferConfig y de la forma manual.
- Intenten monitorear el uso de red para ver cómo suben los hilos en paralelo.
- Crear una API para poder hacer la carga de los archivos teniendo en cuenta los pasos en el flujo de carga del multipart.