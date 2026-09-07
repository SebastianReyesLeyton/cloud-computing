# Tutorial: Infraestructura como Código con Terraform

## 1. ¿Qué es Terraform y por qué lo usamos?
Terraform es una herramienta de HashiCorp que permite definir recursos de infraestructura en archivos de configuración legibles por humanos.

### Conceptos Clave:
- **Declarativo**: Tú le dices a Terraform "quiero un bucket", no "ejecuta el comando para crear un bucket". Terraform se encarga del cómo.
- **Estado (State)**: Terraform guarda un archivo `terraform.tfstate` que es la "fuente de la verdad". Sabe qué hay en la nube vs. qué hay en tu código.
- **Plan**: Antes de aplicar cambios, te muestra qué va a crear, modificar o destruir.

## 2. Estructura de un proyecto Terraform

Un proyecto estándar se divide en:

- **main.tf**: Lógica principal de los recursos.
- **variables.tf**: Definición de variables.
- **outputs.tf**: Datos que queremos ver al terminar (ej. la URL del bucket).
- **providers.tf**: Configuración de conexión (AWS, Azure, etc.).

## 3. Implementación: S3, DynamoDB e IAM
### A. Proveedor y Variables
```hcl
# providers.tf
provider "aws" {
  region = "us-east-1"
}
```

### B. El Principio del Menor Privilegio (IAM)
Esta es la regla de oro: Un servicio solo debe tener permiso para lo que necesita y nada más.

- ❌ No des AdministratorAccess.
- ❌ No des S3FullAccess.
- ✅ Da permiso solo al Bucket específico y a las acciones específicas (`s3:PutObject`, `s3:GetObject`).

### C. Creación de Roles y Políticas
#### 1. Rol para la API (Acceso a S3 y DynamoDB)
```hcl
# IAM Role para la API
resource "aws_iam_role" "api_role" {
  name = "api-service-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "://amazonaws.com" } # O ://amazonaws.com
    }]
  })
}

# Política de Menor Privilegio para la API
resource "aws_iam_policy" "api_policy" {
  name = "api-combined-policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action   = ["s3:PutObject", "s3:GetObject", "s3:ListBucket"]
        Effect   = "Allow"
        Resource = ["${aws_s3_bucket.data_storage.arn}", "${aws_s3_bucket.data_storage.arn}/*"]
      },
      {
        Action   = ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:UpdateItem", "dynamodb:Query"]
        Effect   = "Allow"
        Resource = [aws_dynamodb_table.games_table.arn]
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "api_attach" {
  role       = aws_iam_role.api_role.name
  policy_arn = aws_iam_policy.api_policy.arn
}
```

#### 2. Rol para el Worker (Solo S3)
```hcl
resource "aws_iam_role" "worker_role" {
  name = "worker-service-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "://amazonaws.com" }
    }]
  })
}

resource "aws_iam_policy" "worker_policy" {
  name = "worker-s3-only-policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action   = ["s3:GetObject"] # Solo lectura
      Effect   = "Allow"
      Resource = ["${aws_s3_bucket.data_storage.arn}/*"]
    }]
  })
}
```

### D. Recursos de Almacenamiento (S3 y Dynamo)
```hcl
# S3 Bucket
resource "aws_s3_bucket" "data_storage" {
  bucket = "mi-academia-ingenieria-bucket-2024" # Debe ser único globalmente
}

# DynamoDB Table
resource "aws_dynamodb_table" "games_table" {
  name           = "GamesInventory"
  billing_mode   = "PAY_PER_REQUEST" # Serverless mode
  hash_key       = "game_id"

  attribute {
    name = "game_id"
    type = "S" # String
  }
}
```

## 4. Comandos de Supervivencia
Para desplegar esto, tus estudiantes deben seguir este orden en la terminal:
- `terraform init`: Descarga los plugins necesarios.
- `terraform plan`: Obligatorio. Revisa qué va a pasar.
- `terraform apply`: Ejecuta los cambios.
- `terraform destroy`: Borra todo (úsalo solo al terminar la práctica para no gastar dinero).
