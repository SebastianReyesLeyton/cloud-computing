# Terraform - Guía de Adopción Profesional
Terraform es una herramienta de Infraestructura como Código (IaC) desarrollada por HashiCorp. Te permite definir, crear y modificar infraestructura en la nube de forma segura y predecible mediante archivos de configuración declarativos.

## 📘 Conceptos Fundamentales
A diferencia de la programación tradicional, Terraform usa un enfoque declarativo. No defines cómo construir algo (paso a paso), sino qué deseas que exista. Terraform se encarga de calcular el orden correcto y los cambios necesarios.

### Componentes Clave
- **Provider (Proveedor)**: Plugins que traducen el código de Terraform en APIs de proveedores de nube (AWS, Azure, GCP).
- **Resource (Recurso)**: El componente de infraestructura que deseas crear (una VPC, una base de datos RDS, un bucket S3).
- **Data Source (Origen de datos)**: Permite consultar información de recursos existentes fuera de tu código actual de Terraform.
- **Variables e Outputs**: Las variables inyectan dinamismo al código; los outputs exponen valores calculados (ej. la IP pública de un servidor).


### 📄 Mapa de Archivos en Terraform
Cuando trabajas en un proyecto de Terraform, te encontrarás con una estructura de archivos estandarizada. Cada extensión y nombre tiene un propósito específico que el motor de Terraform lee en un orden determinado.

#### 1. Archivos de Configuración (.tf)
Son los archivos principales donde escribes tu código en lenguaje HCL (HashiCorp Configuration Language). Terraform lee todos los archivos .tf de la carpeta actual y los combina en memoria; el nombre del archivo no afecta la ejecución, pero por convención profesional se dividen así:
- **providers.tf**: Define los proveedores (AWS, Azure) y sus versiones.
- **main.tf**: Contiene el bloque de recursos principales que vas a crear.
- **variables.tf**: Declara las variables de entrada, sus tipos y valores por defecto.
- **outputs.tf**: Define los datos que deseas mostrar en pantalla o exportar al finalizar el despliegue.

#### 2. Archivos de Asignación de Variables (.tfvars)
Mientras que en variables.tf defines qué variables existen, en los archivos .tfvars defines los valores reales de esas variables para un entorno específico.
- **terraform.tfvars**: Terraform lo lee automáticamente si existe. Se usa para definir valores globales.
- **entorno.tfvars (ej. dev.tfvars, prod.tfvars)**: Se usan para pasar valores específicos según el ambiente ejecutando:

```bash
terraform apply -var-file="prod.tfvars"
```

**⚠️ IMPORTANTE**: Si un archivo `.tfvars` contiene contraseñas o datos sensibles, agrégalo inmediatamente al archivo `.gitignore`.

#### 3. El Archivo de Bloqueo de Dependencias (.terraform.lock.hcl)
Este archivo lo genera automáticamente Terraform cuando ejecutas terraform init.

- **¿Para qué sirve?** Registra las versiones exactas y los hashes de seguridad de los proveedores (providers) de nube que se descargaron.
- **📌 Práctica de Producción**: SÍ debes subir este archivo a Git. Garantiza que todos los desarrolladores del equipo y los servidores de CI/CD usen exactamente las mismas versiones de los plugins, evitando fallos por actualizaciones sorpresa.

#### 4. La Carpeta de Inicialización (.terraform/)
Es una carpeta oculta que se crea al ejecutar `terraform init`, que contiene los binarios descargados de los proveedores y el código de los módulos remotos.
**⚠️ IMPORTANTE**: NUNCA subas esta carpeta a Git. Agrégala siempre a tu `.gitignore`. Pesa mucho y se regenera automáticamente en cualquier máquina ejecutando `terraform init`.

#### 5. Archivos de Estado (.tfstate y .tfstate.backup)
Como aprenderemos en la sección de gestión de estado, el archivo `terraform.tfstate` es la base de datos local que contiene el mapa de tu infraestructura real. Mientras que el archivo `.backup` es una copia de seguridad automática de la última versión estable del estado.
**⚠️ IMPORTANTE**: Ambos archivos DEBEN estar en tu `.gitignore` si trabajas con estado local. Si usas un backend remoto (S3), estos archivos locales no se generarán en tu máquina, lo cual es el escenario ideal.

## 🛠️ Instalación en tu Máquina Local
Para mantener un entorno limpio y evitar conflictos de versiones entre proyectos, la mejor práctica en la industria es usar un gestor de versiones como tfenv.

