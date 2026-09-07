# Tutorial: Despliegue de FastAPI en AWS EC2 con Terraform

## 1. La Arquitectura

No solo lanzaremos un servidor; lo haremos de forma segura:
- **EC2 Instance**: Nuestro servidor virtual (usaremos una `t3.micro` para estar en la capa gratuita).
- **Security Group**: El firewall que solo permitirá tráfico en los puertos necesarios (SSH y el puerto de la API).
- **IAM Role**: El que creamos en el tutorial anterior para que la EC2 pueda hablar con S3 y DynamoDB sin usar llaves configuradas a mano.

## 2. El Código de la API (main.py)
Usaremos FastAPI para integrar lo que aprendimos de S3 y DynamoDB.

```python
from fastapi import FastAPI, UploadFile, File
import boto3
import os

app = FastAPI(title="Ingeniería API v1")

# Inicialización de recursos (Boto3 usará el IAM Role de la EC2 automáticamente)
s3 = boto3.client('s3')
dynamo = boto3.resource('dynamodb', region_name='us-east-1')
table = dynamo.Table('GamesInventory')
BUCKET_NAME = os.getenv("BUCKET_NAME", "mi-academia-ingenieria-bucket-2024")

@app.get("/")
def health_check():
    return {"status": "online", "service": "EC2-API"}

@app.post("/games/")
def create_game(game_id: str, title: str, genre: str):
    table.put_item(Item={'game_id': game_id, 'title': title, 'genre': genre})
    return {"message": "Juego guardado en DynamoDB"}

@app.post("/upload/")
async def upload_document(file: UploadFile = File(...)):
    s3.upload_fileobj(file.file, BUCKET_NAME, file.filename)
    return {"message": f"Archivo {file.filename} subido a S3"}
```

## 3. Terraform: Provisionando el Servidor
Añadiremos esto a nuestro archivo main.tf para crear la infraestructura.
```hcl
# 1. Security Group: El Firewall
resource "aws_security_group" "api_sg" {
  name        = "api-security-group"
  description = "Permitir HTTP y SSH"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # En producción, usa tu IP específica
  }

  ingress {
    from_port   = 8000
    to_port     = 8000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 2. Perfil de Instancia (Para pasar el IAM Role a la EC2)
resource "aws_iam_instance_profile" "api_profile" {
  name = "api-instance-profile"
  role = aws_iam_role.api_role.name
}

# 3. La Instancia EC2
resource "aws_instance" "api_server" {
  ami           = "ami-0c101f26f147fa7fd" # Amazon Linux 2023 (Verifica el ID en tu región)
  instance_type = "t3.micro"
  
  security_groups      = [aws_security_group.api_sg.name]
  iam_instance_profile = aws_iam_instance_profile.api_profile.name

  # Script de inicio (User Data) para instalar todo automáticamente
  user_data = <<-EOF
              #!/bin/bash
              dnf update -y
              dnf install -y python3-pip git
              pip3 install fastapi uvicorn boto3 python-multipart
              # Aquí podrías clonar tu repo de git
              EOF

  tags = { Name = "FastAPI-Server" }
}
```

## 4. User Data vs Configuración Manual
Podemos entrar a la maquina EC2 mediante SSH con el fin de poder instalar cosas (este proceso es más enfocado para pruebas rápidas). Para despliegues reales usamos el User Data (el script arriba).
¿Por qué? Porque si el servidor falla, Terraform puede destruir este y crear uno nuevo que se autoconfigura en minutos sin intervención humana.

## 5. Cómo ponerlo en marcha
- **Ejecutar Terraform**: `terraform apply`.
- **Obtener la IP**: Terraform te dará la IP pública de la instancia.
- **Lanzar la API**:
    - **Entra por SSH**: `ssh -i tu-llave.pem ec2-user@IP-PUBLICA`
    - **Ejecuta la API**:
    ```bash
    uvicorn main:app --host 0.0.0.0 --port 8000
    ```
    Probar: Ve a http://IP-PUBLICA:8000/docs y verás el Swagger de FastAPI listo para probar S3 y DynamoDB.

