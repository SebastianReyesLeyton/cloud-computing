# Guía Técnica: Configuración de Entorno Local para AWS (CLI & SDK)

Como ingenieros de sistemas, nuestra meta es la automatización. Para que una aplicación en Python o un script de terminal pueda hablar con AWS, necesitamos establecer un puente seguro. Este documento detalla cómo configurar ese puente utilizando credenciales de IAM.

## 1. El Concepto: ¿Dónde se guardan mis llaves?
Cuando configuramos AWS en nuestra máquina, el sistema crea una carpeta oculta llamada .aws. Dentro, existen dos archivos críticos:
- **credentials**: Guarda tu Access Key y Secret Access Key.
- **config**: Guarda la región (ej. us-east-1) y el formato de salida (ej. json).

Estos los podrás encontrar en la ruta `~/.aws/`

## 2. Configuración de la AWS CLI
Asumiendo que ya cuentas con tus llaves de acceso, sigue estos pasos:

### A. Configuración Estándar
Ejecuta el siguiente comando en tu terminal:

```bash
aws configure
```

**OJO**: Usa el código con precaución.

Completa los campos cuando se te soliciten:
- **AWS Access Key ID**: Pega tu Access Key.
- **AWS Secret Access Key**: Pega tu Secret Key.
- **Default region name**: us-east-1 (Región estándar para despliegues).
- **Default output format**: json (Estándar para que Python pueda parsear respuestas).

### B. Configuración con perfiles
En la industria, manejarás múltiples cuentas. Para evitar errores, usamos Named Profiles:

```bash
aws configure --profile <profile_name>
```

Donde:
- `<profile_name>`: Es el nombre que le quieres dar al perfil que vas a configurar de AWS. Ten en cuenta que los carácteres permitidos son: [A-Za-z0-9-_]

Esto permite separar las credenciales de tus proyectos personales de las del trabajo (o para gestionar los usuarios de diferentes cuentas).

## 3. Verificación de Conexión
Un ingeniero nunca asume que algo funciona; lo valida. Ejecuta este comando para confirmar que AWS reconoce tu identidad:

```bash
aws sts get-caller-identity
```

o si vas a usar un perfil en concreto, puedes usar estas 2 estrategias:

**Definir el profile en el comando**
```bash
aws sts get-caller-identity --profile <profile_name>
```

**Definir el perfil que vas a usar en esa terminar mediante variables de entorno**
```bash
export AWS_PROFILE=<profile_name>
aws sts get-caller-identity
```

Respuesta exitosa esperada:
```json
{
    "UserId": "AIDASAMPLEUSERID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/estudiante-ingenieria"
}
```

## 4. Integración con Python (Boto3)
Para que Python use estas llaves, necesitamos la librería oficial Boto3.
Instalación del entorno:
```bash
# Se recomienda usar un entorno virtual
python -m venv venv
source venv/bin/activate  # Linux/Mac
.\venv\Scripts\activate     # Windows

pip install boto3
```

### Script de validación (check_aws.py):
Este script probará si Python puede leer tus credenciales de la CLI para listar tus servicios.
```python
import boto3
from botocore.exceptions import NoCredentialsError

def test_aws_connection():
    try:
        # Inicializamos el cliente de S3
        s3 = boto3.client('s3')
        
        # Intentamos listar los buckets
        response = s3.list_buckets()
        
        print("✅ Conexión exitosa con AWS.")
        print(f"Buckets encontrados: {len(response['Buckets'])}")
        
    except NoCredentialsError:
        print("❌ Error: No se encontraron credenciales configuradas.")
    except Exception as e:
        print(f"❌ Error inesperado: {e}")

if __name__ == "__main__":
    test_aws_connection()
```

## 5. Reglas de Oro de Seguridad
- **Archivo de credenciales**: Jamás subas el archivo credentials o el script con las llaves escritas en código duro a GitHub. Si el componente de AWS que vas a subir requiere interacción con resources del mismo, esto lo debes gestionar mediante policies y roles.
- **Uso del profile en código**: Si usaste un perfil nombrado (--profile), en Python debes inicializarlo así:
    ```python
    session = boto3.Session(profile_name='academia-api').
    ```
    o simplemente definelo mediante la variable de entorno y el sistema automaticamente reconocerá esas variables. 
- **Usa siempre el principio de Mínimo Privilegio**: Dale a tus llaves solo los permisos que necesitan para la tarea actual.