### Pasos para Linux y macOS

```bash
# 1. Instalar tfenv usando Homebrew (macOS) o clonando el repo (Linux)
brew install tfenv

# 2. Listar las versiones disponibles e instalar la versión estable deseada
tfenv list-remote
tfenv install 1.7.0

# 3. Definir la versión global en tu sistema
tfenv use 1.7.0

# 4. Verificar la instalación correcta
terraform --version
```

### Pasos para Windows

1. Descarga el binario oficial desde la página de descargas de Terraform.
2. Extrae el archivo terraform.exe.
3. Mueve el archivo a una carpeta dedicada (ej. C:\Terraform).
4. Agrega esa ruta a las Variables de Entorno del Sistema (PATH).

## 💾 Gestión de Estado (State) en Entornos Reales
El archivo terraform.tfstate es la base de datos de Terraform. Mapea tus archivos de código con los recursos reales que existen en AWS.

### El Peligro del Estado Local
Por defecto, Terraform guarda este archivo en tu máquina (local state). Nunca subas el archivo .tfstate a un repositorio Git. Contiene secretos en texto plano (como contraseñas de bases de datos) y causará conflictos masivos si dos desarrolladores ejecutan comandos al mismo tiempo.

**Solución Profesional: Remote State**

En proyectos reales, el estado se almacena de forma remota en un bucket de Amazon S3 (almacenamiento) y se bloquea usando una tabla de Amazon DynamoDB (evita escrituras simultáneas).
Crea un archivo llamado backend.tf con la siguiente estructura:

`providers.tf`
```hcl
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # Configuración del estado remoto profesional
  backend "s3" {
    bucket         = "nombre-unico-de-tu-bucket-tfstate"
    key            = "proyectos/app/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tabla-bloqueo-terraform"
    encrypt        = true
  }
}

provider "aws" {
  region = "us-east-1"
}
```

## 🏢 Uso de Workspaces (Espacios de Trabajo)
Los Workspaces te permiten usar el mismo código base de Terraform para gestionar múltiples entornos aislados (ej. dev, staging, prod) sin duplicar archivos. Cada workspace mantiene su propio archivo de estado independiente dentro del bucket S3 remoto.

### Comandos Esenciales de Workspaces

```bash
# Crear un nuevo espacio de trabajo para desarrollo
terraform workspace new dev

# Crear un espacio de trabajo para producción
terraform workspace new prod

# Listar todos los workspaces disponibles (* indica el activo)
terraform workspace list

# Cambiar entre entornos antes de aplicar cambios
terraform workspace select dev

# Lista los workspaces existentes y marca con un asterisco (*) el que está activo actualmente
terraform workspace show
```

## Aplicación Práctica en Código
Puedes usar la variable implícita `${terraform.workspace}` para nombrar dinámicamente tus recursos según el entorno:
```hcl
resource "aws_s3_bucket" "mi_bucket" {
  # El bucket se llamará "mi-app-data-dev" o "mi-app-data-prod" automáticamente
  bucket = "mi-app-data-${terraform.workspace}"

  tags = {
    Environment = terraform.workspace
    ManagedBy   = "Terraform"
  }
}
```

## 🔌 Variables y Outputs (Inyección y Extracción de Datos)
Para que el código de Terraform sea dinámico y reutilizable entre diferentes entornos, utilizamos Variables de Entrada (Inputs) y Valores de Salida (Outputs).

Imagínalo como una función en programación: las variables son los parámetros que le pasas a la función, y los outputs son el valor que la función retorna (return).

### 📥 1. Variables de Entrada (Input Variables)
Las variables te permiten evitar escribir valores fijos (hardcodear) en tu código. Se declaran habitualmente en el archivo variables.tf.

**Estructura de nombramiento para una Variable**: Debes declarar variables completas, especificando siempre el tipo y una descripción clara

```hcl
# archivo: variables.tf

variable "entorno" {
  type        = string
  description = "El nombre del entorno de despliegue (ej: dev, staging, prod)"
}

variable "instancias_count" {
  type        = number
  description = "Cantidad de servidores web a crear"
  default     = 2 # Si el usuario no provee un valor, se usará este por defecto
}

variable "permitir_trafico_publico" {
  type        = bool
  description = "Habilita o deshabilita el acceso público de internet"
  default     = false
}
```