**IMPORTANTE**: Ten en cuenta que si lanzas el proceso manualmente y cierras la terminal, la API morirá. En producción (o al desplegar a un ambiente), necesitamos que la API sea un demonio (background process) que se reinicie automáticamente si el servidor se reinicia o si el código falla. Para eso usamos *Systemd*.

### El "Por qué" de Systemctl (Systemd)
En Linux, systemd es el administrador de sistemas y servicios. Al crear un archivo .service, garantizamos:
- **Auto-start**: La API arranca sola al encender la EC2.
- **Resiliencia**: Si la API tiene un error y se cierra, Systemd la levanta de nuevo.
- **Logging**: Los errores se registran automáticamente en journalctl.

# Tutorial: Crear el daemon para la API

**Comandos**

- **Ver el estado**: `sudo systemctl status fastapi_app` (Para ver si está vivo).
- **Reiniciar**: `sudo systemctl restart fastapi_app` (Cuando suban cambios en el código).
- **Ver logs en tiempo real**: `journalctl -u fastapi_app -f` (Crucial para debuguear errores de la API).
- **Detener**: `sudo systemctl stop fastapi_app`.

## 1. Actualización del User Data (Automatización total)
Vamos a modificar el bloque de Terraform para que no solo instale las librerías, sino que también configure el servicio de una vez:

```hcl
user_data = <<-EOF
    #!/bin/bash
    dnf update -y
    dnf install -y python3-pip
    pip3 install fastapi uvicorn boto3 python-multipart

    # Creamos el directorio de la app
    mkdir -p /home/ec2-user/app
    
    # (Aquí deberías descargar tu código, por ahora creamos un main.py de prueba)
    cat <<EOT > /home/ec2-user/app/main.py
    from fastapi import FastAPI
    app = FastAPI()
    @app.get("/")
    def root(): return {"message": "API corriendo con Systemd"}
    EOT

    # CREACIÓN DEL ARCHIVO DE SERVICIO
    cat <<EOT > /etc/systemd/system/fastapi_app.service
    [Unit]
    Description=Gunicorn instance to serve FastAPI
    After=network.target

    [Service]
    User=ec2-user
    Group=ec2-user
    WorkingDirectory=/home/ec2-user/app
    # Ejecutamos uvicorn directamente
    ExecStart=/usr/local/bin/uvicorn main:app --host 0.0.0.0 --port 8000
    Restart=always

    [Install]
    WantedBy=multi-user.target
    EOT

    # RECARGAR Y ARRANCAR
    systemctl daemon-reload
    systemctl enable fastapi_app
    systemctl start fastapi_app
EOF
```

Este enfoque, tiene una pequeña mejora y es la extracción de ese código sh a un archivo y poder importarlo luego en Terraform. La razón? Separación de responsabilidades, reutilización e inyección dinámica de valores.

## 2. Crea el archivo de script (scripts/setup.sh)
Crea una carpeta llamada scripts y guarda allí tu configuración de Bash. Fíjate cómo usamos `${bucket_name}` para recibir datos desde Terraform.

```bash
#!/bin/bash
# scripts/setup.sh

dnf update -y
dnf install -y python3-pip
pip3 install fastapi uvicorn boto3 python-multipart

mkdir -p /home/ec2-user/app

# Creamos el archivo de servicio con la variable inyectada
cat <<EOT > /etc/systemd/system/fastapi_app.service
[Unit]
Description=FastAPI Service
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/app
# Pasamos el nombre del bucket como variable de entorno
Environment="BUCKET_NAME=${bucket_name}"
ExecStart=/usr/local/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload
systemctl enable fastapi_app
systemctl start fastapi_app
```

## 3. Importa el script en Terraform (main.tf)
Ahora, en tu recurso de la instancia EC2, llama al archivo usando templatefile. Esta función toma dos argumentos: la ruta del archivo y un mapa de variables.

```hcl
resource "aws_instance" "api_server" {
  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t3.micro"
  
  iam_instance_profile = aws_iam_instance_profile.api_profile.name
  vpc_security_group_ids = [aws_security_group.api_sg.id]

  # IMPORTACIÓN DEL SCRIPT
  user_data = templatefile("${path.module}/scripts/setup.sh", {
    bucket_name = aws_s3_bucket.data_storage.id
  })

  tags = { Name = "FastAPI-Server" }
}
```

## 4. ¿Por qué este enfoque?
- **Separación de responsabilidades**: El código Bash se edita en un entorno con resaltado de sintaxis para Bash, y el HCL de Terraform se queda limpio.
- **Inyección dinámica**: No tienes que escribir el nombre del bucket "a mano" en el script. Si cambias el nombre en Terraform, se actualiza automáticamente en el script de la EC2.
- **Reutilización**: Puedes usar el mismo script para diferentes entornos (Dev, Prod) pasando variables distintas.

>💡**Tip**: `user_data_replace_on_change`
> Por defecto, si cambias el script de user_data, Terraform no reinicia la instancia. Si quieres que Terraform destruya y recree la EC2 cada vez que modifiques tu script de configuración, añade esto a tu recurso:
>
>    ```
>        resource "aws_instance" "api_server" {
>        # ... otros campos ...
>        user_data_replace_on_change = true
>        }
>    ```

# Tutorial: API con FastAPI en un ambiente de producción

Para llevar una API a producción en una EC2, usar `uvicorn` a secas es como conducir un coche sin parachoques. Como developer, debe llevar a cabo la combinación Gunicorn + Uvicorn:

- **Gunicorn (El Manager)**: Se encarga de gestionar los procesos. Si un proceso muere, Gunicorn lo revive. Si hay mucho tráfico, reparte la carga.
- **Uvicorn (El Trabajador)**: Es el que realmente entiende las peticiones asíncronas de FastAPI.

## 1. Actualización de dependencias
Primero, asegúrate de que tu script de instalación (setup.sh) incluya gunicorn:
```bash
pip3 install fastapi uvicorn gunicorn boto3 python-multipart
```

## 2. El comando de ejecución (The Worker Model)
En lugar de lanzar uvicorn directamente, lanzamos gunicorn y le decimos que use la "clase de trabajador" de uvicorn. El comando profesional es:
```bash
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8000
```
- `-w 4`: Lanza 4 procesos independientes (workers). *Regla: Se suele usar (2 x núcleos de CPU) + 1*.
- `-k uvicorn.workers.UvicornWorker`: Le dice a Gunicorn que use Uvicorn para manejar la asincronía.
- `--bind 0.0.0.0:8000`: Abre el puerto para tráfico externo.

## 3. Actualización del archivo de sistema (scripts/setup.sh)
Así es como quedaría tu script de Terraform para configurar el servicio de forma profesional:
```bash
#!/bin/bash
# scripts/setup.sh

dnf update -y
dnf install -y python3-pip
pip3 install fastapi uvicorn gunicorn boto3 python-multipart

mkdir -p /home/ec2-user/app

# (Asumiendo que el código de la API ya está en la carpeta /app)

# CREACIÓN DEL SERVICIO SYSTEMD PROFESIONAL
cat <<EOT > /etc/systemd/system/fastapi_app.service
[Unit]
Description=Gunicorn instance to serve FastAPI
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/home/ec2-user/app
Environment="BUCKET_NAME=${bucket_name}"

# Ejecutamos con Gunicorn para mayor estabilidad y paralelismo
ExecStart=/usr/local/bin/gunicorn \
    --workers 3 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000 \
    main:app

# Si el proceso falla, reinicia en 5 segundos
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOT

systemctl daemon-reload
systemctl enable fastapi_app
systemctl start fastapi_app
```