#### ¿Cómo se usan en el código?
Para usar una variable dentro de tus recursos en el `main.tf`, utilizas el prefijo `var.`:
```hcl
# archivo: main.tf

resource "aws_instance" "servidor" {
  count         = var.instancias_count # Uso de la variable numérica
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server-${var.entorno}-${count.index}" # Interpolación de texto
  }
}
```


#### ¿Cómo se le pasan los valores a las variables? (Orden de Prioridad)
Terraform busca los valores de las variables en el siguiente orden estricto (la última opción sobreescribe a las anteriores):

- **Valores por defecto**: El atributo default dentro de variables.tf.
- **En el archivo terraform.tfvars**: Terraform lo lee automáticamente si existe en la carpeta.
- **En archivos flag -var-file**: Ideal para separar entornos (ej: terraform apply -var-file="prod.tfvars").
- **Variables de Entorno del Sistema**: Deben llevar el prefijo `TF_VAR_` (ej: `export TF_VAR_entorno="dev"`).

## 📤 2. Valores de Salida (Outputs)
Los Outputs se declaran en el archivo outputs.tf. Sirven para dos cosas fundamentales en producción:

- Mostrar datos útiles en la terminal al terminar un terraform apply (ej: la URL del balanceador de carga o la IP del servidor).
- Compartir datos entre diferentes estados de Terraform (usando terraform_remote_state).

**Ejemplo Práctico de Salidas**
```hcl
# archivo: outputs.tf

output "ip_publica_servidor" {
  value       = aws_instance.servidor[*].public_ip
  description = "Las direcciones IP públicas de los servidores web creados"
}

output "url_endpoint" {
  # Concatenación dinámica de strings para generar una URL lista para usar
  value       = "http://${aws_instance.servidor[0].public_dns}:8080"
  description = "URL principal para probar la aplicación"
}
```

Al finalizar la ejecución del comando `apply`, Terraform imprimirá en tu consola algo como esto:
```text
Outputs:

ip_publica_servidor = [
  "54.210.32.11",
  "54.210.32.12",
]
url_endpoint = "amazonaws.com"
```

## 💡 Pro Tips sobre Variables y Outputs

- **🛡️ Usa sensitive = true para datos confidenciales**: Si un output o una variable maneja contraseñas, tokens o llaves privadas, añade la propiedad sensitive = true. Esto evitará que Terraform imprima el valor en texto plano en la terminal o en los logs de tu sistema de CI/CD.
```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```
- **🧪 Implementa validaciones personalizadas**: No confíes ciegamente en lo que el usuario ingrese. Puedes forzar reglas de negocio directamente en tus variables usando bloques validation para atrapar errores antes de que lleguen a AWS:
```hcl
variable "tipo_instancia" {
  type = string
  validation {
    # Evalúa si el valor ingresado cumple con la condición
    condition     = contains(["t3.micro", "t3.small"], var.tipo_instancia)
    error_message = "Por seguridad y costos, solo se permiten instancias t3.micro o t3.small."
  }
}
```

### Uso de los outputs como entrada de otro resource

Para usar los outputs de un recurso como entrada de otro, debes entender cómo Terraform conecta la infraestructura. En producción, esto se maneja de dos formas: dentro del mismo archivo/módulo (referencias directas) o entre proyectos independientes (usando estados remotos).

#### 🔗 Escenario 1: Dentro del mismo proyecto (Referencias Directas)
Este es el caso más común. No necesitas exportar un output formal en outputs.tf para usar los datos entre recursos que viven en la misma carpeta. Terraform lee los atributos en tiempo real mediante la sintaxis: `<TIPO_RECURSO>.<NOMBRE_LOCAL>.<ATRIBUTO>`.

**Ejemplo Práctico: Conectar un Security Group a una Instancia EC2**

En este caso, la instancia EC2 necesita saber el ID del Security Group para poder asociarse a él.
```hcl
# archivo: main.tf

# 1. Creamos el Security Group primero
resource "aws_security_group" "permitir_web" {
  name        = "permitir_trafico_web"
  description = "Permite acceso HTTP"
  vpc_id      = "vpc-12345678"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 2. Creamos la Instancia EC2 y le pasamos el ID del recurso anterior
resource "aws_instance" "servidor_web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # AQUÍ USAMOS EL OUTPUT IMPLÍCITO DEL RECURSO ANTERIOR
  vpc_security_group_ids = [aws_security_group.permitir_web.id]

  tags = {
    Name = "WebServer"
  }
}
```