## 4. ¿Por qué esto es mejor?
- **Zero Downtime (Casi)**: Si un worker de la API tiene un "memory leak" o un error crítico y se cierra, Gunicorn lo detecta y lanza uno nuevo instantáneamente mientras los otros 2 o 3 workers siguen atendiendo clientes.
- **Concurrencia**: Con 4 workers, tu API puede procesar físicamente 4 peticiones al mismo tiempo, aprovechando mejor los recursos de la instancia EC2.
- **Logs**: Systemd capturará los logs de Gunicorn, permitiéndote ver qué worker falló y por qué usando `journalctl -u fastapi_app`.

>💡**Tip**: El archivo `requirements.txt`
> En lugar de hacer pip install de cada cosa en el script, enséñales a crear un archivo `requirements.txt`. El script de Terraform debería hacer:
> `pip3 install -r /home/ec2-user/app/requirements.txt`.
> Esto hace que el despliegue sea mucho más limpio y fácil de mantener.

Y surge una duda para las personas experimentadas en Python, que es solo uvicorn, no es suficiente? Mi respuesta corta es: Para desarrollo sí, para producción no.

Es una pregunta clásica. Si uvicorn ya es un servidor web, ¿para qué añadir la complejidad de gunicorn? Aquí te explico las razones técnicas por las que en el mundo real no lo hacemos así:

1. **El problema del "Proceso Único" (Single Point of Failure)**: `uvicorn` corre en un solo proceso. Si tu API tiene un error crítico (un segmentation fault, una fuga de memoria o un bug que bloquee el event loop), toda tu API muere y el servicio queda fuera de línea hasta que alguien (o systemd) lo reinicie.
    - **Con Gunicorn**: Actúa como un "proceder de procesos" (Process Manager). Si lanzas 4 workers y uno muere, Gunicorn lo detecta y lo reemplaza en milisegundos. Mientras tanto, los otros 3 workers siguen atendiendo a los usuarios. Tu API nunca deja de responder.
2. **Aprovechamiento del Hardware (Multi-core)**: Las instancias EC2 (excepto las más pequeñas) suelen tener más de un núcleo de CPU.
    - **Uvicorn solo**: Solo puede usar un núcleo a la vez, sin importar cuántos tengas.
    - **Gunicorn + Uvicorn**: Te permite lanzar un worker por cada núcleo. Si tienes una instancia con 2 CPUs, puedes procesar el doble de tráfico real al mismo tiempo.
3. **Manejo de Peticiones "Zombis"**: A veces, una petición puede quedarse "colgada" (por ejemplo, una conexión a base de datos que nunca responde).
    - `uvicorn` podría quedarse esperando indefinidamente.
    - `gunicorn` tiene un sistema de Heartbeat y Timeouts. Si un worker no responde en un tiempo determinado (por defecto 30 seg), Gunicorn asume que está "muerto", lo mata y lanza uno nuevo. Esto evita que tu servidor se llene de procesos basura.
4. **Estabilidad en la Red**: Gunicorn es mucho más maduro manejando conexiones TCP imperfectas, ataques lentos (slowloris) y ráfagas de tráfico repentinas. Uvicorn es extremadamente rápido (es su fuerte), pero delega la robustez de la gestión de procesos a Gunicorn.

**¿Cuándo SÍ es suficiente usar solo Uvicorn?**
Puedes usar solo uvicorn en:
- **Entornos de Desarrollo**: Tu propia laptop.
- **AWS Lambda**: Porque ahí AWS se encarga de la escalabilidad y cada ejecución es independiente.
- **Contenedores (Docker/ECS) muy pequeños**: Si limitas tu contenedor a exactamente 1 CPU y dejas que la orquestación (como Kubernetes o ECS) maneje los reinicios.


| Característica | Uvicorn solo | Gunicorn + Uvicorn |
| :--- | :--- | :--- |
| **Velocidad** | ⚡ Máxima | ⚡ Máxima |
| **Resiliencia** | ❌ Si falla el proceso, la API cae. | ✅ Si un worker falla, el manager lo revive. |
| **Uso de CPU** | ❌ Solo usa 1 núcleo de la EC2. | ✅ Puede usar todos los núcleos (Multi-worker). |
| **Uso ideal**	| Desarrollo / Lambdas / Docker. | Producción en Servidores (EC2). |