**🧠 ¿Qué pasa por detrás? (Gráfico de Dependencias)**

Al escribir `aws_security_group.permitir_web.id`, le estás diciendo a Terraform: "No puedes crear la instancia EC2 hasta que el Security Group esté completamente creado y AWS nos devuelva su ID". Terraform creará automáticamente un Gráfico de Dependencias Implicitas y ejecutará todo en el orden correcto.

#### 🏢 Escenario 2: Entre proyectos independientes (Remote State)
En entornos reales y profesionales, la red (VPC) la maneja el equipo de SysOps/DevOps en un repositorio, y los servidores de la aplicación viven en otro repositorio independiente.
Para que el proyecto de la aplicación pueda leer los outputs del proyecto de red, se utiliza el bloque de datos terraform_remote_state.

##### Paso 1: El Proyecto Base (Red/VPC) debe exponer el Output
En la carpeta de infraestructura de red, debes declarar explícitamente el output para que se guarde dentro del archivo `.tfstate` en S3.
```hcl
# repositorio: infra-redes / archivo: outputs.tf

output "vpc_id_produccion" {
  value       = aws_vpc.main.id
  description = "El ID de la VPC de producción para que lo usen otras apps"
}
```

##### Paso 2: El Proyecto Destino (Aplicación) lee ese Output
En la carpeta de tu aplicación, configuras un bloque data para conectarte al bucket de S3 del proyecto de redes y consumir su output.
```hcl
# repositorio: mi-app-backend / archivo: main.tf

# 1. Apuntamos al estado remoto del proyecto de redes
data "terraform_remote_state" "redes" {
  backend = "s3"

  config = {
    bucket = "nombre-de-tu-bucket-tfstate-global"
    key    = "proyectos/infra-redes/terraform.tfstate" # Ruta exacta en S3
    region = "us-east-1"
  }
}

# 2. Usamos el output en nuestro nuevo recurso
resource "aws_security_group" "sg_app" {
  name        = "sg-mi-app"
  
  # SINTAXIS OBLIGATORIA: data.terraform_remote_state.<NOMBRE_LOCAL>.outputs.<NOMBRE_OUTPUT>
  vpc_id      = data.terraform_remote_state.redes.outputs.vpc_id_produccion

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

#### 💡 Pro Tips
- **🕵️‍♂️ Usa terraform show para descubrir atributos**: Si no sabe qué datos expone un recurso como "output implícito" (ej. si debe usar .id, .arn o .name), dile que ejecute terraform show tras un despliegue exitoso. Esto listará absolutamente todos los atributos disponibles del recurso que se pueden mapear.
- **⛓️ Evita dependencias circulares**: Ocurre cuando el Recurso A necesita un output del Recurso B, pero el Recurso B necesita un output del Recurso A para crearse. Terraform arrojará un error de bucle (Cycle Error). Si esto pasa, la lógica está mal diseñada y se debe romper la dependencia creando un tercer recurso intermedio (como una regla de seguridad independiente aws_security_group_rule).
- **🛡️ Manejo de Outputs Vacíos con data**: Al usar terraform_remote_state, si el proyecto raíz (redes) no ha guardado cambios con un terraform apply previo, el proyecto de la app fallará al compilar porque el output no existe en S3. Asegúrate siempre de desplegar la arquitectura base antes de intentar consumirla en tus aplicaciones.

## Consejos adicionales

- **🔒 Bloquea siempre las versiones de Providers**: Las actualizaciones automáticas de los proveedores de nube rompen el código de infraestructura viejo. Utiliza el operador ~> para permitir parches menores de seguridad pero bloquear cambios mayores (ej. version = "~> 5.0").
- **🧪 Ejecuta terraform plan antes de aplicar**: Nunca uses terraform apply directamente. El comando plan genera un informe detallado de qué recursos se van a crear, modificar o destruir. Revísalo minuciosamente como si fuera un Code Review.
- **🛠️ Automatiza el formato con terraform fmt**: Antes de enviar tu código a producción o hacer un commit, ejecuta terraform fmt -recursive. Esto formatea automáticamente la indentación de todos tus archivos .tf garantizando un estándar limpio en el equipo.
- **🛑 Evita hardcodear secretos**: Jamás escribas contraseñas, tokens de API o llaves de AWS directamente en tus archivos de configuración. Utiliza variables de entorno (TF_VAR_db_password), AWS Secrets Manager o archivos .tfvars listados en tu .gitignore.